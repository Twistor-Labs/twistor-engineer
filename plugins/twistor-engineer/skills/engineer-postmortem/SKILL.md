---
name: engineer-postmortem
description: 'Turn raw incident notes (chat logs, timelines, alerts) into a blameless postmortem with an ISO UTC timeline, root causes, the closest public outage precedent and owned action items that each carry a revisit trigger. Use when asked to "write the postmortem", "turn these notes into an incident review", "RCA for this outage" or "postmortem action items".'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.1.0
  summary: 'Incident notes in, blameless postmortem with owned, triggered action items out.'
---

# Engineer postmortem

Fill in `../reliability-incident-response/references/postmortem-template.md` (a sibling skill
folder). Read `../reliability-incident-response/references/case-studies.md` only to match this
incident to a known pattern.

1. **Gather.** If a path was given, read it. If the notes are thin, list what is missing (start and
   end time, detection source, customer impact, the triggering change) and ask once. Then proceed,
   marking unknowns as `unknown`.
2. **Timeline** in ISO 8601 UTC, one event per line, from first trigger to all-clear. Separate
   detection, mitigation and resolution times.
3. **Root causes**, plural. Name the trigger and the conditions that let it spread (no staged
   rollout, no validation of generated input, no kill switch, retries without jitter). Systems and
   processes, never people.
4. **Precedent.** Name the closest public incident from the case studies and the shared lesson,
   with its source link. If none fits, say so.
5. **Action items.** Each has an owner role, priority, the ceiling it raises (what failure it now
   prevents or contains) and a revisit trigger. Flag any item without a trigger `no-trigger`.
   Prefer items that prevent a class of failure over one-off fixes.
6. **Write** to `postmortems/<YYYY-MM-DD>-<slug>.md` in the current repo unless the user named
   another place, then print the path and the action-item table.

Tone: plain, factual, blameless.
