# AGENTS.md

This repo ships The Twistor's Engineer: Agent Skills in the open SKILL.md format, under
`plugins/twistor-engineer/skills/<name>/SKILL.md`. Any agent that reads SKILL.md can use them.

- Nine practice skills: ai-agent-engineering, llm-evals, system-design-scaling,
  reliability-incident-response, performance-engineering, data-ml-platforms,
  dev-productivity-ai-coding, security-engineering, frontend-mobile-engineering.
- Two workflow skills: engineer-review (diff review against the checklists) and
  engineer-postmortem (incident notes to a blameless postmortem).

Each skill's `references/` holds case studies, a checklist and `sources.md`. Every rule cites a
source id that resolves to a public post in `sources.md`. When you quote a figure, keep its
source and say it is that company's own report.
