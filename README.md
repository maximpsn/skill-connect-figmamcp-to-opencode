# skill-connect-figmamcp-to-opencode

Automates connecting the Figma MCP server to OpenCode-based agents (MultiTool) when Figma rejects the client's OAuth identity.

Figma's MCP server uses an OAuth **client allowlist** that does not include every agent host. Method: register under an allowlisted client name (Codex) via [mcp-auth-helper](https://github.com/sdaoudi/mcp-auth-helper), then point the agent at the server. This skill automates that whole flow — checks the config, copies a fresh OAuth token, fixes files, asks only for a restart.

## Layout

- `SKILL.md` — instructions for the LLM agent: check `mcp.figma` in `opencode.json`, verify/refresh the OAuth token in `mcp-auth.json`, elevate via UAC to write the locked file, then restart.

## Scope

- Trigger: a Figma-MCP instruction file/link dropped into the chat, or phrases like «подключи фигму», «настрой MCP фигмы», "set up Figma MCP".
- Full autopilot: only a UAC prompt stays with the user.
- Windows + PowerShell only, no external dependencies.

## Credits

The underlying trick (Figma MCP OAuth client allowlist → authenticate under Codex) comes from [sdaoudi/mcp-auth-helper](https://github.com/sdaoudi/mcp-auth-helper) — a standalone OAuth helper for MCP servers with client allowlists, which works with OpenCode without forking. It also tracks the upstream issue ([sst/opencode#5636](https://github.com/sst/opencode/issues/5636)). This repo automates that helper's flow so the agent performs the setup itself.