# Materialized View Management

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Manage materialized views (matviews) -- query results persisted into a queryable table, optionally refreshed on a schedule. The command group is `dune matview` (alias `mv`).

```bash
dune matview <subcommand> [flags]
```

---

## Naming: bare vs. fully-qualified

A matview has two name forms, and the commands are **not** symmetric about which they take:

- **`create`** takes a **bare** name like `result_token_summary` (must start with `result_`, 8-128 lowercase alphanumerics/underscores, no trailing underscore). The name is immutable after creation.
- **`get`, `update`, `refresh`, `delete`** take the **fully-qualified** SQL name like `dune.my_team.result_token_summary`. The `create`/`update`/`refresh` responses return this fully-qualified name -- capture it from the JSON `name`/`sql_id` field for subsequent calls.

## Refresh schedules

If a matview has a cron schedule, it is refreshed periodically. Cron expressions are standard 5-field syntax (e.g. `0 */6 * * *`) with a **minimum 15-minute interval**. `expires-at` (RFC3339) caps when the schedule stops; it only applies when a schedule is set.

---

## matview create

Materialize a saved query into a table. Triggers an immediate execution.

```bash
dune matview create --name <NAME> --query-id <ID> [flags]
```

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--name` | `string` | Yes | -- | Bare matview name, e.g. `result_token_summary` |
| `--query-id` | `integer` | Yes | -- | ID of the source query to materialize |
| `--private` | `bool` | No | `false` | Make the matview private (requires a supporting plan); defaults to public |
| `--performance` | `string` | No | account default | Execution tier: `small`, `medium`, or `large` |
| `--cron` | `string` | No | none | 5-field cron for periodic refresh (min 15-minute interval) |
| `--expires-at` | `string` | No | none | RFC3339 time the schedule expires (requires `--cron`) |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Requirements

- The source query must be **saved (non-temporary)** and have **no parameters**.
- The query must be owned by the same user/team as the matview.
- The matview is owned by your authenticated context: a team API key creates a team-owned matview, a personal key a personal one.

### Output

- **text**: `Created materialized view <fully-qualified-name>` + the execution ID and a results hint.
- **json**: `{"name": "<fully-qualified-name>", "execution_id": "<id>"}`

### Examples

```bash
# Materialize a query at the medium tier
dune matview create --name result_token_summary --query-id 12345 --performance medium -o json

# Scheduled refresh every 6 hours
dune matview create --name result_daily --query-id 12345 --cron "0 */6 * * *" -o json

# Private matview (public by default; --private requires a supporting plan)
dune matview create --name result_private --query-id 12345 --private -o json

# Capture the fully-qualified name for later commands
MV=$(dune matview create --name result_token_summary --query-id 12345 -o json | jq -r '.name')
```

### Tips

- `create` is **create-or-replace**: if a matview with this name already exists for the same query it is re-run. To change settings on an existing matview, use `dune matview update`.
- Matviews are public by default; `--private` requires a supporting plan.

> [!CAUTION]
> This is a **write** command -- it creates a resource and triggers a (credit-consuming) execution.

---

## matview get

Fetch a matview's metadata, table size, recent executions, and refresh schedule.

```bash
dune matview get <name>
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `name` | `string` | Fully-qualified SQL name, e.g. `dune.my_team.result_token_summary` |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Output

**text** format displays the metadata and, if scheduled, the refresh schedule:

```
Name:        dune.my_team.result_token_summary
Query ID:    12345
Private:     false
Table size:  2.0 KB
Executions:  01HZ...

Refresh schedule:
  Cron:        0 */6 * * *
  Performance: small
  Next run:    2026-06-12T12:41:12Z
  Expires at:  2026-09-11T23:59:59Z
```

When there is no schedule, the last line reads `Refresh schedule: none`.

**json** returns the full object: `sql_id`, `query_id`, `is_private`, `last_execution_ids`, `table_size_bytes`, and `schedule` (`null` when none).

### Examples

```bash
dune matview get dune.my_team.result_token_summary -o json

# Just the schedule
dune matview get dune.my_team.result_token_summary -o json | jq '.schedule'
```

---

## matview list

List the materialized views owned by the authenticated user or team (paginated).

