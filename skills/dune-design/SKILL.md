---
name: dune-design
description: "Restyle Dune dashboard charts via browser JavaScript — change colors, fix palettes, update seriesOptions, and batch-apply brand colors across visualizations. Use this skill whenever asked to: update chart colors, fix inconsistent palettes, apply brand colors to a Dune dashboard, change a chart type, update visualization options (stacking, legend, axis labels, number formats), or make a Dune dashboard look consistent. This skill works directly in the browser without needing a Dune API key — it reuses the JWT from the user's active session."
compatibility: Requires the Claude in Chrome extension and an active Dune browser tab where the user is logged in. Works on any OS.
allowed-tools: mcp__Claude_in_Chrome__javascript_tool mcp__Claude_in_Chrome__computer mcp__Claude_in_Chrome__navigate mcp__Claude_in_Chrome__tabs_context_mcp
metadata:
  author: achillekrtf
  version: "1.0.0"
---

# Dune Design Skill

Update Dune chart visualizations (colors, styles, options) by calling Dune's internal GraphQL API directly from the user's browser tab. No API key required — the skill reuses the JWT already held by the browser session.

## How it works

Dune's chart editor calls a `EditVisual` GraphQL mutation at `https://dune.com/public/graphql`. This is a same-domain endpoint (no CORS). The skill:
1. Installs a `fetch` interceptor in the Dune tab to capture the JWT when the next mutation fires
2. Triggers one throwaway UI mutation (color picker change) to harvest the JWT
3. Uses those headers to batch-fire real `EditVisual` mutations with the correct options

> **Why not use the Dune CLI or REST API?** The `api.dune.com` endpoint is CORS-blocked from browser JS, and the Dune CLI does not expose chart color or `seriesOptions` editing. The internal GraphQL API is the only way to update these fields programmatically.

---

## Prerequisites

- Claude in Chrome extension active, with the user on any `dune.com` page
- The target visualization must be owned by the logged-in user
- Find viz IDs by opening a chart's editor — the viz ID appears in the URL: `https://dune.com/queries/<queryId>/<vizId>`

---

## Step-by-step workflow

### 1. Get the tab ID

```js
// Use tabs_context_mcp to find the dune.com tab
```

### 2. Navigate to the target query/viz page

```
https://dune.com/queries/<queryId>/<vizId>
```

### 3. Install the fetch interceptor

Run once — patches `window.fetch` to log every `EditVisual` mutation to `window._gqlLog`:

```javascript
window._gqlLog = [];
const _origFetch = window.fetch;
window.fetch = function(...args) {
  const [url, opts] = args;
  if (url && url.toString().includes('EditVisual') && opts && opts.body) {
    window._gqlLog.push({
      url: url.toString(),
      headers: JSON.stringify(Object.fromEntries(
        (opts.headers instanceof Headers)
          ? opts.headers.entries()
          : Object.entries(opts.headers || {})
      )),
      body: opts.body
    });
  }
  return _origFetch.apply(this, args);
};
'interceptor installed';
```

### 4. Trigger a throwaway mutation to capture auth headers

Open the color picker in the chart editor UI, then trigger a value change to fire a mutation:

```javascript
const inp = document.querySelector('.OptionInputColor-module__UcMDpW__colorInput');
const setter = Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value').set;
setter.call(inp, '#FFFFFF');
inp.dispatchEvent(new Event('input', { bubbles: true }));
inp.dispatchEvent(new Event('change', { bubbles: true }));
inp.dispatchEvent(new KeyboardEvent('keydown', { key: 'Enter', keyCode: 13, bubbles: true }));
inp.dispatchEvent(new KeyboardEvent('keyup',  { key: 'Enter', keyCode: 13, bubbles: true }));
```

**Alternative:** manually click any color swatch in the Dune UI — it fires the mutation automatically.

### 5. Extract the auth headers

```javascript
const captured = window._gqlLog[0];
// Headers are serialized as a JSON string — always JSON.parse() first
const h = JSON.parse(captured.headers);
// h.authorization       → Bearer JWT
// h['x-dune-access-token'] → JWT
```

### 6. Fire the EditVisual mutation

