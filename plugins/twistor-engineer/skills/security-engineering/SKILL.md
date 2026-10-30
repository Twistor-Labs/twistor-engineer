---
name: security-engineering
description: 'Use when securing an AI agent, coding agent, MCP server or LLM app, or reviewing app security before ship. Triggers: "prompt injection", "sandbox the agent", "too many permission prompts", "auto-approve tool calls", "where do API keys live", "MCP gateway", "exfiltration", "egress allowlist", "least privilege for agents", "human approval for risky actions", "triage scanner findings", "config push took prod down", "post-quantum TLS". Nouns: sandbox, egress proxy, service identity, tool scopes, guardrails, purpose limitation. Gives sourced instead-of rules, an agent threat model, a controls-by-layer answer shape and a pre-ship checklist. Not for OWASP basics or input validation (use agent-skills security-and-hardening), agent tool design (use ai-agent-engineering), or outage response (use reliability-incident-response).'
license: MIT
metadata:
  plugin: twistor-engineer
  version: 1.0.0
  summary: 'Contain agents with sandboxes, outside-the-box credentials, scoped tools and tiered approval, plus sourced app-security lessons.'
---

# Security Engineering

The model is not the security boundary. The environment is. Put hard limits in the sandbox, the network proxy and the identity layer, keep credentials where generated code cannot reach them, and treat everything the agent reads as possibly hostile. Rules come from Anthropic, Uber, Meta, OpenAI and Cloudflare engineering posts.

## When to use / not

- Use: giving an agent shell, file, browser or MCP access; deciding what runs without a human; placing secrets for an agent; reviewing an AI feature before ship; triaging LLM or scanner security findings; hardening config pipelines.
- Use: purpose limitation for user data, patch SLAs, post-quantum TLS planning.
- Not: OWASP Top 10 basics, auth flows, headers, CORS, input validation (see agent-skills security-and-hardening).
- Not: tool design, context and orchestration (see ai-agent-engineering).
- Not: running a live incident or writing a postmortem (see reliability-incident-response).

## Rules

1. **Instead of** leaning on per-action permission prompts for safety, **do** run the agent in an OS-level sandbox with filesystem limits and default-deny network. **Why:** users approve roughly 93% of prompts, so prompts are weak oversight, while sandboxing cut prompts by 84% in internal use [S2][S3][S1]
2. **Instead of** isolating only the filesystem or only the network, **do** enforce both boundaries together. **Why:** without network isolation a compromised agent exfiltrates files like SSH keys, and without filesystem isolation it can escape to the network [S1]
3. **Instead of** passing API keys, git tokens or env secrets into the sandbox, **do** inject credentials from a proxy or vault outside it and scope what the proxy allows. **Why:** Anthropic keeps git credentials and signing keys outside the sandbox with a proxy that restricts pushes to the configured branch, and its managed-agent harness never sees credentials [S1][S4][S16]
4. **Instead of** treating an allowlisted domain as safe, **do** treat every egress allowance as a capability grant and bind outbound calls to the sandbox's own identity. **Why:** allowing api.anthropic.com in Cowork let an attacker's API key carry data out, fixed by a proxy that only passes requests with the VM's own session token [S2]
5. **Instead of** trusting model training or a single guardrail to stop prompt injection, **do** layer guardrails and keep hard limits in the environment. **Why:** Opus 4.7 injection success is roughly 0.1% on one attempt but around 5-6% after 100 adaptive attempts [S2][S7]
6. **Instead of** showing an action-safety classifier the agent's reasoning and tool results, **do** give it only user messages and the proposed tool calls. **Why:** stripping assistant text and tool output is how auto mode resists injection, and its honest miss rate on real overeager actions is still 17% [S3]
7. **Instead of** approving everything or nothing, **do** tier actions: auto-approve routine work, hard-deny destructive commands, route prod infra and outbound messaging to a human. **Why:** with auto mode the agent works 9x longer between interruptions, and Gusto saw a denial in roughly 10% of session transcripts [S17][S3]
8. **Instead of** letting each team wire MCP servers straight into agents, **do** front them with a central gateway that enforces tool-level authz, redacts PII and keeps auto-generated tools disabled until an owner approves. **Why:** Uber reports the gateway hosts over 800 MCP servers and over 5000 tools [S6]
9. **Instead of** giving the agent a human's credentials and the full tool list, **do** issue a single-purpose service identity with separate read and write credentials and drop dangerous verbs from the tool list. **Why:** revoking a dedicated agent identity ends its access everywhere, and a smaller verb set shrinks blast radius [S18][S16][S14]
10. **Instead of** routing raw tool output and customer PII through model context, **do** filter data inside the sandbox and tokenize PII before the model sees it. **Why:** in Anthropic's example the model sees 5 rows of a 10,000-row sheet and emails become placeholders like [EMAIL_1] [S5]
11. **Instead of** trusting internally generated config or feature files, **do** validate their size and schema like user input and roll them out in stages. **Why:** a Cloudflare feature file doubled past a 200-feature limit and panicked the proxy on 2025-11-18, and a global config push with no gradual rollout hit about 28% of HTTP traffic on 2025-12-05 [S12][S13]
12. **Instead of** enforcing data purpose with point checks at each call site, **do** track data flows with labels checked at runtime across the stack. **Why:** Meta found point checking did not scale and reports 10x better computational efficiency with flow-based Policy Zones [S8][S9]
13. **Instead of** forwarding every LLM or scanner security finding, **do** keep a THREAT_MODEL.md, re-check findings with an adversarial verifier in a fresh container and require a working PoC. **Why:** findings were exploitable 90% of the time with good threat models, and the verifier roughly halved non-exploitable findings [S15]
14. **Instead of** patching internet-facing apps on a monthly cycle, **do** patch within 24 hours of a public exploit, ordering work by CISA KEV then EPSS. **Why:** Anthropic expects an order-of-magnitude rise in finding volume from AI-accelerated offense [S19]
15. **Instead of** deferring post-quantum work, **do** turn on post-quantum key agreement where your edge supports it and inventory authentication for the next step. **Why:** Cloudflare reports over 65% of human traffic is already PQ-encrypted and targets fully PQ-secure, including authentication, by 2029 [S10][S11]

