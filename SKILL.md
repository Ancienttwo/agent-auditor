---
name: agent-auditor
description: Audit existing AI agents, agentic coding systems, and agent-like workflows for architecture quality, implementation risks, failure patterns, and optimization opportunities. Use when asked to audit this agent, review agent architecture, run an agent health check, assess agent design, evaluate agent implementation, inspect agent engineering quality, check an AI workflow's maturity, audit a Claude Code system, audit a Claude Agent SDK application, review a transcript or tool trace of agent behavior, 审计 Agent 架构, 评估 Agent 工程质量, 审计 Claude Code 配置, 审计 Agent SDK 应用, or scan a codebase for agent anti-patterns. Do not use when building a new agent from scratch, debugging a single bug without architectural implications, or doing prompt-only optimization without architecture review.
---

# Agent Auditor

Audit an existing Agent or AI workflow without modifying the target codebase.

Produce a structured audit plus an evidence-backed optimization plan with file- or trace-backed evidence, per-dimension scores, an anti-pattern scan, blocking issues, and a prioritized set of fixes.

## Start With Intake

Confirm the minimum audit context before reading deeply:

- target repo, path, design document set, transcript, or tool trace
- audit target type (see below)
- full audit or scoped audit
- known constraints or intentionally missing subsystems
- desired response mode

### Evidence Source

Classify the evidence source before reading deeply:

- `repo`: implementation files in the current workspace are the main source of truth
- `design docs`: architecture docs or plans are the main source of truth
- `transcript trace`: conversation logs, tool traces, screenshots, or user feedback samples are the main source of truth
- `mixed`: more than one of the above matters materially

Treat transcripts and tool traces as first-class audit evidence, not as secondary anecdotes.

### Audit Target Type

Ask the user to confirm which aspect to audit:

1. **Project Agent implementation** — audit the Agent built in the current workspace (Claude Agent SDK, LangGraph, custom framework, etc.)
2. **Development environment configuration** — audit the .claude/ .codex/ engineering setup quality (CLAUDE.md, Skills, Hooks, MCP, etc.)
3. **Both** — audit project code and development environment together

Scan file markers first to help the user decide, then confirm their choice.

Default to an in-chat report. Write a file only when the user explicitly asks.

Treat missing context as a discoverable fact first. Inspect the repo before asking clarifying questions.

## Classify The System

Classify the target before scoring it:

- `workflow`: execution path is mostly code-defined and linear
- `agent`: the model dynamically decides next actions, tools, or delegation
- `hybrid`: both patterns are present in meaningful ways

Also identify the primary control pattern:

- Prompt Chaining (sequential steps, each LLM call processes the previous output)
- Routing (classify input, dispatch to specialized handlers)
- Parallelization (split into independent subtasks or run multiple times for consensus)
- Orchestrator-Workers (central LLM decomposes and delegates to worker LLMs)
- Evaluator-Optimizer (generator produces, evaluator gives feedback, loop until good enough)

Use these classifications to explain why some dimensions are strong, weak, or `N/A`.

## Audit Workflow

0. Reconstruct the failure or success path when behavioral evidence exists.
1. Recon the implementation surface.
2. Map the architecture.
3. Audit the scored dimensions.
4. Run the anti-pattern scan.
5. Judge maturity and blockers.
6. Build the optimization plan.
7. Return the report.

### 0. Failure Reconstruction

When the evidence source includes `transcript trace`, user feedback, screenshots, or tool logs, start here before reading implementation details.

Reconstruct the behavior in concrete terms:

- what the user was actually trying to achieve
- what tool path or response path the system actually took
- where the first meaningful failure occurred
- which fallback, Skill, tool, or escalation path should have triggered next
- whether the miss is best explained by capability gap, routing error, missing recovery path, or missing evaluation coverage

If the target already had a relevant capability, call that out explicitly. Distinguish:

- capability missing
- capability exists but did not trigger
- capability triggered but failed unrecoverably
- capability succeeded and the complaint is elsewhere

Do not jump into dimension scores until the failure chain is clear.

### 1. Recon the implementation surface

Inspect likely entry points and routing files first:

