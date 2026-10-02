---
name: fx-quote
description: >
  Get an indicative FX quote from CurrencyTransfer: check the pair is tradable, pick a valid delivery date, request quotations from the user's brokers, and explain rate, fees and settlement deadline. Use when the user asks "how much would X cost in Y", "get me a quote", "what rate can I get", "what's GBP/EUR right now", or wants to compare brokers for a conversion. Do NOT use to book a trade (book-trade), for balances (cash-position) or for existing trades (trade-status).
metadata:
  author: CurrencyTransfer
  version: 0.1.0
compatibility: Works with the CurrencyTransfer MCP server (connected and authenticated). Needs create_quote, which the hosted server exposes by default.
---

# FX Quote

## Scope

This skill gets **indicative** prices. A quote commits the user to nothing, so it needs no approval.

If the user wants to book, accept or "go ahead with" a quote, switch to the **book-trade** skill. Booking always needs the user's explicit approval, and this skill never calls `create_trade` itself.

There is no way to hold or lock a rate. Quotations expire after **15 seconds**.

## Inputs

`create_quote` takes:

| Field | Notes |
| --- | --- |
| `sell_currency` | ISO code the user **has** (pays with), uppercase |
| `buy_currency` | ISO code the user **wants** (receives), uppercase |
| `side` | `sell` if `amount` is in the sell currency, `buy` if it's in the buy currency |
| `amount` | Decimal **string**, e.g. `"10000.00"`. No thousands separators. |
| `delivery_date` | `YYYY-MM-DD` |
| `reason` | Required. Allowed characters: letters, digits, spaces and `- / ? : ( ) . , ' +` |
| `only_from_brokers` | Optional list of broker codes |

### Work out `side` from the wording

- "Sell 10,000 GBP for EUR" / "convert 10k GBP to EUR": sell GBP, buy EUR, `side: sell`, amount 10000.
- "I need 10,000 EUR, paying with GBP" / "pay a €10k invoice from GBP": sell GBP, buy EUR, `side: buy`, amount 10000.
- "How much is 10,000 GBP in EUR?": treat it as `side: sell`.

If the currencies or amount are missing, ask once for everything missing in one message. Don't guess an amount. Ask the delivery date question (step 2) in that same message.

## Steps

1. **Check the pair.** Call `get_currency_pairs`. `data.currency_pairs` maps each sell currency to the buy currencies allowed. `data.delivery_dates` lists the earliest `delivery_date` per pair. If the pair isn't allowed, say so and list the buy currencies available for that sell currency.
2. **Ask about the delivery date. Always.** Never choose a date for the user, and don't quote until they answer. Ask whether they want the **earliest delivery date** (name it, e.g. "Mon 6 Oct") or a **future date**:

   > Do you want delivery on the earliest date, Mon 6 Oct, or on a later date?

   Skip the question only if the user already chose in this request, e.g. "earliest", "spot", "as soon as possible", or a specific date.
   - **Earliest:** use the pair's date from `data.delivery_dates`.
   - **Future date:** it must be after the earliest date, a weekday, and not a holiday for either currency. Check holidays with `get_currency_holidays`, whose `data` maps each currency to its holiday dates. If the date is invalid, say why and suggest the nearest valid dates, and let the user choose. A later date may make it a **forward** trade, which can need forward trading activated on the account and a deposit.
3. **Reason.** Use the purpose the user gave (e.g. "Supplier invoice INV-1043"), stripped to the allowed characters. If they gave none, use `Indicative price check`.
4. **Quote.** Call `create_quote`. The response has `data.quote` (the request, with `uuid` and `market_rate`) and `data.broker_quotations`, ordered **best rate first**.
5. **Present it** as shown below.
6. **Refresh** only when asked, or when the user comes back after the 15 seconds and wants a current price. Call `refresh_quote` with `uuid` = `data.quote.uuid` (the quote, **not** a broker quotation), and pass the same `only_from_brokers` if you used one. Never refresh in a loop.

## Present the result

Read the direction of `rate` from the numbers, not from assumptions: if `buy_amount ≈ sell_amount × rate`, then `rate` is buy currency per 1 unit of sell currency. Label it explicitly, e.g. `1 GBP = 1.1642 EUR`.

Lead with the best quotation in one or two sentences, then the details:

```
Best indicative price: 10,000.00 GBP → 11,642.00 EUR with <broker name>
Rate 1 GBP = 1.1642 EUR (market 1.1675, about 0.28% from mid)
Payment fee: 5.00 GBP per payment
Delivery 2026-10-06 · funds must reach the broker by <settlement_at>

Valid for 15 seconds. Indicative only. Say if you'd like to book it.
```

- Spread from mid: `|rate − market_rate| / market_rate`, as a percentage with two decimals. Leave it out if `market_rate` is missing.
- `payment_fee` is charged per payment if the quote is booked. Say which currency it's in only if the response makes that clear. Otherwise say "per payment".
- `settlement_at` is when the broker must receive the sell amount if the quote is booked. Show it in the user's terms (date and time with timezone, as returned).
- If there are several quotations, add a compact table of all brokers (broker, rate, they receive or pay, fee) under the headline. Keep the order as returned.
- Format amounts with thousands separators and 2 decimals (or the currency's usual minor units). Never round the rate to fewer than 4 significant decimals.

If the user then asks to book, follow the book-trade skill. It re-quotes and asks for approval. Never book from this skill.

## Errors

The provider's error `code` tells you what went wrong:

- **`unprocessable_entity`:** validation errors in `field_codes`. The usual causes are an invalid delivery date, a pair that isn't allowed, or a disallowed character in `reason`. Fix what you can (next valid date, cleaned reason) and retry **once**. Otherwise explain. An `over_limit` error on `delivery_date` with `forward_disabled` means the date needs forward trading, which the account hasn't activated. Offer the earliest spot date instead, and say forwards are activated in the web app.
- **`market_closed`:** the market is closed for this pair. If `field_codes.base` has `opens_after`, say when it reopens. Don't retry.
- **`no_results`:** no broker returned a price. Suggest trying again shortly or with a different amount or date. Don't retry in a loop.
- **`is_verified` false:** if quoting fails with a permission error, call `get_user`. An unverified account can't quote yet, and they should finish verification in the web app.
- **Tool missing:** if `create_quote` isn't available, the CurrencyTransfer server isn't connected or signed in, or the URL isn't one of the hosted servers. Point the user to the connection check in the `ct-best-practices` skill.
