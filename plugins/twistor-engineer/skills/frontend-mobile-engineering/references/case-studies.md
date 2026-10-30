# Case studies

Each entry: context, what they did, numbers (self-reported by the company), lesson, source. Rule numbers point back to SKILL.md.

## Airbnb: Ghost Platform server-driven UI (rules 10, 11)

- Context: the same product surfaces had to ship across web, iOS and Android.
- What: the Ghost Platform, which Airbnb calls "a unified, opinionated, server-driven UI system". Layout, sections and actions are "controlled by a single backend response across our web, iOS, and Android apps".
- Lesson: when a surface changes faster than store releases and looks the same on every platform, move its layout into the response and keep a typed component library on the client.
- Source: [S1]

## Airbnb: Enzyme to React Testing Library with LLMs (rule 15)

- Context: nearly 3.5K React component test files on Enzyme, estimated at 1.5 years of engineering time to migrate by hand.
- What: an LLM-driven migration pipeline with validation, followed by a "sample, tune, sweep" loop over failures.
- Numbers: 75% of target files migrated in four hours; 4 days of tuning took it from 75% to 97%; the last 3% was finished by hand; whole migration in 6 weeks.
- Lesson: automate the bulk, spend days not months tuning on sampled failures, and plan a manual tail from the start.
- Source: [S4]

## Uber: rider app as a tree of Riblets (rule 12)

- Context: the 2016 rider app rewrite.
- What: "the entire application is structured as a tree of Riblets": Router, Interactor, Builder, with optional Presenter and View. Routing follows business logic, not the view hierarchy.
- Naming: the 2016 post says "Riblets". The pattern was open-sourced later as RIBs (repo created 2017-10). Do not call the 2016 architecture "RIBs, 2016".
- Lesson: flows with heavy state and business rules stay client-owned; shape them around logic so views can change without moving the logic.
- Source: [S2]

## Discord: faster React Native app (rules 1, 2)

- Context: Discord's mobile app on React Native, with chat lists and media as the heavy paths.
- Numbers: Discord reports Android median startup cut in half during 2023; up to 60% fewer slow frames in the chat list; ~12% less chat-list memory; animated WebP lets low-end devices reach 60 fps; iOS media-picker previews 4.5 s faster.
- Lesson: name the heavy screen, measure startup, slow frames and memory on it separately, and choose formats by what low-end devices can render.
- Note: this 2025-03-05 post is the latest Discord-authored perf post found. Newer New Architecture work appears only in partner posts, not cited.
- Source: [S3]

## Zepto: Phoenix, an in-house CodePush (rules 6, 7)

- Context: React Native app with OTA updates. Before Phoenix, updates were ~6 MB, only 18-20% of users adopted within 4 days, and Android rollbacks reached 40-50% of installs.
- What: bsdiff patches between Hermes bytecode bundles, generated against every live bundle for the JS version; a background runner on Android; bundle version tied to native compatibility, not native version.
- Numbers: average update ~6 MB to ~500 KB; download P90 305 ms, ready in 1.3 s total; adoption 60% to 80% in one day, Android 80%+ in under 24 h, ~70% in 2 days on iOS and default Android; 15M requests/day.
- Lesson: OTA reach is a size problem first. Diff, do not resend.
- Source: [S5]

## Zepto: webpack build from 20 min to ~2 min (rule 5)

- Context: micro-frontend platform with 20 apps, 7 shared libs, ~3,950 TSX/JSX and 6,600+ TS files, 600+ PRs a month from 10+ engineers. Builds took over 20 min.
- What: profiled the build, found 13+ min inside Terser, swapped the minifier to esbuild and kept the bundler.
- Numbers: esbuild minify ~295 ms vs Terser ~6.7 s on the same input; output within ~5-8% of best size; JS minify ~10x faster, CSS ~6x; build ~2 min.
- Lesson: profile before you migrate. The bottleneck was one stage, not the tool.
- Source: [S6]

## Zepto: CDN waiting room for traffic spikes (rule 14)

- Context: high-demand events that overwhelm origin.
- What: a CloudFront Function (not Lambda@Edge) reads rules from CloudFront KeyValueStore, admits eligible users (for example logged-in, Android only, a region) and returns a wait or fallback to the rest. Rule changes go live in seconds with no deploy.
- Numbers: 5xx/4xx down 80%; zero downtime in high-demand events.
- Lesson: admission control belongs at the edge, and the client must render a wait state well.
- Source: [S7]

## Zepto: search inside a sub-200 ms budget (rules 3, 4)

- Context: inventory-aware search for 10-minute delivery.
- What: ML ranking reads pre-materialized features because live feature compute "is incompatible with a sub-200ms latency budget". Lexical and semantic retrieval run in parallel.
- Ranker: ~40M labeled query-product pairs; compared ModernBERT-base (100M params, ~600 MB), MiniLM-L6 (22M, ~90 MB) and TinyBERT-L2 (4M, ~17 MB); shipped a 4M-param model deciding in under 100 ms.
- Measurement: head-query benchmarks ignore ~80% of query space, so Zepto reports segment-level metrics instead of one global number.
- Lesson: give every stage of the search request a share of the budget and pick the smallest model that holds quality.
- Sources: [S8][S9][S10]

## Zepto: catalog service behind the app (rule 3)

- Context: product enrichment serves 1.5M batch reads/min to listings, cart, search and recs. Denormalized writes pushed reads from under 10 ms to 100 ms+.
- What: normalized storage so one product update became one write instead of 1,000+.
- Numbers: 99.9% fewer writes; cluster 7 x M40 to 3 x M60 (85% less infra); API latency down 50-70%; per-pod throughput 3-4K to 15K req/min.
- Lesson: a client budget includes the API behind it. When a screen is slow, check the read path's write amplification too.
- Source: [S12]

## Zepto: grid density by intent (rule 11)

- What: 3x3 grids for "garbage bag", 2x2 with bigger images for "T-shirt", from ~300 customer conversations. Zepto also reports search-to-add-to-cart up ~18% in a year, as one result of the wider push beyond groceries; the post does not attribute it to grids alone.
- Lesson: layout density is a per-intent decision the server can make.
- Source: [S11]

## Release safety from outside mobile (rules 8, 9, 13)

- Cloudflare 2025-12-05: a WAF change was followed by disabling an internal test tool through the global config system with no gradual rollout; ~25 min, ~28% of all HTTP traffic affected, served as HTTP 500. Lesson: config is code; stage it. [S15]
- Google Cloud 2025-06-12: the change "did not have appropriate error handling nor was it feature flag protected", and lacked "randomized exponential backoff"; us-central1 took up to ~2h 40m to recover. Lesson: flag every new path, and make clients back off with jitter. [S16]
- Uber Piranha: used to "remove around two thousand stale feature flags and their related code" in Objective-C, Swift and Java. Lesson: flags need an exit. [S17]
