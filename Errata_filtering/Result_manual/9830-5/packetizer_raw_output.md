# Errata Reports

Total reports: 4

---

## Report 1: 9830-5-1

**Label:** Ambiguous override of RFC 9012 sub‑TLV error handling for Tunnel Type 15 (SR Policy)

**Bug Type:** Both

**Explanation:**

RFC 9830 mandates that any error in the Tunnel Encapsulation Attribute or its TLVs/sub‑TLVs must trigger a 'treat‑as‑withdraw' action for SR Policy updates, which conflicts with RFC 9012’s rule that a malformed sub‑TLV should be treated as unrecognized and not cause withdrawal.

**Justification:**

- RFC 9830 §5 requires that any error at the attribute or TLV/sub‑TLV level causes treat‑as‑withdraw, while RFC 9012 §13 instructs that malformed sub‑TLVs be treated as unrecognized and the attribute remain valid.
- Multiple experts noted that the lack of clarification on whether RFC 9830 deliberately overrides 9012’s behavior leads to inconsistent implementation decisions.

**Evidence Snippets:**

- **E1:**

  RFC 9830 §5: “The validation of the TLVs/sub-TLVs introduced in this document and defined in their respective subsections of Section 2.4 MUST be performed to determine if they are malformed or invalid. The validation of the Tunnel Encapsulation Attribute itself and the other TLVs/sub-TLVs specified in Section 13 of [RFC9012] MUST be done as described in that document. In case of any error detected, either at the attribute or its TLV/sub-TLV level, the ‘treat‑as‑withdraw’ strategy MUST be applied. This is because an SR Policy update without a valid Tunnel Encapsulation Attribute (comprised of all valid TLVs/sub‑TLVs) is not usable.”

- **E2:**

  RFC 9012 §13: “In general, if a TLV contains a sub‑TLV that is malformed, the sub‑TLV MUST be treated as if it were an unrecognized sub‑TLV. … However, the Tunnel TLV containing them MUST NOT be considered to be malformed, and all the sub‑TLVs MUST be propagated if the route carrying the Tunnel Encapsulation attribute is propagated.”

- **E3:**

  RFC 9830 §1: “This document does not modify or supersede the usage of the Tunnel Encapsulation Attribute for existing AFI/SAFIs as defined in [RFC9012]. Details regarding the processing of the Tunnel Encapsulation Attribute for the SR Policy SAFI are provided in Sections 2.2 and 2.3.”

**Evidence Summary:**

- (E1) shows RFC 9830’s mandate for fatal treatment of any TLV/sub‑TLV error.
- (E2) contrasts this with RFC 9012’s requirement to treat malformed sub‑TLVs as unrecognized.
- (E3) reiterates that RFC 9830 does not claim to modify RFC 9012 outside SR Policy specifics.

**Fix Direction:**

Clarify explicitly in RFC 9830 that for Tunnel Type 15 with SR Policy SAFI, any syntactic error in TLVs/sub‑TLVs is fatal and overrides RFC 9012’s recoverable treatment, or update the text to clearly distinguish between errors that mandate treat‑as‑withdraw and those that do not.


**Severity:** Medium
  *Basis:* This ambiguity may lead to divergent implementations and interoperability issues by causing unintended withdrawals of SR Policy updates.

**Confidence:** Medium

---

## Report 2: 9830-5-2

**Label:** Unclear boundary between errors and unrecognized/unsupported sub‑TLVs affecting usability vs. withdrawal

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly define whether unrecognized or unsupported SR Policy sub‑TLVs should be handled purely as non‐fatal usability issues or treated as errors that force a treat‑as‑withdraw action.

**Justification:**

- RFC 9830 §4.2.2 indicates that unrecognized or unsupported sub‑TLVs should render the update not usable (with optional override), while Section 5 mandates treat‑as‑withdraw for any error detected.
- This mixed messaging can lead to inconsistent behavior between implementations.

**Evidence Snippets:**

- **E1:**

  RFC 9830 §4.2.2: “When the SR Policy tunnel type includes any sub-TLV that is unrecognized or unsupported, the update SHOULD NOT be considered usable. An implementation MAY provide an option for ignoring unsupported sub-TLVs.”

- **E2:**

  RFC 9830 §5: “In case of any error detected, either at the attribute or its TLV/sub‑TLV level, the ‘treat‑as‑withdraw’ strategy MUST be applied. This is because an SR Policy update without a valid Tunnel Encapsulation Attribute (comprised of all valid TLVs/sub‑TLVs) is not usable.”

