# Layer 2：工具能力层

> **在六层架构中的位置**：第二层，告诉 Claude "能做什么"。通过脚本、MCP Server 或内置工具，让 CC 具备执行实际操作的能力。
>
> **前置阅读**：`Layer1-长期上下文.md`（Layer 1 定义"是什么"，Layer 2 建立在 Layer 1 之上）
>
> **关联层**：Layer 3（Skills）会引用 Layer 2 的工具来定义工作流。工具是"手脚"，Skill 是"肌肉记忆"。

---

## 一、一句话理解

**工具层 = CC 的"手脚"。** CC 本身只能思考和写文本。要和外部世界交互（读文件、调 API、处理文档），必须通过工具。工具层决定了 CC 能做什么、做得多快、做得多准。

<!-- 批注区 -->

---

## 二、脚本作为工具的设计原则

你在 `~/Dev/scripts/scripts/document/` 下积累了 30+ 个脚本。这些脚本通过 Bash 工具调用，本质上就是 CC 的"工具库"。

### 2.1 好工具的标准

| 标准 | 好的例子 | 坏的例子 |
|------|---------|---------|
| **做一件事** | `docx_extract.py`（只管提取） | `docx_all_in_one.py`（提取+审阅+写入混在一起） |
| **名字自解释** | `docx_format_check.py` | `process.py`、`run.py` |
| **CLI 接口清晰** | `--split-chapters`、`--preview` | 需要改代码里的变量才能换模式 |
| **输出面向 CC 优化** | 错误信息说"缺少文件 X，请先运行 Y" | 抛一个 Python traceback |
| **双模式（check/fix）** | `bid_standardize.py --check` / `--fix` | 只有 fix，不能先预览 |
| **退出码规范** | 有问题返回 1，无问题返回 0 | 不管对错都返回 0 |

### 2.2 工具边界原则

每个脚本只做一件事，这样：
- 出了问题知道是哪个脚本的锅
- 可以独立迭代——改了格式检查不影响内容提取
- 可以自由组合——流水线中按需串联

```
docx_format_check.py   只管格式——不碰内容
docx_extract.py        只管提取——不碰格式
docx_track_changes.py  只管修订标记——不碰字体和排版
review_summary.py      只管规则合并——不碰文档
```

<!-- 批注区 -->

---

## 三、命名空间

你的脚本按照 `{格式}_{动作}.py` 的命名规范组织，形成了几个命名空间：

### 3.1 文档处理（docx_*）

| 脚本 | 功能 | 常用参数 |
|------|------|---------|
| `docx_extract.py` | 提取文本（MD/JSON/分章节） | `--split-chapters`、`--info` |
| `docx_format_check.py` | 格式快照与对比 | `snapshot`、`compare` |
| `docx_track_changes.py` | 读写修订标记 | `read`、`review --rules` |
| `docx_style_cleanup.py` | 样式重命名/合并/删除 | `--preview`、`--clean` |

### 3.2 标书处理（bid_*）

| 脚本 | 功能 | 常用参数 |
|------|------|---------|
| `bid_standardize.py` | 6 项结构标准化 | `--check`、`--fix`、`--scoring` |
| `bullet_to_paragraph.py` | Bullet point 转段落/表格 | `--output-dir` |

### 3.3 图表生成（chart_*）

| 脚本 | 功能 | 常用参数 |
|------|------|---------|
| `chart_common.py` | 公共模块（字体、配色） | 不直接调用 |
| `chart_gantt.py` | 甘特图生成 | JSON 配置 → PNG |
| `chart_bar.py` | 柱状图/条形图生成 | JSON 配置 → PNG |
| `chart_flow.py` | 流程图/架构图生成 | JSON 配置 → PNG |
| `chart_insert.py` | ASCII art 替换为 PNG 引用 | `--charts-dir` |

### 3.4 通用工具

| 脚本 | 功能 | 常用参数 |
|------|------|---------|
| `report_quality_check.py` | 质量检查 + 自动修复 | `--bid`、`--fix` |
| `table_name_check.py` | 表格是否有表名 | `--fix`、`--min-intro` |
| `fix_table_order.py` | 修复表格编号顺序 | — |
| `md_to_docx.py` | MD 转 DOCX | `-t template.docx` |
| `md_split_by_heading.py` | 按标题拆分 MD | `--level` |
| `review_summary.py` | 规则合并去重与摘要 | `--merge` |

### 3.5 命名规范的价值

