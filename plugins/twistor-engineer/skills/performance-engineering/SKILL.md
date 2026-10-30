---
name: performance-engineering
description: 'Use when a service, API, build or LLM endpoint is slow or too costly, p99 spikes, CPU is throttled, GC pauses bite, pods keep scaling out, pools run dry, or someone types profile this, why is p99 high, make it faster, cut infra cost, benchmark this, TTFT too slow. Covers percentiles, pprof, flame graphs, continuous profiling, GC, virtual threads, Kubernetes CPU limits, GOMAXPROCS, connection reuse, cache tiers, TTFT, ITL, throughput at concurrency, prompt caching and tool output sizing. Gives sourced instead-of rules, a fixed answer shape and a CI regression guard. Not for Core Web Vitals or page load (use agent-skills performance-optimization or Cloudflare web-perf), app startup and frame budgets (use frontend-mobile-engineering), or capacity design (use system-design-scaling).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Measure the tail, profile, fix the biggest share first, re-measure, then guard it in CI. Backend, build and LLM inference.'
---

# Performance Engineering

A performance claim without a percentile, a load level and a profile is a guess. The teams in this skill's sources won by measuring the tail, finding the one hot spot that owned most of the cost, fixing only that, and proving it at the same throughput. Bigger machines and rewrites came last, if at all.

## When to use / not

- Use: latency or tail regressions, CPU or memory cost, slow builds or CI, GC pauses, thread or pool starvation, container CPU throttling, cache hit rate questions.
- Use: LLM serving and agent cost, TTFT, ITL, throughput at concurrency, prompt caching, tool output size, benchmark design.
- Not: Core Web Vitals, LCP, INP, CLS, bundle-on-the-wire tuning (use agent-skills performance-optimization or the Cloudflare web-perf skill).
- Not: mobile startup, frame drops, app-side budgets (use frontend-mobile-engineering).
- Not: picking a data store, sharding or region topology for growth (use system-design-scaling).
- Not: an outage in progress (use reliability-incident-response; come back here once stable).

## Rules

