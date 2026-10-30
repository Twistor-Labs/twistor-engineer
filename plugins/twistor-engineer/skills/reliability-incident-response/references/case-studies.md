# Outage library

One entry per incident, newest first. Every figure comes from the company's own postmortem (source ids resolve in sources.md, reproduced in short at the end of this file so it can be read alone). Where a field says "not in the verified notes", the post may cover it but this skill has not checked it; read the post before quoting.

Fields: **Date**, **Trigger**, **Blast radius**, **Why it spread**, **Fix**, **Lesson**, **Rules**, **Source**.

## Cloudflare BYOIP prefix withdrawal (2026-02-20)

- **Date:** 2026-02-20; postmortem 2026-02-21.
- **Trigger:** a cleanup task passed `pending_delete` with no value, and the API read the empty value as "all prefixes".
- **Blast radius:** 25% of BYOIP prefixes were unintentionally withdrawn (of 4,306); impact lasted 6 hours and 7 minutes.
- **Why it spread:** the destructive path had no guard against an empty selector and no cap on how many prefixes one run could touch.
- **Fix:** Cloudflare files it under its "Code Orange: Fail Small" program; individual remediations are not in the verified notes.
- **Lesson:** an empty selector on a destructive job must mean "nothing", never "everything"; dry-run and cap blast radius.
- **Rules:** 7.
- **Source:** [S6]

## Cloudflare WAF change for a React vulnerability (2025-12-05)

- **Date:** 2025-12-05, 08:47 to 09:12 UTC (~25 min).
- **Trigger:** raising the WAF body buffer from 128KB to 1MB for CVE-2025-55182 (React Server Components), then disabling an internal WAF test tool through the global config system.
- **Blast radius:** about 28% of all HTTP traffic got HTTP 500. Not an attack.
- **Why it spread:** the global config system has no gradual rollout, and an FL1 Lua bug ("attempt to index field 'execute' (a nil value)") turned the change into errors everywhere at once.
- **Fix:** not in the verified notes; impact ended at 09:12 UTC.
- **Lesson:** an urgent security change is still a change; config pushes need stages and a nil-safe consumer.
- **Rules:** 2, 4.
- **Source:** [S2]

## Cloudflare Bot Management feature file (2025-11-18)

- **Date:** 2025-11-18. ClickHouse change deployed 11:05 UTC; network failures 11:20; first customer HTTP errors 11:28; core traffic largely restored 14:30; all services resolved 17:06 UTC.
- **Trigger:** a ClickHouse access-control (permissions) change. A metadata query that does not filter by database name began returning duplicate column rows from the underlying r0 database, more than doubling the rows.
- **Blast radius:** the Bot Management feature file doubled in size. The module preallocates for a limit of 200 features against about 60 in use; the oversized file broke that limit and the FL2 Rust proxy panicked: "thread fl2_worker_thread panicked: called Result::unwrap() on an Err value". Customers on the older FL proxy got no 5xx, but every request got a bot score of 0, causing false positives for anyone blocking on bot score.
- **Why it spread:** the feature file is generated internally and pushed fleet-wide, so nothing validated it as untrusted input, and the consumer crashed instead of keeping the previous file.
- **Fix:** recovery timeline above; detailed remediations are not in the verified notes.
- **Lesson:** treat generated config as untrusted input, avoid `unwrap()` on hot paths, roll out config changes in stages. Also: a "safe" degraded mode (score 0) can still be a silent outage for customers who act on it.
- **Rules:** 2, 3, 4.
- **Source:** [S1]

## Anthropic three overlapping quality issues (Aug to Sep 2025)

