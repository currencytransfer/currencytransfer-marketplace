# Troubleshooting

## The server shows "needs authentication" or "failed"

The first connection always needs a sign-in.

- **Claude Code:** run `/mcp` → **currencytransfer** → **Authenticate**. If it keeps failing, choose **Clear authentication** and try again.
- **Codex:** run `codex mcp login currencytransfer`. If it keeps failing, run `codex mcp logout currencytransfer` first. `codex mcp list` shows whether you're logged in.
- **ChatGPT:** open the CurrencyTransfer app from settings and reconnect it.
- Check the URL is the bare domain from [environments.md](environments.md): `https://mcp-beta.currencytransfer.com`, not `…/mcp` or `http://…`.
- Check the server is up: `curl -s -o /dev/null -w '%{http_code}\n' -X POST https://mcp-beta.currencytransfer.com` should print `401`. A 401 here is correct, because it's the server asking for a sign-in.

## Sign-in page shows `invalid_target`

CurrencyTransfer doesn't recognise the server URL as a resource it issues tokens for. This usually means a typo or extra path in the URL. Use the exact URL from [environments.md](environments.md). If it still happens, contact CurrencyTransfer support with the URL you used.

## Sign-in works but every call fails with 401

The token belongs to a different environment than the server. For example, you signed in on beta and are connected to production, or a client reused an old token. Sign out of that server and sign in again (Claude Code: **Clear authentication**. Codex: `codex mcp logout currencytransfer`, then `codex mcp login currencytransfer`).

## Calls start failing after a while

Access tokens expire after about 2 hours and are refreshed automatically. If refresh fails (for example, after a long time away, or a password or 2FA change), clear authentication and sign in again.

## "Session not found" (404)

The server restarted or your session expired. Your client normally reconnects on its own. If it doesn't, restart Claude Code or Codex, or reconnect the connector or app.

## A tool I expected is missing

Reads and quotes are always available. Booking trades, payments, beneficiary changes, top-ups and rate-alert changes appear only when the server operator has enabled them for that environment. If they're missing, use the CurrencyTransfer web app.

## The assistant keeps asking me to confirm

That's by design. Every booking, payment, beneficiary or other account change needs a fresh **yes** to an exact summary. The assistant asks again if anything changed, for example when the price moved against you between the quote and your reply.

## Balances come back empty

`list_balances` returns an empty list if your account has no balance currency set up, or none of your brokers supports balance information. `get_user` shows whether balances are activated for your account (`activated_balances`).

## A quote fails with a validation error

- The pair may not be tradable for your account. Ask for your currency pairs (`get_currency_pairs`).
- The delivery date may be before the earliest delivery date for the pair, or on a holiday for one of the currencies (`get_currency_holidays`).
- The `reason` field allows only letters, digits, spaces and `- / ? : ( ) . , ' +`.
- Your account may not be verified for quoting yet (`get_user` → `is_verified`).

## Error categories the server returns

| Category | Meaning | What to do |
| --- | --- | --- |
| `invalid_input` | Rejected by the MCP server before reaching CurrencyTransfer (bad UUID, date format, or a disabled tool) | Fix the input. If the tool is disabled, it isn't available on this server. |
| `provider_error` | CurrencyTransfer returned an error (4xx/5xx) | Read the status and field errors. 401 means sign in again, 404 means wrong ID or no access, 422 means invalid values. |
| `timeout` / `transport_error` | The server couldn't reach CurrencyTransfer | Reads are retried automatically. Try again later. |
| `uncertain` | A write was sent but the outcome is unknown | Don't repeat it blindly. Check the current state first. |
