# Agent Auditor Rubric

## Table of Contents

- D1. Control Flow
- D2. Harness
- D3. Context Engineering
- D4. Tool Design
- D5. Memory
- D6. Autonomy and State Externalization
- D7. Multi-Agent Organization
- D8. Evaluation
- D9. Tracing and Observability
- D10. Security Boundaries
- Anti-Pattern Scan

## D1. Control Flow

Audit whether the core runtime stays small, stable, and composable. Good systems keep the loop simple and push new behavior into tools, prompts, or external state. The core loop (sense → decide → act → feedback) should be under ~50 lines.

- Checklist:
  - Is there a recognizable loop or orchestration core (sense-decide-act-feedback)?
  - Does the loop avoid becoming a large state machine?
  - Are new capabilities added outside the loop body?
  - Are new capabilities introduced through standard extension points: expanding tool sets, adjusting prompt structure, or externalizing state?
  - Are stop conditions explicit (end_turn, max iterations)?
  - Is tool feedback reinjected in a clear way?
  - Which control pattern is primary? Identify one: Prompt Chaining, Routing, Parallelization, Orchestrator-Workers, or Evaluator-Optimizer
  - Is the system correctly classified as Workflow (code-driven path) vs Agent (LLM-driven decisions) vs Hybrid?
  - Does the core loop stay under ~50 lines of code? (larger loops signal that policy, state, or side effects are leaking in)
- Search hints:
  - `rg -n "while|loop|stop_reason|tool_use|orchestr" .`
  - filenames containing `agent`, `loop`, `runtime`, `session`
- Scoring anchors:
  - `0-2`: no coherent runtime shape or hidden control flow everywhere
  - `3-4`: loop exists but mixes policy, state, and side effects heavily
  - `5-6`: workable loop with some extension seams, but brittle growth
  - `7-8`: stable loop with clear separation of reasoning and state handling
  - `9-10`: minimal loop plus clean extension model and explicit boundaries
- Typical issues:
  - loop body grows with every feature
  - prompt logic, tool execution, and persistence are tightly coupled
  - system labeled as "Agent" but is actually a Workflow (or vice versa)
- Optimization directions:
  - Quick Win: separate loop control from tool registration
  - Medium: move mutable task state into files or storage
  - Strategic: define a runtime contract for events, state, and termination

## D2. Harness

Audit whether the system has executable acceptance criteria, clear execution boundaries, feedback signals, and rollback or retry paths. Strong harness quality often matters more than raw model choice — the article is explicit that Harness and verification test quality have a larger impact on success rate than upgrading to a more expensive model.

- Checklist:
  - Are success and failure machine-checkable for key tasks?
  - Are execution boundaries encoded in tools, tests, or policy?
  - Are logs, metrics, or traces available as feedback?
  - Is there a retry, revert, or rollback path?
  - Are constraints enforced mechanically instead of only described?
  - Is knowledge stored in the codebase itself, not in external docs the Agent cannot see? ("Agent 看不到的内容等于不存在")
  - Can the Agent complete tasks end-to-end without human intervention?
  - Where does the system sit in the task quadrant? (clarity of goal × degree of verification automation — the "right-upper" quadrant is the sweet spot)
- Search hints:
  - `rg -n "test|assert|lint|ci|retry|rollback|revert|guard|policy" .`
  - CI config, test folders, validator utilities, hooks
  - `AGENTS.md`, `CLAUDE.md` or equivalent in-repo documentation (should be concise index, not knowledge dump)
- Scoring anchors:
  - `0-2`: no clear acceptance path or safety envelope
  - `3-4`: ad hoc validation and mostly manual recovery
  - `5-6`: partial automation with notable blind spots
  - `7-8`: solid validation, boundaries, and recovery for core flows
  - `9-10`: robust harness that reliably moves work into machine-checkable space
- Typical issues:
  - agent claims completion without validation
  - policies live only in prose
  - knowledge lives in Notion/Confluence but not in the repo
- Optimization directions:
  - Quick Win: turn one real failure into a regression check
  - Medium: encode boundaries in lint, types, or validators; move critical docs into codebase
  - Strategic: build an end-to-end harness tied to representative tasks; push all core tasks into the "goal-clear + auto-verifiable" quadrant

## D3. Context Engineering

Audit how the system manages context density, layering, compaction, and stable prefixes. Strong systems separate always-on rules from optional knowledge and runtime state. Context Rot — where irrelevant content drowns out useful signals — is the most common failure mode.

