# Errata Reports

Total reports: 3

---

## Report 1: draft-ietf-tls-rfc8446bis-14-5-1

**Label:** Ambiguous AEAD Key-Usage Limits Scope in Section 5.5

**Bug Type:** Both

**Explanation:**

The specification ambiguously describes key usage limits as applying both to an entire connection and to a given set of keys, which may cause implementers to enforce limits inconsistently.

**Justification:**

- The text uses conflicting language, stating limits apply 'under a given set of keys' (per key) and also 'on a given connection' (aggregated over multiple keys) (see E1 and E3).
- It instructs implementations to perform a KeyUpdate or close the connection before the limits are reached, but does not clarify if the counter resets with each new key (see E2).

**Evidence Snippets:**

- **E1:**

  There are cryptographic limits on the amount of plaintext which can be safely encrypted under a given set of keys. [AEAD-LIMITS] provides an analysis of these limits… (Section 5.5)

- **E2:**

  Implementations MUST either close the connection or do a key update as described in Section 4.6.3 prior to reaching these limits. Note that it is not possible to perform a KeyUpdate for early data and therefore implementations MUST NOT exceed the limits when sending early data. (Section 5.5)

- **E3:**

  For AES-GCM, up to 2^24.5 full-size records … may be encrypted on a given connection while keeping a safety margin of approximately 2^-57 for Authenticated Encryption (AE) security. (Section 5.5)

**Evidence Summary:**

- (E1) indicates limits are described per a given set of keys.
- (E2) mandates a key update or connection close before limits are reached.
- (E3) phrases the limit as applying on a given connection.

**Fix Direction:**

Clarify that the AEAD usage limits are applied per write traffic key (or per direction) rather than cumulatively over the lifetime of a connection.

**Severity:** Medium
  *Basis:* The ambiguity may lead to overly conservative enforcement or varied interpretations among implementers.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---

## Report 2: draft-ietf-tls-rfc8446bis-14-5-2

**Label:** Conflicting RFC Numbers for record_size_limit Extension (RFC8849 vs RFC8449)

**Bug Type:** Inconsistency

**Explanation:**

The document inconsistently cites the record_size_limit extension with two different RFC numbers, leading to a normative reference conflict.

**Justification:**

- The extensions table in Section 4.2 lists record_size_limit as coming from [RFC8849] (see E1).
- Later, Section 5.4 refers to the record_size_limit extension from [RFC8449], and the referenced header confirms the correct document is RFC8449 (see E2 and E3).

**Evidence Snippets:**

- **E1:**

  Extensions table row in Section 4.2:

        | record_size_limit [RFC8849]                      |      CH, EE |

- **E2:**

  Record padding text in Section 5.4:

        “If the maximum fragment length is reduced -- as for example by the record_size_limit extension from [RFC8449] -- then the reduced limit applies to the full plaintext, including the content type and padding.”

- **E3:**

  From the referenced document header:

        “Record Size Limit Extension for TLS … RFC 8449 …”

**Evidence Summary:**

- (E1) shows the extension is referenced as coming from [RFC8849] in the extensions table.
- (E2) indicates the extension should be referenced as from [RFC8449] in the record padding section.
- (E3) confirms via the document header that RFC8449 is the authoritative reference.

**Fix Direction:**

Update the extensions table in Section 4.2 to reference [RFC8449] consistently.

**Severity:** Medium
  *Basis:* Incorrect normative references can misguide implementers and lead to inconsistency in citing the proper specification.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1
- Terminology Expert: Issue-1

---

## Report 3: draft-ietf-tls-rfc8446bis-14-5-3

**Label:** Inconsistent ContentType Enum Definition Regarding heartbeat(24)

**Bug Type:** Inconsistency

**Explanation:**

There is an inconsistency in the definition of valid ContentType values: Section 5.1 omits the heartbeat(24) type while Appendix B.1 and the extensions table include it.

**Justification:**

- Section 5.1 defines ContentType with only four values (handshake, application_data, alert, change_cipher_spec) and omits heartbeat(24) (see E1).
- Appendix B.1 includes heartbeat(24) as defined in RFC6520, and the extensions table also references heartbeat (see E2 and E3).
- This discrepancy creates ambiguity about whether heartbeat should be recognized in TLS 1.3 records.

**Evidence Snippets:**

- **E1:**

  Snippet (Section 5.1): The ContentType definition lists:
        invalid(0), change_cipher_spec(20), alert(21), handshake(22), application_data(23), (255)
        and the prose: “This document specifies four content types: handshake, application_data, alert, and change_cipher_spec. The change_cipher_spec record is used only for compatibility purposes…”

- **E2:**

  Snippet (Appendix B.1): The ContentType definition there is:
        invalid(0), change_cipher_spec(20), alert(21), handshake(22), application_data(23), heartbeat(24),  /* RFC 6520 */ (255)

- **E3:**

  Snippet (Section 4.2, Table 1): The extensions table includes:
        heartbeat [RFC6520] with permitted messages CH, EE.

**Evidence Summary:**

- (E1) shows Section 5.1 omits the heartbeat content type.
- (E2) shows Appendix B.1 includes heartbeat(24) as per RFC6520.
- (E3) confirms that heartbeat is referenced in the extensions table.

**Fix Direction:**

Decide whether heartbeat should be supported in TLS 1.3 and update Section 5.1 and Appendix B.1 to consistently reflect the intended set of ContentType values.

**Severity:** Medium
  *Basis:* The inconsistency could lead to divergent implementations and potential interoperability issues regarding record parsing.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-2

---
