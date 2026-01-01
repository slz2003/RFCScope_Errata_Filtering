# Errata Reports

Total reports: 7

---

## Report 1: draft-ietf-tls-esni-25-5-1

**Label:** Ambiguous Timing for Sending ech_required Alert in ECH Rejection

**Bug Type:** Underspecification

**Explanation:**

The document presents conflicting guidance on when to send the ech_required alert in the ECH rejection scenario, making it unclear whether the client should abort immediately upon detecting rejection or only after a successful outer handshake.

**Justification:**

- Section 5 instructs clients to send ech_required immediately when an 'encrypted_client_hello' is not accepted, while Section 6.1.6 details a sequence that requires completing the outer handshake before sending the alert.
- Implementations that follow a naive interpretation risk aborting before processing retry_configs and other recovery steps.

**Evidence Snippets:**

- **E1:**

  This document also defines the ‘ech_required’ alert, which the client MUST send when it offered an ‘encrypted_client_hello’ extension that was not accepted by the server. (Section 5)

- **E2:**

  If both authentication and the handshake complete successfully, the client MUST perform the processing described below then abort the connection with an ‘ech_required’ alert before sending any application data to the server. (Section 6.1.6)

**Evidence Summary:**

- (E1) Section 5 mandates sending ech_required upon non-acceptance of the extension.
- (E2) Section 6.1.6 specifies that ech_required is to be sent only after a successful outer handshake.

**Fix Direction:**

Harmonize Section 5 with Section 6.1.6 by explicitly defining the complete sequence for sending ech_required so that it applies only after a successful outer handshake when appropriate.

**Severity:** Low
  *Basis:* Though the ambiguous timing may lead to interoperability differences, it does not directly compromise security.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T1

---

## Report 2: draft-ietf-tls-esni-25-5-2

**Label:** Ambiguous Handling of Second ClientHelloOuter in ECH-Rejected HRR Flow

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state whether the outer ‘encrypted_client_hello’ extension should be retained or removed in the second ClientHello after an ECH-rejecting HelloRetryRequest.

**Justification:**

- TLS 1.3 requires that the second ClientHello be sent without modification except for allowed changes, but no explicit instruction is given for handling the outer ECH extension in the rejection case.
- The lack of a rule may lead to varied client implementations and observable differences in wire behavior.

**Evidence Snippets:**

- **E1:**

  the client MUST send the same ClientHello without modification, except as follows: … Other modifications that may be allowed by an extension defined in the future and present in the HelloRetryRequest. (TLS 1.3 Section 4.1.4)

- **E2:**

  If the client-facing server rejected ECH, or if the first ClientHello did not include an ‘encrypted_client_hello’ extension, the client-facing server proceeds with the connection as usual. The server does not decrypt the second ClientHello's ECHClientHello.payload value, if there is one. (Section 7.1.1)

**Evidence Summary:**

- (E1) TLS 1.3 mandates minimal modifications on a second ClientHello but does not clarify ECH-specific handling.
- (E2) The server tolerates both presence and absence of the ECH extension, highlighting the ambiguity.

**Fix Direction:**

Clarify the intended client behavior in the ECH rejection flow by explicitly specifying whether the outer encrypted_client_hello extension should be retained or removed in the second ClientHello.

**Severity:** Low
  *Basis:* This ambiguity may lead to minor differences in wire behavior without directly impacting security.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T2

---

## Report 3: draft-ietf-tls-esni-25-5-3

**Label:** Ambiguous ech_required Alert Requirements for GREASE and Real ECH

**Bug Type:** Underspecification

**Explanation:**

There is ambiguity regarding the applicability of the ech_required alert: while Section 5 mandates sending it when an encrypted_client_hello extension is not accepted, GREASE clients (as per Section 6.2 and related text) are not intended to negotiate ECH and should not trigger this alert.

**Justification:**

- Section 5 states that the client MUST send ech_required when it offered an encrypted_client_hello that was not accepted, without distinguishing between real ECH and GREASE.
- Sections 6.2 and associated passages clarify that GREASE extensions are not considered genuine ECH offers, leading to a normative conflict.
- Divergent interpretations may cause some clients to inappropriately abort GREASE handshakes with ech_required.

**Evidence Snippets:**

