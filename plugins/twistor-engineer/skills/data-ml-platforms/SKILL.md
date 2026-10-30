---
name: data-ml-platforms
description: 'Use when building or debugging event pipelines, analytics ingestion, feature stores, ML training or serving data, orchestration, backfills or experiment metrics. Phrases: "duplicate events", "dashboard numbers do not match", "stale features", "training/serving skew", "feature file too big", "backfill the table", "Kafka to ClickHouse", "warehouse bill too high", "sample ratio mismatch", "A/B readout looks wrong", "LLM gateway", "text-to-SQL". Nouns: data contract, schema version, dedup window, freshness SLA, materialized view, sketch, feature platform, write-ahead log, orchestrator. Gives sourced instead-of rules from Zepto, Netflix, Uber, Airbnb, Meta and Cloudflare and a fixed answer shape. Not for model or LLM quality evals (use llm-evals) or picking a database or shard key (use system-design-scaling).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Data contracts, online/offline parity, orchestration and backfill, experiment metrics and pipeline monitoring, from Zepto, Netflix, Uber and Airbnb posts.'
---

# Data and ML Platforms

Data platforms fail quietly: a duplicated row, a feature computed two ways, a job that reran half a day. This skill treats every pipeline as a contract between a producer and a consumer and asks, before any tool choice, what the contract is, how fresh it must be, and how you would know it broke. Every rule comes from a company that shipped it and wrote it up.

## When to use / not

- Use: designing an event or analytics pipeline (Kafka, Flink, ClickHouse, warehouse), its dedup and freshness story.
- Use: feature pipelines for ML, where training data and serving data must agree.
- Use: orchestrating batch jobs, planning a backfill, or cutting warehouse cost from dashboards.
- Use: reading an experiment or ranking readout and deciding whether the metric is trustworthy.
- Use: giving many teams shared access to LLMs or to self-serve SQL.
- Use: a recommender or ranking model whose offline gains do not show up in production.
- Not: scoring model or LLM output quality, judge prompts, eval sets (see llm-evals).
- Not: choosing a primary database, cache or shard key for an online service (see system-design-scaling).
- Not: a live outage on the serving path (see reliability-incident-response).

## Rules

1. **Instead of** letting every producer write one shared event shape that consumers parse defensively, **do** version each event schema, give each version its own ingest path, and keep the source offsets on every row. **Why:** Zepto reports its 1B events/day ads analytics uses one ClickHouse materialized view per event-schema version and keeps Kafka `_topic/_partition/_offset` for audit. [S1]
2. **Instead of** trusting a generated feature or config file because your own pipeline built it, **do** validate its row count and size against the consumer's hard limit before you publish it. **Why:** At Cloudflare a ClickHouse permission change made a metadata query return duplicate rows, the Bot Management feature file doubled, broke a 200-feature limit (about 60 in use), and the proxy panicked. [S17]
3. **Instead of** assuming the stream delivers exactly once, **do** dedup at write time over a bounded window and apply the same dedup again in the read view. **Why:** Zepto dedups ad events with argMax over a 4-hour window and repeats it at read time in a view. [S1]
4. **Instead of** rerunning freshness-critical jobs over the whole table, **do** scope each run to keys with recent activity so the job finishes well inside its interval. **Why:** Zepto's budget-overspend job runs every 3 minutes, finishes in about 1 minute, and only touches campaigns with events in the last 3 hours. [S1]
5. **Instead of** storing every raw event to answer reach and audience counts, **do** keep probabilistic sketches when an approximate distinct count is enough. **Why:** Zepto estimated 100M+ events/day as about 3B records and 1.2 TB a month raw, and switched to sketches whose size stays constant. [S2]
6. **Instead of** computing ranking features live in the request path, **do** pre-materialize them and define each feature once so training and serving read the same definition. **Why:** Zepto says live feature compute "is incompatible with a sub-200ms latency budget"; Airbnb open-sourced its feature platform Chronon, co-maintained with Stripe. [S4][S8]
7. **Instead of** fanning one catalog change into many denormalized writes, **do** normalize the source of truth and batch downstream updates in windows. **Why:** Zepto cut 1,000+ writes per product update to one, moved from 7 x M40 to 3 x M60 (85% less infra) and cut API latency 50-70%. [S3]
8. **Instead of** training a separate recommender per surface on its own slice of behavior, **do** evaluate one foundation model on all interactions and adapt it per surface. **Why:** Netflix reports over 300M users at end of 2024 produce hundreds of billions of interactions, comparable to LLM token volume. [S11]
9. **Instead of** each team running its own cron scripts, **do** put batch and ML jobs on one orchestrator and measure the orchestrator's own overhead as a first-class metric. **Why:** Netflix Maestro runs about half a million jobs a day (around 2M on busy days), and a 2025 engine rework cut overhead from seconds to milliseconds, 100x. [S12][S13]
10. **Instead of** dual-writing to several stores from application code and hoping they agree, **do** put a durable write-ahead log in front so failed or partial writes can be replayed. **Why:** Netflix built a WAL abstraction specifically against data loss, corruption and cross-store entropy. [S14]
11. **Instead of** letting every dashboard chart re-run the same heavy virtual-dataset SQL, **do** materialize the dataset when it goes stale, with a timeout, and point charts at the result. **Why:** Zepto had 16K+ virtual datasets firing up to 12 identical queries per dashboard; materialization cut warehouse cost 30-35%. [S7]
12. **Instead of** reading one global metric built from head queries or the biggest segment, **do** report the metric per segment, including the long tail and new items. **Why:** Zepto found head-query benchmarks ignore about 80% of query space and now reports segment numbers such as retrieval efficiency for new listings. [S5]
13. **Instead of** a human triaging every failed job by reading logs, **do** classify failures automatically and route each class to an owner, with a human checking the classifier. **Why:** Netflix reports about 100,000 workloads fail on peak days and built Pensive for LLM-assisted diagnosis across Titus, Spark, Trino and Druid. [S15]
14. **Instead of** each team wiring its own LLM vendor keys and clients, **do** route all model calls through one gateway that speaks a single API shape. **Why:** Uber's GenAI Gateway mirrors the OpenAI HTTP/JSON interface and serves 60+ use cases for close to 30 teams across OpenAI, Vertex AI and in-house models. [S9]
15. **Instead of** routing every ad hoc question through an analyst writing SQL by hand, **do** offer text-to-SQL scoped to known tables and keep the generated SQL visible for review. **Why:** Uber reports QueryGPT cut query authoring from about 10 minutes to about 3, against roughly 1.2M interactive queries a month. [S10]

