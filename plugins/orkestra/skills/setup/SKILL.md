---
name: setup
description: One-time Orkestra setup. Sets worktree.baseRef to "head" in the user's Claude Code settings (with a backup), enables git long paths on Windows, and checks git and Claude Code versions.
disable-model-invocation: true
---

# Orkestra setup

The user ran `/orkestra:setup`. Configure their machine so Orkestra workers branch from the current HEAD. Do each step in order, show what you changed, and stop on any error.

## 1. Check the tools

Run and record:

```
git --version
claude --version
```

- git must be 2.20 or newer (worktree support). If it is older, tell the user to upgrade git and stop.
- If `claude --version` fails, continue; it only means the CLI is not on PATH in this shell.

## 2. Set `worktree.baseRef` to `"head"`

Without this, Claude Code creates worker worktrees from the default branch (`main`), so workers cannot see the commits the manager has just made.

1. The settings file is `~/.claude/settings.json` (on Windows: `%USERPROFILE%\.claude\settings.json`).
2. If the file exists, read it. If `worktree.baseRef` is already `"head"`, say so and skip to step 3.
3. If the file exists, copy it to `settings.json.backup-<YYYYMMDD-HHMMSS>` next to it first, and tell the user the backup path.
4. Set `"worktree": {"baseRef": "head"}`, keeping every other key exactly as it was. If the file does not exist, create it with only that key. Write valid JSON (2-space indent).
5. Read the file back and confirm the JSON parses and the value is set.

## 3. Windows only: git long paths

Deep `node_modules` paths inside `.claude/worktrees/...` can exceed the 260-character limit on Windows. Detect Windows (`uname -s` contains `MINGW`/`MSYS`/`CYGWIN`, or `$OS` is `Windows_NT`), then run:

```
git config --global core.longpaths true
```

Skip this step on macOS and Linux.

## 4. Finish

Tell the user, in a short list:

- What changed (settings key, backup path, git config) and what was already correct.
- **Restart Claude Code** so the setting takes effect.
- Then, inside a git project with a clean working tree, run one of:
  - `/orkestra:site <what to build>`
  - `/orkestra:webapp <what to build>`
  - `/orkestra:mobile <what to build>`

To undo: restore the backup file, or delete the `worktree` key from `~/.claude/settings.json`.
