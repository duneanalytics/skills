# Multi-Chain Analytics Patterns

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication and the [DuneSQL cheatsheet](dunesql-cheatsheet.md) for core syntax.

Advanced DuneSQL patterns for querying tokens, balances, and events across EVM chains, Starknet, and Stellar.

---

## 1. Cross-chain token supply (mint/burn)

The unified pattern for tracking any ERC-20 token's net supply across all EVM chains via a single query:

```sql
WITH token_moves AS (
  SELECT
    DATE_TRUNC('day', evt_block_time) AS day,
    blockchain,
    SUM(CASE
      WHEN "from" = 0x0000000000000000000000000000000000000000 THEN  CAST(value AS DOUBLE) / 1e18
      WHEN "to"   = 0x0000000000000000000000000000000000000000 THEN -CAST(value AS DOUBLE) / 1e18
      ELSE 0
    END) AS net_change
  FROM evms.erc20_transfers
  WHERE evt_block_time >= TIMESTAMP '2024-01-01'
    AND blockchain IN ('ethereum', 'polygon', 'arbitrum', 'base', 'optimism')
    AND contract_address = 0x<TOKEN_ADDRESS>
  GROUP BY 1, 2
),
calendar AS (
  SELECT date_trunc('day', period) AS day
  FROM UNNEST(SEQUENCE(DATE '2024-01-01', CURRENT_DATE, INTERVAL '1' DAY)) AS t(period)
),
cumulative AS (
  SELECT
    c.day,
    SUM(COALESCE(m.net_change, 0)) OVER (ORDER BY c.day) AS net_supply
  FROM calendar c
  LEFT JOIN token_moves m ON c.day = m.day
    AND m.blockchain = 'ethereum'  -- adjust per product
)
SELECT day, net_supply FROM cumulative ORDER BY day
```

**Key points:**
- `evms.erc20_transfers` is the unified cross-chain ERC-20 table. Always add a `blockchain IN (...)` filter — without it, Dune scans all chains.
- Mint = transfer FROM the zero address. Burn = transfer TO the zero address.
- Adjust divisor (1e18, 1e6, 1e5, etc.) to match the token's decimals.

---

## 2. Per-wallet balance reconstruction

Reconstruct current holdings for every wallet from transfer history:

```sql
WITH all_flows AS (
  SELECT CAST("to" AS VARCHAR) AS wallet, CAST(value AS DOUBLE) / 1e18 AS amount
  FROM evms.erc20_transfers
  WHERE contract_address = 0x<TOKEN>
    AND "to" != 0x0000000000000000000000000000000000000000
    AND blockchain = 'ethereum'
  UNION ALL
  SELECT CAST("from" AS VARCHAR) AS wallet, -CAST(value AS DOUBLE) / 1e18
  FROM evms.erc20_transfers
  WHERE contract_address = 0x<TOKEN>
    AND "from" != 0x0000000000000000000000000000000000000000
    AND blockchain = 'ethereum'
)
SELECT wallet, SUM(amount) AS balance
FROM all_flows
GROUP BY wallet
HAVING SUM(amount) > 0.001
ORDER BY balance DESC
```

---

## 3. Starknet Transfer events

Starknet uses events rather than ERC-20 style logs. The Transfer event structure differs from EVM:

```sql
SELECT
  CAST(keys[2] AS VARCHAR) AS sender,
  CAST(keys[3] AS VARCHAR) AS receiver,
  CAST(bytearray_to_uint256(data[1]) AS DOUBLE) / 1e18 AS amount
FROM starknet.events
WHERE block_date >= DATE '2024-01-01'
  AND keys[1] = 0x0099cd8bde557814842a3121e8ddfd433a539b8c9f14bf31ebf108d12e6196e9  -- Transfer event key
  AND from_address = 0x<STARKNET_TOKEN_CONTRACT>
  AND cardinality(keys) >= 3
  AND cardinality(data) >= 1
```

- `from_address` = the token contract address (not the sender wallet)
- `keys[2]` = sender, `keys[3]` = recipient
- `data[1]` = amount (u256 low word; for Starknet token amounts the high word is always 0)
- Zero address on Starknet is 64 hex zeros: `0x0000...0000` (64 chars)

---

## 4. Stellar contract state (Soroban)

Stellar/Soroban stores **state snapshots** (balance entries), not transfer events. Each row is a balance at a point in time. To get the **current balance**, take the latest snapshot per holder:

```sql
WITH latest_balances AS (
  SELECT
    json_extract_scalar(key_decoded, '$.vec[1].address') AS holder,
    TRY_CAST(json_extract_scalar(val_decoded, '$.i128') AS DOUBLE) / 1e7 AS balance,
    ROW_NUMBER() OVER (
      PARTITION BY json_extract_scalar(key_decoded, '$.vec[1].address')
      ORDER BY ledger_sequence DESC
    ) AS rn
  FROM stellar.contract_data
  WHERE contract_id = '<STELLAR_CONTRACT_ID>'
    AND deleted = false
    AND contract_key_type = 'ScValTypeScvVec'
    AND json_extract_scalar(key_decoded, '$.vec[0].symbol') = 'Balance'
)
SELECT holder, balance
FROM latest_balances
WHERE rn = 1 AND balance > 0
```

