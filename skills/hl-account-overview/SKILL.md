---
name: hl-account-overview
description: Query account state and understand position/balance data structures on Hyperliquid. Use when user asks about account data, positions, margin, balances, or clearinghouse state.
allowed-tools: Read
---

# Hyperliquid Account Overview

This skill provides guidance for querying and understanding account data on Hyperliquid, including perpetual positions, spot balances, and margin information.

## Key Endpoints

### Tier 1: Critical Data (Poll every 5 seconds)

| Endpoint | Purpose | Request |
|----------|---------|---------|
| `clearinghouseState` | Perp positions + margin summary | `{ type: "clearinghouseState", user, dex? }` |
| `spotClearinghouseState` | Spot token balances | `{ type: "spotClearinghouseState", user }` |
| `frontendOpenOrders` | Open orders with TP/SL info | `{ type: "frontendOpenOrders", user, dex? }` |

### Tier 2: Staking/Vault Data (On-demand, 5min cache recommended)

| Endpoint | Purpose | Request |
|----------|---------|---------|
| `delegations` | User's staked amounts per validator | `{ type: "delegations", user }` |
| `delegatorSummary` | Delegated/undelegated totals | `{ type: "delegatorSummary", user }` |
| `userVaultEquities` | User's vault positions | `{ type: "userVaultEquities", user }` |

### Tier 3: Historical Data (On-demand)

| Endpoint | Purpose | Request |
|----------|---------|---------|
| `userFills` | Trade history (max 2000) | `{ type: "userFills", user, startTime? }` |
| `userFunding` | Funding payments | `{ type: "userFunding", user, startTime }` |
| `userNonFundingLedgerUpdates` | Deposits/withdrawals | `{ type: "userNonFundingLedgerUpdates", user, startTime }` |

---

## Clearinghouse State Response

### Request

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq
```

### Response Structure

```json
{
  "marginSummary": {
    "accountValue": "10523.45",
    "totalMarginUsed": "2105.67",
    "totalNtlPos": "8420.12",
    "totalRawUsd": "2103.33"
  },
  "crossMarginSummary": {
    "accountValue": "8000.00",
    "totalMarginUsed": "1500.00",
    "totalNtlPos": "6000.00",
    "totalRawUsd": "2000.00"
  },
  "assetPositions": [
    {
      "position": {
        "coin": "BTC",
        "szi": "0.05",
        "entryPx": "105000.0",
        "positionValue": "5250.00",
        "unrealizedPnl": "125.00",
        "leverage": {
          "type": "cross",
          "value": 10
        },
        "liquidationPx": "95000.0",
        "marginUsed": "525.00",
        "returnOnEquity": "0.0238"
      },
      "type": "oneWay"
    }
  ],
  "withdrawable": "5000.00"
}
```

### Key Fields

| Field | Description |
|-------|-------------|
| `marginSummary.accountValue` | Total account value (margin + unrealized PnL) |
| `marginSummary.totalMarginUsed` | Margin locked in positions |
| `marginSummary.totalNtlPos` | Total notional position value |
| `marginSummary.totalRawUsd` | USDC balance (excluding unrealized PnL) |
| `withdrawable` | Amount available for withdrawal |

### Position Fields

| Field | Description |
|-------|-------------|
| `coin` | Asset symbol (e.g., "BTC", "ETH") |
| `szi` | Signed size (positive = long, negative = short) |
| `entryPx` | Average entry price |
| `positionValue` | Current position value |
| `unrealizedPnl` | Unrealized profit/loss |
| `leverage.type` | "cross" or "isolated" |
| `leverage.value` | Leverage multiplier |
| `liquidationPx` | Liquidation price |
| `marginUsed` | Margin allocated to this position |

---

## Spot Clearinghouse State

### Request

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotClearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq
```

### Response Structure

```json
{
  "balances": [
    {
      "coin": "HYPE",
      "token": 73,
      "total": "1500.5",
      "hold": "100.0",
      "entryNtl": "37512.50"
    },
    {
      "coin": "USDC",
      "token": 1,
      "total": "5000.00",
      "hold": "0",
      "entryNtl": "5000.00"
    }
  ]
}
```

### Spot Balance Fields

