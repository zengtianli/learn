# Claude Code Hooks 实战教学

> **前置知识**：了解六层架构（`Layer1-长期上下文.md` / `Layer2-工具能力.md` / `Layer3-工作流Skills.md`），知道 Hooks 是 Layer 4 强制执行层。
> **目标**：读完这份文件后，你能独立给自己的项目配置 Hooks，并理解什么场景该用 Hook、什么场景不该用。
> **预计阅读时间**：30 分钟（含动手实践约 1 小时）

**怎么读这份文件**：

1. 先读第一、二、三章，建立概念框架（15 分钟）
2. 然后跳到第四章，挑一个 Hook 动手配置（30 分钟）
3. 最后读第五、六章，理解 Hooks 在整个防护体系中的位置

**读完能做什么**：
- 给 zdwp 标书项目配一个敏感词检查 Hook（防废标）
- 给所有 Python 项目配语法检查 Hook（防低级错误）
- 理解 CLAUDE.md / Skill / Hook 三层防护各自的职责边界

---

## 一、Hooks 是什么（一句话）

**Hook = CC 执行动作时自动触发你的脚本。不依赖 CC "记住"规则。**

你在 CLAUDE.md 里写"标书中禁止出现投标单位名称"，这是**口头嘱咐**——CC 可能忘记，可能在长对话中丢失这条规则，可能判断"这次不需要检查"。

Hook 是**门禁系统**——不管 CC 怎么想，每次它写文件，你的检查脚本一定会跑。CC 无法跳过、无法忽略、无法"判断这次不需要"。

回忆六层架构里的那句话：

> Layer 1-3 是"建议"，Claude 可以灵活处理；Layer 4-6 是"强制"，不依赖 Claude 的判断力。

Hook 就是 Layer 4 的落地机制。

<!-- 批注区 -->

---

## 二、生命周期速查表

CC 在运行过程中有多个"关键时刻"，你可以在这些时刻插入自己的脚本。下面列出最常用的 5 个（CC 实际支持 20+ 个生命周期事件，但你先掌握这 5 个就够了）：

| 生命周期 | 触发时机 | 能阻断吗 | 典型用途 |
|---------|---------|---------|---------|
| **SessionStart** | 新会话启动、`/clear`、`/compact` 后 | 不能 | 注入动态上下文（Git 状态、项目信息） |
| **PreToolUse** | CC 准备用某个工具**之前** | **能** | 阻断危险操作（`rm -rf`、直接 push main） |
| **PostToolUse** | CC 用完某个工具**之后** | 不能直接阻断，但能反馈 | 自动 lint、敏感词检查、格式验证 |
| **Notification** | CC 发出通知时 | 不能 | 推送到飞书/钉钉 |
| **Stop** | CC 完成回复、准备停下来时 | **能** | 生成任务总结、检查是否真的完成了 |

### 理解"阻断"

**PreToolUse 的阻断**是最强大的能力。举个例子：

- CC 准备执行 `rm -rf /important/directory`
- 你的 PreToolUse Hook 检测到危险命令
- Hook 返回"拒绝"，CC 收到反馈："这个命令被安全策略阻止了，因为包含 rm -rf"
- CC 不得不换一种方式完成任务

**PostToolUse 的反馈**虽然不能阻断（工具已经执行完了），但它的输出会进入 CC 的上下文。比如敏感词检查发现问题后，CC 会看到检查结果并自动修复。

### 完整生命周期一览（进阶参考）

除了上面 5 个常用的，CC 还支持这些生命周期事件（你暂时不需要用，但知道有这些东西）：

| 生命周期 | 说明 |
|---------|------|
| UserPromptSubmit | 用户提交 prompt 时（能阻断） |
| PostToolUseFailure | 工具执行失败后 |
| SubagentStart / SubagentStop | subagent 启动/停止时 |
| PreCompact / PostCompact | 上下文压缩前/后 |
| SessionEnd | 会话结束时 |
| ConfigChange | 配置文件变更时 |
| InstructionsLoaded | CLAUDE.md 加载时 |

**重要**：Hooks 对 subagent 同样生效。如果你配了一个 PostToolUse 敏感词检查 Hook，不管是主线程还是 subagent 写文件，这个 Hook 都会触发。这就是为什么 Hook 比 CLAUDE.md 更可靠——subagent 可能没读到项目级 CLAUDE.md 的某条规则，但 Hook 一定会跑。

<!-- 批注区 -->

---

## 三、配置方式

### 3.1 在哪里配置

Hook 写在 `settings.json` 里。有三个位置，作用域不同：

| 位置 | 作用域 | 适用场景 |
|------|--------|---------|
| `~/.claude/settings.json` | **所有项目** | 通用检查（Python lint、危险命令拦截） |
| `项目目录/.claude/settings.json` | **单个项目，可提交到 Git** | 项目特有检查（标书敏感词） |
| `项目目录/.claude/settings.local.json` | **单个项目，不提交到 Git** | 本地个人偏好 |

