# Layer 6：验证闭环实战教学

> 这篇教学专门展开六层框架中的第六层——验证闭环。
> 目标：把你手头已有的零散验证工具，组织成一套分层验证体系，让"做完"有可检验的定义。

---

## 阅读指引

**这篇文档解决什么问题**：你已经有了 `docx_format_check.py`、`table_name_check.py` 等验证工具，但它们各自为战——什么时候该跑哪个、跑完怎么判断通过、怎么串起来形成流水线，目前没有体系。

**读完你会获得**：
1. 一个分层验证框架，知道每个工具属于哪个层级
2. 验收标准的写法，让"做完"变成可检查的清单
3. 一个验证流水线的设计模板，把零散工具串成一条线
4. Hook + 流水线的联动思路，实时验证 + 交付前终检

**建议阅读方式**：先通读一遍建立全貌，然后在下一个标书项目中实际跑一遍验证流水线。

---

## 一、核心原则

**如果无法说清什么叫做完，就不适合自动完成。**

这句话的意思是：验收标准要提前写进 Prompt、Skill 或 CLAUDE.md，不是事后补。

为什么？因为 CC 是"执行力很强但判断力有限"的协作者。你不告诉它什么叫合格，它就只能猜——猜对了是运气，猜错了浪费你的时间。

举个例子。你让 CC 写标书第 3 章，如果 Prompt 里没写验收标准，CC 写完就会说"已完成第 3 章的撰写"。但你心里的"完成"可能是：

- 3000 字以上
- 0 个 bullet point
- 所有表格有表名
- 没有"确保""我们"等禁用词
- 引用了评分标准中的每个得分点

这些标准不写出来，CC 不知道，你事后检查也容易遗漏。

**原则**：验收标准是任务定义的一部分，不是质检环节的事。

---

## 二、分层验证框架

验证不是一锅煮。不同层级的检查，自动化程度不同、发现问题的成本不同、修复的难度也不同。

| 层级 | 验证内容 | 工具 | 自动化程度 | 修复成本 |
|------|---------|------|-----------|---------|
| **L0 基础** | 脚本退出码、文件是否存在 | Bash `test -f` | 全自动 | 极低 |
| **L1 语法** | Python 语法、MD 格式 | `py_compile`、`markdownlint` | 全自动 | 低 |
| **L2 结构** | 表格有表名、标题层级正确、无 bullet point | `table_name_check.py`、`bid_standardize.py`、`report_quality_check.py` | 全自动 | 低 |
| **L3 格式** | DOCX 页眉页脚、样式完整、水印正确 | `docx_format_check.py` snapshot/compare | 全自动 | 中 |
| **L4 内容** | 4 维度审阅质量（D1 来龙去脉、D2 乙方立场与措辞、D3 报告结构、D4 预判评审） | CC + report-writing Skill | 半自动（需人确认） | 高 |
| **L5 业务** | 数据准确性、政策引用正确、技术参数合理 | 人工审查 | 手动 | 最高 |

**关键洞察**：越往上，自动化越难，但发现问题的价值越高。验证闭环的目标不是把所有层都自动化，而是：

- L0-L3 全自动，无需人介入
- L4 半自动，CC 做初筛，人做终审
- L5 纯人工，但有清单辅助

这样你的注意力只需要花在 L4-L5，低层级的问题在你看到之前就已经被拦截了。

---

## 三、我们已有的验证工具

你的 `~/Dev/scripts/scripts/document/` 目录下，已经有一批可以直接用于验证的工具。下面逐个盘点。

### L2 层：结构检查

#### 1. `table_name_check.py` — 表格命名检查

**位置**：`~/Dev/scripts/scripts/document/table_name_check.py`

**做什么**：扫描 MD 文件中的所有表格，检查每个表格前是否有 `表X-Y 描述` 格式的表名，以及是否有 >=80 字的引导段落。

**用法**：
```bash
# 检查模式（只报告，不修改）
python3 table_name_check.py md_final/

# 修复模式（插入占位表名和 TABLE_NEEDS_INTRO 标记）
python3 table_name_check.py md_final/ --fix

# 检查单个文件，放宽引导段落要求到 40 字
python3 table_name_check.py md_final/01.md --min-intro 40
```

