# Shared Data Collection Workflow

This is the standard workflow for collecting token price data across CEX and DEX sources. Referenced by both the `arbitrage` and `detail` skills.

## Trade Amount

Default trade amount is **$1,000 USD**. The user may change this value at any time by saying "设置金额为 $X" or "set amount to $X". All calculations (buy quantity, slippage, profit) are based on this amount.

**First-run behavior**: The first time any onchain-arb skill is invoked in a session, append this line after the output:
> 💡 当前模拟交易金额为 **$1,000**。如需更改，请输入"设置金额为 $X"（如"设置金额为 $5000"）

After the user has seen this prompt once, do not repeat it in the same session.

## Steps

1. **Read CEX registry**: Load `${CLAUDE_PLUGIN_ROOT}/references/cex-registry.md` to get the list of supported exchanges and their corresponding skill names.

2. **Query CEX prices**: For each exchange in the registry, invoke the corresponding skill to query the token's spot price. Also query the order book to estimate how many tokens $1,000 (or the user-set amount) can actually buy/sell, accounting for order book depth and slippage.
   - OKX: invoke `okx-cex-market`
   - Binance: invoke `spot`
   - Other exchanges: use the skill name from the registry
   - If a skill is not installed or fails, skip that exchange and note: "⚠ <Exchange> 查询失败，跳过"

3. **Query DEX prices**: Invoke `okx-dex-market` to query the token's on-chain price.
   - Default chain: Ethereum
   - If the token does not exist on Ethereum, or if on-chain data shows it is available on multiple chains, ask the user which chain(s) to query
   - Query all available/selected chains to compare cross-chain prices
   - Estimate how many tokens $1,000 (or user-set amount) can buy on each DEX, accounting for pool liquidity and slippage

4. **Compute baseline**: Use the highest price across all sources as the baseline (价差% = 0, marked as "基准"). Calculate all other prices as percentage deviation from the baseline.