命名空间让 CC 能快速定位工具。当 CC 需要"检查表格有没有表名"时，它会在 `table_` 命名空间中找到 `table_name_check.py`。如果你的脚本叫 `check3.py`、`util_v2.py`，CC 根本不知道该用哪个。

**命名规范**：`{领域}_{动作}.py`
- 领域：docx、bid、chart、table、md、report
- 动作：extract、check、fix、insert、standardize、cleanup

<!-- 批注区 -->

---

## 四、工具链：脚本之间的协作

单个脚本解决单个问题。多个脚本串联起来，解决完整的工作流。

### 4.1 审阅工具链

```
docx_format_check.py snapshot    → 格式基线
        ↓
docx_extract.py --split-chapters → 分章节 MD
        ↓
CC 按 D1-D4 审阅                 → rules_chN.json
        ↓
review_summary.py --merge        → merged_rules.json
        ↓
docx_track_changes.py review     → 带修订的 output.docx
        ↓
docx_format_check.py compare     → 格式验证
```

这条链路在 `docx-review` Skill 中有完整定义（见 `Layer3-工作流Skills.md`）。

### 4.2 标书后处理工具链

```
md/（subagent 初稿）
  ↓ bullet_to_paragraph.py
md_clean/（无 bullet 版本）
  ↓ report_quality_check.py --bid --fix
  ↓ bid_standardize.py --fix
md_final/（结构标准化版本）
  ↓ table_name_check.py + subagent 填表名
  ↓ chart_insert.py
  ↓ check_sensitive_words.py
  ↓ md_to_docx.py
merged.docx
```

这条链路在 `实战-钱塘江标书全流程.md` 第九章有完整命令序列。

### 4.3 工具链设计原则

1. **每一步可独立运行**：中间产物可以单独检查，出了问题知道是哪一步
2. **中间产物持久化**：`md/` → `md_clean/` → `md_final/`，每个阶段都有独立目录，可以回退
3. **先 check 再 fix**：先看报告了解有多少问题，确认脚本理解正确后再自动修复

<!-- 批注区 -->

---

## 五、MCP 的 Token 成本

tw93 建议把常用工具注册为 MCP Server，这样 CC 在工具列表里能直接看到。但这不是免费的。

**每个 MCP Server 约 4-6K tokens 的上下文开销。** 这意味着：

| MCP 数量 | 固定 token 开销 | 占 200K 总量 |
|---------|----------------|-------------|
| 0 个 | 0K | 0% |
| 1 个 | ~5K | 2.5% |
| 3 个 | ~15K | 7.5% |
| 5 个 | ~25K | 12.5% |

**你的现状**：通过 Skill 告诉 CC 工具链的调用方式（脚本路径 + 参数），不用 MCP Server。这个策略是对的——脚本不占上下文空间，Skill 只在触发时加载。

**什么时候该考虑 MCP**：
- 如果 CC 反复忘记某个脚本的用法或参数
- 如果脚本需要复杂的认证或状态管理
- 如果你需要非 Bash 的工具协议（比如直接调用 REST API）

**当前结论**：暂不需要 MCP。你的"脚本 > Skill > CC 直接"三级优先级已经够用。

<!-- 批注区 -->

---

## 六、好工具 vs 坏工具的标准

### 6.1 输出面向 CC 优化

**坏的输出**（原始 Python traceback）：
```
Traceback (most recent call last):
  File "script.py", line 42, in <module>
    with open(path) as f:
FileNotFoundError: [Errno 2] No such file or directory: '/tmp/missing.docx'
```

CC 看到这个输出会做什么？它知道文件不存在，但不知道应该怎么修复。

**好的输出**：
```
错误：找不到文件 /tmp/missing.docx
建议：请先确认文件路径是否正确，或者先运行 docx_extract.py 生成文件
```

好的错误信息面向 CC 优化——告诉它**该怎么修复**，而不只是**出了什么错**。

### 6.2 输出长度控制

CC 的 context 窗口是有限的。如果一个脚本输出 500 行日志，全部进入 CC 的上下文，挤占了有用信息的空间。

**原则**：脚本输出控制在 30 行以内。超过的部分用摘要代替。

```python
# 好的做法：只输出摘要
print(f"检查完成：{total} 个文件，{errors} 个问题")
if errors > 0:
    for err in error_list[:5]:  # 只展示前 5 个
        print(f"  - {err}")
    if len(error_list) > 5:
        print(f"  ... 还有 {len(error_list) - 5} 个问题，详见 report.md")
```

