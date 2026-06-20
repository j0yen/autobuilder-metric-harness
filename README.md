# autobuilder-metric-harness

The binary autobuilder's loop will poll each iteration for its one unfakeable scalar metric — so that the metric is produced by code built under the verified loop, not by an untested shell script.

> **Status: scaffold, not yet implemented.** This repo is an autobuilder scaffold whose implementation has not been filled in. `src/main.rs` and `src/lib.rs` are generated stubs; the binary prints `stub — implement me` and exits 2. The acceptance tests under `tests/` exist and encode the intended contract, but they currently fail because the behavior they check isn't built yet. Read this README as the spec, not as a description of working software.

## Why it should exist

autobuilder's iterate-and-prove loop advances or reverts each iteration on a single scalar metric. Today that metric comes out of a shell script — and a bash file with no tests is a weak point in a trust model whose whole premise is that nothing self-certifies. The loop's decisions are only as trustworthy as the thing that emits the number it decides on. Moving the metric harness into a binary that autobuilder built and proved closes that gap: the measurement is then held to the same standard as the code being measured.

## Intended contract

The MUST-level acceptance criteria, from `agent/intent-card.json`. Each has a matching integration test under `tests/acceptance_ac<n>.rs` — these define the behavior and are the gate the implementation has to pass.

- **AC1** — given a project path, invoke `<path>/scripts/run-metrics.sh`; exit 2 if it's missing or not executable, with stderr naming the missing script.
- **AC2** — capture the project script's stdout and stderr into `<path>/target/autobuilder/run.log`.
- **AC3** — parse `<path>/target/autobuilder/metrics.json` and validate it against the `autobuilder.metrics.v1` schema embedded as a build-time const; exit 3 with a JSON diagnostic on validation failure.
- **AC4** — emit a normalized `autobuilder.metrics.v1` document to stdout and overwrite `<path>/target/autobuilder/metrics.json` with the same content, including `head_sha` (from `--head-sha`), `iteration` (from `--iteration N`), the scalars, and `ac_passing_count`.
- **AC5** — exit 0 only if the project script exited 0, `audit.blocking_count == 0`, and `ac_results.length == ac_total_count`; otherwise exit 1 — but emit the metrics document either way, so the loop can record the failed iteration.
- **AC6** — change nothing outside `<path>/target/autobuilder/`. Snapshot-hash the project before and after; only `target/autobuilder/*` may differ.

The CLI surface the tests drive: a positional project path, `--head-sha <sha>`, and `--iteration <n>`.

## Audience

The intended callers are autobuilder's Stage 3 loop runner and the authors of per-project `run-metrics.sh` scripts. No human runs this binary directly except to debug it.

## Build

```sh
cargo build --release
```

The stub compiles today; `cargo test` does not pass yet, by design — the acceptance tests stay red until the implementation lands. Rust 1.85, edition 2024. `install.sh` is present and will build and `cargo install` the binary, but installing it now gets you only the stub.

## Provenance

Scaffolded by the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline — PRD intake → intent-card → scaffold. The iterate-and-prove step that fills in `src/` to make the acceptance tests pass has not been run. Originally a subdirectory of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this is a fresh-init standalone snapshot.

## License

MIT OR Apache-2.0 — see [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE).
