# bragfast-skills

Claude Code skills for [brag.fast](https://brag.fast) — generate branded release announcement images and videos.

## Prerequisites

Add the MCP server and authenticate:

```bash
claude mcp add bragfast -- npx -y @bragfast/mcp-server@latest
npx @bragfast/mcp-server login <your-api-key>
```

Get an API key at [brag.fast/dashboard/account](https://brag.fast/dashboard/account).

## Install

### Option 1: CLI Install (Recommended)

Use [npx skills](https://github.com/anthropics/skills) to install skills directly:

```bash
npx skills add rob-vb/bragfast-skills
```

This automatically installs to your `.agents/skills/` directory (and symlinks into `.claude/skills/` for Claude Code compatibility).

### Option 2: Claude Code Plugin

Install via Claude Code's built-in plugin system:

```bash
# Add the marketplace
/plugin marketplace add rob-vb/bragfast-skills

# Install all skills
/plugin install bragfast-skills
```

### Option 3: Clone and Copy

Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/rob-vb/bragfast-skills.git
cp -r bragfast-skills/skills/* .agents/skills/
```

## Skills

| Skill | Description |
|-------|-------------|
| `/bragfast` | Generate release images or videos. Auto-reads your git history to compose slides with zero input — just type `/bragfast` after shipping. Falls back to conversation context if no git history is available. |

## License

MIT
