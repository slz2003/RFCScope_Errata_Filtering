# Errata Reports

Total reports: 4

---

## Report 1: 9886-6-1

**Label:** Mismatch between HHIT Entity Type examples (10, 14, 15) and IANA HHIT Registry

**Bug Type:** Both

**Explanation:**

The document’s examples in Appendix A use HHIT Entity Type values 10, 14, and 15 with specific reserved semantics, yet the IANA HHIT Entity Types registry (Section 6.2.2.3) lists only values 0, 1, 5, 9, 13, and 16–27, creating a scope mismatch and underspecification.

**Justification:**

- Section 5.1.2 restricts the HHIT Entity Type field to values defined in Section 6.2.2, but the examples use additional values.
- The registry’s initial value table omits 10, 14, and 15, while the examples assign these values reserved DKI semantics.

**Evidence Snippets:**

- **E1:**

  Section 5.1.2 says: “HHIT Entity Type: The HHIT Entity Type field is a number with values defined in Section 6.2.2.” (explicitly scoping valid field values to that registry).

- **E2:**

  Section 6.2.2.3 (“Initial Values”) lists only the following HHIT Entity Types: 0, 1, 5, 9, 13, and 16–27, with specific semantics such as “9: Registered Assigning Authority (RAA)” and “13: HHIT Domain Authority (HDA)” (Table 2).

- **E3:**

  Appendix A examples decode HHIT RRType RDATA where the first element (the entity type) is: `10` with the comment “# Reserved (RAA Auth from DKI)” in Figure 10; `14` with the comment “# Reserved (HDA Auth from DKI)” in Figure 14; `15` with the comment “# Reserved (HDA Issue from DKI)” in Figure 16.

**Evidence Summary:**

- (E1) Section 5.1.2 limits HHIT Entity Type values to those defined in Section 6.2.2
- (E2) The registry initial values list does not include 10, 14, or 15
- (E3) Appendix A shows the use of values 10, 14, and 15 with reserved DKI semantics

**Fix Direction:**

Either update the IANA HHIT Entity Types registry to explicitly include or reserve values 10, 14, and 15 with their intended semantics, or change the examples to use only the defined registry values.

**Severity:** Medium
  *Basis:* This inconsistency creates ambiguity over the global interpretation of HHIT entity types, potentially leading to conflicting registrations.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---

## Report 2: 9886-6-2

**Label:** Omission of explicit change controller field in HHIT Entity Types registry

**Bug Type:** Underspecification

**Explanation:**

The HHIT Entity Types registry, managed under a First Come First Served policy, lacks an explicit 'change controller' field as recommended by RFC 8126, leaving the authority for future modifications ambiguous.

**Justification:**

- The registry definition lists only Value, HHIT Type, and Reference, with no dedicated change controller field.
- RFC 8126’s guidance for FCFS registries recommends including a change controller or equivalent contact information.

**Evidence Snippets:**

- **E1:**

  HHIT Entity Type: Numeric, field of the HHIT RRType to encode the HHIT Entity Type. All entries in this registry are under the First Come First Served policy (Section 4.4 of [RFC8126]).

- **E2:**

  HHIT Type Registry Fields  Value: HHIT Type value of the entry.  HHIT Type: Name of the entry and an optional abbreviation.  Reference: Public document allocating the value and any additional information such as semantics.

- **E3:**

  When creating a new registry with First Come First Served as the registration policy, in addition to the contact person field or reference, the registry should contain a field for change controller.

**Evidence Summary:**

- (E1) The registry is defined under a FCFS policy without a clear change controller field
- (E2) Only Value, HHIT Type, and Reference are provided
- (E3) RFC 8126 recommends including a change controller field for clarity in FCFS registries

**Fix Direction:**

Add an explicit change controller field to the HHIT Entity Types registry or clarify the process by designating an alternative, structured change authorization mechanism.

**Severity:** Low
  *Basis:* This omission affects administrative clarity for future updates rather than on-the-wire protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 3: 9886-6-3

**Label:** Inconsistency between universal nibble-split delegation and exclusion for Private Use RAAs

**Bug Type:** Inconsistency

**Explanation:**

The document asserts that each RAA is given 4 delegations via nibble-split but then states that RAAs within the Private Use range (15360–16383) are not delegated, resulting in a contradictory instruction.

**Justification:**

- Section 6.2.1.3 describes that every RAA gets 4 delegations by borrowing bits from the HDA space.
- Section 6.2.1.5 specifies that for RAAs in the Private Use range, IANA will not delegate any value, conflicting with the nibble-split rule.

**Evidence Snippets:**

- **E1:**

  To support DNS delegation in 3.0.0.1.0.0.2.ip6.arpa., a single RAA is given 4 delegations by borrowing the upper two bits of HDA space … These HDAs (0, 4096, 8192 and 12288) are reserved for the RAA.

- **E2:**

  …the RAAs (with its subordinate HDAs) in this range [15360–16383] may be used in like manner and IANA will not delegate any value in this range to any party (as per Private Use, Section 4.1 of [RFC8126]).

**Evidence Summary:**

- (E1) Nibble-split delegation assigns 4 delegations per RAA
- (E2) Private Use RAAs are explicitly stated to receive no delegation

**Fix Direction:**

Clarify in the text that the nibble-split delegation rule applies only to non–Private Use RAAs, or revise the delegation instructions to avoid the contradiction.

**Severity:** Medium
  *Basis:* The contradiction in delegation rules could lead to confusion over DNS delegation practices, despite not affecting on-wire encoding.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary Expert: Finding-1

---

## Report 4: 9886-6-4

**Label:** Unclear IANA procedure for optional /40 delegation for ISO 3166‑1 RAAs

**Bug Type:** Underspecification

**Explanation:**

The document mentions an optional /40 delegation for RAAs in the ISO 3166‑1 range without providing explicit instructions or a registration form, leaving the IANA procedure for this delegation ambiguous.

**Justification:**

- Section 6.2.1.4 states that a shorter prefix (2001:3x:xx00::/40) MAY be delegated for ISO 3166‑1 RAAs, covering all 4 RAAs and reserved HDAs.
- There is no clear guidance or specified procedure detailing how the /40 delegation should be processed, unlike the per-RAA NS field delegation explained elsewhere.

**Evidence Snippets:**

- **E1:**

  For RAAs under this range, a shorter prefix of 2001:3x:xx00::/40 MAY be delegated to each CAA, which covers all 4 RAAs (and reserved HDAs) assigned to them.

- **E2:**

  The NS RRType Content (HDA=X) fields are used by IANA to perform the DNS delegations under 3.0.0.1.0.0.2.ip6.arpa.. See Section 6.2.1.3 for technical details.

- **E3:**

  Future additions to this registry are to be made based on the following range and policy table…

**Evidence Summary:**

- (E1) An optional /40 delegation is mentioned for ISO 3166‑1 RAAs
- (E2) The NS RRType fields are described for standard delegation but not for a /40 case
- (E3) The policy table reference does not detail the procedure for /40 delegation

**Fix Direction:**

Include explicit procedural details and a registration form for the optional /40 delegation to ISO 3166‑1 RAAs to ensure clear and consistent implementation.

**Severity:** Medium
  *Basis:* The underspecified procedure could lead to inconsistent DNS delegation practices for ISO 3166‑1 RAAs, affecting administrative clarity.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary Expert: Finding-2

---