- Checklist:
  - Are stable rules separated from dynamic state?
  - Is context layered? Five layers to check: resident (identity/constraints), on-demand (Skills/knowledge), runtime (time/user/channel), memory (cross-session), system (Hooks/code rules that stay outside context entirely)
  - Are large knowledge sources loaded on demand (not all at once)?
  - Is there a compaction or summarization strategy (sliding window, LLM summary, or tool-result replacement)?
  - Are compression priorities explicitly defined? (architecture decisions > modified files > verification status > TODOs > tool output)
  - Is prompt caching friendliness considered (stable prefix, dynamic content appended after)?
  - Are deterministic rules kept outside the prompt via Hooks or code (never re-read by model)?
  - Is the file system used as a context interface? (tools write to files, Agent reads on demand — Dynamic Context Discovery)
  - For Skill-based systems: do Skill descriptions include anti-examples? (without anti-examples, routing accuracy drops from 85% to 53%)
  - Is Skill loading disciplined? (scan before every reply, load one at a time, prefer most specific match when multiple Skills could apply)
  - Are Skill bodies lean? (content in supporting files not inlined, single responsibility per Skill, side-effect Skills rate-limited)
- Search hints:
  - `rg -n "prompt|system|summary|compact|cache|skill|memory|SOUL|AGENTS" .`
  - prompt builders, skill loaders, memory injectors
  - look for `micro_compact`, `auto_compact`, or similar compaction triggers
- Scoring anchors:
  - `0-2`: prompt is a dumping ground
  - `3-4`: some layering exists but context rot risk is high
  - `5-6`: basic layering and some compaction, inconsistent discipline
  - `7-8`: clear layering, selective loading, and compaction strategy
  - `9-10`: context is intentionally designed for signal quality and cache stability
- Typical issues:
  - giant system prompt doubles as knowledge base
  - external content enters context without filtering
  - compression discards architecture decisions or corrupts identifiers (UUIDs, hashes, URLs)
  - Skill descriptions are too long (45+ tokens) or lack "don't use when" anti-examples
- Optimization directions:
  - Quick Win: move bulky knowledge into references or files; add anti-examples to Skill descriptions
  - Medium: add summarization or tool-result replacement; define explicit compression priority list
  - Strategic: redesign prompt assembly around stable prefix plus dynamic suffix; use file system as context interface for tool outputs

## D4. Tool Design

Audit whether tools are designed for agent goals rather than raw APIs. Good tools are easy to choose, hard to misuse, and capable of returning structured repair hints. Tool design has evolved through three generations: API wrapping → ACI (Agent-Computer Interface) → Advanced Tool Use (dynamic discovery, code orchestration, examples).

- Checklist:
  - Do tool names and descriptions reveal when to use and not use them?
  - Do schemas constrain parameters clearly?
  - Are errors structured enough to support recovery (error code + suggestion, not just a string)?
  - Are tool definitions close to implementations (e.g., Zod schema co-located with handler)?
  - Are examples available for complex tools? (adding 1-5 real call examples raises accuracy from 72% to 90%)
  - Is the active tool count controlled? (5 MCP servers alone can consume ~55K tokens of definitions — nearly 30% of a 200K context)
  - Are framework-level messages (compression events, notifications) filtered before reaching the LLM? (AgentMessage vs Message separation)
  - Is there dynamic tool discovery (search_tools) rather than loading all definitions at once?
  - For multi-step tool sequences: can the model orchestrate via code (programmatic tool calling) to keep intermediate results out of LLM context? (token reduction from ~150K to ~2K)
  - Identify the tool design generation: raw API wrapping, ACI (goal-oriented), or Advanced (dynamic discovery + programmatic tool calling + examples)
  - When a behavior needs to be reliable, is it implemented as a dedicated tool rather than an optional parameter or format instruction? (optional parameter → required format → dedicated tool = increasing reliability)
- Search hints:
  - `rg -n "tool|input_schema|zod|json schema|mcp|function" .`
  - tool registries, API wrappers, handler definitions
  - count total tool definitions to assess sprawl
- Scoring anchors:
  - `0-2`: thin API wrappers with vague contracts
  - `3-4`: usable tools but frequent ambiguity or error loops
  - `5-6`: mostly workable tools with uneven descriptions and repair paths
  - `7-8`: agent-oriented tools with good schemas and structured failures
  - `9-10`: highly legible, example-driven tools that support self-correction
- Typical issues:
  - too many overlapping tools
  - stringly typed error returns with no repair guidance
  - internal framework events pollute LLM context
