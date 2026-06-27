# iOS Shortcut — trip suggestion → GitHub Issue

Capture a restaurant or activity tip on your phone and create a GitHub Issue for Claude Code to process later.

---

## Quick install (recommended)

Use the pre-built file in this repo.

1. Complete **Part 1** (GitHub token) and **Part 2** (labels) below.
2. Copy `shortcuts/Noorwegen-tip.shortcut` to your iPhone (AirDrop, iCloud Drive, or open from GitHub in Safari → download).
3. Tap the file → **Add Shortcut**.
4. In Shortcuts, open **Noorwegen tip** → first **Text** action → replace `PASTE_YOUR_GITHUB_TOKEN_HERE` with your token.
5. Tap **ⓘ** → **Show in Share Sheet** → enable **Text**.

---

## Part 1 — GitHub token (issues only, one repo)

Do this once on github.com (desktop or mobile browser).

### 1. Open token settings

1. GitHub profile photo → **Settings**
2. Left sidebar (bottom): **Developer settings**
3. **Personal access tokens** → **Fine-grained tokens**
4. **Generate new token**

### 2. Configure the token

| Setting | Value |
|---------|--------|
| Token name | `Noorwegen2026 iOS Shortcut` |
| Expiration | 90 days or Custom (your trip + buffer) |
| Resource owner | Your account (`aksie`) |
| Repository access | **Only select repositories** → tick **Noorwegen2026** |

### 3. Permissions (Repository permissions)

| Permission | Access |
|------------|--------|
| **Issues** | Read and write |
| Everything else | No access |

Metadata read is added automatically. You do **not** need Contents, Pull requests, or Actions.

### 4. Generate and copy

1. Click **Generate token**
2. Copy the token (`github_pat_...`) — you only see it once
3. Store it in a password manager; you will paste it into the Shortcut in Part 3

---

## Part 2 — Create labels on the repo

Labels must exist before the Shortcut can attach them. Run on your Mac (or GitHub.com → Issues → Labels):

```bash
gh label create suggestion --repo aksie/Noorwegen2026 --color 1D76DB --description "Trip tip for the site"
gh label create pending --repo aksie/Noorwegen2026 --color FBCA04 --description "Not yet added to trip-data.json"
gh label create restaurant --repo aksie/Noorwegen2026 --color F9D0C4 --description "Restaurant suggestion"
gh label create activity --repo aksie/Noorwegen2026 --color C5DEF5 --description "Activity suggestion"
```

Or create the same four labels manually in the repo **Issues → Labels** UI.

---

## Part 3 — Build the Shortcut manually (optional)

Only if you prefer not to use the pre-built `shortcuts/Noorwegen-tip.shortcut`.

App: **Shortcuts** on iPhone.

### 3.1 New shortcut

1. **Shortcuts** → **+** (new shortcut)
2. Name it: **Noorwegen tip**
3. Tap **ⓘ** (details):
   - Turn on **Show in Share Sheet**
   - Share Sheet Types: **Text** (and **URLs** if you want)
   - Optional: add to Home Screen

### 3.2 Store your token (private)

Add these actions at the **top** of the shortcut (before anything else):

1. **Text** — paste your `github_pat_...` token into the text field  
   - Rename the action result: tap **ⓘ** on the action → **Rename** → `GitHub Token`  
   - ⚠️ Never share or export this shortcut publicly

Alternative: run a one-time “Setup” shortcut that saves the token to a local file — the Text action above is simplest.

### 3.3 Ask for the suggestion text

1. **If** — `Shortcut Input` **has any value**  
   - **Then:** **Set variable** `Suggestion` = `Shortcut Input`  
   - **Otherwise:** **Ask for Input** → prompt “Wat is de tip?” → **Set variable** `Suggestion`

### 3.4 Choose city

1. **Choose from Menu** — prompt: **Stad**
   - Hamburg → **Set variable** `City` = `Hamburg`, `DayId` = `0`
   - Bergen → `City` = `Bergen`, `DayId` = `2`
   - Baldersheim → `City` = `Baldersheim`, `DayId` = `4`
   - Vik → `City` = `Vik`, `DayId` = `8`
   - Flåm → `City` = `Flåm`, `DayId` = `12`
   - Oslo → `City` = `Oslo`, `DayId` = `17`
   - Anders → **Ask for Input** “Stad?” → **Set variable** `City`, **Set variable** `DayId` = `` (empty text)

Tip: use **Choose from Menu** with each option containing two **Set Variable** actions in sequence.

### 3.5 Choose type

