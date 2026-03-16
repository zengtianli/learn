# 主线 C：VPS 基础设施搭建 —— 从一台裸机到 24/7 AI 工作站

> **前置阅读**：`00-总览与主线地图.md`（了解三条主线的关系）、`Layer1-长期上下文.md` / `Layer2-工具能力.md` / `Layer3-工作流Skills.md`（六层架构）
>
> **关联文件**：`~/vps/usage-guide.md`（日常操作手册）、`~/vps/domain-cloudflare-guide.md`（域名配置详解）、`~/vps/openclaw-vps-plan.md`（部署计划）
>
> **源事件**：3/13 一天内，在 VPS 上搭建了完整的远程 AI 工作站

---

## 怎么读这份文件

这份文件讲的是：你为什么需要一台 VPS，怎么在一天之内把它从一台只跑代理的服务器变成一个 24/7 运转的 AI 工作站。

**重点不是安装步骤**（那些在 `~/vps/` 目录下的操作文档里都有），而是**每个组件为什么存在、它解决什么问题、以及它们之间怎么配合**。

**七章的逻辑**：

```
第一章  全景      → 画整体架构，理解为什么需要这些组件
第二章  中转站    → OpenClaw + 飞书机器人（AI 的入口层）
第三章  Agent     → Edict 多 Agent 系统（AI 的执行层）
第四章  同步      → Syncthing 双向同步（数据流通层）
第五章  域名      → 子域名 + Nginx + CF Origin Rule（Web 访问层）
第六章  分流      → Shadowrocket 规则重构（本地网络层）
第七章  回顾      → 这套基础设施的完整价值
```

**建议读法**：

- 如果你想**理解整体设计**：重点读第一章和第七章
- 如果你想**理解某个具体组件**：按章节跳读
- 如果你想**复习踩坑经历**：第五章有最密集的踩坑记录
- 每章末尾的 `<!-- 批注区 -->` 是留给你追问的位置

---

## 第一章：为什么需要 VPS —— 整体架构

### 1.1 一个核心问题

3/13 之前，你的 AI 工作流是这样的：

```
打开 Mac → 启动 Claude Code → 干活 → 合上电脑 → AI 停了
```

问题很明显：**AI 只在你坐在电脑前时才能工作**。

你想要的是：手机上发一条飞书消息，VPS 上的 AI 就开始干活。你在地铁上、在工地上、在开会——AI 都在后台运转。等你回到电脑前，打开文件夹，活已经干完了。

这就是 VPS 的核心价值：**把 AI 能力从"人在电脑前"变成"随时随地可触发"**。

### 1.2 全景架构图

这是 3/13 之后你的完整系统架构：

```
本地 Mac（tianli 的工作机）
  ├── Claude Code（日常主力，直接交互）
  ├── 工作文件
  │     ├── ~/Work/zdwp/      （水利公司项目）
  │     ├── ~/Personal/essays/ （论文）
  │     ├── ~/Learn/           （学习笔记）
  │     └── ~/Work/reports/    （工作报告）
  │
  ╠══════ Syncthing 双向实时同步 ══════╗
  │                                     │
VPS (104.218.100.67)                    │
  ├── OpenClaw Gateway (:18789)         │
  │     └── MMKG 中转站 (code.mmkg.cloud) → Claude API
  ├── 飞书机器人（长连接，移动入口）     │
  │     └── 飞书 WebSocket → OpenClaw → Claude
  ├── Edict 三省六部（12 个 Agent）     │
  │     └── 看板 Dashboard (:7891)       │
  ├── 工作文件副本 /root/sync/ ←────────┘
  │     ├── /root/sync/zdwp/
  │     ├── /root/sync/essays/
  │     ├── /root/sync/learn/
  │     └── /root/sync/reports/
  ├── Xray Reality (:443)（代理，核心功能）
  ├── Nginx 反向代理 (:8443)
  │     ├── panel.tianlizeng.cloud → Marzban (:8000)
  │     ├── board.tianlizeng.cloud → Edict 看板 (:7891)
  │     └── sub.tianlizeng.cloud   → Marzban 订阅 (:8000)
  └── Marzban (Docker, :8000)
```

### 1.3 每个组件解决什么问题

先不看技术细节，只看每个组件**为什么存在**：

