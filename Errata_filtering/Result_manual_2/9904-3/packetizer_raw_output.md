# Errata Reports

Total reports: 1

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