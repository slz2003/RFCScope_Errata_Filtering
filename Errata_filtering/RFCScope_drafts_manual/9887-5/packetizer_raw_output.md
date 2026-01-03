# Errata Reports

Total reports: 4

---

## Report 1: 9887-5-1

**Label:** Inconsistent Normative Levels for Wildcard Subdomain Confinement (Sections 3.4.2 vs 5.1.6)

**Bug Type:** Inconsistency

**Explanation:**

The document inconsistently applies normative language to the restriction of wildcard certificate usage—using a SHOULD in Section 3.4.2 but a MUST in Section 5.1.6—creating ambiguity over which deployments are compliant.

**Justification:**

- Section 3.4.2 mandates following RFC9525 Section 6.3 and recommends (SHOULD) confining wildcards to a subdomain dedicated to TACACS+ servers.
- Section 5.1.6 imposes an unconditional MUST to limit wildcards to a subdomain dedicated solely to TLS TACACS+ servers, leaving operators uncertain about acceptable scope.

**Evidence Snippets:**

- **E1:**

  Wildcards in TLS TACACS+ server identities… introduce security risks… To address these risks, the guidelines in Section 6.3 of [RFC9525] MUST be followed, and the wildcard SHOULD be confined to a subdomain dedicated solely to TACACS+ servers.

- **E2:**

  The use of wildcards in TLS server identities creates a single point of failure… Their use MUST follow the recommendations of Section 7.1 of [RFC9525]. Operators MUST ensure that the wildcard is limited to a subdomain dedicated solely to TLS TACACS+ servers.

**Evidence Summary:**

- (E1) Section 3.4.2 uses SHOULD-level language for confining wildcards to a TACACS+ subdomain.
- (E2) Section 5.1.6 enforces a MUST-level requirement for confining wildcards to a TLS TACACS+ subdomain.

**Fix Direction:**

Align the normative language across the sections by either uniformly using SHOULD or MUST, and clearly define whether the scope applies to all TACACS+ servers or only TLS variants.

**Severity:** Medium
  *Basis:* Inconsistent normative requirements may cause varying interpretations and compliance challenges among implementers and operators.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 2: 9887-5-2

**Label:** Unachievable 'Impervious to Redirection' Requirement for Wildcard Certificates

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that TLS TACACS+ servers using wildcard certificates must be impervious to redirection, a requirement that is practically impossible to implement or verify given typical network conditions.

**Justification:**

- Section 5.1.6 requires that servers be impervious to redirection from on-path attacks or DNS cache poisoning without specifying concrete mechanisms.
- The absolute nature of the requirement does not align with the inherent limitations of network routing and on-path attack mitigation.

**Evidence Snippets:**

- **E3:**

  Operators MUST ensure that the TLS TACACS+ servers covered by a wildcard certificate are impervious to redirection of traffic to a different server (for example, due to on-path attacks or DNS cache poisoning).

**Evidence Summary:**

- (E3) Section 5.1.6 imposes an untestable, absolute requirement for being impervious to redirection.

**Fix Direction:**

Replace the absolute requirement with language that mandates specific mitigation measures (e.g., deployment of DNSSEC or traffic pinning) or downgrade the requirement to a SHOULD.

**Severity:** Medium
  *Basis:* An unachievable requirement can lead to inconsistent security practices and confusion regarding compliance measures.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Issue on redirection
- Deontic Expert: Issue-2
- CrossRFC Expert: Issue-2

---

## Report 3: 9887-5-3

**Label:** Ambiguity in TLS Resumption Certificate Revocation Checking

**Bug Type:** Ambiguity/Underspecification

**Explanation:**

The certificate revocation check requirement during TLS resumption does not clearly specify if it applies to all forms of PSK resumption, leading to potential implementation ambiguities.

**Justification:**

- The TLS resumption text mandates that certificates be verified for revocation since the last NewSessionTicket without clarifying its applicability to both external PSKs and ticket‑based resumption methods.
- This underspecification may result in inconsistent implementation approaches and uncertainty during compliance verification.

**Evidence Snippets:**

- **E4:**

  TLS resumption text (“certificates must be verified to check for revocation during the period since the last NewSessionTicket Message”) does not clearly state whether this applies to all forms of PSK resumption (external PSKs vs ticket‑based), but it is non‑BCP14 “must” and does not directly affect protocol interoperability.

**Evidence Summary:**

- (E4) The text regarding certificate revocation in TLS resumption is ambiguous about its scope with respect to different PSK resumption methods.

**Fix Direction:**

Clarify the revocation checking requirements by explicitly stating which PSK resumption methods are covered by the mandate.

**Severity:** Low
  *Basis:* While the ambiguity may not directly impact interoperability, it can cause minor inconsistencies in implementation.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: NotedAmbiguities on TLS resumption

---

## Report 4: 9887-5-4

**Label:** Mixed Operational and Implementer Requirements in Section 5.2

**Bug Type:** Editorial/Inconsistency

**Explanation:**

Section 5.2 blends operational guidance with implementer requirements, using inconsistent normative language that may confuse stakeholders about their respective responsibilities.

**Justification:**

- The phrase 'Implementors must ensure that the configuration scheme… is straightforward' in Section 5.2 uses lower-case 'must' and mixes UI/UX design guidance with protocol-level mandates.
- This blending of roles can lead to differing interpretations of what is strictly required versus what is recommended for usability.

**Evidence Snippets:**

- **E5:**

  Operational vs implementer subjects are sometimes blended (e.g., 'Implementors must ensure that the configuration scheme… is straightforward' in Section 5.2 uses lower‑case 'must' and mixes UI/UX guidance with wire‑protocol requirements).

**Evidence Summary:**

- (E5) Section 5.2 exhibits a mix of operational and implementer mandates, leading to potential confusion over normative requirements.

**Fix Direction:**

Separate the operational guidance from the implementer requirements, using clear, consistent normative language for protocol mandates.

**Severity:** Low
  *Basis:* This issue is primarily editorial and stylistic, likely having minimal impact on technical interoperability but affecting clarity.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Operational vs implementer blending comment

---
