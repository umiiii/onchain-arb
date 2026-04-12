# onchain-arb

CEX-DEX and cross-chain arbitrage opportunity detector powered by onchainOS data.

## Features

- Compare token prices across centralized exchanges (OKX, Binance, etc.) and on-chain DEXs
- Cross-chain DEX price comparison (Ethereum, BSC, Arbitrum, Solana, etc.)
- ±1% depth analysis for liquidity assessment
- Detailed arbitrage reports with gas costs and net profit estimates
- Batch scanning of trending tokens for arbitrage opportunities
- Auto-discovery of installed CEX skills for extensibility

## Commands

| Command | Description |
|---------|-------------|
| `/onchain-arb <token>` | Quick arbitrage snapshot for a single token |
| `/onchain-arb:detail <token>` | Detailed analysis with gas, slippage, net profit |
| `/onchain-arb:scan` | Scan trending tokens for arbitrage opportunities |
| `/onchain-arb:support` | List supported exchanges and their status |

## Prerequisites

This plugin requires the following skills to be installed:

- **`okx-dex-market`** — On-chain price data (required)
- **`okx-cex-market`** — OKX CEX price data (recommended)
- **`binance/binance-skills-hub`** — Binance CEX price data (recommended)
- **`okx-dex-token`** — Trending token list, needed for `/onchain-arb:scan`

## Adding More Exchanges

The plugin maintains a CEX registry at `references/cex-registry.md`. New exchanges can be added:

1. **Automatically** — The `cex-discovery` agent scans installed skills for CEX capabilities
2. **Manually** — Tell the assistant "add Coinbase support" and provide the skill name

## Installation

```bash
claude --plugin-dir /path/to/onchain-arb
```
