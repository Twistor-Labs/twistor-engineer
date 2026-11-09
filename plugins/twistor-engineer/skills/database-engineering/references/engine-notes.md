# Engine notes

Postgres first, from the PostgreSQL 18 docs (read at 18.6). Behaviour changes between major versions, so check the docs for the version you run. MySQL appears only where a verified source covers it. Other engines (document, key-value, wide-column) are not covered here; say so instead of applying these notes.

## Postgres

### Lock modes that matter for migrations [S15][S13][S14]

| Statement | Lock | Blocks |
|---|---|---|
| Most `ALTER TABLE` forms (default) | ACCESS EXCLUSIVE | everything, including plain SELECT |
| `CREATE INDEX` | SHARE | inserts, updates, deletes (reads continue) |
| `CREATE INDEX CONCURRENTLY`, `REINDEX CONCURRENTLY` | SHARE UPDATE EXCLUSIVE | other schema changes and VACUUM on the table, not reads or writes |
| `ALTER TABLE ... VALIDATE CONSTRAINT` | SHARE UPDATE EXCLUSIVE | not reads or writes |
| `ALTER TABLE ... ADD FOREIGN KEY` | SHARE ROW EXCLUSIVE | writes |

- "Once acquired, a lock is normally held until the end of the transaction." Keep migration transactions short and single-purpose.
- A lock request waits "indefinitely" for conflicting locks unless a timeout is set. LLM Gateway saw new queries queue behind a waiting migration [S9]; the docs do not describe queue order, so treat that as observed behaviour.

### Timeouts [S12]

- `lock_timeout`: aborts a statement that waits longer than this for any lock; "The time limit applies separately to each lock acquisition attempt." Default 0 (off).
- `statement_timeout`: aborts any statement running longer than this. Default 0 (off). The docs note that with a nonzero `statement_timeout`, setting `lock_timeout` to the same or a larger value is pointless, since the statement timeout fires first.
- `idle_in_transaction_session_timeout`: ends sessions idle inside an open transaction. Default 0 (off).
- `transaction_timeout`: ends sessions whose transaction runs longer than this. Default 0 (off).
- The docs say setting `lock_timeout`, `statement_timeout` or `transaction_timeout` in postgresql.conf "is not recommended because it would affect all sessions." Set them per session, role or database.
- Reference point: LLM Gateway now runs migrations with a 5 second lock timeout plus a statement timeout and retries with backoff [S9].

### Adding a NOT NULL column to a large table [S13]

1. `ADD COLUMN` with a non-volatile (constant) default, or nullable. The default is stored in metadata: "In neither case is a rewrite of the table required." A volatile default (for example `clock_timestamp()`) rewrites the table and its indexes. Still takes ACCESS EXCLUSIVE briefly, so set `lock_timeout`.
2. If existing rows need computed values, backfill in batches (rule 6).
3. `ADD CONSTRAINT ... CHECK (col IS NOT NULL) NOT VALID`: skips the scan, but locks out other writes until it commits, so keep it in its own short transaction.
4. `VALIDATE CONSTRAINT`: scans, but takes only SHARE UPDATE EXCLUSIVE.
5. `ALTER COLUMN ... SET NOT NULL`: with a valid CHECK proving no NULLs, "the table scan is skipped".

### Indexes [S14]

- `CREATE INDEX CONCURRENTLY` does two table scans and waits for existing transactions that could use the index; it "takes significantly longer" and cannot run inside a transaction block.
- On failure it leaves an "invalid" index that is ignored for queries but still costs update overhead. Drop it and retry, or `REINDEX INDEX CONCURRENTLY`.

### Sharding signals [S4][S3]

- Notion: VACUUM stalling consistently and transaction ID wraparound risk; their per-shard limits were 500 GB per table and 10 TB per physical database.
- Figma: tables of several terabytes and billions of rows, vacuum reliability impact, write-heavy tables nearing the RDS IOPS ceiling.

### Pools [S10][S11]

- Zepto: Postgres `idle_session_timeout` at 5 minutes was killing idle pooled connections; make server-side idle timeouts and client pool idle settings agree.
- Figma: pools of about 100 connections per Postgres instance behind a shared pooler; connection creation rate-limited with a token bucket.

### Managed Postgres maintenance [S7]

- Clerk: a Google Cloud SQL live migration raised disk latency and lock contention into a full outage; moving reads to replicas did not help. Clerk now does Postgres maintenance by replica promotion it controls. Ask your provider how live maintenance works and whether it can be pinned.

## MySQL

### Online schema change [S1]

- Trigger-based tools add load and lock contention to every write on the master, and their triggers cannot be paused or removed mid-run without data loss.
- gh-ost is triggerless: it reads the binary log and applies changes to a ghost table over one connection.
- Throttle on `Threads_running`, replica lag (`max-lag-millis`), a custom query or a flag file; change chunk size and thresholds while it runs.
- Postpone the cut-over with a flag file so it happens when an operator is ready.
- `--test-on-replica` runs the full migration on a replica and swaps tables back for comparison.

No other MySQL claims were verified for v1.2.
