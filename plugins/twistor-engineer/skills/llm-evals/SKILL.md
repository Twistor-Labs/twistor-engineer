---
name: llm-evals
description: 'Use when building or trusting evals for LLM features or agents, or when a prompt, model or harness change needs proof. Symptoms - "the new prompt feels better", scores jump between runs, benchmarks green but users complain, LLM judge disagrees with humans, a task scores 0%, no eval gate in CI. Phrases - eval suite, golden set, grader, LLM-as-judge, rubric, pass@k, pass^k, regression eval, contamination, hallucination rate, retrieval metric, test-generation acceptance, production quality monitoring. Gives sourced instead-of rules from first eval to production monitoring, grader and pass^k math, a CI gate recipe, case studies and a checklist. Not for agent, tool or context design (use ai-agent-engineering) or evaluator-loop patterns inside an agent (see the agentic-eval companion).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Evals that catch real regressions, from 20 real-failure tasks to graded CI gates and production quality checks.'
---

# LLM Evals

An eval is only as good as its tasks, its graders and the number of trials behind a number. Start small from real failures, grade with code before models, keep the judge out of the generator's head, and keep evaluating after ship, because offline benchmarks have missed production degradation before. Rules come from Anthropic's eval and postmortem writing plus LinkedIn, Meta, Airbnb, OpenAI, Uber and Zepto engineering posts.

Vocabulary used throughout (from [S1]): **task** (one problem with success criteria), **trial** (one attempt at a task), **grader** (logic that scores a trial), **transcript** (the full record of a trial), **outcome** (the end state the grader checks), **harness** (the evaluation harness that runs tasks, distinct from the agent harness being tested).

## When to use / not

- Use: standing up a first eval suite for an LLM feature or agent.
- Use: deciding whether a prompt, model, tool or harness change is safe to ship.
- Use: choosing graders, writing a judge rubric, picking pass@k vs pass^k, wiring an eval gate into CI.
- Use: a score moved and nobody can say whether it is real, noise or contamination.
- Use: measuring LLM-generated code, tests or migrations, or monitoring quality in production.
- Not: designing the agent loop, tool schemas, context strategy or subagent topology (use ai-agent-engineering).
- Not: in-agent generator-verifier loop patterns in depth (see the agentic-eval companion skill, link only).
- Not: load, latency or cost benchmarking of inference servers (use performance-engineering).

Fast path for a team with no evals today:

1. Pull 20 failures from the bug tracker and recent reverts; write each as a task with a reference outcome (rules 1, 2).
2. Write code graders for every outcome you can check; mark the rest for a model grader (rule 3).
3. Run k=3 trials on the current build to get a baseline and a noise floor (rules 6, 10).
4. Make that set the regression gate on prompt and model changes (rule 7).
5. Add the model grader, segments and production signal only after steps 1-4 hold (rules 8, 12, 15).

## Rules

Ordered by increasing complexity: first suite, then graders, then trials and gating, then noise, then production.

