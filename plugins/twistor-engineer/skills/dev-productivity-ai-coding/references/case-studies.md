# Case studies

Company figures are the company's own report. Source ids point to `sources.md`.

## Anthropic: agentic coding practice (living docs)
- Context: the April 2025 best-practices post is now a living docs page [S1].
- What they say: split controls into non-deterministic (prompt, CLAUDE.md, skills) and deterministic (hooks, deny rules, managed settings); a CLAUDE.md line that keeps getting missed belongs in a hook. The test or typecheck command is the most useful CLAUDE.md line. Write a SPEC.md, build in a fresh session, review with a fresh subagent.
- Numbers: a Stop hook blocks with exit code 2; after eight consecutive blocks with no tool call, Claude Code overrides it.
- Lesson: prose steers, hooks enforce. Keep the enforced set small. Rules 2, 3, 4.

## Anthropic: long-running agent harness
- Context: building a claude.ai clone across many sessions [S2].
- What they did: an initializer session writes `init.sh`, a progress file and an initial commit; later sessions read progress and `git log`, run an end-to-end check, take one feature, commit.
- Numbers: feature list of over 200 features in JSON, each starting `"passes": false`.
- Lesson: state lives on disk, not in the context window. Testing like a user (browser automation) caught what unit tests missed; browser-native alert modals were invisible to the agent. Rule 7.
- Follow-up [S3]: a solo agent took 20 min and $9 but its core feature did not work; the full planner-generator-evaluator harness took 6 hr and $200. Re-check harness parts on each new model.

## Anthropic: C compiler with parallel agents
- Context: 16 parallel agents, nearly 2,000 sessions, about $20,000 in API fees, a 100,000-line Rust C compiler [S4].
- What they did: hardened the verifier, added CI once new features broke old ones, made test output grep-friendly, added a `--fast` flag running a random 1% or 10% sample, used GCC as an oracle to split the failing Linux build so agents fixed different files.
- Numbers: 99% pass rate on most test suites; still calls GCC for 16-bit x86 boot.
- Lesson: the verifier is the spec. Rule 6.

## Anthropic: source control under agent load
- Context: merges went from about 2,500/day (May 2026) to over 30,000/day by September [S5].
- What they did: server-side source control queries (Scion), optional sparse checkouts where CI asks the server for diffs, tiered caches.
- Numbers: main history almost 100 GB; about 25 million CI checkouts/day; checkouts minutes to seconds; CI gained 25% CPU; over 99% cache hit rate per tier; 200x less CPU and memory than the old git mirrors.
- Lesson: every bottleneck removed produces more code. Plan review and CI capacity before adding agents. Rules 8, 14.

## Anthropic: code review for agent-heavy repos
- Context: the Claude Code team's own review process [S6].
- What they did: a classifier sets the approval tier (low 0 human approvals, medium 1, high 2; agent approval never counts as human), a 3-round cap then diff-only re-review, past incident and revert PRs as a daily eval for the reviewer.
- Numbers: one-round pass 65% to 78%; 5+ rounds 8.9% to 1.5%; mean reviews to pass 2.08 to 1.32.
- Related [S7]: PRs over 1,000 lines get findings 84% of the time (7.5 issues on average); under 50 lines, 31% (0.5). Budget around 20 minutes and $15-25 per review. [S12]: report at most five nits per review.
- Lesson: small PRs, capped rounds, tiered humans. Rules 11, 12.

## Uber: uReview
- Context: AI code review across Uber's monorepos [S14].
- Numbers (Uber reports): analyzes over 90% of ~65,000 weekly diffs; engineers mark 75% of comments useful; over 65% of posted comments addressed; about 1,500 hours saved weekly. One section of the post says 65,000 per month.
- Lesson: judge an AI reviewer by useful and addressed, not by volume. Rule 11.

## Uber: Software Factory
- Context: agents at org scale, 2026 [S15].
- Numbers (Uber reports): more than 70% of pull requests attributed to local or cloud agents; 3,600+ agent skills; cost per session down 52% from the June peak; code-mode cut tokens 55-71% on small SQL tasks.
- Lesson: at this share of agent PRs, cost per session is a tracked metric, and review and CI become the constraint.

## Uber: SubmitQueue and CI
- Context: keeping a monorepo mainline green [S16].
- Numbers (Uber reports): CI resource usage down 53%, wait times down 37%; more than 4,500 engineers rely on it daily.
- Lesson: a merge queue that keeps main green is the base layer for agent volume. Rule 14.

## Uber: dynamic data race detection in Go
- Context: ~50M LoC Go monorepo, ~2,100 services [S17] (search-indexed).
- Numbers (Uber reports): more than 2,000 races detected, ~1,000 fixed by 200+ engineers.
- Lesson: automated detectors in CI find classes of bugs that review misses; feed new bug classes back into CLAUDE.md and checks.

## Meta: code review time
- Context: Meta picked P75 Time In Review as the north star because the slowest quartile grew by as much as a day [S18].
- What they did: NudgeBot for stale diffs, Next Reviewable Diff for reviewers.
- Numbers (Meta reports): average Time In Review down 7%; diffs waiting over 3 days down 12%; review actions per day up 17%.
- Lesson: measure the tail, nudge the stuck. Rule 13.

## Meta: Buck2
- Context: build system rewrite [S19] (search-indexed).
- Numbers (Meta reports): builds "twice as fast" as Buck1 in internal tests.
- Lesson: when the build is the bottleneck, it is worth rung 5, but only after profiling shows no smaller rung holds.

## Meta: TestGen-LLM and ACH
- Context: LLM-written tests accepted into production [S20]; mutation-guided test generation for privacy hardening [S21].
- Numbers (Meta reports): TestGen-LLM 73% of recommendations accepted; improved 11.5% of classes; on Reels/Stories 75% built, 57% passed reliably, 25% raised coverage. ACH paper: 73% of tests accepted at test-a-thons, 36% judged privacy-relevant.
- Lesson: filter generated tests by build, flakiness and added value before a human sees them. Rule 15.

## Airbnb: Enzyme to React Testing Library
- Context: nearly 3.5K React component test files [S13].
- What they did: LLM pipeline, then 4 days of "sample, tune, sweep" on failures.
- Numbers (Airbnb reports): 75% of files in four hours; 97% after 4 days of prompt tuning; last 3% by hand; 6 weeks against an estimated 1.5 years by hand.
- Lesson: automate the bulk, tune on failures, humans take the tail. Rule 10, and `llm-migrations.md`.

## LinkedIn: the last 20%
- Context: a generative AI product [S23].
- Numbers (LinkedIn reports): 80% of the basic experience in the first month; four more months trying to pass 95%.
- Lesson: budget the tail separately; it costs more than the bulk. Rule 10.

## Zepto: webpack builds 20 to 2 minutes
- Context: micro-frontend platform, 20 apps, 7 shared libs, 600+ PRs/month, builds over 20 minutes [S22].
- What they did: profiled, found 13+ minutes inside Terser, swapped the minifier for esbuild instead of migrating bundlers.
- Numbers (Zepto reports): esbuild minify ~295 ms vs terser ~6.7 s; output within ~5-8% of best size; build ~20 min to ~2 min.
- Lesson: profile, then take the smallest rung that holds. Rule 1.
