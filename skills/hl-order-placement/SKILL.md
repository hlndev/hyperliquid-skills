---
name: hl-order-placement
description: Guide for placing orders on Hyperliquid. Use when user asks about order placement, order types, bracket orders, TWAP orders, or HIP-3 trading. Covers asset indexing, price/size formatting, and order type configurations.
allowed-tools: Read
---

# Hyperliquid Order Placement Guide

This skill provides comprehensive guidance for placing orders on Hyperliquid, including native perps and HIP-3 builder-deployed markets.

## Key Concepts

### Asset Indexing

Use the `meta` endpoint to map coin names to asset indices:

| Coin | Index | Notes |
|------|-------|-------|
| BTC | 0 | Bitcoin perpetual |
| ETH | 1 | Ethereum perpetual |
| SOL | 5 | Solana perpetual |
| HYPE | 132 | Hyperliquid token |

**Always fetch `meta` to get current indices** - they may change as new assets are listed.

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "meta"}' | jq '.universe[] | {name, index: .szDecimals}'
```

### Minimum Order Value

**$10 minimum** - All orders must have `size * price >= $10`

### Price Format

- Prices are **strings** (e.g., `"181.5"`, `"105234.0"`)
- Must respect tick sizes from `meta` endpoint
- Set to `"0"` for market orders

### Size Format

- Sizes are **strings** (e.g., `"0.1"` for 0.1 BTC, `"4.96"` for 4.96 SOL)
- Must respect lot sizes from `meta` endpoint

---

## Order Types

### 1. Limit Order (Good-til-Canceled)

```json
{
  "asset": 0,
  "isBuy": true,
  "size": "0.001",
  "price": "100000.0",
  "orderType": {
    "limit": {
      "tif": "Gtc"
    }
  },
  "reduceOnly": false
}
```

**Time-in-Force options**:
- `Gtc` - Good til canceled (default)
- `Ioc` - Immediate or cancel
- `Alo` - Add liquidity only (post-only)

### 2. Market Order

Set price to `"0"` to execute at best available price:

```json
{
  "asset": 0,
  "isBuy": true,
  "size": "0.001",
  "price": "0",
  "orderType": {
    "limit": {
      "tif": "Ioc"
    }
  },
  "reduceOnly": false
}
```

### 3. Trigger Orders (Take Profit / Stop Loss)

```json
{
  "asset": 0,
  "isBuy": false,
  "size": "0.001",
  "price": "0",
  "orderType": {
    "trigger": {
      "isMarket": true,
      "triggerPx": "110000.0",
      "tpsl": "tp"
    }
  },
  "reduceOnly": true
}
```

**Trigger parameters**:
- `isMarket`: `true` for market execution, `false` for limit
- `triggerPx`: Price at which the order triggers
- `tpsl`: `"tp"` for take profit, `"sl"` for stop loss

**Direction rules**:
- Long position TP: `isBuy: false`, trigger price above entry
- Long position SL: `isBuy: false`, trigger price below entry
- Short position TP: `isBuy: true`, trigger price below entry
- Short position SL: `isBuy: true`, trigger price above entry

### 4. Bracket Order (Entry + TP + SL)

A bracket order places entry, take profit, and stop loss in a single atomic batch:

```json
{
  "asset": 5,
  "isBuy": true,
  "size": "4.96",
  "entryPrice": "181.5",
  "takeProfitPrice": "200.0",
  "stopLossPrice": "170.0",
  "entryOrderType": {
    "limit": {
      "tif": "Gtc"
    }
  },
  "reduceOnly": false
}
```

**Key points**:
- TP and SL are automatically set as `reduceOnly: true`
- TP and SL use trigger order types
- All three orders are placed atomically

### 5. TWAP Order (Time-Weighted Average Price)

Executes a large order over time to minimize market impact:

```json
{
  "coin": "SOL",
  "isBuy": true,
  "size": "100.0",
  "minutes": 30,
  "randomize": true,
  "reduceOnly": false
}
```

**TWAP parameters**:
- `minutes`: Duration (minimum 2 minutes)
- `randomize`: Randomize execution intervals to reduce predictability

---

## HIP-3 Builder-Deployed Markets

HIP-3 markets are perpetuals deployed by third-party builders. They have special requirements:

### Ticker Format

Use `dex:ticker` format:

| Market | Format |
|--------|--------|
| NVIDIA on xyz | `xyz:NVDA` |
| Apple on hyna | `hyna:AAPL` |
| Tesla on flx | `flx:TSLA` |

### Available Dexes

| Dex | Collateral |
|-----|------------|
| `xyz` | USDC |
| `flx` | USDH |
| `vntl` | USDH |
| `hyna` | USDE |
| `km` | USDH |

### HIP-3 Specifics

1. **Isolated margin only** - Cross margin not supported yet
2. **Higher fees** - 2x standard fees (50% goes to deployer)
3. **Dex parameter required** - Must specify `dex` in API calls

### API Calls for HIP-3

```bash
# Get HIP-3 market metadata
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "meta", "dex": "xyz"}' | jq

# Get HIP-3 positions
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS", "dex": "xyz"}' | jq
```

---

## Reduce-Only Orders

Set `reduceOnly: true` when:
- Closing an existing position
- Placing TP/SL orders
- You don't want to accidentally open a new position

```json
{
  "reduceOnly": true
}
```

---

## Order Modification

To modify an existing order, use the order ID (oid):

```json
{
  "oid": 12345678,
  "coin": "BTC",
  "isBuy": true,
  "size": "0.002",
  "price": "95000.0",
  "orderType": {
    "limit": {
      "tif": "Gtc"
    }
  },
  "reduceOnly": false
}
```

---

## Order Cancellation

Cancel by coin and order ID:

```json
{
  "coin": "BTC",
  "oid": 12345678
}
```

Cancel all orders:
```json
{
  "cancelAll": true
}
```

---

## Common Mistakes

1. **Using coin name instead of asset index** - Orders use numeric indices, not strings
2. **Forgetting minimum order value** - Must be >= $10
3. **Wrong trigger direction** - TP/SL direction depends on position side
4. **Missing dex parameter for HIP-3** - Always include `dex` for builder markets
5. **Using cross margin on HIP-3** - Only isolated margin is supported

---

## Quick Reference

| Order Type | Key Config |
|------------|------------|
| Limit GTC | `{ limit: { tif: "Gtc" }}` |
| Market | `price: "0"`, `{ limit: { tif: "Ioc" }}` |
| Take Profit | `{ trigger: { tpsl: "tp", ... }}` |
| Stop Loss | `{ trigger: { tpsl: "sl", ... }}` |
| Post-Only | `{ limit: { tif: "Alo" }}` |
