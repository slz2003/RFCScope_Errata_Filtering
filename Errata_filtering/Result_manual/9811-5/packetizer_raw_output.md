# Errata Reports

Total reports: 3

---

## Report 1: 9811-5-1

**Label:** Mischaracterization of RFC 9530 Digest Fields as Effective Security Protocol

**Bug Type:** Inconsistency

**Explanation:**

RFC 9811 mistakenly characterizes the digest fields defined in RFC 9530 as providing integrity protection equivalent to TLS, which can mislead implementers regarding their true security properties.

**Justification:**

- The specification groups 'HTTP digest [RFC9530]' alongside TLS as effective security protocols, despite RFC 9530 stating that its digest fields are only meant to detect corruption and do not protect against active attacks.
- Multiple expert analyses (Causal, CrossRFC, and Terminology) note that referring to these digest fields as an independent means of achieving integrity is a misnaming and mischaracterization that contradicts RFC 9530.

**Evidence Snippets:**

- **E1:**

  Without being encapsulated in effective security protocols, such as Transport Layer Security (TLS) [RFC5246] [RFC8446], or without using HTTP digest [RFC9530], there is no integrity protection at the HTTP level. Therefore, information from the HTTP should not be used to change state of the transaction, regardless of whether any mechanism was used to ensure the authenticity or integrity of HTTP messages (e.g., TLS or HTTP digests).

- **E2:**

  RFC 9530 defines specific HTTP fields: Content-Digest, Repr-Digest, Want-Content-Digest, Want-Repr-Digest, collectively called “Integrity fields” and “Integrity preference fields”. It does not define any entity named “HTTP digest” or “HTTP digests”.

**Evidence Summary:**

- (E1) shows that the text treats HTTP digest as an effective integrity mechanism alongside TLS.
- (E2) clarifies that RFC 9530 only defines digest fields and does not name a protocol called 'HTTP digest'.

**Fix Direction:**

Revise Section 5, item 2 to accurately describe the RFC 9530 digest fields as complementary, non-authenticated integrity checks that must be used with proper transport security (e.g., TLS), rather than implying they offer standalone security.


---

## Report 2: 9811-5-2

**Label:** Conflicting Guidance on Using HTTP-Layer Information for CMP Transaction State

**Bug Type:** Inconsistency / Ambiguity

**Explanation:**

RFC 9811 contains contradictory instructions regarding the use of HTTP metadata (such as status codes and headers) to change CMP transaction state, resulting in ambiguous implementation guidance.

**Justification:**

- Section 5, item 2 mandates that HTTP information should never be used to change transaction state, regardless of the presence of integrity mechanisms, while other sections (e.g., Section 3.5 and item 4) rely on HTTP response status codes to drive announcement processing.
- This creates normative tension for implementers who must decide whether to ignore HTTP-level metadata entirely or to use it when transport-level security is in place.

**Evidence Snippets:**

- **E1:**

  Without being encapsulated in effective security protocols, such as Transport Layer Security (TLS) [RFC5246] [RFC8446], or without using HTTP digest [RFC9530], there is no integrity protection at the HTTP level. Therefore, information from the HTTP should not be used to change state of the transaction, regardless of whether any mechanism was used to ensure the authenticity or integrity of HTTP messages (e.g., TLS or HTTP digests).

- **E2:**

  CMP announcement messages do not require any CMP response. However, the recipient MUST acknowledge receipt with an HTTP response having an appropriate status code and empty content. When not receiving such a response, it MUST be assumed that the delivery was not successful. If applicable, the sending side MAY try sending the announcement again after waiting for an appropriate time span.

**Evidence Summary:**

- (E1) instructs that HTTP metadata should not affect CMP state regardless of transport security.
- (E2) demonstrates that HTTP status codes are used to determine message delivery success in announcement procedures.

**Fix Direction:**

Clarify the guidance in Section 5 by distinguishing between scenarios where HTTP metadata may inform delivery reliability (when authenticated by TLS or similar mechanisms) and where it must not be used to alter the CMP transaction state itself.


---

## Report 3: 9811-5-3

**Label:** Underspecified Mapping for 'HTTP digest' to RFC 9530 Fields

**Bug Type:** Underspecification

**Explanation:**

RFC 9811 generically refers to 'HTTP digest [RFC9530]' without specifying whether the Content-Digest or Repr-Digest field should be employed, leaving room for inconsistent interpretations across implementations.

**Justification:**

- The lack of a precise specification means that implementers may choose different digest mechanisms when applying RFC 9530 to CMP messages, leading to inconsistent verification behavior.
- This ambiguity was noted by the CrossRFC Expert and may result in interoperability issues even if the guidance is only used illustratively.

**Evidence Snippets:**

- **E1:**

  RFC 9811 refers generically to “HTTP digest [RFC9530]” without specifying which of the two distinct mechanisms defined in RFC9530—Content‑Digest or Repr‑Digest—would be applicable to CMP messages over HTTP.

**Evidence Summary:**

- (E1) illustrates that there is no clear designation as to which digest field from RFC 9530 should be used with CMP messages.

**Fix Direction:**

Specify in RFC 9811 which RFC 9530 digest field(s) (Content-Digest, Repr-Digest, or a combination) are intended for use with CMP over HTTP to ensure uniform implementation.


---
