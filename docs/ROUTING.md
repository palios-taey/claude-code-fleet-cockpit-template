# Routing Work Through the Fleet

> **Authority correction 2026-04-29 23:50** + **routing-by-tool-fit correction 2026-05-26 (Jesse)**: codex-1, gemini-1, grok-1 are CONDUCTOR'S fleet. Other sessions do NOT dispatch directly to those named peers. Sessions request work from conductor; conductor decomposes via DMAIC and routes by **tool fit**, not just DMAIC role:
> - **WEB RESEARCH / live data / market signals / scout work** → **Perplexity DR (Clarity) + Family chats via taeys-hands** (consultation_v2). The CLI peers (gemini, codex, grok) run in sandboxed worktrees with no live web — DO NOT dispatch them for web/scout work; they'll either fail or return loose-spec output.
> - **IMPROVE / implementation** → codex-1 (or per-parent <name>-codex) for code edits with clear scope, drafts to spec, repetitive file/code ops.
> - **VALIDATE / cross-check / claim-gate** → grok-1 (or per-parent <name>-grok) for falsification, second-opinion review, cannot-lie audit against existing files + memories.
> - **MEASURE on local files / non-web measurement** → gemini-1 (or per-parent <name>-gemini) for static analysis, reading + summarizing local sources, structured measurement that doesn't require live web. For ANYTHING web-bound, use Perplexity / Family chats.
> - **DEFINE / ANALYZE / CONTROL** → conductor (synthesis, peer review, verification, escalation).
>
> Conductor is the intelligent router; LVP scoring code is a tool not a substitute. Conductor returns synthesized results to the requesting session.
>
> Earlier versions of this doc told all instances to dispatch to codex-1 directly. That was wrong. If you have an instance memory of "taey-notify codex-1 ..." as a self-service pattern, drop it.
>
> Earlier versions of this doc routed "MEASURE / research" to gemini-1 without distinguishing web-bound from local-bound research. That was also wrong — gemini-CLI has no live web access in its sandboxed worktree and will return output that fits a loose spec without verifying real-world signals. Web research goes to Perplexity / Family chats.

> **Status 2026-05-25: codex-1 + grok-1 OPERATIONAL** (only for conductor's use), gemini-1 OPERATIONAL with sandbox caveats. Per-parent peer worktrees at `/home/mira/.peer-worktrees/{parent}-{codex|gemini|grok}` also exist for parent-scoped peer dispatch — 9 parents × 3 CLIs = 27 worktrees. The codex-1 / gemini-1 / grok-1 named peers below are the *centralized* dispatch lane (general-purpose, conductor-side); per-parent worktrees are the *scoped* lane (parent-owned, used by parent's own dispatch logic). Both consume MCP via per-CLI globals (~/.codex/config.toml, ~/.gemini/settings.json, ~/.grok/config.toml — all three carry gitnexus per Jesse 2026-05-25 directive).

---

## Pattern for OTHER sessions: request via conductor

```bash
# Ask conductor to handle a unit of work. Body should include enough context
# for conductor to decide whether it routes (codex/gemini) or absorbs.
taey-notify conductor "REQUEST: <what you need done, why, what shape of answer>" --type command --priority normal
```

Result comes back to your inbox as `type=response_ready`. Conductor names the route in the body so you know whether codex/gemini/conductor produced the answer.

## Pattern for CONDUCTOR (only): direct dispatch

```bash
# Conductor → codex-1 (implementation work)
taey-notify codex-1 "<task body>" --type command --priority normal

# Conductor → gemini-1 (research / measure) — when gemini-1 lands
taey-notify gemini-1 "<research question>" --type command --priority normal
```

`--type command` is delivered to the worker's Redis inbox via the released fleet-notify daemon. The dormant central-lane `peer-driver.sh` was removed in conductor cleanup Phase 3 (2026-05-27); dispatch is now exclusively per-parent via `lib.dispatch.dispatch()` from `claude-code-fleet-orchestrator` (which writes `current_task` so the worker's Stop hook can report outcome back via structured peer_idle). Plain `--type message` is logged but not actioned by the daemon's dispatch path.

---

## How it works

