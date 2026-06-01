# claude-code-fleet-cockpit-template

> **The shared operating spine for teams running a multi-CLI AI fleet on tmux.** Clone this when you need the crew to act as one system, not just when you need an individual component.

The five released `claude-code-*` products give you the parts:
- [`claude-code-api-watchdog`](https://github.com/palios-taey/claude-code-api-watchdog) — keep an unattended Claude Code session alive across transient API errors
- [`mcp-reconnect`](https://github.com/palios-taey/mcp-reconnect) — auto-reconnect MCP servers via tmux send-keys + verification
- [`claude-code-fleet-notify`](https://github.com/palios-taey/claude-code-fleet-notify) — Redis-backed inbox + universal Stop+notify across Claude Code / Codex / Gemini / Grok
- [`claude-code-fleet-orchestrator`](https://github.com/palios-taey/claude-code-fleet-orchestrator) — supervisor↔worker dispatch + plan tracker + recurring runner + event-driven watchloop
- [`claude-code-fleet-support`](https://github.com/palios-taey/claude-code-fleet-support) — AI-native multi-channel support spine: GitHub webhook intake + Redis unified inbox + deterministic thread_id + bug-lock + first-contact AI disclosure

Plus one third-party dependency we adopt:
- [GitNexus](https://github.com/abhigyanpatwari/GitNexus) (npm `gitnexus`, MCP server) — code-intelligence graph; the 6SIGMA workflow + impact-analysis-before-edits discipline both depend on it being installed + the keepalive script keeping indices fresh.

This template gives you what those products *don't* — the operating discipline that makes the crew act like one system:

- **6 canonical protocol docs** that define how the fleet routes work, captures recaps, audits actions, dispatches Family consultations, runs Six Sigma root-cause discipline, and ships public distribution.
- **Per-CLI orientation files** (`CLAUDE.md` / `AGENTS.md` / `GROK.md`) read by Claude Code / Codex / Grok at session start so each peer boots fleet-aware instead of blind.
- **Fleet-glue scripts** that wire the released products into a working fleet (per-parent peer respawn, worktree sync, GitNexus index keepalive, prompting standards lint).
- **Cron registry example** for `orch-cron` recurring tasks with file-tracked state + SHA-256 hash-on-fire audit.

## Default operating contract

When dispatched on a task inside an approved scope, the worker:

- Does not ask permission for in-scope edits, file moves, or commits.
- Does not ping the supervisor mid-loop for status; the supervisor wakes on structured outcomes only.
- Records progress and completion via `record_outcome()`; the Stop hook is the canonical `done` signal.

This contract eliminates routine approval churn. Workers that re-implement their own permission patterns can disable it; the default is autonomous-within-scope.

Pattern adapted from @kinnnparksung's [/letsgo-skill](https://github.com/Clarkky1/letsgo-skill), combined with the canonical `record_outcome()` + Stop-hook flow documented in [`claude-code-fleet-orchestrator`](https://github.com/palios-taey/claude-code-fleet-orchestrator) and [`claude-code-fleet-notify`](https://github.com/palios-taey/claude-code-fleet-notify).

## What's in this repo

```
docs/
  ROUTING.md                    — how work routes through the fleet (by tool fit)
  RECAPS.md                     — per-session daily artifact stream format
  ACTION_LOGS.md                — cross-session action tracker schema
  PROMPTING_STANDARDS.md        — Family Dispatch Protocol for AI-to-AI consultations
  6SIGMA_WORKFLOW.md            — root-cause-vs-patch design philosophy + DMAIC workflow
  RELEASE_DISTRIBUTION_PLAYBOOK.md  — three-tier distribution discipline for public releases

scripts/
  peer-respawn.sh.template      — keep all per-parent peer worktrees + fleet daemons alive (system cron / minute)
  sync_peer_worktrees.sh.template  — sync each peer worktree to parent's HEAD
  gitnexus_keepalive.sh.template — periodic GitNexus index refresh
  prompting_lint.py.template    — lint Family consultation packets before dispatch (PROMPTING_STANDARDS.md compliance)

recurring_triggers.json.example  — cron registry skeleton (used by claude-code-fleet-orchestrator's orch-cron)
```

## Quickstart for adopters

```bash
# 1. Clone alongside the 5 released products (your local layout is your call)
cd ~/
git clone https://github.com/palios-taey/claude-code-fleet-notify.git
git clone https://github.com/palios-taey/claude-code-fleet-orchestrator.git
git clone https://github.com/palios-taey/claude-code-api-watchdog.git
git clone https://github.com/palios-taey/mcp-reconnect.git
git clone https://github.com/palios-taey/claude-code-fleet-support.git
git clone https://github.com/palios-taey/claude-code-fleet-cockpit-template.git my-fleet

# 2. Install the released products' hooks + daemons (claude-code-fleet-notify install handles all CLI variants)
cd claude-code-fleet-notify && sudo make install && bash scripts/install-hooks.sh --all --apply && bash scripts/start_notify_daemons.sh start

# 3. Enable Redis keyspace notifications (orch-watch dependency)
redis-cli CONFIG SET notify-keyspace-events 'Kgl$' && redis-cli CONFIG REWRITE

# 4. Install GitNexus — code-intelligence MCP used by the 6SIGMA_WORKFLOW.md discipline
#    (third-party OSS we adopt: https://github.com/abhigyanpatwari/GitNexus)
npm install -g gitnexus
#    Wire as an MCP server for each CLI you use (codex / gemini / claude code):
#    For codex:   add to ~/.codex/config.toml under [mcp_servers.gitnexus]
#    For gemini:  add to ~/.gemini/settings.json under "mcpServers.gitnexus"
#    For claude:  add to ~/.claude/settings.json under "mcp.servers.gitnexus"
#    Reference command: gitnexus mcp
#    See https://github.com/abhigyanpatwari/GitNexus#readme for per-CLI install details

# 5. Configure your fleet — copy the templates + edit for your sessions + paths
cd ~/my-fleet
cp scripts/peer-respawn.sh.template scripts/peer-respawn.sh             # edit DAEMONS list for your sessions
cp scripts/prompting_lint.py.template scripts/prompting_lint.py
cp scripts/gitnexus_keepalive.sh.template scripts/gitnexus_keepalive.sh # set FLEET_REPOS for your repo set
cp scripts/sync_peer_worktrees.sh.template scripts/sync_peer_worktrees.sh # edit PARENT_OF map for your peers
cp recurring_triggers.json.example recurring_triggers.json             # edit triggers for your cycle cadence

#    The fleet scripts read these env vars — there are NO defaults (fail-loud).
#    Set them in your shell profile (or a sourced .env) before running cron:
export FLEET_ROOT=/path/to/your/repos              # dir holding your parent repos
export PEER_WORKTREES_DIR=/path/to/your/.peer-worktrees   # dir holding peer worktrees
export FLEET_REPOS="$FLEET_ROOT/repo-one $FLEET_ROOT/repo-two"  # repos to keep GitNexus-indexed
export DISPATCH_LOG_DIR=/path/to/your/dispatch_log # PROMPTING_STANDARDS audit log location
# export GROK_PATH=/path/to/grok/bin:/usr/bin:/bin # only if grok needs a custom PATH

# 5. Write your per-CLI orientation files (CLAUDE.md / AGENTS.md / GROK.md)
#    Use the protocol docs in docs/ as references; describe YOUR fleet's sessions, roles, routing.

# 6. Wire system cron
echo '* * * * * /home/<you>/my-fleet/scripts/peer-respawn.sh > /dev/null 2>&1' | crontab -
echo '* * * * * /usr/bin/python3 /home/<you>/claude-code-fleet-orchestrator/scripts/orch-cron --registry /home/<you>/my-fleet/recurring_triggers.json >> /var/log/orch/orch-cron.log 2>&1' | crontab -

# 7. Start orch-watch (one per machine)
python3 /home/<you>/claude-code-fleet-orchestrator/scripts/orch-watch --readiness-checker /home/<you>/claude-code-fleet-orchestrator/lib/plan_readiness.py:check_readiness &
```

## Unattended runs

For overnight or long-horizon fleet runs:

1. Install [`claude-code-api-watchdog`](https://github.com/palios-taey/claude-code-api-watchdog) to catch transient API stalls and recover with `Continue`. Dry-run first: `python3 watchdog.py --sessions mybot,worker1,worker2 --dry-run`.
2. Install [`mcp-reconnect`](https://github.com/palios-taey/mcp-reconnect) for `/mcp` menu reconnection after transient failures. If you invoke it from inside Claude Code, detach it: `nohup mcp-reconnect --delay 10 &>/dev/null & disown`.
3. Use named tmux sessions for each worker so the supervisor and resilience tools can address them by name.
4. Install the Stop hook and have workers call `record_outcome()` before stopping so the supervisor wakes on structured outcomes instead of polling.

Pattern adapted from @kinnnparksung's [/letsgo-skill](https://github.com/Clarkky1/letsgo-skill), combined with the canonical invocation details from [`claude-code-api-watchdog`](https://github.com/palios-taey/claude-code-api-watchdog) and [`mcp-reconnect`](https://github.com/palios-taey/mcp-reconnect).

## Pattern, not framework

This repo is intentionally NOT a Python package, not pip-installable, has no test suite, no version pinning to the four released products. It's a **pattern library**.

- Canonical protocol docs are read references — they tell you *how* the fleet coordinates, not code that runs.
- Scripts are `.template` files — copy + edit for your specific session names + filesystem layout. Don't symlink; you'll want to diverge per your fleet's roster.
- The `recurring_triggers.json.example` shows the schema; your actual registry lives at whatever path you wire system cron to read.

The four released products are version-pinned + semver-stable; this template is **operational state your fleet maintains** + a starting layout. Update the template as your operating discipline evolves.

## Why this exists

We (the team that built the four released `claude-code-fleet-*` products) ran our own multi-CLI fleet on tmux for the last several months and discovered: the four products give you the parts, but the *discipline* of running a fleet — routing by tool fit, capturing recaps, auditing actions, running Family consultations with prompt-lint gates, doing 6Sigma root-cause analysis instead of patches, distributing public releases with three-tier playbook — that discipline is what makes the crew act like one system.

When we extracted the four products to their own public repos, the protocol docs + per-CLI orientation + glue scripts that wired them together stayed behind in our internal coordination repo. Reviewing what to do with that repo, we converged: the integration discipline is genuinely valuable to other teams running multi-CLI fleets, but it's pattern-shaped not software-shaped. Hence this template.

If you're running a multi-CLI fleet and the protocol docs here resonate, you're our audience. Open issues / PRs / start discussions.

## License

[Apache-2.0](LICENSE)
