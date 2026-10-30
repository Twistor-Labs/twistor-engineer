# Case studies

All numbers are self-reported by the company in its own post. Source ids point to sources.md.

## Zepto: ads analytics at 1B events a day (rules 1, 3, 4) [S1]
- Context: ad impressions and clicks feeding advertiser dashboards and budget control.
- v1 was Flink with 5-minute tumbling windows writing into Postgres.
- v2 moved to ClickHouse via the Kafka Engine, one materialized view per event-schema version, ReplicatedMergeTree, run by the Altinity operator on Kubernetes. Kafka `_topic/_partition/_offset` stay on each row for audit.
- Dedup is argMax over a 4-hour window, applied again at read time in a view. Nodes grew from 4 CPU/16 GB/1 TB to 32 CPU/90 GB/3 TB.
- Budget-overspend job runs every 3 minutes, finishes in about 1 minute, scoped to campaigns with events in the last 3 hours.
- Lesson: version the contract, keep lineage on the row, dedup twice, and scope fast jobs to active keys.

## Zepto: audience targeting with sketches (rule 5) [S2]
- Context: 100M+ events a day (4.16M an hour) for advertiser audience building.
- Raw storage would be about 3B records and 1.2 TB a month at 400 bytes a record.
- Zepto switched to probabilistic sketches whose size stays constant whether they hold 1M or 1B events.
- Lesson: if the question is "how many distinct", store the sketch, not the events.

## Zepto: catalog and product enrichment rebuild (rule 7) [S3]
- Context: product enrichment serves 1.5M batch reads a minute for listings, cart, search and recommendations.
- Denormalized documents caused write amplification; once the dirty cache passed 20%, WiredTiger eviction pushed reads from under 10 ms to 100 ms+.
- Interim fix: batch catalog updates in 5-minute windows and rate-limit downstream writes.
- Real fix: normalize storage so 1,000+ writes per product update became one (99.9% fewer writes). Cluster went from 7 x M40 to 3 x M60 (85% less infra), API latency fell 50-70%, per-pod throughput rose from 3-4K to 15K requests a minute.
- Lesson: write fan-out is a data-model bug, not a capacity problem.

## Zepto: pre-materialized ranking features (rule 6) [S4]
- Context: search keyed by city and product id with per-store stock as nested docs; the Product Service resolves inventory and price at query time.
- ML ranking reads pre-materialized features because live feature compute "is incompatible with a sub-200ms latency budget."
- Lesson: decide where each feature is computed from the latency budget, then keep one definition for training and serving.

## Zepto: search quality measured by segment (rule 12) [S5]
- Context: head-query benchmarks ignored about 80% of the query space.
- Zepto generates 5-10 queries per product and LLM-judges the top-K; 1M products x 10 queries gives 10M query-product pairs.
- Results are reported per segment, for example "retrieval efficiency for new listings is 0.7", instead of one global percentage.
- Lesson: a global metric hides the tail. Judge method details belong in llm-evals.

## Zepto: Voyager journeys and in-house A/B (rule 12, answer shape step 4) [S6]
- Context: marketing and lifecycle journeys built in a drag-and-drop UI that compiles to a custom DSL on Temporal.
- Zepto runs an in-house CDP and A/B tool (AB360) alongside it; the platform handles 5M+ concurrent executions and 50+ concurrent journeys at sub-second trigger latency.
- Lesson: keep experiment assignment inside the platform that delivers the treatment, so exposure and outcome land in the same data.

## Zepto: Superset dataset materialization (rule 11) [S7]
- Context: 16K+ virtual datasets each re-ran a 200-line CTE once per chart, up to 12 identical parallel queries per dashboard.
- Fix: inline CTAS when a dataset goes stale, 5-minute timeout, no queue.
- Result: warehouse cost down 30-35%, runtime down up to 60% on average and 90-95% on top dashboards.
- Lesson: BI tools are pipeline consumers; give them materialized tables.

## Airbnb: Chronon feature platform (rule 6) [S8]
- Context: Airbnb's ML feature platform.
- Open-sourced in April 2024 "along with our partners at Stripe, who are early adopters and co-maintainers of the project."
- Lesson: feature definition is shared infrastructure worth a dedicated platform, not per-model glue.

## Uber: GenAI Gateway in Michelangelo (rule 14) [S9]
- Context: many teams adopting LLMs at once.
- A Go service that mirrors the OpenAI HTTP/JSON interface, fronting OpenAI, Vertex AI and Uber-hosted models.
- Over 60 distinct use cases, close to 30 customer teams.
- Lesson: one API shape and one choke point for keys, logging and policy.

## Uber: QueryGPT (rule 15) [S10]
- Context: about 1.2M interactive queries a month, Operations about 36% of them.
- A manual query took about 10 minutes; with QueryGPT about 3.
- Lesson: text-to-SQL pays where query volume is high; keep the SQL visible so users can check it.

## Netflix: foundation model for recommendation (rule 8) [S11]
- Context: over 300M users at the end of 2024.
- Netflix: that yields "hundreds of billions of interactions - an immense dataset comparable in scale to the token volume of large language models."
- Lesson: at this scale one shared model over all interactions becomes the baseline to beat.

## Netflix: Maestro orchestrator (rule 9) [S12][S13]
- Context: Netflix's workflow orchestrator for data and ML jobs, open-sourced in 2024 (Apache-2.0).
- Runs about half a million jobs a day on average, around 2M on busy days.
- 2025 rework improved engine performance 100x, overhead from seconds to milliseconds; shipped to open source.
- Lesson: orchestrator overhead multiplies by job count; measure it.

## Netflix: write-ahead log (rule 10) [S14]
- Context: a data platform writing to several stores.
- Netflix built a WAL abstraction to guard against data loss, corruption and cross-store entropy.
- Lesson: durability and replay belong in a shared layer, not in every service's retry loop.

## Netflix: Pensive failure diagnosis (rule 13) [S15]
- Context: on peak days roughly 100,000 workloads fail.
- Pensive does LLM-assisted automatic diagnosis across Titus, Spark, Trino and Druid.
- Lesson: at that failure volume, classification is the only way a human sees the failures that matter.

## Meta: two 24k-GPU clusters for Llama 3 (training infra context) [S16]
- Two clusters of 24,576 H100 GPUs each: one on RoCE (Arista 7800), one on NVIDIA Quantum2 InfiniBand, 400 Gbps endpoints.
- Meta reports ongoing Llama 3 training on the RoCE cluster; target 350,000 H100 by end of 2024 (about 600,000 H100-equivalent).
- Lesson: at frontier scale Meta built both fabrics and compared them on real training; benchmark your own workload before committing to one.

## Cloudflare: generated feature file broke a hard limit (rule 2) [S17]
- At 11:05 UTC on 2025-11-18 a ClickHouse access-control change made a metadata query (no database-name filter) return duplicate column rows.
- The Bot Management feature file doubled in size, exceeded a preallocated 200-feature limit (about 60 in use), and FL2 Rust code panicked on `unwrap()`.
- Core traffic was largely restored by 14:30; all services resolved at 17:06 UTC. Customers on the older FL proxy got bot score 0 on every request instead of errors.
- Lesson: a pipeline's output is untrusted input to its consumer; check it against the consumer's limits and roll it out in stages.
