---
name: hl-hip3-markets
description: Guide for HIP-3 builder-deployed perpetuals on Hyperliquid. Use when user asks about HIP-3 markets, dex-prefixed tickers (xyz:NVDA), builder perps, or multi-dex trading. Covers ticker format, collateral tokens, API parameters, and common mistakes.
allowed-tools: Read
---

# HIP-3 Builder-Deployed Perpetuals Guide

HIP-3 (Hyperliquid Improvement Proposal 3) enables permissionless deployment of perpetual markets on Hyperliquid. Any builder can deploy new perp markets by staking 500k HYPE.

## Overview

HIP-3 markets operate alongside native Hyperliquid perps but with distinct characteristics:

- Different collateral tokens per dex
- Isolated margin only (cross margin coming later)
- Higher fees (2x standard, with 50% going to deployers)
- `dex:ticker` naming convention

---

## Dex Identifiers

Currently active HIP-3 dex names:

| Dex | Description | Collateral |
|-----|-------------|------------|
| `xyz` | First major HIP-3 deployer | USDC |
| `flx` | Felix DEX | USDH |
| `vntl` | Vental Markets | USDH |
| `hyna` | Hyena Finance | USDE |
| `km` | KM Markets | USDH |

**Native Hyperliquid** (no dex prefix) uses **USDC** collateral.

---

## Ticker Format

HIP-3 markets use a `dex:ticker` naming convention:

| Market | Native Perp | HIP-3 |
|--------|-------------|-------|
| NVIDIA | `NVDA` | `xyz:NVDA` |
| Tesla | `TSLA` | `xyz:TSLA` |
| Apple | `AAPL` | `hyna:AAPL` |

The API returns market names with the dex prefix already included.

---

## API Parameters

**CRITICAL**: Pass the `dex` parameter for all HIP-3 API calls. Without it, you only get native Hyperliquid perps.

### Market Metadata

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "meta", "dex": "xyz"}' | jq
```

### User Positions

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS", "dex": "xyz"}' | jq
```

### Open Orders

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "frontendOpenOrders", "user": "0xYOUR_ADDRESS", "dex": "xyz"}' | jq
```

### Asset Contexts (prices, volume, etc.)

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "metaAndAssetCtxs", "dex": "xyz"}' | jq
```

### Price Subscriptions (WebSocket)

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "allMids",
    "dex": "xyz"
  }
}
```

---

## Response Format

API responses include the dex prefix in market names:

### Meta Response

```json
{
  "universe": [
    {
      "name": "xyz:NVDA",
      "szDecimals": 2,
      "maxLeverage": 20,
      "onlyIsolated": true
    }
  ]
}
```

### Position Response

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

---

## Margin Mode

HIP-3 markets currently **only support isolated margin**. The `onlyIsolated` field will be `true` for all HIP-3 markets.

Cross margin support is planned for a future update.

---

## Fee Structure

| Aspect | Native Perps | HIP-3 Perps |
|--------|--------------|-------------|
| Fee Rate | Standard | 2x standard |
| Fee Split | 100% Hyperliquid | 50% HL, 50% deployer |
| Deployer Stake | N/A | 500,000 HYPE |

---

## Lookup Key Construction

**CRITICAL**: Always construct a lookup key when accessing HIP-3 market data:

```typescript
const selectedTicker = "NVDA";
const selectedDex = "xyz";

// Construct lookup key for store/API access
const lookupKey = selectedDex
  ? `${selectedDex}:${selectedTicker}`
  : selectedTicker;

