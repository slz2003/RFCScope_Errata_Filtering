# Errata Reports

Total reports: 3

---

## Report 1: 9829-3-1

**Label:** Ambiguous Definition of 'Issuer’s Current Manifest' and CRL Selection

**Bug Type:** Underspecification/Inconsistency

**Explanation:**

The specification does not clearly define which manifest (and thus which CRL) is considered 'current' in cases such as after the nextUpdate time, during repository update races, or when multiple candidate manifests are present.

**Justification:**

- Temporal analysis (T1) questions whether an expired-but-cached manifest remains 'current' for validation.
- Scope and Causal analyses note that the term 'issuer’s current manifest' is introduced without a definitive RP‐side algorithm, leading to potential divergent interpretations.
- Boundary analysis highlights that when no unique current CRL (via manifest plus CRLDP) can be determined, it is ambiguous how the RP should proceed.

**Evidence Snippets:**

- **E1:**

  RFC 9829’s update to the path-validation rule (step 5) says that revocation status must be checked against “the issuer’s current CRL, as identified by the issuer’s current manifest and the CRLDP of the certificate,” and Section 2 adds that “a resource certificate cannot be validated without consulting the current manifest of the certificate’s issuer.”

- **E2:**

  Scope Analysis: The term “issuer’s current manifest” is introduced in RFC 9829’s update but is not explicitly defined, especially in cases with multiple manifests or repository inconsistencies.

- **E3:**

  Boundary Analysis: … when no unique 'issuer’s current CRL' can be determined via manifest + CRLDP, the updated step 5 depends on such a CRL existing, but does not say what the RP MUST or SHOULD do when it cannot identify one.

**Evidence Summary:**

- (E1) Describes the reliance on an ambiguously defined 'current' manifest for CRL selection.
- (E2) Points out that the specification lacks precise guidance for choosing the current manifest in edge cases.
- (E3) Highlights that ambiguous conditions (e.g., missing or multiple manifests) are not normatively resolved.

**Fix Direction:**

Add explicit normative text defining the RP algorithm for selecting the 'issuer’s current manifest' and associated CRL, including guidance for handling stale, missing, or ambiguous manifests (for example, by reference to RFC 9286’s RP behavior rules).

**Severity:** Low
  *Basis:* The ambiguity affects edge-case scenarios and may lead to inconsistent revocation decisions across implementations, though it is unlikely to cause direct security compromise.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ScopeExpert: Issue-1
- CausalExpert: Issue 2
- BoundaryExpert: Finding-2

---

## Report 2: 9829-3-2

**Label:** Underspecified RP Behavior on CRL Number Extension Check Failures

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that RPs check the CRL Number extension for being non-critical and within a defined numerical range but does not specify the consequences if these checks fail.

**Justification:**

- Causal analysis explicitly notes that the text does not state what action an RP must take if the CRL Number check fails.
- Deontic analysis raises concerns that divergent interpretations (rejecting vs. ignoring a malformed extension) can lead to inconsistent validation outcomes.
- Boundary analysis further emphasizes that failure modes (such as an absent, critical, or out-of-range CRL Number extension) are not normatively resolved.

**Evidence Snippets:**

- **E1:**

  However, the text never says what to do if step 3 fails.

- **E2:**

  The new text in RFC 9829 requires that “RPs MUST ignore the CRL Number extension except for checking that it is marked as non-critical and contains a non-negative integer less than or equal to 2^159-1.”

- **E3:**

  Deontic Analysis: RP behavior on malformed or out‑of‑profile CRL Number is not specified.

- **E4:**

  Boundary Analysis: … the text does not say what an RP MUST or SHOULD do if any of these [CRL Number check] failures occur.

**Evidence Summary:**

- (E1) Points out the absence of guidance on handling CRL Number check failures.
- (E2) Shows that although a check is mandated, no outcome is defined if it fails.
- (E3) Emphasizes that the deontic analysis identifies this as an underspecification.
- (E4) Highlights potential divergent RP behaviors in response to check failures.

**Fix Direction:**

Clarify in the specification that if the CRL Number extension fails the mandated check, the RP MUST treat the CRL as invalid for RPKI validation (or define an alternative unified behavior).

**Severity:** Medium
  *Basis:* Differing RP responses to CRL Number check failures could lead to inconsistent revocation decisions, impacting interoperability and security posture.

**Confidence:** High

**Experts mentioning this issue:**

- CausalExpert: Issue 1
- DeonticExpert: Issue-1
- BoundaryExpert: Finding-1

---

## Report 3: 9829-3-3

**Label:** Off-by-One Numeric Bound Inconsistency for CRL Number

**Bug Type:** Inconsistency

**Explanation:**

RFC 9829 requires that CRL Number values be less than or equal to 2^159-1, which is slightly more restrictive than the 20-octet (up to 160-bit) range allowed by RFC 5280 and RFC 9286, risking rejection of otherwise valid CRL Numbers.

**Justification:**

- CrossRFC analysis indicates that the 2^159-1 limit permits only 159 bits, whereas RFC 5280 and RFC 9286 allow values encodable in 20 octets (up to 160 bits).
- This discrepancy could cause interoperability issues where a CRL Number valid under RFC 5280 is rejected under RFC 9829's check.

**Evidence Snippets:**

- **E1:**

  RFC 9829’s updated Section 5 text for RFC 6487 tells RPs to “ignore the CRL Number extension except for checking that it is marked as non-critical and contains a non-negative integer less than or equal to 2^159-1.”

- **E2:**

  CrossRFC Report: … the range (≤ 2^159 − 1) allows at most 159 bits, which is stricter than the 20‑octet (up to 160‑bit) limit that RFC 5280 and RFC 9286 explicitly allow.

**Evidence Summary:**

- (E1) Specifies the numeric bound imposed by RFC 9829.
- (E2) Highlights the discrepancy between this bound and the 20-octet allowance in RFC 5280 and RFC 9286.

**Fix Direction:**

Reconcile the numeric boundary in RFC 9829 with the 20-octet (160-bit) limit specified in RFC 5280 and RFC 9286, either by adjusting the upper bound to 2^160-1 or by providing a clear normative explanation for the difference.

**Severity:** Low
  *Basis:* Although this numeric mismatch may lead to some CRL Numbers being rejected by RFC 9829-conformant implementations, it poses mainly an interoperability issue rather than a direct security risk.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFCExpert: Issue-1

---
