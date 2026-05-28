# Claude Code 健康检查报告

**项目**：~/Work/zdwp/（浙江水资源项目 ZDWP）
**日期**：2026-03-16
**工具**：tw93/claude-health v1.5.0
**审计层级**：六层框架 `CLAUDE.md -> rules -> skills -> hooks -> subagents -> verifiers`

---

## 项目分级判定

| 指标 | 实际值 | 说明 |
|------|--------|------|
| 源代码文件数 | 5,133 | >5K，触达 Complex 门槛 |
| 贡献者数 | 1 | 单人项目 |
| CI 工作流 | 0 | 无 CI |
| Skills 数量 | 16（本地）+ 23（全局）= 39 | 大量 |
| CLAUDE.md 行数 | 104（本地）| 中等 |
| 会话文件数 | 11 | 活跃项目 |

**判定结果：STANDARD 级别**

虽然源文件数触达 Complex 门槛（>5K），但单人贡献、无 CI，实际复杂度为 Standard。以下按 Standard 级别审查。

---

## 启动上下文预算分析

| 项目 | 词数 | 估算 Token |
|------|------|-----------|
| 全局 CLAUDE.md | 341 | ~443 |
| 本地 CLAUDE.md | 291 | ~378 |
| rules/ 文件 | 0 | 0 |
| Skill 描述词数 | 134 | ~174 |
| **小计** | 766 | ~996 |
| x1.3 安全系数 | — | ~1,295 |
| MCP Token 开销 | 0 | 0 |
| **总启动上下文** | — | **~1,295 tokens** |

结论：启动上下文极轻量（1,295 / 200,000 = 0.6%），远低于 30K 阈值。**无问题。**

---

## 检查结果汇总

### :red_circle: Critical -- 立即修复

#### C1. 全局 CLAUDE.md 包含硬编码 API Token

**文件**：`~/.claude/CLAUDE.md` 第 82 行

```
- Auth Token: `sk-314b13f94f1aeba82a992b54e0100734827647a7a14a9a4956ca1947842ac6cf`
```

**风险**：CLAUDE.md 会被注入到每次会话的系统 prompt 中。虽然 `~/.claude/CLAUDE.md` 不在 git 仓库中，但该 token 会出现在：
- 每次 API 请求的 prompt 中（通过中转站传输）
- 任何 conversation export 中
- 可能被 subagent 读取

**修复建议**：
- 从 `~/.claude/CLAUDE.md` 中移除 Auth Token 的明文值
- 改为引用环境变量名：`Auth Token: 见环境变量 ANTHROPIC_AUTH_TOKEN`
- 确保 token 只通过 `.zshrc` 或 `.env` 文件设置

#### C2. settings.local.json 未加入 .gitignore

**文件**：`~/Work/zdwp/.gitignore`

虽然当前 `settings.local.json` 文件不存在（因此不构成即时泄漏），但一旦创建（例如配置 hooks、MCP 等），就有被提交到 git 的风险——其中可能包含 API token 和个人路径。

**修复建议**：
在 `~/Work/zdwp/.gitignore` 中添加：
```
# Claude Code
.claude/settings.local.json
```

#### C3. 3 个嵌套 CLAUDE.md 文件造成不可预测的上下文叠加

**发现位置**：
1. `projects/eco-flow/lishui-景宁/New Folder With Items/CLAUDE.md`（30 行，详细的子项目配置）
2. `projects/reclaimed-water/huzhou-吴兴/docs/02_技术方法/CLAUDE.md`（详细的技术方法文档）
3. `钱塘江流域（杭州部分）取用水监测计量能力提升重点项目/CLAUDE.md`（标书项目配置）

**风险**：当 Claude Code 在这些子目录中操作时，会叠加 全局 CLAUDE.md + 根 CLAUDE.md + 子目录 CLAUDE.md，产生不可预测的行为。特别是：
- 景宁的 CLAUDE.md 包含项目特有的工作流（替换规则等），可能与根 CLAUDE.md 冲突
- "New Folder With Items" 这个目录名本身就暗示临时性
- 钱塘江项目的 CLAUDE.md 包含完整的优先级体系，可能与全局规则冲突