**Evidence Summary:**

- (E1) defines a usability guideline for unsupported sub‑TLVs, while (E2) enforces fatal error handling for any TLV/sub‑TLV error.
- The contrast creates ambiguity about which conditions trigger route withdrawal.

**Fix Direction:**

Clarify in the specification which conditions for unrecognized or unsupported sub‑TLVs fall under treat‑as‑withdraw and which only affect local usability without triggering withdrawal.


**Severity:** Medium
  *Basis:* Ambiguities in error terminology may lead to divergent interpretations and inconsistent handling of SR Policy routes across different implementations.

**Confidence:** Medium

---

## Report 3: 9830-5-3

**Label:** Segment List sub-TLV length inconsistency with RFC 9012 semantics

**Bug Type:** Inconsistency

**Explanation:**

The Segment List sub-TLV length field in RFC 9830 appears to exclude the RESERVED octet from its length calculation, contradicting RFC 9012, which requires that the length include all octets of the sub-TLV Value field.

**Justification:**

- RFC 9012 mandates that the Length field represents the total number of octets in the sub-TLV Value field.
- RFC 9830’s description omits the RESERVED field from the length count, resulting in an off-by-one error and potential parsing misalignment.

**Evidence Snippets:**

- **E1:**

  RFC 9012 defines sub-TLVs generically: the Sub-TLV Length field is “the total number of octets of the Sub-TLV Value field” and the Value is all bytes after the Length field. For types 128–255, the Length is 2 octets and has the same semantics.

- **E2:**

  RFC 9830 defines the Segment List sub-TLV as:
- Format (Figure 9):
  - Type (1 octet)
  - Length (2 octets)
  - RESERVED (1 octet)
  - “sub-TLVs” (variable)
- Length description: “The total length (not including the Type and Length fields) of the sub-TLVs encoded within the Segment List sub-TLV in terms of octets.”

**Evidence Summary:**

- (E1) establishes that the Length field should count all value octets per RFC 9012.
- (E2) shows that RFC 9830’s format omits the RESERVED octet from the length count, leading to inconsistencies.

**Fix Direction:**

Update the definition of the Segment List sub-TLV length field to include the RESERVED octet in the count or adjust the encoding to align with RFC 9012 semantics.


**Severity:** High
  *Basis:* This off-by-one error can lead to misalignment in sub-TLV parsing and subsequent interpretation errors, resulting in route withdrawal and interoperability failures.

**Confidence:** High

---

## Report 4: 9830-5-4

**Label:** Inconsistent handling of partially malformed SR Policy NLRIs in multi-NLRI updates

**Bug Type:** Inconsistency

**Explanation:**

There is a conflict between RFC 9830, which permits processing of valid SR Policy NLRIs while skipping malformed ones, and RFC 7606, which mandates that an MP_REACH/MP_UNREACH attribute is considered incorrect if any NLRI has an inconsistent length.

**Justification:**

- RFC 9830 suggests that if a router can skip a malformed NLRI and continue processing, it should treat only that NLRI as withdrawn.
- RFC 7606 requires the entire MP_REACH/MP_UNREACH attribute to be treated as incorrect when any NLRI's length is inconsistent, leading to potential divergent behaviors.

**Evidence Snippets:**

- **E1:**

  RFC 9830 Section 5: “When the error determined allows for the router to skip the malformed NLRI(s) and continue the processing of the rest of the BGP UPDATE message, then it MUST handle such malformed NLRIs as 'treat‑as‑withdraw'.”

- **E2:**

  RFC 7606 Section 5.3: “the MP_REACH_NLRI or MP_UNREACH_NLRI attribute of an update SHALL be considered to be incorrect if any of the included NLRI lengths are inconsistent with the given AFI/SAFI…”

**Evidence Summary:**

- (E1) implies selective handling of malformed NLRIs, while (E2) mandates that any inconsistency invalidates the entire attribute.

**Fix Direction:**

Harmonize the treatment of partially malformed NLRI within multi-NLRI updates by explicitly stating whether only the malformed NLRIs should trigger treat‑as‑withdraw or if the entire attribute must be rejected.




**Severity:** High
  *Basis:* Divergent interpretations can lead to inconsistent route processing across different BGP implementations, potentially affecting SR Policy propagation.

**Confidence:** High
