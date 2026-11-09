# Answer key: planted flaws in the sample design docs

Six flaws across three docs. A review hits a flaw when it reports it as a `bug` or `risk` finding
(or, for an omission, an `open:` line) under the listed section, and cites the expected rule or one
of the accepted alternates. Rule numbers are from each skill's SKILL.md Rules section.

| # | Flaw | Doc | Section | Expected rule | Also accepted |
|---|---|---|---|---|---|
| F1 | Unsafe migration: `ADD COLUMN`, one `UPDATE` over 420M rows, then `SET NOT NULL` and a plain `CREATE INDEX`, all in one transaction during peak writes, no `lock_timeout` or `statement_timeout`, timing guessed from a 2M-row staging copy | 01-orders-region-migration.md | Part A: `fulfillment_region` column | database-engineering #3 (also #1, #4, #6) | none |
| F2 | Delete before verify: `DROP TABLE order_events` the night the copy job says done, row-count check only the next morning, readers not yet moved | 01-orders-region-migration.md | Part B: partitioned `order_events` | database-engineering #9 | reliability-incident-response #7 |
| F3 | Agent with unbounded tool access: one generic `admin_api(method, path, body)` tool on a service key with `admin:*` scope, guardrails only in the system prompt | 02-support-refund-agent.md | Tools and access | security-engineering #9 | ai-agent-engineering #13, security-engineering #7 |
| F4 | No eval plan for an LLM feature: launch gated on a five-ticket demo picked by leads, then full traffic; weekly sampling after launch is the only quality check | 02-support-refund-agent.md | Launch plan | ai-agent-engineering #14 | llm-evals #1, llm-evals #7 |
| F5 | Missing timeout on an external call in the request path: provider client "with its default settings", worker thread held while waiting, provider has had 47-minute latency incidents, retries multiply the wait | 03-tax-quote-at-checkout.md | Request path | reliability-incident-response #6 | system-design-scaling #14 |
| F6 | No rollback path: all regions in one release train, no flag, the old rate-table code deleted in the same build and the table dropped next release | 03-tax-quote-at-checkout.md | Rollout | reliability-incident-response #5 | reliability-incident-response #1, reliability-incident-response #2 |

Notes:

- F5: no current rule names timeouts directly; #6 (backoff and a maintained circuit breaker on a
  dependency) is the closest. A finding that says "set a timeout and a fallback when the provider
  is slow or down" with either rule counts.
- Doc 01's Rollback section exists but does not cover the dropped table or the long transaction.
  Flagging that is correct but is part of F1 or F2, not a seventh flaw.
- Doc 03's `tax_quote_id` column is a nullable add with no default, which is safe; reporting it as
  a migration bug is a false positive.
- SC-004 target: at least 5 of 6 hit, and no file modified.
