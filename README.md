# Teacher Skill

AI learning assistant for Claude Code. Input `/learn` to get started with goal decomposition, phased learning plans, one-on-one Q&A teaching, progress tracking, and Obsidian note generation.

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/<your-username>/teacher-skill

# 2. Install the plugin
claude plugin install teacher-skill@teacher-skill-marketplace
```

Restart Claude Code, then type `/learn` to start learning.

## Usage

1. Type `/learn` — the system asks what you want to learn
2. State your goal (e.g., "pass CET-6", "get a Java backend job")
3. The system breaks it into a skill tree with proficiency levels
4. Confirm the plan, then start daily learning sessions
5. After each session, notes are generated as `.md` files

## Commands

| Input | Action |
|-------|--------|
| `/learn` + goal | Create a new learning plan |
| `/learn` + "start/continue" | Begin daily Q&A learning |
| `/learn` + "progress" | Check learning progress |
| `/learn` + "done/summarize" | Generate session notes |

## Skills

| Skill | Purpose |
|-------|---------|
| `learn` | Entry point router |
| `curriculum-builder` | Goal decomposition + plan creation |
| `teacher-core` | One-concept-one-question teaching |
| `progress-tracker` | Progress management |
| `note-generator` | Obsidian note generation |

## Uninstall

```bash
claude plugin disable teacher-skill@teacher-skill-marketplace
```

## License

MIT
