# Errata Reports

Total reports: 5

---

## Report 1: 9870-4-1

**Label:** Unclear Flow-Aggregation and Deduplication Semantics for udpSafeExIDList/udpUnsafeExIDList

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define whether udpSafeExIDList/udpUnsafeExIDList should aggregate all distinct ExIDs observed in a Flow or simply report a single occurrence, nor does it specify ordering or deduplication behavior.

**Justification:**

- The IE definitions only state that the lists report 'Observed ExIDs' without clarifying if they are to be aggregated over the entire Flow or if duplicates are allowed.
- There is no normative guidance on whether multiple occurrences of the same ExID should be collapsed into one element or retained as separate entries.

**Evidence Snippets:**

- **E1:**

  udpExID: “Observed ExID in an Experimental (EXP, Kind=127) Option or an UNSAFE Experimental (UEXP, Kind=254) Option. A basicList of udpExID is used to report udpSafeExIDList and udpUnsafeExIDList values.”

- **E2:**

  udpSafeExIDList: “Observed ExIDs in the Experimental (EXP, Kind=127) Option. A basicList of udpExID Information Elements in which each udpExID Information Element carries the ExID observed in an EXP Option.”

**Evidence Summary:**

- (E1) shows that the IE simply reports observed ExIDs without specifying aggregation semantics.
- (E2) reinforces the lack of guidance regarding deduplication or ordering.

**Fix Direction:**

Clarify the IE definitions to state that each list MUST contain each distinct ExID observed in a Flow exactly once and, if desired, define a canonical ordering.


**Severity:** Medium
  *Basis:* Ambiguity in aggregation and deduplication may lead to inconsistent Flow metrics across implementations.

**Confidence:** High

---

## Report 2: 9870-4-2

**Label:** Contradictory EXP/UEXP Bit Semantics in Presence of ExID Lists

**Bug Type:** Inconsistency

**Explanation:**

The document defines that a bit is set to 1 if the corresponding option is observed, yet mandates that when the corresponding ExID list is present the EXP/UEXP bit MUST be set to 0, creating an apparent contradiction in bit semantics.

**Justification:**

- The general rule states that a bit is set to 1 if the option is observed in the Flow, but later text forces the bit to 0 when an ExID list is exported.
- This creates an internal inconsistency by implying an option can be both observed (via the list) and not observed (via a cleared bit).

**Evidence Snippets:**

- **E1:**

  “The bit is set to 1 if the corresponding SAFE UDP Option is observed at least once in the Flow. The bit is set to 0 if the option is never observed in the Flow.” … “The presence of udpSafeExIDList takes precedence over setting the corresponding bit in the udpSafeOptions IE for the same Flow. In order to optimize the use of the reduced-size encoding in the presence of udpSafeExIDList IE, the Exporter MUST NOT set the EXP flag of the udpSafeOptions IE that is reported for the same Flow to 1.”

- **E2:**

  “In order to optimize the use of the reduced-size encoding in the presence of udpUnsafeExIDList IE, the Exporter MUST NOT set the UEXP flag of the udpUnsafeOptions IE that is reported for the same Flow to 1.”

**Evidence Summary:**

- (E1) shows the contradictory rule for udpSafeOptions where the generic observation rule is overridden by a MUST NOT condition when an ExID list is present.
- (E2) provides a similar example for udpUnsafeOptions.

**Fix Direction:**

Revise the wording to explicitly state that the generic ‘observed’ rule does not apply to EXP/UEXP bits when the corresponding ExID list is present.


**Severity:** Low
  *Basis:* The conflict is primarily a wording issue and is mitigated by the explicit MUST NOT rule; it is unlikely to impact interoperability.

**Confidence:** High

---

## Report 3: 9870-4-3

**Label:** Incorrect '24 Octets' Example for Multiplexing IE

**Bug Type:** Inconsistency

**Explanation:**

The documentation claims that reporting mandatory SAFE Options using a single multiplexing IE would require at least 24 octets, which does not align with the reduced-size encoding calculations based on the mandatory option range.

**Justification:**

- Mandatory SAFE options are defined for Kinds 0–7, which would require only 1 octet when encoded with reduced-size encoding.
- The stated 24 octets correspond to a full 192-bit field, an interpretation that does not match any realistic scenario for mandatory options.

**Evidence Snippets:**

- **E1:**

  Section 4 says: “using one single IE that would multiplex both types of options will limit the benefits of reduced-size encoding in the presence of UNSAFE Options. For example, at least 24 octets would be needed to report mandatory SAFE Options that are observed in a Flow. … As further detailed in Section 5.1, only one octet is needed to report mandatory SAFE Options.”