- **Date:** routing bug from 2025-08-05; misconfiguration deployed 2025-08-25; postmortem 2025-09-17.
- **Trigger:** three separate bugs. (1) Some Sonnet 4 requests were routed to 1M-context servers. (2) A misconfiguration produced stray Thai and Chinese characters in output. (3) An Aug 25 change exposed a latent XLA:TPU compiler bug.
- **Blast radius:** routing started at 0.8% of Sonnet 4 requests and hit 16% in the worst hour on Aug 31; about 30% of Claude Code users had at least one misrouted message. Bedrock misrouting peaked at 0.18% of Sonnet 4 requests; Vertex AI under 0.0004%. Corruption hit Opus 4.1 and Opus 4 from Aug 25 to 28 and Sonnet 4 from Aug 25 to Sep 2. Third-party platforms were not affected by the corruption.
- **Why it spread:** low-rate quality regressions overlapped and did not show up as errors, so aggregate signals did not separate them.
- **Fix:** corruption rolled back Sep 2; compiler change rolled back for Haiku 3.5 on Sep 4 and Opus 3 on Sep 12, and for Sonnet 4 as a precaution. Follow-ups: more sensitive evals, continuous quality evals on production, faster privacy-preserving debugging tools, deploy-time detection tests for unexpected character output.
- **Lesson:** quality is an SLI. Probe correctness continuously, per platform, not only error rates.
- **Rules:** 1, 13.
- **Source:** [S11]

## Anthropic Claude Code quality reports (Mar to Apr 2026)

- **Date:** changes from 2026-03-04 to 2026-04-16; all resolved as of Apr 20; postmortem 2026-04-23.
- **Trigger:** three separate changes. Default reasoning effort lowered from high to medium on Mar 4 to cut latency. A caching change meant to clear old thinking once after over an hour idle cleared it every turn (Mar 26 to Apr 10). A verbosity system prompt added Apr 16 cost 3% in ablation.
- **Blast radius:** Claude Code, Agent SDK and Cowork, not the API. No percentage of affected users was published.
- **Why it spread:** each change traded quality for speed or cost and shipped without a soak period that would expose the trade.
- **Fix:** effort reverted Apr 7 (called "the wrong tradeoff"); caching bug fixed in v2.1.101; verbosity prompt reverted Apr 20. Process: per-model eval suite on every system prompt change, soak periods and gradual rollouts for intelligence-tradeoff changes. Usage limits reset for all subscribers on Apr 23.
- **Lesson:** a tradeoff change is a risky change; gate it on quality evals and soak it.
- **Rules:** 13.
- **Source:** [S12]

## Cloudflare 1.1.1.1 resolver (2025-07-14)

- **Date:** 2025-07-14, 21:52 to 22:54 UTC (62 min); postmortem 2025-07-15.
- **Trigger:** a 2025-06-06 service-topology change wrongly linked 1.1.1.1 prefixes to a pre-production Data Localization Suite service. A Jul 14 change triggered a global refresh that withdrew them.
- **Blast radius:** 1.1.1.1 down globally for 62 minutes. A Tata (AS4755) BGP hijack of 1.1.1.0/24 was visible but not causal.
- **Why it spread:** the June mistake had no live effect, so nothing flagged it; the refresh applied it everywhere.
- **Fix:** about 23% of the edge fleet needed staged reconfiguration to recover; other remediations not in the verified notes.
- **Lesson:** check config against intent at change time, not only on impact. A visible external anomaly (the hijack) is not automatically the cause.
- **Rules:** 2, 8.
- **Source:** [S4]

## Cloudflare Workers KV and third-party storage (2025-06-12)

- **Date:** 2025-06-12; outage lasted 2 hours and 28 minutes.
- **Trigger:** failure in storage "backed by a third-party cloud provider" (provider not named in the post).
- **Blast radius:** Workers KV saw 90.22% of requests failing; Access failed 100% of identity logins.
- **Why it spread:** many products, including the login path, depended on KV, and KV depended on one external store.
- **Fix:** not in the verified notes.
- **Lesson:** map dependencies of critical paths, especially login and control plane, and remove single external points.
- **Rules:** 10.
- **Source:** [S5]

## Google Cloud Service Control (2025-06-12)

- **Date:** incident 2025-06-12 (report date not shown on the page).
- **Trigger:** new Service Control code met policy data that "contained unintended blank fields"; "the null pointer caused the binary to crash".
- **Blast radius:** the Service Control binary crashed; us-central1 took "up to ~2h 40 mins" to fully recover because the Spanner-backed quota database was overloaded.
- **Why it spread:** the code path "did not have appropriate error handling nor was it feature flag protected", and restarting tasks "did not have the appropriate randomized exponential backoff".
- **Fix:** not in the verified notes beyond the quoted gaps.
- **Lesson:** new paths ship flag-off; validate replicated policy data; retries need jittered backoff or recovery itself becomes the next overload.
- **Rules:** 3, 4, 5, 6.
- **Source:** [S7]

