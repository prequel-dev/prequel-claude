---
description: List and describe the MCP tools exposed by the Prequel server
---

Enumerate the tools currently exposed by the `prequel` MCP server and describe what each one does.

For each tool:
- Show the tool name (as namespaced by Claude, e.g. `mcp__prequel__<tool>`)
- Summarize its purpose in one sentence
- Note its required vs optional parameters

If `$ARGUMENTS` is non-empty, treat it as a tool name or keyword the user wants to focus on, and only describe matching tools.

If no Prequel MCP tools are available in this session, tell the user to run `/prequel:status` to diagnose the connection.