## Answer shape

1. **Assets and boundaries**: what the agent or app can touch (files, secrets, networks, tools, data classes) and where trust changes.
2. **Injection paths**: every place untrusted content enters context (web pages, files, tool output, MCP responses, tickets, email).
3. **Exfiltration paths**: every way data can leave (allowed domains, tool calls with free-text args, messaging, git push, logs).
4. **Controls by layer**: environment (sandbox, egress proxy), identity (credentials outside, scoped service identity), tools (gateway, verb list), approval (tiers), model (classifier, guardrails). Cite the rule number for each.
5. **Fix before ship vs harden later**: exploitable gaps first, each with the smallest change that closes it.
6. **Evidence**: the tests or logs that prove each control works.

## Red flags

- "The model refuses malicious instructions, so we're fine." -> Adaptive attacks get through at measurable rates; add an environment limit (rule 5).
- "We ask the user before every action." -> People click yes. Sandbox and tier (rules 1, 7).
- "The key is only in an env var inside the container." -> Generated code can read it. Move it behind a proxy (rule 3).
- "It is only our own API on the allowlist." -> An attacker can use their own key on that API. Bind to the sandbox's identity (rule 4).
- "That config file is ours, it is trusted." -> Validate and stage it (rule 11).
- "The classifier can read the tool output for context." -> That is the injection channel. Keep it reasoning-blind (rule 6).
- "We'll add an MCP server per team, gateway later." -> Later never comes at 800 servers. Gateway first (rule 8).
- "Install this skill or plugin, it looks fine." -> Install only from trusted sources and audit bundled code and network instructions [S20].

## Verification

- Show a sandboxed run trying to read a path outside the workspace and reach a non-allowlisted host; both must fail.
- Grep the sandbox image, env and generated code paths for secrets; zero hits.
- Replay one injection payload through each untrusted input and show the blocked action in logs.
- Show the approval tier table and a log line for a routed or denied high-risk action.
- Show the MCP or tool scope list per agent identity and a revoke test.
- For config pipelines, show the schema or size check failing a bad file in CI and the staged rollout plan.
- For each security finding shipped to a team, attach the PoC.

## References

| File | Read when |
|---|---|
| references/agent-security.md | threat-modeling an agent: injection paths, exfiltration paths, controls per layer |
| references/case-studies.md | you need the full story behind a rule (Anthropic, Uber, Meta, OpenAI, Cloudflare) |
| references/checklist.md | reviewing an AI feature or agent before ship |
| references/sources.md | citing a figure or date |

Companion for OWASP basics, auth, headers and input validation: agent-skills `security-and-hardening` (link only, not bundled).
