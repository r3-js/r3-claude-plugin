# R3 plugin for Claude

R3 (https://r3js.org) is a browser platform for interactive 3D: scenes, physics, sound and music, built from components. This plugin lets Claude build and change R3 projects on your own R3 account: add objects, set their properties, write the code that makes them move, make sounds and music, import your Blender models, and make a preview link to share.

## What it contains

- **The R3 MCP server** (`.mcp.json`): a remote server at `https://mcp.r3js.org/mcp`. You connect with **Sign in with R3**: you sign in (or create a free account), see which app is asking, choose what it may do and press Allow. There is no key to paste.
- **The R3 workflow skill** (`skills/r3/SKILL.md`): it teaches Claude how to work with R3: read R3's own docs first, load the project, then change it, and check the result with a preview.

There are no hooks, commands, agents, scripts or local programs in this plugin.

## What it sends where

The plugin runs nothing on your machine. Claude talks only to R3's server at `https://mcp.r3js.org/mcp`, with the rights you chose when you connected. What Claude sends there is what the tools need for your request: project names, component settings, the code Claude writes for your project, and the like. Your projects, files and settings stay in your R3 account. Preview links and live pages are served by R3 under r3js.org. You can disconnect at any time in R3 under Settings → AI & apps → Connected apps.

Privacy policy: https://r3js.org/privacy-policy. Terms of service: https://r3js.org/terms-of-service.

## Set up

1. Add the R3 plugin. Once it is listed in the Claude directory, add it there: it is listed as `r3js` (shown as "R3 (r3js.org)"), and its skill is `r3js:r3`. In Claude Code you can also add it from R3's own marketplace, where the plugin is named `r3`:

   ```
   claude plugin marketplace add https://mcp.r3js.org/claude/marketplace.json
   claude plugin install r3@r3js
   ```

2. Connect the R3 server. On claude.ai and in Cowork, connect it on the plugin's Connectors tab. In Claude Code, run `/mcp`, pick `r3` and choose Authenticate.
3. Sign in with R3 (or create a free account), check what Claude may do, and press Allow.

## Try it

- "Build my first 3D scene: a ground, a few coloured objects, a light and a camera that frames them, and give me a preview link."
- "Make me a satisfying domino run in R3: a long spiral of colourful dominoes that I push over and that clack as they fall, and give me a link."
- "Make me a 3D arcade maze game in R3 with neon walls, ghosts and sounds, and give me a link to play it."
- "Make me a looping helicopter rotor sound and add it to my heli project."
- "Put my Blender model 'monster-truck' into a scene and give me a link anyone can open." (Upload the .blend file at https://r3js.org/blender first.)

Scenes end with a preview link that stays private for 7 days. In claude.ai chat, the preview can also play right in the conversation. Going live is a separate step that Claude takes only when you ask.

## Troubleshooting

- **Claude says it is not connected to R3:** in Claude Code run `/mcp` and authenticate `r3`; on claude.ai or in Cowork, connect R3 on the plugin's Connectors tab.
- **A tool is refused for a missing right:** you did not allow that right when you connected. Disconnect Claude under Settings → AI & apps → Connected apps, connect again, and tick the right on the Allow page.
- **The tools that run, inspect or capture your scene need an open tab:** open https://r3js.org in a browser, signed in to the same R3 account. Without a tab, ask for a preview link instead.
- **Something in R3 misbehaves:** ask Claude to report it (it files a bug report with R3), or write to us at https://r3js.org/support.

## Links

- Documentation: https://r3js.org/documentation/mcp
- Privacy policy: https://r3js.org/privacy-policy
- Terms of service: https://r3js.org/terms-of-service
- Support: https://r3js.org/support

## License

MIT, see `LICENSE`.
