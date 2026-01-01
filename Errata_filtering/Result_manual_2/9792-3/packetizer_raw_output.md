# Errata Reports

Total reports: 1

---

## Report 1: 9792-3-1

**Label:** No apparent inconsistency or underspecification in backward-compatibility text (Section 3)

**Bug Type:** None

**Explanation:**

The backward-compatibility text in Section 3 is consistent with existing TLV processing rules in RFC 3630 and RFC 8362, and no specification issues are evident.

**Justification:**

- The excerpt summary explains that implementations ignore the new sub-TLV per the generic rules, indicating alignment with the referenced RFCs (E1).
- The candidate analysis explicitly notes that Section 3’s backward-compatibility claim aligns with the generic 'ignore unknown TLVs/sub-TLVs' rule (E2).

**Evidence Snippets:**

- **E1:**

  Section 3 states that the new OSPFv2/OSPFv3 Prefix Extended Flags sub-TLV is backward compatible because implementations that do not recognize it will ignore it per the generic TLV processing rules in RFC 3630 (OSPFv2) and RFC 8362 (OSPFv3 Extended LSAs).

- **E2:**

  Section 3’s backward-compatibility claim aligns with the generic “ignore unknown TLVs/sub-TLVs” rule...

**Evidence Summary:**

- (E1) Section 3 asserts that the new sub-TLV is designed to be backward compatible by relying on RFC 3630 and RFC 8362 guidelines.
- (E2) The analysis confirms that the backward-compatibility claim adheres to the general rule of ignoring unknown TLVs/sub-TLVs.

**Severity:** Low
  *Basis:* Each evaluated dimension (temporal, causal, structural, crossRFC, etc.) is rated as low, indicating minimal impact with behavior aligning to established RFCs.

**Confidence:** High

**Experts mentioning this issue:**

- Router Analysis: Issue 1

---