| 组件 | 解决的问题 | 没有它会怎样 |
|------|-----------|-------------|
| OpenClaw | API 统一管理 | 每个服务各自直连 Claude API，无法统一监控 token 消耗和计费 |
| 飞书机器人 | 移动触发入口 | 只能坐在电脑前才能给 AI 发指令 |
| Edict (12 Agent) | 复杂任务自动编排 | 每个任务都要你手动拆解、手动分配 |
| Syncthing | 文件双向同步 | VPS 上的 Agent 改了文件，你在本地看不到 |
| Nginx + 子域名 | Web 管理入口 | 每次看面板都要开 SSH 隧道 |
| Shadowrocket 规则 | 智能分流 | 国内网站也走代理，浪费流量且变慢 |

> **关键洞察**：这些组件不是孤立的，它们形成了一条完整的链路——**触发 → 执行 → 同步 → 管理**。飞书机器人是触发器，Edict 是执行引擎，Syncthing 是数据管道，子域名是管理窗口。少了任何一环，整套系统的价值都会打折。

### 1.4 本地 CC vs VPS Edict：什么时候用哪个

这两个不是替代关系，是互补关系：

| 场景 | 用本地 CC | 用 VPS Edict |
|------|----------|-------------|
| 写代码、改 CLAUDE.md | 是 | 否 |
| 写报告、改文档 | 是（坐在电脑前）| 是（不在电脑前） |
| 需要多轮对话微调 | 是 | 否（飞书交互不够灵活） |
| 定时巡检、自动汇总 | 否 | 是 |
| 紧急任务、不在电脑旁 | 否 | 是 |

简单说：**复杂交互用本地 CC，触发式任务用 VPS Edict**。

<!-- 批注区：回想一下，你日常有哪些任务是"不需要坐在电脑前也能触发"的？列出来，这就是 Edict 的适用场景。 -->

---

## 第二章：OpenClaw + 飞书机器人 —— AI 的入口层

### 2.1 OpenClaw 是什么

OpenClaw 是 Claude API 的中转和管理平台。你可以把它理解为一个"AI 调度中心"：

```
                        ┌─── 飞书机器人
                        │
所有 AI 请求 ──→ OpenClaw ──→ MMKG 中转站 ──→ Claude API
                        │
                        ├─── Edict (12 Agent)
                        │
                        └─── 未来的其他服务
```

**为什么需要中转，不直接调 Claude API？**

| 直连 Claude API | 通过 OpenClaw 中转 |
|----------------|-------------------|
| 每个服务各自管理 API key | 一个入口统一管理 |
| token 消耗分散，算不清楚 | 集中监控，知道哪个 Agent 花了多少 |
| 每个服务各自处理限流 | 统一限流和排队 |
| 换 API 提供商要改每个服务 | 只改 OpenClaw 配置 |

### 2.2 VPS 上的安装要点

OpenClaw 在 VPS 上的安装和本地有几个关键差异：

| 项目 | 本地 macOS | VPS Ubuntu |
|------|-----------|------------|
| Daemon 管理 | launchd (plist) | systemd (service) |
| Node.js | 已有 | 通过 NodeSource 安装 (>=22) |
| pnpm | 已有 | 版本 10.32.0 |
| 网络 | 需要 Shadowrocket 代理 | 直连（VPS 在海外） |
| Gateway 绑定 | loopback (127.0.0.1) | loopback + 内部服务通过本地端口连接 |

**最终配置**：
- OpenClaw 版本：2026.3.8
- Gateway 地址：`ws://127.0.0.1:18789`
- 运行方式：systemd 服务（开机自启）
- 中转站：`code.mmkg.cloud`

> **设计决策**：Gateway 绑定 loopback (127.0.0.1) 而不是 0.0.0.0。原因：Gateway 只需要被 VPS 本机上的飞书机器人和 Edict 访问，不需要对外暴露。安全原则——不对外暴露的端口就不暴露。

### 2.3 飞书机器人：移动入口

飞书机器人是整套系统的"遥控器"——你在手机上发一条消息，VPS 上的 AI 就开始工作。

**创建过程**：
1. 在飞书开放平台（open.feishu.cn）创建企业自建应用，叫「AI助手」
2. 配置权限：`im:message`（接收消息）、`im:message:send_as_bot`（发送消息）等
3. 事件订阅方式：**长连接**（WebSocket）

