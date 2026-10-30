---
name: system-design-scaling
description: 'Use when designing or reviewing a backend for growth or when it is already straining, for example "will this scale", "we need to shard", "p99 keeps climbing", "DB CPU at 90%", "too many connections", "cache or read replica", "split into microservices", "launch day spike", "one AZ took us down", "which database for this", "migrate to a new store without downtime", "dual writes". Covers capacity estimates, connection pools, caches, queues, cells and AZ isolation, data abstractions, write-ahead logs, edge admission and store choice. Gives a capacity estimate, the first component to break, fixes ordered by cost, what not to do and open questions, backed by sourced case studies. Not for one slow function or page load (use performance-engineering) or a live outage and postmortem (use reliability-incident-response).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Capacity math first, then the cheapest fix for the component that breaks first, with sourced case studies.'
---

# System Design and Scaling

Most systems break at the database, the connection pool or one oversized cluster long before they need a new architecture. Do the arithmetic, find the first thing that breaks, and fix it with the cheapest change that holds. Every rule below comes from a company that published its numbers; treat them as that company's claims, not laws.

## When to use / not

- Use: sizing a new service, reviewing a design doc, a store choice, a "we need to shard / split / rewrite" proposal, a launch or event traffic plan, a cluster whose tail latency grows with node count.
- Use: deciding where a cache, queue, cell boundary or write-ahead log belongs.
- Not: one slow endpoint, flame graphs, GC tuning, frontend load time (see performance-engineering).
- Not: an outage in progress or a postmortem (see reliability-incident-response).
- Companion: scale-engineering (separate skill, link only) covers the same answer shape without the case-study base.

## Rules

1. **Instead of** picking an architecture from a diagram, **do** write the capacity estimate first: peak requests per second, read/write ratio, bytes per day, working-set size and total DB connections, then size the database before the app tier. **Why:** Zerodha notes a framework doing 500K rps in front of a DB doing 1K qps serves 1K rps [S6]
2. **Instead of** sizing for average load or a smooth growth curve, **do** size for the burst ratio and the launch or event step, and load-test that number. **Why:** Zerodha treats 10x bursts over 10 s as normal [S6], Threads passed 100M users in its first five days [S13], and one Netflix live event peaked at 65M concurrent streams [S10]
3. **Instead of** sharding or adding a cluster by reflex, **do** check whether the working set fits in RAM on one box and whether one process can hold the connections. **Why:** Zerodha reports a full trading day of data for millions of users fits in a few hundred GB of Redis, and one Go service holds hundreds of thousands of WebSockets [S6]
4. **Instead of** giving every pod a generous connection pool, **do** set pool max about 20% above observed peak and add up connections across all pods against the DB limit. **Why:** Zepto found pools 10x oversized (20 active plus 70 overflow per pod) and set out to cut 16k active Postgres connections to 5k [S17]; Zerodha says 10-100 pooled connections serve 10K HTTP requests [S6]
5. **Instead of** buying storage-tier cores for a read-heavy hot path, **do** put a shared cache inside the data access layer, with a measured hit rate, a written invalidation path and a client-side staleness metric (share of reads that disagree with the source of truth after N minutes). **Why:** Uber reports one 6M+ RPS use case on about 3K Redis cores versus about 60K CPU cores from storage [S3], CacheFront now serves 150M+ reads/s [S4], and Meta says fixing cache consistency starts with measurement; its Polaris checker took TAO from six to ten nines within five minutes [S23]
6. **Instead of** adding nodes when a big database cluster's p99 climbs, **do** put a data-services layer between clients and the database and measure it before swapping the engine. **Why:** Discord went from 177 Cassandra nodes to 72 ScyllaDB nodes with p99 reads 40-125 ms to 15 ms, crediting part of the gain to its Rust data services layer [S1]
7. **Instead of** using Redis as a work queue that must survive backlogs, **do** use a queue with guaranteed delivery that can absorb a large backlog while consumers are down. **Why:** Discord's Redis indexing queue maxed CPU and dropped messages during Elasticsearch node failures, so it moved to Pub/Sub [S2]
8. **Instead of** growing one cluster past a few hundred nodes, **do** split it into many smaller clusters behind a routing layer so each failure and each operation stays small. **Why:** Discord split 200+ node Elasticsearch clusters into 40 clusters; median query latency went from 500 ms to under 100 ms and p99 from 1 s to under 500 ms [S2]
9. **Instead of** spreading every request across all availability zones, **do** silo services per AZ so a sick AZ can be drained by weight, and list the services that cannot be siloed. **Why:** Slack's design goal is draining an AZ within 5 minutes in 1% steps for a 99.99% SLA, but Vitess still needs a single backend per write [S5]
10. **Instead of** letting each team hand-roll clients and schemas on raw stores, **do** offer a few shared data abstractions (key-value, time series, counter) with the store behind them. **Why:** Netflix built its Distributed Counter on its TimeSeries abstraction and reports close to 75K count requests/s at single-digit ms latency [S7][S8][S9]
11. **Instead of** dual-writing to several stores from application code, **do** send the write through a durable write-ahead log that replays into each store. **Why:** Netflix built a WAL abstraction specifically against data loss, corruption and entropy between stores [S11]
12. **Instead of** denormalizing a hot document so every read is one lookup, **do** count writes per business update and normalize when one change fans out to many writes. **Why:** Zepto cut 1,000+ writes per product update to one, moved from 7 x M40 to 3 x M60 (85% less infra) and cut API latency 50-70% [S18]
13. **Instead of** letting a demand spike reach the origin, **do** admit traffic at the edge with priority rules you can change in seconds without a deploy. **Why:** Zepto's CDN visitor prioritization cut 5xx and 4xx errors by 80% during high-demand events [S19]
14. **Instead of** retrying failed dependency calls immediately or on a fixed interval, **do** use randomized exponential backoff and put new code paths behind a feature flag. **Why:** Google Cloud's 2025-06-12 Service Control crash lacked randomized backoff, and us-central1 took up to about 2 h 40 min to recover as the quota DB overloaded [S21]
15. **Instead of** rewriting in a new language because a service "is slow", **do** prove the runtime is the bottleneck with a measured, repeating signature first. **Why:** Discord moved Read States from Go to Rust after measuring latency and CPU spikes roughly every 2 minutes caused by forced Go GC runs [S16]

