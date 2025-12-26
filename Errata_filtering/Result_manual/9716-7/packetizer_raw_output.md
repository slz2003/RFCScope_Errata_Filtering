# Errata Reports

Total reports: 2

---

## Report 1: 9716-7-1

**Label:** Ambiguity in sub‐TLVs 46–48 scope across TLVs 1/16 vs TLV 21 (RFC 9716 vs RFC 7110)

**Bug Type:** Both

**Explanation:**

RFC 9716 allocates sub‐TLVs 46–48 from a shared registry for TLVs 1, 16, and 21, but restricts their operational use exclusively to TLV 21, creating ambiguity regarding whether these sub‐TLVs should be considered valid in TLVs 1 and 16.

**Justification:**

- RFC 7110 Section 7.1 mandates that sub‐TLV assignments be the same across the Target FEC Stack TLV and the Reply Path TLV, while RFC 9716 Section 4 specifies that these sub‐TLVs apply only when present in TLV 21, causing a scope mismatch. (E1, E2)
- The IANA assignment in RFC 9716 Section 7.1 and the TLV-scoped semantics per RFC 8029 (E3, E4) leave it unclear if sub‐TLVs 46–48 are intended to be operable in TLVs 1 and 16 or strictly reserved for TLV 21.

**Evidence Snippets:**

- **E1:**

  RFC 7110 Section 7.1: “The sub-TLV space and assignments for the Reply Path TLV will be the same as that for the Target FEC Stack TLV. Sub-types for the Target FEC Stack TLV and the Reply Path TLV MUST be kept the same. Any new sub-type added to the Target FEC Stack TLV MUST apply to the Reply Path TLV as well.”

- **E2:**

  RFC 9716 Section 4: “The below types of Segment sub-TLVs apply to the Reply Path TLV. The code points for the sub-TLVs are taken from the IANA registry common to TLVs 1, 16, and 21. This document defines the usage and processing … when they appear in TLV 21 (Reply Path TLV). If these sub-TLVs appear in TLVs 1 or 16, appropriate error codes MUST be returned as defined in [RFC8029].”

- **E3:**

  RFC 9716 Section 7.1: “IANA has assigned three new sub-TLVs from the ‘Sub-TLVs for TLV Types 1, 16, and 21’ registry … Sub-Type 46/47/48.” (Table 3).

- **E4:**

  RFC 8029 Section 6.2.2: “Note that the meaning of a sub-TLV is scoped by the TLV. The number spaces for the sub-TLVs of various TLVs are independent.”

**Evidence Summary:**

- (E1) Shows RFC 7110’s requirement for identical sub‐TLV assignments across TLVs.
- (E2) Indicates that RFC 9716 restricts the operational use of these sub‐TLVs to TLV 21 and mandates errors in TLVs 1 or 16.
- (E3) Provides the IANA allocation from a shared registry.
- (E4) Reminds that sub‐TLV semantics are scoped by the TLV, underscoring the ambiguity.

**Fix Direction:**

Clarify the IANA and processing text in RFC 9716 to explicitly state that sub‐TLVs 46–48 are valid only when used in the Reply Path TLV (TLV 21) and must be treated as invalid in TLVs 1 and 16.


**Severity:** Medium
  *Basis:* The ambiguous scoping may lead to diverging interpretations among implementers, potentially causing interoperability or error-handling inconsistencies.

**Confidence:** Low

---

## Report 2: 9716-7-2

**Label:** Inconsistent naming of the Reply Path Return Code field

**Bug Type:** Editorial

**Explanation:**

There is an inconsistency in naming where the field is sometimes referred to as 'Reply Path TLV Return Code' instead of the consistent 'Reply Path Return Code', which may lead to confusion.

**Justification:**

- An instance in the text uses the term 'Reply Path TLV Return Code' (E5), diverging from RFC 7110 which defines the field as 'Reply Path Return Code' and potentially leading to misinterpretation.

**Evidence Snippets:**

- **E5:**

  Internal nodes or non-domain border nodes might not set the Reply Path TLV Return Code to 0x0006… (note the extra “TLV” in the field name).

**Evidence Summary:**

- (E5) Highlights the extra use of 'TLV' in the field name, which is inconsistent with the standard naming in RFC 7110.

**Fix Direction:**

Replace 'Reply Path TLV Return Code' with 'Reply Path Return Code' in Section 5.5 to ensure naming consistency.


**Severity:** Low
  *Basis:* This issue is editorial in nature and does not affect the protocol semantics or interoperability.

**Confidence:** High

---
