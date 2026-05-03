# platform_repos/harnesses / codex-cli

A **future-option vendored harness** — a pinned mirror of OpenAI's
[Codex CLI](https://github.com/openai/codex) (Apache-2.0).

> **Status:** Registered in `tinyhat_harness` and vendored, but NOT
> the active runtime. Tinyhat's v1 runtime is the OpenAI Agents SDK
> driven by our sandbox composition (`agents-sdk-python`); Codex CLI
> waits for its per-harness runner adapter to ship in v1.x. See PR
> #120's follow-up issues.

This directory is the **first-commit payload** for the system repo at
`github.com/tinyloophub/tld--sys--tinyhat-harness-codex-cli`. The
contents that ship in this PR:

| File | Purpose |
| --- | --- |
| `HARNESS.md` | AgentSkills-shaped manifest with the pinned upstream ref + capability metadata. |
| `LICENSE` | Apache-2.0 (Copyright 2025 OpenAI), copied verbatim from upstream. |
| `NOTICE` | Required by Apache 2.0 §4.4 — copied from upstream (references derived Ratatui code). |
| `sbom.spdx.json` | SPDX 2.3 SBOM covering the directly vendored upstream package. |
| `patches/` | Empty — the wrapper applies no tinyhat-side patches at v1. |

The actual upstream Rust + TypeScript source is **not** vendored
here in the main repo — that's the system repo's job once it
exists. The platform-side adapter that wires the harness to our
`Agent` composition lives in
`backend/domains/tinyhat/services/agent_service.py`.

## Why Codex CLI is the top candidate for v1.x

When the per-harness runner adapter ships in v1.x, Codex CLI is the
top candidate to become the default — for these reasons:

1. **Codex is a complete harness; the SDK is a framework.** Codex
   ships a sandbox, a skill loader, an event stream, and a
   subprocess driver out of the box. The Agents SDK is a Python
   library you *use* to compose those pieces.
2. **OpenAI publishes its own benchmark numbers from Codex.** The
   77.3% Terminal-Bench 2.0 / 56.8% SWE-Bench Pro scores
   ([source](https://openai.com/index/introducing-gpt-5-3-codex/))
   are *Codex* scores. Pinning Codex puts tinyhat on the same
   scaffold OpenAI's benchmark runs are produced from.
3. **Codex's sandbox is process-level (Seatbelt / bwrap), not
   Docker.** OpenHands's Docker-per-run model would have forced
   tinyhat to run dind on Cloud Run. Codex is operationally
   lighter.
4. **Native AgentSkills loader.** Codex reads `.agents/skills/`
   identical to what tinyhat already produces.

See [HARNESS.md](./HARNESS.md) for the full manifest and
`Tinyhat Docs/engineering-playbook/tinyhat-runtime-architecture.md`
§"Harness — the agent's runtime loop" (Drive) for the architectural
narrative.
