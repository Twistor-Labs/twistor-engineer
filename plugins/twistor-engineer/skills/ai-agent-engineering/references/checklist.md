# Agent design review checklist

Each item ties to a rule number in SKILL.md. An unchecked item is either fixed or written down as a known risk.

## Before building

- [ ] A single-call or fixed-workflow baseline exists and its result is recorded. (rule 1)
- [ ] The workflow-vs-agent table was applied and the chosen pattern is named. (rule 1)
- [ ] Multi-agent is justified by breadth-first, parallel work and a token budget at about 15x chat. (rule 2)
- [ ] Model per step is chosen; routing or an advisor with `max_uses` is considered. (rule 15)
- [ ] Done criteria are written as testable checks before work starts. (rule 12)

## Tools

- [ ] Names are namespaced by service and resource. (rule 3)
- [ ] Ambiguous schemas have 1-5 examples; paths are absolute. (rule 3)
- [ ] Every tool caps and paginates output and offers concise vs detailed. (rule 4)
- [ ] Over 10 tools or 10K tokens of definitions: rare tools deferred behind tool search. (rule 5)
- [ ] Bulk data is filtered in code before reaching the model. (rule 6)
- [ ] CLIs an agent calls offer JSON output, read stdin, document exit codes, and keep going past a failed item in a batch while naming it. (rule 3)
- [ ] New tools pass through a gateway or owner approval; credentials never reach generated code. (rule 13)

## Context

- [ ] Preloaded vs just-in-time content is listed. (rule 7)
- [ ] Stale tool results are cleared; compaction trigger and prompt are defined. (rule 7)
- [ ] Notes or memory persist outside the window for long tasks. (rule 7)
- [ ] Each subagent brief has objective, output format, tool guidance, boundaries, and a summary size. (rule 8)
- [ ] Retrieval starts with agentic search; RAG only above 200K tokens, with chunk context and reranking. (rule 9)
- [ ] Extended thinking is the default; any think tool has a measured gain. (rule 10)

## Long-running work

- [ ] Progress file, JSON feature list and per-feature commits exist. (rule 11)
- [ ] Every session starts by reading state and running an end-to-end test. (rule 11)
- [ ] Evaluator runs in fresh context, separate from the generator, with an iteration cap. (rule 12)
- [ ] Harness pieces were re-tested on the current model. (rule 12)

## Ship gate

- [ ] Eval of real cases (10 golden examples minimum) run at least twice; pass rate, tokens, tool calls and tool errors recorded. (rule 14)
- [ ] Traces reviewed for one full run before widening access. (rule 14)
- [ ] Fresh-context adversarial review tried to break the done criteria; every hit fixed or logged. (rule 12)
- [ ] Cost per task estimated and within budget. (rules 2, 15)
- [ ] Eval design reviewed with llm-evals; sandbox and injection defenses reviewed with security-engineering.
