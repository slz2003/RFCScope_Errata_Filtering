# Errata Reports

Total reports: 4

---

## Report 1: 9830-2-1

**Label:** Distinguisher Misrepresentation in SR Policy NLRI

**Bug Type:** Inconsistency

**Explanation:**

The Distinguisher field is defined as uniquely identifying the SR Policy in the context of <Color, Endpoint>, which contradicts RFC 9256 where it acts only as the candidate path discriminator with a scope limited by Protocol‑Origin, potentially confusing implementers about its intended uniqueness.

**Justification:**

- RFC 9830 Section 2.1 describes the Distinguisher as uniquely identifying the SR Policy, yet later states it is used as the candidate path discriminator per RFC 9256 (E1, E2).
- This mismatch may lead some implementations to enforce overly strict uniqueness criteria rather than the intended per‑candidate path identification.

**Evidence Snippets:**

- **E1:**

  RFC 9830 Section 2.1:  Distinguisher:  4-octet value uniquely identifying the SR Policy in the context of <Color, Endpoint> tuple.  The distinguisher has no semantic value.  It is used by the SR Policy originator to form unique NLRIs ... The distinguisher is the discriminator of the SR Policy CP as specified in Section 2.5 of [RFC9256].

- **E2:**

  RFC 9256 Section 2.5: “When signaling is via BGP SR Policy, the BGP process receiving the route provides the distinguisher … as the Discriminator.  Note that the BGP best path selection is applied before the route is supplied as a candidate path, so only a single candidate path for a given SR Policy will be seen for a given Discriminator.”

**Evidence Summary:**

- (E1) Shows RFC 9830 defining Distinguisher as uniquely identifying the SR Policy.
- (E2) Indicates its role as candidate path discriminator per RFC 9256, creating a scope conflict.

**Fix Direction:**

Revise the Distinguisher definition to clarify that it distinguishes SR Policy Candidate Paths (including the Protocol‑Origin dimension) rather than serving as a global identifier of the SR Policy.


**Severity:** Low
  *Basis:* The issue primarily affects clarity and specification hygiene without causing interoperability failures if implementations follow RFC 9256 for candidate path identification.

**Confidence:** High

---

## Report 2: 9830-2-2

**Label:** Segment List sub-TLV Length Off-by-One Ambiguity

**Bug Type:** Inconsistency

**Explanation:**

The definition of the Segment List sub-TLV Length field in RFC 9830 excludes the RESERVED octet, conflicting with the RFC 9012 requirement that the length covers the entire Value field, and thus introduces an off‐by‐one discrepancy.

**Justification:**

- RFC 9830 specifies the Length as accounting only for the nested sub‑TLVs, while RFC 9012’s generic sub‑TLV format requires the length to include everything after the Type/Length header.
- This discrepancy risks misparsing nested sub‑TLVs and may lead to treat‑as‑withdraw behavior if implementations follow different interpretations.

**Evidence Snippets:**

- **E1:**

  The Segment List sub-TLV is defined as: Type (1 octet) | Length (2 octets) | RESERVED (1 octet) | // sub-TLVs //. Length is specified as: “The total length (not including the Type and Length fields) of the sub‑TLVs encoded within the Segment List sub‑TLV in terms of octets.”

- **E2:**

  RFC 9012’s generic sub‑TLV format: “Sub‑TLV Length (1 or 2 octets): The total number of octets of the Sub‑TLV Value field” where the Value is everything after the Type/Length header.

**Evidence Summary:**

- (E1) Provides the RFC 9830 definition that excludes the RESERVED octet from the Length count.
- (E2) Demonstrates RFC 9012’s requirement to count the entire Value field, including any RESERVED bytes.

**Fix Direction:**

Amend the specification to either include the RESERVED octet in the Length field calculation or explicitly state that it is excluded—and adjust the parser accordingly—to align with the generic RFC 9012 sub‑TLV encoding.


**Severity:** Medium
  *Basis:* The off‐by‐one discrepancy can lead to misinterpretation of TLV boundaries, causing divergent parsing behavior and potential treat‑as‑withdraw of valid attributes.

**Confidence:** High

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

## Report 4: 9830-2-4

**Label:** Ambiguous Route Target to BGP Identifier Matching Semantics

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define how Route Target extended communities in the IPv4-address format should be matched to the receiver's BGP Identifier, leaving open whether only the Global Administrator field or the full 8‑octet value is used in the comparison.

**Justification:**

- The text mandates that at least one Route Target must match the BGP Identifier but fails to specify if the Local Administrator field is to be considered, creating room for divergent implementations.
- This underspecification in the matching criteria can lead to inconsistent determination of a route's usability across different platforms.

**Evidence Snippets:**

- **E1:**

  Section 4.2.2: “If one or more route targets are present, then at least one route target MUST match the BGP Identifier of the receiver for the update to be considered usable. The BGP Identifier is defined in [RFC4271] as a 4-octet IPv4 address and is updated by [RFC6286] as a 4-octet, unsigned, non-zero integer. Therefore, the Route Target extended community MUST be of the same format.”

- **E2:**

  Section 2.1 (bullet list): “One or more IPv4 address-specific format Route Target extended community ([RFC4360]) attached to the SR Policy CP advertisement that indicates the intended headend of such an SR Policy CP advertisement.”

**Evidence Summary:**

- (E1) Specifies the requirement for route target matching but does not detail the field-by-field comparison method.
- (E2) Indicates the use of IPv4-address-specific Route Targets to signal the intended headend without clarifying the matching algorithm.

**Fix Direction:**

Clarify the matching algorithm by explicitly stating that only the Global Administrator field of the Route Target should be compared to the receiver's BGP Identifier.


**Severity:** Medium
  *Basis:* This ambiguity could lead to divergent interpretations among implementations, potentially causing inconsistencies in route usability assessment in multi-vendor environments.

**Confidence:** High

---
