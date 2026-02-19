# Hyperliquid Skills

Agent skills for working with the [Hyperliquid API](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api). Compatible with Claude Code, Cursor, GitHub Copilot, OpenAI Codex, Gemini CLI, and any agent that supports the [agentskills.io](https://agentskills.io) standard.

## Quick Start

```bash
npx skills add cezar-r/hyperliquid-skills
```

### Install specific skills

```bash
npx skills add cezar-r/hyperliquid-skills --skill hl-market-data
npx skills add cezar-r/hyperliquid-skills --skill hl-order-placement --skill hl-account-overview
```

### Install to specific agents

```bash
npx skills add cezar-r/hyperliquid-skills --agent claude-code
npx skills add cezar-r/hyperliquid-skills --agent cursor --agent claude-code
```

### Install everything at once

```bash
npx skills add cezar-r/hyperliquid-skills --all
```

### List available skills

```bash
npx skills add cezar-r/hyperliquid-skills --list
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `hyperliquid-api-docs` | Fetches official API documentation on-demand for 18+ topics |
| `hl-api-samples` | Ready-to-run bash/curl commands for all API endpoints |
| `hl-order-placement` | Order placement guide: limit, market, trigger, bracket, TWAP |
| `hl-account-overview` | Account state, positions, balances, and margin data structures |
| `hl-market-data` | Market data: prices, order book, candles, funding, WebSocket |
| `hl-hip3-markets` | HIP-3 builder-deployed perpetuals (`dex:ticker` format, collateral) |
| `hl-spot-markets` | Spot markets (`@index` format, balance matching, wrapped tokens) |

## Problem → Skill

| I need to... | Start with |
|--------------|------------|
| Look up API docs for a specific topic | `hyperliquid-api-docs` |
| Get a quick curl command to test an endpoint | `hl-api-samples` |
| Place an order (limit, market, TP/SL, bracket) | `hl-order-placement` |
| Read account positions, balances, or margin | `hl-account-overview` |
| Fetch prices, order book, candles, or funding | `hl-market-data` |
| Work with HIP-3 builder-deployed markets | `hl-hip3-markets` |
| Work with spot markets or token balances | `hl-spot-markets` |
| Understand `@107` format or `baseTokenIndex` | `hl-spot-markets` |
| Understand `xyz:NVDA` format or dex collateral | `hl-hip3-markets` |
| Subscribe to real-time WebSocket data | `hl-market-data` |

## Skill Details

### hyperliquid-api-docs

Fetches official Hyperliquid API documentation on-demand. Maps 18 API topics to their docs URLs including:
- API overview, notation, asset IDs, tick sizes, nonces
- Info endpoints (perpetuals & spot)
- Exchange endpoint, WebSocket (subscriptions, post-requests, timeouts)
- Error responses, signing, rate limits
- Activation gas fee, Bridge2, HIP-3

### hl-api-samples

Generates ready-to-run bash/curl commands for common API operations:
- **Public data**: meta, allMids, l2Book, recentTrades, candleSnapshot
- **User data**: clearinghouseState, openOrders, userFills, userFunding
- **HIP-3 & spot**: dex-specific metadata, spot candles with `@index`

### hl-order-placement

Comprehensive guide for placing orders:
- Asset indexing (coin name → asset index mapping)
- Order types: limit (Gtc/Ioc/Alo), market, trigger (TP/SL), bracket, TWAP
- HIP-3 builder-deployed perpetuals (`dex:ticker` format)
- Price/size formatting, minimum order value ($10)

### hl-account-overview

Query and understand account data:
- Clearinghouse state (perp positions, margin summary)
- Spot clearinghouse state (token balances)
- Response structures: marginSummary, assetPositions, withdrawable
- Balance matching with `baseTokenIndex`
- Rate limits and polling tiers

### hl-market-data

Fetch market data from Hyperliquid:
- REST: allMids, l2Book, recentTrades, candleSnapshot, metaAndAssetCtxs
- WebSocket: allMids, l2Book, trades, candle subscriptions
- Candle intervals: 1m, 5m, 15m, 1h, 4h, 1d
- HIP-3 and spot market specifics

### hl-hip3-markets

Guide for HIP-3 builder-deployed perpetuals:
- Active dexes: xyz (USDC), flx (USDH), vntl (USDH), hyna (USDE), km (USDH)
- Ticker format (`dex:ticker` like `xyz:NVDA`)
- API parameters, response format, lookup key construction
- Isolated margin only, 2x fee structure

### hl-spot-markets

Guide for spot market handling:
- Three identifier types: universe index, base token index, market name
- The PURR special case (human-readable name)
- Wrapped token mapping: UBTC→BTC, USOL→SOL, UETH→ETH
- Balance matching with `baseTokenIndex` (not `index`)
- Common mistakes to avoid

## License

[MIT](LICENSE)
