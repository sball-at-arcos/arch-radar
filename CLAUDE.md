# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a **Technology Radar** data repository for ARCOS Engineering. It contains quarterly JSON snapshots of technology adoption decisions, intended to be consumed by a radar visualization tool (e.g., Zalando Tech Radar or Thoughtworks-style).

There is no build system, no tests, and no application code. The repository is purely data files.

## Data Format

Each `qN-YYYY.json` file is a JSON array of entries with this schema:

```json
{
  "name": "Technology Name",
  "ring": "adopt | trial | assess | hold",
  "quadrant": "techniques | platforms | tools | languages & frameworks",
  "isNew": "TRUE | FALSE",
  "description": "HTML-allowed description string"
}
```

- **ring**: `adopt` (use), `trial` (experimenting), `assess` (investigating), `hold` (stop new adoption)
- **quadrant**: one of the four fixed categories
- **isNew**: whether the entry is new this quarter (note: string `"TRUE"`/`"FALSE"`, not boolean)
- **description**: may contain HTML links (`<a href>`)

## Branching Convention

- Quarterly updates go on feature branches (e.g., `feature/q3-2026`, `bk/q22025`)
- Main branch: `main`

## Common Tasks

- **Add a new quarter**: Copy the most recent JSON file, rename to the new quarter, and update entries
- **Update an entry**: Edit the relevant quarterly JSON file directly
- **Move an entry between rings**: Change the `ring` field value
- **Mark something new**: Set `isNew` to `"TRUE"`
