---
name: hl-market-data
description: Fetch market data from Hyperliquid including prices, order book, candles, funding rates, and asset contexts. Use when user asks about market data, price feeds, charts, or real-time data subscriptions.
allowed-tools: Read
---

# Hyperliquid Market Data Guide

This skill provides guidance for fetching market data from Hyperliquid, including prices, order books, candles, and real-time WebSocket subscriptions.

## REST Endpoints

### All Mid Prices

Get current mid prices for all assets:

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "allMids"}' | jq
```

**Response**:
```json
{
  "BTC": "105234.5",
  "ETH": "3456.78",
  "SOL": "181.25",
  "@107": "28.45"
}
```

Note: Spot markets use `@{index}` format (e.g., `@107` for HYPE/USDC).

---

### Order Book (L2)

Get order book depth for a specific asset:

```bash
# BTC order book
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "l2Book", "coin": "BTC"}' | jq
```

**Response**:
```json
{
  "coin": "BTC",
  "levels": [
    [
      {"px": "105230.0", "sz": "1.5", "n": 3},
      {"px": "105225.0", "sz": "2.1", "n": 5}
    ],
    [
      {"px": "105235.0", "sz": "0.8", "n": 2},
      {"px": "105240.0", "sz": "1.2", "n": 4}
    ]
  ],
  "time": 1704067200000
}
```

- `levels[0]` = bids (buy orders)
- `levels[1]` = asks (sell orders)
- `n` = number of orders at that price

---

### Recent Trades

Get recent trades for an asset:

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "recentTrades", "coin": "BTC"}' | jq
```

**Response**:
```json
[
  {
    "coin": "BTC",
    "side": "B",
    "px": "105234.5",
    "sz": "0.1",
    "time": 1704067200000,
    "hash": "0x..."
  }
]
```

---

### Candle Data (OHLCV)

Get historical candlestick data:

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "BTC", "interval": "1h", "startTime": 1704067200000, "endTime": 1704153600000}}' | jq
```

**Response**:
```json
[
  {
    "t": 1704067200000,
    "T": 1704070800000,
    "s": "BTC",
    "i": "1h",
    "o": "105000.0",
    "c": "105234.5",
    "h": "105500.0",
    "l": "104800.0",
    "v": "1234.5",
    "n": 5678
  }
]
```

### Candle Intervals

| Interval | Description |
|----------|-------------|
| `1m` | 1 minute |
| `5m` | 5 minutes |
| `15m` | 15 minutes |
| `1h` | 1 hour |
| `4h` | 4 hours |
| `1d` | 1 day |

### Candle Fields

| Field | Description |
|-------|-------------|
| `t` | Open timestamp (ms) |
| `T` | Close timestamp (ms) |
| `o` | Open price |
| `h` | High price |
| `l` | Low price |
| `c` | Close price |
| `v` | Volume |
| `n` | Number of trades |

---

### Meta and Asset Contexts

Get comprehensive market metadata with 24h stats:

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "metaAndAssetCtxs"}' | jq
```

**Response**:
```json
[
  {
    "universe": [
      {"name": "BTC", "szDecimals": 5, "maxLeverage": 50}
    ]
  },
  [
    {
      "funding": "0.0001",
      "openInterest": "12345.67",
      "prevDayPx": "104000.0",
      "dayNtlVlm": "1234567890.0",
      "premium": "0.0002",
      "oraclePx": "105230.0",
      "markPx": "105234.5",
      "midPx": "105234.5",
      "impactPxs": ["105230.0", "105240.0"]
    }
  ]
]
```

### Asset Context Fields

| Field | Description |
|-------|-------------|
| `funding` | Current funding rate |
| `openInterest` | Total open interest |
| `prevDayPx` | Previous day close price (for 24h change) |
| `dayNtlVlm` | 24h notional volume |
| `oraclePx` | Oracle price |
| `markPx` | Mark price |
| `midPx` | Mid price |
| `impactPxs` | Impact prices [bid, ask] |

---

### Historical Funding Rates

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "fundingHistory", "coin": "BTC", "startTime": 1704067200000}' | jq
```

---

## WebSocket Subscriptions

Connect to: `wss://api.hyperliquid.xyz/ws`

### All Mids (Real-time Prices)

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "allMids"
  }
}
```

### Order Book Updates

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "l2Book",
    "coin": "BTC"
  }
}
```

### Trade Stream

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "trades",
    "coin": "BTC"
  }
}
```

### Candle Updates

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "candle",
    "coin": "BTC",
    "interval": "1h"
  }
}
```

**Key format**: `coin|interval` (e.g., `BTC|1h`, `xyz:NVDA|15m`)

---

## HIP-3 Markets

For HIP-3 builder-deployed markets, add the `dex` parameter:

### REST Requests

```bash
# HIP-3 market metadata
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "meta", "dex": "xyz"}' | jq

# HIP-3 asset contexts
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "metaAndAssetCtxs", "dex": "xyz"}' | jq
```

### WebSocket Subscriptions

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "allMids",
    "dex": "xyz"
  }
}
```

### Candles for HIP-3

Use the full `dex:ticker` format:

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "xyz:NVDA", "interval": "1h", "startTime": 1704067200000}}' | jq
```

WebSocket key format: `xyz:NVDA|1h`

---

## Spot Markets

Spot markets use different identifiers:

| Identifier | Example | Use For |
|------------|---------|---------|
| Market name | `HYPE/USDC` | Store keys, display |
| API format | `@107` | API calls, WebSocket |
| Base token index | `73` | Balance matching |

### Spot Candles

Use `@{index}` format (NOT the market name):

```bash
# HYPE/USDC candles (market index 107)
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "@107", "interval": "1h", "startTime": 1704067200000}}' | jq
```

### PURR Special Case

PURR (index 0) is the only spot market that uses a human-readable name in the API:

```bash
# PURR candles - use "PURR/USDC" not "@0"
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "PURR/USDC", "interval": "1h", "startTime": 1704067200000}}' | jq
```

### Wrapped Tokens

Some spot tokens are wrapped versions displayed differently:

| API Token | Display |
|-----------|---------|
| `UBTC` | `BTC` |
| `USOL` | `SOL` |
| `UETH` | `ETH` |

---

## Rate Limits

- **Aggregate limit**: 1200 weight per minute per IP
- **Weight 20**: Most market data requests
- **Variable weight**: Historical data (+1 weight per 20 items)

### Polling Recommendations

| Data Type | Frequency |
|-----------|-----------|
| Prices (allMids) | WebSocket preferred |
| Order book | WebSocket preferred |
| Candles | WebSocket for updates, REST for history |
| Asset contexts | REST every 5-30 seconds |

---

## Quick Reference

### Get All Market Data

```bash
# Current prices
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "allMids"}' | jq

# Market metadata + 24h stats
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "metaAndAssetCtxs"}' | jq

# Order book
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "l2Book", "coin": "BTC"}' | jq

# Recent trades
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "recentTrades", "coin": "BTC"}' | jq
```

### Timestamp Helpers

```bash
# Current timestamp in milliseconds
date +%s000

# 24 hours ago
echo $(($(date +%s) - 86400))000

# 7 days ago
echo $(($(date +%s) - 604800))000
```

### Common Calculations

**24h Price Change**: `(currentPrice - prevDayPx) / prevDayPx * 100`

**Spread**: `askPrice - bidPrice`

**Spread %**: `(askPrice - bidPrice) / midPrice * 100`
