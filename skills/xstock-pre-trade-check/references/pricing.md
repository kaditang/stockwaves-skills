# Pricing a tokenized stock correctly

## Per raw token vs per share

Some tokenized stocks reinvest dividends by raising a **share multiplier** instead of paying cash
(xStocks on Solana use the Token-2022 `ScaledUiAmount` extension). After a few dividends, one raw
token represents slightly more than one share, for example 1.0057 shares.

Price sources disagree on which unit they quote:

| source | quotes |
| --- | --- |
| DexScreener, GeckoTerminal, most pool math | per **raw token** |
| Jupiter price API, the issuer's own quote | per **share** |

So, on a dividend-paying stock, comparing a DEX price straight to the stock price shows a fake
premium roughly equal to the multiplier (about +0.5% to +0.6% on SPYx in 2026), and two "correct"
APIs disagree with each other by that much.

**Rule:** fair raw-token price = `share_multiplier.value` x stock price. StockWaves' `depeg_bps`
and `basis.bps` are already computed this way (net of the multiplier). If you compute a premium
yourself, use the multiplier, and say which unit your quote is in.

`share_multiplier.status`:
- `applied`: a multiplier is in force; `value` is shares per raw token.
- `none_on_mint`: the mint has no multiplier, `value` is 1.
- `unavailable` / `not_read`: unknown. Do not assume 1 for a dividend-paying stock.

## Market hours

US stocks trade about 6.5 regular hours plus extended sessions on weekdays; tokenized stocks trade
24/7. When the US tape is shut there is no live price to compare against:

- `depeg_bps` is measured against a Chainlink equity feed with a 24-hour heartbeat. Off-hours its
  latest value is an arbitrary intraday print, **not the close**. Do not call a token "off peg"
  from `depeg_bps` alone when `market_status` is `closed`.
- `basis.bps` compares against the last live print of the underlying (including pre/post market).
  `basis.source: no_live_tape` means the tape is shut (weekend/overnight): a calendar fact, not a
  problem with the token.
- Weekend DEX prices for tokenized stocks are thin and can drift; expect the token to re-anchor
  when the US market reopens.

## Execution cost

A quoted mid is not a fill. Measured round-trip cost (buy then sell) on Solana DEX pools for xStocks
in 2026 ranged from about 0.17% for the deepest names to about 1.25% for thin ones, which is far
larger than the typical gap between the token and the stock (median under 0.1%). Check
`liquidity_usd`, `price_dispersion_bps` and `thin_liquidity` before sizing, and get a real swap
quote from the user's router.
