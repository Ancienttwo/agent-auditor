# Claude Agent SDK Platform Overlay

当审计目标使用 Claude Agent SDK 构建时加载此文件。提供 Agent SDK 特定的附加评分标准和反模式，与 base rubric 叠加使用。

Agent SDK 是产品级平台，提供内置工具、安全机制和 agentic 能力，用于构建 AI Agent 应用。

## 适用条件

检测到以下任一标记时加载：

- `claude_agent_sdk` 或 `@anthropic-ai/claude-agent-sdk` 在 import 或依赖中
- `requirements.txt`、`pyproject.toml`、`package.json` 引用 SDK 包
- 代码使用 `query()`、`ClaudeSDKClient`、`ClaudeAgentOptions`（Python）
- 代码使用 `query()`、`AgentDefinition`（TypeScript）
- `settingSources` / `setting_sources` 配置
- Session resume / fork 使用模式
- `permission_mode` / `permissionMode` 配置
- Subagent 工具范围声明

## 平台 Recon 目标

在 base rubric 的 recon 文件列表之外，额外检查：

- SDK 导入和初始化模式（`query()` 一次性 vs `ClaudeSDKClient` 生命周期控制）
- `ClaudeAgentOptions` / options 对象的配置完整性
- `AgentDefinition` 声明（description / prompt / tools 约束）
- `permission_mode` 设置和 approval callback 实现
- `allowed_tools` / `disallowed_tools` 范围
- `max_turns` / `max_budget_usd` 上限设置
- `system_prompt` / `systemPrompt` 内容和大小
- 自定义 MCP 工具定义（`create_sdk_mcp_server` / `createSdkMcpServer`）
- Session ID 捕获和 resume 模式
- 错误处理模式（ResultMessage.subtype 检查、typed exceptions）
- Hook callback 定义

## D1. 控制流 — Agent SDK 子标准

- 是否正确选择 `query()`（一次性任务）vs `ClaudeSDKClient`（生命周期控制、自定义工具、中断能力）？
- 是否设置 `max_turns` 防止无限循环？（默认无限制，生产环境必须设置）
- 是否设置 `max_budget_usd` 成本上限？（默认无限制）
- `ResultMessage.subtype` 是否完整处理？（success / error_max_turns / error_max_budget_usd / error_during_execution / error_max_structured_output_retries）
- `stop_reason` 是否检查？（end_turn / max_tokens / refusal）
- `effort` 参数是否按任务复杂度调整？（简单任务用 low 降低成本，复杂推理用 high/max）

## D2. 治具 — Agent SDK 子标准

- `permission_mode` 是否按场景配置？（交互应用：default + approval callback / 自动开发：acceptEdits / 隔离 CI：bypassPermissions）
- `allowDangerouslySkipPermissions` 是否仅在隔离环境中使用且不以 root 运行？
- 交互模式下是否实现 approval callback 将权限请求展示给用户？
- `system_prompt` 是否包含明确的验证标准和完成定义？
- 是否使用 typed exceptions 进行错误恢复？（CLINotFoundError / CLIConnectionError / ProcessError）
- 错误路径是否仍然追踪 cost/usage 和保存 session_id 用于恢复？

## D3. 上下文工程 — Agent SDK 子标准

- `system_prompt` 是否精简约束（而非知识库）？
- 是否显式配置 `allowed_tools` / `allowedTools` 作为 auto-approve 集合，并结合 `disallowedTools`、`permissionMode`、`canUseTool` 形成真实权限边界？
- `settingSources` 是否受控？（默认不加载 CLAUDE.md，需要时显式设置 `["project"]`）
- 持久规则是否放在 CLAUDE.md 而非初始 prompt 中？（CLAUDE.md 在压缩后重新注入，初始 prompt 可能被压缩丢失）
- `effort` 参数是否用于成本-质量权衡？（subagent 或简单任务用 low）
- MCP 工具是否按需加载？（ToolSearch 替代全量定义）
- 是否理解 Subagent 用于上下文隔离？（每个 Subagent 从空上下文开始，仅最终消息返回父代理）

## D4. 工具设计 — Agent SDK 子标准

- 是否避免把 `allowed_tools` / `allowedTools` 误当成限制机制？真实限制应由 `disallowedTools`、`permissionMode`、`canUseTool` 或 subagent `tools` 字段承担
- 自定义 MCP 工具（`create_sdk_mcp_server` / `createSdkMcpServer`）的 schema 是否清晰？
- `@tool` 装饰器（Python）/ `betaZodTool`（TypeScript）是否有完整的输入验证和描述？
- 工具描述是否支持正确路由？（Claude 依赖描述决定何时使用工具）
- 只读工具是否标记 `readOnly` 以支持并行执行？
- MCP server 是否在运行时管理？（reconnect / toggle 断开闲置连接）
- Bash 命令是否通过 `Bash(npm:*)` 语法限定范围？

## D5. 记忆 — Agent SDK 子标准

- Session 是否持久化到磁盘？（默认：~/.claude/projects/<encoded-cwd>/<session-id>.jsonl）
- 是否从 SystemMessage init 事件捕获 `session_id` 用于后续 resume？
- 多步工作流是否利用 `resume` 参数恢复之前的会话上下文？
- 是否使用 `list_sessions()` / `get_session_messages()` 进行历史查询？
- 是否使用 `rename_session()` / `tag_session()` 组织会话？
- 是否使用 `forkSession()` 分支对话？（注意：fork 分支历史，不分支文件系统——文件编辑是真实的）
- `persistSession: false`（TypeScript）是否仅在确实不需要历史时使用？

