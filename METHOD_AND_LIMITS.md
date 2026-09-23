# Method and limits

**Cutoff.** The decision point is 06:00 UTC on 29 August 2025, the last full hour before the first major-exchange spot-listing notice (HTX, 06:28:15 UTC, from the exchange's announcement API).

Every source is classed by what a committee could have seen at that moment:

| Class | Definition | Use |
| --- | --- | --- |
| Available | Dated before the cutoff | Used in the decision |
| Availability uncertain | Same-day items without an hour; undated documents | Flagged where cited |
| Published after the cutoff | Launch week, up to 1 September | Used only as retrospective confirmation |
| Retrospective | After trading opened | Used only as retrospective confirmation |

Issuer documents are taken from Internet Archive captures dated 26–28 August 2025. The gold-paper capture matches the Archive's content digest byte for byte.

**On-chain state.** Owner, multisig threshold and signers, guardian status, pause status and balances are read at block 23,244,849 with `eth_call`, not at the current block. Control powers are established in two ways:
- from the verified source of both implementations;
- by pinned-block simulations of the reassignment path. The owner succeeds; a blacklisted source reverts; a paused token reverts; a non-owner reverts.

Event histories (guardian appointments, pauses, reassignments, blacklist changes) come from full log queries starting at block 0.

**Decision rules.** Nine domains are rated Low, Medium, High or Critical. There is no composite score.

A finding is **Critical** only if all three conditions hold:
- it affects custody of customer assets or a mandatory legal or policy requirement;
- the exchange's own controls cannot neutralise it;
- it is unresolved on the record.

The outcome follows from the Critical findings:

| Critical findings | Recommendation |
| --- | --- |
| Pending information the issuer can provide | Defer |
| Resolved by verified information into enforceable pre-trade conditions | Conditional approval |
| Not convertible (issuer refuses mandatory diligence, legal ineligibility, custody risk beyond appetite, evidence of unlawful conduct) | Reject |

Two findings met the Critical test: the undocumented control powers and the ownership gap. Governance was rated High. Company-screened, advisory voting limits holders but does not by itself threaten custody.

**Neutrality.** The rubric is applied to structure, not to the identity of the people involved. Politically exposed persons among the disclosed beneficiaries call for enhanced due diligence and senior sign-off; they do not raise a rating. Remove them and keep the structure, and the ownership gap is still a blocking KYB item.

**Corrections made during review.** Earlier drafts of this review contained errors that re-reading the source and on-chain record caught:
- They said reassignment bypassed the blacklist and pause. It does not.
- They treated pausing as new in V2. It existed in V1.
- They inferred an "activation rate" from two populations that do not match.
- They projected today's multisig configuration back to 2025. It is now read at the cutoff block.
- They gave the margin before the first notice as "at least an hour". It is 28 minutes.

Each correction was checked against saved primary evidence before it was accepted. None was accepted on argument alone.

**Limits.**
- Public record only: no issuer answers, no exchange-internal data (order books, deposits, market-maker terms), no interviews.
- 4.977 B of the tokens sold is not attributed in the launch table.
- The historical configuration of the sale-proceeds multisig was not reconstructed.
- SEC filing texts were not retrieved; filing acceptance times were used instead.
- Version history of the issuer's documentation site was not recovered from the Internet Archive, and other archives were not tried.
- Legal questions (securities status, MiCA admission) are identified, not answered.
- Not yet reviewed by an exchange listing or compliance practitioner.

The public package covers the headline claims only. The full working record is retained separately for detailed review. It comprises source, evidence and calculation registers, saved RPC responses, verified contract source, reproduction scripts and a change log.

[Return to overview](README.md) · [Evidence trail](EVIDENCE_TRAIL.md)
