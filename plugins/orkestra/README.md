# Orkestra Free

**A manager and a team of parallel AI workers for Claude Code, for building websites.** One command plans the job, splits it into tasks with strict file ownership, runs workers side by side in isolated git worktrees, verifies their work with real command output, merges it, and runs a read-only audit.

![Orkestra building three pages of a demo site: plan, parallel workers in git worktrees, audit, merge](https://raw.githubusercontent.com/getorkestra/orkestra/main/assets/demo.gif)

A real 8-minute run on a demo site, sped up. [Watch the 36-second video with sound](https://github.com/getorkestra/orkestra/blob/main/assets/demo.mp4).

```
                       YOU
                        |  /orkestra:site "services, about and contact pages in English and German"
                        v
            +--------------------------+
            |  MANAGER (your session)  |  plan -> your approval -> waves -> verify -> merge
            +------------+-------------+
                         |   Wave 1 (parallel)                                Wave 2
      +--------------+ +-----------------+ +------------+          +---------------+
      | site-builder | | site-copywriter | | site-seo   |          | site-auditor  |
      | worktree #1  | | worktree #2     | | worktree #3|          | read-only     |
      +------+-------+ +--------+--------+ +-----+------+          +-------+-------+
             +------ branch ----+--- branch -----+                         |
                         v                                                 v
            manager: ownership check -> git merge -> shared files -> build/lint -> commit
```

Version 1.0.0 - Claude Code plugin - MIT licensed - works with any Claude Code plan.

## Why

Running several Claude Code agents at once is easy. Getting their work to merge cleanly is not. Orkestra adds the missing discipline:

- **File ownership:** every task gets a glob list; globs in the same wave may not overlap. Shared files (layout, navigation, config, package manifest) belong to the manager only.
- **Git worktrees:** every worker runs in its own worktree and branch, so agents never write over each other.
- **Approved plan first:** a readable task table in `.orkestra/plan.md`. Nothing starts until you approve it.
- **Evidence, not claims:** the manager re-runs verification and checks every changed file against ownership before merging.
- **Read-only auditor:** accessibility (WCAG 2.2 AA), responsive layout, SEO, performance, broken links, cookie consent and privacy law.
- **Safe on Windows:** `node_modules` is linked into worktrees and unlinked before cleanup, so `git worktree remove` can't wipe your main install.

## Install

Requirements: Claude Code (recent version) and git 2.20+.

```bash
claude plugin marketplace add getorkestra/orkestra
claude plugin install orkestra@orkestra-free
```

Then, inside Claude Code, run setup once and restart Claude Code:

```
/orkestra:setup
```

Setup sets `"worktree": {"baseRef": "head"}` in `~/.claude/settings.json` (with a backup) so workers branch from your current work instead of `main`, and enables git long paths on Windows.

## Use

In a git project with a clean working tree:

```
/orkestra:site landing page with hero, features, pricing and FAQ
/orkestra:site services, about and contact pages in English and German
```

1. **Preflight:** git state, `.gitignore`, stack detection (static HTML, Astro, Next.js, WordPress theme), baseline build/lint.
2. **Plan:** a task table with owned paths and acceptance criteria. **You approve it.**
3. **Waves:** up to 3 workers in parallel by default.
4. **Integration:** ownership check, merge, shared-file changes, verification, commit.
5. **Audit:** at most 2 fix rounds; unresolved findings are reported, never hidden.
6. **Close:** worktrees cleaned up, short report, lessons appended to `.orkestra/lessons.md`.

Every worker has its own context, so three parallel workers use roughly three times the tokens.

## Orkestra Pro

Need more than websites? **Orkestra Pro** adds two more teams with the same protocol:

- `/orkestra:webapp`: data layer and migrations with RLS, UI, payments/email/webhooks, **blind** Playwright and unit tests (IDOR tests mandatory) and an Opus security auditor.
- `/orkestra:mobile`: React Native / Expo screens, backend, native config (permissions, push, deep links), blind tests with Jest and Maestro, App Store and Google Play listing prep, and an Opus auditor that knows the store guidelines.

Plus updates and support. One-time payment, private GitHub repo access:

- **[Solo, $10](https://buy.polar.sh/polar_cl_ViKQBhn3miSAWkpNiJwZ8WvFCy7oDkarqkr4h3QTHba)**: 1 person, unlimited projects, client work included.
- **[Team, $30](https://buy.polar.sh/polar_cl_r5L3eKziwNqKAI3BzGga7oh5fMlkYrnbepwaW2voS4Y)**: up to 10 people in one organization.

After checkout, open the customer portal link in your order email and click **Connect GitHub**: you get an invitation to the private repo with install steps.

## What it changes on your machine

Orkestra is instructions for Claude Code (skills and agent definitions). It ships no scripts, hooks, MCP servers or binaries, makes no network calls of its own and sends no data anywhere.

- `/orkestra:setup` sets `"worktree": {"baseRef": "head"}` in `~/.claude/settings.json` after saving a timestamped backup next to it, and on Windows runs `git config --global core.longpaths true`. Undo: restore the backup or delete the `worktree` key.
- In your project, the manager creates `.orkestra/` (plan, quality gate script, lessons) and adds it to `.gitignore`, makes commits on your current branch, and creates temporary git worktrees and branches under `.claude/worktrees/` that it removes after merging.
- If your project has `node_modules`, workers link it into their worktree (a junction on Windows, a symlink elsewhere) and remove the link before cleanup.
- Workers and the auditor run the build, lint and test commands your project already has.

## Support

Questions and bugs: [open an issue](https://github.com/getorkestra/orkestra/issues).

## License

MIT. See [LICENSE](LICENSE).