> **为什么用长连接而不是 Webhook？** Webhook 需要一个公网可访问的回调 URL，意味着你要额外开一个端口、配一个域名、处理 SSL。长连接（WebSocket）不需要——飞书主动连过来，你的 VPS 只需要能访问飞书的服务器就行。对于自用机器人，长连接是最省事的方案。

**完整链路**：

```
你在手机飞书发消息
       ↓
飞书云端服务器
       ↓ WebSocket（飞书主动推送到 VPS）
VPS 上的飞书机器人进程
       ↓ 调用 OpenClaw API
OpenClaw Gateway (ws://127.0.0.1:18789)
       ↓
MMKG 中转站 (code.mmkg.cloud)
       ↓
Claude API
       ↓ 回复原路返回
你的飞书收到 AI 回复
```

**教学点**：飞书机器人的价值不只是"能在手机上聊天"。它的真正价值是**打通了"人不在电脑前"这个场景**——你可以在地铁上给 AI 下达"帮我整理这周的工作日志"这样的指令，等到了办公室打开电脑，文件已经通过 Syncthing 同步到本地了。

<!-- 批注区：试着想一个你在工地或开会时可能需要 AI 帮忙的场景，这就是飞书机器人的典型用例。 -->

---

## 第三章：Edict 多 Agent 系统 —— 古廷架构

### 3.1 为什么需要多个 Agent

一个 Claude 实例能力很强，但有一个根本限制：**上下文窗口**。

一个 Agent 如果同时要理解任务背景、拆解子任务、执行代码、审查质量——它的上下文会被塞满，质量下降。

Edict 的设计思路是：**把一个全能 Agent 拆成多个专职 Agent，每个只管自己的事**。这跟 `Layer5-Subagents.md` 里 tw93 文章说的 Subagent 设计原则一致——每个 Agent 有明确的职责边界，互不干扰。

### 3.2 古廷架构：12 个 Agent 的分工

Edict 用中国古代朝廷的架构来命名和组织 Agent。这不只是取名好玩——古廷架构本身就是一套成熟的分工体系：

```
                    ┌──────────────┐
                    │   太子(taizi)  │  ← 入口，分拣和路由
                    │   统筹调度     │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
       ┌────────────┐ ┌────────────┐ ┌────────────┐
       │   中书省     │ │   门下省     │ │   尚书省     │
       │   起草规划   │ │   审核质控   │ │   派发执行   │
       └────────────┘ └────────────┘ └─────┬──────┘
                                           │
              ┌──────┬──────┬──────┬──────┬┴─────┐
              ↓      ↓      ↓      ↓      ↓      ↓
           ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐
           │吏部││户部││礼部││兵部││刑部││工部│
           │人事││数据││文档││代码││合规││工程│
           └────┘└────┘└────┘└────┘└────┘└────┘
```

| Agent | 角色 | 职责 | 什么时候被调用 |
|-------|------|------|-------------|
| 太子 | 统筹 | 接收飞书消息，判断是闲聊还是任务，路由到对应流程 | 每次收到消息 |
| 中书省 | 起草 | 把任务拆解为子任务，制定执行方案 | 太子判断为正式任务时 |
| 门下省 | 审核 | 审查中书省的方案质量，不合格打回重做 | 中书省出方案后 |
| 尚书省 | 派发 | 把审核通过的任务分配给对应的六部 | 门下省审核通过后 |
| 吏部 | 人事 | 人员相关任务 | 尚书省指派 |
| 户部 | 数据 | 数据分析、统计 | 尚书省指派 |
| 礼部 | 文档 | 文档写作、排版 | 尚书省指派 |
| 兵部 | 代码 | 代码编写、脚本开发 | 尚书省指派 |
| 刑部 | 合规 | 合规检查、规范审查 | 尚书省指派 |
| 工部 | 工程 | 工程类、部署类任务 | 尚书省指派 |

> 这套架构的具体分工你可以根据实际需要调整。上面的对应关系是从实际对话历史推断的，不是固定不变的。重要的不是"吏部一定管人事"，而是"每个 Agent 有且只有一个明确的职责边界"。

### 3.3 任务流转过程

一条飞书消息变成一个完成的任务，经过这样的流程：

