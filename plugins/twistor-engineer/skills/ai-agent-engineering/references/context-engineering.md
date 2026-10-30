# Context engineering

Context engineering is "the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference". Treat the window as a finite budget. Rules 7 to 10 in SKILL.md. [S6]

## Why the budget is real

- Context rot: recall degrades as the window fills. Attention spans "n² pairwise relationships for n tokens". [S6]
- On BrowseComp, token usage alone explained 80% of performance variance and three factors (tokens, tool calls, model choice) explained 95%. This is one benchmark, not a law for every agent. [S4]

## Techniques, lightest first (rule 7)

1. **Just-in-time retrieval.** Hold lightweight identifiers (file paths, queries, links) and fetch on demand. Hybrid preload plus just-in-time is often best. [S6]
2. **Tool result clearing.** Drop old raw tool outputs; the lightest form of compaction. [S6]
3. **Context editing.** On the Claude Developer Platform, context editing alone gave a 29% improvement; on a 100-turn search eval it cut tokens 84%. [S25]
4. **Memory outside the window.** Structured notes in a file or memory store. The Pokemon example tracked progress across 1,234 steps. Context editing plus the memory tool improved performance 39% over baseline. The memory tool is client-side; you pick the storage. [S6][S25]
5. **Memory folder.** Sonnet 4.5 went from 60.4% to 67.2% on BrowseComp-Plus with a memory folder. [S26]
6. **Compaction.** Summarize near the limit and restart from the summary. Tune the compaction prompt for recall first, then precision. Claude Code keeps "the five most recently accessed files" after compaction (as of Sep 2025). [S6]

## Subagents as context firewalls (rule 8)

- Subagents use "tens of thousands of tokens or more" and return "a condensed, distilled summary of its work (often 1,000-2,000 tokens)". [S6]
- Isolate subtasks producing "more than 1000 tokens" of mostly irrelevant output and return a "50-100 tokens" summary. [S23]

## Progressive disclosure and skills

- Agent Skills load in three levels: metadata at startup, full SKILL.md when relevant, linked files only as needed. The description drives triggering, so write it carefully. Keep the core file lean. [S8]
- Bundle scripts for deterministic work; the agent can run them without loading code or data into context. [S8]
- Only skill frontmatter sits in the prompt by default. [S26]
- Rerun skill evals on each new model and retire skills the base model no longer needs; tuning descriptions improved triggering on 5 of 6 public skills. [S31]

## Caching

- Cached tokens cost 10% of base. Put stable content first, append updates as messages, and do not switch models mid-session. [S26]

## Retrieval for agents (rule 9)

- Start with agentic search (bash, grep, tail over files). Add semantic search only for faster results or more variations; it is less accurate, harder to maintain and less transparent. This is vendor advice, not a benchmark. [S7]
- Under 200,000 tokens of knowledge base, skip RAG and put it all in the prompt. [S14]
- Contextual Retrieval: prepend 50-100 tokens of generated chunk context before embedding and BM25 indexing. [S14]

| Setup | Top-20 retrieval failure | Reduction |
|---|---|---|
| Baseline | 5.7% | - |
| Contextual embeddings | 3.7% | 35% |
| + contextual BM25 | 2.9% | 49% |
| + reranking (top 150, rerank, pass 20) | 1.9% | 67% |

One-time cost with prompt caching: $1.02 per million document tokens. [S14]

## Thinking (rule 10)

- The think tool helps for tool-output analysis, policy-heavy environments and costly sequential decisions; skip it for non-sequential calls or simple instructions. [S3]
- tau-bench airline pass^1: page text says 0.370 baseline to 0.570 with think tool plus optimized prompt (54% relative); the page's own table says 0.332 to 0.584 at k=1. Quote both or neither. [S3]
- Retail: think tool with no prompt 0.812 vs baseline 0.783; extended thinking 0.770. SWE-bench: think tool added 1.6% on average. [S3]
- 2025-12-15 update: extended thinking is now recommended over a dedicated think tool in most cases. Default to it. [S3]