**选择建议**：
- 标书敏感词检查 → 放项目级 `.claude/settings.json`（因为是这个项目特有的）
- Python 语法检查 → 放全局 `~/.claude/settings.json`（因为所有 Python 项目都需要）
- SessionStart 注入上下文 → 放全局（所有项目都有用）

### 3.2 你当前的 settings.json

你的 `~/.claude/settings.json` 现在长这样——**没有任何 Hooks 配置**：

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "model": "opus",
  "language": "中文",
  "skipDangerousModePermissionPrompt": true,
  "effortLevel": "high"
}
```

加上 Hooks 后会变成这样（结构预览，具体内容在第四章）：

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "model": "opus",
  "language": "中文",
  "skipDangerousModePermissionPrompt": true,
  "effortLevel": "high",
  "hooks": {
    "PreToolUse": [
      { "matcher": "...", "hooks": [{ "type": "command", "command": "..." }] }
    ],
    "PostToolUse": [
      { "matcher": "...", "hooks": [{ "type": "command", "command": "..." }] }
    ],
    "SessionStart": [
      { "hooks": [{ "type": "command", "command": "..." }] }
    ]
  }
}
```

### 3.3 配置结构详解

```json
{
  "hooks": {
    "PostToolUse": [          // 生命周期事件名
      {
        "matcher": "Edit|Write",   // 正则匹配工具名（可选）
        "hooks": [                 // 该匹配条件下要执行的 Hook 列表
          {
            "type": "command",          // Hook 类型：command / http / prompt / agent
            "command": "/path/to/script.sh",  // 要执行的命令
            "timeout": 60,              // 超时（秒，可选）
            "statusMessage": "正在检查..."  // 执行时显示的消息（可选）
          }
        ]
      }
    ]
  }
}
```

**matcher 值**：匹配的是 CC 的工具名，常用值包括：
- `Bash` — CC 执行 bash 命令
- `Edit` — CC 编辑文件（Edit 工具）
- `Write` — CC 写入新文件（Write 工具）
- `Read` — CC 读文件
- `Glob` / `Grep` — CC 搜索文件
- `Agent` — CC 启动 subagent
- `Edit|Write` — 用 `|` 同时匹配多个工具
- `mcp__.*` — 匹配所有 MCP 工具

### 3.4 Hook 脚本怎么拿到数据

你的 Hook 脚本通过 **stdin 读取 JSON** 来获取 CC 的操作信息。不同生命周期传入的 JSON 字段不同：

**PostToolUse 的 stdin 输入**：
```json
{
  "session_id": "abc123",
  "hook_event_name": "PostToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/Users/tianli/Work/zdwp/workspace/chapter1.md",
    "old_string": "旧文本",
    "new_string": "新文本"
  },
  "tool_response": { ... },
  "cwd": "/Users/tianli/Work/zdwp"
}
```

**PreToolUse 的 stdin 输入**：
```json
{
  "session_id": "abc123",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /some/directory"
  }
}
```

**在脚本中读取**：
```bash
# 用 jq 从 stdin 解析 JSON
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
```

### 3.5 退出码的含义

| 退出码 | 含义 | 行为 |
|--------|------|------|
| `exit 0` | 成功 | 正常继续；如果 stdout 有 JSON，CC 会解析 |
| `exit 2` | 阻断 | **仅对 PreToolUse 等可阻断事件有效**，stderr 的内容会反馈给 CC |
| 其他非零 | 非阻断错误 | stderr 显示给用户（verbose 模式），不影响 CC 继续工作 |

**关键理解**：exit 2 不是"脚本出错了"，而是"我主动要求阻止这个操作"。这是 PreToolUse Hook 拦截危险操作的核心机制。

### 3.6 Hook 的四种类型

除了 `command`（执行 shell 脚本），还有三种类型你将来可能会用到：

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| **command** | 执行 shell 脚本 | 绝大多数场景（本教学全部用这种） |
| **http** | 发 HTTP POST 请求 | 调用外部服务（webhook、API） |
| **prompt** | 调用一次 LLM | 需要语义理解的检查（但会消耗 token） |
| **agent** | 启动一个多轮 Agent | 需要深度验证的场景（代价最高） |

**新手建议**：先只用 `command` 类型。它最轻量、最可控、最好调试。

<!-- 批注区 -->

---

## 四、4 个实战 Hook

### Hook 1：标书敏感词检查（防废标，最重要！）

#### 场景回顾

在 `实战-钱塘江标书全流程.md` 第八章中我们提到过这个事故：

> subagent 在标书中写入了错误的投标单位名称。根因：subagent 的权限和主线程一样宽——它能读写任何文件、调用任何工具。没有任何机制阻止它读到别的项目的信息并错误引用。

