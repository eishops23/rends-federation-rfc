# RFC-0: Exchange of Verifiable Decision Records Across Independent Governance Deployments

**Status:** DRAFT v0.1 — HYPOTHESIS
**Repository:** github.com/eishops23/rends-federation-rfc *(planned move to github.com/rends-ai)*
**Author:** Fed Alcius (Rends)
**First reviewer:** *(pending — see Acknowledgments)*
**Date:** June 2026
**License:** MIT

---

> **Read this first.** This document is a hypothesis, not a protocol. No part of the
> exchange layer described here is implemented anywhere, including by Rends. The
> single-deployment primitives it builds on (§3.1) are shipped and independently
> verifiable today; the cross-deployment exchange layer (§3.2 onward) is a proposal.
> We are publishing the draft before building it because a protocol designed by one
> implementer is a product, not a protocol. Implementations, counter-proposals, and
> hostile reads are explicitly invited — see §7.

---

## 1. Problem Statement

> **[JON REWRITE — this section must be in your voice. My stub below captures the
> structure; replace the prose. Two paragraphs maximum. The test: Danielle should
> be able to quote a sentence from this section in his own writing.]**

**[STUB]** An AI agent's action is governed at runtime by a policy decision: this
action, evaluated against this policy, at this moment, with this outcome. Within a
single governance deployment, that decision can be made verifiable — anchored to the
exact policy revision in force, hash-chained against tampering, checkable by anyone
holding the record and the public verification procedure. Rends ships this today.

**[STUB]** But agents do not act within single deployments. An agent operated by
company A calls a tool exposed by company B. An insurer underwriting agent risk needs
to verify decision records produced by infrastructure it does not control. A regulator
auditing an incident needs records from three vendors' governance layers to agree on
what policy was in force when. In every one of these cases, the decision record must
*travel* — and a record that is only verifiable inside the deployment that produced it
is testimony, not evidence. The gap this RFC addresses: there is no standard way for
one governance deployment to verify another's decision records without trusting the
other's infrastructure.

**[JON — candidate scenarios to pick from or replace; the stub uses three, you may
want one sharp one instead: (a) cross-org agent-to-agent tool calls, (b) insurer /
underwriter verification, (c) regulator multi-vendor audit, (d) a customer switching
governance vendors without losing the evidentiary value of historical records.]**

## 2. Trust Model

The question the exchange layer must answer precisely: **what must hold for verifier
V (in deployment B) to accept a decision record produced by deployment A, without
trusting A's infrastructure, operators, or continued existence?**

We propose the following trust assumptions, ordered from what this RFC takes as given
to what it leaves open:

**Assumed (provided by the shipped substrate, §3.1):**

- **T1 — Record integrity is self-evident.** A decision record carries enough
  cryptographic material (content hashes, chain links) that any modification after
  production is detectable by recomputation alone. No callback to deployment A is
  required to check integrity.
- **T2 — Policy identity is content-addressed.** The policy a decision was evaluated
  against is identified by the hash of its canonical serialization, not by a name or
  version label deployment A controls. Two deployments computing the hash of the same
  policy content arrive at the same identifier.
- **T3 — Records are sequenced per origin.** Each producing context maintains an
  append-only hash chain; omission and reordering within a context are detectable by
  any holder of the chain segment.

**Required of the exchange layer (proposed in this RFC):**

- **T4 — Origin authenticity.** V must be able to establish that a record was
  produced by *deployment A* and not fabricated by a third party who understands the
  record format. (Integrity per T1 does not imply authenticity — anyone can produce a
  well-formed, internally consistent chain.) §5 and Open Question Q1 address this.
- **T5 — No liveness dependency.** Verification must not require deployment A to be
  online, cooperative, or still in business. Records must be evidence after the
  producer is gone.

**Explicitly out of scope for v0.1:**

- **Semantic trust.** This RFC makes a record's *production* verifiable — that this
  policy, applied to this input, yielded this outcome, in this sequence. It does not
  and cannot make the policy *good*, the input *honest*, or the deployment's sensors
  *accurate*. Garbage policy, faithfully anchored, verifies as garbage policy.
