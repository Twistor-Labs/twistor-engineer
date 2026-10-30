# Case studies

All figures are the company's own report. Dates are publication dates. Source ids point to `sources.md`.

## Zerodha: scaling with common sense (2020)
- Context: a retail trading platform with very spiky traffic at market open.
- What they did: kept a small stack, measured the database as the real ceiling, pooled connections and kept hot data in memory.
- Numbers: a framework doing 500K rps in front of a 1K qps DB serves 1K rps; 10-100 pooled connections serve 10K HTTP requests; 10x bursts over 10 s are normal; a full day of data for millions of users fits in a few hundred GB of Redis; one Go service holds hundreds of thousands of WebSockets; peak 7M+ trades/day (Apr 2020), later 8-9M.
- Lesson: arithmetic on the DB and the working set beats architecture changes. Rules 1, 2, 3, 4. [S6]

## Discord: trillions of messages (2023)
- Context: 12 Cassandra nodes in 2017 holding billions of messages grew to 177 nodes holding trillions by early 2022.
- What they did: migrated the messages cluster to ScyllaDB and added a Rust data services layer between the API and the database.
- Numbers: 177 nodes to 72; p99 historical reads 40-125 ms to 15 ms; p99 inserts 5-70 ms to a steady 5 ms.
- Lesson: part of the gain came from the data services layer, not only the engine swap. Figures are from 2022-2023 and cover the messages cluster only. Rule 6. [S1]

## Discord: search rebuild (2025)
- Context: realtime indexing ran through a Redis queue into Elasticsearch clusters of more than 200 nodes each.
- What they did: moved the queue to Google Cloud Pub/Sub after Redis maxed CPU and dropped messages during Elasticsearch node failures; split the big clusters into a multi-cluster cell layout of 40 Elasticsearch clusters with thousands of indices on Kubernetes (ECK operator).
- Numbers: median query latency 500 ms to under 100 ms; p99 1 s to under 500 ms; indexing throughput doubled across trillions of messages.
- Lesson: a queue must survive the backlog its consumers create when they fail; many small clusters beat one huge one. Rules 7, 8. [S2]

## Discord: Go to Rust for Read States (2020)
- Context: the Read States service showed latency and CPU spikes roughly every 2 minutes.
- What they did: traced the spikes to Go's forced GC run and rewrote the service in Rust.
- Numbers: Rust beat Go on every single performance metric; cache raised to 8 million Read States.
- Lesson: change runtime only when you can show a repeating, measured runtime signature. Rule 15. [S16]

## Uber: CacheFront (2024, 2025)
- Context: Docstore reads at very high volume.
- What they did: integrated Redis into Docstore's query engine as CacheFront, so caching lives in the data access path rather than in each caller.
- Numbers: in 2024, 40M+ requests/s across production Docstore; one use case at 6M+ RPS on about 3K Redis cores versus about 60K CPU cores from storage (the post gives both 99% and ~99.9% hit rate). In 2025, 150M+ reads/s with stronger consistency guarantees.
- Lesson: a shared cache in the data layer is cheaper than scaling storage, but the consistency story has to be designed, not assumed. Rule 5. [S3][S4]

## Slack: cellular architecture (2023)
- Context: on 2021-06-30 a network link between one us-east-1 AZ and several others failed intermittently from 11:45am PDT, was removed at 12:33pm, failed again at 5:22pm and was removed for good at 5:31pm. A single-AZ fault still caused user-visible errors.
- What they did: treated each AZ as a cell; most services only take traffic from and send traffic to their own AZ, so a failing AZ can be drained.
- Numbers: design goal of draining as much traffic as possible within 5 minutes, in 1% weight steps, with no user-visible errors, driven by a 99.99% SLA (under 1 hour of downtime a year).
- Lesson: not every service can be siloed (Vitess needs a single backend per write); list the exceptions. The 5-minute figure is a goal, not a measured result. Rule 9. [S5]

## Netflix: data abstractions (2024)
- Context: many teams storing similar shapes of data on Cassandra and other stores.
- What they did: published Key-Value and TimeSeries abstraction layers, then built a Distributed Counter on top of TimeSeries (Cassandra, with optional Elasticsearch index).
- Numbers: Distributed Counter serves close to 75K count requests/s globally at single-digit millisecond latency.
- Lesson: a few shared abstractions let the platform team own the store; product teams own the access pattern. KV and TimeSeries dates are search-indexed. Rule 10. [S7][S8][S9]

## Netflix: write-ahead log (2025)
- Context: data spread across several stores drifts and can be lost or corrupted.
- What they did: built a WAL abstraction as a durable write path in front of the stores.
- Lesson: one durable log beats application-level dual writes. Rule 11. [S11]