1. **Instead of** reporting mean latency or one benchmark number, **do** report p50, p95 and p99 at a stated load and concurrency, with the run config. **Why:** averages hide the tail; Discord reports Go GC forced "latency and CPU spikes roughly every 2 minutes", and Netflix judged ZGC on P99 as well as average. [S9][S3]
2. **Instead of** guessing the hot path from code reading, **do** take CPU and allocation profiles under production-like load before changing anything. **Why:** Zepto reports string split, trim and concat were 20% of CPU and 50% of allocations, and Meta's Strobelight post reports a one-character "&" fix worth an estimated 15,000 servers of capacity per year. [S12][S2]
3. **Instead of** fixing whatever is easiest, **do** rank by share of total time or cost and attack the largest share first. **Why:** Zepto reports 13+ of 20 build minutes were inside Terser, so swapping only the minifier took the build to about 2 min with no bundler migration. [S13]
4. **Instead of** moving to bigger nodes when p99 drifts, **do** profile the code and data path first. **Why:** Zepto reports c6i.2xl up to 12xl nodes and node isolation did not fix a 25 ms to 200 ms+ p99 drift, while code and cache fixes later cut peak pods by 50%. [S12]
5. **Instead of** watching container CPU usage, **do** alert on CPU throttling and set runtime parallelism such as GOMAXPROCS to the container limit. **Why:** Go saw node cores and the HPA averaged the app with sidecars; Zepto reports up to 95% lower latency on critical APIs after fixing requests, limits and GOMAXPROCS. [S10]
6. **Instead of** rewriting a service in another language for tail latency, **do** measure GC pauses and try a low-pause collector first, and rewrite only when the runtime is the floor. **Why:** Netflix runs over half its critical streaming services on JDK 21 Generational ZGC with pauses "usually under one millisecond"; Discord rewrote Read States in Rust after Go GC forced spikes every ~2 min. [S3][S9]
7. **Instead of** adopting virtual threads as a drop-in swap, **do** load-test with lock contention and at least as many blocked tasks as carrier threads, and keep thread dumps ready. **Why:** Netflix hit "a variation of the classic deadlock problem" where 4 virtual threads pinned in `synchronized` on a 4 vCPU instance exhausted the carrier pool and Tomcat stopped serving. [S4]
8. **Instead of** opening upstream connections per request or per worker, **do** share and reuse them, and size pools from observed peak. **Why:** Cloudflare reports Pingora lifted reuse for one major customer from 87.1% to 99.92% and used about 70% less CPU and 67% less memory; Zepto set pool max about 20% above observed peak after pools ran 10x too large. [S1][S14]
9. **Instead of** serving hot reads from the primary store, **do** put a cache tier in front and track hit rate and miss penalty per tier. **Why:** Uber reports 6M+ RPS at a 99% hit rate on about 3K Redis cores versus about 60K cores from storage; Zepto cut miss rate from 3% to 0.1% and miss penalty from ~200 ms to ~1 ms. [S15][S12]
10. **Instead of** trusting a microbenchmark win, **do** re-measure end to end at the same throughput and report pods or cost saved. **Why:** Zepto reports a goroutine finishing in microseconds still took ~39 ms end to end, and counted its pprof win as p99 down ~40% and pods down 35% at the same throughput. [S12][S11]
11. **Instead of** comparing runs from different machines or configs, **do** pin and document hardware and resource limits, repeat across times and days, and distrust small gaps. **Why:** Anthropic measured a 6 percentage point swing on Terminal-Bench 2.0 from resource configuration alone. [S23]
12. **Instead of** quoting LLM tokens per second, **do** report TTFT and ITL percentiles plus throughput at a fixed concurrency. **Why:** in Red Hat's llm-d test TTFT P95 fell from 745 to 272 ms while throughput went from 0.52 to 0.49 req/s, a trade one number hides; GuideLLM suggests TTFT under 200 ms for chat. [S6][S5]
13. **Instead of** resending a large static prompt on every call, **do** put a byte-stable prefix first, cache it, and keep the same model for the session. **Why:** caching took a 100K-token prompt from 11.5 s to 2.4 s TTFT, and cached tokens cost 10% of base input. [S19][S20]
14. **Instead of** piping raw tool output into the model, **do** cap, paginate and filter it in code before it reaches context. **Why:** Claude Code caps tool responses at 25,000 tokens by default (configurable), and code execution cut one MCP workflow from 150,000 to 2,000 tokens. [S16][S17]
15. **Instead of** profiling only during incidents, **do** run always-on fleet profiling and a CI perf budget that fails on regression. **Why:** Meta runs 42 profilers under Strobelight and uses its LBR profiles for FDO worth up to 20% fewer CPU cycles. [S2]

## Answer shape

Every answer follows these sections, in order. Skip none; write "unknown" and the step to get it.

1. **Target.** The metric, percentile, load level and budget (for example p99 under X ms at Y RPS, or TTFT P95 at concurrency N). Name the user-facing effect.
2. **Baseline.** Current p50 / p95 / p99 (TTFT and ITL for LLMs), throughput, error rate, cost or pod count, plus the exact run config. Rules 1, 11, 12.
3. **Profile.** Where time, CPU, allocations, waits or tokens go, as ranked shares. Name the profiler and sample interval. Rules 2, 5, 14.
4. **Biggest share first.** The one change aimed at the top share, its cost, and why the cheaper options (config, flag, collector, pool size, cache) do or do not cover it. Rules 3, 4, 6, 8, 9, 13.
5. **Re-measure.** Before / after table at the same load and config, with percentiles and resource or cost delta. Rule 10.
6. **Regression guard.** The CI budget or benchmark gate, the production alert (throttling, tail, hit rate, cache miss), and who owns it. Rule 15.
7. **What not to do.** Moves that look tempting here and why they lose (bigger boxes, rewrite, average-only dashboards).
8. **Open questions.** Missing data that could change the call.

### Where to look first, by symptom

