# Case studies

Company numbers are self-reported by the company that ran the work.

## Anthropic: building effective agents (2024-12-19)
- Context: patterns seen across many customer agent builds.
- What they did: separated workflows (predefined code paths) from agents (model directs its own process), named five composable patterns, and treated tool design as an HCI problem.
- Numbers: none headline. Concrete fix: relative paths broke after the agent left the root directory; absolute paths were then used "flawlessly".
- Lesson: "find the simplest solution possible"; add complexity "only when it demonstrably improves outcomes". [S1]

## Anthropic: SWE-bench Verified scaffold (2025-01-06)
- Context: a minimal agent scaffold for real GitHub issues.
- What they did: two tools (Bash, Edit), a simple prompt, sample until done or the 200k context runs out.
- Numbers: upgraded Claude 3.5 Sonnet 49% vs previous best 45% on the 500-problem Verified set; many successful runs took hundreds of turns and over 100k tokens.
- Lesson: a strong model with a small, precise toolset beats a heavy scaffold. Check environment and grading failures before blaming the model. [S2]

## Anthropic: think tool (2025-03-20, updated 2025-12-15)
- Context: complex tool chains under policy constraints (tau-bench).
- What they did: gave the model a no-op "think" tool to reason between tool calls.
- Numbers: airline pass^1 0.370 to 0.570 in the text, 0.332 to 0.584 in the table on the same page; SWE-bench +1.6% on average.
- Lesson: the 2025-12-15 update recommends extended thinking over a dedicated think tool in most cases. [S3]

## Anthropic: multi-agent research system (2025-06-13)
- Context: Research feature needing broad, parallel search.
- What they did: Opus 4 lead agent spawning Sonnet 4 subagents with explicit briefs.
- Numbers: 90.2% better than single-agent Opus 4 on internal research eval; about 15x chat tokens; research time cut up to 90% on complex queries; a tool-testing agent made later agents 40% faster.
- Lesson: pay the token bill only for breadth-first, high-value work. The 80% variance figure is BrowseComp only. [S4]

## Anthropic: writing tools for agents (2025-09-11)
- Context: tools built for humans performed poorly for agents.
- What they did: namespaced tools, `response_format` enums, capped outputs, agent-driven tool evals.
- Numbers: 25,000-token default tool response cap in Claude Code; Slack example 206 vs 72 tokens.
- Lesson: test naming and output shape with evals; hold out a test set. [S5]

## Anthropic: context engineering and context management (2025-09-29)
- Context: agents degrading as context fills.
- What they did: just-in-time retrieval, tool result clearing, compaction, memory outside the window.
- Numbers: context editing alone +29%; with memory tool +39% over baseline; 84% fewer tokens on a 100-turn search eval.
- Lesson: context is a budget you spend on purpose. [S6][S25]

## Anthropic: Claude Agent SDK (2025-09-29)
- Context: the Claude Code SDK renamed because people used it beyond coding.
- What they did: loop of gather context, take action, verify work, repeat.
- Numbers: none; search guidance is vendor advice.
- Lesson: start with agentic file search; verify with rules (lint) first, then visual checks; LLM-as-judge is "generally not a very robust method". [S7]

## Anthropic: Agent Skills (2025-10-16)
- Context: giving general agents domain procedures without bloating the prompt.
- What they did: three-level progressive disclosure (metadata, SKILL.md, linked files) plus bundled scripts.
- Lesson: the description drives triggering; build skills from observed failures and audit any bundled code. [S8]

## Anthropic: code execution with MCP (2025-11-04)
- Context: tool definitions and intermediate results flooding context.
- What they did: exposed MCP servers as code the agent calls inside a sandbox.
- Numbers: Drive-to-Salesforce 150,000 to 2,000 tokens (98.7%); 10,000-row sheet, model sees 5 rows.
- Lesson: move data through code, not through the model; budget for sandboxing. [S9]