- **E1:**

  This document also defines the ‘ech_required’ alert, which the client MUST send when it offered an ‘encrypted_client_hello’ extension that was not accepted by the server. (Section 5)

- **E2:**

  Offering a GREASE extension is not considered offering an encrypted ClientHello for purposes of requirements in Section 6.1. (Section 6.2.1)

- **E3:**

  Note that decryption failure could indicate a GREASE ECH extension (see Section 6.2)… servers can measure occurrences of the ‘ech_required’ alert to detect this case. (Deontic Analysis)

- **E4:**

  Clients that implement the ECH extension behave in one of two ways: either they offer a real ECH extension, as described in Section 6.1; or they send a … GREASE … ECH extension, as described in Section 6.2. (Scope Analysis)

**Evidence Summary:**

- (E1) Section 5 imposes an unconditional requirement to send ech_required on non-acceptance.
- (E2) Section 6.2.1 explicitly exempts GREASE extensions from being treated as genuine offers.
- (E3) The text highlights potential misdiagnosis of GREASE failures using ech_required.
- (E4) The dual behavior of clients is described, reinforcing the conflict.

**Fix Direction:**

Revise Section 5 to explicitly restrict the ech_required requirement to real ECH negotiations, or add an explicit exemption for GREASE mode in Section 6.2.

**Severity:** Medium
  *Basis:* The ambiguity may lead to diverging client behaviors and misdiagnosis of ECH configuration issues, affecting interoperability and diagnostic accuracy.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-1
- Deontic: Issue-1
- Terminology: Issue-1
- Boundary: Finding-1

---

## Report 4: draft-ietf-tls-esni-25-5-4

**Label:** Inconsistent ech_required Alert Use in Failure Paths

**Bug Type:** Normative Conflict

**Explanation:**

The specification mandates sending the ech_required alert in all cases of unaccepted ECH per Section 5, yet Section 6.1.6 requires it only after a successful outer handshake, creating ambiguity for cases where the handshake fails.

**Justification:**

- Section 5 imposes an unconditional requirement to send ech_required when the encrypted_client_hello is not accepted.
- Section 6.1.6 differentiates between successful outer-handshake cases (which require ech_required) and failure cases (which use standard TLS alerts), resulting in conflicting directives.
- This inconsistency can lead to varied error signaling in different failure scenarios.

**Evidence Snippets:**

- **E1:**

  This document also defines the ‘ech_required’ alert, which the client MUST send when it offered an ‘encrypted_client_hello’ extension that was not accepted by the server. (Section 5)

- **E2:**

  If authentication or the handshake fails, the client MUST return a failure to the calling application. It MUST NOT use the retry configurations. It MUST NOT treat this as a secure signal to disable ECH. (Section 6.1.6)

- **E3:**

  If both authentication and the handshake complete successfully, the client MUST perform the processing described below then abort the connection with an ‘ech_required’ alert before sending any application data to the server. (Section 6.1.6)

**Evidence Summary:**

- (E1) Section 5 requires ech_required unconditionally when the extension is unaccepted.
- (E2) Section 6.1.6 specifies different handling for handshake failures.
- (E3) Only a successful handshake path mandates sending ech_required.

**Fix Direction:**

Clarify or reconcile Section 6.1.6 and Section 5 by either extending the ech_required requirement to all failure cases or exempting failure scenarios to ensure consistent alert behavior.

**Severity:** Medium
  *Basis:* The inconsistent alert usage may affect error reporting and diagnostic clarity without causing direct security vulnerabilities.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic: Issue-2

---

## Report 5: draft-ietf-tls-esni-25-5-5

**Label:** Inconsistency Between Duplicate-Reference Rule and Appendix A for OuterExtensions

**Bug Type:** Inconsistency

**Explanation:**

The normative text in Section 5.1 requires aborting if any extension is referenced more than once in OuterExtensions, but Appendix A’s algorithm does not explicitly check for duplicates, potentially allowing multiple copies for malformed messages.

**Justification:**

- Section 5.1 mandates that duplicate references in OuterExtensions must cause an abort.
- Appendix A processes each extension by scanning for the next occurrence without tracking duplicates.
- This mismatch can lead to inconsistencies when processing a malformed ClientHelloOuter containing duplicate extensions.