| Symptom | First measurement | Likely lever | Rule |
|---|---|---|---|
| p99 spikes on a regular beat | latency time series at 1 s resolution, GC logs | collector choice, heap sizing, allocation cuts | 1, 6 |
| Latency rises but CPU "looks fine" | throttled periods, runtime core count | requests and limits, GOMAXPROCS | 5 |
| p99 drifts up as traffic grows | CPU and allocation profile, scheduler latency | hot-path allocations, cache miss rate | 2, 4, 9 |
| Service stops serving, threads idle | thread dump, carrier pool state | remove pinning, narrow locks | 7 |
| Connection errors or slow first byte | new connections per second, reuse rate, pool waits | shared pools sized from peak | 8 |
| Reads slow after a write burst | dirty cache, evictions, writes per update | batch or normalize writes | 4, 9 |
| Build or CI takes minutes | build profile per plugin or step | swap the slowest step, not the tool | 3 |
| LLM chat feels slow | TTFT and ITL P95 at target concurrency | prefix caching, KV-aware routing | 12, 13 |
| Agent burns tokens | tokens per task, tool output sizes | cap and filter tool output, load tools on demand | 14 |

### Output skeleton

```
Target:     p99 < __ ms at __ RPS (or TTFT P95 < __ ms at __ concurrent)
Baseline:   p50 __ / p95 __ / p99 __ | throughput __ | pods __ | config: __
Profile:    1) __ __%  2) __ __%  3) __ __%   (profiler, interval)
Fix:        <one change aimed at #1>   cost: __   cheaper options ruled out: __
Re-measure: p50 __ / p95 __ / p99 __ | pods __ | same load, same config
Guard:      CI gate __ | alert __ | owner __
Not doing:  __ because __
Open:       __
```

## Red flags

- "The average looks fine." -> Pull p95 and p99 at peak load before closing anything.
- "I know where it is slow." -> Take a profile anyway; the hot spot is often a library call, a copy or a lock.
- "Let's just give it bigger nodes." -> Profile first; throttling, GC and cache misses survive a node upgrade.
- "Rewrite it in Rust." -> Show the GC or runtime floor in a profile and that tuning was tried.
- "CPU is only at 60%, it can't be CPU." -> Check throttling counters and the runtime's view of available cores.
- "Virtual threads are free concurrency." -> Load-test with locks held across blocking calls.
- "The microbenchmark is 10x faster." -> Re-run end to end at the same throughput before you claim it.
- "Run A beat run B by 2 points." -> Same hardware, same limits, repeated runs, or it is noise.
- "Tokens per second went up, ship it." -> Check TTFT P95 and ITL at the target concurrency too.
- "We'll add a perf test later." -> The guard ships with the fix, in the same PR.
- "The cache will fix it." -> Measure hit rate and miss penalty first; a cache that misses adds a hop.
- "Raise the pool size." -> Check observed peak and server-side idle timeouts before adding connections.
- "It is fast on my laptop." -> Reproduce at production load, limits and data size.
- "The model is slow." -> Split TTFT from ITL and check the cached-token share before blaming the model.

## Verification

Do not call it done until you can show:

- A before / after table with p50, p95 and p99 (or TTFT and ITL percentiles) at the same load, concurrency and config.
- The profile (or its top entries) that justified the change, and a second profile showing the share dropped.
- Resource or cost delta at equal throughput: pods, cores, memory, tokens or dollars.
- Run config written down: hardware, limits, warm-up, number of runs, dataset or prompt set.
- The CI gate or alert that would have caught this regression, linked in the PR.
- For LLM work: cache hit rate and cached-token share, plus tool output sizes after the cap.
- The change is scoped to the top-ranked share; unrelated tweaks are split into their own PRs so each can be measured.

## References

| File | Read when |
|---|---|
| references/llm-inference.md | tuning LLM serving or agents: TTFT, ITL, throughput at concurrency, caching levers, tool output size, benchmark hygiene |
| references/case-studies.md | you need the full story and numbers behind a rule |
| references/checklist.md | reviewing a perf PR or shipping a perf fix |
| references/sources.md | citing a figure or checking a source status |

Companion skills (link only): agent-skills performance-optimization and the Cloudflare web-perf skill for Core Web Vitals; frontend-mobile-engineering for app-side startup and frame budgets; system-design-scaling for capacity and topology.
