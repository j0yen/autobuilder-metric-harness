# autobuilder-metric-harness

> Bootstrap a load-bearing tool autobuilder needs: every iteration's unfakeable scalar must be emitted by a binary that was itself built under autobuilder's verified loop.

## Install

### One-liner

```sh
curl -fsSL https://raw.githubusercontent.com/j0yen/autobuilder-metric-harness/main/install.sh | bash
```

### Manual

```sh
git clone --depth 1 https://github.com/j0yen/autobuilder-metric-harness.git
cd autobuilder-metric-harness
./install.sh
```

Installs the `autobuilder-metric-harness` binary via `cargo install --path . --locked`. Requires `cargo` / `rustc 1.85+` and `git`. Built binary lands in `~/.cargo/bin/`.

## Why

Bootstrap a load-bearing tool autobuilder needs: every iteration's unfakeable scalar must be emitted by a binary that was itself built under autobuilder's verified loop. The current shell-script fallback is a crack in the trust model — until the metric harness is owned by autobuilder, the loop's advance/revert decisions are only as trustworthy as a bash file with no tests.

## Build

```sh
cargo build --release
```

Produces `target/release/autobuilder-metric-harness`. Symlink into `~/.local/bin/` if you want it on `$PATH`.

## Usage

```sh
autobuilder-metric-harness --help
```

## Audience

The autobuilder skill (specifically the Stage 3 loop runner) and per-project run-metrics.sh authors. No human invokes this binary directly except for debugging.

## Acceptance criteria

This project was scaffolded from a PRD via the `autobuilder` pipeline. The MUST-level acceptance criteria are:

- **AC1**: Given a project path, invoke <path>/scripts/run-metrics.sh. Return non-zero (exit 2) if missing or not executable, with stderr identifying the missing script.
- **AC2**: Capture project script's stdout AND stderr into <path>/target/autobuilder/run.log.
- **AC3**: Parse <path>/target/autobuilder/metrics.json, validate against the autobuilder.metrics.v1 schema embedded as a build-time const. Return exit 3 with a JSON diagnostic on validation failure.
- **AC4**: Emit a normalized autobuilder.metrics.v1 document to stdout AND overwrite <path>/target/autobuilder/metrics.json with the same content. Include head_sha (from --head-sha flag), iteration (from --iteration N), scalars, ac_passing_count, a...
- **AC5**: Exit 0 only if project script exited 0 AND audit.blocking_count == 0 AND ac_results.length == ac_total_count. Otherwise exit 1 — but emit the metrics document either way so the loop can record the failed iteration.
- **AC6**: Operate without modifying anything outside <path>/target/autobuilder/. Snapshot-hash the project before and after; only target/autobuilder/* paths may have changed.

Each AC has a matching integration test under `tests/acceptance_ac<n>.rs`.

## Provenance

Built via the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline (PRD intake -> intent-card -> scaffold -> iterate-and-prove). Originally consolidated as a subdir of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this standalone repo is a fresh-init snapshot for easier consumption and distribution.

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.
