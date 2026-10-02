# ADR-0002: Capture JSON v1 and generate one literal rule

- **Status:** Proposed
- **Date:** 2026-10-02
- **Deciders:** the owner

## Context

Requirements §3.1 already selects `~/.config/command-fence/config.json`, JSON v1,
and exactly one `/bin/ls` rule with one configured absolute home argument excluding
`argv[0]`. Elevation must not substitute root's home or reinterpret the target.

## Decision drivers

- Keep editing, validation, preview, and explicit application straightforward.
- Produce identical semantic content from identical captured input.
- Bound input and prevent configuration text from becoming code or shell syntax.

## Considered options

1. **Direct strict JSON input and a fixed generator** — reuse `serde` and `serde_json`.
2. **A durable plan document** — add a second format and stale-plan reconciliation.
3. **Free-form CEL** — give configuration an arbitrary executable policy language.

## Decision

Use the agreed config schema unchanged, with UTF-8 input limited to 65,536 bytes.
Version is integer 1; rule IDs contain 1–64 ASCII lowercase letters, digits, or
hyphens. Reject unknown/duplicate fields, duplicate IDs, unsupported shapes, NUL,
missing values, relative paths, and unsupported versions. The chosen home string is
literal; no tilde, environment, symlink, Unicode, or trailing-slash normalization.

Privileged operations require an explicit absolute config locator. Capture a regular
file's bytes and identity once; validate that capture before inspecting or changing
Santa. Root operation input must not be a symlink or writable by unrelated users.
Editing the file after capture cannot alter the payload submitted by that invocation.
Remove uses the receipt's binding even if the file is now missing or invalid.

The sole engine identity is `(SIGNINGID, platform:com.apple.ls)`, with policy `CEL`.
Generate the following expression using a real string-literal encoder:

```cel
!(path == "/bin/ls" && size(args) == 2 && args[1] == "<target-home>")
```

Use JSON-compatible CEL escapes and literal UTF-8 for non-BMP characters; never emit
UTF-16 surrogate escapes into CEL. Serialize the outer rule document separately with
`serde_json`. Keep message/comment wording fixed and free of private path values.
Test quotes, backslashes, control escapes, and Unicode using neutral fixtures; those
tests prove generation, while only Santa proves compilation and enforcement.

Preview contains one rule and does not contact Santa. A matching execution denies;
nonmatching executions return allow under this CEL rule. That allow can override
lower-precedence policy, so the preflight in ADR-0003 must assess the whole relevant
precedence chain. Config IDs do not authorize adopting an existing Santa rule.

## Consequences

### Positive

- One editable format and no new dependency, shell parsing, or plan lifecycle.
- Small deterministic core with precise validation and generation fixtures.

### Negative

- Literal variants remain outside scope; validation cannot prove live enforcement.
- One SigningID identity cannot represent several independently owned home rules.

### Follow-ups

- Implement config types, typed errors, generator, and `validate`/`preview` contracts
  in [the proposed module homes](../../architecture.md#principles-and-shape).

## Open questions

- Acceptance needs the owner's confirmation of the implementation bounds and encoding.
- Unverified: the generated literal compiles and matches the real engine; close with
  ADR-0001's PoC before expanding the supported input/rule surface.

## Sources

- [CEL language definition](https://github.com/cel-expr/cel-spec/blob/master/doc/langdef.md)
  — string literals and surrogate restrictions; checked 2026-10-02.
- [Santa argument collection](https://github.com/northpolesec/santa/blob/2026.8/Source/common/es/EndpointSecurityAPI.mm)
  and [CEL activation](https://github.com/northpolesec/santa/blob/2026.8/Source/santad/CELActivation.mm)
  — full execution argument input; checked 2026-10-02.

## Related

- [ADR-0001](0001-signed-santa-and-human-poc.md) is the engine/PoC gate.
- [ADR-0003](0003-owned-state-and-conservative-mutations.md) prevents foreign adoption.