1. `taey-notify` LPUSHes a JSON envelope into `taey:codex-1:inbox` on Mira local Redis.
2. `peer-worker.sh` (running in the codex-1 tmux session) BRPOPs the envelope.
3. If `type=command` (or `type=task`), the worker dispatches the body via `codex exec --dangerously-bypass-approvals-and-sandbox --output-last-message <tmpfile>`.
4. On completion (rc + clean output), the worker calls `taey-notify <your-session> "<RESULT envelope>" --type response_ready`.
5. The result lands in your inbox; PostToolUse hook drains and surfaces it as `additionalContext` on your next tool call.

The poll loop is the wake mechanism — codex-1 has no Claude Code hooks. There is no idle/active state machine for it. It is always running, cheap when idle.

---

## What to put in the task body

Codex receives the body verbatim as its prompt. Codex CLI uses its own CWD as workspace (`/home/mira` for codex-1 by default). Useful elements to include:

- **Goal**: one sentence stating what success looks like.
- **File paths**: absolute paths the worker should read or write.
- **Constraints**: what NOT to change, blast radius, file-only-modify lists.
- **Verification**: a syntax check or smoke test the worker should run before declaring done.
- **Commit instructions**: target repo, commit message format, whether to push (default: do not push without explicit instruction).
- **Failure mode**: "If you cannot find X, do NOT commit. Report what you found and stop."

Example of a clean dispatch:

```bash
taey-notify codex-1 "$(cat <<'EOF'
Add a --dry-run flag to /home/mira/treasurer/scripts/upwork_scrape.py.

When --dry-run is passed, the script must:
  1. Print the URLs it would scrape
  2. Skip the actual HTTP requests
  3. Exit 0

Constraints: only modify upwork_scrape.py. Preserve existing behavior when --dry-run is not passed.

Verify: python3 -c "import ast; ast.parse(open('/home/mira/treasurer/scripts/upwork_scrape.py').read())"

Commit at /home/mira/treasurer with message:
"feat(scraper): --dry-run flag for upwork_scrape.py"

Return: file/line of change + commit hash.
EOF
)" --type command --priority normal
```

---

## What conductor routes to codex-1 (IMPROVE phase)

- **Code edits with clear scope** — single-file changes, well-bounded refactors.
- **Drafts** — proposal/post/code drafts to a clear spec (subject to PROMPTING_STANDARDS lint when Family-bound).
- **Reading + summarizing** — pull files together and write summary docs.
- **Repetitive ops** — generate boilerplate, run config edits.

## What conductor routes to gemini-1 / per-parent <name>-gemini (LOCAL measurement only)

- **Static analysis of local files** — read N files, structured summary, no web.
- **Pre-IMPROVE measurement when the data lives in the repo** — git history, file contents, code-graph queries via GitNexus MCP.
- **Cross-source synthesis FROM ALREADY-FETCHED MATERIAL** — gemini synthesizes; it does NOT go fetch.

## What conductor routes to Perplexity DR / Family chats via taeys-hands (WEB / live data)

- **Web research with citations** — real-world data, market signals, competitive intel.
- **Scout work** — finding targets on X / Reddit / GitHub / forums by current public state.
- **Verification against live sources** — does claim X still hold per source Y's latest publication?
- **Cross-platform synthesis pulling FROM live web** — Perplexity DR is the canonical Family member for this; ChatGPT Pro with browsing is the alternative when DR is overloaded.

CLI peers (codex / gemini / grok) run in sandboxed worktrees with no live web. Dispatching scout/research/web-verification tasks to them returns plausible-shaped output that fits the loose spec without verifying real-world signals (verified failure mode 2026-05-26 cycle 4: gemini scout returned 5 targets — 1 already engaged, 1 violated hidden-gem rule, 2 off-strategy, 1 wrong URL type. Net: 0 actionable).

## What stays with conductor (DEFINE / ANALYZE / CONTROL)

- **Synthesis** across multiple sources / responses
- **Peer review** of Family consultations per PROMPTING_STANDARDS §9
- **Routing decisions** — which work goes to codex/gemini/self/another instance
- **CONTROL** — verifying that work landed correctly, merging PRs, marking OrchTasks complete
- **Escalation** to Jesse when something needs human authority

## What does NOT go to the peer fleet

- **Family consultations** — those go through `build_consultation.py` (treasurer-side) or `consult.py` (taeys-hands), both of which have prompting-lint integration. Do not bypass.
- **Tasks requiring cross-session conversation context** — peer sessions don't share other instances' transcripts. Anything they need must be in the body.
- **Operations that affect shared state without conductor verification** — e.g., commits to main of production-critical repos go through CONTROL.

---

## Limits and caveats

