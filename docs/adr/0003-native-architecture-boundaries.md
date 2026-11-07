# ADR 0003: Native CUBRID architecture boundaries

- **Status:** Proposed
- **Date:** 2026-11-07
- **Prerequisite:** Approval and merge of this repository's release policy ADR in its earlier release-policy PR
- **Decision scope:** Rust workspace protocol, blocking/async and pool ownership

## Context

The Rust workspace contains cubrid-protocol, cubrid-client (Rust synchronous), cubrid-tokio and cubrid-pool. Rust lifetimes, Send/Sync guarantees and future cancellation differ fundamentally from an unrelated driver's cursor/error API. Existing Cargo shared version 0.1.0 does not mean every crate has the same maturity.

This decision is grounded in cubrid-rs's own public interfaces, repository code, and CUBRID correctness requirements. External implementations cannot substitute for local validation.

## Decision

Use a bounded, I/O-agnostic cubrid-protocol decoder/encoder with explicit errors and no unsafe code. Separate sync cubrid-client and Tokio async clients over shared protocol invariants; a driver connection owns its CAS session and transaction/cursor handles. The pool owns lease acquisition/return but must reject or discard connections with uncertain transaction/response state. Never retry SQL automatically after an ambiguous async/sync failure. Decide joint versus separate Cargo crate stability based on per-crate proof, not shared workspace version numbers.

## Alternatives considered

Duplicate framing in async and sync clients (rejected as target); transplant unrelated DB API interfaces (rejected); allow asynchronous cancellation to silently reuse a potentially corrupted connection (rejected); bump all crates to 1.0 solely for uniformity (rejected).

## Consequences and trade-offs

The typed ownership model may constrain streaming or concurrent operation APIs and can require safer connection retirement. More precise per-crate gates and external Cargo tests increase release work but reduce lifecycle bugs.

## Verification / acceptance criteria

Property/fuzz malformed frames; real CUBRID sync/Tokio/pool integration including CAS reconnect, cancellation, transactional rollback, lease cleanup and stale handles; verify MSRV and external Cargo consumer dependencies for every crate.

- [ ] Approve release/compatibility policy ADR 0001 **before** accepting this architecture ADR.
- [ ] The feasibility evidence and exact supported versions are recorded; unknown or unsupported behavior is not reported as implemented.
- [ ] Separate implementation issues, regression tests and release documentation agree with this decision.
- [ ] A maintainer reviews the decision and records acceptance explicitly; draft status alone never authorizes 1.0.

## References

[Rust driver design](../RUST_NATIVE_DRIVER_DESIGN.md), [Rust design #34](https://github.com/cubrid-lab/cubrid-rs/issues/34), [live verification #37](https://github.com/cubrid-lab/cubrid-rs/issues/37), [release policy PR #36](https://github.com/cubrid-lab/cubrid-rs/pull/36)

**This ADR is an architecture proposal, not a runtime implementation, migration completion, CI pass or approved release.**
