# R3 plugin for Claude

R3 (https://r3js.org) is a browser platform for interactive 3D: scenes, physics, sound and music, built from components. This plugin lets Claude build and change R3 projects on your own R3 account: add objects, set their properties, write the code that makes them move, make sounds and music, import your Blender models, and make a preview link to share.

## What it contains

- **The R3 MCP server** (`.mcp.json`): a remote server at `https://mcp.r3js.org/mcp`. You connect with **Sign in with R3**: you sign in (or create a free account), see which app is asking, choose what it may do and press Allow. There is no key to paste.
- **The R3 workflow skill** (`skills/r3/SKILL.md`): it teaches Claude how to work with R3: read R3's own docs first, load the project, then change it, and check the result with a preview.

## What it sends where

The plugin runs nothing on your machine. Claude talks only to R3's server at `https://mcp.r3js.org/mcp`, with the rights you chose when you connected. Your projects, files and settings stay in your R3 account. You can disconnect at any time in R3 under Settings → AI & apps → Connected apps.

## Install in Claude Code

```
claude plugin marketplace add https://mcp.r3js.org/claude/marketplace.json
claude plugin install r3@r3js
```

Then ask Claude, for example: "Make me a bouncing ball with a sound in R3 and show me a preview."

## Links

- Documentation: https://r3js.org/documentation/mcp
- Privacy policy: https://r3js.org/privacy-policy
- Support: https://r3js.org/support

## License

MIT, see `LICENSE`.
