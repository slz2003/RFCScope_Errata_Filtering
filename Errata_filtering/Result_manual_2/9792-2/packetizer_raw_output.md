# Errata Reports

Total reports: 3

---

## Report 1: 9792-2-1

**Label:** Over‐broad LSA Malformation Rule for non‐4‐octet Prefix Extended Flags sub‐TLV

**Bug Type:** Both

**Explanation:**

When the Prefix Extended Flags sub‐TLV has a length that is not a multiple of 4 octets, the specification mandates that the entire LSA be discarded, which conflicts with the generic TLV error‐handling model that would normally isolate such errors to the sub‐TLV.

**Justification:**

- RFC 9792 mandates that if the sub‐TLV length is not a multiple of 4, 'the Link State Advertisement (LSA) is malformed and MUST be ignored', despite the sub‐TLV being optional.
- RFC 8362 states that for backward compatibility, an LSA is not considered malformed unless a required TLV is missing or too short, indicating that errors in optional TLVs should not invalidate the whole LSA.
- Multiple experts note that this discrepancy can lead to divergent processing of LSAs among compliant implementations.

**Evidence Snippets:**

- **E1:**

  The length MUST be a multiple of 4 octets. If the length is not a multiple of 4 octets, the Link State Advertisement (LSA) is malformed and MUST be ignored.

- **E2:**

  For backward compatibility, an LSA is not considered malformed from a TLV perspective unless either a required TLV is missing or a specified TLV is less than the minimum required length.

- **E3:**

  Under the generic OSPF TLV rules (RFC 8362), a TLV with an odd length is processed with padding and does not render the entire LSA malformed, contrasting with the rule in RFC 9792.

**Evidence Summary:**

- (E1) RFC 9792 mandates LSA malformation on non‐4‐octet length.
- (E2) RFC 8362 limits LSA malformation to required TLV errors.
- (E3) The generic TLV framework handles non‐4‐octet lengths via padding rather than LSA-wide rejection.

**Fix Direction:**

Revise the error condition so that if the Prefix Extended Flags sub‐TLV’s length is not a multiple of 4, only the sub‐TLV is ignored rather than the entire LSA. For example, change the text to 'if the length is not a multiple of 4 octets, this sub‐TLV is malformed and MUST be ignored; the rest of the LSA MUST still be processed normally.'

**Severity:** Medium
  *Basis:* The rule conflicts with established TLV error-handling guidelines and can lead to divergent LSDB states, impacting interoperability even though it arises only in error cases.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- CrossRFC Expert: Issue-1
- Boundary Expert: Finding-1

---

## Report 2: 9792-2-2

**Label:** OSPFv3 IANA Registry Misreference to OSPFv2 Prefix Extended Flags sub‐TLV

**Bug Type:** Inconsistency

**Explanation:**

The OSPFv3 IANA registry text incorrectly refers to the OSPFv2 Prefix Extended Flags sub‐TLV, creating confusion about which sub‐TLV the registry is meant to define.

**Justification:**

- Section 4.2.2 of RFC 9792, under the heading 'OSPFv3 Prefix Extended Flags Registry', states that the registry defines bits in the OSPFv2 Prefix Extended Flags sub‐TLV.
- This is inconsistent with Section 4.1.2 which correctly refers to the OSPFv2 sub‐TLV and contradicts the clear separation intended between OSPFv2 and OSPFv3 parameters.
- Experts note that this is likely a copy‐and‐paste error that may mislead implementers or IANA consumers.

**Evidence Snippets:**

- **E1:**

  Section 4.2.2, heading: “OSPFv3 Prefix Extended Flags Registry”. Immediately following text: “The registry defines the bits in the Prefix Extended Flags field in the OSPFv2 Prefix Extended Flags sub-TLV as specified in Section 2.”

- **E2:**

  In Section 4.1.2, the OSPFv2 registry is correctly described as defining the bits in the Prefix Extended Flags field in the OSPFv2 Prefix Extended Flags sub‐TLV.

**Evidence Summary:**

- (E1) The OSPFv3 registry text incorrectly names the OSPFv2 sub‐TLV.
- (E2) The correct OSPFv2 registry text contrasts with the misreference in the OSPFv3 section.

**Fix Direction:**

Replace the reference to 'OSPFv2 Prefix Extended Flags sub-TLV' with 'OSPFv3 Prefix Extended Flags sub-TLV' in Section 4.2.2.

**Severity:** Low
  *Basis:* The misreference is an editorial error that may cause confusion but does not impact on‐the‐wire behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- Deontic Expert: Issue-2
- Structural Expert: Issue-3
- Terminology Expert: Issue-1
- CrossRFC Expert: Issue-2

---

## Report 3: 9792-2-3

**Label:** Stylistic Imprecision in Sub‐TLV Length Limiting Requirement

**Bug Type:** Editorial

**Explanation:**

The instruction 'MUST limit the length of the sub‑TLV so as to signal the bits that are set to 1' is imprecisely phrased without a clear, algorithmic definition, although receiver behavior remains well-defined.

**Justification:**

- The wording does not explicitly require that the sender use the minimal length required to cover the highest set bit, potentially leading to inconsistent sender implementations.
- Despite the imprecision, receiver obligations remain clear since undefined bits beyond the transmitted length must be treated as 0.

**Evidence Snippets:**

- **E1:**

  An implementation MUST limit the length of the sub‑TLV so as to signal the bits that are set to 1. Defined prefix flags that are not transmitted due to being beyond the transmitted length MUST be treated as being set to 0.

**Evidence Summary:**

- (E1) The requirement for limiting the sub‐TLV length is stated in an informal manner, lacking a precise specification for how to choose the length.

**Fix Direction:**

Clarify the requirement by specifying a concrete algorithm (e.g., the length should be the smallest multiple of 4 that includes the highest-numbered set bit) to remove any ambiguity for senders.

**Severity:** Low
  *Basis:* The imprecise wording only affects encoding clarity and does not compromise overall interoperability or receiver processing.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-3
- Boundary Expert: Finding-2

---
