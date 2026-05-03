---
name: tinyhat-codex-cli
description: Pinned mirror of OpenAI's Codex CLI — vendored as a future-option harness; runner adapter ships in v1.x.
license: Apache-2.0
metadata:
  tinyhat:
    harness:
      upstream_url: https://github.com/openai/codex
      upstream_ref: rust-v0.125.0
      upstream_sha: 637f7dd6d737f3961e6bf32fbb3861c4953269c5
      system_repo_url: https://github.com/tinyloophub/tld--sys--tinyhat-harness-codex-cli
      sbom: ./sbom.spdx.json
      slsa_level: 1
      sandbox_tiers: [0, 1, 2]
      agent_skills_features:
        - progressive_disclosure
        - scripts
        - references
        - assets
      mcp_support: true
      otel_support: false
      owns_sandbox: true
      headless_driver: codex exec --json
      supported_providers:
        - openai
---

# tinyhat-codex-cli

A **future-option vendored harness** — a pinned mirror of
[`openai/codex`](https://github.com/openai/codex), OpenAI's
production-grade coding-agent harness. Apache-2.0, Rust + TypeScript,
maintained by OpenAI.

> **Status:** Registered in `tinyhat_harness` and vendored, but NOT
> the active runtime. Tinyhat's v1 runtime is the OpenAI Agents SDK
> driven by our sandbox composition (`agents-sdk-python`); Codex CLI
> waits for its per-harness runner adapter to ship in v1.x.

## Why Codex CLI is the top candidate for v1.x

- **OpenAI-maintained, production-grade.** Codex is the harness OpenAI
  publishes its own agentic coding numbers from
  ([77.3% Terminal-Bench 2.0 and 56.8% SWE-Bench Pro on
  GPT-5.3-Codex](https://openai.com/index/introducing-gpt-5-3-codex/)).
- **Native AgentSkills loader.** Codex reads `.agents/skills/`
  natively — tinyhat's existing skill mounts drop in unmodified.
- **Owns its sandbox.** Seatbelt on macOS, bwrap+seccomp on Linux —
  no Docker-per-run requirement. Per-run network allowlist is a
  first-class config (`[sandbox_workspace_write] network_access`).
- **Headless `codex exec --json`.** A stable JSONL event stream
  (`thread.started`, `turn.completed`, `function_tool_call`,
  `function_tool_output`, `agent_message`, …) the platform Python
  backend can drive per-turn. Watchdog hook = subprocess timeout +
  kill.
- **Apache-2.0** — permissive, vendor-friendly, with explicit patent
  grant under §3.

## What this directory is

The **first-commit payload** for the system repo at
`github.com/tinyloophub/tld--sys--tinyhat-harness-codex-cli`.
The admin seed action creates/registers that repository, then seeds
it from this directory plus the upstream's source tree at the pinned
tag (`rust-v0.125.0`, sha `637f7dd6...`). Until the repo exists,
this directory is the source of truth for the platform-side metadata;
the upstream code is fetched fresh at vendor time.

## Vendor compliance

Upstream is licensed [Apache-2.0](./LICENSE) (Copyright 2025 OpenAI).
This mirror preserves:

- The upstream `LICENSE` file at the root of this directory.
- The upstream `NOTICE` file (Codex's NOTICE references derived
  Ratatui code and copyright headers — preserved verbatim per
  Apache 2.0 §4.4).

Significant tinyhat-side modifications live as patch files under
`patches/`, named descriptively (e.g.
`patches/0001-tinyhat-watchdog-hook.patch`). At the v1 launch this
directory is empty — the wrapper applies no patches; the
tinyhat-side adapter logic lives in
`backend/domains/tinyhat/services/agent_service.py` in the main
tinyloop repo, not here.

## SBOM

`sbom.spdx.json` is a [SPDX 2.3](https://spdx.github.io/spdx-spec/)
document covering the upstream package's transitive dependency tree
at the pinned version. It's regenerated on every version bump.

## Where the runtime hookup lives

The harness's `Agent`-shaped composition is built in
`backend/domains/tinyhat/services/agent_service.py` in the main
tinyloop repo:

- `_build_agent` — picks the model + ModelSettings out of the
  agent's `tinyhat_agents.model_provider / model_name /
  model_options` columns.
- `harness_resolver_service` — joins `tinyhat_agents` to this
  harness through `harness_version_id` and refuses any
  `model_provider` outside the `supported_providers` list above.
- The Codex CLI subprocess driver (forthcoming PR — the v1
  resolver still routes through the OpenAI Agents SDK Python in
  this repo's current adapter) will land alongside the bot-manager
  bootstrap (#104).

That split is the load-bearing thing: this directory carries the
upstream code we mirror; the tinyloop-side `agent_service` is the
thin adapter that does *not* live in the mirror, so we never have
to fork upstream history to maintain the platform integration.

## Links

- Upstream: <https://github.com/openai/codex>
- Codex sandbox model:
  <https://developers.openai.com/codex/sandbox>
- Codex AgentSkills loader:
  <https://developers.openai.com/codex/skills>
- Codex non-interactive (`codex exec --json`):
  <https://developers.openai.com/codex/noninteractive>
- AgentSkills standard: <https://agentskills.io>
- Tinyhat runtime architecture (Drive):
  `Tinyhat Docs/engineering-playbook/tinyhat-runtime-architecture.md`
  §"Harness — the agent's runtime loop".
- Issue introducing the schema: `tinyloophub/tinyloop#100`.
