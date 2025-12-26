# Errata Reports

Total reports: 2

---

## Report 1: 9761-3-1

**Label:** Ambiguous Is-Supported Attribute Role (Manufacturer vs. MUD Manager)

**Bug Type:** Both

**Explanation:**

RFC 9761 ambiguously describes the role of the 'is-supported' attribute by suggesting that the MUD manager indicates that the IoT manufacturer no longer supports the device, which conflicts with RFC 8520’s definition where the flag is set by the manufacturer.

**Justification:**

- RFC 9761 Section 3 implies the MUD manager can indicate that the manufacturer no longer supports the device, which is inconsistent with the manufacturer-originated signal defined in RFC 8520.
- This inversion of roles may lead to divergent interpretations and implementations regarding who sets versus who interprets the flag.

**Evidence Snippets:**

- **E1:**

  RFC 9761 Section 3: “the ‘is-supported’ attribute defined in Section 3.6 of [RFC8520] can be used by the MUD manager to indicate that the IoT manufacturer no longer supports the device.”

- **E2:**

  RFC 8520 Section 3.6: “This boolean is an indication from the manufacturer to the network administrator as to whether or not the Thing is supported.”

- **E3:**

  RFC 8520 Section 2.1 model: the MUD file is authored by the manufacturer, retrieved and processed by a MUD manager.

**Evidence Summary:**

- (E1) RFC 9761 implies that the MUD manager uses the flag to indicate lack of support.
- (E2) RFC 8520 clearly defines the flag as a manufacturer-originated indication.
- (E3) RFC 8520’s model reinforces that the manufacturer authors the MUD file.

**Fix Direction:**

Clarify the phrasing so that the MUD manager only uses the manufacturer-set 'is-supported' flag as an indication of device support status without modifying it.


**Severity:** High
  *Basis:* The ambiguity in role assignment can lead to inconsistent implementations and operational confusion in network deployments.

**Confidence:** High

---

## Report 2: 9761-3-2

**Label:** Overbroad DDoS Mitigation Claim via Trust-Anchor Enforcement

**Bug Type:** Underspecification/Inconsistency

**Explanation:**

The document overstates the effectiveness of trust-anchor enforcement by implying that verifying the victim's server certificate is sufficient to block specific DDoS attacks such as Slowloris and TLS renegotiation.

**Justification:**

- RFC 9761 Section 3 claims that DDoS attacks like Slowloris and TLS renegotiation can be blocked if the victim's server certificate is not signed by a trusted CA, which is misleading.
- The trust-anchor mechanism only enforces certificate validation and does not address the actual connection-level or application-layer behaviors exploited by these DDoS attacks.

**Evidence Snippets:**

- **E1:**

  RFC 9761 Section 3: “For example, DDoS attacks like Slowloris [SLOWLORIS ] and Transport Layer Security (TLS) re-negotiation can be blocked if the victim's server certificate is not be signed by the same certifying authorities trusted by the IoT device.”

- **E2:**

  RFC 9761 Section 5.1 (trust anchors bullet): “List of trust anchor certificates used by the IoT device. If the server certificate is signed by one of the trust anchors, the middlebox continues with the connection as normal. Otherwise, the middlebox will react as if the server certificate validation has failed and takes appropriate action (e.g, blocks the (D)TLS session). … This empowers the middlebox to reject TLS sessions to servers that the IoT device does not trust.”

- **E3:**

  Slowloris (per supplied summary) is an application-layer slow-HTTP DoS that depends on keeping HTTP connections open by dribbling headers, not on certificate trust; TLS renegotiation DDoS relies on inducing many expensive renegotiations, again orthogonal to the CA that signed the server certificate.

**Evidence Summary:**

- (E1) RFC 9761 Section 3 suggests that blocking untrusted certificates will stop specific DDoS attacks.
- (E2) Section 5.1 outlines a generic certificate-based blocking mechanism.
- (E3) The attack descriptions clarify that Slowloris and TLS renegotiation strategies are not inherently mitigated by certificate trust checks.

**Fix Direction:**

Revise the DDoS examples to state that trust-anchor enforcement only blocks TLS/DTLS sessions with untrusted certificates and does not specifically mitigate the tactics of Slowloris or TLS renegotiation attacks.


**Severity:** Medium
  *Basis:* The misleading example may cause operators to overestimate the security benefits of the mechanism, although it does not impede protocol interoperability.

**Confidence:** Medium

---
