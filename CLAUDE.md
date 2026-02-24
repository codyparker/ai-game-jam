# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a Claude Code **skill** (invoked via `/game-jam`) that orchestrates two AI agents — a game designer and a game developer — to autonomously collaborate and produce a complete, playable game. The user provides a personality seed for the designer and optional constraints (theme, game type), with optional advanced settings (design complexity, game scope, design rounds), then the system runs the design and build phases without human interaction.

## Usage Modes

- **Start new jam**: `/game-jam` — Prompts for personality, theme, game type, and optional advanced settings (design complexity, game scope, design rounds)
- **Resume jam**: `/game-jam resume` — Lists all incomplete jams in the current directory and lets user choose which to resume

## Repository Structure

- `skills/game-jam/SKILL.md` — Skill definition with metadata, process flow, and orchestration logic. This is the entry point that Claude Code reads when the skill is invoked.
- `skills/game-jam/prompts.md` — Complete agent prompts for all phases, referenced by SKILL.md via `@prompts.md`.

## Architecture

The skill works as a **phase-based orchestrator**:

1. **Setup**: Asks for personality seed, optional theme, optional game type, then offers advanced settings (design complexity, game scope, design rounds). Creative inputs are always asked one-by-one. Advanced settings are gated behind a yes/no prompt. Checks for resumable sessions, creates a dated game directory (`YYYY-MM-DD-game/`) with `state.json`, `plans/`, and `logs/`.
2. **Design Rounds (3, 5, or 7 phases based on designRounds setting)**: Dispatches Task subagents (inheriting the session's model). Designer and developer alternate, producing plan documents in `plans/`. Both append reasoning to `logs/collaboration.md`.
3. **Build Phase (final phase)**: Dispatches a Task subagent with full tool access to build the actual game.
4. **Completion**: Updates `state.json` to `"complete"`, generates `stats.md` with collaboration summary and timing data, renames the game directory from `YYYY-MM-DD-game/` to the game name, and presents a summary.

**State management**: `state.json` tracks `currentPhase`, `completedPhases`, `phaseTimings`, `status`, `designComplexity`, `gameScope`, `designRounds`, and optional constraints (`theme` and `gameType`) to enable resumability. The orchestrator records start/end timestamps for each phase and checks/updates this file before/after each phase. Users can pause at any time (the skill saves progress after each phase) and resume later with `/game-jam resume`.

**Design complexity**: `light`, `standard`, or `deep` controls how deeply the designer agent explores ideas. Does not affect the developer.

**Game scope**: `tiny`, `small`, or `medium` controls game ambition, build step limits, and allowed build complexity. Affects both agents.

**Design rounds**: `3`, `5`, or `7` controls how many design phases run before the build.

**Agent prompts**: Each phase combines a system prompt (designer or developer persona from `prompts.md`) with a phase-specific prompt. Placeholders get replaced:
- `{{PERSONALITY}}` → user's personality seed
- `{{THEME_CONSTRAINT}}` → optional theme requirement (if provided)
- `{{GAME_TYPE_CONSTRAINT}}` → optional game type requirement (if provided)
- `{{DESIGNER_COMPLEXITY_GUIDANCE}}` → per-run designer depth/communication guidance derived from `designComplexity`
- `{{GAME_SCOPE_GUIDANCE}}` → per-run scope constraints for both designer and developer, derived from `gameScope`
- `{{IMPL_PLAN_FILE}}` → build phase only, implementation plan file path (varies by `designRounds`)
- `{{FINAL_SPEC_FILE}}` → build phase only, final spec file path (varies by `designRounds`)

**Tool restrictions**: Design phases instruct agents to only use Read/Write. The build phase grants all tools. These are enforced via prompt instructions, not programmatic restrictions.

## Key Constraints

- Games must be macOS-compatible (browser games, Python/Pygame, Node, etc.)
- Game scope is configurable: `tiny` (micro-game), `small` (classic jam, default), or `medium` (ambitious)
- Optional theme constraint: if provided, the game must incorporate this theme creatively
- Optional game type constraint: if provided, the game must fit this genre/style
- Design complexity controls designer agent depth and handoff verbosity
- Game scope controls build step limits and allowed build complexity
- Design rounds (3/5/7) controls number of design phases before building
- All game assets go in an `assets/` subdirectory inside the game source directory

## Pause and Resume

The skill is fully pausable and resumable across Claude sessions:
- **Pause**: Simply stop the session at any time. Progress is saved after each phase completes.
- **Resume**: Run `/game-jam resume` to see all incomplete jams and choose which to resume.
- The skill automatically skips completed phases and picks up where it left off.
- Resumed jams pick up where they left off using the current session's model.
