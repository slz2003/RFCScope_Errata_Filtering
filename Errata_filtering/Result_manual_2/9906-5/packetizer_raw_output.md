# Errata Reports

Total reports: 1

---


## Report 1: 9906-5-1

**Label:** Misnamed IANA Registry Columns in RFC 9906 Section 5 Notes

**Bug Type:** Inconsistency

**Explanation:**

The explanatory note lines in RFC 9906 Section 5 incorrectly mix column names from the DNS Security Algorithm Numbers and Digest Algorithms registries, leading to references to non-existent columns in each registry.

**Justification:**

- In the DNS Security Algorithm Numbers entry (GOST R 34.10-2001, algorithm 12), the note erroneously refers to 'Implement for DNSSEC Delegation', which is not defined for that registry (the valid columns are 'Implement for DNSSEC Signing' and 'Implement for DNSSEC Validation'). (E1, E3)
- In the Digest Algorithms entry (GOST R 34.11‑94, value 3), the note incorrectly mentions 'Use for DNSSEC Signing', a column that does not exist there (the valid column is 'Use for DNSSEC Delegation'). (E2, E4)

**Evidence Snippets:**

- **E1:**

  RFC 9906 §5, for the GOST R 34.10-2001 (12) entry, says in the note: “Note: The "Use for DNSSEC Signing" and "Implement for DNSSEC Delegation" columns were already set to MUST NOT.”

- **E2:**

  RFC 9906 §5, for the GOST R 34.11-94 (3) entry, includes the note: “Note: The "Use for DNSSEC Signing" and "Implement for DNSSEC Delegation" columns were already set to MUST NOT.”

- **E3:**

  RFC 9904 defines for the “DNS Security Algorithm Numbers” registry the four added columns as: “Use for DNSSEC Signing”, “Use for DNSSEC Validation”, “Implement for DNSSEC Signing”, and “Implement for DNSSEC Validation” (Section 2.1 and reiterated in 7.1).

- **E4:**

  RFC 9904 defines for the “Digest Algorithms” registry the four added columns as: “Use for DNSSEC Delegation”, “Use for DNSSEC Validation”, “Implement for DNSSEC Delegation”, and “Implement for DNSSEC Validation” (Section 2.1 and reiterated in 7.2).

**Evidence Summary:**

- (E1) RFC 9906’s note for the DNS Security Algorithm Numbers entry incorrectly uses 'Implement for DNSSEC Delegation'.
- (E2) RFC 9906’s note for the Digest Algorithms entry incorrectly uses 'Use for DNSSEC Signing'.
- (E3) RFC 9904 specifies the correct column names for the DNS Security Algorithm Numbers registry.
- (E4) RFC 9904 specifies the correct column names for the Digest Algorithms registry.

**Fix Direction:**

Update the note lines in RFC 9906 Section 5 to reference only the correct column names as defined in RFC 9904: for the DNS Security Algorithm Numbers entry use 'Implement for DNSSEC Signing' (or the appropriate valid column) instead of 'Implement for DNSSEC Delegation', and for the Digest Algorithms entry use 'Use for DNSSEC Delegation' instead of 'Use for DNSSEC Signing'.

**Severity:** Medium
  *Basis:* The error is purely editorial and does not affect the actual registry updates or implementation behavior; however, the misnamed columns can mislead readers and future document authors regarding the normative definitions in RFC 9904.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Scope Expert: Issue-2
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1
- Terminology Expert: Issue-1

---