### 6.3 返回值与 CC 决策相关

脚本的输出应该帮助 CC 做决策，而不是返回原始数据。

| 坏的返回 | 好的返回 |
|---------|---------|
| 500 行 JSON 原始数据 | "发现 3 个禁用词：确保(2处)、我们(1处)" |
| 完整的 diff 输出 | "5 个文件有变化，其中 2 个是预期内的" |
| 二进制文件内容 | "PNG 图片已生成：800x600，文件大小 45KB" |

### 6.4 支持 `response_format`

tw93 建议工具支持 `concise` / `detailed` 两种输出模式：

```bash
# 简洁模式：给 CC 看的
python3 script.py --brief
# 输出：检查通过，0 个问题

# 详细模式：给人看的
python3 script.py --verbose
# 输出：完整的检查报告
```

你的核心脚本目前还没有这个功能，但值得考虑加上（见 TODO）。

<!-- 批注区 -->

---

## 七、我们的 30+ 脚本现状

### 7.1 做得好的地方

- **功能明确**：每个脚本做一件事，职责清晰
- **命名规范**：`{领域}_{动作}.py`，CC 能通过名字猜出功能
- **CLI 接口**：所有脚本都可以通过 Bash 直接调用
- **工具链形成**：提取 → 审阅 → 写入 → 验证，形成了完整的流水线
- **双模式设计**：大部分脚本支持 check（只看）和 fix（修复）

### 7.2 不够好的地方

- **输出格式不统一**：有的输出纯文本，有的输出 JSON，有的混合。CC 需要针对不同格式做解析
- **错误信息不面向 CC**：有些脚本出错时直接抛 Python traceback，CC 不知道怎么修复
- **没有 `--brief` 选项**：所有脚本的输出量都是固定的，不能根据场景调整
- **隐式工具**：这些脚本是通过 Bash 调用的"隐式工具"，CC 需要知道脚本路径和参数才能用。Skill 里写了调用方法，但如果 Skill 没被触发，CC 可能不知道有这些工具

### 7.3 三级优先级

你在实践中形成了一个重要原则：

```
优先级 1（最高）：Python 脚本  — 确定性、可重复、不消耗 token
优先级 2：       Skill（CC + 约束）— 需要 AI 判断但有框架约束
优先级 3（最低）：CC 直接对话  — 最灵活但最贵、最不可控
```

这个原则本质上就是在管理工具层：
- 能用脚本的不靠 CC——脚本不会忘、不会跑偏、不花 API 费用
- 能用 Skill 约束的不靠 CC 临场发挥——Skill 是标准化的 SOP
- 只有真正一次性的讨论才让 CC 自由发挥

<!-- 批注区 -->

---

## 八、常见误区

这层**不该做**什么：

- **把业务逻辑写进工具**：工具只做"操作"（提取、写入、格式化），业务判断（该不该改、怎么改）留给 CC 或 Skill
- **返回大量原始数据**：比如一个 API 返回 500 行 JSON，CC 需要的可能只是 3 个字段——工具应该预处理
- **工具名称模糊**：`query()`、`process()`、`run()` 都是坏名字；`docx_extract`、`bid_standardize` 才是好名字
- **一个工具做太多事**：`docx_all_in_one.py` 不如拆成提取、审阅、写入三个独立脚本

---

## 九、TODO

- [ ] **核心脚本加 `--brief` 选项**（P1）：减少输出噪声，节省 CC 的上下文空间
- [ ] **脚本错误信息面向 CC 优化**（P2）：告诉 CC 该怎么修复，而不只是出了什么错
- [ ] **考虑统一输出格式**（P2）：核心脚本的输出统一为 JSON + 人类可读摘要双模式
- [ ] **如果 CC 反复忘记某个脚本的用法**，考虑封装成 MCP Server（P3）

<!-- 批注区：你觉得哪些脚本最常被 CC "忘记"？那些可能是 MCP 的候选。 -->

---

## 参考

- `Layer1-长期上下文.md` — Layer 1 定义了"是什么"，Layer 2 建立在其上
- `Layer3-工作流Skills.md` — Layer 3 使用 Layer 2 的工具定义工作流
- `实战-写作质量体系.md` 第四章 — 审阅工具链实战
- `实战-钱塘江标书全流程.md` 第九章 — 标书工具链全景图
- `~/Dev/scripts/scripts/document/` — 你的工具库（实物）