```javascript
const GQL = 'https://dune.com/public/graphql?operationName=EditVisual';
const MUTATION = `mutation EditVisual($input: UpdateVisualizationInput!) {
  updateVisualization(input: $input) { id type name options __typename }
}`;

const result = await fetch(GQL, {
  method: 'POST',
  headers: {
    'accept':               h['accept'],
    'content-type':         'application/json',
    'authorization':        h['authorization'],
    'x-dune-access-token':  h['x-dune-access-token']
  },
  body: JSON.stringify({
    operationName: 'EditVisual',
    variables: {
      input: {
        id:          <vizId>,        // integer
        type:        'chart',
        name:        'Chart Name',   // required — omitting causes a 400 error
        description: '',
        options:     <optionsObject>
      }
    },
    extensions: { clientLibrary: { name: '@apollo/client', version: '4.1.6' } },
    query: MUTATION
  })
}).then(r => r.json());

result.data?.updateVisualization?.id  // confirm success
```

### 7. Batch-update multiple charts

Reuse the same captured headers — they stay valid for the whole session:

```javascript
const updates = [
  { id: 12345, name: 'TVL by Product', options: { ...TVL_OPTS } },
  { id: 12346, name: 'Holder Count',   options: { ...HOLDER_OPTS } },
];

for (const u of updates) {
  const res = await fetch(GQL, {
    method: 'POST',
    headers: { 'accept': h.accept, 'content-type': 'application/json',
                'authorization': h.authorization, 'x-dune-access-token': h['x-dune-access-token'] },
    body: JSON.stringify({
      operationName: 'EditVisual',
      variables: { input: { id: u.id, type: 'chart', name: u.name, description: '', options: u.options } },
      extensions: { clientLibrary: { name: '@apollo/client', version: '4.1.6' } },
      query: MUTATION
    })
  }).then(r => r.json());
  console.log(u.name, res.data?.updateVisualization?.id ? 'OK' : res.errors);
}
```

---

## chartStyles.colorPalettes

The `colorPalettes` field in `chartStyles` controls how `seriesOptions[name].color` is applied:

| Value | Behavior |
|---|---|
| `'custom'` | Respects `seriesOptions[name].color` for **all** series names |
| `'autoBrandColors'` | Only applies `seriesOptions.color` for recognized product names. For generic labels (e.g. `"Micro (<1K)"`, `"bucket_1"`), color is ignored. |
| `'defaultColors'` | Dune assigns colors sequentially — no custom control |

**Rule:** Use `'custom'` for any chart with bucket labels, cohort names, or non-product series. Use `'autoBrandColors'` only when all series names are recognized Dune product/protocol names.

---

## Reading current options before overwriting

Always fetch the current options before replacing them to avoid losing axis config or number formats:

```javascript
const res = await fetch('https://dune.com/public/graphql?operationName=GetVisualization', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({
    operationName: 'GetVisualization',
    variables: { id: <vizId> },
    query: `query GetVisualization($id: Int!) {
      visualization(id: $id) { id name type options }
    }`
  })
}).then(r => r.json());
console.log(JSON.stringify(res.data.visualization.options, null, 2));
```

---

## Known gotchas

1. **`name` is required.** Omitting it causes `"Field 'name' of required type 'String!' was not provided"`.
2. **Headers are a JSON string.** Always `JSON.parse(captured.headers)` before accessing `.authorization`.
3. **`autoBrandColors` ignores custom colors for non-product series.** Distribution / cohort charts with bucket labels must use `colorPalettes: 'custom'`.
4. **`api.dune.com` is CORS-blocked.** Always use `dune.com/public/graphql` (same-domain).
5. **JWT expires with the browser session.** If mutations return 401, the user needs to refresh their login.
6. **Color picker class name may change.** Use `document.querySelector('input[value^="#"]')` as a fallback if the module class doesn't match.
7. **`seriesOptions` keys must match query output exactly.** Check the column alias in the SQL to get the real series label strings.

---

## References

- [`references/color-palettes.md`](references/color-palettes.md) — Common color palette patterns and seriesOptions templates
- [`references/viz-options.md`](references/viz-options.md) — Complete `options` objects for every chart type (stacked bar, area, line, counter, pie, scatter)
