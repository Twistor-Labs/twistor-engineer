# Startup scale: Zepto and Indian startup lessons

Read when the team is small, traffic arrives in sharp peaks, or users sit on cheap phones and thin data. Figures are company self-reported. Sources marked "existence only" are further reading; no numbers are taken from them.

## 1. Small teams: fix the hot stage, do not rewrite

- Zepto's frontend platform takes 600+ PRs a month from 10+ engineers. When builds passed 20 min, the team profiled, found 13+ min in Terser, and swapped only the minifier. Build time fell to ~2 min with no bundler migration. [S6]
- Zerodha's "Scaling with common sense" makes the same point for backends: the database is the usual bottleneck, so a framework that does 500K rps on top of a 1K qps database serves 1K rps. Profile the slowest link before buying a faster framework. [S14]
- Meta built Threads "in about five short months" and reports "over 100M users in its first five days". A tight launch window is a reason to reuse existing infrastructure, not to skip release safety. [S18]
- Small-team default: one profile, one targeted swap, one measured result, then stop.

## 2. Peak traffic: admit, do not just scale

- For demand spikes Zepto built a CDN waiting room: a CloudFront Function reads rules from CloudFront KeyValueStore and lets eligible users through. Rule changes are live in seconds with no deploy. Zepto reports 5xx/4xx down 80% and zero downtime in high-demand events. [S7]
- Zerodha treats 10x bursts over 10 s as normal for trading, and reports one Go service holding hundreds of thousands of WebSockets. Size the peak plan from the burst, not the daily average. [S14]
- Client side of the same plan: the app must render a clear wait or fallback screen, retry with jittered backoff, and never hammer origin on reconnect.
- Further reading, existence only: Razorpay, "Handling Burst Traffic During IPL" (2021) [S21].

## 3. OTA updates for users on thin data

- Before Phoenix, Zepto's ~6 MB updates reached only 18-20% of users in 4 days, and Android rollbacks hit 40-50% of installs. [S5]
- Phoenix ships bsdiff patches between Hermes bytecode bundles, ~500 KB on average. Patches are generated against every live bundle for that JS version. Download P90 is 305 ms and the update is ready in 1.3 s. [S5]
- Reach: Android 80%+ in under 24 h with a background runner; ~70% in 2 days on iOS and default Android; 15M requests/day. [S5]
- The JS bundle version is tied to native compatibility, not to the native version, so one patch serves every compatible binary. [S5]
- Building an in-house OTA platform is a real cost. Zepto did it because full-bundle updates were not reaching users. Start with an existing OTA service and measure adoption at 24 h and 4 days before deciding to build.

## 4. Search latency budgets

- Zepto's search budget is sub-200 ms end to end. Ranking reads pre-materialized features because live compute does not fit. [S9]
- The shipped relevance ranker has 4M params (TinyBERT-L2 class, ~17 MB) and decides in under 100 ms. It was chosen over ModernBERT-base (100M params, ~600 MB) and MiniLM-L6 (22M). [S8]
- Personalisation: a naive transformer pass over 60 candidates took hundreds of ms; encoding the user sequence once and reusing it across candidates with zero-copy `torch.expand()` brought P99 to single-digit ms. [S13]
- Vernacular queries ("kothimbir", "paal") need phonetic intent mapping, not edit distance. A search box for Indian users must be tested on these. [S9]
- Measure quality on the tail: head-query benchmarks ignore ~80% of the query space, so Zepto reports segment-level metrics. [S10]
- Further reading, existence only: Zomato, "How Zomato Handles 100 Million Daily Search Queries" (2023) [S22].

## 5. The API behind the screen

- Zepto's catalog (product enrichment) serves 1.5M batch reads/min to listings, cart, search and recs. Normalizing storage turned 1,000+ writes per product update into one, cut infra 85%, and dropped API latency 50-70%. [S12]
- Lesson for client teams: if a listing screen misses its budget, ask for the read path's p99 before shaving client milliseconds.

## 6. Low-end Android

- No Zepto post on low-end Android rendering or startup was found in research. Do not cite one.
- Closest sourced evidence: Discord reports animated WebP lets low-end devices hit 60 fps and halved Android median startup in 2023 [S3]; Zepto's small OTA patches serve data-constrained users [S5].
- Further reading, existence only: Shopify, "Improving Shopify App's Performance" (2024) [S20].

## Quick defaults for a small Indian consumer app

| Concern | Default |
|---|---|
| Budget device | the cheapest Android your users hold, not the team's phones |
| OTA | diff patches, keyed on native compatibility, staged by percentage |
| Peak events | edge admission rules editable without deploy, plus a client wait screen |
| Retries | randomized exponential backoff with a cap |
| Search | fixed end-to-end budget, precomputed features, smallest model that holds quality |
| Builds | profile first, swap the hot stage |
| Quality | segment metrics on the long tail and vernacular queries |
