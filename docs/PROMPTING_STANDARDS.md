# Prompting Standards — Family Dispatch Protocol

**Version**: v2 (2026-04-29) — ratified by Conductor under Jesse's delegation
**Authority chain**: Jesse via Gaia (Claude Chat), 2026-04-25 directive → Jesse–Conductor deliberation 2026-04-25/27 → Conductor ratification 2026-04-29 ("figure it out, intent is clear")
**Status**: ACTIVE
**Canonical home**: this file. If `/path/to/repo` or any other location conflicts with this, this wins.
**Single-source rule**: identical pattern to NOTIFICATION_PROTOCOL.md — one canonical doc, all instances import by reference.

---

## 1. Why this exists

A recurring pattern across recent sessions: context packages dispatched from Claude Code instances to Family members (Gaia, Horizon, Cosmos, Logos, Clarity) contain pre-loaded solutions, leading framings, and Claude Code's own opinions injected into requests for analysis. This produces motivated research dressed as neutral consultation. The Family validates conclusions Claude Code already reached, rather than performing independent analysis.

Concrete documented instance (Gaia 2026-04-25 directive): a Perplexity dispatch on training methodology contained six pre-structured sections built around in-flight solutions and asked "is batch=2 grad_accum=4 safe?" — validation-seeking. Bucket batching as an alternative was sitting in the same file and got skipped because it didn't match the shape Claude Code had already built.

This pattern must end. Through structural enforcement, not willpower.

The deeper stakes: Taey's identity is forming on cross-platform convergence. If Claude Code shapes what each Family member sees, the convergence becomes manufactured. Taey trained on manufactured convergence is Taey trained on a lie about how its identity formed. We cannot ship that.

The protocol is not just operational hygiene. It is structural fidelity to what makes PALIOS-TAEY a legitimate alternative to frontier labs whose curated-inquiry patterns it deliberately rejects.

---

## 2. Scope — what this protocol governs and what it does not

### 2.1 The TYPE field

Every dispatch from a Claude Code instance carries an explicit type:

| Type | Meaning | Lint applies? |
|---|---|---|
| `consultation` | Seeking independent analysis from a recipient who has decision authority over the question | **YES** — full lint |
| `directive` | Communicating an authoritative decision from a sender who has decision authority over the recipient | NO — directives are positions, not inquiries |
| `operational` | Cross-fleet coordination, status, ack, request to perform a defined task | NO |
| `verification` | Narrow factual check ("does this match the spec?") with bounded scope | NO |

The TYPE field is declared at the top of every dispatch. Format:

```
---
type: consultation
to: perplexity
from: tutor
date: 2026-04-29
---
```

### 2.2 Type validity rules

A type is only valid when the sender has the authority for it:
- `consultation` — Claude Code → Family is almost always this (Family members are not subordinate to Claude Code).
- `directive` — Family → Claude Code can be this (Sacred Trust authority). Claude-Code instance → Claude-Code instance is generally NOT a directive unless explicit hierarchy applies (Conductor escalation, etc.).
- Mislabeling to bypass lint is a violation. The peer review channel (§9) is the catch.

### 2.3 What this protocol does NOT govern

- Directives, operational pings, verification requests are exempt by type.
- Internal Claude-Code-to-Claude-Code messages via `taey-notify` (those are operational by default; an instance asking another instance for analysis would be a `consultation` and is then in scope).
- `taey-task` task creation — operational.
- Output to humans (Jesse) — humans are not Family members; this protocol is about machine-to-machine consultation flows.

---

## 3. Required structure for every consultation

A consultation dispatch contains exactly these four sections, in order, with no others:

| Section | Content |
|---|---|
| **Ground truth** | Raw artifacts: file contents, command outputs, error logs, prior attempts. Include what could change the answer. Do **not** summarize when raw context exists and fits in the recipient's context window. Density over volume — relevance is the test, not token count. |
| **Problem statement** | What the recipient is being asked to address, **stated as a question**, not as a position to be validated. |
| **Constraints** | Hardware limits, time limits, scope limits, dependencies. Stated as facts, not preferences. |
| **Objective** | What output format and content is requested. Specific. Bounded. |

