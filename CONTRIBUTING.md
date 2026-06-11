# Contributing to RFC-0

RFC-0 is a **pre-review draft** as of v0.1. Contributions during this phase have outsized influence on the final shape — small comments now beat large changes later.

## Scope

This repository tracks the RFC text and its review history only. It does NOT track implementations — those live in the reference implementations linked from `README.md`.

## How to contribute

1. **Read the current draft** at `rfc-0.md`.
2. **For prose / framing / scope critique:** open an issue. Tag with `prose` or `scope`.
3. **For technical objections (canonicalization, hash algorithm, chaining rules):** open an issue first, then a pull request with the proposed change. Tag with `technical`.
4. **For implementation reports:** open an issue with the link, conformance status, and any gaps you hit. Tag with `implementation`.

## Review process

- Drafts cut from `main` carry a version label (`v0.1`, `v0.2`, …).
- Public review opens at `v0.1` and remains open through `v0.x` iterations.
- A `v1.0` cut requires: (a) two independent reference implementations or formal review by external standards reviewer(s), AND (b) Fed Alcius (`@eishops23`) merge-signing the cut commit.
- Breaking changes between `v0.x` revisions are permitted with a release note in `VERSIONING.md`.
- After `v1.0`, semantic versioning applies (see `VERSIONING.md`).

## Code of conduct

Be precise, be kind. Spec work is high-leverage; bad-faith argumentation wastes everyone's time. Disagreements should produce written rationale, not heat.
