# Errata Reports

Total reports: 4

---


## Report 3: 9824-3-3

**Label:** Incorrect 'Immediate Lexicographic Successor' Calculation for Next Domain Name

**Bug Type:** Inconsistency/Underspecification

**Explanation:**

The procedure of adding a leading null label to generate the Next Domain Name is claimed to yield the immediate lexicographic successor, but it instead produces a lexicographically smaller name and omits guidance to ensure that no existing names are covered.

**Justification:**

- Section 3.1 claims that the Next Domain Name is obtained by adding a leading label with a single null octet, implying an immediate lexicographic successor, but canonical DNS ordering shows that the created name is a predecessor.
- RFC 4470 requires that epsilon functions must not cover any existing names, yet the construction is presented without an accompanying coverage check.

**Evidence Snippets:**

- **E1:**

  Section 3.1: “The Next Domain Name field SHOULD be set to the immediate lexicographic successor of the QNAME. This is accomplished by adding a leading label with a single null (zero-value) octet. … For example, a request for the nonexistent name 'a.example.com.' would result in … a.example.com. 300 IN NSEC \000.a.example.com. RRSIG NSEC NXNAME” (Quantitative Expert Issue-1)

**Evidence Summary:**

- (E1) illustrates the construction that produces \000.a.example.com., which is lexicographically less than a.example.com., contradicting its intended immediate successor role.

**Fix Direction:**

Revise the description to remove or qualify the 'immediate lexicographic successor' claim and include explicit instructions to check that the generated [owner, Next) interval does not cover any existing names.


**Severity:** Medium
  *Basis:* While the degenerate interval does not break DNSSEC validation, misleading instructions may result in non-minimally covering NSEC records.

**Confidence:** High

---


## Report 4: 9824-3-4

**Label:** Unsigned-Referral Epsilon Function Lacks 'No Coverage' Check

**Bug Type:** Underspecification

**Explanation:**

The epsilon function for constructing the Next Domain Name in unsigned referrals, which appends a zero octet to the first label, does not mandate a check to ensure that the resulting NSEC record does not inadvertently cover existing names.

**Justification:**

- Section 3.4 describes forming the Next Domain Name by appending a zero octet to the first label, but omits any requirement to check against existing owner names.
- RFC 4470’s minimally covering NSEC requirement mandates that the generated interval must not cover any existing names.

**Evidence Snippets:**

- **E1:**

  With Compact Denial of Existence, the Next Domain Name field for this NSEC record is computed with a slightly different epsilon function than the immediate lexicographic successor of the owner name… Instead, the Next Domain Name field is formed by appending a zero octet to the first label of the owner name. For example, … sub.example.com. 300 IN NSEC sub\000.example.com. NS RRSIG NSEC (Structural Expert Issue-2)

**Evidence Summary:**

- (E1) demonstrates the unsigned-referral construction without a related check for existing names.

**Fix Direction:**

Amend Section 3.4 to explicitly require that authoritative servers verify no other owner name falls between the constructed Next Domain Name and the owner name.


**Severity:** Medium
  *Basis:* Omitting this check could cause an NSEC record to falsely assert nonexistence for valid names in the zone.

**Confidence:** Medium

---


## Report 5: 9824-3-5

**Label:** Lack of Guidance for Epsilon Functions at DNS Name/Label Length Limits

**Bug Type:** Underspecification

**Explanation:**

The specification does not address how to handle cases where adding a leading null label or appending a null octet may exceed DNS name or label length limits.

**Justification:**

- Sections 3.1 and 3.4 provide constructions for generating Next Domain Names without regard for the maximum allowed lengths imposed by DNS standards.
- RFC 4470 notes that its epsilon functions do not take into account constraints on the number of labels or total name length, leaving implementation behavior undefined for edge cases.

**Evidence Snippets:**

- **E1:**

  RFC 9824 Section 3.1: Next Domain Name for nonexistent names is ‘accomplished by adding a leading label with a single null (zero-value) octet’ to the QNAME. … Section 3.4: for unsigned referrals, the Next Domain Name is formed by ‘appending a zero octet to the first label of the owner name’, e.g., sub.example.com. → sub\000.example.com. (Boundary Expert Finding-2)

**Evidence Summary:**

- (E1) shows the constructions used without any discussion of handling cases where the addition might exceed DNS name or label length limits.

**Fix Direction:**

Provide explicit fallback behavior or additional conditions for selecting an alternative epsilon function when the standard construction would result in an invalid DNS name.


**Severity:** Medium
  *Basis:* This gap may lead to divergent behavior in extreme cases, affecting robustness and interoperability.

**Confidence:** High

---


## Report 7: 9824-3-7

**Label:** Mismatch in EDNS CO Flag Bit Position Description

**Bug Type:** Inconsistency

**Explanation:**

There is a discrepancy between the textual description claiming the CO flag occupies the second most significant bit and the IANA table assigning it as ‘Bit 1’, which is the second least significant bit.

**Justification:**

- Section 5.1 describes the CO flag as defined in the second most significant bit of the 16‑bit EDNS flags field.
- Section 9, Table 2 allocates the CO flag as ‘Bit 1’, leading to confusion about its actual numeric bit position.

**Evidence Snippets:**

- **E1:**

  Section 5.1: “A new EDNS0 [RFC6891] header flag is defined in the second most significant bit of the flags field in the EDNS0 OPT header. This flag is referred to as the Compact Answers OK (CO) flag.” … Section 9, Table 2: “| Bit 1 | CO   | Compact Answers OK | RFC 9824  |” (Quantitative Expert Issue-2)

**Evidence Summary:**

- (E1) evidences the conflict between the descriptive text and the IANA registry assignment for the CO flag.

**Fix Direction:**

Revise the documentation and IANA table to use consistent EDNS flag numbering (with the CO flag in the second most significant bit, e.g. Bit 14 in standard RFC6891 terms) or update both to a consistent alternate numbering.


**Severity:** Medium
  *Basis:* This inconsistency may lead to incorrect interpretation and handling of the CO flag by different implementations.

**Confidence:** High

---