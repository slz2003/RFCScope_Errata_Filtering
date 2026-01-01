# Errata Reports

Total reports: 5

---

## Report 1: 9904-2-1

**Label:** Conflicting registration policy for Digest Algorithms with mixed 'MAY' and non-'MAY' values

**Bug Type:** Both

**Explanation:**

The Digest Algorithms registry contains overlapping rules that lead to conflicting requirements when an entry has a mix of 'MAY' and non-'MAY' values across its recommendation columns.

**Justification:**

- The text specifies that an entry with a 'MAY' value in any column must follow Specification Required, while any entry with any non-'MAY' value requires Standards Action, leading to a conflict when both conditions are met.
- Multiple experts (Scope, Deontic, Structural, and Boundary) identified this issue as causing process ambiguity for future registry updates.

**Evidence Snippets:**

- **E1:**

  Adding a new entry to the ‘Digest Algorithms’ registry with a recommended value of ‘MAY’ in the ‘Use for DNSSEC Delegation’, ‘Use for DNSSEC Validation’, ‘Implement for DNSSEC Delegation’, or ‘Implement for DNSSEC Validation’ columns SHALL follow the Specification Required policy as defined in [RFC8126].

- **E2:**

  Adding a new entry to, or changing an existing value in, the ‘Digest Algorithms’ registry that has any value other than ‘MAY’ in the ‘Use for DNSSEC Delegation’, ‘Use for DNSSEC Validation’, ‘Implement for DNSSEC Delegation’, or ‘Implement for DNSSEC Validation’ columns requires Standards Action.

**Evidence Summary:**

- (E1) Digest Algorithms rule that mandates Specification Required when a 'MAY' value is present.
- (E2) Digest Algorithms rule that mandates Standards Action when any non-'MAY' value is present, leading to a conflict for mixed entries.

**Fix Direction:**

Align the Digest Algorithms registration note with the DNS Security Algorithm Numbers note by clarifying that Specification Required applies only when all columns are 'MAY', with any deviation triggering Standards Action.

**Severity:** Medium
  *Basis:* The ambiguity may delay or complicate registry updates due to inconsistent interpretations of IANA registration policies.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- Structural Expert: Issue-2
- Boundary Expert: Finding-1

---

## Report 2: 9904-2-2

**Label:** Ambiguity in allowed-values constraints for Digest Delegation columns

**Bug Type:** Underspecification

**Explanation:**

The specification defines allowed recommendation values explicitly for DNSKEY registry columns but does not clearly state whether the same constraints apply to the Digest Algorithms registry's Delegation columns.

**Justification:**

- The normative text restricts values for ‘Use for DNSSEC Signing’ and ‘Use for DNSSEC Validation’ (and their Implement counterparts) without mentioning the delegation columns, leaving their allowed values ambiguous.
- This ambiguity could result in inconsistent application of recommendation keywords in the Digest Algorithms registry.

**Evidence Snippets:**

- **E1:**

  Only values of ‘MAY’, ‘RECOMMENDED’, ‘MUST NOT’, and ‘NOT RECOMMENDED’ may be placed into the ‘Use for DNSSEC Signing’ and ‘Use for DNSSEC Validation’ columns. Only values of ‘MAY’, ‘RECOMMENDED’, ‘MUST’, ‘MUST NOT’, and ‘NOT RECOMMENDED’ may be placed into the ‘Implement for DNSSEC Signing’ and ‘Implement for DNSSEC Validation’ columns. Note that a value of ‘MUST’ is not an allowed value for the two ‘Use for’ columns.

- **E2:**

  Table 1 earlier introduces, for the Digest Algorithms registry, “Use for DNSSEC Delegation”, “Use for DNSSEC Validation”, “Implement for DNSSEC Delegation”, and “Implement for DNSSEC Validation” columns.

**Evidence Summary:**

- (E1) Allowed-values constraints are explicitly stated for DNSKEY registry columns.
- (E2) Introduction of Delegation columns in the Digest Algorithms registry is not clearly covered by these constraints.

**Fix Direction:**

Clarify in the specification whether the allowed-values constraints apply uniformly to all ‘Use for’ and ‘Implement for’ columns across both registries or provide distinct rules for the Delegation columns.

**Severity:** Medium
  *Basis:* The lack of clear constraints may lead to inconsistent interpretations by implementers and IANA reviewers.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2

---

## Report 3: 9904-2-3

**Label:** Incorrect provenance claim for 'Implement for' columns extending beyond RFC 8624

