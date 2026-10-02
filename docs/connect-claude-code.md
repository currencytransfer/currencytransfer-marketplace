# Connect Claude Code

There are two ways to connect. Option A installs the plugin, which brings the server **and** the skills. Option B adds only the server.

## Option A: install the plugin (recommended)

```sh
claude plugin marketplace add https://github.com/currencytransfer/currencytransfer-marketplace
claude plugin install currencytransfer@currencytransfer-marketplace
```

You can also run these from inside Claude Code as `/plugin marketplace add …` and `/plugin install …`.

The plugin registers an MCP server called `currencytransfer` pointing at **beta** (`https://mcp-beta.currencytransfer.com`). To use another environment, see [environments.md](environments.md).

### Sign in

1. Start (or restart) Claude Code and run `/mcp`.
2. Pick **currencytransfer**. It shows *needs authentication*.
3. Choose **Authenticate**. Your browser opens the CurrencyTransfer sign-in for that environment.
4. Sign in, complete 2FA, and **choose the account** Claude should act on.
5. Return to Claude Code. `/mcp` now shows the server as connected, with its tools.

Claude Code stores the tokens and refreshes them automatically. Access tokens last about 2 hours and refresh tokens about 2 months, so you'll sign in again only occasionally.

### Check it works

Ask Claude:

```
Who am I signed in as on CurrencyTransfer?
```

Claude should call `get_user` and reply with your name and trading account. Then try a skill:

```
/currencytransfer:fx-quote 10,000 GBP to EUR
/currencytransfer:cash-position
/currencytransfer:trade-status
```

### Update or remove

```sh
claude plugin marketplace update currencytransfer-marketplace
claude plugin update currencytransfer@currencytransfer-marketplace
claude plugin uninstall currencytransfer@currencytransfer-marketplace
```

## Option B: add only the MCP server

```sh
claude mcp add --transport http currencytransfer https://mcp-beta.currencytransfer.com
```

Add `--scope user` to make it available in every project, or `--scope project` to share it with your team through `.mcp.json`. Then sign in with `/mcp` → **Authenticate**, as above.

This gives Claude the tools but not the skills.

## Sign out or switch account

The connection is bound to the account you picked at sign-in. To use a different account, run `/mcp` → **currencytransfer** → **Clear authentication**, then **Authenticate** again and pick the other account.
