---
name: xstock-pre-trade-check
description: Use before buying, swapping into, quoting, or holding a tokenized US stock on-chain (xStocks like AAPLx/TSLAx/SPYx, Ondo Global Markets like AAPLon, Coinbase B20 like NVDAc, or any token claiming to be a stock). Checks that the token is the REAL issuer's token and not an impersonator, whether it trades near the stock's fair value (net of the dividend share multiplier), whether the market is open and liquid enough, and whether the issuer can freeze it. Free verdict via StockWaves; full signals for $0.03 over x402. Works with any wallet, including agent wallets that can pay x402.
license: MIT
metadata:
  author: stockwaves
  version: "0.1.0"
---

# xStock pre-trade check

Tokenized stocks are easy to fake and easy to misprice. Copycat tokens reuse the real ticker and
name; DEX price feeds quote some tokens per *raw token* while the issuer quotes per *share*; and
off-hours there is no live stock price to compare against. This skill makes the agent check the
token with StockWaves before any buy, and read the answer correctly.

StockWaves only reads public on-chain data and issuer registries. It never holds keys, never
signs, and never places a trade. It is not investment, legal, or eligibility advice.

## When to run it

| Situation | Do |
| --- | --- |
| About to buy or swap into a tokenized stock | [workflows/pre-trade.md](workflows/pre-trade.md) — always, before building the transaction |
| User gives a token address or symbol and asks "is this real / safe?" | [workflows/pre-trade.md](workflows/pre-trade.md) |
| Quoting or comparing a tokenized stock's price to the real stock | [references/pricing.md](references/pricing.md) first |
| Choosing between issuers for the same stock (AAPLx vs AAPLon vs AAPLc) | [workflows/compare-issuers.md](workflows/compare-issuers.md) |
| Already holding tokenized stocks, wants to know if anything changed | [workflows/watch.md](workflows/watch.md) |
| Need the full signal breakdown (exact peg, liquidity, venues, yield) | [workflows/full-check-x402.md](workflows/full-check-x402.md) — costs $0.03 |

Field meanings and the traps in each: [references/fields.md](references/fields.md).

## Hard rules

1. **`avoid` means stop.** Do not build, sign, or route a buy for a token whose decision is
   `avoid`. Tell the user why, using the response's `recommendation` and `red_flags`.
2. **`review` means ask.** Show the user the reasons and get an explicit yes before continuing.
3. **Authenticity beats price.** A token that is not `verified` is not the stock, however good
   its price or liquidity looks. Never "pick the cheapest AAPLx" across pools.
4. **Never compare a DEX price to the stock price without the share multiplier**
   ([references/pricing.md](references/pricing.md)). A dividend-paying xStock can look 0.5%+
   "rich" when it is fairly priced.
5. **Paying is the user's call.** The full check costs $0.03 (x402). Show the price and get
   approval before paying, unless the user has already approved paid checks for this session.
6. If the check itself fails (HTTP 5xx, timeout, `reference_status: degraded`), say so. A failed
   check is not a pass. Retry once; if it still fails, ask the user before proceeding unchecked.

## The free check in one call

```bash
curl -s "https://stockwaves.net/api/xstock/health/preview?symbol=AAPLx"
# or by contract: ?address=<Solana mint or 0x address>
```

Returns `decision` (`ok` | `review` | `avoid`), `token` (issuer, chain, `issuer_verified`),
`authenticity` (`verified`, `red_flags`), and a plain-English `recommendation`. Free, 25 calls
per day per IP, cached 5 minutes. Exact numbers (peg in bps, liquidity, venues, multiplier,
yield) are masked as `[paid]`; get them from the full check.
