# LLM-driven migrations

For codemods, framework swaps and test migrations across hundreds or thousands of files. The shape below is our recommended pipeline. Figures are Airbnb's and LinkedIn's own reports [S13][S23]; the per-file state machine is a design pattern, not a claim about how Airbnb built theirs.

## Before you start

- Climb the ladder first: can a deterministic codemod do it? Use the LLM for what the codemod cannot.
- Define the validator before the prompt. For test migrations: the file compiles, the migrated test passes, and the old framework import is gone. The verifier must be near-perfect or the agent solves the wrong problem [S4].
- Pick a small sample of real files that spans the hard cases, and measure on it first.

## Per-file state machine

Each file is one row in a state file (JSON, so the agent edits fields rather than rewriting it [S2]):

```
pending -> migrating -> validating -> passed
                           |
                           v
                        failed(n) -> migrating   (retry with the error output)
                           |
                           v (n == max retries)
                        manual
```

- One file per unit of work. Same-file edits stay in one context [S9].
- On failure, retry with the full error output in the prompt [S11]. Cap retries; a capped loop hands off to a human.
- Record the attempt count and last error per file, so tuning works from data.
- Commit passed files in batches small enough to review.

## Tune on the failures

- Airbnb reports 75% of ~3.5K files migrated in four hours, then 4 days of "sample, tune, sweep" moved it to 97% [S13].
- Loop: sample failed files, read the errors, change the prompt or add context, sweep all failed files again, re-measure.
- Stop tuning when a sweep stops moving the pass count. Airbnb finished the last 3% by hand [S13]; LinkedIn spent four months chasing 95% after 80% in the first month on a different product [S23]. The tail is a human job.

## Review

- Humans read a sample of passed files, not only the failed ones. A passing test can still test less than before.
- Diff test assertions count before and after per file; a drop is a red flag.
- Keep PRs small: findings rise sharply on PRs over 1,000 lines [S7].

## Done means

- Counts per state reported: passed, retried, manual.
- Zero files left in pending or migrating.
- Old dependency removed and CI green on the merge queue [S16].
