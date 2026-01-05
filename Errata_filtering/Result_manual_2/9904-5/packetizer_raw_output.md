# Errata Reports

Total reports: 2

---


## Report 1: 9904-5-1

**Label:** Over‑broad scope of 'not known to be broken' claim for validation‐only SHA‑1 algorithms

**Bug Type:** Both

**Explanation:**

Section 5’s blanket claim that all algorithms marked as MUST or RECOMMENDED to implement are 'not known to be broken' fails to distinguish between algorithms intended for new signing/delegation and legacy algorithms (e.g. SHA‑1) retained solely for validation.

**Justification:**

- Section 5 explicitly states that the algorithms identified as MUST or RECOMMENDED to implement are 'not known to be broken', without qualifying the role (signing vs validation). (E1)
- Table 2 shows RSASHA1 and RSASHA1‑NSEC3‑SHA1 are MUST for DNSSEC Validation but NOT RECOMMENDED for DNSSEC Signing, implying a distinction that is lost in the blanket claim. (E2)
- Table 3 indicates that the SHA‑1 digest is MUST for validation but MUST NOT for delegation, underscoring the different treatment for active use versus legacy support. (E3)
- RFC 6781 and RFC 5702 excerpts acknowledge known cryptanalytic weaknesses with SHA‑1, which conflicts with a universal assertion of being 'not known to be broken'. (E4, E5)

**Evidence Snippets:**

- **E1:**

  Section 5: “In this document, the algorithms identified as MUST or RECOMMENDED to implement are not known to be broken at the current time…” and “this document concerns itself… specifically with the selection of ‘mandatory‑to‑implement’ algorithms.”

- **E2:**

  Table 2: RSASHA1 and RSASHA1‑NSEC3‑SHA1 have “Implement for DNSSEC Validation” = MUST and “Implement for DNSSEC Signing” = NOT RECOMMENDED; SHA‑1‑based algorithms thus appear with MUST in an “Implement for … Validation” column.

- **E3:**

  Table 3: SHA‑1 digest has “Use for DNSSEC Delegation” = MUST NOT, “Implement for DNSSEC Delegation” = MUST NOT, but “Implement for DNSSEC Validation” = MUST.

- **E4:**

  RFC 6781 Section 3.4.1: “At the time of publication, it is known that the SHA‑1 hash has cryptanalysis issues… The use of public‑key algorithms based on hashes stronger than SHA‑1 (e.g., SHA‑256) is recommended…”

- **E5:**

  RFC 5702 Section 8.1: “confidence in SHA‑1’s strength is being eroded by recently announced attacks… SHA‑2 is the better choice for use in DNSSEC records.”

**Evidence Summary:**

- (E1) Section 5 states all MUST/RECOMMENDED algorithms are 'not known to be broken'.
- (E2) Table 2 reveals that SHA‑1–based algorithms are mandated for validation but not for signing.
- (E3) Table 3 shows a similar dichotomy for the SHA‑1 digest regarding delegation versus validation.
- (E4) RFC 6781 notes known cryptanalytic issues with SHA‑1.
- (E5) RFC 5702 points out the eroding confidence in SHA‑1’s strength.

**Fix Direction:**

Revise Section 5 to either restrict the 'not known to be broken' claim to algorithms intended for new signing/delegation or explicitly exempt those maintained solely for validation.

**Severity:** Medium
  *Basis:* Misinterpretation of the security narrative could lead to undue reliance on algorithms with known weaknesses, particularly regarding legacy SHA‑1 validation.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1

---


## Report 2: 9904-5-2

**Label:** Ambiguous actor in 'users MUST upgrade the DS algorithm first' requirement

**Bug Type:** Underspecification

**Explanation:**

The normative requirement in Section 6 uses the generic term 'users' without specifying the responsible party, which may lead to divergent interpretations of who is required to perform the DS algorithm upgrade.

**Justification:**

- Section 6 states: “…Upgrading an algorithm at the same time as rolling to the new Key Signing Key (KSK) key will lead to DNSSEC validation failures, and users MUST upgrade the DS algorithm first before rolling to a new KSK.” (E1)
- Elsewhere the document specifies precise actors such as 'DNSSEC operators', 'implementers', 'authoritative servers', and 'validating resolvers' (e.g., Sections 1.1, 2.1), indicating that a similarly specific term should be used here. (E2)

**Evidence Snippets:**

- **E1:**

  Section 6: “…Upgrading an algorithm at the same time as rolling to the new Key Signing Key (KSK) key will lead to DNSSEC validation failures, and users MUST upgrade the DS algorithm first before rolling to a new KSK.”

- **E2:**

  Elsewhere, the document uses more specific actors such as “DNSSEC operators,” “implementers,” “authoritative servers,” and “validating resolvers” when giving guidance (e.g., Sections 1.1, 2.1).

**Evidence Summary:**

- (E1) Section 6 mandates DS algorithm upgrade using the undefined term 'users'.
- (E2) Other sections of the document clearly specify the operational actors responsible.

**Fix Direction:**

Amend Section 6 to explicitly name the intended actor (e.g., 'zone operators' or 'DNSSEC signers') responsible for upgrading the DS algorithm.

**Severity:** Medium
  *Basis:* The ambiguity in identifying the responsible party may lead to inconsistent implementation of the DS algorithm upgrade process, potentially resulting in DNSSEC validation failures.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2

---