# Development setup — Rust CUBRID driver workspace

> **Status:** Based on workspace `Cargo.toml`, crate manifests, Makefile and CI. This file does **not** certify that any command has been executed successfully. [ADR 0005](adr/0005-development-setup.md) is Proposed.

## Native Rust workspace

The workspace contains four crates:
`cubrid-protocol` (bounded CAS framing/codecs), `cubrid-client` (sync), `cubrid-tokio` (async), `cubrid-pool` (pool). Their manifests inherit `workspace.package.version = 0.1.0`, edition 2021 and **declared MSRV 1.75**. Current CI tests **1.85 and stable**, not the declared minimum; do not claim 1.75 is verified.

Use rustup with a toolchain supporting Cargo, rustfmt and clippy, then:

```sh
git clone https://github.com/cubrid-lab/cubrid-rs.git
cd cubrid-rs
rustc --version
cargo --version
cargo fmt --all -- --check
cargo clippy --workspace -- -D warnings
cargo test --workspace --lib
cargo build --workspace
```

The library workspace does not commit a root `Cargo.lock` on inspected main; Cargo may resolve dependency versions on each checkout. Do not add a lockfile or pin toolchain automatically before the separate decision about MSRV, crate publication and reproducible library testing. CI tooling and a source build are **not** a crates.io publish/install guarantee.

## Live CUBRID setup

The repository contains `docker-compose.yml` with default CUBRID 11.2, `CUBRID_DB=testdb` and host port 33000. Integration helpers use **both** `CUBRID_TEST_URL` (clients) and `CUBRID_DSN` (pool). Run only with an isolated disposable broker/database:

```sh
docker compose up -d
export CUBRID_TEST_URL='cubrid://dba:@localhost:33000/testdb'
export CUBRID_DSN='cubrid://dba:@localhost:33000/testdb'
cargo test --workspace -- --ignored
docker compose down
```

The ignored-test selector alone is insufficient evidence; verify a nonzero count of actual SQL-executing tests and capture pass/fail/skip on named CUBRID versions. CI currently provisions 11.0 and 11.2.

## Cargo publication feasibility

The internal crate dependencies currently use `path = "../..."` without version constraints. Before claiming crates.io publishing or a stable 1.x, design and verify per-crate versioned dependencies, publishing order, external Cargo consumer, Rust MSRV and feature compatibility. This is tracked in [#30](https://github.com/cubrid-lab/cubrid-rs/issues/30) and [#37](https://github.com/cubrid-lab/cubrid-rs/issues/37).

## Setup exit criteria

- [ ] One compatible toolchain is installed with rustfmt/clippy, and normal unit checks genuinely pass.
- [ ] Minimum Rust version is **actually tested** before support is claimed.
- [ ] Real CUBRID client/Tokio/pool tests run with nonzero broker SQL assertions.
- [ ] Path/version dependencies and publish order are proven with clean external consumer builds before 1.0.
- [ ] [Architecture draft PR #35](https://github.com/cubrid-lab/cubrid-rs/pull/35) and [release policy](RELEASE_POLICY.md) remain prerequisites.
