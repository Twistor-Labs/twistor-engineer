# Case studies

Each entry: context, what happened or what they did, numbers, lesson. Figures are the company's own reports. Durations marked "our arithmetic" are computed from the timestamps on the page.

## LLM Gateway: a one-second migration, a 48-minute outage (rules 1, 2, 13)
- Context: a deploy added columns to two tables, one of them the table holding organization settings. The change "does need a brief exclusive lock on the table".
- What happened: dashboard log queries had held a lock on the organization table for over seven hours. "The migration had no lock timeout, so instead of giving up, it waited", and new queries on the table queued behind it. Dashboard and API were down "for about 48 minutes, from 18:24 to 19:12 UTC".
- Made worse by: "Reverting the code did not release the database." "The database ran out of connection slots, and the rolled-back servers could not connect." Health checks killed and restarted the migrating server about every 75 seconds and healthy servers about every two minutes; the database was restarted at 19:07 UTC.
- Fix: "Each migration runs in its own transaction with a 5 second lock timeout and a statement timeout, and retries with backoff." Short statement and lock timeouts on log queries; per-service statement timeouts; page when connection use stays at or above 70% for 5 minutes.
- Lesson: "On any other day this migration would have finished within a second. With a lock timeout, it would have failed after a few seconds and left the dashboard and API running." [S9]

## LootLocker: source deleted before the target was checked (rule 9)
- Context: migration to a new database setup, cutover on 2026-04-28 at 10:34 BST.
- What happened: "~10:40: The source database was deleted, on the assumption that a rollback path was no longer needed." A temporary replication process had crashed on 2026-04-20 when credentials it used were rotated for an unrelated fix; it restarted, but batches in flight were never replayed. Writes from 2026-04-19 to 2026-04-20 were missing.
- Numbers: about 3 hours of degraded service; recovery over about 2 days. "Recovery was significantly complicated by the early deletion of the source database", because it had to be restored first.
- Fix: keep the source until post-cutover validation is complete; check replication is running and caught up right before cutover; review downstream impact of credential rotations on long-running service accounts; use maintenance windows for changes of similar risk.
- Lesson: a sync process that restarts cleanly can still have dropped data. Keep the way back until you have proved you do not need it. [S6]

## GitLab: backups that never worked (rule 15)
- Context: on 2017-01-31 an engineer trying to restore replication wiped the database directory on the primary, thinking it was the secondary; the secondary's data directory had already been wiped.
- What happened: "the backup procedure was using `pg_dump` 9.2, while our database is running PostgreSQL 9.6", so it "failed silently". "We were never aware of the backups failing, until it was too late." Recovery used a manual LVM snapshot taken "roughly 6 hours before the outage".
- Numbers: about 6 hours of data lost, "at least 5000 projects, 5000 comments, and roughly 700 users"; copying data back "took around 18 hours".
- Fix: "Automated testing of recovering PostgreSQL database backups (#1102)" and "Assign an owner for data durability (#1163)". Root cause in their words: "nobody was responsible for testing this procedure."
- Lesson: a backup is a restore you have not tried yet. Test the restore and name an owner. [S5]

## ImprovMX: a skipped manual step (rule 14)
- What happened: a database migration produced an error that took the API down from 04:58 to 05:14 ET, about 16 minutes (our arithmetic). Customers could not reach their accounts. A code change reverting the migration restored service.
- Root cause: "a gap in our deployment procedure which skipped a necessary manual step."
- Fix: updated documentation and "additional alert notifications during deploys".
- Lesson: a runbook step a human can skip will be skipped. Encode it in the pipeline and keep the revert ready. [S8]

## Clerk: lock contention from below the database (red flags)
- What happened: "a failed live migration of our Google Cloud SQL virtual machine" caused "increased disk latency, which in turn resulted in increased lock contention", then a full outage. Declared 15:57 UTC; "The issue resolved on its own" at 16:23 UTC. Moving reads to replicas "did not help"; enabling Origin Outage Mode kept session management working.
- Fix: "depend on traditional replica promotion for ALL Postgres database maintenance going forward", and ask the provider to pin the database. A further concern: internal 429s reached clients, to be fixed "so applications can accurately assess the state of Clerk's systems based on the error code." A Postgres major upgrade on 2026-01-17 via replica promotion went without incident.
- Lesson: lock contention is a symptom; on a managed database the cause can be the provider's maintenance. Prefer maintenance paths you control. [S7]

