# Changelog

All notable changes to this project will be documented in this file.

## 2026-03-22

### Added

- Claude Code platform overlay with platform-specific audit checks for context, tools, hooks, subagents, observability, and security.
- Claude Agent SDK platform overlay with SDK-specific checks for sessions, permissions, subagents, tracing, and failure handling.
- Intake support for choosing whether to audit project agent implementation, development environment configuration, or both.
- Platform-aware anti-pattern reporting for Claude Code (AP9-AP14) and Claude Agent SDK (AP15-AP20).

### Changed

- Expanded the base audit rubric with additional general architecture checks for loop size, skill loading discipline, dedicated tools for reliable behaviors, long-task state handling, multi-agent collaboration order, and eval-first debugging.
- Updated the report template to capture platform information alongside the base classification and control-pattern summary.
- Refreshed skill UI metadata to reflect auditing of agent systems and AI development setups, not only generic agent architecture.
- Documented the new platform-aware audit flow in the README.
