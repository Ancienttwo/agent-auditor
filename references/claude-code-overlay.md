# Claude Code Platform Overlay

当审计目标使用 Claude Code 平台时加载此文件。提供 Claude Code 特定的附加评分标准和反模式，与 base rubric 叠加使用。

## 适用条件

检测到以下标记中的 2 个或以上时加载：

- `.claude/` 目录
- `CLAUDE.md`（项目根或嵌套）
- `.claude/settings.json`（含 skills / hooks / mcpServers 键）
- `.claude/rules/` 目录
- `SKILL.md` 文件（独立或 `.claude/skills/` 下）
- Hook 配置（PostToolUse / PreToolUse / Notification 等）
- MCP server 配置
- `MEMORY.md`
- `HANDOFF.md`

## 平台 Recon 目标

在 base rubric 的 recon 文件列表之外，额外检查：

- `.claude/settings.json`：hooks、mcpServers、permissions 配置
- `.claude/rules/*.md`：路径/语言特定规则
- `.claude/skills/*/SKILL.md`：skill 定义和 supporting files
- `CLAUDE.md`：大小（推荐 <2.5K tokens）、是否含 Compact Instructions 和 Verification section
- `MEMORY.md`：语义记忆索引结构
- `HANDOFF.md`：会话交接状态

## D1. 控制流 — Claude Code 子标准

- 系统是否依赖 Claude Code 内置的 gather-act-verify 循环，而非自建控制流？
- 新能力是否通过 Skills / Tools / Hooks 扩展，而非修改核心交互模式？
- 停止条件是否通过 `maxTurns` 或显式工具信号管理，而非隐式超时？
- 是否正确使用 Plan Mode 将探索和执行分离？

## D2. 治具 — Claude Code 子标准

- `CLAUDE.md` 是否精简契约（推荐 ~2.5K tokens 以内），而非知识库？
- `CLAUDE.md` 是否包含 Verification section，定义机器可检查的完成标准？
- 是否使用 Hooks + Skills + CLAUDE.md 三层执行栈？（CLAUDE.md 声明策略 → Skill 定义工作流 → Hook 机械执行）
- PostToolUse Hook 是否用于 shift-left 验证（Edit 后自动 lint/typecheck/compile check）？
- 约束是否通过 Hook 机械执行，而非仅在 CLAUDE.md 中描述？

## D3. 上下文工程 — Claude Code 子标准

- 上下文是否按五层模型分层？（常驻：CLAUDE.md → 按路径：.claude/rules/ → 按需：Skills → 隔离：Subagents → 不进上下文：Hooks）
- `.claude/rules/` 是否用于路径/语言特定规则，避免根 CLAUDE.md 过载？
- 是否定义了 Compact Instructions，显式控制压缩优先级？（架构决策 > 修改文件 > 验证状态 > TODO > 工具输出）
- Skill 描述是否精简（推荐 <15 tokens），包含"何时用/何时不用"反例？
- MCP server 工具定义的 token 成本是否受控？（5 个 server 可消耗 ~55K tokens，占 200K 上下文的 ~30%）
- Prompt 布局是否缓存友好？（静态前缀 → 动态内容追加在后）
- 是否使用 HANDOFF.md 模式在长任务中传递会话状态？

## D4. 工具设计 — Claude Code 子标准

- MCP server 数量是否受控？（推荐 ≤3-4 个活跃连接）
- 是否使用 `/mcp` 管理连接、断开闲置 server？
- 大工具集是否使用 `defer_loading` / ToolSearch 按需发现，而非启动时全量加载？
- 是否正确区分 Skill（静态知识/工作流）和 Tool（动作能力）的边界？
- 工具输出是否管理噪声？（如 `| head -30` 截断长输出，避免污染上下文）

## D5. 记忆 — Claude Code 子标准

- 是否使用 MEMORY.md 作为语义记忆，由 Agent 主动维护？
- 四类记忆是否可区分？（工作记忆：上下文窗口 / 程序记忆：Skills / 情景记忆：session .jsonl 历史 / 语义记忆：MEMORY.md）
- Compact Instructions 是否保留正确内容？（压缩后架构决策不丢失）
- 是否使用 HANDOFF.md 在长任务结束时记录进度、尝试过什么、下一步该做什么？

## D6. 自主性和状态外化 — Claude Code 子标准

