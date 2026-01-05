# Errata Reports

Total reports: 1

---


## Report 1: 9773-6-1

**Label:** Ambiguous Normative Update for RenewalInfo GET Access vs RFC 8555 POST-as-GET

**Bug Type:** Underspecification

**Explanation:**

RFC 9773 specifies that RenewalInfo resources are accessed via unauthenticated GET requests as a departure from RFC 8555’s POST-as-GET requirement, but it does not explicitly clarify whether POST-as-GET remains allowed, creating ambiguity for implementers.

**Justification:**

- RFC 8555 Section 6.3 mandates that GET requests be rejected (except for directory and newNonce resources), whereas RFC 9773 notes an exception for RenewalInfo without explicitly normative language.
- The lack of explicit normative guidance forces implementers to infer whether both GET (unauthenticated) and POST-as-GET are acceptable for RenewalInfo, potentially leading to interoperability issues.

**Evidence Snippets:**

- **E1:**

  RFC 8555 §6.3: “Except for the cases described in this section, if the server receives a GET request, it MUST return an error with status code 405 (Method Not Allowed)… and “The server MUST allow GET requests for the directory and newNonce resources…”

- **E2:**

  RFC 9773 §4.1: “To request the suggested renewal information for a certificate, the client sends an unauthenticated GET request to a path under the server's renewalInfo URL.”

- **E3:**

  RFC 9773 §6: “This document specifies that RenewalInfo resources are exposed and accessed via unauthenticated GET requests, a departure from the requirement in RFC 8555 that clients send POST-as-GET requests to fetch resources from the server.”

**Evidence Summary:**

- (E1) RFC 8555 requires GET to be disallowed (except for directory and newNonce), enforcing POST-as-GET for other resources.
- (E2) RFC 9773 instructs the use of an unauthenticated GET for accessing RenewalInfo resources.
- (E3) RFC 9773 highlights the use of GET as a departure from RFC 8555, without a clear normative statement on whether POST-as-GET remains valid.

**Fix Direction:**

Explicitly update the normative text to state that servers implementing ARI MUST accept unauthenticated GET for RenewalInfo and clarify whether POST-as-GET is also permitted or deprecated for these resources.


**Severity:** Medium
  *Basis:* The ambiguity may lead to interoperability issues as different implementations could interpret the renewalInfo access methods inconsistently.

**Confidence:** High

---