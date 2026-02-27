# AI Game Jam Skill for Claude Code

A Claude Code skill that orchestrates two AI agents — a game designer and a game developer — to autonomously collaborate and produce a complete, playable game.

## What It Does

Give the skill a personality seed (like "chaotic goblin energy" or "zen minimalist"), optionally add a theme and game type, and watch as two AI agents work together to design and build a complete game. No human intervention needed!

**The agents collaborate to:**
- Create an initial game concept
- Develop technical specifications
- Refine the design through multiple rounds
- Build a complete implementation plan
- Actually build the game with all assets

**Games are:**
- Configurable scope: tiny (micro-game), small (classic jam), or medium (ambitious)
- macOS-compatible (browser, Python/Pygame, Node, etc.)
- Fully playable with assets and setup instructions included

## Installation

### Prerequisites

- [Claude Code](https://claude.com/code) installed
- macOS, Linux, or Windows with WSL

### Install

```bash
# For personal-level installation (available in all projects):
git clone https://github.com/codyparker/ai-game-jam ~/.claude/skills/game-jam
```

Restart Claude Code and the `/game-jam` skill will be available.

## Usage

### Start a New Game Jam

```bash
/game-jam
```

You'll be prompted for creative inputs (one at a time):

1. **Designer personality seed** (required)
   - Examples: "chaotic goblin energy", "zen minimalist", "1920s art deco enthusiast"
   - This shapes how the designer agent thinks creatively

2. **Game jam theme** (optional)
   - Examples: "bananas", "killer clown", "space", "time travel"
   - The game must incorporate this theme if provided

3. **Game type** (optional)
   - Examples: "roguelike", "2-bit color style", "text-based", "puzzle platformer"
   - The game must fit this genre/style if provided

4. **Adjust advanced settings?** (yes/no, default: no)
   - If yes, you can configure:
     - **Design complexity**: `light` / `standard` / `deep` — how deeply the designer explores ideas
     - **Game scope**: `tiny` / `small` / `medium` — how ambitious the game is
     - **Design rounds**: `3` / `5` / `7` — how many back-and-forth rounds before building

The skill will create a dated directory (e.g., `2026-02-08-game/`), run through the design and build phases, then rename the directory to the game name (e.g., `gravity-hopper/`).

### Resume an Incomplete Game Jam

If you stopped a game jam mid-process, you can resume it:

```bash
/game-jam resume
```

The skill will list all incomplete jams in your current directory and let you choose which to resume. It automatically skips completed phases and picks up where it left off.

## How It Works

### The Phases

Designer and developer alternate rounds (configurable: 3, 5, or 7 rounds), then the developer builds the game:

**5 rounds (default):**
1. **Designer** — Initial game concept
2. **Developer** — Technical feasibility response
3. **Designer** — Revised design incorporating feedback
4. **Developer** — Complete implementation plan
5. **Designer** — Final design sign-off and spec
6. **Developer** — Build the game

**3 rounds** compress to: concept → tech response + plan → sign-off → build.
**7 rounds** add an extra feedback/revision cycle before the implementation plan.

### Output Structure

```
2026-02-08-game/
├── state.json              # Progress tracking, phase timings, personality/theme/type/complexity/scope/rounds
├── plans/                  # Design documents from each phase
│   ├── 01-concept.md
│   ├── 02-tech-response.md
│   ├── 03-revised-design.md
│   ├── 04-impl-plan.md
│   └── 05-final-spec.md
├── logs/
│   └── collaboration.md    # Agent reasoning and decision log
├── stats.md                # Collaboration summary, timing, and game stats
└── [game-source]/          # The actual game
    ├── README.md           # How to run the game
    ├── SETUP.md           # Installation instructions (if needed)
    └── assets/            # All images, sounds, etc.
```

## Advanced Settings

Optional settings you can adjust (all have sensible defaults):

### Design Complexity
How deeply the *designer agent* explores ideas. Does not affect the developer.
- **light**: picks a direction quickly, brief handoff notes
- **standard** (default): considers alternatives, clear handoff
- **deep**: explores multiple directions with detailed rationale

### Game Scope
How big and ambitious the game is.
- **tiny**: single mechanic, minimal visuals, 2-3 minutes of play
- **small** (default): one core mechanic, simple visuals, ~5 minutes of fun
- **medium**: 1-2 interlocking mechanics, more polish, 10-15 minutes of play

### Design Rounds
How many back-and-forth rounds between designer and developer.
- **3**: fast — concept, tech response + plan, sign-off
- **5** (default): full cycle with revision
- **7**: extra feedback/revision loop for thorough iteration

## Examples

### Example 1: Simple Personality

```
Personality: "grumpy old wizard"
Theme: (none)
Type: (none)
```

Result: A spell-mixing puzzle game where you play as a cantankerous wizard trying to clean up your messy tower.

### Example 2: With Constraints

```
Personality: "hyperactive hamster"
Theme: "space"
Type: "roguelike"
```

Result: A fast-paced roguelike where you're a hamster in a hamster ball bouncing through space stations, collecting seeds and avoiding vacuum hazards.

### Example 3: Artistic Style

```
Personality: "1980s arcade machine enthusiast"
Theme: "pizza"
Type: "2-bit color style"
```

Result: A retro arcade game in limited color palette where you defend a pizza shop from invading toppings.

## Features

- **Fully autonomous** — No human intervention required during design and build
- **Pausable & resumable** — Stop at any time, resume later with `/game-jam resume`
- **Progress tracking** — All state saved in `state.json`
- **Stats tracking** — Generates `stats.md` with collaboration summary, phase timing, and game stats
- **Configurable** — Tune design complexity, game scope, and number of design rounds
- **Collaboration log** — See how the agents reasoned and made decisions
- **Quality constraints** — Built-in scope limits keep games simple and completable

## Troubleshooting

### Skill not appearing

- Make sure you've restarted Claude Code after installation
- Check that the files are in `.claude/skills/game-jam/` (project-level) or `~/.claude/skills/game-jam/` (personal-level)
- Verify that `SKILL.md` and `prompts.md` are present

### Game won't run

- Check the game's `README.md` for specific instructions
- Look for `SETUP.md` for any required installations
- Make sure you have the required runtime (Python, Node, browser, etc.)

### Agent phases getting stuck

- The agents are designed to be autonomous, but occasionally they may need a nudge
- Check `logs/collaboration.md` to see what they were thinking
- You can always resume with `/game-jam resume` if you stop the session

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

### Development Setup

1. Clone the repository
2. The skill files are in `skills/game-jam/`
3. See `CLAUDE.md` for detailed documentation on the architecture

## License

MIT License - see LICENSE file for details

## Credits

Created for Claude Code by the Claude Code community.

---

**Have fun!** The beauty of this skill is in its unpredictability. The same personality seed can produce wildly different games. Try experimenting with different combinations of personality, theme, and game type to see what the agents create.
