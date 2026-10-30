---
name: dev-productivity-ai-coding
description: 'Use when agent coding is slow, noisy or unsafe, or when setting up AI-assisted development for a repo or team. Symptoms: agent says "done" but tests fail, CLAUDE.md gets ignored, context fills mid-task, agent PRs back up review, CI is slow under agent load, a codemod or test migration stalls at 80%. Phrases: "set up Claude Code", "hooks", "agent permissions", "LLM code review on every PR", "worktrees", "headless run", "TDD with the agent", "AI code review", "LLM migration", "merge queue", "build is slow". Gives sourced instead-of rules, a change-size ladder, an answer shape and an evidence gate, with Anthropic, Uber, Meta, Airbnb and Zepto case studies. Not for building agent products (use ai-agent-engineering), model evals (use llm-evals), or runtime tuning (use performance-engineering).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Agentic coding that ships: smallest change first, a verifier before scale, review and CI that keep up with agent volume.'
---

# Dev Productivity and AI Coding

Agents multiply whatever loop they sit in. A good verifier and a small change compound; a vague spec and a slow review queue compound too. This skill takes its stance from teams that measured it: Anthropic, Uber, Meta, Airbnb and Zepto.

## When to use / not

- Use: planning or running an agent coding session, setting up CLAUDE.md, hooks, worktrees or headless runs, designing an LLM-driven migration, setting review or CI policy for agent-authored PRs, picking what to fix in a slow dev loop.
- Use: you are about to say "done" on agent-written code.
- Use: review queue, merge queue or CI wait time grew after agents arrived.
- Use: the build or test loop is slow and someone proposes a rewrite.
- Not: designing tools, harnesses or multi-agent products for end users (see ai-agent-engineering).
- Not: building eval suites or judges for model quality (see llm-evals).
- Not: profiling production latency or CPU (see performance-engineering).
- Companions, link only: ponytail (smallest-change discipline), superpowers (systematic debugging, verification before completion, worktrees), agent-skills (spec-driven and test-driven development).

## Rules

Each rule names the common move, the better move, and the evidence. Cite the [S] id when you repeat a figure.

1. **Instead of** opening with a refactor, a bundler migration or a new abstraction, **do** profile, then climb the change-size ladder and stop at the first rung that holds. **Why:** Zepto found 13+ of 20+ build minutes inside Terser and swapped the minifier, taking builds to about 2 minutes with no bundler migration [S22]
2. **Instead of** letting the agent code straight from a chat prompt, **do** write a SPEC.md with done criteria, build it in a fresh session, and review with a fresh-context subagent that never saw the reasoning. **Why:** a reviewer without the author's reasoning catches what the author rationalized [S1]
3. **Instead of** a long CLAUDE.md of preferences, **do** keep it to what the code cannot tell the agent: the test and typecheck commands, pointers and critical gotchas, under 200 lines, with an owner. **Why:** the check command is the most useful line, and personal preferences belong in a gitignored local file [S1][S8][S24]
4. **Instead of** repeating a CLAUDE.md line the agent keeps missing, **do** move it into a command hook or deny rule, and run the test check from a Stop hook. **Why:** CLAUDE.md and skills are non-deterministic; hooks, deny rules and managed settings are deterministic, and exit code 2 blocks [S1][S8]
5. **Instead of** asking for the feature and its tests in one go, **do** write the failing test first and ask for just enough code to pass one test at a time, pasting the full error output on each failure. **Why:** Anthropic's org rollout guidance is tests first, one test at a time, full error output [S11]
6. **Instead of** scaling agents against a test suite you have not hardened, **do** make the verifier near-perfect first and make its output agent-readable: a few stdout lines, ERROR plus reason on one line, details in a log, a fast sampled mode. **Why:** Anthropic's 16-agent C compiler build found the verifier must be "nearly perfect, otherwise Claude will solve the wrong problem" [S4]
7. **Instead of** one marathon session on a long build, **do** run one feature per session with a progress file, a JSON feature list where the agent may only flip `passes`, a commit per feature, and a start ritual that reads git log and runs an end-to-end check. **Why:** JSON was less likely than Markdown to be overwritten, across a list of over 200 features [S2]
8. **Instead of** launching many parallel agents on day one, **do** start with two or three sessions in separate worktrees and add more only while review keeps up. **Why:** Anthropic's merges rose from about 2,500/day in May 2026 to over 30,000/day by September, and each bottleneck removed produced more code to review [S12][S5]
9. **Instead of** carrying one session across unrelated tasks, **do** start fresh per task, rewind failed attempts, and hand exploration of ten or more files, or three or more independent pieces, to subagents that return a summary. **Why:** noisy side work in the main context crowds out the task; same-file edits stay in one context [S9][S10]
10. **Instead of** hand-migrating thousands of files or chasing 100% automation, **do** run the LLM per file with a validator and retries fed by the error, tune the prompt on a sample of failures, and finish the tail by hand. **Why:** Airbnb reports 75% of ~3.5K test files in four hours, 97% after 4 days of tuning, last 3% by hand, 6 weeks against 1.5 years estimated [S13]
11. **Instead of** letting an AI reviewer post every thought, **do** measure the share of comments marked useful and addressed, cap nits per review and cap review rounds. **Why:** Uber reports 75% of uReview comments marked useful; Anthropic's 3-round cap moved one-round pass from 65% to 78% [S14][S6]
12. **Instead of** one approval rule for every PR, **do** classify risk and require 0, 1 or 2 human approvals by tier, never counting an agent approval as a human one. **Why:** Anthropic's Claude Code team runs exactly this tiering for agent-heavy review load [S6]
13. **Instead of** tracking average review time, **do** track P75 time-in-review and nudge diffs that sit. **Why:** Meta's NudgeBot cut average time-in-review 7% and diffs waiting over 3 days 12% [S18]
14. **Instead of** full CI and full clones on every agent commit, **do** gate main with a merge queue, run tests by impact, and use sparse checkouts that ask the server for diffs. **Why:** Uber reports SubmitQueue cut CI resource usage 53% and wait times 37%; Anthropic's sparse checkouts went from minutes to seconds and freed 25% CI CPU [S16][S5]
15. **Instead of** accepting LLM-written tests because coverage went up, **do** gate each one on builds, passes repeatedly, and adds coverage or kills a mutant, then let a human accept. **Why:** on Meta Reels/Stories TestGen-LLM tests built 75%, passed reliably 57% and raised coverage 25% of the time, and ACH targets tests at generated mutants [S20][S21]

