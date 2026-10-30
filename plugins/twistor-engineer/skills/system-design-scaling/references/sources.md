# Sources

Every figure in this skill traces to one line below. All numbers are self-reported by the company that published them. Status was checked against the primary post on 2026-10-30.

- [S1] How Discord Stores Trillions of Messages. Discord, 2023-03-06. https://discord.com/blog/how-discord-stores-trillions-of-messages (VERIFIED)
  Note: figures date from 2022-2023 and cover the messages cluster only; part of the latency gain came from the Rust data services layer, not the engine swap alone.
- [S2] How Discord Indexes Trillions of Messages. Discord, 2025-04-24. https://discord.com/blog/how-discord-indexes-trillions-of-messages (VERIFIED)
- [S3] How Uber Serves Over 40 Million Reads Per Second Using an Integrated Cache. Uber, 2024-02-15. https://www.uber.com/blog/how-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache/ (CORRECTED)
  Note: the 40M figure is superseded by 150M+ reads/s [S4]. The post gives both 99% and ~99.9% hit rate for the same use case, and says "reads" in the title and "requests" in the body.
- [S4] How Uber Serves Over 150 Million Reads Per Second. Uber, 2025-08-26. https://www.uber.com/us/en/blog/how-uber-serves-over-150-million-reads/ (VERIFIED)
  Note: only the 150M+ headline and "stronger consistency guarantees" were checked; mechanism details were not.
- [S5] Slack's Migration to a Cellular Architecture. Slack, 2023-08-22. https://slack.engineering/slacks-migration-to-a-cellular-architecture/ (CORRECTED)
  Note: earlier summaries said every service is siloed per AZ. Not true: Vitess needs a single backend for any given write. The 5-minute drain is a design goal, no measured drain time is reported.
- [S6] Scaling with common sense. Zerodha, 2020-06-14. https://zerodha.tech/blog/scaling-with-common-sense/ (VERIFIED)
- [S7] Netflix's Distributed Counter Abstraction. Netflix, 2024-11-12. https://netflixtechblog.com/netflixs-distributed-counter-abstraction-8d0c45eb66b2 (VERIFIED)
- [S8] Introducing Netflix TimeSeries Data Abstraction Layer. Netflix, 2024-10-08. https://netflixtechblog.com/introducing-netflix-timeseries-data-abstraction-layer-31552f6326f8 (VERIFIED)
  Note: date and URL slug are search-indexed; confirm the slug before linking from user-facing material.
- [S9] Introducing Netflix's Key-Value Data Abstraction Layer. Netflix, 2024-09-18. https://netflixtechblog.com/introducing-netflixs-key-value-data-abstraction-layer-1ea8a0a11b30 (VERIFIED)
  Note: approximate date and URL slug are search-indexed; confirm the slug before linking from user-facing material.
- [S10] 60 Million Households Tuned In Live for Jake Paul vs. Mike Tyson. Netflix, 2024-11-16. https://about.netflix.com/en/news/60-million-households-tuned-in-live-for-jake-paul-vs-mike-tyson (VERIFIED)
- [S11] Building a Resilient Data Platform with Write-Ahead Log at Netflix. Netflix, 2025-09-26. https://netflixtechblog.com/building-a-resilient-data-platform-with-write-ahead-log-at-netflix-127b6712359a (VERIFIED)
- [S12] Netflix Live Origin. Netflix, 2025-12-15. https://netflixtechblog.com/netflix-live-origin-41f1b0ad5371 (VERIFIED)
- [S13] How Meta built Threads in 5 months. Meta, 2023-11-06. https://engineering.fb.com/2023/11/06/android/how-meta-built-threads-in-5-months/ (VERIFIED)
- [S14] Viaduct, five years on: modernizing the data-oriented service mesh. Airbnb, 2025-09-17. https://airbnb.tech/data/viaduct-five-years-on-modernizing-the-data-oriented-service-mesh/ (VERIFIED)
- [S15] Viaduct 1.0 and the future of Airbnb's data mesh. Airbnb, 2026-05-13. https://medium.com/airbnb-engineering/viaduct-1-0-and-the-future-of-airbnbs-data-mesh-6bab4ec98b89 (VERIFIED)
- [S16] Why Discord is switching from Go to Rust. Discord, 2020-02-04. https://discord.com/blog/why-discord-is-switching-from-go-to-rust (VERIFIED)
- [S17] Symptoms to Root Cause: Debugging PostgreSQL at Scale. Zepto, 2025-04-15. https://blog.zepto.com/scaling-a-distributed-monolith-our-postgresql-journey-at-zepto-a2da988be62a (VERIFIED)
- [S18] From Bottleneck to Breakthrough: How We Rebuilt Product Enrichment at Scale. Zepto, 2026-04-07. https://blog.zepto.com/from-bottleneck-to-breakthrough-how-we-rebuilt-product-enrichment-at-scale-d40b9d93c186 (VERIFIED)
- [S19] What Happens When Zepto Gets Hit With a Traffic Tsunami? Zepto, 2025-05-28. https://blog.zepto.com/what-happens-when-zepto-gets-hit-with-a-traffic-tsunami-6724578d30e4 (VERIFIED)
- [S20] Performance engineering at Zepto. How we build low-latency systems: Part 2. Zepto, 2025-12-15. https://blog.zepto.com/performance-engineering-at-zepto-how-we-build-low-latency-systems-part-2-4315de1f8e83 (VERIFIED)
- [S21] Google Cloud Service Health incident ow5i3PPK96RduMcb1SsW (Service Control). Google Cloud, 2025-06-12. https://status.cloud.google.com/incidents/ow5i3PPK96RduMcb1SsW (VERIFIED)
  Note: 2025-06-12 is the incident date; the report page shows no publication date.
- [S22] From 10-minute recording to edited content in 14 seconds. Desert Ant Labs, 2026-09-14. https://desertant.com/blog/detail-auto-edit-on-device/ (VERIFIED)
  Note: vendor-reported, single source.
- [S23] Cache made consistent. Meta, 2022-06-08. https://engineering.fb.com/2022/06/08/core-infra/cache-made-consistent/ (VERIFIED)
  Note: re-verified 2026-10-30; TAO consistency went from six nines to ten nines by one measure (paraphrase); "99.99999999 percent of cache writes are consistent within five minutes" on the five-minute timescale.

## Left out on purpose

Often repeated elsewhere but not confirmed against a primary post, so not cited: Discord request coalescing and migration speed, Uber CacheFront CDC invalidation details, Uber DOMA service counts, memcache leases, Netflix live-event monitoring rates.
