---
description: Review and analyze the ARCOS Architecture Radar — summary, diff, and staleness
---

# Radar Review

Produce a comprehensive analysis of the current quarter's Architecture Radar: strategic summary, quarter-over-quarter diff, and staleness report.

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

**Ring values:** `adopt`, `trial`, `assess`, `hold` (also `Archive`/`Archived` for soft-deleted entries)
**Quadrant values:** `techniques`, `platforms`, `tools`, `languages & frameworks`

## Instructions

Read the current quarter file AND all prior quarter files in the repo. Then produce a single structured report with these three sections:

### Section 1: Strategic Summary

1. **Counts:** Total blips, count per quadrant, count per ring.
2. **By quadrant:** Group blips by quadrant. Within each quadrant, list entries grouped by ring (adopt first, then trial, assess, hold).
3. **New entries:** Highlight all entries where `isNew` is `"TRUE"` with a marker.
4. **Hold warnings:** Call out all `hold` ring entries prominently — these represent technologies the organization is actively moving away from.

### Section 2: Quarter-over-Quarter Diff

1. **Identify the previous quarter file** (the second-most-recent `qN-YYYY.json`). If none exists, skip this section and note that no prior quarter is available.
2. **New entries:** Blips in the current quarter that don't exist in the previous quarter (by name, case-insensitive).
3. **Removed entries:** Blips in the previous quarter that don't exist in the current quarter.
4. **Ring movements:** Entries that exist in both but changed ring. Show direction (e.g., "assess → trial" = promoted, "trial → hold" = demoted). Flag any movement to `hold` as significant.
5. **Description changes:** Entries where the description changed meaningfully (ignore trivial whitespace/punctuation differences).

### Section 3: Staleness Analysis

1. **Compare across ALL available quarter files** in the repo (not just current vs previous).
2. **Flag stale entries:** Entries where the ring AND description are identical (or near-identical) across 2+ consecutive quarter files, and `isNew` is `"FALSE"`.
3. **Severity:**
   - 2 consecutive quarters unchanged = "Worth reviewing"
   - 3+ consecutive quarters unchanged = "Likely stale"
4. **Present as a table:** Name | Ring | Quadrant | Quarters Unchanged | Suggested Action
5. **Suggested actions:** Update description to reflect current status, promote or demote ring based on new evidence, or archive if no longer relevant.

### Footer

End with the preview link:
`https://radar.arcos-inc.com/?documentId=https://raw.githubusercontent.com/sball-at-arcos/arch-radar/refs/heads/{current-branch}/{filename}`