```
你发飞书消息："帮我整理水利公司的项目周报"
       ↓
太子：这是正式任务，转中书省
       ↓
中书省：拆解为 3 个子任务
  1. 读取 /root/sync/zdwp/ 下本周修改的文件
  2. 按项目分类汇总
  3. 生成周报 Markdown
       ↓
门下省：审核通过（方案合理，文件路径正确）
       ↓
尚书省：子任务 1、2 给户部（数据），子任务 3 给礼部（文档）
       ↓
户部 + 礼部：并行执行
       ↓
结果通过飞书回复给你
```

你可以通过看板 Dashboard（`board.tianlizeng.cloud` 或 SSH 隧道 `localhost:7891`）实时看到任务在哪个阶段。看板有 10 个面板，包括旨意看板（Kanban）、省部调度、奏折阁等——详见 `~/vps/usage-guide.md` 的看板章节。

### 3.4 从 /tmp 迁移到 /opt/edict/

最初 Edict 部署在 `/tmp` 目录下。这是一个典型的"先跑起来再说"的决定，但 `/tmp` 是临时目录——系统重启可能清空、其他进程可能干扰。

迁移到 `/opt/edict/` 是正式化的标志：

| | /tmp/edict/ | /opt/edict/ |
|---|---|---|
| 持久性 | 系统重启可能清空 | 永久保留 |
| 语义 | "临时试试" | "正式部署的第三方软件" |
| 权限管理 | 所有人可读写 | 可以限制权限 |
| 运维习惯 | 不会备份 /tmp | /opt 是标准部署路径 |

> **经验法则**：如果一个服务你打算长期运行，就不要放在 /tmp。`/opt/` 是 Linux 下第三方软件的标准安装路径，`/srv/` 是服务数据的标准路径。选哪个都行，但别用 /tmp。

### 3.5 教学点：Agent 的价值在于职责边界

回到 `Layer5-Subagents.md` 里的观点：Edict 的价值不在于 Agent 数量多，而在于**每个 Agent 有明确的职责边界**。

为什么这很重要？因为 LLM 的一个根本特点是：**你给它越聚焦的上下文，它表现越好**。一个只负责"审核方案质量"的门下省 Agent，比一个"又要写方案又要审核又要执行"的全能 Agent，在审核这件事上一定做得更好——因为它的系统提示、上下文、工具都围绕"审核"这一件事优化。

<!-- 批注区：想想你在本地用 CC 时，有没有遇到过"一个会话承担太多职责导致质量下降"的情况？ -->

---

## 第四章：Syncthing 双向同步 —— 数据流通层

### 4.1 为什么需要文件同步

VPS 上的 Edict Agent 执行任务时需要读写文件——你的水利项目文档、论文、学习笔记。这些文件都在你的 Mac 上，VPS 上没有。

你需要一个机制，让 Mac 上的文件和 VPS 上的文件保持一致。

### 4.2 三选一：为什么是 Syncthing

> **决策记录**：文件同步方案选型
>
> - [ ] **SSHFS**：把 VPS 的目录挂载到本地（或反过来）
>   - 优点：配置简单，不占双份空间
>   - 缺点：需要持续网络连接，断网就无法访问文件；延迟敏感（每次读写都走网络）
>   - 否决原因：你的 Mac 经常合盖休眠、切换网络，SSHFS 会频繁断开
>
> - [ ] **Git**：用 Git 仓库同步
>   - 优点：有版本历史，可以回滚
>   - 缺点：需要手动 commit/push/pull；不适合频繁小改（每改一个字就要 commit？）
>   - 否决原因：文档类内容修改频繁且细碎，手动同步太麻烦
>
> - [x] **Syncthing**（推荐）：P2P 实时双向同步
>   - 优点：自动同步、离线也能工作、来网自动合并、不依赖第三方服务器
>   - 缺点：占双份存储空间、大文件首次同步慢
>   - 选择原因：文档目录总计约 500MB，双份存储不是问题；自动化是核心需求

### 4.3 同步的 4 个目录

| Mac 路径 | VPS 路径 | 内容 | 大约大小 |
|----------|----------|------|---------|
| `~/Work/zdwp/` | `/root/sync/zdwp/` | 水利公司项目 | ~200MB |
| `~/Personal/essays/` | `/root/sync/essays/` | 论文 | ~100MB |
| `~/Learn/` | `/root/sync/learn/` | 学习笔记 | ~50MB |
| `~/Work/reports/` | `/root/sync/reports/` | 工作报告 | ~150MB |

总计约 500MB。