- **Revocation and key compromise recovery** beyond naming them as open questions (Q2).
- **Transport.** How records move between deployments (push, pull, gossip, manual
  export) is deliberately unspecified; this RFC defines what a record must contain to
  be verifiable on arrival, by any transport.

## 3. The Exchangeable Decision Record

### 3.1 Shipped substrate (implemented, verifiable today)

The exchange format does not start from zero. The following primitives are implemented
in the Rends reference deployment and published as open specifications:

- **Agent Audit Log Schema (AALS)** — MIT, github.com/eishops23/agent-audit-log-schema.
  Defines the per-org append-only audit record with SHA-512 hash chaining: each record
  commits to its predecessor, making the sequence tamper-evident and gap-evident.
- **Rends Policy Language (RPL)** — MIT, github.com/eishops23/rends-policy-language.
  JSON Schema (Draft 2020-12) policy definition language, third-party implementable.
- **Content-addressed policy revisions.** Policy mutations are captured in an
  append-only revision store; each revision is identified by SHA-512 over the
  canonical-JSON serialization of its content. Reversions produce new revisions
  (A→B→A yields three revisions, not a dedupe), so the revision sequence is a faithful
  history, not a deduplicated state.
- **Anchored findings.** Every governance finding is stamped with the revision
  identifier, revision number, and content hash of the exact policy revision in force
  at evaluation time; the containing evaluation carries an anchoring-schema version
  marking which anchoring contract its findings satisfy. A standalone verifier
  re-derives each finding's content hash from the policy revision content itself —
  SHA-512 over the canonical-JSON serialization — rather than comparing against a
  stored hash, so verification does not depend on trusting the producer's revision
  table.
- **Canonical JSON serialization** as the basis for all content addressing, so
  independent implementations hash identical content to identical identifiers (T2).

> **Pre-anchoring records.** Implementations that supersede an earlier non-anchored
> evaluation format MAY mark such records with `anchoringSchema='v0'` to signal
> *documented absence* of revision anchoring. Verifiers MUST treat documented absence
> as a non-failure but distinct from a verified `v1` record. This preserves chain
> integrity across the cutover from non-anchored to anchored evaluation without
> retroactively invalidating historical records.

Everything in this subsection is checkable now: the schemas are public, the reference
implementation is in production, and the verification procedures operate on records
plus public data — no privileged access to the producing deployment.

### 3.2 Proposed: the exchange envelope *(HYPOTHESIS — not implemented)*

To travel, a decision record needs more than its in-deployment form. We propose an
**exchange envelope** wrapping one or more AALS records with the material a foreign
verifier needs:

1. **The record set** — one or more contiguous AALS records (the decision(s) of
   interest plus enough chain context to verify sequencing locally).
2. **Policy revision content** (or a content-addressed reference resolvable without
   the producer) for every revision the enclosed findings anchor to — so the verifier
   can recompute T2 identities rather than take them on faith.
3. **Chain attestation** — the producing context's chain-head commitment at export
   time, signed by the origin (see T4 / Q1), allowing the verifier to place the
   record set within a sequence the origin is committed to.
4. **Origin identity material** — whatever Q1 resolves to: a public key, a
   certificate chain, a transparency-log inclusion proof. Deliberately left as the
   primary open question rather than prematurely specified.
5. **Envelope metadata** — format version, anchoring-schema version, export
   timestamp, producing-context identifier.

The envelope is itself canonical-JSON serialized and content-addressed, so envelopes
can be referenced, deduplicated, and counter-signed.

### 3.3 Design constraints carried forward from the substrate

These constraints are active in the reference implementation now, precisely so the
exchange layer remains buildable later without breaking changes: canonical JSON
everywhere; content hashes as the only cross-boundary identifiers; standalone
verifiability (no callbacks); per-context chains. Any implementation claiming
substrate compatibility must preserve all four.

## 4. Verification Across Boundaries

The proposed verification procedure for a foreign verifier V receiving an envelope
from deployment A, in order, failing fast:

1. **Envelope integrity** — recompute the envelope's content hash; reject on mismatch.
2. **Record chain replay** — recompute the SHA-512 chain across the enclosed AALS
   records; reject on any broken link (this is the same math as the shipped
   single-deployment verifier, applied by a foreign party).
