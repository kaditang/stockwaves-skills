# Watch tokenized stocks the user already holds

```text
GET https://stockwaves.net/api/xstock/health?watch=<addr1,addr2,...>&since=<minutes>   ($0.03, x402)
```

Up to 10 token addresses per call. The response says, per token, whether anything **material**
changed in the look-back window, with a severity (`critical` | `high` | `medium`) and a
plain-English consequence. Material means: authenticity flipping, the issuer halting transfers,
a freeze authority appearing, liquidity halving, or the peg crossing a band. Ordinary price moves
and the market opening or closing are deliberately never reported, so a quiet answer means
nothing material changed.

Suggested use: when the user asks "anything I should know about my tokenized stocks?", or on a
schedule the user has approved (daily is plenty). Escalate `critical` and `high` changes to the
user immediately; batch `medium` ones into a summary.