### 4.4 什么不同步

> **重要原则**：不同步代码仓库（有 Git 管），只同步文档类内容。

| 目录 | 是否同步 | 原因 |
|------|---------|------|
| `~/Work/zdwp/` | 是 | 文档为主，没有 Git |
| `~/Personal/essays/` | 是 | 论文、文档 |
| `~/Dev/scripts/` | **否** | 代码仓库，用 Git 管理 |
| `~/Dev/oa-project/` | **否** | 代码仓库，用 Git 管理 |
| `~/Personal/website/` | **否** | 代码仓库，用 Git 管理 |

为什么代码仓库不用 Syncthing？因为 Syncthing 的冲突解决策略是"保留两个版本让你选"——对文档来说没问题，对代码来说可能导致 `.git` 目录损坏、合并冲突搞乱仓库状态。代码有 Git，用 Git 的分支和合并机制就好。

### 4.5 同步的实际工作流

```
场景：你在地铁上用飞书给 AI 发指令"整理水利项目周报"

1. 飞书消息 → VPS 太子 → 中书省 → 户部
2. 户部在 VPS 的 /root/sync/zdwp/ 中读取文件、生成周报
3. 生成的周报文件 /root/sync/reports/weekly-report-0313.md
4. Syncthing 检测到文件变化，秒级同步到你 Mac 的 ~/Work/reports/
5. 你到办公室后，打开 ~/Work/reports/weekly-report-0313.md 就能看到

反过来也一样：
6. 你在 Mac 上修改了周报，保存
7. Syncthing 秒级同步回 VPS 的 /root/sync/reports/
8. 下次 AI 操作时读到的就是你改过的版本
```

**Syncthing 的运行方式**：
- Mac 端：Syncthing 应用，后台运行
- VPS 端：systemd 服务 (`syncthing@root`)，开机自启
- 同步延迟：通常秒级（取决于网络）
- Mac 休眠恢复：自动补同步，不丢数据

<!-- 批注区：你有没有遇到过 Syncthing 冲突的情况？冲突文件会带一个 `.sync-conflict-` 后缀，需要手动选择保留哪个版本。 -->

---

## 第五章：子域名 + Nginx + CF Origin Rule —— Web 访问层

这一章踩坑最多，也最值得详细记录。

### 5.1 需求

VPS 上有多个 Web 服务需要通过浏览器访问：

| 服务 | 端口 | 用途 |
|------|------|------|
| Marzban 面板 | 8000 | 管理代理账号 |
| Edict 看板 | 7891 | 查看 Agent 任务状态 |
| Marzban 订阅 | 8000 | Shadowrocket 拉取节点配置 |

3/13 之前，每次访问都要开 SSH 隧道：

```bash
# 看 Marzban 面板
ssh -L 8000:localhost:8000 root@104.218.100.67
# 然后浏览器打开 http://127.0.0.1:8000/dashboard/

# 看 Edict 看板
ssh -L 7891:localhost:7891 root@104.218.100.67
# 然后浏览器打开 http://127.0.0.1:7891
```

每次都要开终端、敲命令、维持连接。能不能直接在浏览器输入一个域名就访问？

### 5.2 难点：443 端口被占了

正常思路是：Nginx 监听 443 端口，配 SSL 证书，根据域名分流。但你的 VPS 上 443 端口已经被 Xray Reality 占了——这是代理的核心功能，**绝对不能让**。

```
端口 443 → Xray Reality → 处理代理流量（Shadowrocket 连这个）
端口 443 → Nginx        → 处理 Web 访问（浏览器连这个）
              ↑
         两个服务不能同时监听同一个端口！
```

有两个解决思路：

| 方案 | 做法 | 优缺点 |
|------|------|--------|
| A. Xray fallback | Xray 监听 443，把非代理流量 fallback 给 Nginx | 配置复杂，Xray 升级可能破坏配置 |
| B. Nginx 另用端口 + CF 端口重写 | Nginx 监听 8443，让 Cloudflare 把 443 流量转到 8443 | 简单直接，互不干扰 |

> **决策**：选方案 B。原因：Xray 和 Nginx 完全解耦，互不影响。Xray 升级不用担心 Nginx 配置，Nginx 配置也不用管 Xray 的 fallback 语法。

### 5.3 最终架构

