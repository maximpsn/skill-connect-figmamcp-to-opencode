# skill-connect-figmamcp-to-opencode

Skill that automates connecting the Figma MCP server to an OpenCode-based agent
(MultiTool, OpenCode, …) when Figma rejects the client's OAuth identity.

Figma's MCP server uses an OAuth **client allowlist** that does not include every
agent host. The workaround registers under an allowlisted client (Codex) via
`mcp-auth-helper`. This skill automates that flow: detects the host/agent, checks
the `mcp.figma` entry in `opencode.json`, verifies or copies a fresh OAuth token
into `mcp-auth.json`, writes the locked file through a UAC prompt, and asks for a
restart.

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