1. **Choose from Menu** — prompt: **Type**
   - Restaurant → **Set variable** `Type` = `restaurant`, `TypeLabel` = `restaurant`
   - Activiteit → **Set variable** `Type` = `activity`, `TypeLabel` = `activity`

### 3.6 Build issue title

1. **Text** action with:

```
[City] Suggestion
```

2. **Replace Text** — find `[City]`, replace with variable `City`
3. **Replace Text** — find `Suggestion`, replace with variable `Suggestion`  
   - Optional: add **Get Text from Input** → **Get first line** if titles get too long
4. **Set variable** `IssueTitle`

### 3.7 Build issue body

1. **Text** action (multi-line):

```
## Suggestion

**Raw message:**
SUGGESTION_TEXT

**City:** CITY
**Day id:** DAY_ID
**Type:** TYPE
**Source:** ios-shortcut

## Agent checklist
- [ ] Confirm city/day mapping
- [ ] Find URL
- [ ] Add to trip-data.json
- [ ] Append ` · AI` if not personally vetted
- [ ] Close issue when merged
```

2. **Replace Text** — `SUGGESTION_TEXT` → variable `Suggestion`
3. **Replace Text** — `CITY` → variable `City` (lowercase optional; skill accepts both)
4. **Replace Text** — `DAY_ID` → variable `DayId`
5. **Replace Text** — `TYPE` → variable `Type`
6. **Set variable** `IssueBody`

### 3.8 Build JSON for GitHub API

1. **Dictionary** action:
   - `title` → variable `IssueTitle`
   - `body` → variable `IssueBody`
   - `labels` → **List** with two items:
     - Text `suggestion`
     - Text `pending`
     - Text — variable `TypeLabel` (third item: add `restaurant` or `activity` as third list entry)

   In Shortcuts, build labels as a list:
   - **List** → `suggestion`, `pending`, then variable `Type`

2. **Get Dictionary from Input** (the dictionary above)
3. **Dictionary to JSON** (or **Get Text from Input** if your iOS version uses “Dictionary” → “Make JSON” from the dictionary action’s output)

4. **Set variable** `RequestBody`

### 3.9 POST to GitHub

1. **URL** action:

```
https://api.github.com/repos/aksie/Noorwegen2026/issues
```

2. **Get contents of URL**
   - Method: **POST**
   - Headers:
     - `Authorization` → `Bearer ` + variable `GitHub Token`  
       (Use **Text** action: `Bearer ` then **Combine** with token, or one Text: `Bearer YOUR_TOKEN` if not using variable combine)
     - `Accept` → `application/vnd.github+json`
     - `X-GitHub-Api-Version` → `2022-11-28`
   - Request Body: **File** or **JSON** → variable `RequestBody`

3. **Get Dictionary from Input** (response)
4. **Get Dictionary Value** key `html_url`
5. **Show notification** — “Tip opgeslagen” + URL  
   - Or **Show Result** / **Quick Look**

### 3.10 Authorization header (exact)

Use a **Text** action:

```
Bearer 
```

Then **Combine** with space… Actually in Shortcuts:

1. **Text**: `Bearer ` (with trailing space)
2. **Combine** — first item: that text, second: variable `GitHub Token`
3. Use combined text as `Authorization` header value

---

## Part 4 — Test

1. Run **Noorwegen tip** from the Shortcuts app
2. Enter: `Test café — ignore me`
3. Pick Bergen → Restaurant
4. Check https://github.com/aksie/Noorwegen2026/issues — new issue with labels `suggestion`, `pending`, `restaurant`
5. Close/delete the test issue

---

## Part 5 — Process with Claude Code

On your Mac (in a clone of this repo):

```bash
cd path/to/Noorwegen2026
claude
```

Then:

```
/process-trip-suggestion
```

Or: “Process issue #N”

The skill reads the issue, edits `trip-data.json`, shows a diff, commits after you approve, and closes the issue.

Deep link (Mac, after `claude` has been run in this repo once):

```
claude-cli://open?repo=aksie/Noorwegen2026&q=Process%20all%20pending%20suggestion%20issues%20using%20process-trip-suggestion%20skill
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| API returns 401 | Token wrong or expired; regenerate fine-grained token |
| API returns 404 | Repo name typo; must be `aksie/Noorwegen2026` |
| Labels error | Create all four labels (Part 2) |
| “Resource not accessible” | Token missing **Issues: Read and write** on this repo only |
| Shortcut not in Share Sheet | Enable **Show in Share Sheet** in shortcut details |