**退出码**：有问题返回 `1`，无问题返回 `0`。可以直接用于流水线判断。

**输出示例**：
```
文件数: 12
表格总数: 89
缺表名: 3
缺引导段落: 7
总问题: 10
```

#### 2. `bid_standardize.py` — 标书结构标准化

**位置**：`~/Dev/scripts/scripts/document/bid_standardize.py`

**做什么**：六项结构检查——标题级别校正、评分引用注入、编号标签清理、加粗标题提升、标题编号补全、数据来源提取。

**用法**：
```bash
# 检查模式
python3 bid_standardize.py md_fixed/ --scoring scoring.json

# 修复模式
python3 bid_standardize.py md_fixed/ --scoring scoring.json --fix

# 修复到新目录（不覆盖原文件）
python3 bid_standardize.py md_fixed/ --scoring scoring.json --fix --output-dir md_std/
```

**退出码**：同上，有问题返回 `1`。

#### 3. `report_quality_check.py` — 报告/标书质量检查

**位置**：`~/Dev/scripts/scripts/document/report_quality_check.py`

**做什么**：六项本地检查（不调 API）——禁用词（确保/我们/我司）、bullet point、数据来源标注、评分对齐、重复行、有序列表。支持自动修复。

**用法**：
```bash
# 报告模式检查
python3 report_quality_check.py 01.md

# 标书模式（额外检查评分对齐）
python3 report_quality_check.py 01.md --bid --scoring 评分标准.md

# 检查 + 自动修复
python3 report_quality_check.py ./md/ --fix

# 修复到新目录
python3 report_quality_check.py ./md/ --bid --scoring s.md --fix --output-dir ./md_fixed/
```

**退出码**：同上，有问题返回 `1`。

#### 4. `chart_insert.py` — ASCII Art 图片替换检查

**位置**：`~/Dev/scripts/scripts/document/chart_insert.py`

**做什么**：检查 MD 文件中的 ASCII art 代码块是否已被替换为 PNG 图片引用。支持 check（报告）和 fix（替换）两种模式。

**用法**：
```bash
# 检查模式
python3 chart_insert.py md_final/ --config insert_config.json

# 替换模式
python3 chart_insert.py md_final/ --config insert_config.json --fix
```

### L3 层：格式检查

#### 5. `docx_format_check.py` — Word 文档格式检查

**位置**：`~/Dev/scripts/scripts/document/docx_format_check.py`

**做什么**：两层检查机制。A 层是 ZIP 完整性——逐文件 hash 比对，只允许 `document.xml` 和 `comments.xml` 变化。B 层是格式语义——提取页面设置、样式定义、页眉页脚、水印等信息。

**用法**：
```bash
# 生成格式报告
python3 docx_format_check.py snapshot input.docx

# 存为 JSON 快照（用于后续比对）
python3 docx_format_check.py snapshot input.docx -o snap.json

# 对比两个文件（比如模板和成品）
python3 docx_format_check.py compare before.docx after.docx
```

**核心思路**：先对模板做 snapshot，生成基线。每次生成 DOCX 后，用 compare 对比成品和基线，任何"unexpected"的变化都需要关注。

这是目前你最强的格式守门员——它不关心内容写了什么，只关心格式有没有被破坏。

### L4 层：内容审阅

#### 6. `review_summary.py` — 审阅规则摘要生成器

**位置**：`~/Dev/scripts/scripts/document/review_summary.py`

**做什么**：从审阅规则 JSON 生成摘要报告。支持按 4 维度分类（D1 来龙去脉、D2 乙方立场与措辞、D3 报告结构、D4 预判评审），按修改类型分类（文字修订、删除建议、仅批注），以及多个规则文件的合并去重。

**用法**：
```bash
# 生成摘要
python3 review_summary.py rules.json

# 输出到文件
python3 review_summary.py rules.json -o summary.md

# 合并多个规则文件
python3 review_summary.py rules1.json rules2.json --merge -o merged.json
```

**在验证中的角色**：这个工具不直接做检查，而是帮你统计审阅结果——哪些维度命中了、命中了多少。用于 L4 层的质量度量。

### 工具总览

