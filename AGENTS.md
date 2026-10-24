# AGENTS.md

## Purpose
`cubrid-rs` provides native Rust database drivers for CUBRID - sync (`cubrid-client`) and async (`cubrid-tokio`) - implementing the CAS wire protocol directly, with no FFI or C dependencies.

## Read First
- `README.md`
- `docs/PRD.md`
- `docs/ARCHITECTURE.md`
- `docs/PROTOCOL_RESEARCH.md`
- `CONTRIBUTING.md`

## Working Rules
- All crates must compile with `#![deny(unsafe_code)]` - zero unsafe.
- Wire protocol details live in `cubrid-protocol`; clients depend on it, never the reverse.
- Keep public API surface minimal - expose only what users need.
- All protocol parsing must handle malformed input gracefully (no panics).
- Prefer `thiserror` for error types, never `anyhow` in library code.
- Tests go in `tests/` (integration) or inline `#[cfg(test)]` modules (unit).
- Match the ecosystem patterns from other cubrid-lab repos (README structure, badges, labels, workflows).

## Development Workflow (cubrid-lab org standard)

Use **one review policy** for human-authored contributions and all coding
tools (OpenCode, Codex, Claude Code, ChatGPT, Copilot, and others).
No Oracle pre/post review or particular AI reviewer is required by tool.

1. **Implement:** scope a focused change, add relevant tests, and update
   affected documentation in the same PR (or explain why docs are unaffected).
2. **GitHub CI:** required GitHub Actions checks on the final PR head are
   authoritative. Run applicable language-specific lint, tests, documentation,
   integration and security checks. Local runs are encouraged, not a separate
   prerequisite for merging.
3. **Review:** check correctness, risk, tests, security and maintainability.
   Oracle, Copilot and `agent-review` are optional **advisory** evidence.
4. **Merge:** address or disposition findings, verify required checks, and
   have the responsible maintainer decide on merge. Security-sensitive
   CI permissions, authentication and publishing changes require an
   independent maintainer's assessment.

Preserve existing branch protections, CI gates, integration tests, releases
and protected security paths. No automatic approval follows from green CI or
AI summaries.

## Validation
- `cargo check --workspace`
- `cargo clippy --workspace -- -D warnings`
- `cargo fmt --all -- --check`
- `cargo test --workspace`

## Issue triage lifecycle

New issues begin as `status: needs triage`, even when the author supplies
priority and size labels. The issue-opened workflow marks external reports;
automated issues created using `GITHUB_TOKEN` must supply the label at
creation since those events do not trigger another workflow. Maintainers,
not submitters or AI severity summaries, decide relative priority.

1. Confirm the actual problem and evidence (or say what is unverified),
   duplicate open/closed issues and open PRs, repository scope, issue type,
   dependencies and a testable completion boundary. Close duplicates or
   invalid issues with a concise reason.
2. Compare impact and urgency with **all open issues in this repository**;
   re-evaluate relative order rather than granting new issues automatic
   precedence. Choose exactly one of `priority: critical`, `priority: high`,
   `priority: medium` and `priority: low`.
3. Estimate effort with exactly one of `size: small` (focused edit),
   `size: medium` (several related files) or `size: large`
   (cross-cutting). Split large implementation work into independently
   verifiable issues when appropriate, retaining parent acceptance.
4. Read GitHub's *persisted* labels, confirm type, one valid priority and
   one valid size along with the completed backlog comparison, then remove
   `status: needs triage`. Do not auto-remove it based on labels alone.

For existing open work, migrate legacy XS/S → small, M → medium, L/XL →
large; preserve priority, claims, other labels and closed issue history.
Unclear labels stay queued for manual triage. Keep existing CI and release
review boundaries intact; this is not a new merge gate.

