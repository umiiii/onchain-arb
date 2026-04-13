# onchain-arb

CEX-DEX and cross-chain arbitrage opportunity detector powered by onchainOS data.

## Features

- Compare token prices across centralized exchanges (OKX, Binance, etc.) and on-chain DEXs
- Cross-chain DEX price comparison (Ethereum, BSC, Arbitrum, Solana, etc.)
- Simulated trade analysis based on configurable amount (default $1,000)
- Detailed arbitrage reports with gas costs and net profit estimates
- Batch scanning of trending tokens for arbitrage opportunities
- Price anomaly detection: automatically identifies stale/legacy token contracts and finds the correct chain
- Auto-discovery of installed CEX skills for extensibility

## Commands

| Command | Description |
|---------|-------------|
| `/onchain-arb <token>` | Quick arbitrage snapshot for a single token |
| `/onchain-arb:detail <token>` | Detailed analysis with gas, slippage, net profit |
| `/onchain-arb:scan` | Scan trending tokens for arbitrage opportunities |
| `/onchain-arb:support` | List supported exchanges and their status |

## Agents

### arbitrage-checker (main agent)

The main agent of this plugin. Can be triggered in two ways:

**Natural language trigger** — No need for slash commands, just ask naturally:
- "帮我看看 OKB 有没有套利机会"
- "ETH 在链上和交易所的价格差多少？"
- "check arb for BNB"

**Proactive anomaly detection** — When a token's DEX price on a chain deviates >15% from CEX prices, the agent automatically:
1. Flags the anomalous price source (e.g., Ethereum legacy ERC-20 contract)
2. Searches other chains for the correct price
3. Diagnoses the cause (stale pool, low liquidity, token migration, etc.)
4. Produces a corrected arbitrage table excluding anomalous sources

### cex-discovery

Discovers and registers new CEX skills. Scans installed plugins for exchange-related skills and maintains the CEX registry. Invoked when users ask about supported exchanges or want to add a new one.

## Trade Amount

All arbitrage conclusions are based on a simulated trade amount, default **$1,000**. Change it anytime:

```
设置金额为 $5000
set amount to $10000
```

## Prerequisites

This plugin requires the following skills to be installed:

- **`okx-dex-market`** — On-chain price data (required)
- **`okx-cex-market`** — OKX CEX price data (recommended)
- **`spot`** — Binance CEX price data (recommended)
- **`okx-dex-token`** — Trending token list, needed for `/onchain-arb:scan`

A `SessionStart` hook automatically checks dependency status when the plugin loads.

## Adding More Exchanges

The plugin maintains a CEX registry at `references/cex-registry.md`. New exchanges can be added:

1. **Automatically** — The `cex-discovery` agent scans installed skills for CEX capabilities
2. **Manually** — Tell the assistant "add Coinbase support" and provide the skill name

## Installation

```bash
claude --plugin-dir /path/to/onchain-arb
```