## Answer shape

Every answer follows these six parts, in order. Short is fine; skipping a part is not.

1. **Rung**: which ladder rung holds, and why the lower rungs do not. Stop at the first rung that holds:
   - Rung 1, no code: config, flag, existing tool, or remove the need.
   - Rung 2, one line or one swap of a dependency or tool (Zepto swapped a minifier, not the bundler [S22]).
   - Rung 3, one file, existing patterns.
   - Rung 4, several files, no new abstraction.
   - Rung 5, new abstraction, codemod, or a fleet of parallel agents.
2. **Done means**: one line of acceptance criteria and the exact check command.
3. **Plan**: sessions, worktrees or subagents; what goes in CLAUDE.md versus a hook.
4. **Risks**: review load, CI load, what the verifier cannot see.
5. **Evidence**: commands run and their output, or "not yet run".
6. **Open questions**: at most three.

### Example: "our frontend build takes 20 minutes, should we move off webpack?"

1. **Rung**: 2. Profile first; if one plugin owns most of the time, swap that plugin. Zepto found 13+ minutes in Terser and swapped the minifier, no bundler migration [S22].
2. **Done means**: CI build under the target on the same commit, output size within an agreed margin of the old build.
3. **Plan**: one session, one worktree; the profile command and the size check go in CLAUDE.md.
4. **Risks**: minifier output differences, source maps, a slower cold cache.
5. **Evidence**: before and after build times and bundle sizes from the same CI runner.
6. **Open questions**: which size margin is acceptable; who owns the build config.

### Example: "migrate 3,000 test files to a new framework with an LLM"

1. **Rung**: 5, after ruling out a pure codemod for most files.
2. **Done means**: every file passed or marked manual, old dependency removed, merge queue green.
3. **Plan**: per-file state machine with validator and capped retries, tuning sweeps on failures, humans on the tail (see `references/llm-migrations.md`). Airbnb reports 75% in four hours and 97% after 4 days of tuning on ~3.5K files [S13].
4. **Risks**: tests that pass but assert less; reviewer load from large PRs [S7].
5. **Evidence**: counts per state, sampled passed files read by a human.
6. **Open questions**: retry cap; who owns the manual tail.

## Red flags

Stop when you catch yourself saying one of these.

- "The agent said it is done" -> run the check yourself and paste the output.
- "Tests pass" on a suite the agent also edited -> diff the tests before trusting them (rule 6, 15).
- "Let's rewrite it properly while we're here" -> you skipped the ladder (rule 1).
- "I already put that in CLAUDE.md" for the third time -> it belongs in a hook (rule 4).
- "Just one more task in this session" -> fresh session (rule 9).
- "We'll get the migration to 100% with a better prompt" -> the tail is for humans (rule 10).
- "Spin up ten agents" with a review queue already behind -> two or three, then measure (rule 8).
- "The AI reviewer left 30 comments, so it's thorough" -> count useful, not posted (rule 11).
- "The PR is big but the agent wrote it, so review is quick" -> split it; findings rise sharply past 1,000 lines [S7].
- "An agent approved it" on a high-risk change -> agent approvals never count as human (rule 12).
- "CI is slow, let's skip it for agent commits" -> merge queue and test impact, not bypass (rule 14).
- "Coverage went up, ship the generated tests" -> check they build, pass repeatedly and add value (rule 15).
- "We need a new bundler" before any profile -> profile, then the smallest rung (rule 1).
- A headless `claude -p` run that silently stops -> in auto mode, headless runs terminate on denial instead of continuing; pre-allow what the job needs [S25].

## Verification

Evidence before "done". Do not claim success without all that apply:

- The check command from CLAUDE.md ran in this session, with its output shown.
- New tests failed before the change and pass after (rule 5).
- Test files changed by the agent were read, not just run.
- For migrations: counts per state (pending, passed, retried, manual) and a sample of passed files read by a human.
- A fresh-context review ran and its findings are resolved or listed (rule 2).
- The ladder rung is named in the PR and lower rungs were ruled out (rule 1).
- For long builds: the progress file and feature list are updated and committed (rule 7).
- For review and CI changes: a before and after number on the metric you meant to move (P75 time-in-review, CI wait, useful-comment share).
- Repeated manual checks are captured as a verification skill so the next run does them by itself [S26].

## References

| File | Read when |
|---|---|
| references/case-studies.md | you need the full story and numbers behind a rule |
| references/llm-migrations.md | planning or running a codemod, framework swap or test migration with an LLM |
| references/checklist.md | setting up a repo for agents, or reviewing an agent PR before merge |
| references/sources.md | citing a figure or checking a source's status |
