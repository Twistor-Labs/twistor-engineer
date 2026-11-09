# Attack classes

Trimmed and adapted from Cloudflare's security-audit skill ([S1], MIT, notice in
`../LICENSE-cloudflare-security-audit`): ATTACK-CLASSES.md plus its web and auth, cloud and
deployment, AI and LLM, supply chain and release, and client-side companions. Pick at most 4
classes that match the stack. Each hunter reads only its own section.

Every class: name the lower-trust actor, the input, the control that should stop it, the sink
and what the actor gains. No boundary, no candidate.

## 1. Web and auth

Hunt:
- Injection: untrusted input reaching SQL, shell, HTML, templates, file paths, redirects or
  deserialization. Include field names, headers and stored values read back later.
- Access control: routes that check login but not ownership; a second path to the same state
  change with a weaker check; body fields that override the user or tenant id.
- Sessions and tokens: JWT signature and `alg` checks, audience and expiry, OAuth `state` and
  redirect binding, cookie flags, CSRF on state-changing routes, reset tokens.
- Secrets: hardcoded keys and passwords, secrets in logs, URLs, error messages or client bundles.

Verify: SQL is safe when values go through bind parameters (`?`, `$1`, `.bind()`), even if the
query string is built in pieces; it is unsafe when a request value is concatenated or
interpolated into the SQL text. A key is a finding only if non-test code loads it and it is not
a documented placeholder; whether it is live is a `needs_validation` fact.

## 2. Workers / serverless and cloud config

Hunt:
- Secrets in `wrangler.toml` / `wrangler.jsonc` `vars`, `serverless.yml`, Terraform or committed
  `.env` files instead of secret bindings.
- Bindings and roles broader than the code needs (a Worker with an account-wide token, an IAM
  role with `*`).
- Routes or functions reachable without auth that the code assumes are internal; trust in
  `X-Forwarded-*` or client-set headers.
- Outbound `fetch` to a URL the caller controls (SSRF), including redirects.
- Missing limits on costly calls (AI, email, third-party APIs) that let one caller spend the
  account's quota.

Verify: whether a route is public, whether a token is scoped, and dashboard-only settings are
outside the repo. If the decision depends on them, use `needs_validation` and name the setting.

## 3. LLM and agent tools

Hunt:
- Indirect prompt injection: fetched pages, email, files, tickets or tool output placed into a
  prompt, where model output then drives a tool call, query or outbound message.
- Excessive agency: destructive or outbound tools (delete, send, pay, push) with no human
  approval step and no allowlist.
- Tool argument injection: model output passed unchecked into SQL, shell, URLs or file paths.
- Insecure output handling: model output rendered as HTML or Markdown with links or images that
  can leak data.
- Secrets or other users' data in the prompt context.

Verify: trace from the untrusted source through the prompt to the tool call. It is confirmed when
nothing between model output and the side effect checks the action (approval, allowlist, fixed
target). Point to security-engineering #5 and #7.

## 4. Supply chain and CI

Hunt:
- `pull_request_target` or `workflow_run` workflows that check out PR head code and then run it
  (install, build, test, scripts) with secrets or a write token.
- Expression injection: `${{ github.event.* }}` (titles, branch names, bodies) inside `run:`.
- Broad `permissions:` or default write tokens; credentials persisted by checkout.
- Actions pinned to a branch or tag instead of a commit SHA.
- Dependencies pinned to versions with published advisories; install scripts from unpinned or
  typo-squat-prone names; lockfile missing or out of sync.

Verify: for workflows, confirm the trigger, the checked-out ref and which step runs that code with
which secrets. For dependencies, cite the advisory id (GHSA or CVE) and the fixed version; whether
the vulnerable function is reached is often a `needs_validation` fact. Point to
security-engineering #14 for patch speed.

## 5. Client side

Hunt:
- DOM XSS: untrusted data into `innerHTML`, `dangerouslySetInnerHTML`, `document.write`, `href`
  with `javascript:`, or eval-like sinks.
- `postMessage` handlers that do not check `event.origin`.
- Credentialed CORS that reflects any origin.
- Tokens or PII in `localStorage` readable by any script on the origin.
- Open redirects in client routing.

Verify: find the sanitizer or framework escaping on the path; a value React renders as text is
escaped, `dangerouslySetInnerHTML` is not. CSP and response headers set outside the repo are
`needs_validation` facts.