```
用户浏览器
  ↓ HTTPS 请求 (:443)
  ↓ 例如 https://panel.tianlizeng.cloud
  ↓
Cloudflare CDN（全球节点）
  ├── SSL 终止（用户 ↔ CF 之间的 HTTPS 由 CF 处理）
  ├── Origin Rule: 把端口从 443 重写为 8443
  └── 用 Origin Certificate 加密回源（CF ↔ VPS）
  ↓ HTTPS (:8443)
  ↓
VPS Nginx（监听 :8443，SSL 用 CF Origin 证书）
  ├── server_name = panel.tianlizeng.cloud → proxy_pass localhost:8000
  ├── server_name = board.tianlizeng.cloud → proxy_pass localhost:7891
  └── server_name = sub.tianlizeng.cloud   → proxy_pass localhost:8000

与此同时，完全不受影响：
VPS Xray（监听 :443）→ 处理代理流量
```

**关键配置层级**：

| 层 | 配置 | 在哪设置 |
|----|------|---------|
| DNS | 三个 A 记录（panel/board/sub）→ VPS IP，开启 Proxied（橙色云） | Cloudflare 后台 |
| SSL | Full 模式 + Origin Certificate（通配符 *.tianlizeng.cloud，15 年有效） | Cloudflare 后台 |
| 端口重写 | Origin Rule：当主机名匹配 *.tianlizeng.cloud 时，端口改为 8443 | Cloudflare 后台 |
| 反向代理 | Nginx 监听 8443，按 server_name 分发到不同 localhost 端口 | VPS /etc/nginx/ |

### 5.4 踩坑记录

这一步踩了三个坑，每个都值得记住。

**坑 1：子域名拼错**

在 Cloudflare 后台添加 DNS 记录时，把 `panel` 拼成了 `pannel`（多了一个 n）。结果 `panel.tianlizeng.cloud` 解析不到 VPS IP，浏览器报 DNS 错误。

教训：**域名配置出问题时，第一步永远是检查拼写**。用 `dig panel.tianlizeng.cloud` 或 `nslookup` 验证 DNS 解析结果。

**坑 2：Shadowrocket 本地 DNS 劫持干扰测试**

在 Mac 上测试 `curl https://panel.tianlizeng.cloud` 时，发现请求被 Shadowrocket 拦截了——因为 Shadowrocket 的代理规则把这个域名走了 PROXY。

```
你以为的链路：Mac → Cloudflare → VPS Nginx
实际的链路：  Mac → Shadowrocket → VPS 代理 → Cloudflare → VPS Nginx
                    ↑ 这里多了一跳！
```

结果请求绕了一大圈，而且可能因为 Shadowrocket 的 DNS 解析和 Cloudflare 的 DNS 解析不一致，导致连接失败。

解决：在 Shadowrocket 规则里给自己的域名加 DIRECT 规则：

```
DOMAIN-SUFFIX,tianlizeng.cloud,DIRECT
```

教训：**本地有代理时，测试网络问题要先排除代理干扰**。最简单的方法：暂时关掉代理再测。

**坑 3：CF Origin Rule 端口重写不是即时生效**

在 Cloudflare 后台设置了 Origin Rule（443 → 8443），立刻测试，发现不生效——请求还是打到 443 端口。等了几分钟后自动生效了。

教训：**Cloudflare 的配置变更需要传播时间，通常 1-5 分钟**。不要因为没立刻生效就怀疑配置写错了。等几分钟再测，同时检查 Cloudflare 的 Audit Log 确认配置确实保存了。

### 5.5 最终效果

配好之后的日常体验：

| 之前 | 之后 |
|------|------|
| 开终端，敲 `ssh -L 8000:localhost:8000 root@...`，打开 `http://127.0.0.1:8000` | 浏览器直接打开 `https://panel.tianlizeng.cloud` |
| SSH 隧道断了就访问不了 | 随时随地访问（手机也行） |
| 订阅链接暴露 VPS IP | 订阅链接用域名，IP 被 Cloudflare 隐藏 |

> **附带好处**：Marzban 的订阅地址从 `http://104.218.100.67:8000/sub/xxx` 变成了 `https://sub.tianlizeng.cloud/sub/xxx`。好处：HTTPS 加密、隐藏 IP、以后换 VPS 只改 DNS 记录客户端不用动。

<!-- 批注区：你还记得配置 Origin Rule 的具体步骤吗？如果忘了，详细的操作步骤在 ~/vps/domain-cloudflare-guide.md 里。 -->

