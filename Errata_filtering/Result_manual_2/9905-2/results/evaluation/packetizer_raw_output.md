# Errata Reports

Total reports: 3

---

## Report 1: 9905-2-1

**Label:** Conflicting Operational Guidance for SHA‑1‐based DNSSEC Algorithms (RSASHA1 & RSASHA1‐NSEC3‐SHA1)

**Bug Type:** Inconsistency

**Explanation:**

The RFC simultaneously requires implementations to support SHA‑1‐based algorithms while mandating that operators treat them as unsupported/insecure, which conflicts with the IANA registry’s recommendation to use these algorithms for DNSSEC validation.

**Justification:**

- The text directs validating resolver implementations to continue supporting these algorithms while instructing operators to treat them as insecure if no alternative validation method is available.
- IANA registry entries list ‘Use for DNSSEC Validation:  RECOMMENDED’ and ‘Implement for DNSSEC Validation:  MUST’, which contradicts Section 2’s mandate to treat these algorithms as unsupported for security.
- This normative conflict may lead to divergent resolver behavior and inconsistent security state outcomes.

**Evidence Snippets:**

- **E1:**

  Validating resolver implementations ([RFC9499], Section 10) MUST continue to support validation using these algorithms...

- **E2:**

  Operators of validating resolvers MUST treat DNSSEC signing algorithms RSASHA1 and RSASHA1‑NSEC3‑SHA1 as unsupported, rendering responses insecure if they cannot be validated by other supported signing algorithms.

- **E3:**

  IANA tables (for SHA‑1 digest and RSASHA1 / RSASHA1‑NSEC3‑SHA1):
          - "Use for DNSSEC Validation:  RECOMMENDED"
          - "Implement for DNSSEC Validation:  MUST"

- **E4:**

  Operators should take care when deploying software packages and operating systems that may have already removed support for the SHA-1 algorithm.  In these situations, software may need to be manually built and deployed by an operator to continue supporting the required levels indicated by the 'Use for DNSSEC Validation' and 'Implement for DNSSEC Validation' columns, which this document is not changing.

**Evidence Summary:**

- (E1) The RFC mandates continued implementation support for SHA‑1–based algorithms.
- (E2) Operators are told to treat these algorithms as unsupported and to render responses insecure.
- (E3) IANA registry entries still recommend using these algorithms for validation.
- (E4) Operational considerations acknowledge that support may need to be manually maintained.

**Fix Direction:**

Clarify and align the normative guidance: either update the IANA registry recommendations to reflect the requirement to treat these algorithms as unsupported/insecure, or explicitly state that the 'RECOMMENDED' status applies only to implementation and not to operational security classification.

**Severity:** High
  *Basis:* The conflicting instructions can lead to divergent DNSSEC validation outcomes and undermine the intended security deprecation of SHA‑1–based algorithms.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: Issue-1
- ActorDirectionality: NewIssue-1
- Scope: Issue-1
- Deontic: Issue-1
- CrossRFC: Issue-1
- Boundary: Finding-1

---

## Report 2: 9905-2-2

**Label:** Underspecified Definition of 'Accepted/Supported' Cryptographic Algorithms

**Bug Type:** Underspecification

**Explanation:**

The RFC uses terms like 'accepted cryptographic algorithms' and 'supported signing algorithms' without providing a clear definition or mapping to the formal IANA registry categories, leading to ambiguity in determining when a zone should be regarded as insecure.

**Justification:**

- Section 2 refers to DS records being treated as insecure if no DS records of 'accepted cryptographic algorithms' are available, but does not define what constitutes an accepted algorithm.
- Similarly, the term 'supported signing algorithms' is used to decide validation outcomes without clear cross‐referencing to IANA definitions.
- This lack of explicit definition can result in inconsistent implementation and interpretation among different resolvers.

**Evidence Snippets:**

- **E1:**

  Operators of validating resolvers MUST treat RSASHA1 and RSASHA1‑NSEC3‑SHA1 DS records as insecure.  If no other DS records of accepted cryptographic algorithms are available, the DNS records below the delegation point MUST be treated as insecure.

- **E2:**

  …Operators of validating resolvers MUST treat DNSSEC signing algorithms RSASHA1 and RSASHA1‑NSEC3‑SHA1 as unsupported, rendering responses insecure if they cannot be validated by other supported signing algorithms.

- **E3:**

  Section 5, IANA tables: introduce formal categories “Use for DNSSEC Signing/Validation” and “Implement for DNSSEC Signing/Validation” for digest and algorithm values, but these terms are not directly tied to “accepted cryptographic algorithms” or “supported signing algorithms” in section 2.

**Evidence Summary:**

- (E1) The RFC mandates insecure treatment if no DS records of accepted cryptographic algorithms are present.
- (E2) It similarly instructs that signing algorithms be treated as unsupported without defining 'supported'.
- (E3) The formal IANA categories are not explicitly linked to these informal terms.

**Fix Direction:**

Provide explicit definitions or cross-references linking 'accepted cryptographic algorithms' and 'supported signing algorithms' to the formal IANA registry categories to ensure consistent interpretation.

**Severity:** Medium
  *Basis:* Ambiguity in key terminology may lead to inconsistent security state determinations among different validating resolvers, though it is primarily a configuration and policy issue.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2

---

## Report 3: 9905-2-3

**Label:** Unclear Long‑Term Status and SIG(0) Handling for SHA‑1‑Based Algorithms

**Bug Type:** Ambiguity

**Explanation:**

The RFC does not clarify the long‑term deprecation strategy for SHA‑1–based validation support or its applicability to SIG(0)/transaction security, leaving future operator actions and interpretations uncertain.

**Justification:**

- The normative text mandates that implementations MUST continue to support validation for transitional purposes but provides no explicit sunset mechanism.
- There is no discussion of how RSASHA1/RSASHA1‑NSEC3‑SHA1 should be handled in the context of SIG(0), despite IANA indicating they remain usable for transaction security.
- This lack of explicit guidance could lead to divergent long‑term practices in DNSSEC deployment.

**Evidence Snippets:**

- **E1:**

  The intended long‑term status of SHA‑1‑based validation support is unclear. The phrase “MUST continue to support validation… as they are diminishing in use but still actively in use for some domains as of this publication” suggests a transitional intent, but the normative text has no explicit sunset or mechanism to update the IANA “Use for DNSSEC Validation” status when operational practice changes.

- **E2:**

  The document does not say anything about RSASHA1/RSASHA1‑NSEC3‑SHA1 when used solely for SIG(0)/transaction security, even though the IANA table (“Trans. Sec.: Y”) implies they remain usable there.

**Evidence Summary:**

- (E1) The RFC implies a transitional approach to SHA‑1 support without defining a sunset mechanism.
- (E2) It is silent on the use of these algorithms for SIG(0)/transaction security despite IANA indications.

**Fix Direction:**

Include explicit language regarding the eventual deprecation of SHA‑1–based validation support and clarify whether and how these algorithms should be used for SIG(0)/transaction security.

**Severity:** Medium
  *Basis:* The absence of clear long‑term policy may lead to inconsistent practices and uncertainty in future deployments, although it does not create an immediate protocol failure.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: ResidualUncertainties

---
