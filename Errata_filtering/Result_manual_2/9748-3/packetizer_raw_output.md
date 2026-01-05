# Errata Reports

Total reports: 1

---


## Report 1: 9748-3-1

**Label:** Unclear interaction between 'Specification Required' and Reserved Private/Experimental Ranges in NTP Registries

**Bug Type:** Underspecification

**Explanation:**

The document states a global 'Specification Required' policy while reserving certain subranges for private/experimental use where IANA cannot assign codes, causing ambiguity about which values are governed by expert review.

**Justification:**

- Section 3 presents a registry‑wide policy alongside reserved ranges, leading to potential misinterpretation on whether the policy applies to all values.
- There is no explicit clarification that 'Specification Required' applies only to the IANA‑assignable portions of the registries.

**Evidence Snippets:**

- **E1:**

  Section 3 (NTP Registry Updates):  
        - “A partition of the ‘NTP Extension Field Types’ registry is reserved for Private or Experimental Use.”  
        - “In the ‘NTP Reference Identifier Codes’ and ‘NTP Kiss-o'-Death Codes’ registries, entries with ASCII fields are now limited to uppercase letters or digits. Fields starting with 0x58, the uppercase letter ‘X’, are reserved for Private or Experimental Use.”  
        - “The policy for each registry is now Specification Required, as defined in [RFC8126], Section 4.6.”

- **E2:**

  Section 4.1 / 4.2 notes: “Codes beginning with the character ‘X’ are reserved for experimentation and development. IANA cannot assign them.”

- **E3:**

  Section 4.3 note: “Field Types in the range 0xF000 through 0xFFFF, inclusive, are reserved for experimentation and development. IANA cannot assign them.”

**Evidence Summary:**

- (E1) Section 3 shows that the overall policy is Specification Required even as specific ranges are reserved for private/experimental use.
- (E2) Sections 4.1/4.2 clarify that codes beginning with ‘X’ are reserved and not assignable.
- (E3) Section 4.3 reserves Field Types 0xF000–0xFFFF for experimentation.

**Fix Direction:**

Clarify that the 'Specification Required' policy applies only to IANA-assignable ranges while the reserved private/experimental subranges remain outside its scope.

**Severity:** Medium
  *Basis:* The ambiguous interplay of policies could lead to misinterpretations in the assignment and review process.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---