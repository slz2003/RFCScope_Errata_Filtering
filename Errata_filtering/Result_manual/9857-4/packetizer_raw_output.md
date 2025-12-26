# Errata Reports

Total reports: 2

---

## Report 1: 9857-4-1

**Label:** Ambiguous multiplicity of SR Policy Candidate Path Descriptor TLV (Type 554) within SR Policy Candidate Path NLRI

**Bug Type:** Underspecification

**Explanation:**

The specification does not explicitly state that exactly one instance of TLV 554 must appear per SR Policy Candidate Path NLRI, leaving receiver behavior undefined in case of duplicates.

**Justification:**

- The text states: The SR Policy Candidate Path Descriptor TLV identifies an SR Policy candidate path and is declared as a mandatory TLV for the NLRI, yet it does not specify that it MUST appear exactly once, unlike other TLVs in Section 5.
- Figure 3 shows a single TLV instance even though later sections explicitly define duplicate handling for other TLVs.
- This ambiguity can lead to divergent implementations when multiple instances are received.

**Evidence Snippets:**

- **E1:**

  “The SR Policy Candidate Path Descriptor TLV identifies an SR Policy candidate path… It is a mandatory TLV for the SR Policy Candidate Path NLRI type.” (Section 4)

- **E2:**

  Figure 3 shows the NLRI format with a single “// SR Policy Candidate Path Descriptor TLV //” line.

- **E3:**

  By contrast, many attribute TLVs later in Section 5 explicitly specify cardinality and behavior on duplicates, e.g., “Only a single instance of this TLV is advertised for a given candidate path. If multiple instances are present, then the first valid one… is used and the rest are ignored.” (Sections 5.1, 5.3, 5.4, 5.5, 5.6, etc.).

**Evidence Summary:**

- (E1) indicates that TLV 554 is mandatory in the NLRI but without an explicit uniqueness requirement.
- (E2) visually suggests a single instance.
- (E3) contrasts TLV 554 with other TLVs that have explicit duplicate handling instructions.

**Fix Direction:**

Clarify that exactly one SR Policy Candidate Path Descriptor TLV MUST be present per NLRI and specify receiver behavior if duplicates appear.


**Severity:** Medium
  *Basis:* The underspecification may result in inconsistent handling by different implementations, affecting candidate path correlation and interoperability.

**Confidence:** High

---

## Report 2: 9857-4-2

**Label:** Implicit mapping ambiguity between 4/16-octet Originator Address and RFC 9256’s 128-bit Node Address

**Bug Type:** Underspecification

**Explanation:**

The specification leaves the conversion of the compact 4- or 16-octet Originator Address field into the architecturally defined 128-bit Node Address implicit, which can lead to inconsistent interpretation across protocols.

**Justification:**

- Section 4 specifies the Originator Address as 4 or 16 octets (as indicated by the flags) and refers to RFC 9256 for details, but does not explicitly state the mapping rules for IPv4 addresses.
- RFC 9256 defines a 128-bit Node Address with IPv4 addresses encoded in the low 32 bits and high-order bits set to 0, yet the RFC 9857 text does not clarify that the 4-octet representation should be zero-extended.
- CrossRFC analysis highlights that the ambiguity in mapping may affect inter-protocol comparisons and tie-breaking decisions.

**Evidence Snippets:**

- **E1:**

  Section 4: “Originator ASN: 4 octets to carry the 4-byte encoding of the ASN of the originator. Refer to Section 2.4 of [RFC9256] for details.”

- **E2:**

  Section 4: “Originator Address: 4 or 16 octets (as indicated by the flags) to carry the address of the originator. Refer to Section 2.4 of [RFC9256] for details.”

- **E3:**

  RFC 9256 §2.4: “The Originator is expressed in the form of a 160-bit numerical value formed by the concatenation of the fields of the tuple <Autonomous System Number (ASN), node-address>… Node Address: represented as a 128-bit value. IPv4 addresses MUST be encoded in the lowest 32 bits, and the high-order bits MUST be set to 0.”

- **E4:**

  For IPv4 Originator addresses, RFC 9857 clearly defines an on-the-wire size of 4 octets (not 16 with zero padding), but it never explicitly states how that 4-octet field is to be interpreted in terms of RFC 9256’s conceptual 128-bit node address (where high bits MUST be zero).

**Evidence Summary:**

- (E1) and (E2) provide the on-the-wire field definitions while deferring details to RFC 9256.
- (E3) specifies the canonical 128-bit representation for the Originator along with IPv4 mapping rules.
- (E4) emphasizes the lack of explicit mapping instructions for the 4-octet Originator Address field in RFC 9857.

**Fix Direction:**

Add a clarifying statement that explicitly indicates a 4-octet Originator Address must be zero-extended to form the 128-bit Node Address as defined in RFC 9256 §2.4 for IPv4, while a 16-octet field is used for IPv6.


**Severity:** Medium
  *Basis:* The implicit mapping can lead to inconsistent interpretation when comparing Originator values across different protocols, potentially causing interoperability issues.

**Confidence:** Medium

---
