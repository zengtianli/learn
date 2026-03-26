# AI 与工程学习笔记

[English](README.md) | **中文**

AI 工程化学习笔记——Claude Code 上下文工程、全栈开发和技术写作。

[![AI Engineering](https://img.shields.io/badge/AI-工程学习笔记-blue?style=for-the-badge)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

---

## 核心内容

### Claude Code 上下文工程 (`claude-code/`)

一套 **6 层架构**，用于构建生产级 Claude Code 工作流：

| 层级 | 内容 | 涵盖什么 |
|------|------|----------|
| Layer 1 | 长期上下文 | CLAUDE.md、Memory、规则——Claude 的"性格设定" |
| Layer 2 | 工具能力 | 脚本、MCP Server、内置工具——Claude *能做什么* |
| Layer 3 | 技能（工作流） | 斜杠命令、SOP——*怎么做*重复性任务 |
| Layer 4 | Hooks | 自动强制执行——"必须做"的层 |
| Layer 5 | Subagents | 并行工作者——带隔离的扩展 |
| Layer 6 | 验证闭环 | 测试、验证——"完成"有可检验的定义 |

另有 3 个真实案例：
- **写作质量体系** — docx 格式检查流水线
- **钱塘江标书全流程** — 12 章技术标书，多 Agent 并行写作
- **VPS 基础设施** — 用 Claude Code 自动化服务器搭建

### 其他主题

- `ai/` — AI 学习路线、LangChain、RAG 模式
- `fullstack/` — 全栈开发笔记
- `writing/` — 技术报告写作框架
- `leveling/` — 技能升级追踪

## 许可证

MIT
