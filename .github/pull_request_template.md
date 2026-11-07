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
