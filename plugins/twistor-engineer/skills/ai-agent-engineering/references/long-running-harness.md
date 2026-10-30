# Long-running harnesses

When work spans hours or many sessions, state lives on disk and in git, not in the model's memory. Rules 11, 12 and 14 in SKILL.md.

## Initializer and worker (rule 11)

From Anthropic's harness for a claude.ai clone [S11]:

- Two roles that differ only in the first user prompt: an initializer (first session) and a coding agent (every later session).
- The initializer creates `init.sh` (runs the dev server), `claude-progress.txt` (work log) and an initial git commit.
- Feature list in JSON, "over 200 features", each starting `"passes": false`. Agents may only flip `passes`. JSON was chosen because the model is less likely to overwrite it than Markdown.
- One feature per session. Commit with descriptive messages so bad changes revert cleanly.
- Session start ritual: `pwd`, read the progress file and `git log --oneline -20`, check the feature list, run a basic end-to-end test before coding.
- Test like a user. Browser automation via a Puppeteer MCP server; browser-native alert modals were invisible to the agent, and features that used them came out buggier.

## Planner, generator, evaluator (rule 12)

From Anthropic's harness for application builds [S12]:

- Retro game maker: solo agent 20 min and $9 vs full planner-generator-evaluator harness 6 hr and $200 (over 20x cost). The solo app's core feature did not work.
- "Tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work." Frontend runs used 5 to 15 iterations; full runs up to four hours.
- Sprint contracts: generator and evaluator agree testable criteria with hard thresholds before each sprint. One level-editor sprint had 27 criteria.
- Context anxiety: Sonnet 4.5 needed context resets with structured handoffs; Opus 4.5 largely removed this and automatic compaction sufficed.
- Re-examine the harness per model. With Opus 4.6 the sprint construct was dropped and evaluation became one final pass. DAW build: 3 hr 50 min, $124.70 total (planner 4.7 min and $0.46, first build 2 hr 7 min and $71.08, QA rounds about $3-4 each).
- Cap evaluator-optimizer loops with a hard iteration limit plus a quality threshold. [S24]

## Decouple brain, hands and log

From Managed Agents [S13]:

- Three independently replaceable parts: harness (brain), sandboxes and tools (hands), append-only session log.
- Sandboxes are disposable ("The container became cattle"). A container failure surfaces as a tool-call error and the harness provisions a replacement.
- The session log is the recoverable context store, read by positional slices, so you never trim irreversibly.
- Lazy sandbox provisioning lets inference start first: p50 time-to-first-token down roughly 60%, p95 down over 90%.
- Credentials never reach generated code (vault or resource-bundled auth, proxy for MCP calls). Mechanics in security-engineering.

## Oversight without fatigue

- Users accept 93% of manual permission prompts, so per-action prompts are weak oversight. [S30]
- Deny-and-continue: a blocked action returns a reason and the agent keeps going; escalate to a human after 3 consecutive or 20 total denials. Headless runs terminate instead. [S30]
- Classifier design and sandbox boundaries live in security-engineering.

## Observe before widening access

- Send OpenTelemetry traces per turn, model call and tool call, with timing, tokens and IDs, using GenAI semantic conventions. Review traces before broadening agent access. [S28]

## The last mile (rule 14)

- LinkedIn reports reaching 80% of the basic experience in the first month, then four more months to pass 95%. [S18]
- Start an end-to-end eval before changing the harness; "Even a set of 10 golden examples is enough to start." Run each eval more than once: one regression "only showed up in one run out of two." [S21]
- Eval suite design, graders and pass^k live in llm-evals.
