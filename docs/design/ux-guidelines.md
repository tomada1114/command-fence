# CommandFence — UX guidelines

Status: Settled 2026-10-02 under the user's recommended-direction/simplicity delegation.
Platform: personal macOS CLI. Related: [requirements](../product/requirements.md),
[flows](../product/ux-flows.md), [presentation direction](design-direction.md).

## Interaction

Five direct commands, standard help, and editable JSON are the complete first-version
interaction. Preserve read-only preview and explicit privileged apply/remove. Normal
commands never request a password, take over a terminal, or launch an installer.

Use the template's English wording and noninteractive clap conventions. Native desktop
navigation, web breakpoints, animation, focus rings, login states, and touch targets do
not apply. The user chooses terminal font, zoom, foreground and background. Output
works in pipes and with terminal assistive technology.

## Feedback and streams

- Stdout is the result; stderr carries `error: ...` or `warning: ...` diagnostics.
- No ANSI sequences, spinner, terminal control, emoji, banner, or cursor rewrites.
  Emit complete lines so redirected output remains readable.
- Preview emits valid Santa-import JSON only. `status --json` emits one documented JSON
  object only, with a documented UTC timestamp representation when a timestamp exists.
- Report successful application/removal only after verified completion, not submission.
- Errors use stable wording for the failed concept and next safe action. Do not log raw
  configuration, home paths, unrelated process arguments, or directory listings.
  Explicit preview data contains the literal configured home for the owner to inspect.

## Exit contract

| Exit | Meaning |
|---|---|
| 0 | Action/report completed, including help/version and a status report containing an unavailable/unverified state. |
| 1 | Validation, application, removal, explicit verification, reading required input, or writing output failed. |
| 2 | clap usage error, before adapters are touched. |

A successful status exit is not a protection assertion: scripts inspect engine, rule,
and runtime-evidence fields. A successfully classified degraded state may include a
stderr warning. Failed explicitly requested privileged verification exits 1 and does
not emit a success report. These semantics preserve the template's 0/1/2 convention.

## States and safe action

| State | Output/action |
|---|---|
| Missing config | Error; refer to example/help. Do not create/apply automatically. |
| Invalid/unsupported config | Error; keep file and live rule. |
| Missing Santa | Not installed; installation is a separately approved human step. |
| Unreachable engine | Unavailable/unknown; no auto-restart or invented diagnosis. |
| Diagnosed extension/FDA problem | State observed problem; human owns OS changes. |
| Non-root rule readback | Unverified; document explicit privileged verification. |
| Sync/static/foreign/conflicting policy | Reject mutation; preserve policy. |
| Edited config differs from applied receipt | Pending change; no auto-apply. |
| Import warning/failed post-write verification | Incomplete; preserve evidence; no broad rollback. |
| Matching current rule | Registration evidence with a separate dated PoC observation. |
| Changed engine/rule/reboot context | Earlier runtime evidence is historical/stale until checked. |

## Machine-readable status

Use a versioned JSON object with `engine`, `rule`, and `runtime_verification` groups.
Distinguish `unverified` from `matching`. Missing observations use typed states/null
values, not invented versions or a true protected flag. Pin field names/types in CLI
contract tests. Human PoC evidence is labeled historical/manual, not continuous proof.

No first-version refusal timeline or watch command. State queries do not execute the
prohibited vector. Rule verification is a human-invoked privileged read-only action.

## Accessibility and verification

Every operation is keyboard/script accessible. Status meaning appears in words; color,
symbol, position, and animation are never the only carrier. Text is a short line-oriented
report usable at terminal zoom/narrow widths; JSON is independent of locale or terminal.

Implementation verification captures stdout/stderr and exit status, parses JSON, and
uses missing/unknown/permission/conflict fixtures. No gate requires a TTY, raw mode,
real keyboard input, Santa installation, or a permission prompt. App-selected contrast
measurement does not apply because CommandFence chooses no output colors.

## Decision ledger

| Decision | Source/reason | Rejected |
|---|---|---|
| Five direct commands | User asked for simplicity; inspect and undo remain necessary. | GUI/TUI, wizard, mandatory plan format. |
| Explicit privileged writes, ordinary read-only status | Accepted scope and Santa privilege boundary. | Auto-sudo/automatic repair. |
| Separate engine/rule/PoC rows | Registration and execution are different evidence. | Green state inferred from a receipt. |
| Text plus JSON; no color/control | User choice and template CLI policy. | Decorative terminal UI, parsing human text. |
| English only | Template fixed language and minimal scope delegation. | First-version localization. |

References checked 2026-10-02: template
[CLI skill](https://github.com/tomada1114/rust-template/blob/d67059f8994a6ed55951b9e094f1619b1e75dec5/.agents/skills/designing-clis/SKILL.md),
Santa [rule CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandRule.mm)
and [status CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandStatus.mm).
No additional UX question blocks repository preparation; implementation/live evidence
remain later work.
