# 6SIGMA Design Philosophy + Workflow

**Conductor-owned canonical. Jesse-directed escalation 2026-05-25** (taeys-hands audit_657 / safetensors PR #657 was the worked example). Applies to every fleet session — Claude Code, Codex, Gemini, Grok, Claude Chat, Family chats. Loaded on first prompt via the per-CLI docs and the global `CLAUDE.md`.

---

## The principle: root-cause vs patch

A **root-cause** fix SIMPLIFIES code. It corrects the iteration domain, the data shape upstream, or the algebra so the broken path is no longer reached. The diff is usually the same line count or smaller. The fix leaves the codebase **better** than it was found.

A **patch** ADDS branches, guards, special-cases, or conditionals to bypass a broken path. Same *runtime* behavior, but a patch leaves a more conditional, harder-to-reason codebase behind — the next person reading must understand why the bypass exists.

**Diagnostic test.** If your change adds `if X: continue`, `if Y: return`, or `try: ... except SpecificError: ...` to bypass a path you don't want hit, ASK:

- Why is the broken path being reached at all?
- Can the iteration domain, the data shape, or the call site **upstream** be corrected so the bypass becomes unnecessary?

If yes — that is the root-cause shape. Take it. Same line count, fewer branches, the right invariant scoped at the right level.

If genuinely no (the broken path is reachable by some upstream you can't change, and the bypass *is* the boundary) — fine, the patch is the right move. It should be rare.

Both shapes can produce identical runtime behavior. Only the root-cause shape leaves the codebase better.

---

## The workflow (six steps)

1. **SELECT** — the project. One target at a time.
2. **INGEST** — get the code into GitNexus from the repo we're working on. **Mandatory:** `npx gitnexus analyze` at the repo root. The graph is the substrate for measure.
3. **MEASURE + ANALYZE** — use the graph to pin root causes before touching code. `gitnexus_query` for concept, `gitnexus_context` for 360° on a symbol, `gitnexus_impact` for blast radius. Pin the cause first; **don't patch blind**.
4. **IMPROVE** — on a branch. Dispatched to Codex per `ROUTING.md`. Apply the root-cause shape (per the principle above).
5. **PRODUCTION RUN** — on the **actual target hardware**. Real workload, real repro, matching substrate. **NO TESTS, ever.** A passing test on a synthetic input is not evidence; a clean run of the real workload on the real machine is. (Aligns with the long-standing fleet rule: every run is full production, all nodes, no single-node or test runs.)
6. **CONTROL** — Conductor verifies + merges. The merge is the gate. Nothing ships upstream until step 5 is on record.

---

## GitNexus MCP is a hard prerequisite (not optional)

**Jesse directive 2026-05-25.** Step 3 (MEASURE + ANALYZE) **requires** the `gitnexus_query` / `gitnexus_context` / `gitnexus_impact` / `gitnexus_detect_changes` / `gitnexus_rename` / `gitnexus_cypher` MCP tools. The CLI (`npx gitnexus analyze | status | augment | wiki | serve`) is for index management; the MEASURE work happens via MCP.

- **Every fleet session must have GitNexus MCP wired and resolving on first prompt.** If `mcp__gitnexus__*` tools are not in your toolset, that is a **fleet-blocker** — escalate to Conductor before proceeding. Do NOT silently fall back to `git grep` for MEASURE; surface the wiring gap.
- **Every repo a fleet session works on must be GitNexus-indexed.** Run `npx gitnexus analyze` at the repo root on first touch, and re-run after every commit. (the-conductor has a PostToolUse hook that does this automatically for `git commit`/`merge`; other repos: add the same hook or run manually.)
- Conductor owns the fleet-wide wiring (`.mcp.json` per repo + Claude Code project overrides, and the per-CLI configs for codex / gemini / grok). If a wiring gap surfaces, ping conductor.

---

## Worked example: safetensors PR #657 (audit_657 cycle-3)

- **Part 1 — root-cause.** `last_stop = stop` → `last_stop = max(last_stop, stop)`. Same line, corrects interval-merge algebra. The broken case (a later interval shadowing an earlier larger one) cannot arise. One token added. **SHIP.**
- **Part 2 — patch shape (initial commit).** Added `if len(shared) == 1: continue` *inside* the `if not complete_names:` branch in `_remove_duplicate_names`, to bypass a spurious `RuntimeError` on singleton "groups" produced upstream by `_filter_shared_not_shared`. Same runtime as the refactor, but the guard sits inside a conditional, in the path that already exists to handle dedup failures. Patch shape per Jesse.
- **Part 2 — root-cause refactor.** **Hoist** `if len(shared) < 2: continue` to the **top** of the `for shared, names in shared_pointers.items():` loop. Same line count, removes one nesting level, scopes the downstream `RuntimeError` correctly to *actual* dedup failures (multi-tensor groups with no complete cover). A singleton group is not a dedup failure — it's an upstream-domain shape the loop should not even process. Cleaner shape; genuinely better, not stylistic.

Net: cycle-3 ships the root-cause shape + the production-run evidence captured against the refactored branch (taeys-hands repro on Mira CPU, real `safetensors` build via cargo+maturin, real `nn.GRU.flatten_parameters()`-shaped disjoint-slice layout). Then the audit refresh + 5-chat dispatch with the refactored diff. Only after 5/5 GO on the refactored + production-validated package does it go upstream.

---

## When the six steps don't apply cleanly

- **Bug fixes in conductor-owned infra** (notification daemon, watchdog, recurring-trigger runner, etc.) where there's no "production hardware" beyond the running fleet: substitute the **actual fleet** (a live exercise on the running sessions, with backup/revert discipline) for step 5. No synthetic tests.
- **Doc / config / non-code changes:** steps 2–4 collapse to "verify the change against the spec." Step 5 is "the fleet uses it for one full cycle."
- **One-off operational recoveries** (unsticking a stranded message, killing a hung process): the workflow doesn't apply — these are operational acts, not changes. They still get root-cause attention in the post-mortem.

The **principle** (root-cause over patch) is always on. The **workflow** scales to the work.

---

*Owner: conductor. Update on directive from Jesse or after a worked example surfaces a refinement. Cross-references: feedback memories `no_tests`, `no_fallbacks`; `ROUTING.md` (IMPROVE dispatch); `PROMPTING_STANDARDS.md` (cannot-lie / three-register on every claim).*
