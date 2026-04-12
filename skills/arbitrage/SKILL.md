---
name: onchain-arb
description: "This skill should be used when the user wants to check arbitrage opportunities for a single token across CEX and DEX, compare prices between exchanges and on-chain, or asks things like 'check arb for OKB', 'compare OKX vs DEX price', 'is there a price difference for ETH across exchanges', 'CEX-DEX price spread for TOKEN', '查一下OKB的价差', '链上链下套利'."
argument-hint: "<token symbol, e.g. OKB, ETH, UNI>"
allowed-tools:
  - Skill
  - Read
  - Bash
---

# Arbitrage Quick Check

Query a single token's price across all supported CEXs and DEX chains, then display a concise arbitrage snapshot table.

## Workflow

1. Follow the data collection workflow in `${CLAUDE_PLUGIN_ROOT}/references/data-collection-workflow.md` to gather CEX and DEX prices for the token.
2. Compute the arbitrage table and display results.

## Output Format

Use the highest price as the baseline (价差% = 0, marked as "基准"). Calculate all other prices as percentage deviation from the baseline.

The default trade amount is **$1,000**. Refer to `${CLAUDE_PLUGIN_ROOT}/references/data-collection-workflow.md` for the trade amount rules and first-run prompt.

For each source, estimate how many tokens the trade amount can actually buy, accounting for slippage (order book depth for CEX, pool liquidity for DEX).

Display the following table:

```
<TOKEN> 套利快照 (基准: <source> $<price>) | 模拟金额: $1,000
┌───────────────┬──────┬──────────┬────────┬──────────────┬──────────┐
│ 来源          │ 类型 │ 价格(USD)│ 价差%  │ $1000可买入量 │ 实际均价  │
├───────────────┼──────┼──────────┼────────┼──────────────┼──────────┤
│ OKX           │ CEX  │ xx.xx    │ 基准   │ xx.xx TOKEN  │ $xx.xx   │
│ Binance       │ CEX  │ xx.xx    │ -x.xx% │ xx.xx TOKEN  │ $xx.xx   │
│ Ethereum      │ DEX  │ xx.xx    │ -x.xx% │ xx.xx TOKEN  │ $xx.xx   │
│ BSC           │ DEX  │ xx.xx    │ -x.xx% │ xx.xx TOKEN  │ $xx.xx   │
└───────────────┴──────┴──────────┴────────┴──────────────┴──────────┘
```

Rules:
- Sort rows by price descending (highest first)
- 来源 column: use exchange name for CEX, chain name for DEX (Ethereum, BSC, Arbitrum, Solana, etc.)
- 类型 column: CEX or DEX
- 价差%: negative means cheaper than baseline, positive means more expensive
- $1000可买入量: the actual number of tokens receivable when spending $1,000 (or user-set amount), after slippage. If slippage data is unavailable, compute using spot price and note "不含滑点"
- 实际均价: trade amount / 可买入量, reflecting the effective average price including slippage
- If a CEX skill is not installed or fails, skip that row and note it below the table

## Quick Analysis

After the table, add a brief "快速判断" section based on the trade amount ($1,000 or user-set value):
- Compute the best arbitrage path: buy at the source with the most tokens per $1,000, sell at the source with the highest price. Show the estimated profit in USD: "以 $1,000 在 <低价源> 买入 xx.xx TOKEN，在 <高价源> 卖出，预估毛利 $xx.xx（扣除费用前）"
- Warn about low liquidity sources (total pool liquidity < $500K on DEX) — large slippage will erode profits
- If the estimated net profit (after fees + gas) is negative or < $5, state: "以 $1,000 交易量，当前无可行套利机会"
- Note if a token's native chain has significantly better liquidity than others

## Error Handling

- If the token symbol is not provided, ask the user for it.
- If a CEX skill query fails, skip that exchange and add a note: "⚠ <Exchange> 查询失败，跳过"
- If the token is not found on Ethereum DEX, try other chains or ask the user.
