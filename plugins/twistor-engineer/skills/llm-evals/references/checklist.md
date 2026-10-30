# Checklist

Use when reviewing an eval PR or shipping a prompt, model, tool or harness change. Each item names the rule it enforces.

## Task set
- [ ] Tasks come from real failures: bugs, tickets, incident or revert PRs (rule 1).
- [ ] At least 20 tasks to start; growth path written down (rule 1).
- [ ] Every task has a reference solution that passes its grader (rule 2).
- [ ] Any task at 0% has been audited for task, grader and environment bugs (rule 2).
- [ ] Negative tasks exist: should abstain, refuse, escalate, or not call a tool (rule 5).

## Graders
- [ ] Code graders used wherever the outcome is checkable (rule 3).
- [ ] Each model grader has a rubric with anchored criteria and JSON output (rule 3).
- [ ] Judge-human agreement measured on a labelled sample, sample size recorded (rule 3).
- [ ] Format validity reported separately from content correctness (rule 4).
- [ ] Abstention scored above a confident wrong answer (rule 5).
- [ ] Judge runs in a fresh context without the generator's reasoning, with an iteration cap (rule 8).

## Trials and metrics
- [ ] k trials per task; k stated in the report (rule 6).
- [ ] pass^k used for user-facing paths, pass@k only for capability exploration (rule 6).
- [ ] Tokens, tool calls, turns and wall time logged per trial; cost per pass reported (rule 9).
- [ ] Metrics reported per segment; retrieval measured apart from generation (rule 12).
- [ ] Segments where the system loses are published, and automated actions use a stricter confidence cutoff than suggestions (rule 12).

## Gate
- [ ] Capability and regression suites are separate (rule 7).
- [ ] Regression suite runs on every prompt, model, tool or harness change and blocks merge (rule 7).
- [ ] Resource limits, timeouts, model id, prompt hash and harness version stored with results (rule 10).
- [ ] Delta exceeds the measured noise floor before it is called a win (rule 10).

## Integrity
- [ ] Failing and top-scoring transcripts read for contamination or eval awareness (rule 11).
- [ ] Benchmark material blocked at content level where agents can search (rule 11).

## Generated code and migrations
- [ ] Generated tests or code pass build, repeat-pass and coverage or mutant filters before review (rule 13).
- [ ] Human acceptance rate tracked, not volume (rule 13).
- [ ] Bulk migrations run as a sample, tune, sweep loop with tests as grader; tail owned by a human (rule 14).

## Production
- [ ] Continuous quality eval runs on production traffic (rule 15).
- [ ] A usefulness signal from real users is tracked with an owner (rule 15).
- [ ] Every production miss becomes a new task in the suite (rules 1, 15).
