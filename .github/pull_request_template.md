## High level description of this Pull-request
Include motivations, reasons, and background to add context to your contribution.
Include a description of changes associated with your commit(s)

## Related Issues
- List all related issues or NA

## Reviewers
- Use @Mentions to specify the reviewers for your PR.

# CHECKLIST:
Make sure all items are marked when you submit the pull-request.

- [ ] Relevant documentation for functions, tests, crates, or docs has been updated
- [ ] All tests pass with no errors (`cargo test --workspace`)
- [ ] Code passes `cargo clippy --workspace -- -D warnings`
- [ ] Code is formatted (`cargo fmt --all`)
- [ ] Update `CHANGELOG.md` with a summary of your changes if requested

## CUBRID correctness and regression evidence

<!-- For affected behavior, show the observed checks or mark unavailable honestly. See docs/ENGINEERING_INVARIANTS.md. -->

- [ ] Changes to protocol, sync/async I/O or pools include malformed-frame, cancellation, stale-handle and transaction ownership/failure reasoning as applicable.
- [ ] Rust public API, `Send`/`Sync`/lifetime and Cargo/MSRV compatibility were assessed if affected.
- [ ] Real CUBRID tests were executed and their versions/results recorded if database behavior changed, or their absence is explicitly stated; no green result comes from zero/fully skipped tests.
- [ ] The affected crate's packaging and versioned dependency contract is reviewed for user-visible changes.

## Test-first evidence and design approval

<!-- Applicable to behavioral implementation. For docs-only or design feasibility work, write N/A and the actual evidence. See docs/adr/0004-test-first-development.md and docs/DESIGN_SIGNOFF.md (in the design PR where applicable). -->

- [ ] **Design:** The public behavior/SQL/transaction/error contract and owning layer were reviewed before implementation (link ADR/issue or explain N/A).
- [ ] **Red:** A targeted test failed for the expected **behavioral** reason; summarize the observed failure or state why test-first did not apply.
- [ ] **Green:** The smallest implementation passes the new test; name the exact test command and outcome.
- [ ] **Refactor:** Related regression tests and required final-head checks pass, with observed CI links when available.
- [ ] **Live CUBRID:** For behavior depending on the server, distinguish real DB tests from mocks; include exact server/runtime/driver versions and passed/failed/skipped cases, or explicitly mark the behavior unverified.
- [ ] **Release:** No version/support claim or publication is inferred from a green mock or skipped integration job; final release candidates follow the separately reviewed full DB gate.
