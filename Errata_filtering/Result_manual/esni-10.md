# Errata Reports

Total reports: 4

---

## Report 1: draft-ietf-tls-esni-25-10-1

**Label:** Ambiguous ech_required Alert Timing on Handshake Failure vs Success

**Bug Type:** Inconsistency

**Explanation:**

The specification ambiguously requires the ech_required alert to be sent when an encrypted_client_hello extension is not accepted, yet only mandates its transmission after a successful handshake, leaving the failure branch undefined.

**Justification:**

- Section 5 globally requires sending ech_required when the encrypted_client_hello extension was offered and not accepted, but Section 6.1.6 distinguishes between a success path (where the alert is sent) and a failure path (where no alert is mentioned). (E1)
- This discrepancy can lead different implementations to behave inconsistently when the handshake fails, complicating interoperability. (E2)

**Evidence Snippets:**

- **E1:**

  Section 5 (end): “This document also defines the `ech_required` alert, which the client MUST send when it offered an `encrypted_client_hello` extension that was not accepted by the server.”

- **E2:**

  Section 6.1.6 (success path): “If both authentication and the handshake complete successfully, the client MUST perform the processing described below then abort the connection with an `ech_required` alert before sending any application data to the server.”; (failure path): “If authentication or the handshake fails, the client MUST return a failure to the calling application. It MUST NOT use the retry configurations. It MUST NOT treat this as a secure signal to disable ECH.”

**Evidence Summary:**

- (E1) The global requirement in Section 5 does not differentiate based on handshake outcome.
- (E2) Section 6.1.6 clearly distinguishes a success path (with ech_required) from a failure path (without ech_required), creating ambiguity.

**Fix Direction:**

Clarify the conditions under which the ech_required alert must be sent by explicitly specifying whether it applies to failure paths as well as successful handshakes.

**Severity:** Medium
  *Basis:* The ambiguity may lead to divergent implementations and inconsistent behavior in error handling when handshakes fail.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1

---

## Report 2: draft-ietf-tls-esni-25-10-2

**Label:** Overbroad ech_required Alert Requirement Applies to GREASE ECH

**Bug Type:** Inconsistency / Underspecification

**Explanation:**

The specification mandates sending the ech_required alert whenever an encrypted_client_hello extension is not accepted, without distinguishing between real ECH offers and GREASE ECH, potentially forcing GREASE clients to abort valid TLS handshakes.

**Justification:**

- Section 5 states that the client MUST send the ech_required alert upon offering an encrypted_client_hello extension that is not accepted, with no qualifier excluding GREASE. (E1)
- Section 6.2 explicitly clarifies that GREASE ECH is not considered a real ECH offer, being merely dummy cover traffic that should not alter TLS handshake outcomes. (E2)

**Evidence Snippets:**

- **E1:**

  Section 5 (end): “This document also defines the `ech_required` alert, which the client MUST send when it offered an `encrypted_client_hello` extension that was not accepted by the server.”

- **E2:**

  Section 6.2 (GREASE ECH): “Clients of the latter type do not negotiate ECH. Instead, they generate a dummy ECH extension that is ignored by the server.” and “Offering a GREASE extension is not considered offering an encrypted ClientHello for purposes of requirements in Section 6.1.”

**Evidence Summary:**

- (E1) The global ech_required requirement in Section 5 does not differentiate between GREASE and non‑GREASE ECH offers.
- (E2) Section 6.2 explicitly states that GREASE ECH is ignored and not treated as a genuine ECH offer, which conflicts with the global mandate.

**Fix Direction:**

Amend Section 5 to narrow the ech_required requirement so that it applies only to real (non‑GREASE) ECH offers or explicitly exempt GREASE ECH from triggering the alert.

**Severity:** Low
  *Basis:* While the risk is less severe, a literal interpretation could lead to unnecessary handshake aborts in GREASE-mode, undermining its cover traffic purpose.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T2
- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- Terminology Expert: Issue-1

---

## Report 3: draft-ietf-tls-esni-25-10-3

**Label:** Incorrect Rationale for ClientHelloInner.random’s Role in Handshake Key Derivation

**Bug Type:** Descriptive Error

**Explanation:**

The draft incorrectly claims that the secrecy of ClientHelloInner.random prevents an attacker from producing the correct handshake keys, even though TLS 1.3 derives keys from the (EC)DHE secret and transcript hash rather than from any secret random value.

**Justification:**

- Section 10.12.1 asserts that “ClientHelloInner.random prevents this attack” by keeping attackers from computing the necessary handshake keys. (E1)
- In reality, TLS 1.3 key derivation relies on the (EC)DHE secret and transcript hash, and ClientHello.random is transmitted in clear text. (E2)

**Evidence Snippets:**

- **E1:**

  Section 10.12.1: “ClientHelloInner.random prevents this attack. In particular, since the attacker does not have access to this value, it cannot produce the right transcript and handshake keys needed for encrypting the Certificate message. Thus, the client will fail to decrypt the Certificate and abort the connection.”

- **E2:**

  TLS 1.3 specifies that handshake traffic keys are derived from the (EC)DHE secret (and/or PSK) and a transcript hash of the handshake messages, with ClientHello.random being sent in the clear.

**Evidence Summary:**

- (E1) The draft claims that the secrecy of ClientHelloInner.random is critical for preventing an attack.
- (E2) However, TLS 1.3 key derivation does not use ClientHelloInner.random as a secret input, relying instead on (EC)DHE and the handshake transcript.

**Fix Direction:**

Revise the explanatory text in Section 10.12.1 to accurately describe the role of ClientHelloInner.random and the actual derivation of handshake keys in TLS 1.3.

**Severity:** Low
  *Basis:* This error is confined to the non-normative rationale and does not affect the actual cryptographic processing.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Issue 2
- CrossRFC Expert: Issue-1

---

## Report 4: draft-ietf-tls-esni-25-10-4

**Label:** Inconsistent Naming: Singular 'retry_config' vs Defined 'retry_configs'

**Bug Type:** Inconsistency

**Explanation:**

The document inconsistently uses the term 'retry_config' in some sections even though the protocol element is defined as 'retry_configs', which could confuse implementers regarding the correct identifier.

**Justification:**

- The specification defines the field as 'retry_configs' within the ECHEncryptedExtensions structure. (E1)
- Later normative text refers to 'retry_config' in quotes, suggesting a distinct element, which is a naming inconsistency. (E2)

**Evidence Snippets:**

- **E1:**

  Definition of the server’s retry configurations field: “When a client offers the outer version of an 'encrypted_client_hello' extension, the server MAY include an 'encrypted_client_hello' extension in its EncryptedExtensions message … with the following payload: struct { ECHConfigList retry_configs; } ECHEncryptedExtensions;” (Section 5)

- **E2:**

  Usage in Section 6.1.6: “Clients SHOULD NOT accept 'retry_config' in response to a connection initiated in response to a 'retry_config'. Sending a 'retry_config' in this situation is a signal that the server is misconfigured…”

**Evidence Summary:**

- (E1) The field is defined as 'retry_configs' in the protocol specification.
- (E2) The later usage of 'retry_config' introduces a naming inconsistency that may lead to confusion.

**Fix Direction:**

Standardize all references to use 'retry_configs' to ensure consistency throughout the document.

**Severity:** Low
  *Basis:* This is a minor terminology issue that may cause confusion but does not affect protocol functionality.

**Confidence:** High

**Experts mentioning this issue:**

- Terminology Expert: Issue-2

---
