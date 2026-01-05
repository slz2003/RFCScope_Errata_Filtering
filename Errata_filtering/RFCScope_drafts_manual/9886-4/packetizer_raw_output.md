# Errata Reports

Total reports: 6

---

## Report 1: 9886-4-1

**Label:** Ambiguous Applicability of HHIT/BRID Publication “MUST” vs. Optional Pre-Use Withholding (“Just-in-Time”)

**Bug Type:** Underspecification

**Explanation:**

The document uses unconditional “MUST resolve” / “BRID MUST be present” language for DET reverse-DNS lookups, but later recommends (when practical) withholding all RRTypes under a DET name until needed. This is likely intended as a lifecycle distinction (records required once a DET is published/used), but the current text does not explicitly state when the Section 4 MUSTs apply (e.g., “for published/active DETs”), which can lead to divergent implementations and compliance interpretations.

**Justification:**

- Section 4 states that a DET’s reverse-DNS name MUST resolve to an HHIT RRType and, for UAS RID, the BRID RRType MUST be present—without an explicit condition such as “when published for use” or “once registered/activated”.
- Section 7.2 RECOMMENDS delaying publication of any RRTypes under the DET name until required by other parties, explicitly noting this would mean at least the HHIT RRType is absent.
- Without an explicit applicability condition, an implementer could reasonably conclude either:

(A) HHIT/BRID must always be present for every allocated DET (continuous publication), or

(B) HHIT/BRID may be absent until operational need arises (just-in-time publication),
resulting in inconsistent expectations for observers performing lookups and for registries asserting compliance.

**Evidence Snippets:**

- **E1:**

  “DETs, being IPv6 addresses, are to be under ip6.arpa. … and MUST resolve to an HHIT RRType. … For UAS RID, the BRID RRType MUST be present …”

- **E2:**

  “When practical, it is RECOMMENDED that no RRTypes under a DET's specific domain name be published unless and until it is required for use by other parties. Such action would cause at least the HHIT RRType to not be in the DNS …”

**Evidence Summary:**

- (E1) uses unconditional MUST language for HHIT/BRID presence in DNS for DET lookups.
- (E2) recommends withholding RRTypes pre-use, implying HHIT (and thus BRID) may be absent until needed.
Suggested clarification: qualify Section 4 MUSTs with an explicit condition such as “for DETs published for public discovery / in active use” or similar lifecycle wording.

**Fix Direction:**

The fix should clarify applicability and lifecycle of existing MUST/RECOMMENDED language, e.g., by qualifying when the Section 4 MUSTs apply (“for published / active / discoverable DETs”) or by adding a short cross-reference note between Sections 4 and 7.2.

**Severity:** High
  *Basis:* Interoperability risk

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ScopeExpert: Issue-1
- CausalExpert: Underspecification
- DeonticExpert: Issue-1
- QuantitativeExpert: Issue-3
- BoundaryExpert: Finding-1

---

## Report 2: 9886-4-2

**Label:** Unspecified Timing for DNS Publication Leading to Races

**Bug Type:** Underspecification

**Explanation:**

The document does not specify precise temporal requirements for when DNS records for a DET should be published relative to its operational use, leaving potential gaps during which lookups may fail.

**Justification:**

- Section 3.1 describes Dynamic DRIP registration where HHIT records are added before a flight and deleted afterwards without specifying exact timings.
- The text states that 'Optimally this requires that the UAS somehow signal to the DIME that a flight using a Specific Session ID will soon be underway or complete,' but gives no concrete guidance.

**Evidence Snippets:**

- **E1:**

  Dynamic DRIP registration is another possible solution, for example when the operator of a UAS device registers its corresponding HHIT record and other resources before a flight and deletes them afterwards.

- **E2:**

  Optimally this requires that the UAS somehow signal to the DIME that a flight using a Specific Session ID will soon be underway or complete.

**Evidence Summary:**

- (E1) The registration and deletion pattern is described without precise time bounds.
- (E2) The expected signaling for publication is mentioned without detailed timing requirements.

**Fix Direction:**

Introduce explicit timing constraints for the publication and retention of DNS records, including requirements on minimum publication windows and handling of negative caching.

**Severity:** Medium
  *Basis:* Ambiguity in timing may lead to race conditions where DET lookups intermittently fail, impacting interoperability without necessarily breaking conformance.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T2
- CausalExpert

---

## Report 3: 9886-4-3

**Label:** Ambiguous Scope for Mandatory BRID RRType in UAS RID

**Bug Type:** Underspecification

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