## Answer shape

Every answer from this skill follows these sections, in order. Skip a section only by saying why it does not apply.

1. **Data contract and freshness** (rules 1 to 5)
   - Producer, consumer, owner of each.
   - Schema and version; what a new version does to the ingest path.
   - Primary key, dedup key, dedup window, where dedup runs (write, read, both).
   - Freshness target in minutes, and the job interval and runtime that meet it.
   - Late, duplicate and malformed data: dropped, quarantined or corrected, and who is told.
   - Raw rows versus sketches or aggregates, with the question each one answers.
2. **Online/offline parity** (rules 6 to 8)
   - Where each feature or metric is defined, once.
   - How training reads it and how serving reads it; pre-materialized or live, against the latency budget.
   - The single point where training and serving could diverge, and how you would detect it.
   - Model scope: per surface or shared, and what the shared baseline is.
3. **Pipeline orchestration and backfill** (rules 9 to 11)
   - Orchestrator, schedule, dependencies, and its own overhead.
   - Idempotency of each step: what makes a rerun safe.
   - Backfill plan: date range, order, how totals are checked against the live run.
   - Write recovery: durable log, replay, and how cross-store drift is found.
   - Downstream BI: which datasets are materialized and how staleness is defined.
4. **Experiment design** (rule 12)
   - Primary metric and guardrail metrics, written before launch.
   - Segments the decision depends on, including long tail and new items.
   - Assignment and exposure logged in the same pipeline as the outcome.
   - Any statistical method named without a source is stated as general practice, with no figures.
5. **Monitoring: drift, skew, freshness** (rules 2 and 13)
   - Freshness lag per table or feature, with an alert threshold and owner.
   - Row count and artifact size against the consumer's hard limit.
   - Schema drift: new, missing or retyped fields per version.
   - Training/serving skew: a sample of live requests compared with the offline value.
   - Job failures classified and routed by class, with a human checking the classifier.
6. **Open questions**
   - What you could not decide without numbers from the user: event rate, latency budget, freshness target, cost ceiling.

For platform access questions (LLM gateway, text-to-SQL), answer sections 1 and 5 for the gateway itself and apply rules 14 and 15.

## Red flags

- "Kafka gives us exactly once, we do not need dedup." -> Rule 3. Show the dedup key and window.
- "The feature file is generated by our own job, it is safe." -> Rule 2. Validate size and count before publish.
- "We will compute that feature on the fly at request time." -> Rule 6. Check the latency budget first.
- "Offline AUC went up, ship it." -> Ask how serving computes the same features, then read segment metrics (rule 12). Model quality method lives in llm-evals.
- "The backfill is just rerunning the job." -> Prove the step is idempotent and that the write path cannot double count.
- "The overall number moved, the test won." -> Rule 12. Which segment moved, and did the long tail move the other way?
- "Each team can call the vendor directly for now." -> Rule 14. A gateway later means migrating every caller.
- "We will just dual-write to both stores." -> Rule 10. Name the log and the replay path.
- "Dashboards are slow, add warehouse capacity." -> Rule 11. Count identical queries per dashboard first.
- "We store every event in case we need it later." -> Rule 5. Name the question; keep raw only if it needs raw.
- "The orchestrator is fine, our jobs are slow." -> Rule 9. Measure scheduler overhead per job before tuning jobs.
- Citing a variance-reduction method or a statistical power figure with no source -> this skill has none verified; state it as general practice and do not quote numbers.

## Verification

Before calling a pipeline or design done, show:

- The contract written down: schema version, key, dedup window, freshness target, owner.
- A replay of a real duplicate or late event through ingest and the read view, with the count unchanged.
- A size and row-count check on any generated artifact, with the consumer's limit named.
- One feature traced from definition to the training table and to the serving read, same code path or same spec.
- A backfill of one past day run twice with identical totals.
- Backfills write to staging, validate, then swap in atomically, so readers never see a half-written partition (general practice).
- Training joins are point-in-time: no feature value from after the label timestamp; split train and test by time (general practice).
- Freshness lag and failure-rate panels that exist and alert to a named owner.
- The orchestrator's overhead per job, measured, next to the job's own runtime.
- A sample of live requests whose served feature values match the offline values.
- For experiments: the metric broken out by at least the segments the decision depends on.

## References

| File | Read when |
|---|---|
| references/case-studies.md | you need the full story behind a rule, or a comparable company design to cite |
| references/checklist.md | reviewing a pipeline PR, a feature store design or an experiment readout before ship |
| references/sources.md | citing a figure or checking a source's status |

Companion skills: llm-evals for model and LLM quality evaluation, system-design-scaling for storage and database choices, reliability-incident-response when a pipeline failure is a live incident.
