# Errata Reports

Total reports: 1

---

## Report 1: 9774-4-1

**Label:** Overstated ASPA Route Selection Ineligibility for AS_SET Routes

**Bug Type:** Inconsistency

**Explanation:**

RFC 9774 states that in ASPA-based AS_PATH verification, routes with AS_SET are always treated as Invalid and hence ineligible for route selection, which conflates the mandatory classification with a recommended mitigation policy that allows operator discretion.

**Justification:**

- RFC 9774 Section 4 includes the statement that a route with AS_SET is 'always considered Invalid and hence ineligible for route selection' (E1).
- The ASPA verification draft explicitly specifies that while routes with AS_SET are classified as Invalid (as seen in Sections 6.2 and 6.3), the mitigation policy in Section 6.4 only 'SHOULD' treat such routes as ineligible, leaving room for operator discretion (E2, E3).

**Evidence Snippets:**

- **E1:**

  From RFC 9774 Section 4: “In ASPA-based AS_PATH verification, a route with AS_SET is always considered Invalid and hence ineligible for route selection.”

- **E2:**

  From ASPA verification draft: “In the current document, routes with AS_SET are given Invalid evaluation in the AS_PATH verification procedures (Section 6.2 and Section 6.3).”

- **E3:**

  From ASPA mitigation policy: “The specific configuration of a mitigation policy based on AS_PATH verification using ASPA is at the discretion of the network operator. However, the following mitigation policy is RECOMMENDED. *Invalid*: If the AS_PATH is determined to be Invalid, then the route SHOULD be considered ineligible for route selection …”

**Evidence Summary:**

- (E1) RFC 9774 mandates that routes with AS_SET are unconditionally treated as Invalid and ineligible for route selection.
- (E2) The ASPA draft specifies that routes with AS_SET are flagged as Invalid.
- (E3) The ASPA mitigation policy recommends (using SHOULD) that such Invalid routes be considered ineligible, thereby preserving operator discretion.

**Fix Direction:**

Revise the RFC 9774 wording to clarify that while ASPA-based verification mandates classification of AS_SET routes as Invalid, the decision to treat these Invalid routes as ineligible for route selection is a recommended policy subject to operator configuration.


**Severity:** Medium
  *Basis:* The misrepresentation may mislead implementers and operators by implying a hard requirement rather than a recommendation, potentially constraining deployment choices.

**Confidence:** High

---
