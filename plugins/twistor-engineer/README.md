# The Twistor's Engineer

Engineering rules your coding agent can cite. Fourteen Agent Skills: ten practice skills (AI agents, LLM evals, system design and scaling, reliability and incident response, performance, data and ML platforms, AI-assisted coding, security, frontend and mobile, databases) with 15 rules each, 150 in total, written as "instead of X, do Y" with every figure linked to the public engineering post it came from. Four workflow skills use them: engineer-review for diffs, engineer-design-review for design docs, engineer-security-audit for a whole repository, and engineer-postmortem for incident notes.

## What it runs

The skills are Markdown instructions. The bundled `engineer-reviewer` agent reads code with Read, Grep, Glob and Bash to review a diff and returns one line per finding. It does not edit files.

## What it sends

Nothing. The plugin has no hooks, no MCP servers, no scripts and makes no network calls. Source links in the rules are references for you to open; the plugin does not fetch them.

## Links

- Site: https://engineer.twistorlabs.com
- Source: https://github.com/Twistor-Labs/twistor-engineer
- Publisher: Twistor Labs Private Limited, hello@twistorlabs.com
- License: MIT. Twistor Labs is not affiliated with the companies cited.
