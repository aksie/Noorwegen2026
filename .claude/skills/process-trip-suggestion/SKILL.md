---
name: process-trip-suggestion
description: Processes GitHub Issues labeled suggestion and adds restaurants or activities to trip-data.json for the Noorwegen2026 trip site. Use when the user asks to process a suggestion issue, pending trip tips, or references an issue number with restaurant/activity content.
---

# Process trip suggestion

Turn GitHub Issues into edits in `trip-data.json`. Issues are created by the iOS Shortcut or the trip-suggestion issue template.

## When invoked

1. If the user gives an issue number, process that issue only.
2. Otherwise list open issues: `gh issue list --repo aksie/Noorwegen2026 --label suggestion --label pending --state open`
3. Process one issue at a time; confirm with the user before committing.

## Read the issue

```bash
gh issue view <number> --repo aksie/Noorwegen2026
```

Parse from the body (markdown fields):

| Field | Required | Notes |
|-------|----------|-------|
| Raw message | yes | The suggestion text |
| City | yes | Lowercase slug or display name |
| Type | yes | `restaurant` or `activity` |
| Day id | no | Integer index into `days[]`; use mapping below if missing |

Labels on the issue also indicate type: `restaurant` or `activity`.

## City → day id

Use the **first day** of each stay (where `restaurants` lists already exist on the site):

| City | day `id` | `phase` |
|------|----------|---------|
| Hamburg | 0 | reis |
| Bergen | 2 | bergen |
| Baldersheim / Bjørnafjorden | 4 | bald |
| Vik / Sognefjord | 8 | vik |
| Flåm / Aurland | 12 | flam |
| Oslo | 17 | oslo |

If city is unclear, ask the user. Do not guess wrong day.

## Find a URL

1. Search for the place name + city (official site or Google Maps).
2. Prefer official website; fallback `https://maps.google.com/?q=...`
3. If ambiguous, show options and ask the user.

## Edit trip-data.json

Open `trip-data.json`, find the day object where `"id"` matches the day id.

### Restaurant

Ensure `restaurants` array exists on that day. Append one line (do not duplicate names already listed):

```json
"[Name](https://url) — short note · AI"
```

- Use markdown link syntax (site renders markdown).
- Always append ` · AI` unless the issue or user says it was personally vetted (then omit ` · AI`).

### Activity

Append one string to the `activities` array:

```json
"**[Name](https://url)** — short note"
```

Or plain prose with `**bold**` for highlights. Split into multiple array items only if the user wants separate time blocks (Ochtend/Middag/Avond).

## Show diff and wait

1. Show the exact JSON lines added.
2. Ask: “Add this and commit?” (or user may say push as well).
3. Only then edit, commit, and push.

Suggested commit message: `Add [Bergen] Bryggeloftet from suggestion #12`

## Close the issue

After a successful push:

```bash
gh issue close <number> --repo aksie/Noorwegen2026 --comment "Added to trip-data.json in <commit-sha>"
```

Remove label `pending` is optional; closing is enough.

## Example

Issue body:

```markdown
**Raw message:** Bryggeloftet — traditioneel vis, aanrader
**City:** bergen
**Day id:** 2
**Type:** restaurant
```

Result on day id 2:

```json
"[Bryggeloftet](https://bryggeloftet.no/) — traditioneel vis, aanrader · AI"
```
