# Agent Auditor Report Template

Use this structure for the final response. Keep it readable in chat. Write a file only if the user explicitly asks.

## Audit Target

- Target:
- Scope:
- Classification: `workflow` | `agent` | `hybrid`
- Primary control pattern: `Prompt Chaining` | `Routing` | `Parallelization` | `Orchestrator-Workers` | `Evaluator-Optimizer`
- Platform: `claude-code` | `agent-sdk` | `claude-code + agent-sdk` | `custom` | `unknown`
- Evidence base:
- Constraints or blind spots:

## Overall Conclusion

- Maturity: `early` | `developing` | `mature` | `exemplar`
- Summary:
- Blocking issues:

## Dimension Score Table

| Dimension | Score | Evidence |
| --- | --- | --- |
| Control flow |  |  |
| Harness |  |  |
| Context engineering |  |  |
| Tool design |  |  |
| Memory |  |  |
| Autonomy and state externalization |  |  |
| Multi-agent organization |  |  |
| Evaluation |  |  |
| Tracing and observability |  |  |
| Security boundaries |  |  |

### ASCII Snapshot

Use filled blocks (█) for the score and light blocks (░) for the remainder. Each character represents 1 point out of 10. Use N/A when a dimension does not apply.

```text
Control flow      ████████░░  8/10
Harness           ██████░░░░  6/10
Context eng       █████████░  9/10
Tool design       ███████░░░  7/10
Memory            ████░░░░░░  4/10
Autonomy/state    ██████░░░░  6/10
Multi-agent       ░░░░░░░░░░  N/A
Evaluation        ███░░░░░░░  3/10
Tracing/observ    █████░░░░░  5/10
Security          ████████░░  8/10
```

## Per-Dimension Analysis

Repeat this block for each applicable dimension:

### {Dimension Name}

- Score:
- Evidence:
- Assessment:
- Recommendation:

When a dimension does not apply, write `N/A` and explain why.

## Anti-Pattern Scan

| Anti-pattern | Status | Evidence | Smallest credible fix |
| --- | --- | --- | --- |
| System prompt as knowledge base |  |  |  |
| Tool sprawl |  |  |  |
| Missing validation loop |  |  |  |
| Boundary-less multi-agent |  |  |  |
| Unconsolidated memory |  |  |  |
| Missing evaluation |  |  |  |
| Premature multi-agent |  |  |  |
| Constraints by hope, not mechanism |  |  |  |

When platform is `claude-code`, also scan:

| Anti-pattern | Status | Evidence | Smallest credible fix |
| --- | --- | --- | --- |
| CLAUDE.md as knowledge dump (AP9) |  |  |  |
| MCP server sprawl (AP10) |  |  |  |
| Skills without anti-examples (AP11) |  |  |  |
| Hooks doing semantic judgment (AP12) |  |  |  |
| Subagent privilege leak (AP13) |  |  |  |
| Cache-breaking prompt design (AP14) |  |  |  |

When platform is `agent-sdk`, also scan:

| Anti-pattern | Status | Evidence | Smallest credible fix |
| --- | --- | --- | --- |
| bypassPermissions as default (AP15) |  |  |  |
| Unbounded agent runs (AP16) |  |  |  |
| Overly broad tool access (AP17) |  |  |  |
| Missing error recovery (AP18) |  |  |  |
| Session state amnesia (AP19) |  |  |  |
| Hardcoded secrets (AP20) |  |  |  |

## Prioritized Roadmap

### Quick Wins

- 

### Medium-Term Improvements

- 

### Strategic Changes

- 

## Reviewed Files

- 
