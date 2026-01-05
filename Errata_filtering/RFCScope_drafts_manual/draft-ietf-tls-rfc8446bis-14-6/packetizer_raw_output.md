# Errata Reports

Total reports: 1

---


## Report 4: draft-ietf-tls-rfc8446bis-14-6-4

**Label:** Ambiguity in Handling Unknown and RESERVED Alert Codes Across TLS Versions

**Bug Type:** Editorial

**Explanation:**

The guidance on treating unknown alert types as fatal in TLS 1.3, together with references to RESERVED alert codes from earlier TLS versions, may confuse implementers regarding how to handle these alerts in mixed-version deployments.

**Justification:**

- Section 6 instructs that unknown alert types MUST be treated as error alerts in TLS 1.3, yet it does not explicitly delineate this behavior from TLS 1.2, where RFC 5246 applies.
- The inclusion of RESERVED alert codes in Appendix B.2 without clear scoping further adds to potential misinterpretation in multi-version contexts.

**Evidence Snippets:**

- **E4:**

  “All the alerts listed in Section 6.2 MUST be sent with AlertLevel=fatal and MUST be treated as error alerts when received regardless of the AlertLevel in the message. Unknown Alert types MUST be treated as error alerts.” (Section 6)
“Values listed as ‘_RESERVED’ were used in previous versions of TLS and are listed here for completeness. TLS 1.3 implementations MUST NOT send them but might receive them from older TLS implementations.” (Appendix B.2)

**Evidence Summary:**

- (E4) The evidence highlights that unknown alerts are mandated to be fatal in TLS 1.3 while referencing RESERVED codes from earlier versions, without explicit clarification on the differences in handling.

**Fix Direction:**

Add explicit clarification distinguishing the handling of unknown and RESERVED alert codes in TLS 1.3 from the rules that apply under RFC 5246 for TLS 1.2.

**Severity:** Low
  *Basis:* The potential confusion is mainly editorial and is unlikely to result in critical interoperability failures.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- Deontic Expert: Issue-2
- Boundary Expert: Finding-2

---