# Errata Reports

Total reports: 1

---


## Report 1: 9773-4-1

**Label:** Ambiguous Handling of Invalid RenewalInfo Object (end ≤ start) Across Error Sections

**Bug Type:** Underspecification

**Explanation:**

The specification provides conflicting guidance on handling RenewalInfo objects with an inverted time window, leaving implementers unsure whether to apply temporary error backoff or immediately use the six‑hour long‐term error retry.

**Justification:**

- Section 4.2 states that an invalid RenewalInfo object must be treated as though no response was received, suggesting flexible retry intervals, while Section 4.3.3 explicitly classifies it as a long‐term error with a fixed six‑hour wait.
- This divergence in normative language can lead to different client behaviors under identical error conditions.

**Evidence Snippets:**

- **E1:**

  A RenewalInfo object in which the end timestamp equals or precedes the start timestamp is invalid. Servers MUST NOT serve such a response, and clients MUST treat one as though they failed to receive any response from the server (e.g., retry at an appropriate interval, renew on a fallback schedule, etc.).

- **E2:**

  Examples of long-term errors include: Retry-After is invalid or not present; RenewalInfo object is invalid; DNS lookup failure; Connection refused; Non-5xx HTTP error. On receiving a long-term error, clients MUST make the next renewalInfo request as soon as possible after six hours have passed (or some other locally configured default).

**Evidence Summary:**

- (E1) Specifies to treat an invalid object as if no response was received, implying use of flexible intervals.
- (E2) Classifies the same event as a long-term error with a fixed 6-hour retry.

**Fix Direction:**

Clarify the normative treatment by explicitly stating that invalid RenewalInfo objects MUST be handled as long-term errors with a fixed six-hour retry interval, overriding temporary error backoff.


**Severity:** Medium
  *Basis:* Divergent interpretations could lead to inconsistent polling behavior and affect client responsiveness and server load.

**Confidence:** High

---