---
name: book-trade
description: >
  Book an FX trade on CurrencyTransfer after the user explicitly approves it: quote, show an approval summary, re-quote on approval, book at the approved price or better, then show settlement instructions. Use when the user says "book it", "buy 10k EUR with GBP", "convert", "execute the trade", "go ahead with that quote", or asks to lock in a rate now. Do NOT use for price checks only (fx-quote), paying a beneficiary from an existing trade (create-payment), or checking an existing trade (trade-status).
metadata:
  author: CurrencyTransfer
  version: 0.1.0
compatibility: Works with the CurrencyTransfer MCP server (connected and authenticated) with money-moving tools enabled (create_trade).
---

# Book Trade

Booking a trade is a **binding commitment**. The user must then send the sell amount to the broker by the settlement deadline. **Never call `create_trade` without the user's explicit approval of the summary below**, and follow the approval rule in the `ct-best-practices` skill.

If `create_trade` isn't available, the connected server doesn't allow booking. Say so, offer an indicative quote (fx-quote) instead, and point to the CurrencyTransfer web app.

## 1. Collect the details

Work out sell currency, buy currency, side, amount, delivery date and reason exactly as the **fx-quote** skill describes: check the pair with `get_currency_pairs`, validate the date against `get_currency_holidays`, and clean `reason`.

**Always ask about the delivery date** before quoting: the **earliest delivery date** (name it) or a **future date**. Never pick one for the user. Skip the question only if they already chose in this request, or chose for the quote they're now booking. Ask it together with any other missing details, in one message. For a booking, `reason` should be the real purpose (e.g. "Payment for invoice INV-1043"). If the user didn't give one, ask; don't use a placeholder.

Call `get_user`:

- `is_verified` must be true, or the account can't book yet.
- Note `trading_account_name` for the summary.

If the user wants a specific broker, pass `only_from_brokers`.

## 2. Quote

Call `create_quote`. If there are multiple quotes returned, as the client which one to use. **Never assume, which is the best quote**.

When you list the quotations to choose from, give each one's broker, rate, amounts, fee and a link to **that broker's terms and conditions** (`broker.terms_url` on that quotation).

- `data.quote.uuid` (to refresh)
- the chosen quotation's `broker.code`
- the chosen quotation's `broker.terms_url`
- the approved figure: the `buy_amount` if `side` is `sell`, or the `sell_amount` if `side` is `buy`

## 3. Approval summary

Show this, then stop and wait for the reply:

```
Please confirm this trade on <environment> (account: <trading_account_name>):

  You sell     10,000.00 GBP
  You buy      11,642.00 EUR
  Rate         1 GBP = 1.1642 EUR   (market 1.1675)
  Broker       <broker.name>
  Delivery     Mon 6 Oct 2026
  Pay by       <settlement_at>: you must send 10,000.00 GBP to the broker by then
  Fee          <payment_fee> per payment you add to this trade
  Reason       <reason>
  Terms        By booking you accept <broker.name>'s terms and conditions:
               <broker.terms_url>

Booking is binding and can't be cancelled here.
Prices last 15 seconds, so when you confirm I'll get a fresh price and book only
if it's the same or better for you. Otherwise I'll show you the new price first.

Reply **yes** to book.
```

- Read the rate direction from the numbers, as described in fx-quote.
- For a forward (`delivery_date` well beyond the earliest date), also state that a deposit may be required. Its amount and deadline appear on the trade after booking.
- On production, start with **"This is a live trade with real money."**
- **The terms link is mandatory.** Show it as a clickable link, taken from `broker.terms_url` on the **chosen quotation** in this response. Never take it from another broker, an earlier quote or memory, and never shorten or rewrite it.
- If the chosen quotation has no `terms_url`, **don't ask for approval and don't book.** Say the broker's terms couldn't be retrieved, and offer another broker's quotation that has a terms link, or booking in the CurrencyTransfer web app.
- Leave out any other line whose value isn't in the response.

## 4. On a clear "yes"

The original quotation will have expired by now. In the **same** turn:

1. Call `refresh_quote` with `uuid` = `data.quote.uuid`, and `only_from_brokers` if you used it.
2. Find the quotation from the **same broker** (`broker.code`). Compare its approved figure:
   - side `sell`: new `buy_amount` ≥ approved `buy_amount`
   - side `buy`: new `sell_amount` ≤ approved `sell_amount`
3. Check that the refreshed quotation's `broker.terms_url` is the same link the user saw. If it changed or is missing, don't book. Show the new terms link in a new summary and ask again.
4. If the price is the same or better and the terms link is unchanged, immediately call `create_trade` with `broker_quotation_uuid` = **that refreshed quotation's `uuid`** (never the quote uuid) and `agree_to_terms: true`.
5. If it's worse, or that broker is missing, **don't book**. Show the new figures as a new summary and ask again.

Any reply other than a clear yes is not approval (see ct-best-practices). If the user changes the amount, currency or date, start again from step 2.

## 5. Booking errors

| Response | Meaning | Do |
| --- | --- | --- |
| 404 | Quotation expired or not found | Refresh once and repeat the step 4 comparison. If it still fails, show a new summary. |
| 409 `market_shifted` | The rate moved more than 0.15% against the user | Refresh, then show the new price as a new summary and ask again. **Never** book a worse price without new approval. |
| 409 `already_booked` | This quotation was already booked | **Don't book again.** Find the trade with `list_trades` (same currencies, `created_at_from` today) and report it. |
| 409 `invalid_delivery_date` | The broker can't book that date | Explain, ask the user to pick a later delivery date, then quote again and ask for approval |
| 403 | The trader isn't allowed to book | Explain. The account owner can grant permission in the web app. |
| `uncertain` / timeout | It may or may not have booked | **Never retry.** Check `list_trades` for a new trade with these currencies and amounts, and report what you find. |

## 6. After booking

Report the result in one sentence, then the next steps:

> Booked: trade **<trade_reference>**. You're selling 10,000.00 GBP for 11,642.00 EUR at 1.1642. Send 10,000.00 GBP to <broker> by **<settlement_at>**.

Then:

1. Call `get_trade` with the new `uuid` and show the **settlement instructions** exactly as the **trade-status** skill describes. Include the reference, and respect `externally_managed_settlement_accounts`. For a forward, show `forward_deposit` and `deposit_by`.
2. Mention `confirmation_pdf_url` if present.
3. Offer the next step: "Who should receive the EUR? I can add the payment." That's the **create-payment** skill, which needs its own approval.
