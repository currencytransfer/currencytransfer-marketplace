# currencytransfer plugin

Connects Claude to the hosted CurrencyTransfer MCP server and adds skills for common FX workflows.

- **MCP server:** `currencytransfer` → `https://mcp-beta.currencytransfer.com` (beta). Sign in with your CurrencyTransfer account through `/mcp` → **Authenticate**. To use stage or production, see [../../docs/environments.md](../../docs/environments.md).
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

Claude loads the right skill on its own when your request matches. The slash commands are there for when you want a specific one.

## Install

```sh
claude plugin marketplace add https://github.com/currencytransfer/currencytransfer-marketplace
claude plugin install currencytransfer@currencytransfer-marketplace
```

See the [repository README](../../README.md) for other clients.
