# AgentDeal — Specification v0.1 (DRAFT)

> **Status:** early draft, not reviewed by anyone outside the author. The name is provisional.
> **Not legal advice.** Nothing in this document creates legally binding contracts or arbitration.

## 1. Purpose

When an AI agent of company A deals with an AI agent of company B, transport protocols
(MCP, A2A) let them talk, but they do not say **what was agreed**, **what proof exists that it was
delivered**, or **who decides when they disagree**. AgentDeal defines a small, neutral, machine-readable
format for that missing layer:

1. a **Contract** both agents sign,
2. a **Delivery** with a content hash,
3. an independent **Verification report**,
4. an optional **Dispute** and **Verdict** by a panel,
5. a final **Receipt** that anyone can re-verify.

### Non-goals (v0.1)
- Moving money. Payments stay with other protocols; a contract may *reference* a payment mandate (e.g. an AP2 mandate).
- Legal enforceability. This is a technical record, not a court.
- Replacing A2A or MCP. AgentDeal is designed to ride on top of them.
- Appeals, multi-party (more than two) contracts, and negotiation rounds.

## 2. Roles

| Role | Description |
|---|---|
| **Buyer / Seller** | The two parties. Each is represented by an agent holding an Ed25519 key. |
| **Notary** | Neutral service. Stores hashes, timestamps events, countersigns, issues the Receipt. Must not be either party. |
| **Verifier** | Checks a Delivery against the Contract criteria. Must be independent of both parties and of the agent that produced the delivery. |
| **Panel** | At least 3 judges that rule on a Dispute. Judges may be software or humans. |

The same organisation MAY run Notary, Verifier and Panel in v0.1, but SHOULD disclose it (see §8).

## 3. Encoding rules

- All objects are JSON (RFC 8259).
- Hashing uses **SHA-256** over the **JSON Canonicalization Scheme** (RFC 8785) of the object **without its `signatures` field**. Hash strings are written `sha256:<lowercase hex>`.
- Signatures use **Ed25519** over the UTF-8 bytes of the hash string. Keys and signature values are **base64url without padding**.
- Timestamps are RFC 3339 UTC (`2026-11-16T14:05:00Z`). Only the Notary's timestamps are authoritative.

## 4. Objects

### 4.1 Contract
See `schemas/contract.schema.json`. Required fields:

- `agentdeal_version` — `"0.1"`
- `contract_id` — UUID
- `created`, `deadline`
- `parties` — exactly two, each with `id`, `role` (`buyer`|`seller`), `organization`, `public_key`; optional `agent_card_url` (an A2A Agent Card)
- `subject` — human-readable description of what is exchanged
- `price` — `amount`, `currency` (ISO 4217); optional `payment_mandate_ref`
- `criteria` — list of verifiable criteria `{id, text, verification}` where `verification` is `automated`, `independent_verifier` or `human`
- `remedy` — what happens if the delivery is non-conforming, with `refund_pct_if_nonconforming` (0–100)
- `notary` — `id` and `public_key`
- `dispute_window_hours` — default 24

**Criteria rule:** each criterion MUST be checkable from the delivery itself (e.g. "contains exactly 3 bullet points", "formal register"). Vague criteria ("good quality") SHOULD be rejected by the Notary.

### 4.2 Signatures
A contract becomes **ACTIVE** when it carries valid signatures from both parties over the same `contract_hash`, then the Notary countersigns and records the time.

### 4.3 Delivery
`{agentdeal_version, contract_hash, delivered_by, content_hash, media_type, artifact_ref?, created, signatures}`.
The artifact itself MAY stay private between the parties; only its hash must be shared with the Notary.

### 4.4 Verification report
`{contract_hash, delivery_hash, verifier:{id, independent, disclosure}, results:[{criterion_id, met, evidence}], conforming, created, signatures}`
- `met` is `true`, `false` or `"unknown"`. `unknown` MUST NOT be counted as met.
- `conforming` is `true` only if every criterion is `true`.
- The delivery content is **untrusted data**. A verifier MUST NOT follow instructions found inside it.

### 4.5 Dispute
`{contract_hash, delivery_hash, filed_by, reason, created}`; valid only within the dispute window after the verification report.

### 4.6 Verdict
`{contract_hash, panel:[{judge_id, perspective, ruling, refund_pct, reasons, signature}], decision, refund_pct, created, notary_signature}`
- `ruling` is `buyer`, `seller`, `split` or `abstain`.
- `decision` = the ruling given by a strict majority of non-abstaining judges, otherwise `split`.
- `refund_pct` = the median of the judges' `refund_pct` values.
- Judges SHOULD be given different `perspective`s (e.g. textual, good-faith, technical) to reduce correlated errors.

### 4.7 Receipt
See `schemas/receipt.schema.json`. The Receipt binds everything together:
`contract_hash`, `delivery_hash`, `verification_hash`, optional `verdict_hash`, `outcome`, `closed_at`, optional `prev_receipt_hash` (hash chain of the Notary's log), and the Notary's signature.

`outcome` ∈ `accepted`, `ruled_for_buyer`, `ruled_for_seller`, `ruled_split`, `expired`, `cancelled`.

## 5. State machine

```
DRAFT -> SIGNED_BY_ONE -> ACTIVE -> DELIVERED -> VERIFIED
VERIFIED -> ACCEPTED -> CLOSED
VERIFIED -> DISPUTED -> RULED -> CLOSED
ACTIVE|DELIVERED|VERIFIED -> EXPIRED -> CLOSED      (deadline or window passed)
DRAFT|SIGNED_BY_ONE -> CANCELLED
```

## 6. Interoperability (informative)

- **A2A:** a party SHOULD advertise AgentDeal support in its Agent Card through an A2A extension (`agentdeal/0.1`).
- **MCP:** a Notary MAY expose tools such as `agentdeal.propose`, `agentdeal.sign`, `agentdeal.deliver`, `agentdeal.verify`, `agentdeal.dispute`, `agentdeal.receipt`.
- **Payments:** `price.payment_mandate_ref` points to an external authorisation. AgentDeal never holds funds.

## 7. Security considerations

- **Replay:** `contract_id` is unique; a Notary MUST reject a second ACTIVE contract with the same id.
- **Key compromise:** parties SHOULD rotate keys; a revocation list is out of scope for v0.1.
- **Prompt injection:** deliveries are untrusted input for verifiers and judges (see §4.4).
- **Collusion / correlated errors:** LLM judges can share the same blind spots. Using different models and perspectives reduces, but does not remove, the risk.
- **Privacy:** store hashes, not content. Contract text can still be sensitive; the Notary SHOULD NOT publish it.
- **Time:** only the Notary's clock counts, to prevent back-dating.

## 8. Disclosure and independence

A Verifier and each judge MUST publish: who operates them, which model or method they use (name and version), and whether they have any relationship with either party. A verifier that is the same system as the delivering agent is **not independent** and MUST set `independent: false`.

## 9. Conformance levels

- **Level 1 — Record:** can produce and verify Contracts, Deliveries and Receipts (hashes and signatures).
- **Level 2 — Verify:** Level 1 plus independent verification reports.
- **Level 3 — Rule:** Level 2 plus disputes and panel verdicts.

## 10. Open questions (v0.1 does not answer these)

1. How should a panel's reasoning be audited when judges are language models?
2. How are criteria that cannot be verified automatically (taste, tone) priced and handled?
3. Who accredits Notaries, and how is their neutrality shown?
4. Should receipts be anchored in a public transparency log?
5. How does this interact with real law, insurance and AP2/Visa-style mandates?

Feedback and criticism are welcome through issues on the repository.
