# currencytransfer plugin

Connects Claude Code or OpenAI Codex to the hosted CurrencyTransfer MCP server and adds skills for common FX workflows. One plugin and one set of skills serve both.

- **MCP server:** `currencytransfer` → the URL in the plugin option `mcp_url`, by default `https://mcp-beta.currencytransfer.com` (beta). Set it with `--config mcp_url=…` on install, or with `/plugin configure currencytransfer@currencytransfer-marketplace` (see [../../docs/environments.md](../../docs/environments.md)). Sign in through `/mcp` → **Authenticate**. In Codex the server is fixed to beta unless you replace it with `codex mcp add currencytransfer --url …` (see [../../docs/connect-codex.md](../../docs/connect-codex.md#switch-environment)), and you sign in with `codex mcp login currencytransfer`.
- **Surface:** reads and indicative quotes, plus booking, payments and beneficiaries where the server enables them. **Every money movement or account change needs your explicit approval in the chat.**

## Skills

| Skill | Invoke | Use for |
| --- | --- | --- |
| [fx-quote](skills/fx-quote/SKILL.md) | `/currencytransfer:fx-quote` | Indicative quotes, broker comparison, refreshing an expired quote |
| [book-trade](skills/book-trade/SKILL.md) | `/currencytransfer:book-trade` | Book a trade after approval, then settlement instructions |
| [create-payment](skills/create-payment/SKILL.md) | `/currencytransfer:create-payment` | Pay a beneficiary from a trade after approval. Remove a payment within 5 minutes of creation (payments can't be edited). |
| [add-beneficiary](skills/add-beneficiary/SKILL.md) | `/currencytransfer:add-beneficiary` | Add a payee after validation, a name check and approval, or email the payee a form |
| [cash-position](skills/cash-position/SKILL.md) | `/currencytransfer:cash-position` | Balances, funding obligations, unallocated currency, deposit details |
| [trade-status](skills/trade-status/SKILL.md) | `/currencytransfer:trade-status` | A trade's status, settlement instructions, payment outcomes |
| [ct-best-practices](skills/ct-best-practices/SKILL.md) | `/currencytransfer:ct-best-practices` | Connection checks, the approval rule, other tasks (beneficiaries, countries, holidays, alerts), errors |

The assistant loads the right skill on its own when your request matches. The commands above are Claude Code's. In Codex, use `$fx-quote`, `$book-trade` and so on.

## Install

```sh
claude plugin marketplace add https://github.com/currencytransfer/currencytransfer-marketplace
claude plugin install currencytransfer@currencytransfer-marketplace
```

OpenAI Codex:

```sh
codex plugin marketplace add currencytransfer/currencytransfer-marketplace
codex plugin add currencytransfer@currencytransfer-marketplace
codex mcp login currencytransfer
```

## Files

| File | Used by |
| --- | --- |
| `.claude-plugin/plugin.json`, `.mcp.json` | Claude Code. The server URL comes from the `mcp_url` plugin option. |
| `.codex-plugin/plugin.json`, `.codex-mcp.json` | OpenAI Codex. The server URL is written in the file (beta). |
| `skills/` | Both |

Keep `version` the same in both `plugin.json` files.

See the [repository README](../../README.md) for other clients.
