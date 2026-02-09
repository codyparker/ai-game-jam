# AI Game Jam Skill for Claude Code

A Claude Code skill that orchestrates two AI agents — a game designer and a game developer — to autonomously collaborate and produce a complete, playable game.

## What It Does

Give the skill a personality seed (like "chaotic goblin energy" or "zen minimalist"), optionally add a theme and game type, and watch as two AI agents work together through 6 phases to design and build a complete game. No human intervention needed!

**The agents collaborate to:**
- Create an initial game concept
- Develop technical specifications
- Refine the design through multiple rounds
- Build a complete implementation plan
- Actually build the game with all assets

**Games are:**
- Small, game-jam scope (one core mechanic, ~5 minutes of fun)
- macOS-compatible (browser, Python/Pygame, Node, etc.)
- Fully playable with assets and setup instructions included

## Installation

### Prerequisites

- [Claude Code](https://claude.com/code) installed
- macOS, Linux, or Windows with WSL

### Quick Install (Recommended)

Install directly from the Claude Skills marketplace:

```bash
# In Claude Code, add the marketplace
/plugin marketplace add codyparker/claude-skills

# Install the game-jam plugin
/plugin install game-jam@codyparker-claude-skills
```

The `/game-jam` skill will be immediately available!

### Manual Installation (Alternative)

If you prefer to install manually:

```bash
# For personal-level installation (available in all projects):
git clone https://github.com/codyparker/ai-game-jam ~/.claude/skills/game-jam
```

Restart Claude Code and the skill should appear when you type `/game-jam`.

## Usage

### Start a New Game Jam

```bash
/game-jam
```

You'll be prompted for:

1. **Designer personality seed** (required)
   - Examples: "chaotic goblin energy", "zen minimalist", "1920s art deco enthusiast"
   - This shapes how the designer agent thinks creatively

2. **Game jam theme** (optional)
   - Examples: "bananas", "killer clown", "space", "time travel"
   - The game must incorporate this theme if provided

3. **Game type** (optional)
   - Examples: "roguelike", "2-bit color style", "text-based", "puzzle platformer"
   - The game must fit this genre/style if provided

4. **Model selection**
   - Choose between Opus 4.6 (recommended, higher quality) or Sonnet 4.5 (faster) for each agent
   - Designer model for creative phases
   - Developer model for technical phases and building

The skill will create a dated directory (e.g., `2026-02-08-game/`) and run through all 6 phases automatically.

### Resume an Incomplete Game Jam

If you stopped a game jam mid-process, you can resume it:

```bash
/game-jam resume
```

The skill will list all incomplete jams in your current directory and let you choose which to resume. It automatically skips completed phases and picks up where it left off.

## How It Works

### The 6 Phases

1. **Designer Round 1** — Initial game concept
2. **Developer Round 1** — Technical feasibility response
3. **Designer Round 2** — Revised design incorporating feedback
4. **Developer Round 2** — Complete implementation plan
5. **Designer Round 3** — Final design sign-off and spec
6. **Developer Build** — Actually builds the game

### Output Structure

```
2026-02-08-game/
├── state.json              # Progress tracking, model selections, personality/theme
├── plans/                  # Design documents from each phase
│   ├── 01-concept.md
│   ├── 02-tech-response.md
│   ├── 03-revised-design.md
│   ├── 04-impl-plan.md
│   └── 05-final-spec.md
├── logs/
│   └── collaboration.md    # Agent reasoning and decision log
└── [game-source]/          # The actual game
    ├── README.md           # How to run the game
    ├── SETUP.md           # Installation instructions (if needed)
    └── assets/            # All images, sounds, etc.
```

## Model Selection

The skill supports two model choices for each agent role:

- **Opus 4.6** (recommended, default): More capable, produces higher quality and more playable games
- **Sonnet 4.5**: Faster, but may produce confusing or lower quality results

You'll be asked to choose models for:
- **Designer agent** (phases 1, 3, 5)
- **Developer agent** (phases 2, 4, 6)

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

- **Fully autonomous** — No human intervention required during the 6 phases
- **Pausable & resumable** — Stop at any time, resume later with `/game-jam resume`
- **Progress tracking** — All state saved in `state.json`
- **Model flexibility** — Choose Opus 4.6 or Sonnet 4.5 for each agent role
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
2. The skill files are in `.claude/skills/game-jam/`
3. See `CLAUDE.md` for detailed documentation on the architecture

## License

MIT License - see LICENSE file for details

## Credits

Created for Claude Code by the Claude Code community.

---

**Have fun!** The beauty of this skill is in its unpredictability. The same personality seed can produce wildly different games. Try experimenting with different combinations of personality, theme, and game type to see what the agents create.
