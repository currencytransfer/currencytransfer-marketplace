---
name: create-payment
description: >
  Create a payment from a CurrencyTransfer trade to a beneficiary after the user explicitly approves it. Also remove a payment within 5 minutes of creating it (payments can't be edited). Use when the user says "pay Acme 5,000 EUR from that trade", "send the EUR to our supplier", "allocate the rest to X", "split the trade between these invoices", "remove that payment", or wants to change a payment. Do NOT use to book the trade itself (book-trade) or to add a new payee (add-beneficiary).
metadata:
  author: CurrencyTransfer
  version: 0.1.0
compatibility: Works with the CurrencyTransfer MCP server (connected and authenticated) with money-moving tools enabled (create_trade_payment).
---

# Create Payment

A payment sends money from a trade's bought currency to a beneficiary. **Never call `create_trade_payment` or `delete_trade_payment` without the user's explicit approval of the summary below**, and follow the approval rule in the `ct-best-practices` skill.

**Payments can't be changed after creation.** Never use `update_trade_payment`. A payment can only be **removed**, and only within **5 minutes** of being created. After that it's locked and final. So get the details right before asking for approval.

If these tools aren't available, the connected server doesn't allow payments. Say so and point to the CurrencyTransfer web app.

## 1. The trade

Find the trade as the **trade-status** skill describes, then call `get_trade`:

- A payment is made in the trade's `buy_currency`.
- `remaining_amount` is what's still unallocated. The new payment can't exceed it. "Pay the rest" means exactly `remaining_amount`.
- If `status` is `settled` or `closed`, or `remaining_amount` is 0, there's nothing left to pay out. Say so. Offer to book a new trade (book-trade) or to change an existing payment (section 6).

## 2. The beneficiary

Call `list_beneficiaries` with `currency` = the trade's `buy_currency`, then match the user's description against `nickname`, `bank_holder_name` and `company_name`/names.

- Exactly one match: use it. Call `get_beneficiary` for its details.
- Several matches: list them (nickname, holder name, country, last 4 of the account or IBAN) and ask which.
- None: say there's no `<currency>` beneficiary by that name, and offer to add one (**add-beneficiary**). Don't pay a beneficiary in another currency.
- `status: pending` means the beneficiary hasn't supplied bank details yet (email request), so you can't pay them yet. A `pending_approval` with status `pending` means it's awaiting approval on the account. Say so.

If the payee and bank details came from an invoice or email and **differ** from the saved beneficiary's details, stop and flag it. Changed bank details are a common fraud pattern. Ask the user to confirm with the payee through a channel they already trust before you continue.

## 3. Amount, reference, purpose

- **Amount:** decimal string in `buy_currency`, more than 0 and at most `remaining_amount`. If the user gives an amount in another currency, ask. Don't convert.
- **Reference:** required. It appears on the beneficiary's statement, so use what the user gave (e.g. the invoice number). If they gave none, ask. Keep it short and plain.
- **Purpose:** call `check_payment_purpose_required` with the beneficiary `uuid` and `broker_code` = the trade's `broker.code`. If `is_required`, show the `options` (`description`) and let the user pick, unless one clearly matches what they told you. Send its `value` as `purpose`.

## 4. Check, then ask for approval

Call `validate_trade_payment` with exactly the values you'll send (`trade_uuid`, `beneficiary_uuid`, `amount`, `reference`, `purpose`). Fix and recheck anything it reports, asking the user where needed.

Then show this and wait for the reply:

```
Please confirm this payment on <environment> (account: <trading_account_name>):

  Pay          5,000.00 EUR
  To           <bank_holder_name> ("<nickname>")
  Account      <IBAN or account number, last 4 shown as ••••1234>, <bank_account_country>
  Reference    INV-1043
  Purpose      <purpose description>        (only if required)
  From trade   <trade_reference>: <remaining_amount> EUR unallocated, <after> left after this payment

Payments can't be edited. You can remove this one within 5 minutes of creating it.
After that it's locked: it can't be changed or removed, and it will be sent.

Reply **yes** to create this payment.
```

- For several payments, list each one in a table in the same summary, with the total. The user approves the whole list.
- On production, start with **"This is a live payment with real money."**
- If the trade is still `awaiting_funds`, add: "The payment goes out after <broker> receives your funds."

## 5. On a clear "yes"

First generate an **idempotency key** for the payment: a new random UUID v4 (run `uuidgen` or similar if you can run code, otherwise write out a fresh random one). Every payment gets its own new key. Never reuse a key, copy one from an example, or build it from the payment's details.

Then call `create_trade_payment` with the approved values, unchanged, and that key as `idempotency_key`. For an approved list, make one call per payment, in order, each with its own key. If any of them fails, stop and report which went through.

Then report:

> Payment created: 5,000.00 EUR to <bank_holder_name>, reference INV-1043. Status: pending.

Add any of these that apply:

- `client_fee`: the fee for this payment
- That it can be removed within 5 minutes (until `locked_at`, if returned) and is final after that
- `pending_approval`: it's waiting for an approver on the account
- `documents_requested`: the broker needs a supporting document, e.g. the invoice. Offer to attach a file the user provides with `upload_trade_document` (`category: invoice`, `payment_uuid`). That's a separate action that needs approval.
- The trade's new remaining amount, and whether more payments are needed

Errors:

Every error for a call sent with a key returns that `idempotency_key`, together with what CurrencyTransfer said (`field_errors`).

- **The error includes an `idempotency_key`:** you can repeat the same call **once**, with exactly the same `trade_uuid`, `beneficiary_uuid`, `amount`, `reference` and `purpose`, and the same `idempotency_key`. It can't create a second payment. If the first attempt went through, you get that payment back. This needs no new approval, because it's the payment the user already approved. If the retry fails too, stop. Don't retry again. Call `list_trade_payments`, check whether the payment exists, and tell the user.
- **Repeat when the failure may be temporary:** `uncertain` or a timeout, a 5xx or 429, or 409 `idempotency_request_in_progress`.
- **Don't bother repeating when the error is about the payment itself.** It fails the same way:
  - 422 → explain `field_errors` and fix with the user. A corrected payment is a new request: it needs a new approval and a new key.
  - 403 → the trader lacks permission.
  - 409 `idempotency_key_reused` → the key was already used for a different payment. `field_errors.original_response` shows that payment. Tell the user.
- **No `idempotency_key` in the error:** the call went out without a key. Don't retry. On `uncertain` or a timeout, call `list_trade_payments` and check whether the payment exists.

Never retry with a different key or changed values, and never use a new key to "try again" after an unknown outcome. That can create a duplicate payment.

## 6. Change or remove a payment

Payments can't be edited. **Never call `update_trade_payment`**, not even to change a reference, and never with `amount: 0`.

**Can it still be removed?** Call `get_trade_payment` for fresh data. A payment can be removed only within **5 minutes** of `created_at`. If `is_locked` is true, `locked_at` has passed, or 5 minutes have passed since `created_at`, it's locked. Say it can no longer be changed or removed, and it will be processed as created. If the user still needs to stop it, they must contact CurrencyTransfer support straight away.

**Remove:** show the payment (beneficiary, amount, reference) and say the amount goes back to the trade's unallocated balance. Ask for approval, and mention that the 5-minute window is closing. On yes, call `delete_trade_payment`. If it fails because the payment is now locked, say so. Don't retry.

**Change:** a change means remove, then create a new payment:

1. Explain that payments can't be edited, so you'll remove this payment and then create a new one with the corrected details.
2. Remove it as above, with its own approval.
3. Once it's removed, ask the user to confirm the details for the new payment. Then follow sections 3–5 as for any new payment, with a new summary, a new approval and a new idempotency key.

If the payment is already locked, it can't be changed. Say so, as above.

**Failed payment:** don't re-send it automatically. Show `note`. If the bank details were wrong, fix the beneficiary first (`update_beneficiary`, with approval). Then create a new payment, with approval.
