# Cash position: response templates

Adapt the wording. Keep the structure and order. Replace `<…>` with real values and drop lines that don't apply.

## Overview (default)

```
Hi <first name>, here's <trading account name> as of <cached_at time>.

<One sentence: e.g. "You're in good shape. One trade needs funding by Thursday." /
 "Nothing needs your attention right now.">

**Needs attention**
- Send 25,000.00 GBP to <broker> by Thu 9 Oct 14:00 for trade <trade_reference> (buys 29,100.00 EUR).
- 3,400.00 USD from trade <trade_reference> hasn't been assigned to a payment yet.

**Current position**
- EUR 48,210.55 (withdrawable 48,210.55)
- GBP 12,004.10 (withdrawable unknown)
- USD 0.00

**Coming up**
- <n> trades awaiting funds: <total per sell currency>, earliest deadline <date>.
- <n> trades being processed. Payments are on their way.
```

## Single-currency question ("how much EUR do I have?")

```
You have 48,210.55 EUR (48,210.55 withdrawable) as of 14:05 UTC,
held with <broker>.
```

If it's held with several brokers, add one line per broker. If trades will change it soon, add one line: "A trade buying 29,100.00 EUR settles on Thu 9 Oct."

## No balances

```
Balances aren't set up for <trading account name>, so I can't show account balances.
Here's what's in flight instead: …
```

## Deposit details

```
To add EUR to your <broker> balance, send a bank transfer to:

Beneficiary   <beneficiary_name>
IBAN          <iban>
BIC/SWIFT     <bic_swift>
Bank          <bank_name>, <bank_address>
Reference     <reference>  ← include this exactly

Check these against the CurrencyTransfer web app before your first transfer.
```

Show only the fields the response contains, in this order. Never invent missing ones.

## Detail table (on request or more than 3 currencies)

| Currency | Balance | Withdrawable | Broker |
| --- | ---: | ---: | --- |
| EUR | 48,210.55 | 48,210.55 | <broker> |
| GBP | 12,004.10 | n/a | <broker> |