如果当时有敏感词检查 Hook，subagent 写入错误名称的瞬间就会被发现。CC 会立即看到"检测到敏感词：杭州市水文水资源监测中心，这不是本项目的投标单位"，然后自动修正。

**这个 Hook 能防住的事故**：
- subagent 写入竞争对手名称
- 复制粘贴时带入其他项目的公司名
- 投标单位名称写错（字不同但很像）
- 项目名称前后不一致

#### 工作原理

```
CC 写入文件（Edit/Write）
        ↓
PostToolUse Hook 触发
        ↓
检查文件内容是否包含敏感词
        ↓
发现敏感词 → 输出警告到 CC 上下文 → CC 自动修复
没有敏感词 → 静默通过
```

#### 第一步：创建敏感词配置文件

在项目目录下创建 `sensitive_words.json`：

```bash
# 文件路径：~/Work/zdwp/sensitive_words.json
```

```json
{
  "description": "标书敏感词检查配置",
  "forbidden_words": [
    {
      "word": "杭州市水文水资源监测中心",
      "reason": "竞争对手名称，绝对不能出现在投标文件中"
    },
    {
      "word": "XX水利设计院",
      "reason": "竞争对手名称"
    }
  ],
  "required_exact": [
    {
      "field": "投标单位",
      "correct_value": "你的公司全称",
      "reason": "投标单位名称必须与营业执照完全一致，差一个字都可能废标"
    }
  ],
  "file_patterns": ["*.md", "*.txt", "*.py"],
  "exclude_patterns": ["sensitive_words.json", "*.jsonl", "node_modules/*"]
}
```

> **注意**：上面的竞争对手名称和公司名称是示例，你需要根据实际项目替换。`required_exact` 用于检查"投标单位"字段是否与正确名称完全一致。

#### 第二步：创建检查脚本

```bash
# 文件路径：~/Work/zdwp/.claude/hooks/check_sensitive_words.sh
# 记得 chmod +x
```

```bash
#!/bin/bash
# check_sensitive_words.sh — 标书敏感词检查 Hook
# 触发时机：PostToolUse（Edit/Write）
# 作用：检查被修改的文件是否包含敏感词

set -euo pipefail

# 从 stdin 读取 Hook 输入
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# 如果没有文件路径（比如 Bash 工具），直接退出
if [ -z "$FILE_PATH" ] || [ ! -f "$FILE_PATH" ]; then
    exit 0
fi

# 敏感词配置文件路径（相对于项目目录）
PROJECT_DIR=$(echo "$INPUT" | jq -r '.cwd // empty')
CONFIG_FILE="${PROJECT_DIR}/sensitive_words.json"

# 如果项目目录下没有敏感词配置，静默退出
if [ ! -f "$CONFIG_FILE" ]; then
    exit 0
fi

# 检查文件是否在排除列表中
FILENAME=$(basename "$FILE_PATH")
EXCLUDE_MATCH=$(echo "$INPUT" | jq -r --arg fn "$FILENAME" '
    .cwd as $cwd |
    null
')

# 简单的扩展名检查：只检查文本文件
case "$FILENAME" in
    *.md|*.txt|*.py|*.json|*.yaml|*.yml|*.csv|*.rst|*.html)
        ;;
    sensitive_words.json|*.jsonl|*.docx|*.pdf|*.png|*.jpg)
        exit 0
        ;;
    *)
        exit 0
        ;;
esac

# 读取敏感词列表
FORBIDDEN_WORDS=$(jq -r '.forbidden_words[].word' "$CONFIG_FILE" 2>/dev/null)

if [ -z "$FORBIDDEN_WORDS" ]; then
    exit 0
fi

# 检查文件内容
FOUND_WORDS=""
while IFS= read -r word; do
    if grep -qF "$word" "$FILE_PATH" 2>/dev/null; then
        REASON=$(jq -r --arg w "$word" '.forbidden_words[] | select(.word == $w) | .reason' "$CONFIG_FILE")
        FOUND_WORDS="${FOUND_WORDS}\n  - \"${word}\" — ${REASON}"
    fi
done <<< "$FORBIDDEN_WORDS"

# 如果发现敏感词，输出警告（进入 CC 上下文）
if [ -n "$FOUND_WORDS" ]; then
    echo "[SENSITIVE WORD ALERT] 在 ${FILE_PATH} 中发现敏感词：${FOUND_WORDS}" >&2
    echo ""
    echo "请立即检查并替换这些内容。标书中出现竞争对手名称可能导致废标。" >&2
    # exit 0 而不是 exit 2：PostToolUse 不支持阻断，但 stderr 会显示给用户
    # 真正的反馈通过 stdout JSON 传给 CC
    echo "{\"decision\": \"block\", \"reason\": \"发现敏感词：${FOUND_WORDS}\\n请立即修正这些内容。\"}"
    exit 0
fi

# 没有问题，静默退出
exit 0
```

#### 第三步：配置 settings.json

在 **项目级** 配置文件中添加（如果文件不存在则创建）：

