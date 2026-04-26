---
description: Explain how to configure or reconfigure the Prequel plugin (environment + JWT token)
---

Explain to the user how to configure (or change) the Prequel plugin's settings.

The plugin exposes two user-config options:

1. `environment` — either `dev` (https://mcp-dev.prequel.dev) or `prod` (https://mcp-beta.prequel.dev).
2. `jwt_token` — a Bearer JWT token, stored as a sensitive credential (system keychain, not settings.json).

To configure or reconfigure:

- **Recommended:** run `/plugin` and pick **prequel** to edit its user configuration through the plugin manager.
- **Manual (non-sensitive):** edit `~/.claude/settings.json` under `pluginConfigs.prequel.options.environment`.
- **Manual (sensitive token):** the JWT token lives in the OS keychain and is best updated via `/plugin`.

After changing either value, run `/reload-plugins` (or restart the session) so the MCP server reconnects with the new URL/token.

Then, to verify the connection works, suggest the user run `/prequel:status`.

Do not print the JWT token. If the user pastes one in chat, remind them it is sensitive and should be set through `/plugin` instead.
