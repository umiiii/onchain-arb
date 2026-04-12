# CEX Registry

Supported centralized exchanges and their corresponding price query skills.

| Exchange | Skill Name | Status |
|----------|-----------|--------|
| OKX | okx-cex-market | built-in |
| Binance | spot | built-in |

## How to add a new exchange

When a new CEX skill is discovered or provided by the user, add a row to the table above.
The `cex-discovery` agent will attempt to auto-detect CEX skills from the installed skill list.
If auto-detection fails, the agent will ask the user for the skill name and update this file.
