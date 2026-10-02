# CommandFence — Requirements

- **Status:** Signed off 2026-10-02. The user selected all four recommended remaining
  decisions and delegated further routine choices in the recommended, simple direction.
- **Platform and stack:** Personal macOS CLI based on rust-template; the official signed
  Santa release provides pre-execution enforcement. Source-research baseline: Santa
  2026.8. Live feasibility is not yet verified.

## 1. Overview

CommandFence helps this Mac's owner prevent an AI agent from accidentally executing a
specific command. The owner edits a local configuration, previews it, manually applies
it with administrator privileges, and checks the resulting state. Santa evaluates the
execution before it starts, across the apps, terminals, and sessions receiving its
execution authorization coverage. CommandFence itself need not remain running.

The initial match is the actual executable `/bin/ls` with exactly one argument after
`argv[0]`: the configured owner's absolute home path, illustrated as `/Users/example`. It is an
execution-vector match rather than a match on the shell text `ls ~`. The match does not
restrict the caller or effective user ID.

## 2. Scope

### MVP

- Validate one versioned JSON configuration and preview the generated Santa rule (§3.1).
- Manually apply/remove that rule, preserve existing policy, and verify results (§3.2).
- Report engine state, rule evidence, and runtime verification separately, as text or
  JSON (§3.3).
- Pass a human-run harmless-ls PoC early in implementation, before adding targets (§3.4).

### Later

Additional commands, options and relative-path variants; refusal-history presentation;
menu-bar UI. Add these only after the first execution-control PoC succeeds and a real
need is established.

### Non-goals

A TUI or GUI in the first version; a resident CommandFence process; automatic rule
reload or elevation; automatic Santa installation, restart, or permission changes;
cloud sync/accounts; multiple Macs; distribution, release artifacts, notarization or
Apple Developer Program enrollment; deliberate tamper resistance; preventing another
program from reading the same directory; tests that execute dangerous commands.

## 3. Features

### 3.1 Configuration and preview

Default file: `~/.config/command-fence/config.json`.

```json
{
  "version": 1,
  "rules": [
    {
      "id": "deny-home-ls",
      "executable": "/bin/ls",
      "args": ["/Users/example"]
    }
  ]
}
```

`args` excludes `argv[0]`. Version 1 supports exactly one `/bin/ls` rule with one literal
argument equal to the selected target home. An empty list does not implicitly remove a
live rule. Free-form CEL and generic arbitrary-command policies wait for later work.

Use a real JSON parser. Proposed implementation limits, chosen under the user's
simplicity delegation: UTF-8 input up to 64 KiB; version must be integer 1; IDs use 1–64
ASCII lowercase letters, digits, or hyphens. Reject unknown/duplicate fields or IDs,
missing values, NUL characters, unsupported versions, and unsupported rule shapes.

Do not expand `~` or environment variables, infer the path from root's HOME, or
normalize execution arguments. Validate configuration before touching Santa. Invalid
configuration leaves existing rules unchanged. A generated preview is not proof of
Santa CEL compilation or enforcement.

`ls -la ~`, relative paths, no-argument `ls`, a trailing slash, `--` as an extra
argument, another `ls` executable, and Python directory enumeration are outside this
literal rule. A shell alias that adds flags likewise does not match.

### 3.2 Manual application and removal

CommandFence owns explicit apply/remove operations. It never invokes sudo itself.
Preview and validation run as the normal user. Privileged operations require an
explicit absolute `--config` path, so elevation never changes which home is targeted.
Read that file once, validate it, and generate the rule from the captured values. A
separate durable plan-file format is not required for this first version.

Before a mutation, check engine version/availability, management state, relevant rule
identities and precedence, and existing owned-rule evidence. Refuse foreign/conflicting
rules, uncertain ownership, unsupported versions, configured sync management, or
StaticRules. Do not overwrite a foreign rule or change Santa mode/settings.

Import only the one generated SigningID CEL rule, without cleanup flags. Its identifier
is `platform:com.apple.ls`; its initial expression denies only the literal vector:

```cel
!(path == "/bin/ls" && size(args) == 2 && args[1] == "/Users/example")
```

Read back the exact registered rule and compare unrelated exported execution rules.
An import exit code of zero, a rule count, or a saved receipt alone is insufficient to
claim success. Warnings, skipped rules, mismatches, and failed post-write verification
produce an incomplete/unknown result, retain recovery evidence, and never trigger a
whole-database restore. An export's confirmed empty baseline must be distinguished
from an export failure; uncertainty stops mutation.

Reapplying an unchanged owned rule is idempotent. For recovery, retain the latest
successful applied-rule receipt and pre-change execution-rule evidence. Remove only
the owned identity whose current content still matches that receipt; foreign or
concurrently modified content stops the action. If recovery needs the preceding owned
contents restored, use the preserved evidence in a separate human-run targeted recovery
step; remove itself always removes the current owned rule.
A comment alone does not establish ownership. Never remove a rule by installing a broad
allow rule in its place. Backup scope must accurately describe Santa's export, which
omits static and transitive execution rules and is not a complete management backup.

### 3.3 State reporting

Show three separate observations:

| Observation | Meaning |
|---|---|
| Engine | Missing, unavailable/unknown, diagnosed permission issue, or reachable with version/mode. |
| Rule | Not applied, privileged verification needed, matching, pending config change, missing/changed/conflicting, or incomplete apply. |
| Runtime verification | Not yet tested, a dated human-run PoC result, or stale evidence after a relevant change. |

Normal status is read-only and never raises a password or OS permission prompt. A
saved receipt is historical evidence; if current rule contents cannot be checked,
report `unverified`. Explicit privileged verification is available as a separate
human-invoked option. Ordinary status uses Santa's non-root JSON status command.

