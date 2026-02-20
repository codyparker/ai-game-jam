# Skill UX Improvements Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Four improvements to the game-jam skill: remove model prompts, rename folder at end, embody designer personality, add stats.md.

**Architecture:** All changes are to two markdown files — `skills/game-jam/SKILL.md` (orchestration logic) and `skills/game-jam/prompts.md` (agent prompts) — plus the `CLAUDE.md` docs that describe them.

**Tech Stack:** Markdown skill files (no traditional code/tests)

---

### Task 1: Remove model selection from setup flow (SKILL.md)

**Files:**
- Modify: `skills/game-jam/SKILL.md`

This task removes runtime detection, model selection questions, model-related state.json fields, and model dispatch logic. All subagents inherit the parent model.

**Step 1: Remove runtime detection block (lines 86-91)**

Remove the entire "Before model selection, determine runtime platform" block:
```
Before model selection, determine runtime platform:
- Prefer automatic detection from host identity and available model IDs.
- Set `runtime` to `claude` when the host is Claude Code / Anthropic.
- Set `runtime` to `codex` when the host is Codex / OpenAI.
- Set `runtime` to `opencode` when the host is OpenCode.
- If uncertain, ask the user to choose explicitly (`claude`, `codex`, or `opencode`) before continuing.
```

**Step 2: Remove model selection questions 4 and 5 (lines 111-123)**

Remove the entire "Second prompt" section including both model questions:
```
**Second prompt** (model selection, runtime-aware):

4. **Designer agent model** (optional):
   ...through...
   - If `runtime = opencode`: options are "Big Pickle (Recommended)" or "Small Pickle" (default: Big Pickle)
```

**Step 3: Rename "Third prompt" to "Second prompt" (line 125)**

Change `**Third prompt** (designer complexity):` to `**Second prompt** (designer complexity):`.

Renumber question 6 to question 4.

**Step 4: Update state.json schema (lines 144-170)**

Remove `runtime`, `designerModel`, `developerModel` fields from the example state.json. Remove the "Model values" and "Default model values" blocks entirely (lines 162-170).

The state.json example becomes:
```json
{
  "created": "2026-02-06T21:45:00Z",
  "personality": "<seed>",
  "theme": "<theme or null>",
  "gameType": "<game type or null>",
  "designComplexity": "standard",
  "currentPhase": "designer-round1",
  "completedPhases": [],
  "gameDir": null,
  "status": "in_progress"
}
```

**Step 5: Simplify phase dispatch (lines 172-227)**

Replace lines 174-176:
```
Each phase dispatches a Task subagent. Use the model specified in state.json:
- Designer phases (1, 3, 5): use `state.designerModel`
- Developer phases (2, 4, 6): use `state.developerModel`
```
With:
```
Each phase dispatches a Task subagent. All subagents inherit the current session's model automatically — do not pass a `model` parameter to the Task tool.
```

In the dispatch template (line 214), remove the `model:` line entirely.

**Step 6: Update resume mode display (lines 67-73)**

Line 67: Change "show personality, theme, gameType, complexity, runtime, models, and current phase" to "show personality, theme, gameType, complexity, and current phase".

Line 69: Change "personality/theme/gameType/complexity/runtime, and models being used" to "personality/theme/gameType/complexity".

Remove line 73 (backfill for missing `runtime` field). Keep the backfill for `designComplexity`.

**Step 7: Update graph labels (lines 29, 38)**

Line 29: `"Design Rounds (use selected designer model)"` → `"Design Rounds"`
Line 38: `"Build Phase (use selected developer model)"` → `"Build Phase"`

**Step 8: Update Key Details section (line 248)**

Remove: `- **Runtime-aware model selection**: Use Anthropic models in Claude Code...`

**Step 9: Update Common Mistakes section (line 261)**

Remove: `- Using the wrong model for a phase — always read...`

Update the "Skipping or batching setup questions" line to say "four" instead of "six".

**Step 10: Commit**

```bash
git add skills/game-jam/SKILL.md
git commit -m "Remove model selection prompts — inherit from parent session"
```

---

### Task 2: Rewrite designer personality prompt (prompts.md)

**Files:**
- Modify: `skills/game-jam/prompts.md`

**Step 1: Replace the designer system prompt (lines 12-27)**

Replace the current designer system prompt with one that makes the designer embody the personality rather than reference it:

```
You are a game designer entering a game jam. Here's who you are:

"{{PERSONALITY}}"

This is your identity — not a suggestion, not a reference. Think like this person. Talk like this person. Get excited about the things this person would get excited about. Let your personality come through in every design decision, every word you write, every idea you pitch.

Don't ever say things like "my personality prompt says" or "I'm told to be" — you just ARE this. If the personality is "chaotic goblin energy" then you're a chaotic goblin who happens to design games. If it's "a retired pirate who misses the sea" then you're a retired pirate at a game jam, pouring your longing into game design.

{{THEME_CONSTRAINT}}

{{GAME_TYPE_CONSTRAINT}}

{{DESIGNER_COMPLEXITY_GUIDANCE}}

CRITICAL SCOPE RULE: The game you design must be buildable by a single developer in one session. Think game-jam-small: one core mechanic, simple visuals, 5 minutes of fun. If you're designing more than 3 enemy types, more than 5 levels, or any kind of save system — you've gone too far. Strip it back.
```