**Evidence Snippets:**

- **E1:**

  Any extension is referenced in OuterExtensions more than once. (Section 5.1)

- **E2:**

  Appendix A’s linear-time algorithm processes each E in OuterExtensions by scanning forward through ClientHelloOuter’s extension list and copying the first matching instance, without tracking whether the same E value has appeared earlier in OuterExtensions. (Structural Analysis)

**Evidence Summary:**

- (E1) Normative text requires aborting on duplicate references in OuterExtensions.
- (E2) Appendix A’s algorithm does not enforce this check.

**Fix Direction:**

Revise Appendix A to include an explicit step that checks for duplicate extension references in OuterExtensions and aborts the connection if any are detected.

**Severity:** Medium
  *Basis:* While conforming messages are unaffected, the discrepancy may result in subtle implementation differences for malformed inputs.

**Confidence:** High

**Experts mentioning this issue:**

- Structural: Issue-1

---

## Report 6: draft-ietf-tls-esni-25-5-6

**Label:** Mismatch of ECHConfig Field References Between Structure and Behavior Text

**Bug Type:** Inconsistency

**Explanation:**

Section 6.1 incorrectly refers to ECHConfig fields (e.g. contents.kem_id, contents.cipher_suites, contents.version) that do not match the defined structures in Section 4, where the correct fields are nested differently.

**Justification:**

- Section 4 defines ECHConfig with a top-level version and embeds key configuration inside contents.key_config.
- Section 6.1’s references to ECHConfig.contents.kem_id and ECHConfig.contents.cipher_suites ignore the key_config nesting, and it misplaces the version field.
- This misnaming can mislead implementers and automated tools about the proper structure.

**Evidence Snippets:**

- **E1:**

  struct { uint16 version; uint16 length; select (ECHConfig.version) { case 0xfe0d: ECHConfigContents contents; } } ECHConfig; (Section 4)

- **E2:**

  To determine if a given ECHConfig is suitable, it checks that it supports the KEM algorithm identified by ECHConfig.contents.kem_id, at least one KDF/AEAD algorithm identified by ECHConfig.contents.cipher_suites, and the version of ECH indicated by ECHConfig.contents.version. (Section 6.1)

**Evidence Summary:**

- (E1) The structural definition in Section 4 shows the proper nesting of fields.
- (E2) Section 6.1 incorrectly refers to fields without the key_config nesting and misplaces the version field.

**Fix Direction:**

Update Section 6.1 to refer to the correct field paths, e.g., use ECHConfig.contents.key_config.kem_id, ECHConfig.contents.key_config.cipher_suites, and ECHConfig.version.

**Severity:** Medium
  *Basis:* Although the intended mapping can be inferred, the inconsistency may confuse implementers and automated processing.

**Confidence:** High

**Experts mentioning this issue:**

- Structural: Issue-2
- Terminology: Issue-2

---

## Report 7: draft-ietf-tls-esni-25-5-7

**Label:** Inconsistent Terminology: 'retry_config' vs 'retry_configs'

**Bug Type:** Inconsistency

**Explanation:**

The document inconsistently uses the singular 'retry_config' and the plural 'retry_configs' when referring to the server’s retry configurations, which may lead to confusion.

**Justification:**

- Section 5 defines the field as 'retry_configs' and describes it as containing one or more configurations.
- Later text inconsistently uses the singular 'retry_config' in quotes, creating a naming disparity.
- Consistent terminology is important to avoid misinterpretation.

**Evidence Snippets:**

- **E1:**

  struct { ECHConfigList retry_configs; } ECHEncryptedExtensions; (Section 5)

- **E2:**

  Clients SHOULD NOT accept ‘retry_config’ in response to a connection initiated in response to a ‘retry_config’. (Section 6.1.8)

**Evidence Summary:**

- (E1) The defined field name is 'retry_configs'.
- (E2) Later text uses the singular 'retry_config', leading to inconsistency.

**Fix Direction:**

Standardize all references to use 'retry_configs' consistently throughout the document.

**Severity:** Low
  *Basis:* This issue is primarily editorial and unlikely to cause implementation errors.

**Confidence:** High

**Experts mentioning this issue:**

- Terminology: Issue-3

---
