# EVM Token Info & Holders

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Token-centric lookups: metadata/pricing and holder distribution.

- **`evm token-info`** -- Token metadata, current price, supply, and market cap
- **`evm token-holders`** -- Top holders ranked by balance

---

## evm token-info

Return metadata and real-time pricing for a token contract address on a specified EVM chain. Use `native` as the address for the chain's native asset (e.g. ETH on chain 1).

```bash
dune sim evm token-info <address> --chain-ids <chain_id> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | Token contract address (`0x...`) or the literal string `native` |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | **Yes** | -- | Single numeric chain ID (e.g. `1`, `8453`) |
| `--historical-prices` | `string` | No | -- | Include historical USD prices at hour offsets (e.g. `720,168,24` for 30d, 7d, 1d ago) |
| `--limit` | `int` | No | server default | Max results |
| `--offset` | `string` | No | -- | Pagination cursor |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns token metadata and real-time pricing. Each token entry includes the chain, token identity (symbol, name, decimals, contract address), current USD price sourced from on-chain DEX pools, total supply, estimated market capitalization, and a logo URL. If `--historical-prices` is set, past USD prices at the requested hour offsets are included.

Use `-o json` to get the full structured response. The text table shows chain, symbol, price, market cap, and supply.

### Examples

```bash
# ETH price and metadata on Ethereum
dune sim evm token-info native --chain-ids 1 -o json

# USDC on Base
dune sim evm token-info 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 --chain-ids 8453 -o json

# Token info with 30d, 7d, 1d price history
dune sim evm token-info native --chain-ids 1 --historical-prices 720,168,24 -o json
```

### Tips

- `price_usd` comes from on-chain DEX liquidity pools, not centralized exchange feeds.
- Use `--historical-prices` to track price changes over time (e.g. `720` hours = 30 days ago).
- This command is **not** wallet-scoped. It returns token metadata regardless of who holds it. For wallet-specific balances, use `dune sim evm balances` or `dune sim evm balance`.

---

## evm token-holders

Return a leaderboard of top holders for a given ERC20 token on a single chain, ranked by balance in descending order.

```bash
dune sim evm token-holders <token_address> --chain-id <chain_id> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `token_address` | `string` | ERC20 token contract address (`0x...`) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-id` | `string` | **Yes** | -- | Single numeric chain ID (e.g. `1`, `8453`). Note: singular `--chain-id`, not `--chain-ids` |
| `--limit` | `int` | No | `500` | Max holders to return (1-500) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns a ranked list of token holders in descending order by balance. Each holder entry includes the wallet address, raw token balance, the timestamp of their earliest acquisition, and whether they have ever initiated an outgoing transfer (useful for distinguishing active holders from passive airdrop recipients).

The response is paginated -- if `next_offset` is present, pass it as `--offset` to get holders beyond the current page.

Use `-o json` to get the full structured response. The text table shows wallet address, balance, and acquisition date.

### Examples

```bash
# Top 500 holders on Base
dune sim evm token-holders 0x63706e401c06ac8513145b7687A14804d17f814b --chain-id 8453 -o json

# Top 50 holders of USDC on Ethereum
dune sim evm token-holders 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 --chain-id 1 --limit 50 -o json
```

### Tips

- **Single chain only**: Unlike most other `evm` commands, `token-holders` uses `--chain-id` (singular) and queries exactly one chain.
- **`has_initiated_transfer`** is useful for distinguishing real holders from airdrop recipients who never interacted with the token.
- Paginate using `--offset` with the `next_offset` value to get holders beyond the top 500.

---

## See Also

- [evm-balances.md](evm-balances.md) -- Wallet-scoped token balances
- [evm-defi.md](evm-defi.md) -- DeFi positions
- [Main skill](../SKILL.md) -- Command overview and workflows
