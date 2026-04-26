# prequel

A Claude Code plugin for working with the Prequel MCP servers over HTTP with JWT authentication.

The plugin registers a single remote MCP server (`prequel`) whose URL is selected by environment:

| Environment | URL                            |
| ----------- | ------------------------------ |
| `dev`       | https://mcp-dev.prequel.dev    |
| `prod`      | https://mcp-beta.prequel.dev   |

Authentication is a Bearer JWT sent in the `Authorization` header.

## Layout

```
prequel-claude/
├── .claude-plugin/
│   └── plugin.json      # manifest + userConfig (environment, jwt_token)
├── .mcp.json            # HTTP MCP server with Bearer auth
└── commands/
    ├── configure.md     # /prequel:configure
    ├── status.md        # /prequel:status
    └── tools.md         # /prequel:tools
```

## Testing the plugin

You have two options for running the plugin against a local checkout.

### Option 1: launch Claude Code with `--plugin-dir`

```sh
claude --plugin-dir /Users/amaus/prequel/git/prequel-claude
```

Claude Code loads the plugin for that session only and prompts for `environment` and `jwt_token` on first use.

### Option 2: register as a local plugin

From inside Claude Code:

1. Run `/plugin`.
2. Add a local plugin pointing at this directory.
3. Enable it. Claude Code prompts for `environment` (`dev` or `prod`) and `jwt_token`.
4. Verify the MCP server is connected with `/prequel:status`.

After editing any plugin file (manifest, `.mcp.json`, commands) run `/reload-plugins` to pick up changes without restarting the session.

### Smoke test checklist

- `/prequel:status` reports the active environment and at least one MCP tool.
- `/prequel:tools` lists the tools the server exposes.
- Switching `environment` via `/plugin` and reloading flips the URL the MCP client connects to (you can confirm with `/prequel:status`).
- An invalid or expired JWT yields a connection error and zero tools — fix by updating the token through `/plugin`.

## Using the plugin

Once configured, the `prequel` MCP server's tools become available to Claude under the `mcp__prequel__*` namespace. You can either:

- Ask Claude to perform a task and let it pick the right Prequel tool, or
- Run one of the slash commands the plugin ships:

| Command              | What it does                                                         |
| -------------------- | -------------------------------------------------------------------- |
| `/prequel:configure` | Explains how to view and change the plugin's configuration.          |
| `/prequel:status`    | Verifies the MCP server is reachable and reports the active env.    |
| `/prequel:tools`     | Lists and describes the MCP tools currently exposed by the server.  |

Slash commands are namespaced with the plugin name, so `/prequel:status` will not collide with other plugins.

## How configuration is stored

The plugin declares two `userConfig` options in [.claude-plugin/plugin.json](.claude-plugin/plugin.json):

- `environment` — string, **not sensitive**.
- `jwt_token` — string, marked `sensitive: true`.

These two values are stored in different places.

### `environment` → `settings.json`

Plain text, readable on disk:

```json
// ~/.claude/settings.json (user scope) or .claude/settings.json (project scope)
{
  "pluginConfigs": {
    "prequel": {
      "options": {
        "environment": "prod"
      }
    }
  }
}
```

The exact file depends on which scope the plugin was enabled in:

- User scope: `~/.claude/settings.json`
- Project scope: `<repo>/.claude/settings.json` or `.claude/settings.local.json`

### `jwt_token` → OS keychain

Because `sensitive: true` is set, the token is **not** written to `settings.json`. It is stored in the operating system's secret store:

- **macOS** — Keychain Access, under the `Claude Code` service.
- **Linux** — Secret Service / libsecret (GNOME Keyring, KWallet, etc).
- **Fallback** — if no keychain is available, `~/.claude/.credentials.json` with file mode `600`.

### How the values reach the MCP server

[.mcp.json](.mcp.json) references both options via the `${user_config.<key>}` substitution syntax:

```json
{
  "mcpServers": {
    "prequel": {
      "type": "http",
      "url": "https://mcp-${user_config.environment}.prequel.dev",
      "headers": {
        "Authorization": "Bearer ${user_config.jwt_token}"
      }
    }
  }
}
```

Substitution happens at plugin load time. The `environment` value is read from `settings.json`; the `jwt_token` is fetched out of the keychain. The token never lands on disk in `settings.json`.

### Changing or clearing the configuration

- **Recommended:** run `/plugin`, select **prequel**, and edit the user configuration. Run `/reload-plugins` afterwards.
- **Manual (env only):** edit `pluginConfigs.prequel.options.environment` in the relevant `settings.json` and reload.
- **Manual (token):** update the entry in your OS keychain (or in `~/.claude/.credentials.json` on the fallback path), then reload.

To remove the plugin's stored configuration entirely, disable the plugin via `/plugin` — that removes the `pluginConfigs.prequel` block from `settings.json` and the keychain entry.
