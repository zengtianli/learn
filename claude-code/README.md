# Claude Code 工程实践知识库

基于实际项目（写作质量体系、钱塘江标书、VPS 基础设施）总结的 Claude Code 上下文工程方法论。

## 目录结构

```
claude-code/
├── theory/      ← 理论框架（6层架构 + 补充话题）
├── practice/    ← 实战案例与实操日志
├── strategy/    ← 工具策略与工作流优化
└── audit/       ← 配置审计报告与评分参考
```

## theory/ — 理论框架

建议阅读顺序：

| 顺序 | 文件 | 内容 |
|------|------|------|
| 1 | 00-总览与主线地图 | 三条主线全局视图 + 认知升级路径 |
| 2 | 01-底层运作原理 | CC 是 Agent：收集上下文 → 行动 → 验证 |
| 3 | 02-概念边界 | 8 个核心概念的精确定义 |
| 4 | Layer1-长期上下文 | CLAUDE.md / Memory / Rules + Token 分层加载 |
| 5 | Layer2-工具能力 | 脚本设计、命名规范、三级优先级 |
| 6 | Layer3-工作流Skills | Skill 类型、SOP 设计、描述符优化 |
| 7 | Layer4-Hooks | 确定性拦截层，4 个实战 Hook |
| 8 | Layer5-Subagents | 隔离 > 并行，七要素模板，权限控制 |
| 9 | Layer6-验证闭环 | L0-L5 分层验证，流水线设计 |
| 10 | Prompt-Caching | 前缀匹配机制，缓存命中率优化 |
| 11 | 高频命令 | /compact /clear /cost 等命令的工程意义 |

## practice/ — 实战案例

| 文件 | 项目 | 核心收获 |
|------|------|---------|
| 实战-写作质量体系 | 景宁水生态报告 | 4 维度框架（D1-D4）的诞生过程 |
| 实战-钱塘江标书全流程 | 95万技术标 | 12 章 subagent 并行写作 |
| 实战-VPS基础设施 | 服务器部署 | 裸机 → 24/7 AI 工作站 |
| 实战更新记录-2026-03-16 | — | 学完理论后的实操日志 |

## strategy/ — 工具策略

| 文件 | 内容 |
|------|------|
| augment-context-engineering | Augment 语义搜索替代 grep，3 个杠杆提升会话效率 |

## audit/ — 审计与评分

| 文件 | 内容 |
|------|------|
| health-check-2026-03-16 | ZDWP 项目六层框架审计报告 |
| harness-scoring-reference | D1-D6 六维度评分标准参考 |

## 命名规范

| 内容类型 | 目录 | 命名 |
|---------|------|------|
| 理论/框架 | theory/ | 描述性名称 |
| 实战案例 | practice/ | `实战-{项目名}.md` |
| 实操日志 | practice/ | `实战更新记录-{YYYY-MM-DD}.md` |
| 工具策略 | strategy/ | `{tool}-{topic}.md` |
| 审计报告 | audit/ | `health-check-{YYYY-MM-DD}.md` |
