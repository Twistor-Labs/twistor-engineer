---
name: engineer-security-audit
description: 'Audit a whole repo or folder for security bugs before ship: map trust boundaries, run free scanners that are already installed, hunt by attack class, send every candidate to a fresh verifier that tries to disprove it, and report confirmed and needs_validation findings in engineer-review line format. Use when asked to "audit this repo for security", "security audit", "is this safe to ship", "find vulnerabilities in", "scan for secrets", "triage this scanner output", "scan for vulnerable deps", "check my GitHub workflows", or "should I install this skill / MCP server". Read-only; never edits source and never installs tools. Not for one diff or PR (use engineer-review), a design doc (use engineer-design-review), sandboxing or containing an agent (use security-engineering), or OWASP basics and input validation (use agent-skills security-and-hardening).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.2.0
  summary: 'Recon, scanners, capped hunters, one fresh verifier per candidate, verdict-tagged findings.'
---

# Engineer security audit

Find vulnerabilities that cross a real trust boundary, prove each one from source, and report.
The method is adapted from Cloudflare's MIT security-audit skill (see
`LICENSE-cloudflare-security-audit` and `references/sources.md` [S1][S2]). Report; never edit.

Hard limits for the whole run:

- Never edit, create or delete files in the target. Output goes to chat. If the report is longer
  than about 60 lines, write it to one folder outside the target, `~/security-audit/<repo-name>/`,
  and say so.
- Never install anything. Never run the target's build, tests or install scripts.
- Never send traffic to a URL unless the user has said, in this conversation, that they own the
  target or are authorized to test it.
- Use dummy values for any check. Never copy any part of a secret value into the report; name the
  variable or key, the file and the line only.

## 1. Recon

Read, do not run. In one short paragraph to yourself, record:

- **Scope**: the path given, else the repo root. Skip `node_modules/`, `vendor/`, build output.
- **Stack**: languages, frameworks, runtime (Workers, Lambda, containers, browser, mobile), data
  stores, LLM or agent code, CI files (`.github/workflows/`, other CI configs).
- **Trust boundaries and inputs**: every place untrusted data enters (HTTP routes, webhooks,
  queue consumers, fetched URLs, files, model output, PR content in CI) and every sink it can
  reach (SQL, shell, HTML, file paths, outbound fetch, tool calls, secrets).
- **Prior evidence**: an existing `THREAT_MODEL.md`, `SECURITY.md` or earlier audit. Read it
  first; hunt what it does not cover, and do not re-report what it accepts as a known risk.

If the target is a third-party skill or MCP server the user wants to install, scope is that
package: hunt instructions, scripts, tool descriptions and requested permissions.

## 2. Scanners

Pick the rows from `references/scanners.md` that fit the stack (at most one tool per situation).
For each:

1. Check it is installed: `command -v <tool>`.
2. Installed and passive: run the exact command from the table, read-only, from the repo root.
   Treat every hit as a candidate for step 4, not as a finding.
3. Not installed: print the exact command under a `scanners not run:` line and continue. Do not
   install it, and do not suggest `curl | bash`.
4. Active tools (Nuclei, Strix) on any target, local path or URL: ask "Do you own or have written
   authorization to test <target>?" and run nothing until the user says yes. Strix also spends the
   user's LLM quota; say so in the same question. No answer means no scan.
5. TruffleHog verification sends each found key to its provider. Run it with `--no-verification`
   unless the user says yes to "Send found keys to their providers to check if they are live?".
6. Flag paid keys when you name a tool: Strix needs an LLM API key, Claude Code Security Review
   needs an Anthropic API key, SkillSpector's LLM stage needs one (use `--no-llm`).

## 3. Hunt

Choose at most 4 attack classes from `references/attack-classes.md` that fit what recon found:
web and auth, Workers / serverless and cloud config, LLM and agent tools, supply chain and CI,
client side. Name them in one line.

Run one hunter per class, each isolated: it gets the recon notes, its class section and the
scanner hits for its area, and nothing from other hunters. A hunter:

- Traces each untrusted input to a sink, and states the boundary before filing: who is the
  lower-trust actor, what control should stop them, what they gain.
- Files a candidate as `path:line`, class, one-line claim, source trace (entry -> steps -> sink).
- Does not file missing best practice with no reachable path. That is a hardening note, at most
  a `nit`, and only if the user asked for hardening.

## 4. Verify

Merge duplicates (same root cause, same sink). Then give each unique candidate to one fresh
verifier that did not hunt it. The verifier's job is to disprove it:

1. Re-read every cited line in current source. A line that does not say what the claim says
   rejects the candidate.
2. Look for the control that stops it: parameter binding, auth middleware, an allowlist, an
   approval step, a test-only path, a value that is a documented placeholder, an env binding that
   replaces the literal at deploy.
3. Decide:
   - `confirmed`: the full trace from input to sink holds in source and the impact is concrete.
     Give severity.
   - `needs_validation`: the source trace holds but one fact outside the repo decides it (a
     dashboard setting, a deployed env var, whether a key is live, whether a branch is
     protected). Name that one fact. No severity.
   - `rejected`: disproved. Count it; do not list it.

Never promote a candidate to `confirmed` on a scanner hit alone. A secret in a test fixture, a
`.example` file or docs is rejected unless something outside tests loads it. A dependency pinned
inside a published advisory's affected range is never rejected on reasoning about library code
you cannot see; if reachability is unproven, it is `needs_validation` with reachability as the
open fact. One root cause that reaches two sinks is two lines.

## Delegation

In Claude Code, hunters and verifiers can be subagents (the Agent or Task tool): one per class,
then one per candidate, each started fresh with only the inputs listed above plus the "Hard limits" list from the top of
this file, copied verbatim into every subagent prompt. Elsewhere, run the
phases in order in one session and re-read source at step 4 as if you had not seen the hunt.

## Severity (confirmed only)

- `bug`: exploitable now by the lower-trust actor (injection reaching a sink, live secret in
  source, untrusted code running with secrets, unapproved destructive tool call).
- `risk`: exploitable under a plausible condition or with limited blast radius (vulnerable
  dependency on a reachable path, overbroad token, no in-code limit on an expensive call when
  nothing in the repo shows an outside limit).
- `nit`: low impact, worth fixing with other work.

## Output

Follow `references/report-format.md`. One line per finding, confirmed first (bug, risk, nit),
then needs_validation, then the scanners line if any were skipped, then totals. No headings, no
praise, no summary of the repo.

```
path:line: confirmed: <bug|risk|nit>: <attack class>: <problem>. <fix>. (security-engineering #N)
path:line: needs_validation: <attack class>: <problem>. Open fact: <the one fact>. <how to check>.
scanners not run: <tool> (`<command>`), ...
totals: N confirmed (N bug, N risk, N nit), N needs_validation, N rejected
```

Add `(security-engineering #N)` only when a rule there applies, for example #3 (credentials
outside the sandbox), #5 (layered injection defense), #7 (approval for destructive or outbound
actions), #9 (scoped service identity), #13 (verify findings), #14 (patch on public exploit).

If nothing survives step 4, print `No issues.` and then the totals line.