## Answer shape

Every answer follows these five sections, in order. Skip none; write "none found" instead.

1. **Capacity estimate.** Peak rps (average times burst factor), read/write split, payload size, bytes per day and per year, working set, connection count across all instances, fan-out per request. Show the arithmetic and label assumptions.
2. **Where it breaks first.** Name the single component that hits its limit first (usually DB connections, DB CPU, one hot partition, one oversized cluster or a cross-AZ dependency) and at what load. One component, with the number.
3. **Fixes ordered by cost.** Cheapest first: config (pool sizes, timeouts, backoff), then query and schema changes, then a cache or queue, then partitioning or cells, then a new store or language. Stop at the first rung that holds the estimate with headroom. Cite the matching case study for each fix.
4. **What not to do.** The tempting moves that do not address the bottleneck from section 2 (microservices, a new database, more nodes, a rewrite), each with one line on why.
5. **Open questions.** The unknowns that would change the answer: missing traffic numbers, consistency requirements, which services cannot be siloed, cache invalidation path, data that must not be lost.

For store choice, read `references/data-store-matrix.md` and name the access pattern before the product.

### Cost ladder for section 3

| Rung | Change | Typical rules | Reversible |
|---|---|---|---|
| 0 | Config: pool sizes, timeouts, retry backoff, flags | 4, 14 | yes, minutes |
| 1 | Query, index and schema changes; normalize write fan-out | 12 | yes, with a migration |
| 2 | Shared cache in the data layer; durable queue; edge admission | 5, 7, 13 | mostly |
| 3 | Data services layer; shared data abstractions; write-ahead log | 6, 10, 11 | costly |
| 4 | Split clusters into cells; per-AZ silos | 8, 9 | costly, months |
| 5 | New store, new language, service split | 6, 15 | rarely |

Climb one rung at a time. Each rung needs a number from section 1 or 2 that the rung below cannot meet.

### Skeleton

```
## Capacity estimate
Peak = <avg rps> x <burst factor> = <peak rps>. Reads <r>%, writes <w>%.
Payload <bytes> -> <GB/day>, <TB/year>. Working set <GB> (fits one box: yes/no).
Connections = <pool max> x <instances at max scale> = <n> vs DB limit <m>.
Assumptions: <list>.

## Where it breaks first
<component> at <load>, because <arithmetic>.

## Fixes ordered by cost
1. Rung <k>: <change>. Holds to <load>. Precedent: <company, year> [S<n>].
2. ...

## What not to do
- <tempting move>: <why it misses the bottleneck>.

## Open questions
- <unknown> -> changes <which section> if <condition>.
```

Label every borrowed number as the company's own report, with its year.

## Red flags

- "We'll need to shard eventually, so let's shard now." -> Do rule 3 arithmetic first; most working sets fit on one box longer than expected.
- "Microservices will fix the latency." -> Section 2 names the bottleneck; if it is the DB, splitting the app tier moves nothing.
- "Bigger instances will do it." -> Check pool totals (rule 4) and write amplification (rule 12) before paying for cores.
- "The cache handles it." -> Ask for the hit rate, the invalidation path, how staleness is measured and what happens on a cold start.
- "Just add more nodes to the cluster." -> Past a few hundred nodes, split into cells (rule 8).
- "Retries will smooth it over." -> Unjittered retries turn a blip into an overload (rule 14).
- "We're multi-AZ, so we're safe." -> Multi-AZ without per-AZ silos still fails on one AZ (rule 9).
- "Their numbers prove it works." -> Every figure here is the company's own report, from its own workload.

## Verification

- The estimate shows its arithmetic and names every assumption.
- One component is named as first to break, with the load at which it breaks.
- Each proposed fix names its cost rung and the source id of a matching case.
- Total DB connections across all instances are compared with the DB limit.
- Retries in the design use randomized exponential backoff.
- For any cache: hit rate target, invalidation path, staleness metric and alert, cold-start behaviour.
- For any cell or AZ silo: the list of services that cannot be siloed.
- Run `references/checklist.md` before signing off a design review.

## References

| File | Read when |
|---|---|
| references/case-studies.md | you need the full story, numbers and lesson behind a rule |
| references/data-store-matrix.md | choosing a store, cache, queue or abstraction for an access pattern |
| references/checklist.md | reviewing a design doc or shipping a scaling change |
| references/sources.md | citing a figure, or checking its date and caveats |

Related skills: performance-engineering (single-service hot paths), reliability-incident-response (outages, postmortems, rollout safety), scale-engineering (companion, link only).