| Field | Description |
|-------|-------------|
| `coin` | Token name (e.g., "HYPE", "USDC") |
| `token` | Base token index (for market matching) |
| `total` | Total balance |
| `hold` | Amount locked in open orders |
| `entryNtl` | Entry notional (cost basis in USD) |

---

## Balance Matching with Spot Markets

**CRITICAL**: Use `baseTokenIndex` to match balances to markets:

```
SpotBalance.token (73) → SpotMarket.baseTokenIndex (73) → HYPE/USDC market
```

**NOT** `SpotMarket.index` (universe index) which is different!

### Wrapped Token Mapping

Some spot tokens are wrapped versions:

| API Token | Display As |
|-----------|------------|
| `UBTC` | `BTC` |
| `USOL` | `SOL` |
| `UETH` | `ETH` |

---

## HIP-3 Multi-Dex Positions

For HIP-3 builder-deployed markets, query each dex separately:

```bash
# Native Hyperliquid positions
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq

# xyz dex positions
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS", "dex": "xyz"}' | jq
```

### HIP-3 Position Format

Positions from HIP-3 dexes include the dex prefix:

```json
{
  "coin": "xyz:NVDA",
  "szi": "10.5",
  "entryPx": "142.50",
  "leverage": {
    "type": "isolated",
    "value": 5
  }
}
```

### Collateral by Dex

| Dex | Collateral |
|-----|------------|
| *(native)* | USDC |
| `xyz` | USDC |
| `flx` | USDH |
| `vntl` | USDH |
| `hyna` | USDE |
| `km` | USDH |

---

## Rate Limits

| Weight | Endpoints |
|--------|-----------|
| 2 | `clearinghouseState`, `spotClearinghouseState`, `frontendOpenOrders` |
| 20 | Most other info requests |
| Variable | `userFills`, `userFunding` (+1 weight per 20 items) |

**Aggregate limit**: 1200 weight per minute per IP

### Polling Recommendations

| Data Type | Frequency |
|-----------|-----------|
| Positions, orders, balances | Every 5 seconds |
| Staking data | On-demand, cache 5 minutes |
| Historical data (fills, funding) | On-demand |

---

## Open Orders

### Request

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "frontendOpenOrders", "user": "0xYOUR_ADDRESS"}' | jq
```

### Response Structure

```json
[
  {
    "coin": "BTC",
    "oid": 12345678,
    "side": "B",
    "limitPx": "100000.0",
    "sz": "0.001",
    "origSz": "0.001",
    "timestamp": 1704067200000,
    "orderType": "Limit",
    "tif": "Gtc",
    "reduceOnly": false,
    "triggerPx": null,
    "triggerCondition": null,
    "children": []
  }
]
```

### Order Fields

| Field | Description |
|-------|-------------|
| `oid` | Order ID (use for modifications/cancellations) |
| `side` | "B" for buy, "A" for sell (ask) |
| `limitPx` | Limit price |
| `sz` | Remaining size |
| `origSz` | Original size |
| `orderType` | "Limit", "Stop Market", "Take Profit", etc. |
| `tif` | Time-in-force: "Gtc", "Ioc", "Alo" |
| `triggerPx` | Trigger price (for TP/SL orders) |
| `children` | Linked TP/SL orders |

---

## User Fills (Trade History)

### Request

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "userFills", "user": "0xYOUR_ADDRESS", "startTime": 1704067200000}' | jq
```

### Response Structure

```json
[
  {
    "coin": "BTC",
    "px": "105234.5",
    "sz": "0.001",
    "side": "B",
    "time": 1704067200000,
    "startPosition": "0.004",
    "dir": "Open Long",
    "closedPnl": "0",
    "hash": "0x...",
    "oid": 12345678,
    "crossed": true,
    "fee": "0.52"
  }
]
```

---

## Quick Reference

### Get Full Account State

```bash
# All positions and margin
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq

# Spot balances
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotClearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq

# Open orders with TP/SL
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "frontendOpenOrders", "user": "0xYOUR_ADDRESS"}' | jq
```

### Common Calculations

**Available balance**: `marginSummary.totalRawUsd - marginSummary.totalMarginUsed`

**Position PnL**: `(currentPrice - entryPx) * szi` (for longs)

**Margin ratio**: `marginSummary.totalMarginUsed / marginSummary.accountValue`
