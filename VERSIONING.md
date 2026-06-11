# Versioning policy

## Pre-1.0 (current)

- `v0.x` drafts are explicitly unstable. Any field may be renamed, retyped, or removed between minor versions.
- Each version is tagged in this repo (`v0.1`, `v0.2`, …) with a release note in this file.
- The `Status:` line in `README.md` reflects the current draft state.

## Post-1.0 (future)

- Semantic versioning applies: `MAJOR.MINOR.PATCH`.
- `MAJOR` increments only for breaking changes to canonical JSON construction, hash algorithm, or chain linkage rules.
- `MINOR` adds new optional fields, new optional sections, or expanded enums.
- `PATCH` is clarification, typo, or rewording with no semantic change.

## Hash algorithm agility

The RFC reserves the `hashAlgorithm` enum (initially constrained to `sha-512`) for future agility. Adding SHA-3 or BLAKE3 is a `MINOR` change; switching the default away from SHA-512 is a `MAJOR` change.

## Reference implementations

A reference implementation may declare conformance to a specific RFC version. The `Status:` line of each implementation's README should cite the exact RFC version it targets.

## Release notes

### v0.1 — 2026-06-11

Initial pre-review draft. HYPOTHESIS status. Not yet circulated.
