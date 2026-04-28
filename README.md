# Prequel Claude Code plugin

A Claude Code plugin for working with the Prequel MCP server over HTTP with JWT authentication.

The plugin registers a single remote MCP server (`prequel`) at:

```
https://mcp-beta.prequel.dev
```

Authentication is a Bearer JWT sent in the `Authorization` header.

## Installation

From inside Claude Code:

```
/plugin marketplace add prequel-dev/prequel-claude
/plugin install prequel@prequel
```

The first command registers this repo as a marketplace (the marketplace's `name` is `prequel`, defined in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json)). The second installs the `prequel` plugin from it.

On install, Claude Code prompts for the one `userConfig` value: `jwt_token`. The token is stored as a sensitive credential — see [Configuration storage](#how-configuration-is-stored) below.

The `jwt_token` can be created through the web UI: https://app-beta.prequel.dev/api-tokens.

Choose the `"viewer"` role for the token, this is sufficient for MCP and the plugin functionality.

To pull in updates after the upstream repo changes:

```
/plugin marketplace update prequel
```

## Layout

```
prequel-claude/
├── .claude-plugin/
│   ├── plugin.json        # manifest + userConfig (jwt_token)
│   └── marketplace.json   # marketplace catalog (one plugin: prequel)
├── .mcp.json              # HTTP MCP server with Bearer auth
└── commands/
    ├── configure.md       # /prequel:configure
    ├── status.md          # /prequel:status
    └── tools.md           # /prequel:tools
```

## Testing the plugin

You have two options for running the plugin against a local checkout.

### Option 1: launch Claude Code with `--plugin-dir`

```sh
claude --plugin-dir ./prequel-claude
```

Claude Code loads the plugin for that session only and prompts for `jwt_token` on first use.

### Option 2: register as a local plugin

From inside Claude Code:

1. Run `/plugin`.
2. Add a local plugin pointing at this directory.
3. Enable it. Claude Code prompts for `jwt_token`.
4. Verify the MCP server is connected with `/prequel:status`.

After editing any plugin file (manifest, `.mcp.json`, commands) run `/reload-plugins` to pick up changes without restarting the session.

### Smoke test checklist

- `/prequel:status` reports at least one MCP tool.
- `/prequel:tools` lists the tools the server exposes.
- An invalid or expired JWT yields a connection error and zero tools — fix by updating the token through `/plugin`.

## Using the plugin

Once configured, the `prequel` MCP server's tools become available to Claude under the `mcp__prequel__*` namespace. You can either:

- Ask Claude to perform a task and let it pick the right Prequel tool, or
- Run one of the slash commands the plugin ships:

| Command              | What it does                                                         |
| -------------------- | -------------------------------------------------------------------- |
| `/prequel:configure` | Explains how to view and change the plugin's configuration.          |
| `/prequel:status`    | Verifies the MCP server is reachable and reports tool availability. |
| `/prequel:tools`     | Lists and describes the MCP tools currently exposed by the server.  |

Slash commands are namespaced with the plugin name, so `/prequel:status` will not collide with other plugins.

## How configuration is stored

The plugin declares one `userConfig` option in [.claude-plugin/plugin.json](.claude-plugin/plugin.json):

- `jwt_token` — string, marked `sensitive: true`.

### `jwt_token` → OS keychain

Because `sensitive: true` is set, the token is **not** written to `settings.json`. It is stored in the operating system's secret store:

- **macOS** — Keychain Access, under the `Claude Code` service.
- **Linux** — Secret Service / libsecret (GNOME Keyring, KWallet, etc).
- **Fallback** — if no keychain is available, `~/.claude/.credentials.json` with file mode `600`.

### How the value reaches the MCP server

[.mcp.json](.mcp.json) references the token via the `${user_config.<key>}` substitution syntax:

```json
{
  "mcpServers": {
    "prequel": {
      "type": "http",
      "url": "https://mcp-beta.prequel.dev",
      "headers": {
        "Authorization": "Bearer ${user_config.jwt_token}"
      }
    }
  }
}
```

Substitution happens at plugin load time. The `jwt_token` is fetched out of the keychain — it never lands on disk in `settings.json`.

### Changing or clearing the configuration

- **Recommended:** run `/plugin`, select **prequel**, and edit the user configuration. Run `/reload-plugins` afterwards.
- **Manual:** update the token entry in your OS keychain (or in `~/.claude/.credentials.json` on the fallback path), then reload.

To remove the plugin's stored configuration entirely, disable the plugin via `/plugin` — that removes the `pluginConfigs.prequel` block from `settings.json` and the keychain entry.
