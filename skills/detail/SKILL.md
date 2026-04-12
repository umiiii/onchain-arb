---
name: onchain-arb:detail
description: "This skill should be used when the user wants a detailed arbitrage analysis including gas costs, slippage, net profit calculations, and optimal trade paths. Triggers on phrases like 'detailed arb analysis for ETH', 'is the arbitrage profitable after gas', 'show me the best arb path for UNI with fees', 'full arbitrage breakdown', '详细套利分析', '扣除gas后还有利润吗'."
argument-hint: "<token symbol, e.g. OKB, ETH, UNI>"
allowed-tools:
  - Skill
  - Read
  - Bash
---

# Detailed Arbitrage Analysis

Perform a comprehensive arbitrage analysis for a single token, extending the quick check with gas costs, slippage estimates, and actionable insights.

## Workflow

1. Follow the data collection workflow in `${CLAUDE_PLUGIN_ROOT}/references/data-collection-workflow.md` to gather CEX and DEX prices for the token.
2. Collect additional data:
   - Gas price on each queried chain (via `okx-dex-market` or `okx-onchain-gateway` if available).
   - Estimated swap gas cost in USD for a standard trade on each DEX.
   - Cross-chain bridge cost estimates if multiple chains are involved.
3. Compute and display the detailed report.

## Output Format

### Section 1: Price Comparison Table

Same format as the quick check table (see `arbitrage` skill).

### Section 2: Arbitrage Opportunity Analysis

All calculations based on the trade amount ($1,000 or user-set value). For each pair where price difference exceeds 0.1%, display:

```
套利机会分析 (模拟金额: $1,000)
┌─────────────────────┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ 路径                │ 买入均价 │ 卖出均价 │ 买入量   │ 总费用   │ 净利润   │ 收益率   │
├─────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ Ethereum → OKX      │ $47.98   │ $48.52   │ 20.84    │ ~$4.50   │ ~$6.75   │ +0.68%  │
│ BSC → OKX           │ $47.82   │ $48.52   │ 20.91    │ ~$1.30   │ ~$13.33  │ +1.33%  │
│ Arbitrum → Binance  │ $48.12   │ $48.48   │ 20.78    │ ~$1.15   │ ~$6.33   │ +0.63%  │
└─────────────────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

- 买入均价 / 卖出均价: effective average price after slippage for the trade amount
- 买入量: actual tokens received when spending the trade amount at the buy source
- 总费用: sum of DEX swap fee + CEX trading fee + Gas + bridge cost (if cross-chain)
- 净利润: (卖出均价 - 买入均价) × 买入量 - 总费用, in USD
- 收益率: 净利润 / trade amount, as percentage

### Section 3: Summary & Notes

- Highlight the top 1-3 most profitable paths by net profit in USD.
- Show a clear verdict: "以 $1,000 投入，最佳路径 <path> 预估净利润 $xx.xx（收益率 x.xx%）"
- Note any risks: low liquidity, high gas, bridge delay, CEX withdrawal restrictions.
- If all paths have net profit < $5, explicitly state: "以 $1,000 交易量，当前无可行套利机会"
- If cross-chain arbitrage involves bridge costs that exceed the profit, mark as "不划算".

## Rules

- All monetary values in USD.
- Gas estimates are approximate — note this in the output.
- Sort opportunity table by net profit % descending.
- If gas or bridge cost data is unavailable, show "未知" and warn the user.
- CEX trading fees: assume 0.1% maker/taker unless the CEX skill provides fee info.
- DEX swap fees: use the fee tier from the liquidity pool data if available, otherwise assume 0.3%.
