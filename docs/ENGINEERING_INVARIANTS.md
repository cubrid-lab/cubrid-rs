# Engineering invariants for cubrid-rs

> **Status:** Proposed repository-specific engineering checklist, not evidence that existing tests pass.

cubrid-rs owns the Rust CAS protocol, sync/async clients, and pooling crates. Correctness requirements follow Cargo workspace crates, Rust ownership and lifetimes, Tokio cancellation, MSRV, crate packaging, and real CUBRID tests. Use this repository's implementation, public API, relevant live-CUBRID evidence and official CUBRID/JDBC results as appropriate; never assume that another language's driver or ORM test result establishes behavior here.

## Evidence and adoption method

The Python projects' reusable contribution is **how correctness was established**, not a shared API: model physical CAS lifecycle, test uncertain SQL execution without replay, invalidate stale handles, verify transaction rollback, preserve type fidelity, compare semantic outcomes with an independent official driver/JDBC, refuse vacuous live-test passes, and publish only after final candidate checks.

Sources: [CAS reconnect/session tests](https://github.com/cubrid-lab/pycubrid/blob/main/tests/test_cas_session_persistence.py), [cursor handle ownership](https://github.com/cubrid-lab/pycubrid/blob/main/tests/test_autocommit_handle_ownership.py), [live state machine](https://github.com/cubrid-lab/pycubrid/blob/main/tests/test_connection_state_machine.py), [official driver differential](https://github.com/cubrid-lab/pycubrid/blob/main/tests/test_official_differential.py), [SQLAlchemy CUBRID architectural boundaries](https://github.com/cubrid-lab/sqlalchemy-cubrid/blob/main/docs/ARCHITECTURE.md), [DDL rollback tests](https://github.com/cubrid-lab/sqlalchemy-cubrid/blob/main/test/test_transactional_ddl.py), [integration gate tests](https://github.com/cubrid-lab/sqlalchemy-cubrid/blob/main/test/test_integration_gate.py), [release note style](https://github.com/cubrid-lab/sqlalchemy-cubrid/blob/main/docs/RELEASE_NOTES_STYLE.md).

Do **not** infer CUBRID support from syntactic similarity with MySQL. In particular, CUBRID DDL rollback is exercised by real CUBRID behavior verified for the claimed database version; do not state that CUBRID always auto-commits DDL without version-and-transaction-specific real evidence.

## Rust-specific protocol, ownership and async safety

1. **Frame and buffer correctness:** `cubrid-protocol` must reject malformed, truncated, negative or unreasonable-size frames with typed errors rather than panic, `unsafe`, lossy default values or unbounded allocations. Test malformed packets, charsets, collection metadata and error codes with property/fuzz fixtures.
2. **CAS session generations:** `CAS_INFO=OUT_TRAN` means the transaction ended, not that the socket died. Verify CHECK_CAS and broker redirect before replacing sessions. A stale statement, cursor, result or LOB handle cannot operate on another physical session, even if integer handle IDs are reused.
3. **Cancellation/timeout uncertainty:** dropping or cancelling a Tokio future after request send must not transparently replay SQL or leak an unread framed response into a later command. Dispose/retire unsafe connections. Sync and Tokio clients share the CUBRID protocol contract but need different runtime-specific tests and ownership.
4. **Transaction and pool ownership:** tie transaction, cursor and pool lease lifetimes to a valid connection; no release of a connection to another task while its transaction or cursor is active. Report failed begin, commit, rollback and close with their error causes; never silently commit after a failed rollback.
5. **Value fidelity:** use idiomatic Rust typed `Result` and `Option` for NULL; preserve BIGINT, decimal precision, binary and date/time meaning. Collections (SET/MULTISET/SEQUENCE) have different ordering/multiplicity rules; verify true round trips, no guessed conversions.
6. **Independent confirmation:** `cargo test --workspace --lib`, live `--ignored` suites, `cargo fmt`, Clippy, crate packaging and MSRV are separate checks. Compare selected SQL/transaction scenarios to JDBC on the *same CUBRID version*; do not copy a Python driver implementation.

## Per-crate review matrix

| Crate | Mandatory failure/regression themes |
| --- | --- |
| `cubrid-protocol` | Incomplete frames, invalid length/type tags, encoding and allocation limits |
| `cubrid-client` (Rust) | Request may-have-executed state, sync transactions, stale cursors and LOB handles |
| `cubrid-tokio` | Cancellation at await points, concurrent use, cleanup, timeout and connection retirement |
| `cubrid-pool` | Lease cancellation, saturated pool, idle validation, aborted transaction, safe return to pool |

## PR and release checklist

- [ ] Specify `Send`/`Sync`, ownership/lifetime and public trait/feature changes explicitly.
- [ ] Include an adversarial state/error case and a normal case for the changed behavior, with mock protocol and live DB evidence kept separate.
- [ ] Report which supported CUBRID versions actually executed tests; assert that the release integration lane ran nonzero real SQL cases.
- [ ] Avoid overclaims from workspace-shared `0.1.0`; follow Cargo per-crate SemVer and ensure released inter-crate dependencies resolve without local-only overrides.
- [ ] Keep README/architecture/API docs and release notes aligned with actual tested behavior; do not infer readiness from badge text.

Tracked: [Rust design #34](https://github.com/cubrid-lab/cubrid-rs/issues/34), [release live DB #37](https://github.com/cubrid-lab/cubrid-rs/issues/37), [Cargo version policy #30](https://github.com/cubrid-lab/cubrid-rs/issues/30), [MSRV #33](https://github.com/cubrid-lab/cubrid-rs/issues/33). [Release policy](RELEASE_POLICY.md) precedes the [architecture draft PR #35](https://github.com/cubrid-lab/cubrid-rs/pull/35).
