---
name: onchain-arb:scan
description: "This skill should be used when the user wants to scan multiple trending tokens for arbitrage opportunities in bulk, or asks things like 'scan for arb opportunities', 'which trending tokens have price spreads', 'batch arbitrage check', 'find tokens with CEX-DEX price differences', '扫描套利机会', '哪些热门币有价差'."
allowed-tools:
  - Skill
  - Read
  - Bash
---

# Arbitrage Scan

Batch scan trending tokens for CEX-DEX and cross-chain arbitrage opportunities.

## Workflow

1. Fetch the trending/hot token list from onchainOS by invoking the `okx-dex-token` skill (trending tokens / 热门代币).
2. Read `${CLAUDE_PLUGIN_ROOT}/references/cex-registry.md` for supported exchanges.
3. For each trending token (top 10-15 tokens):
   - Query each supported CEX for the token's spot price (skip if the token is not listed).
   - Query `okx-dex-market` for the token's on-chain price on Ethereum (and other chains if the token is multi-chain).
   - Compute the maximum price spread.
4. Display the scan results table, sorted by spread.

## Output Format

```
onchain-arb 套利扫描 (基于 onchainOS 热门代币)
扫描时间: YYYY-MM-DD HH:MM UTC
已检测交易所: OKX, Binance | 默认链: Ethereum

┌──────────┬──────────────┬──────────────┬────────┬──────────────────────┐
│ 代币     │ 最高价       │ 最低价       │ 价差%  │ 价差来源             │
├──────────┼──────────────┼──────────────┼────────┼──────────────────────┤
│ TOKEN_A  │ $xx.xx (OKX) │ $xx.xx (ETH) │ 2.35%  │ CEX↔DEX              │
│ TOKEN_B  │ $xx.xx (BSC) │ $xx.xx (BIN) │ 1.80%  │ 跨链                 │
│ TOKEN_C  │ $xx.xx (OKX) │ $xx.xx (ARB) │ 1.12%  │ CEX↔DEX              │
│ ...      │              │              │        │                      │
└──────────┴──────────────┴──────────────┴────────┴──────────────────────┘
发现 X 个代币存在 >0.5% 价差
```

## Rules

- Sort by 价差% descending (largest spread first).
- 最高价 / 最低价: show price and source in parentheses.
- 价差来源: "CEX↔DEX" for exchange-to-chain spread, "跨链" for cross-chain spread, "CEX↔CEX" for inter-exchange spread.
- Only display tokens where spread > 0.3%. If no tokens meet this threshold, display: "当前热门代币无显著套利机会 (>0.3%)"
- At the bottom, summarize how many tokens have >0.5% spread.
- If a token is only available on one source (e.g., only on DEX, not on any CEX), skip it.
- If the trending token list is unavailable, report the error and suggest the user try `/onchain-arb <token>` for individual queries.

## Error Handling

- If `okx-dex-token` skill is unavailable, inform the user: "⚠ 无法获取热门代币列表（okx-dex-token 未安装），请使用 /onchain-arb <token> 查询单个代币" and abort.
- If a CEX skill times out or errors, skip that exchange for the affected token and note it at the bottom.

## Performance Notes

- This operation queries multiple tokens across multiple sources — note to the user that it may take a moment.
