# Errata Reports

Total reports: 1

---


## Report 1: 9870-2-1

**Label:** Misreferenced RFC9868 'datagram' Terminology in RFC9870

**Bug Type:** Inconsistency

**Explanation:**

RFC9870 inaccurately claims that a standalone 'datagram' term is defined in RFC9868 Section 3, whereas RFC9868 only defines specific terms like 'IP datagram' and 'User datagram', creating potential ambiguity.

**Justification:**

- RFC 9870 Section 2 states that the document uses the terms defined in Section 3 of [RFC9868], especially 'datagram' and 'surplus area', yet the referenced section does not define a standalone 'datagram' (CrossRFC Expert Issue-1).
- The Terminology Expert analysis confirms that only compound terms ('IP datagram' and 'User datagram') are defined in RFC9868, not a single term 'datagram' (Terminology Expert Issue-1).

**Evidence Snippets:**

- **E1:**

  RFC 9870, Section 2:  Also, this document uses the terms defined in Section 3 of [RFC9868], especially "datagram" and "surplus area".

- **E2:**

  In the provided Section 3 of RFC 9868, there is **no standalone defined term** labeled simply “datagram”; only “IP datagram” and “User datagram” (and related “UDP packet”) are explicitly defined.

- **E3:**

  To fill that void, [RFC9868] extends UDP with a mechanism to insert extensions in datagrams. The document focuses exclusively on exporting observed UDP Options in datagrams.

**Evidence Summary:**

- (E1) RFC9870 Section 2 claims to use 'datagram' as defined in RFC9868.
- (E2) RFC9868 Section 3 does not include a standalone definition for 'datagram' but instead defines 'IP datagram' and 'User datagram'.
- (E3) The usage of the term 'datagrams' implies ambiguity in the intended reference.

**Fix Direction:**

Revise RFC9870 Section 2 to reference the precise terms 'IP datagram' and/or 'User datagram' as defined in RFC9868.


**Severity:** Low
  *Basis:* The inconsistency is minor and mostly editorial in nature, with little risk of causing implementation issues.

**Confidence:** High

---