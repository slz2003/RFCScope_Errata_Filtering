# Errata Reports

Total reports: 3

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

## Report 2: 9748-3-2

**Label:** Ambiguous Temporal Scope for ASCII ID Restrictions in NTP Registries

**Bug Type:** Underspecification

**Explanation:**

The requirement for IDs to contain only ASCII uppercase letters or digits is unclear regarding its applicability to existing entries versus new assignments.

**Justification:**

- Section 3 imposes the restriction as a blanket rule, while Sections 4.1 and 4.2 specify that existing entries are left unchanged.
- The document does not explicitly state that the restriction is intended solely for future registrations.

**Evidence Snippets:**

- **E1:**

  Section 3 (second bullet): “In the ‘NTP Reference Identifier Codes’ and ‘NTP Kiss-o'-Death Codes’ registries, entries with ASCII fields are now limited to uppercase letters or digits.”

- **E2:**

  Section 4.1 (Reference Identifier Codes): 
        - Column definition: “ID (required): a four-byte value padded on the right with all-bits-zero. Each byte other than padding must be ASCII uppercase letters or digits.” 
        - “The existing entries are left unchanged.”
Section 4.2 (Kiss-o'-Death Codes): similarly indicates that existing entries are left unchanged.

**Evidence Summary:**

- (E1) Section 3 enforces the ASCII restriction broadly without temporal qualification.
- (E2) Sections 4.1 and 4.2 indicate that existing entries remain unchanged, implying a prospective application of the rule.

**Fix Direction:**

Add explicit language to state that the ASCII restriction applies only to new registrations, with legacy entries remaining unaffected.

**Severity:** Medium
  *Basis:* The lack of clarity may result in inconsistent interpretation and potential validation issues between legacy and future assignments.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2

---

## Report 3: 9748-3-3

**Label:** Ambiguous Reference of 'the NTP registries' in Section 3

**Bug Type:** Ambiguity

**Explanation:**

The term 'the NTP registries' is used without explicit definition, which may lead some readers to mistakenly include additional registries such as the NTS registries.

**Justification:**

- The residual uncertainties note that 'the NTP registries' appears to refer only to the specific registries being updated, but this is not explicitly stated.
- Section 2.3 clarifies that no changes apply to NTS registries, yet the global phrasing may still cause confusion.

**Evidence Snippets:**

- **E1:**

  ResidualUncertainties:
  - The term “the NTP registries” in Section 3 appears, from context, to mean the specific NTP registries that this document is updating (Reference IDs, Kiss-o’-Death Codes, Extension Field Types), not all registries under the ntp‑parameters umbrella and not the NTS registries; Section 2.3 explicitly says there are no changes for NTS.

**Evidence Summary:**

- (E1) The residual uncertainties highlight that the phrase 'the NTP registries' might be misinterpreted to include registries other than those intended, such as the NTS registries.

**Fix Direction:**

Provide an explicit list of the NTP registries subject to the update to remove any ambiguity.

**Severity:** Low
  *Basis:* The ambiguity is mainly editorial and is unlikely to result in significant operational issues.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope Expert: ResidualUncertainties

---
