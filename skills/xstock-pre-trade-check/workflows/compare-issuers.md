# Compare issuers for one stock

Several issuers tokenize the same stock: Backed's xStocks (`AAPLx`, mainly Solana, also minted on
many EVM chains), Ondo Global Markets (`AAPLon`), and Coinbase B20 on Base (`AAPLc`). They differ
in where they actually trade, how deep the market is, whether dividends are reinvested through a
share multiplier, and whether the issuer can freeze the token.

```text
GET https://stockwaves.net/api/xstock/health?underlying=AAPL      ($0.03, x402; see full-check-x402.md)
```

Returns every **verified** tokenized version of that stock with its decision and signals.
Unverified look-alikes are never included, so anything a swap route offers that is not in this
list should be treated as unverified.

How to present the choice to the user:

1. Drop anything with decision `avoid`.
2. For each remaining version, show: issuer, chain, decision, `liquidity_usd`, the peg
   (`depeg_bps`, or `basis.bps` off-hours), and `freeze_authority`.
3. Point out a version that is minted on the user's chain but has no DEX market there
   (`tradeable_on`): buying it there means bridging before you can sell.
4. Let the user pick. Do not choose for them on price alone.
