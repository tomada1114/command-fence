# ADR-0004: Use synchronous ports and separate evidence in status

- **Status:** Proposed
- **Date:** 2026-10-02
- **Deciders:** the owner

## Context

Requirements §§3.2–3.3 need operation decisions and separate engine, rule, and manual
verification evidence. The existing core boundary and Linux coverage floor already
require I/O through synchronous ports and values supplied as arguments.

## Decision drivers

- Test safety decisions without a real engine or administrator access.
- Preserve typed distinctions among unavailable, historical, matching, and incomplete.
- Keep adapters translational and retain the existing workspace/dependency direction.

## Considered options

1. **Three domain ports plus existing Clock** — explicit calls and one status view.
2. **A general command-runner port** — expose CLI mechanics to core decisions.
3. **Async service and a protected boolean** — add lifecycle while collapsing evidence.

## Decision

Propose `ConfigSource` (bounded captured input), `ExecutionEngine` (ordinary status,
privileged inspection, single-rule import/removal), and `RuleStateStore` (trusted
receipt, scoped transaction/journal, manual summary). Each is synchronous `Send + Sync`
over core types, with typed errors, a fake, and one shared contract. Reuse `Clock` and
`UnixMillis`; add no crate, async runtime, new workspace member, or user environment
variable. Core owns validation, conflict/ownership decisions, transitions, and views.

The Santa adapter invokes a verified packaged executable with argument arrays, bounded
captured output, and child timeout/termination/reaping. Initial tuning: 10 seconds per
child, 1 MiB status/diagnostic output, 16 MiB exported JSON. Exceeding a bound stops the
operation; a post-submission timeout is incomplete. Fixtures replace the process
boundary during tests; no routine gate invokes a real privileged engine operation.
Linux reports live mutation unsupported while running core and fixture contracts.

One `StatusView` supplies text and versioned JSON. Adopt the group names, state
vocabulary, nullable UTC Unix-millisecond timestamps, and verification semantics in
[the design](../../architecture.md#core-operations). Ordinary receipt data never
produces a fresh `matching` rule observation. A manual summary is user-authored,
historical, and bound to the recorded engine/rule generation and tested contexts.
Missing permission diagnosis must have evidence; an unexplained failure is unavailable.
Malformed manual data is unavailable evidence, never a protection assertion.

## Consequences

### Positive

- Full operation call-order/failure tests stay inside core's coverage floor.
- Scripts inspect typed observations without parsing prose or inferring protection.

### Negative

- Port contracts and compatibility fixtures must evolve together.
- Ordinary status cannot prove current root-only registration or unobserved restarts.

### Follow-ups

- Implement ports/views/fakes, then Santa/state adapters and CLI contract tests.
- Preserve existing gates while replacing the scaffold, as specified in ADR-0005.

## Open questions

- Acceptance needs the owner's confirmation of the ports, initial bounds, and JSON v1.
- Unverified: live status/version observations and useful restart context. Unknown
  context stays unknown; the implementation must not invent freshness.

## Sources

- [2026.8 status CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandStatus.mm)
  — non-root JSON observations; checked 2026-10-02.
- Repository `Cargo.toml`, core clippy config, and architecture layers — inspected
  2026-10-02; existing synchronous boundary, dependencies, and gates.

## Related

- [ADR-0001](0001-signed-santa-and-human-poc.md) supplies the external engine boundary.
- [ADR-0003](0003-owned-state-and-conservative-mutations.md) owns trusted evidence.
