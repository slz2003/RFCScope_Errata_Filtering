# Errata Reports

Total reports: 3

---

## Report 1: 9904-3-1

**Label:** Undefined Scope for DNSSEC Columns in Omitted and Non-DNSSEC Algorithm Numbers

**Bug Type:** Underspecification

**Explanation:**

The RFC does not clearly specify how the new DNSSEC columns apply to registry entries that are not DNSSEC zone-signing algorithms or that are omitted from Table 2.

**Justification:**

- Table 2 only covers algorithms 1, 3, 5, 6, 7, 8, 10, 12, 13, 14, 15, 16, 17, 23, 253, and 254, omitting other entries such as 0, 2, 4, 252, 255, leaving their DNSSEC column values undefined (E3).
- Section 2.2 restricts allowed values without providing an option for an empty or 'N/A' state and Section 7.1 does not exempt transaction-security-only or non‑DNSSEC entries (E4, E5).
- RFC 4034 indicates different usage domains, yet the document does not clarify which entries the DNSSEC-specific columns should apply to (E1, E2).

**Evidence Snippets:**

- **E1:**

  RFC 4034 notes that the “DNS Security Algorithm Numbers” registry contains algorithms with different usage domains: “Some algorithms are usable only for zone signing (DNSSEC), some only for transaction security mechanisms (SIG(0) and TSIG), and some for both. Those usable for zone signing may appear in DNSKEY, RRSIG, and DS RRs. Those usable for transaction security would be present in SIG(0) and KEY RRs…”

- **E2:**

  The same section lists values such as 2 (Diffie‑Hellman) and 252 (Indirect), whose “Zone Signing” field is “n”, indicating they are not for DNSSEC zone signing.

- **E3:**

  Section 3 then states: “Initial values for the use and implementation recommendation columns in the ‘DNS Security Algorithm Numbers’ registry under the ‘Domain Name System Security (DNSSEC) Algorithm Numbers’ registry group are shown in Table 2.” , but Table 2 only includes a subset of values (1, 3, 5, 6, 7, 8, 10, 12, 13, 14, 15, 16, 17, 23, 253, 254) and omits others known to exist in the registry (e.g., 0, 2, 4, 252, 255, and any newer entries).

- **E4:**

  Section 2.2 further constrains what *values* may appear in the new columns (“Only values of ‘MAY’, ‘RECOMMENDED’, ‘MUST NOT’, and ‘NOT RECOMMENDED’ may be placed into the ‘Use for DNSSEC Signing’ and ‘Use for DNSSEC Validation’ columns. Only values of ‘MAY’, ‘RECOMMENDED’, ‘MUST’, ‘MUST NOT’, and ‘NOT RECOMMENDED’ may be placed into the ‘Implement for DNSSEC Signing’ and ‘Implement for DNSSEC Validation’ columns.”) but does not allow for an “N/A” or empty state.

- **E5:**

  Section 7.1 says IANA “has updated the ‘DNS Security Algorithm Numbers’ registry … with the following columns and has populated these columns with the values from Table 2 of this document” , without any exception for transaction‑security‑only or non‑DNSSEC entries.

**Evidence Summary:**

- (E1) Indicates different usage domains without clarifying treatment of non‑DNSSEC entries.
- (E3) Table 2 is incomplete, omitting several registry entries.
- (E4) Allowed values do not include an option for empty or 'N/A'.
- (E5) No exception is made for transaction‑security‑only or non‑DNSSEC entries.

**Fix Direction:**

Clarify in the RFC whether non‑DNSSEC or omitted entries should have explicit default values (e.g., MUST NOT or N/A) or be excluded from the DNSSEC columns.

**Severity:** Medium
  *Basis:* The ambiguity in applying DNSSEC columns to the full registry could lead to inconsistent implementations or registry population.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---

## Report 2: 9904-3-2

**Label:** Over-broad Claim on Transcription of 'Implement for' Values from RFC 8624

**Bug Type:** Inconsistency

**Explanation:**