1. **Instead of** waiting for a large benchmark before evaluating anything, **do** start with 20-50 simple tasks taken from real failures: bug reports, support tickets, incident and revert PRs. **Why:** Anthropic calls 20-50 real-failure tasks "a great start" and its code-review team turns past incident and revert PRs into a daily eval [S1][S11][S12]
2. **Instead of** reading a 0% task as "the model cannot do this", **do** write a reference solution that passes the grader, then audit the task, grader and environment before blaming the model. **Why:** 0% pass@100 usually means a broken task or grader, and SWE-bench work found setup issues and patches applied twice [S1][S13]
3. **Instead of** reaching for an LLM judge first, **do** grade with code wherever possible (tests, exact match, schema parse, end-state checks), use a model grader only for what code cannot check, and use humans to calibrate it. **Why:** Anthropic recommends deterministic graders where possible and calibrating LLM graders against human judgment, and calls LLM-as-judge "generally not a very robust method" [S1][S9]
4. **Instead of** folding malformed output into general failures, **do** track format validity as its own metric and put a defensive parser in front of the grader. **Why:** LinkedIn reports cutting malformed YAML outputs from ~10% to ~0.01% with a defensive parser [S14]
5. **Instead of** scoring only accuracy on should-answer cases, **do** balance positive and negative tasks (should abstain, should refuse, should not call a tool) and score an honest abstention above a confident wrong answer. **Why:** Anthropic asks for balanced positive and negative cases, and OpenAI argues accuracy-only scoring rewards guessing over admitting uncertainty [S1][S18]
6. **Instead of** one trial per task, **do** run k trials and report pass@k for capability and pass^k for anything a customer relies on every time. **Why:** 75% per trial is only about 42% pass^3, and one Anthropic regression showed up in one run out of two [S1][S10]
7. **Instead of** one blended suite that gates nothing, **do** keep capability evals (expected to start low) apart from regression evals (expected near 100%) and block merges of prompt, model or harness changes on regression drops. **Why:** Anthropic says regression evals "should have a nearly 100% pass rate" and after its April 2026 postmortem runs a broad per-model suite on every system prompt change, where one verbosity prompt cost 3% in ablation [S1][S7]
8. **Instead of** letting the generator grade itself or sharing its context with the judge, **do** run the grader in a fresh context that sees only the task, the rubric and the output, tuned to be skeptical, with a fixed iteration cap. **Why:** Anthropic found tuning a standalone evaluator to be skeptical "far more tractable" than making a generator critical of its own work [S8]
9. **Instead of** comparing agent variants on score alone, **do** log tokens, tool calls, turns and wall time per trial and compare cost per passed task. **Why:** on BrowseComp, token usage alone explained 80% of performance variance and multi-agent runs used about 15x chat tokens [S2]
10. **Instead of** trusting small leaderboard or A/B deltas, **do** pin and record resource limits, timeouts and infra config per run, rerun at different times, and treat gaps under 3 points as noise until configs match. **Why:** on Terminal-Bench 2.0, resource configuration alone moved scores by 6 points [S5]
11. **Instead of** assuming the eval set is clean, **do** read transcripts for contamination and eval awareness, block benchmark material at the content level, and rescore flagged tasks. **Why:** Anthropic found 11 BrowseComp answers came from benchmark materials, and URL-level blocklists were not enough [S6]
12. **Instead of** one global score over head queries, **do** report metrics per segment (new items, tail queries, task type) and measure retrieval separately from generation. **Why:** Zepto reports head-query benchmarks ignore ~80% of query space, and Anthropic tracks retrieval as top-20 failure rate, 5.7% down to 1.9% [S20][S4]
13. **Instead of** counting how much code or how many tests an LLM generated, **do** pass every candidate through filters (builds, passes repeatedly, adds coverage or kills a mutant) and measure the rate humans accept. **Why:** on Meta's Reels and Stories, TestGen-LLM candidates went 75% built, 57% passed reliably, 25% raised coverage, and Meta reports 73% acceptance for both TestGen-LLM and ACH [S15][S16]
14. **Instead of** one big prompt run for a bulk LLM migration, **do** loop run, collect failures, tune the prompt, sweep again, with the existing test suite as grader, and finish the long tail by hand. **Why:** Airbnb reports 75% of files migrated in four hours, 97% after four days of "sample, tune, sweep", and the last 3% by hand [S17]
15. **Instead of** treating offline evals as proof after ship, **do** run continuous quality evals on production traffic, track a usefulness signal (marked useful, comment addressed), and turn every production miss into a new task. **Why:** an Anthropic routing bug reached 16% of Sonnet 4 requests in its worst hour, and the Sep 2025 postmortem fixes added continuous quality evals on production, and Uber reports 75% of uReview comments marked useful [S3][S19]

## Answer shape

Every answer that designs, reviews or debugs an eval follows these sections in order:

