# Arch Radar Skills Design

## Overview

Four standalone Claude Code user skills (`~/.claude/commands/`) for managing the ARCOS Architecture Radar — a ThoughtWorks-style BYOR technology assessment tracked as quarterly JSON files.

## Context

- **Repo:** `/Users/sball/dev/arch-radar` (local), `sball-at-arcos/arch-radar` (GitHub)
- **Files:** `qN-YYYY.json` — one per quarter, JSON array of blip objects
- **Renderer:** `https://radar.arcos-inc.com/?documentId=https://raw.githubusercontent.com/sball-at-arcos/arch-radar/refs/heads/{branch}/{filename}`
- **Existing reference:** `arch-radar.skill` (ZIP) contains a SKILL.md for a different environment; this design replaces it with native Claude Code skills

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

- **ring:** `adopt` (proven, use it), `trial` (try on a real project), `assess` (explore/POC), `hold` (stop new adoption). Also `Archive`/`Archived` for soft-deleted entries (not rendered by BYOR).
- **quadrant:** process/pattern → `techniques`, infra/runtime → `platforms`, dev tool/CLI → `tools`, language/SDK/framework → `languages & frameworks`
- **isNew:** String `"TRUE"` or `"FALSE"` (not boolean). `TRUE` renders as triangle, `FALSE` as circle.
- **description:** May contain HTML links. Should explain what the tech is and why it matters to ARCOS. Include Jira ticket links where applicable.

## Skills

### Shared Preamble

All four skills include a common context block with: repo location, JSON schema, quadrant classification heuristics, renderer URL pattern, and current quarter detection logic (identify latest `qN-YYYY.json` by quarter/year parsing, not file modification date).

---

### 1. `/radar-add`

**Purpose:** Add a new blip to the current quarter's radar file.

**Behavior:**
- Accept flexible input — all details upfront, partial, or none (fully guided)
- If partial, ask for missing fields one at a time: name, quadrant, ring, description
- Parse natural language input (e.g., "Add Playwright as assess in languages & frameworks — end-to-end testing framework")
- Before adding, check if entry exists by name (case-insensitive). If it does, warn and suggest `/radar-update`
- Set `isNew` to `"TRUE"` automatically
- Write updated JSON back to file (pretty-printed, 2-space indent)
- Show the added entry as confirmation
- Output a preview link using the current branch name

**Post-add staleness check:**
- Compare entries against prior quarter files
- Flag entries with `isNew: "FALSE"` that have the same ring and identical/near-identical description across 2+ consecutive quarters
- Present stale entries as a summary for the user to act on (do not auto-modify)

---

### 2. `/radar-update`

**Purpose:** Modify an existing blip — change ring, quadrant, description, or any combination.

**Behavior:**
- Accept flexible input — entry name and changes upfront, or just the name
- Fuzzy match entry name against current quarter file (case-insensitive, partial match). If ambiguous, present candidates and ask
- If only a name is provided, show the current entry and ask what to change
- When changing rings, require a rationale and append it to the description (e.g., "Moved from trial to adopt — proven in production on Lighthouse SSO integration")
- Write updated JSON back, pretty-printed
- Show a before/after diff of the changed entry
- Output a preview link

---

### 3. `/radar-review`

**Purpose:** Analyze the current quarter's radar with strategic summary, quarter-over-quarter diff, and staleness analysis.

**Strategic summary:**
- Group blips by quadrant, then by ring within each quadrant
- Highlight all `isNew: "TRUE"` entries
- Call out `hold` ring entries as warnings/risks
- Show counts: total blips, per quadrant, per ring

**Quarter-over-quarter diff:**
- Auto-detect the previous quarter file
- Report: new entries added, entries removed, ring movements (with direction — promoted vs demoted), description changes
- Flag entries that moved to `hold` as significant

**Staleness analysis:**
- Compare across all available quarter files
- Flag entries unchanged (same ring, same or near-identical description) for 2+ consecutive quarters
- Severity: 2 quarters = "worth reviewing", 3+ quarters = "likely stale"
- For stale entries, suggest action: update description, promote/demote ring, or archive

**Output:** Single structured report with all three sections. Ends with preview link.

---

### 4. `/radar-roll`

**Purpose:** Create a new quarter's radar file from the most recent one.

**Auto-detect with confirmation:**
- Scan repo for `qN-YYYY.json` files, identify latest by quarter/year
- Calculate next quarter (q3-2026 → q4-2026, q4-2026 → q1-2027)
- Present the plan: "Rolling from `q3-2026.json` to `q4-2026.json`. Proceed?"
- Wait for confirmation before creating

**Rolling logic:**
- Copy all entries from source file
- Set `isNew` to `"FALSE"` on all carried-forward entries
- Remove entries with ring `"Archive"` or `"Archived"`
- Save as new quarter filename, pretty-printed

**Post-roll actions:**
- Run staleness analysis (same as `/radar-review`) on the new file
- Suggest a Git branch name following existing convention (e.g., `feature/q4-2026`)
- Output preview link for the new file

## File Locations

Skills are installed to `~/.claude/commands/`:
- `~/.claude/commands/radar-add.md`
- `~/.claude/commands/radar-update.md`
- `~/.claude/commands/radar-review.md`
- `~/.claude/commands/radar-roll.md`

## Design Decisions

- **Four standalone skills over one monolith:** Each skill is focused, easy to maintain, and maps to a distinct workflow. Duplication of the preamble is minimal and acceptable.
- **Staleness detection is a cross-cutting concern:** Surfaced in `/radar-add` (post-add), `/radar-review` (dedicated section), and `/radar-roll` (post-roll). This ensures stale entries get attention at natural touchpoints.
- **Auto-detect with confirmation for quarter rolling:** Prevents mistakes while minimizing manual input.
- **Flexible input parsing:** Supports power-user one-liners and guided step-by-step equally.
- **Preview links use current branch:** So the user can verify rendering before merging to main.
