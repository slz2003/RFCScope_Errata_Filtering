# Errata Reports

Total reports: 1

---


## Report 6: 9857-5-6

**Label:** Inconsistent Terminology: SR Binding SID Referred to as TLV and Sub-TLV

**Bug Type:** Inconsistency

**Explanation:**

The SR Binding SID (code point 1201) is inconsistently described as both a top-level TLV and as a sub‑TLV, which may confuse implementers as to its correct placement in the BGP‑LS Attribute.

**Justification:**

- Section 5.1 describes the object as the “SR Binding SID TLV” with a defined format and code point.
- Later, the same object is referred to as the “SR Binding SID sub‑TLV” when discussing SRv6 deprecation, while Table 4 registers it as a top-level TLV.

**Evidence Snippets:**

- **E1:**

  Section 5.1 titles and describes “SR Binding SID TLV” (Type 1201) as: “The SR Binding Segment Identifier (BSID) is an optional TLV … The TLV has the following format… Type: 1201 … Length: Variable (valid values are 12 or 36 octets).”

- **E2:**

  The closing paragraph of Section 5.1 states: “In the case of an SRv6, the SR Binding SID sub‑TLV does not have the ability to signal the SRv6 Endpoint behavior … Therefore, the SR Binding SID sub‑TLV SHOULD NOT be used for the advertisement of an SRv6 Binding SID. Instead, the SRv6 Binding SID TLV defined in Section 5.2 SHOULD be used…”

- **E3:**

  Section 8.3 (Table 4) lists code point 1201 as “SR Binding SID” under “BGP‑LS NLRI and Attribute TLVs”, i.e., as a top-level TLV, not a sub‑TLV.

**Evidence Summary:**

- (E1) Introduces SR Binding SID as a TLV with a specific format.
- (E2) Later, refers to the same object as a sub-TLV.
- (E3) Confirms in the registry that it is a top-level TLV.

**Fix Direction:**

Revise Section 5.1 to consistently refer to code point 1201 as a top-level TLV and remove or clearly distinguish any legacy sub‑TLV terminology.


**Severity:** Low
  *Basis:* Although the inconsistency may cause confusion, it does not affect on‐wire encoding.

**Confidence:** High

---