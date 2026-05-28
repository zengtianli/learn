# Claude Code 命令清单

我的 Claude Code 环境中所有可用命令的完整盘点——内置命令、自定义命令、插件命令，附使用频率数据。

> 数据来源：`~/.claude/history.jsonl`（65 条命令记录）

---

## 使用频率排行

| 排名 | 命令 | 次数 | 占比 | 类型 |
|------|------|------|------|------|
| 1 | `/rate-limit-options` | 15 | 23.1% | 内置 |
| 2 | `/usage` | 12 | 18.5% | 内置 |
| 3 | `/resume` | 9 | 13.8% | 内置 |
| 4 | `/login` | 8 | 12.3% | 内置 |
| 5 | `/mcp` | 4 | 6.2% | 内置 |
| 6 | `/model` | 3 | 4.6% | 内置 |
| 7 | `/plugin` | 3 | 4.6% | 内置 |
| 8 | `/btw` | 2 | 3.1% | 插件 |
| 9 | `/reload-plugins` | 2 | 3.1% | 内置 |
| 10 | `/branch` | 2 | 3.1% | 内置 |
| 11 | `/users` | 1 | 1.5% | 内置 |
| 12 | `/loop` | 1 | 1.5% | 内置 Skill |
| 13 | `/plan` | 1 | 1.5% | 内置 |
| 14 | `/hooks` | 1 | 1.5% | 内置 |
| 15 | `/permissions` | 1 | 1.5% | 内置 |

**规律：** API 管理类命令（rate-limit、usage）占 41.6%，会话管理（resume、login）占 26.1%。

---

## 一、内置命令

CC 自带，所有用户都有。

| 命令 | 功能 | 我用过？ |
|------|------|---------|
| `/help` | 显示帮助信息 | ❌ |
| `/init` | 初始化项目 CLAUDE.md | ❌ |
| `/login` | 登录认证 | ✅ 高频 |
| `/logout` | 退出登录 | ❌ |
| `/model` | 切换模型（opus/sonnet/haiku） | ✅ |
| `/fast` | 切换快速模式（同模型，更快输出） | ❌ |
| `/cost` | 查看当前会话费用 | ❌ |
| `/compact` | 压缩上下文 | ❌ |
| `/clear` | 清空对话历史 | ❌ |
| `/config` | 打开配置 | ❌ |
| `/permissions` | 权限管理 | ✅ |
| `/hooks` | 管理 hooks | ✅ |
| `/status` | 显示状态信息 | ❌ |
| `/doctor` | 诊断环境问题 | ❌ |
| `/bug` | 报告 bug | ❌ |
| `/review` | 代码审查 | ❌ |
| `/memory` | 管理 auto memory | ❌ |
| `/vim` | 切换 vim 模式 | ❌ |
| `/terminal-setup` | 终端集成设置 | ❌ |
| `/listen` | 监听模式 | ❌ |
| `/branch` | 分支对话 | ✅ |
| `/plan` | 进入 plan 模式 | ✅ |
| `/resume` | 恢复之前的会话 | ✅ 高频 |
| `/mcp` | MCP 服务器管理 | ✅ |
| `/plugin` | 插件管理 | ✅ |
| `/reload-plugins` | 重载插件 | ✅ |
| `/rate-limit-options` | 查看速率限制选项 | ✅ 最高频 |
| `/usage` | 查看 API 用量 | ✅ 高频 |
| `/users` | 用户管理（团队） | ✅ |

---

## 二、内置 Skill 命令

CC 内置但以 Skill 形式提供，触发条件更智能。

| 命令 | 功能 | 触发条件 |
|------|------|---------|
| `/update-config` | 通过 settings.json 配置 CC | 用户提到自动化行为 |
| `/keybindings-help` | 自定义快捷键 | 用户想改快捷键 |
| `/simplify` | 审查代码质量并优化 | 代码改完后 review |
| `/loop` | 定时循环执行命令 | 需要轮询/定时任务 |
| `/schedule` | 创建定时远程 agent | 需要 cron 式调度 |
| `/claude-api` | 用 Claude API 写代码 | 代码 import anthropic |

---

## 三、自定义命令（我的）

### 全局命令 `~/.claude/commands/`

| 命令 | 功能 | 使用场景 |
|------|------|---------|
| `/cf-dns` | Cloudflare DNS 管理 | 增删查子域名 A 记录，用 CF API |
| `/deploy` | 部署当前项目到 VPS | 检查 deploy.sh → 构建 → 部署 → 验证 |
| `/vps-status` | VPS 健康检查 | SSH 查服务、端口、磁盘、内存、Docker |

### 项目级 Skill `~/Dev/configs/.claude/skills/`

| Skill | 功能 | 触发条件 |
|-------|------|---------|
| `macos-wm` | macOS 窗口管理 | 配置 Yabai/Hammerspoon/Karabiner |
| `dotfiles` | 配置部总览 | 需要了解 dotfiles 整体架构 |
| `nvim` | Neovim 配置 | 修改 Neovim LSP/插件/快捷键 |
| `zsh` | Zsh 配置 | 修改别名/FZF/函数 |

### 全局 Skill `~/.claude/skills/`

| Skill | 功能 |
|-------|------|
| `repo-manage` | 仓库管理 SOP：repo_audit.py 审计、screenshot 截图、promote 推广 |
| `restart` | 会话恢复指南：`claude -c` 或 `claude -r SESSION_ID` |

---

## 四、插件命令

| 命令 | 来源 | 功能 |
|------|------|------|
| `/claude-hud:setup` | claude-hud 插件 | 配置状态栏插件 |
| `/claude-hud:configure` | claude-hud 插件 | 调整 HUD 布局/预设/显示元素 |
| `/btw` | 插件 | 未知（使用 2 次） |

---

## 五、从未使用的内置命令

这些命令存在但我从未用过，值得了解：

| 命令 | 为什么可能有用 |
|------|--------------|
| `/compact` | 长对话时手动压缩上下文，避免自动截断丢失关键信息 |
| `/cost` | 实时查看当前会话费用，比 `/usage` 更聚焦 |
| `/review` | 让 CC 审查代码变更，提交前质检 |
| `/fast` | 同模型更快输出，适合简单任务 |
| `/doctor` | 环境诊断，排查问题时用 |
| `/memory` | 管理自动记忆，查看/清理已存记忆 |
| `/init` | 新项目初始化 CLAUDE.md，比手写更规范 |
| `/listen` | 监听模式，适合搭配其他工具 |

---

## 配置文件位置

```
~/.claude/
├── commands/           ← 全局自定义命令（cf-dns, deploy, vps-status）
├── skills/             ← 全局 skill（repo-manage, restart）
├── settings.json       ← 全局设置（statusLine, plugins）
├── history.jsonl       ← 命令历史
├── plugins/            ← 插件缓存（claude-hud）
└── projects/
    └── -Users-tianli-Dev/
        └── commands/   ← 项目级命令（当前为空）

~/Dev/configs/.claude/
└── skills/             ← dotfiles 项目的 skill（macos-wm, nvim, zsh, dotfiles）
```