3. **Anchor recomputation** — for each finding, recompute the policy revision content
   hash from the enclosed policy content via canonical JSON; compare against the
   stamped anchor. This proves the finding references the policy content the verifier
   is holding — not a label the producer could redefine.
4. **Chain placement** — check the record set against the signed chain-head
   attestation: the records must be consistent with a chain the origin has committed
   to. Detects selective disclosure of an alternate history.
5. **Origin authentication** — validate the origin identity material per whatever Q1
   specifies. *(In v0.1 this step is named but not specified.)*

Steps 1–3 are mechanically the shipped verification procedures relocated to a foreign
party — no new cryptography, only new packaging. Steps 4–5 are the genuinely new
surface, and they are where this proposal is weakest and review is most needed.

## 5. Security Considerations *(skeleton — to be expanded with first reviewer)*

- **Fabricated origins** (the T4 gap): a well-formed chain proves internal
  consistency, not provenance. Without Q1 resolved, an attacker can manufacture a
  plausible deployment wholesale.
- **Selective disclosure:** exporting only flattering record ranges. Chain-head
  attestation (envelope item 3) narrows this; periodic public chain-head publication
  (Q3) would narrow it further.
- **Policy substitution:** mitigated by anchor recomputation (§4 step 3) — the
  verifier hashes the policy content it actually holds.
- **Replay/recontextualization:** a true record presented as evidence for a different
  question. Envelope metadata helps; ultimately partly a semantic-trust issue (out of
  scope per §2).

## 6. Open Questions

> **[These are deliberately genuine, not rhetorical. Q1–Q3 are written for review
> through a C2PA / content-provenance lens — they map onto problems that community
> has worked longer than anyone in agent governance.]**

- **Q1 — Origin identity and attestation.** What establishes that an envelope's
  signing key belongs to "deployment A" as a real-world entity? Options span
  self-certifying identifiers, web-PKI binding, transparency logs, and C2PA-style
  certificate ecosystems. Each carries a different failure mode and a different
  centralization cost. *This is the question the rest of the proposal hinges on.*
- **Q2 — Key compromise and revocation.** If an origin key is compromised, what
  happens to the evidentiary status of records signed before vs. after the
  compromise window? Content provenance has wrestled with exactly this.
- **Q3 — Chain-head publication.** Should origins periodically publish chain-head
  commitments to a public, append-only location (transparency log, timestamping
  service) to bound selective-disclosure and back-dating attacks? At what cost in
  operational coupling?
- **Q4 — Minimum disclosure.** Can an envelope prove a decision occurred under a
  given policy without disclosing the full record contents (inputs may be
  confidential)? Hash-committed fields with selective opening? Deferred unless an
  implementer needs it for v0.x.
- **Q5 — What makes this a protocol.** Authorship is not authority. This document
  describes a protocol-in-effect only when a second, independent implementation
  exchanges and verifies records against the reference implementation. Until then it
  is a draft with one interested party.

## 7. Invitations

Three, matching the standing invitations at rends.ai/standards:

1. **Implementers:** build any piece — an envelope producer, a standalone verifier,
   a counter-proposal format. The substrate schemas (AALS, RPL) are MIT and
   implementable without Rends.
2. **Reviewers:** especially from content-provenance, transparency-log, and supply-
   chain-attestation backgrounds. Q1–Q3 are your territory; tear them apart.
3. **Operators:** if you run agent infrastructure and these records would change a
   real decision you make (procurement, underwriting, audit), tell us which scenario
   in §1 is yours — it determines which open question gets resolved first.

Discussion: GitHub issues on this repository.

## Acknowledgments

*(To be completed at publication: first reviewer credit; statement that review
shaped Q1–Q3.)*

## Changelog

- **v0.1** (June 2026) — initial public draft. HYPOTHESIS status. §3.2 onward
  unimplemented by design. Every claim in §3.1 was audited against the reference
  implementation source before publication; the standalone verifier's
  re-hash-from-content behavior was shipped as part of that audit rather than
  claimed ahead of it.
