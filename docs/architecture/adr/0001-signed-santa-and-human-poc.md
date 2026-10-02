# ADR-0001: Use signed Santa and gate feasibility with a human PoC

- **Status:** Proposed
- **Date:** 2026-10-02
- **Deciders:** the owner

## Context

Requirements §§1–3.4 already select a personal Rust CLI and official signed Santa.
The execution-control goal crosses applications and sessions; the inherited counter
and TUI provide no enforcement. Runtime CEL evaluation and denial remain unverified.

## Decision drivers

- Deny the literal target before execution across tested launch contexts.
- Avoid building, signing, or entitling a new Endpoint Security client.
- Preserve installed policy and separate development from live administrative work.

## Considered options

1. **Official signed Santa plus CLI management** — reuse its authorization engine.
2. **A custom Endpoint Security client** — own a privileged service and its lifecycle.
3. **A shell wrapper** — depend on every caller passing through a controlled shell.

## Decision

Use the standard official Santa package as an external engine, with 2026.8 as the
initial source/fixture baseline. Initially permit privileged mutation only for the
reviewed 2026.8 compatibility family; do not assume later versions are compatible.
Unknown versions may be reported but cannot be mutated until separately reviewed.

Santa receives the execution events; CommandFence invokes packaged `santactl` with
argument arrays and never invokes sudo, installs a package, grants permissions,
restarts Santa, changes mode, or changes management. The proposed packaged command
location must be checked during the PoC; privileged calls never search user PATH.

The human owns any Santa system-extension approval and Full Disk Access grant.
CommandFence requests no TCC permission of its own. A new installation is proposed in
Monitor mode; an existing mode, sync server, or StaticRules configuration is preserved.
Managed or uncertain policy stops mutations. Keep Linux builds and fixture tests;
the Linux live adapter reports unsupported rather than dropping the existing CI target.

Before the first milestone is accepted, run the [safe PoC](../safe-poc.md). Shell
wrappers do not satisfy direct launches; a custom service adds work outside the chosen
scope. The PoC records tested coverage without asserting that every OS execution is
covered. No broader target is added before it passes.

## Consequences

### Positive

- One CLI and one existing engine; no CommandFence resident process or new entitlement.
- Development and CI use deterministic fixtures without taking over the owner's Mac.

### Negative

- Installation and privileged rule observations require the human.
- Engine upgrades and authorization coverage need fresh evidence; feasibility may fail.

### Follow-ups

- Implement `ExecutionEngine` and the bounded Santa adapter, as mapped in
  [the design](../../architecture.md#commandfence-design-proposal).
- Prepare the exact package/version/digest, permissions, payload, baseline, and targeted
  recovery for separate approval before any live work.

## Open questions

- Acceptance needs the owner's confirmation of this proposal; product direction alone
  does not accept this ADR or authorize live setup.
- Unverified: compilation, argument handling, caching, denial, and recovery on the
  actual Mac. The human matrix is the acceptance evidence.

## Sources

- [Santa 2026.8 release](https://github.com/northpolesec/santa/releases/tag/2026.8) —
  baseline release and standard package direction; checked 2026-10-02.
- [Package installation](https://northpole.dev/deployment/install-package/) and
  [TCC deployment](https://northpole.dev/deployment/profile-tcc/) — human setup;
  checked 2026-10-02.
- [Authorization](https://northpole.dev/features/binary-authorization/) and
  [known limitations](https://northpole.dev/limitations/) — engine scope;
  checked 2026-10-02.

## Related

- [ADR-0002](0002-json-config-and-literal-rule.md) defines the one rule.
- [ADR-0003](0003-owned-state-and-conservative-mutations.md) constrains every mutation.