// Use lookupKey for ALL market data lookups
const price = prices[lookupKey];           // prices["xyz:NVDA"]
const context = assetContexts[lookupKey];  // assetContexts["xyz:NVDA"]
```

### When to Use Combined Key vs Separated

| Scenario | Use | Example |
|----------|-----|---------|
| Store lookups (prices, contexts) | Combined `lookupKey` | `prices["xyz:NVDA"]` |
| API subscriptions | Combined `lookupKey` | `candle({ coin: "xyz:NVDA" })` |
| UI ticker display | Ticker only | `"NVDA"` |
| UI dex badge | Dex only | `"XYZ"` (uppercase) |
| Collateral display | Collateral mapping | `"USDC"` |

---

## Candle Subscriptions

Use the full `dex:ticker` format for candle data:

### REST

```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "candleSnapshot", "req": {"coin": "xyz:NVDA", "interval": "1h", "startTime": 1704067200000}}' | jq
```

### WebSocket

```json
{
  "method": "subscribe",
  "subscription": {
    "type": "candle",
    "coin": "xyz:NVDA",
    "interval": "1h"
  }
}
```

Key format: `xyz:NVDA|1h`

---

## Fetching All Dex Data

To get complete account state, query each dex separately:

```bash
# Native positions
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS"}' | jq

# xyz positions
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS", "dex": "xyz"}' | jq

# flx positions
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xYOUR_ADDRESS", "dex": "flx"}' | jq

# ... repeat for vntl, hyna, km
```

---

## Comparison: Native Perp vs HIP-3 Perp

| Aspect | Native Perp | HIP-3 Perp |
|--------|-------------|------------|
| Ticker format | `"BTC"` | `"xyz:NVDA"` |
| Collateral | USDC only | Varies by dex |
| Margin modes | Cross + Isolated | Isolated only |
| Fee rate | Standard | 2x standard |
| API `dex` param | Not needed | Required |
| Market availability | Core assets | Deployer's choice |

---

## Common Mistakes

### 1. Forgetting the Lookup Key

```typescript
// WRONG - HIP-3 data won't be found
const price = prices[selectedTicker];           // prices["NVDA"] = undefined

// RIGHT - construct lookup key first
const lookupKey = selectedDex ? `${selectedDex}:${selectedTicker}` : selectedTicker;
const price = prices[lookupKey];                // prices["xyz:NVDA"] = "142.50"
```

### 2. Wrong Collateral Mapping

```typescript
// WRONG - dex names are NOT collateral tokens
const collateral = selectedDex || 'USDC';       // "xyz" is not a token

// RIGHT - use the collateral mapping
const HIP3_COLLATERAL = {
  '': 'USDC',
  xyz: 'USDC',
  flx: 'USDH',
  vntl: 'USDH',
  hyna: 'USDE',
  km: 'USDH',
};
const collateral = HIP3_COLLATERAL[selectedDex || ''];  // "USDC"
```

### 3. Missing Dex Parameter in API Calls

```typescript
// WRONG - only gets native perp data
const positions = await fetch('...', {
  body: JSON.stringify({ type: "clearinghouseState", user })
});

// RIGHT - include dex parameter
const positions = await fetch('...', {
  body: JSON.stringify({ type: "clearinghouseState", user, dex: "xyz" })
});
```

### 4. Double-Prefixing Market Names

```typescript
// WRONG - API already returns "xyz:NVDA"
const marketKey = `${dex}:${market.name}`;      // "xyz:xyz:NVDA"

// RIGHT - use market.name directly (already has prefix)
const marketKey = market.name;                   // "xyz:NVDA"
```

### 5. Assuming Cross Margin Works

```typescript
// WRONG - HIP-3 only supports isolated
const order = {
  leverage: { type: "cross", value: 10 }  // Will fail
};

// RIGHT - use isolated margin
const order = {
  leverage: { type: "isolated", value: 10 }
};
```

---

## Quick Reference

### Collateral Tokens

| Dex | Collateral |
|-----|------------|
| *(native)* | USDC |
| `xyz` | USDC |
| `flx` | USDH |
| `vntl` | USDH |
| `hyna` | USDE |
| `km` | USDH |

### API Dex Parameter

Add `"dex": "xyz"` (or other dex name) to all HIP-3 API requests.

### Ticker Format

- Native: `BTC`, `ETH`, `SOL`
- HIP-3: `xyz:NVDA`, `flx:TSLA`, `hyna:AAPL`
