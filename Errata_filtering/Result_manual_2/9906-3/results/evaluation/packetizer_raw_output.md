# Errata Reports

Total reports: 1

---

## Report 1: 9906-3-1

**Label:** No substantive security-specification inconsistency in Section 3

**Bug Type:** None

**Explanation:**

Section 3’s text on deprecating ECC-GOST-related algorithms is a high-level, qualitative statement and does not present a substantive security specification error.

**Justification:**

- The Excerpt Summary indicates that Section 3 provides a brief Security Considerations text which mentions that deprecating the ECC-GOST-related algorithms 'potentially increases DNSSEC security by removing algorithms no longer recommended' (E1).
- The Candidate Issue explicitly states 'No substantive security-specification inconsistency in Section 3' and includes a sketch noting that the section merely states that removing deprecated algorithms 'potentially increases' security (E2).

**Evidence Snippets:**

- **E1:**

  Excerpt Summary: Section 3 of RFC 9906 provides a brief Security Considerations text, stating that deprecating the ECC-GOST-related algorithms potentially increases DNSSEC security by removing algorithms no longer recommended.

- **E2:**

  Candidate Issues: 1

  Issue 1:
    Type: None
    Label: No substantive security-specification inconsistency in Section 3
    Relevant Dimensions: 
    Sketch: Section 3 merely states that removing deprecated algorithms “potentially increases” security. There ...

**Evidence Summary:**

- (E1) Section 3 provides brief Security Considerations stating that deprecating ECC-GOST-related algorithms potentially increases DNSSEC security by removing algorithms no longer recommended.
- (E2) Candidate Issue 1 indicates 'No substantive security-specification inconsistency in Section 3' with a sketch explaining that the section only notes that removal of deprecated algorithms potentially increases security.

**Severity:** Low
  *Basis:* All dimension ratings are LOW and the candidate issue is judged as non-substantive.

**Confidence:** High

**Experts mentioning this issue:**

- Router Analysis: Issue 1

---
