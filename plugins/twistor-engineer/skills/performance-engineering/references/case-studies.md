# Case studies

All numbers are self-reported by the company named. Each entry gives the context, what they did, the numbers, and the lesson.

## Cloudflare, Pingora (2022)
- Context: Cloudflare replaced its old proxy service with Pingora, compared at the same traffic load.
- Numbers: "about 70% less CPU and 67% less memory" than the old service at the same traffic. For one major customer, connection reuse went from 87.1% to 99.92% (160x fewer new origin connections for that customer); overall new connections per second fell to about a third. Median TTFB -5 ms, p95 -80 ms.
- Lesson: connection setup is a cost centre; measure reuse rate, not just pool size. Rule 8. [S1]

## Meta, Strobelight (2025)
- Context: fleet-wide profiling service combining 42 profilers.
- Did: an engineer "typed an "&" after the auto keyword" to avoid vector copies.
- Numbers: an estimated 15,000 servers in capacity savings per year. LBR profiles feed FDO for "up to 20% reduction in CPU cycles" (about 10-20% fewer servers).
- Lesson: always-on profiling turns one-character fixes into fleet-sized wins. Rules 2, 15. [S2]

## Netflix, Generational ZGC (2024)
- Context: GC pauses in latency-critical Java streaming services.
- Did: moved services to JDK 21 with Generational ZGC.
- Numbers: "More than half of our critical streaming video services" now on it; pauses "usually under one millisecond"; for a given CPU target, ZGC improved both average and P99 latency with equal or better CPU use than G1.
- Lesson: try the collector before the rewrite, and judge it on the tail. Rules 1, 6. [S3]

## Netflix, virtual threads stall (2024)
- Context: Java 21 virtual threads in a Tomcat service on 4 vCPU instances.
- What happened: 4 virtual threads pinned inside `synchronized` while waiting on a lock exhausted the fork-join carrier pool; Tomcat stopped serving.
- Netflix's framing: "a variation of the classic deadlock problem, but instead of 2 locks we have one lock and a semaphore with 4 permits". Not a two-lock deadlock.
- Lesson: load-test new concurrency models with contention at carrier-pool size. Rule 7. [S4]

## Discord, Read States Go to Rust (2020)
- Context: a Go service with a large in-memory cache of Read States.
- Problem: "latency and CPU spikes roughly every 2 minutes" from Go's forced GC run.
- Did: rewrote in Rust; Discord says it "beat Go on every single performance metric" and raised the cache to 8 million Read States.
- Lesson: the spike only shows in the tail and in time series, not in averages. Rewrite when the runtime is the floor. Rules 1, 6. [S9]

## Zepto, Kubernetes CPU throttling (2025)
- Context: Go services on Kubernetes with sidecars.
- Problem: the HPA averaged CPU across app and sidecars, so an app at 80% looked like 68% and never scaled; Go saw node cores, not the container limit.
- Did: realistic requests and limits, GOMAXPROCS set to container CPUs, dashboards on throttling instead of usage.
- Numbers: up to 95% latency cut on critical APIs. Their line: "CPU usage lies. Throttling doesn't."
- Lesson: alert on throttling; make the runtime see its real limit. Rule 5. [S10]

## Zepto, pprof Part 1 and low-latency Part 2 (2025)
- Part 1: profiling cut p99 of a critical high-throughput service by ~40% and pods by 35% at the same throughput. Default SIGPROF interval ~10 ms. [S11]
- Part 2 context: a service at millions of requests per minute drifted from 25 ms to 200 ms+ p99. Bigger nodes (c6i.2xl up to 12xl) and node isolation did not fix it.
- Part 2 findings: string split/trim/concat was 20% of CPU and 50% of allocations; a goroutine finishing in microseconds still took ~39 ms end to end (scheduling). DB cache miss 3% to 0.1%, page evictions 10,000/s to low hundreds/s, miss penalty ~200 ms to ~1 ms, read/write latency down ~50%. Cache warm-up moved to S3 written by one scheduled pod. Peak pods down 50%. [S12]
- Lesson: hardware does not fix allocation churn or cache misses; re-measure end to end. Rules 2, 4, 9, 10.

