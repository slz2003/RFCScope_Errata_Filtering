# Errata Reports

Total reports: 1

---


## Report 4: 9803-1-4

**Label:** Underspecified handling of 'custom' attribute when 'for' is not 'custom'

**Bug Type:** Underspecification

**Explanation:**

The specification does not clarify how to interpret a <ttl:ttl> element that includes a 'custom' attribute when the 'for' attribute does not equal 'custom'.

**Justification:**

- While the normative rules describe the use of the 'custom' attribute only when for="custom", there is no guidance on behavior if a 'custom' attribute is provided with any other value.
- This lack of clarification might result in inconsistent processing across different implementations.

**Evidence Snippets:**

- **E1:**

  the document does not specify how to handle a <ttl:ttl> where for != "custom" but a custom attribute is present; the most likely intended behavior is to ignore custom in that case.

**Evidence Summary:**

- (E1) Points out the absence of guidance for handling a 'custom' attribute when the 'for' attribute is not set to 'custom'.

**Fix Direction:**

Include a clarifying statement indicating that any 'custom' attribute should be ignored if the 'for' attribute is not 'custom'.


**Severity:** Low
  *Basis:* Although this ambiguity is less likely to cause major issues, it may still lead to minor interoperability differences.

**Confidence:** High

---