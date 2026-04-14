# claude-skills

A collection of reusable skills for [Claude Code](https://claude.ai/code). Each skill is a playbook that extends Claude's capabilities — invoke them manually with `/skill-name` or let Claude use them automatically when relevant.

## Skills

| Skill | Command | Description |
|---|---|---|
| [document](./document/SKILL.md) | `/document` | Generate C4 architecture docs, security analysis, dead code detection, request maps, and env var inventory for any project |

## Usage

Skills in this repo are installed globally at `~/.claude/skills/` and available across all projects.

```bash
# Full documentation suite
/document

# Targeted reports
/document c4          # C4 architecture diagrams only
/document security    # Security analysis only
/document dead-code   # Unused code detection only
/document env         # Environment variables inventory
/document bugs        # Bugs and code quality
/document requests    # HTTP request map
/document update      # Update existing docs after code changes
```

## Adding a Skill

Create a new directory with a `SKILL.md` file:

```
skills/
└── your-skill-name/
    └── SKILL.md
```

Minimum `SKILL.md` structure:

```markdown
---
name: your-skill-name
description: One-line description of what this skill does and when to use it
---

Instructions for Claude to follow when this skill is invoked...
```

Changes pushed to this repo are picked up immediately by Claude Code.

## License

MIT