- **E2:**

  Mandatory SAFE options (‘must‑support’ marked with ‘*’) in RFC 9868 are Kinds 0–7 (EOL, NOP, APC, FRAG, MDS, MRDS, REQ, RES), not the entire 0–191 SAFE range.

**Evidence Summary:**

- (E1) provides the conflicting statements regarding field length, with one part claiming 24 octets and another stating only one octet is needed.
- (E2) clarifies that mandatory options only cover Kinds 0–7, leading to a different expected encoding length.

**Fix Direction:**

Correct the explanatory text to accurately reflect the reduced-size encoding calculations based on the mandatory option range.


**Severity:** Low
  *Basis:* This error is confined to the explanatory text and does not affect the normative wire format, though it may confuse implementers about overhead expectations.

**Confidence:** High

---

## Report 4: 9870-4-4

**Label:** Ambiguous Reduced-Size Encoding for unsigned256

**Bug Type:** Inconsistency

**Explanation:**

The document specifies that reduced-size encoding per RFC 7011 applies to an unsigned256 type (udpSafeOptions) and provides examples of encoding as unsigned32/unsigned64, even though RFC 7011 only permits reduced-size encoding for up to 64-bit integers.

**Justification:**

- The examples imply a change in abstract data type based on the maximum observed option Kind, conflicting with the static type assignment of unsigned256.
- RFC 7011 §6.2 explicitly lists allowed types and does not include unsigned256, leaving it ambiguous whether the reduced-size encoding rules are meant to extend to this new type.

**Evidence Snippets:**

- **E1:**

  Abstract Data Type:  unsigned256 … The reduced-size encoding per Section 6.2 of [RFC7011] is followed whenever fewer octets are needed to report observed SAFE UDP Options. For example, if only option Kinds <= 31 are observed, then the value of the udpSafeOptions IE can be encoded as unsigned32, or if only option Kinds <= 63 are observed, then the value of the udpSafeOptions IE can be encoded as unsigned64.

- **E2:**

  RFC 7011 Section 6.2: Reduced-size encoding MAY be applied to the following integer types: unsigned64, signed64, unsigned32, signed32, unsigned16, and signed16. … Reduced-size encoding MUST NOT be applied to any other data type defined in [RFC7012] that implies a fixed length…

**Evidence Summary:**

- (E1) shows that the document provides examples of encoding an unsigned256 field with reduced sizes that resemble other integer types.
- (E2) highlights that the original RFC only permits reduced-size encoding for types up to 64 bits, leaving unsigned256 ambiguous.

**Fix Direction:**

Amend the specification to clarify that udpSafeOptions remains an unsigned256 field with a reduced field length achieved by dropping leading zero octets, and explicitly extend or reference the appropriate update that permits this behavior.


**Severity:** Medium
  *Basis:* This ambiguity can lead to divergent exporter implementations and confusion over the allowed encoded lengths on the wire.

**Confidence:** High

---

## Report 5: 9870-4-5

**Label:** Ambiguous 'EXP flag/bit' Terminology in UDP Options

**Bug Type:** Inconsistency

**Explanation:**

The specification uses the terms 'EXP flag/bit' and 'UEXP flag' without explicitly clarifying that these refer to the bits corresponding to UDP Option Kind 127 and Kind 254, respectively, which may lead to confusion.

**Justification:**

- The text discusses setting or not setting the EXP/UEXP flag without tying them explicitly to the respective bit positions within the flags fields.
- This ambiguity in terminology could lead implementers or readers to mistakenly infer that separate or dedicated fields are used for EXP/UEXP rather than the designated bits in the mapped bitfields.

**Evidence Snippets:**

- **E1:**

  Section 4.1: “The presence of udpSafeExIDList takes precedence over setting the corresponding bit in the udpSafeOptions IE for the same Flow. In order to optimize the use of the reduced-size encoding in the presence of udpSafeExIDList IE, the Exporter MUST NOT set the EXP flag of the udpSafeOptions IE that is reported for the same Flow to 1.”

- **E2:**

  Section 5.2: “If a udpSafeOptions IE is exported for this Flow, then that IE will have the EXP bit set to 1 (Figure 4).”

**Evidence Summary:**

- (E1) demonstrates the use of the term ‘EXP flag’ in a way that does not explicitly identify it as the bit corresponding to UDP Option Kind 127.
- (E2) similarly uses ‘EXP bit’ without clarifying its exact mapping.

**Fix Direction:**

Revise the document to explicitly state that the 'EXP flag/bit' is the bit corresponding to UDP Option Kind 127 in udpSafeOptions and that the 'UEXP flag/bit' corresponds to UDP Option Kind 254 in udpUnsafeOptions.


**Severity:** Low
  *Basis:* This is a minor clarity issue that does not affect protocol functionality but may confuse implementers regarding the mapping of option kinds.

**Confidence:** High

---
