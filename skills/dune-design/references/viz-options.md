# Visualization Options Reference

Complete `options` objects for each Dune chart type. Copy-paste and adjust `columnMapping`, `seriesOptions`, and number formats for your query columns.

---

## Stacked bar chart (distribution / breakdown)

```json
{
  "sortX": true,
  "reverseX": false,
  "xAxis": { "type": "-" },
  "yAxis": [{ "type": "linear", "includeZero": true }],
  "legend": { "enabled": true },
  "series": { "stacking": "stack", "showTotal": true, "percentValues": false },
  "chartStyles": { "colorPalettes": "custom" },
  "globalSeriesType": "column",
  "numberFormat": "0,0",
  "numberFormatRightYAxisSeries": "0,0",
  "valuesOptions": {},
  "columnMapping": {
    "<x_column>": "x",
    "<series_column>": "series",
    "<value_column>": "y"
  },
  "seriesOptions": {
    "<series_name_1>": { "type": "column", "color": "#2376FB", "yAxis": 0, "zIndex": 0 },
    "<series_name_2>": { "type": "column", "color": "#0D52C9", "yAxis": 0, "zIndex": 1 }
  }
}
```

Set `"percentValues": true` for a 100% stacked view.

---

## Area / line timeseries

```json
{
  "sortX": false,
  "reverseX": false,
  "xAxis": { "type": "time" },
  "yAxis": [{ "type": "linear", "includeZero": true }],
  "legend": { "enabled": true },
  "series": { "stacking": "stack" },
  "chartStyles": { "colorPalettes": "custom" },
  "globalSeriesType": "area",
  "numberFormat": "$0,0.00a",
  "numberFormatRightYAxisSeries": "0,0",
  "valuesOptions": {},
  "columnMapping": {
    "day": "x",
    "value": "y",
    "series": "series"
  },
  "seriesOptions": {
    "<series_name>": { "type": "area", "color": "#2376FB", "yAxis": 0, "zIndex": 0 }
  }
}
```

For a **pure line** (no fill), change `"type": "line"` in each series and `"globalSeriesType": "line"`.

---

## Counter (KPI single number)

```json
{
  "counterColName": "<value_column>",
  "rowNumber": 1,
  "targetRowNumber": 1,
  "numberFormat": "$0,0.00a",
  "suffix": "",
  "prefix": "",
  "coloredPositiveValues": true
}
```

`counterColName` must match the exact column alias in the query output. `rowNumber` is 1-based.

---

## Bar chart (sorted leaderboard)

```json
{
  "sortX": true,
  "reverseX": true,
  "xAxis": { "type": "-" },
  "yAxis": [{ "type": "linear", "includeZero": true }],
  "legend": { "enabled": false },
  "series": { "stacking": null },
  "chartStyles": { "colorPalettes": "custom" },
  "globalSeriesType": "bar",
  "numberFormat": "$0,0.00a",
  "valuesOptions": {},
  "columnMapping": {
    "<label_column>": "x",
    "<value_column>": "y"
  },
  "seriesOptions": {
    "<series_name>": { "type": "bar", "color": "#2376FB", "yAxis": 0, "zIndex": 0 }
  }
}
```

---

## Pie chart

```json
{
  "globalSeriesType": "pie",
  "sortX": true,
  "showDataLabels": true,
  "chartStyles": { "colorPalettes": "custom" },
  "legend": { "enabled": true },
  "numberFormat": "0,0.00a",
  "columnMapping": {
    "<label_column>": "x",
    "<value_column>": "y"
  },
  "seriesOptions": {
    "<value_column>": { "type": "pie", "yAxis": 0, "zIndex": 0 }
  }
}
```

---

## Scatter chart

```json
{
  "globalSeriesType": "scatter",
  "xAxis": { "type": "linear" },
  "yAxis": [{ "type": "linear", "includeZero": false }],
  "legend": { "enabled": true },
  "chartStyles": { "colorPalettes": "custom" },
  "numberFormat": "0,0.00",
  "columnMapping": {
    "<x_column>": "x",
    "<y_column>": "y",
    "<series_column>": "series"
  },
  "seriesOptions": {}
}
```

---

## columnMapping roles

| Role | Meaning |
|---|---|
| `"x"` | X-axis (categories, dates, labels) |
| `"y"` | Y-axis value (one series per `"y"` column, or use `"series"` column for dynamic series) |
| `"series"` | Dynamic series name (one series per distinct value in this column) |
| `"size"` | Bubble size (scatter plots) |

---

## Numeral.js format strings

| Format | Output | Use case |
|---|---|---|
| `"0,0"` | 1,234 | Integer |
| `"0,0.00"` | 1,234.56 | Two decimals |
| `"0.0a"` | 1.2k | Abbreviated |
| `"$0,0.00a"` | $1.23m | USD abbreviated |
| `"0%"` | 50% | Percentage |
| `"0.00%"` | 50.00% | Percentage with decimals |