```bash
# 文件路径：~/Work/zdwp/.claude/settings.json
```

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check_sensitive_words.sh",
            "statusMessage": "正在检查敏感词..."
          }
        ]
      }
    ]
  }
}
```

#### 第四步：测试

```bash
# 1. 给脚本加执行权限
chmod +x ~/Work/zdwp/.claude/hooks/check_sensitive_words.sh

# 2. 手动测试（模拟 PostToolUse 输入）
echo '{
  "hook_event_name": "PostToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/tmp/test_sensitive.md"
  },
  "cwd": "/Users/tianli/Work/zdwp"
}' | bash ~/Work/zdwp/.claude/hooks/check_sensitive_words.sh

# 3. 创建测试文件验证检测
echo "本项目由杭州市水文水资源监测中心负责实施" > /tmp/test_sensitive.md
echo '{
  "hook_event_name": "PostToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/tmp/test_sensitive.md"
  },
  "cwd": "/Users/tianli/Work/zdwp"
}' | bash ~/Work/zdwp/.claude/hooks/check_sensitive_words.sh

# 4. 在 CC 中验证：进入 zdwp 目录启动 CC，输入 /hooks 查看是否加载
```

#### 关键设计决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 放在哪一层 | PostToolUse（不是 PreToolUse） | 敏感词检查需要看到文件内容；PreToolUse 时文件还没被修改 |
| 配置放哪里 | 项目级 `.claude/settings.json` | 每个项目的敏感词不同，不应该放全局 |
| 敏感词存哪里 | 独立的 JSON 文件 | 方便维护，不同项目可以有不同的敏感词列表 |
| 用 exit 0 还是 exit 2 | exit 0 + JSON decision:block | PostToolUse 中 exit 2 只是显示给用户，用 JSON 的 decision 字段反馈给 CC 更有效 |

<!-- 批注区 -->

---

### Hook 2：Python 语法检查（防低级错误）

#### 场景

CC 编辑 Python 文件后，可能引入语法错误（缺少冒号、缩进不对、括号不匹配）。虽然 CC 犯这种错的概率不高，但一旦发生，后续的调试会浪费很多时间。

**这个 Hook 能防住的问题**：
- 编辑后文件有语法错误
- 缩进混乱（tab/space 混用）
- 引号/括号不匹配

#### 检查脚本

```bash
# 文件路径：~/.claude/hooks/check_python_syntax.sh
# 记得 chmod +x
```

```bash
#!/bin/bash
# check_python_syntax.sh — Python 语法检查 Hook
# 触发时机：PostToolUse（Edit/Write）
# 作用：检查被修改的 Python 文件是否有语法错误

set -euo pipefail

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# 不是文件操作或文件不存在，直接退出
if [ -z "$FILE_PATH" ] || [ ! -f "$FILE_PATH" ]; then
    exit 0
fi

# 只检查 .py 文件
case "$FILE_PATH" in
    *.py) ;;
    *) exit 0 ;;
esac

# 用 py_compile 检查语法
PYTHON="/Users/tianli/miniforge3/bin/python3"
SYNTAX_OUTPUT=$($PYTHON -m py_compile "$FILE_PATH" 2>&1) || {
    # 语法错误，反馈给 CC
    echo "{\"decision\": \"block\", \"reason\": \"Python 语法错误：${SYNTAX_OUTPUT}\\n请修复上述语法错误。\"}"
    exit 0
}

# 语法正确，静默退出
exit 0
```

#### 配置（全局）

这个 Hook 放在**全局** `~/.claude/settings.json` 里，因为所有 Python 项目都需要：

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "model": "opus",
  "language": "中文",
  "skipDangerousModePermissionPrompt": true,
  "effortLevel": "high",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/check_python_syntax.sh",
            "statusMessage": "正在检查 Python 语法..."
          }
        ]
      }
    ]
  }
}
```

#### 测试

```bash
# 加执行权限
chmod +x ~/.claude/hooks/check_python_syntax.sh

# 测试：正确的 Python 文件（应该静默通过）
echo 'print("hello")' > /tmp/test_syntax.py
echo '{
  "tool_name": "Edit",
  "tool_input": {"file_path": "/tmp/test_syntax.py"}
}' | bash ~/.claude/hooks/check_python_syntax.sh
echo "退出码: $?"

# 测试：有语法错误的 Python 文件（应该输出错误信息）
echo 'def foo(
    print("missing closing paren"' > /tmp/test_syntax_bad.py
echo '{
  "tool_name": "Edit",
  "tool_input": {"file_path": "/tmp/test_syntax_bad.py"}
}' | bash ~/.claude/hooks/check_python_syntax.sh
echo "退出码: $?"
```

#### 进阶：加上 ruff 检查

如果你安装了 ruff（Python linter），可以在语法检查的基础上加上代码质量检查：