- Optimization directions:
  - Quick Win: add explicit "use when / do not use when" wording to tool descriptions
  - Medium: tighten schemas, standardize error envelopes with recovery suggestions, filter framework messages before LLM
  - Strategic: redesign around ACI primitives; implement dynamic tool discovery for large tool sets

## D5. Memory

Audit whether the system preserves useful state across turns or sessions without letting memory become unbounded noise. The four memory types to look for: working memory (current context window), procedural memory (Skills/how-to knowledge), episodic memory (session history — what happened), and semantic memory (curated facts — MEMORY.md or equivalent).

- Checklist:
  - Is any memory persisted beyond the current turn?
  - Are the four memory types distinguishable: working (context window), procedural (Skills), episodic (session logs), semantic (curated facts)?
  - Is there a MEMORY.md or equivalent semantic memory that the Agent actively maintains?
  - Is consolidation triggered automatically (e.g., at a token usage threshold like 50%)?
  - Is consolidation pointer-based and reversible (move pointer, don't delete originals)?
  - Can memory be reviewed, regenerated, or rolled back by the user?
  - Is retrieval selective (hybrid 70% vector + 30% keyword, or at least keyword-searchable) instead of blindly injecting everything?
- Search hints:
  - `rg -n "memory|consolidat|summary|retrieve|embed|vector|MEMORY" .`
  - storage files, retrieval services, background consolidation jobs
  - look for `MEMORY.md`, `memory/` directories, `.jsonl` session logs
- Scoring anchors:
  - `0-2`: no meaningful memory beyond message history
  - `3-4`: memory exists but is noisy, opaque, or hard to trust
  - `5-6`: useful persistence with limited consolidation or governance
  - `7-8`: selective retrieval and manageable consolidation behavior
  - `9-10`: deliberate multi-type memory model with rollback and hygiene
- Typical issues:
  - memory grows forever without consolidation
  - long-running tasks degrade after ~20 turns
  - consolidation is destructive (deletes originals instead of archiving)
- Optimization directions:
  - Quick Win: separate session notes from durable memory; create MEMORY.md
  - Medium: trigger consolidation at explicit token thresholds with archive fallback
  - Strategic: define memory classes and retrieval policy by task type; implement hybrid search

## D6. Autonomy and State Externalization

Audit whether long tasks can survive restarts, pauses, or session changes. Strong systems externalize progress instead of relying on one growing context window. The article recommends an Initializer Agent (runs once, produces feature-list.json + init state) and Coding Agent (loops, resumes from file state each session) pattern.

- Checklist:
  - Is task progress written to files, storage, or queues?
  - Can work resume after interruption without restarting from scratch?
  - Is there a clear boundary between runtime context and durable state?
  - Are task checkpoints visible to humans or tooling?
  - Is restart behavior intentional rather than accidental?
  - Is there an Initializer / Coding Agent separation for complex multi-session tasks?
  - Is task state stored as structured JSON (not Markdown) for reliable machine parsing?
  - Is there a heartbeat or cron mechanism for proactive task resumption (not waiting for user messages)?
  - Is only one task `in_progress` at a time, with explicit status transitions?
  - Are slow I/O operations (shell, network) offloaded to background threads with results injected before the next LLM call?
  - Is there automatic progress reminder injection when task status has not been updated for multiple turns?
- Search hints:
  - `rg -n "checkpoint|resume|queue|task|state|persist|heartbeat|cron" .`
  - task folders, session stores, job records, durable planners
  - look for `feature-list.json`, `claude-progress.txt`, `.tasks/` directories
- Scoring anchors:
  - `0-2`: autonomy depends entirely on the current session
  - `3-4`: some persistence exists but restarts are lossy or manual
  - `5-6`: partial resumability with gaps in state fidelity
  - `7-8`: durable progress model and resumable long-running work
  - `9-10`: robust externalized state with clean crash recovery behavior
- Typical issues:
  - task status is buried only in chat history
  - crashes force full restarts
  - progress stored as Markdown that the model mutates unreliably
- Optimization directions:
  - Quick Win: persist task checkpoints to JSON files; enforce one in_progress at a time
  - Medium: separate planner state from execution state; add heartbeat check
  - Strategic: build Initializer/Coding Agent split with file-system-based progress tracking

## D7. Multi-Agent Organization

Audit whether delegation is structured, isolated, and worth the complexity. Good systems add multiple agents only after single-agent limits are understood. Build in this order: protocol first → isolation → then collaboration and parallelism.

- Checklist:
  - Are roles or ownership boundaries explicit?
  - Is communication structured (e.g., JSONL inbox protocol with request_id, from_agent, to_agent, status) rather than free-text natural language?
  - Are isolation boundaries present for files (worktrees) and permissions?
  - Do sub-agents return only summaries, keeping exploration and debugging traces in their own context?
  - Do sub-agents have minimal system prompts (Tooling + Workspace + Runtime only — no Skills, no Memory leakage)?
  - Are recursion or depth limits enforced (max_depth to prevent infinite sub-agent spawning)?
  - Is there cross-validation to break hallucination amplification chains (Agent A → B → C converging on a confident wrong answer)?
  - Is the collaboration mode intentional? Distinguish Conductor (synchronous, human-in-loop each turn, ephemeral output) from Orchestrator (asynchronous delegation, human reviews persistent artifacts at end)
  - Is there a task graph with explicit dependency tracking (.tasks/ or equivalent)?
  - Was multi-agent design introduced in the right order? (protocol first → isolation → then collaboration and parallelism)
- Search hints:
  - `rg -n "spawn|delegate|worker|planner|subagent|orchestrator|worktree" .`
  - coordinator modules, task routers, worker protocols
  - look for `.team/inbox/`, `.tasks/`, `.worktrees/` directories
- Scoring anchors:
  - `0-2`: no boundaries and no clear responsibility split
  - `3-4`: multi-agent behavior exists but drifts or overlaps heavily
  - `5-6`: useful delegation with rough edges in protocol or isolation
  - `7-8`: defined roles, concise outputs, and bounded delegation
  - `9-10`: disciplined multi-agent system with clear isolation and observability
- Typical issues:
  - many agents without a task graph
  - workers return full transcripts instead of summaries
  - sub-agents inherit full Skills and Memory from parent (privilege leak)
  - errors cascade and amplify across agents without independent verification
- Optimization directions:
  - Quick Win: require structured worker outputs (summaries only); set max_depth
  - Medium: add worktree isolation and structured communication protocol; strip sub-agent prompts to minimum
  - Strategic: formalize task graph, add cross-validation step, implement JSONL inbox protocol

## D8. Evaluation

Audit whether the team can measure capability and regressions separately. Strong systems evaluate both behavior and outcomes, and they distrust flaky harnesses. Critical insight from the article: "先修评测，再改 Agent" — when eval scores drop, fix the eval system first before changing the Agent.

- Checklist:
  - Are representative tasks captured as repeatable evaluations (at least 20 cases)?
  - Are graders appropriate for the task type? Three types: code graders (deterministic, prefer these), model graders (semantic), human graders (baseline calibration)
  - Is Pass@k (capability — "can it ever do this?") distinguished from Pass^k (regression — "does it still do this reliably")?
  - Are both transcript (how it behaved) and outcome (what actually changed in the environment) evaluated?
  - Are environment failures distinguishable from model failures? (resource limits can kill processes; infra noise looks identical to model regression)
  - Is environment isolation maintained between test runs (clean state, no shared cache)?
  - Are real failures converted into future test cases immediately?
  - Are both positive and negative test cases present? (only testing "should do X" leads to one-directional optimization)
  - Is there a process for adding harder cases when pass rates approach 100%?
  - When eval scores drop, is the eval system checked first before changing the Agent? ("先修评测，再改 Agent")
- Search hints:
  - `rg -n "eval|benchmark|grader|pass@k|regression|fixture|golden" .`
  - eval folders, harnesses, grader code, fixture repos
- Scoring anchors:
  - `0-2`: no meaningful evaluation practice
  - `3-4`: one-off tests or manual spot checks only
  - `5-6`: some reusable evaluation coverage with weak diagnosis
  - `7-8`: good task coverage and clear grader strategy
  - `9-10`: reliable capability plus regression measurement with harness discipline
- Typical issues:
  - flaky infra is mistaken for model regression
  - only aggregate scores are watched; per-task breakdowns are ignored
  - eval suite is saturated (100% pass rate) and no longer tests real capability boundaries
  - Agent is modified based on bad eval signal without checking the eval system first
- Optimization directions:
  - Quick Win: capture one production failure as an eval case; add a negative test case
  - Medium: split infra errors from model outcome metrics; implement environment reset between runs; distinguish Pass@k from Pass^k
  - Strategic: maintain a living eval suite with task diversity, grader fit, and regular difficulty escalation

## D9. Tracing and Observability

Audit whether the system can explain what happened during execution. Good systems capture both low-level events and higher-level task narratives. Two layers work together: human sampling (calibration) and LLM auto-evaluation (coverage).

- Checklist:
  - Is full Trace recorded? (complete prompt with system prompt, all messages[], every tool call with args and return values, reasoning chain if thinking mode is used, final output, token consumption + latency)
  - Are events emitted at key lifecycle points (tool_start, tool_end, turn_end) without polluting model context?
  - Is the architecture event-based (emit once, multiple consumers: logs, UI, eval, review queue)?
  - Is there two-layer observability? Layer 1: human sampling of errors, long conversations, and negative feedback for failure pattern discovery and calibration. Layer 2: LLM auto-evaluation at scale, calibrated against Layer 1 labels
  - Is online eval sampling rule-based rather than random? (negative feedback → 100%, high-token conversations → priority, time-window random → baseline, post-model/prompt change → 48h full coverage)
  - Can Traces be semantically searched (not just string matching)?
  - Can the team inspect a failed run end to end?
- Search hints:
  - `rg -n "trace|span|event|log|metric|telemetry|observ" .`
  - tracing SDK setup, event buses, logging adapters, review queues
- Scoring anchors:
  - `0-2`: almost no usable execution history
  - `3-4`: logs exist but do not explain the task path
  - `5-6`: partial traces or eventing with major gaps
  - `7-8`: solid event trail and practical debugging surfaces
  - `9-10`: full-fidelity execution insight with layered observability
- Typical issues:
  - tool events are invisible or mixed into prompt history
  - no way to reconstruct a failure path
  - only random sampling, missing high-value traces
- Optimization directions:
  - Quick Win: add tool_start and tool_end event emission
  - Medium: standardize task and span identifiers; implement rule-based sampling
  - Strategic: build event-driven architecture with trace-first debugging; add LLM auto-eval calibrated by human labels

## D10. Security Boundaries

Audit whether the system constrains who can do what, where actions can land, and how risky inputs are handled. The article is emphatic: safety boundaries must precede functionality. Three things must be in place first: who can use it (authorization), where it can operate (isolation), and what it did (audit trail).

- Checklist:
  - Is there an authorization whitelist (only approved users can trigger the Agent)?
  - Is workspace or path isolation enforced (path boundary checks, symlink resolution)?
  - Are sensitive actions logged to an append-only audit trail (JSONL or equivalent)?
  - Is there prompt injection defense using source-sink analysis? (minimize sinks/capabilities, not just filter sources/inputs)
  - Is external content explicitly wrapped as untrusted when entering context? (e.g., `<untrusted_content source="email">` wrapper pattern)
  - Are sensitive operations gated by explicit user confirmation before execution?
  - Are high-risk actions (git push, rm, database writes) validated or double-checked?
  - Is there provider failover for model service outages (Anthropic 503, OpenAI rate limits → automatic switch)?
  - Is `execFile` preferred over `exec` to prevent shell injection?
- Search hints:
  - `rg -n "sandbox|allowlist|denylist|path|permission|auth|policy|audit" .`
  - shell wrappers, filesystem guards, auth gates, approval logic
  - look for `untrusted`, `confirm`, `approval`, `fallback`, `failover`
- Scoring anchors:
  - `0-2`: broad capability with little or no safety envelope
  - `3-4`: basic checks exist but are easy to bypass or incomplete
  - `5-6`: moderate controls with notable gaps in coverage or auditability
  - `7-8`: strong operational boundaries for common risky paths
  - `9-10`: comprehensive least-privilege design with reliable traceability
- Typical issues:
  - shell or file access is wider than task scope
  - external content is treated as trusted instructions (prompt injection vector)
  - no fallback when primary model provider goes down
- Optimization directions:
  - Quick Win: enforce workspace path checks; use execFile instead of exec
  - Medium: add structured audit logs, untrusted content wrapping, and confirmation gates for destructive operations
  - Strategic: design capability tiers, separate trust domains, implement provider failover chain, add independent LLM verification for critical paths

## Anti-Pattern Scan

Scan these eight anti-patterns independently of the numeric score:

1. System prompt used as a knowledge base instead of a stable rule layer
2. Tool count grows without namespace or selection discipline
3. Validation loop is missing, so completion claims are not checked
4. Multi-agent delegation lacks role boundaries or isolation
5. Memory accumulates without consolidation
6. Evaluation is missing or too weak to catch regressions
7. Multi-agent design appears before single-agent limits are understood
8. Constraints rely on expectations in docs instead of enforceable mechanisms

For each anti-pattern, report:

- present, absent, or unclear
- concrete evidence
- why it matters
- the smallest credible fix