```bash
dune matview list [flags]
```

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--limit` | `integer` | No | `100` | Max matviews per page (max 1000) |
| `--offset` | `integer` | No | `0` | Pagination offset |
| `--all` | `bool` | No | `false` | Fetch all pages (ignores `--offset`) |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Output

- **text**: an aligned table with `NAME`, `QUERY ID`, `PRIVATE`, `SIZE`; a hint with the next offset when more pages exist.
- **json**: `{"materialized_views": [...], "next_offset": <int>}` (`next_offset` omitted when there are no more pages).

### Examples

```bash
dune matview list -o json
dune matview list --all -o json | jq '.materialized_views | length'
```

---

## matview update

Update a matview's privacy, performance tier, or refresh schedule. Updating re-executes the source query. The name and source query are immutable.

```bash
dune matview update <name> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `name` | `string` | Fully-qualified SQL name, e.g. `dune.my_team.result_token_summary` |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--private` | `bool` | No | preserved | Set the matview's privacy |
| `--performance` | `string` | No | preserved | Execution tier: `small`, `medium`, or `large` |
| `--cron` | `string` | No | preserved | Set or replace the refresh schedule (min 15-minute interval) |
| `--no-schedule` | `bool` | No | -- | Remove the refresh schedule |
| `--expires-at` | `string` | No | preserved | RFC3339 time the schedule expires |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

`--cron` and `--no-schedule` are mutually exclusive.

### Schedule preservation (important)

`update` is **read-modify-write**: it fetches the current matview first, then re-submits the full state. Settings you don't pass are preserved -- **in particular, the existing refresh schedule is kept** unless you change it with `--cron` or remove it with `--no-schedule`. So `dune matview update <name> --private=false` will *not* silently drop a schedule.

### Output

- **text**: `Updated materialized view <fully-qualified-name>` + the execution ID.
- **json**: `{"name": "<fully-qualified-name>", "execution_id": "<id>"}`

### Examples

```bash
# Change the tier, keep the schedule
dune matview update dune.my_team.result_token_summary --performance large -o json

# Replace the schedule
dune matview update dune.my_team.result_token_summary --cron "0 0 * * *" -o json

# Remove the schedule
dune matview update dune.my_team.result_token_summary --no-schedule -o json

# Make it public
dune matview update dune.my_team.result_token_summary --private=false -o json
```

> [!CAUTION]
> This is a **write** command -- it modifies the matview and triggers a (credit-consuming) re-execution.

---

## matview refresh

Trigger an on-demand refresh: re-execute the source query and update the table.

```bash
dune matview refresh <name> [flags]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `name` | `string` | Fully-qualified SQL name, e.g. `dune.my_team.result_token_summary` |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--performance` | `string` | No | account default | Execution tier: `small`, `medium`, or `large` |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Output

- **text**: `Refreshing materialized view <fully-qualified-name>` + the execution ID and a results hint.
- **json**: `{"sql_id": "<fully-qualified-name>", "execution_id": "<id>"}`

### Examples

```bash
dune matview refresh dune.my_team.result_token_summary -o json
dune matview refresh dune.my_team.result_token_summary --performance large -o json
```

> [!CAUTION]
> This is a **write** command -- it consumes credits based on the compute used. You must own the matview.

---

## matview delete

Permanently delete a matview and its refresh schedule. Drops the underlying table.

```bash
dune matview delete <name>
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `name` | `string` | Fully-qualified SQL name, e.g. `dune.my_team.result_token_summary` |

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Output

- **text**: `Deleted materialized view <fully-qualified-name>`
- **json**: `{"message": "ok"}`

### Examples

```bash
dune matview delete dune.my_team.result_token_summary -o json
```

> [!CAUTION]
> This is a **destructive, irreversible** command -- the table can no longer be queried. You must own the matview.

---

## Common Workflow

### Materialize a query, then schedule and manage it

```bash
# 1. Create a saved (non-temp, non-parameterized) source query
QUERY_ID=$(dune query create --name "Token Summary" --sql "SELECT 1 AS x" -o json | jq -r '.query_id')

# 2. Materialize it with a 6-hour refresh schedule; capture the fully-qualified name
MV=$(dune matview create --name result_token_summary --query-id $QUERY_ID --cron "0 */6 * * *" -o json | jq -r '.name')

# 3. Inspect it (shows the schedule)
dune matview get $MV -o json

# 4. Refresh on demand
dune matview refresh $MV -o json

# 5. Remove the schedule but keep the matview
dune matview update $MV --no-schedule -o json

# 6. Delete when no longer needed
dune matview delete $MV -o json
```

---

## See Also

- [query-management.md](query-management.md) -- Create the source query to materialize
- [query-execution.md](query-execution.md) -- Fetch results of the execution a create/refresh triggers
- [docs-and-usage.md](docs-and-usage.md) -- Check credit usage consumed by refreshes
