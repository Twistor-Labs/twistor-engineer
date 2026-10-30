# LLM inference and agent performance

Read when tuning an LLM serving stack, an agent loop, or the token bill. Same loop as the main skill: measure percentiles, profile where time and tokens go, fix the biggest share, re-measure, guard.

## Metrics that matter

| Metric | Meaning | Note |
|---|---|---|
| TTFT | time to first token | what a chat user feels first; GuideLLM suggests under 200 ms for chat [S5] |
| ITL | inter-token latency, gap between streamed tokens | sets perceived streaming speed [S5] |
| Throughput at concurrency | completed requests per second at a fixed number of concurrent users | always pair with TTFT and ITL percentiles [S6] |
| Cache hit rate | share of prompt tokens served from a prefix or KV cache | the lever behind most TTFT wins below [S6][S19] |
| Tokens per task | input plus output tokens for one completed task | for agents this is the cost and latency driver [S17][S18] |

Report P50 and P95 for TTFT and ITL. A single "tokens per second" number hides the trade.

Worked example (Red Hat llm-d, single node, 4x L4, multi-turn workload): TTFT P50 123 -> 92 ms, P95 745 -> 272 ms, KV cache hit about 62% -> about 90%, throughput 0.49 req/s for llm-d vs 0.52 for the vLLM baseline. Tail latency improved a lot while throughput dipped slightly. GuideLLM appears there only as the vLLM baseline tool; GuideLLM and llm-d are separate posts, not one benchmark. [S6][S5]

## Caching levers

- Prompt caching: a 100K-token prompt went from 11.5 s to 2.4 s TTFT; a 10K many-shot prompt from 1.6 s to 1.1 s. Cache reads were $0.30 / MTok vs $3 / MTok base, writes $3.75 / MTok (3.5 Sonnet pricing in that post). [S19]
- Anthropic says caching cuts cost "up to 90%" and latency "up to 85%". [S26]
- Cached tokens cost 10% of base. Put stable content first, append updates as new messages, and do not switch models mid-session, since that breaks the cache. [S20]
- Cache prefixes must be byte-stable. Cache writes cost 1.25x normal input price (2x for a 1-hour cache). Pre-warm with a `max_tokens: 0` request. [S21]
- KV-cache-aware routing in the server (llm-d) lifted hit rate from about 62% to about 90% and cut TTFT P95 from 745 to 272 ms. [S6]
- Small knowledge bases: under 200,000 tokens, skip RAG and put it all in the prompt with caching. With caching, contextual chunk enrichment cost $1.02 per million document tokens one time. [S25]

## Tool output and context sizing (agents)

- Cap tool output. Claude Code restricts tool responses to 25,000 tokens by default (configurable). Offer a concise vs detailed `response_format`; the Slack example was 206 tokens detailed vs 72 concise. [S16]
- Filter in code, not in context. Presenting MCP servers as code cut a Google Drive to Salesforce workflow from 150,000 to 2,000 tokens (98.7%); a 10,000-row sheet can reach the model as 5 rows. [S17]
- Load tools on demand. Tool Search cut upfront tool definitions from about 77K to about 8.7K tokens (85%). Programmatic tool calling cut average tokens 37% on complex research tasks (43,588 -> 27,297). [S18]
- Overlap setup with inference. Lazy container provisioning let inference start before the sandbox existed: p50 TTFT down roughly 60%, p95 down over 90%. [S22]
- Output tokens: the `token-efficient-tools-2025-02-19` beta cut output "up to 70%" (average 14%). [S26]

## Training-side lever

- Fused kernels: LinkedIn reports Liger-Kernel improves training throughput by 20% and cuts memory by 60% vs Hugging Face implementations (Llama, Gemma, Qwen). Profile GPU time and memory first, then swap kernels. [S7]

## Benchmarking hygiene

- Pin and document hardware, accelerator count, resource limits, model version, prompt set and concurrency. The llm-d numbers are only meaningful as "single node, 4x L4". [S6]
- Resource config alone moved Terminal-Bench 2.0 scores by 6 percentage points (p < 0.01). Infra error rate fell from 5.8% at strict limits to 2.1% at 3x. Treat gaps under 3 points skeptically unless configs match, and run at different times and days. [S23]
- Warm the cache the same way in both arms, or report cold and warm separately. [S21]
- Fix concurrency per run and sweep it; report the curve, not one point. [S5][S6]
- Keep the before / after harness in the repo so the CI gate reruns the same thing.
