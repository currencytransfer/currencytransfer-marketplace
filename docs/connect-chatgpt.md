# Connect ChatGPT

ChatGPT connects to remote MCP servers as **apps**. A custom server like CurrencyTransfer's is added in **developer mode**. For Codex (CLI, IDE extension, or Codex in the ChatGPT desktop app), see [connect-codex.md](connect-codex.md).

> ChatGPT's menus change often. The steps below follow OpenAI's developer mode guide as of October 2026. If a label has moved, see [OpenAI's guide](https://developers.openai.com/api/docs/guides/developer-mode).

## Requirements

- A ChatGPT **Pro, Plus, Business, Enterprise or Education** account, on the web.
- On Business, Enterprise and Education workspaces, an admin may need to allow developer mode first.

## Add CurrencyTransfer as an app

1. In ChatGPT, open **Settings → Security and login** and turn on **Developer mode**.
2. Open the **Plugins** page and select the **plus** button to create a developer-mode app.
3. Fill it in:

   | Field | Value |
   | --- | --- |
   | Name | `CurrencyTransfer` (add the environment, e.g. `CurrencyTransfer (Beta)`, if you'll connect more than one) |
   | MCP server URL | The URL for your environment, from the table below |
   | Authentication | **OAuth** |

   | Environment | URL |
   | --- | --- |
   | Beta | `https://mcp-beta.currencytransfer.com` |
   | Stage | `https://mcp-stage.currencytransfer.com` |
   | Production | `https://mcp.currencytransfer.com` |

4. Leave the OAuth client ID and secret empty. ChatGPT registers itself with CurrencyTransfer automatically.
5. Create the app. When ChatGPT sends you to CurrencyTransfer, sign in, complete 2FA and **choose the account** to connect.

Use the URL exactly as shown: the bare domain, with no `/mcp` at the end.

## Use it in a chat

1. In the message box, open the **+** menu and select **Developer mode**.
2. Switch on **CurrencyTransfer**.
3. Ask, for example:
   - *What are my balances on CurrencyTransfer?*
   - *Get me an indicative quote to sell 10,000 GBP for EUR on CurrencyTransfer.*
   - *What's the status of my last CurrencyTransfer trade?*

Naming CurrencyTransfer in the request helps ChatGPT pick this app instead of a built-in tool.

## Confirmations

ChatGPT asks you to confirm **write actions** before it runs them. For CurrencyTransfer those are booking a trade, creating or removing a payment, and adding or changing a beneficiary, where the server has them enabled.

ChatGPT lets you remember your choice for a tool for the rest of a conversation. **Don't remember "approve" for tools that book trades, create payments or change beneficiaries.** Confirm each one, every time.

With the skills installed, ChatGPT also shows you an exact summary and waits for your **yes** before any of these actions. Without the skills, ChatGPT's own confirmation is the only check, so read it carefully.

## Skills

The skills in this repo reach ChatGPT through the **CurrencyTransfer plugin**, which bundles the server and the skills:

- **ChatGPT desktop app:** add the marketplace and install the plugin as described in [connect-codex.md](connect-codex.md#option-a-install-the-plugin-recommended). It then appears in the app's **Plugins** tab, and you sign in when prompted.
- **ChatGPT on the web and mobile:** skills come only from installed plugins. Open the **Plugins** tab, find **CurrencyTransfer** and install it, if it's available to your account or workspace. If it isn't listed, use the developer-mode app above. You get all the tools, but not the skills.

Once the plugin is installed, type `@` to pick CurrencyTransfer or one of its skills (e.g. `@fx-quote`), or just describe what you want.

## Update, switch account, remove

Open the CurrencyTransfer app from ChatGPT's settings. From its details page you can:

- **Refresh** it, to pick up new or changed tools after a server update.
- **Disconnect and reconnect**, to sign in with a different CurrencyTransfer account.
- **Delete** it.

To use another environment, add a second app with that environment's URL and a clear name, e.g. `CurrencyTransfer (Prod)`. Each environment has its own sign-in.
