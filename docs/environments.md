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

Then authenticate each one in `/mcp`. The skills work with any of them. Claude uses whichever CurrencyTransfer server you name in your request ("on production, …"). If you don't name one and several are connected, the `ct-best-practices` skill tells Claude to ask you which.

To use **only** production, disable the plugin's beta server in `/mcp` (or with `claude mcp` settings) and keep the skills.

## claude.ai / Claude Desktop

Add one custom connector per environment and name them clearly, e.g. `CurrencyTransfer (Prod)` and `CurrencyTransfer (Beta)`. See [connect-claude-ai.md](connect-claude-ai.md).
