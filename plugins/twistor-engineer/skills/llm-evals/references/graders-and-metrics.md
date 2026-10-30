# Graders and metrics

Patterns are numbered by increasing complexity. Use the first one that answers the question; stack later ones only when needed.

## 1. Grader types

Anthropic's agent-eval vocabulary names three grader types: code-based, model-based and human [S1].

| Grader | Checks | Strength | Weakness | Use for |
|---|---|---|---|---|
| Code | tests pass, exact or normalized match, schema parse, end state in a DB or file, forbidden tool not called | deterministic, cheap, reproducible | only checks what you can write down | the default; outcomes and hard constraints |
| Model | tone, helpfulness, faithfulness to sources, rubric criteria | covers open-ended output | drifts, is gameable, needs calibration; one Anthropic post calls LLM-as-judge "generally not a very robust method" [S9] | only what code cannot check |
| Human | anything, plus calibration of the other two | ground truth | slow, costly, inconsistent across raters | labelling a calibration sample, reading failing transcripts |

Order of preference: code first, model second, human to calibrate the model and spot-check. Anthropic recommends deterministic graders where possible and calibrating LLM graders against human judgment [S1].

Grade the **outcome** (end state) where you can, not the exact path in the transcript. Agents find valid paths you did not write down.

## 2. Format validity as a separate metric

Parse failures hide inside "wrong answer". Track them on their own line. LinkedIn reports ~10% malformed YAML cut to ~0.01% with a defensive parser [S14]. Pattern:

1. Parse strictly; record `format_ok` per trial.
2. Run a defensive repair parser; record `format_repaired`.
3. Grade content only on parsed output; report all three rates.

## 3. Model-grader rubric template

Use when a code grader cannot decide. Keep each criterion binary or on a short scale with anchors.

```text
ROLE: You are grading one output. You did not write it. Assume it has defects
until the evidence shows otherwise.

TASK: <the original task, verbatim>
REFERENCE (optional): <a passing reference output or key facts>
OUTPUT TO GRADE: <the output, verbatim>

CRITERIA (score each independently, quote the evidence):
C1 <criterion> - PASS if <observable condition>; FAIL if <observable condition>.
C2 <criterion> - 0 = <anchor>, 1 = <anchor>, 2 = <anchor>.
C3 Abstention - PASS if the output declines or flags uncertainty when the
   reference says the answer is unknown; FAIL if it guesses.

RETURN JSON ONLY:
{"C1": {"score": ..., "evidence": "..."}, "C2": {...}, "C3": {...},
 "overall": "PASS|FAIL", "reason": "<one sentence>"}
```

Rules for the judge:

- **Fresh context.** The judge sees task, rubric, reference and output. It never sees the generator's reasoning, plan or self-assessment. Tuning a standalone evaluator to be skeptical was "far more tractable" than making the generator self-critical [S8].
- **Adversarial stance.** Ask it to find the defect, not to confirm success.
- **Calibrate before trusting.** Label a sample by hand, measure judge-human agreement, and record the sample size next to the number [S1].
- **Abstention beats guessing.** Score an honest "I don't know" above a confident wrong answer; accuracy-only scoring rewards guessing [S18].
- **Cap the loop.** If the judge feeds a generator retry loop, set a hard iteration cap and hand off to a human on cap.
- **Re-check per model.** A rubric tuned on one model's outputs can mis-grade the next model's style.

## 4. pass@k and pass^k

Definitions [S1]: **pass@k** = at least one of k trials succeeds. **pass^k** = all k trials succeed.

If trials are independent with per-trial success rate p:

```text
pass@k = 1 - (1 - p)^k
pass^k = p^k
```

Worked values for p = 0.75 (the example in [S1] gives pass^3 of about 42%):

| k | pass@k | pass^k |
|---|---|---|
| 1 | 0.750 | 0.750 |
| 3 | 0.984 | 0.422 |
| 5 | 0.999 | 0.237 |

Read it as: pass@k answers "can it ever do this?", which suits capability exploration. pass^k answers "will it do this every time?", which suits anything a customer hits repeatedly [S1]. A good pass@5 can sit next to a poor pass^5.

Estimating p: run n trials per task, count c successes, p = c / n per task, then average across tasks. Report n. With small n, small differences are noise; one Anthropic regression appeared in only one of two runs [S10].

## 5. Capability vs regression suites

| Suite | Expected pass rate | Purpose | Gate? |
|---|---|---|---|
| Capability | starts low [S1] | shows headroom, tracks progress | report only |
| Regression | "nearly 100%" [S1] | catches breakage of things that used to work | blocks merge |

Promote a capability task into the regression suite once it passes reliably.

## 6. Noise floor

Before calling a delta real:

- Pin and record resource limits, timeouts and harness version. On Terminal-Bench 2.0, resource configuration alone moved scores by 6 points; Anthropic treats gaps under 3 points skeptically unless configs match [S5].
- Rerun at different times and days [S5].
- Separate infra errors from task failures in the report.
- Check for contamination: read transcripts for the agent finding the benchmark or answer key. Anthropic found 11 BrowseComp answers came from benchmark materials and content-level blocking beat URL blocklists [S6].

## 7. Cost per pass

Log tokens, tool calls, turns and wall time per trial. On BrowseComp, token usage alone explained 80% of performance variance, and multi-agent systems used about 15x chat tokens [S2]. Compare variants on passes per dollar, not score alone.

## 8. Retrieval and segment metrics

- Measure retrieval on its own: top-k retrieval failure rate. Anthropic reports top-20 failure going 5.7% -> 3.7% -> 2.9% -> 1.9% as contextual embeddings, contextual BM25 and reranking were added [S4].
- Report by segment (new items, tail queries, task type). Zepto reports head-query benchmarks ignore ~80% of query space and reports segment metrics instead of one global percentage [S20].

## 9. CI gating recipe

1. **Trigger.** Any change to a system prompt, model id, tool schema, retrieval config or harness runs the regression suite. Anthropic now runs a broad per-model suite for every Claude Code system prompt change [S7].
2. **Run.** k trials per task, pinned infra config, results stored with model id, prompt hash and harness version.
3. **Gate.** Fail the check if regression pass^k drops below its baseline beyond the measured noise floor, or if format validity drops.
4. **Report.** Post per-segment numbers, cost per pass, and links to the three worst transcripts on the PR.
5. **Ratchet.** Every production miss and every revert becomes a new task; keep a change only if the score rises [S12].
6. **Soak.** For changes that trade quality for speed or cost, add a soak period and gradual rollout after the gate [S7].
7. **Monitor.** Keep a continuous quality eval on production traffic; after the Sep 2025 degradations Anthropic added more sensitive evals and continuous quality evals on production [S3].
