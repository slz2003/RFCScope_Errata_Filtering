# Errata Reports

Total reports: 3

---

## Report 1: 9829-2-1

**Label:** Ambiguous requirement on consulting current manifest versus cached validation

**Bug Type:** Underspecification

**Explanation:**

RFC 9829 Section 2 states that a resource certificate cannot be validated without consulting the current manifest, but it is ambiguous whether this applies to every validation (forcing a fresh manifest fetch) or only to the initial validation, given that RFC 9286 permits continued use of cached objects during fetch failures.

**Justification:**

- Temporal Expert (T1) notes that the text can be interpreted as requiring a fresh manifest each time, conflicting with RFC 9286’s allowance for using cached validation results during temporary failures.
- Scope Expert (Issue-1) and CrossRFC Expert (Issue-1) also highlight that the over‑broad statement could be misread to forbid continued use of cached state, thereby narrowing the intended operational scope.

**Evidence Snippets:**

- **E1:**

  This document clarifies that, in the RPKI, there is exactly one CRL that is appropriate and relevant for determining the revocation status of a given resource certificate. It is the unique CRL object that is simultaneously: … the target of the certificate's CRL Distribution Points extension, and … listed in the issuing CA's current manifest fileList and has a matching hash … In particular, a resource certificate cannot be validated without consulting the current manifest of the certificate's issuer.

- **E2:**

  Each RP MUST use the current manifest of a CA to control addition of listed files to the set of signed objects the RP employs for validating basic RPKI objects: certificates, ROAs, and CRLs.

**Evidence Summary:**

- (E1) Excerpt from RFC 9829 Section 2 describing the validation requirement.
- (E2) Excerpt from RFC 9286 specifying the use of the current manifest in validation.

**Fix Direction:**

Clarify that the requirement to consult the current manifest applies only during a fresh validation process after a successful manifest fetch, while allowing continued reliance on cached objects per RFC 9286 Section 6.6 during fetch failures.

**Severity:** Low
  *Basis:* The ambiguity may lead to divergent interpretations in failure scenarios, but normative caching behavior as per RFC 9286 mitigates the risk.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ScopeExpert: Issue-1
- CrossRFCExpert: Issue-1
- DeonticExpert: Issue-1

---

## Report 2: 9829-2-2

**Label:** Inconsistency in guarantee of exactly one current CRL versus potential absence during failures

**Bug Type:** Inconsistency

**Explanation:**

RFC 9829 guarantees that an RP will always identify exactly one current CRL for each CA, yet RFC 9286 defines failure states during which no current CRL is available, creating a potential misinterpretation of the guarantee.

**Justification:**

- Temporal Expert (T2) explains that while RFC 9829 claims an unqualified guarantee of one current CRL, RFC 9286 explicitly permits temporary conditions where no current CRL is available due to manifest fetch failures.
- This discrepancy may mislead implementers regarding the expected behavior during publication or network failures.

**Evidence Snippets:**

- **E1:**

  Together, these properties guarantee that RPKI RPs will always be able to unambiguously identify exactly one current CRL for each RPKI CA.

- **E2:**

  If an RP cannot retrieve a manifest … or if the manifest is not valid … an RP MUST treat this as a failed fetch; proceed to Section 6.6.

**Evidence Summary:**

- (E1) Excerpt from RFC 9829 asserting the guarantee of exactly one current CRL.
- (E2) RFC 9286 instructions describing manifest fetch failure leading to no current CRL.

**Fix Direction:**

Revise the language in RFC 9829 to specify that the guarantee of exactly one current CRL applies only under conditions of a successful manifest fetch and to acknowledge that failure intervals may result in no current CRL.

**Severity:** Low
  *Basis:* The issue is conceptual and affects interpretative clarity under failure conditions while normative behavior remains governed by RFC 9286.

**Confidence:** Medium

**Experts mentioning this issue:**

- TemporalExpert: T2

---

## Report 3: 9829-2-3

**Label:** Undefined behavior when CRL Number extension fails its syntactic check

**Bug Type:** Underspecification

**Explanation:**

The updated RFC 9829 text instructs RPs to ignore the CRL Number extension except to verify it is non-critical and within an acceptable range, but does not specify how to proceed if these syntactic checks fail.

**Justification:**

- Boundary Expert (Finding-2) points out that while the new text mandates a check on the CRL Number extension, it does not define whether the CRL should be rejected, treated as missing, or accepted despite the failure.
- This lack of defined behavior can cause inconsistent revocation checking across RPs if a CRL with a malformed CRL Number is encountered.

**Evidence Snippets:**

- **E1:**

  RFC 6487 Section 5 (old text): “An RPKI CA MUST include the two extensions, Authority Key Identifier and CRL Number, in every CRL that it issues. RPs MUST be prepared to process CRLs with these extensions. No other CRL extensions are allowed.”

- **E2:**

  RFC 9829 Section 3.1 (second change, NEW text): “An RPKI CA MUST include exactly two extensions in every CRL that it issues: an Authority Key Identifier (AKI) and a CRL Number. No other CRL extensions are allowed.  -  RPs MUST process the AKI extension.  -  RPs MUST ignore the CRL Number extension except for checking that it is marked as non-critical and contains a non-negative integer less than or equal to 2^159-1.”

**Evidence Summary:**

- (E1) Excerpt from the old RFC 6487 text regarding required CRL extensions.
- (E2) Excerpt from the updated RFC 9829 text outlining the new handling for the CRL Number extension.

**Fix Direction:**

Specify explicit error handling for cases where the CRL Number extension does not meet the syntactic requirements (e.g., defining whether the CRL should be rejected or treated as absent).

**Severity:** Medium
  *Basis:* Ambiguous handling of syntactic failures in the CRL Number extension could lead to divergent revocation decisions and interoperability issues, especially in adversarial scenarios.

**Confidence:** High

**Experts mentioning this issue:**

- BoundaryExpert: Finding-2

---
