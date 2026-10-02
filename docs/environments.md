# Environments

CurrencyTransfer runs three environments. Each has its own MCP server, its own sign-in and its own accounts.

| Environment | MCP server URL | Authorization server (sign-in) | Use it for |
| --- | --- | --- | --- |
| Beta | `https://mcp-beta.currencytransfer.com` | `https://beta.currencytransfer.com` | Trying things out. **Plugin default.** |
| Stage | `https://mcp-stage.currencytransfer.com` | `https://stage.currencytransfer.com` | Pre-release testing |
| Production | `https://mcp.currencytransfer.com` | `https://app.currencytransfer.com` | Your live account |

Rules:

- **The URL decides the environment.** A server serves exactly one environment, and nothing a client sends can switch it.
- **Tokens don't cross environments.** A beta sign-in doesn't work against stage or production.
- Use the URLs exactly as shown: the bare domain, no `/mcp` and no trailing path. The server advertises `https://<domain>/` as its OAuth resource, and CurrencyTransfer only issues tokens for that.

## Point the plugin at another environment

The plugin's server is fixed to beta in `plugins/currencytransfer/.mcp.json`. To use production or stage as well, add a second server with its own name, so you never mix them up:

```sh
claude mcp add --transport http --scope user currencytransfer-prod  https://mcp.currencytransfer.com
claude mcp add --transport http --scope user currencytransfer-stage https://mcp-stage.currencytransfer.com
```

`claude plugin configure currencytransfer@currencytransfer-marketplace` (without `--values-stdin`) shows the current value. To go back to beta, set `mcp_url` to `https://mcp-beta.currencytransfer.com`.

Check which server you're on with `claude mcp list`. The plugin's server is listed as `plugin:currencytransfer:currencytransfer` with its URL.

## Use several environments at once

The plugin connects to one environment. To work with another at the same time, e.g. production through the plugin and stage for testing, add the other as a separate server with a clear name:

```sh
claude mcp add --transport http --scope user currencytransfer-stage https://mcp-stage.currencytransfer.com
```

Authenticate it in `/mcp`. The skills work with every CurrencyTransfer server. Name the environment in your request ("on stage, …"). If you don't and several are connected, the `ct-best-practices` skill tells Claude to ask you which.

## claude.ai / Claude Desktop

Add one custom connector per environment and name them clearly, e.g. `CurrencyTransfer (Prod)` and `CurrencyTransfer (Beta)`. See [connect-claude-ai.md](connect-claude-ai.md).
