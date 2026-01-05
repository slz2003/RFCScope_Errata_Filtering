# Errata Reports

Total reports: 1

---


## Report 3: 9830-1-3

**Label:** Color-Only Type 3 Mismatch: 'Reserved' in Text vs. 'Unassigned' in IANA Table

**Bug Type:** Inconsistency

**Explanation:**

The normative text in Section 3 mandates that Color-Only Type 3 is reserved and must be treated like Type 0, yet the IANA table in Section 6.9 lists it as 'Unassigned', presenting a conflicting description.

**Justification:**

- Section 3 clearly states that Type 3 is 'Reserved for future use and SHOULD NOT be used' and requires receivers to treat it as Type 0.
- The IANA registry (Table 9 in Section 6.9) labels Type 3 as 'Unassigned', which may mislead implementers regarding its proper handling.

**Evidence Snippets:**

- **E1:**

  Section 3 (Color Extended Community):  
“Type 3 (bits 11):  Reserved for future use and SHOULD NOT be used.  Upon reception, an implementation MUST treat it like Type 0.”

- **E2:**

  Section 6.9, IANA registry “Color Extended Community Color-Only Types”, Table 9:
| 3    | Unassigned                            | RFC 9830  |

**Evidence Summary:**

- (E1) Indicates that Type 3 is normatively reserved and must be handled as Type 0.
- (E2) Shows that the IANA table describes Type 3 as unassigned, conflicting with the text.

**Fix Direction:**

Update the IANA table and associated registry text to reflect the normative status of Type 3 as 'Reserved; SHOULD NOT be used. Upon reception, treat as Type 0', ensuring consistency with Section 3.


**Severity:** Low
  *Basis:* While this discrepancy is unlikely to cause direct interoperability issues, it can lead to confusion about the correct handling of Type 3.

**Confidence:** High

---