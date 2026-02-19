---
name: game-jam
description: Use when the user wants to create a game using two collaborating AI agents - a designer and a developer - that autonomously produce a complete, playable game from a personality seed with optional theme/type constraints and configurable complexity
---

# AI Game Jam

## Overview

Orchestrate two AI agents — a game designer and a game developer — to autonomously collaborate and produce a complete, playable game. The user provides a personality seed for the designer plus optional constraints, then the system runs 6 phases without human interaction.

## Arguments

- **No arguments** (`/game-jam`): Start a new game jam (prompts for personality and constraints)
- **`resume`** (`/game-jam resume`): Resume an incomplete game jam (lists all in-progress jams in current directory)

## The Process

```dot
digraph game_jam {
    rankdir=TB;

    "Ask for personality seed" [shape=box];
    "Check for incomplete session" [shape=diamond];
    "Offer resume or new" [shape=box];
    "Create game directory" [shape=box];

    subgraph cluster_design {
        label="Design Rounds (use selected designer model)";
        "Phase 1: Designer — Initial Concept" [shape=box];
        "Phase 2: Developer — Technical Response" [shape=box];
        "Phase 3: Designer — Revised Design" [shape=box];
        "Phase 4: Developer — Implementation Plan" [shape=box];
        "Phase 5: Designer — Final Sign-off" [shape=box];
    }

    subgraph cluster_build {
        label="Build Phase (use selected developer model)";
        "Phase 6: Developer — Build Game" [shape=box];
    }

    "Show completion summary" [shape=box];

    "Ask for personality seed" -> "Check for incomplete session";
    "Check for incomplete session" -> "Offer resume or new" [label="found"];
    "Check for incomplete session" -> "Create game directory" [label="none"];
    "Offer resume or new" -> "Create game directory" [label="new"];
    "Offer resume or new" -> "Phase 1: Designer — Initial Concept" [label="resume\n(skip completed)"];
    "Create game directory" -> "Phase 1: Designer — Initial Concept";
    "Phase 1: Designer — Initial Concept" -> "Phase 2: Developer — Technical Response";
    "Phase 2: Developer — Technical Response" -> "Phase 3: Designer — Revised Design";
    "Phase 3: Designer — Revised Design" -> "Phase 4: Developer — Implementation Plan";
    "Phase 4: Developer — Implementation Plan" -> "Phase 5: Designer — Final Sign-off";
    "Phase 5: Designer — Final Sign-off" -> "Phase 6: Developer — Build Game";
    "Phase 6: Developer — Build Game" -> "Show completion summary";
}
```

## Step-by-Step Execution

### 1. Setup

**Resume Mode** (`/game-jam resume`):

1. Search the working directory for all `*-game*/state.json` files with `"status": "in_progress"`
2. If none found, tell the user "No incomplete game jams found. Run `/game-jam` to start a new one."
3. If one found, announce which jam will resume (show personality, theme, gameType, complexity, runtime, models, and current phase), then load its state
4. If multiple found, present them to the user using AskUserQuestion:
   - Show each jam's directory name, creation date, current phase, personality/theme/gameType/complexity/runtime, and models being used
   - Let user select which to resume
5. Load the selected `state.json` and skip to "Run Phases" (step 2)
6. Backfill missing legacy fields before resuming:
   - If `runtime` is missing, set it to `claude`
   - If `designComplexity` is missing, set it to `standard`

**New Game Mode** (`/game-jam` with no arguments):

Ask the user for game jam parameters using AskUserQuestion.
Prompting rules for new games:
- Ask all six setup questions every time, in order, one question at a time.
- Wait for the user's answer before asking the next question.
- Do not skip optional questions; explicitly ask them and allow blank input to mean "none".
- Do not infer or auto-fill answers from prior context except documented defaults when the user leaves a question blank.
- Do not start phase execution until all setup questions have been asked and resolved.

Before model selection, determine runtime platform:
- Prefer automatic detection from host identity and available model IDs.
- Set `runtime` to `claude` when the host is Claude Code / Anthropic.
- Set `runtime` to `codex` when the host is Codex / OpenAI.
- If uncertain, ask the user to choose explicitly (`claude` or `codex`) before continuing.

**First prompt** (game jam configuration):

