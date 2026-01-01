# Errata Reports

Total reports: 2

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

## Report 2: 9904-6-2

**Label:** Underspecified Procedure for DS Digest Algorithm Rollover Prior to KSK Roll

**Bug Type:** Underspecification

**Explanation:**

RFC 9904 mandates that DS digest algorithm changes must precede KSK rollovers, but it fails to provide concrete procedural details or timing guidelines for the DS digest algorithm rollover.

**Justification:**

- RFC 9904 §6 states that upgrading the DS algorithm concurrently with a KSK rollover will cause validation failures and mandates upgrading the DS algorithm first.
- There is no accompanying detailed phase or timing model in RFC 9904, RFC 6781, or RFC 7583 that explains the exact sequence, wait intervals, or record coexistence needed for a safe DS digest algorithm change.

**Evidence Snippets:**

- **E3:**

  RFC 9904 §6 introduces a strict temporal ordering between two operations: changing the DS “algorithm” (i.e., the DS digest algorithm) and rolling the KSK key. It states that upgrading the DS algorithm at the same time as rolling to a new KSK “will lead to DNSSEC validation failures” and that users MUST change the DS algorithm first before rolling to a new KSK.

- **E4:**

  However, neither RFC 9904 nor the cited guidance in RFC 6781 and RFC 7583 specify a concrete phase/timing model for DS-digest-algorithm rollovers analogous to the detailed key and algorithm-rollover timelines given for DNSKEYs.

**Evidence Summary:**

- (E3) RFC 9904 mandates that the DS algorithm must be upgraded before a new KSK is rolled to avoid validation failures.
- (E4) The documents lack a detailed procedural or timing model for how the DS digest algorithm rollover should be executed.

**Fix Direction:**

Provide explicit procedural instructions and timing guidelines for DS digest algorithm rollovers, including the steps for DS record publication, coexisting multiple digest algorithms, and required wait intervals before initiating a KSK rollover.

**Severity:** Medium
  *Basis:* The underspecification may lead to varying interpretations by implementers, which could result in intermittent DNSSEC validation failures across different caching scenarios.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T2

---
