# Errata Reports

Total reports: 1

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


**Severity:** Medium
  *Basis:* Overstating the security properties can lead implementers to mistakenly rely on digest fields for integrity, potentially compromising security even though the issue is primarily one of miscommunication.

**Confidence:** High

---