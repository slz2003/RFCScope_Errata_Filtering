# Errata Reports

Total reports: 1

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