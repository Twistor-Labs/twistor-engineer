---
name: frontend-mobile-engineering
description: 'Use when a mobile or web client is slow to start, janky, heavy to download, risky to release or brittle on bad networks. Triggers on "app startup is slow", "dropped frames", "OTA update", "CodePush", "roll this out safely", "kill switch", "server-driven UI", "should the backend control layout", "React Native performance", "webpack build is slow", "app falls over during a sale", "retry storm", "migrate Enzyme to RTL". Nouns: performance budget, cold start, bundle size, Hermes, staged rollout, remote config, SDUI, RIBs, waiting room, backoff. Gives a client plan in fixed order (budget, release safety, server vs client driven, network resilience, tests) backed by Airbnb, Uber, Discord, Zepto and Zerodha cases. Not for backend p99 or GC tuning (use performance-engineering), visual polish (use hallmark), or a web vitals audit (use Cloudflare web-perf).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Budgets, safe releases, server-driven UI and peak-proof clients, from Airbnb, Uber, Discord and Zepto.'
---

# Frontend and Mobile Engineering

The client is where every backend win either reaches the user or dies. This skill treats a mobile or web client as a system with a budget, a release pipeline and a failure mode, not as a pile of screens. Every rule comes from a company that shipped it and wrote the numbers down.

## When to use / not

- Use: setting or defending a startup, frame or size budget for an app or web client.
- Use: designing OTA updates, staged rollouts, remote flags or a kill switch for a client.
- Use: deciding whether a surface should be server-driven (layout from the backend) or client-owned.
- Use: a client that melts down at peak traffic, retries into an outage, or needs a waiting room.
- Use: choosing a test strategy or migrating a large frontend test suite.
- Not: server p99, GC, CPU throttling or query tuning (see performance-engineering).
- Not: visual design, typography, motion taste (see hallmark).
- Not: a Core Web Vitals audit of a marketing site (see Cloudflare web-perf, link only).

## Rules

1. **Instead of** a vague "make the app faster" goal, **do** set numeric budgets for median cold start, slow frames on your heaviest list, and memory, then track each per release. **Why:** Discord reports halving Android median startup in 2023 and up to 60% fewer slow frames in the chat list, plus ~12% less chat-list memory [S3]
2. **Instead of** testing performance on flagship phones, **do** pick asset formats and render paths by what the low-end device holds at frame rate. **Why:** Discord reports animated WebP let low-end devices hit 60 fps [S3]
3. **Instead of** budgeting only the client, **do** give each stage of a user-facing request a latency share and move expensive work out of the request path. **Why:** Zepto says live feature compute "is incompatible with a sub-200ms latency budget" and reads pre-materialized features instead [S9]
4. **Instead of** shipping the biggest model or payload that fits, **do** shrink to the smallest one that meets quality inside the budget. **Why:** Zepto compared a 100M-param ModernBERT with smaller models and shipped a 4M-param ranker deciding in under 100 ms [S8]
5. **Instead of** migrating the bundler when frontend builds crawl, **do** profile the build and replace the one hot stage. **Why:** Zepto found 13+ min inside Terser, swapped the minifier to esbuild and went from 20 min to ~2 min with no bundler migration [S6]
6. **Instead of** pushing full JS bundles as OTA updates, **do** ship binary diffs generated against every live bundle. **Why:** Zepto's Phoenix cut updates from ~6 MB to ~500 KB and moved Android to 80%+ adoption in under 24 h, against 18-20% in 4 days before [S5]
7. **Instead of** keying the OTA bundle version on the app version, **do** key it on native compatibility so one bundle serves every binary that can run it. **Why:** Phoenix ties the JS bundle version to native compatibility, not to native version [S5]
8. **Instead of** flipping a config or flag for everyone at once, **do** roll it out by percentage with a tested off switch and a halt on error-rate rise. **Why:** Cloudflare's 2025-12-05 global config change had no gradual rollout and hit ~28% of HTTP traffic for ~25 min; Google Cloud's 2025-06-12 change was not flag protected [S15][S16]
9. **Instead of** letting flags live forever, **do** give each flag an owner and an expiry and delete dead branches with tooling. **Why:** Uber's Piranha removed around two thousand stale flags across Objective-C, Swift and Java [S17]
10. **Instead of** hardcoding layout per platform for surfaces that change every sprint, **do** let one backend response define layout, sections and actions over typed client components. **Why:** Airbnb's Ghost Platform drives web, iOS and Android from a single backend response [S1]
11. **Instead of** one fixed grid for every query or category, **do** treat density and layout as data the server picks per intent. **Why:** Zepto uses 3x3 grids for "garbage bag" and 2x2 with bigger images for "T-shirt", picked by intent [S11]
12. **Instead of** server-driving flows with deep state and business logic, **do** keep them client-owned and structure them as a tree of business-logic units. **Why:** Uber's 2016 rider app is "a tree of Riblets" whose routing is driven by business logic, not views [S2]
13. **Instead of** clients retrying failed calls at once, **do** use randomized exponential backoff and honor a server-sent wait or fallback state. **Why:** Google Cloud's Service Control lacked randomized backoff and us-central1 took up to ~2h 40m to recover [S16]
14. **Instead of** deploying code to survive a peak event, **do** put admission rules at the CDN edge, editable in seconds, that let priority users through and send the rest a wait screen. **Why:** Zepto reports 80% fewer 5xx/4xx and zero downtime in high-demand events; Zerodha treats 10x bursts over 10 s as normal [S7][S14]
15. **Instead of** hand-migrating a test suite file by file, **do** run an LLM migration loop with validation, then sample, tune and sweep the failures, and finish the tail by hand. **Why:** Airbnb moved ~3.5K React test files in 6 weeks against a 1.5-year manual estimate: 75% in four hours, 97% after 4 days of tuning, last 3% by hand [S4]

