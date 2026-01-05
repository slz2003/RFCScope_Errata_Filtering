# Errata Reports

Total reports: 1

---


## Report 1: 9870-6-1

**Label:** Mis-scoped reference to UDP Options security considerations (RFC9870 misnumbering error)

**Bug Type:** Inconsistency

**Explanation:**

RFC 9870 incorrectly directs readers to RFC 9868 Section 24 for UDP Options security considerations, while the actual security guidance is in Section 25.

**Justification:**

- RFC 9870 Section 6 states that readers may refer to Section 24 of RFC9868 for security considerations, but Section 24 is titled 'Network Management Considerations' and does not cover the expected security aspects.
- Experts note that the intended section for UDP Options security considerations is Section 25, creating an editorial cross-reference error that could mislead readers.

**Evidence Snippets:**

- **E1:**

  RFC 9870 Section 6: “This document does not introduce new security considerations other than those already discussed in Section 11 of [RFC7011] and Section 8 of [RFC7012]. The reader may refer to Section 24 of [RFC9868] for the security considerations related to UDP Options.”

- **E2:**

  RFC 9868 Section 24 heading: “24. Network Management Considerations.”

- **E3:**

  RFC 9868 Section 25 heading: “25. Security Considerations,” with detailed subsections 25.1–25.6 on option security, on-path attacks, DoS, fragmentation, covert channels, and security options such as AUTH and UENC.

**Evidence Summary:**

- (E1) RFC 9870 states to refer to Section 24 of RFC9868 for UDP Options security considerations.
- (E2) RFC9868 Section 24 is for network management, not security.
- (E3) The correct security considerations for UDP Options are in RFC9868 Section 25.

**Fix Direction:**

Change the reference in RFC 9870 Section 6 to point to Section 25 of RFC9868 (or to Sections 24 and 25 if both network management and security aspects are intended).


**Severity:** Low
  *Basis:* The issue is editorial; it may mislead readers but does not affect protocol operation or implementation.

**Confidence:** High

---