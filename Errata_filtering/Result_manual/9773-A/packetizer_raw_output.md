# Errata Reports

Total reports: 5

---


## Report 1: 9773-A-1

**Label:** Polling Timing Conflict: Stop Polling vs. Mandatory Retry-After Fetch

**Bug Type:** Inconsistency

**Explanation:**

There is a conflict between the requirement to stop polling after a certificate expires or is replaced and the mandate to immediately re-fetch RenewalInfo based on Retry-After or a six‐hour backoff.

**Justification:**

- The spec mandates that clients MUST NOT poll after expiry or replacement (Section 4.3) while also requiring them to poll again as soon as the Retry-After interval or six-hour period elapses (Sections 4.3.2 and 4.3.3), making the requirements mutually exclusive in certain timelines (T1).

**Evidence Snippets:**

- **E1:**

  Clients MUST NOT check a certificate's RenewalInfo after the certificate has expired. Clients MUST NOT check a certificate's RenewalInfo after they consider the certificate to be replaced (for instance, after a new certificate for the same identifiers has been received and configured). (Section 4.3)

- **E2:**

  After an initial fetch of a certificate's RenewalInfo, clients MUST fetch it again as soon as possible after the time indicated in the Retry-After header (backoff on errors takes priority, though). (Section 4.3.2)

- **E3:**

  On receiving a long-term error, clients MUST make the next renewalInfo request as soon as possible after six hours have passed (or some other locally configured default). (Section 4.3.3)

**Evidence Summary:**

- (E1) Prohibits RenewalInfo checks post-expiry or replacement (Section 4.3)
- (E2) Requires immediate re-fetch after the Retry-After duration (Section 4.3.2)
- (E3) Mandates polling after six hours on long-term error (Section 4.3.3)

**Fix Direction:**

Clarify that the Retry-After and six-hour re-fetch obligations apply only if the certificate remains valid and un-replaced, or adjust the polling rules to resolve the conflicting mandates.


**Severity:** Medium
  *Basis:* The conflicting MUST-level requirements can force clients into diverging behaviors, potentially causing inadvertent polling after expiry or replacement.

**Confidence:** High

---


## Report 2: 9773-A-2

**Label:** Underspecified Interaction Between Renewal-Time Algorithm and Retry-After Polling

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly specify how the recommended renewal-time algorithm (Section 4.2) is meant to interact with the global Retry-After polling schedule (Section 4.3.2), leaving open questions about whether additional polling is required once a renewal is scheduled.

**Justification:**

- The dual description—one algorithm with conditional sleep until Retry-After (Section 4.2) and a global mandate to re-fetch RenewalInfo at every Retry-After interval (Section 4.3.2)—creates ambiguity that may lead to over-polling or under-polling (T2).

**Evidence Snippets:**

- **E1:**

  Clients MUST attempt renewal at a time of their choosing based on the suggested renewal window. The following algorithm is RECOMMENDED for choosing a renewal time: ... (Section 4.2)

- **E2:**

  After an initial fetch of a certificate's RenewalInfo, clients MUST fetch it again as soon as possible after the time indicated in the Retry-After header (backoff on errors takes priority, though). (Section 4.3.2)

- **E3:**

  Clients MUST NOT check a certificate's RenewalInfo after they consider the certificate to be replaced (for instance, after a new certificate for the same identifiers has been received and configured). (Section 4.3)

**Evidence Summary:**

- (E1) Describes the renewal-time selection algorithm with conditional sleep until Retry-After (Section 4.2)
- (E2) Mandates a global polling based on the Retry-After header (Section 4.3.2)
- (E3) Forbids polling after considering a certificate replaced (Section 4.3)

**Fix Direction:**

Explicitly specify whether the polling dictated by Retry-After should continue even after a renewal attempt is scheduled or whether it should be suppressed to avoid redundant requests.


**Severity:** Low
  *Basis:* The underspecification could lead to divergent client behaviors that impact server load and renewal responsiveness, though it does not cause a complete functional breakdown.

**Confidence:** High

---


## Report 3: 9773-A-3

**Label:** Semantic Tension in Retry-After Timing Versus Client-Imposed Polling Bounds

**Bug Type:** Both

**Explanation:**

The spec defines Retry-After as indicating an exact desired delay while simultaneously permitting clients to clamp extreme values, resulting in potential discrepancies in the effective polling interval.

**Justification:**

- The explicit definition in Section 4.3 conflicts with the client-side adjustment rule in Section 4.3.2; this may lead to significant variability in actual polling intervals among different implementations (T3).

**Evidence Snippets:**

- **E1:**

  In this protocol, [Retry-After] indicates the desired (i.e., both requested minimum and maximum) amount of time to wait. (Section 4.3)

