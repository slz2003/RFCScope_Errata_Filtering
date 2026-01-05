# Errata Reports

Total reports: 1

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