# Scanners

Run a tool only if `command -v <tool>` finds it. Otherwise print the command and move on. Never
install. Active tools need the user's stated ownership or authorization of the target first.
Every hit is a candidate for the verifier, not a finding. Commands are from each tool's own
README or docs, checked 2026-11-09 ([S3] to [S13] in `sources.md`).

| Situation | Tool | License | Cost | Passive / active | Command | Notes |
|---|---|---|---|---|---|---|
| Secrets in files or new commits | Gitleaks | MIT | Free | Passive, local | `gitleaks git -v .` (history) or `gitleaks dir -v .` (no `.git`, or files only) | `detect` and `protect` are deprecated since v8.19.0. |
| Secrets in history, is the key live? | TruffleHog | AGPL-3.0 | Free | Passive on the repo; verification calls each provider's API with the found key | `trufflehog git file://. --no-verification` (verify only after the user agrees: `--results=verified`) | `--only-verified` still works but is hidden; `--results=verified` is the documented flag. Tell the user keys get sent to their providers to check them. |
| Vulnerable dependencies | OSV-Scanner | Apache-2.0 | Free | Passive; sends package names and versions to OSV.dev unless `--offline` | `osv-scanner scan source -r .` | Reads lockfiles; a bare manifest may give partial results. |
| Custom bug patterns (injection, unsafe APIs) | Semgrep CE | LGPL-2.1 | Free (CE) | Passive, local | `semgrep scan` | No login needed. CE analyzes within one function or file, so it misses cross-file flows. `--config auto` needs metrics on. |
| GitHub Actions workflows | zizmor | MIT | Free | Passive; some audits go online with a GitHub token | `zizmor .` (add `--offline` to stay local) | Flags `pull_request_target`, template injection, credential persistence, broad permissions. Never pass `--fix` during an audit. |
| Third-party skill or MCP server before install | SkillSpector | Apache-2.0 | Free with `--no-llm`; LLM stage needs an OpenAI-compatible key | Passive static; dependency lookup queries OSV.dev | `skillspector scan <path-or-git-url> --no-llm` | Covers prompt injection, exfiltration, excessive agency, MCP tool poisoning. |
| Known CVEs on a running app | Nuclei | MIT | Free CLI | **Active**: sends requests to the target | `nuclei -target https://<host> -t http/cves/` | Ask for ownership or authorization first. Never against hosts you do not control. |
| AI pentest of a running app or code dir | Strix | Apache-2.0 | Free OSS, but needs Docker and an LLM API key (paid unless a local model) | **Active**: runs attack agents in Docker, hits URLs | `STRIX_LLM=<provider/model> LLM_API_KEY=<key> strix --target <path-or-url>` | Ask first for every run, path or URL. Say it spends the user's LLM quota. |
| PR review in CI | Claude Code Security Review | MIT | Needs an Anthropic API key (`claude-api-key`, required) | Runs on each PR in GitHub Actions | `uses: anthropics/claude-code-security-review@main` | Not the default. Its README says it is not hardened against prompt injection; use only on trusted PRs. |
