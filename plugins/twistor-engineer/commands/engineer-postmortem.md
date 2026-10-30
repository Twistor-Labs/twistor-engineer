---
description: Turn raw incident notes (chat logs, timelines, alerts) into a blameless postmortem with owned, triggered action items.
argument-hint: "[path to notes]  or paste notes after the command"
allowed-tools: Read, Grep, Glob, Write
---

Incident notes: $ARGUMENTS

Use the `reliability-incident-response` skill. Read
`${CLAUDE_PLUGIN_ROOT}/skills/reliability-incident-response/references/postmortem-template.md`
and fill it in. Read `references/case-studies.md` only to match this incident to a known pattern.

1. Gather. If a path was given, read it. If the notes are thin, list what is missing (start and
   end time, detection source, customer impact, the change that triggered it) and ask once.
   Then proceed with what you have, marking unknowns as `unknown`.
2. Timeline in ISO 8601 UTC, one event per line, from first trigger to all-clear. Separate
   detection, mitigation and resolution times.
3. Root causes, plural. Name the trigger and the conditions that let it spread (no staged rollout,
   no validation of generated input, no kill switch, retries without jitter). Systems and
   processes, never people.
4. Match to the outage library: name the closest public incident and the shared lesson, with its
   source link. If none fits, say so.
5. Action items: each has an owner role, priority, the ceiling it raises (what failure it now
   prevents or contains), and a revisit trigger. Any item without a trigger gets flagged
   `no-trigger`. Prefer items that prevent a class of failure over one-off fixes.
6. Write the postmortem to `postmortems/<YYYY-MM-DD>-<slug>.md` in the current repo unless the
   user named another place, then print the path and the action-item table.

Tone: plain, factual, blameless. No em-dashes.
