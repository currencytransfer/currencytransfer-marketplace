# Connect claude.ai and Claude Desktop

claude.ai and Claude Desktop connect to remote MCP servers as **custom connectors**. A connector you add on claude.ai is also available in Claude Desktop and Claude mobile when you're signed in to the same account.

## Add the connector

1. Open **Settings → Connectors** on claude.ai (or in Claude Desktop).
2. Click **Add custom connector**.
3. Name: `CurrencyTransfer` (add the environment, e.g. `CurrencyTransfer (Beta)`, if you'll connect more than one).
4. URL: the server for your environment:

   | Environment | URL |
   | --- | --- |
   | Beta | `https://mcp-beta.currencytransfer.com` |
   | Stage | `https://mcp-stage.currencytransfer.com` |
   | Production | `https://mcp.currencytransfer.com` |

5. Leave **Advanced settings** (OAuth client ID and secret) empty. The client registers itself with CurrencyTransfer automatically.
6. Click **Add**, then **Connect**. Sign in to CurrencyTransfer, complete 2FA and choose the account.

On Team and Enterprise plans, an owner may need to add the connector for the organization first. Members then connect it from **Settings → Connectors**.

## Use it in a chat

Open the **+** (or tools) menu in the message box and make sure **CurrencyTransfer** is switched on. Then ask, for example:

- *What are my balances on CurrencyTransfer?*
- *Get me an indicative quote to sell 10,000 GBP for EUR.*
- *What's the status of my last trade?*

Claude asks for permission the first time it uses each tool. You can allow read tools always, but keep tools that book trades, create payments or change beneficiaries on **allow once**, as a second check alongside the approval summary.

## Using the skills without the plugin

The plugin format is for Claude Code. On claude.ai and Claude Desktop you can upload the same skills by hand:

1. Zip a skill folder, e.g. `plugins/currencytransfer/skills/fx-quote/`. The zip must contain the folder with its `SKILL.md`.
2. In **Settings → Capabilities → Skills**, click **Upload skill** and pick the zip.
3. Repeat for every other folder in `plugins/currencytransfer/skills/`. Upload `ct-best-practices` too: it holds the approval rule the booking, payment and beneficiary skills rely on.

```sh
cd plugins/currencytransfer/skills
for s in */; do (zip -qr "../../../${s%/}.zip" "$s"); done
```

Skills need code execution to be enabled for your account. The skills assume the CurrencyTransfer connector is connected, as above.
