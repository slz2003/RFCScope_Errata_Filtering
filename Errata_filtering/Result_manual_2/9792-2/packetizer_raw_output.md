# Errata Reports

Total reports: 1

---


## Report 2: 9792-2-2

**Label:** OSPFv3 IANA Registry Misreference to OSPFv2 Prefix Extended Flags sub‐TLV

**Bug Type:** Inconsistency

**Explanation:**

The OSPFv3 IANA registry text incorrectly refers to the OSPFv2 Prefix Extended Flags sub‐TLV, creating confusion about which sub‐TLV the registry is meant to define.

**Justification:**

- Section 4.2.2 of RFC 9792, under the heading 'OSPFv3 Prefix Extended Flags Registry', states that the registry defines bits in the OSPFv2 Prefix Extended Flags sub‐TLV.
- This is inconsistent with Section 4.1.2 which correctly refers to the OSPFv2 sub‐TLV and contradicts the clear separation intended between OSPFv2 and OSPFv3 parameters.
- Experts note that this is likely a copy‐and‐paste error that may mislead implementers or IANA consumers.

**Evidence Snippets:**

- **E1:**

  Section 4.2.2, heading: “OSPFv3 Prefix Extended Flags Registry”. Immediately following text: “The registry defines the bits in the Prefix Extended Flags field in the OSPFv2 Prefix Extended Flags sub-TLV as specified in Section 2.”

- **E2:**

  In Section 4.1.2, the OSPFv2 registry is correctly described as defining the bits in the Prefix Extended Flags field in the OSPFv2 Prefix Extended Flags sub‐TLV.

**Evidence Summary:**

- (E1) The OSPFv3 registry text incorrectly names the OSPFv2 sub‐TLV.
- (E2) The correct OSPFv2 registry text contrasts with the misreference in the OSPFv3 section.

**Fix Direction:**

Replace the reference to 'OSPFv2 Prefix Extended Flags sub-TLV' with 'OSPFv3 Prefix Extended Flags sub-TLV' in Section 4.2.2.

**Severity:** Low
  *Basis:* The misreference is an editorial error that may cause confusion but does not impact on‐the‐wire behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- Deontic Expert: Issue-2
- Structural Expert: Issue-3
- Terminology Expert: Issue-1
- CrossRFC Expert: Issue-2

---