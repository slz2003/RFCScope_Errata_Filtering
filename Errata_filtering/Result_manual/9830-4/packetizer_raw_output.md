# Errata Reports

Total reports: 2

---

## Report 1: 9830-4-1

**Label:** Ambiguous Route Target–BGP Identifier Matching Semantics in SR Policy Usability

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define how an IPv4-address‑specific Route Target should be compared to the receiver’s BGP Identifier to decide if an SR Policy NLRI is usable.

**Justification:**

- Section 4.2.2 requires that if one or more route targets are present then at least one must match the receiver’s BGP Identifier, but it does not specify whether only the 4‑octet Global Administrator field or the entire 6‑octet value (including the 2‑octet Local Administrator) should be compared.
- Multiple expert analyses note that this underspecification could lead to different interpretations and inconsistent behavior among implementations.

**Evidence Snippets:**

- **E1:**

  Section 4.2.2: “If one or more route targets are present, then at least one route target MUST match the BGP Identifier of the receiver for the update to be considered usable. The BGP Identifier is defined in [RFC4271] as a 4-octet IPv4 address and is updated by [RFC6286] as a 4-octet, unsigned, non-zero integer. Therefore, the Route Target extended community MUST be of the same format.” (Section 4.2.2)

- **E2:**

  DeonticAnalysis Issue-1: The specification never normatively defines what “match” means at the field level for IPv4-address‑specific RTs.

- **E3:**

  TerminologyAnalysis noted that it is unclear whether only the 4‑octet Global Administrator field is compared to the receiver’s BGP Identifier, or if the full 6‑octet RT value is used.

**Evidence Summary:**

- (E1) Section 4.2.2 mandates that a route target must match the BGP Identifier but leaves the field-level details ambiguous.
- (E2) DeonticAnalysis highlights the lack of a precise definition for the ‘match’ operation.
- (E3) TerminologyAnalysis emphasizes that different interpretations (comparing only the Global Administrator versus the full RT) are possible.

**Fix Direction:**

Specify that the 4‑octet Global Administrator field of the IPv4‑address‑specific Route Target MUST equal the receiver’s BGP Identifier, and explicitly state that the 2‑octet Local Administrator field shall be ignored in the matching process.


---

## Report 2: 9830-4-2

**Label:** Conflicting Treatment of Unsupported/Unrecognized Tunnel Encapsulation Sub‑TLVs for SR Policy Usability

**Bug Type:** Inconsistency

**Explanation:**

The document provides conflicting instructions on handling unsupported or unrecognized sub‑TLVs in the SR Policy Tunnel Encapsulation Attribute, making it unclear whether their presence should be ignored or cause the NLRI to be considered not usable.

**Justification:**

- Section 2.3 mandates that sub‑TLVs without explicitly defined applicability to the SR Policy SAFI MUST be ignored and may be removed during propagation.
- In contrast, Section 4.2.2 states that if any unrecognized or unsupported sub‑TLV is present, the update SHOULD NOT be considered usable, with an optional override.
- This conflict creates an ambiguity that can lead to inconsistent decisions on the usability of an SR Policy across different implementations.

**Evidence Snippets:**

- **E1:**

  Section 2.3: “Similarly, any other sub-TLVs, including those specified in [RFC9012], that do not have explicitly defined applicability to the SR Policy SAFI MUST be ignored by the BGP speaker and MAY be removed from the Tunnel Encapsulation Attribute during propagation.”

- **E2:**

  Section 4.2.2: “When the SR Policy tunnel type includes any sub-TLV that is unrecognized or unsupported, the update SHOULD NOT be considered usable. An implementation MAY provide an option for ignoring unsupported sub-TLVs.”

- **E3:**

  BoundaryAnalysis Finding-1: The overlap between sections creates an ambiguous decision surface for sub‑TLVs that are both non‑applicable and unrecognized, leading to potential divergent interpretations in their impact on NLRI usability.

**Evidence Summary:**

- (E1) Section 2.3 requires that non-applicable sub‑TLVs be ignored and may be removed.
- (E2) Section 4.2.2 indicates that unrecognized/unsupported sub‑TLVs should cause the update to be marked as not usable.
- (E3) BoundaryAnalysis points out that this overlap creates ambiguity affecting the usability determination.

**Fix Direction:**

Clarify the specification by explicitly separating the treatment of sub‑TLVs: sub‑TLVs with no defined applicability to the SR Policy SAFI MUST be ignored (without affecting usability), while only sub‑TLVs defined as applicable but unsupported should trigger the non-usability of the update unless an override is configured.



