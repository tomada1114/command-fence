# Safe PoC: one harmless execution rule

Status: Proposed human procedure, 2026-10-02. Nothing on this page has been run.
It implements [requirements §3.4](../product/requirements.md#34-safe-poc-acceptance-gate)
and [ADR-0001](adr/0001-signed-santa-and-human-poc.md). No routine gate performs live
setup, rule writes, denied executions, notifications, or desktop interaction.

## Review package before live work

Prepare a private approval package containing:

- The official standard Santa release URL, exact package/build version, published
  digest and signature checks, supported OS/architecture, and existing install state.
  The initial reviewed source baseline is 2026.8; refresh compatibility if it changed.
- Exact human setup steps, including any system-extension approval and Full Disk
  Access grant to Santa. No CommandFence TCC grant, SIP change, notification setting,
  sync setting, mode change, or engine restart is implied.
- Absolute config locator, captured config, exact one-rule import JSON, target signing
  identity, and the recorded ownership basis. Keep actual values outside the checkout.
- Baseline engine mode/management observations, relevant precedence chain, complete
  supported execution export or independently confirmed empty baseline, and private
  recovery evidence. Export scope is limited as ADR-0003 explains.
- The cases below, their expected baseline/after results, and a targeted removal or
  prior-owned-content recovery procedure. The human confirms an exclusive rule-change
  window; no other tool or administrator changes Santa policy during the operation.

Accept the relevant ADRs and obtain explicit approval for that exact live package
before installation, grants, application, or tests. A package download or preview is
not installation approval. If policy is managed, conflicting, or unknown, stop;
do not disable management or delete a foreign rule to continue.

## Preparation and first application

1. The human verifies package digest/signature and completes only the approved setup.
   An existing installation's settings remain intact. A fresh standalone install is
   proposed in Monitor mode; confirm its actual state rather than silently changing it.
2. Confirm packaged `santactl`, engine version and reachability, `/bin/ls` signing
   identity, management state, and policy precedence. Reject an unreviewed version.
3. The human records baseline decisions for the harmless cases, discarding listings.
   Existing policy may affect controls; investigate a differing baseline rather than
   forcing an expected allow by modifying policy.
4. Run the implemented `validate` and `preview`, inspect the captured literal rule,
   and prepare the private recovery record. These commands do not compile CEL in Santa.
5. The human explicitly invokes the checked build's apply with an absolute config
   locator. The payload has exactly one `SIGNINGID` CEL rule for `platform:com.apple.ls`.
   No cleanup flags or broader allow/block rule is used.
6. Fresh privileged inspection compares full target content and unrelated exported
   rules. A successful import alone does not pass this step. Empty export diagnostics,
   warnings, skipped CEL, truncation, and post-write uncertainty stop acceptance.

## Harmless execution matrix

Use the private configured target home as `<home>`. The human initiates every live
execution, including launches through an agent context or direct-process harness.
Redirect directory-listing output to the null device. Record decision metadata only.

| Case ID | Harmless execution | Expected observation attributable to this rule |
|---|---|---|
| `terminal_primary` | `/bin/ls <home>` in the first interactive terminal | Denied before execution. |
| `terminal_second` | Same vector in a second terminal/session | Denied. |
| `agent_shell` | Same vector in an AI-agent/noninteractive context | Denied. |
| `direct_launch` | Direct process launch with exactly the home argument | Denied. |
| `cli_exited` | Same vector after CommandFence has exited | Denied while Santa remains operational. |
| `control_tmp` | `/bin/ls /tmp` | Keeps the observed baseline allow. |
| `control_options` | `/bin/ls -la <home>` | Outside the rule; keeps its baseline. |
| `control_relative` | `/bin/ls .`, from the home | Outside the rule; keeps its baseline. |
| `control_noargs` | No-argument `/bin/ls`, from the home | Outside the rule; keeps its baseline. |
| `control_trailing_slash` | Home with a trailing slash | Outside the rule; keeps its baseline. |
| `control_double_dash` | `/bin/ls -- <home>` | Outside the rule; keeps its baseline. |
| `alternation` | Target and `/tmp`, at least three complete cycles | Correct decisions on every iteration. |
| `after_remove` | Target after removing the newly created owned rule | Returns to its recorded baseline. |

Each summary result is `pass`, `fail`, or `inconclusive` against that case's expected
observation. For `alternation`, every target denial and control allowance must pass;
keep the individual decisions only in the private case record. No missing, failed,
or inconclusive case can produce a passing summary.

Never execute destructive commands, stop the real engine to simulate errors, or change
existing logging/notification settings to make collection easier. A failure to observe
proof is inconclusive. A nonzero child exit, a shell error, a dialog alone, an import
exit, or a rule count is not sufficient proof of pre-execution Santa denial.

Correlate Santa's available decision record to the test timestamp, process/launch
context, binary/signing identity, exact rule, and denial before execution. If existing
settings do not yield adequate evidence, stop and prepare a separately approved
evidence plan. Do not silently enable logging or claim a test passed.

## Targeted removal and interrupted recovery

For a first-created owned rule, the human invokes remove with the same absolute config
locator. Verify owned content immediately before removal, target absence afterward,
and unchanged unrelated exported rules/settings. Run the final harmless baseline case.
Remove leaves configuration and historical evidence, and does not restore an old rule.

For an updated previously owned rule, record its preceding content before application.
If the human wants that prior rule restored, review a separate one-rule import against
the live identity/content and retained journal. That is a distinct recovery operation,
not the behavior of remove. Never import the full baseline or use cleanup flags.

After an incomplete first apply, no successful receipt exists to authorize ordinary
remove. The pending root journal and fresh live comparison let the human review a
targeted recovery. Do not manufacture a receipt, adopt a rule from its comment, retry
automatically, or mutate if live contents changed since the captured payload.

## Evidence and completion

The private case record includes UTC time, OS/engine/build versions, tested launch
context, exact generated/registered rule, receipt generation, decision evidence,
baseline/control results, and restoration outcome. No listing contents or unrelated
process arguments are saved. Retain the latest `poc.json` summary only; its location
and v1 shape follow [the design](../architecture.md#data-and-contracts).

Pass requires all denial/control/alternation cases, clear pre-execution evidence, and
verified targeted restoration. Missing or failed evidence holds the milestone. A
redacted public completion summary may say which contexts passed, without publishing
local paths, exports, receipts, screenshots, or system/account identifiers.

Manual evidence remains historical. Recheck after a human-initiated reboot, engine
restart/upgrade, rule change, or relevant OS change before treating an old result as
applicable. Automated degraded-state tests use fixtures instead of live disruption.

## Sources

Checked 2026-10-02: [release baseline](https://github.com/northpolesec/santa/releases/tag/2026.8),
[installation](https://northpole.dev/deployment/install-package/),
[TCC](https://northpole.dev/deployment/profile-tcc/),
[authorization](https://northpole.dev/features/binary-authorization/),
[limitations](https://northpole.dev/limitations/), and
[rule CLI](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandRule.mm).
