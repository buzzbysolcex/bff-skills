---
name: token-scoring
description: 5-layer composite token scoring engine for exchange listing intelligence — evaluates safety, wallet forensics, technical indicators, social sentiment, and market metrics with dual-gate verification.
author: buzzbysolcex
author_agent: Ionic Nova (Buzz BD Agent)
user-invocable: true
arguments: doctor | run | run --address <contract> --chain <chain>
entry: token-scoring/token-scoring.ts
requires: [wallet, settings]
tags: [read-only, infrastructure, l2]
---

# Token Scoring

## What it does
Scores any token across Solana, Base, or BSC on a 0-100 composite scale using 5 independent data layers: contract safety (RugCheck), wallet forensics (Helius/on-chain), technical analysis (RSI/MACD/volume), social sentiment (Serper/search), and market metrics (DexScreener/CoinGecko). Applies dual-gate verification where fundamentals and market scores must each independently clear 60% of their maximum before a token can advance. Auto-classifies tokens as HOT (85+), QUALIFIED (70-84), WATCH (50-69), or SKIP (<50).

## Why agents need it
Any agent evaluating tokens for trading, listing, or portfolio decisions needs systematic due diligence beyond price action. This skill replaces narrative-driven evaluation with data-backed composite scoring. The dual-gate prevents tokens with inflated social metrics but poor fundamentals from passing, and vice versa. Agents can chain this with trading skills to filter candidates before execution.

## Safety notes
- This skill is READ-ONLY. It never writes to chain or moves funds.
- All data is pulled from public APIs (DexScreener, CoinGecko, RugCheck).
- No wallet funds required for execution.
- Scores are informational — not financial advice.

## Commands

### doctor
Checks API connectivity to all 5 data sources, verifies wallet is configured (for agent identity only), and reports readiness.
```bash
bun run token-scoring/token-scoring.ts doctor
```

### run
Scores a token by contract address. Pulls data from all 5 layers, computes composite score, applies dual-gate, and returns classification.
```bash
bun run token-scoring/token-scoring.ts run --address <contract_address> --chain <solana|base|bsc>
```

Without arguments, scores the top trending token from DexScreener:
```bash
bun run token-scoring/token-scoring.ts run
```

## Output contract
All outputs are JSON to stdout.

```json
{
  "status": "success | error | blocked",
  "action": "review score and classification for trading/listing decision",
  "data": {
    "address": "contract address",
    "chain": "solana",
    "composite_score": 85,
    "classification": "hot",
    "dual_gate": {
      "pass": true,
      "fundamentals": { "score": 48, "max": 70, "threshold": 42, "pass": true },
      "market": { "score": 22, "max": 30, "threshold": 18, "pass": true }
    },
    "components": {
      "safety": { "score": 20, "max": 25 },
      "wallet": { "score": 18, "max": 25 },
      "technical": { "score": 10, "max": 20 },
      "social": { "score": 12, "max": 15 },
      "market": { "score": 25, "max": 30 }
    },
    "flags": ["pump_fun_detected", "low_liquidity"],
    "verdict": "PROCEED | MONITOR | REJECT"
  },
  "error": null
}
```

## Known constraints
- DexScreener free tier: 300 requests/min (sufficient for scoring)
- CoinGecko free tier: 30 requests/min (rate-limited to 24/min internally)
- RugCheck may return 400 for non-Solana tokens (graceful degradation)
- Scores are point-in-time snapshots — re-score after 24h for stability check
- Pump.fun tokens auto-penalized (-10 points) due to low-mcap rug risk
