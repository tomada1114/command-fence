# Roadmap

This page records the initial direction approved during kickoff. Product scope is in
[AGENTS.md](../../AGENTS.md#product) and [requirements](../product/requirements.md).
The initial backlog is filed under the two tracking outcomes below. The owner accepted
the five initial ADRs on 2026-10-02, so implementation can start with the first issue.
The concrete private live-work package still needs review before live setup and testing.
The [design](../architecture.md#commandfence-design-proposal) and
[five accepted ADRs](README.md#decisions) describe the implementation; they do not
change these approved horizons.

- **Last reviewed:** 2026-10-02; nine kickoff issues are open (two tracking, seven work).

## Now

- **Prove the first harmless execution rule.** Done when the separately approved,
  human-run [PoC matrix](../product/requirements.md#34-safe-poc-acceptance-gate) shows
  pre-execution denial across the tested launch contexts, allows its controls, and
  restores the baseline while preserving unrelated policy. Before live work: approve
  the concrete setup, payload, and recovery procedure. Runtime feasibility is currently
  unverified.
  The detailed [human procedure](safe-poc.md) requires a concrete private live package.
  Tracking: [#2](https://github.com/tomada1114/command-fence/issues/2). The human PoC
  remains [externally blocked](https://github.com/tomada1114/command-fence/issues/8).
- **Manage one rule through a plain CLI.** Done when the five agreed operations follow
  the [CLI flows](../product/ux-flows.md), fixtures cover unavailable and conflicting
  engine states, and the inherited project checks pass. Replace the sample counter/TUI
  with this behavior; do not broaden the target before the first PoC passes.
  Tracking: [#3](https://github.com/tomada1114/command-fence/issues/3). Status and final
  integration follow the PoC pass; body dependencies carry the implementation order.

## Next

- **Establish reliable everyday use of the verified rule.** Before it moves up: the
  initial PoC and CLI are complete, and actual use identifies a concrete gap. Review
  stale verification evidence after relevant engine/rule changes without adding a
  resident CommandFence process.

## Later

- **Additional command patterns or a menu-bar UI.** Consider only after the initial
  rule works and a real need is identified. Each requires an explicit product-scope
  decision; neither is a first-version requirement.
