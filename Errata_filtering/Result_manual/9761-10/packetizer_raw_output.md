# Errata Reports

Total reports: 1

---

## Report 1: 9761-10-1

**Label:** Over‑broad and Underspecified ‘MUD URL MUST be encrypted’ Requirement Conflicts with RFC 8520 Emission Mechanisms

**Bug Type:** Both

**Explanation:**

Section 10 mandates that the MUD URL MUST be encrypted and shared only with authorized components without defining the encryption mechanism, applicable emission methods, or responsible entity, causing a conflict with RFC 8520’s clear yet unencrypted emission methods.

**Justification:**

- RFC 9761 Section 10 applies a blanket MUST for encrypting the MUD URL without qualification, which is not supported by the emission methods defined in RFC 8520. (E1)
- RFC 8520 Section 1.5 permits the MUD URL to be emitted via DHCP, LLDP, or X.509 without requiring on‐wire encryption. (E2)
- The text fails to specify which encryption techniques (e.g., TEAP/EAP-TLS, MACsec, WPA2, IPsec) or network segments must be protected, creating ambiguous implementation guidance. (E3)

**Evidence Snippets:**

- **E1:**

  RFC 9761 Section 10: “The MUD URL MUST be encrypted and shared only with the authorized components in the network (see Sections 1.5 and 1.8 of [RFC8520]) so that an on-path attacker cannot read the MUD URL and identify the IoT device.”

- **E2:**

  RFC 8520 Section 1.5: defines three ways for a device to *emit* the MUD URL: DHCP option, LLDP frame, or X.509 certificate extension, and notes “Implementors are encouraged to allow for the flexibility of how MUD URLs may be learned.” There is no requirement that these emissions are encrypted on the wire.

- **E3:**

  Without specifying the relevant layer and threat model, the scope of “MUST be encrypted” is unclear and not interoperable.

**Evidence Summary:**

- (E1) shows that RFC 9761 imposes an unconditional encryption mandate.
- (E2) demonstrates that RFC 8520 allows for unencrypted emission methods.
- (E3) highlights the lack of clarity on which encryption mechanisms and network segments must be protected.

**Fix Direction:**

Clarify the requirement by explicitly stating which emission methods are subject to encryption and specifying acceptable encryption mechanisms (e.g., TEAP/EAP-TLS, MACsec) and the responsible network component, or limit the mandate to new TEE storage contexts.


**Severity:** High
  *Basis:* The ambiguity retroactively tightens privacy guarantees and may render existing RFC 8520-compliant deployments non-compliant, potentially leading to security vulnerabilities and interoperability issues. (E1, E2, E3)

**Confidence:** High