**修复建议**：
- 将子项目特有信息迁移到对应的 skill 中（例如 `zdwp-eco-flow` skill 的 references/ 目录）
- 或将这些 CLAUDE.md 重命名为 `PROJECT_NOTES.md`，仅作参考而非指令

---

### :yellow_circle: Structural -- 尽快修复

#### S1. 无 Hooks 配置

**现状**：`settings.local.json` 文件不存在，因此没有任何 hooks。

**影响**：Standard 级别的项目应该有 PostToolUse hooks 用于主要语言的即时检查。ZDWP 项目涉及 Python（5000+ 文件中大量 .py），但没有任何自动化检查。

**修复建议**：
创建 `~/Work/zdwp/.claude/settings.local.json`：
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_TOOL_INPUT_FILE_PATH\" == *.py ]]; then /Users/tianli/miniforge3/bin/python3 -m py_compile \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>&1 | head -5 || echo 'SYNTAX CHECK FAILED'; fi"
          }
        ]
      }
    ]
  }
}
```

#### S2. rules/ 目录为空

**现状**：`~/Work/zdwp/.claude/rules/` 不存在或无 .md 文件。

**影响**：Standard 级别建议将特定语言/路径的规则从 CLAUDE.md 中拆分到 rules/ 目录。当前的本地 CLAUDE.md 包含了 Python 环境路径、OA 系统命令、风险图项目要点等混合内容。

**修复建议**：
- 创建 `~/Work/zdwp/.claude/rules/python.md`：Python 环境和脚本规范
- 创建 `~/Work/zdwp/.claude/rules/oa-system.md`：OA 系统相关命令
- 将 CLAUDE.md 中的业务细节（风险图、QGIS 工作流等）迁移到对应 skill 中（大部分已做到）

#### S3. 全局和本地存在重复 Skill：plan-first

**发现**：
- 全局：`~/.claude/skills/plan-first/SKILL.md`（484 词）
- 本地：`~/Work/zdwp/.claude/skills/zdwp-plan-first/SKILL.md`（245 词）

两个 skill 的 description 完全相同："先写方案再操作规范。当执行批量操作、文件整理、目录重构等破坏性操作时触发。"

**影响**：Claude 可能同时加载两个 skill，浪费上下文空间。

**修复建议**：
- 保留全局 `plan-first`（更完整，484 词）
- 删除本地 `zdwp-plan-first`，或将其改为仅包含 zdwp 项目特有的补充规则

#### S4. 全局 Skills 中混入项目特有 Skill

**发现**：`~/.claude/skills/zdwp-interactive-organize/SKILL.md` 是一个 zdwp 项目特有的 skill，但放在了全局 skills 目录中。

**影响**：在非 zdwp 项目中也会被加载到 skill 描述中，浪费上下文。

**修复建议**：
将 `~/.claude/skills/zdwp-interactive-organize/` 移到 `~/Work/zdwp/.claude/skills/zdwp-interactive-organize/`

#### S5. 8 个全局 Skill 缺少 YAML Frontmatter

以下 skills 缺少标准的 YAML frontmatter（name, description, version）：

| Skill | 路径 |
|-------|------|
| website-dev | `~/.claude/skills/website-dev/SKILL.md` |
| zdwp-interactive-organize | `~/.claude/skills/zdwp-interactive-organize/SKILL.md` |
| report-writing | `~/.claude/skills/report-writing/SKILL.md` |
| docx-review | `~/.claude/skills/docx-review/SKILL.md` |
| resume-ops | `~/.claude/skills/resume-ops/SKILL.md` |
| streamlit-数据分析应用开发工作流 | `~/.claude/skills/streamlit-数据分析应用开发工作流/SKILL.md` |
| claude-cursor-config-sync | `~/.claude/skills/claude-cursor-config-sync/SKILL.md` |
| claude-code-advanced-workflow | `~/.claude/skills/claude-code-advanced-workflow/SKILL.md` |

**影响**：没有 frontmatter 意味着 Claude 无法正确解析 skill 的元数据（name、version、disable-model-invocation），可能导致错误的 skill 匹配。

**修复建议**：
为每个 skill 添加标准 frontmatter，参考格式：
```yaml
---
name: skill-name
description: 一句话描述。当 xxx 时触发。
version: "1.0.0"
---
```

#### S6. 所有 Skill 均未设置 disable-model-invocation: true

**现状**：除了 health skill 自身外，39 个 skills 中没有任何一个设置了 `disable-model-invocation: true`。

**影响**：低频使用的 skill（如 career、learn-fullstack、learn-ai、learn-solutions 等）在每次会话中都可能被模型主动调用，消耗不必要的上下文。

**修复建议**：
为以下低频 skills 添加 `disable-model-invocation: true`：
- `career`、`job-match`、`resume-ops` —— 求职相关，非日常
- `learn-fullstack`、`learn-ai`、`learn-solutions`、`learn-tracker` —— 学习相关
- `essay-writer` —— 论文相关
- `claude-cursor-config-sync` —— 配置同步
- `website-dev` —— 个人网站开发
- `skill-standard` —— 创建 skill 时才用

#### S7. 本地 CLAUDE.md 缺少 Verification 节

**现状**：本地 CLAUDE.md 没有 Verification / 验证 章节，没有定义任务完成条件。

**影响**：Standard 级别项目应该有明确的 done-conditions。当前 CLAUDE.md 定义了 "交付物必须包含" 的规则，但没有对应的验证命令。

**修复建议**：
在本地 CLAUDE.md 中添加：
```markdown
## 验证规则

