---
description: Update an existing blip on the ARCOS Architecture Radar
---

# Radar Update

Modify an existing technology blip on the current quarter's Architecture Radar — change ring, quadrant, description, or any combination.

## Radar Context

- **Repo:** The arch-radar repo contains quarterly JSON files named `qN-YYYY.json` (e.g., `q3-2026.json`)
- **Renderer:** `https://radar.arcos-inc.com/?documentId=https://raw.githubusercontent.com/sball-at-arcos/arch-radar/refs/heads/{branch}/{filename}`
- **Current quarter detection:** Parse all `qN-YYYY.json` filenames in the repo root, sort by year then quarter number, and use the latest one. Do NOT rely on file modification dates.

## JSON Schema

Each file is a JSON array of blip objects:

```json
{
  "name": "Technology Name",
  "ring": "adopt | trial | assess | hold",
  "quadrant": "techniques | platforms | tools | languages & frameworks",
  "isNew": "TRUE | FALSE",
  "description": "HTML-allowed description string"
}
```

**Field rules:**
- **ring:** `adopt` (proven, use it), `trial` (try on a real project), `assess` (explore/POC), `hold` (stop new adoption)
- **quadrant:** process/pattern/methodology → `techniques`, infrastructure/runtime/managed service → `platforms`, dev tool/CLI/IDE plugin → `tools`, programming language/SDK/application framework → `languages & frameworks`
- **isNew:** String `"TRUE"` or `"FALSE"` (NOT boolean)
- **description:** May contain HTML links (`<a href>`). Should explain what the tech is and why it matters to ARCOS. Include Jira ticket links where applicable.

## Instructions

1. **Identify the current quarter file** by parsing `qN-YYYY.json` filenames in the repo root.

2. **Parse user input.** The user may provide:
   - Full update: "Move Playwright to trial — validated on Lighthouse regression suite"
   - Partial: "Update Playwright" or just the name
   - No details (just invoked `/radar-update`)
   If no entry name is given, ask for it.

3. **Find the entry.** Match the name case-insensitively with partial matching. If multiple entries match, present all candidates and ask the user to pick one. If no match, suggest `/radar-add`.

4. **Show the current entry** and ask what to change (if not already specified by the user).

5. **Ring change rationale.** If the ring is changing, ask the user for a rationale (if not already provided). Append it to the description, e.g.: "Moved from trial to adopt in Q3 2026 — proven in production on Lighthouse SSO integration."

6. **Write the updated JSON** back to the file — pretty-printed with 2-space indentation.

7. **Show a before/after diff** of the changed entry (display both the old and new JSON object).

8. **Output a preview link:**
   `https://radar.arcos-inc.com/?documentId=https://raw.githubusercontent.com/sball-at-arcos/arch-radar/refs/heads/{current-branch}/{filename}`