```bash
# 在 check_python_syntax.sh 的末尾，语法检查通过后加上：

# 可选：用 ruff 做代码质量检查
if command -v ruff &>/dev/null; then
    RUFF_OUTPUT=$(ruff check "$FILE_PATH" --no-fix 2>&1 | head -10) || {
        echo "{\"hookSpecificOutput\": {\"hookEventName\": \"PostToolUse\", \"additionalContext\": \"Ruff 检查发现问题：${RUFF_OUTPUT}\"}}"
        exit 0
    }
fi
```

<!-- 批注区 -->

---

### Hook 3：SessionStart 注入上下文

#### 场景

每次启动新会话，你经常需要手动告诉 CC 当前项目的状态："我在 zdwp 项目"、"刚切了分支"、"上次改到第三章"。SessionStart Hook 可以自动注入这些信息。

**这个 Hook 能解决的问题**：
- 新会话不知道当前 Git 分支和状态
- 不知道最近修改了哪些文件
- 不知道有没有未提交的变更

#### 检查脚本

```bash
# 文件路径：~/.claude/hooks/session_context.sh
# 记得 chmod +x
```

```bash
#!/bin/bash
# session_context.sh — 会话启动时注入项目上下文
# 触发时机：SessionStart
# 作用：自动收集当前目录信息，注入 CC 的上下文

set -euo pipefail

# 收集上下文信息
CONTEXT=""

# 1. 当前目录
CONTEXT="[会话上下文]\n"
CONTEXT+="工作目录: $(pwd)\n"

# 2. Git 信息（如果在 Git 仓库中）
if git rev-parse --is-inside-work-tree &>/dev/null; then
    BRANCH=$(git branch --show-current 2>/dev/null || echo "detached")
    STATUS=$(git status --short 2>/dev/null | head -10)
    RECENT_COMMITS=$(git log --oneline -5 2>/dev/null || echo "无提交记录")
    UNCOMMITTED_COUNT=$(git status --short 2>/dev/null | wc -l | tr -d ' ')

    CONTEXT+="Git 分支: ${BRANCH}\n"
    CONTEXT+="未提交变更: ${UNCOMMITTED_COUNT} 个文件\n"

    if [ -n "$STATUS" ]; then
        CONTEXT+="变更文件:\n${STATUS}\n"
    fi

    CONTEXT+="最近提交:\n${RECENT_COMMITS}\n"
fi

# 3. 最近修改的文件（排除隐藏目录和常见缓存）
RECENT_FILES=$(find . -maxdepth 3 -type f \
    -not -path '*/.git/*' \
    -not -path '*/.claude/*' \
    -not -path '*/node_modules/*' \
    -not -path '*/__pycache__/*' \
    -not -path '*/.venv/*' \
    -newer /tmp/.claude_session_marker 2>/dev/null \
    | head -10 || true)

# 如果没有 marker 文件，用最近 24 小时内修改的文件
if [ ! -f /tmp/.claude_session_marker ]; then
    RECENT_FILES=$(find . -maxdepth 3 -type f -mtime -1 \
        -not -path '*/.git/*' \
        -not -path '*/.claude/*' \
        -not -path '*/node_modules/*' \
        -not -path '*/__pycache__/*' \
        -not -path '*/.venv/*' \
        2>/dev/null | head -10 || true)
fi

if [ -n "$RECENT_FILES" ]; then
    CONTEXT+="最近修改的文件:\n${RECENT_FILES}\n"
fi

# 更新 marker 文件
touch /tmp/.claude_session_marker

# 输出 JSON（additionalContext 会被注入 CC 的上下文）
# 注意：需要把换行符转义为 JSON 兼容格式
ESCAPED_CONTEXT=$(echo -e "$CONTEXT" | jq -Rs '.')

echo "{\"hookSpecificOutput\": {\"hookEventName\": \"SessionStart\", \"additionalContext\": ${ESCAPED_CONTEXT}}}"
exit 0
```

#### 配置（全局）

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/session_context.sh",
            "statusMessage": "正在加载项目上下文..."
          }
        ]
      }
    ]
  }
}
```

> **注意**：SessionStart 不需要 `matcher` 字段。它支持的 matcher 值是 `startup|resume|clear|compact`，如果不写 matcher 就对所有启动方式都触发。

#### 测试

```bash
chmod +x ~/.claude/hooks/session_context.sh

# 在一个 Git 项目目录下测试
cd ~/Work/zdwp
echo '{}' | bash ~/.claude/hooks/session_context.sh
```

#### 效果

配置后，每次启动 CC 会话时，CC 的上下文里会自动包含类似这样的信息：

```
[会话上下文]
工作目录: /Users/tianli/Work/zdwp
Git 分支: feat/chapter-3-revise
未提交变更: 3 个文件
变更文件:
 M workspace/chapter3.md
 M workspace/chapter3_tables.md