- Python 脚本修改后：`/Users/tianli/miniforge3/bin/python3 -m py_compile <file>`
- 交付前检查：成果文件 + 成果说明.md + 数据来源
- 文件命名验证：`[日期]_[业务]_[内容]_[版本].[后缀]` 格式
```

#### S8. 无 HANDOFF.md 且无跨会话交接机制

**现状**：项目没有 HANDOFF.md 文件。

**影响**：虽然 MEMORY.md 已经记录了不少项目决策和学习状态，但缺少当前正在进行的工作的交接信息。11 个会话文件表明这是活跃项目，跨会话丢失上下文的风险存在。

**修复建议**：
- 全局 CLAUDE.md 已有 Compact Instructions 来缓解这个问题
- 可选：创建 `~/Work/zdwp/HANDOFF.md` 记录当前活跃的工作项和进度

#### S9. 三层防御一致性不足

以下关键规则只存在于单一层，缺乏纵深防御：

| 规则 | CLAUDE.md（意图层） | Skill（知识层） | Hook（控制层） |
|------|:---:|:---:|:---:|
| 交付物必须包含成果说明.md | 有 | 无 | 无 |
| 文件命名规范 | 有 | 有（zdwp-structure） | 无 |
| Python 环境路径 | 有 | 无 | 无 |
| 先写方案再操作 | 有 | 有（plan-first x2） | 无 |
| 破坏性操作前确认 | 有（全局） | 有（plan-first） | 无 |

**关键缺口**：没有任何 hook 来强制执行这些规则。全部依赖 Claude 的自律性，在上下文压力大时可能被忽略。

---

### :green_circle: Incremental -- 可选改进

#### I1. 全局 CLAUDE.md 过于庞大，包含过多解释性文本

**现状**：全局 CLAUDE.md 包含 341 词（~443 tokens），虽然绝对值不高，但其中包含：
- 协作模式的详细描述（"教练 + 助手" 理念）
- 禁止行为的重复列举（7 条）
- 路径规范的完整表格（包含所有项目路径）
- API 配置的详细说明

**建议**：
- 将 "协作模式" 和 "行为规范" 的详细说明移到 `~/.claude/standards/` 中，CLAUDE.md 只保留简短的引用
- 路径规范可以移到 `architecture` skill 中（已有类似内容）
- 减少重复：全局 CLAUDE.md 和 `user-preferences` skill 有部分重叠

#### I2. 3 个轻量级 Skill 内容过少

以下 skills 只有 41 词（仅 frontmatter + 占位内容）：
- `zdwp-shoreline`
- `zdwp-water-rights`
- `zdwp-reclaimed-water`

**建议**：
- 如果这些项目还不活跃，可以暂时删除，待真正需要时再创建
- 或补充项目的关键知识（数据来源、处理方法、交付格式等）

#### I3. Skill 描述可以更精炼

以下 skill 描述超过 12 词，可能导致匹配过于宽泛：

- `zdwp-eco-flow`：31 词（"生态流量保障项目知识。当处理河湖生态流量计算、Tennant法、保障方案、水库筛选排查时触发。"）
- `zdwp-architecture`：24 词
- `oa-ops`：22 词

**建议**：将描述控制在 12 词以内，关键触发词放在 description 中，详细说明放在 SKILL.md 正文。

#### I4. Skill 来源追踪

- `health` skill 是唯一的符号链接，指向 `/Users/tianli/.agents/skills/health`（来自 tw93/claude-health 项目）
- 其余 38 个 skills 均为直接创建，无 git 来源信息
- 多个 skill 缺少 `version` 字段

**建议**：
- 为所有 skill 的 frontmatter 添加 version 字段（便于追踪变更）
- 考虑在自建 skills 的 frontmatter 中添加 `author` 或 `source` 字段

#### I5. OA 系统使用 Next.js 与全局规则冲突

**发现**：本地 CLAUDE.md 中 `.oa/` 目录使用 Next.js 14 + Tailwind 构建，但全局 CLAUDE.md 规定 "Web 应用统一用 Streamlit"，且 "OA 系统：Streamlit，端口 3000"。

这是一个历史遗留的技术栈矛盾——本地 CLAUDE.md 反映了实际情况（Next.js），全局 CLAUDE.md 可能是后来更新的规范。

**建议**：
- 统一口径：如果 OA 已确定用 Streamlit 重写，从本地 CLAUDE.md 中移除 Next.js 相关说明
- 如果 .oa/ 仍在使用 Next.js，在全局 CLAUDE.md 中标注例外

#### I6. MEMORY.md 质量良好

**正面发现**：
- ZDWP 项目的 MEMORY.md 非常充实，包含报告写作体系、工具链、公文规范、用户学习状态等
- 使用了分链接文件（`reference_*.md`、`feedback_*.md`、`user_*.md`、`project_*.md`）来组织不同类型的记忆
- 这是一个值得推广到其他项目的最佳实践

---

## 跳过的检查项

| 检查项 | 原因 |
|--------|------|
| MCP Token 开销 | 无 MCP 配置 |
| allowedTools 检查 | 无 settings.local.json |
| Prompt Cache 检查 | 无 hooks/动态内容注入 |
| 行为模式审计（对话分析） | 未提取对话内容（数据量大，非本次重点） |
| Subagents 卫生检查 | 无 hooks 中的 Agent 调用 |
| CI/CD 集成检查 | 无 CI 工作流 |

---

## 修复优先级路线图

### Phase 1: 立即（今天）
1. **C1** 从全局 CLAUDE.md 移除硬编码 API Token
2. **C2** 在 .gitignore 中添加 settings.local.json
3. **C3** 处理嵌套 CLAUDE.md（重命名或迁移内容到 skills）

### Phase 2: 本周
4. **S1** 创建 settings.local.json，配置 Python 语法检查 hook
5. **S3** 合并重复的 plan-first skill
6. **S4** 将 zdwp-interactive-organize 移到本地 skills
7. **S5** 为 8 个 skill 补充 frontmatter

### Phase 3: 本月
8. **S6** 为低频 skills 设置 disable-model-invocation
9. **S7** 添加 Verification 节到本地 CLAUDE.md
10. **S9** 逐步添加 hooks 实现三层防御
11. **I1** 精简全局 CLAUDE.md
12. **I5** 统一 OA 技术栈描述

---

## 安全扫描总结

对全部 39 个 skills 进行了安全扫描（prompt injection、data exfiltration、destructive commands、hardcoded credentials、obfuscation、safety override），**未发现任何安全问题**。所有 skills 均为干净的知识型内容。

---

> 需要我起草具体的修复代码吗？可以按层分别处理：全局 CLAUDE.md / 本地 CLAUDE.md / hooks / skills。
