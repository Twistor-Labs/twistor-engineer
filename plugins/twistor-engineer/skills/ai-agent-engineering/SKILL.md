---
name: ai-agent-engineering
description: 'Use when designing or debugging an LLM agent, tool-using assistant or agent harness: "agent or workflow?", "agent picks the wrong tool", "context keeps filling up", "agent forgets what it did", "too many MCP tools", "should I add subagents", "agent drifts over hours", exploding token bills. Covers tool design, context engineering, compaction, agent RAG, multi-agent orchestration, long-running harnesses, think tool vs extended thinking, MCP gateways, Agent SDK, Agent Skills. Gives a workflow-vs-agent decision, tool and context plan, token budget and done criteria, backed by sourced Anthropic, OpenAI, Uber and LinkedIn numbers. Not for eval suite or grader design (use llm-evals), sandboxing or prompt-injection defense (use security-engineering), or editor-level AI coding habits (use dev-productivity-ai-coding).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Workflow before agent, tools as interfaces, context as a budget, evaluator separate from generator.'
---

# AI Agent Engineering

Most agent failures are design failures: an agent where a workflow would do, tools written for humans, a context window treated as free, and a generator grading its own work. This skill holds the line on the simplest thing that works, then adds autonomy only where measurements say it pays. Rules come from Anthropic's engineering posts (2024-2026), OpenAI's agent guide, and Uber and LinkedIn production reports.

## When to use / not

- Use: choosing between a single call, a workflow, one agent or many; designing or renaming tools and MCP servers; an agent that loses state, loops, overspends tokens or picks wrong tools; planning a harness for multi-hour or multi-session work; RAG or memory for an agent.
- Not: building the eval set, graders, pass@k math (see llm-evals). Sandboxes, egress allowlists, credential vaults, prompt-injection defense (see security-engineering). Editor-level habits for coding with an assistant (see dev-productivity-ai-coding).

## Rules

