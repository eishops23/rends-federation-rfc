# RFC-0: Exchange of Verifiable Decision Records Across Independent Governance Deployments

**Status:** DRAFT v0.1 — HYPOTHESIS — pre-review, not yet announced.

Rends is designed so independent deployments can eventually exchange verifiable decision records. This repository tracks the RFC-0 draft and its review history.

See [`rfc-0.md`](./rfc-0.md) for the draft.

## Reference implementations

- **Rends (first)** — the [Rends Comply engine](https://rends.ai/standards#anchoring) is the first implementation. The audit trail, rule revisions, and standalone verifier described in §3 are shipped at Rends and exercised in production.

## Move planned

Spec repos currently live under our personal GitHub. We plan to move them to a `github.com/rends-ai` org as soon as the org name is available — the current URLs will redirect.

## License

MIT. See [`LICENSE`](./LICENSE).

## Get involved

- Read the draft, open an issue with prose critique or specific objections.
- Propose alternate canonicalization or hash-algorithm agility rules via pull request.
- Implement a second reference, even partial — link the implementation in an issue and we'll co-develop conformance fixtures.
