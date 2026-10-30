# Checklist

Use on a pipeline PR, a feature platform design, or an experiment readout. Each item names the rule it enforces.

## Data contract and freshness
- [ ] Every event type has a schema version, and a new version gets its own ingest path or view. (rule 1)
- [ ] Source offsets or equivalent lineage columns are stored on every row. (rule 1)
- [ ] Generated files and configs are checked for row count and size against the consumer's limit before publish. (rule 2)
- [ ] Dedup key and window are written down; dedup runs at write and again in the read view. (rule 3)
- [ ] Each scheduled job states its interval and its runtime, and runtime is well inside the interval. (rule 4)
- [ ] Distinct-count questions use sketches unless exact counts are a stated requirement. (rule 5)

## Online/offline parity
- [ ] Every serving feature is pre-materialized or its live compute fits the latency budget, with the budget named. (rule 6)
- [ ] Each feature has one definition used by both training and serving. (rule 6)
- [ ] Source-of-truth data is normalized; one logical change does not fan into many writes. (rule 7)
- [ ] Per-surface models are compared against a shared model trained on all interactions. (rule 8)

## Orchestration and backfill
- [ ] Jobs run on the shared orchestrator, not standalone cron. (rule 9)
- [ ] Orchestrator overhead per job is measured. (rule 9)
- [ ] Multi-store writes go through a durable log with replay. (rule 10)
- [ ] A backfill of one day run twice gives identical totals. (rules 3, 10)
- [ ] Heavy BI datasets are materialized on staleness with a timeout. (rule 11)

## Experiment design
- [ ] Primary metric is reported per segment, including long tail and new items. (rule 12)
- [ ] Assignment and exposure are logged in the same pipeline as the outcome metric. (rule 12)
- [ ] Any statistical method or figure quoted has a source; otherwise it is stated as general practice without numbers. (rule 12)

## Monitoring
- [ ] Freshness lag, row count and artifact size have alerts with a named owner. (rules 2, 4)
- [ ] Schema drift (new, missing or retyped fields per version) alerts. (rule 1)
- [ ] Training/serving skew is checked on a sample of live requests. (rule 6)
- [ ] Training joins are point-in-time correct and the train/test split is by time. (general practice, no source)
- [ ] Backfills publish by atomic swap after validation. (general practice, no source)
- [ ] Failed jobs are classified automatically and routed by class. (rule 13)

## Platform access
- [ ] All LLM calls go through one gateway with one API shape. (rule 14)
- [ ] Text-to-SQL is limited to known tables and shows the generated SQL. (rule 15)
