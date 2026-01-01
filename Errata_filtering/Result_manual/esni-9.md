# Errata Reports

Total reports: 1

---

## Report 1: draft-ietf-tls-esni-25-9-1

**Label:** Ambiguous definition of 'compliant ECH application' for mandatory HPKE cipher suite

**Bug Type:** Underspecification

**Explanation:**

The document mandates a specific HPKE cipher suite in Section 9 using the term 'compliant ECH application' without clearly defining which roles or endpoints are covered, leading to inconsistent interpretations.

**Justification:**

- Section 9 requires that 'a compliant ECH application MUST implement' a specific HPKE cipher suite, but the term is never explicitly defined, leaving open whether this applies to clients, client‑facing servers, backend servers, or higher‑level applications (E1).
- Other parts of the document refer explicitly to roles (e.g., clients and servers in Sections 6.1, 6.2, and 7.1), creating a conflict with the ambiguous 'ECH application' language (E2, E3).

**Evidence Snippets:**

- **E1:**

  Section 9: “In the absence of an application profile standard specifying otherwise, a compliant ECH application MUST implement the following HPKE cipher suite: *KEM: DHKEM(X25519, HKDF-SHA256)… KDF: HKDF-SHA256… AEAD: AES-128-GCM*.”

- **E2:**

  Clients that implement the ECH extension behave in one of two ways: either they offer a real ECH extension, as described in Section 6.1; or they send a … GREASE ECH extension, as described in Section 6.2.

- **E3:**

  Unless specified by the application using (D)TLS or externally configured, implementations MUST NOT use this mode. (Section 10.4)

**Evidence Summary:**

- (E1) Section 9 mandates the HPKE suite using the ambiguous term 'compliant ECH application'.
- (E2) The document elsewhere clearly identifies roles for ECH operation, conflicting with the ambiguous usage.
- (E3) Additional language in Section 10.4 reinforces uncertainty in which implementations the rule applies.

**Fix Direction:**

Revise Section 9 to explicitly define 'ECH application' by limiting the requirement to endpoints that perform HPKE operations (e.g., ECH-capable clients and client‑facing servers) and clarify the scope of an overriding application profile standard.

**Severity:** Low
  *Basis:* The ambiguity may lead to inconsistent implementations without posing a direct security risk.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1

---
