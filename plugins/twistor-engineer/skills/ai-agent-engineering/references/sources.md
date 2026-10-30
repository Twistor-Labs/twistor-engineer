# Sources

Every figure in this skill traces to one line below. Anthropic posts use the `/research/` form (old `/engineering/` URLs redirect). claude.com resource summaries came from a page reader; re-check a quoted number against the page before citing it externally.

- [S1] Building effective agents. Anthropic, 2024-12-19. https://www.anthropic.com/research/building-effective-agents (VERIFIED)
- [S2] Raising the bar on SWE-bench Verified with Claude 3.5 Sonnet. Anthropic, 2025-01-06. https://www.anthropic.com/research/swe-bench-sonnet (VERIFIED)
- [S3] The "think" tool: Enabling Claude to stop and think in complex tool use situations. Anthropic, 2025-03-20. https://www.anthropic.com/research/claude-think-tool (CORRECTED)
  Note: page text says tau-bench airline pass^1 0.370 -> 0.570 (54% relative); the table on the same page says 0.332 -> 0.584 at k=1. Cite both. A 2025-12-15 update recommends extended thinking over a dedicated think tool in most cases.
- [S4] How we built our multi-agent research system. Anthropic, 2025-06-13. https://www.anthropic.com/research/multi-agent-research-system (CORRECTED)
  Note: "token usage explains 80% of variance" is BrowseComp only; three factors (tokens, tool calls, model choice) explain 95%. Not a general law.
- [S5] Writing effective tools for agents - with agents. Anthropic, 2025-09-11. https://www.anthropic.com/research/writing-tools-for-agents (CORRECTED)
  Note: the 25,000-token tool response cap is Claude Code's configurable default, not a hard limit. The 206 vs 72 token Slack figure is one illustrative example, not an average.
- [S6] Effective context engineering for AI agents. Anthropic, 2025-09-29. https://www.anthropic.com/research/effective-context-engineering-for-ai-agents (VERIFIED)
- [S7] Building agents with the Claude Agent SDK. Anthropic, 2025-09-29. https://claude.com/blog/building-agents-with-the-claude-agent-sdk (CORRECTED)
  Note: older links use anthropic.com/engineering; canonical is now claude.com/blog. "Start with agentic search" is vendor advice, not a benchmark; triggers to add semantic search are faster results or more variations.
- [S8] Equipping agents for the real world with Agent Skills. Anthropic, 2025-10-16. https://www.anthropic.com/research/equipping-agents-for-the-real-world-with-agent-skills (VERIFIED)
- [S9] Code execution with MCP: Building more efficient agents. Anthropic, 2025-11-04. https://www.anthropic.com/research/code-execution-with-mcp (VERIFIED)
- [S10] Introducing advanced tool use on the Claude Developer Platform. Anthropic, 2025-11-24. https://www.anthropic.com/research/advanced-tool-use (CORRECTED)
  Note: often dated "late 2025"; published 2025-11-24, beta header `advanced-tool-use-2025-11-20`.
- [S11] Effective harnesses for long-running agents. Anthropic, 2025-11-26. https://www.anthropic.com/research/effective-harnesses-for-long-running-agents (VERIFIED)
- [S12] Harness design for long-running application development. Anthropic, 2026-03-24. https://www.anthropic.com/research/harness-design-long-running-apps (VERIFIED)
- [S13] Scaling Managed Agents: Decoupling the brain from the hands. Anthropic, 2026-04-08. https://www.anthropic.com/research/managed-agents (VERIFIED)
- [S14] Introducing Contextual Retrieval. Anthropic, 2024-09-19. https://www.anthropic.com/research/contextual-retrieval (VERIFIED)
- [S15] Building a C compiler with a team of parallel Claudes. Anthropic, 2026-02-05. https://www.anthropic.com/research/building-c-compiler (VERIFIED)
- [S16] Desktop Extensions: One-click MCP server installation for Claude Desktop. Anthropic, 2025-06-26. https://www.anthropic.com/research/desktop-extensions (VERIFIED)
- [S17] A practical guide to building agents. OpenAI, 2025-04-07. https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf (VERIFIED)
- [S18] Musings on building a Generative AI product. LinkedIn Engineering, 2024-04-25. https://www.linkedin.com/blog/engineering/generative-ai/musings-on-building-a-generative-ai-product (VERIFIED)
- [S19] Designing MCP Gateway: Uber's MCP Management Platform. Uber Engineering, 2026-10-01. https://www.uber.com/blog/designing-mcp-gateway/ (VERIFIED)
- [S20] Running a Software Factory Efficiently at Uber Scale. Uber Engineering, 2026-08-27. https://www.uber.com/blog/efficient-software-factory/ (VERIFIED)
- [S21] Building agents: When to fold subagents into your main agent loop. Anthropic (claude.com), 2026-10-22. https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop (VERIFIED)
- [S22] Multi-agent coordination patterns: Five approaches and when to use them. Anthropic (claude.com), 2026-04-10. https://claude.com/resources/articles/multi-agent-coordination-patterns (VERIFIED)
- [S23] Building multi-agent systems: When and how to use them. Anthropic (claude.com), 2026-01-23. https://claude.com/resources/articles/building-multi-agent-systems-when-and-how-to-use-them (VERIFIED)
- [S24] Common workflow patterns for AI agents-and when to use them. Anthropic (claude.com), 2026-03-05. https://claude.com/resources/articles/common-workflow-patterns-for-ai-agents-and-when-to-use-them (VERIFIED)
- [S25] Managing context on the Claude Developer Platform. Anthropic (claude.com), 2025-09-29. https://claude.com/resources/articles/context-management (VERIFIED)
- [S26] Agent Harness Design: 3 Patterns for Harnessing Claude's Intelligence. Anthropic (claude.com), 2026-04-02. https://claude.com/resources/articles/harnessing-claudes-intelligence (VERIFIED)
- [S27] The advisor strategy: Give agents an intelligence boost. Anthropic (claude.com), 2026-04-09. https://claude.com/resources/articles/the-advisor-strategy (VERIFIED)
- [S28] Agent observability on the Claude Platform. Anthropic (claude.com), 2026-10-14. https://claude.com/resources/articles/agent-observability-on-the-claude-platform (VERIFIED)
- [S29] How we contain Claude across products. Anthropic, 2026-05-25. https://www.anthropic.com/research/how-we-contain-claude (VERIFIED)
- [S30] How we built Claude Code auto mode: a safer way to skip permissions. Anthropic, 2026-03-25. https://www.anthropic.com/research/claude-code-auto-mode (VERIFIED)
- [S31] Improving skill-creator: Test, measure, and refine Agent Skills. Anthropic (claude.com), 2026-03-03. https://claude.com/resources/articles/improving-skill-creator-test-measure-and-refine-agent-skills (VERIFIED)
- [S32] The Desert Ant CLI: on-device AI models for you and your agents. Desert Ant Labs, 2026-09-30. https://desertant.com/blog/desert-ant-cli/ (VERIFIED)
  Note: vendor-reported, single source.

## Left out on purpose

- Cloudflare Code Mode (Sep 2025): seen only in secondary summaries, not checked against a primary post. Uber's code-mode figure [S20] covers the same idea.
- OpenAI Agents SDK + Responses API tracing (Mar 2025): seen only in secondary summaries, not checked against a primary post.
