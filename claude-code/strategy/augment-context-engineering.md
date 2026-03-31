# Augment Code Search × Context Engineering 策略

> 2026-03-16 | 目标：用 Augment 语义搜索替代 grep 猜关键词，每个会话节省 30-50% context 消耗

## 1. 前置准备：推仓库到 GitHub

| 优先级 | 仓库 | 路径 | 状态 | 备注 |
|--------|------|------|------|------|
| P0 | scripts | `~/Dev/scripts` | ✅ 已推 | zengtianli/scripts (private), develop 分支 |
| P0 | oa-project | `~/Dev/oa-project` | ✅ 已推 | zengtianli/oa-project (private), main 分支 |
| P1 | learn | `~/Learn/` | ✅ 已推 | zengtianli/learn (private) |
| P1 | website | `~/Personal/website` | ✅ 已有 | zengtianli/web |
| P2 | essays | `~/Personal/essays` | ✅ 已推 | zengtianli/essays (private) |
| P2 | resume | `~/Personal/resume` | ✅ 已推 | zengtianli/resume (private) |
| P2 | zdwp | `~/Work/zdwp` | ✅ 已推 | zengtianli/zdwp (private), 白名单模式仅文本 167MB |
| P2 | reports | `~/Work/reports` | ✅ 已推 | zengtianli/reports (private) |

命令模板：
```bash
cd ~/Dev/scripts
gh repo create zengtianli/scripts --private --source=. --push
```

## 2. 对比测试方案

### 测试思路

同一个搜索任务，分别用两种方法完成，记录 **工具调用次数** 和 **找到目标的准确率**。

### 测试场景（5 个）

| # | 搜索任务 | 难度 | 为什么选这个 |
|---|---------|------|-------------|
| T1 | "找到处理 Word 修订标记的脚本" | 低 | 有明确功能描述，baseline |
| T2 | "OA 里数据同步的逻辑在哪" | 中 | 跨文件，关键词不明显 |
| T3 | "坐标系转换相关的代码" | 中 | 可能散落在多个项目 |
| T4 | "之前学过的 RAG 笔记在哪" | 高 | 文件名和内容都不确定 |
| T5 | "Raycast command 调用某个脚本的 wrapper" | 高 | 需要理解调用关系 |

### 测试方法

每个场景执行两遍：

**方法 A — 传统 grep/glob：**
```
1. glob 找文件列表
2. grep 猜关键词（可能多轮）
3. read 确认内容
→ 记录：总轮数、是否命中、context 消耗（行数）
```

**方法 B — Augment 语义搜索：**
```
1. augment_code_search(自然语言描述)
2. read 确认（如果需要）
→ 记录：总轮数、是否命中、context 消耗（行数）
```

### 记录模板

```markdown
## T1: 找到处理 Word 修订标记的脚本

### 方法 A（grep）
- 工具调用次数：__
- 命中目标：是/否
- 关键词猜了几次：__
- 返回的无关结果行数：__

### 方法 B（Augment）
- 工具调用次数：__
- 命中目标：是/否
- 返回结果精准度：__/5（几个结果相关）

### 结论
省了 __ 轮调用，context 节省约 __%
```

## 3. Context Engineering 全局策略

### 核心公式

```
会话效率 = 有效决策 / context 消耗
```

提升效率的三个杠杆：

### 杠杆 1：搜索提速（Augment）

```
以前：glob → grep × N → read × M → 找到目标（5-6 轮）
现在：augment → read 确认（1-2 轮）
```

**适用**：不知道文件名/关键词，只知道功能描述
**不适用**：知道确切文件名或关键词（直接 grep 更快）

### 杠杆 2：主会话瘦身（Subagent）

```
主会话职责：决策 + 调度 + 验收
Subagent 职责：搜索 + 读写 + 分析

Subagent 内部用 Augment 定位 → 省去自己探索项目的 context
```

### 杠杆 3：记忆外置（Memory + 方案文件）

```
不在 context 里记东西 → 写到 memory/方案文件
下次会话直接读文件 → 不用重新搜索和推理
```

### 三者配合的典型工作流

```
用户："帮我给 OA 加一个新的部门页面"

主会话：
  ├─ Augment 搜索 "OA department page template" → 2 秒定位模板
  ├─ 读 memory 拿到 OA 架构信息 → 0 轮搜索
  ├─ 写任务描述，派 Subagent
  │
  Subagent：
  │  ├─ Augment 搜索 "data loader function" → 定位 lib/data_loader.py
  │  ├─ 参照模板写新页面
  │  └─ 返回结果
  │
  └─ 主会话验收，context 始终干净
```

## 4. 下一步

- [ ] 推 P0 仓库（scripts + oa-project）到 GitHub Private
- [ ] 等 Augment 索引完成（通常几分钟）
- [ ] 跑 T1-T5 对比测试
- [ ] 根据结果决定是否推 P1/P2 仓库
- [ ] 将 Augment 使用规则写入 CLAUDE.md
