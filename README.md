# Hyperliquid Skills

A collection of Claude Code skills for working with the Hyperliquid API.

## Installation

Install via the Claude Code plugin marketplace:

```bash
/plugin marketplace add cezar-r/hyperliquid-skills
```

Or install individual skills:

```bash
/plugin install cezar-r/hyperliquid-skills@hyperliquid-api-docs
/plugin install cezar-r/hyperliquid-skills@hl-api-samples
/plugin install cezar-r/hyperliquid-skills@hl-order-placement
/plugin install cezar-r/hyperliquid-skills@hl-account-overview
/plugin install cezar-r/hyperliquid-skills@hl-market-data
/plugin install cezar-r/hyperliquid-skills@hl-hip3-markets
/plugin install cezar-r/hyperliquid-skills@hl-spot-markets
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `hyperliquid-api-docs` | Fetch official Hyperliquid API documentation on-demand |
| `hl-api-samples` | Generate ready-to-run bash/curl commands for the Hyperliquid API |
| `hl-order-placement` | Guide for placing orders on Hyperliquid (limit, market, bracket, TWAP) |
| `hl-account-overview` | Query account state and understand position/balance data structures |
| `hl-market-data` | Fetch market data (prices, order book, candles, funding rates) |
| `hl-hip3-markets` | Guide for HIP-3 builder-deployed perpetuals (dex:ticker format, collateral) |
| `hl-spot-markets` | Guide for spot markets (@index format, balance matching, wrapped tokens) |

## Skill Details

### hyperliquid-api-docs

Fetches official Hyperliquid API documentation on-demand. Maps 18 API topics to their docs URLs including:
- API overview, notation, asset IDs, tick sizes, nonces
- Info endpoints (perpetuals & spot)
- Exchange endpoint
- WebSocket (subscriptions, post-requests, timeouts)
- Error responses, signing, rate limits
- Activation gas fee, Bridge2, HIP-3

### hl-api-samples

Generates ready-to-run bash/curl commands for common API operations:
- **Public Data**: meta, allMids, l2Book, recentTrades, candleSnapshot
- **User Data**: clearinghouseState, openOrders, userFills, userFunding

### hl-order-placement

Comprehensive guide for placing orders:
- Asset indexing (coin name → asset index mapping)
- Minimum order value ($10)
- Order types: limit, market, trigger (TP/SL), bracket, TWAP
- HIP-3 builder-deployed perpetuals (`dex:ticker` format)
- Price and size formatting

### hl-account-overview

Query and understand account data:
- Clearinghouse state (perp positions, margin summary)
- Spot clearinghouse state (token balances)
- Response structure (marginSummary, assetPositions)
- Balance matching with baseTokenIndex
- Rate limits and polling tiers

### hl-market-data

Fetch market data from Hyperliquid:
- All mid prices, order book, recent trades
- OHLCV candle data (1m, 5m, 15m, 1h, 4h, 1d)
- Asset contexts (funding, OI, 24h stats)
- WebSocket subscriptions
- HIP-3 and spot market specifics

### hl-hip3-markets

Comprehensive guide for HIP-3 builder-deployed perpetuals:
- Dex identifiers (xyz, flx, vntl, hyna, km)
- Ticker format (`dex:ticker` like `xyz:NVDA`)
- Collateral tokens per dex (USDC, USDH, USDE)
- API parameters and response format
- Isolated margin requirements
- Common mistakes and lookup key construction

### hl-spot-markets

Guide for spot market handling:
- Three identifier types (universe index, base token index, market name)
- The PURR special case
- Wrapped token mapping (UBTC→BTC, USOL→SOL, UETH→ETH)
- Balance matching with baseTokenIndex
- Candle subscriptions with @index format
- Common mistakes to avoid
