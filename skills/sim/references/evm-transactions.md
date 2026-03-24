# EVM Transactions

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Query raw transaction history for an EVM wallet address.

---

## evm transactions

Return raw transaction history for a wallet address across supported EVM chains. Transactions are returned in reverse-chronological order.

```bash
dune sim evm transactions <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | EVM wallet address (`0x...`, 42 characters) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | No | `default` | Restrict to specific chains by numeric ID or tag name (comma-separated) |
| `--decode` | `bool` | No | `false` | Include ABI-decoded function calls and event logs (only visible in JSON output) |
| `--limit` | `int` | No | server default | Max results per page (1-100) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns a reverse-chronological list of raw transactions. Each transaction includes the chain, transaction hash, sender/recipient addresses, native value transferred (in wei), block context (number, time, hash), gas parameters (gas price, EIP-1559 fees), nonce, transaction type, and raw calldata.

When `--decode` is used, each transaction additionally includes the ABI-decoded function name and typed inputs, plus an array of event logs with decoded event names and parameters. Decoded data is only visible in JSON output.

The response is paginated -- if `next_offset` is present, pass it as `--offset` to fetch more results.

Use `-o json` to get the full structured response. The text table shows chain, hash, from, to, value, and block number.

### Examples

```bash
# Transaction history across default chains
dune sim evm transactions 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Ethereum only with decoded function calls
dune sim evm transactions 0xd8da... --chain-ids 1 --decode -o json

# Limited to 20 most recent transactions
dune sim evm transactions 0xd8da... --limit 20 -o json
```

### Tips

- **`--decode` requires `-o json`**: Decoded data (function names, inputs, event logs) is nested and cannot be displayed in the text table format. Always use `-o json` with `--decode`.
- **Use `activity` for semantic data**: If you need classified, human-readable activity (sends, swaps, approvals), use `dune sim evm activity` instead. Transactions gives you the raw on-chain data.
- **Value is in wei**: The `value` field is the native currency amount in wei (1 ETH = 10^18 wei). Divide by 10^18 for human-readable ETH amounts.

### When to Use Transactions vs Activity

| Need | Command |
|------|---------|
| Raw calldata, gas parameters, nonce | `evm transactions` |
| Decoded function calls and event logs | `evm transactions --decode` |
| Human-readable activity (sends, swaps, etc.) | `evm activity` |
| USD values on transfers | `evm activity` |
| Contract interaction details | `evm transactions --decode` or `evm activity` (calls) |

---

## See Also

- [evm-activity.md](evm-activity.md) -- Decoded, classified activity feed
- [evm-balances.md](evm-balances.md) -- Token balances
- [Main skill](../SKILL.md) -- Command overview and workflows
