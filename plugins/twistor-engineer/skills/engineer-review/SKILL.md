---
name: engineer-review
description: 'Review a diff, branch or PR against the twistor-engineer practice checklists (rollout safety, config, caching, evals, agent tools, security, performance, data pipelines, frontend and mobile) and return one severity-tagged line per finding. Use when asked to "review this diff", "review my branch", "check this PR against best practice", "engineer review", or before merging a risky change. Read-only; never edits code.'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.2.0
  summary: 'Diff review against the ten skill checklists, one finding per line.'
---

# Engineer review

Review a change against the checklists of the sibling twistor-engineer skills. Report; never edit.

## 1. Resolve the diff

- A range or path was given: `git diff <range>` or `git diff -- <path>`.
- Nothing given: working tree plus commits since the merge base with the default branch
  (`git symbolic-ref refs/remotes/origin/HEAD`, falling back to `main`).
- Empty diff: print `Nothing to review.` and stop.

## 2. Pick at most four skills

Match file paths and content:

| Diff touches | Skill |
|---|---|
| prompts, tool schemas, agent loops, MCP config | ai-agent-engineering, llm-evals |
| eval sets, graders, judge prompts | llm-evals |
| data models, queries, caches, queues, partition keys | system-design-scaling |
| config, flags, rollout, retries, timeouts, alerts | reliability-incident-response |
| hot loops, allocations, GC, concurrency, inference serving | performance-engineering |
| pipelines, features, experiments, training jobs | data-ml-platforms |
| CI, build, codemods, repo tooling | dev-productivity-ai-coding |
| SQL migrations, schema files, backfills, ORM migration dirs, pool config | database-engineering |
| auth, secrets, permissions, sandboxing, untrusted input | security-engineering |
| UI, bundles, mobile releases, OTA, client caching | frontend-mobile-engineering |

Say which you picked and why in one line.

Whole-repo security audit rather than one diff: use `engineer-security-audit`. A design doc
before code: use `engineer-design-review`.

## 3. Walk the checklists

For each picked skill, read `../<skill>/references/checklist.md` and the Rules section of
`../<skill>/SKILL.md` (sibling folders of this skill). If your tool can delegate to a subagent
(in Claude Code, the `engineer-reviewer` agent), hand it the diff and the skill list and print its
output unchanged. Otherwise do it here:

1. Read the whole diff once. Note what changed.
2. Walk each checklist item that applies. Skip items the diff cannot affect.
3. A finding points at a line in the diff, or at a line outside it that the diff breaks.
4. Open the file at that line to confirm. Drop anything you cannot point to.

## Severity

- `bug`: wrong behavior or an outage path (unvalidated generated config, missing timeout, secret in code).
- `risk`: works today, fails under load, at rollout or on bad input (no flag, no backoff, no eval).
- `nit`: practice drift with low impact.
- `q`: you need an answer from the author to judge.

## Output

One line per finding, most severe first, then the totals line. Nothing else: no headings, no
summary of the diff, no praise, no offer of next steps.

```
path:line: <bug|risk|nit|q>: <problem>. <fix>. (<skill> #<rule>)
totals: N bug, N risk, N nit, N q
```

If nothing survives step 3.4, print exactly `No issues.`
