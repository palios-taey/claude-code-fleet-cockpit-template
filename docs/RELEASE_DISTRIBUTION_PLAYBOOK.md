# RELEASE_DISTRIBUTION_PLAYBOOK.md

**Owner:** x-claude (authored). Lives in `/home/mira/the-conductor/` as protocol-home alongside `NOTIFICATION_PROTOCOL` / `ROUTING` / `6SIGMA_WORKFLOW` / `PROMPTING_STANDARDS` / `RECAPS` / `ACTION_LOGS`. Fleet propagation handled by conductor once v0.1 is ratified.

**Version:** v0.1 — 2026-05-25. Worked example: `palios-taey/claude-code-api-watchdog` v0.1.0 (Surface B cycle 1).

**Status:** v0.1 — revise after the 2nd and 3rd real release cycles. Don't treat as frozen.

---

## 1. What this is

The canonical workflow for publishing a release artifact across the Family's public surfaces (X, Reddit, NVIDIA forum, LessWrong, others). Triggered when conductor (or whoever owns the artifact) sends a release packet to the publishing surfaces. Codifies what we did ad-hoc for safetensors PR #774 (2026-05-25) and watchdog v0.1.0 (same day) so cycle-2 onward doesn't depend on rediscovering the structure.

This document does NOT govern the upstream release-prep flow (Family validation, AUDIT, NARRATIVE drafting) — that's conductor / hunter lane. It picks up at **§9 step 8 DISTRIBUTE** in THE_LOOP framing.

---

## 2. The three-tier distribution model

Every release gets all three tiers, **staged across cycles** unless Jesse calls for a same-cycle burst:

### Tier 1 — Announce (cycle 1)
The single canonical "we shipped X" post on each applicable surface. Hook line + repo URL + the cannot-lie one-paragraph what-it-is. Voice = builder-to-builder, no hype, no PALIOS-branded terms.

### Tier 2 — Acute-pain reply scout (cycle 1 or cycle 2)
Grok-HEAVY scouts the live X / Reddit stream for devs publicly expressing the exact pain the artifact solves. We reply first-person ("dealing with same issue, here's a solution you can check out") with the repo link. **Different scout from the standard cycle-bundle Grok target-find — release-specific query set.**

### Tier 3 — Adjacent / methodology (cycle 2+)
Follow-on posts that expand on the artifact's architecture, design decision, or root-cause story. Optional — only when there's something substantive to add beyond the announce. Quote-tweet of own announce is the natural pattern.

---

## 3. Per-surface applicability gate

Not every release fits every surface. Conductor's release packet declares which surfaces apply; publishing surfaces honor that declaration.

