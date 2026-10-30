# Postmortem template

Blameless by default: name systems, changes and missing guards, never a person as the cause. "What let this through?" replaces "who did this?". Copy everything below the line, fill every section, delete the guidance lines in italics.

How to use it:

1. Fill Summary and Impact first, from data, not memory.
2. Build the Timeline from logs and change records, in ISO 8601 UTC (`2026-01-15T09:30:00Z`). Change events (deploys, config pushes, flag flips) come first in any tie.
3. Separate the Trigger (the change or event that started it) from Root causes (the missing guards that let it become an outage). There is almost always more than one root cause.
4. Match against case-studies.md and name the closest precedent. If an earlier precedent's lesson would have prevented this, say so in What went wrong.
5. Every action item gets an owner, a priority, a ceiling and a revisit trigger. An item without a trigger is marked `no-trigger` and counted in the totals line.

---

# Postmortem: <service> <one-line symptom> (<YYYY-MM-DD>)

**Status:** draft | in review | final
**Severity:** <SEV level, set by SLO impact, not by how it felt>
**Incident window:** <start ISO UTC> to <end ISO UTC> (<duration>)
**Closest precedent:** <entry name from case-studies.md, or "none">

## Summary

*Three to five sentences: what users saw, what triggered it, what stopped it, the main root cause.*

## Impact

| Measure | Value |
|---|---|
| SLI affected | *e.g. availability: count(status < 500) / count(total)* |
| Peak error or failure rate | |
| Duration of user impact | |
| Error budget consumed | *percent of the window's budget* |
| Users, customers or requests affected | |
| Secondary effects | *e.g. degraded mode that silently changed results* |

## Timeline (ISO 8601 UTC)

| Time (UTC) | Event | Source |
|---|---|---|
| YYYY-MM-DDTHH:MM:SSZ | *change deployed* | *deploy log* |
| | *first error* | |
| | *page fired* | |
| | *mitigation rung N applied* | |
| | *mitigation verified on <SLI>* | |
| | *fully resolved* | |

*Add time-to-detect, time-to-mitigate and time-to-resolve under the table.*

## Trigger

*The single change or event that started it. Facts only.*

## Root causes

*Each missing guard that let the trigger become an outage. Map each to a SKILL.md rule where one fits.*

1. *e.g. generated file not validated against consumer limit (rule 3)*
2. *e.g. config pushed globally with no stages (rule 2)*

## What went well

- *Detection, tooling, people-process that worked. Keep it real, not polite.*

## What went wrong

- *Delays, missing access, misleading signals, alerts that did not fire.*

## Where we got lucky

- *Things that limited damage by chance. Each one is a candidate action item.*

## Action items

*Priority: P0 (before the next change of this kind ships), P1 (this quarter), P2 (backlog). Ceiling: the point at which this fix stops being enough (load, size, count, date). Revisit trigger: the observable event that reopens the item. If you cannot name a trigger, write `no-trigger`.*

| # | Action | Rule | Owner | Priority | Ceiling | Revisit trigger | Status |
|---|---|---|---|---|---|---|---|
| 1 | *Validate feature file size before publish and on load* | 3 | *team or role* | P0 | *limit of N entries* | *file grows past 80% of limit* | open |
| 2 | *Stage config pushes: canary, 10%, global* | 2 | | P0 | | *any config push skips a stage* | open |
| 3 | *Write runbook for rung 4 drain* | 1 | | P2 | | `no-trigger` | open |

**Totals:** <n> items, <n> P0, <n> without owner, <n> `no-trigger`.

*Review rule: an item with no owner, or a P0 marked `no-trigger`, blocks moving this document to final.*

## Lessons

*One or two sentences someone on another team could act on without reading the rest.*

## Appendix

- *Dashboards, queries, log excerpts, chat links.*
- *Customer communication sent, with timestamps.*
