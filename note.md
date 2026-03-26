# Claude Code 文档结构观察

## 笔记更新策略
- Claude 会在每次回答后，视情况将重要知识点整理到此笔记
- 本文件是用户的学习笔记，记录核心概念和学习路径

## 文档来源
- 本地镜像: `claude-code-docs/` 目录
- 原始来源: https://docs.anthropic.com/en/docs/claude-code/
- 共 70 个文档文件

## 核心结构 (10大类别)

### 2. 核心功能
- commands.md (斜杠命令)
- tools-reference.md (工具参考: Read/Edit/Grep/Bash等)
- cli-reference.md (CLI命令行)
- permissions.md / permission-modes.md (权限系统)
- sandboxing.md (沙箱机制)

### 3. 高级功能 ⭐重点
- agent-teams.md, sub-agents.md (Agent协作)
- hooks.md / hooks-guide.md (钩子系统)
- mcp.md (Model Context Protocol)
- memory.md (记忆系统)
- skills.md (技能系统)
- plugins.md (插件系统)
- scheduled-tasks.md (定时任务)

### 4. 配置与个性化
- settings.md, env-vars.md, model-config.md
- keybindings.md, output-styles.md

### 5. IDE/编辑器集成
- vs-code.md, jetbrains.md, chrome.md
- desktop.md, remote-control.md

### 6. 平台与部署
- headless.md (无头模式)
- github-actions.md, gitlab-ci-cd.md
- devcontainer.md

### 7. 云服务提供商
- amazon-bedrock.md, google-vertex-ai.md
- microsoft-foundry.md, llm-gateway.md

### 8. 监控与分析
- analytics.md, monitoring-usage.md, costs.md

### 9. 安全与合规
- security.md, data-usage.md, authentication.md
- zero-data-retention.md

### 10. 其他
- troubleshooting.md, changelog.md
- best-practices.md, common-workflows.md

## 学习路径建议
1. 先看: quickstart.md + overview.md
2. 核心: tools-reference.md + commands.md
3. 进阶: sub-agents.md + hooks.md + memory.md + skills.md
4. 实践: best-practices.md + common-workflows.md
