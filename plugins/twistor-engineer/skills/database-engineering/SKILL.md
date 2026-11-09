---
name: database-engineering
description: 'Use when changing a live database safely or when it is choking on locks or connections. Triggers: "add a NOT NULL column to a huge table", "migration without downtime", "backfill millions of rows", "ALTER TABLE hung and took the API down", "lock contention", "lock_timeout", "statement_timeout", "CREATE INDEX blocking writes", "too many connections after we scaled", "connection pool exhausted", "cutover to the new database", "dual write", "restore drill", "when do we need to shard". Nouns: Postgres, MySQL, gh-ost, migration, backfill, expand and contract, pgbouncer, pool, replica, failover. Gives sourced instead-of rules, a step-by-step change plan, engine notes and a pre-ship checklist for database changes. Not for choosing a data store or designing partition keys (use system-design-scaling), tuning a slow query or hot path (use performance-engineering), or LLM-driven code migrations across a codebase (use dev-productivity-ai-coding), or a live outage in progress (use reliability-incident-response).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.2.0
  summary: 'Change a live database without an outage: lock and statement timeouts, batched backfills, expand and contract, verified cutovers, sane pools and restore drills.'
---

# Database Engineering

Most database outages in a migration are not exotic. A DDL statement waits on a lock with no timeout, a backfill runs as one giant update, the old store is deleted before anyone checks the new one, or a backup that never restored turns out to be empty. Every change below is small, reversible until the last step, and checked before the next. Rules come from GitHub, Stripe, Figma, Notion, GitLab, LootLocker, LLM Gateway, ImprovMX, Zepto and the Postgres docs. Company numbers are that company's claims.

## When to use / not

- Use: adding or changing columns, constraints or indexes on a table that serves traffic; planning a backfill; moving data to a new table, schema, store or shard; deciding when the old store can go.
- Use: lock waits, migrations that hung, "too many connections", pool exhaustion after scaling out, connection storms after a restart.
- Use: backup and restore checks, sharding triggers (when, not how to pick the key).
- Not: which database to pick or how to design a partition key (see system-design-scaling).
- Not: one slow query, index choice for a hot path, query plans (see performance-engineering).
- Not: rewriting application code across a repo with an LLM (see dev-productivity-ai-coding).
- Engine scope: rules marked (Postgres) or (MySQL) apply to that engine only. Do not carry Postgres lock advice to a document or key-value store as if it were universal; say what is unknown for that engine.

## Rules

1. **Instead of** running DDL with no lock timeout, **do** set a short `lock_timeout` on the migration session (Postgres) and retry with backoff when it fires. **Why:** LLM Gateway's column add waited behind a long-held lock, queued every query on the table and caused a 48-minute outage; with a lock timeout it "would have failed after a few seconds", and they now use 5 seconds [S9][S12]
2. **Instead of** a global `statement_timeout` in postgresql.conf or none at all, **do** set statement and idle-in-transaction timeouts per role or service, sized to its workload (Postgres). **Why:** LLM Gateway's dashboard log queries held a lock on a core table for over seven hours, and the Postgres docs warn that setting these timeouts in postgresql.conf affects all sessions [S9][S12]
3. **Instead of** `ADD COLUMN ... NOT NULL` with a volatile default or a direct `SET NOT NULL` on a large table, **do** add the column with a constant default or nullable, backfill, add `CHECK (col IS NOT NULL) NOT VALID`, `VALIDATE` it, then `SET NOT NULL` (Postgres). **Why:** ALTER TABLE takes ACCESS EXCLUSIVE, which blocks even reads, and SET NOT NULL scans the whole table unless a valid CHECK proves no NULLs, while VALIDATE needs only SHARE UPDATE EXCLUSIVE [S13][S15]
4. **Instead of** a plain `CREATE INDEX` on a table taking writes, **do** use `CREATE INDEX CONCURRENTLY` outside a transaction block and check for an INVALID index afterwards (Postgres). **Why:** a plain build blocks inserts, updates and deletes until it finishes, and a failed concurrent build leaves an invalid index that still costs write overhead until dropped and retried [S14]
5. **Instead of** trigger-based online schema change on a busy MySQL primary, **do** use a triggerless, binlog-based tool such as gh-ost that throttles on replica lag and lets you postpone the cut-over (MySQL). **Why:** GitHub found triggers add lock contention and load on the master and cannot be paused, while gh-ost throttles on lag and load and can be tuned mid-run [S1]
6. **Instead of** backfilling with one large UPDATE on the primary, **do** backfill in small throttled batches (or offline from a snapshot) that never overwrite a newer write. **Why:** Stripe ran its backfill from database snapshots on Hadoop to keep load off production, and Notion's backfill compared record versions and still took about three days on 96 CPUs [S2][S4]
7. **Instead of** switching a data model or store in one deploy, **do** expand and contract: dual write, move reads, move writes, then remove the old data. **Why:** Stripe moved hundreds of millions of subscription objects this way and never changed more than a few hundred lines of code at a time [S2]
8. **Instead of** trusting a backfill because it finished, **do** compare old and new before moving reads, with shadow reads or sampled key ranges, and have someone other than the migration author build the check. **Why:** Stripe compared read paths with Scientist and alerted on mismatches, and Notion ran dark reads and sampled UUID ranges with verification built by different people [S2][S4]
9. **Instead of** deleting the source store soon after cutover, **do** keep it until post-cutover validation passes, and confirm right before cutover that every sync or replication process is running and caught up. **Why:** LootLocker deleted its source about 6 minutes after cutover, then found missing writes from a sync process that a credential rotation had crashed, and the deletion delayed recovery [S6]
10. **Instead of** a one-way cutover, **do** stage it so the first step is reversible by config: route logically first, move data physically second. **Why:** Figma's rollback of logical sharding "was a simple configuration change", and its first physical shard failed over with ten seconds of partial availability on primaries [S3]
11. **Instead of** waiting to shard until the primary is strained, **do** start planning when table size, vacuum health or IOPS headroom near your limits (Postgres). **Why:** Notion's lesson was "Shard earlier" after VACUUM stalls and wraparound risk, and Figma moved when tables reached several terabytes and billions of rows and vacuums hurt reliability [S4][S3]
12. **Instead of** giving every pod a generous pool, **do** set pool max about 20% above observed peak and add up pool size times pod count against the database's connection limit. **Why:** Zepto found pools 10x oversized (20 active plus 70 overflow per pod) and set out to cut 16k active Postgres connections to 5k [S10]
13. **Instead of** letting many stateless app replicas each open their own connections, **do** put a shared pooler in front of the database that caps concurrency and rate-limits new connections. **Why:** Figma sizes each Postgres pool to about 100 connections and found spreading it across many replicas "is not feasible", and LLM Gateway's restarted servers exhausted connection slots [S11][S9]
14. **Instead of** relying on manual steps in a migration runbook, **do** put every step in the deploy pipeline, alert during deploys and keep a tested revert. **Why:** ImprovMX's API went down for about 16 minutes when its deploy procedure "skipped a necessary manual step", and reverting the migration restored it [S8]
15. **Instead of** assuming backups work because a job runs, **do** run automated restore drills to a scratch host on a schedule and give data durability a named owner. **Why:** GitLab's pg_dump 9.2 failed silently against Postgres 9.6, so it lost about 6 hours of data and needed about 18 hours to copy a snapshot back [S5]

