# Checklist

Use before shipping a perf fix or when reviewing a perf PR. Each item maps to a rule in SKILL.md.

## Measure
- [ ] Target states metric, percentile, load and budget, not "faster". (Rule 1)
- [ ] Baseline has p50, p95 and p99 at a stated load and concurrency, not a mean. (Rule 1)
- [ ] Run config is written down: hardware, limits, warm-up, runs, dataset. (Rule 11)
- [ ] Comparison runs use the same machine class and limits, repeated more than once. (Rule 11)
- [ ] LLM work reports TTFT and ITL percentiles plus throughput at fixed concurrency. (Rule 12)

## Profile
- [ ] A CPU and allocation profile under realistic load is attached or linked. (Rule 2)
- [ ] Shares are ranked; the change targets the top one. (Rule 3)
- [ ] Container services checked for throttling and runtime core count (for Go, GOMAXPROCS). (Rule 5)
- [ ] GC pause data checked before any runtime or language change. (Rule 6)
- [ ] Data path checked: cache hit rate, miss penalty, write amplification, pool waits. (Rules 4, 9)

## Fix
- [ ] Cheaper levers (config, collector, pool size, cache) ruled in or out before a rewrite or bigger nodes. (Rules 4, 6)
- [ ] Connection pools sized from observed peak; reuse rate measured. (Rule 8)
- [ ] Virtual threads or new async model load-tested with lock contention at carrier-pool size. (Rule 7)
- [ ] Prompts have a byte-stable cached prefix; model is not switched mid-session. (Rule 13)
- [ ] Tool outputs capped, paginated or filtered in code. (Rule 14)

## Prove
- [ ] Before / after table at the same throughput and config. (Rule 10)
- [ ] Resource or cost delta shown: pods, cores, memory, tokens or dollars. (Rule 10)
- [ ] Second profile shows the targeted share dropped. (Rules 2, 10)
- [ ] Changes that overlap or reorder work show identical output before and after, and report results per device or runtime. (Rule 11)

## Guard
- [ ] CI perf budget or benchmark gate added in the same PR, with its CI cost considered. (Rule 15)
- [ ] Production alert on the right signal: tail latency, throttling, hit rate. (Rules 1, 5, 9, 15)
- [ ] Continuous profiling enabled or confirmed for the service. (Rule 15)
