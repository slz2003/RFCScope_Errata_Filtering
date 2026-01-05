# Errata Reports

Total reports: 3

---


## Report 2: 9886-5-2

**Label:** BRID enumeration ambiguity: Allowed values for id_type/auth_type unclear

**Bug Type:** Inconsistency

**Explanation:**

The CDDL for BRID defines fixed enumerated values for fields such as id_type and a_type, yet the prose states that other values may be valid. This creates ambiguity regarding whether non‐enumerated values should be accepted.

**Justification:**

- The CDDL (Figure 5) defines uas-id-types as (none: 0, serial: 1, session_id: 4) and auth-types as (none: 0, specific_method: 5).
- The text indicates that while these enumerated values are relevant, other values may be valid but are considered outside the DRIP operation scope.

**Evidence Snippets:**

- **E1:**

  CDDL (Figure 5): uas-id-types = (none: 0, serial: 1, session_id: 4) and auth-types = (none: 0, specific_method: 5)

- **E2:**

  Text below Figure 5: “The explicitly enumerated values included in the CDDL above are relevant to DRIP for its operation. Other values may be valid but are outside the scope of DRIP operation.”

**Evidence Summary:**

- (E1) The CDDL limits id_type and a_type to specific enumerated values.
- (E2) The descriptive text allows for other valid values, leading to ambiguity.

**Fix Direction:**

Clarify whether non‐enumerated codepoints must be accepted; if so, adjust the CDDL to allow a numeric range rather than a fixed set.

**Severity:** Medium
  *Basis:* Ambiguous allowed values may result in divergent implementations and interoperability issues.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2

---


## Report 5: 9886-5-5

**Label:** BRID auth group mismatch: a_data length constraint conflicts with 'none' authentication type semantics

**Bug Type:** Inconsistency

**Explanation:**

The CDDL for the auth group in a BRID record requires the a_data field to have a length between 1 and 362 bytes, even when the authentication type is 'none' (0), which semantically should not include any data. This creates a conflict between the schema and the intended meaning of a 'none' type.

**Justification:**

- The CDDL (Figure 5) specifies a_data: bstr .size(1..362) for each auth-grp entry.
- ASTM and the associated semantics for a_type = 0 ('none') imply that there should be no authentication data present.

**Evidence Snippets:**

- **E1:**

  CDDL for auth-grp: a_data: bstr .size(1..362) (Figure 5).

- **E2:**

  Boundary Analysis notes: ‘a_type = 0 (none)’ implies that no authentication data should be present, conflicting with the CDDL requirement.

**Evidence Summary:**

- (E1) The CDDL mandates a minimum length of 1 for a_data.
- (E2) Semantic expectations for a 'none' authentication type require an empty a_data field.

**Fix Direction:**

Either allow a zero-length a_data for a_type 'none' or disallow a_type 'none' altogether in the context of BRID records.

**Severity:** Medium
  *Basis:* This inconsistency may lead to different implementations handling the 'none' case inconsistently, harming interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary: Finding-2

---


## Report 6: 9886-5-6

**Label:** BRID UAS ID ‘none’ underspecification: Mandatory 20-byte uas_id lacks guidance for id_type 0

**Bug Type:** Underspecification

**Explanation:**

While the CDDL requires every uas-id-grp in a BRID record to include a 20-byte uas_id, there is no normative guidance on what this value should be when the id_type is 'none' (0). This leaves implementations uncertain about how to handle this boundary case.

**Justification:**

- The CDDL (Figure 5) specifies uas_id: bstr .size(20) and includes 'none: 0' in uas-id-types.
- Boundary Analysis points out that ASTM semantics imply that when id_type is 'none', there is no meaningful uas_id; however, the record still mandates a 20-byte field without further instruction.

**Evidence Snippets:**

- **E1:**

  CDDL for uas-id-grp: uas_id: bstr .size(20) and uas-id-types = (none: 0, serial: 1, session_id: 4) (Figure 5).

- **E2:**

  Boundary Analysis: The document does not specify what the 20-byte uas_id should be when id_type is 0 (none), leaving implementations free to choose different conventions.

**Evidence Summary:**

- (E1) The CDDL requires a 20-byte uas_id regardless of id_type.
- (E2) There is no guidance for the case when id_type is 'none', leading to ambiguity.

**Fix Direction:**

Provide clear normative guidance for the uas_id value when id_type is 'none', such as a fixed constant or defined structure, or consider disallowing id_type 'none' in BRID records.

**Severity:** Medium
  *Basis:* Without clear instructions, different implementations may handle id_type 'none' inconsistently, affecting interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary: Finding-3

---