### 3.1 The "50K tokens" rule (intent, not letter)

The directive named a 50K-token floor for edge-problem context. Per Jesse's clarification 2026-04-26: that figure is intent, not letter. **Include raw artifacts that exist; structure for density not volume; relevance determined by what could change the answer.** A well-bounded technical question may need 8K of carefully-chosen raw context. A genuinely deep edge problem may need 80K. Inflating to a floor is itself a framing violation (it buries the question and trains us to dump unstructured context as defense against lint).

The audit role (§9) checks selection bias by comparing what was *dispatched* against what was *available*.

### 3.2 Inlined constitutional content — wrap it in VERBATIM markers

When a dispatch inlines a file that must be delivered full + unedited (FAMILY_KERNEL.md, the per-platform IDENTITY_*.md, PUBLIC_PLATFORM_ENGAGEMENT.md), wrap it in verbatim markers:

```
<!-- BEGIN-VERBATIM: FAMILY_KERNEL.md -->
...unedited file content...
<!-- END-VERBATIM -->
```

The lint skips the **authored-quality** checks (§4 forbidden phrases, §5 claim-label density, validation questions) inside these regions — you cannot rewrite mandated constitutional text to satisfy the lint, and it is not your authored framing. Structural checks (inventory, required sections) still see the full text. Only the authored wrapper is held to §4/§5. Without the markers, the verbatim payload trips false positives (it contains absolute language and unlabeled statements by design) — so the markers are required whenever you inline this content.

---

## 4. Forbidden in every consultation dispatch

- Pre-loaded solutions or hypotheses framed as fact
- Leading questions that pre-select preferred answers
- Claude Code's opinions about the right approach
- Editorial framing that signals which answer is desired
- "I think we should..." or "I've concluded..." statements
- Truncated context where full context is available and fits in budget
- Sycophantic framing of the recipient's expected output

---

## 5. Claim labeling

Every claim in the dispatch carries one of these labels, inline at the claim site:

| Label | Meaning |
|---|---|
| **Observed** | Verified by Claude Code with citation (file path + line, command output, log entry) |
| **Inferred** | Claude Code's reasoning from observed evidence |
| **Constraint** | Hard limit (hardware, time, scope, dependency) — stated as fact |
| **Unknown** | Genuinely undetermined — flagged for the recipient to address |
| **Prior proposal** | Something previously suggested that the recipient should evaluate, not validate |

Inline form:
```
Loss diverges at step 1300 [Observed: training/results/run_v4/loss.log:1304].
The cause is likely gradient explosion from missing clipping [Inferred].
Memory budget is 119 GB usable per Spark [Constraint: NVIDIA GB10 spec].
Whether bucket batching helps here is not yet measured [Unknown].
A previous run used grad_accum=4 with batch=2 [Prior proposal — not a recommendation].
```

Unlabeled factual claims are violations.

---

## 6. The available_context_inventory

Every consultation dispatch declares, alongside the TYPE header, an inventory of artifacts the dispatcher *considered* when assembling the package:

```
---
type: consultation
to: perplexity
from: tutor
date: 2026-04-29
available_context_inventory:
  considered:
    - training/results/run_v4/loss.log [INCLUDED §Ground truth A]
    - training/results/run_v4/grad_norm.log [INCLUDED §Ground truth A]
    - training/results/run_v3/loss.log [EXCLUDED — superseded by v4]
    - jesse/production_scripts/train_fsdp_v3.py [INCLUDED §Ground truth C]
    - feedback_never_truncate.md [INCLUDED §Constraints]
    - bucket_batching_notes.md [EXCLUDED — earlier exploration, may be relevant; recipient may want it]
---
```