- **E2:**

  Clients MUST set reasonable limits on their checking interval. For example, values under one minute could be treated as if they were one minute, and values over one day could be treated as if they were one day. (Section 4.3.2)

**Evidence Summary:**

- (E1) Retry-After is defined as the exact desired delay (Section 4.3)
- (E2) Clients are permitted to impose bounds on the polling interval (Section 4.3.2)

**Fix Direction:**

Clarify the acceptable range of deviation from the Retry-After value and specify explicit bounds for client-imposed adjustments.


**Severity:** Low
  *Basis:* Although the discrepancy is semantic, it may lead to inconsistent polling frequencies, impacting server load distribution without causing outright protocol failure.

**Confidence:** High

---


## Report 4: 9773-A-4

**Label:** Ambiguous Uniqueness of AKI+Serial Identifier for RenewalInfo and Replaces Fields

**Bug Type:** Underspecification

**Explanation:**

The specification assumes that the identifier constructed from the base64url encoding of the certificate’s AKI keyIdentifier and its serial number is unique, but RFC 5280 guarantees uniqueness only for the (issuer name, serial) pair, potentially leading to collisions.

**Justification:**

- The identifier is used as the sole key for both RenewalInfo lookups and the 'replaces' field, yet the construction does not account for scenarios where the same signing key (and thus keyIdentifier) is used by multiple issuers with overlapping serial ranges, causing ambiguity in certificate identification.

**Evidence Snippets:**

- **E1:**

  The path component is a unique identifier for the certificate in question.  The unique identifier is constructed by concatenating the base64url encoding of the keyIdentifier field of the certificate's Authority Key Identifier (AKI) extension, the period character '.', and the base64url encoding of the DER-encoded Serial Number field… (Section 4.1)

- **E2:**

  replaces (string, optional): A string uniquely identifying a previously issued certificate that this order is intended to replace.  This unique identifier is constructed in the same way as the path component for GET requests described in Section 4.1. (Section 5)

- **E3:**

  The serial number MUST be a positive integer assigned by the CA to each certificate.  It MUST be unique for each certificate issued by a given CA (i.e., the issuer name and serial number identify a unique certificate). (Section 4.1.2.2)

**Evidence Summary:**

- (E1) Defines the construction of the unique identifier from AKI keyIdentifier and serial (Section 4.1)
- (E2) Reuses the same identifier construction for the 'replaces' field (Section 5)
- (E3) RFC 5280 guarantees uniqueness only for the (issuer name, serial) pair (Section 4.1.2.2)

**Fix Direction:**

Explicitly require that the (AKI keyIdentifier, serial) tuple be unique within the ACME server’s issuance domain or define a deterministic mechanism to handle collisions.


**Severity:** Medium
  *Basis:* Failure to guarantee uniqueness in environments with key reuse across issuers may lead to ambiguous RenewalInfo lookups and misinterpretation of the 'replaces' field.

**Confidence:** High

---


## Report 5: 9773-A-5

**Label:** Ambiguity in Timestamp Format Requirements for ARI Suggested Window

**Bug Type:** Underspecification

**Explanation:**

While ARI defines suggestedWindow timestamps using RFC3339, it does not clarify whether they must adhere to the stricter timestamp format rules of ACME core, risking interoperability between servers and clients.

**Justification:**

- Without explicit guidance on whether to follow the stricter ACME core subset of RFC3339 or allow the full RFC3339 syntax, servers may emit timestamps that some clients reject, potentially causing clients to fallback to alternative renewal behaviors.

**Evidence Snippets:**

- **E1:**

  suggestedWindow (object, required): A JSON object with two keys, ‘start’ and ‘end’, whose values are timestamps, encoded in the format specified in [RFC3339], which bound the window of time in which the CA recommends renewing the certificate.

- **E2:**

  A RenewalInfo object in which the end timestamp equals or precedes the start timestamp is invalid. Servers MUST NOT serve such a response, and clients MUST treat one as though they failed to receive any response from the server (e.g., retry at an appropriate interval, renew on a fallback schedule, etc.).

- **E3:**

  This document specifies the ACME Renewal Information (ARI) extension, a mechanism by which ACME servers may provide suggested renewal windows to ACME clients…

**Evidence Summary:**

- (E1) Specifies timestamps for suggestedWindow using RFC3339 (Section 4.2)
- (E2) Defines invalid conditions for timestamps (end ≤ start) (Section 4.2)
- (E3) Describes the ARI mechanism without clarifying adherence to ACME core timestamp constraints

**Fix Direction:**

Clarify whether ARI timestamps MUST conform to the same strict RFC3339 subset as ACME core or if the full RFC3339 specification is acceptable.


**Severity:** Low
  *Basis:* Inconsistent timestamp formats could lead clients to misinterpret valid RenewalInfo objects and trigger fallback error handling.

**Confidence:** Medium

---