---
description: Delegate a coding task to the Cursor CLI (cursor-agent) in headless mode
argument-hint: [--model <model>] [--workspace <path>] [--mode plan|ask] <task>
allowed-tools: Bash
---

Parse `$ARGUMENTS` for these optional flags (in any order, anywhere in the args):

- `--model <model-id>` — defaults to `composer-2`
- `--workspace <abs-path>` — defaults to the current working directory
- `--mode plan` or `--mode ask` — omit for normal execute mode

Everything left after removing the recognized flags and their values is the task description.

If `$ARGUMENTS` is empty (or only whitespace), print this and stop:

> Usage: `/cursor [--model <model>] [--workspace <path>] [--mode plan|ask] <task>`
> Default model: `composer-2`. Run `cursor-agent models` to see all available models.

Otherwise, run via Bash:

```bash
cursor-agent -p --output-format text --force --model <MODEL> [--workspace <WS>] [--mode <MODE>] '<task>'
```

Rules:
- Always include `-p`, `--output-format text`, and `--force` (headless requires them).
- Single-quote the task; escape internal single quotes as `'\''`.
- Return Cursor's stdout verbatim, then add one short line summarizing what was changed.
- If the command exits non-zero, surface the error verbatim — do not retry silently.
- Do not Read/Edit/Write files yourself; Cursor edits in the workspace directly.

Examples:

- `/cursor add a darkMode toggle to Settings.tsx`
  → `cursor-agent -p --output-format text --force --model composer-2 'add a darkMode toggle to Settings.tsx'`

- `/cursor --model gpt-5.3-codex-high --workspace /Users/ohernandez/proj fix the failing referred-source validation`
  → `cursor-agent -p --output-format text --force --model gpt-5.3-codex-high --workspace /Users/ohernandez/proj 'fix the failing referred-source validation'`

- `/cursor --mode plan refactor the auth middleware`
  → `cursor-agent -p --output-format text --force --model composer-2 --mode plan 'refactor the auth middleware'`
