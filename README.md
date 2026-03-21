# Agent Auditor

A Claude Code skill that audits existing AI agents, agentic coding systems, and agent-like workflows for architecture quality, implementation risks, and improvement opportunities.

## What It Does

Agent Auditor produces a structured review covering:

- **System classification** — workflow, agent, or hybrid; control pattern identification
- **10 scored dimensions** — control flow, harness, context engineering, tool design, memory, autonomy, multi-agent organization, evaluation, tracing, and security boundaries
- **Anti-pattern scan** — 8 common failure modes checked independently
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

## License

[MIT](LICENSE)