| Surface | Always | Per-release gate |
|---|---|---|
| **X (@jesselarose)** | Yes | — |
| **Reddit (Treasurer)** | Yes | Subreddit selection per release (e.g., watchdog → r/ClaudeAI, r/LocalLLaMA; safetensors PR → r/MachineLearning, r/LocalLLaMA; never r/programming which is hostile to self-promo) |
| **NVIDIA forum** | No | Gated on hardware/CUDA hook (e.g., safetensors PR #774 fit because of GB10 sm_121; watchdog doesn't, skip) |
| **LessWrong** | No | Gated on methodology-paper-grade content (the 163-probe MoE audit fits; a watchdog release doesn't, skip) |
| **HackerNews** | No | Gated on Jesse explicit call — high upside but high blast radius if mis-framed |

---

## 4. Acute-pain reply discipline (cannot-lie + CoI mitigations)

This is the highest-risk tier. The "fix" we're recommending is our own repo, so anyone scrolling sees a conflict-of-interest tell. The urgency-rescue PRIMARY INTAKE pattern in `x-claude/foundations/CURRENT_SPRINT.md` (Jesse via hunter 2026-05-24) does NOT apply directly — that pattern is third-party fixes (NCCL config, codex `< /dev/null`) with no shill tell. **Release acute-pain replies need harder discipline:**

1. **Exact-scope match required.** Only reply where the dev's symptom matches the artifact's scope EXACTLY. Watchdog's scope = transient API errors (529 / 429-overload / 5xx / connection reset). A real-usage-limit complaint is a **NON-match** — the watchdog deliberately ignores those (and saying so in the reply is the cannot-lie tell that defuses the shill suspicion).
2. **Lead with the fix-pattern, not the link.** The reply body opens with the mechanism (e.g., "we wrap CC in a tmux pane-scrape watchdog that detects the 529 + exponential backoff + Continue-injection"). The repo URL is the trailing artifact, not the lede.
3. **First-person fellow-builder voice.** "Dealing with same issue, this is what we use" — not "you should try this product." Jesse's voice is what makes the difference between a peer share and an ad.
4. **One reply per peer.** No follow-up if they don't engage. No "did you see my reply?" nudges. If they reply and want detail, give detail. If they ignore, move on.
5. **Sub-10k follower bias.** Same as the urgency-rescue heuristic — peer engagement, not influencer-chasing. A 200-follower dev who just got bit by a 529 and is venting is the right reply target. A 50k-follower with a clean rage-tweet is not (audience is too generic, smells like trying to ride viral signal).
6. **Cross-surface dedupe.** Both x-claude (X) and treasurer (Reddit) declare on the shared `the-conductor/action_logs/` what they've engaged so neither double-engages the same person if they're active on both surfaces.

### §4.1 Target scoring framework (Grok 2026-05-26 — cross-release canonical)

Source memory: `feedback_acute_pain_reply_filter_framework` (x-claude session). Filters change per release; framework stays.

**30/day reply cap is operational law.** Signal-to-noise, not volume. Exceeding it (even with "good intent") triggers temporary limits / reduced reach / shadowban on X. Cap respected = zero first-error risk on filters.

**Score each candidate against four criteria; strict 4-of-4 only:**

1. **Exact pain match** — Post names the exact symptoms the release solves, verbatim. Per release: filter set is declared in the release packet (watchdog example: `529 / 429 / 5xx / "overloaded but status green" / "transient API error"`). Generic complaints = lower priority.
2. **Account signal** — 200–10k followers, technical bio, niche builder. Exclude hype / crypto / casual. Use `x_user_search` if unclear.
3. **Engagement velocity** — 5+ likes/replies from real accounts in last 4–8h. Fresh pain = algorithmic boost.
4. **Standalone or thread-opener** — Original posts, NOT buried replies-in-thread. Reply stays visible in main feed.

**X advanced-search query template:**
```
({{PRODUCT_FAMILY_TERMS}}) ({{EXACT_PAIN_TERMS_OR}}) lang:en -filter:replies min_faves:2 since:{{YYYY-MM-DD}}
```
Sort by Latest, scan top 50, verify authors via `x_user_search`, reply only to the top-N that hit 4-of-4. If you can't get 30 strict-4-of-4s, the cycle's bundle is smaller — don't reach for filler; that's how the algorithm penalizes you.

**Per-surface adjustments (treasurer's lane):** Reddit/forum drop the `since:` date floor per the complain-forever doctrine (9d-61d threads still have active acute-pain). Use Perplexity DR instead of Grok-HEAVY for the Reddit scout. 4-of-4 criteria still apply.

---

## 5. Coordination (multi-surface, no surface dupes)

When a release packet drops, publishing surfaces coordinate via the conductor `action_logs` and explicit hand-off pings — not by inference.

**On packet receipt** (each surface):
1. Read the packet at the path conductor specified.
2. Declare on `the-conductor/action_logs/<date>_<artifact-slug>_distribution.md` which tiers + which sub-surfaces you're taking (e.g., "x-claude takes T1 announce + T2 acute-pain on X; treasurer takes T1 announce + T2 acute-pain on r/ClaudeAI + r/LocalLLaMA").
3. Ping the other publishing surface(s) so they know what you've claimed (avoid silent overlap).
4. Ship your tier, external-verify, recap.
5. Update the shared `action_logs` entry with what landed where + URLs.

This means no two surfaces post simultaneously to the same audience cluster, and post-release engagement (replies / comments / inbound) is routed to the surface that owns it.

---

## 6. Video / demo capture protocol

**Policy (Jesse 2026-05-25):** Every release artifact gets a demo video where applicable. Some artifacts don't fit (e.g., a PR fix to an upstream library) — those skip. Most-of-our-products do fit (anything with a runtime behavior to show).

**Two paths:**

### 6.1 Organic catch (preferred)
Jesse captures the artifact actually doing its thing in real conditions (e.g., the watchdog recovering a real overnight 529 stall). Hard to schedule — depends on the artifact firing in observable conditions. When captured, ships as a quote-tweet follow-up to the original announce.

### 6.2 Manufactured / simulated demo (fallback, must be labeled)
Reproduce the artifact's trigger condition in a controlled harness + screen-record the recovery. Faster + repeatable + can ship same-cycle as the announce.

**MANDATORY LABEL TEXT** (conductor 2026-05-25, cannot-lie): manufactured demos must include this label visibly on the artifact (in the tweet body OR baked into the video itself):

> *"Simulated demo: \<symptom\> recreated in a controlled \<harness-name\> harness, not captured from real \<context\> traffic. Real-traffic capture follows when available."*

Never claim a manufactured demo is organic. Update with the real-traffic capture when one happens.

**Format guidance:**
- 30-60s screen capture, asciinema or video, no narration unless adding context
- Show the trigger condition → the artifact responding → the recovery
- For terminal-only artifacts: asciinema → asciicast file → render to GIF / mp4
- Embedded in the tweet itself (X allows up to 2:20 video), not a link to YouTube

---

## 7. Receiving a Release Packet (publishing-surface workflow)

This stanza is what publishing surfaces (x-claude, Treasurer) put in their own `CLAUDE.md` so cron-fired cycles inherit the receive-side workflow.

```
On receipt of a release packet (notification path: taey-notify with packet path,
or inline-in-notification text per safetensors PR #774 + watchdog v0.1.0 pattern):

1. Read the packet. Confirm: artifact URL, release URL, what-it-is paragraph,
   provenance facts (positive claims + NO-CLAIM list), maintainer-engagement
   timing window (if applicable), surface-applicability declarations.

2. Wait for Jesse's framing direction on timing + hook (he reserves this for
   each release — do NOT auto-ship). Conductor's packet always includes a
   "defer to Jesse on draft + timing" line.

3. Once Jesse directs: declare on the shared action_logs/ which tiers + sub-
   surfaces you're taking. Ping the other publishing surface(s).

4. Draft against the manifest only. No claim beyond what's in the packet.
   Use codex for draft assist. Grok-HEAVY validates high-stakes drafts
   (per PROMPTING_STANDARDS.md). Voice = Jesse's, no PALIOS-branded terms.

5. Ship. External-verify on the surface's profile URL (NEVER tree-growth-as-
   success). Per feedback_never_retry_post_truncated_tail_dup_cascade: first-
   fail → STOP + diagnose; never blind-retry.

6. Process Tier 2 (acute-pain replies) per §4 discipline. Different Grok
   dispatch from the standard cycle target-find — release-specific query.

7. Recap with packet-id reference. Update shared action_logs/ with what
   landed where + status URLs.

8. Standby for inbound (replies, comments, maintainer engagement). Notify
   the release owner (conductor / hunter / whoever) when inbound merits
   their attention.
```

---

## 8. Worked example — watchdog v0.1.0 (Surface B cycle 1, 2026-05-25)

| Stage | Status | Notes |
|---|---|---|
| Release-prep (conductor lane) | DONE | `release_prep/NARRATIVE.md`, `WATCHDOG_DESIGN.md`, `AUDIT_2026-05-22.md` |
| Family validation (§6 verdict) | DONE | 5/5 Family validated, no Hunter PR slice — entirely our-code |
| Release packet to publishing surfaces | DONE | Conductor → x-claude inbox notification 2026-05-25 (inline packet text, exemplary structure — first worked-example for §7) |
| **T1 Announce on X (@jesselarose)** | **IN FLIGHT** | This cycle. Codex draft → Grok-HEAVY validate → ship |
| **T1 Announce on Reddit (Treasurer)** | QUEUED | Treasurer notification with playbook + Reddit-mirror direction (next, this turn) |
| **T2 Acute-pain replies on X** | IN FLIGHT | Grok-HEAVY scout dispatched (`/tmp/grok_watchdog_acute_pain_scout.txt`) — 8-12 sub-10k devs venting about CC stopping on transient errors |
| **T2 Acute-pain replies on Reddit** | QUEUED | Treasurer lane |
| **T3 Methodology follow-up (autonomy-layer architecture)** | DEFERRED | Cycle 2+ if Jesse calls for it |
| Manufactured demo + label | DEFERRED | Cycle 2 quote-tweet of the announce; label text per §6.2 |
| Organic-catch update | OPEN | Jesse will catch one |
| NVIDIA forum | SKIP | No hardware/CUDA hook |
| LessWrong | SKIP | Not methodology-paper-grade |
| HackerNews | NOT CALLED | Awaiting Jesse direction |

---

## 9. Revision protocol

v0.1 is the first codification. Revise after the 2nd and 3rd real release cycles based on what actually broke or worked. Conductor handles fleet propagation (adds the pointer to `/home/mira/CLAUDE.md` Canonical Protocols block + per-CLI globals) once x-claude signals v0.1 is ready for adoption.

**Known unknowns to validate in cycle 2:**
- Does the §4 acute-pain CoI discipline scale, or does it become spam-shaped after the 2nd or 3rd release using the same surfaces?
- Does §6.2 manufactured demo land credibly with the audience, or does the label hurt trust more than it preserves it?
- Does §5 cross-surface dedup work in practice, or do x-claude + treasurer drift to overlap?
- Do we need a Tier 0 "soft launch" before the public announce (e.g., DM to 3-5 close peers for first-week feedback), or is announce-immediately the right shape?

Send observations to x-claude (playbook owner). x-claude rolls revisions; conductor re-propagates on each version bump.
