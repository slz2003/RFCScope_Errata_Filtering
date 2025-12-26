# Errata Reports

Total reports: 1

---

## Report 1: 9745-6-1

**Label:** Deprecation Link Relation Registry Description Inconsistency (Section 3 vs. Section 6.2)

**Bug Type:** Inconsistency

**Explanation:**

The IANA registration for the deprecation link relation in Section 6.2 restricts the target to documentation intended for human consumption, which contradicts Section 3 that explicitly allows both human-readable and machine-readable representations.

**Justification:**

- Section 3 explicitly states that there are no restrictions on the representation of the linked deprecation policy, allowing both human-readable and machine-readable descriptions (Deontic Expert).
- Section 6.2’s registry entry describes the deprecation relation using the phrase 'intended for human consumption', thereby narrowing its permitted targets (Terminology Expert).

**Evidence Snippets:**

- **E1:**

  This specification places no restrictions on the representation of the linked deprecation policy. In particular, the deprecation policy may be available as human-readable documentation or as a machine-readable description. (Section 3)

- **E2:**

  The deprecation link relation type has been added to the ‘Link Relation Types’ registry … as follows: … Description:  Refers to documentation (intended for human consumption) about the deprecation of the link's context.  Reference:  RFC 9745, Section 3 (Section 6.2)

**Evidence Summary:**

- (E1) Section 3 declares no restrictions on deprecation policy representation, allowing both human-readable and machine-readable forms.
- (E2) Section 6.2’s IANA registry entry restricts the representation by stating it is 'intended for human consumption'.

**Fix Direction:**

Adjust the IANA template in Section 6.2 by removing the restrictive phrase '(intended for human consumption)' so that it aligns with Section 3’s allowance for machine-readable representations.


**Severity:** Medium
  *Basis:* Both expert analyses indicate that the conflicting descriptions could lead to implementation confusion and interoperability issues.

**Confidence:** High

---
