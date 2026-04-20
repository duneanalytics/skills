# EVM DeFi Positions

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Query DeFi positions across lending, AMM, and vault protocols for an EVM wallet.

---

## evm defi-positions

Return DeFi positions for a wallet address across supported EVM chains and protocols. Each position includes USD valuation and protocol-specific metadata. The response also includes aggregation summaries with total USD value and per-chain breakdowns.

```bash
dune sim evm defi-positions <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | EVM wallet address (`0x...`, 42 characters) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | No | `default` | Restrict to specific chains by numeric ID or tag name (comma-separated) |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Position Types

| Type | Description | Examples |
|------|-------------|---------|
| `Erc4626` | ERC-4626 vault positions (yield vaults, staking wrappers) | Yearn vaults, Lido wstETH |
| `Tokenized` | Lending protocol receipt tokens | Aave aTokens, Compound cTokens |
| `UniswapV2` | AMM liquidity provider positions (V2-style) | Uniswap V2, SushiSwap |
| `Nft` | Uniswap V3 concentrated liquidity (NFT-based) | Uniswap V3 positions |
| `NftV4` | Uniswap V4 concentrated liquidity (NFT-based) | Uniswap V4 positions |

### Response

Returns a list of DeFi positions and an aggregation summary. Each position includes the position type (see table above), chain, USD value, protocol name, and a logo URL. The structure of type-specific fields varies by position type:

- **Erc4626 / Tokenized**: Includes the receipt token and underlying token (as nested objects with address, symbol, name, decimals, holdings, and price), the effective balance, and for Tokenized positions, the token classification and lending pool address.
- **UniswapV2**: Includes the pool address, both pool tokens (as nested objects), LP balance, and effective value.
- **Nft / NftV4** (concentrated liquidity): Includes the pool tokens plus an array of individual tick-range sub-positions, each with its own tick boundaries, NFT token ID, and per-token holdings and uncollected rewards.

The aggregation summary provides a total USD value across all positions and a per-chain breakdown.

Use `-o json` to get the full structured response. The text table shows type, chain, protocol, USD value, and a summary of the position details (token pair, balance, number of sub-positions).

### Examples

```bash
# All DeFi positions across default chains
dune sim evm defi-positions 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Ethereum and Base only
dune sim evm defi-positions 0xd8da... --chain-ids 1,8453 -o json
```

### Tips

- The `aggregations` object provides a quick portfolio summary without needing to sum individual positions.
- Position types are polymorphic -- different types have different fields. Always check the `type` field to know which additional fields are available.
- Token fields (`token`, `underlying_token`, `token0`, `token1`) are nested objects, not flat strings. Access symbol via `token.symbol`, price via `token.price_usd`, etc.
- Concentrated liquidity positions (Nft/NftV4) may have multiple sub-positions in the `positions` array, each representing a different tick range. Each sub-position has its own `token0` and `token1` objects with `holdings` and `rewards`.

---

## evm supported-protocols

List the DeFi protocol families, sub-protocols, and chains that `evm defi-positions` can return data for. Use this to discover coverage before querying a wallet.

```bash
dune sim evm supported-protocols [flags]
```

### Arguments

None.

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--include-preview-chains` | `bool` | No | `false` | Include chains marked as preview (not yet publicly available) |
| `--include-preview-protocols` | `bool` | No | `false` | Include protocols marked as preview on the requested chains |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns `protocol_families[]`, where each family contains:

- `family` -- protocol family name (e.g. `aave`, `uniswap`)
- `chains[]` -- each with `chain_id`, `chain_name`, and `status` (`stable` or `preview`)
- `sub_protocols[]` -- recognized forks / sub-protocols within the family

The text table has columns `FAMILY | CHAINS | SUB_PROTOCOLS`. Chain entries render as `name(id)` with a `*` suffix indicating preview status.

Use `-o json` for the full structured response.

### Examples

```bash
# Stable protocols / chains only
dune sim evm supported-protocols -o json

# Include preview chains
dune sim evm supported-protocols --include-preview-chains -o json

# Include preview protocols too
dune sim evm supported-protocols --include-preview-chains --include-preview-protocols -o json
```

### Tips

- Run this before `evm defi-positions` whenever the user asks "is protocol X / chain Y covered?".
- Preview entries aren't guaranteed to return data from `defi-positions` yet; treat them as coverage roadmap rather than current state.

---

## See Also

- [evm-balances.md](evm-balances.md) -- Token balances (non-DeFi)
- [evm-tokens.md](evm-tokens.md) -- Token metadata and holders
- [Main skill](../SKILL.md) -- Command overview and workflows
