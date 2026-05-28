# Harness 六维度评分参考

> 工作 Skill 位置：`~/.agents/skills/health/SKILL.md`（v2.0.0）
> 评分记录追加到：`~/.claude/plans/harness-score-card.md`

## 项目分级

| 级别 | 信号 | 要求 |
|------|------|------|
| **Simple** | <500 源文件、1 人、无 CI | 只需 CLAUDE.md；0-1 skills；hooks 可选 |
| **Standard** | 500-5K 文件、小团队或有 CI | CLAUDE.md + 1-2 rules；2-4 skills；基本 hooks |
| **Complex** | >5K 文件、多贡献者、多语言、活跃 CI | 完整六层配置 |

## 六维度定义（每项 0-10 分）

### D1 Context Engineering
- Token 占用 <10%
- 规则无重复（全局/本地不冲突）
- 无孤立 standards
- MEMORY.md 已更新
- Compact Instructions 结构化
- Scratchpad 规则存在

### D2 Hooks 系统
- PreToolUse 存在
- PostToolUse 覆盖主要语言
- Notification 存在
- 脚本可执行
- 无误拦截

### D3 Sub-agent 模式
- Skills 数量 <20
- Agent 职责不重叠
- agent-dispatch.md 存在
- 委派/不委派规则清晰
- 启动清单完整

### D4 评估与验证
- Verification 表覆盖所有任务类型
- 自动化验证 >70%
- 生成者与验证者分离

### D5 会话管理
- Compact Instructions 结构化
- Scratchpad 规则
- Handoff 格式标准化
- Context Budget 规范存在
- 方案模板标准化

### D6 文件结构
- 零孤立文件
- 分层完整
- 引用链 100%
- 命名规范

## 评分表格式

```markdown
## Harness Score Card — {日期}

| 维度 | 分数 | 变化 | 关键发现 |
|------|------|------|---------|
| D1 Context | X/10 | ±N | ... |
| D2 Hooks | X/10 | ±N | ... |
| D3 Agents | X/10 | ±N | ... |
| D4 验证 | X/10 | ±N | ... |
| D5 会话 | X/10 | ±N | ... |
| D6 结构 | X/10 | ±N | ... |
| **总分** | **X/10** | **±N** | |
```

## 审计报告结构

报告按严重程度分三级：
- **Critical** — 立即修复（安全漏洞、凭证泄露、配置冲突）
- **Structural** — 尽快修复（缺失的 hooks/rules/verification、重复 skills）
- **Incremental** — 可选改进（精简描述、添加 version、优化上下文）