## Netflix: live events (2024, 2025)
- Context: live streaming produces a single, sharp peak rather than gradual growth.
- Numbers: the Nov 2024 fight drew 60M households and peaked at 65M concurrent streams.
- What they did: in 2025 described Live Origin, a custom multi-tenant origin that brokers segments from cloud packagers to the Open Connect CDN.
- Lesson: an event is a step function; plan the origin and CDN path for the peak. Rules 2, 13. [S10][S12]

## Meta: Threads (2023)
- Context: a new app launched into an existing user base.
- Numbers: built in about five months; over 100M users in its first five days.
- Lesson: the launch week, not the growth curve, sets the capacity target. Rule 2. [S13]

## Airbnb: Viaduct (2025, 2026)
- Context: a data-oriented service mesh in front of Airbnb's services since 2020.
- Numbers: traffic through Viaduct grew 8x since 2020; teams using it doubled to 130+; code tripled to over 1.5M lines. Open-sourced, with a stable 1.0 public API in May 2026.
- Lesson: a shared data layer can scale with the organization if its API is held stable. Context for rule 10. [S14][S15]

## Zepto: Postgres connection pools (2025)
- Context: a 10-minute delivery promise, where "even a single minute disruption is unaffordable".
- What they did: found each pod's pool hardcoded at 10x need (20 active plus 70 overflow), and Postgres `idle_session_timeout` of 5 min killing idle pooled connections. Set pool max about 20% above observed peak, lowered connection timeout from 5 s, capped idle connections at 5-10% of active.
- Numbers: next step was cutting active connections from 16k to 5k, with RDS Proxy under evaluation.
- Lesson: pool size times pod count is a DB capacity number. Rule 4. [S17]

## Zepto: product enrichment rebuild (2026)
- Context: 1.5M batch reads/min on MongoDB for listings, cart, search and recommendations.
- What they did: denormalized documents caused write amplification and WiredTiger eviction once dirty cache passed 20%, spiking reads from under 10 ms to 100 ms+. Interim fix batched updates in 5-min windows; final fix normalized storage.
- Numbers: 1,000+ writes per product update became one (99.9% fewer writes); cluster 7 x M40 to 3 x M60 (85% less infra); API latency down 50-70%; per-pod throughput 3-4K to 15K req/min.
- Lesson: denormalization is a write-cost decision; count writes per update. Rule 12. [S18]

## Zepto: traffic tsunami (2025)
- Context: high-demand events send more traffic than the origin should take.
- What they did: visitor prioritization at the CDN with a CloudFront Function reading rules from CloudFront KeyValueStore, letting eligible users through and returning a wait or fallback to the rest. Rule changes go live in seconds without a deploy.
- Numbers: 5xx and 4xx errors down 80%, zero downtime in high-demand events.
- Lesson: admission control at the edge is cheaper than scaling the origin for a few peak minutes. Rule 13. [S19]

## Zepto: low-latency service and cache warm-up (2025)
- Context: a service at millions of requests per minute drifted from 25 ms to 200 ms+ p99; bigger nodes did not fix it.
- What they did: cut string-handling allocations, tuned DB caching, and moved cache warm-up off the database (one scheduled pod writes a snapshot to S3, other pods load from S3).
- Numbers: DB cache miss 3% to 0.1%; miss penalty about 200 ms to about 1 ms; peak pod count down 50%.
- Lesson: cold starts that read from the primary DB are a load pattern; give warm-up its own path. Context for rule 5. [S20]

## Google Cloud: Service Control (2025)
- Context: a new policy code path with no feature flag hit data with unintended blank fields; a null pointer crashed the binary.
- What happened: retries lacked randomized exponential backoff, and in us-central1 the Spanner-backed quota DB overloaded, taking up to about 2 h 40 min to fully recover.
- Lesson: retry storms turn a crash into a capacity problem; jitter and flags are scaling controls. Rule 14. [S21]

## Desert Ant Labs: moving a cloud pipeline on-device (2026)
- Desert Ant Labs reports its video editor's Auto Edit made three round trips off the device (speech recognition, audio enhancement, LLM prompts for clips and titles).
- Numbers before: 90-120 seconds for a 10-minute video, "thousands of euros a month", "with costs doubling every 60-90 days".
- What they did: replaced it with six narrow on-device models; total 14.18s on iPhone 17 Pro, model loads included, zero network calls.
- Lesson: per-call paid inference scales cost with usage; project the cost curve in the capacity estimate and price the on-device option. Rule 1. [S22]