- `README*`, `docs/`, `src/`, `app/`, `server/`, `packages/`
- `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `MEMORY.md`
- files matching `agent`, `loop`, `tool`, `prompt`, `memory`, `trace`, `eval`, `harness`, `sandbox`, `policy`

Read enough code to locate:

- loop or runtime control flow
- prompt and context construction
- tool registration and execution
- memory or state persistence
- evaluation harness
- observability hooks
- security boundaries

If transcript evidence exists, inspect the implementation after reconstructing behavior so you can validate or falsify the suspected root cause.

#### Platform detection

After the general recon, detect platform markers to determine which overlay to load:

**Claude Code markers** (2+ present → `platform: claude-code`):
`.claude/`, `CLAUDE.md`, `.claude/settings.json`, `.claude/rules/`, `SKILL.md`, Hook config, MCP config in settings.json, `MEMORY.md`, `HANDOFF.md`

**Agent SDK markers** (any present → `platform: agent-sdk`):
`claude_agent_sdk` or `@anthropic-ai/claude-agent-sdk` in imports/dependencies, `query()`, `ClaudeSDKClient`, `ClaudeAgentOptions`, `AgentDefinition`, `settingSources`/`setting_sources` config, `permission_mode`/`permissionMode` config

Both can be present simultaneously → `platform: claude-code + agent-sdk`.

### 2. Map the architecture

Summarize the system in terms of:

- runtime model: workflow, agent, or hybrid
- state model: in-memory, files, database, queue, or mixed
- tool model: built-in, MCP, shell, browser, API wrappers, or mixed
- evaluation model: tests, harnesses, graders, or manual only
- failure handling model: retry-only, fallback ladder, user handoff, or mixed
- fallback / escalation path: what should happen after the first failure
- user handoff policy: when the system should ask the user for extra input
- existing capabilities to reuse: what already exists that could solve the problem without new tooling
- safety model: isolation, authorization, logging, validation, and fallback
- platform: `claude-code` | `agent-sdk` | `claude-code + agent-sdk` | `custom` | `unknown`

### 3. Audit the scored dimensions

Score these ten dimensions in order (they build on each other — understanding the loop informs harness assessment, which informs context engineering, and so on):

1. Control flow
2. Harness
3. Context engineering
4. Tool design
5. Memory
6. Autonomy and state externalization
7. Multi-agent organization
8. Evaluation
9. Tracing and observability
10. Security boundaries

Read [audit-rubric.md](references/audit-rubric.md) once for orientation, then revisit only the relevant section while scoring each dimension.

If the target platform includes `claude-code`, also load [claude-code-overlay.md](references/claude-code-overlay.md) and apply its per-dimension sub-criteria alongside the base rubric.

If the target platform includes `agent-sdk`, also load [agent-sdk-overlay.md](references/agent-sdk-overlay.md) and apply its per-dimension sub-criteria alongside the base rubric.

For each scored dimension, always output:

- `score`
- `evidence`
- `assessment`
- `recommendation`

Allow `N/A` only when the dimension truly does not apply. Explain why.

When a dimension contains a blocking issue, also output:

- `problem`
- `root cause`
- `smallest credible fix`
- `how to verify`

Do not stop at "improve fallback" or "tighten prompts." Keep tracing the mechanism until the recommendation is implementable.

### 4. Run the anti-pattern scan

Check the target for the eight base anti-patterns in [audit-rubric.md](references/audit-rubric.md).

If the target platform includes `claude-code`, also scan AP9–AP14 from [claude-code-overlay.md](references/claude-code-overlay.md).

If the target platform includes `agent-sdk`, also scan AP15–AP20 from [agent-sdk-overlay.md](references/agent-sdk-overlay.md).

Do not fold anti-pattern findings into the numeric score. Report them separately.

### 5. Judge maturity and blockers

Use the dimension scores and anti-pattern findings to assign one overall maturity judgment:

- `early`
- `developing`
- `mature`
- `exemplar`

List the blocking issues that most limit reliability, safety, or maintainability.

For each blocking issue, make the opinionated recommendation explicit. Say what you would change first and why.

### 6. Build the optimization plan

Group changes into:

- Immediate fixes
- Design corrections
- Eval additions
- Not in scope

Make recommendations mechanism-level and specific. Prefer suggestions such as tightening tool schemas, adding explicit fallback triggers, externalizing state, adding trace events, or turning real failures into regression cases.

Include one "What already exists" check:

- what existing capability or workflow already solves part of the problem
- whether the system should reuse it, route to it, or replace it
- whether the gap is missing capability or missing activation of an existing capability

When sequencing recommendations across dimensions, prefer: security → memory consolidation → skills/context → evaluation → multi-agent.

### 7. Return the report

Use [report-template.md](references/report-template.md) as the response skeleton when assembling the final report.

Default to chat output. Write a file only on request.

Keep unresolved decisions and evidence blind spots visible. Do not pretend design intent is settled when the evidence is incomplete.

## Scoring Rules

Use integer scores from `0` to `10`.

- `0-2`: missing or effectively absent
- `3-4`: weak or partial with major gaps
- `5-6`: functional but inconsistent or fragile
- `7-8`: solid and mostly aligned with best practice
- `9-10`: reference-grade implementation

Do not compute a weighted total score.

Use the score to support the narrative, not to replace it.

## Evidence Rules

Back every conclusion with concrete evidence:

- cite real files and line numbers when possible
- prefer implementation evidence over README claims
- distinguish code evidence from documentation evidence
- distinguish transcript evidence from code evidence
- say `not found` when something is expected but absent
- call out uncertainty when only design docs are available

Treat OpenClaw as a pattern source from the reference article, not as its own dimension.

## Reference Loading

Load [audit-rubric.md](references/audit-rubric.md) when:

- deciding what to inspect for a dimension
- calibrating scores
- checking anti-patterns
- drafting improvement options

Load [report-template.md](references/report-template.md) only when assembling the final report.

Load [claude-code-overlay.md](references/claude-code-overlay.md) when:

- the target platform is identified as `claude-code` during recon
- scoring any dimension for a claude-code system
- checking claude-code-specific anti-patterns (AP9–AP14)

Load [agent-sdk-overlay.md](references/agent-sdk-overlay.md) when:

- the target platform is identified as `agent-sdk` during recon
- scoring any dimension for an agent-sdk system
- checking agent-sdk-specific anti-patterns (AP15–AP20)

Do not load overlay files for systems where the platform is `custom` or `unknown`.

## Quality Bar

Before finishing, verify that:

- every applicable dimension has a score or justified `N/A`
- every dimension includes evidence
- recommendations are concrete and actionable
- blocking issues include root cause, smallest credible fix, and verification path
- the maturity judgment matches the actual findings
- anti-patterns are reported separately from numeric scoring
- the failure chain is explicit when behavioral evidence exists
- the report makes clear whether the gap is missing capability or a failure to use existing capability
- the report remains readable without access to your scratch notes
