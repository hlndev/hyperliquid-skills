---
name: hl-spot-markets
description: Guide for spot markets on Hyperliquid. Use when user asks about spot trading, token balances, spot tickers, wrapped tokens (UBTC, USOL), or the @index format. Covers market identifiers, balance matching, and common mistakes.
allowed-tools: Read
---

# Hyperliquid Spot Markets Guide

Spot markets on Hyperliquid enable direct token-to-token trading (e.g., HYPE/USDC). Unlike perpetual markets, spot markets have no leverage and trade actual tokens.

## Overview

Spot markets differ from perpetuals:

- No leverage or margin requirements
- Trade actual tokens (not contracts)
- Use `@{index}` format in API responses
- Different identifiers for storage, API calls, and balance matching
- Support multiple quote tokens (USDC, USDH)

---

## Market Identifiers

**CRITICAL**: Spot markets have THREE different identifiers. Using the wrong one causes wrong data or no data.

| Identifier | Field | Example | Use For |
|------------|-------|---------|---------|
| Universe Index | `market.index` | `107` | API calls (`@107`) |
| Base Token Index | `market.tokens[0]` | `73` | Matching `SpotBalance.token` |
| Market Name | `market.name` | `HYPE/USDC` | Store keys, display |
| API Name | `market.apiName` | `@107` or `PURR/USDC` | API subscriptions |

### Identifier Summary

| Market | `name` | `index` | `apiName` | `baseTokenIndex` |
|--------|--------|---------|-----------|------------------|
| PURR | PURR/USDC | 0 | PURR/USDC | 0 |
| HYPE | HYPE/USDC | 107 | @107 | 73 |
| BTC (wrapped) | BTC/USDC | 101 | @101 | 51 |

---

## The PURR Special Case

PURR (index 0) is the **only** spot market where the API returns a human-readable name instead of `@{index}` format:

| Market | API Returns |
|--------|-------------|
| PURR | `PURR/USDC` |
| HYPE | `@107` |
| BTC | `@101` |

Always use `apiName` which stores whatever the API returns.

---

## Wrapped Tokens

Some spot tokens are wrapped versions of native assets. Map these for display:

| API Token | Display As | Notes |
|-----------|------------|-------|
| `UBTC` | `BTC` | Wrapped Bitcoin |
| `USOL` | `SOL` | Wrapped Solana |
| `UETH` | `ETH` | Wrapped Ethereum |

```typescript
const SPOT_TICKER_MAP = {
  'UBTC': 'BTC',
  'USOL': 'SOL',
  'UETH': 'ETH',
};
```

---

## API Endpoints

### Get Spot Market Metadata

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotMeta"}' | jq
```

**Response**:
```json
{
  "universe": [
    {
      "name": "PURR/USDC",
      "index": 0,
      "tokens": [0, 1]
    },
    {
      "name": "@107",
      "index": 107,
      "tokens": [73, 1]
    }
  ],
  "tokens": [
    { "index": 0, "name": "PURR" },
    { "index": 1, "name": "USDC" },
    { "index": 73, "name": "HYPE" }
  ]
}
```

### Get Spot Asset Contexts

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotMetaAndAssetCtxs"}' | jq
```

### Get Spot Balances

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotClearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq
```

**Response**:
```json
{
  "balances": [
    {
      "coin": "HYPE",
      "token": 73,
      "total": "100.5",
      "hold": "0",
      "entryNtl": "2512.50"
    }
  ]
}
```

### Balance Fields

| Field | Description |
|-------|-------------|
| `coin` | Token name (e.g., "HYPE") |
| `token` | Base token index (for market matching) |
| `total` | Total balance |
| `hold` | Amount locked in open orders |
| `entryNtl` | Entry notional (cost basis in USD) |

---

## Price Lookups

### allMids WebSocket Data

```json
{
  "mids": {
    "BTC": "105234.5",
    "@107": "28.45",
    "@101": "104500.0"
  }
}
```

Prices for spot markets are keyed by `@{index}` format. Convert to market name for storage:

```typescript
// WebSocket returns: "@107" -> "28.45"
// Convert to: "HYPE/USDC" -> "28.45"

if (coin.startsWith('@')) {
  const spotIndex = parseInt(coin.substring(1), 10);
  const spotMarket = spotMarkets.find(m => m.index === spotIndex);
  if (spotMarket) {
    prices[spotMarket.name] = price;  // prices["HYPE/USDC"] = "28.45"
  }
}
```

---

## Balance Matching

**CRITICAL**: Use `baseTokenIndex` (NOT `index`) to match balances to markets.

```typescript
// CORRECT - match by baseTokenIndex
const market = spotMarkets.find(m => m.baseTokenIndex === balance.token);
// balance.token = 73 → market.baseTokenIndex = 73 → HYPE/USDC

