# Pre-trade check (before any buy or swap into a tokenized stock)

## 1. Pin down the exact token

You need either the issuer symbol (`AAPLx`, `AAPLon`, `NVDAc`, ...) or the token's contract
address (Solana mint or `0x...`). If the user only named the stock ("buy some Apple on-chain"),
do not guess a contract: check by symbol, or run [compare-issuers.md](compare-issuers.md) and let
the user choose. If your swap route resolved a token address, check **that address**, not the
symbol: copycats share the symbol.

## 2. Free check

```bash
curl -s "https://stockwaves.net/api/xstock/health/preview?address=<mint or 0x address>"
curl -s "https://stockwaves.net/api/xstock/health/preview?symbol=AAPLx"
```

| HTTP | Meaning | Do |
| --- | --- | --- |
| 200 | verdict returned | go to step 3 |
| 400 | unknown symbol / bad address | ask the user for the contract address; an unknown token is not a pass |
| 429 | free quota used up (25/day per IP) | use the full check ([full-check-x402.md](full-check-x402.md)) or ask the user |
| 502 / 503 | upstream or StockWaves busy | retry once after a few seconds; if it still fails, tell the user the check could not run |

## 3. Act on the decision

- **`avoid`**: stop. Do not build the transaction. Report `recommendation` and
  `authenticity.red_flags` to the user in plain words.
- **`review`**: show the user the `recommendation` and ask before continuing. Common benign
  reasons: the US market is closed (no live reference price), or the token is minted on a chain
  where it has no DEX market (you could buy it but would have to bridge to sell).
- **`ok`**: proceed with the user's wallet as normal.

Always also check `token.issuer_verified` and `authenticity.verified` are `true`. If either is
false, treat the token as `avoid` whatever the decision says.

## 4. When to buy the full check ($0.03)

Run [full-check-x402.md](full-check-x402.md) before trading when any of these hold:

- the trade is large relative to the user's stated budget (default: above $500);
- the decision is `review` and the user wants to know by how much the price is off;
- the market is closed and you need to know where the token trades versus the last live print;
- you are about to quote a price to the user (you need `share_multiplier`; see
  [../references/pricing.md](../references/pricing.md)).

## 5. Tell the user what you checked

One or two lines: issuer, verified or not, the decision, and anything the check could not
cover (the free check does not give exact peg or liquidity). Never present a passed check as
investment advice.
