# Agent Auditor Report Template

Use this structure for the final response. Keep it readable in chat. Write a file only if the user explicitly asks.

## Audit Target

- Target:
- Scope:
- Classification: `workflow` | `agent` | `hybrid`
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

## Prioritized Roadmap

### Quick Wins

- 

### Medium-Term Improvements

- 

### Strategic Changes

- 

## Reviewed Files

- 
