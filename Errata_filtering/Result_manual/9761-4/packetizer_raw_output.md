# Errata Reports

Total reports: 3

---

## Report 1: 9761-4-1

**Label:** Misstatement of (D)TLS 1.3 Handshake Encryption Boundary and EncryptedExtensions

**Bug Type:** Inconsistency

**Explanation:**

The document contains conflicting statements about which handshake messages are encrypted – claiming all messages except ClientHello are encrypted while later indicating that both ClientHello and ServerHello expose visible fields – and mischaracterizes the role of EncryptedExtensions by suggesting its use triggers encryption.

**Justification:**

- RFC 9761 Section 4 first states that 'all (D)TLS handshake messages excluding the ClientHello message are encrypted' but then notes that 'the ClientHello and ServerHello still have some fields visible' (E1, E2).
- RFC 8446 specifies that 'all handshake messages after the ServerHello are now encrypted', meaning the visible messages should be ClientHello and ServerHello, not just ClientHello (E3).

**Evidence Snippets:**

- **E1:**

  In (D)TLS 1.3, full (D)TLS handshake inspection is not possible since all (D)TLS handshake messages excluding the ClientHello message are encrypted.  (D)TLS 1.3 has introduced new extensions in the handshake record layers called Encrypted Extensions.  When using these extensions, handshake messages will be encrypted…

- **E2:**

  However, the ClientHello and ServerHello still have some fields visible, such as the list of supported versions, named groups, cipher suites, signature algorithms, extensions in ClientHello, and the chosen cipher in the ServerHello.

- **E3:**

  All handshake messages after the ServerHello are now encrypted.  The newly introduced EncryptedExtensions message allows various extensions previously sent in the clear in the ServerHello to also enjoy confidentiality protection.

**Evidence Summary:**

- (E1) The spec claims only ClientHello is unencrypted, while (E2) shows visible ServerHello fields.
- (E3) RFC 8446 clarifies that encryption applies to messages after ServerHello, contradicting the earlier claim.

**Fix Direction:**

Update Section 4 to state that in (D)TLS 1.3 only the handshake messages after the ServerHello (and HelloRetryRequest in DTLS) are encrypted, and clarify that EncryptedExtensions is a handshake message that is sent under encryption rather than a trigger for encryption.


**Severity:** Medium
  *Basis:* This inconsistency could mislead implementers regarding which handshake parameters are observable for middlebox inspection, even though it does not break protocol operation.

**Confidence:** High

---

## Report 2: 9761-4-2

**Label:** Misassigned ECH Behavior for ECH-unaware TLS Proxies

**Bug Type:** Both

**Explanation:**

The document instructs a (D)TLS 1.3 proxy that does not support the ECH extension to follow behavior defined in Section 6.1.6 of TLS-ESNI, even though that behavior is designed for ECH‐aware endpoints, resulting in an inconsistent and underspecified guidance for proxy implementations.

**Justification:**

- Section 4.1 states that 'The middlebox acting as a (D)TLS 1.3 proxy that does not support the ECH extension will act as if it is connecting to the public name and follows the behavior discussed in Section 6.1.6 of [TLS-ESNI] to securely signal the client to disable ECH' (E1).
- TLS‑ESNI §6.1.6 describes procedures expecting ECH support, such as handling the 'encrypted_client_hello' extension and sending ECH-specific alerts, which an ECH-unaware proxy cannot implement (E2).

**Evidence Snippets:**

- **E1:**

  The middlebox acting as a (D)TLS 1.3 proxy that does not support the ECH extension will act as if it is connecting to the public name and follows the behavior discussed in Section 6.1.6 of [TLS-ESNI] to securely signal the client to disable ECH.

- **E2:**

  TLS‑ESNI §6.1.6: “If the server rejects ECH, the client proceeds with the handshake, authenticating for ECHConfig.contents.public_name as described in Section 6.1.7. If authentication or the handshake fails, the client MUST return a failure to the calling application… If the server provided ‘retry_configs’… the client … SHOULD retry the handshake…”

**Evidence Summary:**

- (E1) The spec directs an ECH-unaware proxy to implement behavior that assumes ECH support.
- (E2) The referenced TLS‑ESNI section is meant for endpoints that can process ECH-specific extensions and alerts.

**Fix Direction:**

Clarify that only proxies which implement the ECH extension may follow Section 6.1.6, and provide separate, unambiguous guidance for proxies that do not support ECH.


**Severity:** High
  *Basis:* This misassignment may lead to non-interoperable behavior and failed handshakes when ECH is deployed, as it forces an unsupported role on certain proxies.

**Confidence:** High

---

## Report 3: 9761-4-3

**Label:** JSON MUD Example Node Naming Inconsistency with YANG Schema

**Bug Type:** Inconsistency

**Explanation:**

The JSON example in Section 7 uses node names that differ from those defined in the normative YANG module, risking validation failures and interoperability issues.

**Justification:**

- The YANG module defines names such as 'client-profiles', 'tls-dtls-profile', 'supported-tls-version', 'extension-type', and 'supported-group', but the JSON example uses 'client-profile', 'tls-dtls-profiles', 'supported-tls-versions', 'extension-types', and 'supported-groups' (E1).
- This naming discrepancy means that JSON instance documents produced from the example will not validate against the YANG schema (E2).

**Evidence Snippets:**

- **E1:**

  JSON example in Section 7:
"ietf-acl-tls:client-profile" : {
  "tls-dtls-profiles" : [ {
    "supported-tls-versions" : ["tls13"],
    "cipher-suite" : [4865, 4866],
    "extension-types" : [10,11,13,16,24],
    "supported-groups" : [29]
  } ]
}

- **E2:**

  YANG module definition in Section 5.1/5.2:
container client-profiles
list tls-dtls-profile
leaf-list supported-tls-version
leaf-list extension-type
leaf-list supported-group

**Evidence Summary:**

- (E1) The JSON example uses node names like 'client-profile' and pluralized variants that differ from the YANG schema.
- (E2) The normative YANG module specifies singular/plural forms such as 'client-profiles' and 'tls-dtls-profile', creating a direct naming mismatch.

**Fix Direction:**

Update the JSON example to use the exact node names defined in the YANG module (e.g., 'ietf-acl-tls:client-profiles', 'tls-dtls-profile', 'supported-tls-version', 'extension-type', 'supported-group').


**Severity:** High
  *Basis:* Incorrect node naming in the example leads to JSON files that do not conform to the schema, resulting in interoperability and validation issues.

**Confidence:** High

---
