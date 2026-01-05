# Errata Reports

Total reports: 1

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