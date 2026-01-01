# Errata Reports

Total reports: 1

---

## Report 1: 9734-1-1

**Label:** Misreference for XMPP URI specification (RFC 6121 used instead of XMPP‑URI spec)

**Bug Type:** Inconsistency

**Explanation:**

RFC 9734 incorrectly cites RFC6121 as the defining document for the XMPP URI, whereas RFC6121 only provides handling guidance and defers to the XMPP‑URI specification for the actual URI scheme definition.

**Justification:**

- RFC 9734 Section 1 states that a subjectAltName can be an IM URI [RFC3860] or XMPP URI [RFC6121], yet RFC6121 does not define the xmpp: URI scheme, causing confusion for implementers (CrossRFC Expert).
- The evidence from RFC 6121 shows that it explicitly directs users to the [XMPP‑URI] specification, making the initial reference misleading (Terminology Expert).

**Evidence Snippets:**

- **E1:**

  RFC 9734 Section 1 states that a subjectAltName “can be an IM URI [RFC3860] or Extensible Messaging and Presence Protocol (XMPP) URI [RFC6121]”. RFC 3860 is indeed the defining document for the “im:” URI scheme (see its IM URI scheme text), so that part is consistent. However, RFC 6121 is a protocol specification for XMPP instant messaging and presence, not the defining document for the “xmpp:” URI/IRI scheme. In its Section 9, RFC 6121 explicitly says that XMPP stanzas use bare addresses (JIDs) and that XMPP addresses “MUST NOT be prepended with a URI scheme”; when it discusses “xmpp:” URIs/IRIs, it refers out to a separate “XMPP-URI” specification, rather than defining the URI scheme itself.

- **E2:**

  From RFC 9734 Section 1:  A subjectAltName in these certificates can be an IM URI [RFC3860] or Extensible Messaging and Presence Protocol (XMPP) URI [RFC6121], for example. From RFC 6121 Section 9: XMPP clients are encouraged to handle addresses that are encoded as ‘xmpp:’ URIs and IRIs as specified in [XMPP‑URI] and further described in [XEP‑0147]. Also from RFC 6121 Section 9: The addresses of XMPP entities as used in communication over an XMPP network … MUST NOT be prepended with a Uniform Resource Identifier … scheme.

**Evidence Summary:**

- (E1) Demonstrates that RFC6121 does not define the xmpp: URI scheme and instead refers to a separate XMPP‑URI specification.
- (E2) Provides additional context showing how RFC6121 handles XMPP URIs, confirming the misleading citation in RFC 9734.

**Fix Direction:**

Update RFC 9734 Section 1 to correctly reference the XMPP‑URI specification (e.g., [XMPP‑URI] such as RFC 5122) for the definition of the xmpp: URI rather than RFC6121.

**Severity:** Low
  *Basis:* The issue is largely editorial with minimal impact on interoperability, as described by the Terminology Expert.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1
- Terminology Expert: Issue-1

---