> **Critical:** Always deduplicate with `ROW_NUMBER() ORDER BY ledger_sequence DESC` and keep only `rn = 1`. Stellar stores every historical version of each balance entry — summing without deduplication inflates balances by 40-100x.

---

## 5. Fill-forward prices (oracle NAV)

NAV oracle prices don't update every day. Use `LAST_VALUE ... IGNORE NULLS` to forward-fill:

```sql
WITH calendar AS (
  SELECT date_trunc('day', period) AS day
  FROM UNNEST(SEQUENCE(DATE '2024-01-01', CURRENT_DATE, INTERVAL '1' DAY)) AS t(period)
),
daily_prices_sparse AS (
  SELECT
    DATE_TRUNC('day', block_time) AS day,
    ARBITRARY(bytearray_to_uint256(bytearray_substring(data, 33, 32))) * 1e-6 AS nav_price
  FROM ethereum.logs
  WHERE contract_address = 0x<ORACLE_ADDRESS>
    AND topic0 = 0x<PRICE_UPDATE_EVENT>
    AND block_time >= TIMESTAMP '2024-01-01'
  GROUP BY 1
),
filled_prices AS (
  SELECT
    c.day,
    COALESCE(
      LAST_VALUE(p.nav_price) IGNORE NULLS OVER (ORDER BY c.day ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW),
      1.0  -- fallback default
    ) AS nav_price
  FROM calendar c
  LEFT JOIN daily_prices_sparse p ON c.day = p.day
)
SELECT * FROM filled_prices ORDER BY day
```

---

## 6. Incremental query pattern (materialized view)

For queries that get expensive over time, use Dune's incremental pattern to only scan new data on each run:

```sql
WITH prev AS (
  SELECT * FROM TABLE(previous.query.result(
    schema => DESCRIPTOR(
      day        TIMESTAMP(3),
      product    VARCHAR,
      net_supply DOUBLE
    )
  ))
),
checkpoint AS (
  SELECT
    COALESCE(MAX(day), TIMESTAMP '2024-01-01') - INTERVAL '3' DAY AS cutoff,
    CAST(COALESCE(MAX(day), TIMESTAMP '2024-01-01') - INTERVAL '3' DAY AS DATE) AS cutoff_date
  FROM prev
),
-- ... CTEs for new data, filtered by checkpoint ...
new_data AS (
  SELECT day, product, net_supply
  FROM your_logic_here
  WHERE day >= (SELECT cutoff_date FROM checkpoint)
)
-- Final: union old results with new data
SELECT * FROM prev
WHERE day < (SELECT cutoff FROM checkpoint)
UNION ALL
SELECT * FROM new_data
```

**Key rules:**
- The `DESCRIPTOR` must match your output columns and types exactly
- Include a 3-day lookback window to handle delayed data ingestion
- For cumulative metrics (running totals), seed from the last known value in `prev` rather than recomputing from zero
- Stellar queries always need a full recompute (LAG requires full history) — skip the incremental wrapper for Stellar CTEs

---

## 7. Day × series matrix (prevent gaps)

When a series has no data on certain days, LEFT JOIN against a full calendar to prevent gaps in charts:

```sql
WITH calendar AS (
  SELECT date_trunc('day', period) AS day
  FROM UNNEST(SEQUENCE(DATE '2024-01-01', CURRENT_DATE, INTERVAL '1' DAY)) AS t(period)
),
products AS (
  SELECT product FROM (VALUES ('TokenA'), ('TokenB'), ('TokenC')) AS t(product)
),
full_grid AS (
  SELECT c.day, p.product FROM calendar c CROSS JOIN products p
)
SELECT
  g.day,
  g.product,
  COALESCE(d.value, 0) AS value
FROM full_grid g
LEFT JOIN your_data d ON g.day = d.day AND g.product = d.product
ORDER BY 1, 2
```

---

## 8. Common multi-chain pitfalls

| Pitfall | Fix |
|---|---|
| Missing `blockchain` filter on `evms.*` | Always add `AND blockchain IN ('ethereum', ...)` — without it the query scans all chains |
| Summing Stellar balance rows without deduplication | Use `ROW_NUMBER() OVER (PARTITION BY holder ORDER BY ledger_sequence DESC)` and keep `rn = 1` |
| LAG inside a GROUP BY CTE | Trino forbids this — put the LAG in a separate CTE, then GROUP BY in the next one |
| `ORDER BY` without `LIMIT` on large tables | Sorting millions of rows with no limit is slow and expensive |
| Hardcoded FX rates | Use on-chain Chainlink/Redstone oracle feeds when available |
| Using `TIMESTAMP` keyword | Use `CAST('2024-01-01' AS timestamp)` or `DATE '2024-01-01'` instead |
| Bigint overflow in `gas_used * gas_price` | Cast both to `double` before multiplying |

---

## See Also

- [dunesql-cheatsheet.md](dunesql-cheatsheet.md) — Core DuneSQL syntax and functions
- [dataset-discovery.md](dataset-discovery.md) — Find the right table before writing SQL
- [query-management.md](query-management.md) — Save and manage queries via CLI
