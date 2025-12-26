# Errata Reports

Total reports: 1

---

## Report 1: 9811-1-1

**Label:** Mismatch in CMP Message Handling: 'MUST forward' in Section 1.2 vs. 'MUST support handling' in Section 3.1

**Bug Type:** Inconsistency

**Explanation:**

RFC 9811 Section 1.2 uses a normative statement requiring implementations to 'MUST forward CMP messages' on HTTP error codes, which conflicts with Section 3.1 that only requires clients to support handling CMP messages when present, creating ambiguity about the expected behavior.

**Justification:**

- Section 1.2 mandates forwarding with the wording 'Implementations MUST forward CMP messages when an HTTP error status code occurs; see Section 3.1.', implying an obligation that is not defined in the detailed text.
- Section 3.1 instead specifies that clients 'MUST support handling response message content containing a CMP response PKIMessage' when receiving 2xx, 4xx, or 5xx responses, without requiring forwarding by servers or intermediaries.

**Evidence Snippets:**

- **E1:**

  Implementations MUST forward CMP messages when an HTTP error status code occurs; see Section 3.1.

- **E2:**

  All applicable Client Error 4xx or Server Error 5xx status codes MAY be used to inform the client about errors. Whenever a client receives an HTTP response with a status code in the 2xx, 4xx, or 5xx ranges, it MUST support handling response message content containing a CMP response PKIMessage.

**Evidence Summary:**

- (E1) indicates the introductory requirement to forward CMP messages on HTTP errors.
- (E2) shows that the normative text only requires clients to handle CMP message content, not to forward it.

**Fix Direction:**

Reword the Section 1.2 bullet to align with Section 3.1. For example, replace the current text with 'Implementations (in particular, CMP clients) MUST support handling CMP messages contained in the content of HTTP responses, including when an HTTP error status code (4xx or 5xx) occurs; see Section 3.1.'


**Severity:** Medium
  *Basis:* Ambiguity in normative language can mislead implementers and potentially result in divergent behaviors, affecting interoperability.

**Confidence:** High

---
