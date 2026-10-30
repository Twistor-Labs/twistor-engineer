# Sources

Every figure in this skill traces to one line below. All entries were checked against the primary post in our verification pass (2026-10-30). Company numbers are self-reported by that company.

- [S1] How we built Pingora, the proxy that connects Cloudflare to the Internet. Cloudflare Blog, 2022-09-14. https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/ (VERIFIED)
- [S2] Strobelight, a profiling service built on open source technology. Engineering at Meta, 2025-01-21. https://engineering.fb.com/2025/01/21/production-engineering/strobelight-a-profiling-service-built-on-open-source-technology/ (CORRECTED)
  Note: an earlier report said the "&" fix saved "thousands of servers"; the post says an estimated 15,000 servers in capacity savings per year.
- [S3] Bending pause times to your will with Generational ZGC. Netflix Technology Blog, 2024-03-05. https://netflixtechblog.com/bending-pause-times-to-your-will-with-generational-zgc-256629c9386b (VERIFIED)
- [S4] Java 21 Virtual Threads, Dude, Where's My Lock?. Netflix Technology Blog, 2024-07-29. https://netflixtechblog.com/java-21-virtual-threads-dude-wheres-my-lock-3052540e231d (CORRECTED)
  Note: an earlier report called it a deadlock; Netflix calls it "a variation of the classic deadlock problem" with one lock and a semaphore of 4 permits, 4 pinned virtual threads on 4 vCPU.
- [S5] GuideLLM, evaluate LLM deployments for real-world inference. Red Hat Developer, 2025-06-20. https://developers.redhat.com/articles/2025/06/20/guidellm-evaluate-llm-deployments-real-world-inference (VERIFIED)
- [S6] Accelerate multi-turn workloads with llm-d. Red Hat Developer, 2026-01-13. https://developers.redhat.com/articles/2026/01/13/accelerate-multi-turn-workloads-llm-d (CORRECTED)
  Note: an earlier report merged GuideLLM and llm-d into one benchmark; they are separate posts. llm-d figures: TTFT P50 123 -> 92 ms, P95 745 -> 272 ms, throughput 0.49 (llm-d) vs 0.52 (vLLM baseline) req/s, single node, 4x L4.
- [S7] Liger-Kernel, open-source ecosystem for efficient LLM training. LinkedIn Engineering, 2024-12-05. https://www.linkedin.com/blog/engineering/open-source/liger-kernel-open-source-ecosystem-for-efficient-llm-training (VERIFIED)
- [S8] Slashing CI Costs at Uber. Uber Engineering, 2025-06-23. https://www.uber.com/blog/slashing-ci-costs-at-uber/ (VERIFIED)
- [S9] Why Discord is switching from Go to Rust. Discord Blog, 2020-02-04. https://discord.com/blog/why-discord-is-switching-from-go-to-rust (VERIFIED)
- [S10] Everything Was Fine Until Kubernetes Said 'No More CPU'. Zepto TechXPress, 2025-04-18. https://blog.zepto.com/everything-was-fine-until-kubernetes-said-no-more-cpu-3768cda76326 (VERIFIED)
- [S11] Go Performance Tuning at Scale, Zepto's pprof Journey (Part 1). Zepto TechXPress, 2025-07-09. https://blog.zepto.com/go-performance-tuning-at-scale-zeptos-pprof-journey-part-1-459c32780ac6 (VERIFIED)
- [S12] Performance engineering at Zepto. How we build low-latency systems, Part 2. Zepto TechXPress, 2025-12-15. https://blog.zepto.com/performance-engineering-at-zepto-how-we-build-low-latency-systems-part-2-4315de1f8e83 (VERIFIED)
- [S13] Build at Zepto Speed, How We Cut Frontend Webpack Build Times by 95%. Zepto TechXPress, 2026-07-07. https://blog.zepto.com/build-at-zepto-speed-how-we-cut-frontend-webpack-build-times-by-95-676440fd7963 (VERIFIED)
- [S14] Symptoms to Root Cause, Debugging PostgreSQL at Scale. Zepto TechXPress, 2025-04-15. https://blog.zepto.com/scaling-a-distributed-monolith-our-postgresql-journey-at-zepto-a2da988be62a (VERIFIED)
- [S15] How Uber Serves over 40 Million Reads per Second using an Integrated Cache. Uber Engineering, 2024-02-15. https://www.uber.com/blog/how-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache/ (CORRECTED)
  Note: 40M is the Feb 2024 figure; the 2025-08-26 follow-up (https://www.uber.com/us/en/blog/how-uber-serves-over-150-million-reads/) reports 150M+ reads/sec. The 2024 post gives both 99% and ~99.9% hit rate for the same use case; this skill uses 99%.
- [S16] Writing effective tools for agents, with agents. Anthropic, 2025-09-11. https://www.anthropic.com/research/writing-tools-for-agents (CORRECTED)
  Note: the 25,000-token tool response cap is a configurable default, not a hard limit.
- [S17] Code execution with MCP, building more efficient agents. Anthropic, 2025-11-04. https://www.anthropic.com/research/code-execution-with-mcp (VERIFIED)
- [S18] Introducing advanced tool use on the Claude Developer Platform. Anthropic, 2025-11-24. https://www.anthropic.com/research/advanced-tool-use (CORRECTED)
  Note: an earlier report dated this "late 2025"; the post is dated 2025-11-24.
- [S19] Prompt caching with Claude. Anthropic (claude.com), 2025-08-14. https://claude.com/resources/articles/prompt-caching (VERIFIED)
- [S20] Agent Harness Design, 3 Patterns for Harnessing Claude's Intelligence. Anthropic (claude.com), 2026-04-02. https://claude.com/resources/articles/harnessing-claudes-intelligence (VERIFIED)
- [S21] Reducing cost and improving performance with Claude Platform. Anthropic (claude.com), 2026-09-08. https://claude.com/resources/articles/reducing-cost-and-improving-performance-with-claude-platform (VERIFIED)
- [S22] Scaling Managed Agents, decoupling the brain from the hands. Anthropic, 2026-04-08. https://www.anthropic.com/research/managed-agents (VERIFIED)
- [S23] Quantifying infrastructure noise in agentic coding evals. Anthropic, 2026-02-05. https://www.anthropic.com/research/infrastructure-noise (VERIFIED)
- [S24] 100X Faster, How We Supercharged Netflix Maestro's Workflow Engine. Netflix Technology Blog, 2025-09-29. https://netflixtechblog.com/100x-faster-how-we-supercharged-netflix-maestros-workflow-engine-028e9637f041 (VERIFIED)
- [S25] Introducing Contextual Retrieval. Anthropic, 2024-09-19. https://www.anthropic.com/research/contextual-retrieval (VERIFIED)
- [S26] Token-saving updates on the Anthropic API. Anthropic (claude.com), 2025-03-13. https://claude.com/resources/articles/token-saving-updates (VERIFIED)
- [S27] From Bottleneck to Breakthrough, How We Rebuilt Product Enrichment at Scale. Zepto TechXPress, 2026-04-07. https://blog.zepto.com/from-bottleneck-to-breakthrough-how-we-rebuilt-product-enrichment-at-scale-d40b9d93c186 (VERIFIED)
- [S28] Voz and Clear are faster on Mac. Desert Ant Labs, 2026-09-22. https://desertant.com/blog/voz-clear-faster-on-mac/ (VERIFIED)
  Note: vendor-reported, single source.
- [S29] Who and Eye: on-device speaker diarization and image classification. Desert Ant Labs, 2026-10-20. https://desertant.com/blog/who-and-eye/ (VERIFIED)
  Note: vendor-reported, single source.
