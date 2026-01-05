# Errata Reports

Total reports: 1

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