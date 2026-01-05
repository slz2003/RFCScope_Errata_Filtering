# Errata Reports

Total reports: 1

---


## Report 1: 9830-6-1

**Label:** Color-Only Type 3 Inconsistency: Reserved Semantics vs Unassigned in IANA Registry

**Bug Type:** Inconsistency

**Explanation:**

The document defines Color-Only Type 3 as reserved with mandatory receiver behavior in Section 3, yet the IANA registry in Section 6.9 labels it 'Unassigned', creating a contradictory specification.

**Justification:**

- Section 3 explicitly states: Type 3 is 'Reserved for future use and SHOULD NOT be used. Upon reception, an implementation MUST treat it like Type 0.' (E1)
- Section 6.9 lists the same code point as 'Unassigned' in the Color Extended Community registry, conflicting with the normative text (E2)

**Evidence Snippets:**

- **E1:**

  Section 3 defines the Color-Only Type field in the Color Extended Community flags and explicitly gives semantics for Type 3: “Type 3 (bits 11): Reserved for future use and SHOULD NOT be used. Upon reception, an implementation MUST treat it like Type 0.” (Section 3, Color Extended Community)

- **E2:**

  Section 6.9 creates the “Color Extended Community Color-Only Types” registry and Table 9 lists Type 3 as “Unassigned | RFC 9830” (Section 6.9, Table 9)

**Evidence Summary:**

- (E1) Section 3 specifies that Type 3 is reserved and mandates treatment as Type 0.
- (E2) Section 6.9’s IANA registry table marks Type 3 as Unassigned.

**Fix Direction:**

Update the IANA registry in Section 6.9 to indicate that Type 3 is reserved, for example by changing its entry to 'Reserved; on reception treat as Type 0', thereby aligning the registry with the normative text in Section 3.


**Severity:** Medium
  *Basis:* Multiple experts noted that while the normative text clearly defines receiver behavior, the 'Unassigned' label in the registry may mislead implementers and future authors regarding the code point's intended use.

**Confidence:** High

---