## Answer shape

Every answer from this skill uses these sections, in this order. Skip a section only by saying why.

1. **Performance budget.** Numbers for cold start, slow frames, memory, download size and the end-to-end latency of the key interaction. Name the device tier the budget is for. Split the latency across client, network and each backend stage (rules 1-5).
2. **Release safety.** How code and config reach users: store build cadence, OTA patch path, staged percentages, the metric that halts a rollout, the kill switch, and when each flag dies (rules 6-9).
3. **Server-driven vs client-driven.** For each surface in scope, one line: who owns layout, who owns state, and why. Use the decision table below (rules 10-12).
4. **Network resilience.** Retry policy with jitter, what the client shows on failure, offline or cached state, and how peak traffic is admitted (rules 13-14).
5. **Test strategy.** What runs per PR, per release, and on real low-end devices; how quality is measured on the long tail, not just head cases; and how a large migration gets done (rule 15).
6. **Open questions.** What you could not size without data from the team.

### Server-driven vs client-driven

| Signal | Lean server-driven | Lean client-owned |
|---|---|---|
| Change rate | Layout or ranking changes weekly | Flow is stable for quarters |
| State | Mostly read and render | Deep local state, multi-step forms, offline edits |
| Platforms | Same surface on web, iOS, Android | One platform, platform-native feel matters |
| Release lag | Store review would block the change | Change can wait for a binary |
| Failure | A stale cached response is acceptable | Must work with no network at all |

A mixed answer is normal: a server-driven feed inside a client-owned shell.

### Budget table template

Fill this in section 1. Leave a cell as "unknown" rather than guess; an unknown is an open question.

| Metric | Device tier | Target | Current | Owner |
|---|---|---|---|---|
| Cold start, median | low-end Android | | | |
| Slow frames on the heaviest list | low-end Android | | | |
| Memory on the heaviest screen | low-end Android | | | |
| OTA patch size, median | all | | | |
| Key interaction end to end (e.g. search) | all | | | |
| Frontend CI build time | n/a | | | |

### Release ladder

Stop at the first rung that holds the risk. Climb only when a lower rung cannot.

1. Server-side change, no client release (layout, copy, ranking behind SDUI).
2. Remote flag flip, staged by percentage, with a halt metric.
3. OTA patch, staged by percentage, keyed on native compatibility.
4. Store binary with phased release, behind a flag that ships off.

Each rung needs a tested way back down: flag off, previous bundle, previous binary.

## Red flags

- "We will optimize later, ship first." -> Write the budget now; a budget set after launch is a regression report (rule 1).
- "It runs fine on my phone." -> Test on the cheapest device your users actually hold (rule 2).
- "Let's migrate to a new bundler." -> Profile first; the hot stage may be one plugin (rule 5).
- "OTA means we can skip staged rollout." -> OTA is a release; stage it like one (rules 6, 8).
- "The flag is just config, it is safe." -> Config changes caused the Cloudflare and Google Cloud outages cited here (rule 8).
- "Put everything behind server-driven UI." -> Flows with deep state stay client-owned (rule 12).
- "The client should just retry." -> Without jitter, retries are a load test you did not schedule (rule 13).
- "We will scale up for the sale." -> Also put a waiting room at the edge (rule 14).
- "Our search tests pass on the top queries." -> Zepto found head-query benchmarks miss ~80% of query space [S10].

## Verification

Before calling the work done, show:

- The budget table with current values measured on a named low-end device, not targets alone.
- A rollout plan with percentages, the halt metric, and a kill switch you have flipped in staging.
- For OTA: patch size against full bundle size, and the native-compatibility key.
- For each server-driven surface: the cached fallback when the layout response fails.
- Client retry config showing backoff with jitter and a cap.
- Test evidence that covers long-tail segments, not only head cases.

## References

| File | Read when |
|---|---|
| references/case-studies.md | you need the full story behind a rule, with numbers |
| references/startup-scale.md | the team is small, traffic spikes hard, or users are on cheap phones and thin data |
| references/checklist.md | reviewing a client PR or shipping a release |
| references/sources.md | citing a figure or checking a source's status |

Companions: performance-engineering (backend latency), hallmark (visual quality), Cloudflare web-perf (link only, web vitals).
