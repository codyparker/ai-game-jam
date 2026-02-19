# Repository Guidelines

## Project Structure & Module Organization
This repository is a Claude Code skill plugin, not a traditional app. Core files:
- `skills/game-jam/SKILL.md`: orchestration flow, phases, and state handling.
- `skills/game-jam/prompts.md`: system and phase prompts used by designer/developer agents.
- `.claude-plugin/plugin.json`: plugin metadata and skill registration.
- `README.md` and `CLAUDE.md`: user-facing usage and architecture notes.

Generated game outputs (for example `2026-02-08-game/`) are runtime artifacts created when the skill runs and are usually not part of source changes.

## Build, Test, and Development Commands
There is no compile/build pipeline in this repo. Validate behavior through Claude Code:
- `/game-jam`: starts a new 6-phase autonomous run.
- `/game-jam resume`: resumes an incomplete jam from `state.json`.
- `git status` and `git diff`: review only intended doc/prompt/skill edits before PR.
- `rg --files`: quick inventory when verifying changed paths.

For local installation testing:
- `git clone https://github.com/codyparker/ai-game-jam ~/.claude/skills/game-jam`

## Coding Style & Naming Conventions
- Keep files Markdown/JSON clean and readable; use concise, directive language.
- Use 2-space indentation for JSON (`.claude-plugin/plugin.json` style).
- Preserve existing phase file naming (`01-concept.md` … `05-final-spec.md`) and kebab-case directory names for generated games.
- Keep prompt placeholders exact: `{{PERSONALITY}}`, `{{THEME_CONSTRAINT}}`, `{{GAME_TYPE_CONSTRAINT}}`.

## Testing Guidelines
No automated test suite exists yet. Minimum manual checks for contribution readiness:
1. Run `/game-jam` and confirm `state.json`, `plans/`, and `logs/` are created.
2. Stop mid-run, then run `/game-jam resume` and verify completed phases are skipped.
3. Confirm generated game output includes playable instructions (`README.md`) and assets under `assets/`.

## Commit & Pull Request Guidelines
Current history uses short, plain-English subjects (for example: `initial`, `Convert to Claude Code plugin format`). Follow the same style with imperative, specific commit titles.

PRs should include:
- What changed and why.
- Files touched (especially `SKILL.md` vs `prompts.md` sync).
- Manual validation performed (`/game-jam` and/or `/game-jam resume`).