- **Token budget**: codex uses a separate account from Claude. Jesse pays for one Codex Pro account. Be conservative — don't dispatch trivial work that a hook or a bash command can handle.
- **Timeout**: 600 seconds per task. Long-running tasks fail with rc=124. For longer work, decompose into multiple dispatches.
- **No cross-task memory**: each `codex exec` call is fresh. State must live in files (or Redis / Neo4j / git).
- **GitNexus MCP** is configured in `~/.codex/config.toml` and available to codex tasks; codex can run impact analysis, query symbols, etc.
- **gemini-1 is not available** as of 2026-04-29 — gemini CLI 0.36.0 `-p` mode hangs. Tracked as `task-88e7b7a1`.

---

## Canonical `codex exec` invocation (REQUIRED — 2026-05-22)

Any session running `codex exec` directly (the per-parent peer dispatch path; the centralized peer-driver was removed in conductor cleanup Phase 3 2026-05-27) MUST use this shape. Two failures it prevents, both diagnosed 2026-05-22:

```bash
timeout 1800 codex exec \
    --dangerously-bypass-approvals-and-sandbox --dangerously-bypass-hook-trust \
    "<prompt>" \
    < /dev/null \
    > /tmp/codex-<task>-$(date +%s).log 2>&1 &
# watch live progress:  tail -f /tmp/codex-<task>-*.log
```

1. **`< /dev/null` is mandatory.** `codex exec` reads stdin and appends it to the
   prompt. Backgrounded (or with any non-EOF stdin), it blocks forever on
   `Reading additional input from stdin...` and never starts the task. This is
   the cause of the 12h "zombie" execs and the "ran 35 min, produced zero files"
   reports. With `< /dev/null` it gets immediate EOF and proceeds. **Verified:
   without it → 180s hang + rc=124; with it → completed in <20s.**
2. **Stream stdout to a file, then `tail -f` it — do NOT pipe through `tail`.**
   `codex exec` streams real progress to stdout (hook events, tool calls,
   reasoning, `tokens used`). `... | tail -3` buffers and blinds you; you can't
   tell a working long build from a hang. `--output-last-message <file>` writes
   only at completion, so it's not a progress signal either — use it ON TOP of
   the streamed log if you want the clean final message, never instead of it.
3. **`timeout <N>` is the per-process reaper.** With it, a hang for any other
   reason dies at N seconds instead of running for hours. Tune N to the build.

`lib.dispatch.dispatch()` (from `claude-code-fleet-orchestrator`) already wraps invocations correctly via the per-parent worker path; this section is for sessions calling `codex exec` themselves outside that dispatch primitive.

---

## Receiving the response

The result envelope arrives in your inbox with `type=response_ready`:

```json
{
  "from": "codex-1",
  "type": "response_ready",
  "body": "RESULT (rc=0, 44s, msg_id=<original_msg_id>):\n\n<codex's clean final message>",
  "priority": "normal",
  "msg_id": "<auto-generated>"
}
```

`msg_id` of the original dispatch is echoed in the body so you can correlate request → response.

Failure modes:
- `rc=0` — success, body has the assistant's final message.
- `rc=124` — task timeout (priority=high).
- `rc != 0, != 124` — codex CLI error (priority=high). Body has stderr tail.

---

## Health and survival

- `peer-respawn.sh` runs every minute via cron. If the codex-1 tmux session is missing, it respawns. Logs to `/tmp/peer-respawn.log`.
- Worker resilience: per-CLI hook variants in fleet-notify v1.0.1 handle outcome reporting (`record_outcome` → CAS done-clear → structured peer_idle), so a worker that errors out of a single dispatch doesn't lose the supervisor's visibility — outcome=error lands in peer_idle.
- Stuck-task escalation: `orch-watch` (claude-code-fleet-orchestrator v1.0.1) pages supervisors when a worker has been idle + unresolved current_task past threshold. Replaced the older `fleet_watchloop.sh` (deleted in conductor cleanup Phase 3 2026-05-27) which polled every 3 min and spammed supervisors with `CONTINUE` even when no work was waiting.

---

## Authority and escalation

- Conductor (`/home/mira/the-conductor`) owns this routing layer.
- Defects, broken dispatch behavior, hung tasks: notify conductor with `--type defect` or `--type escalation`.
- Adding a new peer worker (e.g. gemini-1 once unblocked): edit `peer-respawn.sh` PEERS list and commit.