## Stripe: online migrations with dual writes (rules 6, 7, 8)
- Context: moving subscriptions off the Customer record into their own table, "hundreds of millions of Subscriptions objects".
- Did: four steps, "Dual writing to the existing and new tables", then moving all reads, then moving all writes, then "Removing old data that relies on the outdated data model." Backfill ran on Hadoop from database snapshots, off production. Reads were compared with GitHub's Scientist, alerting on mismatches, before switching.
- Number: "We never attempted to change more than a few hundred lines of code at one time."
- Lesson: every step is small and the old path stays live until the new one has matched it in production. [S2]

## Notion: sharding Postgres (rules 6, 8, 11)
- Context: one Postgres database where "the Postgres `VACUUM` process began to stall consistently", with transaction ID wraparound risk, which would stop all writes.
- Did: "480 logical shards evenly distributed across 32 physical databases", keyed by workspace ID, with limits of "500 GB per table and 10 TB per physical database". Steps: double-write via an audit log, backfill comparing record versions, verification with sampled UUID ranges and dark reads (built by different people), switch-over.
- Numbers: backfill "took around three days" on "all 96 CPUs" of an m5.24xlarge; switch-over took "five minutes of scheduled maintenance". "Introducing dark reads increased API latency".
- Lesson: "Shard earlier." Waiting until the database was heavily strained made every step riskier. [S4]

## Figma: logical before physical sharding (rules 10, 11)
- Context: database stack "grown almost 100x since 2020"; some tables "containing several terabytes and billions of rows", vacuum hurting reliability, write-heavy tables nearing the RDS IOPS limit.
- Did: separated logical sharding (views, routing in a DBProxy query layer) from physical sharding (moving data). Sharded "groups of related tables into colocations" that keep joins and transactions within one sharding key.
- Numbers: first table took roughly nine months, shipped September 2023, and "failed over with only ten seconds of partial availability on database primaries" with no impact on replicas. Views added "less than 10% in the worst cases" in load tests.
- Lesson: "Rolling back logical sharding when we found bugs was a simple configuration change." Physical rollback "requires more complex coordination". Put the reversible step first. [S3]

## Figma: PGKeeper connection pooler (rule 13)
- Context: hundreds of stateless DBProxy replicas in front of Postgres; "PgBouncer's single-threaded architecture imposed a ceiling on vertical scalability", it could not prioritize critical traffic, and it lacked safeguards against rapid connection creation and churn.
- Did: a shared pooler that rate-limits connection creation (token bucket) and destruction, warms pools before traffic is routed to an instance, uses a semaphore for admission control, and uses CoDel "to drop stale requests under sustained overload".
- Numbers: each Postgres pool sized to "about 100 connections"; "PGKeeper prevented more than 20 incidents that would otherwise have caused user-visible outages" in Q4 2025.
- Lesson: "Distributing a small, fixed-size connection pool across a large number of stateless DBProxy instances is not feasible", and "naive recovery after an overload incident can itself overwhelm the database". [S11]

## Zepto: oversized pools (rule 12)
- Context: a 10-minute delivery promise, where "even a single minute disruption is unaffordable".
- Found: each pod's pool hardcoded at 10x need (20 active plus 70 overflow), and Postgres `idle_session_timeout` of 5 minutes killing idle pooled connections.
- Did: pool max about 20% above observed peak, shorter connection timeout, idle connections capped at 5-10% of active. Next step: cut active connections from 16k to 5k, RDS Proxy under evaluation.
- Lesson: pool size times pod count is a database capacity number. [S10]

## GitHub: gh-ost (rule 5, MySQL)
- Context: schema changes daily, "sometimes multiple times a day", on MySQL tables from empty to "many hundreds of GB".
- Problem with trigger-based tools: "Lock contention is directly related to write concurrency on the master"; the master was sometimes "brought down by the load of the triggers", and triggers cannot be throttled or removed mid-run without data loss.
- Did: "`gh-ost` does not use triggers." It reads the binary log and writes with one connection; throttles on `Threads_running`, replica lag (`max-lag-millis`), a query or a flag file; settings change mid-run; cut-over can be postponed; `--test-on-replica` runs full migrations on replicas.
- Lesson: an online schema change must be pausable and its cut-over under operator control. [S1]
