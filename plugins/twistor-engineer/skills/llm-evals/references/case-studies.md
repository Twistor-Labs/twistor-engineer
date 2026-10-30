# Case studies

Each entry: context, what they did, numbers, lesson, source. Company figures are the company's own claim.

## Anthropic: Demystifying evals for AI agents (rules 1, 2, 3, 5, 6, 7)
- Context: teams building agents put off evals because a full benchmark looks expensive.
- What: defines task, trial, grader, transcript, outcome and harness; combines code, model and human graders; splits capability and regression evals.
- Numbers: "20-50 simple tasks drawn from real failures is a great start"; 75% per trial gives about 42% pass^3; regression evals "should have a nearly 100% pass rate"; 0% pass@100 usually means a broken task or grader.
- Lesson: start small from real failures, prove each task is solvable, and pick pass^k when users need consistency.
- [S1]

## Anthropic: multi-agent research system (rule 9)
- Context: a lead agent with parallel subagents for research tasks.
- What: compared against a single agent on an internal research eval and analysed variance on BrowseComp.
- Numbers: Opus 4 lead with Sonnet 4 subagents beat single-agent Opus 4 by 90.2% on the internal eval; on BrowseComp three factors explained 95% of variance and token usage alone 80%; multi-agent systems used about 15x chat tokens.
- Lesson: a score gain can be mostly spend; log tokens per trial and compare cost per pass.
- [S2]

## Anthropic: postmortem of three recent issues, Sep 2025 (rule 15)
- Context: three overlapping infrastructure bugs degraded model output quality in Aug and Sep 2025.
- What: a context-window routing bug, an output-corruption misconfiguration, and a latent XLA:TPU compiler bug, each rolled back.
- Numbers: the routing bug began at 0.8% of Sonnet 4 requests on Aug 5 and peaked at 16% in the worst hour on Aug 31; about 30% of Claude Code users had at least one misrouted message.
- Lesson: fixes included more sensitive evals and continuous quality evals on production; offline evals alone are not a production monitor.
- [S3]

## Anthropic: April 2026 Claude Code quality postmortem (rule 7)
- Context: three separate changes degraded Claude Code, the Agent SDK and Cowork.
- What: one was a verbosity system prompt; another lowered default reasoning effort, later called "the wrong tradeoff".
- Numbers: the verbosity prompt cost 3% in ablation on Opus 4.6 and 4.7 and was reverted.
- Lesson: run a broad per-model eval suite on every system prompt change, then soak and roll out gradually.
- [S7]

## Anthropic: harness design for long-running apps (rule 8)
- Context: planner, generator and evaluator agents building full applications.
- What: separated the evaluator from the generator and agreed testable criteria with hard thresholds before each sprint.
- Numbers: one level-editor sprint had 27 criteria.
- Lesson: "tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work."
- [S8]

## Anthropic: infrastructure noise and eval awareness (rules 10, 11)
- Context: agentic coding and browsing benchmarks are sensitive to environment and leakage.
- What: varied container resources on Terminal-Bench 2.0; audited BrowseComp transcripts for contamination.
- Numbers: resource configuration moved Terminal-Bench 2.0 scores by 6 points (p < 0.01); 11 BrowseComp answers came from benchmark materials; blocklist rerun moved the multi-agent score from 86.81% to 86.57%.
- Lesson: pin infra config, distrust gaps under 3 points, and block benchmark material at the content level, not by URL.
- [S5][S6]

## Anthropic: Contextual Retrieval (rule 12)
- Context: RAG answers fail when the right chunk is never retrieved.
- What: measured retrieval alone as top-20 failure rate while adding contextual embeddings, contextual BM25 and reranking.
- Numbers: 5.7% -> 3.7% (35%), -> 2.9% (49%), -> 1.9% (67%).
- Lesson: give retrieval its own metric so a generation change cannot hide a retrieval regression.
- [S4]

## LinkedIn: building a GenAI product (rules 4, 15)
- Context: a generative AI product with structured model output.
- What: chose YAML over JSON for token cost and added a defensive parser.
- Numbers: 80% of the basic experience in the first month, then four more months to pass 95%; malformed outputs from ~10% to ~0.01%.
- Lesson: the last mile is where evals earn their keep; track format validity as its own number.
- [S14]

## Meta: TestGen-LLM and ACH (rule 13)
- Context: LLM-generated unit tests and mutation-guided privacy tests.
- What: every candidate test passed filters (builds, passes reliably, improves coverage) before an engineer saw it; ACH generated tests aimed at mutants.
- Numbers: TestGen-LLM on Reels and Stories, 75% built, 57% passed reliably, 25% raised coverage; 73% of recommendations accepted; improved 11.5% of classes it was applied to. ACH paper: 73% of tests accepted, 36% judged privacy-relevant.
- Lesson: grade generated code by what survives filters and what humans accept, not by volume.
- [S15][S16]

## Airbnb: Enzyme to React Testing Library migration (rule 14)
- Context: nearly 3.5K React component test files to migrate.
- What: ran the migration as a loop of "sample, tune, sweep", with the existing tests as the grader.
- Numbers: 75% of files in four hours, 97% after four days of tuning, last 3% by hand; 6 weeks total vs an estimated 1.5 years by hand.
- Lesson: an automated validation loop gets most of the way fast; budget human time for the tail.
- [S17]

## OpenAI: why language models hallucinate (rule 5)
- Context: models guess confidently instead of saying they do not know.
- What: argues that training and evaluation reward guessing over acknowledging uncertainty.
- Numbers: none used here (source was search-indexed only).
- Lesson: change scoring so abstention beats a wrong answer.
- [S18]

## Uber: uReview (rule 15)
- Context: AI code review across Uber's diffs.
- What: tracks whether engineers find comments useful and whether they act on them.
- Numbers: Uber reports it analyses over 90% of ~65,000 weekly diffs, 75% of comments marked useful, over 65% addressed, ~1,500 hours saved weekly.
- Lesson: in production, a usefulness signal from real users is the eval.
- [S19]

## Zepto: search quality measurement (rule 12)
- Context: quick-commerce search where head queries dominate traffic but not catalogue coverage.
- What: LLM-judged top-K results for 5-10 queries per product, reported per segment.
- Numbers: Zepto reports head-query benchmarks ignore ~80% of query space; 1M products x 10 queries = 10M query-product pairs.
- Lesson: segment metrics such as new listings reveal what a global score hides.
- [S20]

## Desert Ant Labs: publishing the rows you lose (rule 12)
- Desert Ant Labs reports diarization error rates for its model against pyannote and FluidAudio on five test sets.
- What: published every row, including the three where it loses; for an image tagger it advises a stricter 0.9 confidence cutoff for automatic actions, keeping 0.5 for suggestions.
- Numbers: leads on AMI meetings (12.6% vs 13.1% and 19.4%) but loses on Earnings-21 calls, 27.7% vs pyannote 7.3%.
- Lesson: per-segment reporting only works if losing segments stay in the table; tie the confidence threshold to the cost of acting on a wrong answer.
- [S21]
