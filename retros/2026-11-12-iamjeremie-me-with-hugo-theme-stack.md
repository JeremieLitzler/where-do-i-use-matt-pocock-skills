# Retro: iamjeremie.me-with-hugo-theme-stack (2026-11-12)

Scope: the last 10 Claude Code sessions in `iamjeremie.me-with-hugo-theme-stack` (2026-10-29 to 2026-11-12): article enrichment, reviews and proofreading, tag and canonical URL fixes, the Dart Sass migration and theme upgrade (implement #13 to #15), registering the repo in this catalog. Ranked by severity.

## Findings

| # | Finding | Evidence |
| - | ------- | -------- |
| 1 | Only the build checks a PR; content bugs ship | The Netlify build runs `hugo` + `test:xml` only, with no pre-commit hook. Two bugs were found by chance: `Csharp`/`CSharp` tag casing (6 vs 2) flipped the tag page title, and `canonicalUrl` came out relative through `relLangURL` in `head.html` |
| 2 | A dead GitHub workflow makes `/close-pr` stall | `.github/workflows/deploy.yml` is not active (`gh workflow list` shows only "Dependency Graph") and stale (Hugo `latest`, deploys `gh-pages`, no Dart Sass); Netlify is the real CI. `close-pr` asks the user when there are no checks and a workflows folder exists; on PR #24 (and fr #10) the Netlify checks had not registered yet, so the agent wrote a 40-round polling loop. Pending checks exit `gh pr checks` with code 8, which was read as an error |
| 3 | `/close-pr` cleanup always fails once | `gh pr merge --rebase --delete-branch` already deletes the local branch; step `git branch -d <headRefName>` then fails and turns the chain red (PR #22) |
| 4 | Edits sent on `master`, then the same batch sent again | Geekom session: 7 parallel Edits blocked by `block-edits-on-main.py`, the same batch resent before branching, about 14 failed calls; several "String not found" from paraphrased anchors. The user commits draft and proof on `master` before asking for changes, so this recurs |
| 5 | `/open-pr` body breaks in PowerShell | `gh pr create --body "<multi-line>"` from PowerShell gave "unknown arguments"; the template uses a bash heredoc (`open-pr.md:62`) |
| 6 | Article reviews have no written standard | "review article" / "proofread" in 3 sessions; the user answered with numbered house rules each time: keep `<-` as is, soften blame, "probable" over "sure", trust the publish date, suggest a Pexels image and alt text |
| 7 | Visual checks by screenshot are expensive | Theme upgrade session read about 2 MB of PNGs to compare the EN and FR sites before and after |
| 8 | `rtk` prefix skipped on some `gh` calls | `gh pr create`, `gh issue view`, `gh pr checks` without `rtk`; the rule exists in the global CLAUDE.md, so it is only followed some of the time |

## Proposed actions

| # | Action | Where |
| - | ------ | ----- |
| 1 | Content linter run from `build:ci`: case-only tag variants, absolute `https://` `canonicalUrl`, required front matter (`title`, `description`, `date`, `image`), no `Credit: Photo by TODO` left in a published article | `scripts/lint-content.mjs`, `package.json` |
| 2 | Delete or repair `deploy.yml`; in `close-pr`, treat "no checks yet" right after a push as "wait" with `gh pr checks <n> --watch` (exit 8 = pending) | repo `.github/workflows/`, `~/.claude/skills/close-pr/SKILL.md` |
| 3 | Delete the local branch only if `git branch --list <name>` still shows it | `~/.claude/skills/close-pr/SKILL.md` |
| 4 | Hook message: the whole batch failed; run `git checkout -b docs/<slug>` before sending any edit | `~/.claude/hooks/block-edits-on-main.py` |
| 5 | Write the PR body to a file under `.git/` and pass `--body-file` (already allowed in `settings.json`) | `~/.claude/commands/open-pr.md` |
| 6 | Review rules doc for articles (judgement calls only; mechanical ones go to action 1), plus one CLAUDE.md pointer for "review" / "proofread" | `docs/agents/article-review.md`, repo `CLAUDE.md` |
| 7 | Build `public/` before and after and diff the HTML as text; keep screenshots for checking how things look | `scripts/diff-build.sh` |
| 8 | Optional: a hook that enforces the `rtk` prefix | `~/.claude/settings.json` |

Status: not started.
