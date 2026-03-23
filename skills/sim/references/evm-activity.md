# EVM Activity

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Query the decoded activity feed for an EVM wallet address.

---

## evm activity

Return a reverse-chronological feed of human-readable on-chain activity for a wallet. Unlike raw transactions, activity entries are decoded and classified into semantic types.

```bash
dune sim evm activity <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | EVM wallet address (`0x...`, 42 characters) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | No | `default` | Restrict to specific chains by numeric ID or tag name (comma-separated) |
| `--token-address` | `string` | No | -- | Filter to activities involving specific token contracts (comma-separated addresses) |
| `--activity-type` | `string` | No | all | Filter by activity classification (comma-separated): `send`, `receive`, `mint`, `burn`, `swap`, `approve`, `call` |
| `--asset-type` | `string` | No | all | Filter by token standard (comma-separated): `native`, `erc20`, `erc721`, `erc1155` |
| `--limit` | `int` | No | server default | Max results per page (1-100) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Activity Types

| Type | Description |
|------|-------------|
| `send` | Outgoing transfer of native currency or tokens from the wallet |
| `receive` | Incoming transfer of native currency or tokens to the wallet |
| `mint` | Token creation event involving the wallet |
| `burn` | Token destruction event involving the wallet |
| `swap` | DEX token exchange (includes both sides of the trade) |
| `approve` | ERC20/ERC721 spending approval granted by the wallet |
| `call` | Contract interaction with decoded function name and inputs |

### Asset Types

| Type | Description |
|------|-------------|
| `native` | Chain's native asset (ETH, MATIC, etc.) |
| `erc20` | ERC20 fungible tokens |
| `erc721` | ERC721 NFTs (unique tokens) |
| `erc1155` | ERC1155 multi-tokens (fungible + non-fungible) |

### Response

Returns a reverse-chronological list of decoded activity items. Each item includes the chain, block context (number, time, tx hash), the activity type (send, receive, swap, etc.), asset type (native, erc20, etc.), transfer amounts with USD values, and token metadata (symbol, name, decimals, price).

Swap entries include both sides of the trade -- the sold token and the bought token with their respective amounts and metadata. Call entries include the decoded function name, signature, and typed inputs, plus contract metadata when available. Approve entries include the approved spender address.

The response is paginated -- if `next_offset` is present, pass it as `--offset` to fetch more results.

Use `-o json` to get the full structured response. The text table shows a summary with chain, type, asset, value, and counterparty.

### Examples

```bash
# All activity across default chains
dune sim evm activity 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Only sends and receives on Ethereum
dune sim evm activity 0xd8da... --activity-type send,receive --chain-ids 1 -o json

# Only ERC20 token activity
dune sim evm activity 0xd8da... --asset-type erc20 -o json

# Activity for a specific token (USDC)
dune sim evm activity 0xd8da... --token-address 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 -o json

# Only swaps, limited to 20 results
dune sim evm activity 0xd8da... --activity-type swap --limit 20 -o json
```

### Tips

- The activity feed is **decoded** -- it shows human-readable types like `swap` and `approve` rather than raw calldata. For raw transaction data, use `dune sim evm transactions` instead.
- Swap entries include **both sides** of the trade (`from_token_*` and `to_token_*` fields), making it easy to see what was exchanged for what.
- Use `--activity-type` and `--asset-type` together for precise filtering (e.g. `--activity-type send --asset-type erc20` for outgoing ERC20 transfers only).

---

## See Also

- [evm-transactions.md](evm-transactions.md) -- Raw transaction data with optional ABI decoding
- [evm-balances.md](evm-balances.md) -- Token balances for a wallet
- [Main skill](../SKILL.md) -- Command overview and workflows
