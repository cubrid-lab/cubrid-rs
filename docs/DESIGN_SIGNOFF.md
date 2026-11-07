# Design sign-off and TDD acceptance plan

> **Status: Not signed off / design under review.** This plan captures *expected* test behavior, not tests already written or run. The release-policy PR has **merged**, adding ADR 0001/0002/0004; those decision records retain their **Proposed** status until approved, not automatically Accepted. The native architecture ADR 0003 (for this repository, if applicable) is proposed in this design draft. No implementation, CI success, approved public API or live DB outcome is implied.

## Project setup prerequisite

**ADR 0005 and the development setup guide** were added by the merged release-policy PR (ADR status remains Proposed); check [the setup documentation](https://github.com/cubrid-lab/cubrid-rs/blob/main/docs/DEVELOPMENT_SETUP.md) and the manifest/CI before checking these boxes.

- [ ] **Clean checkout verified:** language-native runtime/compiler and tooling are selected with a documented developer versus published consumer support distinction.
- [ ] **Dependency resolution verified:** Cargo workspace crate dependencies, MSRV and feature graph resolve without undocumented replacement, incompatible versions, or incorrect import paths.
- [ ] **Read-only setup checks executed:** record **actual** build/type/lint/unit results, not command examples; all known missing config/scripts have explicit blockers.
- [ ] **Real DB harness decided:** identify a disposable real CUBRID service and concrete test selection, or mark missing harness as blocked. A Docker healthcheck, fake broker or zero/fully skipped suite is not evidence.
- [ ] **External package consumer planned:** external Cargo crate packaging/consumer checks, with release preflight separate from initial setup.
- [ ] **Repository developer guidance consistent:** README, CONTRIBUTING, AGENTS, tooling and CI refer to commands that actually exist and do not silently raise the supported runtime floor.
- [ ] **Maintainer approval recorded** for ADR 0005 and any associated Cargo/MSRV/feature compatibility, dependency or CI changes.

These are **not completed** merely because the files are written. If setup is blocked, fix or explicitly scope it in a separate reviewed task before claiming the design is ready for implementation.

## Design-first gate

Before implementation changes are treated as approved, establish each of the following in reviewed project documentation and link the evidence. Check only when genuinely resolved, not merely because this template exists.

- [ ] **Boundaries and interfaces:** owned layer, public API, language-native patterns, tested public dependency interfaces, version and compatibility requirements.
- [ ] **Protocol/SQL contract:** CUBRID-specific wire/query semantics, supported and unsupported behaviors, parameter/result mapping, resource lifetimes.
- [ ] **State and failure model:** connection/session/transaction/statement states, cancellation, retries, ambiguous outcomes and cleanup.
- [ ] **Data types and precision:** NULL, signed integers, exact numeric, text, date/time, collections, binary/LOB and errors.
- [ ] **Schema/migration contract (where applicable):** CUBRID DDL, introspection, reversible behavior, destructive change guardrails.
- [ ] **Security and operations:** credentials/log redaction, timeout limits, resource bounds, cancellation behavior and diagnostics.
- [ ] **Verification design:** red/green cases (below), real CUBRID versions, non-vacuous failure gate, independent JDBC semantic comparison when appropriate.
- [ ] **Release readiness:** published package and supported consumer matrix, reviewed release notes, stable 1.0 exit conditions.
- [ ] **Upstream feasibility:** independently verify the actual driver/protocol API feasibility and unsupported methods before committing to architecture.
- [ ] **Review approval:** maintainers explicitly approve the language-native architecture ADR, test contract and open blockers; no speculative sign-off.

The PR/issue must distinguish **decided**, **proposed**, **blocked**, and **requires live verification**. If a significant feasibility constraint prevents the proposed direction, update the ADR and its alternatives before implementing a workaround.

## TDD acceptance cases

For each feature or bug: first run a targeted test that fails for the intended reason (**Red**), make the smallest implementation pass (**Green**), refactor under the same tests, and verify live database behavior before claiming compatibility. A red failure log may be summarized truthfully in the PR but need not be committed. The final PR must have green required checks.

| Test contract (proposed) | Expected observable outcome | First test layer | TDD starting point |
| --- | --- | --- | --- |
| Protocol bounds | Malformed, truncated or oversized frames never panic/allocate without bound; typed error preserves CUBRID code | Rust unit/proptest/fuzz seeds | red malformed packet first |
| Session handle lifetime | Stale cursor/LOB and prepared handles never work on a new CAS generation | fake CAS + owned handle tests; live broker reset | red reused-ID close hazard |
| Async cancel | Dropping a Tokio future during request does not leave unread response or automatically replay SQL | deterministic fake I/O + Tokio tests; live error test | red cancelled fetch causing next query corruption |
| Pool ownership | No connection is returned to pool with active transaction/cursor or after an uncertain commit | Tokio pool stress and lease state tests | red leaked transaction on next borrower |
| Public crates | `Send`/`Sync`, MSRV and all four crates resolve via external Cargo package with intended version graph | compile/trait tests, `cargo package`, consumer Cargo lock | red packaging for missing versioned path deps, then green after design |

## Project boundary

**Role:** Rust protocol, blocking client, Tokio client and connection pool.

Transport-independent `cubrid-protocol` handles bounded framing and codecs. `cubrid-client` blocking I/O, `cubrid-tokio` async and `cubrid-pool` leases each own separate Rust contracts; SeaORM SQL belongs in the adapter repository.

## Decisions still blocking full sign-off

Choose shared versus independent crate versioning (currently workspace 0.1.0); verify MSRV 1.75 versus CI; #30, #33, #34, #37. Verify actual inter-crate version/path publish constraints before any 1.0.

These are outstanding verifications, not proof of failure. Existing code and currently declared support are not silently reclassified as unsupported merely because a test is absent.

## Release and implementation sequencing

1. Approve the repository release policy and [TDD ADR 0004](https://github.com/cubrid-lab/cubrid-rs/blob/main/docs/adr/0004-test-first-development.md) in the release-policy PR.
2. Resolve the above design/feasibility blockers with evidence, review [Rust driver proposal](RUST_NATIVE_DRIVER_DESIGN.md), and explicitly accept the architecture/contract ADR when applicable.
3. Implement each scoped issue with test-first evidence, one observable contract at a time. No implementation-wide PR before essential extension feasibility.
4. Run real CUBRID and downstream consumer tests on the final release candidate; document exact versions and test counts; do not publish based on mocks alone.

**Tracked design PR:** #35. This file is a **sign-off checklist**, not a signed-off design.
