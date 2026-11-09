# Pre-ship checklist: database changes

For a migration, schema file, backfill job, cutover plan or pool config. Tick each item or write why it does not apply. Numbers point to SKILL.md rules. engineer-review and design review read this file; cite the item and rule in a finding.

## Every schema change
- [ ] Engine and version named; table size and row count known or estimated. (answer shape)
- [ ] Lock mode for each statement written down from the engine's docs, and what it blocks. (3, 4)
- [ ] Migration session sets a short lock timeout and retries with backoff (Postgres `lock_timeout`). (1)
- [ ] Migration session sets a statement timeout. Not set globally in postgresql.conf. (2)
- [ ] Long-running app queries and idle-in-transaction sessions have their own timeouts, so they cannot hold a lock for hours. (2)
- [ ] Each migration (except statements that cannot run in a transaction block, such as `CREATE INDEX CONCURRENTLY`) runs in its own transaction; no unrelated DDL bundled with it. (1, 4)
- [ ] All steps run from the deploy pipeline, no manual step in a runbook; alerts are on during the deploy. (14)
- [ ] A tested revert exists for each step, and the plan says which step is the first one that cannot be undone. (10, 14)
- [ ] Rolled-back app code does not count as rollback for the database: the plan says how to cancel a blocked or waiting migration session. (1)

## Columns and constraints (Postgres)
- [ ] New columns use a constant default or are nullable; no volatile default on a large table. (3)
- [ ] NOT NULL on an existing large table goes CHECK NOT VALID, VALIDATE, then SET NOT NULL. (3)
- [ ] Other new constraints are added NOT VALID and validated in a separate step. (3)

## Indexes
- [ ] Postgres: `CREATE INDEX CONCURRENTLY`, not inside a transaction block. (4)
- [ ] After the build, no INVALID index remains; if one does, drop and retry. (4)
- [ ] MySQL: large-table changes use a triggerless online tool (gh-ost) with lag throttling and a postponed cut-over. (5)

## Backfills
- [ ] Runs in small batches with a throttle on replica lag or load, or offline from a snapshot. (6)
- [ ] Never overwrites a newer row: compares version or updated-at before writing. (6)
- [ ] Progress is measurable and the job can stop and resume. (6)
- [ ] Load on the primary and replica lag are watched while it runs. (6)

## Moving data (new table, store or shard)
- [ ] Expand and contract: dual write, move reads, move writes, remove old, each a separate deploy. (7)
- [ ] Old and new are compared (shadow reads, sampled key ranges) before reads move, and the mismatch count is zero or explained. (8)
- [ ] The comparison is built or reviewed by someone other than the migration author. (8)
- [ ] Right before cutover, every sync or replication process is confirmed running and caught up. (9)
- [ ] Credential rotations during the migration window are checked against long-running sync processes. (9)
- [ ] The source store is kept until post-cutover validation passes; its deletion is a separate, dated step. (9)
- [ ] First cutover step is reversible by config (logical routing before physical moves). (10)

## Connections and pools
- [ ] Pool max per pod is about 20% above observed peak, not a copied default. (12)
- [ ] Pool size times max pod count (including autoscaling and deploy surge) stays under the database or pooler limit. (12)
- [ ] Many stateless replicas share a pooler that caps concurrency, rather than each opening its own pool. (13)
- [ ] New connections are rate-limited so a mass restart cannot exhaust connection slots. (13)
- [ ] An alert fires on sustained high connection use. (13)

## Capacity and sharding
- [ ] Largest tables, vacuum health and IOPS headroom are tracked with a threshold that starts sharding planning. (11)

## Backups and restore
- [ ] A restore drill to a scratch host runs on a schedule and its last success date is known. (15)
- [ ] Backup tool version matches the server version. (15)
- [ ] Data durability has a named owner. (15)
- [ ] A fresh backup or snapshot exists before any destructive step. (9, 15)
