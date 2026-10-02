# CommandFence

[![CI](https://github.com/tomada1114/command-fence/actions/workflows/ci.yml/badge.svg)](https://github.com/tomada1114/command-fence/actions/workflows/ci.yml)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/tomada1114/command-fence/badge)](https://scorecard.dev/viewer/?uri=github.com/tomada1114/command-fence)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

CommandFence is a personal macOS CLI for preventing selected AI-agent command mistakes.
The planned first rule rejects `/bin/ls` when its sole argument is the configured
owner's absolute home path. The official signed Santa engine performs execution
control; CommandFence validates configuration, previews and manually applies/removes
its owned rule, and reports the available evidence.

**Implementation status:** initialized scaffold and agreed product documents. The five
CommandFence commands and live Santa blocking are not implemented or verified yet.
The inherited counter, optional model adapter, and TUI remain as template examples to
replace during implementation. They do not add product requirements. There is no
first-version TUI, resident CommandFence process, or distribution pipeline.

Start with the [requirements](docs/product/requirements.md),
[CLI flows](docs/product/ux-flows.md),
[design](docs/architecture.md#commandfence-design-proposal),
[accepted ADRs](docs/architecture/README.md#decisions), and
[roadmap](docs/architecture/roadmap.md). The
[safe PoC procedure](docs/architecture/safe-poc.md) requires separate live-work approval.

## Development quickstart

These commands exercise the inherited scaffold while CommandFence is being built.

Prerequisites: macOS on Apple Silicon with the Xcode Command Line Tools
(`xcode-select --install`), or Linux with a C toolchain for the linker (`build-essential`
on Debian and Ubuntu); [rustup](https://rustup.rs/), [mise](https://mise.jdx.dev/), and
[Just](https://just.systems/) (`brew install mise just` on a Mac).

```bash
git clone https://github.com/tomada1114/command-fence.git
cd command-fence
mise trust     # approve mise.toml once (mise asks before using an untrusted config)
just install   # pinned tools via mise and lefthook's git hook
just check     # everything the machine can run without a human; takes over no terminal
cargo run --locked -p command-fence -- counter show   # inherited sample
```

rustup installs the Rust toolchain `rust-toolchain.toml` names the first time `cargo`
runs (`RUSTUP_AUTO_INSTALL`, on by default:
<https://rust-lang.github.io/rustup/environment-variables.html>, checked 2026-09-30).
`just install` needs no `sudo` and opens no installer; a missing Command Line Tools
install is reported with the command to run. `cargo run --locked -p command-fence -- tui` opens
the full-screen view in the terminal you run it from; `q` quits.

The inherited optional `openrouter` Cargo feature is a template example, outside
CommandFence scope. See
[OpenRouter](docs/openrouter.md) for the key setup, `command-fence llm ask`, and reuse from a
CLI or TUI action. The default build needs no API key.

## Design Philosophy

Every choice below has a reason. If you disagree with one, you know what to change and
why it was there in the first place.

### Why is CommandFence a CLI over Santa?

The agreed product keeps configuration and explicit operations in a small Rust CLI
and delegates execution authorization to the official signed engine. This avoids a
new resident CommandFence service. The five accepted
[ADRs](docs/architecture/README.md#decisions) record the implementation choices and
their limits; the [human PoC](docs/architecture/safe-poc.md) must establish runtime
feasibility. The remaining philosophy describes the inherited development foundation.

### Why a Cargo workspace with the logic split into crates?

A single crate cannot keep OS code out of the logic: nothing would stop a rule from
reading a file or the clock, and its tests would link the real adapters. So the
repository root is a virtual workspace: `crates/command-fence-core` holds the rules and state,
`crates/command-fence-platform` the OS adapters, `crates/command-fence-test-support` the fakes, and
`crates/command-fence` the `command-fence` binary, the only place the adapters are wired to core. No
crate is named `core`, which would collide with Rust's built-in `core` library. A
repository per layer was rejected: it costs a release process per layer for a personal
tool.
### Why ports and adapters, with synchronous ports?

Core declares each thing it needs from outside the process — storage, time — as a
`Send + Sync` trait. `command-fence-platform` implements it for real, `command-fence-test-support` as a
fake, and the binary picks the real one. Ports are plain synchronous methods, so a
reader new to Rust meets no async in core. Errors are typed variants the binary turns
into words and an exit code, never sentences from core. One contract function per port
runs against both the fake and the real adapter, so the fake cannot drift from the real
thing without a test failing.
### Why are the architecture boundaries enforced three times?

A rule that lives only in prose drifts. Core's `Cargo.toml` names no OS crate, so core
cannot compile a call into one. A harness check reads `cargo metadata` and fails if
core's dependency closure ever gains a macOS binding crate or
`command-fence-platform`, and `cargo deny`'s `wrappers` rule allows `command-fence-platform` as a direct
dependency of the binary only. clippy, configured in `crates/command-fence-core/clippy.toml`,
bans printing and the standard streams, `std::fs`'s files and functions, `Path`'s
file-system queries, `std::net`'s sockets and address lookups, clock reads
(`SystemTime::now`, `Instant::now`, `elapsed`), `std::env`'s argument, variable, and
directory functions, `std::process::Command`, `exit`, and `abort`, and unscoped threads
and `thread::sleep` in core, so I/O, time, and environment arrive only through ports.
### Why is the tool one binary, in its own crate?

One `cargo install --locked --path crates/command-fence` (`just install-cli`) yields the whole
tool, and its subcommands are the only entry points, so there is one composition root to
wire adapters in. Keeping the binary out of core and platform leaves those two as
libraries a test links without a `main`. The binary only translates: arguments to calls,
a view to stdout, a typed error to wording on stderr and an exit code, with every
sentence in `crates/command-fence/src/wording.rs`.

### Why is the sample app a counter?

Because it is small enough to delete and still exercises every frame the architecture
claims: a port with a real adapter, a fake, and a contract suite (`CounterStore`,
writing `counter.json` atomically under a lock); injected time (`Clock`); subcommands
and a full-screen view over the same service, the screen's state and keys in core;
logging; typed errors with their wording in one module; and a help line naming every
key. Every part of it is an illustration to replace.
### Why tracing to daily files?

`tracing` gives one logging API across every crate; only the binary installs a
subscriber, which writes `command-fence.YYYY-MM-DD.log` to `~/Library/Logs/com.tomada.commandfence/`
on macOS and to `$XDG_STATE_HOME/command-fence/logs/` on Linux, rotated daily and keeping 14
files. The writer is synchronous: the volume is low, and a background writer can drop
its last lines at exit. While `command-fence tui` owns the terminal it logs to the file only, so
no line lands in the frame. `just logs` prints the newest file's tail and exits.
### Why clap and ratatui?

clap's derive API declares a subcommand as a type and gives `--help`, `--version`, and
usage errors (exit 2) for free. ratatui, over its crossterm backend, draws the
full-screen view immediate-mode: one `draw` over a state, so the loop the binary runs
stays a thin shell around core's screen and key table, and the drawing is tested by
rendering into an in-memory backend. Styling starts from the terminal's own colors, so
a tool reads in light and dark terminals alike. Another framework, an async runtime, or
a theme of the tool's own is a decision for an app to make and record, not a default.
### Why is every tool pinned in exactly one place?

A version written twice drifts. Rust is pinned in `rust-toolchain.toml`, every other CLI
in `mise.toml`, preferring prebuilt binaries to the `cargo:` backend, which compiles from
source. Nothing is `latest`; bumps arrive as Renovate or Dependabot pull requests after
a 7-day release age.

### Why Just?

One command, `just check`, runs locally what CI runs. Just is a task runner rather than
a build system, and each recipe is a thin call into cargo (`cargo xtask` included) or a
pinned tool, so every recipe also works without it. `cargo xtask` cannot naturally drive
mise.

### Why lefthook, and why does the pre-commit hook only check?

lefthook gives per-language staged-file globs and parallel jobs from one YAML file. The
hook stays check-only and fast — rustfmt and typos on the staged files, the skills
mirror, plus a guard against secret-shaped paths and credential-shaped content — with no
compile, clippy, or tests: those belong to `just check` and CI. `just install` fails
when the hook is missing. JSON, YAML, and Markdown have no formatter: Prettier's check
over them left with the Node toolchain, to keep the toolchain small.

### Why repository automation in Rust?

`cargo xtask <task>` needs no runtime beyond the Rust toolchain the app already pins.
Its tasks read YAML, TOML, and JSON with real parsers, each is a function of a faked
context with its own coverage floor, and a failing task prints a stable
`ERR_<STAGE>_<WHAT>` code, then `Expected:`, `Actual:`, and `Next:` lines. A skill may
bundle Python or shell scripts of its own, which `just test-scripts` tests.

### Why a coverage floor on the core only?

The floor (80% of lines and 80% of functions, measured by `cargo llvm-cov`) sits where
the decisions are. The platform crate and the binary translate and decide nothing, so a numeric gate there would only invite tests of glue. The `xtask` crate
carries its own floors, so one tree cannot subsidize another. clippy runs
at `pedantic` with warnings as errors, `unsafe_code` is forbidden in every crate, and
weakening any gate needs a human's sign-off.

### Why does the harness check itself?

Documentation that lists what is enforced goes stale, so the claims are checks instead.
`just check-harness` runs `cargo xtask check-harness`, one module per claim under
`xtask/src/check_harness/`: every `just` recipe a document names exists;
every workflow pins actions by SHA with a version comment, sets timeouts and job-level
permissions, and never uses `pull_request_target`; the dependency cooldowns agree; each
required status check names a real job; the boundary lists agree; every label an issue
form or workflow applies is declared; and more — each check's header says what it
asserts.

### Why does no check run the real terminal?

A test that drives a real terminal would take over the one the developer is working in,
and a CI runner has no terminal to give it. So each layer is tested where it lives —
core with fakes, adapters with contract suites, the command line by running the built
binary against a temporary `HOME`, the full-screen view by drawing into ratatui's
`TestBackend` and feeding keys as values. The gap that leaves, the real terminal loop
(raw mode, the alternate screen, restoring the terminal on every way out), is named and
kept thin, and a pull request that changes it carries a human's run of `command-fence tui`.
### Why does most CI run on Ubuntu?

macOS runners queue longer. Core, the Linux-buildable crates, the platform tests on
Linux, the scripts, and the repository lint all run on Ubuntu; one macOS job runs
workspace clippy and the platform tests against macOS. Every job pins its actions by
SHA, sets `persist-credentials: false`, least-privilege permissions, and a timeout, and
every cargo command that resolves the lockfile is `--locked`.
### Why this much supply-chain control, and why scope advisories to the built targets?

A tool that runs on your machine with your permissions deserves the same scrutiny as a
server: CodeQL, OSV-Scanner, OpenSSF Scorecard, zizmor, Dependency Review with a license
allow-list, `cargo deny`, a 7-day cooldown on every automated bump, a weekly
full-history gitleaks scan, and the branch ruleset as code. `Cargo.lock` lists every
platform's dependencies, including Windows-only crates this tool never builds; so
`cargo deny` evaluates the `aarch64-apple-darwin` and `x86_64-unknown-linux-gnu`
graphs — defining what is built rather than ignoring anything — and an OSV exception is
allowed only for a crate absent from those graphs, with a reason and an expiry.
### Why no release pipeline?

A tool for its owner is built and installed from its own checkout with
`cargo install --locked --path crates/command-fence` (`just install-cli`): no secret, no tag,
and no CI involved. `CHANGELOG.md` and the workspace version still record what changed.
A release workflow, signed or prebuilt binaries, a package-manager tap, or crates.io
publishing is a decision an app records in an ADR when a tool needs to reach other
people.

### Why an optional OpenRouter adapter?

Some apps need model calls; the counter does not. The `openrouter` feature keeps the
HTTP and credential-file dependencies out of default builds, and an explicit call is
the only place that looks up a key or contacts a provider. Core's synchronous
`TextGenerator` port, `GenerationService`, and `GenerationView` work for either front
end and are tested with a fake; the platform adapter translates blocking HTTPS into
those values. `ureq` supplies HTTPS without an async runtime, and `dotenvy` parses a
local credential file without changing the process environment.

The composition root selects `openai/gpt-6-luna`, Max reasoning effort, and a bounded
token budget in source. `OPENROUTER_KEY` comes from the environment, falling back to
`.env.local` in the working directory. There are no implicit calls, automatic retries,
or conversation history. [OpenRouter](docs/openrouter.md) describes the bounds, error
behavior, and how a TUI can schedule a call without blocking its event loop.

### Why AGENTS.md and skills, but no committed agent permissions?

The repository is developed with coding agents, often unattended. `AGENTS.md` gives them
the architecture, the gates, and the hard prohibitions; path-scoped rules under
`.claude/rules/` and skills authored under `.agents/skills/` (mirrored by
`just agents-sync`) carry the procedures. Which commands an agent runs without a prompt
is each person's choice, so no `.claude/settings.json` is committed: permissions and the
format-on-edit hook live in a user-level or gitignored local settings file, and the
gates and `AGENTS.md`, not a permission list, are what bind every author.

### Why an ADR tree that ships empty?

The decisions in this section are the shared foundation's. Each app decides different
things — where it keeps state, which platforms it targets, which permissions it asks
for, whether it ever ships releases — and records each as an Architecture Decision Record
under [docs/architecture/](docs/architecture/README.md), whose index ships empty.
A replaced decision gets a new ADR rather than a rewrite, so the reasoning that held at
the time stays readable. [docs/architecture/roadmap.md](docs/architecture/roadmap.md)
ships as a skeleton for the app's direction.

### Why may nothing a check runs take over your machine or terminal?

The checks run on the machine you are working on, often while an agent iterates in a
terminal beside yours. So nothing routine — `just check` and every recipe in it, the
pre-commit hook, an agent's own verification — may show a window, take focus, or raise
a permission, Keychain, or Gatekeeper prompt, and none may take over a terminal: no
check runs `command-fence tui`, enables raw mode, enters the alternate screen, or needs a TTY.
An agent's evidence is the tests, a subcommand run against a scratch `HOME` (on Linux,
with `XDG_DATA_HOME` and `XDG_STATE_HOME` unset too), and `just logs`; the full-screen
view is yours to run.

## Development

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full workflow.

```bash
just install      # once per clone
just check        # the full local gate; takes over no terminal
just test-fast increment   # one core test or a group of them, while iterating
just logs         # the newest app log's last lines
just install-cli  # install the command-fence binary into ~/.cargo/bin (a human's step)
```

`just --list` shows every recipe. The tool's data lives in
`~/Library/Application Support/com.tomada.commandfence/` and its logs in
`~/Library/Logs/com.tomada.commandfence/` on macOS; on Linux, in `$XDG_DATA_HOME/command-fence/`
(default `~/.local/share/command-fence/`) and `$XDG_STATE_HOME/command-fence/logs/` (default
`~/.local/state/command-fence/logs/`).

## Documentation

- [Getting Started](docs/getting-started.md)
- [Architecture](docs/architecture.md)
- [OpenRouter](docs/openrouter.md)
- [Architecture Decisions](docs/architecture/README.md) and the
  [Roadmap](docs/architecture/roadmap.md)
- [Contributing](CONTRIBUTING.md), [Security Policy](SECURITY.md),
  [Code of Conduct](CODE_OF_CONDUCT.md), [Changelog](CHANGELOG.md)

## License

[MIT](LICENSE)
