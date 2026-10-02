# Roadmap

This page records the initial direction approved during kickoff. Product scope is in
[AGENTS.md](../../AGENTS.md#product) and [requirements](../product/requirements.md).
The backlog has not been created; issue creation and later implementation retain their
own approval scopes. This direction grants no live-engine or remote-write permission.

- **Last reviewed:** 2026-10-02; the new repository has no open issues.

## Now

- **Prove the first harmless execution rule.** Done when the separately approved,
  human-run [PoC matrix](../product/requirements.md#34-safe-poc-acceptance-gate) shows
  pre-execution denial across the tested launch contexts, allows its controls, and
  restores the baseline while preserving unrelated policy. Before live work: accept
  the relevant engine/configuration/privilege ADRs and approve the concrete setup,
  payload, and recovery procedure. Runtime feasibility is currently unverified.
- **Manage one rule through a plain CLI.** Done when the five agreed operations follow
  the [CLI flows](../product/ux-flows.md), fixtures cover unavailable and conflicting
  engine states, and the inherited project checks pass. Replace the sample counter/TUI
  with this behavior; do not broaden the target before the first PoC passes.

## Next

- **Establish reliable everyday use of the verified rule.** Before it moves up: the
  initial PoC and CLI are complete, and actual use identifies a concrete gap. Review
  stale verification evidence after relevant engine/rule changes without adding a
  resident CommandFence process.

## Later

- **Additional command patterns or a menu-bar UI.** Consider only after the initial
  rule works and a real need is identified. Each requires an explicit product-scope
  decision; neither is a first-version requirement.
