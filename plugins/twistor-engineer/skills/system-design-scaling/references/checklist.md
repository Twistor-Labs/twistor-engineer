# Design review and pre-ship checklist

Each item maps to a rule in SKILL.md. An unchecked item is either fixed or written into "Open questions".

## Estimate
- [ ] Peak rps, read/write split, bytes per day and working set are written down with assumptions (rule 1).
- [ ] The DB's own capacity (qps, connections) is the first number sized, before the app tier (rule 1).
- [ ] Burst factor and the launch or event step are modelled and load-tested, not only the average (rule 2).
- [ ] Someone checked whether the working set fits in RAM on one box before proposing sharding (rule 3).
- [ ] Per-call paid inference on a hot path has a projected cost curve and latency budget, and an on-device or self-hosted option was priced (rule 1).

## Database and connections
- [ ] Pool max per instance is about 20% above observed peak (rule 4).
- [ ] Pool max times instance count, at max autoscale, is below the DB connection limit (rule 4).
- [ ] DB idle timeouts and pool idle settings agree (rule 4).
- [ ] Writes per business update are counted; any fan-out above a handful is justified (rule 12).

## Caching and data layer
- [ ] Any cache has a hit-rate target, an invalidation path, a measured staleness rate with an alert, and a consistency statement (rule 5).
- [ ] Cache warm-up and cold start do not hammer the primary DB (rule 5).
- [ ] A growing database cluster has a data-services layer in front before an engine swap is proposed (rule 6).
- [ ] Teams use the shared KV, time-series or counter abstraction where one exists (rule 10).
- [ ] Writes to more than one store go through a durable log, not application dual writes (rule 11).

## Queues, clusters and isolation
- [ ] Every work queue survives a consumer outage without dropping messages (rule 7).
- [ ] No single cluster is planned past a few hundred nodes without a cell split (rule 8).
- [ ] Services are siloed per AZ where possible; the ones that cannot be siloed are listed (rule 9).
- [ ] An AZ drain has been rehearsed, in weight steps, and its time recorded (rule 9).

## Edge and retries
- [ ] Peak events have edge admission rules that change without a deploy (rule 13).
- [ ] Every retry uses randomized exponential backoff with a cap (rule 14).
- [ ] New code paths on hot dependencies sit behind a feature flag (rule 14).

## Rewrites
- [ ] A language or runtime change cites a measured, repeating runtime signature (rule 15).
- [ ] Each fix in the answer names its cost rung, and cheaper rungs were ruled out with data (Answer shape, section 3).
