# CommandFence — CLI presentation direction

Status: Settled 2026-10-02 under the accepted CLI-only scope and the user's direction to
choose the simplest recommendations. This is a CLI presentation lock, not a GUI theme.
Related: [requirements](../product/requirements.md), [flows](../product/ux-flows.md),
[UX guidelines](ux-guidelines.md).

## Brief and reference lock

Design a short command-line workflow for this Mac's owner and AI-agent scripts. The
core task is edit → inspect → explicitly apply → inspect state. The main confidence
risk is mistaking a saved configuration or imported record for proven execution denial.

Primary reference: rust-template's existing plain CLI convention. Preserve clap help,
short complete output lines, stdout results/stderr diagnostics, the wording module,
and the existing exit-code convention. Borrow Santa's engine/rule vocabulary and Git's
separation between human status and a deliberately stable machine form. Keep those
source roles distinct: Santa defines enforcement semantics; it does not justify
copying a broad engine-management UI into CommandFence.

## References inspected

| Source | Adopt | Omit |
|---|---|---|
| rust-template CLI skill/current sample | Plain output, standard help, 0/1/2 exits, one place for wording. | Sample counter and TUI product behavior. |
| Santa 2026.8 rule/status commands and official authorization docs | Explicit management boundary, engine version/mode, registered rule evidence, manual apply. | Fleet/sync/metrics/notification management surfaces. |
| Git status documentation | Deliberate stable machine output separate from human presentation. | Interactive diff/staging UI, Git-specific status vocabulary. |

All inspected 2026-10-02:
[template CLI convention](https://github.com/tomada1114/rust-template/blob/d67059f8994a6ed55951b9e094f1619b1e75dec5/.agents/skills/designing-clis/SKILL.md),
[Santa rule source](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandRule.mm),
[Santa status source](https://github.com/northpolesec/santa/blob/2026.8/Source/santactl/Commands/SNTCommandStatus.mm),
[authorization docs](https://northpole.dev/features/binary-authorization/),
[Git status machine-format docs](https://git-scm.com/docs/git-status#_porcelain_format_version_1).

Research was grounded in these CLI sources and the existing repository rather than
visual styles for marketing pages. No Refero visual screens/styles or generated visual
assets were needed for this plain command output. The repository has no designing-ui
skill or GUI token target.

## Presentation roles

| Role | Binding value |
|---|---|
| Font and type size | Terminal-owned; no application font selection. |
| Foreground/background, light/dark | Terminal-owned in both modes; no ANSI colors. |
| Status emphasis | Literal words and separate lines, never color-only. |
| Layout | Line-oriented text; no terminal-width-dependent interactive rendering. |
| Spacing | One observation per line; no banner or decorative framing. |
| Machine output | Versioned status JSON; preview is a Santa JSON rule document. |
| Motion/cursor behavior | None. |
| Icons/imagery | None. |
| Radius/shadows/theme tokens | Not applicable to this CLI. |

A compact state report keeps engine, rule, and last execution test on separate rows.
This distinction is the product's recognizable detail and remains present in JSON.

## Contrast applicability

Contrast pass: **not applicable**. CommandFence selects no foreground/background pairs
and renders no GUI or TUI theme. No palette was measured and no WCAG contrast PASS is
claimed. The user's terminal supplies colors. If later scope adds application-selected
colors or UI, conduct a real reference/contrast stage for that scope.

## Decision ledger

| Decision | Source | Reason |
|---|---|---|
| Existing plain CLI presentation | Template and user's simplicity instruction. | Avoid a new UI dependency and extra workflows. |
| Five direct commands | Accepted product behavior and UX inventory. | Each supports preview, operation, inspection, or safe removal. |
| Separate engine/rule/PoC evidence | Santa semantics and accepted requirements. | A successful import is different evidence from observed blocking. |
| Stable JSON alongside human text | User choice and Git machine-format principle. | Agents should not parse prose. |
| No own palette/theme/motion | Existing CLI convention and CLI-only scope. | Keeps all first-version output pipe-safe. |

Stage 3 is complete for this CLI scope. There are no token-application or branding issues
to add to the MVP backlog.
