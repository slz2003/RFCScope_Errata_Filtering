# Errata Reports

Total reports: 2

---

## Report 1: draft-ietf-tls-esni-25-4-1

**Label:** Incorrect ECHConfig Field Paths in Client Selection and HPKE Setup

**Bug Type:** Inconsistency

**Explanation:**

Normative text in Section 6.1 refers to non‐existent field paths such as ECHConfig.contents.kem_id and ECHConfig.contents.public_key, whereas the correct structure defined in Section 4 places these fields within ECHConfig.contents.key_config and ECHConfig.version.

**Justification:**

- Section 4 defines the ECHConfig, ECHConfigContents, and HpkeKeyConfig structures with HPKE fields nested inside 'key_config' and version at the top-level, but Section 6.1 incorrectly uses field references like ECHConfig.contents.kem_id, ECHConfig.contents.cipher_suites, and ECHConfig.contents.public_key.
- Multiple experts (Scope, Causal, Structural, Deontic, and Terminology) have identified this as an editorial inconsistency that, if read literally, could lead to confusion and non‐interoperable implementations.

**Evidence Snippets:**

- **E1:**

  struct {
  HpkeKeyConfig key_config;
  uint8 maximum_name_length;
  opaque public_name<1..255>;
  ECHConfigExtension extensions<0..2^16-1>;
} ECHConfigContents;

struct {
  uint8 config_id;
  HpkeKemId kem_id;
  HpkePublicKey public_key;
  HpkeSymmetricCipherSuite cipher_suites<4..2^16-4>;
} HpkeKeyConfig;

struct {
  uint16 version;
  uint16 length;
  select (ECHConfig.version) {
    case 0xfe0d: ECHConfigContents contents;
  }
} ECHConfig;

- **E2:**

  To determine if a given ECHConfig is suitable, it checks that it supports the KEM algorithm identified by ECHConfig.contents.kem_id, at least one KDF/AEAD algorithm identified by ECHConfig.contents.cipher_suites, and the version of ECH indicated by ECHConfig.contents.version.

pkR = DeserializePublicKey(ECHConfig.contents.public_key)

**Evidence Summary:**

- (E1) Shows the defined wire format where the HPKE parameters reside in 'key_config' and the version is at the top level.
- (E2) Displays the incorrect field references used in Section 6.1 for client selection and HPKE setup.

**Fix Direction:**

Revise Section 6.1 so that all field references match the defined structure, e.g., use ECHConfig.contents.key_config.kem_id, ECHConfig.contents.key_config.cipher_suites, ECHConfig.version, and ECHConfig.contents.key_config.public_key.

**Severity:** Medium
  *Basis:* While the error is editorial and does not change on‐wire behavior, it could mislead implementers and result in non‐interoperable code if interpreted literally.

**Confidence:** High



---

## Report 2: draft-ietf-tls-esni-25-4-2

**Label:** Underspecified Client Behavior for Invalid HPKE Public Key in ECHConfig

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state how a client must behave if the HPKE public key (obtained via DeserializePublicKey) is invalid or if HPKE setup fails, leading to ambiguity in error handling.

**Justification:**

- The spec mandates checks for supported KEMs, cipher suites, and versions but does not cover the case when DeserializePublicKey fails for a malformed public_key.
- Multiple experts (Causal, Deontic, CrossRFC, and Boundary) note that this omission could lead to divergent client behavior in processing malformed or adversarially supplied ECHConfigs.

**Evidence Snippets:**

- **E3:**

  pkR = DeserializePublicKey(ECHConfig.contents.public_key)

Suppose the public key bytes are invalid for that KEM. Then, DeserializePublicKey fails (per RFC 9180) and the ECH spec does not specify what to do next.

- **E4:**

  The ECH draft does not say what a client is required to do when, for an ostensibly supported KEM, DeserializePublicKey (or a subsequent HPKE operation such as SetupBaseS) fails due to a malformed or invalid public_key.

**Evidence Summary:**

- (E3) Illustrates the scenario where DeserializePublicKey fails without defined guidance.
- (E4) Highlights the ambiguity in client behavior when HPKE key deserialization or setup fails.

**Fix Direction:**

Add explicit normative text specifying that if deserialization of the HPKE public key (or subsequent HPKE setup) fails, the client MUST treat that ECHConfig as invalid, ignore it, and proceed according to the GREASE or no‐compatible-configuration procedures.

**Severity:** Medium
  *Basis:* This underspecification could lead to inconsistent client behavior under misconfiguration or attack, potentially affecting downgrade resistance and interoperability, even though core security is maintained.

**Confidence:** High



---
