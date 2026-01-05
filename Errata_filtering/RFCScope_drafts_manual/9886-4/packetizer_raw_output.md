# Errata Reports

Total reports: 5

---


## Report 1: 9886-4-1

**Label:** Conflicting DET DNS Publication Requirements: Unconditional MUST vs. Just-in-Time Recommendation

**Bug Type:** Both

**Explanation:**

The specification unconditionally requires that every DET’s reverse-DNS name must resolve to an HHIT RR (and include a BRID RR for UAS RID), yet later recommends withholding all RR types (just‐in‐time publication) until external use. This normative conflict creates ambiguity about when DNS records must be present.

**Justification:**

- Section 4 states: DETs ... MUST resolve to an HHIT RRType and, for UAS RID, the BRID RRType MUST be present.
- Section 7.2 recommends that no RRTypes be published until and unless they are required for use, which would mean the HHIT RRType is absent.

**Evidence Snippets:**

- **E1:**

  DETs, being IPv6 addresses, are to be under ip6.arpa. (nibble reversed per Section 2.5 of RFC 3596 [STD88]) and MUST resolve to an HHIT RRType. … For UAS RID, the BRID RRType MUST be present to provide the Broadcast Endorsements (BEs) defined in Section 3.1.2.1 of [RFC9575].

- **E2:**

  When practical, it is RECOMMENDED that no RRTypes under a DET's specific domain name be published unless and until it is required for use by other parties. Such action would cause at least the HHIT RRType to not be in the DNS, protecting the public key in the certificate from being exposed before its needed.

**Evidence Summary:**

- (E1) Section 4 mandates constant presence of HHIT (and BRID for UAS RID).
- (E2) Section 7.2 recommends withholding RRTypes until they are needed.

**Fix Direction:**

Clarify that the unconditional MUST in Section 4 applies only when a DET is in active use or subject to public lookup, or alternatively modify Section 7.2 to indicate that just‐in‐time publication is a privacy optimization applicable only pre-activation.

**Severity:** High
  *Basis:* The conflicting requirements could lead to divergent implementations, breaking the intended public lookup and validation mechanisms and impacting security.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ScopeExpert: Issue-1
- CausalExpert: Underspecification
- DeonticExpert: Issue-1
- QuantitativeExpert: Issue-3
- BoundaryExpert: Finding-1

---


## Report 3: 9886-4-3

**Label:** Ambiguous Scope for Mandatory BRID RRType in UAS RID

**Bug Type:** Inconsistency

**Explanation:**

The requirement 'For UAS RID, the BRID RRType MUST be present' is not clearly scoped to specific types of DETs, leaving it unclear whether it applies to all DETs in the UAS RID hierarchy or just a subset (e.g., UA DETs).

**Justification:**

- Section 4 provides the mandate without limiting the requirement to a clearly defined subset of DETs.
- The accompanying discussion and examples suggest that BRID may only be intended for registrant UA DETs, but this is not explicitly stated.

**Evidence Snippets:**

- **E1:**

  Section 4: “For UAS RID, the BRID RRType MUST be present to provide the Broadcast Endorsements (BEs) defined in Section 3.1.2.1 of [RFC9575].”

- **E2:**

  The UAS Broadcast RID Resource Record (BRID, RRType 68) is a format to hold information typically sent over UAS Broadcast RID... Examples in Appendix A of RFC 9886 show: RAAs and HDAs with HHIT RRs only, and a registrant UA DET with both HHIT and BRID RRs.

**Evidence Summary:**

- (E1) Mandatory BRID for UAS RID is asserted without qualification.
- (E2) Examples indicate BRID may be limited to certain DET types but this is not explicitly defined.

**Fix Direction:**

Refine the specification to explicitly list the DET types for which the BRID RRType is required, for example by stating it applies only to UA DETs used in Broadcast RID.

**Severity:** Medium
  *Basis:* This ambiguity may lead to inconsistent configurations, with some implementations over-provisioning BRID and others under-provisioning it, thereby affecting interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- ScopeExpert: Issue-2

---


## Report 4: 9886-4-4

**Label:** HID Abbreviation Length Constraint Conflict in HHIT RRType

**Bug Type:** Inconsistency

**Explanation:**

The CDDL specifies that the hid-abbreviation field must be exactly 15 characters long, yet the normative text and examples mandate and show a 9-character format, causing potential validation and interoperability problems.

**Justification:**

- CDDL for HHIT RRType: hhit-rr = [ hhit-entity-type: uint, hid-abbreviation: tstr .size(15), canonical-registration-cert: bstr ].
- Normative text states that in the absence of policy the abbreviation MUST be in the format '000A 0014', which is 9 characters long.