?? workspace/chapter3_figures/
最近提交:
a1b2c3d 完成第二章初稿
d4e5f6g 添加水文数据表格
...
```

CC 看到这些信息后，就不需要你手动说"我上次改到第三章了"。

<!-- 批注区 -->

---

### Hook 4：通知推送（飞书机器人）

#### 场景

你让 CC 跑一个长任务（比如多 Agent 并行写 12 章标书），然后去做别的事。任务完成时，你想收到飞书通知，而不是每隔几分钟回来看一眼。

**这个 Hook 能解决的问题**：
- 长任务完成后不用反复回来检查
- subagent 全部完成时自动通知
- 遇到需要人工介入的情况时及时提醒

#### 检查脚本

```bash
# 文件路径：~/.claude/hooks/notify_feishu.sh
# 记得 chmod +x
```

```bash
#!/bin/bash
# notify_feishu.sh — 任务完成时推送飞书通知
# 触发时机：Stop 或 Notification
# 作用：通过飞书机器人 Webhook 发送通知

set -euo pipefail

INPUT=$(cat)

# 飞书机器人 Webhook URL（替换为你自己的）
# 建议把这个 URL 存到环境变量或单独的配置文件中，不要硬编码
WEBHOOK_URL="${FEISHU_WEBHOOK_URL:-}"

if [ -z "$WEBHOOK_URL" ]; then
    # 没有配置 Webhook，静默退出
    exit 0
fi

# 获取事件信息
EVENT_NAME=$(echo "$INPUT" | jq -r '.hook_event_name // "Unknown"')
CWD=$(echo "$INPUT" | jq -r '.cwd // "Unknown"')
PROJECT_NAME=$(basename "$CWD")

# 构造通知内容
TIMESTAMP=$(date '+%H:%M:%S')

if [ "$EVENT_NAME" = "Stop" ]; then
    TITLE="CC 任务完成"
    CONTENT="项目 ${PROJECT_NAME} 的任务已完成\n时间: ${TIMESTAMP}\n目录: ${CWD}"
elif [ "$EVENT_NAME" = "Notification" ]; then
    TITLE="CC 需要关注"
    CONTENT="项目 ${PROJECT_NAME} 发出通知\n时间: ${TIMESTAMP}\n目录: ${CWD}"
else
    TITLE="CC 事件"
    CONTENT="事件: ${EVENT_NAME}\n项目: ${PROJECT_NAME}\n时间: ${TIMESTAMP}"
fi

# 发送飞书消息
curl -s -X POST "$WEBHOOK_URL" \
    -H "Content-Type: application/json" \
    -d "{
        \"msg_type\": \"interactive\",
        \"card\": {
            \"header\": {
                \"title\": {
                    \"tag\": \"plain_text\",
                    \"content\": \"${TITLE}\"
                },
                \"template\": \"green\"
            },
            \"elements\": [
                {
                    \"tag\": \"div\",
                    \"text\": {
                        \"tag\": \"lark_md\",
                        \"content\": \"${CONTENT}\"
                    }
                }
            ]
        }
    }" > /dev/null 2>&1 || true

# 异步通知不应该阻塞 CC
exit 0
```

#### 配置

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/notify_feishu.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "idle_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/notify_feishu.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

#### 配置飞书 Webhook

1. 打开飞书群 → 设置 → 群机器人 → 添加自定义机器人
2. 复制 Webhook URL
3. 把 URL 添加到你的 shell 环境变量中：

```bash
# 在 ~/.zshrc 中添加
export FEISHU_WEBHOOK_URL="https://open.feishu.cn/open-apis/bot/v2/hook/your-webhook-id"
```

#### 设计说明

| 决策 | 选择 | 理由 |
|------|------|------|
| 触发时机 | Stop + Notification | Stop 覆盖任务完成；Notification 覆盖需要关注的情况 |
| 超时设置 | 10 秒 | 网络请求可能慢，但不能阻塞太久 |
| 失败处理 | `\|\| true` | 通知失败不应该影响 CC 的工作流 |
| Webhook URL | 环境变量 | 不在配置文件中硬编码敏感信息 |

> **进阶想法**：你可以把这个 Hook 改成 `async: true`，这样它会在后台发送通知，完全不阻塞 CC：
>
> ```json
> { "type": "command", "command": "bash ~/.claude/hooks/notify_feishu.sh", "async": true }
> ```

<!-- 批注区 -->

---

## 五、三层防护对照

现在我们把 CLAUDE.md、Skill、Hook 三层叠加起来看，理解它们各自的角色。

### 用标书安全事故做例子

**事故**：subagent 在标书第 6 章中写入了"杭州市水文水资源监测中心"（竞争对手名称）。

三层防护的作用：

```
第一层：CLAUDE.md（声明层）
┌──────────────────────────────────────────────────┐
│  "标书中禁止出现以下公司名称：..."                      │
│                                                    │
│  效果：CC 在写作时会注意避免                           │
│  弱点：subagent 可能没读到这条规则；                    │
│        长对话中可能被压缩掉；CC 可能"判断"这次不需要检查   │
│  防护等级：★★☆☆☆                                    │
└──────────────────────────────────────────────────┘

