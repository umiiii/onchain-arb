---
name: onchain-arb:support
description: "This skill should be used when the user asks which exchanges are supported, wants to check if a CEX skill is installed, or asks things like 'which exchanges does arb support', 'is Binance connected', 'list supported exchanges', '支持哪些交易所', '检测交易所'."
allowed-tools:
  - Skill
  - Read
  - Bash
  - Agent
---

# Supported Exchange Detection

Detect which CEX price-query skills are currently installed, list supported exchanges, and optionally discover new ones.

## Workflow

1. Read `${CLAUDE_PLUGIN_ROOT}/references/cex-registry.md` to get the known exchange-to-skill mapping.
2. For each registered exchange, check if the corresponding skill is available in the current environment (attempt to detect from the skill list or installed plugins).
3. Report results.
4. If the user explicitly asks to discover or add new exchanges, invoke the `cex-discovery` agent to scan for additional CEX-related skills not yet in the registry. Do not invoke the agent automatically on every call — only when discovery is requested.

## Output Format

```
onchain-arb 交易所支持状态

已注册交易所:
┌───────────┬─────────────────────────┬────────┐
│ 交易所    │ Skill                   │ 状态   │
├───────────┼─────────────────────────┼────────┤
│ OKX       │ okx-cex-market          │ ✅ 可用 │
│ Binance   │ binance/binance-skills-hub │ ✅ 可用 │
│ Coinbase  │ coinbase/coinbase-market │ ❌ 未安装│
└───────────┴─────────────────────────┴────────┘

DEX 数据源: okx-dex-market (onchainOS) ✅ 可用
支持链: Ethereum, BSC, Arbitrum, Solana, ...
```

If the `cex-discovery` agent finds new CEX skills:
```
🔍 发现新的 CEX skill:
  - <skill-name> → 可能对应 <exchange>
  是否将其添加到支持列表？
```

## Rules

- 状态: "✅ 可用" if skill is detected, "❌ 未安装" if not found.
- Always check `okx-dex-market` availability as the DEX data source.
- If no CEX skills are detected at all, warn: "⚠ 未检测到任何 CEX skill，套利功能将仅支持跨链 DEX 对比"
- After displaying the table, remind the user they can manually add exchanges by telling the assistant the skill name.
