# Errata Reports

Total reports: 1

---


## Report 1: 9904-6-1

**Label:** RFC 9904 References RFC 7583 for Algorithm Rollovers Despite Its Exclusion

**Bug Type:** Inconsistency

**Explanation:**

RFC 9904 instructs operators to consult both RFC 6781 and RFC 7583 for algorithm rollover guidance, yet RFC 7583 explicitly excludes algorithm rollovers from its scope, leading to a misleading reference.

**Justification:**

- RFC 9904 Section 6 directs: “DNSKEY algorithm rollover in a live zone is a complex process. See [RFC6781] and [RFC7583] for guidelines on how to perform algorithm rollovers.”
- RFC 7583 Section 1.4 clearly states its exclusion of algorithm rollovers by detailing that it covers only rolling keys of the same algorithm.

**Evidence Snippets:**

- **E1:**

  RFC 9904 §6: “DNSKEY algorithm rollover in a live zone is a complex process. See [RFC6781] and [RFC7583] for guidelines on how to perform algorithm rollovers.”

- **E2:**

  RFC 7583 §1.4: “In particular, it does not cover: … Algorithm rollovers. Only the rolling of keys of the same algorithm is described here: not transitions between algorithms.”

**Evidence Summary:**

- (E1) RFC 9904 directs operators to use RFC 6781 and RFC 7583 for algorithm rollover guidelines.
- (E2) RFC 7583 explicitly excludes algorithm rollovers, stating it only addresses same-algorithm key rollovers.

**Fix Direction:**

Revise RFC 9904 Section 6 to clarify that RFC 7583 provides timing considerations for same‐algorithm key rollovers only, and that algorithm rollover procedures should be taken solely from RFC 6781.

**Severity:** Medium
  *Basis:* The mischaracterization risks misleading operators into applying inappropriate rollover procedures, potentially leading to transient validation issues.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ScopeExpert: Issue-1
- DeonticExpert: Issue-1
- CrossRFCExpert: Issue-1

---