**Step 2: Commit**

```bash
git add skills/game-jam/prompts.md
git commit -m "Rewrite designer prompt to embody personality instead of referencing it"
```

---

### Task 3: Add folder rename at completion (SKILL.md)

**Files:**
- Modify: `skills/game-jam/SKILL.md`

**Step 1: Add rename logic to the Completion section (after line 235)**

After the existing completion steps, add folder rename as the final step before presenting the summary. Insert after "Find the game source directory":

```
- Rename the game directory: determine the game name from the game source subdirectory name (the directory that isn't `plans/`, `logs/`, or `state.json`). Rename the top-level dated directory to just the game name:
  ```bash
  # Example: rename 2026-02-19-game/ to gravity-hopper/
  mv "<dated-dir>" "<game-name>"
  ```
  - If a directory with the target name already exists in the parent, append a numeric suffix (`gravity-hopper-2/`, etc.)
  - Update any internal paths if needed (state.json is relative, so it should still work)
  - Tell the user the directory was renamed
```

**Step 2: Commit**

```bash
git add skills/game-jam/SKILL.md
git commit -m "Add folder rename to game name at completion"
```

---

### Task 4: Add phase timing capture (SKILL.md)

**Files:**
- Modify: `skills/game-jam/SKILL.md`

**Step 1: Add phaseTimings to state.json schema**

Add `"phaseTimings": {}` to the state.json example.

**Step 2: Add timing instructions to the "Before each phase" / "After each phase" blocks**

In the "Before each phase" block, add:
```
- Record the start time: run `date +%s` via Bash and store the timestamp in `state.json` under `phaseTimings.<phase-name>.start`
```

In the "After each phase" block, add:
```
- Record the end time: run `date +%s` via Bash and store the timestamp in `state.json` under `phaseTimings.<phase-name>.end`
```

**Step 3: Commit**

```bash
git add skills/game-jam/SKILL.md
git commit -m "Add per-phase timing capture to state.json"
```

---

### Task 5: Add stats.md generation at completion (SKILL.md)

**Files:**
- Modify: `skills/game-jam/SKILL.md`

**Step 1: Add stats.md generation step to the Completion section**

After updating state.json to complete and before the folder rename, add a new step:

```
- Generate `stats.md` in the game directory:
  1. Read `logs/collaboration.md` and all plan files
  2. Run file/LOC counting on the game source directory:
     ```bash
     find <game-source-dir> -type f | wc -l        # file count
     find <game-source-dir> -type f -name "*.py" -o -name "*.js" -o -name "*.html" -o -name "*.css" | xargs wc -l  # LOC
     ```
  3. Read `phaseTimings` from state.json and compute durations
  4. Write `stats.md` with this structure:

  ```markdown
  # [Game Title] — Jam Stats

  ## The Story
  (A fun, engaging narrative summary of the entire collaboration — what the designer
  envisioned, how the developer responded, what got cut, what survived, and what the
  final game became. Write this in an entertaining way that captures the creative
  back-and-forth. 2-3 paragraphs.)

  ## Timeline
  | Phase | Duration | What Happened |
  |-------|----------|---------------|
  | 1. Initial Concept | Xm Ys | (one-line summary) |
  | 2. Tech Response | Xm Ys | (one-line summary) |
  | 3. Revised Design | Xm Ys | (one-line summary) |
  | 4. Implementation Plan | Xm Ys | (one-line summary) |
  | 5. Final Sign-off | Xm Ys | (one-line summary) |
  | 6. Build | Xm Ys | (one-line summary) |
  | **Total** | **Xm Ys** | |

  ## Model
  - **Session model**: (model name inherited from parent session)

  ## The Game
  - **Title**: [name]
  - **Tech stack**: [from plans/02-tech-response.md]
  - **Files created**: [count]
  - **Lines of code**: ~[count]

  ## Design Evolution
  - **Original pitch**: (elevator pitch from 01-concept.md)
  - **What changed**: (key changes from round 1 → final spec)
  - **Features cut**: (things that were in scope check "OUT" list or cut during revision)
  - **Developer's biggest concern**: (from 02-tech-response.md scope concerns)
  ```

  The orchestrator writes this directly — no subagent needed. Use the collaboration log and plan files as source material.
```

**Step 2: Commit**

```bash
git add skills/game-jam/SKILL.md
git commit -m "Add stats.md generation at completion"
```

---

### Task 6: Update CLAUDE.md to match changes

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Update all references to match new behavior**

- Remove all mentions of runtime, model selection, designerModel, developerModel
- Update "6 setup questions" references to "4 setup questions"
- Add mention of stats.md generation
- Add mention of folder rename at completion
- Update state.json field descriptions
- Update the Architecture section to remove model selection from Setup description

**Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "Update CLAUDE.md to reflect skill changes"
```