## Answer shape

1. **Engine and size**: engine and version, table rows and size, write rate, how long a full scan or rewrite would take. Say which facts are assumed.
2. **What locks or loads**: for each step, the lock it takes, what it blocks, and how long it could hold (cite rules 1 to 5).
3. **Step plan**: numbered, each step independently deployable and reversible until the last (expand, backfill, verify, switch, contract). Include timeouts and batch size.
4. **Verification gate**: what is compared before reads move, before writes move, and before the old store is removed (rules 8, 9).
5. **Rollback**: how to undo each step and what stops being undoable.
6. **Capacity side effects**: connections, replica lag, IOPS and vacuum during the change (rules 6, 12, 13).
7. **Evidence**: queries or dashboards that prove each step worked.

## Red flags

- "It is just adding a column, it takes a second." -> It takes a second only if it gets the lock. Set a lock timeout (rule 1).
- "We will run the backfill in one transaction so it is atomic." -> One giant update holds locks and loads the primary. Batch it (rule 6).
- "Rolling back the deploy will fix it." -> It did not release the waiting sessions at LLM Gateway [S9]. Cancel the blocked migration in the database.
- "Cutover went fine, drop the old DB to save cost." -> Validate first (rule 9).
- "Move reads to replicas, that will relieve it." -> It did not help Clerk when lock contention came from disk latency on the primary [S7].
- "More app pods means more throughput." -> Each pod brings its own pool. Sum them (rule 12).
- "Backups run nightly, we are covered." -> Only a restore proves it (rule 15).
- "Same advice works on Mongo / DynamoDB." -> These rules name their engine. Say what you do not know for other engines.

## Verification

- Show the session settings used by the migration (`lock_timeout`, `statement_timeout`) and a test where the DDL fails fast against a held lock.
- For each DDL step, name the lock mode from the docs and the measured time it held on a production-size copy.
- Show backfill batch size, throttle signal (replica lag, CPU) and a progress query.
- Show the comparison result (mismatch count) before reads move and again before the old store is removed.
- Show `SELECT indexrelid::regclass FROM pg_index WHERE NOT indisvalid` returns nothing after a concurrent index build (Postgres).
- Show peak connections per pod times pod count against `max_connections` or the pooler limit.
- Show the date and duration of the last successful restore drill and who owns it.

## References

| File | Read when |
|---|---|
| references/case-studies.md | you need the full story behind a rule (GitHub, Stripe, Figma, Notion, GitLab, LootLocker, Clerk, ImprovMX, LLM Gateway, Zepto) |
| references/checklist.md | reviewing a migration, schema file or pool config before ship (engineer-review and design review use it) |
| references/engine-notes.md | you need the exact lock mode or setting for Postgres, or the MySQL gh-ost notes |
| references/sources.md | citing a figure or date |
