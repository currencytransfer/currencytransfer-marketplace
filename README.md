# CurrencyTransfer for Claude

Connect Claude to your [CurrencyTransfer](https://www.currencytransfer.com) account through the hosted CurrencyTransfer MCP server, and add skills for common FX workflows.

## What's in this repo

| Plugin | Surface | Description |
| --- | --- | --- |
| [currencytransfer](./plugins/currencytransfer) | MCP | Bundles the CurrencyTransfer MCP connection and workflow skills for indicative FX quotes, cash position and trade tracking. |

### Skills

| Skill | Category | Description |
| --- | --- | --- |
| fx-quote | Quotes | Check the pair and delivery date, then get an indicative quote (valid for 15 seconds) and refresh it on request. |
| book-trade | Trades | Book a trade after you approve it. Re-quotes on approval, books only at the approved price or better, then shows settlement instructions. |
| create-payment | Payments | Pay a beneficiary from a trade after you approve it. Checks remaining amount, purpose and validation first. Payments can't be edited. It can remove one within 5 minutes of creation, so you can re-create it. |
| add-beneficiary | Beneficiaries | Add a payee after you approve it: country-specific fields, validation and bank name check. Or email the payee to fill in their own details. |
| cash-position | Balances | Balances across currencies and brokers, funds in flight, and trades waiting on funding, summarised in plain language. |
| trade-status | Trades | Look up trades and their payments: status, settlement deadline, outstanding funding, and failed or pending payments. |
| ct-best-practices | Fallback | Connection checks, environments, the approval rule, the tool map, error handling, and anything a workflow skill above doesn't cover. |

**You approve every money movement.** Before booking a trade, creating or changing a payment, adding a beneficiary, or making any other change to your account, Claude shows an exact summary and acts only after you reply **yes**. If anything changes, including a worse price, it asks again.

Installed skills are prefixed with the plugin name, e.g. `/currencytransfer:fx-quote`. Claude also loads them on its own when your request matches.

## Quick start (Claude Code)

Add the marketplace, then install the plugin:

```sh
claude plugin marketplace add https://github.com/currencytransfer/currencytransfer-marketplace
claude plugin install currencytransfer@currencytransfer-marketplace
```

Then sign in:

1. Start Claude Code and run `/mcp`.
2. Select **currencytransfer** → **Authenticate**.
3. Your browser opens CurrencyTransfer. Sign in (with 2FA) and choose the account to connect.
4. Back in Claude Code, try: *"What's my cash position?"* or `/currencytransfer:fx-quote`.

> The plugin connects to the **beta** environment by default (`https://mcp-beta.currencytransfer.com`). See [Environments](#environments) to use stage or production.

More detail: [docs/connect-claude-code.md](docs/connect-claude-code.md).

## Other ways to connect

You only need the server URL. Sign-in is OAuth, so there are no API keys to copy.

| Client | How | Guide |
| --- | --- | --- |
| Claude Code, without the plugin | `claude mcp add --transport http currencytransfer https://mcp-beta.currencytransfer.com` | [connect-claude-code.md](docs/connect-claude-code.md) |
| claude.ai / Claude Desktop | Settings → Connectors → **Add custom connector** → paste the URL | [connect-claude-ai.md](docs/connect-claude-ai.md) |

Without the plugin you get the tools but not the skills. To use the skills too, see [Using the skills without the plugin](docs/connect-claude-ai.md#using-the-skills-without-the-plugin).

## Environments

Each environment has its own server, its own CurrencyTransfer sign-in and its own accounts. **The URL decides the environment.** It can't be switched after you connect.

| Environment | MCP server URL | Sign-in at |
| --- | --- | --- |
| Beta (plugin default) | `https://mcp-beta.currencytransfer.com` | `https://beta.currencytransfer.com` |
| Stage | `https://mcp-stage.currencytransfer.com` | `https://stage.currencytransfer.com` |
| Production | `https://mcp.currencytransfer.com` | `https://app.currencytransfer.com` |

How to switch or connect several at once: [docs/environments.md](docs/environments.md).

## What the server can and can't do

- **Always available:** read your profile, currency pairs, holidays, balances, deposit details, trades, payments, beneficiaries and rate alerts. Request an indicative quote and refresh it. A quote commits you to nothing.
- **When the server has them enabled:** book trades, create payments (removable within 5 minutes, never editable), add and edit beneficiaries, top up balances, and manage rate alerts. Each needs your approval in the chat first.

If the connected server doesn't offer an action, Claude says so. You can always do it in the CurrencyTransfer web app.

Having trouble? See [docs/troubleshooting.md](docs/troubleshooting.md).

## Important notice

> **Beta.** This plugin is provided "as is" for use with the CurrencyTransfer MCP server. By using it you acknowledge that:
>
> - Actions taken through your CurrencyTransfer sign-in, whether you start them yourself or an AI agent starts them on your behalf, are governed by the CurrencyTransfer terms you accepted when you opened your account.
> - Anyone holding your session's access token can act on the connected account. Connect only clients you trust, and sign out from `/mcp` when you're done on a shared machine.
> - This plugin does **not** give financial, legal or tax advice. AI output is probabilistic and should be checked by a qualified person before you act on it.
> - The skills can book trades, create payments and add beneficiaries, **but only after you explicitly approve each one in the chat**. Check every approval summary carefully, especially on production: booked trades are binding and sent payments may not be recoverable.
> - Your client's own tool-permission settings are a second safeguard. Don't set the CurrencyTransfer write tools to "always allow".

## License

Apache-2.0. See [LICENSE](LICENSE).
