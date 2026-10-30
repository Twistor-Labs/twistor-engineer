# Checklist

Each item names the rule it enforces.

## Repo setup for agents

- [ ] CLAUDE.md lists the test and typecheck commands, pointers and gotchas, under 200 lines, with an owner (rule 3).
- [ ] Personal preferences live in a gitignored local file, not the shared CLAUDE.md (rule 3).
- [ ] Any instruction that was missed twice is a hook or deny rule now (rule 4).
- [ ] A Stop hook or CI step runs the check command (rule 4).
- [ ] Test output is short on stdout, one-line ERROR with reason, details to a log, with a fast sampled mode (rule 6).
- [ ] Headless runs pre-allow the commands they need, since auto-mode denials end a headless run [S25].
- [ ] Repeated manual checks are written as a verification skill [S26].

## Before an agent session

- [ ] Ladder rung named; lower rungs ruled out with a reason (rule 1).
- [ ] SPEC.md or one-line done criteria written (rule 2).
- [ ] Failing test exists for the behavior (rule 5).
- [ ] Fresh session; unrelated work goes to another session or a subagent (rule 9).
- [ ] Long build: progress file, JSON feature list, one feature per session (rule 7).
- [ ] Parallel work: two or three worktrees, only if review has capacity (rule 8).

## Before merging an agent PR

- [ ] Check command ran in this session with output shown (Verification).
- [ ] Agent-edited tests were read, not only run (rules 6, 15).
- [ ] Fresh-context review ran; findings resolved or listed (rule 2).
- [ ] Risk tier set; human approvals match the tier; agent approval not counted (rule 12).
- [ ] Review rounds within the cap; nits capped (rule 11).
- [ ] PR small enough to review; large PRs split (rule 11).
- [ ] Merged through the queue, not around it (rule 14).

## Team and platform

- [ ] AI reviewer tracked on useful and addressed share, not comment count (rule 11).
- [ ] P75 time-in-review tracked; stale diffs nudged (rule 13).
- [ ] Merge queue keeps main green; tests run by impact; checkouts sparse where possible (rule 14).
- [ ] Generated tests gated on build, repeated pass and added coverage or killed mutant (rule 15).
- [ ] Cost per agent session tracked [S15].
- [ ] Migrations report counts per state and leave the tail to humans (rule 10).
