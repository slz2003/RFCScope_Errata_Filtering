# Errata Reports

Total reports: 3

---

## Report 1: 9716-4-1

**Label:** Underspecified error handling for Segment sub-TLVs 46–48 in TLVs 1 or 16

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that if Segment sub-TLVs 46–48 appear in TLVs 1 or 16, 'appropriate error codes' must be returned, but it fails to specify which error codes or procedures to use, causing ambiguity.

**Justification:**

- Multiple expert analyses note that RFC 9716 delegates error handling to RFC8029 without clarifying the exact Return Code, Subcode, or diagnostic TLV details.
- This underspecification may lead to divergent implementations and inconsistent error reporting in operational diagnostics.

**Evidence Snippets:**

- **E1:**

  If these sub-TLVs appear in TLVs 1 or 16, appropriate error codes MUST be returned as defined in [RFC8029].

- **E2:**

  RFC 8029 defines mechanisms for malformed echo requests and “TLVs not understood” at the top-level, but does not specify a procedure for unrecognized sub-TLV types within TLVs 1 or 16.

- **E3:**

  This document defines the usage and processing of the Type-A, Type-C, and Type-D Segment sub-TLVs when they appear in TLV 21 (Reply Path TLV). If these sub-TLVs appear in TLVs 1 or 16, appropriate error codes MUST be returned as defined in [RFC8029].

**Evidence Summary:**

- (E1) Mandates return of 'appropriate error codes' when sub-TLVs 46–48 are present in TLVs 1 or 16.
- (E2) Highlights that RFC8029 does not define specific error procedures for such sub-TLVs.
- (E3) Reinforces the requirement from RFC9716 without providing further clarification.

**Fix Direction:**

Clarify the exact error handling by specifying the precise Return Code/Subcode and diagnostic behavior (e.g., inclusion of an Errored TLVs TLV) for cases where sub-TLVs 46–48 appear in TLVs 1 or 16.


**Severity:** Medium
  *Basis:* Ambiguity in error handling can result in inconsistent diagnostics and interoperability issues between different implementations.

**Confidence:** High

---

## Report 2: 9716-4-2

**Label:** Ambiguous precedence between SR Algorithm and optional SID in Type-C/Type-D segments

**Bug Type:** Underspecification

**Explanation:**

When both the A-Flag (which enables SR Algorithm label derivation) and an optional SID are present in a Type-C/Type-D segment, the specification gives conflicting instructions on which MPLS label to use.

**Justification:**

- Several expert analyses note that the specification mandates the use of the explicit SID if present, yet also requires deriving the label from the node address and SR Algorithm when the A-Flag is set.
- This ambiguity leaves implementations with divergent choices—whether to log mismatches, drop the packet, or compute different label stacks—thereby impacting interoperability.

**Evidence Snippets:**

- **E1:**

  When the A-Flag … is present, this specifies the SR Algorithm … The SR Algorithm is used by the receiver to derive the label.

- **E2:**

  When the SID field is present, it MUST be used for constructing the Reply Path.

- **E3:**

  If an optional MPLS SID is present in the Type-C/Type-D segments, the SID MUST be used to encode the echo reply with MPLS labels. If the MPLS SID does not match with the IPv4 or IPv6 address field in the Type-C or Type-D SID, log information should be generated.

**Evidence Summary:**

- (E1) Indicates that the SR Algorithm should be used to derive the label when the A-Flag is set.
- (E2) Mandates that if an optional SID is present, it must be used for reply construction.
- (E3) Points out that only a logging mechanism is specified for mismatches without resolving label precedence.

**Fix Direction:**

Specify explicit precedence rules—for example, state that when an optional SID is present, it unequivocally overrides any algorithm-derived label, or alternatively mandate a specific error response if a mismatch occurs.




**Severity:** High
  *Basis:* Divergent interpretations of label selection can lead to inconsistent MPLS label stacks and misrouted echo replies, which undermines the reliability of the OAM mechanism.

**Confidence:** High

---

## Report 4: 9716-4-4

**Label:** Inconsistent use of common Sub-TLV registry for types 46–48

**Bug Type:** Inconsistency

**Explanation:**

RFC 9716 allocates Segment sub-TLV types 46–48 from a common registry used for TLVs 1, 16, and 21, yet it forbids their use in TLVs 1 and 16, leading to a conflict with expectations established in related RFCs.

**Justification:**

- RFC 7110 and RFC 8287 define the common sub-TLV registry to apply uniformly across TLVs 1, 16, and 21, implying that entries from this registry are generally valid for all three.
- RFC 9716’s prohibition against using these sub-TLVs in TLVs 1 and 16 conflicts with this concept, potentially causing non-interoperable behavior and confusion in implementation.

**Evidence Snippets:**

- **E1:**

  RFC 9716 allocates its new Segment sub-TLVs 46, 47, and 48 from that same 'Sub-TLVs for TLV Types 1, 16, and 21' registry... but then states in Section 4 that 'the below types of Segment sub-TLVs apply to the Reply Path TLV' and ... 'If these sub-TLVs appear in TLVs 1 or 16, appropriate error codes MUST be returned as defined in [RFC8029].'

- **E2:**

  RFC 7110 and RFC 8287 define the sub-TLV registry to apply uniformly to TLVs 1, 16, and 21, implying that sub-TLVs allocated from this registry should be valid in all three TLVs.

**Evidence Summary:**

- (E1) Demonstrates that RFC 9716 prohibits the use of sub-TLVs 46–48 in TLVs 1 and 16 despite their allocation from the common registry.
- (E2) Indicates that other RFCs expect entries from the common registry to be valid across TLVs 1, 16, and 21.

**Fix Direction:**

Either adjust the registry allocation so that sub-TLVs 46–48 are exclusive to TLV 21, or revise the prohibition in RFC 9716 to allow their use in TLVs 1 and 16 consistently with RFC 7110 and RFC 8287.


**Severity:** Medium
  *Basis:* This inconsistency between registry use and prohibitive text can lead to divergent interpretations and non-interoperable implementations.

**Confidence:** High

---