Do not display a missing/stopped/unreachable engine or an unverified/unknown rule as
protected. Do not claim a specific missing OS permission without diagnostic evidence.
Report actual observations and dates instead of a single unconditional protected flag.
Status never runs the target command, starts/stops Santa, or creates notifications.

Provide plain text and `status --json`. Save only the latest successful application and
latest manual PoC summary; no refusal-history database or additional monitoring. Do not
collect directory entries or unrelated process arguments. Existing Santa logs, sync,
and retention settings are preserved.

### 3.4 Safe PoC acceptance gate

PoC is required in the first implementation milestone, before broadening targets.
Requirements, CLI UX, and repository preparation may proceed with runtime feasibility
explicitly unverified. No live setup or rule change is authorized by requirements
sign-off; present the exact package, OS permissions, single payload, test matrix, and
targeted recovery before seeking live approval.

The human performs UI/password-bearing setup and tests. Use the official signed
standard release, verify version/signature/published digest, inspect baseline policy,
and prepare targeted recovery before application. A fresh standalone installation uses
Monitor mode; do not change an existing installation's mode or management to make a
check pass. No automated gate shows a Santa dialog, takes focus, or runs a live denial.

| Harmless case | Expected result from this rule |
|---|---|
| Target vector in an interactive terminal | Pre-execution denial. |
| Same vector in another terminal/session | Denial. |
| Same vector from an AI-agent/noninteractive shell | Denial. |
| Same vector from a direct process-launch harness | Denial. |
| Same vector after CommandFence exits | Denial while Santa remains operational. |
| `/bin/ls /tmp` | Allowed. |
| `/bin/ls -la` with the home argument | Allowed. |
| Relative `.` or no argument in the home | Allowed. |
| Home path with trailing slash, or extra `--` argument | Allowed. |
| Target and `/tmp` alternating for at least three cycles | Each decision remains correct. |
| Target after removing the newly created owned rule | Returns to the observed baseline; other rules/settings unchanged. |

Discard directory-listing output. Record case/context, versions, rule identity/content,
decision evidence, and restoration outcome. Child exit failure alone does not prove
Santa blocked it before execution. Record tested coverage without claiming an
unlimited guarantee. Simulate stopped/missing/permission-error states in automated
tests rather than stopping the real engine. Recheck after a human-initiated reboot or
engine restart before treating prior runtime evidence as current.

## 4. Cross-cutting rules

All input and state decisions live in the deterministic core. Platform adapters perform
I/O, environment lookup and calls to Santa; the CLI composes and renders. Keep the
existing template's gates and hooks. Removing the sample TUI is later implementation
work in the new repository, not a reason to alter rust-template now.

English CLI wording, plain stdout data, stderr diagnostics, and 0/1/2 exit conventions
follow the template. Exact UX/output contract is in [UX guidelines](../design/ux-guidelines.md).
Local configuration and runtime receipts must not be committed as repository examples;
use a neutral sample home path when copying documents into a repository.

## 5. Data

| Data | Location/lifetime |
|---|---|
| Configuration | config.json; user edits manually; never auto-applied. |
| Last successful apply receipt and immediate recovery evidence | Under the explicit user's CommandFence config/state location; one retained generation, not a history database. |
| Manual PoC summary | Latest dated summary only; no captured listing contents. |
| Santa live rules/logs/settings | Santa's own storage; only the owned target rule may be changed. |

Architecture must settle receipt ownership/permissions, safe write handling, and
concurrency before privileged implementation. These implementation choices do not add
new product scope.

## 6. Open items

- Runtime CEL/enforcement feasibility: close with the separately approved safe PoC.
- Repository initialization: approved public `tomada1114/command-fence`;
  later remote writes still require their own scope.
- Personal permission files: the separate stage 5 gate.
- Santa installation, OS permissions, and live application: separate explicit approval.
- Platform/engine/configuration ADRs and any new dependency: stage 7 with template policy.

No product-scope question remains that prevents CLI UX/design.

## 7. Decision log

- 2026-10-02: CommandFence / command-fence; Mac-wide execution authorization goal.
- 2026-10-02: Literal `/bin/ls` plus one absolute-home argument; broader listing bans deferred.
- 2026-10-02: Personal use, no distribution, no Apple Developer Program membership.
- 2026-10-02: Mistake prevention; deliberate tampering is outside scope.
- 2026-10-02: Rust CLI from rust-template plus the official signed Santa engine, following
  language/mechanism comparison. A resident CommandFence process and first-version UI
  are not required.
- 2026-10-02: JSON v1 selected over TOML; reuse existing JSON facilities and keep one format.
- 2026-10-02: Explicit CommandFence apply/remove selected over manual-only santactl steps;
  include preflight/readback, without automatic sudo.
- 2026-10-02: Text and JSON state selected over text-only; keep engine/rule/PoC evidence distinct.
- 2026-10-02: PoC selected as the first implementation gate rather than a pre-sign-off gate.
- 2026-10-02: User delegated recommended routine choices and asked for simplicity. Use five
  commands and direct config input; avoid a mandatory separate plan-file format.

Source claims were checked 2026-10-02:
[release 2026.8](https://github.com/northpolesec/santa/releases/tag/2026.8),
[authorization](https://northpole.dev/features/binary-authorization/),
[rule CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandRule.mm),
[rule table](https://github.com/northpolesec/santa/blob/2026.8/Source/santad/DataLayer/SNTRuleTable.mm),
[status CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandStatus.mm),
[argument input](https://github.com/northpolesec/santa/blob/2026.8/Source/common/es/EndpointSecurityAPI.mm),
[installation](https://northpole.dev/deployment/install-package/).
