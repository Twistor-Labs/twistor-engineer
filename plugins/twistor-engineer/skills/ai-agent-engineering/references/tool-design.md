# Tool design

Tools are the agent's interface to the world. Design them for a model reading them cold, not for a human reading docs. Rules 3 to 6 and 13 in SKILL.md.

## Interface (rule 3)

- Treat the agent-computer interface like an HCI: clear descriptions, examples, edge cases, and parameters that make mistakes hard (poka-yoke). [S1]
- Require absolute file paths. In Anthropic's SWE-bench work, relative paths broke after the agent left the root directory; with absolute paths the model used the tool "flawlessly". [S1]
- The SWE-bench scaffold shows how little you need: two tools (Bash, Edit) and a simple prompt. The Edit tool's `str_replace` applies only when `old_str` matches exactly once, which forces a precise target. [S2]
- Namespace by service and resource (`asana_search`, `jira_search`). Prefix vs suffix naming measurably changed eval results, so test the choice instead of guessing. [S5]
- Add 1-5 usage examples per tool where the schema is ambiguous. Tool Use Examples raised complex parameter accuracy from 72% to 90%. [S10]
- Every tool costs context budget; each should be a primary action, not a thin wrapper per endpoint. [S7]
- Put long domain guidance in the system prompt, not inside a tool description. [S3]

## Output (rule 4)

- Cap and paginate. Claude Code restricts tool responses to 25,000 tokens by default (a configurable default). [S5]
- Offer a `response_format` enum. In the Slack example, detailed was 206 tokens and concise 72, about a third. This is one illustrative example, not an average. [S5]
- Return meaningful identifiers and fields the next step needs; filter the rest before it reaches the model. [S5][S9]
- Make test and tool output agent-friendly: a few lines to stdout, details in a log file, "ERROR" and reason on one line so it can be grepped, and a fast mode that samples 1% or 10% of the suite. [S15]

## Loading (rule 5)

- Mark rarely used tools `defer_loading: true` and keep the 3-5 most used loaded. Turn on tool search when definitions exceed 10K tokens or you have 10+ tools. [S10]
- Tool search cut upfront definitions from about 77K to about 8.7K tokens (85%). MCP evals: Opus 4 49% to 74%, Opus 4.5 79.5% to 88.1%. [S10]
- Tool choice degrades past "30-50 tool definitions"; group by intent and load the rest via search. [S21]
- Agents with "often 20+" tools struggle. [S23]

## Code instead of calls (rule 6)

- Present MCP servers as code files the agent loads on demand. A Google Drive to Salesforce task dropped from 150,000 to 2,000 tokens (98.7%). [S9]
- Filter in the sandbox: from a 10,000-row sheet the model sees 5 rows. A 2-hour meeting transcript would add about 50,000 tokens if routed through context. [S9]
- PII can stay out of context by tokenizing it (`[EMAIL_1]`) before the model sees results. [S9]
- Programmatic Tool Calling cut average tokens 37% on complex research tasks (43,588 to 27,297). [S10]
- Letting Claude filter tool output via code execution lifted Opus 4.6 on BrowseComp from 45.3% to 61.6%. [S26]
- Uber reports code mode cut tokens 55-71% on small SQL tasks. [S20]
- Cost of this pattern: you now need a sandbox, resource limits and monitoring (see security-engineering). [S9]

## Shape behind one interface

- Put every tool ("hand") behind one call shape, `execute(name, input) -> string`, so sandboxes and tools are replaceable without touching the harness. [S13]

## Distribution and governance (rule 13)

- Desktop Extensions package a local MCP server as a `.mcpb` file whose only required file is `manifest.json`. Declare `user_config`; fields marked `"sensitive": true` are stored in the OS keychain. [S16]
- Uber reports its MCP Gateway hosts over 800 MCP servers and over 5,000 tools, with tool-level authorization, PII redaction, and auto-generated tools disabled until an owner approves them. [S19]
- Use an agent to improve tools: a tool-testing agent that rewrote flawed MCP tool descriptions made later agents 40% faster on tasks. [S4]

## Measure tool changes

- Run evals as simple agentic loops, one loop per task. Track accuracy, runtime, tool-call count, token use and tool errors. Hold out a test set so you do not overfit descriptions. [S5]
- Eval design itself lives in llm-evals.
