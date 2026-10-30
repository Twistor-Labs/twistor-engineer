# Data store matrix

Name the access pattern first, then the store. The "case" column points at a published example in `case-studies.md`; it is evidence that the pattern worked for that company at that time, not a product endorsement. Figures carry their source id.

| Access pattern | Default choice | Move up when | Case |
|---|---|---|---|
| Relational, transactional, modest scale | One Postgres primary, small pools, read replicas later | Total connections near the DB limit, or one table dominates CPU | Zepto pools 10x oversized, 16k to 5k active connections planned [S17] |
| Hot read path, many callers, same keys | Shared cache inside the data access layer, with a hit-rate target and invalidation path | Storage cores grow faster than traffic | Uber: 6M+ RPS on about 3K Redis cores vs about 60K storage cores [S3]; 150M+ reads/s [S4] |
| Whole working set fits in memory | In-memory store on one box (Redis) | Working set outgrows one box's RAM | Zerodha: a day of data for millions of users in a few hundred GB of Redis [S6] |
| Append-heavy, partitioned by entity and time (messages, events) | Wide-column store (Cassandra or ScyllaDB) behind a data services layer | p99 rises with node count; hot partitions | Discord: 177 to 72 nodes, p99 reads 40-125 ms to 15 ms [S1] |
| Time series, counters | A shared TimeSeries or Counter abstraction over a wide-column store | Several teams building the same thing | Netflix Counter on TimeSeries, close to 75K req/s at single-digit ms [S7][S8] |
| Simple key-value across many teams | A shared KV abstraction that hides the store | Teams hand-roll clients and schemas | Netflix KV abstraction [S9] |
| Writes that must land in several stores | Durable write-ahead log that replays into each store | Stores drift, or writes are lost on partial failure | Netflix WAL [S11] |
| Document reads with frequent partial updates | Normalized documents; denormalize only reads that justify the write cost | One business update fans out to many writes | Zepto: 1,000+ writes per update to one, 85% less infra [S18] |
| Work queue that must survive consumer outages | Queue with guaranteed delivery and large backlog capacity | Not optional once consumers can fail | Discord: Redis queue dropped messages, moved to Pub/Sub [S2] |
| Full-text search at very large scale | Many smaller search clusters behind routing (cells) | A single cluster passes a few hundred nodes | Discord: 40 Elasticsearch clusters, median 500 ms to under 100 ms [S2] |
| Many services exposing data to many clients | Data-oriented service mesh or graph layer with a stable API | Point-to-point calls multiply across teams | Airbnb Viaduct: traffic 8x since 2020, 130+ teams [S14][S15] |
| Long-lived connections (WebSockets) | One efficient process per box before a fleet | Measured per-process limit is reached | Zerodha: hundreds of thousands of WebSockets in one Go service [S6] |
| Event-driven live media delivery | Dedicated origin feeding a CDN, sized for the peak | Live events | Netflix Live Origin [S12]; 65M concurrent streams peak [S10] |

## How to use it
1. Write the access pattern in one line: key shape, read/write ratio, size, latency target, consistency need.
2. Start at the default choice. Move up only when the "move up when" condition is measured, not predicted.
3. Cite the case in the answer, with its date, and note that the numbers are the company's own.
