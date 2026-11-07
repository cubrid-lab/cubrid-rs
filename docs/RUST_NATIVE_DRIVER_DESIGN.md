# Idiomatic Rust driver architecture for CUBRID

**Status:** Proposed design, 2026-11-07. This is not a verified feature or 1.0 readiness statement. Parent: [#34](https://github.com/cubrid-lab/cubrid-rs/issues/34).

## Principles

- Implement **CUBRID CAS and SQL semantics** as the source of truth. Verify CUBRID wire behavior from actual protocol outcomes and live tests; unrelated drivers cannot dictate Rust APIs or async semantics.
- Use Rust's ownership, borrowing, `Result`, `Send`/`Sync`, Cargo feature and SemVer conventions. Preserve idiomatic Rust interfaces without unnecessary `unsafe` or FFI.
- This workspace owns CAS protocol and driver operations; external ORM integrations own their own SQL generation and API compatibility.

## Workspace boundary

```text
Application / sea-orm-cubrid (external ORM integration)
       |
+----------------+-------------------------+
| cubrid-client  | cubrid-tokio            |  sync / async transport API
+----------------+-------------------------+
                |              |
                +---- cubrid-protocol ------+    pure codec/state-independent framing
                |
          cubrid-pool uses async client         pool leases, capacity and lifecycle
                |
       Broker -> CAS -> CUBRID
```

Existing crates (from `Cargo.toml` and `docs/ARCHITECTURE.md`): `cubrid-protocol`, `cubrid-client`, `cubrid-tokio`, `cubrid-pool`. The diagram is target responsibility, not a claim that all dependencies and I/O lifetimes already satisfy it.

| Crate | Owns | Must not own |
| --- | --- | --- |
| `cubrid-protocol` | Versioned messages, framing, type codecs, parse limits and protocol errors | TCP/Tokio, reconnect policy, ORM SQL |
| `cubrid-client` | Blocking connection, query/statement/cursor/transaction API | Tokio execution or ORM clause generation |
| `cubrid-tokio` | Async socket, cancellation-safe request flow and cursor fetching | Re-encoding protocol concepts independently |
| `cubrid-pool` | Pool limits, lease ownership, idle validation, fair waiting and shutdown | Implicitly sharing a physical connection between active transactions |
| `sea-orm-cubrid` (other repo) | SeaORM/SeaQuery semantics, schema mappings, query and result adapter | CAS protocol or transport |

## Rust-specific design constraints

### Ownership, borrowing and connection lifetimes
- Define whether `Row` is owned or borrows buffers; either case must not allow access to freed or overwritten buffers.
- Tie `Statement`, streaming cursor and `Transaction` to a valid physical connection; a handle from an old CAS session is never used on a replacement session.
- Declare which public types are `Send` and `Sync`; do not force those bounds on non-shareable connection and transaction handles.
- A pool lease may hold an in-flight cursor/transaction; it must not return a connection to another borrower until cleanup and session state validation finish.

### Async/cancellation and error model
- A cancelled future may have already sent SQL. Do not retry it implicitly or leave an orphaned framed response for the next request.
- Define timeouts on connect, broker handshake, read/write, command execution and pool acquisition separately.
- Preserve CUBRID server codes, transport source errors and retry classification with idiomatic typed `Result<T, Error>` and `thiserror`. Avoid panics for malformed network input.
- Guard packet size, nesting and allocations; property tests/fuzzing must reject malformed/truncated frames safely.
- Verify temporal and decimal conversions preserve meaning, including NULL, BIGINT, native collections, nested collections, BLOB/CLOB and unsupported types.

### Configuration and deployment
- The Rust minimum supported version (MSRV), Cargo features, optional Tokio integration, dependency resolver and per-crate versioning need explicit support evidence.
- Keep protocol crate useful without selecting an async runtime. Follow `#![deny(unsafe_code)]` per repo rules.
- Verify the pool connects through a true async client or uses blocking I/O safely; do not guess from a name or diagram.

## Protocol and live test plan

1. Pure codec tests with length/type checks and seeded malformed-wire property/fuzz tests.
2. Sync and async contract parity tests over the same SQL cases; independence of implementation does not require byte-for-byte equality in public APIs.
3. Real CUBRID version matrix with exact successful/failed/skipped cases and independently supported JDBC semantic comparisons (rows, errors, commit/rollback, reconnect).
4. Tokio load, pool saturation, cancellation, CAS reset, statement/cursor lifetime, transaction rollback and partial-result tests.
5. Benchmark only after correctness; do not promise performance parity to another language or database based on mock tests.

## Release compatibility

Cargo's `0.x` compatibility and versioning requirements govern this workspace. Each crate needs its own public API audit even when using a workspace-wide version. A future `cubrid-rs` v1 family and `sea-orm-cubrid` v1 adapter may coordinate **verified major generations**, but they need not share minor/patch versions. Do not create artificial version bumps or declare unsupported dependencies as compatible.

Related: [crate versioning #30](https://github.com/cubrid-lab/cubrid-rs/issues/30), [MSRV CI #33](https://github.com/cubrid-lab/cubrid-rs/issues/33), [SeaORM adapter](https://github.com/cubrid-lab/sea-orm-cubrid).

**No release, implementation change, public API removal, or cargo feature change occurs in this design proposal.**
