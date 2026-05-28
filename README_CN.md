# AI 与工程学习笔记

[English](README.md) | **中文**

AI 工程化学习笔记——Claude Code 上下文工程、全栈开发和技术写作。

[![AI Engineering](https://img.shields.io/badge/AI-工程学习笔记-blue?style=for-the-badge)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

---

## 核心内容

### Claude Code 上下文工程 (`claude-code/`)

一套 **6 层架构**，用于构建生产级 Claude Code 工作流：

| 层级 | 涵盖什么 |
|------|----------|
| Layer 1 | 长期上下文——CLAUDE.md、Memory、规则、Token 预算 |
| Layer 2 | 工具能力——脚本、MCP、命名规范 |
| Layer 3 | 技能（工作流）——SOP、描述符优化 |
| Layer 4 | Hooks——确定性拦截层 |
| Layer 5 | Subagents——隔离 > 并行、权限控制 |
| Layer 6 | 验证闭环——L0-L5 分层验证 |

补充话题：Agent 核心循环、概念边界、Prompt Caching、高频命令。

**3 个真实案例** + 六维度 Harness 评分审计报告。

### 其他主题

- `ai/` — AI 学习路线、LangChain、RAG 模式
- `writing/` — 技术报告写作框架

## 目录结构

```
Learn/
├── claude-code/
│   ├── theory/       ← 6 层架构 + 补充话题（11 个文件）
│   ├── practice/     ← 实战案例与实操日志（4 个文件）
│   ├── strategy/     ← 工具策略与工作流优化
│   └── audit/        ← 配置审计报告与评分参考
├── ai/               ← AI/ML 学习笔记
└── writing/          ← 技术报告写作框架
```

## 许可证

MIT