---

## 第六章：Shadowrocket 规则重构 —— 本地网络层

### 6.1 原来的问题

你的 Shadowrocket 配置文件里有大约 200 条手动维护的 DIRECT 规则——每次遇到一个国内网站被误走代理，就手动加一条。

```
# 之前的 shadowrocket.conf 里到处是这种：
DOMAIN-SUFFIX,taobao.com,DIRECT
DOMAIN-SUFFIX,jd.com,DIRECT
DOMAIN-SUFFIX,bilibili.com,DIRECT
DOMAIN-SUFFIX,zhihu.com,DIRECT
... 再来 196 条 ...
```

问题：
- **覆盖不全**：国内网站几千个，你手动加了 200 个，漏的还是多
- **维护成本高**：每次遇到新站点就要手动改配置
- **配置文件膨胀**：200 条规则让配置文件又长又难读

### 6.2 解决方案：RULE-SET 引用社区维护的列表

Shadowrocket 支持 `RULE-SET`——引用外部规则列表，不用自己维护。

`blackmatrix7/ios_rule_script` 是一个社区维护的规则集，其中 `ChinaMax` 列表包含 3600+ 个国内域名和 IP 段，远比你手动维护的 200 条全面。

**改造思路**：

```
改造前：
  200 条手动 DIRECT 规则
  + 几十条手动 PROXY 规则
  = 配置文件很长，覆盖不全

改造后：
  RULE-SET（ChinaMax，3600+ 条，自动更新）→ DIRECT
  + ~20 条自定义规则（个人特殊需求）→ PROXY/DIRECT
  = 配置文件简短，覆盖全面
```

### 6.3 改造后的规则结构

```
[Rule]
# 1. 个人特殊规则（最高优先级，约 20 条）
DOMAIN-SUFFIX,feishu.cn,DIRECT           ← 飞书走直连
DOMAIN-SUFFIX,code.mmkg.cloud,DIRECT     ← 中转站走直连
DOMAIN-SUFFIX,tianlizeng.cloud,DIRECT    ← 自己的域名走直连
... Apple Intelligence 走代理 ...
... AI 服务走代理 ...

# 2. 社区维护的国内列表（自动更新，3600+ 条）
RULE-SET,https://raw.githubusercontent.com/.../ChinaMax.list,DIRECT

# 3. 兜底规则
GEOIP,CN,DIRECT       ← 中国 IP 直连
FINAL,PROXY            ← 其他全走代理
```

### 6.4 效果对比

| | 改造前 | 改造后 |
|---|---|---|
| 手动规则数 | ~200 条 | ~20 条 |
| 国内覆盖率 | 低（遇到才加） | 高（3600+ 条社区维护） |
| 维护方式 | 每次遇到新站手动加 | 列表自动更新 |
| 配置文件可读性 | 差（一大堆 DIRECT） | 好（结构清晰，三层优先级） |

> **设计原则**：自定义规则只放"社区列表覆盖不到的特殊需求"——比如 `code.mmkg.cloud`（你自己的中转站）、`tianlizeng.cloud`（你自己的域名）。这些社区列表不可能有，所以需要手动加。其他通用的国内网站，交给社区列表维护。

<!-- 批注区：配置文件在 ~/vps/shadowrocket.conf，如果以后遇到新的需要特殊处理的域名，加在第 1 部分（个人特殊规则）就行。 -->

---

## 第七章：回顾 —— 这套基础设施的完整价值

### 7.1 组件总览

| 组件 | 解决什么问题 | 日常怎么用 |
|------|-------------|-----------|
| OpenClaw | API 统一管理 | 所有 AI 请求走中转，可监控用量和 token 消耗 |
| 飞书机器人 | 移动入口 | 手机发消息就能触发 VPS 上的 AI 执行任务 |
| Edict 三省六部 | 7x24 Agent 系统 | 定时任务、自动巡检、复杂任务自动编排 |
| Syncthing | 文件双向同步 | 本地改文件 VPS 自动同步，反之亦然 |
| 子域名 + Nginx | Web 管理入口 | 浏览器直接访问各个管理面板，不用 SSH 隧道 |
| Shadowrocket 规则 | 智能分流 | 国内直连、国外代理，自动维护，不用手动加规则 |

### 7.2 端口全景