1. **Instead of** reaching for an autonomous agent or a framework first, **do** try one LLM call, then a fixed workflow (chain, route, parallelize, orchestrator-workers, evaluator-optimizer), and grant agent autonomy only where it measurably beats that baseline. **Why:** Anthropic says "find the simplest solution possible" and OpenAI says maximize a single agent before splitting. [S1][S17][S24]
2. **Instead of** adding subagents because the problem feels big, **do** stay single-agent unless the work is breadth-first, parallelizable and valuable enough to pay for the tokens. **Why:** agents use about 4x chat tokens and multi-agent about 15x, while Opus 4 lead plus Sonnet 4 subagents beat single-agent Opus 4 by 90.2% only on parallel research. [S4][S23]
3. **Instead of** one-line tool descriptions written for humans, **do** design the tool interface like an HCI: namespaced names, clear parameters, 1-5 examples for ambiguous schemas, and poka-yoke inputs such as absolute paths. **Why:** tool use examples raised complex parameter accuracy from 72% to 90%, and absolute paths fixed edit failures after the agent changed directory. [S1][S10][S5]
4. **Instead of** returning raw API payloads from tools, **do** cap, paginate and filter output and expose a `response_format` enum (concise or detailed). **Why:** Claude Code caps tool responses at 25,000 tokens by default, and the Slack example returned 72 tokens concise vs 206 detailed. [S5]
5. **Instead of** loading every tool definition upfront, **do** keep the 3-5 most used tools loaded and defer the rest behind tool search once definitions pass 10K tokens or 10 tools. **Why:** tool search cut definitions from about 77K to 8.7K tokens and lifted Opus 4 on MCP evals from 49% to 74%; tool choice degrades past 30-50 definitions. [S10][S21]
6. **Instead of** routing intermediate data through the context window, **do** let the agent call tools from code in a sandbox and return only filtered results. **Why:** a Drive-to-Salesforce task fell from 150,000 to 2,000 tokens (98.7%), and Uber reports code mode cut tokens 55-71% on small SQL tasks. [S9][S20]
7. **Instead of** filling the window and hoping recall holds, **do** budget context: just-in-time retrieval by file path or ID, clear stale tool results, compact near the limit, keep notes in memory outside the window. **Why:** recall degrades as tokens grow, and context editing plus a memory tool improved an agentic eval 39% over baseline while cutting tokens 84% on a 100-turn search. [S6][S25]
8. **Instead of** letting subagents hand back everything they read, **do** give each one an objective, output format, tool guidance and boundaries, and demand a condensed summary. **Why:** subagents may burn tens of thousands of tokens but should return about 1,000-2,000, and work producing over 1,000 mostly irrelevant tokens belongs in isolation. [S4][S6][S23]
9. **Instead of** standing up a vector index on day one, **do** start with agentic search over files, skip RAG under 200,000 tokens of knowledge, and when you need RAG add chunk context, BM25 and reranking. **Why:** contextual embeddings plus contextual BM25 plus reranking cut top-20 retrieval failures 67% (5.7% to 1.9%). [S7][S14]
10. **Instead of** bolting on a dedicated think tool by default, **do** enable extended thinking and keep a think tool only for policy-heavy sequential tool chains where you measured a gain. **Why:** a 2025-12-15 update prefers extended thinking in most cases, and the original airline gain is reported as 0.370 to 0.570 in the text but 0.332 to 0.584 in the table. [S3]
11. **Instead of** one endless session for multi-hour work, **do** split an initializer from a worker role, persist a progress file, a JSON feature list where agents only flip `passes`, git commits per feature, and a start ritual that runs an end-to-end test. **Why:** Anthropic's harness for a 200+ feature app used exactly this so each fresh session recovers state without trusting memory. [S11]
12. **Instead of** asking the generator to critique its own output, **do** run a separate skeptical evaluator in fresh context against criteria agreed before the work, with a hard iteration cap. **Why:** tuning a standalone evaluator to be skeptical was "far more tractable" than self-critique, and a $9 solo run shipped a broken core feature that the $200 harness caught. [S12][S22][S24]
13. **Instead of** trusting the model to refuse bad actions, **do** keep credentials out of generated code and put every tool behind one gateway with tool-level authz, PII redaction and owner approval for new tools. **Why:** prompt-injection success rose from about 0.1% on one try to 5-6% after 100 adaptive tries, and Uber reports its MCP Gateway fronts 800+ servers and 5,000+ tools this way. [S29][S13][S19]
14. **Instead of** shipping when the demo feels 80% there, **do** start an eval of real cases before launch (10 golden examples is enough to begin), run it more than once, and budget the last mile explicitly. **Why:** LinkedIn reports 80% of the experience in the first month and four more months to pass 95%, and one regression showed in only one run of two. [S18][S21]
15. **Instead of** running every step on the biggest model, **do** route by difficulty or run a cheap executor that consults a stronger advisor only on hard calls, capped with `max_uses`. **Why:** Haiku with an Opus advisor scored 41.2% on BrowseComp vs 19.7% alone, still cheaper than Sonnet, and Sonnet with an advisor gained 2.7 points at 11.9% lower cost per task. [S27][S1]

## Answer shape

Every answer from this skill follows these sections, in order. Skip a section only by saying why.

1. **Decision.** Single call, workflow (name the pattern), single agent, or multi-agent, using the table below. One line on why the simpler option fails, or why it does not.
2. **Loop and models.** The loop (gather context, act, verify, repeat [S7]), which model runs which step, and any advisor or routing.
3. **Tools.** Name, namespace, input schema notes, output cap and format, which load upfront vs deferred, which run from code.
4. **Context plan.** What is preloaded, what is fetched just in time, when to clear and compact, where notes persist, what subagents return.
5. **Done criteria and evaluator.** Testable criteria agreed before work, who checks (separate evaluator), iteration cap, and the starter eval set (defer design to llm-evals).
6. **Boundary.** What the agent can touch, where credentials live, which actions need a human (defer mechanics to security-engineering).
7. **Cost.** Token multiplier vs chat (about 4x agent, about 15x multi-agent [S4]), per-task estimate, and what you cut first if it is too high.
8. **Open questions.** What you could not decide without data.

### Workflow vs agent