**Bug Type:** Inconsistency

**Explanation:**

The document claims that the 'Implement for' column values are transcribed from RFC 8624, yet it includes entries for algorithms and digests that do not appear in RFC 8624, resulting in a provenance mismatch.

**Justification:**

- Section 2.2 states that all ‘Implement for’ values are transcribed from RFC 8624.
- However, Tables 2 and 3 include additional entries (e.g., DNSKEY algorithms 17, 23, 253, 254 and digest algorithms 5 and 6) that are absent from RFC 8624, misleading implementers about the normative source.

**Evidence Snippets:**

- **E1:**

  The following sections state the initial values that have been populated into these columns. The values in the ‘Implement for’ columns are transcribed from [RFC8624].

- **E2:**

  Table 2 in RFC 9904 includes algorithms 17 (SM2SM3), 23 (ECC‑GOST12), 253 (PRIVATEDNS), and 254 (PRIVATEOID); Table 3 in RFC 9904 includes digest algorithms 5 (GOST R 34.11‑2012) and 6 (SM3), none of which are present in RFC 8624.

**Evidence Summary:**

- (E1) Provenance claim that ‘Implement for’ values are transcribed from RFC 8624.
- (E2) Tables include additional algorithm and digest entries not defined in RFC 8624.

**Fix Direction:**

Revise the provenance statement to limit the claim to only those entries defined in RFC 8624 and clearly attribute the source for newly introduced entries.

**Severity:** Low
  *Basis:* This editorial inaccuracy may confuse readers about the normative source of certain registry values, even though it does not affect implementation.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-1
- Terminology Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 4: 9904-2-4

**Label:** Unspecified IANA policy for modifications to all‑MAY entries

**Bug Type:** Underspecification

**Explanation:**

The RFC does not clearly delineate the IANA process for modifying existing registry entries that remain entirely 'MAY', creating a gap in the registration policy.

**Justification:**

- While new entries with all 'MAY' values are covered by the Specification Required policy, modifications that leave all columns as 'MAY' do not trigger either the new entry or the non‑MAY change rules.
- This lack of clarity may lead to differing interpretations regarding whether such modifications require Specification Required or another process.

**Evidence Snippets:**

- **E1:**

  Change to an existing entry where, after the change, all four columns remain “MAY”: It is not “adding a new entry” → the first bullet (Specification Required) doesn’t obviously apply. The resulting entry does not “have any value other than ‘MAY’” → the second bullet (Standards Action) does not apply either.

**Evidence Summary:**

- (E1) Description of the process gap for modifications to entries that retain all 'MAY' values.

**Fix Direction:**

Explicitly state that any modification to an existing entry that does not alter the all‑MAY status should follow the Specification Required process.

**Severity:** Medium
  *Basis:* Although it does not impact on‑the‑wire behaviour, the ambiguity could lead to inconsistent IANA practices and delay registry updates.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Registration-policy gap

---

## Report 5: 9904-2-5

**Label:** Editorial inconsistency in derivation of 'Use for' columns from 'Implement for' columns

**Bug Type:** Editorial

**Explanation:**

The document inconsistently states that the 'Use for' columns are set to the same values as the 'Implement for' columns, while also specifying that a 'MUST' in the 'Implement for' column is downgraded to 'RECOMMENDED' in the corresponding 'Use for' column.

**Justification:**

- The specification claims identical values for both columns yet later clarifies that a 'MUST' is not permitted in 'Use for' columns and is replaced with 'RECOMMENDED'.
- This internal contradiction is noted by experts as an editorial oversight that may confuse readers about the intended derivation of usage recommendations.

**Evidence Snippets:**

- **E1:**

  The 'Use for' columns are set to the same values as those in the 'Implement for' columns …

- **E2:**

  Note that the value in the 'Use for' column is 'RECOMMENDED' when the value in the corresponding 'Implement for' column is 'MUST'.

**Evidence Summary:**

- (E1) Statement claiming that 'Use for' values mirror 'Implement for' values.
- (E2) Clarification that 'MUST' in 'Implement for' is downgraded to 'RECOMMENDED' in 'Use for', creating an editorial inconsistency.

**Fix Direction:**

Revise the wording to accurately describe that 'Use for' values are derived from, rather than identical to, 'Implement for' values when a 'MUST' condition is present.

**Severity:** Low
  *Basis:* This is a minor editorial inconsistency that does not affect operational behavior but may cause slight confusion.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Provenance and 'same values' text
- Terminology Expert: Issue-1

---
