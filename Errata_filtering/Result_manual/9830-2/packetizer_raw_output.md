# Errata Reports

Total reports: 1

---


## Report 3: 9830-2-3

**Label:** Vacuous Condition in SRv6 SID and Endpoint Behavior/Structure Encoding

**Bug Type:** Inconsistency

**Explanation:**

The specification includes a normative clause that the SRv6 Endpoint Behavior and SID Structure MUST NOT be included when the SRv6 SID has not been included, yet the encoding mandates that a 16‑octet SRv6 SID is always present, rendering the condition unreachable and confusing.

**Justification:**

- Both the SRv6 Binding SID sub-TLV and the Segment Type B sub-TLV always include a fixed 16‑octet SRv6 SID field, making the condition ‘when the SRv6 SID has not been included’ impossible to satisfy.
- This contradictory clause can mislead implementers into erroneously thinking there exists a variant of the encoding where the SID field may be omitted.

**Evidence Snippets:**

- **E1:**

  Section 2.4.3, SRv6 Binding SID sub-TLV: … SRv6 Binding SID:  Contains a 16-octet SRv6 SID. The value 0 MAY be used when the controller wants to indicate the desired SRv6 Endpoint Behavior, SID Structure, or flags without specifying the BSID. … SRv6 Endpoint Behavior and SID Structure MUST NOT be included when the SRv6 SID has not been included.

- **E2:**

  Section 2.4.4.2.2, Segment Type B sub-TLV: … The SRv6 Endpoint Behavior and SID Structure MUST NOT be included when the SRv6 SID has not been included.

**Evidence Summary:**

- (E1) Shows the clause in the SRv6 Binding SID sub-TLV that prohibits including the behavior/structure block when the SID is not present.
- (E2) Repeats the same directive in the Segment Type B sub-TLV, despite the fact that the SID field is always present.

**Fix Direction:**

Rephrase or remove the contradictory clause so that the presence of the SRv6 Endpoint Behavior and SID Structure is determined solely by the sub‑TLV Length (i.e., 26 octets versus 18 octets), acknowledging that the 16‑octet SRv6 SID field is always present.


**Severity:** Low
  *Basis:* This issue is mainly editorial and does not affect interoperability, but it poses a risk of confusing implementers regarding the encoding semantics.

**Confidence:** High

---