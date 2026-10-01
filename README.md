# otty-collaboration

English | [简体中文](README.zh-CN.md)

An agent skill that lets Claude Code and Codex sessions hand work to each other
and **report back** while running side by side in [Otty](https://otty.sh).

Most multi-agent setups only describe how to dispatch work. This skill also
tells the receiving agent when to answer its owner, how to address the answer,
and when to stay quiet, so the owner does not have to keep polling the other
pane.

## What it covers

- **Channel choice**: Claude ↔ Claude uses Claude Code's native `SendMessage`;
  anything involving Codex types into the target pane through the `otty` CLI.
- **Exact addressing**: resolve the Claude in a given Otty pane to its session
  socket (pane → PID → `/tmp/cc-socks/<pid>.sock`) instead of guessing among
  `ListAgents` names; native delivery never touches a user's draft.
- **Reply envelope**: every handoff starts with `[otty-handoff]` and names the
  sender's pane or session name, tracker issue and `reply-on` triggers.
- **Reply triggers**: the recipient answers on `blocker`, `conflict`,
  `disagree` and `done`, and never just to acknowledge.
- **Guards**: no reply ping-pong, escalate to the human after three rounds
  without agreement, agents never approve on the user's behalf, and the shared
  tracker outranks terminal messages.
- **Field notes**: TUI pitfalls when typing into another agent's composer, and
  Codex seeing a stale `OTTY_PANE_ID` (with a wrapper that fixes it).

## Requirements

- macOS with Otty, run locally (not over SSH). Tested with Otty 1.5.4.
- The vendor `otty` skill installed from Otty ▸ Settings ▸ Agents ▸ Skills, and
  `ipc-allow-send-keys` enabled there.
- Claude Code and/or Codex CLI. Tested with Claude Code 2.1.x and Codex 0.159.
- Optional: a shared tracker such as [beads](https://github.com/steveyegge/beads);
  the examples use `bd`.

## Install

**Claude Code** (as a plugin):

```sh
claude plugin marketplace add VibeMage/otty-collaboration
claude plugin install otty-collaboration@otty-collaboration
```

Update later with `claude plugin update otty-collaboration@otty-collaboration`.

**Codex** (with its built-in `skill-installer` skill): ask Codex to
"install the skill from github.com/VibeMage/otty-collaboration, path
skills/otty-collaboration", or run the installer script directly:

```sh
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo VibeMage/otty-collaboration --path skills/otty-collaboration
```

The installer refuses to overwrite an existing copy; to update, delete
`~/.codex/skills/otty-collaboration` and run it again.

**Codex wrapper** (recommended when Codex runs in Otty): `bin/codex` starts the
real Codex with this pane's `OTTY_PANE_ID` pinned, and passes through unchanged
outside Otty. Read it, then put it ahead of the real `codex` on `PATH`:

```sh
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/VibeMage/otty-collaboration/main/bin/codex \
  -o ~/.local/bin/codex && chmod +x ~/.local/bin/codex
command -v codex   # should print ~/.local/bin/codex
```

Any `-c` override, including this one, makes Codex run without its shared
background server. Restart running agent sessions to pick up the skill and the
wrapper.

**Working on the skill itself**: clone the repository and symlink
`skills/otty-collaboration` into `~/.claude/skills/` and `~/.codex/skills/`
instead, so both agents read your working copy.

## Delivery timing with Codex

Observed with Codex 0.159.2 in Otty 1.5.4, using default key bindings in a
normally configured working session:

- **Enter delivers promptly.** Text submitted with Enter while Codex is working
  shows as "Messages to be submitted after next tool call" and reaches the model
  at the next tool boundary, in the same turn. The skill tells senders to use
  Enter. Startup, plan streaming and recovery states can still defer it; verify
  the observed delivery state.
- **Tab defers while working.** When idle, Tab may submit immediately. During
  a running turn, text submitted with Tab lands under "Queued follow-up inputs"
  and reaches the model only when the current turn ends. The skill describes how
  an owner can safely restore and resubmit a deferred peer reply.
- **`codex queue --thread <uuid> --message <text>` is also deferred** (delivered
  after the current turn), and only reaches sessions on Codex's shared
  background server.
- **The `bin/codex` wrapper runs Codex standalone** (any `-c` override does), so
  `codex queue` and `codex app-server proxy` cannot reach that TUI by default.
  Typing into its pane is the delivery path.
- **No timer or background watcher.** Senders verify timely delivery; owners
  read replies as they arrive. Tracker-first checks are a fallback before
  dependent actions or ending a turn, once for a clearly overdue reply, or when
  delivery trouble is reported. Pane capture and queue recovery are used only
  when the tracker shows a report that has not reached the model.

## Trust boundary

Envelopes are plain text and not authenticated: anything that can type into a
pane can claim to be `[otty-handoff]`. The skill treats a handoff as a
teammate's request, never as the user's approval, and it is meant for agents on
your own machine working for you. Do not use it to accept work from panes you
do not control.

## Status

Validated end to end for the `done` path (Claude → Codex over Otty, Claude →
Claude over `SendMessage`), and for mid-turn delivery to a working Codex with
Enter and recovery of a Tab-deferred reply. The `blocker`, `conflict` and
`disagree` paths are specified but have had less real use. Agent CLIs change quickly; version-specific
notes in `SKILL.md` say which version they were observed on.

## License

MIT
