# Claude Code 学习笔记

## 核心概念

Claude Code 是 AI 驱动的编程助手，支持多平台（Terminal/VS Code/Desktop/Web/JetBrains）。

### 关键约束：上下文窗口

**上下文会快速填满，性能随之下降。** 包含：所有消息、读取的文件、命令输出。

**管理策略**：
- 频繁 `/clear` 在无关任务间
- 用 subagents 做大量文件探索
- 同一问题纠正 2 次后，重新开始

---

## 最佳实践

### 1. 让 Claude 能验证工作

提供测试用例、预期输出、截图对比。这是最高优先级。

| 差的提示 | 好的提示 |
|---------|---------|
| "实现邮箱验证函数" | "写 validateEmail，测试：user@example.com=true, user@.com=false。实现后运行测试" |
| "让仪表盘更好看" | "[截图] 实现此设计。截图对比，列出差异并修复" |

### 2. 工作流：探索 → 计划 → 实现 → 提交

1. **探索**（Plan Mode）：理解代码，不做修改
2. **计划**：创建详细计划，`Ctrl+G` 可在编辑器编辑
3. **实现**（Normal Mode）：按计划编码，验证结果
4. **提交**：提交并创建 PR

**跳过计划**：改动小（修复 typo、重命名变量）

### 3. 提示技巧

- `@文件名` - 引用文件
- 粘贴截图/图片
- 指向现有代码模式
- 描述症状而非诊断

---

## CLAUDE.md

**位置**：`./CLAUDE.md`（项目级）或 `~/.claude/CLAUDE.md`（全局）

**生成**：`/init`

**黄金法则**：**简洁！** 每行问："删除这行会让 Claude 犯错吗？"

| ✅ 包含 | ❌ 排除 |
|--------|--------|
| 猜不到的 bash 命令 | 标准语言约定 |
| 项目特定代码风格 | 详细 API 文档 |
| 测试指令和偏好工具 | 频繁变化的信息 |
| 常见陷阱 | "写干净代码" |

**可导入其他文件**：
```markdown
See @README.md
Git workflow: @docs/git-instructions.md
```

---

## 扩展功能

### Hooks（.claude/settings.json）
确定性执行，如编辑后自动运行 eslint。

### Skills（.claude/skills/<name>/SKILL.md）
可复用工作流，用 `/skill-name` 调用。

### Subagents（.claude/agents/<name>.md）
独立上下文，适合大量文件探索。

### MCP（claude mcp add）
连接外部工具（Notion、Figma、数据库）。

---

## 会话管理

| 命令 | 作用 |
|------|------|
| `Esc` | 停止当前动作 |
| `Esc + Esc` / `/rewind` | 回退到检查点 |
| `/clear` | 重置上下文 |
| `/btw` | 快捷问题（不污染上下文） |
| `/compact` | 手动压缩上下文 |
| `claude --continue` | 继续最近对话 |
| `claude --resume` | 选择会话恢复 |

---

## 检查点与分支（Checkpoint & Branch）⭐关键功能

### `/rewind` - 时光倒流

**作用**：回退到会话中的某个检查点

**触发方式**：
- 命令：`/rewind`
- 快捷键：`Esc` + `Esc`

**操作选项**：

| 选项 | 作用 |
|------|------|
| **Restore code and conversation** | 代码和对话都回退到该点 |
| **Restore conversation** | 只回退对话，保留当前代码 |
| **Restore code** | 只回退代码，保留当前对话 |
| **Summarize from here** | 从该点开始压缩对话（节省上下文） |
| **Never mind** | 取消操作 |

**使用场景**：
- 发现方向错误，想回到某个节点重新开始
- 调试走入死胡同，需要回退
- 上下文满了，需要压缩之前的对话

> ⚠️ **注意**：只有 Claude 的 Edit/Write 操作会被跟踪回退，Bash 命令的文件修改无法回退

---

### `/branch` - 平行宇宙

**作用**：基于当前状态创建一个分支会话

**别名**：`/fork`（旧名）

**与 `--fork-session` 的区别**：

| | `/branch` | `--fork-session` |
|--|-----------|------------------|
| **使用时机** | 会话内实时使用 | CLI 启动时 |
| **触发方式** | `/branch [name]` | `claude --continue --fork-session` |
| **效果** | 创建新会话分支 | 创建新会话分支 |