## Anthropic: advanced tool use (2025-11-24)
- Context: large tool libraries.
- What they did: Tool Search Tool, Programmatic Tool Calling, Tool Use Examples.
- Numbers: definitions 77K to 8.7K tokens (85%); Opus 4 MCP evals 49% to 74%; programmatic calling 37% fewer tokens; examples 72% to 90% parameter accuracy.
- Lesson: defer rarely used tools and show examples where schemas are ambiguous. [S10]

## Anthropic: long-running harnesses (2025-11-26 and 2026-03-24)
- Context: building full apps over many hours and sessions.
- What they did: initializer plus worker roles with progress file and JSON feature list; later a planner, generator and separate skeptical evaluator.
- Numbers: 200+ feature list; solo $9 run shipped a broken core feature vs $200 harness; DAW build $124.70 in 3 hr 50 min.
- Lesson: state on disk, evaluator separate from generator, and re-test each harness piece per model. [S11][S12]

## Anthropic: Managed Agents (2026-04-08)
- Context: running agents as a hosted service.
- What they did: split harness, sandboxes and an append-only session log; credentials never reach generated code.
- Numbers: p50 time-to-first-token down roughly 60%, p95 down over 90% from lazy sandbox provisioning.
- Lesson: make every part replaceable behind one `execute(name, input)` interface. [S13]

## Anthropic: C compiler with parallel agents (2026-02-05)
- Context: a 100,000-line Rust C compiler built by agents.
- What they did: 16 parallel agents, specialized roles, GCC as an oracle to split work, agent-friendly test output.
- Numbers: nearly 2,000 sessions, about $20,000 in API fees; builds bootable Linux 6.9 on three architectures.
- Lesson: the verifier must be nearly perfect or agents solve the wrong problem. [S15]

## OpenAI: practical guide to building agents (2025-04-07)
- Context: OpenAI's guidance for enterprise agent builds.
- What they did: recommend maximizing a single agent first, then manager (agents as tools) or decentralized (handoffs) patterns, with layered guardrails.
- Lesson: "Think of guardrails as a layered defense mechanism." Guardrail mechanics belong in security-engineering. [S17]

## LinkedIn: the last mile of a GenAI product (2024-04-25)
- Context: LinkedIn's generative AI product experience.
- Numbers: LinkedIn reports 80% of the basic experience in the first month, then four more months to pass 95%; YAML output over JSON for tokens, with a defensive parser cutting malformed outputs from about 10% to about 0.01%.
- Lesson: the demo is the cheap part; plan and measure the last mile. [S18]

## Uber: MCP Gateway (2026-10-01)
- Context: hundreds of internal MCP servers across teams.
- What they did: one gateway with tool-level authorization, PII redaction, and auto-generated tools disabled until owner approval.
- Numbers: Uber reports over 800 MCP servers and over 5,000 tools.
- Lesson: govern tools centrally once they multiply. [S19]

## Uber: software factory (2026-08-27)
- Context: agents at engineering-org scale.
- Numbers: Uber reports more than 70% of pull requests attributed to agents, 3,600+ agent skills, cost per session down 52% from the June peak, code mode cutting tokens 55-71% on small SQL tasks.
- Lesson: code mode and skills pay off at scale; track cost per session. [S20]

## Anthropic: advisor strategy (2026-04-09)
- Context: getting large-model judgment at small-model cost.
- Numbers: Haiku plus Opus advisor 41.2% vs 19.7% on BrowseComp; Sonnet plus advisor +2.7 points at 11.9% lower cost on SWE-bench Multilingual.
- Lesson: consult the expensive model only on hard calls, with a use cap. [S27]

## Desert Ant Labs: a CLI built for coding agents (2026-09-30)
- Desert Ant Labs reports it designed its model CLI "with coding agents in mind".
- What: every command takes `--json`, reads from stdin (a transcript can be piped into the next model so it skips re-transcribing) and exits with 0, 1 or 64; batches load the model once, return one JSON array, and when a file fails the CLI names it and carries on.
- Setup: `desertant setup` writes a skill file for each coding agent on the machine and a section in AGENTS.md for Codex.
- Lesson: a CLI is an agent tool; machine-readable output, chaining, exit codes and partial-failure reporting are its interface. Rule 3. [S32]
