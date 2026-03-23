# EVM Collectibles

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Query NFT holdings for an EVM wallet address.

---

## evm collectibles

Return ERC721 and ERC1155 collectibles (NFTs) held by a wallet address across supported EVM chains. Spam filtering is enabled by default.

```bash
dune sim evm collectibles <address> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `address` | `string` | EVM wallet address (`0x...`, 42 characters) |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--chain-ids` | `string` | No | `default` | Restrict to specific chains by numeric ID or tag name (comma-separated) |
| `--filter-spam` | `bool` | No | `true` | Hide NFTs identified as spam; set `--filter-spam=false` to include all |
| `--show-spam-scores` | `bool` | No | `false` | Include spam classification details (is_spam, spam_score, feature explanations) |
| `--limit` | `int` | No | `250` | Max results per page (1-2500) |
| `--offset` | `string` | No | -- | Pagination cursor from previous response's `next_offset` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Response

Returns a list of NFT collectibles held by the wallet. Each entry identifies the collection (contract address, name, symbol) and the specific token (token ID, token standard, description, image URL). The quantity held is included (always 1 for ERC721, may be more for ERC1155), along with the last acquisition timestamp and last known sale price.

When `--show-spam-scores` is enabled, each entry additionally includes a spam classification flag, numeric score, and per-feature explanations with weights.

The response is paginated -- if `next_offset` is present, pass it as `--offset` to fetch more results.

Use `-o json` to get the full structured response including token metadata and attributes. The text table shows chain, collection, token ID, standard, and name.

### Spam Filtering

Spam filtering uses a scoring model based on collection traits:
- Holder count and distribution
- Transfer patterns (airdrop-like behavior)
- Metadata quality (missing images, duplicate descriptions)
- Contract characteristics

By default, `--filter-spam=true` hides NFTs classified as spam. To see all NFTs including spam, set `--filter-spam=false`. To understand why an NFT was classified as spam, add `--show-spam-scores`.

### Examples

```bash
# All NFTs (spam filtered by default)
dune sim evm collectibles 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 -o json

# Ethereum-only NFTs
dune sim evm collectibles 0xd8da... --chain-ids 1 -o json

# Include spam NFTs with scoring details
dune sim evm collectibles 0xd8da... --filter-spam=false --show-spam-scores -o json

# Large page (up to 2500 NFTs)
dune sim evm collectibles 0xd8da... --limit 2500 -o json
```

### Tips

- The default `--limit` is 250 and can go up to 2500, significantly higher than most other Sim API endpoints.
- `image_url` may be an IPFS URL (`ipfs://...`). To display it, you may need to convert it to an HTTP gateway URL (e.g. `https://ipfs.io/ipfs/...`).
- ERC1155 tokens can have `balance` > 1 since they support semi-fungible tokens.

---

## See Also

- [evm-balances.md](evm-balances.md) -- Fungible token balances (ERC20 + native)
- [evm-activity.md](evm-activity.md) -- Activity feed including NFT transfers
- [Main skill](../SKILL.md) -- Command overview and workflows