RFC 9904 claims that all 'Implement for' values are transcribed from RFC 8624, but several entries are newly defined rather than directly transcribed.

**Justification:**

- Section 2.2 of RFC 9904 states that the 'Implement for' values are transcribed from RFC 8624, yet RFC 8624 only provides explicit values for algorithms 1, 3, 5, 6, 7, 8, 10, 12, 13, 14, 15, and 16, omitting algorithms 17, 23, 253, and 254 (E1, E2).
- For DS digest algorithms, RFC 8624 covers digest types 0–4 and is silent on types 5 and 6, yet RFC 9904 assigns them the value 'MAY', indicating new assignments (E4).

**Evidence Snippets:**

- **E1:**

  The following sections state the initial values that have been populated into these columns.  The values in the ‘Implement for’ columns are transcribed from [RFC8624].  The ‘Use for’ columns are set to the same values as those in the ‘Implement for’ columns…

- **E2:**

  Section 3.1 of RFC 8624 lists implementation recommendations for DNSKEY algorithms only for numbers 1, 3, 5, 6, 7, 8, 10, 12, 13, 14, 15, and 16, with no rows for algorithms 17 (SM2SM3), 23 (ECC-GOST12), 253 (PRIVATEDNS), or 254 (PRIVATEOID).

- **E3:**

  Appendix A.1 of RFC 4034 lists 253 [PRIVATEDNS] and 254 [PRIVATEOID] with “Status” = OPTIONAL, but without any BCP 14 “MUST/SHOULD/MAY” implementation requirement levels.

- **E4:**

  Section 3.3 of RFC 8624 lists digest types 0–4 only; there are no rows for digest algorithms 5 or 6.

**Evidence Summary:**

- (E1) RFC 9904 asserts all 'Implement for' values are directly transcribed from RFC 8624.
- (E2) RFC 8624 does not provide entries for algorithms 17, 23, 253, or 254.
- (E4) RFC 8624 similarly lacks entries for digest types 5 and 6.

**Fix Direction:**

Revise the provenance statement to apply only where RFC 8624 provides explicit definitions and note that for entries not covered, RFC 9904 assigns 'MAY' based on generic guidelines.

**Severity:** Medium
  *Basis:* This inconsistency in provenance may mislead implementers or IANA reviewers regarding the origin of the recommendation levels.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 3: 9904-3-3

**Label:** ECDSAP384SHA384 Implementation Requirement Conflict Between RFC 6605 and RFC 8624/9904

**Bug Type:** Inconsistency

**Explanation:**

There is a normative conflict for algorithm 14 where RFC 6605 mandates MUST support while RFC 8624/9904 assign a lower, optional requirement level.

**Justification:**

- RFC 6605 Section 4 mandates that conformant implementations MUST implement signing and verification for algorithm 14 (ECDSAP384SHA384) (E1).
- RFC 8624’s DNSKEY algorithm table assigns algorithm 14 a value of 'MAY' for signing and 'RECOMMENDED' for validation, which RFC 9904 directly copies (E2).

**Evidence Snippets:**

- **E1:**

  RFC 6605 Section 4 states that “Conformant implementations that create records to be put into the DNS MUST implement signing and verification for both of the above algorithms… Conformant DNSSEC verifiers MUST implement verification for both of the above algorithms.”

- **E2:**

  In RFC 8624’s DNSKEY algorithm table, algorithm 14 (ECDSAP384SHA384) is set to ‘MAY’ for signing and ‘RECOMMENDED’ for validation, and RFC 9904 copies these values into the new columns.

**Evidence Summary:**

- (E1) RFC 6605 mandates a MUST requirement for algorithm 14.
- (E2) RFC 8624/9904 assign a lower requirement level (MAY/RECOMMENDED) for algorithm 14.

**Fix Direction:**

Clarify in RFC 9904 that the later BCP recommendations relax RFC 6605's requirements for algorithm 14 or specify how implementers should reconcile the conflicting normative statements.

**Severity:** Medium
  *Basis:* The conflicting requirement levels for a key DNSSEC algorithm may lead to non-conformant implementations.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-2

---
