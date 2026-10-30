---
name: reliability-incident-response
description: 'Use when production is down or degraded, 5xx or error rates spike, a deploy or config push just went bad, someone asks "roll back or fix forward?", "how do we ship this config safely?" or "write the postmortem", "postmortem action items", or you review a risky change to feature flags, kill switches, global config, generated files, BGP or DNS, retries, cleanup jobs or a third-party dependency. Nouns: SLO, blast radius, canary, staged rollout, last-known-good, backoff, jitter, cells, break-glass, paging rule, blameless postmortem. Gives a smallest-mitigation-first ladder, rules sourced from real outages, an incident answer shape, a safe-change checklist and a postmortem template. Not for bugs with no production impact (use superpowers systematic-debugging), capacity design (use system-design-scaling) or intrusions (use security-engineering).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Mitigate first with the smallest reversible step, treat config as code, and close incidents with owned, triggered action items.'
---

# Reliability and Incident Response

Most large outages in the sources were not exotic. A config or data change shipped everywhere at once, an internally generated input broke an assumption, and a hot path crashed instead of degrading. This skill treats config as code, mitigation as a ladder you climb from the cheapest rung, and the postmortem as the place where systems, not people, get fixed.

**Iron law: mitigate before root-causing. Root-cause before closing.**

Users first, understanding second, but never close an incident on a mitigation alone.

## When to use / not

- Use: an incident is live, or was live and needs a postmortem.
- Use: reviewing a change that ships config, flags, generated files, routing, retries or a destructive job to production.
- Use: deciding rollback vs fix forward, or what to page on.
- Not: a bug with no production impact yet; debug it with superpowers systematic-debugging (link only, not bundled).
- Not: designing for 10x load or sharding (see system-design-scaling).
- Not: an attacker in the system (see security-engineering).

## Rules

1. **Instead of** root-causing while users are still failing, **do** take the lowest rung of the mitigation ladder (see Answer shape) that stops the bleeding, then investigate. **Why:** Anthropic ended its Aug 2025 output-corruption and compiler issues with rollbacks, and Slack built cells so a sick zone can simply be drained. [S11][S13]
2. **Instead of** pushing config through a global system that applies everywhere at once, **do** ship config like code: canary, percentage, then global, with a health gate and an automatic halt between stages. **Why:** Cloudflare's Dec 5 2025 change went out via the global config system with no gradual rollout and returned 500s to ~28% of HTTP traffic for ~25 min. [S2]
3. **Instead of** trusting files your own pipelines generate, **do** validate them before publish and again on load (schema, row count, size against hard limits) and keep the last-known-good to fall back to. **Why:** On Nov 18 2025 duplicate rows doubled Cloudflare's Bot Management feature file past a 200-feature limit (about 60 in use); Google's Service Control crashed on policy data with blank fields. [S1][S7]
4. **Instead of** `unwrap()`, asserts or null dereferences on a hot path fed by config, **do** handle the error by keeping the previous config, emitting a metric and serving traffic. **Why:** Cloudflare's FL2 proxy panicked with "called Result::unwrap() on an Err value" and Google's binary crashed on a null pointer. [S1][S7]
5. **Instead of** shipping a new code path switched on, **do** put it behind a flag that defaults off, enable it by percentage, and keep the flag as the kill switch until it has soaked. **Why:** Google's report says the new Service Control code "did not have appropriate error handling nor was it feature flag protected". [S7]
6. **Instead of** fixed-interval retries or a breaker built on an unmaintained library, **do** use capped exponential backoff with random jitter and a maintained circuit breaker. **Why:** Google's tasks lacked "randomized exponential backoff" and us-central1 took up to ~2h 40m to fully recover while its quota database was overloaded; Hystrix is in maintenance mode and Netflix points new work to resilience4j. [S7][S9]
7. **Instead of** letting a cleanup or delete job accept an empty or missing selector, **do** reject empty scope, dry-run and print the count, and abort above a blast-radius threshold. **Why:** On Feb 20 2026 a Cloudflare task passed `pending_delete` with no value, it was read as all prefixes, and 25% of 4,306 BYOIP prefixes were withdrawn for 6h 7m. [S6]
8. **Instead of** assuming a config change is safe because nothing broke when it shipped, **do** diff the intended and actual state at change time and alert on mismatches that have no live effect yet. **Why:** A Jun 6 2025 topology change wrongly linked 1.1.1.1 prefixes to a pre-production service and sat latent until a Jul 14 refresh withdrew them for 62 min. [S4]
9. **Instead of** reaching recovery tooling over the same network and auth path that is failing, **do** keep out-of-band, break-glass access with its own network and credentials, and rehearse it. **Why:** At Meta in 2021 an audit-tool bug let a backbone command through, then DNS servers withdrew their own BGP advertisements, so a single command took the backbone and the DNS routes down together. [S8]
10. **Instead of** hiding a critical dependency behind a single external store, **do** list each critical path's dependencies and give login and control-plane paths a fallback that does not share them. **Why:** In Cloudflare's Jun 12 2025 outage, Workers KV failed 90.22% of requests for 2h 28m and Access failed 100% of identity logins. [S5]
11. **Instead of** shipping new regexes or rules that run on every request untested, **do** use a linear-time engine or a CPU budget per rule, and canary new rules while watching CPU. **Why:** Cloudflare's 2019 WAF regex "backtracked enormously", pinned CPU near 100% and took the service down for 27 min. [S3]
12. **Instead of** one shared failure domain, **do** split traffic into cells (zone or region) that can be drained in small weighted steps. **Why:** In 2021 a single-AZ link fault still caused user-visible errors at Slack; its cell design targets draining a zone within 5 min in 1% steps. [S13]
13. **Instead of** trusting aggregate error dashboards to catch quality regressions, **do** run continuous correctness probes on production, per platform, plus soak periods for changes that trade quality for speed. **Why:** Anthropic's routing bug began at 0.8% of Sonnet 4 requests and peaked at 16% in one hour, and about 30% of Claude Code users saw at least one misrouted message. [S11][S12]
14. **Instead of** paging on vague thresholds or single spikes, **do** write numeric paging rules with a rate, a duration and known-deploy-window exclusions, and verify every mitigation against them. **Why:** Anthropic's on-call rule pages when "the error rate is greater than 2% for longer than 5 minutes AND it's not a known deploy window". [S14]
15. **Instead of** leaving kill switches and rollout flags in the code after launch, **do** give each flag an owner and an expiry, and delete stale ones with a tool. **Why:** Uber used Piranha to remove around two thousand stale feature flags and their code. [S10]

