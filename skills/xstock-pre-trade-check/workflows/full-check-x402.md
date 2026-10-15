# Full check over x402 ($0.03)

`GET https://stockwaves.net/api/xstock/health` with the same `?symbol=` or `?address=` as the
free check. It returns every signal: exact `depeg_bps`, live `basis`, `share_multiplier`,
`liquidity_usd`, `tradeable_on` (per-chain venues), `freeze_authority`, `yield`, and more
([../references/fields.md](../references/fields.md)).

## Payment

The endpoint answers `402 Payment Required` with an **x402 v2** challenge in the
`PAYMENT-REQUIRED` header. Payment options, all $0.03 in USDC, scheme `exact`:

| network | asset | notes |
| --- | --- | --- |
| `eip155:8453` (Base) | USDC `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | EIP-3009; what most agent wallets pay |
| `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | USDC `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` | for Solana x402 clients |
| `eip155:5042` (Arc) | USDC `0x3600...` | client-broadcast transfer; most wallets skip it, that is fine |

Settlement only happens when the response succeeds (status < 400): a failed check is not charged.

## How to pay

1. **Show the user the cost and get approval** ("This full check costs $0.03 in USDC on Base. OK?"),
   unless they already approved paid checks this session.
2. Pay with whatever x402 client the agent has:
   - **MetaMask Agent Wallet**: follow its *x402 pay* workflow (inspect, confirm with the user,
     pay). Our challenge has exactly one option it can pay (Base USDC), so no `--network` flag is
     needed.
   - **Code**: an x402 **v2** client such as `@x402/fetch` (`wrapFetchWithPayment`). The older
     v1 package `x402-fetch` cannot pay v2 servers.
   - Other x402-capable wallets: any client that signs the v2 `exact` scheme with EIP-3009 on Base.
3. Report the settlement transaction to the user alongside the result.

If the agent has no x402-capable wallet, say so and stay with the free check. Do not ask the user
for a private key to pay.

## Other paid modes (same price, same payment)

- `?underlying=TSLA`: every verified tokenized version of one stock, across issuers and chains.
  See [compare-issuers.md](compare-issuers.md).
- `?watch=<addr1,addr2,...>&since=<minutes>`: only what changed materially. See [watch.md](watch.md).
