# ADR-0005: Keep five plain CLI operations and lock their presentation

- **Status:** Accepted 2026-10-02
- **Date:** 2026-10-02
- **Deciders:** the owner

## Context

Requirements, CLI flows, UX guidelines, and design direction already settle a CLI-only
first version. The current template's counter, TUI, and optional model example are
scaffold. This ADR transcribes the presentation lock; it does not reopen product scope.

## Decision drivers

- One direct interface usable by the owner and scripts.
- Plain output with explicit uncertainty and no dependency on terminal appearance.
- Preserve development gates while retiring sample behavior.

## Considered options

1. **Five plain operations** — use existing clap and one wording module.
2. **Retain the sample TUI as product UI** — add screens and terminal lifecycle work.
3. **Add a desktop/menu-bar UI** — expand scope and presentation dependencies.

## Decision

Keep `validate`, `preview`, `apply`, `remove`, and `status`, with standard help/version.
Keep explicit `--config`, `status --json`, and privileged `status --verify` as specified
in [CLI flows](../../product/ux-flows.md). No sixth PoC/import-record command, automatic
elevation, resident process, installation wizard, or refusal history is added.

The binding lock is [design direction](../../design/design-direction.md#presentation-roles):
English wording, complete lines, stdout data/stderr diagnostics, 0/1/2 exits, no ANSI
colors, spinner, emoji, cursor control, or application font/theme. The terminal owns
light/dark colors; contrast measurement has no application-selected pair to evaluate.
Display engine, registered-rule evidence, and last execution test separately in both
formats. Every user-facing sentence remains in `wording.rs`.

Replace the sample with the real core and commands in coverage-preserving changes.
Remove counter/TUI code and unused ratatui/model-example dependencies when their
callers leave; keep reusable clock/logging, hooks, xtask, Linux checks, and all gate
values. Dependency removal is separate from permission to add a dependency back.
Use the repository's sample-removal checklist and update affected docs/skill examples
and their authored/mirrored copies together. Do not hand-edit the mirror.

## Consequences

### Positive

- Pipe-safe, accessible output and a small interface without UI assets or tokens.
- No new presentation library or resident CommandFence lifecycle.

### Negative

- Manual config editing remains part of use; a menu-bar interface needs a later scope
  decision. Retiring sample CLI contracts needs an explicit changelog entry.

### Follow-ups

- Implement output contracts, then replace scaffold examples and update README,
  getting-started, architecture contracts, agent examples, and changelog in the final
  integration work after the first PoC gate.

## Open questions

- Acceptance needs the owner's formal confirmation of this record; the already
  agreed product/presentation decisions need no further hearing.

## Sources

- Repository requirements, CLI flows, UX guidelines, design direction,
  `starting-an-app`, and current scaffold — inspected 2026-10-02.

## Related

- [ADR-0004](0004-synchronous-ports-and-evidence-status.md) owns status values.
- [ADR-0001](0001-signed-santa-and-human-poc.md) holds runtime feasibility.