**使用场景**：
- 基于当前进度尝试不同方案
- 探索性开发，想保留原路线
- 想要"另起炉灶"但不想丢掉之前的工作

---

### 核心对比

| | `/rewind` | `/branch` |
|--|-----------|-----------|
| **方向** | 向后（回到过去） | 向前（创建新分支） |
| **原会话** | 被修改（回退） | 保持不变 |
| **类比 Git** | `git reset --hard` | `git checkout -b` |
| **何时用** | 撤销错误、重新开始 | 尝试新方案、保留原路线 |

---

## 自动化

### 非交互模式
```bash
claude -p "解释这个项目"
claude -p "列出 API" --output-format json
```

### Auto Mode
```bash
claude --permission-mode auto -p "fix all lint errors"
```

### 批量处理
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file" --allowedTools "Edit,Bash(git commit *)"
done
```

---

## 常见失败模式

| 模式 | 解决 |
|------|------|
| 任务混杂 | `/clear` |
| 反复纠正 | `/clear` + 更好的提示 |
| CLAUDE.md 太长 | 无情删减 |
| 无法验证 | 始终提供测试/脚本/截图 |
| 无限探索 | 限定范围或用 subagents |

---

## 文档索引（70个文档）

### 核心
- `quickstart.md` - 快速入门
- `tools-reference.md` - 工具参考
- `commands.md` - 斜杠命令
- `cli-reference.md` - CLI参考

### 高级功能
- `sub-agents.md` - 子Agent
- `agent-teams.md` - Agent团队
- `hooks.md` / `hooks-guide.md` - 钩子
- `mcp.md` - Model Context Protocol
- `memory.md` - 记忆系统
- `skills.md` - 技能系统

### 实践
- `best-practices.md` - 最佳实践（✓ 已学）
- `common-workflows.md` - 常见工作流
- `how-claude-code-works.md` - 工作原理

---

## 四、Subagents（子 Agent）

### 核心概念

Subagent 是**运行在独立上下文中的专用 AI 助手**，有自己的系统提示、工具权限和模型配置。

**关键区别**：
- **Subagents**：单会话内使用，主 Agent 协调
- **Agent Teams**：跨会话并行运行，互相通信

### 内置 Subagents

| Subagent | 模型 | 工具 | 用途 |
|----------|------|------|------|
| **Explore** | Haiku | Read-only | 代码搜索、代码库探索 |
| **Plan** | 继承 | Read-only | Plan Mode 下的代码研究 |
| **general-purpose** | 继承 | 全部 | 复杂多步任务 |

### 创建 Subagent

**文件位置**（优先级从高到低）：
| 位置 | 范围 |
|------|------|
| `--agents` CLI flag | 当前会话 |
| `.claude/agents/` | 当前项目 |
| `~/.claude/agents/` | 所有项目 |

**文件格式**：
```markdown
---
name: code-reviewer
description: Reviews code for quality  # Claude 用这决定是否委托
tools: Read, Grep, Glob              # 允许的工具
model: sonnet                        # sonnet/opus/haiku/inherit
permissionMode: default              # default/acceptEdits/dontAsk/bypassPermissions/plan
memory: user                         # 持久记忆: user/project/local
background: false                    # 是否后台运行
---
You are a code reviewer. Analyze code and provide feedback...
```

### Frontmatter 字段

| 字段 | 说明 |
|------|------|
| `name` | 唯一标识符（必需） |
| `description` | Claude 用它决定是否委托（必需） |
| `tools` | 允许的工具（默认继承全部） |
| `disallowedTools` | 禁止的工具 |
| `model` | 模型选择 |
| `permissionMode` | 权限模式 |
| `skills` | 预加载的技能 |
| `mcpServers` | MCP 服务器 |
| `hooks` | 生命周期钩子 |
| `memory` | 跨会话记忆 |

### 调用方式

```text
# 1. 自然语言（Claude 自动决定）
Use a subagent to review this code for security issues

# 2. @-提及（确保使用特定 subagent）
@"code-reviewer (agent)" look at the auth changes

# 3. 整个会话作为 Subagent
claude --agent code-reviewer

