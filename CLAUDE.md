# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a Claude Code **skill** (invoked via `/game-jam`) that orchestrates two AI agents — a game designer and a game developer — to autonomously collaborate and produce a complete, playable game. The user provides a personality seed for the designer and optional constraints (theme, game type, and complexity), then the system runs 6 phases without human interaction.

## Usage Modes

- **Start new jam**: `/game-jam` — Prompts for personality, theme, game type, and design complexity, then runs all 6 phases
- **Resume jam**: `/game-jam resume` — Lists all incomplete jams in the current directory and lets user choose which to resume

## Repository Structure

- `SKILL.md` — Skill definition with metadata, process flow, and orchestration logic. This is the entry point that Claude Code reads when the skill is invoked.
- `prompts.md` — Complete agent prompts for all 6 phases, referenced by SKILL.md via `@prompts.md`.

## Architecture

The skill works as a **phase-based orchestrator**:

1. **Setup**: Asks for personality seed, optional theme constraint, optional game type constraint, and design complexity. Setup questions are always asked sequentially (one-by-one), including optional fields. Checks for resumable sessions, creates a dated game directory (`YYYY-MM-DD-game/`) with `state.json`, `plans/`, and `logs/`.
2. **Design Rounds (Phases 1-5)**: Dispatches Task subagents (inheriting the session's model). Designer and developer alternate, producing plan documents in `plans/`. Both append reasoning to `logs/collaboration.md`.
3. **Build Phase (Phase 6)**: Dispatches a Task subagent with full tool access to build the actual game.
4. **Completion**: Updates `state.json` to `"complete"`, generates `stats.md` with collaboration summary and timing data, renames the game directory from `YYYY-MM-DD-game/` to the game name, and presents a summary.

**State management**: `state.json` tracks `currentPhase`, `completedPhases`, `phaseTimings`, `status`, `designComplexity`, and optional constraints (`theme` and `gameType`) to enable resumability. The orchestrator records start/end timestamps for each phase and checks/updates this file before/after each phase. Users can pause at any time (the skill saves progress after each phase) and resume later with `/game-jam resume`.

**Design complexity**: `light`, `standard`, or `deep` controls how much the designer explores and how much detail they communicate to the developer.

**Agent prompts**: Each phase combines a system prompt (designer or developer persona from `prompts.md`) with a phase-specific prompt. Placeholders get replaced:
- `{{PERSONALITY}}` → user's personality seed
- `{{THEME_CONSTRAINT}}` → optional theme requirement (if provided)
- `{{GAME_TYPE_CONSTRAINT}}` → optional game type requirement (if provided)
- `{{DESIGNER_COMPLEXITY_GUIDANCE}}` → per-run designer depth/communication guidance derived from `designComplexity`

**Tool restrictions**: Design phases (1-5) instruct agents to only use Read/Write. Build phase (6) grants all tools. These are enforced via prompt instructions, not programmatic restrictions.

## Key Constraints

- Games must be macOS-compatible (browser games, Python/Pygame, Node, etc.)
- Game scope is "game-jam-small": one core mechanic, simple visuals, ~5 minutes of fun
- Optional theme constraint: if provided, the game must incorporate this theme creatively
- Optional game type constraint: if provided, the game must fit this genre/style
- Optional designer complexity setting controls design depth and handoff verbosity
- All game assets go in an `assets/` subdirectory inside the game source directory
- Build complexity must be SIMPLE or MEDIUM (never HARD)
- Total build steps in the implementation plan must stay under 15

## Pause and Resume

The skill is fully pausable and resumable across Claude sessions:
- **Pause**: Simply stop the session at any time. Progress is saved after each phase completes.
- **Resume**: Run `/game-jam resume` to see all incomplete jams and choose which to resume.
- The skill automatically skips completed phases and picks up where it left off.
- Resumed jams pick up where they left off using the current session's model.