## Answer shape

### Mitigation ladder (rule 1)

Climb from the first rung. Stop at the first rung that holds, then verify on the same metric that paged you.

- Rung 1, **freeze.** Halt every in-flight rollout and config push. Nothing new ships until the graph moves.
- Rung 2, **flip the kill switch.** Turn the new path's flag off (rule 5).
- Rung 3, **roll back** the last code or config change, or pin the last-known-good generated file (rule 3).
- Rung 4, **drain** the failing cell, zone or region and shift traffic (rule 12).
- Rung 5, **degrade.** Shed load, disable the non-critical feature, serve stale.
- Rung 6, **add capacity**, only when saturation is the proven cause.
- Rung 7, **fix forward**, last, and only through the same staged pipeline as any other change (rule 2).

Pick rollback over fix forward when the last change is a plausible cause and rolling it back is tested. Pick fix forward only when rollback is impossible (data migrated, external contract changed) or is itself the riskier change, and say why in the timeline.

### Live incident

Keep it in this order and update it in place.

1. **Status line.** Severity, user impact in SLO terms (which SLI, how far over budget), start time in ISO UTC.
2. **Mitigation.** Current ladder rung, what was done, and the metric showing whether it held.
3. **Recovery rule.** A number, a rate and a duration that mean "recovered", for example checkout errors under the pre-deploy baseline for 15 minutes, outside known deploy windows (rule 14). If the baseline is unknown, say how to read it from the last day before the deploy.
4. **Timeline.** ISO UTC, one line per event, change events first.
5. **Leading hypotheses.** Ranked, each with the one observation that would confirm or kill it.
6. **Next action.** One owner, one action, next check time.
7. **Comms.** What customers and internal teams were told, and when the next update is due.

### Change review

Before shipping a risky change:

1. **Blast radius.** What fails, for whom, if this change is wrong.
2. **Rollout.** Stages, health gates, halt condition.
3. **Kill switch and rollback.** How fast, tested when.
4. **Input validation.** Limits checked before publish and on load.
5. **Checklist result.** Items from references/checklist.md that fail, by rule number.

### Postmortem

Use references/postmortem-template.md as-is. Match the incident against references/case-studies.md and name the closest precedent in the summary.

## Red flags

Stop when you catch yourself saying any of these.

- "Let me find the root cause first, then we roll back." -> Climb the ladder first (rule 1).
- "It's only a config change, it doesn't need a canary." -> Config is code (rule 2).
- "That file comes from our own pipeline, it's fine." -> Validate it (rule 3).
- "That can never be empty / never be None." -> Then handling it costs nothing (rule 4).
- "We'll add the flag after launch." -> Flag first, default off (rule 5).
- "Just retry every second." -> Backoff with jitter (rule 6).
- "Nothing broke when we shipped it, so it's safe." -> Latent config is still live config (rule 8).
- "We'll fix it forward, rollback is scary." -> Fix forward is the last rung, not the first.
- "Mitigated, closing the incident." -> Root-cause before closing.
- "Who pushed that?" -> Ask what let the push through, not who did it.
- Action items with no owner or no revisit trigger -> they will not happen; flag them `no-trigger`.

## Verification

Before calling an incident mitigated or a change safe:

- The metric that paged (or the SLI the change touches) is back inside its threshold for at least one full alert window, shown with numbers and timestamps.
- The mitigation was confirmed on the same signal that detected the problem, not on a proxy.
- The rollback or kill switch for the change was exercised in a non-production stage, with the time it took.
- Generated inputs have a validator that rejects an oversized or empty file in a test.
- For closing: root cause written, every action item has an owner, ceiling and revisit trigger, or is marked `no-trigger`.

## References

| File | Read when |
|---|---|
| references/case-studies.md | you need the full story behind a rule, or a precedent for the incident in front of you |
| references/checklist.md | reviewing or shipping a risky change |
| references/postmortem-template.md | writing or reviewing a postmortem |
| references/sources.md | citing a figure or date |

Related: superpowers systematic-debugging (link only, for code-level root-causing once users are safe).
