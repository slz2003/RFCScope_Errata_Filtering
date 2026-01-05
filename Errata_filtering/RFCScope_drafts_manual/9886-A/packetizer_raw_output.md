# Errata Reports

Total reports: 2

---


## Report 3: 9886-A-3

**Label:** Ambiguous Temporal Scope for DET DNS Resolution

**Bug Type:** Inconsistency

**Explanation:**

Section 4 imposes an unconditional MUST that DET reverse names resolve to an HHIT RRType, while Section 7.2 recommends a 'just-in-time' publication that may result in no RRTypes being published until needed, creating conflicting operational expectations.

**Justification:**

- Section 4 states that DETs MUST resolve to an HHIT RRType unconditionally (and BRID MUST be present for UAS RID).
- Section 7.2 recommends withholding RRTypes until they are required by other parties, allowing for privacy-preserving non-publication before activation.
- This conflict creates ambiguity in deployment, where implementations may enforce the MUST and reject zones following the privacy recommendation.

**Evidence Snippets:**

- **E1:**

  Section 4: “DETs, being IPv6 addresses, are to be under ip6.arpa. (nibble reversed …) and MUST resolve to an HHIT RRType. … For UAS RID, the BRID RRType MUST be present…” (Section 4).

- **E2:**

  Section 7.2: “When practical, it is RECOMMENDED that no RRTypes under a DET’s specific domain name be published unless and until it is required for use by other parties. Such action would cause at least the HHIT RRType to not be in the DNS, protecting the public key in the certificate from being exposed before its needed.” (Section 7.2).

**Evidence Summary:**

- (E1) Section 4 enforces an unconditional MUST for DET DNS resolution.
- (E2) Section 7.2 recommends delaying RRType publication for privacy, creating a temporal conflict.

**Fix Direction:**

Clarify the scope of the MUST requirement in Section 4 by restricting it to active DETs or otherwise explicitly reconciling it with the just-in-time publication recommendation in Section 7.2.

**Severity:** High
  *Basis:* The contradictory requirements create operational ambiguity that can lead to inconsistent deployment and potential security/privacy issues.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- Deontic Expert: Issue-2

---


## Report 5: 9886-A-5

**Label:** BRID UAS ID Length Mismatch (20-byte Requirement vs 17-byte Example)

**Bug Type:** Inconsistency

**Explanation:**

The BRID RR CDDL requires that the UAS ID be exactly 20 bytes long, but the Appendix A example specifies a UAS ID that is only 17 bytes, violating the size constraint.

**Justification:**

- According to the CDDL (Figure 5), the uas_id field in the uas-id-grp must be a byte string of exactly 20 bytes (bstr .size(20)).
- The BRID CBOR example in Appendix A shows uas_ids with a UAS ID given as h'012001003FFE000A05130824699A4BC6B2', which is 34 hex digits corresponding to only 17 bytes.
- This size discrepancy means that a decoder enforcing the CDDL will reject the example as malformed.

**Evidence Snippets:**

- **E1:**

  The BRID CDDL defines uas-id-grp as [ id_type: &uas-id-types, uas_id: bstr .size(20) ] (Figure 5), so uas_id must be exactly 20 bytes.

- **E2:**

  Decoded BRID CBOR example (Figure 21): 1: [4, h'012001003FFE000A05130824699A4BC6B2'] – the hex string here has 34 hex digits, equating to 17 bytes.

**Evidence Summary:**

- (E1) The CDDL mandates a 20-byte uas_id.
- (E2) The provided example contains only a 17-byte uas_id.

**Fix Direction:**

Correct the BRID example to include a 20-byte UAS ID (e.g., by providing a hex string with 40 hex digits) or adjust the CDDL if the intended length is different.

**Severity:** High
  *Basis:* The incorrect byte length directly violates the schema, leading to decoding failures and incompatibility with ASTM F3411-based implementations.

**Confidence:** High

**Experts mentioning this issue:**

- Quantitative Expert: Issue-3
- Structural Expert: Issue-2
- Boundary Expert: Finding-2

---