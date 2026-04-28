---
description: Explain how to configure or reconfigure the Prequel plugin (JWT token)
---

Explain to the user how to configure (or change) the Prequel plugin's settings.

The plugin connects to a single MCP server: `https://mcp-beta.prequel.dev`. There is one user-config option:

- `jwt_token` — a Bearer JWT token, stored as a sensitive credential (system keychain, not settings.json).

To configure or reconfigure:

- **Recommended:** run `/plugin` and pick **prequel** to edit its user configuration through the plugin manager.
- **Manual:** the JWT token lives in the OS keychain and is best updated via `/plugin`.

After updating the token, run `/reload-plugins` (or restart the session) so the MCP server reconnects with the new token.

Then, to verify the connection works, suggest the user run `/prequel:status`.

Do not print the JWT token. If the user pastes one in chat, remind them it is sensitive and should be set through `/plugin` instead.
