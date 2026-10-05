# Connect OpenAI Codex

Codex connects to CurrencyTransfer in two ways. Option A installs the plugin, which brings the server **and** the skills. Option B adds only the server.

Codex CLI, the Codex IDE extension and Codex in the ChatGPT desktop app share one configuration (`~/.codex/config.toml`), so a server you add in one is available in the others. For ChatGPT on the web, see [connect-chatgpt.md](connect-chatgpt.md).

## Option A: install the plugin (recommended)

```sh
codex plugin marketplace add currencytransfer/currencytransfer-marketplace
codex plugin add currencytransfer@currencytransfer-marketplace
```

You can also browse and install it from the plugin browser: run `/plugins` inside Codex.

The plugin registers an MCP server called `currencytransfer` pointing at **beta** (`https://mcp-beta.currencytransfer.com`). To use stage or production, see [Switch environment](#switch-environment).

> The Codex IDE extension doesn't support plugins. It still gets the MCP server from the shared configuration. For the skills there, see [Using the skills without the plugin](#using-the-skills-without-the-plugin).

### Sign in

```sh
codex mcp login currencytransfer
```

Your browser opens the CurrencyTransfer sign-in for that environment. Sign in, complete 2FA, and **choose the account** Codex should act on. Codex stores the tokens and refreshes them automatically.

Check the result:

```sh
codex mcp list
```

`currencytransfer` should show its URL and that you're logged in. Inside Codex, `/mcp` shows the active servers and their tools.

### Check it works

Ask Codex:

```
Who am I signed in as on CurrencyTransfer?
```

Codex should call `get_user` and reply with your name and trading account. Then try a skill. Type `$` and the skill name, or just describe what you want:

```
$fx-quote 10,000 GBP to EUR
$cash-position
$trade-status
```

`/skills` lists the installed skills.

### Update or remove

```sh
codex plugin marketplace upgrade currencytransfer-marketplace
codex plugin remove currencytransfer@currencytransfer-marketplace
codex plugin marketplace remove currencytransfer-marketplace
```

## Option B: add only the MCP server

```sh
codex mcp add currencytransfer --url https://mcp-beta.currencytransfer.com
```

Codex detects that the server uses OAuth and starts the sign-in in your browser. If it doesn't, run `codex mcp login currencytransfer`.

Or add it to `~/.codex/config.toml` yourself, then run `codex mcp login currencytransfer`:

```toml
[mcp_servers.currencytransfer]
url = "https://mcp-beta.currencytransfer.com"
```

In the ChatGPT desktop app you can do the same from **Settings → MCP servers → Add server**: choose **Streamable HTTP**, paste the URL, restart, then choose **Authenticate**.

This gives Codex the tools but not the skills.

## Using the skills without the plugin

Skills are plain folders. Copy them into your personal skills folder, where Codex CLI, the IDE extension and the desktop app all pick them up:

```sh
git clone https://github.com/currencytransfer/currencytransfer-marketplace.git
mkdir -p ~/.agents/skills
cp -R currencytransfer-marketplace/plugins/currencytransfer/skills/* ~/.agents/skills/
```

Copy all of them. `ct-best-practices` holds the approval rule that the booking, payment and beneficiary skills rely on. To share the skills with a team instead, put them in `.agents/skills/` in your repository.

## Switch environment

Codex plugins have no settings, so the plugin's server is fixed to beta. To use another environment, add your own server **with the same name**. It replaces the plugin's:

```sh
codex mcp add currencytransfer --url https://mcp.currencytransfer.com        # production
codex mcp add currencytransfer --url https://mcp-stage.currencytransfer.com  # stage
```

Sign in again when the browser opens. Sign-ins don't carry over between environments. Run `codex mcp list` to confirm the URL. To go back to the plugin's beta server:

```sh
codex mcp remove currencytransfer
```

To keep several environments side by side, give the others their own names:

```sh
codex mcp add currencytransfer-prod --url https://mcp.currencytransfer.com
```

Then name the environment in your request ("on production, …"). More on environments: [environments.md](environments.md).

## Keep approval prompts on

The skills always ask you to approve a trade, payment or beneficiary in the chat first. Keep Codex's own tool approval on as a second check, so Codex asks before each CurrencyTransfer tool call. In `~/.codex/config.toml`:

```toml
# the plugin's bundled server
[plugins."currencytransfer@currencytransfer-marketplace".mcp_servers.currencytransfer]
default_tools_approval_mode = "prompt"

# a server you added yourself with `codex mcp add`
[mcp_servers.currencytransfer]
url = "https://mcp.currencytransfer.com"
default_tools_approval_mode = "prompt"
```

Don't set this to `auto` for a CurrencyTransfer server that can book trades or send payments.

## Sign out or switch account

```sh
codex mcp logout currencytransfer
codex mcp login currencytransfer
```

The connection is bound to the account you picked at sign-in. Sign in again to pick a different one.
