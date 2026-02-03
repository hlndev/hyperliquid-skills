# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) working with this repository—a collection of skills for interacting with the Hyperliquid API.

## Repository Structure

```
hyperliquid-skills/
├── AGENTS.md                    # This file
├── README.md                    # User-facing documentation
└── skills/
    └── {skill-name}/
        └── SKILL.md             # Skill definition (required)
```

## Naming Standards

- **Directories**: kebab-case (e.g., `hl-market-data`)
- **Skill files**: `SKILL.md` (uppercase, exact filename)
- **Skill names**: Match directory name in frontmatter

## SKILL.md Format

Each skill requires YAML frontmatter:

```yaml
---
name: skill-name
description: When to use this skill. Be specific about triggers.
allowed-tools: WebFetch, Read  # or empty for reference-only skills
---
```

Followed by markdown content with:
- Clear section headers
- Code examples (especially curl commands for API skills)
- Tables for quick reference
- Common mistakes section where applicable

## Available Skills

| Skill | Purpose |
|-------|---------|
| `hyperliquid-api-docs` | Fetch official API documentation on-demand |
| `hl-api-samples` | Generate curl commands for API testing |
| `hl-order-placement` | Order placement guide (limit, market, bracket, TWAP) |
| `hl-account-overview` | Account state and position data structures |
| `hl-market-data` | Market data fetching (prices, orderbook, candles) |
| `hl-hip3-markets` | HIP-3 builder-deployed perpetuals |
| `hl-spot-markets` | Spot market handling and identifiers |

## Installation

Users install via Claude Code:

```bash
/plugin marketplace add cezar-r/hyperliquid-skills
/plugin install cezar-r/hyperliquid-skills@{skill-name}
```

## Adding New Skills

1. Create directory: `skills/{skill-name}/`
2. Create `SKILL.md` with valid YAML frontmatter
3. Keep content under 500 lines for context efficiency
4. Include working curl examples where applicable
5. Update README.md with new skill

## Hyperliquid API Reference

- **Base URL**: `https://api.hyperliquid.xyz/info`
- **Method**: POST with `Content-Type: application/json`
- **Docs**: https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api

## Context Efficiency

- Write specific descriptions that trigger on relevant queries
- Use tables for dense reference information
- Prefer curl examples over verbose explanations
- Link to official docs rather than duplicating content
