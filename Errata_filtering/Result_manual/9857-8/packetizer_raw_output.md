# Errata Reports

Total reports: 1

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


**Severity:** Low
  *Basis:* The contradiction is minor and is unlikely to affect interoperability, although it may confuse implementers.

**Confidence:** High

---