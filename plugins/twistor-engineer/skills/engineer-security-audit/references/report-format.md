# Report format

engineer-review's one-line grammar plus a verdict. Confirmed findings first (bug, then risk,
then nit), then needs_validation, then skipped scanners, then totals. Nothing else.

```
path:line: confirmed: <bug|risk|nit>: <attack class>: <problem>. <fix>. (security-engineering #N)
path:line: needs_validation: <attack class>: <problem>. Open fact: <the one fact>. <how to check>.
scanners not run: <tool> (`<command>`), <tool> (`<command>`)
totals: N confirmed (N bug, N risk, N nit), N needs_validation, N rejected
```

Fields:

- `path:line`: repo-relative, the line of the sink or the offending value. For a workflow, the
  step that runs untrusted code. For a dependency, the line in the manifest.
- Verdict: `confirmed` (full source trace, concrete impact) or `needs_validation` (trace holds,
  one outside fact decides it). Rejected candidates appear only in the totals count.
- Severity: confirmed only. `needs_validation` lines never carry one.
- Attack class: one of `web and auth`, `workers and cloud`, `llm and agent tools`,
  `supply chain and ci`, `client side`.
- Problem: what the lower-trust actor can do, in one sentence.
- Fix: the smallest source change that closes it. Describe it; do not apply it.
- Rule pointer: `(security-engineering #N)` only when that rule applies.
- Secrets: never print any part of the value; name the variable or key, file and line.

Example:

```
src/db.ts:14: confirmed: bug: web and auth: `q` from the query string is concatenated into a D1 SELECT, so any caller can read or alter other rows. Use `prepare("... WHERE name = ?").bind(q)`.
.github/workflows/preview.yml:18: confirmed: bug: supply chain and ci: pull_request_target checks out the PR head and runs `npm ci` with DEPLOY_TOKEN, so a fork PR runs its own code with the secret. Use pull_request, or never execute head code in the privileged job. (security-engineering #3)
src/config.ts:3: needs_validation: web and auth: API key `twx_li...` is hardcoded in source. Open fact: whether the key is live. Revoke it at the provider and move it to a secret binding either way.
scanners not run: gitleaks (`gitleaks git -v .`), osv-scanner (`osv-scanner scan source -r .`)
totals: 2 confirmed (2 bug, 0 risk, 0 nit), 1 needs_validation, 3 rejected
```

Nothing survived:

```
No issues.
totals: 0 confirmed (0 bug, 0 risk, 0 nit), 0 needs_validation, 4 rejected
```

Long report (over about 60 lines): write the same lines to
`~/security-audit/<repo-name>/report.md`, outside the target, and print the path plus the totals
line in chat.
