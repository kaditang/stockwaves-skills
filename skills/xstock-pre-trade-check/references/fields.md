# Response fields

Free preview (`/api/xstock/health/preview`) returns the first block only; exact numbers are
masked as `[paid]`. The full check (`/api/xstock/health`, $0.03) returns everything.

## Verdict (free and paid)

| field | meaning |
| --- | --- |
| `decision` | `ok` / `review` / `avoid`. See the hard rules in SKILL.md. |
| `token.symbol`, `token.chain`, `token.issuer` | what was checked |
| `token.issuer_verified` | the token is on the issuer's own published registry |
| `authenticity.verified` | same check, with `registry_status` (`live` / `degraded`) |
| `authenticity.red_flags` | why it looks like an impersonator; empty when clean |
| `recommendation` | plain-English summary; safe to show the user as-is |

## Signals (paid, under `signals`)

| field | meaning | trap |
| --- | --- | --- |
| `dex_price_usd` | liquidity-weighted median across pools, per raw token | not per share: see pricing.md |
| `reference_price_usd`, `reference_source`, `reference_status` | Chainlink equity feed, or the issuer's own quote where no feed exists | `no_feed` is a stable fact; `degraded` means our read failed this time, retry |
| `market_status` | `open` / `closed` / `unknown` for the US listing | |
| `depeg_bps` | token vs reference, net of the share multiplier | off-hours the reference is not a close |
| `basis` | token vs the last live print of the underlying (`bps`, `source`, `session`) | `no_live_tape` = market shut, not a token problem |
| `share_multiplier` | shares per raw token (`value`, `status`) | `unavailable` is not 1 |
| `liquidity_usd` | deepest single pool | the whole map is `tradeable_on` |
| `price_dispersion_bps`, `pools_priced`, `thin_liquidity` | how far apart pools are; wide = your fill depends on routing | |
| `tradeable_on` | per-chain venues with liquidity | a chain missing here has minted supply but no market |
| `venue_concentration_pct` | share of liquidity on the deepest chain | |
| `freeze_authority` | `present` / `absent` / `unknown` / `not_introspected_evm`: can the issuer freeze this token | a neutral on-chain fact, not a compliance finding |
| `onchain_supply` | minted supply | |
| `yield` | what holding it earns (dividends via the multiplier, lending, LP) | informational; does not change `decision` |
| `not_checked` | what this check does not cover (e.g. issuer proof-of-reserve, holder concentration) | mention it if the user asks "is it safe?" |

`disclaimer`: screening only. Not investment, legal, or eligibility advice; StockWaves does not
decide whether a given holder may legally buy a tokenized security.
