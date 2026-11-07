# Parallel issue and pull request execution review

> **Snapshot: 2026-11-08.** 6 open issues, 4 open PRs. This inventory is not a merge approval or successful real CUBRID test.

## Independent Rust-native tracks

`cubrid-protocol` framing/fuzz cases, blocking `cubrid-client` connection/transaction tests, Tokio cancellation tests, `cubrid-pool` lease tests, MSRV, external Cargo packaging, live CUBRID CI, issue maintenance and historical release notes can all **start concurrently**.

Only final integration of **shared protocol types, public Rust ownership/Send/Sync traits, and physical-session lease contracts** creates a hard implementation coordination requirement. Release governance is required for final ADR acceptance/1.0, not for test research or a disposable CUBRID harness. Cargo publishing cannot precede a registry-valid dependency graph.

## All open issues

| Issue | Priority | Size | Parallel scope and real gate |
| --- | --- | --- | --- |
| [#37](https://github.com/cubrid-lab/cubrid-rs/issues/37) ci: verify real CUBRID and packaged Rust crates before release | high | M | Live DB fixture and external Cargo package tests start now; actual SQL case counts required. |
| [#34](https://github.com/cubrid-lab/cubrid-rs/issues/34) feat: design idiomatic Rust CAS protocol, sync, async and pool boundaries | high | M | Per-crate protocol/client/pool architecture research independent until shared API integration. |
| [#33](https://github.com/cubrid-lab/cubrid-rs/issues/33) ci: verify the declared Rust 1.75 minimum supported toolchain | high | S | MSRV 1.75 validation independent of native driver design; currently only 1.85/stable CI evidence. |
| [#31](https://github.com/cubrid-lab/cubrid-rs/issues/31) ci: skip Rust integration matrix for documentation-only pull requests | medium | M | Docs-only classifier independent but main/release matrix stays mandatory. |
| [#30](https://github.com/cubrid-lab/cubrid-rs/issues/30) docs: define Rust crate API compatibility and pre-1.0 versioning policy | high | M | Versioning and versioned Cargo dependencies need external proof before release; research now. |
| [#18](https://github.com/cubrid-lab/cubrid-rs/issues/18) chore: standardize issue intake and existing five-level sizing | medium | M | Issue template/label triage independent of driver implementation. |

## All open PRs

| PR | Priority | Review / merge coordination |
| --- | --- | --- |
| [#36](https://github.com/cubrid-lab/cubrid-rs/pull/36) docs: establish Rust crate release and stability rules | high | Release policy draft; review requirements on final head, no library release implied. |
| [#35](https://github.com/cubrid-lab/cubrid-rs/pull/35) docs: define Rust-native CAS, sync, async and pooling contracts (Draft) | high | Architecture Draft: shared types/ownership decisions required at integration, not before RED research. |
| [#32](https://github.com/cubrid-lab/cubrid-rs/pull/32) ci: skip runtime and CUBRID matrices on docs-only PRs | medium | CI classifier independent of CAS implementations; no release gate bypass. |
| [#19](https://github.com/cubrid-lab/cubrid-rs/pull/19) chore(triage): require reviewed issue priority and size | medium | Issue labels independent; preserve reviewed classifications. |

## Release safety

MSRV 1.75 is declared but not proven by the observed 1.85/stable CI matrix; verify actual source feature compatibility. Tests selected by `--ignored` must connect to real CUBRID and execute nonzero SQL assertions. Package tests must install versioned crates in a clean consumer without workspace-only path resolution. The separate SeaORM project can research a driver integration concurrently but its release depends on published compatible Rust driver crates. The note-only release PR must never change tags or crate versions.
