# ADR 0002: CUBRID correctness and failure-evidence boundaries

- **Status:** Proposed
- **Date:** 2026-11-07
- **Scope:** cubrid-rs correctness obligations and review evidence

## Context

cubrid-rs owns the Rust CAS protocol, sync/async clients, and pooling crates. This project's correctness decisions must cover Cargo workspace crates, Rust ownership and lifetimes, Tokio cancellation, MSRV, crate packaging, and real CUBRID tests, including negative and failure-path behavior. Passing offline tests alone is not proof of database compatibility. The accepted public API, local implementation, and actual CUBRID server behavior are the basis for requirements and test evidence.

## Decision

Use a transport-independent bounded cubrid-protocol codec, separate blocking and Tokio clients, and explicit pool lease ownership. Connection, statement, cursor and transaction lifetimes must be safe under Rust ownership and cancellation. A dropped async future after SQL send must never cause an implicit replay or leak an unread response into a subsequent command. Stale session handles are invalid, malformed packets return errors rather than panic, and unsupported numeric/collection data must not be silently coerced. Verify with property tests, live CUBRID sync/async/pool regression tests, and independent JDBC outcome comparisons.

Create targeted tests for this project's observable contracts. Record whether tests actually communicated with CUBRID, the server/broker and dependency versions, asserted SQL outcomes, and skipped cases. External test results never certify this package.

## Alternatives considered

- **Rejected:** Copying Python cursor/exception APIs into Rust; assuming Tokio mutex serialization equals transaction isolation; returning a potentially corrupted pooled connection; treating Cargo compilation as a real broker test.
- **Rejected:** Treating another package's API, implementation, or test results as this project's contract.
- **Rejected:** Marking an unsupported/unverified feature as absent simply because one server or connector failed in a particular test.

## Consequences and trade-offs

The design may reject or retire connections under uncertain execution, and requires concurrency and lifecycle tests; this reduces silent corruption or use-after-session errors. Ordinary documentation-only PRs need not run an expensive database matrix; **the final release candidate still must run all required real-CUBRID checks** for its promised compatibility.

## Verification / acceptance criteria

- [ ] Normal and failure-path tests cover the actual decision's affected behavior.
- [ ] No silent statement replay, fabricated database values, unknown-success transaction or stale-handle reuse for the supported public API.
- [ ] Real DB tests use an isolated ready-checked CUBRID instance, assert actual stored/returned values and report nonzero executed-case counts; skip/missing is not success.
- [ ] Independent JDBC (or verified official driver) comparisons evaluate meaning and type, not matching source implementation.
- [ ] Public docs, type declarations and compatibility claims are reconciled with observed behavior; unsupported operations fail clearly.
- [ ] Maintainer accepts this ADR and separately reviews release/code changes; neither this record nor a checklist fixes existing defects.

## References

[engineering checks](../ENGINEERING_INVARIANTS.md), [Rust design #34](https://github.com/cubrid-lab/cubrid-rs/issues/34), [release DB tests #37](https://github.com/cubrid-lab/cubrid-rs/issues/37)

**No claim of implementation completion is made.**
