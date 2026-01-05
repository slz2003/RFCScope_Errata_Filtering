# Errata Reports

Total reports: 1

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