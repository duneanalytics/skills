# Color Palettes for Dune Charts

## Using custom colors

Set `chartStyles: { colorPalettes: 'custom' }` in the visualization options, then assign a color to each series in `seriesOptions`:

```json
{
  "chartStyles": { "colorPalettes": "custom" },
  "seriesOptions": {
    "Series A": { "type": "area", "color": "#2376FB", "yAxis": 0, "zIndex": 0 },
    "Series B": { "type": "area", "color": "#FF8B3D", "yAxis": 0, "zIndex": 1 }
  }
}
```

## Wallet size / investor distribution buckets

Blue gradient from light (small) to dark (large). Requires `colorPalettes: 'custom'` — `autoBrandColors` ignores colors for bucket labels.

```json
{
  "chartStyles": { "colorPalettes": "custom" },
  "seriesOptions": {
    "Micro (<1K)":    { "type": "column", "color": "#CFE0FB", "yAxis": 0, "zIndex": 0 },
    "Small (1K-10K)": { "type": "column", "color": "#84ABE8", "yAxis": 0, "zIndex": 1 },
    "Mid (10K-100K)": { "type": "column", "color": "#2376FB", "yAxis": 0, "zIndex": 2 },
    "Large (100K-1M)":{ "type": "column", "color": "#0D52C9", "yAxis": 0, "zIndex": 3 },
    "Whale (>1M)":    { "type": "column", "color": "#020202", "yAxis": 0, "zIndex": 4 }
  }
}
```

## Multi-chain breakdown (chain colors)

Widely-used chain color conventions for consistent cross-dashboard styling:

```json
{
  "chartStyles": { "colorPalettes": "custom" },
  "seriesOptions": {
    "ethereum":  { "type": "area", "color": "#627EEA", "yAxis": 0, "zIndex": 0 },
    "arbitrum":  { "type": "area", "color": "#28A0F0", "yAxis": 0, "zIndex": 1 },
    "polygon":   { "type": "area", "color": "#8247E5", "yAxis": 0, "zIndex": 2 },
    "base":      { "type": "area", "color": "#0052FF", "yAxis": 0, "zIndex": 3 },
    "optimism":  { "type": "area", "color": "#FF0420", "yAxis": 0, "zIndex": 4 },
    "bnb":       { "type": "area", "color": "#F0B90B", "yAxis": 0, "zIndex": 5 },
    "avalanche": { "type": "area", "color": "#E84142", "yAxis": 0, "zIndex": 6 },
    "solana":    { "type": "area", "color": "#9945FF", "yAxis": 0, "zIndex": 7 },
    "stellar":   { "type": "area", "color": "#000000", "yAxis": 0, "zIndex": 8 }
  }
}
```

## Single-product gradient (light to dark)

Use when you want visual depth on a single product/category across multiple metrics:

```json
{
  "chartStyles": { "colorPalettes": "custom" },
  "seriesOptions": {
    "metric_1": { "type": "column", "color": "#CFE0FB", "yAxis": 0, "zIndex": 0 },
    "metric_2": { "type": "column", "color": "#84ABE8", "yAxis": 0, "zIndex": 1 },
    "metric_3": { "type": "column", "color": "#2376FB", "yAxis": 0, "zIndex": 2 },
    "metric_4": { "type": "column", "color": "#0D52C9", "yAxis": 0, "zIndex": 3 }
  }
}
```

## Notes on zIndex

`zIndex` controls the draw order of series (higher = drawn on top). For stacked charts, set `zIndex` to match the visual stack order. For non-stacked charts, it controls which series appears in front when they overlap.
