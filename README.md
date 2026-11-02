# skill-connect-figmamcp-to-opencode

Skill that automates connecting the Figma MCP server to an OpenCode-based agent
(such as multitool.works and others) when the client's OAuth identity is rejected.

It works in two stages, official first:

1. **Official path.** Figma now supports OpenCode in its OAuth client allowlist,
   so pure OpenCode can pass native OAuth out of the box. The skill adds the
   remote server (`https://mcp.figma.com/mcp`) and lets the engine authenticate.
2. **Fallback** (only if native fails — typical for agents with a non-allowlisted
   host id, e.g. MultiTool/`gigatool`): registers under an allowlisted client
   (Codex) via `mcp-auth-helper`, or copies a ready OAuth token from a sibling
   OpenCode config.

The skill detects the host/agent, checks the `mcp.figma` entry in `opencode.json`,
verifies or copies a fresh OAuth token into `mcp-auth.json`, writes the locked file
through a UAC prompt, and asks for a restart.

## Layout

- `SKILL.md` — instructions for the LLM agent.

## Scope

- Trigger: a Figma-MCP instruction file/link dropped into the chat, or phrases
  like «подключи фигму», «настрой MCP фигмы», "set up Figma MCP".
- Full autopilot: only the UAC prompt stays with the user.
- Windows + PowerShell only, no external dependencies.

## How to install

Place the `SKILL.md` under the agent's global skills folder, e.g.
`~/.config/gigatool/skills/<name>/` (MultiTool) or the equivalent path for your
OpenCode-based agent. Restart the agent. Rest is handled by the skill.

## Credits

The underlying trick (Figma MCP OAuth client allowlist → authenticate under an
allowlisted client) comes from
[sdaoudi/mcp-auth-helper](https://github.com/sdaoudi/mcp-auth-helper) — a
standalone OAuth helper for MCP servers with client allowlists, which works with
OpenCode without forking (upstream tracking:
[sst/opencode#5636](https://github.com/sst/opencode/issues/5636)). This repo
automates that helper's flow so the agent performs the setup itself.