第二层：Skill（工作流层）
┌──────────────────────────────────────────────────┐
│  docx-review Skill 的步骤 3：                       │
│  "检查所有章节是否包含竞争对手名称"                      │
│                                                    │
│  效果：审阅流程中会专门做一遍检查                       │
│  弱点：只在触发 docx-review 时才检查；                  │
│        写作阶段不检查，等审阅时才发现，可能已经写了很多    │
│  防护等级：★★★☆☆                                    │
└──────────────────────────────────────────────────┘

第三层：Hook（强制执行层）
┌──────────────────────────────────────────────────┐
│  PostToolUse Hook：每次 Edit/Write 后自动检查敏感词     │
│                                                    │
│  效果：写入的瞬间就被发现，CC 立即修正                   │
│  弱点：无（只要 Hook 配置正确，不可能漏过）              │
│  防护等级：★★★★★                                    │
│                                                    │
│  而且：subagent 也会触发这个 Hook！                    │
└──────────────────────────────────────────────────┘
```

### 三层叠加的效果

| 阶段 | 谁在保护 | 触发条件 |
|------|---------|---------|
| **写作时** | CLAUDE.md 提醒 + **Hook 实时检查** | 每次 Edit/Write |
| **审阅时** | Skill 流程化检查 | 触发 docx-review 时 |
| **最终验收** | 人工检查清单（Layer 6） | 提交前 |

**核心原则：越不可接受的错误，防护层次要越硬。**

- "这个段落可以写得更好" → CLAUDE.md 里写偏好就行（Layer 1）
- "格式要符合标书规范" → 写进 Skill 流程（Layer 3）
- "绝对不能出现竞争对手名称" → 用 Hook 强制检查（Layer 4）
- "投标单位名称必须与营业执照一致" → Hook + 人工双重检查（Layer 4 + Layer 6）

### 什么不该用 Hook

理解了什么该用 Hook，也要知道什么不该用：

| 场景 | 该用什么 | 为什么不用 Hook |
|------|---------|---------------|
| "写作时注意使用正式用语" | CLAUDE.md | 需要理解语境，Hook 做不到 |
| "按照 4 维框架审阅" | Skill | 多步骤流程，Hook 不擅长 |
| "选择合适的数据来源" | CC 自主判断 | 需要理解业务，不是确定性检查 |
| "文件中不能有公司名" | **Hook** | 确定性检查，不需要理解上下文 |

**判断标准**：如果你能用 `grep` 或 `python -m py_compile` 这样的命令检查出来的问题，就适合用 Hook。如果需要"理解"才能判断的问题，不适合用 Hook。

<!-- 批注区 -->

---

## 六、常见陷阱

### 陷阱 1：Hook 输出太长，污染 CC 上下文

**问题**：你的 Hook 脚本输出了 500 行日志，全部进入 CC 的上下文，挤占了有用信息的空间。

**解决方案**：
```bash
# 限制输出长度
RESULT=$(some_check_command 2>&1 | head -30)

# 或者只输出摘要
ERRORS=$(ruff check "$FILE_PATH" 2>&1 | wc -l)
if [ "$ERRORS" -gt 0 ]; then
    FIRST_5=$(ruff check "$FILE_PATH" 2>&1 | head -5)
    echo "发现 ${ERRORS} 个问题，前 5 个：${FIRST_5}"
fi
```

**经验法则**：Hook 的输出控制在 10 行以内。如果检查结果超过 10 行，只输出摘要和前几个问题。

### 陷阱 2：Hook 太慢，阻塞 CC

**问题**：Hook 每次执行要 3 秒。CC 每次 Edit 都要等 3 秒。一个任务中 Edit 50 次，就多等了 150 秒。

**解决方案**：
- Hook 脚本执行时间控制在 **500ms 以内**
- 避免在 Hook 中做网络请求（除非用 `async: true`）
- 不要在 Hook 中运行完整的测试套件
- 如果需要做耗时检查，用 `async: true` 异步执行

```json
{
  "type": "command",
  "command": "bash ~/.claude/hooks/slow_check.sh",
  "async": true,
  "timeout": 30
}
```

**判断标准**：如果你运行 `time bash your_hook.sh`，超过 1 秒就该优化或改成异步。

### 陷阱 3：Hook 不该做需要理解上下文的判断

**反面教材**：
```bash
# 不要这样做：用 Hook 判断"这段话写得好不好"
if grep -q "可能" "$FILE_PATH"; then
    echo "请使用更确定的措辞"  # 错误！"可能"在很多语境下是合理的
fi
```

**正面教材**：
```bash
# 应该这样做：用 Hook 检查确定性的规则
if grep -qF "竞争对手名称" "$FILE_PATH"; then
    echo "发现敏感词"  # 正确！这个词无论在什么语境下都不该出现
