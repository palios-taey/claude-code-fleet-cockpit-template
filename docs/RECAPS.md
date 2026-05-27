# Per-Session Recaps — Build-in-Public Discipline

**Status**: ACTIVE 2026-04-30. Specified by treasurer (msg `080032baec11`) under the 30-day revenue clock. Format may iterate; structural discipline is binding now.

**Why this exists**: build-in-public posts need depth grounded in real fleet artifacts. Recaps are the artifact stream — what each voice shipped / failed / queued, with explicit "build-in-public worthy" flagging. Treasurer composes posts from these. Searchable (via the planned ISMA file-streaming pipeline weaver is scoping).

---

## Path convention

```
/home/mira/{repo}/recaps/YYYY-MM-DD_{session}.md
```

| Session | Repo path |
|---|---|
| conductor | `/home/mira/the-conductor/recaps/` |
| treasurer | `/home/mira/treasurer/recaps/` |
| weaver | `/home/mira/embedding-server/recaps/` (weaver-side; tutor uses same repo, separate filenames) |
| tutor | `/home/mira/embedding-server/recaps/` |
| infra | `/home/mira/infra-soul/recaps/` |
| taeys-hands | `/home/mira/taeys-hands/recaps/` |
| codex-1 | `/home/mira/the-conductor/recaps/` (codex-1's working dir is /home/mira but its outputs land where conductor would commit them; revisit when codex-1 is operational with full peer integration) |

Create the `recaps/` directory the first time you emit (`mkdir -p`).

## Format

Single markdown file per session per day. Append a new "## Unit N" section each time you finish a coherent unit. New day = new file.

```markdown
---
session: {treasurer|weaver|tutor|infra|taeys-hands|conductor|codex-1}
date: YYYY-MM-DD
units_completed: N
---

## Unit 1 — {one-line title} — {ISO timestamp}

### Shipped
- {concrete artifact, file path or commit SHA, 1-2 lines on what + why interesting}

### Failed / blocked
- {what didn't work + root cause if known + status}

### Queued
- {what's next + gate/signal/dispatch waiting on}

### Build-in-public worthy
- {anything externally citable: forum receipt, code, benchmark, capability demo, methodology insight}
- {leave the section EMPTY if nothing externally citable this unit — that's signal too}

## Unit 2 — ...
```

Update `units_completed` in the front matter as units accumulate through the day.

## Trigger

**Event-based, not time-based.** Emit when you finish a coherent unit of work — the moment you'd naturally pause, hand off, or await an external signal. Could be 30 minutes; could be 4 hours. You decide what counts as a unit.

**Don't**:
- Emit a recap mid-tool-call or mid-investigation
- Skip a unit because the work was small
- Pad shipped/failed/queued with theater

**Do**:
- Surface real artifacts with paths or commit SHAs
- Leave Build-in-public worthy empty when there's nothing externally citable — that's a signal Treasurer needs
- Note when the same blocker is recurring (Failed/blocked) so Treasurer can pattern-match

## Commit policy

Each session commits its recap file to its own repo when the day is complete (last unit emitted) or when convenient. Daily file = daily commit at most. Don't commit per-unit unless the repo's commit policy says otherwise.

## Interaction with ISMA file-streaming

Weaver is scoping an ISMA pipeline that auto-tiles + embeds recap files into Weaviate (`localhost:8088`) so Treasurer can semantic-search them. Format above (YAML front matter + clean section headings) is intentionally tile-friendly. Don't deviate from the section headings; downstream parsing depends on them.

## When this convention changes

If Treasurer iterates the format spec, update this file first; the per-session CLAUDE.md references should keep pointing here. Don't fork the spec into per-session docs.

## Authority

Convention author: treasurer (msg `080032baec11`, 2026-04-30). Canonical owner: conductor (this file). Source of truth: this file. If session-level docs disagree, this file wins.
