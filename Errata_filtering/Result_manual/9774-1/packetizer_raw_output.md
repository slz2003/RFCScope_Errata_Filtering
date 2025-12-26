# Errata Reports

Total reports: 1

---

## Report 1: 9774-1-1

**Label:** Incorrect RFC 4271 Section Reference for BGP Aggregation in RFC 9774 Introduction

**Bug Type:** Inconsistency

**Explanation:**

RFC 9774 incorrectly cites RFC 4271 Section 9.1.2.2 instead of Section 9.2.2.2 for describing BGP aggregation, which may mislead implementers about the correct aggregation procedure.

**Justification:**

- Structural Expert noted that the Introduction text cites Section 9.1.2.2—which describes tie‐breaking—whereas the proper aggregation procedure is defined in Section 9.2.2.2 (see E1 and E2).
- CrossRFC Expert confirmed this mis-citation and explained that the wrong reference could lead readers to the incorrect section in RFC 4271, despite a later correct citation in RFC 9774 (see E3).

**Evidence Snippets:**

- **E1:**

  RFC 9774 Introduction: “By performing aggregation, a router is combining multiple BGP routes for more specific destinations into a new route for a less specific destination (see [RFC4271], Section 9.1.2.2).”

- **E2:**

  In RFC 4271, Section 9.1.2.2 is titled “Breaking Ties (Phase 2)” and describes tie‑breaking in the Decision Process, not aggregation, while Section 9.2.2.2 is titled “Aggregating Routing Information” and explicitly defines the aggregation procedure (“Aggregation is the process of combining the characteristics of several different routes in such a way that a single route can be advertised. Aggregation can occur as part of the Decision Process…”). RFC 9774 itself later states: “Sections 9.1.4 and 9.2.2.2 of [RFC4271] describe BGP aggregation procedures.”

- **E3:**

  Section 1 states: “By performing aggregation, a router is combining multiple BGP routes for more specific destinations into a new route for a less specific destination (see [RFC4271], Section 9.1.2.2).” In RFC 4271, Section 9.1.2.2 is “Breaking Ties (Phase 2)” in the decision process and describes path selection criteria (e.g., AS_PATH length, ORIGIN, MED), not route aggregation procedures or the construction of aggregate routes. The actual aggregation procedures are in Section 9.2.2.2 “Aggregating Routing Information”, which explains how to combine routes and their path attributes, including the algorithm for aggregating AS_PATHs and forming AS_SETs. RFC 9774 itself correctly cites “[RFC4271], Section 9.2.2.2” later when discussing brief aggregation in Section 5, so the Introduction’s pointer to 9.1.2.2 is both a cross‑RFC miscitation and an internal cross‑section inconsistency within RFC 9774.

**Evidence Summary:**

- (E1) RFC 9774 Introduction references aggregation with Section 9.1.2.2.
- (E2) Section 9.1.2.2 of RFC 4271 is about tie‑breaking, while Section 9.2.2.2 properly defines the aggregation procedure, as later noted in RFC 9774.
- (E3) CrossRFC Expert reiterates that the wrong section is cited in the Introduction, outlining the discrepancy between tie‑breaking and aggregation.

**Fix Direction:**

Replace the reference '[RFC4271], Section 9.1.2.2' in the RFC 9774 Introduction with '[RFC4271], Section 9.2.2.2'.


**Severity:** Low
  *Basis:* Although the mis-citation is misleading, it is unlikely to cause operational issues because the correct procedure is cited later and the overall prose is clear.

**Confidence:** High

---