| 工具 | 层级 | 模式 | 退出码 | 可自动化 |
|------|------|------|--------|---------|
| `table_name_check.py` | L2 | check / fix | 0/1 | 全自动 |
| `bid_standardize.py` | L2 | check / fix | 0/1 | 全自动 |
| `report_quality_check.py` | L2 | check / fix | 0/1 | 全自动 |
| `chart_insert.py` | L2 | check / fix | 0/1 | 全自动 |
| `docx_format_check.py` | L3 | snapshot / compare | - | 全自动 |
| `review_summary.py` | L4 | 统计 | - | 半自动 |

**共同设计模式**：你的这些工具都遵循 `check`（只报告）/ `fix`（自动修复）的双模式设计，check 模式有问题返回退出码 `1`。这个设计非常适合做验证流水线——流水线里一律用 check 模式，修复另外单独做。

---

## 四、验收标准怎么写

验收标准的核心要求：**可量化、可自动检查、无歧义**。

### 坏的验收标准

```
标书写完了，看起来没问题
```

这句话的问题：
- "写完了"——12 章都写了吗？每章多长？
- "看起来"——看了什么？标题？内容？格式？
- "没问题"——什么叫没问题？跟谁比？

这不是验收标准，这是主观感受。

### 好的验收标准

用钱塘江标书项目举例：

```
验收清单：
  [L0] 12 个章节 MD 文件全部存在（01.md ~ 12.md）
  [L0] merged.md 合并文件已生成
  [L2] 0 个 bullet point（report_quality_check.py 退出码 0）
  [L2] 0 个 [待命名] 占位符
  [L2] 89/89 表格有表名（table_name_check.py 退出码 0）
  [L2] 0 个禁用词（确保/我们/我司）
  [L2] 标题层级正确（bid_standardize.py 退出码 0）
  [L2] 7 张图片引用正确（chart_insert.py 退出码 0）
  [L3] docx_format_check.py compare 无 unexpected 差异
  [L4] 4 维度审阅覆盖率 >= 80%
  [L5] 技术参数已人工核实（水文数据、设计标准）
```

**每一条都可以回答"是或否"**，没有模糊地带。

### 验收标准放在哪

验收标准不是写完再补的，而是任务定义的一部分：

| 场景 | 放在哪 | 示例 |
|------|--------|------|
| 单次任务 | Prompt 里 | "写完后确认：0 个 bullet point，所有表格有表名" |
| 重复流程 | Skill 的 SKILL.md | report-writing Skill 末尾的验收清单 |
| 全局规则 | CLAUDE.md | "所有 MD 输出必须通过 report_quality_check.py" |

---

## 五、验证流水线设计

零散的验证工具，串成一条流水线，才能一键跑完所有检查。

### 设计思路

```
输入（MD 目录）
  │
  ├── L0: 文件存在性检查
  ├── L2: 结构检查（3 个工具并行）
  ├── L2: 敏感词扫描
  ├── L3: 格式检查（需要 DOCX 文件）
  │
  └── 汇总报告（通过/失败/跳过）
```

### 完整示例：标书验证流水线

