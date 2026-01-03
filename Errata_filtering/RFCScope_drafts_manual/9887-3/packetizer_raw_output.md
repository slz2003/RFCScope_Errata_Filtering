# Errata Reports

Total reports: 4

---

## Report 1: 9887-3-1

**Label:** Ambiguous TLS Resumption Revocation Checking Requirement

**Bug Type:** Underspecification

**Explanation:**

The specification mandates certificate revocation checks during TLS resumption without clarifying exactly when or how these checks should be performed, particularly in scenarios where no certificates are available (e.g. PSK-only sessions).

**Justification:**

- Section 3.6 instructs that, 'When processing TLS resumption, certificates must be verified to check for revocation during the period since the last NewSessionTicket Message,' yet the text does not explain how to perform this check when a resumption handshake does not include certificate exchange.
- Multiple experts note that applying an unconditional certificate revocation check creates ambiguity for PSK-only (or non‐X.509) deployment scenarios.

**Evidence Snippets:**

- **E1:**

  When processing TLS resumption, certificates must be verified to check for revocation during the period since the last NewSessionTicket Message. (Section 3.6)

- **E2:**

  RFC 9887’s resumption section states unconditionally: “When processing TLS resumption, certificates must be verified to check for revocation during the period since the last NewSessionTicket Message.” (BoundaryExpert Finding-1)

**Evidence Summary:**

- (E1) Mandated revocation checking during resumption is stated in Section 3.6.
- (E2) The unconditional nature of this requirement is highlighted as an ambiguity in the boundary analysis.

**Fix Direction:**

Clarify and condition the TLS resumption revocation check requirement so that it applies only to sessions where certificate-based authentication is used, and specify the intended behavior for PSK-only or other non‐X.509 scenarios.

**Severity:** Low
  *Basis:* The ambiguity only affects non-standard authentication modes and leads to divergent interpretations without an immediate security breakdown in most deployments.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T1
- Scope: Issue-2
- Boundary: Finding-1

---

## Report 2: 9887-3-2

**Label:** Inconsistent Wildcard Subdomain Scoping Requirements

**Bug Type:** Inconsistency

**Explanation:**

The specification provides conflicting normative levels for limiting wildcard certificates to a TACACS+-dedicated subdomain, using a SHOULD in one section and a MUST in another, which causes ambiguity between implementation and operational enforcement.

**Justification:**

- Section 3.4.2 states that wildcards SHOULD be confined to a subdomain dedicated solely to TACACS+ servers, while Section 5.1.6 mandates that operators MUST ensure the wildcard is limited to such a subdomain.
- This discrepancy in requirement strengths may result in divergent interpretations by implementers and operators.

**Evidence Snippets:**

- **E1:**

  Wildcards in TLS TACACS+ server identities … To address these risks, the guidelines in Section 6.3 of [RFC9525] MUST be followed, and the wildcard SHOULD be confined to a subdomain dedicated solely to TACACS+ servers. (Section 3.4.2)

- **E2:**

  The use of wildcards in TLS server identities creates a single point of failure… Their use MUST follow the recommendations of Section 7.1 of [RFC9525]. Operators MUST ensure that the wildcard is limited to a subdomain dedicated solely to TLS TACACS+ servers. (Section 5.1.6)

**Evidence Summary:**

- (E1) Section 3.4.2 uses a SHOULD for confining wildcards to a dedicated subdomain.
- (E2) Section 5.1.6 uses a MUST for the same requirement, creating a normative inconsistency.

**Fix Direction:**

Align the normative language by clearly distinguishing between implementation (client-side verification) and operational (deployment configuration) requirements, and use a consistent requirement (preferably MUST for operational constraints) across the document.

**Severity:** Medium
  *Basis:* The conflicting guidance may lead to divergent certificate validation practices and inconsistent security postures across deployments.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-1
- Deontic: Issue-1
- CrossRFC: Issue-1

---

## Report 3: 9887-3-3

**Label:** Ambiguous Connection Termination Behavior for TAC_PLUS_UNENCRYPTED_FLAG Errors in Single Connection Mode