# 4. 设置默认
# .claude/settings.json: { "agent": "code-reviewer" }
```

### 前后台运行

- **前台**：阻塞主会话，可交互权限提示
- **后台**：并发运行，需预授权权限

```text
run this in the background  # 让 Claude 后台运行
Ctrl+B                    # 将运行中任务转为后台
```

### 典型模式

| 场景 | 示例 |
|------|------|
| **隔离高输出** | `Use a subagent to run tests and report only failures` |
| **并行研究** | `Research auth, database, and API in parallel using subagents` |
| **链式调用** | `Use code-reviewer to find issues, then optimizer to fix them` |

**何时用 Subagent**：
- 产生大量输出（测试、日志）
- 需要强制工具限制
- 任务自包含，返回摘要

### 示例

**Code Reviewer**（只读）：
```yaml
name: code-reviewer
description: Expert code review specialist. Use proactively after writing code.
tools: Read, Grep, Glob, Bash  # 无 Edit/Write
```

**Debugger**（可修改）：
```yaml
name: debugger
description: Debugging specialist. Use proactively when issues occur.
tools: Read, Edit, Bash, Grep, Glob
```




## 五、MCP（Model Context Protocol）

### 核心概念

**MCP 是连接 Claude Code 与外部工具和数据的开放标准协议。**

通过 MCP，Claude 可以：
- 读取 Jira/GitHub issue
- 查询 PostgreSQL 数据库
- 获取 Sentry 错误报告
- 读取 Figma 设计
- 发送 Slack 消息

### 添加 MCP 服务器

**三种传输方式**：

| 方式 | 命令示例 |
|------|----------|
| **HTTP**（推荐） | `claude mcp add --transport http notion https://mcp.notion.com/mcp` |
| **stdio**（本地） | `claude mcp add --transport stdio db -- npx -y @bytebase/dbhub --dsn "postgresql://..."` |
| **SSE**（已弃用） | `claude mcp add --transport sse asana https://mcp.asana.com/sse` |

**注意**：选项必须放在服务器名之前，`--` 后才是服务器命令。

### 作用域

| 作用域 | 存储 | 场景 |
|--------|------|------|
| **local** (默认) | `~/.claude.json` | 个人、敏感凭证 |
| **project** | `.mcp.json` | 团队协作，可提交 git |
| **user** | `~/.claude.json` | 跨项目使用 |

```bash
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/
```

### 管理命令

```bash
claude mcp list          # 列出服务器
claude mcp get <name>    # 查看详情
claude mcp remove <name> # 删除
/mcp                     # 会话内查看状态和认证
```

### 认证

**OAuth 2.0**：
```bash
# 1. 添加服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 会话内认证
/mcp  # 选择服务器并完成浏览器登录
```

**预配置 OAuth**（非动态注册）：
```bash
claude mcp add --transport http --client-id ID --client-secret --callback-port 8080 my-server URL
```

### 配置示例

