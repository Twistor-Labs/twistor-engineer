# Case studies

Each entry: context, what they did, numbers, lesson. Figures are the company's own reports.

## Anthropic: sandboxing Claude Code (rules 1, 2, 3)
- Context: permission prompts on every shell command slowed work and still let risky actions through.
- Did: OS-level sandbox with filesystem isolation (read/write in the working directory, no writes outside) plus network isolation through an out-of-sandbox proxy with a domain allowlist and confirmation for new domains. The runtime is open source.
- Numbers: sandboxing "safely reduces permission prompts by 84%" in internal usage.
- Web version keeps git credentials and signing keys outside the sandbox; a proxy holds tokens and restricts pushes to the configured branch.
- Lesson: one boundary is not enough. Filesystem without network leaks keys; network without filesystem lets the agent escape. [S1]

## Anthropic: containing Claude across products (rules 1, 4, 5)
- Context: three products with different oversight: claude.ai, Claude Code, Cowork.
- Did: defend three layers (environment with hard limits, model shaping tendencies only, external content). Match isolation to how much the user watches: ephemeral gVisor containers on claude.ai, human-in-the-loop sandbox in Claude Code, a local VM in Cowork.
- Incident: in Cowork, allowlisting api.anthropic.com let an attacker's API key move data out. Fix: a VM-internal proxy that passes only requests carrying the VM's own session token.
- Numbers: users approved "roughly 93% of permission prompts"; Opus 4.7 prompt-injection success "roughly 0.1% on single attempts" rising to "around 5-6% after 100 adaptive attempts".
- Lesson: an allowed domain is a capability, and the model layer is a tendency, not a wall. [S2]

## Anthropic: auto mode classifier (rules 6, 7)
- Context: approval fatigue (93% of prompts accepted) made manual prompts weak oversight.
- Did: two-stage Sonnet 4.6 classifier. Stage 1 is fast and told to "err on the side of blocking"; stage 2 reasons only on flagged calls. Input is reasoning-blind: user messages and tool calls only, assistant text and tool results stripped.
- Numbers: real traffic (n=10,000) FPR 8.5% at stage 1, 0.4% after stage 2. Real overeager actions (n=52) FNR 6.6% at stage 1, 17% after stage 2 ("the honest number"). Synthetic exfiltration (n=1,000) FNR 1.8% then 5.7%.
- Deny-and-continue: blocked action returns a reason; escalate to a human after 3 consecutive or 20 total denials; headless runs terminate instead.
- Lesson: a classifier reduces risk but misses some; keep the sandbox under it. [S3]

## Anthropic: auto mode in production (rule 7)
- Did: hard-deny destructive commands, auto-approve "the routine 90 percent of the time", route prod-infra and messaging actions to a human, log denials.
- Numbers: Claude "works 9x longer between interruptions"; Gusto found a denial in "roughly 10% of session transcripts".
- Lesson: tier approvals by blast radius and read the denial log. [S17]

## Anthropic: managed agents, credentials out of reach (rule 3)
- Did: split harness (brain), sandboxes and tools (hands) and an append-only session log. Sandboxes are disposable.
- Credentials never reach generated code: vault or resource-bundled auth, a proxy for MCP calls, git token wired into the remote at clone time. The harness never sees credentials.
- Lesson: if code the model writes can print a secret, the secret is already exposed. [S4]

## Anthropic: code execution with MCP (rule 10)
- Did: present MCP servers as code files, filter data inside the sandbox, keep intermediate results out of the model, tokenize PII automatically (`[EMAIL_1]`).
- Numbers: Google Drive to Salesforce example 150,000 -> 2,000 tokens (98.7%); a 10,000-row sheet where the model sees 5 rows.
- Cost: needs sandboxing, resource limits and monitoring.
- Lesson: data the model never sees cannot be leaked by an injected instruction. [S5]

## Anthropic: CISO guide and agent identity (rules 3, 9)
- Did: egress through an allowlist proxy the agent cannot change, called the "strongest control against prompt injection"; credentials injected by reverse proxy, not stored in the sandbox; dangerous verbs removed from tool lists; tool calls streamed to a SIEM via OpenTelemetry.
- Agent identity: the agent gets its own service identity ("revoking the identity ends Claude's access everywhere"), a workspace baseline narrowed per channel, and separate read and write credentials per permission level.
- Anthropic reports agents author "more than 50% of all code submitted for pull requests at Anthropic".
- Lesson: least privilege for agents means a distinct identity you can revoke in one step. [S16][S18]

