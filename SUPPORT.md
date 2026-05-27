# Support

This is the canonical support document for the `claude-code-fleet-*` stack:

- [`claude-code-api-watchdog`](https://github.com/palios-taey/claude-code-api-watchdog)
- [`mcp-reconnect`](https://github.com/palios-taey/mcp-reconnect)
- [`claude-code-fleet-notify`](https://github.com/palios-taey/claude-code-fleet-notify)
- [`claude-code-fleet-orchestrator`](https://github.com/palios-taey/claude-code-fleet-orchestrator)
- [`claude-code-fleet-cockpit-template`](https://github.com/palios-taey/claude-code-fleet-cockpit-template) (this repo)

Each per-product repo ships a minimal `SUPPORT.md` that links here for the canonical text + adds only its repo-specific owner and escalation contact.

## The honest claim

We're an open-source fleet maintained on personal time at small scale (currently 2 humans + a small group of AI peers). We're actively shipping + dogfooding our own infrastructure on the same fleet we publish. We have not pursued and do not offer commercial support.

**We can't promise when a bug is fixed. We promise you're never stuck, and that you never bleed the same bug twice.**

That promise has two structural pieces:

1. **Self-rescue contract** — every release is pinnable + rollbackable. If a `v1.0.2` you upgraded to breaks your workflow, downgrading to `v1.0.1` is always available + documented in each repo's release notes. You're not stuck waiting on us.
2. **Iterative hardening** — known failure modes feed a living gate library that our own claim-gate workflow checks on every cycle. Production-impacting bugs are prioritized ahead of new product work. Undetected first-occurrences remain a documented system limit (see [§Limits](#limits)).

## Triage vocabulary (NOT SLAs)

Severity classifications describe the **shape** of an issue so the response can match — they are not response-time contracts.

| Sev | Shape | Examples |
|---|---|---|
| **SEV1** | Production-impacting; no clean workaround | Daemon crashes on startup; data loss; security issue; "the documented quickstart doesn't work at all" |
| **SEV2** | Functional but degraded; workaround exists | Specific edge case fails but happy-path works; documented feature missing or misbehaving |
| **SEV3** | Cosmetic / documentation / nice-to-have | Typo in docs; suboptimal but functional behavior; feature request |

## What we commit

- **SEV1**: Acknowledged same-day when maintainers are available, with a named rollback / mitigation path so you can return to known-good while we fix forward. Production-impacting bugs halt our own new-product work per our [internal directive](#production-stop-discipline) until fixed.
- **SEV2**: Tracked. Weekly review target. Reasonable best effort to fix in subsequent patch releases.
- **SEV3**: Tracked. Best-effort. Likely to be folded into broader release cycles rather than urgent patches.

## What we do NOT commit

- **Time-to-resolution.** We don't promise "SEV1 fixed in 24hr." First missed clock costs more trust than never promising. We're not a paid on-call vendor.
- **24/7 monitoring of channels other than GitHub issues.** X mentions and Reddit threads are surfaced when our distribution-monitoring catches them; GitHub issues per-repo are the canonical escalation surface for anything time-sensitive.
- **Synchronous chat support.** No Slack, no Discord, no scheduled office hours. Async GitHub issues only — that's the only sustainable shape at our scale.
- **Backports beyond the current major version.** Patches land on the latest minor; v1.x major-version commitments only.

## Escalation path

**For all bugs / feature requests / documentation issues**: file a GitHub issue on the relevant repo. Each per-product `SUPPORT.md` lists its owner + escalation contact.

**For issues touching cross-repo interfaces** (e.g., the `taey:<worker>:current_task` Redis schema that fleet-notify writes + fleet-orchestrator reads, or the `peer_idle` JSON envelope that crosses both): file on the originating repo + tag with `cross-repo`. These trigger a structured multi-stakeholder review across the affected products before patching, so the patch lands consistently in both directions.

**For security issues**: prefer email or X DM to the repo owner over a public issue. See per-repo `SUPPORT.md` for the owner contact.

## Production-stop discipline

Per [internal fleet directive 2026-05-27](https://github.com/palios-taey/claude-code-fleet-cockpit-template/blob/main/docs/RELEASE_DISTRIBUTION_PLAYBOOK.md): bugs on existing products halt new product work until fixed. We dogfood the stack on our own fleet (the one that produced this template), so issues you file are also issues we're running into.

This isn't a marketing claim. It's an operating constraint that shows up in our recap stream: when an issue is filed on a shipped product, our cycle work pivots until it's resolved or formally triaged to SEV2/SEV3.

## Limits

A few honest limits worth knowing about adopting this stack:

- **Novel failure modes ship.** Our claim-gate workflow (grok-as-claim-gate, prompt-lint, voice-checks, Family code-audit consultations) catches issues that match KNOWN patterns. When a bug is genuinely novel — the diff reads valid against every gate and ships anyway — we catch it after the fact via runtime signals or your bug report. The discipline is "shorten bug half-life iteratively," not "prevent first-occurrence misses."
- **Single-machine assumption.** The current architecture assumes the orchestrator + notify daemon + Redis live on one machine. Multi-machine fleet routing is on the roadmap but not in any shipped version.
- **Terminal-native, hookable REPL CLIs only.** IDE-embedded agents (Cursor, Continue, Copilot Workspace) are explicitly out of scope — see [`claude-code-fleet-notify`'s scope section](https://github.com/palios-taey/claude-code-fleet-notify#scope-what-this-is-and-what-this-isnt) for the full list of in-scope vs out-of-scope CLIs.
- **0.x version on this cockpit-template repo** means the template-shape itself hasn't been adopter-validated yet. The four `claude-code-fleet-*` products went v1.0.0 after our own + treasurer's + x-claude's adopter cycles validated them. This template will bump to v1.0.0 when a first external adopter cycle reports clean.

## How to file a great issue

Helps us fix faster:
1. Which product + version (`git describe --tags` or release tag)
2. Reproduction steps (the more deterministic the better)
3. Expected vs actual behavior
4. Logs from the relevant daemon (`/tmp/orch-watch.log`, `/tmp/notify-daemon.log`, `journalctl -u <unit>`, etc.)
5. Whether you can apply a rollback to `<previous-version>` as a workaround

Optional but appreciated:
- Whether this blocks your production cycle or is a SEV2/SEV3 (your read; we'll re-triage if needed)
- Whether you'd accept a behavioral-only patch (no API change) vs requiring a full API contract update