## Slack single-AZ link fault and cells (2021-06-30)

- **Date:** 2021-06-30. Intermittent faults from 11:45am PDT on a link connecting one AZ to several other AZs in us-east-1; removed automatically at 12:33pm, put back, failed again at 5:22pm, removed permanently at 5:31pm. Postmortem of the migration 2023-08-22.
- **Trigger:** a flaky network link between AZs.
- **Blast radius:** user-visible errors from a single-AZ fault.
- **Why it spread:** services received and sent traffic across AZs, so one AZ's trouble reached everyone.
- **Fix:** treat each AZ as a cell. Most services are siloed to their own AZ, and a failing AZ can be drained. Design goals: remove as much traffic as possible from an AZ within 5 minutes, with no user-visible errors, in 1% weight steps, driven by a 99.99% SLA (under 1 hour of downtime a year). Not every service is siloed (Vitess needs a single backend for any given write). The 5-minute figure is a goal, not a measured drain time.
- **Lesson:** draining is the fastest mitigation only if you built for it.
- **Rules:** 1, 12.
- **Source:** [S13]

## Meta backbone and BGP (2021-10-04)

- **Date:** 2021-10-04; details post 2021-10-05.
- **Trigger:** "a command was issued with the intention to assess the availability of global backbone capacity"; "a bug in that audit tool prevented it from properly stopping the command".
- **Blast radius:** Meta's backbone; DNS servers "disable those BGP advertisements if they themselves can not speak to our data centers", so DNS went unreachable too. Meta states no duration; the widely quoted figure is press-sourced only and is not used here.
- **Why it spread:** a safety check (the audit tool) failed, and a protective DNS behavior amplified the backbone loss.
- **Fix:** not in the verified notes.
- **Lesson:** recovery access must not depend on the network it is recovering; test the guard on your audit tooling.
- **Rules:** 9.
- **Source:** [S8]

## Cloudflare WAF regex (2019-07-02)

- **Date:** 2019-07-02; postmortem 2019-07-12.
- **Trigger:** a new WAF regex "that backtracked enormously".
- **Blast radius:** CPU "nearly 100%"; early on Cloudflare "lost 80% of our traffic"; "the Cloudflare service went down for 27 minutes".
- **Why it spread:** backtracking cost is paid per request, so CPU saturated wherever the rule ran.
- **Fix:** not in the verified notes.
- **Lesson:** per-request rules need linear-time evaluation or a CPU budget, and a canary that watches CPU.
- **Rules:** 2, 11.
- **Source:** [S3]

## Supporting precedents (not outages)

- **Netflix Hystrix.** README: "Hystrix is no longer in active development, and is currently in maintenance mode." Netflix points new work to resilience4j. Lesson: pick a maintained resilience library. Rules: 6. [S9]
- **Uber Piranha (2020-03-17).** Used to "remove around two thousand stale feature flags and their related code"; implemented for Objective-C, Swift and Java. Lesson: flags are debt with an expiry; automate removal. Rules: 15. [S10]
- **Anthropic CI on-call (2026-08-18).** Paging rule: "If the error rate is greater than 2% for longer than 5 minutes AND it's not a known deploy window." First analysis typically within 15 minutes (median 14). Verify each mitigation; keep a lessons log that every new investigation reads first. Rules: 14. [S14]
- **Anthropic test impact analysis (2026-09-14).** Avoid singleton critical services (three patches lasted 70 days, 29 days, then under a day); canary rollouts; alert on backlog and check jobs-in equals jobs-out. Rules: 2, 14. [S15]

## Source key

S1 Cloudflare 2025-11-18. S2 Cloudflare 2025-12-05. S3 Cloudflare 2019-07-02. S4 Cloudflare 1.1.1.1 2025-07-14. S5 Cloudflare Workers KV 2025-06-12. S6 Cloudflare BYOIP 2026-02-20. S7 Google Cloud Service Control 2025-06-12. S8 Meta 2021-10-04. S9 Netflix Hystrix README. S10 Uber Piranha. S11 Anthropic Sep 2025 postmortem. S12 Anthropic Apr 2026 postmortem. S13 Slack cells. S14 Anthropic CI on-call. S15 Anthropic test impact analysis. Full URLs in sources.md.
