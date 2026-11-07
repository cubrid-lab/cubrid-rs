# ADR 0002: CUBRID correctness and failure-evidence boundaries

- **Status:** Proposed
- **Date:** 2026-11-07
- **Scope:** cubrid-rs correctness obligations and review evidence

## Context

The CUBRID Python driver and its SQLAlchemy dialect revealed that apparently successful queries can hide transaction-scope mistakes, accidental retries after uncertain execution, stale broker handles, precision loss and unverified ORM-generated SQL. The purpose of learning from them is to **reuse verified invariants and test design**, never Python API signatures or implementation code. The current repository has its own runtime, public API and released compatibility obligations.

## Decision

Use a transport-independent bounded cubrid-protocol codec, separate blocking and Tokio clients, and explicit pool lease ownership. Connection, statement, cursor and transaction lifetimes must be safe under Rust ownership and cancellation. A dropped async future after SQL send must never cause an implicit replay or leak an unread response into a subsequent command. Stale session handles are invalid, malformed packets return errors rather than panic, and unsupported numeric/collection data must not be silently coerced. Verify with property tests, live CUBRID sync/async/pool regression tests, and independent JDBC outcome comparisons.

Use source-linked test cases as engineering references, then implement targeted **native-language** regression tests. Identify whether a test actually talked to CUBRID, which server/broker and client/ORM versions were used, what SQL/outcome was asserted, and whether any required case was skipped. The fact that a Python project passes its tests does **not** certify this package.

## Alternatives considered

- **Rejected:** Copying Python cursor/exception APIs into Rust; assuming Tokio mutex serialization equals transaction isolation; returning a potentially corrupted pooled connection; treating Cargo compilation as a real broker test.
- **Rejected:** A generic cross-language implementation copied verbatim from Python.
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
