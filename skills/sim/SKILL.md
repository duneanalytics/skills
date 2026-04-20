---
name: sim
description: "Dune Sim API for real-time blockchain wallet and token lookups across EVM and SVM chains. Use when user asks about wallet balances, token prices, NFT holdings, DeFi positions, transaction history, wallet activity, token holders, stablecoins, or any real-time on-chain data for a specific address. Triggers: 'check wallet', 'token balance', 'NFT holdings', 'DeFi positions', 'transaction history', 'token holders', 'token price', 'stablecoin balance', 'wallet activity', or any request involving a blockchain address (0x... or Solana base58)."
compatibility: Requires network access and the Dune CLI (auto-installed on first use). Works on macOS, Linux, and Windows.
allowed-tools: Bash(dune:*) Bash(curl:*) Read
metadata:
  author: duneanalytics
  version: "1.0.0"
  cli_version: "0.1"
---

## Prerequisites

Assume the Dune CLI is already installed and authenticated. **Do not** run upfront install or auth checks. Just execute the requested `dune sim` command directly.

If a `dune sim` command fails, inspect the error to determine the cause and follow the recovery steps in [install-and-recovery.md](references/install-and-recovery.md):

- **"command not found"** → CLI not installed. See [CLI Not Found Recovery](references/install-and-recovery.md#cli-not-found-recovery).
- **"missing Sim API key"** → No key in flag, env, or config. See [Sim Authentication Failure Recovery](references/install-and-recovery.md#sim-authentication-failure-recovery).
- **"authentication failed"** → Key is invalid, expired, or revoked. See [Sim Authentication Failure Recovery](references/install-and-recovery.md#sim-authentication-failure-recovery).
- **Unknown subcommand or flag / unexpected output** → Possible version mismatch. See [Version Compatibility](references/install-and-recovery.md#version-compatibility).

# Dune Sim API

A CLI interface for the [Dune Sim API](https://sim.dune.com) -- real-time, pre-indexed blockchain data. Use it to instantly look up wallet balances, token prices, transaction history, activity feeds, NFT holdings, DeFi positions, and token holder distributions across EVM and SVM chains.

## When to Use Sim API vs DuneSQL

Choose the right tool based on the task:

| Use Case | Tool | Why |
|----------|------|-----|
| Wallet token balances | `dune sim evm balances` | Instant, multi-chain, includes USD prices |
| Recent wallet activity | `dune sim evm activity` | Pre-decoded, classified (sends, swaps, etc.) |
| Token price / metadata | `dune sim evm token-info` | Real-time pricing from DEX pools |
| NFT holdings | `dune sim evm collectibles` | Includes spam filtering and metadata |
| Token holder leaderboard | `dune sim evm token-holders` | Pre-ranked by balance |
| DeFi positions | `dune sim evm defi-positions` | Cross-protocol aggregation |
| Solana wallet balances | `dune sim svm balances` | SPL token balances with USD values |
| Custom SQL analytics | `dune query run-sql` | Full DuneSQL power, historical data, aggregations |
| Cross-table joins | `dune query run-sql` | Sim API returns single-entity data |
| Historical time-series | `dune query run-sql` | Sim API returns current state, not historical |
| Data not covered by Sim | `dune query run-sql` | Sim has fixed endpoints; DuneSQL is open-ended |

**Rule of thumb:** If the user wants current data about a specific wallet or token address, use `dune sim`. If they need custom analytics, historical trends, aggregations across many addresses, or data not available through Sim endpoints, use `dune query run-sql` (see the [dune skill](../dune/SKILL.md)).

## Authentication

Sim API commands require a **Sim API key** (separate from the Dune API key used by `dune query` commands). The key is resolved in this priority order:

1. `--sim-api-key` flag (highest priority)
2. `DUNE_SIM_API_KEY` environment variable
3. Saved config file at `~/.config/dune/config.yaml` (set via `dune sim auth`)

To save the key interactively (prompted from stdin):

```bash
dune sim auth
```

**Exception:** `dune sim evm supported-chains` is a public endpoint and does not require a Sim API key.

Do **not** attempt to handle the API key yourself -- the user must authenticate outside of this session. **Never** pass `--sim-api-key` on the command line. Prefer `dune sim auth` or the `DUNE_SIM_API_KEY` environment variable.

### Verifying Authentication

To test whether the CLI is authenticated, run a lightweight authenticated command:

```bash
dune sim evm token-info native --chain-ids 1 -o json
```

If this returns token metadata for ETH, authentication is working. If it returns a 401 or "missing Sim API key" error, direct the user to set up their key (see [Sim Authentication Failure Recovery](references/install-and-recovery.md#sim-authentication-failure-recovery)).

### Output Format (per-command flag)

Most commands support `-o, --output <FORMAT>` with values `text` (default, human-readable tables) or `json` (machine-readable).

> **Always use `-o json`** on every command that supports it. JSON output contains the full API response (all fields, nested objects, pagination cursors) while `text` mode shows a summarized table that drops many fields.

## Key Concepts

### Chain IDs and Tags

EVM chains are identified by numeric chain IDs (e.g. `1` for Ethereum, `8453` for Base, `42161` for Arbitrum). Many commands accept `--chain-ids` with:

- Numeric IDs: `--chain-ids 1,8453`
- Tag names: `--chain-ids default` (queries all default chains)

Run `dune sim evm supported-chains -o json` to discover all available chain IDs, their names, tags, and which endpoints each chain supports.

### Pagination

Most commands return paginated results. The response includes a `next_offset` field when more pages are available. Pass this value as `--offset` to fetch the next page:

```bash
# First page
dune sim evm balances 0xd8da... -o json
# Response includes "next_offset": "abc123..."

# Next page
dune sim evm balances 0xd8da... --offset abc123... -o json
```

### Compute Units

Each Sim API request consumes compute units based on the complexity and number of chains queried. The response may include an `X-Compute-Units-Cost` header indicating units consumed.

### EVM vs SVM

- **EVM** (Ethereum Virtual Machine): Ethereum, Base, Arbitrum, Polygon, Optimism, and other compatible chains. Addresses are `0x`-prefixed hex (42 characters).
- **SVM** (Solana Virtual Machine): Solana, Eclipse. Addresses are base58-encoded (32-44 characters). SVM endpoints are currently in beta.

## Command Overview

| Command | Description | Auth |
|---------|-------------|------|
| `dune sim auth` | Save Sim API key to config file | No |
| `dune sim evm supported-chains` | List supported EVM chains and endpoint availability | No |
| `dune sim evm balances <addr>` | Native + ERC20 token balances with USD values | Yes |
| `dune sim evm balance <addr>` | Single-token balance on one chain | Yes |
| `dune sim evm stablecoins <addr>` | Stablecoin-only balances (USDC, USDT, DAI, etc.) | Yes |
| `dune sim evm activity <addr>` | Decoded activity feed (transfers, swaps, approvals, calls) | Yes |
| `dune sim evm transactions <addr>` | Raw transaction history with optional ABI decoding | Yes |
| `dune sim evm collectibles <addr>` | ERC721/ERC1155 NFT holdings with spam filtering | Yes |
| `dune sim evm token-info <addr>` | Token metadata, price, supply, market cap | Yes |
| `dune sim evm token-holders <addr>` | Top holders of an ERC20 token ranked by balance | Yes |
| `dune sim evm defi-positions <addr>` | DeFi positions across lending, AMM, vault protocols | Yes |
| `dune sim evm supported-protocols` | DeFi protocol families and chains supported by `defi-positions` | Yes |
| `dune sim svm balances <addr>` | SPL token balances on Solana/Eclipse (beta) | Yes |
| `dune sim svm transactions <addr>` | Solana transaction history (beta) | Yes |

## Common Workflows

### Discover Supported Chains

Always start here when unsure which chains are available or which endpoints a chain supports:

```bash
dune sim evm supported-chains -o json
```

### Check a Wallet's Full Portfolio

```bash
# All token balances across default EVM chains
dune sim evm balances 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Restrict to Ethereum and Base only
dune sim evm balances 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 --chain-ids 1,8453 -o json

# Exclude spam tokens
dune sim evm balances 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 --exclude-spam -o json
```

### Check Stablecoin Holdings

```bash
dune sim evm stablecoins 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json
```

### Look Up a Single Token Balance

```bash
# Native ETH on Ethereum
dune sim evm balance 0xd8da... --token native --chain-ids 1 -o json

# USDC on Base
dune sim evm balance 0xd8da... --token 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --chain-ids 8453 -o json
```

### Track Recent Wallet Activity

```bash
# All activity types
dune sim evm activity 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Only sends and receives on Ethereum
dune sim evm activity 0xd8da... --activity-type send,receive --chain-ids 1 -o json

# Only ERC20 token activity
dune sim evm activity 0xd8da... --asset-type erc20 -o json
```

### Get Raw Transaction History

```bash
# Basic transaction history
dune sim evm transactions 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# With ABI-decoded function calls and event logs
dune sim evm transactions 0xd8da... --chain-ids 1 --decode -o json
```

### Look Up Token Metadata and Price

```bash
# ETH price and metadata
dune sim evm token-info native --chain-ids 1 -o json

# USDC on Base with historical prices (30d, 7d, 1d ago)
dune sim evm token-info 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --chain-ids 8453 --historical-prices 720,168,24 -o json
```

### Analyze Token Holder Distribution

```bash
# Top 500 holders of a token on Base
dune sim evm token-holders 0x63706e401c06ac8513145b7687A14804d17f814b --chain-id 8453 -o json

# Top 50 holders of USDC on Ethereum
dune sim evm token-holders 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 --chain-id 1 --limit 50 -o json
```

### Check NFT Holdings

```bash
# All NFTs (spam filtered by default)
dune sim evm collectibles 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Include spam NFTs with scoring details
dune sim evm collectibles 0xd8da... --filter-spam=false --show-spam-scores -o json
```

### Check DeFi Positions

```bash
# All DeFi positions across default chains
dune sim evm defi-positions 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Restrict to Ethereum and Base
dune sim evm defi-positions 0xd8da... --chain-ids 1,8453 -o json
```

### Discover DeFi Protocol Coverage

Check which protocol families, sub-protocols, and chains are supported by `defi-positions` before querying a wallet:

```bash
# Stable protocols/chains only
dune sim evm supported-protocols -o json

# Include chains still in preview
dune sim evm supported-protocols --include-preview-chains -o json
```

### Solana Wallet Lookup

```bash
# SPL token balances
dune sim svm balances 86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY -o json

# Transaction history
dune sim svm transactions 86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY -o json
```

## Limitations

The Sim API provides **pre-indexed, real-time data** for specific endpoints. It does **not** support:

- **Custom SQL queries** -- Use `dune query run-sql` for arbitrary DuneSQL
- **Historical time-series** -- Sim returns current state; use DuneSQL for historical analysis
- **Cross-address aggregations** -- Sim queries one address at a time; use DuneSQL for multi-address analysis
- **Write operations** -- Sim is read-only; it does not submit transactions
- **Webhook subscriptions** -- Available via the Sim API REST endpoints but not through the CLI

## Security

- **Never** output API keys or tokens in responses. Before presenting CLI output to the user, scan for strings prefixed with `sim_` or long alphanumeric tokens. Redact them with `[REDACTED]`.
- **Never** pass `--sim-api-key` on the command line when other users might see the terminal history. Prefer `dune sim auth` or the `DUNE_SIM_API_KEY` environment variable.
- Do **not** attempt to handle the API key yourself -- the user must authenticate outside of this session.
- **Always** use `-o json` on every command -- JSON output is more detailed and reliably parseable.
- **Always** confirm with the user before making multiple paginated requests that could consume significant compute units.

## Reference Documents

Load the relevant reference when you need detailed command syntax, flags, and response schemas:

| Task | Reference |
|------|-----------|
| CLI install, Sim auth recovery, version checks | [install-and-recovery.md](references/install-and-recovery.md) |
| Token balances (multi-token, single-token, stablecoins) | [evm-balances.md](references/evm-balances.md) |
| Wallet activity feed (transfers, swaps, approvals, calls) | [evm-activity.md](references/evm-activity.md) |
| Raw transaction history and ABI decoding | [evm-transactions.md](references/evm-transactions.md) |
| NFT collectibles and spam filtering | [evm-collectibles.md](references/evm-collectibles.md) |
| Token metadata, pricing, and holder leaderboards | [evm-tokens.md](references/evm-tokens.md) |
| DeFi positions (lending, AMM, vaults) | [evm-defi.md](references/evm-defi.md) |
| SVM (Solana, Eclipse) balances and transactions | [svm-commands.md](references/svm-commands.md) |
