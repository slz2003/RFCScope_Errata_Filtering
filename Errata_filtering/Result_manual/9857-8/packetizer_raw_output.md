# Errata Reports

Total reports: 4

---

## Report 1: 9857-8-1

**Label:** Metric Type Registry Scope Misdescription in Section 8.6

**Bug Type:** Underspecification

**Explanation:**

Section 8.6 describes the BGP‐LS SR Policy Metric Types registry as applying only to the Metric Type field in Section 5.7.2, even though the Metric Type field in Section 5.6.6 also normatively depends on the same registry.

**Justification:**

- Both Metric Type fields in Sections 5.6.6 and 5.7.2 use the SR Policy Metric Types registry, but Section 8.6 only mentions the field in Section 5.7.2, which may confuse designated experts.
- The discrepancy in registry scope description could lead to misinterpretation during registry maintenance.

**Evidence Snippets:**

- **E1:**

  Section 5.6.6 (SR Metric Constraint sub‑TLV), Metric Type field: “The metric type code points that may be used in this sub-TLV are also listed in Section 8.6 of this document. Note that the metric types in this field are taken from the ‘BGP-LS SR Policy Metric Types’ IANA registry…”

- **E2:**

  Section 5.7.2 (SR Segment List Metric sub‑TLV), Metric Type field: “The semantics are the same as the metric type field in the SR Metric Constraint sub-TLV in Section 5.6.6 of this document.”

- **E3:**

  Section 8.6: “This registry contains the code points allocated to the metric type field defined in Section 5.7.2.”

**Evidence Summary:**

- (E1) Section 5.6.6 directs that its Metric Type code points are listed in Section 8.6 from the BGP-LS SR Policy Metric Types registry.
- (E2) Section 5.7.2 reuses the same semantics as Section 5.6.6.
- (E3) Section 8.6 mentions only the field in Section 5.7.2, omitting the explicit reference to Section 5.6.6.

**Fix Direction:**

Update the wording in Section 8.6 to reference both the Metric Type fields in Sections 5.6.6 and 5.7.2.




---

## Report 3: 9857-8-3

**Label:** Segment List Identifier Field Contradiction: Non‑Zero Requirement vs. Sentinel Value 0

**Bug Type:** Inconsistency

**Explanation:**

The Segment List Identifier field is defined as carrying a non‑zero integer, yet the document assigns a special meaning to the value 0, resulting in an internal contradiction.

**Justification:**

- The field is described as a 32‑bit unsigned non‑zero integer that serves as an identifier, implying that 0 should never be used.
- Immediately following this definition, the text states that a value of 0 indicates that no identifier is associated, leading to a clear conflict in the allowed value set.

**Evidence Snippets:**

- **E1:**

  “Segment List Identifier: 4 octets that carry a 32‑bit unsigned non‑zero integer that serves as the identifier associated with the segment list.”

- **E2:**

  “A value of 0 indicates that there is no identifier associated with the segment list. The scope of this identifier is the SR Policy candidate path.”

**Evidence Summary:**

- (E1) The field is initially defined as a non‑zero integer.
- (E2) The text then permits 0 as a sentinel value indicating no identifier.

**Fix Direction:**

Revise the field definition to either exclude 0 entirely or clearly permit 0 as a defined sentinel value, ensuring internal consistency.


---

## Report 4: 9857-8-4

**Label:** Underspecified Scope for SR Policy Protocol‐Origin Registry

**Bug Type:** Underspecification

**Explanation:**

The registry for the Protocol‐Origin field is described solely in reference to Section 4, yet the document hints that the same semantic space might be used by other SR Policy encodings, leaving its intended scope ambiguous.

**Justification:**

- The Residual Uncertainties note raises the possibility that the registry’s description is either intentionally BGP‑LS‐focused or under‑scoped for broader SR Policy use.
- This ambiguity could lead to confusion in future extensions or in interpreting the registry’s applicability outside of the BGP‑LS context.

**Evidence Snippets:**

- **E1:**

  The SR Policy Protocol-Origin registry created in Section 8.4 is described as containing code points for the “Protocol-Origin” field defined in Section 4. In practice, the same semantic space may also be used by other SR Policy encodings defined outside this RFC, but without those documents in view, it is unclear whether the description here is intentionally BGP-LS-focused or somewhat under-scoped for the broader SR ecosystem.

**Evidence Summary:**

- (E1) The text in the Residual Uncertainties section questions whether the registry’s scope is meant to be limited to BGP‑LS or extended to other SR Policy implementations.

**Fix Direction:**

Clarify the intended scope of the SR Policy Protocol‐Origin registry in Section 8.4 to indicate whether it applies solely to the BGP‑LS Protocol-Origin field or to a broader set of SR Policy encodings.


---