```bash
#!/bin/bash
# verify_bid.sh - 标书验证流水线
#
# 用法：
#   ./verify_bid.sh /path/to/md_final/ /path/to/scoring.json
#
# 退出码：
#   0 = 全部通过
#   1 = 有检查失败

set -euo pipefail

MD_DIR="${1:?用法: verify_bid.sh <md_dir> <scoring.json>}"
SCORING="${2:?用法: verify_bid.sh <md_dir> <scoring.json>}"
SCRIPT_DIR="$HOME/Dev/scripts/scripts/document"
PYTHON="/Users/tianli/miniforge3/bin/python3"

PASS=0
FAIL=0
SKIP=0
ERRORS=()

# 辅助函数：运行检查并记录结果
run_check() {
    local name="$1"
    shift
    echo "--- $name ---"
    if "$@" 2>&1; then
        echo "  => PASS"
        ((PASS++))
    else
        echo "  => FAIL"
        ((FAIL++))
        ERRORS+=("$name")
    fi
    echo ""
}

echo "========================================"
echo "  标书验证流水线"
echo "  目录: $MD_DIR"
echo "  时间: $(date '+%Y-%m-%d %H:%M:%S')"
echo "========================================"
echo ""

# ── L0: 文件存在性 ──────────────────────────────────────

echo "=== L0: 文件存在性 ==="
check_files() {
    local missing=0
    for i in $(seq -w 1 12); do
        if [ ! -f "$MD_DIR/${i}.md" ]; then
            echo "  缺少: ${i}.md"
            missing=$((missing + 1))
        fi
    done
    [ "$missing" -eq 0 ]
}
run_check "L0-文件存在性" check_files

# ── L2: 结构检查 ────────────────────────────────────────

echo "=== L2: 结构检查 ==="

run_check "L2-表格命名" \
    "$PYTHON" "$SCRIPT_DIR/table_name_check.py" "$MD_DIR"

run_check "L2-质量检查" \
    "$PYTHON" "$SCRIPT_DIR/report_quality_check.py" "$MD_DIR" --bid --scoring "$SCORING"

run_check "L2-标书结构" \
    "$PYTHON" "$SCRIPT_DIR/bid_standardize.py" "$MD_DIR" --scoring "$SCORING"

# ── L2: 敏感词扫描 ──────────────────────────────────────

echo "=== L2: 敏感词扫描 ==="
check_sensitive() {
    # 定义敏感词列表（根据项目调整）
    local keywords=("竞争对手公司名" "内部项目代号" "报价金额")
    local found=0
    for kw in "${keywords[@]}"; do
        if grep -rl "$kw" "$MD_DIR"/*.md 2>/dev/null; then
            echo "  发现敏感词: $kw"
            found=$((found + 1))
        fi
    done
    [ "$found" -eq 0 ]
}
run_check "L2-敏感词" check_sensitive

# ── L2: 占位符检查 ──────────────────────────────────────

echo "=== L2: 占位符检查 ==="
check_placeholders() {
    local patterns=("\[待命名\]" "\[TODO\]" "\[待补充\]" "XXX" "\[待确认\]")
    local found=0
    for pat in "${patterns[@]}"; do
        local hits
        hits=$(grep -rn "$pat" "$MD_DIR"/*.md 2>/dev/null | wc -l)
        if [ "$hits" -gt 0 ]; then
            echo "  发现占位符 $pat: ${hits} 处"
            grep -rn "$pat" "$MD_DIR"/*.md 2>/dev/null | head -3
            found=$((found + hits))
        fi
    done
    [ "$found" -eq 0 ]
}
run_check "L2-占位符" check_placeholders

# ── L3: 格式检查 ────────────────────────────────────────

echo "=== L3: DOCX 格式检查 ==="
DOCX_FILE="$MD_DIR/../output.docx"
TEMPLATE="$MD_DIR/../template.docx"

if [ -f "$DOCX_FILE" ] && [ -f "$TEMPLATE" ]; then
    run_check "L3-DOCX格式" \
        "$PYTHON" "$SCRIPT_DIR/docx_format_check.py" compare "$TEMPLATE" "$DOCX_FILE"
else
    echo "  跳过（未找到 DOCX 文件或模板）"
    ((SKIP++))
fi

# ── 汇总 ────────────────────────────────────────────────

echo ""
echo "========================================"
echo "  验证结果汇总"
echo "========================================"
echo "  通过: $PASS"
echo "  失败: $FAIL"
echo "  跳过: $SKIP"

if [ "$FAIL" -gt 0 ]; then
    echo ""
    echo "  失败项:"
    for err in "${ERRORS[@]}"; do
        echo "    - $err"
    done
    echo ""
    echo "  结论: 未通过验证，请修复上述问题后重新运行"
    exit 1
else
    echo ""
    echo "  结论: 全部通过"
    exit 0
fi
```

### 流水线的设计原则

1. **每个检查独立运行**：一个失败不影响其他检查继续执行，最后汇总所有结果
2. **退出码决定通过/失败**：利用你工具已有的退出码设计（`0` = 通过，`1` = 失败）
3. **可跳过**：某些检查依赖特定文件（比如 DOCX），不存在时跳过而非报错
4. **汇总清晰**：最后输出通过/失败/跳过的数量和失败项列表

---

## 六、验证 + Hook 联动

验证流水线是交付前的"终检"。但如果问题在写作过程中就不断积累，等终检时才发现，修复成本很高。

