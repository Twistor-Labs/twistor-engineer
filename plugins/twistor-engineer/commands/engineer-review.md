---
description: Review the current diff against the engineering-practice skills (rollout safety, caching, evals, agents, security, performance, frontend) and list severity-tagged findings.
argument-hint: "[git range or path]  default: changes since the merge base with the default branch"
allowed-tools: Bash(git diff:*), Bash(git merge-base:*), Bash(git rev-parse:*), Bash(git symbolic-ref:*), Read, Grep, Glob, Agent
---

Review target: $ARGUMENTS

1. Resolve the diff.
   - If a range or path was given, use `git diff <range>` or `git diff -- <path>`.
   - Otherwise diff the working tree plus commits since the merge base with the default branch
     (`git symbolic-ref refs/remotes/origin/HEAD`, falling back to `main`).
   - If the diff is empty, say "Nothing to review." and stop.
2. Pick the skills whose domain the diff touches. Use file paths and content, for example:
   - prompts, tool schemas, agent loops, MCP config -> ai-agent-engineering, llm-evals
   - eval sets, graders, judge prompts -> llm-evals
   - schemas, queries, caches, queues, partition keys -> system-design-scaling
   - config, flags, rollout, retries, timeouts, alerts -> reliability-incident-response
   - hot loops, allocations, GC, concurrency, inference serving -> performance-engineering
   - pipelines, features, experiments, training jobs -> data-ml-platforms
   - CI, build, codemods, migrations, repo tooling -> dev-productivity-ai-coding
   - auth, secrets, permissions, sandboxing, untrusted input -> security-engineering
   - UI, bundles, mobile releases, OTA, client caching -> frontend-mobile-engineering
   Pick at most four. Say which you picked and why in one line.
3. Delegate to the `engineer-reviewer` agent with the diff range and the chosen skills.
4. Print the agent's output unchanged, one finding per line in its exact format, then its `totals:` line. No headings, no regrouping, no summary, no offer of next steps.