3/13 之后，VPS 上的端口分布：

```
端口      服务                 监听方式      外部访问方式
────────────────────────────────────────────────────────
22        SSH                  0.0.0.0       ssh root@104.218.100.67
443       Xray Reality         0.0.0.0       Shadowrocket 客户端
1080      Shadowsocks          0.0.0.0       客户端连接
8443      Nginx 反向代理       0.0.0.0       CF → 域名访问
7891      Edict 看板           127.0.0.1     通过 Nginx 反代
8000      Marzban 面板         127.0.0.1     通过 Nginx 反代
8384      Syncthing Web UI     127.0.0.1     SSH 隧道（偶尔用）
18789     OpenClaw Gateway     127.0.0.1     内部服务调用
```

注意 `127.0.0.1` 和 `0.0.0.0` 的区别：

- `127.0.0.1`（loopback）：只有 VPS 本机能访问，外部无法直连。安全。
- `0.0.0.0`：所有网络接口都能访问，包括外部。必须有防护（如 Cloudflare、防火墙）。

**安全原则**：内部服务（OpenClaw、Edict 看板、Marzban）都绑定 127.0.0.1，通过 Nginx 反代或 SSH 隧道间接访问。不对外暴露不必要的端口。

### 7.3 核心逻辑

回到第一章的问题：这套系统到底在解决什么？

**一句话：让 AI 能力从"坐在电脑前才能用"变成"随时随地可触发、自动执行、结果自动同步"。**

画成链路：

```
触发（飞书/浏览器）
  ↓
路由（OpenClaw 统一入口）
  ↓
执行（Edict 12 Agent 自动编排）
  ↓
同步（Syncthing 结果自动同步到本地）
  ↓
管理（子域名 Web 面板随时查看状态）
```

### 7.4 这一天做了什么

3/13 这一天的产出，按时间线：

| 时间段 | 做了什么 | 对应章节 |
|--------|---------|---------|
| 上午 | 安装 OpenClaw + 配置飞书机器人 | 第二章 |
| 中午 | 部署 Edict 三省六部，从 /tmp 迁到 /opt | 第三章 |
| 下午 | 配置 Syncthing 同步 4 个目录 | 第四章 |
| 傍晚 | 搞子域名 + Nginx + CF Origin Rule（踩坑最多） | 第五章 |
| 晚上 | 重构 Shadowrocket 规则 | 第六章 |

一天之内从零搭建了完整的远程 AI 工作站。这个速度说明什么？说明 **Claude Code 本身就是搭建这套基础设施的最佳工具**——你不需要精通 Linux 运维，CC 帮你写配置文件、排查错误、调试网络。你只需要知道"我要什么"（架构设计），CC 负责"怎么实现"（具体操作）。

### 7.5 下一步可以做什么

| 方向 | 说明 | 优先级 |
|------|------|--------|
| Edict 任务模板化 | 把常用任务（周报整理、文档审阅）做成看板模板 | 高 |
| 监控告警 | OpenClaw token 消耗超阈值时飞书通知 | 中 |
| 自动备份 | VPS 关键配置定期备份到本地 | 中 |
| Zero Trust | 给 Web 面板加 Cloudflare Zero Trust 认证 | 低（目前够安全） |

<!-- 批注区：回顾这套系统运行到现在（3/15），有哪些组件用得多、哪些用得少？用得少的是不是说明当初的需求判断有偏差，还是只是还没养成习惯？ -->

---

## 附录：关键文件索引

| 文件 | 说明 |
|------|------|
| `~/vps/CLAUDE.md` | VPS 项目总览，已部署服务清单 |
| `~/vps/usage-guide.md` | 日常使用手册（飞书命令、看板操作、运维命令） |
| `~/vps/domain-cloudflare-guide.md` | 域名 + CF + Nginx 的详细知识文档和操作步骤 |
| `~/vps/openclaw-vps-plan.md` | OpenClaw VPS 部署计划（含本地 vs VPS 差异对比） |
| `~/vps/vps-proxy-guide.md` | VPS 硬件、协议、网络优化原理 |
| `~/vps/shadowrocket.conf` | Shadowrocket 配置文件（重构后版本） |
| `~/vps/origin.pem` | Cloudflare Origin Certificate（公钥） |
| `~/vps/origin-key.pem` | Cloudflare Origin Certificate（私钥） |
