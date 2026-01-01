# Errata Reports

Total reports: 3

---

## Report 1: draft-ietf-tls-esni-25-6-1

**Label:** Incorrect Field References for ECHConfig in Section 6

**Bug Type:** Inconsistency

**Explanation:**

Section 6 uses incorrect field paths such as ECHConfig.contents.kem_id, ECHConfig.contents.cipher_suites, and ECHConfig.contents.public_key instead of the properly nested fields defined in Section 4.

**Justification:**

- Section 6.1 states: “To determine if a given ECHConfig is suitable, it checks that it supports the KEM algorithm identified by ECHConfig.contents.kem_id, at least one KDF/AEAD algorithm identified by ECHConfig.contents.cipher_suites, and the version of ECH indicated by ECHConfig.contents.version.” yet Section 4’s struct definitions place these values inside ECHConfig.contents.key_config and as a top‐level field for version.
- The HPKE setup pseudocode also erroneously uses “pkR = DeserializePublicKey(ECHConfig.contents.public_key)” instead of referencing the nested public_key in key_config.

**Evidence Snippets:**

- **E1:**

  Struct definitions in Section 4:

        - Top-level ECHConfig:  
          struct { uint16 version; uint16 length; select (ECHConfig.version) { case 0xfe0d: ECHConfigContents contents; } } ECHConfig;

        - ECHConfigContents:  
          struct { HpkeKeyConfig key_config; uint8 maximum_name_length; opaque public_name<1..255>; ECHConfigExtension extensions<0..2^16-1>; } ECHConfigContents;

        - HpkeKeyConfig:  
          struct { uint8 config_id; HpkeKemId kem_id; HpkePublicKey public_key; HpkeSymmetricCipherSuite cipher_suites<4..2^16-4>; } HpkeKeyConfig;

- **E2:**

  Section 6.1 text:
“To determine if a given ECHConfig is suitable, it checks that it supports the KEM algorithm identified by ECHConfig.contents.kem_id, at least one KDF/AEAD algorithm identified by ECHConfig.contents.cipher_suites, and the version of ECH indicated by ECHConfig.contents.version.”

HPKE setup:
pkR = DeserializePublicKey(ECHConfig.contents.public_key)

**Evidence Summary:**

- (E1) Provides the normative struct definitions showing that HPKE parameters are nested in key_config and version is at the top level.
- (E2) Shows the incorrect references in Section 6 that do not match the defined structure.

**Fix Direction:**

Replace each reference in Section 6 with the correctly scoped fields: use ECHConfig.contents.key_config.kem_id, ECHConfig.contents.key_config.cipher_suites, ECHConfig.contents.key_config.public_key, and use ECHConfig.version for the version.

**Severity:** Medium
  *Basis:* While the on‐wire encoding remains defined by Section 4, the normative text in Section 6 may confuse implementers and lead to misinterpretation of the intended field locations.

**Confidence:** High

**Experts mentioning this issue:**

- ScopeExpert: Issue-1
- CausalExpert: Issue-1
- StructuralExpert: Issue-1
- TerminologyExpert: Issue-1

---

## Report 2: draft-ietf-tls-esni-25-6-2

**Label:** Misnamed 'retry_config' vs 'retry_configs' in Retry Configuration Handling

**Bug Type:** Inconsistency

**Explanation:**

Section 6.1.6 inconsistently uses the singular 'retry_config' while the defined payload and other references use the plural 'retry_configs', creating ambiguity in the protocol specification.

**Justification:**

- Section 5 defines the payload as “struct { ECHConfigList retry_configs; } ECHEncryptedExtensions;” and other parts of Section 6.1.6 refer correctly to “retry_configs”, but a normative sentence uses the undefined term “retry_config”.
- This discrepancy may lead implementers to misinterpret the intended retry handling rules.

**Evidence Snippets:**

- **E3:**

  Definition of the payload in Section 5:

“struct { ECHConfigList retry_configs; } ECHEncryptedExtensions;”

and correct plural usage in Section 6.1.6:
“If the server provided "retry_configs" and if at least one of the values contains a version supported by the client, the client can regard the ECH configuration as securely replaced by the server. It SHOULD retry the handshake…”

- **E4:**

  Normative singular usage in Section 6.1.6:

“Clients SHOULD NOT accept "retry_config" in response to a connection initiated in response to a "retry_config".”

**Evidence Summary:**

- (E3) Shows that the defined field is 'retry_configs' and the majority of the text uses this term correctly.
- (E4) Demonstrates the isolated use of the singular 'retry_config', which is not defined anywhere else.

**Fix Direction:**

Replace all instances of 'retry_config' in Section 6.1.6 with 'retry_configs' to match the defined payload and maintain consistency.

**Severity:** Low
  *Basis:* This error is primarily editorial; while it may cause temporary confusion, it is easily correctable and unlikely to affect interoperability significantly.

**Confidence:** High

**Experts mentioning this issue:**

- DeonticExpert: Issue-1
- StructuralExpert: Issue-2
- TerminologyExpert: Issue-2

---

## Report 3: draft-ietf-tls-esni-25-6-3

**Label:** Ambiguous Application of ech_required Alert for GREASE ECH

**Bug Type:** Underspecification

**Explanation:**

The specification unconditionally mandates the ech_required alert when an encrypted_client_hello extension is unaccepted, yet it does not clearly distinguish between real ECH offers and GREASE ECH offers, potentially leading to divergent client behavior.

**Justification:**

- Section 5 requires that the client send an ech_required alert when an encrypted_client_hello extension is not accepted by the server.
- Section 6.2.1 explicitly notes that GREASE ECH is not considered an offer for ECH and should be ignored, creating ambiguity in whether the ech_required alert should be sent when GREASE is used.

**Evidence Snippets:**

- **E5:**

  Section 5: “This document also defines the ech_required alert, which the client MUST send when it offered an encrypted_client_hello extension that was not accepted by the server.”

Section 6.2.1 (GREASE client): … Offering a GREASE extension is not considered offering an encrypted ClientHello for purposes of requirements in Section 6.1.

**Evidence Summary:**

- (E5) Highlights the conflicting requirements between the general ech_required mandate in Section 5 and the GREASE-specific exception in Section 6.2.1.

**Fix Direction:**

Clarify the text to specify that the ech_required alert applies only to real ECH offers as described in Section 6.1, and not to GREASE ECH offers.

**Severity:** Medium
  *Basis:* Misinterpretation of this requirement could lead GREASE clients to mistakenly abort connections, affecting interoperability and undermining the GREASE mechanism.

**Confidence:** High

**Experts mentioning this issue:**

- BoundaryExpert: Finding-1

---
