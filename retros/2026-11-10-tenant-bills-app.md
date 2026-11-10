# Retro: tenant-bills-app (2026-11-10)

Scope: the last 15 Claude Code sessions in `tenant-bills-app` (grill-with-docs on issues 4 to 11, to-spec, to-tickets on specs #27 to #36, implement #18). Goal: turn repeated directions into a few lines in the right places without growing always-loaded context.

## Findings

| # | Finding | Evidence |
| - | ------- | -------- |
| 1 | The user had to keep saying "on its own branch" | Said in 5 skill calls; a memory file did not stop it; the global CLAUDE.md had no such rule |
| 2 | The RTK block in the global CLAUDE.md was mostly ignored | About 120 always-loaded lines; 34 of 474 shell calls used `rtk`, all of them `rtk git` or `rtk gh` |
| 3 | `/open-pr` duplicated `/close-pr` and had stale content | The user added "Refs #9" and "Closes #18" by hand; the test plan came from another repo; its last steps merged the PR, which `/close-pr` also does |
| 4 | `/close-pr` stopped on repos without CI | "no checks reported" 3 times; the user answered "skip CI since it doesn't yet exist" |
| 5 | Questions with unclear wording in to-tickets | "what is 'it'?", "you didn't name … as (b)" |
| 6 | The horizontal-rule section was in the global CLAUDE.md twice | Same text twice |
| 7 | Auto mode refused `gh pr create --body-file` | The user ran the command by hand |
| 8 | A `cd` into the repo started 249 of 474 shell calls | Commands already run there; costs a few tokens, never caused an error; left as is |

## Actions

| # | Action | Where |
| - | ------ | ----- |
| 1 | A hook that runs before every file edit and refuses it in a git repo on `main`/`master` (lets through `.git/` files, files outside repos, and cases where git can't answer); branch memory deleted | `~/.claude/hooks/block-edits-on-main.py`, `settings.json` |
| 2 | RTK block cut to one line about `git`/`gh` (the RTK hook is not available on Windows) | `~/.claude/CLAUDE.md` |
| 3 | Ask `Closes`/`Refs` for the issue number in the branch name, build the test plan from the diff, stop after creating the PR | `~/.claude/commands/open-pr.md` |
| 4 | Merge without asking when the repo has no `.github/workflows/` | `~/.claude/skills/close-pr/SKILL.md` |
| 5 | "Number each question, letter each option, and name the subject inside every question" | `~/.claude/CLAUDE.md` |
| 6 | Duplicate removed | `~/.claude/CLAUDE.md` |
| 7 | Allowed `gh pr create` and `gh issue comment` (with and without `rtk`) | `~/.claude/settings.json` |

All changes are versioned in the dotfiles repo (JeremieLitzler/dotfiles#11, merged); the default model stays `sonnet`.
