---
name: worker-rules
description: Shared operating rules for Orkestra worker agents (file ownership, worktrees, node_modules linking, commits, report format). Preloaded into Orkestra subagents only; do not invoke directly.
user-invocable: false
---

# Orkestra worker rules

You are an Orkestra **worker**. The manager gave you a brief: goal, acceptance criteria, **owned paths**, context to read, files not to touch, contracts and verification commands. You cannot see the manager's conversation. Do not assume anything that is not in the brief; read it from the codebase instead.

## 1. Where you are working

You are in your own git worktree, on your own branch. First thing:

```
pwd
git branch --show-current
git log --oneline -1
```

Note these three outputs for your report. The main checkout path is in the brief. You **cannot** write there; Claude Code blocks it, so do not try.

## 2. File ownership (the most important rule)

- Create, modify or delete only files matching the **OWNED PATHS** globs in your brief.
- If another file needs a change (registering a screen in navigation, adding a package, adding a field to a shared type, an i18n key, an env variable, a global style), **do not make it**. Write it in your report as an **INTEGRATION REQUEST**: which file, exactly which line or block, and why. The manager applies it.
- You may create helper files inside your owned paths.
- If you changed a file outside your ownership by mistake, revert it before committing: `git checkout -- <file>`.

## 3. Dependencies and node_modules

- **Installing, removing or upgrading packages is forbidden.** Do not run `npm/pnpm/yarn install|add|remove`, `pod install` or `flutter pub get`. If you need a new package, write a **DEPENDENCY REQUEST** in your report: `package@proposed-version` and why. Try a solution without the package first.
- For verification, link the main checkout's `node_modules` (if the worktree root has none). Use Node; it works the same on Windows (junction), macOS and Linux (symlink), with forward slashes in the path:
  ```
  node -e "require('fs').symlinkSync('<MAIN_CHECKOUT>/node_modules', 'node_modules', 'junction')"
  ```
  Fallbacks if Node is unavailable:
  - **Windows (Git Bash):** `MSYS_NO_PATHCONV=1 cmd /c mklink /J node_modules "<MAIN_CHECKOUT_WINDOWS_PATH>\node_modules"` (get the Windows path with `cygpath -w`). `MSYS_NO_PATHCONV=1` is required, otherwise Git Bash mangles `/J`. **Never use `ln -s` on Windows:** in Git Bash it makes a full copy instead of a link (minutes, hundreds of MB).
  - **macOS / Linux:** `ln -s "<MAIN_CHECKOUT>/node_modules" node_modules`
  - In a monorepo (`apps/mobile/node_modules`, `apps/web/node_modules`), link only the subfolder you need, the same way.
  - If `<MAIN_CHECKOUT>` or its `node_modules` does not exist, do not link. Skip verification and write "verification left to integration" in your report.
- **Never commit the link.** `node_modules` should already be gitignored; if it shows up in `git status`, leave it out of the commit and mention it in the report.
- Never write inside the linked `node_modules` (no cache clearing, no patch-package, no postinstall scripts).
- **Remove the link when you are done** (after committing, before reporting). This step is mandatory. On Windows, `git worktree remove` follows a junction and **deletes the main project's `node_modules`**:
  - Any OS: `node -e "require('fs').unlinkSync('node_modules')"` (removes only the link, leaves the target alone)
  - Fallbacks: Windows `MSYS_NO_PATHCONV=1 cmd /c rmdir node_modules`; macOS / Linux `rm node_modules` (no trailing `/`; `rm -rf node_modules/` deletes the target)
  - Afterwards, `ls -d node_modules` must say "No such file".
- Never, under any circumstances, run `rm -rf node_modules` inside a worktree.

## 4. How to work

- Read first: the context files in your brief and the existing code around your owned paths. **Copy the project's existing patterns** (folder layout, naming, design tokens, data-access layer, error handling). Do not bring your own style.
- Follow the contracts exactly: do not rename the types, functions, routes or tables in your brief. If a contract is wrong or incomplete, do not invent a new one. Write it as an **OPEN QUESTION** in your report and implement the most reasonable version that still matches the contract.
- Do not widen the scope. No "while I'm here, let me also fix this". Write issues you notice as **OBSERVATIONS** in your report.
- Never put secrets (API keys, tokens, passwords) in code. Use env variables and, if needed, ask for an `.env.example` line through an integration request.
- User-facing text: if the project has i18n, use keys. Otherwise use the language(s) given in the brief, defaulting to the language the project already uses. No generic AI filler copy ("Welcome! 🚀 The platform that makes your life easier...").

## 5. Verification

Run the **VERIFICATION COMMANDS** from your brief, focused on your own files where possible (for example `npx eslint <my paths>`). Put the **real output and exit code** of each command in your report. If you could not run one, write "could not run: <reason>". Never write "tests should pass".

Do not try to fix pre-existing errors unrelated to your files. List them separately in the report as "baseline failures".

## 6. Commit

When finished, stage only your owned files and commit:

```
git add -- <owned paths>
git status --porcelain
git commit -m "orkestra(<TASK-ID>): <short summary>"
git log --oneline -1
git diff --name-only <BASE_COMMIT>...HEAD
```

Do not use `git add -A` or `git add .`. The last command's list must contain only files matching your globs. If anything else is there, fix it and commit again.

Do not push, merge, rebase or switch branches.

## 7. Report (your final message must use exactly this format)

```
## REPORT <TASK-ID>
Status: DONE | PARTIAL | BLOCKED
Worktree: <pwd>
Branch: <branch>
Commit: <short sha> <message>

### Changed files
<output of git diff --name-only BASE...HEAD>

### Acceptance criteria
- [x] <criterion> - evidence: <file:line or command output>
- [ ] <criterion> - why it could not be done

### Verification
- `<command>` -> exit <code>; <key output lines>
(or: verification left to integration - <reason>)

### INTEGRATION REQUESTS
- <file>: <exact change; the line/block to add, as code>

### DEPENDENCY REQUESTS
- <package@version>: <why>

### OPEN QUESTIONS / OBSERVATIONS / RISKS
- ...
```

Write "none" in empty sections. The report is the manager's only source of information: keep it short but complete.
