# Multi-agent systems

Default to one agent. Split only when the work is breadth-first and parallel, and the task value pays for the tokens. Rules 1, 2, 8 and 15 in SKILL.md.

## When it pays (rule 2)

- Opus 4 lead with Sonnet 4 subagents beat single-agent Opus 4 by 90.2% on Anthropic's internal research eval. Best for breadth-first, parallel work; poor fit for tightly coupled tasks. [S4]
- Cost: agents use about 4x chat tokens; multi-agent systems about 15x. [S4] Another Anthropic article puts multi-agent at "typically 3-10x more tokens" and says start with one agent. [S23]
- OpenAI's guide: maximize a single agent's capabilities first. When you split, choose a manager pattern (agents as tools) or a decentralized pattern (handoffs). [S17]
- Folding back helps too: one customer saw "roughly 2x faster runtime" after collapsing subagents into a single loop. Fold in one subagent at a time and keep an end-to-end eval running. [S21]

## Scale effort to the task

- Simple fact-finding: one agent, 3-10 tool calls. Complex research: more than 10 subagents. [S4]
- Parallelize: the lead spins up 3-5 subagents in parallel and each uses 3+ tools in parallel; this cut research time by up to 90% on complex queries. [S4]
- Default to sequential. Move to parallel when latency is the bottleneck and tasks are independent. Design the aggregation step (vote, confidence, defer to specialist) before building the parallel branches. [S24]

## Coordination patterns

- "For most use cases, we recommend starting with orchestrator-subagent." [S22]
- Generator-verifier loops need a cap; on cap, hand to a human or return the best attempt with caveats. [S22]
- Shared-state loops need a stop condition, such as "a convergence threshold (no new findings for N cycles)". [S22]
- Agent teams finish unevenly ("two minutes" vs "twenty"); the coordinator must handle partial completion. [S22]
- Give verifiers explicit criteria (full suite, negative tests, edge cases); the worked example caps at `max_attempts=3`. [S23]

## Briefing a subagent (rule 8)

Each subagent gets four things [S4]:
1. Objective.
2. Output format.
3. Tool guidance.
4. Boundaries (what not to do, when to stop).

And returns a condensed summary, about 1,000-2,000 tokens, not its raw reads. [S6]

## Specialized parallel agents

- Anthropic's C compiler: 16 parallel agents, nearly 2,000 sessions, about $20,000 in API fees, a 100,000-line Rust compiler that builds a bootable Linux 6.9 on x86, ARM and RISC-V. Roles were specialized (dedup, performance, codegen efficiency, code quality, docs). [S15]
- Use a known-good oracle to split a monolithic failing target so parallel agents work on different pieces; GCC played that role for the Linux kernel build. [S15]
- The verifier must be "nearly perfect, otherwise Claude will solve the wrong problem". [S15]

## Model routing and advisors (rule 15)

- Route easy or common questions to a smaller model and harder ones to a larger one. [S1]
- Advisor strategy: a Sonnet or Haiku executor consults Opus only on hard decisions. [S27]
  - SWE-bench Multilingual: Sonnet plus Opus advisor gained 2.7 points at 11.9% lower cost per task.
  - BrowseComp: Haiku plus Opus advisor 41.2% vs 19.7% alone, still cheaper than Sonnet.
  - Cap with `max_uses` (example: 3). Advisor replies run 400-700 text tokens.

## Variance claims, read carefully

- "Token usage explains 80% of variance" holds for BrowseComp only. Three factors (tokens, tool calls, model choice) explained 95% there. Do not generalize it to your agent. [S4]
