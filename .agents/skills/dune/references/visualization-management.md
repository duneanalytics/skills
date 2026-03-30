# Visualization Management

> **Prerequisite:** Read the [main skill](../SKILL.md) for authentication, global flags, and key concepts.

Create and manage visualizations on saved Dune queries.

```bash
dune visualization <subcommand> [flags]
dune viz <subcommand> [flags]         # alias
```

---

## viz create

Create a new visualization attached to an existing saved query.

```bash
dune viz create --query-id <ID> --name <NAME> [flags]
```

### Flags

| Flag | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `--query-id` | `integer` | Yes | -- | ID of the query to attach the visualization to |
| `--name` | `string` | Yes | -- | Visualization name, max 300 characters |
| `--type` | `string` | No | `table` | Visualization type (see table below) |
| `--description` | `string` | No | `""` | Visualization description, max 1000 characters |
| `--options` | `string` | No | `{}` | JSON string of visualization-specific options |
| `-o, --output` | `string` | No | `text` | Output format: `text` or `json` |

### Visualization Types

| Type | Description |
|------|-------------|
| `chart` | Line, bar, area, scatter, or pie chart |
| `table` | Tabular data display |
| `counter` | Single-value counter display |
| `pivot` | Pivot table |
| `cohort` | Cohort analysis |
| `funnel` | Funnel visualization |
| `choropleth` | Geographic heat map |
| `sankey` | Flow diagram |
| `sunburst_sequence` | Hierarchical sunburst |
| `word_cloud` | Word cloud |

### Output

- **text**: `Created visualization <id> on query <query_id>`
- **json**: `{"id": <id>}`

### Examples

```bash
# Create a simple table visualization
dune viz create --query-id 12345 --name "Results Table" --type table -o json

# Create a chart visualization with options
dune viz create --query-id 12345 --name "Token Volume" --type chart \
  --options '{"globalSeriesType":"column","xColumn":"block_date","yColumn1":"volume"}' -o json

# Create a counter visualization
dune viz create --query-id 12345 --name "Total Transfers" --type counter \
  --options '{"counterColName":"count","counterRowNumber":1}' -o json

# Extract the visualization ID from JSON output
VIZ_ID=$(dune viz create --query-id 12345 --name "My Chart" --type chart -o json | jq -r '.id')
```

### Tips

- The `--options` JSON structure depends on the visualization type. Each type has its own configuration schema for axes, series, formatting, etc.
- You must have write access to the query to create a visualization on it.
- Use `-o json` to get the visualization ID for programmatic use.

> [!CAUTION]
> This is a **write** command -- it creates a resource in the user's Dune account.

---

## See Also

- [query-management.md](query-management.md) -- create and manage the queries that visualizations are attached to
- [query-execution.md](query-execution.md) -- execute queries to generate data for visualizations
