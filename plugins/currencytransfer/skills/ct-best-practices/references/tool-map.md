# CurrencyTransfer MCP tool map

**Hosted** = exposed by the hosted server by default. **Approval** tools change the account. They appear only if the server operator has enabled all tools, and you call them only after the user approves (see the approval rule in SKILL.md).

| Domain | Tool | Kind | Hosted | Purpose |
| --- | --- | --- | --- | --- |
| User | `get_user` | read | ✓ | Signed-in user, trading account, verification, activated features, relationship manager |
| Currencies | `get_currency_pairs` | read | ✓ | Tradable pairs (sell → buys) and earliest delivery date per pair |
| | `get_supported_currencies` | read | ✓ | All supported currencies |
| | `get_currency_holidays` | read | ✓ | Holiday dates per currency (affect delivery and settlement) |
| | `get_same_currency_transfers` | read | ✓ | Supported same-currency transfer options |
| | `get_bank_account_fields` | read | ✓ | Required bank fields per currency/country (`currency`, `country`) |
| | `get_rsi_currency_pairs` | read | ✓ | Pairs for RSI (MOEX) alerts |
| Countries | `get_supported_countries` | read | ✓ | Supported countries |
| | `get_address_requirements` | read | ✓ | Address fields for all countries |
| | `get_address_requirements_by_country` | read | ✓ | Address fields for one country (`country_code`) |
| Quotes | `create_quote` | quote | ✓ | Indicative broker quotations, valid for 15 s |
| | `refresh_quote` | quote | ✓ | New quotations for an existing quote (`uuid`) |
| Trades | `list_trades` | read | ✓ | Filter by `status`, `buy_currency`, `sell_currency`, `created_at_from`/`to`. Set `with_beneficiaries` for payees. |
| | `get_trade` | read | ✓ | Full trade incl. settlement accounts, documents, confirmation PDF |
| | `get_trade_metadata` | read | ✓ | Extra metadata for a trade |
| | `list_trade_payments` | read | ✓ | Payments in a trade (`trade_uuid`) |
| | `get_trade_payment` | read | ✓ | One payment (`trade_uuid`, `uuid`) |
| | `validate_trade_payment` | read | ✓ | Pre-flight check for a payment, with the same fields as `create_trade_payment` |
| | `create_trade`, `upload_trade_document`, `delete_trade_document` | write | if enabled | **Approval** |
| | `create_trade_payment` | write | if enabled | **Approval.** Takes `idempotency_key`: always send a new unique UUID with each new payment. If an error returns the key, repeat the same call once with the same arguments and key. |
| | `delete_trade_payment` | write | if enabled | **Approval.** Only within 5 minutes of the payment's creation. After that it's locked. |
| | `update_trade_payment` | write | if enabled | **Don't use.** Payments can't be changed after creation. Remove the payment and create a new one. |
| Beneficiaries | `list_beneficiaries` | read | ✓ | Paginated. Filter by `currency`. |
| | `get_beneficiary` | read | ✓ | One beneficiary |
| | `list_beneficiary_payments` | read | ✓ | Payments made to a beneficiary |
| | `list_beneficiary_email_requests` | read | ✓ | Requests sent to a beneficiary for their bank details |
| | `check_payment_purpose_required` | read | ✓ | Whether paying this beneficiary needs a purpose (`uuid`, `broker_code`) |
| | `validate_beneficiary`, `verify_beneficiary` | read | ✓ | Pre-flight checks before creating or updating a beneficiary. `verify` runs a payee-name check (e.g. Confirmation of Payee): `full_match`, `close_match`, `no_match` or `unable_to_verify`. |
| | `create_beneficiary`, `update_beneficiary`, `delete_beneficiary`, `*_beneficiary_email_request` | write | if enabled | **Approval** |
| Balances | `list_balances` | read | ✓ | Balances per currency and broker, `cached_at` |
| | `get_balance_deposit_details` | read | ✓ | Bank details for funding a balance (`broker_code`, `currency`) |
| | `initiate_balance_topup` | write | if enabled | **Approval** |
| Rate alerts | `list_rate_alerts`, `get_rate_alert` | read | ✓ | Target-rate alerts and market orders. Active alerts with `book=true` auto-book when hit. |
| | `validate_rate_alert` | read | ✓ | Pre-flight check for an alert |
| | `create_rate_alert`, `update_rate_alert`, `delete_rate_alert` | write | if enabled | **Approval** |

RSI alert tools (`*_rsi_alert`) are disabled on the hosted server.
