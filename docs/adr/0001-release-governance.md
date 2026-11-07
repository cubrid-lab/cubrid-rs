# ADR 0001: Release governance and compatibility contract

- **Status:** Proposed
- **Date:** 2026-11-07
- **Decision scope:** cubrid-rs release and supported-consumer contract
- **Owner:** Repository maintainers (approval not yet recorded)

## Context

A tag, passing mock suite, or green job without executed database cases does not establish a ready release of cubrid-rs. The release contract follows this repository's own responsibilities as the Rust CAS protocol, sync/async client and pool crates, including Cargo per-crate SemVer, public Rust interfaces, MSRV, packaging, and real CUBRID tests. External package versions cannot substitute for local compatibility evidence.

## Decision

Respect Cargo's semver-compatible 0.x family and rust-version/MSRV declarations. Current Rust workspace has cubrid-protocol, cubrid-client (Rust crate), cubrid-tokio and cubrid-pool, sharing workspace version 0.1.0; a shared number alone is not joint API maturity. Before 1.0 choose and document either jointly verified 1.0 for every crate, or independently versioned crates with correct dependency ranges and publish order. Verify each crate's packaged external consumer, Cargo registry-resolvable dependencies, async/ownership API, features, MSRV and real CUBRID tests. Coordinate a tested Rust driver generation with sea-orm-cubrid, not automated equal tags.

Applicable SemVer classification: a compatible bug/security fix is generally PATCH, an additive feature MINOR, and an incompatible public behavior or supported-environment change requires a reviewed breaking release (under current 0.x rules, commonly the next 0.y.0; at 1.x+ a MAJOR). Changes to SQL generation, data types, migrations or transactions can be compatibility-breaking without exported-type changes. A release candidate must have a reviewed version and notes, a final immutable commit, actual live CUBRID evidence when database behavior is claimed, and verified published artifacts. Keep omitted/failed checks visible; never invent approval or successful publication.

## Alternatives considered

- **Rejected:** Assume all workspace crates are ready for 1.0 because workspace.package has a single version; publish path-only dependencies directly; treat unrelated DB API policies as Cargo contracts.
- **Rejected:** Making this package's releases contingent on matching another repository's version number or calendar.
- **Rejected:** Treat an issue, green mock suite, GitHub tag or draft changelog as sufficient release evidence.

## Consequences and trade-offs

Independent crate gates and Cargo dependency ordering introduce release planning, but prevent premature version commitments and broken registry installs.

Maintainers must explicitly review the supported matrix and any runtime/API removal. Compatible fixes may be released independently when their own gates pass. Release automation can prepare PRs but must not bypass a maintainer review, exact candidate test or registry verification.

## Verification / acceptance criteria

- [ ] Maintainer reviews and accepts this ADR, links the approval and records acceptance date.
- [ ] Published README/release guide and dependency version declarations agree with the reviewed compatibility contract.
- [ ] A candidate-specific CI check identifies supported CUBRID server versions, tests run/passed/failed/skipped, runtime and driver/ORM versions, and actual artifact.
- [ ] Required release checks reject failed, missing, zero-case or skipped live tests, with scoped exceptions documented as unsupported/unverified—not counted as passes.
- [ ] User-visible breaking changes have accurate migration notes; correct author attribution in curated release notes comes from verified merged contributions.
- [ ] Existing immutable tags and published versions remain untouched by adopting this ADR.

## References

[release policy](../RELEASE_POLICY.md), [Cargo version policy #30](https://github.com/cubrid-lab/cubrid-rs/issues/30), [real DB release checks #37](https://github.com/cubrid-lab/cubrid-rs/issues/37)

**This ADR does not update version numbers, create a release, authorize publishing, or claim any database test has passed.** Its status remains **Proposed** until explicit maintainer approval.
