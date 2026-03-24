# SVM Commands (Solana, Eclipse)

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Query SVM (Solana Virtual Machine) blockchain data. Both commands are currently in beta.

Supported chains: **Solana**, **Eclipse**.

---

## svm balances

Return SPL token balances for an SVM wallet address with USD valuations and liquidity data.

```bash
dune sim svm balances <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | SVM wallet address (base58, 32-44 characters) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chains` | `string` | No | `solana` | SVM chains to query (comma-separated): `solana`, `eclipse` |
| `--limit` | `int` | No | `1000` | Max results per page (1-1000) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns a list of SPL token balances for the wallet. Each balance entry identifies the chain and token (with mint address, symbol, name, and decimals), and includes both the raw amount and a human-readable balance, current USD price, total USD value, and on-chain liquidity depth. Additional fields include the owning SPL program, total supply, mint authority, metadata URI, and pricing pool details.

The response is paginated -- if `next_offset` is present, pass it as `--offset` to fetch more results. A `balances_count` field indicates the total number of token balances.

Use `-o json` to get the full structured response. The text table shows chain, symbol, balance, price, and value.

### Examples

```bash
# Solana token balances
dune sim svm balances 86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY -o json

# Both Solana and Eclipse
dune sim svm balances 86xCnPeV... --chains solana,eclipse -o json

# Limited to top 50 by response
dune sim svm balances 86xCnPeV... --limit 50 -o json
```

### Tips

- By default, only Solana is queried. Add `--chains eclipse` or `--chains solana,eclipse` to include Eclipse.
- The `balance` field is human-readable (already divided by decimals), while `amount` is the raw value.
- `program_id` identifies the token standard program (SPL Token, Token-2022, etc.).

---

## svm transactions

Return transaction history for an SVM wallet address. Transactions are returned in reverse-chronological order by block slot.

```bash
dune sim svm transactions <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | SVM wallet address (base58) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--limit` | `int` | No | `100` | Max results per page (1-1000) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns a reverse-chronological list of transactions. Each transaction includes the chain, wallet address, block slot number, and block timestamp (in microseconds since Unix epoch). In JSON output, each transaction also includes the full raw Solana transaction object with signatures, instructions, account keys, and metadata (logs, balance changes, compute units consumed).

The response is paginated -- if `next_offset` is present, pass it as `--offset` to fetch more results.

Use `-o json` to access the full raw transaction data. The text table shows chain, block slot, timestamp, and the first transaction signature.

### Examples

```bash
# Recent Solana transactions
dune sim svm transactions 86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY -o json

# Limited to 20 transactions
dune sim svm transactions 86xCnPeV... --limit 20 -o json
```

### Tips

- The text table shows chain, block slot, timestamp, and the first transaction signature. Use `-o json` to access the full `raw_transaction` with instructions, logs, and balance changes.
- `block_time` is in **microseconds** since Unix epoch. Divide by 1,000,000 to get seconds for standard Unix timestamp conversion.
- Unlike EVM transactions, there is no `--chains` flag -- the command queries all SVM chains where the address has transactions.

---

## See Also

- [evm-balances.md](evm-balances.md) -- EVM token balances (for Ethereum-family chains)
- [evm-transactions.md](evm-transactions.md) -- EVM transaction history
- [Main skill](../SKILL.md) -- Command overview and workflows
