# Checklist

Use before merging a client change or shipping a release. Each item names the rule it enforces.

## Performance budget

- [ ] Budget table exists with target and current values for cold start, slow frames, memory (rule 1).
- [ ] Numbers were measured on a named low-end device, not only a flagship (rule 2).
- [ ] New media or animation formats checked for frame rate on that device (rule 2).
- [ ] The key interaction's latency is split across client, network and each backend stage (rule 3).
- [ ] No expensive compute added on the request path that could be precomputed (rule 3).
- [ ] Any on-device or in-path model is the smallest one that meets the quality bar (rule 4).
- [ ] If CI builds got slower, a build profile was taken before proposing a tool migration (rule 5).

## Release safety

- [ ] OTA updates ship as diffs against every live bundle, and patch size is recorded (rule 6).
- [ ] OTA bundle version is keyed on native compatibility; incompatible binaries cannot receive it (rule 7).
- [ ] New behavior ships behind a remote flag, default off (rule 8).
- [ ] Rollout percentages and the metric that halts the rollout are written down (rule 8).
- [ ] The kill switch has been flipped once in staging and the client recovered (rule 8).
- [ ] Every new flag has an owner and an expiry date; expired flags are on a cleanup list (rule 9).

## Server-driven vs client-driven

- [ ] Each surface in scope has a one-line ownership call: layout owner, state owner, reason (rules 10, 12).
- [ ] Server-driven surfaces render a cached or default layout when the response fails (rule 10).
- [ ] Unknown component types from the server degrade safely on old clients (rule 10).
- [ ] Layout density or variant choices that differ by intent are data, not hardcoded (rule 11).
- [ ] Flows with deep local state or multi-step logic stay client-owned (rule 12).

## Network resilience

- [ ] Retries use randomized exponential backoff with a cap (rule 13).
- [ ] The client honors a server-sent wait or fallback response (rules 13, 14).
- [ ] The waiting-room or fallback screen has been seen on a real device (rule 14).
- [ ] Peak-event admission rules can change without an app or server deploy (rule 14).
- [ ] Peak plan assumes bursts well above average traffic, sized from your own history (rule 14).

## Test strategy

- [ ] Per-PR tests cover user-visible behavior of changed components (rule 15).
- [ ] Release tests run on at least one low-end physical device (rules 2, 15).
- [ ] Quality metrics report long-tail segments, not only head cases (rule 15, see [S10]).
- [ ] Large test migrations use an automated loop with validation and a planned manual tail (rule 15).