## Zepto, webpack builds (2026)
- Context: micro-frontend platform, 20 apps, 600+ PRs a month, builds over 20 min.
- Profile: 13+ min inside Terser alone.
- Did: swapped the minifier, not the bundler. esbuild minify ~295 ms vs Terser ~6.7 s, output within ~5-8% of best size.
- Numbers: build 20 min to ~2 min with no bundler migration.
- Lesson: profile the build like a service, then fix the biggest share. Rule 3. [S13]

## Zepto, Postgres connection pools (2025)
- Problem: each pod had a hardcoded pool 10x what it needed (20 active + 70 overflow); Postgres `idle_session_timeout` of 5 min killed idle pooled connections.
- Did: pool max about 20% above observed peak, lower connection timeout, idle connections capped at 5-10% of active.
- Lesson: size pools from measured peak, not defaults. Rule 8. [S14]

## Zepto, product enrichment on MongoDB (2026)
- Problem: write amplification from denormalized documents pushed dirty cache past 20%, triggering eviction and spiking reads from under 10 ms to 100 ms+.
- Did: normalized storage so 1,000+ writes per product update became one (99.9% fewer writes).
- Numbers: cluster 7 x M40 to 3 x M60 (85% less infra), API latency down 50-70%, per-pod throughput 3-4K to 15K req/min.
- Lesson: read latency spikes can be a write problem; profile the data path too. Rule 4. [S27]

## Uber, CacheFront (2024)
- Did: Redis integrated into Docstore's query engine.
- Numbers: one use case at 6M+ RPS with a 99% hit rate on about 3K Redis cores, versus about 60K CPU cores to serve it from storage. Fleet: over 40M requests/s (Feb 2024); a 2025 follow-up reports 150M+ reads/s.
- Lesson: a cache tier with a tracked hit rate is often the cheapest big win. Rule 9. [S15]

## Uber, CI costs (2025)
- Context: SubmitQueue keeps mainline green for a large monorepo.
- Numbers: Uber reports CI resource usage down 53% and wait times down 37%; more than 4,500 engineers rely on it daily.
- Lesson: CI is a performance target too, and perf gates in CI must be budgeted against its cost. Rule 15. [S8]

## Netflix, Maestro engine (2025)
- Context: Maestro orchestrates Netflix workflows at scale.
- Numbers: "improved its performance by 100X"; engine overhead from seconds to milliseconds; shipped to open source.
- Lesson: platform overhead per step compounds across millions of jobs; measure it per step. Rule 3. [S24]

## LinkedIn, Liger Kernel (2024)
- Did: open-sourced Liger-Kernel, LLM training kernels compared against Hugging Face implementations (Llama, Gemma, Qwen).
- Numbers: LinkedIn reports training throughput +20% and memory -60%.
- Lesson: for GPU work, kernel-level fusion is a profile-driven fix like any other. See llm-inference.md. [S7]

## Desert Ant Labs, Voz and Clear on Mac (2026)
- Desert Ant Labs reports its on-device speech model ran encoder and decoder one after the other, and Clear serialized runs on one shared set of Core ML input buffers.
- Did: decoder on its own thread starting per window as soon as the encoder finishes, decoding on CPU so the Neural Engine only encodes, four windows encoded at a time; a buffer pool so Clear runs overlap.
- Numbers: 10 minutes of audio on M3 Ultra went from 221x to 640x realtime (2.7s to 0.9s); M1 only 223x to 228x; "Every transcript in those tests was identical, word for word."
- Lesson: pipeline stages across idle units, prove output is unchanged, and report per-device results since gains scale with the chip. Rules 3, 11. [S28]

## Desert Ant Labs, one model across runtimes (2026)
- Desert Ant Labs reports per-photo latency for one image classifier on an M6 Mac, by runtime.
- Numbers: Core ML on the Neural Engine 5.5ms, LiteRT GPU 30ms, LiteRT.js WebGPU in Chrome 41ms, LiteRT CPU 4 threads 49ms, LiteRT.js WebAssembly with threads 176ms.
- Lesson: runtime choice alone moves latency about 32x for the same model and machine; state the runtime with every number. Rule 11. [S29]
