# Errata Reports

Total reports: 4

---

## Report 1: 9748-4-1

**Label:** Ambiguous Dual Assignment for Field Type 0x0204 (Autokey vs NTS Cookie)

**Bug Type:** Both (Underspecification and Inconsistency)

**Explanation:**

The specification assigns the same Field Type value (0x0204) to both Autokey Message Request and NTS Cookie without providing explicit, normative rules for disambiguating the two meanings.

**Justification:**

- RFC 9748’s note and Table 1 list two entries for 0x0204, one for Autokey Message Request (RFC5906) and one for NTS Cookie (RFC8915), but state that 'other parts of the message' determine the proper interpretation.
- Several expert analyses (Scope, Deontic, Structural, CrossRFC, and Boundary) note that relying on context rather than a clear, normative condition creates ambiguity for implementations that support both Autokey and NTS.

**Evidence Snippets:**

- **E1:**

  Both NTS Cookie and Autokey Message Request have the same Field Type; in practice this is not a problem as the field semantics will be determined by other parts of the message.

- **E2:**

  Table 1 has two rows with Field Type 0x0204: one “Autokey Message Request” [RFC5906] and one “NTS Cookie” [RFC8915, Section 5.4].

**Evidence Summary:**

- (E1) RFC 9748 explicitly notes the dual use of 0x0204 with disambiguation left to 'other parts of the message'.
- (E2) The registry table shows two different entries for 0x0204 under different RFCs.

**Fix Direction:**

Add explicit normative disambiguation rules (for example, tie the interpretation of 0x0204 to the presence of NTS-specific elements such as the 0x0404 extension field or specific NTP modes) to clearly determine when 0x0204 is to be read as NTS Cookie versus Autokey Message Request.

**Severity:** Medium
  *Basis:* The ambiguity may lead to inconsistent interpretation in implementations that support both security schemes, risking interoperability issues despite not breaking on‐wire operation.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-1
- Deontic: Issue-1
- Structural: Issue-1
- CrossRFC: Issue-1
- Boundary: Finding-1

---

## Report 2: 9748-4-2

**Label:** Conflicting Registration-Policy Scope for Private/Experimental Use Range

**Bug Type:** Both (Inconsistency and Underspecification)

**Explanation:**

Section 3’s blanket statement that all registry entries require Specification Required conflicts with Section 4.3’s reservation of the 0xF000–0xFFFF range for Private/Experimental Use, creating an ambiguous registration-policy scope.

**Justification:**

- Section 3 states that the policy for each registry is 'Specification Required', while Section 4.3 and Table 1 explicitly reserve the 0xF000–0xFFFF range for private or experimental use where IANA assigns no values.
- This discrepancy may mislead future authors or IANA reviewers into assuming uniform policy across the entire registry.

**Evidence Snippets:**

- **E1:**

  Section 3: “The following general guidelines apply to the NTP registries: … The policy for each registry is now Specification Required, as defined in [RFC8126], Section 4.6.”

- **E2:**

  Section 4.3 Note: “Field Types in the range 0xF000 through 0xFFFF, inclusive, are reserved for experimentation and development. IANA cannot assign them.”

**Evidence Summary:**

- (E1) Section 3 mandates a uniform Specification Required policy for all registry entries.
- (E2) Section 4.3 explicitly reserves the 0xF000–0xFFFF range, contradicting the uniform policy.

**Fix Direction:**

Revise Section 3 to clarify that the Specification Required policy applies only to assignable portions of the registry, or explicitly partition the registry to reflect the different policies.

**Severity:** Low
  *Basis:* While the inconsistency may lead to confusion for future registry management and review, it does not impact on-wire operation.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2
- CrossRFC: Issue-2

---

## Report 3: 9748-4-3

**Label:** Inconsistency in 'Replace Existing Entry' Directive in the NTP Extension Field Types Registry

**Bug Type:** Inconsistency

**Explanation:**

The preamble in Section 4 states that each new entry should replace an existing one with the same name; however, the NTP Extension Field Types registry adds a new NTS Cookie entry (0x0204) alongside the existing Autokey Message Request entry, violating the replacement claim.

**Justification:**

- RFC 9748 Section 4 declares that 'Each entry described in the subsections below is intended to completely replace the existing entry with the same name,' yet in Section 4.3 a second entry for Field Type 0x0204 is added without replacing any existing entry.
- This mismatch between the stated intent and the actual registry changes may mislead IANA reviewers and implementers.

**Evidence Snippets:**

- **E1:**

  Section 4: “Each entry described in the subsections below is intended to completely replace the existing entry with the same name.”

- **E2:**

  Section 4.3: Table 1 adds a second entry for Field Type 0x0204 (“NTS Cookie”) alongside the 'Autokey Message Request' entry.

**Evidence Summary:**

- (E1) The RFC preamble indicates that new entries should replace existing ones by name.
- (E2) The registry table shows an additional NTS Cookie entry for 0x0204, contradicting the replacement directive.

**Fix Direction:**

Clarify the language in Section 4 to accurately indicate when entries are replaced versus when new entries are added, and explicitly address the dual assignment for 0x0204.

**Severity:** Low
  *Basis:* This is an editorial and descriptive inconsistency that may cause confusion during IANA actions but does not affect protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC: Issue-2

---

## Report 4: 9748-4-4

**Label:** Editorial Inconsistency in Field Type Column Definition for Private/Experimental Use Range

**Bug Type:** Editorial Inconsistency

**Explanation:**

The 'Field Type (required)' column is defined to hold a single two-byte hexadecimal value, yet a range (0xF000–0xFFFF) is used in the registry to denote reserved values, creating an editorial inconsistency.

**Justification:**

- RFC 9748 defines the Field Type column as a two-byte hexadecimal value, implying individual values should be listed.
- The final row of Table 1 uses a range to denote that all values from 0xF000 to 0xFFFF are reserved for Private or Experimental Use, which deviates from the column’s defined format.

**Evidence Snippets:**

- **E1:**

  RFC 9748 defines the “Field Type (required)” column as “a two-byte value in hexadecimal.”

- **E2:**

  The last row of Table 1 is “0xF000-0xFFFF | Reserved for Private or Experimental Use | RFC 9748” and is accompanied by the note stating that these values are reserved for experimentation and development.

**Evidence Summary:**

- (E1) The column definition specifies individual two-byte hexadecimal values.
- (E2) The use of a range in the registry table contradicts the individual value format.

**Fix Direction:**

Either update the column definition to indicate that ranges are permitted as shorthand for reserving blocks, or revise the reserved entry to list individual values.

**Severity:** Low
  *Basis:* This is an editorial issue with no impact on protocol operations but may affect clarity in registry documentation.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary: Finding-2

---