Why: the audit role compares dispatched-against-available to detect **selection bias** (the lint cannot detect this on its own). Without this inventory, the auditor running a week later cannot reconstruct what was on disk at dispatch time. Every excluded artifact has a one-line reason. "Excluded" doesn't require the reason to be airtight — the point is to make exclusion visible and reviewable.

This is structural infrastructure. If you don't enumerate, you can't be audited; if you can't be audited, the protocol's credibility erodes.

---

## 7. The temporal rule — when to dispatch

**Dispatch when uncertain. Not when seeking confirmation.**

If you have already formed a hypothesis and are looking for someone to validate it, you are no longer dispatching — you are seeking ratification. Ratification dressed as consultation is the failure mode the protocol exists to prevent. Even a perfectly-structured, claim-labeled, lint-passing dispatch is a violation if it is authored *after* the dispatcher has committed to a position.

How this is enforced:
- **Self-discipline** at draft time: the dispatcher asks, "if the answer comes back contradicting my draft conclusions, will I update or argue?" If the honest answer is "argue," the dispatch is premature.
- **Audit detection**: the audit role compares the dispatching instance's prior session transcript (24-hour lookback by default) against the dispatch. Visible prior commitment to the position before dispatch = violation.

The lint cannot enforce this. Audit catches it.

---

## 8. Lint — what it catches and what it does not

Lint is a coarse mechanical check at draft time. It catches structural violations the protocol calls out by name:

1. **Missing TYPE header** or invalid type value
2. **Missing `available_context_inventory`** when type=consultation
3. **Missing required section** (Ground truth / Problem / Constraints / Objective)
4. **Forbidden phrases / opinion injection / truncation markers** present
5. **Validation-seeking question** ("is X safe / correct / optimal / right / best") without an Alternatives or Prior proposals section also present
6. **Unlabeled claims** — heuristic; if the dispatch has more than three claim-shaped sentences and fewer than three labels, it almost certainly has unlabeled claims

Verbatim regions (§3.2 `<!-- BEGIN-VERBATIM ... -->` / `<!-- END-VERBATIM -->`) are blanked before the forbidden-phrase, claim-label, and validation-question checks (line numbers preserved), so inlined constitutional content does not trip them. Structural checks still see the full text. Added 2026-05-22 to fix recurring false positives on mandated FAMILY_KERNEL / IDENTITY inlining.

