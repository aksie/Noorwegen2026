# Noorwegen2026

One-page trip site (GitHub Pages). Trip content lives in `trip-data.json`; `index.html` loads it and renders the UI.

## Key files

- `trip-data.json` — days, activities (markdown arrays), optional `restaurants` per day
- `index.html` — layout and rendering (markdown links, AI badges on restaurants)

## Suggestions workflow

Travel tips arrive as GitHub Issues (label `suggestion`), often from the iOS Shortcut.

To process them, use the **`process-trip-suggestion`** skill:

```
/process-trip-suggestion
```

Or: “Process issue #12” / “Process all pending suggestion issues”.

Do not commit changes unless the user explicitly approves.