// WRONG - using universe index
const market = spotMarkets.find(m => m.index === balance.token);
// balance.token = 73 ≠ market.index = 107 → no match!
```

**Why are they different?**
- `market.index` (107) = **universe index** - used for API calls
- `balance.token` (73) = **base token index** - identifies the token

---

## Candle Data

### REST API

Use `@{index}` format (NOT the market name):

```bash
# HYPE/USDC candles (market index 107)
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "@107", "interval": "1h", "startTime": 1704067200000}}' | jq
```

### PURR Exception

Use `PURR/USDC` for PURR candles:

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "PURR/USDC", "interval": "1h", "startTime": 1704067200000}}' | jq
```

### WebSocket Subscription

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "candle",
    "coin": "@107",
    "interval": "1h"
  }
}
```

---

## Comparison: Spot vs Perpetual

| Aspect | Perpetual | Spot |
|--------|-----------|------|
| Price key format | `"BTC"` or `"xyz:NVDA"` | `"HYPE/USDC"` |
| API coin format | Ticker name | `@{index}` or market name (PURR) |
| Leverage | Up to 50x | None (1x only) |
| Margin | USDC/USDH/USDE | N/A (direct token) |
| Balance type | Position (szi) | Token holding (total) |
| Funding | Yes | No |

---

## When to Use Each Identifier

| Scenario | Use | Example |
|----------|-----|---------|
| Store lookups (prices, contexts) | `market.name` | `"HYPE/USDC"` |
| API candle/sparkline calls | `market.apiName` | `"@107"` or `"PURR/USDC"` |
| Matching user balance | `market.baseTokenIndex` | `73` matches `balance.token` |
| Display to user | `getDisplayTicker(name)` | `"BTC/USDC"` (not `"UBTC/USDC"`) |
| Finding market by universe index | `market.index` | `107` |

---

## Common Mistakes

### 1. Using Wrong Index for API Calls

```bash
# WRONG - tokens[0] is baseTokenIndex, NOT universe index
curl ... -d '{"type": "candleSnapshot", "req": {"coin": "@73", ...}}'  # WRONG!

# RIGHT - use index (universe index) or apiName
curl ... -d '{"type": "candleSnapshot", "req": {"coin": "@107", ...}}'  # CORRECT
```

### 2. Looking Up Price with Wrong Key

```typescript
// WRONG - API format doesn't match store keys
const price = prices["@107"];            // undefined
const price = prices["HYPE"];            // undefined

// RIGHT - store uses market name
const price = prices["HYPE/USDC"];       // "28.45"
```

### 3. Matching Balance to Wrong Market

```typescript
// WRONG - using universe index
const market = spotMarkets.find(m => m.index === balance.token);

// RIGHT - use baseTokenIndex
const market = spotMarkets.find(m => m.baseTokenIndex === balance.token);
// or
const market = spotMarkets.find(m => m.tokens[0] === balance.token);
```

### 4. Forgetting PURR Special Case

```typescript
// WRONG - assumes all spot markets use @{index}
const apiCoin = `@${market.index}`;      // "@0" for PURR - WRONG!

// RIGHT - use apiName which handles the special case
const apiCoin = market.apiName;          // "PURR/USDC" for PURR, "@107" for others
```

### 5. Displaying Wrapped Tokens Without Mapping

```typescript
// WRONG - shows internal token name
displayTicker = market.name;             // "UBTC/USDC" - confusing

// RIGHT - map wrapped tokens
const SPOT_TICKER_MAP = { 'UBTC': 'BTC', 'USOL': 'SOL', 'UETH': 'ETH' };
const baseTicker = market.name.split('/')[0];
displayTicker = `${SPOT_TICKER_MAP[baseTicker] || baseTicker}/${quote}`;  // "BTC/USDC"
```

### 6. Using Market Name in WebSocket Subscription

```bash
# WRONG - API doesn't understand market name format
{"type": "candle", "coin": "HYPE/USDC", "interval": "1h"}

# RIGHT - use @{index} format
{"type": "candle", "coin": "@107", "interval": "1h"}
```

---

## Quick Reference

### Get Spot Data

```bash
# Market metadata
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotMeta"}' | jq

# User balances
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotClearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq

# Candles (use @index)
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "@107", "interval": "1h", "startTime": 1704067200000}}' | jq
```

### Index Cheat Sheet

| Purpose | Field | Example |
|---------|-------|---------|
| API calls | `market.index` or `market.apiName` | `@107` |
| Balance matching | `market.baseTokenIndex` or `market.tokens[0]` | `73` |
| Store keys | `market.name` | `HYPE/USDC` |