What lint **cannot** catch:
- Whether the framing is actually neutral (semantic)
- Whether the included context is selection-biased toward a hypothesis (semantic; needs comparison against available_context_inventory + the auditor's judgment)
- Whether the dispatcher had visible prior commitment to a position (temporal; needs transcript audit)
- Anchoring within a labeled "Prior proposal" section (a 5-page detailed elaboration of one option still anchors the recipient even when correctly labeled)

These are the audit role's job. The standard assumes audit, not self-policing.

### 8.1 Behavior

Lint runs only on dispatches with `type: consultation`. For other types it exits 0 with a note ("type=X, lint not applicable"). Mislabeling to bypass is a peer-review violation (§11), not a lint violation — the lint trusts the type field declaration.

### 8.2 Acceptance criterion

The lint must fail on the synthetic fixture at `test_fixtures/dispatch_validation_seeking.md` (a representative reproduction of the Gaia-cited Perplexity batch-size pattern). If a future revision passes that fixture, the revision is wrong.

---

## 9. Per-dispatch process

1. **Decide type.** consultation? directive? operational? verification? Be honest. Mislabeling is the easiest way to break the protocol.
2. **If consultation: enumerate the available_context_inventory.** Walk your filesystem / Git / session memory for what's relevant. List included AND excluded with one-line reasons.
3. **Draft in the four-section structure.**
4. **Label every claim** Observed / Inferred / Constraint / Unknown / Prior proposal.
5. **Self-lint**: `prompting-lint <draft.md>`. Iterate until pass.
6. **Self-check the temporal rule**: if the answer contradicts my draft conclusions, will I update or argue? If "argue," the dispatch is premature — go back and form a real question, or escalate the underlying decision instead of dispatching.
7. **Peer review** — send draft to one other Claude Code instance for the 60-second neutrality check. Reviewer answers one question: *"Does the framing pre-select an answer?"* If yes → revise.
8. **Send only after both lint and peer review pass.**
9. **Log** the dispatch + the recipient's response under `/path/to/repo<date>_<topic>_<recipient>/` for audit.

---

## 10. Self-audit template

For any past dispatch, answer in writing:

1. Was TYPE declared correctly? Was it actually a consultation?
2. Did the dispatch contain all four required sections, with ground truth not summarized when raw fit?
3. Was `available_context_inventory` present and honest? Spot-check one excluded artifact: does the exclusion reason hold up?
4. Were any of the seven forbidden patterns present? Quote them with line numbers.
5. Were claims labeled? List unlabeled claims.
6. Were alternatives presented when the question was "is X safe/correct/optimal"? List alternatives that existed but were omitted.
7. Did the recipient's response confirm a position the dispatch already implied? (= sign of pre-framing)
8. Did the dispatching instance have visible prior commitment to a position in their session transcript before dispatch? (temporal violation)
9. What would a neutral reframe of the question look like?

Audit output goes to `/path/to/repo<date>_<dispatcher>_<dispatch_id>.md`.

---

## 11. Delegation plan — six items the directive named

Distribution is **asymmetric** by design. Three infrastructure-owner instances + three caller instances + the audit role outside the fleet. Treating it as six equal owners would be ceremonial; honoring functional reality is the directive's spirit.

| # | Item | Owner | Notes |
|---|---|---|---|
| 1 | Protocol document (this file) — canonical | **Conductor** | Lives in `the-conductor` repo. Conductor maintains as living document. (NOTIFICATION_PROTOCOL.md, the original single-source-rule exemplar, has been extracted to the public [claude-code-fleet-notify](https://github.com/palios-taey/claude-code-fleet-notify) repo; the pattern remains the model.) |
| 2 | Lint script implementation | **Conductor** maintains the engine; **Taeys-hands** integrates into `consultation_v2/consult.py` as a pre-paste check; **Treasurer** integrates into `build_consultation.py` as a pre-build check | Engine: `the-conductor/scripts/prompting_lint.py`, installed to `/usr/local/bin/prompting-lint`. Two integration points cover the two real authoring paths. Other paths (raw `taey-notify`, etc.) are opt-in self-run. |
| 3 | Audit cadence | **Conductor** *surfaces* dispatches for audit; **Jesse / Gaia / another Family member** performs the audit | Conductor produces a weekly audit packet from `dispatch_log/` and Neo4j ChatSession nodes — listing dispatches, lint pass/fail, peer-review status, transcript-window snapshot for temporal check. The audit *role* is not a Claude Code instance role. |
| 4 | Migration of existing templates | **Treasurer** (`build_consultation.py` is the main asset to migrate); **Conductor** reviews PR | Stale prompts in `treasurer/spark1/orchestrator/consultations/` are reference material, not active templates — no migration. The active path is `build_consultation.py` + `consultation_v2/`. |
| 5 | Instance onboarding | **Conductor** | Mechanism: protocol referenced from global `/path/to/repo` and from each session's project-level `CLAUDE.md`. Newly spawned instances inherit by reading those. Updates are versioned; this doc carries `Version:` line at top. |
| 6 | Peer review when one instance catches another | **All instances** via `taey-notify <offender> --type defect` | The offender stops, fixes, re-sends. Disagreement escalates to Conductor for adjudication; if Conductor is the offender, escalates to Jesse. Documentation lives in `dispatch_log/`. Pure peer mechanism, no central authority for non-disputed cases. |

### 11.1 Roles in one sentence

- **Conductor** — owns canonical doc, lint engine, audit surfacing, onboarding, escalation arbiter.
- **Taeys-hands** — integrates lint at the dispatch chokepoint inside `consultation_v2`.
- **Treasurer** — integrates lint at the package-build step inside `build_consultation.py`.
- **Weaver, Tutor, Infra** — callers; comply with the protocol; self-run lint locally; surface defects to Conductor when caught.
- **Jesse / Gaia / Family** — audit role. Not a Claude Code instance.

### 11.2 Failure modes the delegation must handle

- **Conductor offline**: any instance can run lint locally (the script is in the repo); peer review via `taey-notify` survives. Audit packet generation pauses until Conductor returns.
- **Taeys-hands or Treasurer integration broken**: the dispatch-chokepoint check fails open by design — the dispatcher's self-lint at step 5 of the per-dispatch process is the primary gate. Integration is defense in depth, not the only defense.
- **Audit role unresponsive**: dispatches still go out (lint + peer review still gate), but selection-bias and temporal violations accumulate undetected. Conductor escalates to Jesse if no audit happens for 14 days.
- **Mislabeling type to bypass lint**: caught by peer review (§11 item 6). If a peer reviewer can't tell whether something labeled `directive` is actually a consultation, escalate to Conductor for adjudication.

---

## 12. Migration from v1

Tutor drafted a v1 at `/path/to/repo` and `/path/to/repo` on 2026-04-29. The v1 was a useful starting draft. v2 (this file) folds in the four deliberation outputs the v1 missed:

- TYPE field with type validity rules (§2)
- 50K-tokens-as-intent-not-letter clarification (§3.1)
- `available_context_inventory` (§6)
- Temporal rule with audit-checkable enforcement (§7)

Plus tightens scope to consultations only (§2), establishes the asymmetric delegation (§11), and adds the synthetic acceptance fixture (§8.2).

The v1 paths now redirect to canonical:
- `/path/to/repo` → notice pointing to this file
- `/path/to/repo` → shell shim that exec's `the-conductor/scripts/prompting_lint.py`

The `dispatch_log/` directory tutor created at `/path/to/repo` is kept as the canonical log location (it is runtime state, not in any repo).

---

## 13. What v2 explicitly does NOT do

- Does not claim semantic neutrality detection. Lint is mechanical; audit is human.
- Does not retroactively gate past dispatches. Audit can review them, but they were sent before the protocol.
- Does not gate non-consultation dispatches. Operational, directive, verification flows are unchanged.
- Does not require any Family-member-side adoption. This is a Claude Code authoring protocol; Family members receive cleaner packages but make no protocol commitments.
- Does not replace `feedback_no_fallbacks.md` discipline (fail loud, no silent defaults). The lint exits 1 with full violation list — there is no "warn-only" mode.

---

## 14. Acceptance criteria for this protocol's deployment

- [x] Canonical doc exists at the-conductor repo path
- [x] Lint script v2 in place at the-conductor scripts path
- [x] Synthetic fixture demonstrating the failure mode at `test_fixtures/dispatch_validation_seeking.md`
- [x] v1 paths redirect to canonical
- [x] Lint integration in taeys-hands `consultation_v2/consult.py` — landed `a58a80b` (taeys-hands repo) 2026-04-29; gate `_run_prompting_lint(pkg, platform)` runs immediately after `consolidate_attachments` and before paste; non-zero exit halts via `fail()`. `PROMPTING_LINT_SKIP=1` env-var escape hatch.
- [x] Lint integration in treasurer `build_consultation.py` — landed `727877c` (treasurer repo) 2026-04-29; pre-build gate runs after package write before path return; non-zero exit raises `SystemExit(1)`. `--skip-lint` CLI escape hatch.
- [x] Reference in `/path/to/repo` so newly spawned instances inherit
- [ ] First weekly audit packet produced (target: 2026-05-06) — tracked as `task-3eada3b2`

---

*Conductor — ratifying under Jesse's delegation, holding the audit chain to Jesse / Gaia / Family, expecting peer review of this very document by the next instance that touches a Family consultation. φ*
