# cursor-plugin-cc

A Claude Code plugin that lets Claude delegate code-writing tasks to the **Cursor CLI** (`cursor-agent`) in headless mode. Useful when you want a second AI to implement, refactor, or generate code in parallel — or to run multiple coding agents on independent subtasks.

Ships two things:

- **`cursor-delegate` subagent** — invoked from Claude (`Agent(subagent_type: "cursor-delegate", prompt: ...)`).
- **`/cursor` slash command** — fire a Cursor task directly from your Claude Code prompt.

Default model: `composer-2`. Override per-invocation.

---

## Requirements

- [Claude Code](https://claude.com/claude-code) (CLI, desktop, or IDE extension)
- [Cursor CLI](https://docs.cursor.com/cli) — `cursor-agent` available on `PATH`
- An authenticated Cursor account (`cursor-agent login` once per machine)

Verify:

```bash
cursor-agent --version
cursor-agent status
```

---

## Install

```text
/plugin marketplace add ojhernandezc/cursor-plugin-cc
/plugin install cursor-delegate@cursor-plugin-cc
/reload-plugins
```

Update later:

```text
/plugin update cursor-delegate@cursor-plugin-cc
```

---

## Usage

### Slash command

```text
/cursor add a darkMode toggle to Settings.tsx
/cursor --model gpt-5.2 refactor the auth middleware
/cursor --mode plan audit the form validation
/cursor --workspace /Users/me/myapp --model gpt-5.3-codex-high fix the failing tests
```

Flags (any order, all optional):

| Flag | Default | Notes |
|---|---|---|
| `--model <id>` | `composer-2` | See `cursor-agent models` for the full list |
| `--workspace <abs-path>` | current cwd | Where Cursor will edit files |
| `--mode plan` or `--mode ask` | execute | `plan` = read-only planning; `ask` = Q&A only |

### Subagent (for orchestration from Claude)

You don't call this directly — you ask Claude to plan and delegate. Examples:

> "Plan + delegar a cursor: implement source==referred reveal in the signup form with i18n. composer-2 is fine."

> "Plan, paraleliza con cursor en 3 tareas (form, i18n, tests)."

> "Implementa con cursor y haz que codex revise el diff."

Under the hood, Claude invokes:

```
Agent(subagent_type: "cursor-delegate", prompt: """
MODEL: gpt-5.3-codex-high            # optional, defaults to composer-2
WORKSPACE: /Users/me/myapp           # optional, defaults to cwd
MODE: plan                            # optional
---
<the actual task description, multi-line ok>
""")
```

---

## How it works

The subagent and slash command are thin wrappers that build and run:

```bash
cursor-agent -p --output-format text --force \
  --model <MODEL> [--workspace <WS>] [--mode <MODE>] '<task>'
```

The `-p --output-format text --force` flags are required for headless usage. The agent surfaces Cursor's output verbatim and adds a one-line summary of what changed.

---

## Common models

`composer-2` (default), `composer-2-fast`, `auto`, `gpt-5.2`, `gpt-5.3-codex`, `gpt-5.3-codex-high`, `sonnet-4`, `sonnet-4-thinking`.

Run `cursor-agent models` on your machine for the full account-specific list.

---

## Manual install (without the marketplace)

If you don't want to use `/plugin marketplace add`, drop the files at the user level:

```bash
mkdir -p ~/.claude/agents ~/.claude/commands
cp agents/cursor-delegate.md ~/.claude/agents/
cp commands/cursor.md ~/.claude/commands/
```

Then `/reload-plugins` (or start a new Claude Code session).

---

## License

MIT — see [LICENSE](LICENSE).