**Bug Type:** Underspecification

**Explanation:**

The specification is unclear about how to manage the underlying TLS/TCP connection when a TAC_PLUS_UNENCRYPTED_FLAG error occurs in Single Connection Mode, leaving it ambiguous whether the connection should be terminated immediately or after existing sessions complete.

**Justification:**

- The text mandates that when the TAC_PLUS_UNENCRYPTED_FLAG bit is not set to 1, the connection’s associated TACACS+ session must be terminated, yet it is silent on how the underlying connection is to be managed in Single Connection Mode.
- There is conflicting guidance comparing non-Single Connection Mode (immediate connection closure) with Single Connection Mode behaviors described in RFC 8907.

**Evidence Snippets:**

- **E1:**

  A TLS TACACS+ server that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 over a TLS connection MUST return an error ... and terminate the session. This behavior corresponds to that defined in Section 4.5 of [RFC8907] regarding data obfuscation for TAC_PLUS_UNENCRYPTED_FLAG or key mismatches. (Section 4, RFC 9887)

- **E2:**

  If Single Connection Mode was enabled, but an ERROR occurred due to connection issues (such as an incorrect secret (see Section 4.5)), then any further new sessions MUST NOT be accepted on the connection. If there are any sessions that have already been established, then they MAY be completed. Once all active sessions are completed, then the connection MUST be closed. (Section 4.4, RFC 8907)

**Evidence Summary:**

- (E1) The requirement to terminate a session on TAC_PLUS_UNENCRYPTED_FLAG errors is given for TLS connections.
- (E2) RFC 8907 outlines a behavior in Single Connection Mode that may allow existing sessions to complete before closing the connection, creating ambiguity.

**Fix Direction:**

Explicitly specify the connection management rules for Single Connection Mode after a TAC_PLUS_UNENCRYPTED_FLAG error, clarifying whether the connection is to be closed immediately or after active sessions are concluded.

**Severity:** Low
  *Basis:* The ambiguity impacts error recovery and connection lifecycle management, which might lead to divergent implementations in resource handling.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T2

---

## Report 4: 9887-3-4

**Label:** Unclear Scope of Certificate Verification in RPK-based Authentication Deployments

**Bug Type:** Underspecification

**Explanation:**

The specification mandates certificate verification (including revocation checks) during TLS resumption without clarifying whether these requirements apply only to certificate-based (X.509) authentication, leaving the applicability to RPK or external PSK deployments ambiguous.

**Justification:**

- Section 3.3 and 3.5 distinguish between certificate-based mutual authentication and alternative mechanisms such as PSKs and RPKs, yet Section 3.6 imposes certificate verification requirements unconditionally.
- The document explicitly states that it is not fully clear whether the generic 'certificates' requirements are intended to apply to RPK deployments, creating uncertainty for implementers using non–X.509 methods.

**Evidence Snippets:**

- **E1:**

  In addition to certificate-based TLS authentication, implementations MAY support the following alternative authentication mechanisms: * Pre-Shared Keys (PSKs)… * Raw Public Keys (RPKs). (Sections 3.3 and 3.5)

- **E2:**

  When processing TLS resumption, certificates must be verified to check for revocation during the period since the last NewSessionTicket Message. (Section 3.6)

- **E3:**

  It is not fully clear whether those generic “certificates” requirements are intended to apply to RPK deployments as well, or only to X.509-based authentication. (Scope Expert, ResidualUncertainties)

**Evidence Summary:**

- (E1) Alternative authentication mechanisms such as RPKs are allowed alongside certificate-based methods.
- (E2) Certificate verification is mandated during TLS resumption.
- (E3) The applicability of certificate requirements to RPK deployments is explicitly noted as unclear.

**Fix Direction:**

Clarify that certificate verification and associated revocation checks apply exclusively to X.509-based authentication, or provide separate guidelines for deployments using RPKs and external PSKs.

**Severity:** Low
  *Basis:* This ambiguity is limited to non–X.509 authentication deployments and is unlikely to affect most implementations but may cause confusion for those opting for RPK.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope: ResidualUncertainties

---
