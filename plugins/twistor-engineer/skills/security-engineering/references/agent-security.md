# Agent threat model

Use this to fill sections 1 to 4 of the answer shape. Assume any content the agent reads can carry instructions, and that the model will sometimes follow them: injection success that is roughly 0.1% on one try reaches around 5-6% after 100 adaptive attempts [S2]. Design so that a fully compromised agent still cannot do much.

## Three layers to defend [S2]
| Layer | What it gives you | What it cannot do |
|---|---|---|
| Environment (sandbox, proxy, identity) | Hard limits that hold even if the model is fooled | Judge intent |
| Model (training, system prompt, classifiers) | Shapes tendencies, catches obvious misuse | Guarantee anything under adaptive attack |
| External content | Where attacks arrive | Be trusted |

## Injection paths (how hostile instructions get in)
- Web pages and search results the agent fetches.
- Files in the repo or workspace: READMEs, issues, comments, test fixtures, dependency docs.
- Tool and MCP responses, including third-party and auto-generated tools [S6].
- Tickets, chat messages, emails and documents the agent is asked to process.
- Installed skills and plugins: audit bundled code and any instruction that reaches external networks [S20].
- Internally generated config and data files are inputs too: validate them like user input [S12].

Control: keep these out of anything that decides whether an action is allowed. Auto mode's classifier sees only user messages and tool calls; assistant text and tool results are stripped [S3].

## Exfiltration paths (how data gets out)
- Allowlisted domains used with an attacker's own credentials (the Cowork api.anthropic.com case) [S2].
- Free-text tool arguments: messages, issue comments, emails, webhook bodies.
- Git push to a branch or remote the attacker can read [S1].
- Secrets readable by generated code: env vars, config files, SSH keys in the home directory [S1][S4].
- Model context itself, when raw PII or bulk data is routed through it [S5].
- Logs and traces that capture prompt content, if shipped to a broad audience [S16].

## Controls by path
| Threat | Control | Rule |
|---|---|---|
| Agent reads or writes outside its workspace | Filesystem sandbox | 1, 2 |
| Agent sends data to arbitrary hosts | Default-deny egress proxy the agent cannot change | 1, 2 |
| Data sent to an allowed API under attacker's key | Proxy passes only the sandbox's own token | 4 |
| Generated code prints a secret | Credentials injected outside the sandbox | 3 |
| Agent pushes to the wrong place | Proxy restricts pushes to the configured branch | 3 |
| Over-broad tool access | Gateway with tool-level authz, approval for new tools | 8 |
| Stolen or misused agent access | Single-purpose identity, split read/write creds, one-step revoke | 9 |
| PII leaks through context | Filter and tokenize in the sandbox | 10 |
| Injected instruction triggers a risky action | Tiered approval, reasoning-blind classifier, hard deny on destructive verbs | 6, 7 |
| One guardrail bypassed | Layered guardrails: relevance, safety, PII, moderation, tool safeguards, rules, output validation [S7] | 5 |

## Matching isolation to oversight [S2]
- Little or no human watching: ephemeral container per session (claude.ai uses gVisor).
- Human in the loop on a dev machine: OS sandbox plus approvals for what leaves it (Claude Code).
- Desktop agent touching local apps: a full local VM with its own proxy (Cowork).

## Approval design
- Prompts alone fail: users approve roughly 93% [S2][S3].
- Tier by blast radius: routine work auto-approved, destructive commands hard-denied, prod infra and outbound messaging to a human [S17].
- Deny-and-continue with a reason; escalate after 3 consecutive or 20 total denials [S3].
- Expect misses: the auto-mode classifier still missed 17% of real overeager actions after stage 2 [S3]. The sandbox is the backstop.

## Quick template
```
Agent: <name>          Oversight: <none | async | live>
Can read:   <paths, data classes>
Can write:  <paths, systems>
Egress:     <domains, each with "attacker-usable? y/n">
Secrets:    <where they live; must be outside sandbox>
Identity:   <service identity, read/write split, revoke tested?>
Tools:      <via gateway? dangerous verbs removed?>
Injection paths: <list>
Exfil paths:     <list>
Human-required actions: <list>
Evidence:   <test or log per control>
```
