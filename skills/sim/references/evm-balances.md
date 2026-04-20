# EVM Balances

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Query token balances for EVM wallet addresses. Three commands with shared response format:

- **`evm balances`** -- All native + ERC20 token balances across multiple chains
- **`evm balance`** -- Single token balance on one chain
- **`evm stablecoins`** -- Stablecoin-only balances across multiple chains

---

## evm balances

Return native and ERC20 token balances for a wallet address across supported EVM chains. Each balance includes the token amount, current USD price, and total USD value.

```bash
dune sim evm balances <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | EVM wallet address (`0x...`, 42 characters) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | No | `default` | Restrict to specific chains by numeric ID or tag name (comma-separated, e.g. `1,8453` or `default`) |
| `--filters` | `string` | No | -- | Filter by token standard: `erc20` or `native` |
| `--asset-class` | `string` | No | -- | Filter by asset classification: `stablecoin` |
| `--metadata` | `string` | No | -- | Request additional fields (comma-separated): `logo`, `url`, `pools` |
| `--exclude-spam` | `bool` | No | `false` | Exclude low-liquidity tokens (< $100 pool size) |
| `--exclude-unpriced` | `bool` | No | `true` | Exclude tokens without a USD price. Pass `--exclude-unpriced=false` to include them. |
| `--historical-prices` | `string` | No | -- | Include historical USD prices at hour offsets (e.g. `720,168,24` for 30d, 7d, 1d ago) |
| `--limit` | `int` | No | server default | Max results per page (1-1000) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns a list of token balances for the wallet. Each balance entry identifies the chain and token (with symbol, name, and contract address), and includes the raw balance amount, current USD price, and total USD value. A liquidity indicator flags tokens with thin on-chain pricing pools. If `--historical-prices` is set, past USD prices at the requested offsets are included. If `--metadata` is set, additional token metadata (logo, URL, pool details) is included.

The response is paginated -- if `next_offset` is present, pass it as `--offset` to fetch more results. The `warnings` array surfaces issues like unsupported chain IDs.

Use `-o json` to get the full structured response. The text table shows chain, symbol, balance, price, and value.

### Examples

```bash
# All token balances across default chains (priced tokens only by default)
dune sim evm balances 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Ethereum and Base only, exclude spam
dune sim evm balances 0xd8da... --chain-ids 1,8453 --exclude-spam -o json

# Only ERC20 tokens with logo metadata
dune sim evm balances 0xd8da... --filters erc20 --metadata logo -o json

# Include historical prices (30d, 7d, 1d)
dune sim evm balances 0xd8da... --historical-prices 720,168,24 -o json

# Include tokens without a USD price (full token list, not just priced)
dune sim evm balances 0xd8da... --exclude-unpriced=false -o json
```

---

## evm balance

Return the balance of a **single token** for a wallet on **one chain**. Both `--token` and `--chain-ids` are required.

```bash
dune sim evm balance <wallet_address> --token <token> --chain-ids <chain_id> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `wallet_address` | `string` | EVM wallet address (`0x...`) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--token` | `string` | **Yes** | -- | Token contract address (`0x...`) or the literal string `native` for the chain's native asset (e.g. ETH) |
| `--chain-ids` | `string` | **Yes** | -- | Single numeric chain ID (e.g. `1`, `8453`) |
| `--metadata` | `string` | No | -- | Additional fields: `logo`, `url`, `pools` |
| `--historical-prices` | `string` | No | -- | Hour offsets for historical prices (e.g. `720,168,24`) |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Same format as `evm balances` but returns exactly one balance entry (or none if the token is not held). Use `-o json` for the full response.

### Examples

```bash
# Native ETH balance on Ethereum
dune sim evm balance 0xd8da... --token native --chain-ids 1 -o json

# USDC balance on Base
dune sim evm balance 0xd8da... --token 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --chain-ids 8453 -o json

# With historical prices
dune sim evm balance 0xd8da... --token native --chain-ids 1 --historical-prices 168,24 -o json
```

---

## evm stablecoins

Return stablecoin-only balances (USDC, USDT, DAI, FRAX, etc.) for a wallet. This is a convenience shorthand for `evm balances <addr> --asset-class stablecoin`.

```bash
dune sim evm stablecoins <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | EVM wallet address (`0x...`) |

### Flags

Same as `evm balances` (except `--asset-class` which is automatically set to `stablecoin`):

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | No | `default` | Comma-separated chain IDs or tags |
| `--filters` | `string` | No | -- | Token filter: `erc20` or `native` |
| `--metadata` | `string` | No | -- | Additional fields: `logo`, `url`, `pools` |
| `--exclude-spam` | `bool` | No | `false` | Exclude low-liquidity tokens |
| `--exclude-unpriced` | `bool` | No | `true` | Exclude tokens without a USD price. Pass `--exclude-unpriced=false` to include them. |
| `--historical-prices` | `string` | No | -- | Hour offsets for historical prices |
| `--limit` | `int` | No | server default | Max results per page (1-1000) |
| `--offset` | `string` | No | -- | Pagination cursor |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Same format as `evm balances` (filtered to stablecoins only). Use `-o json` for the full response.

### Examples

```bash
# Stablecoin balances across default chains
dune sim evm stablecoins 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Stablecoins on Ethereum and Base only
dune sim evm stablecoins 0xd8da... --chain-ids 1,8453 -o json
```

---

## When to Use Which

| Scenario | Command |
|----------|---------|
| Full wallet portfolio | `evm balances` |
| Check a specific token balance | `evm balance` (with `--token` and `--chain-ids`) |
| Stablecoin holdings only | `evm stablecoins` |
| Need token price without wallet context | Use `evm token-info` instead (see [evm-tokens.md](evm-tokens.md)) |

---

## See Also

- [evm-tokens.md](evm-tokens.md) -- Token metadata, pricing, and holder leaderboards
- [evm-activity.md](evm-activity.md) -- Decoded activity feed for a wallet
- [Main skill](../SKILL.md) -- Command overview and workflows
