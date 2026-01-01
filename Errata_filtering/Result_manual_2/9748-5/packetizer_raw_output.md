# Errata Reports

Total reports: 2

---

## Report 1: 9748-5-1

**Label:** No substantive security-specification defect visible in RFC 9748 Section 5

**Bug Type:** None

**Explanation:**

The analysis indicates that Section 5 of RFC 9748 does not introduce any new security considerations beyond those already defined in the referenced documents.

**Justification:**

- The Excerpt Summary clarifies that the section merely defers security considerations to underlying documents.
- The Candidate Issue explicitly states that no substantive security-specification defect is visible.

**Evidence Snippets:**

- **E1:**

  Excerpt Summary: Section 5 of RFC 9748 states that the RFC introduces no new security considerations and that security aspects remain those defined in the underlying documents referenced from the IANA registries (e.g., RFC 7821, 7822, 8915), plus RFC 8126 guidance on registries.

- **E2:**

  Candidate Issues: 1

  Issue 1:
    Type: None
    Label: No substantive security-specification defect visible in RFC 9748 Section 5
    Relevant Dimensions: 
    Sketch: Section 5 behaves like a standard “registry-maintenance” security considerations text: it asserts no...

**Evidence Summary:**

- (E1) The Excerpt Summary explains that Section 5 defers security considerations to pre-existing referenced documents.
- (E2) The Candidate Issue explicitly indicates that no significant security defect is present in Section 5.

**Severity:** Low
  *Basis:* The analysis rates all dimensions as LOW, indicating that the text behaves as expected for a registry-maintenance update.

**Confidence:** High

**Experts mentioning this issue:**

- Router: Issue 1

---

## Report 2: 9748-5-2

**Label:** Loose usage of 'extension' terminology in RFC 9748 Section 5

**Bug Type:** Terminology

**Explanation:**

The analysis notes that the term 'extension' is applied in a slightly loose manner, which may be ambiguous in its intended coverage across registries.

**Justification:**

- The 'Terminology' dimension calls out that the term does not literally apply to all registries (e.g., Reference IDs), potentially leading to ambiguity.

**Evidence Snippets:**

- **E3:**

  Terminology: LOW - The use of “extension” is slightly loose (it doesn’t literally apply to all registries, like Reference IDs), but this is wording, not a protocol inconsistency.

**Evidence Summary:**

- (E3) The text critiques the loose application of the term 'extension', highlighting possible ambiguity over its scope.

**Severity:** Low
  *Basis:* The terminology issue is rated LOW, signifying minor editorial ambiguity without a broader protocol impact.

**Confidence:** High

**Experts mentioning this issue:**

- Router: Terminology

---
