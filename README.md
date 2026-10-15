# StockWaves xStock Check

An agent skill that checks a tokenized US stock **before** your agent buys it.

Tokenized stocks (Backed's xStocks such as `AAPLx`, Ondo's `AAPLon`, Coinbase's B20 `NVDAc`) are
easy to fake and easy to misprice. The skill makes the agent:

- confirm the token is on the issuer's own registry, not a copycat with the same ticker;
- compare its price to the real stock **net of the dividend share multiplier**, so a fairly priced
  token is not reported as 0.5% "rich";
- know when the US market is shut and no live price exists to compare against;
- check liquidity, where it actually trades, and whether the issuer can freeze it;
- stop on `avoid`, ask on `review`, and never pay without the user's approval.

It works with any wallet. Agent wallets that pay x402 (for example MetaMask Agent Wallet) can buy
the full check directly.

## What it costs

| check | endpoint | price |
| --- | --- | --- |
| Verdict + authenticity | `GET https://stockwaves.net/api/xstock/health/preview?symbol=AAPLx` | free, 25/day per IP |
| Full signals | `GET https://stockwaves.net/api/xstock/health?symbol=AAPLx` | $0.03 USDC via x402 (Base or Solana) |
| Compare issuers / watch holdings | `?underlying=AAPL` / `?watch=addr1,addr2` | $0.03 each |

A failed check is never charged.

## Install

```bash
# Any agent that supports Agent Skills
npx skills add kaditang/stockwaves-skills

# Claude Code
claude plugin marketplace add kaditang/stockwaves-skills
claude plugin install stockwaves-xstock-check@stockwaves

# Codex
codex plugin marketplace add kaditang/stockwaves-skills
codex plugin add stockwaves-xstock-check@stockwaves

# Grok Build
grok plugin install kaditang/stockwaves-skills

# Antigravity
agy plugin install https://github.com/kaditang/stockwaves-skills

# Cursor: Customize → Plugins → add from Git: https://github.com/kaditang/stockwaves-skills
```

The skill has no hooks and no scripts: it is instructions plus `curl` calls. It never reads keys,
never signs, and never trades.

## Not advice

Screening only: issuer authenticity, peg, liquidity, market hours, and on-chain facts. It is not
investment, legal, or eligibility advice, and it does not decide whether anyone may legally buy a
tokenized security.

## License

MIT
