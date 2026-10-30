# Safe-change checklist

Run before shipping, or while reviewing, any change to config, flags, generated files, routing, retries, rules or destructive jobs. Each item names the SKILL.md rule it enforces. A "no" is a finding; report it as `<rule>: <problem>. <fix>.`

## Rollout

- [ ] Config ships through stages (canary, percentage, global), not a system that applies everywhere at once. (Rule 2)
- [ ] Each stage has a health gate on a named SLI and an automatic halt, not a human watching a graph. (Rule 2)
- [ ] The change is not a singleton critical service with no canary path. (Rule 2, see S15 in case-studies.md)
- [ ] Changes that trade quality for latency or cost have a soak period and a quality eval, not only error-rate checks. (Rule 13)

## Kill switch and rollback

- [ ] New code paths sit behind a flag that defaults off. (Rule 5)
- [ ] The flag works as a kill switch and was flipped off at least once in a non-production stage. (Rule 5)
- [ ] Rollback of this change was exercised and its time is known. (Rule 1)
- [ ] The flag has an owner and an expiry date for removal. (Rule 15)

## Inputs and failure handling

- [ ] Internally generated files (feature files, policy data, routing tables) are validated before publish: schema, row count, size against the consumer's hard limits. (Rule 3)
- [ ] The consumer validates again on load and keeps the last-known-good when validation fails. (Rule 3)
- [ ] No `unwrap()`, assert or unchecked null on the hot path that reads this input; the error path keeps serving and emits a metric. (Rule 4)
- [ ] New per-request rules (regex, WAF, filters) use a linear-time engine or a CPU budget, and the canary watches CPU. (Rule 11)

## Destructive operations

- [ ] Empty, missing or wildcard selectors are rejected, never read as "all". (Rule 7)
- [ ] A dry run prints the exact count of objects affected. (Rule 7)
- [ ] The job aborts above a blast-radius threshold (count or percentage). (Rule 7)

## Latent state

- [ ] Intended and actual state are diffed at change time, including parts with no live effect yet. (Rule 8)
- [ ] An alert fires on config that points at the wrong environment (for example production prefixes linked to a pre-production service). (Rule 8)

## Dependencies and recovery access

- [ ] Retries use capped exponential backoff with random jitter. (Rule 6)
- [ ] Circuit breakers come from a maintained library. (Rule 6)
- [ ] Login and control-plane paths do not share a single external store with the data plane. (Rule 10)
- [ ] Break-glass access exists on a separate network and credentials, and was rehearsed in the last quarter. (Rule 9)
- [ ] The service can be drained by cell, zone or region in small weighted steps. (Rule 12)

## Detection

- [ ] Paging rules state a rate, a duration and known-deploy-window exclusions. (Rule 14)
- [ ] A correctness probe runs on production per platform, not only an error-rate dashboard. (Rule 13)
- [ ] The rollout dashboard shows the SLI the mitigation will be verified on. (Rule 14)
