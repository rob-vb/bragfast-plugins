# bragfast-skills

Claude Code skills for [brag.fast](https://brag.fast) — generate branded release announcement images and videos.

## Prerequisites

Add the MCP server and authenticate:

```bash
claude mcp add bragfast -- npx -y @bragfast/mcp-server
npx @bragfast/mcp-server login <your-api-key>
```

Get an API key at [brag.fast/dashboard/account](https://brag.fast/dashboard/account).

## Install

### Option 1: CLI Install (Recommended)

Use [npx skills](https://github.com/anthropics/skills) to install skills directly:

```bash
# Install all skills
npx skills add rob-vb/bragfast-skills

# Install specific skill
npx skills add rob-vb/bragfast-skills --skill bragfast

# List available skills
npx skills add rob-vb/bragfast-skills --list
```

This automatically installs to your `.agents/skills/` directory (and symlinks into `.claude/skills/` for Claude Code compatibility).

### Option 2: Clone and Copy

Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/rob-vb/bragfast-skills.git
cp -r bragfast-skills/skills/* .agents/skills/
```

## Skills

| Skill | Description |
|-------|-------------|
| `/bragfast` | Generate release images or videos. Reads your conversation context to auto-compose slides, suggests templates based on what you're building. |

## License

MIT
