# Errata Reports

Total reports: 3

---

## Report 1: 9774-5-1

**Label:** Undefined behavior when 'desired origin AS' is absent from the brief aggregated AS_PATH

**Bug Type:** Underspecification

**Explanation:**

The specification does not state what an implementation must do when the configured desired origin AS does not appear in the brief aggregated AS_PATH, leaving the behavior undefined and open to divergent interpretations.

**Justification:**

- Section 5.2 requires truncation after the right‑most instance of the desired origin AS yet explicitly notes that the desired origin AS could be the aggregating AS itself, a case in which it is normally absent.
- Multiple expert analyses (Scope, Causal, Deontic, CrossRFC, and Boundary) highlight that this ambiguity may lead to inconsistent aggregate origin AS values and divergent RPKI-ROV behavior across implementations.

**Evidence Snippets:**

- **E1:**

  §5.2: “the implementation MUST be configured to truncate the AS_PATH after the right-most instance of the desired origin AS for the aggregate. The desired origin AS could be the aggregating AS itself.”

- **E2:**

  Appendix B.4: “The trivial solution to addressing the issue is to simply discard all of the ASes for the contributing routes. … simply originating the route … with its own AS, 64500, means that a consistent ROA could be registered…”

**Evidence Summary:**

- (E1) shows the normative truncation requirement and the possibility that the aggregating AS (i.e. the desired origin) may not be present in the brief AS_PATH.
- (E2) provides an example of a trivial solution that implies discarding all contributing ASes, highlighting that no fallback behavior is normatively defined.

**Fix Direction:**

Add explicit normative text in the affected section(s) to define the required behavior when the desired origin AS is absent from the brief AS_PATH (for example, by specifying a fallback such as truncating to an empty AS_PATH or treating the configuration as invalid).


**Severity:** High
  *Basis:* The ambiguity can lead to inconsistent origin AS attribution and unpredictable RPKI validation outcomes, potentially causing routing disruptions or security issues.

**Confidence:** High

---

## Report 2: 9774-5-2

**Label:** Ambiguity in actor responsibility for enforcing consistent brief aggregation

**Bug Type:** Underspecification

**Explanation:**

The RFC uses language that ambiguously assigns responsibility for ensuring consistent brief aggregation, mixing references to both the implementation and the network operator.

**Justification:**

- Section 5.2 states that the implementation MUST be configured to truncate the AS_PATH, while Section 6.1 mandates that network operators MUST use consistent brief aggregation.
- This dual phrasing creates uncertainty about who is responsible for ensuring the intended behavior, potentially leading to inconsistent practices between vendors and operators.

**Evidence Snippets:**

- **E1:**

  To ensure a consistent BGP origin AS is announced for aggregate BGP routes for implementations of 'brief' BGP aggregation, the implementation MUST be configured to truncate the AS_PATH after the right‑most instance of the desired origin AS for the aggregate.

- **E2:**

  When aggregating prefixes, network operators MUST use consistent brief aggregation as described in Section 5.2.

**Evidence Summary:**

- (E1) indicates that the configuration obligation is phrased in terms of the implementation’s setup.
- (E2) imposes a normative requirement on network operators, leading to an unclear division of responsibilities.

**Fix Direction:**

Clarify the text to explicitly delineate the responsibilities between implementers (e.g., by providing a configuration mechanism) and network operators (who must correctly deploy and use that mechanism).


**Severity:** Medium
  *Basis:* The lack of clarity in role assignment may result in inconsistent behavior, but it is less likely to cause immediate routing failures compared to undefined algorithm outcomes.

**Confidence:** High

---

## Report 3: 9774-5-3

**Label:** Ambiguous stage for AS_PATH truncation in BGP processing

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly indicate whether the AS_PATH truncation for consistent brief aggregation should be applied before or after the local AS is prepended for eBGP advertisement.

**Justification:**

- The CrossRFC analysis notes that RFC 9774 §5.2 does not specify if the truncation should be executed on the AS_PATH produced by RFC 4271 §9.2.2.2 (before prepending) or on the final AS_PATH after applying §5.1.2’s advertising rules.
- This timing ambiguity can lead to different effective origin AS values, impacting ROA validation and overall routing consistency.

**Evidence Snippets:**

- **E1:**

  RFC 9774 §5.2 describes “consistent brief” aggregation as truncating “the AS_PATH after the right‑most instance of the desired origin AS” relative to “the otherwise expected result of BGP AS_PATH aggregation,” and notes that “the desired origin AS could be the aggregating AS itself.” It is not stated whether the truncation step in RFC 9774 is to be applied to (1) the AS_PATH produced by §9.2.2.2 before the local AS is prepended for eBGP advertisement, or (2) the final AS_PATH after applying §5.1.2’s advertising rules.

**Evidence Summary:**

- (E1) highlights the lack of clarity regarding the stage at which AS_PATH truncation is applied, creating potential discrepancies in the origin AS as observed by peers.

**Fix Direction:**

Specify the exact point in the BGP processing pipeline (e.g., before or after local AS prepending) at which the truncation must be performed to ensure consistent on‐the‐wire AS_PATHs.


**Severity:** Medium
  *Basis:* Inconsistent truncation timing can lead to different origin AS outputs and affect ROA validation, although it is a more technical ambiguity rather than a complete operational failure.

**Confidence:** Low

---
