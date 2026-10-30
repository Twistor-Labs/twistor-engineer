# The Twistor's Engineer

Nine engineering-practice skills for Claude Code, plus a diff reviewer and a postmortem writer.
Each skill is a short list of rules in the form "instead of X, do Y", and every figure in a rule
links to the public engineering post it came from: Anthropic, Cloudflare, Netflix, Meta, Uber,
Discord, Airbnb, LinkedIn, Zepto and others.

The skills load on their own when your question matches. If you run many plugins at once and a
skill doesn't load, call it by name, for example `/twistor-engineer:system-design-scaling`.

Site: https://engineer.twistorlabs.com

## Install

```
/plugin marketplace add Twistor-Labs/twistor-engineer
/plugin install twistor-engineer@twistor-engineer
```

Optional: live search over the Cloudflare blog through its public MCP server. We don't ship this
inside the plugin, because a plugin's MCP servers start as soon as it is enabled.

```
claude mcp add --transport http cloudflare-blog https://blog.mcp.cloudflare.com/mcp
```

## Skills

| Skill | Use it for |
|---|---|
| `ai-agent-engineering` | Workflow vs agent, tool design, context budgets, multi-agent, long-running harnesses |
| `llm-evals` | Eval sets from real failures, code and model graders, pass@k vs pass^k, production quality checks |
| `system-design-scaling` | Capacity math, where it breaks first, data store choice, caches, cells |
| `reliability-incident-response` | Staged rollouts, config as code, kill switches, mitigation ladder, postmortems |
| `performance-engineering` | Tail latency, profiling, GC and runtime traps, LLM inference benchmarking |
| `data-ml-platforms` | Data contracts, online/offline parity, orchestration and backfill, experiment metrics |
| `dev-productivity-ai-coding` | Agentic coding workflow, review at agent volume, CI cost, LLM-driven migrations |
| `security-engineering` | Sandboxing agents, credentials outside the box, scoped tools, approval tiers |
| `frontend-mobile-engineering` | Performance budgets, safe mobile releases and OTA, server-driven UI, startup-scale lessons |

Each skill has the same shape: when to use it, 15 rules, the shape of a good answer, red flags,
how to verify, and a `references/` folder with case studies, a checklist and `sources.md`.

## Commands and agent

- `/engineer-review [range]` picks the skills your diff touches and hands the diff to the
  `engineer-reviewer` agent. Findings come back one per line:
  `path:line: bug|risk|nit|q: problem. fix. (skill #rule)`, then a totals line, or `No issues.`
- `/engineer-postmortem [notes]` turns incident notes into a blameless postmortem with an ISO UTC
  timeline and action items that each carry an owner and a revisit trigger.

## Sources

Every rule cites its sources by id (`[S3]`). Each id points to a line in that skill's
`references/sources.md` with the URL, the date and a status: VERIFIED, CORRECTED (the number most
people repeat is wrong, so we use the fixed one) or PRESS-SOURCED. Numbers a company reports about
itself are written as that company's claim.

Company names identify the public posts we cite. Twistor Labs is not affiliated with or endorsed
by any of them.

## License

MIT. Copyright 2026 Twistor Labs Private Limited.