## Anthropic: securing the AI-native SDLC (rules 9, 13)
- Did: several narrow review agents that must prove findings; new bug classes go into CLAUDE.md and org skills; agents run on remote VMs with egress allowlists and single-purpose identities; new reviewers run in shadow mode and auto-approvals are sampled.
- Numbers: PRs with substantive comments grew "from 16 to 54%"; the process would have caught "approximately a third of the bugs behind past claude.ai incidents".
- Lesson: treat review agents like any new control: shadow, sample, then trust. [S14]

## Anthropic: using LLMs to secure source code (rule 13)
- Did: keep a THREAT_MODEL.md; re-check findings with an adversarial verifier in a fresh container; deterministic dedupe (same file and category, within ten lines) then model merge; patch gate cheapest-first (build and new tests, PoC no longer works, suite passes, fresh agent re-attacks).
- Numbers: with good threat models findings "were exploitable 90 percent of the time"; the verifier "roughly halved the rate of non-exploitable findings"; requiring a working PoC brought false positives near zero.
- Lesson: no PoC, no ticket. [S15]

## Anthropic: preparing for AI-accelerated offense (rule 14)
- Did: patch internet-facing apps "within 24 hours of an exploit becoming available", CISA KEV first then EPSS; pilot AI triage on one noisy rule for two weeks against a human.
- Expectation: an "order-of-magnitude increase in finding volume".
- Lesson: patch SLAs built for human-speed attackers are too slow. [S19]

## Uber: MCP Gateway (rule 8)
- Context: MCP servers and tools spreading across teams.
- Did: one gateway and management platform with tool-level authorization, PII redaction, and auto-generated tools disabled until the owner approves.
- Numbers: Uber reports "hosting over 800 MCP servers and over 5000 tools".
- Lesson: central control of tool access is cheaper to build before the sprawl than after. [S6]

## OpenAI: layered guardrails (rule 5)
- Did: "Think of guardrails as a layered defense mechanism." Types: relevance classifier, safety classifier, PII filter, moderation, tool safeguards, rules-based protections, output validation.
- Lesson: no single guardrail is enough; stack cheap deterministic checks with model-based ones. [S7]

## Meta: Privacy Aware Infrastructure and Policy Zones (rule 12)
- Context: purpose limitation for user data across a huge codebase and warehouse.
- Did: information flow control that checks data flows "in real-time during code execution", integrated with HHVM, Presto and Spark; the 2025 follow-up covers exabyte-scale batch processing.
- Numbers: Meta reports "10x improvements in computational efficiency"; "point checking controls did not scale".
- Lesson: label the data and check the flow, not every call site. [S8][S9]

## Cloudflare: generated config as untrusted input (rule 11)
- 2025-11-18: a ClickHouse access-control change made a metadata query return duplicate rows, the Bot Management feature file doubled in size, broke a preallocated limit of 200 features (about 60 in use), and Rust code panicked on `unwrap()`. First customer errors 11:28 UTC, core traffic largely restored 14:30, all resolved 17:06 UTC. Older FL proxy customers got bot score 0 instead of errors, causing false positives.
- 2025-12-05: raising a WAF body buffer for a React Server Components CVE, then disabling an internal test tool through the global config system with no gradual rollout, caused about 25 minutes of HTTP 500s on about 28% of HTTP traffic. Not an attack.
- Lesson: validate internally produced config like user input, avoid panics on hot paths, roll out in stages. [S12][S13]

## Cloudflare: post-quantum rollout (rule 15)
- Trend: "well over a third" of human traffic PQ-encrypted in March 2025, "over 50% (and rising!)" in October 2025, "over 65% of human traffic" by April 2026.
- Target: "2029 to be fully post-quantum (PQ) secure", including PQ authentication.
- Lesson: PQ key agreement is mainstream now; authentication is the next migration to plan. [S10][S11]

## Anthropic: confidential inference (background for rule 3)
- Did: data stays encrypted except where processed; only a small trusted loader on an isolated VM sees plaintext; keys released only to TPM-attested environments; only programs signed by the reviewed CI server run on the accelerator.
- Lesson: shrink the set of code that ever sees plaintext secrets or data. [S21]
