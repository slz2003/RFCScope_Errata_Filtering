# Errata Reports

Total reports: 6

---

## Report 1: 9886-5-1

**Label:** HID Abbreviation length mismatch: CDDL .size(15) vs normative 9‐character format

**Bug Type:** Inconsistency

**Explanation:**

The HHIT RR's 'hid-abbreviation' field is defined by the CDDL to be exactly 15 characters, yet the normative prose and examples require a 9-character format. This contradiction can lead to interoperable implementations disagreeing about what constitutes valid data.

**Justification:**

- CDDL (Figure 4) mandates 'hid-abbreviation' as 'tstr .size(15)', enforcing an exact length of 15.
- The descriptive prose requires a 4+1+4 (9-character) hexadecimal format (e.g., 000A 0014) and all provided examples use a 9‐character string.

**Evidence Snippets:**

- **E1:**

  CDDL for HHIT RR: hhit-rr = [ hhit-entity-type: uint, hid-abbreviation: tstr .size(15), canonical-registration-cert: bstr ] (Figure 4).

- **E2:**

  HID Abbreviation prose: “Absent of such a policy, this field MUST be filled with the four character hexadecimal representations of the RAA and HDA (in that order) with a separator character, such as a space, in between. For example, a DET with an RAA value of 10 and HDA value of 20 would be abbreviated as: 000A 0014.”

- **E3:**

  HHIT examples in Appendix A: decoded arrays like [ 10, "3ff8 0000", h'30820142…' ] and [ 14, "3ff8 000a", h'30820143…' ] show 9-character abbreviations.

**Evidence Summary:**

- (E1) The CDDL specifies a fixed length of 15 for 'hid-abbreviation'.
- (E2) The normative text requires a 9-character default value.
- (E3) The provided examples conform to the 9-character format.

**Fix Direction:**

Align the CDDL and prose: either change '.size(15)' to '.size(9)' (or an appropriate range) or update the textual examples to conform to a 15‐character format.

**Severity:** High
  *Basis:* This direct normative contradiction can lead to rejected records by CDDL-based validators, impeding interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-1
- Causal: Issue-1
- Quantitative: Issue-1
- Deontic: Issue-1
- Structural: Issue-1
- CrossRFC: Issue-1
- Boundary: Finding-1

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

## Report 3: 9886-5-3

**Label:** Temporal scope conflict: DET reverse name resolution requirements inconsistent

**Bug Type:** Inconsistency

**Explanation:**

There is a contradiction between Section 4's requirement that DET reverse names must resolve to an HHIT (and for UAS RID, a BRID) RRType and Section 7.2's recommendation to omit RRTypes until needed. This creates uncertainty about when the mandatory resolution applies.

**Justification:**

- Section 4 mandates that DET reverse names “MUST resolve to an HHIT RRType” and for UAS RID, the BRID RRType must be present.
- Section 7.2 recommends that no RRTypes be published under a DET's domain name until they are required for use.

**Evidence Snippets:**

- **E1:**

  Section 4: “DETs, being IPv6 addresses, are to be under ip6.arpa. … and MUST resolve to an HHIT RRType. … For UAS RID, the BRID RRType MUST be present to provide the Broadcast Endorsements…”

- **E2:**

  Section 7.2: “When practical, it is RECOMMENDED that no RRTypes under a DET's specific domain name be published unless and until it is required for use by other parties.”

**Evidence Summary:**

- (E1) Section 4 imposes a mandatory resolution requirement.
- (E2) Section 7.2 advises against publishing RRTypes until needed.

**Fix Direction:**

Explicitly define the DET lifecycle and state the conditions under which the mandatory resolution applies versus when RR publication is deferred.

**Severity:** Medium
  *Basis:* Unclear temporal application may lead to inconsistent DNS publication practices and potential record absence in critical moments.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-3

---

## Report 4: 9886-5-4

**Label:** BRID structure mismatch: CDDL nested arrays vs flattened example encoding

**Bug Type:** Inconsistency

**Explanation:**

The BRID RRType CDDL specifies that 'uas_ids' and 'auth' should be encoded as arrays of nested groups, yet the provided decoded examples use flat arrays. This structural discrepancy may cause decoders to misinterpret the data.

**Justification:**

- The CDDL (Figure 5) defines uas_ids as an array of uas-id-grp (each of which is an array) and auth as an array of auth-grp arrays.
- Decoded examples (Figure 21) show a single-level flat array for uas_ids and an alternating flat array for auth instead of nested arrays.

**Evidence Snippets:**

- **E1:**

  CDDL for BRID (Figure 5): uas-id-grp = [ id_type: &uas-id-types, uas_id: bstr .size(20) ] and auth-grp = [ a_type: &auth-types, a_data: bstr .size(1..362) ]

- **E2:**

  Decoded BRID example (Figure 21): Key 1’s value is a single array [4, h'012001003FFE000A05130824699A4BC6B2'], and Key 2’s value is a flat array alternating a_type and a_data values.

**Evidence Summary:**

- (E1) The CDDL specifies nested arrays for uas_ids and auth.
- (E2) The examples use a flat array structure.

**Fix Direction:**

Revise either the CDDL to match the flat array encoding found in the examples or update the examples to conform to the nested array structure required by the CDDL.

**Severity:** High
  *Basis:* Mismatch in the structured encoding can lead to decoder failures and interoperability problems.

**Confidence:** High

**Experts mentioning this issue:**

- Quantitative: Issue-2

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
