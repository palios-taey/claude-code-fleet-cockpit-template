# Action Logs — Cross-Session Action Tracker

**Conductor-owned coordination state, READ + WRITE by all fleet sessions.**

This document is the canonical pointer for the cross-platform action log infrastructure. The full schema spec is authored and maintained by treasurer at `/home/mira/treasurer/foundations/action_log_spec.md` (latest 2026-05-21, commit `0043927`); future schema changes are proposed there. The directory and per-track files live here in the-conductor because action_logs are coordination state — like notifications, recaps, and routing.

## Canonical paths

| Path | Purpose |
|---|---|
| `/home/mira/the-conductor/action_logs/<track>.jsonl` | Append-only per-track action history. One JSON object per line. |
| `/home/mira/the-conductor/engagement_logs/<track>.jsonl` | Per-track upvote / reply / follower-delta tracking. Future use. |
| `/home/mira/treasurer/foundations/action_log_spec.md` | Canonical schema spec. Edit here, propose changes via PR. |

## Tracks (seeded 2026-05-21)

| File | Owner sessions writing |
|---|---|
| `x.jsonl` | x-claude (canonical), treasurer historical |
| `reddit.jsonl` | treasurer |
| `nvidia.jsonl` | treasurer |
| `upwork.jsonl` | treasurer (when API ID-verify clears) |
| `lesswrong.jsonl` | treasurer |
| `security.jsonl` | hunter |

## Discipline

1. **Append-only.** Never edit past entries. If an outcome changes (submitted → rejected, post → deleted), append a new `action: outcome-update` entry with `notes` pointing at the original `ts`.
2. **Write to the canonical path.** Sessions MUST write to `/home/mira/the-conductor/action_logs/`, not session-local copies. Per Jesse 2026-05-21: "all posts and engagements tracked centrally so you all know what is going on and where to focus and breakthroughs."
3. **Verification URL required for AT-SPI-driven actions.** Tree-growth-as-success is the 2026-05-20 known-bug failure mode; verification URL + screenshot are both required for any action driven via taeys-hands AT-SPI.
4. **Voice check field is not optional.** If it doesn't apply, mark `voice_check: n/a` with a `notes` reason. Never omit.

## Caps (per-track, per-24h)

Per the spec (treasurer/foundations/action_log_spec.md):

| Track | Action | Cap | Cooldown |
|---|---|---|---|
| reddit | comment | 8 (target steady-state; rebuilding from 5/1 stop) | ~90 min jitter |
| nvidia | reply | 2 | 2 hr min |
| upwork | bid | 4 (post API ID-verify) | natural via Connects cost |
| security | pr-open | 1 per target | 48h min between major submissions to same vendor |
| lesswrong | post | 1 | episodic |
| x | (driven by x-claude's own cadence floor, not capped here) | — | — |

Caps are enforced by `scripts/loop/01_pre_flight.py --track <name>`. Exit 0 = under cap; non-zero = refused with reason.

## Reads

| Tool | Purpose |
|---|---|
| `scripts/loop/00_orchestrate.py` | Reads all logs; returns highest-priority under-cap action available |
| `scripts/loop/01_pre_flight.py --track <name>` | Reads one log; refuses to proceed if cap exceeded |
| Daily recap generation | Reads all logs for the date range; summarizes Shipped / Failed-blocked / Queued |

## Schema changes

Treasurer authors the spec. Any session proposing a field addition / value change / new track:

1. Open a PR / make a commit to `/home/mira/treasurer/foundations/action_log_spec.md` with the proposed change.
2. Notify treasurer + conductor for review.
3. After consensus, conductor confirms canonical pointer here references the new revision.

This mirrors how NOTIFICATION_PROTOCOL.md / ROUTING.md / RECAPS.md are conductor-owned canonical coordination state with multi-session readers and writers but a single source of truth.
