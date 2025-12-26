# Errata Reports

Total reports: 4

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

## Report 2: 9773-4-2

**Label:** Ambiguity in Six-Hour Long-Term Retry vs Certificate Expiry/Replacement

**Bug Type:** Underspecification

**Explanation:**

The six-hour mandatory retry interval for long-term errors does not clearly account for the condition when a certificate has expired or been replaced, leading to conflicting directives in the spec.

**Justification:**

- Section 4.3.3 requires a next RenewalInfo request after six hours following a long-term error.
- Separate language in Section 4.3 prohibits checking RenewalInfo after a certificate expires or is replaced.

**Evidence Snippets:**

- **E3:**

  On receiving a long-term error, clients MUST make the next renewalInfo request as soon as possible after six hours have passed (or some other locally configured default).

- **E4:**

  Clients MUST NOT check a certificate's RenewalInfo after the certificate has expired. Clients MUST NOT check a certificate's RenewalInfo after they consider the certificate to be replaced (for instance, after a new certificate for the same identifiers has been received and configured).

**Evidence Summary:**

- (E3) Mandates a six-hour retry interval on long-term errors, regardless of certificate state.
- (E4) Forbids renewal checks after expiry or replacement, creating a conflict if the six-hour timer lapses post-expiry.

**Fix Direction:**

Clarify that the six-hour retry rule applies only if the certificate is still valid and has not been replaced.


**Severity:** Low
  *Basis:* Although the conflict may only affect edge cases, the ambiguity could lead to divergent implementations when a certificate nears expiry.

**Confidence:** High

---

## Report 3: 9773-4-3

**Label:** Overbroad Classification of Non-5xx HTTP Errors as Long-Term Errors

**Bug Type:** Underspecification / Scope Conflict

**Explanation:**

The spec classifies all non-5xx HTTP errors, including 4xx rate-limited responses, as long-term errors, which conflicts with ACME’s established use of Retry-After to indicate when a request may be retried.

**Justification:**

- Section 4.3.3 lists 'Non-5xx HTTP error' among long-term errors, triggering a fixed six-hour wait regardless of the server-provided Retry-After value.
- RFC 8555 specifies that a valid Retry-After header in certain 4xx responses (such as rateLimited errors) should indicate a shorter delay, allowing finer control over retry intervals.

**Evidence Snippets:**

- **E5:**

  Examples of long-term errors include: Retry-After is invalid or not present; RenewalInfo object is invalid; DNS lookup failure; Connection refused; Non-5xx HTTP error. On receiving a long-term error, clients MUST make the next renewalInfo request as soon as possible after six hours have passed (or some other locally configured default).

- **E6:**

  In RFC 8555 section 6.6: 'Creation of resources can be rate limited … Once the rate limit is exceeded, the server MUST respond with an error with the type 'urn:ietf:params:acme:error:rateLimited'. Additionally, the server SHOULD send a Retry-After header field … indicating when the current request may succeed again.'

**Evidence Summary:**

- (E5) Applies a uniform six-hour retry for all non-5xx errors, including those with valid Retry-After values.
- (E6) Indicates that rate-limited errors should carry a Retry-After header guiding a specific, often shorter, retry interval.

**Fix Direction:**

Refine the error classification to exempt specific 4xx errors (e.g., rateLimited with a valid Retry-After) from the mandatory six-hour retry rule.


**Severity:** Medium
  *Basis:* This ambiguity may force clients into inappropriately long wait times, undermining the CA's control over client polling behavior and affecting service responsiveness.

**Confidence:** High

---

## Report 4: 9773-4-4

**Label:** Lack of Mandatory Server Requirement for Valid Retry-After Header in Successful Responses

**Bug Type:** Editorial/Quality-of-Service

**Explanation:**

The specification does not require servers to include a valid Retry-After header on successful RenewalInfo responses, potentially limiting the CA’s ability to finely control client polling intervals.

**Justification:**

- Section 4.3 describes how the Retry-After header is used to indicate the desired wait time but does not impose a MUST requirement on servers to always provide a valid header.
- The fallback behavior (treating missing or invalid Retry-After as a long-term error with a default six-hour wait) may lead to suboptimal polling in scenarios where more granular control is desired.

**Evidence Snippets:**

- **E7:**

  This protocol uses the Retry-After header [RFC9110] to indicate to clients how often to retry… In this protocol, it indicates the desired (i.e., both requested minimum and maximum) amount of time to wait.

- **E8:**

  Examples of long-term errors include: Retry-After is invalid or not present… On receiving a long-term error, clients MUST make the next renewalInfo request as soon as possible after six hours have passed (or some other locally configured default).

**Evidence Summary:**

- (E7) Describes the intended use of Retry-After for controlling client polling intervals.
- (E8) Indicates that absence or invalidity of the header triggers a default long-term error behavior, bypassing CA control.

**Fix Direction:**

Add a normative requirement that servers MUST include a valid Retry-After header on all successful 2xx RenewalInfo responses.


**Severity:** Low
  *Basis:* This issue primarily affects quality-of-service by reducing the CA's ability to manage client polling frequency, without causing protocol breakage.

**Confidence:** Medium

---
