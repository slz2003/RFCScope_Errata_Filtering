# Errata Reports

Total reports: 1

---


## Report 1: 9745-3-1

**Label:** IANA Registry Description Narrows 'deprecation' Link Relation Scope

**Bug Type:** Inconsistency

**Explanation:**

The specification’s normative text (Section 3) allows the deprecation policy to be either human-readable or machine-readable, but the IANA registration (Section 6.2) restricts it to human-readable documentation, creating a semantic conflict.

**Justification:**

- Section 3 explicitly states that 'this specification places no restrictions on the representation of the linked deprecation policy' and permits both human-readable and machine-readable representations (E1).
- In contrast, the IANA registration in Section 6.2 describes the relation as 'Refers to documentation (intended for human consumption)' (E2), thereby narrowing the intended target type.
- This discrepancy may lead to divergent implementations and interoperability issues if different parts of the community rely on different parts of the specification.

**Evidence Snippets:**

- **E1:**

  Section 3 (immediately after introducing the deprecation link relation): “This specification places no restrictions on the representation of the linked deprecation policy. In particular, the deprecation policy may be available as human-readable documentation or as a machine-readable description.”

- **E2:**

  Section 6.2 IANA registration: `Description:  Refers to documentation (intended for human consumption) about the deprecation of the link's context.`

**Evidence Summary:**

- (E1) Section 3 allows both human-readable and machine-readable deprecation policy representations.
- (E2) Section 6.2 explicitly limits the representation to human-readable documentation.

**Fix Direction:**

Align the IANA registration description with Section 3 by removing or rephrasing the phrase '(intended for human consumption)' to acknowledge that machine-readable representations are also allowed.


**Severity:** Medium
  *Basis:* Interoperability and semantic understanding could be impaired if implementers follow only the IANA description, leading to fragmented implementations.

**Confidence:** High

---