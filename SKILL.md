---
name: agent-auditor
description: Audit existing AI agents, agentic coding systems, and agent-like workflows for architecture quality, implementation risks, and improvement opportunities. Use when asked to audit this agent, review agent architecture, run an agent health check, assess agent design, evaluate agent implementation, inspect agent engineering quality, check an AI workflow's maturity, audit a Claude Code system, audit a Claude Agent SDK application, 审计 Agent 架构, 评估 Agent 工程质量, 审计 Claude Code 配置, 审计 Agent SDK 应用, or scan a codebase for agent anti-patterns. Do not use when building a new agent from scratch, debugging a single bug, or doing prompt-only optimization without architecture review.
---

# Agent Auditor

Audit an existing Agent or AI workflow without modifying the target codebase.

Produce a structured review with file-backed evidence, per-dimension scores, an anti-pattern scan, blocking issues, and a prioritized improvement roadmap.

## Start With Intake

Confirm the minimum audit context before reading deeply:

- target repo, path, or design document set
- audit target type (see below)
- full audit or scoped audit
- known constraints or intentionally missing subsystems
- desired response mode

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

1. Recon the implementation surface.
2. Map the architecture.
3. Audit the scored dimensions.
4. Run the anti-pattern scan.
5. Judge maturity and blockers.
6. Build the roadmap.
7. Return the report.

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

### 6. Build the roadmap

Group changes into:

- Quick Wins
- Medium-term improvements
- Strategic changes

Make recommendations mechanism-level and specific. Prefer suggestions such as tightening tool schemas, externalizing state, adding trace events, or turning failures into regression cases.

When sequencing recommendations across dimensions, prefer: security → memory consolidation → skills/context → evaluation → multi-agent.

### 7. Return the report

Use [report-template.md](references/report-template.md) as the response skeleton when assembling the final report.

Default to chat output. Write a file only on request.

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
- the maturity judgment matches the actual findings
- anti-patterns are reported separately from numeric scoring
- the report remains readable without access to your scratch notes
