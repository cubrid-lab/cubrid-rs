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

New issues in **this repository** enter `status: needs triage`, including
issues opened by maintainers or agents. GitHub's `issues: opened` workflow
applies this label to external submissions; issue forms set it immediately.
Automation using `GITHUB_TOKEN` must add it at creation because its own
issue events may not retrigger Actions.

1. Inspect problem evidence and unknowns, existing issues and open PRs,
   validity, duplicates, issue type, scope/non-goals and real dependencies.
   Explain closures of duplicates and invalid requests.
2. Examine **all open issues in this repository** (including later API pages)
   before setting relative priority, considering impact, active work and
   blocking prerequisites rather than issue age or ID.
3. Use exactly one `priority: critical/high/medium/low` and exactly one
   existing `size: XS/S/M/L/XL`. Size is effort, not urgency. Split tasks
   too large for independent verification; preserve contributors and owners.
4. Fetch saved GitHub labels to confirm type, priority and size together
   with scope and duplication status. Only after that check, remove
   `status: needs triage`; keep it if further review is required.

No automation chooses a severity or marks triage complete. This routine is
not a new CI gate and leaves existing validation, security and releases alone.
