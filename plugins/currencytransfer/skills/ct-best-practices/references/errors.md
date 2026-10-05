# CurrencyTransfer MCP errors

Tool errors come back as an error result with a **category**. Provider errors also carry the HTTP status, the provider's `code`, and `field_codes` (per-field validation errors).

| Category | Cause | What to do |
| --- | --- | --- |
| `invalid_input` | Rejected by the MCP server before reaching CurrencyTransfer: malformed UUID or date, bad amount format, or a tool disabled on this server ("…is disabled in this deployment") | Fix the input and retry once. For a disabled tool, explain it isn't available here and point to the web app. |
| `provider_error` 401 | Access token expired or revoked, or from another environment | Ask the user to sign in again. See "Signing in, by client" below. |
| `provider_error` 403 | The signed-in user isn't allowed to do this (trader permissions, feature not activated, user not verified) | Call `get_user` and explain which flag is off (`is_verified`, `activated_balances`, …). |
| `provider_error` 404 | Unknown ID, or the record belongs to another account | Check that the UUID came from this conversation's results for this account. |
| `provider_error` 422 | Validation failed | Read `field_codes` and explain in plain words. For quotes: `market_closed`, `no_results`, or `over_limit` with `forward_disabled` on `delivery_date` (see the fx-quote skill). |
| `provider_error` 409 (booking) | `already_booked`, `market_shifted` or `invalid_delivery_date` | See the book-trade skill. Never book twice. |
| `provider_error` 429 / 5xx | Rate-limited or provider trouble | Reads were already retried. Tell the user to try again shortly. |
| `timeout`, `transport_error` | CurrencyTransfer unreachable | Same as above |
| `uncertain` | A write was sent but the result is unknown | **Never retry.** Check whether it happened (`list_trades`, `list_trade_payments`, `list_beneficiaries`) and tell the user. For a quote, just request a new one. |
| `malformed_response` | Unexpected response from the provider | Report it, then retry once at most |

## Signing in, by client

| Client | Sign in (or sign in again) |
| --- | --- |
| Claude Code | `/mcp` → the CurrencyTransfer server → **Authenticate**. If it's stuck on an old account, choose **Clear authentication** first. |
| claude.ai / Claude Desktop | **Settings → Connectors** → CurrencyTransfer → **Connect**, then switch it on in the chat's tools menu |
| Codex CLI | `codex mcp login currencytransfer` in a terminal (`codex mcp logout currencytransfer` first to switch account). `/mcp` in Codex shows the server's status. |
| ChatGPT (web, desktop) | Open the CurrencyTransfer plugin or app in ChatGPT's settings and connect or reconnect it |

## HTTP-level errors

Errors at the HTTP level, before any tool runs:

- **401 with `WWW-Authenticate: Bearer resource_metadata=…`** means the client isn't signed in. This is normal before the first sign-in.
- **404 "session not found"** means the server restarted or the session expired. The client reconnects.
- **`invalid_target` during sign-in** means the server URL isn't one CurrencyTransfer issues tokens for. Use the bare domain URL (`https://mcp-beta.currencytransfer.com`).
