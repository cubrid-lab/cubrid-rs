# Contributing to cubrid-rs

**Issue titles** use type-only prefixes such as `feat:` and `bug:` without module scope. **PRs and commits** continue to use Conventional Commits and local CI. See [AGENTS.md](AGENTS.md) for evidence, triage and verification responsibilities.

Thank you for your interest in contributing. This guide explains the development workflow and quality standards for this repository.

## Table of Contents

- [Development Setup](#development-setup)
- [Running Tests](#running-tests)
- [Code Style](#code-style)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Reporting Issues](#reporting-issues)

---

## Development Setup

### Prerequisites

- Rust 1.75 or later
- Git
- Docker (for integration tests)

### Installation

```bash
git clone https://github.com/cubrid-lab/cubrid-rs.git
cd cubrid-rs
cargo build --workspace
```

---



## Reproducible development setup

Before changing code, follow [the repository-specific development setup](docs/DEVELOPMENT_SETUP.md) and [ADR 0005](docs/adr/0005-development-setup.md). Start from the committed language-native manifest and a **clean checkout**, run non-mutating local checks, and record the actual toolchain and results. Do not interpret a mock suite or missing/fully skipped integration tests as real CUBRID verification, and do not alter Cargo crate compatibility, feature graph or MSRV or release tooling to work around an unverified local setup issue. The setup ADR is Proposed and awaits review.

## Test-driven implementation

For bug fixes or behavioral features, start from a reviewed interface/SQL/transaction design. Use the [TDD decision](docs/adr/0004-test-first-development.md): **Red** (run a new focused test and capture the genuine target failure), **Green** (smallest production change passing that test), **Refactor** (rerun related tests and final PR CI). In the PR description record the failing assertion/error **actually observed**, the passing assertion, which regressions ran, and whether real CUBRID validation ran or remains unverified. A mock-only pass cannot establish server compatibility; the exact release candidate still requires full live DB validation. No deliberate failing tests or skipped release checks may be left to satisfy the TDD process. For documentation and upstream feasibility work, describe the design/probe evidence rather than forcing an irrelevant Red test. The ADR and design contracts are proposals until reviewed and accepted.

## CUBRID correctness and regression evidence

For changes to Rust client/crate ownership, error and protocol semantics, read [this repository's engineering invariants](docs/ENGINEERING_INVARIANTS.md) **before** implementation and review. The requirements follow the Rust CAS workspace and its own transport/ownership contracts, not another package's failure reports or test results. In the pull request, describe the behavioral invariant affected, the normal and failure cases tested, and which checks were offline, real CUBRID or independently compared with JDBC. State the exact CUBRID/runtime/driver versions and test outcomes when claiming database compatibility. **Do not mark tests as passed when they were skipped, missing or never executed.**

For documentation-only or nonbehavioral changes, explain why a database run is unnecessary rather than manufacturing an integration result. The final **release candidate** still follows the repository's full live-CUBRID release gate, separate from ordinary PR path optimization. These expectations supplement, and do not weaken, the established review/CI rules.

## Running Tests

### Unit Tests

```bash
cargo test --workspace
```

### Integration Tests (Requires CUBRID)

```bash
docker compose up -d
export CUBRID_TEST_DSN="cubrid://dba:@localhost:33000/testdb"
cargo test --workspace -- --ignored
docker compose down
```

---

## Code Style

Use standard Rust tooling and keep CI checks clean.

### Required checks

```bash
cargo fmt --all
cargo clippy --workspace -- -D warnings
```

### Conventions

- Keep APIs small and explicit
- Avoid breaking changes to public interfaces without discussion
- Add tests for protocol/driver behavior changes
- Keep docs and examples aligned with actual behavior

---

## Pull Request Review and Merge Policy

All contributors follow the same process regardless of whether they use
OpenCode, Codex, Claude Code, ChatGPT, Copilot or no AI tool. Keep a PR
focused, provide relevant tests and affected docs, and describe validations
not performed locally. **Required GitHub Actions CI** is the authoritative
test record for the final head; local runs are recommended but not an extra
merge gate. Oracle, Copilot and `agent-review` may help reviewers but their
results are advisory, not required tool-specific approvals. The responsible
maintainer checks correctness, tests, security, maintainability, CI and
outstanding review findings before merging. Changes affecting credentials,
workflow write permissions or release/publishing operations receive explicit
independent maintainer scrutiny. Existing branch protection and repository
test commands remain unchanged.

## Pull Request Guidelines

### Before submitting

1. Create a focused branch from `main`.
2. Add or update tests for behavior changes.
3. Run:
   - `cargo test --workspace`
   - `cargo clippy --workspace -- -D warnings`
   - `cargo fmt --all -- --check`
4. Update docs and changelog-related files when requested.

### PR content

- Keep each PR scoped to one feature/fix.
- Explain both what changed and why.
- Link related issues (for example: `Fixes #42`).
- Include migration notes if behavior changes.

### Review process

- At least one review before merge.
- Required CI checks must pass.
- Backward compatibility is preferred unless intentionally changed.

---

## Issue ownership and coding agents

Before implementing a change, inspect the latest `main`, open issues and
overlapping PRs in **cubrid-rs**. New issues start with `status: needs triage`:
maintainers review evidence, duplicate work, type, scope and relative urgency
against this repository's open queue. Use exactly one
`priority: critical/high/medium/low` and one `size: XS/S/M/L/XL`.
Only remove the intake status after verifying the saved GitHub labels and
actual triage decision. A priority label never means a PR is merge-ready.

A human contributor can work on an unclaimed issue without a named AI reviewer
or preapproval; maintainer-operated autonomous agents need explicit authorization
and a confirmed assignee for internally assigned/reserved work. Check active
contributors and agree on handoffs before overlapping the same change.

Keep protocol framing and malformed-input handling panic-free, including in
both sync and Tokio clients. Inline `#[cfg(test)]` modules are valid Rust unit
tests; separate `tests/` integration cases run only when appropriate.
`cargo test --workspace` does **not** prove ignored, live CUBRID scenarios
passed; run `cargo test --workspace -- --ignored` with a real database and
report that evidence separately. No additional agent-specific test gate.

## Reporting Issues

When reporting bugs, include:

- Rust version (`rustc --version`)
- CUBRID version
- `cubrid-rs` version
- Reproduction steps or minimal example
- Full error output/logs

For feature requests, include problem statement and expected API behavior.