Hook 的作用是"实时验证"——每次 CC 编辑文件后，立即检查是否引入了问题。

### 两者的分工

| | Hook（实时验证） | 流水线（终检） |
|---|---|---|
| **触发时机** | 每次 Edit 后自动触发 | 交付前手动运行一次 |
| **检查范围** | 单个文件 | 整个项目目录 |
| **检查深度** | 轻量级（L0-L2） | 全面（L0-L3，L4 可选） |
| **速度要求** | 必须快（<2秒） | 可以慢（30秒以内可接受） |
| **失败后果** | 阻断当前操作 | 阻断交付 |

### Hook 做什么

适合放进 Hook 的检查：

```jsonc
// .claude/hooks.json（项目级）
{
  "hooks": {
    "PostEdit": [
      {
        "command": "python3 ~/Dev/scripts/scripts/document/report_quality_check.py $EDITED_FILE",
        "description": "编辑后检查禁用词和 bullet point"
      }
    ]
  }
}
```

Hook 适合做的检查：
- 禁用词扫描（快，单文件）
- bullet point 检查（快，单文件）
- 占位符检查（grep，极快）

Hook 不适合做的检查：
- 跨文件的表格编号连续性（需要扫描整个目录）
- DOCX 格式对比（需要先生成 DOCX）
- 内容审阅（需要调 API，太慢）

### 实际工作流

```
写作过程：
  CC 写第 3 章 → Hook 自动检查 → 发现 2 个 bullet point → CC 立即修复
  CC 写第 4 章 → Hook 自动检查 → 通过
  CC 写第 5 章 → Hook 自动检查 → 发现禁用词"确保" → CC 替换为"保障"
  ...

交付前：
  你运行 verify_bid.sh → 全面检查 → 发现 3 个表格缺表名 → 修复 → 再跑一次 → 全部通过
```

这样，大部分低级问题在写作过程中就被拦截了，终检时只会发现少量遗漏。

---

## 七、常见反模式

### 反模式 1：只在最后验证

**现象**：12 章全写完了，最后才跑检查，发现 47 个 bullet point、12 个禁用词、5 个表格缺表名。

**问题**：批量修复容易引入新问题，而且你已经不记得每个 bullet point 的上下文了。

**正确做法**：Hook 做实时拦截，每章写完做一次中间检查，最后终检只是确认。

### 反模式 2：验证标准模糊

**现象**："帮我检查一下标书质量"——检查什么？怎么算通过？

**问题**：CC 会自己发明检查项，可能漏掉你关心的，也可能检查你不在意的。

**正确做法**：把验收标准写成清单，放进 Prompt 或 Skill。每一条都是"是/否"判断。

### 反模式 3：验证结果不阻断

**现象**：跑了检查，发现 3 个问题，但觉得"问题不大"，直接交付了。

**问题**：如果问题不大可以不查，那这个检查项本身就不该存在。每个检查项要么阻断、要么删除，不能有"发现了但忽略"的灰色地带。

**正确做法**：如果确实有可以容忍的情况，用 `--ignore` 参数或配置文件显式豁免，而不是人肉忽略。

### 反模式 4：没有对比基线

**现象**：生成了 DOCX，打开看了看，"格式好像没问题"。

**问题**：人眼看不出页边距差了 0.5cm、页眉字号从五号变成了小五。

**正确做法**：`docx_format_check.py` 的 snapshot/compare 机制就是解决这个问题——先对模板做 snapshot 生成基线 JSON，每次生成成品后 compare，任何非内容区域的变化都会被标记。

```bash
# 第一次：对模板做基线快照
python3 docx_format_check.py snapshot template.docx -o baseline.json

# 每次生成后：对比成品和基线
python3 docx_format_check.py compare template.docx output.docx
```

---

## 八、TODO

- [ ] 写一个 `verify_bid.sh` 标书验证流水线（基于第五节的设计，放到 `~/Dev/scripts/scripts/document/`）
- [ ] 给 report-writing Skill 加验收清单（在 SKILL.md 末尾加一节"验收标准"）
- [ ] 建立"做完"的标准定义（在 CLAUDE.md 中添加规则：所有交付物必须附带验收清单）

---

## 批注区

> 读完有问题或想法，写在这里，下次会话时一起讨论。

-
