---
name: engineer-design-review
description: 'Use when asked to "review this design doc", "review my RFC", "sanity check this plan before we build", "design review", "poke holes in this proposal" or "is this plan a go", including when the doc is not attached yet (load this skill first, then ask for the doc). Reviews a design doc, RFC or build plan against the twistor-engineer practice checklists (migrations and schema changes, rollout and rollback, failure handling, agent tool access, evals, security, scaling) before any code exists, and returns one severity-tagged line per finding with the doc section it concerns, plus the open questions that block a go/no-go. Read-only; never edits the doc or any file. Not for: reviewing a diff, branch or PR (use engineer-review), writing or drafting a design doc, or writing a postmortem (use engineer-postmortem).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.2.0
  summary: 'Design doc review against up to four skill checklists, one finding per line, plus go/no-go questions.'
---

# Engineer design review

Review a design before it is built, against the checklists and Rules of the sibling
twistor-engineer skills. Report; never edit.

## 1. Resolve the input

- A path was given: read that file. A pasted doc: use the text as given.
- Nothing given: ask for a path or pasted text, then stop.
- Not a design (meeting notes, a ticket, a status update, a diff, a question): print exactly
  `No design to review.` and stop. A design proposes a change and says how it will be built.
- Long doc (more than about 600 lines or many appendices): read the overview, the proposal, data,
  rollout and risk sections first. Name every section you skipped in the first output line.

## 2. Pick at most four skills

Match what the doc describes:

| Doc describes | Skill |
|---|---|
| schema change, migration, backfill, dual writes, data cutover, sharding a live database, connection pooling | database-engineering |
| prompts, tool lists, agent loops, MCP servers | ai-agent-engineering, llm-evals |
| LLM output quality, eval sets, graders, judges | llm-evals |
| capacity, data store choice, caches, queues, partition keys | system-design-scaling |
| rollout, flags, config, external calls, retries, timeouts, deletes, alerts | reliability-incident-response |
| hot paths, latency budgets, concurrency, inference serving | performance-engineering |
| pipelines, features, experiments, training jobs | data-ml-platforms |
| CI, build, codemods, bulk code migration, repo tooling | dev-productivity-ai-coding |
| auth, secrets, permissions, agent credentials, untrusted input | security-engineering |
| UI, bundles, mobile releases, OTA, client caching | frontend-mobile-engineering |

More than four match: keep the four whose rules the riskiest parts of the design touch.
Say which you picked and why in one line.

## 3. Walk the checklists

For each picked skill, read `../<skill>/references/checklist.md` and the Rules section of
`../<skill>/SKILL.md` (sibling folders of this skill). If a checklist is missing, use the Rules
section alone; if the skill folder is missing, drop it and say so in the first line. If your tool can delegate to a subagent (in Claude Code, the `engineer-reviewer`
agent), hand it the doc text, the skill list and this output format, and print its output
unchanged. Otherwise do it here:

1. Read the doc once. Note what it changes: stored data, a prod code path, external calls, model
   or agent behavior, credentials, traffic shape.
2. Walk each checklist item that applies to what the doc proposes. Skip items it cannot affect.
3. Every finding cites the doc section it concerns (the heading text) and one rule as
   `<skill> #<n>`. A real problem that no picked rule covers becomes an `open:` line instead.
   Drop anything you cannot tie to a section.
4. Quote or paraphrase the doc's own words in the problem, so the author can find the line.

## 4. Check what the doc leaves out

A design that needs one of these and lacks it gets a finding or an open question, never silence:

| The design | Must say |
|---|---|
| changes stored data or schema | migration steps, lock and timeout settings, batched backfill, verify before delete |
| ships a new prod code path, config or cutover | rollout stages and how to roll back (flag, revert, restore), with a time |
| calls an external or another team's service | timeout, retry with backoff, behavior when it is down or slow |
| uses an LLM or agent | eval plan with real cases before launch, and what tools and credentials it gets |

Missing and the gap can cause an outage or data loss: `bug` or `risk` finding under `§(missing)` or
the section it belongs in. Missing but you cannot tell whether it is needed: open question.

## Severity

Same as engineer-review.

- `bug`: wrong behavior, data loss or an outage path as written (table rewrite under lock, delete
  before verify, missing timeout on a call in the request path, secret in config).
- `risk`: works on paper, fails under load, at rollout or on bad input (no flag, no rollback, no
  backoff, no eval, unscoped agent tools).
- `nit`: practice drift with low impact.
- `q`: you need an answer from the author to judge.

## Output

Exactly this, nothing else: no headings, no summary of the doc, no praise, no offer of next steps.

```
picked: <skills> (<why, one clause>). read: <sections>. skipped: <sections or none>.
§<section heading>: <bug|risk|nit|q>: <problem>. <fix>. (<skill> #<rule>)
open: §<section>: <what is missing>. <why it blocks a go/no-go>.
totals: N bug, N risk, N nit, N q, N open
```

- Findings first, most severe first, one per line.
- A finding about something the doc leaves out uses the section it belongs in, or `§(missing)`.
- At most 5 `open:` lines, after the findings.
- Nothing survives step 3: print exactly `No issues.` then any `open:` lines and the totals line.

## Never edit

Do not change the doc or any file, even when asked to "fix it" or "update the doc". Say you only
review, and offer the findings as a plain list the author can apply.

## Samples

`references/samples/` holds three design docs with planted flaws and `ANSWER-KEY.md`. Read them
only to test this skill, never as input to a real review.
