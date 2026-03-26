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

---


*笔记更新: 2026-03-26*
