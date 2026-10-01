# otty-collaboration

[English](README.md) | 简体中文

一个 agent 技能：让在 [Otty](https://otty.sh) 里并排运行的 Claude Code 和 Codex
会话互相交接任务，并且**主动回报**。

多数多智能体方案只讲怎么派活。这个技能还告诉接收方：什么时候该回复发起方、回复
发到哪里、什么时候保持安静。这样发起方不用一直去盯对方的窗口。

## 内容

- **选择通道**：Claude 之间用 Claude Code 原生的 `SendMessage`；只要涉及 Codex，
  就通过 `otty` CLI 往目标 pane 里输入。
- **回信信封**：每次交接都以 `[otty-handoff]` 开头，写明发送方的 pane 或会话名、
  任务单号，以及 `reply-on` 回话时机。
- **回话时机**：接收方在 `blocker`（卡住）、`conflict`（冲突）、`disagree`（有异议）
  和 `done`（完成）时回复，从不只为了说"收到"而回复。
- **防护规则**：不来回客套；同一问题三轮没谈拢就交给人；agent 不能替用户批准；
  共享任务追踪器的记录优先于终端消息。
- **实战记录**：往别的 agent 输入框里打字时的 TUI 陷阱，以及 Codex 读到过期
  `OTTY_PANE_ID` 的问题（附修复用的包装脚本）。

## 环境要求

- macOS 上的 Otty，本地运行（不支持 SSH）。测试版本：Otty 1.5.4。
- 从 Otty ▸ Settings ▸ Agents ▸ Skills 安装官方 `otty` 技能，并在那里开启
  `ipc-allow-send-keys`。
- Claude Code 和/或 Codex CLI。测试版本：Claude Code 2.1.x、Codex 0.159。
- 可选：共享任务追踪器，比如 [beads](https://github.com/steveyegge/beads)；示例用
  的是 `bd`。

## 安装

**Claude Code**（作为插件安装）：

```sh
claude plugin marketplace add VibeMage/otty-collaboration
claude plugin install otty-collaboration@otty-collaboration
```

以后用 `claude plugin update otty-collaboration@otty-collaboration` 更新。

**Codex**（用它内置的 `skill-installer` 技能）：直接对 Codex 说"从
github.com/VibeMage/otty-collaboration 安装技能，路径 skills/otty-collaboration"，
或者直接运行安装脚本：

```sh
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo VibeMage/otty-collaboration --path skills/otty-collaboration
```

安装器不会覆盖已有的副本；要更新，先删除 `~/.codex/skills/otty-collaboration`
再重新运行。

**Codex 包装脚本**（在 Otty 里用 Codex 时建议安装）：`bin/codex` 会带上当前 pane
的 `OTTY_PANE_ID` 启动真正的 Codex；不在 Otty 里时原样透传。先读一遍脚本，再把它
放到 `PATH` 里真正的 `codex` 之前：

```sh
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/VibeMage/otty-collaboration/main/bin/codex \
  -o ~/.local/bin/codex && chmod +x ~/.local/bin/codex
command -v codex   # 应输出 ~/.local/bin/codex
```

任何 `-c` 覆盖参数（包括这个）都会让 Codex 不连共享后台服务、独立运行。安装后重启
正在运行的 agent 会话，技能和包装脚本才会生效。

**修改技能本身**：克隆仓库，把 `skills/otty-collaboration` 软链接到
`~/.claude/skills/` 和 `~/.codex/skills/`，让两个 agent 都读你的工作副本。

## 信任边界

信封是纯文本，没有认证：任何能往 pane 里输入的东西都能冒充 `[otty-handoff]`。
技能把交接视为队友的请求，而不是用户的批准；它面向的是在你自己机器上、为你工作的
agent。不要用它接收来自你无法控制的 pane 的任务。

## 状态

`done` 路径已端到端验证（Claude → Codex 走 Otty，Claude → Claude 走
`SendMessage`）。`blocker`、`conflict`、`disagree` 三条路径已有规则，但实际使用
还较少。agent CLI 更新很快，`SKILL.md` 中与版本相关的说明都注明了观察到的版本。

## 许可证

MIT