| Task signal | Pick | Source |
|---|---|---|
| Steps known in advance, fixed order, each checkable | Prompt chaining workflow | [S1] |
| Inputs fall into distinct categories with different handling | Routing (also routes easy work to a cheaper model) | [S1] |
| Independent subtasks, or you want several votes | Parallelization; design the aggregation step first | [S1][S24] |
| Subtasks cannot be known until the input is read | Orchestrator-workers | [S1] |
| Clear quality criteria and refinement measurably helps | Evaluator-optimizer with a hard cap | [S1][S24] |
| Open-ended, step count unpredictable, results verifiable, environment trusted | Single agent with tools | [S1][S17] |
| Breadth-first research, parallel branches, high task value | Orchestrator plus subagents | [S4][S22] |
| Tightly coupled work, shared files, sequential dependencies | Single agent; fold subagents back in | [S4][S21] |
| Latency is the bottleneck and tasks are independent | Parallel; otherwise default to sequential | [S24] |

### Cost quick reference

| Lever | Reported effect | Source |
|---|---|---|
| Agent vs chat | about 4x tokens | [S4] |
| Multi-agent vs chat | about 15x tokens (another post: 3-10x) | [S4][S23] |
| Tool search | definitions 77K to 8.7K tokens | [S10] |
| Code execution with MCP | 150,000 to 2,000 tokens on one task | [S9] |
| Programmatic tool calling | 37% fewer tokens on complex research | [S10] |
| Context editing | 84% fewer tokens on a 100-turn search eval | [S25] |
| Prompt cache reads | 10% of base input price | [S26] |
| Advisor (Sonnet executor) | 11.9% lower cost per task, +2.7 points | [S27] |

## Red flags

- "We'll use a framework so it's production-ready." -> Write the loop against the raw API first; frameworks hide prompts and tool calls you will need to debug [S1].
- "More agents will make it smarter." -> Measure the single-agent baseline and the token multiplier before adding one [S4][S23].
- "The model will figure out which of these 60 tools to use." -> Group by intent, defer the rest behind tool search [S21][S10].
- "Just give it the whole document." -> Filter in code, pass identifiers, fetch on demand [S9][S6].
- "It said it's done." -> A separate evaluator or a test run says it's done, not the generator [S12].
- "Tokens explain 80% of agent performance." -> That is BrowseComp only; three factors explain 95% there [S4].
- "This scaffold worked on the last model." -> Re-test each harness piece per model; Anthropic dropped its sprint construct with Opus 4.6 [S12].
- "Let's add a think tool, it helped on tau-bench." -> Try extended thinking first; the think-tool page's own numbers disagree with each other [S3].
- "We need a vector DB for the docs." -> Under 200,000 tokens, put it in the prompt; otherwise start with file search [S14][S7].
- "The evaluator can loop until it passes." -> Cap iterations and hand off or return best attempt with caveats [S22][S24].
- "It's only one run, but it passed." -> Run at least twice; regressions can show in one run of two [S21].
- "Users will approve risky actions." -> They approve about 93% of prompts; move the boundary into the environment [S30].

## Verification

Evidence before "done". Show, do not assert:

- The single-call or workflow baseline result next to the agent result on the same tasks.
- Eval runs (at least two) on real cases with pass rate, tokens per task, tool-call count and tool errors [S5][S21].
- A trace of one full run showing tool inputs, output sizes and where context was cleared or compacted [S28].
- For long-running work: the progress file, feature list diff and the end-to-end test output from the last session [S11].
- Token cost per task measured, not estimated, and compared to the budget in the answer's cost section.
- A fresh-context adversarial pass: hand the design and one transcript to a reviewer that never saw your reasoning, and ask it to break the done criteria. Fix or record every hit before claiming done.

## References

| File | Read when |
|---|---|
| references/tool-design.md | writing or renaming tools, MCP servers, tool output formats, tool search |
| references/context-engineering.md | the window fills up, the agent forgets, compaction, memory, RAG, skills |
| references/multi-agent.md | deciding on subagents, coordination patterns, advisor and routing |
| references/long-running-harness.md | work spans hours or sessions, initializer/worker, evaluator loops |
| references/case-studies.md | you need the full story and numbers behind a rule |
| references/checklist.md | reviewing an agent design or shipping one |
| references/sources.md | citing any figure |
