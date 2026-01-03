# Errata Reports

Total reports: 2

---

## Report 1: 9887-6-1

**Label:** Section 6 lower‑case 'must/should' used as advisory guidance instead of normative terms

**Bug Type:** Editorial

**Explanation:**

Section 6 employs lower‑case 'must' and 'should' to provide operational guidance rather than normative requirements, relying on the document’s mandate that only all‑caps keywords are binding.

**Justification:**

- Section 6 uses lower‑case 'should' and 'must' in phrases (e.g. 'TLS should be universally applied' and 'Operators must follow the recommendation...') which are intended only as non‑normative guidance because Section 2.1 reserves normative meaning to all‑caps keywords.
- The operational advice in Section 6 aligns with the normative requirements detailed in Sections 3 and 5, but the use of lower‑case may be ambiguous to readers.

**Evidence Snippets:**

- **E1:**

  The key words ‘MUST’, ‘MUST NOT’, … ‘MAY’, and ‘OPTIONAL’ in this document are to be interpreted as described in BCP 14 … when, and only when, they appear in all capitals, as shown here.

- **E2:**

  New TACACS+ production deployments SHOULD use TLS authentication and encryption.

- **E3:**

  TLS TACACS+ servers … MUST NOT allow non-TLS connections… Instead, separate non‑TLS TACACS+ servers SHOULD be set up to cater for these clients.

- **E4:**

  It is NOT RECOMMENDED that TLS TACACS+ servers and non‑TLS TACACS+ servers be deployed on the same host…

- **E5:**

  TACACS+ clients MUST NOT fail back to a non‑TLS connection if a TLS connection fails. This prohibition includes during the migration of a deployment (Section 6.1).

- **E6:**

  Section 5.2 mentions that for an optimal deployment of TLS TACACS+, TLS should be universally applied…

- **E7:**

  The period where any client is configured with both TLS and non‑TLS TACACS+ servers should be minimized.

- **E8:**

  Operators must follow the recommendation of Section 5.1.1 and deploy separate non-TLS TACACS+ servers for these non‑TLS clients from those used for the TLS clients.

**Evidence Summary:**

- (E1) The document specifies that only all‑caps keywords carry normative force, while Section 6 uses lower‑case forms.
- (E2) Examples of lower‑case usage in Section 6 (E2–E8) indicate advisory guidance rather than binding requirements.

**Severity:** Low
  *Basis:* The issue is primarily editorial and may only cause minor ambiguity for readers who do not consult Section 2.1 closely.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-1

---

## Report 2: 9887-6-2

**Label:** Inconsistency between TLS 1.3‑only mandate and BCP 195’s requirement for TLS 1.2 support

**Bug Type:** Inconsistency

**Explanation:**

RFC 9887 mandates a TLS 1.3‑only profile while referencing BCP 195, which requires new application protocols to support both TLS 1.2 and 1.3; this results in a normative inconsistency that could affect interoperability.

**Justification:**

- RFC 9887 (Section 3.2) requires a minimum of TLS 1.3 and explicitly disallows earlier versions, yet later claims that disregarding TLS 1.2 requirements is permissible under BCP 195 (Section 5.1).
- BCP 195 (RFC 9325 Section 3.1.1) clearly mandates that new application protocols must integrate with both TLS 1.2 and TLS 1.3, creating a conflict with the RFC 9887 profile.

**Evidence Snippets:**

- **E1:**

  RFC 9887 mandates “A minimum of TLS 1.3 [RFC8446] MUST be used for transport. Earlier versions of TLS MUST NOT be used.” (Section 3.2) and later states that its profile “outlines additional restrictions permissible under [BCP195]. For example, any recommendations referring to TLS 1.2, including the mandatory support, are not relevant for Secure TACACS+, as TLS 1.3 or above is mandated.” (Section 5.1). By contrast, BCP 195 / RFC 9325 states that “New application protocols that employ TLS/DTLS for channel or session encryption MUST integrate with both TLS/DTLS versions 1.2 and 1.3; nevertheless, in rare cases where broad interoperability is not a concern, application protocol designers MAY choose to forego TLS 1.2.” (Section 3.1.1) and generally expects new application protocols to conform to its best practices (Section 5).

**Evidence Summary:**

- (E1) RFC 9887 mandates only TLS 1.3 while BCP 195 requires that new protocols support both TLS 1.2 and 1.3, highlighting a normative conflict.

**Severity:** Medium
  *Basis:* This normative inconsistency may lead to non‑conformance with BCP 195 and potential interoperability issues, particularly given the importance of dual TLS support in AAA protocols.

**Confidence:** Medium

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1

---