## D6. 自主性和状态外化 — Agent SDK 子标准

- 多步工作流是否实现 session resumption 模式？（capture session_id → persist → resume later）
- `cwd` 是否为文件操作显式设置？
- structured output 是否用于状态序列化？
- 是否在 `ClaudeSDKClient` 上下文中通过多次 `client.query()` 调用共享 session？
- `persistSession: false` 和持久化的权衡是否有意为之？

## D7. 多代理组织 — Agent SDK 子标准

- `AgentDefinition` 是否有显式的 `description`（何时触发）和 `prompt`（角色定义）？
- `Agent` 工具是否在 `allowed_tools` 中？（否则无法生成 subagent）
- Subagent 工具范围是否收窄？（`tools` 字段显式限制，而非继承父代理全部工具）
- Subagent 是否排除 `Agent` 工具？（Subagent 不能再生 subagent）
- 是否处理 task 事件？（TaskStartedMessage / TaskProgressMessage / TaskNotificationMessage）
- 并行 subagent 是否用于减少延迟？（多个 subagent 并发运行）
- Subagent model 是否按用途选择？（analysis 用 sonnet/haiku，critical review 用 opus）

## D8. 评估 — Agent SDK 子标准

- 是否使用 `query()` 构建自动化测试 harness，对 `ResultMessage` 进行断言？
- `max_turns` 是否用于限制评估成本？
- structured output schema 是否用于验证响应格式？
- session history 是否用于回放和回归测试？
- `ResultMessage.subtype` 是否在测试中区分 success 和各种 error 类型？

## D9. 追踪和可观测 — Agent SDK 子标准

- Message type 是否完整处理？（ResultMessage / SystemMessage / AssistantMessage / RateLimitEvent）
- 是否使用 `agentProgressSummaries` 监控 subagent 进度？（AI 生成的周期性摘要）
- Hook callback 是否利用 `agent_id` / `agent_type` 进行归因？（区分主代理和 subagent 的工具调用）
- `PreCompact` hook 是否用于在压缩前存档完整 transcript？
- `RateLimitEvent` 是否用于速率限制感知和回退？
- session history（`list_sessions` / `get_session_messages`）是否可用于事后分析？

## D10. 安全边界 — Agent SDK 子标准

- `permission_mode` 升级策略是否合理？
  - 交互应用：`default` + approval callback
  - 自动开发机器：`acceptEdits`（自动接受文件编辑，Bash 仍需审批）
  - 隔离 CI/容器：`bypassPermissions`（不能以 root 运行）
- 权限边界是否由 `disallowedTools` / `permissionMode` / `canUseTool` 明确实现，而不是把 `allowed_tools` / `allowedTools` 当成白名单？
- 秘密值是否通过 `env` option 传递？（而非硬编码在 system_prompt 或工具定义中）
- `Bash` 工具对不可信输入是否默认拒绝？
- `cwd` 是否用于文件操作隔离？
- 多用户应用是否实现 per-user session 隔离？

## Agent SDK 反模式

与 base rubric 的 8 个反模式独立扫描。

### AP15. bypassPermissions 默认化

在非隔离环境中将 `permission_mode` 设置为 `bypassPermissions`。

- 症状：Agent 无需任何确认即可执行破坏性操作
- 为何重要：跳过所有安全提示，仅适用于隔离的 CI/容器环境
- 最小修复：交互应用使用 `default` + approval callback；自动环境使用 `acceptEdits`；仅在确认隔离时使用 `bypassPermissions`

### AP16. 无限制代理运行

未设置 `max_turns` 或 `max_budget_usd`。

- 症状：Agent 可能无限运行，产生不可控的 API 成本
- 为何重要：开放式 prompt 可能导致 Agent 进入无限工具调用循环
- 最小修复：为每个 `query()` 调用设置合理的 `max_turns`（如 10-50）和 `max_budget_usd` 上限

### AP17. 工具访问过宽

工具访问范围过宽，且实现依赖错误的权限控制假设（例如把 `allowed_tools` / `allowedTools` 当成硬性白名单）。

- 症状：Agent 执行超出预期的操作（如编辑文件、运行命令）
- 为何重要：最小权限原则——只授予完成任务所需的工具
- 最小修复：按任务类型收窄 `disallowedTools`、`canUseTool` 或 subagent `tools`，再用 `allowed_tools` / `allowedTools` 仅降低审批噪声

### AP18. 缺少错误恢复

未处理 `ResultMessage` 的 error subtypes，未 catch typed exceptions。

- 症状：error_max_turns 时尝试读取不存在的 result 字段；API 连接失败时程序崩溃
- 为何重要：ResultMessage.result 仅在 subtype 为 success 时存在
- 最小修复：检查 `ResultMessage.subtype` 后再读取 result；catch CLINotFoundError / CLIConnectionError / ProcessError

### AP19. Session 状态失忆

多步工作流中未捕获 `session_id` 进行 resume。

- 症状：每次 query 从零开始，丢失之前的分析和上下文
- 为何重要：Session resume 让后续 query 保留完整会话历史
- 最小修复：从 SystemMessage init 事件捕获 session_id，存储后通过 `resume` 参数传递给后续 query

### AP20. 秘密硬编码

API keys、tokens 或密码硬编码在 `system_prompt`、工具定义或代码中。

- 症状：秘密值出现在 session 历史、日志或 API 请求中
- 为何重要：Session 历史持久化到磁盘（.jsonl），秘密值可被其他进程读取
- 最小修复：通过 `env` option 传递秘密值作为环境变量，在工具实现内部读取
