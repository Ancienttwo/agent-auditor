# Agent Auditor

A Claude Code skill that audits existing AI agents, agentic coding systems, Claude Code setups, and Claude Agent SDK applications for architecture quality, implementation risks, failure patterns, and optimization opportunities.

## What It Does

Agent Auditor produces a structured audit plus optimization plan covering:

- **System classification** — workflow, agent, or hybrid; control pattern identification
- **10 scored dimensions** — control flow, harness, context engineering, tool design, memory, autonomy, multi-agent organization, evaluation, tracing, and security boundaries
- **Failure reconstruction** — transcript-first analysis of what the user wanted, what path the system took, where it failed, and which fallback should have happened next
- **Platform-aware overlays** — Claude Code and Claude Agent SDK sub-criteria layered on top of the base rubric when relevant
- **Anti-pattern scan** — 8 base failure modes, plus Claude Code AP9-AP14 and Agent SDK AP15-AP20 when platform markers are present
- **Maturity judgment** — early, developing, mature, or exemplar
- **Optimization plan** — immediate fixes, design corrections, eval additions, and explicit not-in-scope items

Every conclusion is backed by file-level or trace-level evidence, with line numbers when available.

## Install

Add to your Claude Code settings (`~/.claude/settings.json` or project-level):

```json
{
  "skills": [
    "/path/to/agent-auditor"
  ]
}
```

## Usage

```
/agent-auditor
```

Or ask naturally:

- "Audit this agent's architecture"
- "Run an agent health check"
- "Review the agent engineering quality of this repo"
- "Audit this Claude Code setup"
- "Audit this Claude Agent SDK app"
- "Audit this transcript and tell me how to improve the skill"
- "Review this tool trace and explain why the fallback path failed"

## Evidence Sources

Agent Auditor supports multiple evidence sources:

- **Repo-first audit** — inspect implementation files in the current workspace
- **Design-doc audit** — inspect architecture docs or plans when code is incomplete
- **Transcript / tool-trace audit** — reconstruct a failure chain from conversation logs, tool traces, screenshots, or user feedback
- **Mixed audit** — combine behavior evidence with code or design docs to validate the likely root cause

When transcript evidence is available, Agent Auditor reconstructs the failure chain before scoring dimensions.

## Audit Targets

Agent Auditor now supports three intake modes:

- **Project Agent implementation** — audit the agent built in the current workspace
- **Development environment configuration** — audit `.claude/`, `CLAUDE.md`, skills, hooks, MCP, and related setup
- **Both** — audit the project implementation and the development environment together

The skill scans for platform markers before asking the user which audit target they want.

## Platform-Aware Overlays

The base rubric stays framework-agnostic. When the target matches a supported platform, Agent Auditor layers in extra checks:

- **Claude Code overlay** — CLAUDE.md quality, rules layering, hooks, MCP sprawl, subagent isolation, context budgeting, and cache-friendly prompt design
- **Claude Agent SDK overlay** — `query()` vs `ClaudeSDKClient`, session handling, permission controls, tool boundaries, subagent definitions, and SDK-specific tracing

The final report includes a `Platform` field with one of:

- `claude-code`
- `agent-sdk`
- `claude-code + agent-sdk`
- `custom`
- `unknown`

## Audit Dimensions

| # | Dimension | What It Checks |
|---|-----------|---------------|
| 1 | Control Flow | Loop shape, extension model, stop conditions |
| 2 | Harness | Acceptance criteria, execution boundaries, rollback paths |
| 3 | Context Engineering | Layering, compaction, cache stability, skill routing |
| 4 | Tool Design | Naming, schemas, error recovery, ACI maturity |
| 5 | Memory | Persistence types, consolidation, retrieval selectivity |
| 6 | Autonomy & State | Checkpointing, resumability, crash recovery |
| 7 | Multi-Agent | Role boundaries, isolation, communication protocol |
| 8 | Evaluation | Task coverage, grader strategy, capability vs regression |
| 9 | Tracing | Event emission, trace completeness, layered observability |
| 10 | Security | Authorization, isolation, audit trail, injection defense |

## Report Output

The report template includes:

- audit target, scope, classification, control pattern, and detected platform
- failure chain, primary root cause, and optimization thesis
- what capabilities already exist and whether the issue is missing capability or failed routing
- per-dimension scores with evidence and recommendations
- a separate anti-pattern table so anti-pattern findings do not distort numeric scoring
- a prioritized optimization plan grouped into immediate fixes, design corrections, eval additions, and not-in-scope items
- unresolved decisions and blind spots when the evidence is incomplete

## License

[MIT](LICENSE)