**Evidence Snippets:**

- **E1:**

  CDDL for HHIT RDATA:
hhit-rr = [ hhit-entity-type: uint, hid-abbreviation: tstr .size(15), canonical-registration-cert: bstr ]

- **E2:**

  Absent of such a policy, this field MUST be filled with the four character hexadecimal representations of the RAA and HDA (in that order) with a separator character, such as a space, in between. For example, a DET with an RAA value of 10 and HDA value of 20 would be abbreviated as: 000A 0014. Example decoded HHIT RDATA: [ 10, "3ff8 0000", ... ]

**Evidence Summary:**

- (E1) The CDDL enforces a fixed size of 15 characters for hid-abbreviation.
- (E2) The default format and examples use a 9-character string, creating a direct conflict.

**Fix Direction:**

Adjust the CDDL to allow a range (e.g., 9 to 15 characters) or otherwise align the size constraint with the normative default format and examples.

**Severity:** High
  *Basis:* A strict validator enforcing the CDDL will reject valid text formats as described in the specification, leading to interoperability failures.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-1
- StructuralExpert: Issue-1

---


## Report 5: 9886-4-5

**Label:** Inconsistent CBOR Array Structure for BRID uas_ids and auth Fields

**Bug Type:** Inconsistency

**Explanation:**

The CDDL indicates that the 'uas_ids' and 'auth' fields in the BRID RRType should be encoded as arrays of grouped substructures, but the provided decoded examples show them as flat arrays of alternating values, leading to incompatible interpretations.

**Justification:**

- CDDL for BRID: uas_ids => [+ uas-id-grp] where uas-id-grp is defined as an array [ id_type, uas_id: bstr .size(20) ].
- Decoded BRID RDATA example shows: 1: [4, h'012001003FFE000A05130824699A4BC6B2'] and 2: [5, h'01FADEF6...', 5, h'0197E0F6...', ...] without nested grouping.

**Evidence Snippets:**

- **E1:**

  CDDL for BRID:

bcast-rr = {
    uas_type => nibble-field,
    uas_ids => [+ uas-id-grp],
    ? auth => [+ auth-grp],
    ...
}
uas-id-grp = [
    id_type: &uas-id-types,
    uas_id: bstr .size(20)
]

- **E2:**

  Decoded BRID RDATA example:
{
    0: 0,
    1: [4, h'012001003FFE000A05130824699A4BC6B2'],
    2: [
        5, h'01FADEF6...',   5, h'0197E0F6...',
        5, h'010AE1F6...',   5, h'01DCE2F6...'
    ]
}

**Evidence Summary:**

- (E1) The CDDL requires uas_ids and auth as arrays of arrays (grouped substructures).
- (E2) The example shows a flat array structure, causing ambiguity in interpretation.

**Fix Direction:**

Revise the CDDL and/or the examples to clearly indicate whether the arrays must be nested (arrays of arrays) or flat arrays, ensuring consistent CBOR encoding.

**Severity:** High
  *Basis:* Differing interpretations in CBOR structure can result in encoding/decoding failures between implementations, undermining interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-2

---


## Report 6: 9886-4-6

**Label:** Optional BRID auth Field Conflicts with Mandatory Broadcast Endorsements Requirement

**Bug Type:** Underspecification

**Explanation:**

While the CDDL marks the BRID 'auth' field as optional, the normative text implies that in the UAS RID use case the BRID must provide Broadcast Endorsements, leaving it unclear whether the 'auth' field must be present and populated.

**Justification:**

- CDDL for BRID: '? auth => [+ auth-grp]' indicates that the auth field is optional.
- Normative text states that for UAS RID the BRID RRType MUST be present to provide the Broadcast Endorsements (BEs) as defined in RFC9575.

**Evidence Snippets:**

- **E1:**

  CDDL: bcast-rr = { uas_type => nibble-field, uas_ids => [+ uas-id-grp], ? auth => [+ auth-grp], ... }

- **E2:**

  Section 4: “For UAS RID, the BRID RRType MUST be present to provide the Broadcast Endorsements (BEs) defined in Section 3.1.2.1 of [RFC9575].”

**Evidence Summary:**

- (E1) The CDDL makes the auth field optional.
- (E2) The normative requirement implies that Broadcast Endorsements must be provided via the auth field for UAS RID.

**Fix Direction:**

Clarify that when BRID is used in the UAS RID context, the auth field is mandatory and must contain at least one Broadcast Endorsement, updating the CDDL accordingly.

**Severity:** Medium
  *Basis:* This ambiguity can lead to divergent implementations where some systems may omit necessary endorsements, affecting security and validation without necessarily breaking basic functionality.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-4

---