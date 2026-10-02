# ADR-0003: Keep trusted ownership state and stop on uncertain mutations

- **Status:** Accepted 2026-10-02
- **Date:** 2026-10-02
- **Deciders:** the owner

## Context

Requirements §§3.2 and 5 require verified ownership, idempotent reapplication,
targeted removal, and one generation of recovery evidence. A user-editable receipt
or rule comment cannot safely authorize a privileged overwrite or removal.

## Decision drivers

- Preserve all unrelated rules and management settings.
- Keep ownership evidence distinct from editable desired configuration.
- Survive a partial operation without claiming success or losing recovery evidence.

## Considered options

1. **Root-owned singleton receipt, lock, and journal** — fit the one-rule product.
2. **Authoritative receipt beside config** — ordinary user can alter mutation authority.
3. **A database or service** — additional schema, lifecycle, and dependencies.

## Decision

Propose `/var/db/command-fence/` as the macOS state root, created only by explicitly
invoked privileged operations. Use its trusted system-resolved parent; reject
user-owned/writable parents, symlinked state entries, unexpected owners, and unsafe
permissions. Never repair those conditions automatically or redirect writes with HOME.

| File | Owner/mode | Purpose |
|---|---|---|
| State directory | root, 0755 | Fixed location for the one owned identity. |
| `receipt.json` | root, 0644 | V1 receipt; ordinary status can read historical evidence. |
| `operation.lock` | root, 0600 | Stable lock file, never removed or replaced. |
| `recovery.json` and private import/export temporary files | root, 0600 | Prior receipt/rule, baseline, captured payload, operation phase. |

The receipt binds the explicit absolute config locator and source-file owner UID to
the complete owned rule and a checked monotonic generation. Changing the binding is
a conflict, not an automatic adoption. Store last successful apply separately from
latest verified removal so history survives removal. Readers check root ownership,
permissions, and format; inaccessible/corrupt state is different from absent state.

Use `File::try_lock` for a scoped mutation lease; a second CommandFence operation
fails busy. Persist and sync a pending journal before changing Santa. Write receipt
and journal via exclusive temporary files in the trusted directory, sync, rename,
and sync the directory. A failed operation keeps the prior receipt and unresolved
journal; the next mutation refuses until a separately reviewed recovery resolves it.
Never overwrite incomplete recovery evidence with a new attempt.

Preflight compares full typed rule content, not comments or a count. Refuse sync,
StaticRules, unsupported/unknown observations, higher-priority rules that mask the
target, and relevant lower-priority policy that the CEL allow branch would override.
A same-identity foreign rule conflicts even if its content resembles the preview.
Only a trusted receipt plus fresh content agreement authorizes replacement/removal.

Capture the complete supported execution export, with identity/content comparisons
for unrelated entries. Missing, malformed, or truncated output is a failed baseline.
The 2026.8 export can exit 1 for an empty database; accept empty only when the exact
version-specific empty response, fresh zero counts, and absent target agree. An export
is not a full Santa backup: static/transitive rules and management are outside it.
Unknown fields or omitted policy cannot be silently treated as preserved evidence.

Immediately recheck management, target, and baseline before the write; any detected
change stops it. After the write, verify the complete target and unchanged unrelated
exported content. Warnings, skipped CEL, timeout, failed readback, or failed receipt
commit result in incomplete status and exit 1. Apply never imports cleanup flags.
Remove deletes the matching current identity; restoring prior owned content is a
separate human operation, never an allow rule or whole-database import.

The local lock excludes only CommandFence. The inspected CLI has no conditional
mutation token, so a different administrator can race between recheck and write.
Require an exclusive human maintenance window; post-write differences are detected
and preserved, but preflight cannot guarantee that a foreign concurrent update was
never overwritten. Owner acceptance must explicitly consider this residual risk.

## Consequences

### Positive

- No writable user record grants rule ownership, and ordinary status still works.
- One generation, one stable lock, no new database or crate dependency.

### Negative

- Readable receipts disclose their configured target/locator to other local accounts;
  recovery exports remain root-private. More restrictive access would need a new design.
- Missing trusted state never grants adoption; an interrupted first apply may need
  manual targeted recovery. External administrator races are not made atomic.

### Follow-ups

- Implement `RuleStateStore` with isolated filesystem and fault-contract tests.
- Implement the operation state machine with foreign/change/crash/readback fixtures.
- Validate the empty-export classification and targeted recovery in the human PoC.

## Open questions

- Acceptance needs the owner's confirmation of the state root, receipt readability,
  exclusive maintenance precondition, and external-race limit.
- Unverified: real empty-export diagnostics and crash/recovery behavior on the Mac.
  If dependable classification cannot be established, live mutation remains blocked.

## Sources

- [2026.8 rule CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandRule.mm)
  — privileges, management restrictions, export scope/empty behavior and warning
  reporting; checked 2026-10-02.
- [2026.8 rule table](https://github.com/northpolesec/santa/blob/2026.8/Source/santad/DataLayer/SNTRuleTable.mm)
  — replacement semantics and invalid-CEL handling; checked 2026-10-02.
- [Rust File locking](https://doc.rust-lang.org/std/fs/struct.File.html#method.try_lock)
  — scoped exclusive lock support, stable since 1.89; checked 2026-10-02.

## Related

- [ADR-0002](0002-json-config-and-literal-rule.md) supplies captured rule content.
- [ADR-0004](0004-synchronous-ports-and-evidence-status.md) classifies incomplete state.
