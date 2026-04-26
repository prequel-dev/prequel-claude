---
description: Check the Prequel MCP server connection and list available tools
---

Verify that the Prequel MCP server is reachable and report its status.

Steps:

1. Identify which environment is configured (look at `pluginConfigs.prequel.options.environment` in `~/.claude/settings.json`, or infer from the active MCP server URL).
2. List the MCP tools currently exposed by the `prequel` server. If no tools are visible, the server is likely not connected — tell the user to check:
   - Their JWT token is valid and not expired
   - The selected environment (`dev` or `prod`) is correct
   - They have run `/reload-plugins` after changing config
3. Report a one-line summary: which environment is in use and how many tools are available.

Do not print or log the JWT token under any circumstances.