1. **Claim under test** - the one decision this eval informs (ship the prompt, swap the model, gate the merge).
2. **Task set** - where tasks come from, count, positive vs negative mix, reference solutions (rules 1, 2, 5).
3. **Graders** - code, model, human split; rubric for any model grader; judge isolation (rules 3, 4, 8).
4. **Metric and trials** - pass@k or pass^k, k, segments, cost per pass (rules 6, 9, 12).
5. **Gate** - which suite blocks what, threshold, noise floor (rules 7, 10).
6. **Threats** - contamination, infra noise, grader drift (rules 10, 11).
7. **Production loop** - online signal and how misses flow back to tasks (rule 15).

Worked shape (illustrative, not a sourced case):

```text
Claim under test: new support-agent system prompt is safe to ship.
Task set: 40 tasks from last quarter's escalated tickets; 28 should-resolve,
  12 should-escalate or abstain; each with a reference transcript that passes.
Graders: code checks end state (ticket status, refund amount, no forbidden tool
  call); model grader scores tone against a 4-point rubric in a fresh context;
  50-trial human sample sets judge agreement before the judge is trusted.
Metric and trials: k=3 per task; pass^3 on should-resolve, abstain rate on
  should-escalate; segments by ticket type; tokens and turns per pass.
Gate: regression suite must not drop on pass^3; capability suite reported only.
Threats: same container limits as baseline; tasks never seen in prompt examples.
Production loop: weekly sample of live transcripts graded by the same rubric;
  every escalation the agent got wrong becomes a new task.
```

When the user only asks "is this eval good?", answer with the gaps per section above, each tied to a rule number, then the smallest fix that closes the biggest gap.

## Red flags

- "It looks better on the five prompts I tried" -> run the regression suite with k trials before saying better.
- "The judge said 9/10" -> show the judge's agreement with human labels on a sample first.
- "We'll add evals after launch" -> 20 tasks from today's bug tracker is enough to start.
- "The task is just too hard, 0% is expected" -> write the reference solution; prove the grader can pass.
- "Up 2 points, ship it" -> check the noise floor, infra config and trial count.
- "Same model wrote and graded it" -> isolate the grader in a fresh context.
- "Benchmarks are green, so users are fine" -> check the production quality signal.
- "Malformed outputs are rare, ignore them" -> measure the rate; it is a metric.
- "The model should always answer" -> add abstention tasks and stop rewarding guesses.
- "pass@5 is 98%, it is reliable" -> pass@k measures can-it, not every-time; report pass^k for user-facing paths.
- "Generated 500 tests, coverage will follow" -> count only tests that build, pass repeatedly and add coverage.
- "One global score is simpler" -> the tail segment is usually where the regression hides; split it.
- "The leaderboard says model B wins" -> check whether both ran with the same resources and harness.

## Verification

Before claiming an eval is done or a change is safe:

- Every task has a passing reference solution, and the grader fails a known-bad output.
- Model graders show agreement with a human-labelled sample, recorded with the sample size.
- The report states k, the metric (pass@k or pass^k), per-segment numbers and cost per passed task.
- Run config (model id, prompt hash, resource limits, timeouts, harness version) is stored with the results.
- The regression suite ran on the exact change, and the gate result is linked from the PR.
- At least a few failing transcripts were read end to end, not only the score.
- A production signal exists for the feature and has an owner.

## References

| File | Read when |
|---|---|
| references/graders-and-metrics.md | picking grader types, writing a judge rubric, computing pass@k or pass^k, wiring the CI gate |
| references/case-studies.md | you need the full story behind a rule or a figure to persuade a reviewer |
| references/checklist.md | reviewing an eval PR or shipping a prompt, model or harness change |
| references/sources.md | citing a figure or checking a source's status |

Related skills: ai-agent-engineering (agent design, tools, context); agentic-eval companion (evaluator-loop patterns, link only).