- 任务状态是否存储为 JSON 文件（而非 Markdown），确保机器可靠解析？
- 复杂多会话任务是否有 Initializer Agent / Coding Agent 分离？
- 是否利用 `--continue` / `--resume` / `--fork` 实现会话连续性？
- 进度是否记录在 Agent 启动时读取的文件中？（如 `claude-progress.txt`、`feature-list.json`）

## D7. 多代理组织 — Claude Code 子标准

- Subagent 是否配置了显式约束？（`tools` / `model` / `maxTurns` / `isolation: worktree`）
- Subagent 系统提示是否最小化？（仅 Tooling + Workspace + Runtime，不泄漏 Skills 或 Memory）
- 是否为不同目的选择正确的 Subagent 类型？（Explore + Haiku 用于扫库、Plan 用于规划、General-purpose 用于执行）
- Subagent 输出是否限制为摘要，探索轨迹保留在子上下文中？

## D8. 评估 — Claude Code 子标准

- 是否使用 `claude -p`（非交互模式）运行自动化评估任务？
- 评估运行是否通过 `--worktree` 隔离，防止状态污染？
- 是否有将生产失败转化为评估用例的流程？

## D9. 追踪和可观测 — Claude Code 子标准

- `~/.claude/projects/` 下的 session 历史（.jsonl）是否可用于事后分析？
- Hook 输出是否用于结构化事件日志，且不污染模型上下文？
- 是否使用 `/context` 监控 token 消耗、诊断上下文膨胀？
- 自动化流水线是否使用 `--output-format stream-json` 获取机器可读的 trace 输出？

## D10. 安全边界 — Claude Code 子标准

- 权限是否通过 `/permissions` 或 `settings.json` 白名单显式配置？
- 破坏性操作（git push --force、rm、数据库写入）是否通过 Hook 阻断或确认门控？
- 高自动化场景是否启用 sandbox 模式？
- MCP server 是否遵循最小权限原则？（仅连接必要能力）
- 是否优先使用 `execFile` 而非 `exec` 防止 shell 注入？

## Claude Code 反模式

与 base rubric 的 8 个反模式独立扫描。

### AP9. CLAUDE.md 知识库化

根 CLAUDE.md 超过 ~3K tokens，或混合项目契约与领域知识、教程、参考资料。

- 症状：CLAUDE.md 随时间不断膨胀，每次有人"加一条"
- 为何重要：CLAUDE.md 每次请求都全量加载，膨胀直接挤压动态可用上下文
- 最小修复：将知识移到 Skills 的 supporting files；CLAUDE.md 仅保留命令、约束、架构边界

### AP10. MCP server 膨胀

同时连接 3-4 个以上 MCP server，工具定义消耗 30%+ 上下文预算。

- 症状：对话变浅，尽管上下文窗口足够大
- 为何重要：每个 MCP server 含 20-30 个工具定义，每个约 200 tokens
- 最小修复：断开闲置 server，使用 `defer_loading`，对可过滤数据优先用 Skill + shell 替代 MCP

### AP11. Skill 缺少反例描述

Skill 描述缺少"何时不用"条款。

- 症状：错误的 Skill 触发，或多个 Skill 竞争
- 为何重要：没有反例时路由准确率从 ~85% 降至 ~53%
- 最小修复：为每个 Skill 描述添加 1-2 个具体反例

### AP12. Hook 做语义判断

Hook 尝试复杂推理（读取大量上下文、多步决策），而非确定性检查。

- 症状：Hook 执行变慢、不可靠、产生误报
- 为何重要：Hook 应该是确定性拦截层，语义判断应回到模型
- 最小修复：限制 Hook 为确定性操作（lint、format、compile check、regex guard）；语义判断推回 Skills 或 CLAUDE.md 策略

### AP13. Subagent 权限泄漏

Subagent 继承父代理的全部 Skills、Memory 或 Tools。

- 症状：Subagent 上下文被无关指令膨胀，或执行超出预期范围的操作
- 为何重要：Subagent 的核心价值是隔离，权限泄漏破坏隔离
- 最小修复：配置最小系统提示（Tooling + Workspace + Runtime），显式限制工具列表

### AP14. 破坏缓存的 Prompt 设计

动态内容（时间戳、随机工具顺序、每请求 ID）放在系统提示的静态前缀中。

- 症状：API 成本高于预期，尽管 prompt 内容稳定
- 为何重要：Prompt 缓存按前缀匹配，动态内容破坏缓存命中
- 最小修复：将动态内容移到稳定前缀之后；保持系统提示、工具定义、CLAUDE.md 的固定顺序
