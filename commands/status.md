---
description: Check the Prequel MCP server connection and list available tools
---

Verify that the Prequel MCP server (`https://mcp-beta.prequel.dev`) is reachable and report its status.

Steps:

1. List the MCP tools currently exposed by the `prequel` server. If no tools are visible, the server is likely not connected — tell the user to check:
   - Their JWT token is valid and not expired
   - They have run `/reload-plugins` after changing config
2. Report a one-line summary of how many Prequel MCP tools are currently available.

Do not print or log the JWT token under any circumstances.
