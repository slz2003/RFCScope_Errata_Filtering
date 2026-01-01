# Errata Reports

Total reports: 2

---

## Report 1: 9829-5-1

**Label:** Overbroad replay protection claim in manifest hash in Section 5

**Bug Type:** Underspecification

**Explanation:**

RFC 9829 Section 5 overstates the security offered by the manifest hash by claiming it guarantees that the CRL is the most recent and eliminates replay vectors, whereas in reality it only binds the CRL to a given manifest without preventing replay of older, self‐consistent manifest+CRL sets.

**Justification:**

- The text states that the manifest hash 'provides a cryptographic guarantee on the Certification Authority's intent that this is the most recent CRL and removes possible replay vectors,' which merges the binding property with a broader replay protection claim (Temporal Expert T1, Scope Expert Issue-1).
- Additional analysis shows the hash only prevents substitution of a different CRL within a fixed manifest, leaving replay or rollback of an older valid manifest+CRL pair unaddressed (Causal Expert, CrossRFC Expert Issue-1, Boundary Expert Finding-1).

**Evidence Snippets:**

- **E1:**

  This document explicates that, in the RPKI, the CRL listed on the certificate issuer's current manifest is the one relevant and appropriate for determining the revocation status of a resource certificate. The hash in the manifest's fileList provides a cryptographic guarantee on the Certification Authority's intent that this is the most recent CRL and removes possible replay vectors.

- **E2:**

  Section 5 of RFC 9829 states:  “This document explicates that, in the RPKI, the CRL listed on the certificate issuer's current manifest is the one relevant and appropriate for determining the revocation status of a resource certificate. The hash in the manifest's fileList provides a cryptographic guarantee on the Certification Authority's intent that this is the most recent CRL and removes possible replay vectors.”  The broader context asserts that there is “exactly one CRL that is appropriate” and that it is “listed in the issuing CA's current manifest fileList and has a matching hash” (Section 2).

**Evidence Summary:**

- (E1) Describes the claim that the manifest hash guarantees the 'most recent CRL' and removes replay vectors.
- (E2) Provides contextual evidence from Section 5 and Section 2 that highlights the overbroad scope of the guarantee.

**Fix Direction:**

Revise the text in Section 5 to clarify that the manifest hash solely binds the CRL to the manifest, and that prevention of replay or rollback of older manifest+CRL sets requires additional freshness and manifestNumber checks as defined elsewhere.

**Severity:** Medium
  *Basis:* This mischaracterization could mislead implementers into omitting necessary replay protection measures, potentially compromising the freshness of revocation data in adversarial settings.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1
- Scope Expert: Issue-1
- Causal Expert: CA1
- CrossRFC Expert: Issue-1
- Boundary Expert: Finding-1

---

## Report 2: 9829-5-2

**Label:** Undefined RP behavior for malformed CRL Number extension

**Bug Type:** Underspecification

**Explanation:**

While the specification requires CRLs to include a non-critical CRL Number within a specific range, it does not define the expected behavior of a relying party (RP) if the CRL Number extension is malformed or does not meet these criteria.

**Justification:**

- The text mandates that each CRL include an Authority Key Identifier and a CRL Number, with RPs required to ignore the CRL Number except to verify it is non-critical and its value is a non-negative integer ≤ 2^159-1 (Boundary Expert Finding-2).
- This omission leaves RP behavior undefined in cases where the CRL Number extension is missing, marked critical, or fails the numeric constraints, which can lead to inconsistent revocation handling.

**Evidence Snippets:**

- **E1:**

  Section 3.1 (second change to RFC 6487 Section 5) says:  “An RPKI CA MUST include exactly two extensions in every CRL that it issues: an Authority Key Identifier (AKI) and a CRL Number. No other CRL extensions are allowed.  -  RPs MUST process the AKI extension.  -  RPs MUST ignore the CRL Number extension except for checking that it is marked as non-critical and contains a non-negative integer less than or equal to 2^159-1.”  Step 5 of the path validation (Section 3.2) refers to “the issuer's current CRL” and adds “The CRL is itself valid…” but does not define validity beyond signature and standard checks.

**Evidence Summary:**

- (E1) Specifies the requirements for the CRL Number extension and highlights that the normative behavior for RPs when this extension fails the checks is not defined.

**Fix Direction:**

Define explicit RP behavior for handling cases when the CRL Number extension is malformed, missing, marked critical, or falls outside the allowed numerical range.

**Severity:** Medium
  *Basis:* Ambiguity in how RPs should react to non-conforming CRL Number extensions can lead to divergent validation behaviors, undermining consistency and potentially security across implementations.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary Expert: Finding-2

---
