# Pre-ship checklist: agents and AI apps

Tick each item or write why it does not apply. Numbers point to SKILL.md rules.

## Environment
- [ ] Agent runs in an OS-level sandbox; writes outside the workspace fail. (1, 2)
- [ ] Network is default-deny through a proxy the agent cannot reconfigure. (1, 2)
- [ ] Every allowlisted domain is reviewed as a capability: could an attacker use their own account or key on it? (4)
- [ ] Outbound calls to allowlisted APIs are bound to the sandbox's own token. (4)
- [ ] Isolation level matches how closely a human watches the session. (1)

## Credentials and identity
- [ ] No API key, git token or signing key exists in the sandbox image, env or filesystem. (3)
- [ ] Credentials are injected by a proxy or vault outside the sandbox, with scope limits (for example, push only to one branch). (3)
- [ ] The agent has its own service identity, not a human's; read and write credentials are separate. (9)
- [ ] One revoke action removes the agent's access everywhere; tested. (9)

## Tools and MCP
- [ ] MCP servers are reached through a gateway with tool-level authz. (8)
- [ ] Auto-generated tools stay disabled until an owner approves. (8)
- [ ] Dangerous verbs (delete, drop, force push, send to external) are removed or gated. (9)
- [ ] Skills and plugins come only from trusted sources; bundled code and network instructions audited. (see Red flags)

## Untrusted input and data
- [ ] Every injection path (web, files, tool output, MCP responses, tickets, email) is listed in the threat model. (5)
- [ ] Guardrails are layered: deterministic rules, PII filter, tool safeguards, output validation. (5)
- [ ] Large data and PII are filtered or tokenized inside the sandbox before reaching the model. (10)
- [ ] Data purpose is enforced by flow labels, not ad hoc call-site checks, where user data crosses systems. (12)

## Approval
- [ ] Action tiers written down: auto-approve, hard-deny, human-required (prod infra, outbound messaging). (7)
- [ ] Any action-safety classifier sees only user messages and tool calls, not tool results or agent reasoning. (6)
- [ ] Repeated denials escalate to a human; headless runs stop instead. (6, 7)
- [ ] Denials and tool calls are logged to the SIEM and reviewed. (7)

## Config and pipelines
- [ ] Internally generated config and feature files are validated for schema and size before load. (11)
- [ ] Config changes roll out in stages with a kill switch; no global instant push. (11)
- [ ] Hot paths fail closed or fall back, not panic, on bad config. (11)

## Findings and patching
- [ ] THREAT_MODEL.md exists and is fed to review agents. (13)
- [ ] Each finding has a working PoC and was re-checked by a fresh-context verifier. (13)
- [ ] Internet-facing exploited CVEs patched within 24 hours; queue ordered by KEV then EPSS. (14)
- [ ] TLS edge has post-quantum key agreement enabled where supported; auth migration on the roadmap. (15)
