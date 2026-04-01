# Arch Radar Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create four Claude Code user skills for managing the ARCOS Architecture Radar (add, update, review, roll quarter).

**Architecture:** Four standalone markdown files in `~/.claude/commands/`, each containing a complete prompt with shared preamble and skill-specific instructions. No code dependencies — these are pure prompt files.

**Tech Stack:** Claude Code custom slash commands (markdown files)

---

### Task 1: Create commands directory and `/radar-add` skill

**Files:**
- Create: `~/.claude/commands/radar-add.md`

- [ ] **Step 1: Create the commands directory**

```bash
mkdir -p ~/.claude/commands
```

- [ ] **Step 2: Write the `/radar-add` skill file**

Create `~/.claude/commands/radar-add.md` with the following content:

````markdown
---
description: Add a new blip to the ARCOS Architecture Radar
---

# Radar Add

Add a new technology blip to the current quarter's Architecture Radar JSON file.

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

1. **Identify the current quarter file** by parsing `qN-YYYY.json` filenames in the repo root (sort by year, then quarter).

2. **Parse user input.** The user may provide:
   - All details: "Add Playwright, assess, languages & frameworks — end-to-end testing framework"
   - Partial details: "Add Playwright" or "Playwright, assess"
   - No details (just invoked `/radar-add`)
   For any missing fields, ask one at a time in this order: name, quadrant, ring, description.

3. **Check for duplicates.** Search the current quarter file for a matching name (case-insensitive). If found, warn the user and suggest `/radar-update` instead. Do not add a duplicate.

4. **Add the entry** with `"isNew": "TRUE"` set automatically.

5. **Write the updated JSON** back to the file — pretty-printed with 2-space indentation.

6. **Show confirmation:** Display the added entry and output a preview link:
   `https://radar.arcos-inc.com/?documentId=https://raw.githubusercontent.com/sball-at-arcos/arch-radar/refs/heads/{current-branch}/{filename}`

7. **Staleness check.** After adding, compare all entries in the current quarter file against prior quarter files. Flag entries where:
   - `isNew` is `"FALSE"`
   - The ring AND description are identical (or near-identical) across 2+ consecutive quarter files
   Present stale entries as a summary table with columns: Name, Ring, Quarters Unchanged, Suggested Action. Do NOT auto-modify — just report for the user to act on.
````

- [ ] **Step 3: Verify the file was created**

```bash
cat ~/.claude/commands/radar-add.md | head -5
```

Expected output starts with:
```
---
description: Add a new blip to the ARCOS Architecture Radar
---
```

- [ ] **Step 4: Commit**

```bash
cd /Users/sball/dev/arch-radar
git add --all
git commit -m "feat: add /radar-add skill for Architecture Radar"
```

---

### Task 2: Create `/radar-update` skill

**Files:**
- Create: `~/.claude/commands/radar-update.md`

- [ ] **Step 1: Write the `/radar-update` skill file**

Create `~/.claude/commands/radar-update.md` with the following content:

````markdown
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
````

- [ ] **Step 2: Verify the file was created**

```bash
cat ~/.claude/commands/radar-update.md | head -5
```

Expected output starts with:
```
---
description: Update an existing blip on the ARCOS Architecture Radar
---
```

- [ ] **Step 3: Commit**

```bash
cd /Users/sball/dev/arch-radar
git add --all
git commit -m "feat: add /radar-update skill for Architecture Radar"
```

---

### Task 3: Create `/radar-review` skill

**Files:**
- Create: `~/.claude/commands/radar-review.md`

- [ ] **Step 1: Write the `/radar-review` skill file**

Create `~/.claude/commands/radar-review.md` with the following content:

````markdown
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
````

- [ ] **Step 2: Verify the file was created**

```bash
cat ~/.claude/commands/radar-review.md | head -5
```

Expected output starts with:
```
---
description: Review and analyze the ARCOS Architecture Radar — summary, diff, and staleness
---
```

- [ ] **Step 3: Commit**

```bash
cd /Users/sball/dev/arch-radar
git add --all
git commit -m "feat: add /radar-review skill for Architecture Radar"
```

---

### Task 4: Create `/radar-roll` skill

**Files:**
- Create: `~/.claude/commands/radar-roll.md`

- [ ] **Step 1: Write the `/radar-roll` skill file**

Create `~/.claude/commands/radar-roll.md` with the following content:

````markdown
---
description: Roll a new quarter for the ARCOS Architecture Radar
---

# Radar Roll

Create a new quarter's Architecture Radar file from the most recent one, with auto-detection and confirmation.

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

### Phase 1: Auto-detect and confirm

1. **Scan the repo root** for all `qN-YYYY.json` files. Parse the quarter (N) and year (YYYY) from each filename.
2. **Sort by year then quarter** to find the latest file.
3. **Calculate the next quarter:**
   - q1 → q2 (same year)
   - q2 → q3 (same year)
   - q3 → q4 (same year)
   - q4 → q1 (next year)
4. **Present the plan to the user:**
   > Rolling from `q3-2026.json` → `q4-2026.json`. This will:
   > - Copy N entries (carry forward all non-archived blips)
   > - Set all `isNew` to `"FALSE"`
   > - Remove N archived entries
   >
   > Proceed?

5. **Wait for user confirmation.** Do NOT create the file until confirmed.

### Phase 2: Create the new quarter file

1. **Read the source file** (latest quarter).
2. **Copy all entries**, applying these transformations:
   - Set `"isNew"` to `"FALSE"` on every entry
   - Remove any entry where ring is `"Archive"` or `"Archived"` (case-insensitive)
3. **Write the new file** with the calculated filename — pretty-printed with 2-space indentation.

### Phase 3: Post-roll actions

1. **Staleness analysis.** Compare across ALL available quarter files (including the new one). Flag entries where the ring AND description are identical across 2+ consecutive quarter files:
   - 2 consecutive quarters = "Worth reviewing"
   - 3+ consecutive quarters = "Likely stale"
   Present as a table: Name | Ring | Quadrant | Quarters Unchanged | Suggested Action.

2. **Suggest a Git branch** following existing convention: `feature/qN-YYYY` (e.g., `feature/q4-2026`).

3. **Output the preview link:**
   `https://radar.arcos-inc.com/?documentId=https://raw.githubusercontent.com/sball-at-arcos/arch-radar/refs/heads/{suggested-branch}/{new-filename}`
````

- [ ] **Step 2: Verify the file was created**

```bash
cat ~/.claude/commands/radar-roll.md | head -5
```

Expected output starts with:
```
---
description: Roll a new quarter for the ARCOS Architecture Radar
---
```

- [ ] **Step 3: Commit**

```bash
cd /Users/sball/dev/arch-radar
git add --all
git commit -m "feat: add /radar-roll skill for Architecture Radar"
```

---

### Task 5: End-to-end verification

- [ ] **Step 1: Verify all four skills are installed**

```bash
ls -la ~/.claude/commands/radar-*.md
```

Expected: four files listed:
```
radar-add.md
radar-review.md
radar-roll.md
radar-update.md
```

- [ ] **Step 2: Verify each file has valid frontmatter**

```bash
for f in ~/.claude/commands/radar-*.md; do echo "=== $(basename $f) ==="; head -3 "$f"; echo; done
```

Expected: each file starts with `---`, has a `description:` line, and closes with `---`.

- [ ] **Step 3: Commit the plan document**

```bash
cd /Users/sball/dev/arch-radar
git add docs/superpowers/plans/2026-04-01-arch-radar-skills.md
git commit -m "docs: add arch-radar skills implementation plan"
```