fi
```

**分界线**：Hook 做"有没有"的检查（模式匹配），不做"好不好"的判断（语义理解）。

### 陷阱 4：全局 vs 项目级选择错误

**错误做法**：
- 把标书敏感词 Hook 放到全局 → 在其他项目中每次 Edit 都白白检查一遍
- 把 Python 语法检查 Hook 放到项目级 → 每个 Python 项目都要配一遍

**正确做法**：

| Hook | 应该放在 | 判断依据 |
|------|---------|---------|
| Python 语法检查 | 全局 | 所有 Python 项目都需要 |
| 标书敏感词检查 | 项目级 | 每个项目的敏感词不同 |
| SessionStart 上下文 | 全局 | 所有项目都有用 |
| 飞书通知 | 全局 | 跟项目无关 |
| DOCX 格式守护 | 项目级 | 只有文档项目需要 |

### 陷阱 5：忘记处理"文件不存在"的情况

**问题**：Hook 脚本假设文件存在，但 CC 可能传入一个即将创建（Write）或刚删除的文件路径。

**解决方案**：
```bash
# 总是检查文件是否存在
if [ -z "$FILE_PATH" ] || [ ! -f "$FILE_PATH" ]; then
    exit 0  # 静默退出，不要报错
fi
```

### 陷阱 6：忘记脚本需要 jq

Hook 脚本从 stdin 读取 JSON，需要 `jq` 来解析。确保你的系统装了 jq：

```bash
# 检查是否安装了 jq
which jq || brew install jq
```

<!-- 批注区 -->

---

## 七、完整配置汇总

把前面所有 Hook 汇总到一起，方便你直接复制使用。

### 全局配置：`~/.claude/settings.json`

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "model": "opus",
  "language": "中文",
  "skipDangerousModePermissionPrompt": true,
  "effortLevel": "high",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/check_python_syntax.sh",
            "statusMessage": "正在检查 Python 语法..."
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/session_context.sh",
            "statusMessage": "正在加载项目上下文..."
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/notify_feishu.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### 项目级配置：`~/Work/zdwp/.claude/settings.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check_sensitive_words.sh",
            "statusMessage": "正在检查敏感词..."
          }
        ]
      }
    ]
  }
}
```

### 脚本文件清单

| 脚本 | 路径 | 作用 |
|------|------|------|
| check_python_syntax.sh | `~/.claude/hooks/` | Python 语法检查 |
| session_context.sh | `~/.claude/hooks/` | 会话启动上下文注入 |
| notify_feishu.sh | `~/.claude/hooks/` | 飞书通知推送 |
| check_sensitive_words.sh | `~/Work/zdwp/.claude/hooks/` | 标书敏感词检查 |

### 快速部署命令

```bash
# 创建 Hook 脚本目录
mkdir -p ~/.claude/hooks
mkdir -p ~/Work/zdwp/.claude/hooks

# 给所有 Hook 脚本加执行权限
chmod +x ~/.claude/hooks/*.sh
chmod +x ~/Work/zdwp/.claude/hooks/*.sh

# 验证 jq 已安装
which jq || brew install jq

# 在 CC 中输入 /hooks 查看所有已加载的 Hook
```

<!-- 批注区 -->

---

## 八、TODO（留给你动手）

### 立即可做

- [ ] 给 zdwp 标书项目配一个敏感词 Hook（按 Hook 1 的步骤）
- [ ] 给所有 Python 项目配语法检查 Hook（按 Hook 2 的步骤）
- [ ] 在 CC 中输入 `/hooks` 确认 Hook 已加载

### 下一步

- [ ] 给 SessionStart 配上下文注入 Hook
- [ ] 根据实际使用情况调整敏感词列表
- [ ] 如果有飞书机器人，配通知推送 Hook

### 进阶探索

- [ ] 试试 PreToolUse Hook 拦截危险的 bash 命令（比如 `rm -rf`、`git push --force`）
- [ ] 试试 `type: "prompt"` 类型的 Hook——用 LLM 做语义级别的检查
- [ ] 探索 PostToolUseFailure，在工具执行失败时自动给 CC 提示修复方向

<!-- 批注区 -->

---

## 参考

- [Claude Code Hooks 官方文档](https://code.claude.com/docs/en/hooks) — 所有生命周期事件、配置格式、环境变量的完整参考
- [Claude Code Hooks Tutorial (Blake Crosley)](https://blakecrosley.com/blog/claude-code-hooks-tutorial) — 5 个生产级 Hook 实例
- [Claude Code Hook Examples (Steve Kinney)](https://stevekinney.com/courses/ai-development/claude-code-hook-examples) — 更多 Hook 模式
- `Layer1-长期上下文.md` / `Layer2-工具能力.md` / `Layer3-工作流Skills.md` — 六层架构的前三层
