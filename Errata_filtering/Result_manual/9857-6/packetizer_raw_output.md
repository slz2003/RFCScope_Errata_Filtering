# Errata Reports

Total reports: 3

---

## Report 1: 9857-6-1

**Label:** Ambiguous usage of 'originator' term in SR Policy TLVs (Section 4)

**Bug Type:** Underspecification

**Explanation:**

The document uses the term 'originator' in two different senses – one for the SR Policy candidate-path originator and another for the BGP‑LS Producer – which may lead to incorrect population of the Originator ASN/Address fields.

**Justification:**

- The text in Section 4 defines 'Originator ASN' and 'Originator Address' with references that are used both for candidate-path creation and for BGP‑LS transmission, causing potential confusion.
- The repeated instruction that 'Other bits MUST be cleared by the originator and MUST be ignored by a receiver' adds to the ambiguity about which entity is responsible.

**Evidence Snippets:**

- **E1:**

  Originator ASN: 4 octets to carry the 4-byte encoding of the ASN of the originator. Refer to Section 2.4 of [RFC9256] for details. (Section 4)

- **E2:**

  Originator Address: 4 or 16 octets … to carry the address of the originator. Refer to Section 2.4 of [RFC9256] for details. (Section 4)

- **E3:**

  Repeated patterns such as 'Other bits MUST be cleared by the originator and MUST be ignored by a receiver.' (Referenced in Section 5)

**Evidence Summary:**

- (E1) defines the Originator ASN in Section 4 for a candidate-path originator.
- (E2) defines the Originator Address in Section 4 with similar ambiguity.
- (E3) shows repeated instructions that do not disambiguate the roles.

**Fix Direction:**

Clarify the terminology by introducing distinct terms such as 'SR Policy Originator' for the entity that creates the candidate path and 'BGP‑LS Producer' for the node sending the update.


**Severity:** Low
  *Basis:* The ambiguity could lead to minor implementation misinterpretations without necessarily breaking interoperability.

**Confidence:** High

---

## Report 2: 9857-6-2

**Label:** Unclear mandatory/optional status for SR Candidate Path State and SR Segment List TLVs in Section 6

**Bug Type:** Underspecification

**Explanation:**

Section 6 describes the inclusion of TLVs such as the SR Candidate Path State TLV (1202) and the SR Segment List TLV (1205) in a descriptive manner without normative RFC 2119 terminology, leaving their mandatory versus optional status ambiguous.

**Justification:**

- The procedural text in Section 6 uses phrases like 'is included to report' without explicitly stating if these TLVs are mandatory, despite RFC 9552’s default that TLVs are optional unless specified.
- This ambiguity could cause different implementations to either require or treat these TLVs as optional, leading to inconsistent interpretations across devices.

**Evidence Snippets:**

- **E1:**

  The SR Candidate Path State TLV as defined in Section 5.3 is included to report the state of the candidate path. (Section 6)

- **E2:**

  The SR Segment List TLV is included for each SID list(s) associated with the candidate path. (Section 6)

- **E3:**

  For both the NLRI and BGP‑LS Attribute parts, all TLVs are considered as optional except where explicitly specified as mandatory or required in specific conditions. (RFC 9552 reference)

**Evidence Summary:**

- (E1) quotes the descriptive inclusion of the SR Candidate Path State TLV in Section 6.
- (E2) shows the similar inclusion of the SR Segment List TLV in Section 6.
- (E3) cites the default rule from RFC 9552 that TLVs are optional unless explicitly mandated.

**Fix Direction:**

Revise Section 6 to explicitly state which TLVs are mandatory and which remain optional, using RFC 2119 language (e.g., MUST, SHOULD).


**Severity:** Medium
  *Basis:* Ambiguity in TLV mandatory status could lead to interoperability issues if different implementations adopt divergent interpretations.

**Confidence:** High

---

## Report 3: 9857-6-3

**Label:** Ambiguous reference to 'SR BSID TLV' conflates distinct TLVs (SR Binding SID TLV and SRv6 Binding SID TLV)

**Bug Type:** Inconsistency

**Explanation:**

Section 6 refers to a singular 'SR BSID TLV' even though Sections 5.1 and 5.2 define two distinct TLVs—SR Binding SID TLV (Type 1201) and SRv6 Binding SID TLV (Type 1212)—with different usage rules and multiplicity.

**Justification:**

- The text in Section 6 states 'The SR BSID TLV as defined in Sections 5.1 and 5.2 is included to report the BSID…' which implies a single TLV.
- Sections 5.1 and 5.2 clearly separate the definitions and intended usage: the SR Binding SID TLV is defined with a single-instance usage, while the SRv6 Binding SID TLV allows multiple instances and is the proper container for SRv6 BSIDs.

**Evidence Snippets:**

- **E1:**

  The SR BSID TLV as defined in Sections 5.1 and 5.2 is included to report the BSID of the candidate path when one is either specified or allocated by the headend. (Section 6)

- **E2:**

  ### 5.1. SR Binding SID TLV … The SR Binding Segment Identifier (BSID) is an optional TLV … Type: 1201 (Section 5.1)

- **E3:**

  ### 5.2. SRv6 Binding SID TLV … The SRv6 Binding SID (BSID) is an optional TLV … Type: 1212 (Section 5.2)

- **E4:**

  There is no TLV named 'SR BSID TLV' in the IANA TLV registry, indicating that the term is not formally defined. (Terminology Analysis)

**Evidence Summary:**

- (E1) shows the Section 6 reference to a singular 'SR BSID TLV'.
- (E2) provides the definition of the SR Binding SID TLV from Section 5.1.
- (E3) provides the definition of the SRv6 Binding SID TLV from Section 5.2.
- (E4) confirms that the umbrella term 'SR BSID TLV' is not present in the IANA registry.

**Fix Direction:**

Update Section 6 to explicitly refer to the two distinct TLVs—'SR Binding SID TLV' (Type 1201) and 'SRv6 Binding SID TLV' (Type 1212)—and clarify their individual roles and allowed multiplicity.


**Severity:** Medium
  *Basis:* This inconsistency can lead to confusion among implementers and potential misinterpretation of BSID reporting rules, affecting protocol interoperability.

**Confidence:** High

---