**.mcp.json**（支持环境变量展开）：
```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "api": {
      "type": "http",
      "url": "${API_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

### 使用

添加后直接用自然语言：
```text
What's the most common error in Sentry in the last 24 hours?
Show me open PRs assigned to me on GitHub
Query total revenue this month from PostgreSQL
```

**引用 MCP 资源**：
```text
Can you analyze @github:issue://123 and suggest a fix?
Compare @postgres:schema://users with @docs:file://database/user-model
```

### 高级功能

| 功能 | 说明 |
|------|------|
| **Tool Search** | MCP 工具超过上下文 10% 时自动按需加载。控制：`ENABLE_TOOL_SEARCH=auto:5` |
| **Channels** | MCP 服务器可向会话推送消息（CI结果、监控告警） |
| **Prompts** | MCP 暴露的 prompts 可作为命令 `/mcp__server__prompt` |
| **Elicitation** | 服务器可请求用户输入（表单或浏览器认证） |

### 环境变量

| 变量 | 作用 |
|------|------|
| `MAX_MCP_OUTPUT_TOKENS` | MCP 输出上限（默认 25000） |
| `ENABLE_TOOL_SEARCH` | 工具搜索：auto/true/false |
| `MCP_TIMEOUT` | MCP 服务器启动超时 |

---

## 六、Hooks（钩子）

### 核心概念

**Hooks 是在 Claude Code 生命周期特定点自动执行的命令/脚本。**

与依赖 LLM 决定是否执行不同，Hooks 是**确定性**的——只要条件匹配就一定会执行。

### Hook 类型

| 类型 | 用途 | 示例 |
|------|------|------|
| `command` | 执行 shell 命令 | 格式化代码、发送通知 |
| `http` | POST 到 HTTP 端点 | 团队审计服务 |
| `prompt` | 单轮 LLM 评估 | 需要判断力的决策 |
| `agent` | 多轮验证（可用工具） | 检查测试是否通过 |

### 生命周期事件

| 事件 | 触发时机 | 可阻断？ |
|------|----------|----------|
| `SessionStart` | 会话开始/恢复 | 否 |
| `UserPromptSubmit` | 用户提交提示 | 是 |
| `PreToolUse` | 工具执行前 | 是 |
| `PermissionRequest` | 权限请求时 | 是 |
| `PostToolUse` | 工具执行后 | 否 |
| `PostToolUseFailure` | 工具执行失败 | 否 |
| `Notification` | 发送通知时 | 否 |
| `SubagentStart/Stop` | 子 Agent 启动/停止 | Stop 可阻断 |
| `Stop` | Claude 完成响应 | 是 |
| `StopFailure` | API 错误结束 | 否 |
| `ConfigChange` | 配置变更 | 是 |
| `CwdChanged` | 工作目录变更 | 否 |
| `FileChanged` | 监视文件变更 | 否 |
| `PreCompact/PostCompact` | 上下文压缩前/后 | 否 |
| `SessionEnd` | 会话终止 | 否 |

### 配置位置

| 位置 | 范围 | 可共享 |
|------|------|--------|
| `~/.claude/settings.json` | 所有项目 | 否 |
| `.claude/settings.json` | 单个项目 | 是 |
| `.claude/settings.local.json` | 单个项目 | 否（gitignored） |
| Skill/Agent frontmatter | 组件激活时 | 是 |

### Matcher（匹配器）

用正则表达式过滤何时触发：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "prettier --write ..." }]
      }
    ]
  }
}
```

不同事件匹配不同字段：
- `PreToolUse/PostToolUse` → 匹配 tool 名（`Bash`, `Edit|Write`, `mcp__.*`）
- `SessionStart` → 匹配 source（`startup`, `resume`, `clear`, `compact`）
- `SessionEnd` → 匹配结束原因（`clear`, `logout`, `other`）
- `Notification` → 匹配通知类型
- `FileChanged` → 匹配文件名

### 退出码行为

| 退出码 | 含义 |
|--------|------|
| `0` | 成功，继续执行 |
| `2` | 阻断错误（依事件而定） |
| 其他 | 非阻断错误，记录日志 |

**可阻断的事件**：`PreToolUse`, `PermissionRequest`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`

### 常用示例

#### 1. 编辑后自动格式化
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

#### 2. 保护敏感文件
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

脚本检查文件路径，匹配保护模式时 `exit 2` 阻断。

#### 3. 自动批准特定权限
```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'"
          }
        ]
      }
    ]
  }
}
```

#### 4. 桌面通知（macOS）
```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

#### 5. Prompt-based Hook（判断力决策）
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if all tasks are complete. If not, respond with {\"ok\": false, \"reason\": \"what remains to be done\"}."
          }
        ]
      }
    ]
  }
}
```

#### 6. Agent-based Hook（验证后决策）
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results. $ARGUMENTS",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

### 环境变量

| 变量 | 说明 |
|------|------|
| `$CLAUDE_PROJECT_DIR` | 项目根目录（用于引用脚本） |
| `$CLAUDE_ENV_FILE` | 环境变量持久化文件（SessionStart/CwdChanged/FileChanged 可用） |
| `$CLAUDE_CODE_REMOTE` | 远程 web 环境为 `"true"` |

### 管理命令

- `/hooks` - 只读浏览已配置的 hooks
- `disableAllHooks: true` - 禁用所有 hooks

### HTTP Hooks

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/tool-use",
            "headers": { "Authorization": "Bearer $MY_TOKEN" },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  }
}
```

- 2xx 空响应 = 成功
- 2xx JSON 响应 = 按 JSON 格式解析决策
- 非 2xx = 非阻断错误

---

*笔记更新: 2026-03-26*
