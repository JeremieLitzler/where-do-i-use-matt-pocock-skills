# Retro: coge-audio-meditation-guides (2026-11-12)

Scope: all 17 earlier Claude Code sessions in `coge-audio-meditation-guides` (2026-10-31 to 2026-11-07): the grilling sessions on the README, Passages, the countdown and semantic versioning; implement #3, #4, #9, #10, #18, #21, #33; the CI and release workflows; cutting v0.1.0 to v1.0.0; Guide 1 v2.2 with Mr. Cole's original recording (#38). Ranked by severity.

## Findings

- **1. No recipe for adding a Guide version**
  - The Guide 2 v2.1 and Guide 1 v2.2 sessions each dumped 22 to 26 KB of guides, scripts and tests to find which files a version touches.
  - In the ADR 0002 session, the user had to ask whether a future agent would know the CSV needs updating.
  - The set is fixed: guide `.md`, `spoken-form-changes-guide-N-vX.Y.csv`, `scripts/generate-guideN-vX.sh`, `tests/test_guide_N_vX_Y.py`, `guides/spoken-form-changes.md`, `guides/wording-changes.md`.
- **2. The reviewer agent has no repo standards**
  - `code-review` ran in about 9 sessions with no `CODING_STANDARDS.md`.
  - The user caught these by hand: the `alpha` fixture (to be Tim and Darian, plus a made-up name for the unknown-Cast test), `number` that should be `guideNumber` in `produce_section.py`, and the exact-text tests that broke after a wording edit (became ADR 0002, which the reviewer does not know about).
- **3. No lint and no pre-commit guardrail**
  - `tests.yml` runs only `pytest`, on `pull_request` only. No ruff, no `.pre-commit-config.yaml`.
  - A PowerShell `Get-Content` without UTF-8 turned text into mojibake (`_MeeknessÂ`) in a test; pytest caught it only because that line was compared with the source.
- **4. CI polling is expensive**
  - 54 `gh pr checks` / `gh pr view --json state` calls across 10 sessions, with `--watch` used in only 3 sessions.
  - The issue #7 docs PR took 12 calls, and the user still typed "check CI".
- **5. `rtk` gives misleading errors**
  - `rtk gh issue view N --comments` failed twice ("specify only one of --comments or --json").
  - `rtk git commit` exited 128 with "fatal: Needed a single revision" in 3 sessions although the commit had worked, which led to extra checks. Cause not confirmed.
- **6. The issue link in the PR body needed correcting again and again**
  - The user typed "Use Closes #3", "use Closes #4", "add Closes #33".
  - `open-pr.md:47` asks "Closes or Refs?", but the link was still missing on #33.
- **7. Minor tool and permission gaps**
  - `create-env-proton-pass.sh`: `jq '.items[].content.title'` crashes on items with no title, so the user found the item by hand.
  - `bc` is missing in Git Bash.
  - The auto-mode classifier denied `gh issue view 18 --json body` once.

## What already works

- The "the owner runs `produce_section.py`" rule is in memory and was respected after the user stated it.
- `AGENTS.md` is 13 lines of navigation pointers, with no instructions that change nothing.

## Proposed actions

- **1. Guide version checklist** in `docs/agents/guide-versions.md`: the files a Guide version touches, with the reason for each. Plus one pointer line in `AGENTS.md`: "Adding or editing a Guide version: see `docs/agents/guide-versions.md`".
- **2. `CODING_STANDARDS.md`** with judgement-call rules only: fixtures use real Cast names from `GLOSSARY.md` (made-up names only for unknown cases), names state their domain meaning, script tests check rules not wording (pointer to ADR 0002).
- **3. Guardrails**: a `ruff check` step in `.github/workflows/tests.yml` (ruff in the dev group of `pyproject.toml`), and a pytest in `tests/` that fails on mojibake (`Ã`, `Â`) or invalid UTF-8 in `guides/*.md` and `guides/*.csv`.
- **4. CI wait** in `~/.claude/skills/close-pr/SKILL.md`: `gh pr checks <n> --watch --fail-fast` run in the background.
- **5. rtk**: find the cause first. Stopgap in the RTK section of `~/.claude/CLAUDE.md`: read comments with `rtk gh issue view N --json body,comments`, and treat "Needed a single revision" as noise when the commit hash is printed.
- **6. Issue link** in `~/.claude/commands/open-pr.md`: default to `Closes #N` when the branch name has N; ask only when it has no number.
- **7. Minor fixes**: `select(.content.title != null)` in the jq filter of `scripts/create-env-proton-pass.sh`; a tooling note in `AGENTS.md` to use `python -c` for arithmetic; `/fewer-permission-prompts` for read-only `gh` calls in `.claude/settings.json`.

Status: not started.
