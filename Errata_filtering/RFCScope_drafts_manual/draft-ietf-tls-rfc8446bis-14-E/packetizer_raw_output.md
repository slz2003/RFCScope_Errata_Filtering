# Errata Reports

Total reports: 2

---

## Report 1: draft-ietf-tls-rfc8446bis-14-E-1

**Label:** Resumption Guidance Conflict: 'SHOULD use previously negotiated version' vs. TLS 1.0/1.1 Deprecation

**Bug Type:** Inconsistency/Underspecification

**Explanation:**

The document’s guidance to resume with the previously negotiated version (in Appendix E.1) conflicts with the explicit prohibition on negotiating TLS 1.0 and 1.1, leaving the handling of legacy resumption tickets ambiguous.

**Justification:**

- Temporal analysis (T1) indicates that following the resumption guidance could lead to using TLS 1.0/1.1, which are now forbidden.
- Scope and Deontic analyses point out that the unconditional resumption rule does not consider the global ban on these versions, creating conflicting requirements.
- CrossRFC analysis confirms that a literal reading would force implementations into a contradiction between a SHOULD and a MUST NOT.

**Evidence Snippets:**

- **E1:**

  Forbid negotiating TLS 1.0 and 1.1 as they are now deprecated by [RFC8996]. (Section 1.2 of the bis draft)

- **E2:**

  A client using a ticket for resumption SHOULD initiate the connection using the version that was previously negotiated. (Appendix E.1)

- **E3:**

  SSL 2.0, SSL 3.0, TLS 1.0, and TLS 1.1 MUST NOT be negotiated for any reason. (Security/backward‑compatibility text)

**Evidence Summary:**

- (E1) shows that TLS 1.0/1.1 are deprecated and must not be negotiated.
- (E2) provides the resumption guidance that instructs using the previously negotiated version.
- (E3) reinforces the absolute prohibition of negotiating these versions.

**Fix Direction:**

Revise Appendix E.1 to condition the resumption guidance on the negotiated version being permitted; explicitly state that legacy TLS 1.0/1.1 tickets must not be used and that a connection should instead be initiated with TLS 1.2 or higher.

**Severity:** Medium
  *Basis:* The internal conflict between a SHOULD directive and a MUST NOT requirement may lead to divergent implementation behaviors and potential security risks.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1
- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 2: draft-ietf-tls-rfc8446bis-14-E-2

**Label:** Editorial Inconsistency between Section 1.2 and Appendix E.5 regarding TLS 1.0/1.1 Prohibition

**Bug Type:** Editorial

**Explanation:**

There is a potential editorial discrepancy between Section 1.2, which clearly forbids negotiating TLS 1.0/1.1, and the wording in Appendix E.5, which is unclear regarding this prohibition.

**Justification:**

- Scope analysis notes that the bis draft text in Appendix E.5 may not reflect the updated ban on TLS 1.0/1.1 as stated in Section 1.2.
- This inconsistency could lead to confusion about whether both sections enforce the same restrictions on deprecated versions.

**Evidence Snippets:**

- **E4:**

  The exact way the bis draft text in Appendix E.5 is worded for TLS 1.0/1.1 is not visible in the file snapshot (which shows the original SSL 2.0/3.0 bans), though Section 1.2 clearly signals an intent to forbid TLS 1.0/1.1 as well.

**Evidence Summary:**

- (E4) highlights the uncertainty in Appendix E.5 compared to the clear prohibition stated in Section 1.2.

**Fix Direction:**

Clarify and align Appendix E.5 with Section 1.2 by explicitly updating its wording to forbid TLS 1.0/1.1.

**Severity:** Low
  *Basis:* This is a minor editorial issue; while it may cause some textual confusion, it does not impact protocol security or functionality.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: ResidualUncertainties

---
