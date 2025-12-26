# Errata Reports

Total reports: 3

---

## Report 1: 9774-B-1

**Label:** Underspecification: Missing behavior when desired origin AS is absent in aggregated AS_PATH

**Bug Type:** Underspecification

**Explanation:**

The specification does not define what an implementation should do if dynamic route changes cause the desired origin AS to disappear from the aggregated AS_PATH, resulting in ambiguous behavior.

**Justification:**

- Section 5.2 mandates truncation of the AS_PATH after the right‑most instance of the desired origin AS but does not specify what to do when that AS is not present (E1).
- Appendix B.4 illustrates a scenario where contributing routes evolve, yet no handling is defined for an aggregate that lacks the desired origin AS (E2).
- References to RFC 4271 and RFC 4632 indicate expected behaviors for aggregation, but they do not resolve this edge case (E3, E4).

**Evidence Snippets:**

- **E1:**

  To ensure a consistent BGP origin AS is announced for aggregate BGP routes for implementations of 'brief' BGP aggregation, the implementation MUST be configured to truncate the AS_PATH after the right‑most instance of the desired origin AS for the aggregate. The desired origin AS could be the aggregating AS itself. (Section 5.2)

- **E2:**

  In more complex proxy aggregation scenarios, there may be a desire to permit some stable (i.e., common) portion of the contributing AS_PATHs to be kept in the aggregate route. Consider the case for Scenario 3, where the neighbor AS is the same for both R3 and R4 -- AS 64504. In such a case, an implementation may permit the aggregate's brief AS_PATH to be '64504', and a ROA would be created for the aggregate prefix with 64504 as the origin AS. (Appendix B.4)

- **E3:**

  RFC 4271 aggregation rule: 'determine the longest leading sequence ... common to all the AS_PATH attributes of the routes to be aggregated'  (quoted in Section 5.1 of RFC 9774)

- **E4:**

  RFC 4632: 'In general, a dynamic aggregate route advertisement is added when at least one component of the aggregate becomes reachable and it is withdrawn only when all components become unreachable.' (Section 5.4)

**Evidence Summary:**

- (E1) Section 5.2 calls for AS_PATH truncation after the desired origin AS but provides no alternative when it is absent.
- (E2) Appendix B.4 describes a dynamic proxy aggregation scenario without guidance for the missing desired origin.
- (E3) The RFC 4271 aggregation rule is referenced but does not address the edge case.
- (E4) RFC 4632 is noted without clarifying behavior for this situation.

**Fix Direction:**

Clarify in Section 5.2 the expected behavior when the desired origin AS is not present in the aggregated AS_PATH due to route evolution.


**Severity:** Low
  *Basis:* This is a corner-case underspecification that may lead to modest interoperability issues without affecting normative protocol operation.

**Confidence:** High

---

## Report 2: 9774-B-2

**Label:** Underspecification: Ambiguity in ordering of AS_PATH truncation and local AS prepending

**Bug Type:** Underspecification

**Explanation:**

The document does not clarify whether AS_PATH truncation occurs before or after the BGP process of prepending the local AS, which can result in different wire-visible AS_PATHs.

**Justification:**

- RFC 4271 Section 5.1.2 describes the process of prepending the local AS, but Section 5.2’s requirement for truncation is not explicitly positioned relative to this step (E1).
- The ordering ambiguity directly affects the outcome in proxy aggregation cases, potentially altering the perceived origin AS in the advertised update (E2, E3).

**Evidence Snippets:**

- **E1:**

  When a given BGP speaker advertises the route to an external peer, the advertising speaker updates the AS_PATH attribute as follows: ... if the AS_PATH is empty, the local system creates a path segment of type AS_SEQUENCE, places its own AS into that segment, and places that segment into the AS_PATH. (RFC 4271 Section 5.1.2)

- **E2:**

  To ensure a consistent BGP origin AS is announced for aggregate BGP routes for implementations of 'brief' BGP aggregation, the implementation MUST be configured to truncate the AS_PATH after the right‑most instance of the desired origin AS for the aggregate. (Section 5.2)

- **E3:**

  In such a case, an implementation may permit the aggregate's brief AS_PATH to be '64504', and a ROA would be created for the aggregate prefix with 64504 as the origin AS. (Appendix B.4)

**Evidence Summary:**

- (E1) RFC 4271 Section 5.1.2 details AS_PATH prepending without relation to truncation.
- (E2) Section 5.2 requires AS_PATH truncation but is ambiguous about its position relative to prepending.
- (E3) Appendix B.4 exemplifies the potential impact of this ordering on the final AS_PATH.

**Fix Direction:**

Specify in Section 5.2 whether the truncation of the AS_PATH should be performed before or after prepending the local AS, in accordance with RFC 4271.


**Severity:** Low
  *Basis:* This underspecification may lead to moderate interoperability differences in deployments using proxy aggregation, though its impact is limited.

**Confidence:** High

---

## Report 3: 9774-B-3

**Label:** Inconsistency in Scenario 3 Aggregate AS_PATH for R3 in Appendix B

**Bug Type:** Inconsistency

**Explanation:**

The worked example in Scenario 3 erroneously shows the aggregate AS_PATH for R3 as '64504 64501', which contradicts the defined route R3 having AS_PATH '64504 64502' and standard RFC 4271 aggregation behavior.

**Justification:**

- The defined routes specify R3 with AS_PATH '64504 64502', yet the example states 'Receive R3.  Aggregate 192.0.2.0/24 AS_PATH "64504 64501"', creating a mismatch. (E1, E2)
- A single contributing route should yield an aggregate identical to its own AS_PATH, so the discrepancy in the first aggregate line leads to confusion about expected behavior.

**Evidence Snippets:**

- **E1:**

  Routes are defined earlier as:  R3 - 192.0.2.128/26 AS_PATH "64504 64502" and  R4 - 192.0.2.192/26 AS_PATH "64504 64501".

- **E2:**

  In Scenario 3, the first step says:  Receive R3.  Aggregate 192.0.2.0/24 AS_PATH "64504 64501"  Then, after R4 is received:  Receive R4.  Aggregate 192.0.2.0/24 AS_PATH "64504 [ 64501 64502 ]"  and  If brief aggregation is in use, the AS_PATH is truncated to "64504".

**Evidence Summary:**

- (E1) R3 is defined with AS_PATH '64504 64502' while R4 is defined with '64504 64501'.
- (E2) The example incorrectly shows R3's aggregate as '64504 64501' instead of matching its defined AS_PATH.

**Fix Direction:**

Replace the first aggregate AS_PATH line in Scenario 3 with '64504 64502' to match the defined route for R3 and comply with RFC 4271 aggregation rules.


**Severity:** Medium
  *Basis:* Although the error is limited to a non‑normative illustrative example, it can significantly confuse readers regarding proper aggregation behavior.

**Confidence:** High

---
