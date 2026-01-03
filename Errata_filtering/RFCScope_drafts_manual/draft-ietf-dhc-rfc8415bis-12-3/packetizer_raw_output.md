# Errata Reports

Total reports: 1

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-3-1

**Label:** Use of RFC 8415 solely as historical background reference

**Bug Type:** Reference Usage

**Explanation:**

Section 3 directs readers to RFC 8415 for IPv6 background material even though the background has been removed, which may be confusing as the reference is used solely for historical context.

**Justification:**

- The analysis excerpt explains that background material on IPv6 has been removed and readers are directed to RFC 8415 for that background, keeping the section only for numbering alignment (E1).
- The candidate issue explicitly labels this as a use of RFC 8415 solely as historical background reference (E2).

**Evidence Snippets:**

- **E1:**

  Excerpt Summary: Section 3 states that the former background material on IPv6 relevant to DHCPv6 has been removed from this revision and directs readers to RFC 8415 for that background, keeping the section only for numbering alignment.

- **E2:**

  Candidate Issues: 1

  Issue 1:
    Type: None
    Label: Use of RFC 8415 solely as historical background reference
    Relevant Dimensions: 
    Sketch: Section 3 only says that background material has been removed and directs readers to RFC 8415 for th...

**Evidence Summary:**

- (E1) indicates that section 3 has been modified to remove background material and now refers readers to RFC 8415 for IPv6 background.
- (E2) identifies a candidate issue regarding the exclusive use of RFC 8415 as a historical reference.

**Severity:** Low
  *Basis:* All dimensions (Temporal, ActorDirectionality, Scope, Causal, etc.) are rated LOW, indicating a minimal impact or merely editorial nature.

**Confidence:** High

**Experts mentioning this issue:**

- RouterAnalysis: Issue 1

---
