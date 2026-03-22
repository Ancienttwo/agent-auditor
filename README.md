# Agent Auditor

A Claude Code skill that audits existing AI agents, agentic coding systems, Claude Code setups, and Claude Agent SDK applications for architecture quality, implementation risks, and improvement opportunities.

## What It Does

Agent Auditor produces a structured review covering:

- **System classification** — workflow, agent, or hybrid; control pattern identification
- **10 scored dimensions** — control flow, harness, context engineering, tool design, memory, autonomy, multi-agent organization, evaluation, tracing, and security boundaries
- **Platform-aware overlays** — Claude Code and Claude Agent SDK sub-criteria layered on top of the base rubric when relevant
- **Anti-pattern scan** — 8 base failure modes, plus Claude Code AP9-AP14 and Agent SDK AP15-AP20 when platform markers are present
- **Maturity judgment** — early, developing, mature, or exemplar
- **Prioritized roadmap** — quick wins, medium-term improvements, and strategic changes

Every conclusion is backed by file-level evidence and line numbers.

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
- per-dimension scores with evidence and recommendations
- a separate anti-pattern table so anti-pattern findings do not distort numeric scoring
- a prioritized roadmap grouped into quick wins, medium-term improvements, and strategic changes

## License

[MIT](LICENSE)
