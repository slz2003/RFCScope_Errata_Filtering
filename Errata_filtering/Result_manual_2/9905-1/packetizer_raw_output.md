# Errata Reports

Total reports: 2

---


## Report 1: 9905-1-1

**Label:** Incorrect Attribution of DNSSEC Origin to RFC 3110 in Section 1

**Bug Type:** Inconsistency

**Explanation:**

RFC 9905 incorrectly states that DNSSEC was 'originally defined in RFC3110', which misrepresents the historical origin of DNSSEC and may mislead readers about where to find the primary specification.

**Justification:**

- Section 1 of RFC 9905 asserts that DNSSEC (as described in RFC9364) was originally defined in RFC3110, even though RFC3110 only specifies the usage of RSA/SHA1 and defers to RFC2535 for the foundational DNSSEC concepts.
- The RFC3110 Introduction clarifies that digital signatures and key distribution were first established in RFC2535, highlighting the error in attributing the original definition to RFC3110.

**Evidence Snippets:**

- **E1:**

  DNSSEC [RFC9364] (originally defined in [RFC3110]) made extensive use of SHA-1, for example, as a cryptographic hash algorithm in Resource Record Signature (RRSIG) and Delegation Signer (DS) records.

- **E2:**

  The DNS has been extended to include digital signatures and cryptographic keys as described in [RFC2535]. Thus the DNS can now be secured and used for secure key distribution.

**Evidence Summary:**

- (E1) shows the problematic attribution in Section 1.
- (E2) demonstrates that RFC3110 is not the original source for DNSSEC definitions.

**Fix Direction:**

In Section 1, remove or revise the parenthetical '(originally defined in [RFC3110])' so as not to misattribute the original DNSSEC specification.

**Severity:** Low
  *Basis:* The issue is primarily editorial and historical; while it may mislead readers, it does not directly impact implementation or interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC: Issue-1
- Terminology: Issue-1

---


## Report 2: 9905-1-2

**Label:** Misnaming DS Digest Registry as 'DNS Security Algorithm Numbers' in Section 1

**Bug Type:** Inconsistency

**Explanation:**

RFC 9905 incorrectly refers to the DS digest registry as 'DNS Security Algorithm Numbers' in Section 1, even though the DS digest registry is correctly named 'Digest Algorithms' elsewhere, leading to potential confusion over the proper registry for DS digest algorithms.

**Justification:**

- Section 1 instructs operators to consult two registries with identical names, causing ambiguity since the DS digest registry is, in fact, distinct and should be identified as 'Digest Algorithms'.
- Later in Section 5, the DS digest registry is correctly identified, indicating an inconsistency that may mislead implementers regarding which registry governs DS records.

**Evidence Snippets:**

- **E1:**

  Operators are encouraged to consider switching to one of the recommended algorithms listed in the ‘DNS Security Algorithm Numbers’ [DNSKEY-IANA ] and ‘DNS Security Algorithm Numbers’ [DS-IANA ] registries, respectively.

- **E2:**

  IANA has updated the SHA-1 (1) entry in the ‘Digest Algorithms’ registry [DS-IANA ] [RFC9904] as follows… and IANA has updated the RSASHA1 (5) and RSASHA1-NSEC3-SHA1 (7) algorithm entries in the ‘DNS Security Algorithm Numbers’ registry [DNSKEY-IANA ] [RFC9904] as follows…

- **E3:**

  DS‑IANA registry summary: it is the “DNSSEC Delegation Signer (DS) RR Type Digest Algorithms” / “Digest Algorithms” registry, distinct from the “DNS Security Algorithm Numbers” registry used for DNSKEY/RRSIG/DS Algorithm fields.

**Evidence Summary:**

- (E1) shows that Section 1 uses the same name for both registries.
- (E2) contrasts this by demonstrating the correct naming in Section 5.
- (E3) clarifies the actual title and purpose of the DS digest registry.

**Fix Direction:**

Update Section 1 by replacing the second instance of 'DNS Security Algorithm Numbers' with 'Digest Algorithms' (or the full title 'DNSSEC Delegation Signer (DS) RR Type Digest Algorithms') to accurately reflect the registry.

**Severity:** Medium
  *Basis:* The inconsistency may confuse operators and implementers about where DS digest algorithms are maintained, impacting clear interpretation of the specification.

**Confidence:** High

**Experts mentioning this issue:**

- Structural: Issue-1
- CrossRFC: Issue-2
- Terminology: Issue-2

---