1. **Designer personality seed** (required):
   - Can be a short phrase ("chaotic goblin energy") or a full character description
   - This shapes the designer agent's creative instincts

2. **Game jam theme** (optional):
   - A creative constraint or subject matter for the game
   - Examples: "bananas", "killer clown", "space", "happiness", "time travel"
   - If provided, the game must incorporate this theme
   - Leave blank to skip

3. **Game type** (optional):
   - A genre or style constraint for the game
   - Examples: "roguelike", "2-bit color style", "text-based", "puzzle platformer", "bullet hell"
   - If provided, the game must fit this type
   - Leave blank to skip

**Second prompt** (model selection, runtime-aware):

4. **Designer agent model** (optional):
   - Which model to use for designer phases (1, 3, 5)
   - If `runtime = claude`: options are "Opus 4.6 (Recommended)" or "Sonnet 4.5" (default: Opus 4.6)
   - If `runtime = codex`: options are "GPT-5.3 Codex (Recommended)" or "GPT-5 mini" (default: GPT-5.3 Codex)

5. **Developer agent model** (optional):
   - Which model to use for developer phases (2, 4, 6)
   - If `runtime = claude`: options are "Opus 4.6 (Recommended)" or "Sonnet 4.5" (default: Opus 4.6)
   - If `runtime = codex`: options are "GPT-5.3 Codex (Recommended)" or "GPT-5 mini" (default: GPT-5.3 Codex)

**Third prompt** (designer complexity):

6. **Design complexity** (optional):
   - Controls how much the designer explores ideas and how detailed their handoff is to the developer
   - Options:
     - `light` — fast convergence, concise communication
     - `standard (recommended)` — balanced exploration and detail
     - `deep` — broader ideation and detailed handoff
   - Default: `standard`

Create the game directory structure:

```
YYYY-MM-DD-game/        (increment suffix if exists: -game-2, -game-3)
├── state.json
├── plans/
└── logs/
```

Initialize `state.json`:
```json
{
  "created": "2026-02-06T21:45:00Z",
  "personality": "<seed>",
  "theme": "<theme or null>",
  "gameType": "<game type or null>",
  "runtime": "claude",
  "designerModel": "opus",
  "developerModel": "opus",
  "designComplexity": "standard",
  "currentPhase": "designer-round1",
  "completedPhases": [],
  "gameDir": null,
  "status": "in_progress"
}
```

Model values:
- If `runtime = claude`: `"sonnet"` for Sonnet 4.5, `"opus"` for Opus 4.6
- If `runtime = codex`: `"gpt-5-mini"` for GPT-5 mini, `"gpt-5.3-codex"` for GPT-5.3 Codex

Default model values:
- If `runtime = claude`: `designerModel = "opus"`, `developerModel = "opus"`
- If `runtime = codex`: `designerModel = "gpt-5.3-codex"`, `developerModel = "gpt-5.3-codex"`

### 2. Run Phases

Each phase dispatches a Task subagent. Use the model specified in state.json:
- Designer phases (1, 3, 5): use `state.designerModel`
- Developer phases (2, 4, 6): use `state.developerModel`

**Before each phase:**
- Check if phase is already in `completedPhases` — skip if so
- Update `currentPhase` in state.json
- Tell the user which phase is running

**After each phase:**
- Add phase to `completedPhases` in state.json
- Tell the user the phase is complete

**Phase-to-file mapping and agent prompts:** See @prompts.md for complete agent prompts for each phase.

| Phase | Agent | Output File | Tools Needed |
|-------|-------|-------------|--------------|
| 1. designer-round1 | Designer | `plans/01-concept.md` | Write, Read |
| 2. developer-round1 | Developer | `plans/02-tech-response.md` | Write, Read |
| 3. designer-round2 | Designer | `plans/03-revised-design.md` | Write, Read |
| 4. developer-round2 | Developer | `plans/04-impl-plan.md` | Write, Read |
| 5. designer-round3 | Designer | `plans/05-final-spec.md` | Write, Read |
| 6. developer-build | Developer | Game source directory | All tools |

**Dispatching each phase subagent:**

