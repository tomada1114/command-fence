# CommandFence — CLI flows

Status: Settled 2026-10-02 from the accepted requirements and the user's delegation to
choose recommended, simple defaults. These are proposed commands, not an implemented
binary. See [requirements](requirements.md) for scope and [UX guidelines](../design/ux-guidelines.md)
for shared output/state rules.

## Operation inventory

| Command | Purpose | Privilege |
|---|---|---|
| `command-fence validate` | Check the one configuration file. | Normal user. |
| `command-fence preview` | Print the generated Santa rule as JSON without applying it. | Normal user. |
| `command-fence apply --config <absolute-path>` | Validate, preflight, import the owned rule, read back, save success/recovery evidence. | Human explicitly invokes with sudo. |
| `command-fence remove --config <absolute-path>` | Remove the current owned matching rule using its receipt. | Human explicitly invokes with sudo. |
| `command-fence status` | Inspect engine and available evidence; support `--json` and explicit privileged `--verify`. | Normal status is non-root. Verification is human-invoked with sudo and an absolute config path. |

`--help` and `--version` are standard clap surfaces. `--config` is an optional absolute
override for normal-user commands and required for privileged actions. There is no
editor, interactive menu, extra plan-file format, automatic elevation, or background UI.

## 1. Prepare and preview

1. Create/edit `~/.config/command-fence/config.json` using the documented example.
2. Run `validate`; correct any structural/unsupported-input error.
3. Run `preview`; inspect the literal executable/home match in the JSON payload.
4. Invoke `apply` explicitly when ready; no earlier step changes Santa.

```text
[Edit config] -> [validate] -> [preview JSON] -> [Human chooses apply]
                    |
                    +-> [Error; keep file/live rules] -> [Edit again]
```

Example validation output:

```text
Configuration: valid
Rules: 1
Engine validation: not performed
```

A missing file produces an error and help points to the example; it does not create or
apply a default automatically. `preview` emits only the JSON payload on stdout.

## 2. Apply

The human invokes `sudo command-fence apply --config <absolute-path>`. Read the file
once; validate it, confirm reachable supported Santa and nonconflicting local policy,
preserve baseline evidence, import the single rule without cleanup, then read it back.

```text
[Explicit apply] -> [Validate + inspect] -> [Preserve baseline] -> [Import one rule]
                         |                                         |
                         +-> [Conflict/unknown: no mutation]       v
                                                              [Read back]
                                                               |       |
                                                               v       v
                                                          [Matches] [Incomplete]
                                                               |       |
                                                        [Save receipt] [Retain evidence;
                                                                         report recovery]
```

Success:

```text
Rule: applied and verified
Execution test: not performed
```

The explicit sudo apply invocation is the write action; no additional in-command
confirmation is needed. A config edited since preview is revalidated and applied from
the values captured at this invocation, not from a prior preview. Before-import
failures change no live rule. After-write uncertainty is incomplete until inspected;
retain evidence and do not automatically retry or restore a whole database.

## 3. Inspect status

Normal command: `command-fence status`; machine form: `command-fence status --json`.
These do not request privileges or run a denial test.

Uninstalled example:

```text
Engine: not installed
Rule: unverified
Last application: none
Last execution test: not recorded
```

Reachable-engine, ordinary-user example (illustrative, not a current observation):

```text
Engine: running (Santa 2026.8, Monitor)
Rule: unverified; administrator verification required
Configuration: matches last applied settings
Last execution test: not recorded
```

An explicit `sudo command-fence status --verify --config <absolute-path>` performs fresh
readback, without importing a rule or executing the target vector.

```text
[status] -> [Read available engine/receipt evidence] -> [Text or JSON observations]
                       |
                       +-> [Missing/unreachable] -> [Show that state; no auto-fix]

[Human requests --verify with sudo] -> [Read live rule] -> [Matches/missing/conflicting]
```

Pending config edits are separate from a currently registered prior rule. Engine
health, registration, and historical PoC evidence remain separate rows.

## 4. Remove or recover

The human invokes `sudo command-fence remove --config <absolute-path>`. Locate the
applied receipt independently of whether the current desired config still parses.
Compare it with the live identity/content, then delete only that owned identity. Read
back the absence and check unrelated rules. A separate human-run recovery may restore
the previous owned contents from preserved evidence; removal does not silently restore
an older blocking rule.

```text
[Explicit remove] -> [Receipt + live content agree] -> [Remove owned rule] -> [Verify]
                              |
                              +-> [Foreign/missing evidence/changed content: stop]
```

Remove disables CommandFence's current owned rule. It does not delete configuration,
install an allow rule, clear the database, or alter Santa management. No verified
receipt means no mutation; do not adopt a foreign rule silently.

## 5. Manual PoC

This is a human-run documented procedure, not an extra CLI command. After separate
live-work approval, use requirements §3.4's harmless cases, discard listing output,
record results, and check recovery. Routine project gates use fixtures/fakes and never
trigger live Santa changes or notifications.

The operation inventory implements requirements §§3.1–3.3; this procedure implements
§3.4. Screen/window/popover inventory is not applicable. Shared policy is in
[UX guidelines](../design/ux-guidelines.md); presentation references are in
[design direction](../design/design-direction.md).