Before dispatching, replace placeholders in prompts.md:
- `{{PERSONALITY}}` → user's personality seed from state.json
- `{{THEME_CONSTRAINT}}` → if theme exists: `"GAME JAM THEME: Your game must incorporate this theme: \"[theme]\"\n\nThis is a required constraint. Find creative ways to make this theme central to your game."` | if null: remove this line entirely
- `{{GAME_TYPE_CONSTRAINT}}` → if gameType exists: `"GAME JAM TYPE: Your game must be this type: \"[gameType]\"\n\nThis is a required constraint. Design your game to fit this genre/style."` | if null: remove this line entirely
- `{{DESIGNER_COMPLEXITY_GUIDANCE}}` → for designer phases only, replace with:
  - if `designComplexity = light`:
    `"DESIGN COMPLEXITY: LIGHT.\nThink briefly and converge fast. Share only implementation-critical details with the developer.\nKeep designer logs to one short paragraph per round."`
  - if `designComplexity = standard`:
    `"DESIGN COMPLEXITY: STANDARD.\nUse balanced exploration (consider at least one alternative) and provide clear, practical handoff details.\nKeep designer logs to around two short paragraphs per round."`
  - if `designComplexity = deep`:
    `"DESIGN COMPLEXITY: DEEP.\nExplore multiple candidate ideas before converging. Provide richer design rationale, interaction detail, and edge-case notes for the developer.\nKeep designer logs to around three to four short paragraphs per round."`

```
Task tool (general-purpose):
  model: <from state.json - designerModel for phases 1,3,5 or developerModel for phases 2,4,6>
  description: "Phase N: [phase name]"
  prompt: |
    [System prompt from prompts.md with placeholders replaced]

    [Phase prompt from prompts.md]

    IMPORTANT: Your working directory is [absolute path to game dir].
    Write all files relative to this directory.

    TOOL RESTRICTIONS: You should ONLY use [Read, Write] tools.
    Do not use Bash, Edit, or any other tools.
    (For phase 6: You have full tool access.)
```

**Note on tool restrictions:** Task subagents cannot have tools restricted programmatically. Include explicit instructions in the prompt about which tools the agent should use. Design phase agents should be told to only use Read and Write. The build phase agent gets full access.

### 3. Completion

After all 6 phases complete:
- Update `state.json` status to `"complete"`
- Find the game source directory (any subdirectory that isn't `plans/` or `logs/`)
- Read the game's `README.md` if it exists
- Present a summary to the user:
  - Game name and location
  - How to run it
  - Whether any installs are needed (check for `SETUP.md`)
  - Pointer to `plans/` and `logs/` for reviewing the creative process

## Key Details

- **Personality seed** shapes designer creativity but isn't a literal directive — the designer is told to "surprise yourself"
- **Theme constraint** (optional): If provided, the game must incorporate this theme creatively
- **Game type constraint** (optional): If provided, the game must fit this genre/style
- **Runtime-aware model selection**: Use Anthropic models in Claude Code (`opus`/`sonnet`) and GPT models in Codex (`gpt-5.3-codex`/`gpt-5-mini`)
- **Design complexity**: `light`, `standard`, or `deep` changes designer ideation depth and handoff verbosity to the developer
- **Scope enforcement**: Designer prompt limits to game-jam-small; Developer rejects anything beyond "medium" complexity
- **macOS-compatible** tech only — browser games, Python/Pygame, Node, etc.
- **Assets directory**: Developer is instructed to put all art/images/sounds in `assets/` inside the game source directory
- **Collaboration log**: Both agents append reasoning to `logs/collaboration.md` each round
- **Pausable**: User can stop at any time (Ctrl+C or close Claude). Progress is automatically saved in `state.json` after each phase completes
- **Resumable**: Use `/game-jam resume` to resume any incomplete game jam. The skill tracks `currentPhase` and `completedPhases` in state.json and skips already-completed work

## Common Mistakes

- Skipping or batching setup questions — always ask all six questions one-by-one and wait for each answer
- Forgetting to update `state.json` between phases — always update before and after
- Using the wrong model for a phase — always read `designerModel` or `developerModel` from state.json
- Forgetting to apply `{{DESIGNER_COMPLEXITY_GUIDANCE}}` for designer phases
- Not providing the absolute working directory path in each subagent prompt
- Skipping the collaboration log instruction — agents need to be told to append to `logs/collaboration.md`
