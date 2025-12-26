# Errata Reports

Total reports: 6

---

## Report 1: 9815-5-1

**Label:** Ambiguous Handling of Missing Sequence Number TLV vs Attribute Discard

**Bug Type:** Both

**Explanation:**

There is ambiguity in how to handle BGP‐LS‐SPF NLRIs that lack the mandatory Sequence Number TLV, since its absence may be due either to an omitted TLV within a present BGP‐LS Attribute (which should be treated as malformed and withdrawn) or to the entire attribute being discarded (in which case the NLRI should be preserved but not used in SPF).

**Justification:**

- Section 5.2.4 says that if the Sequence Number TLV is not received, the NLRI is malformed and must be handled as described in Section 7.1, while Section 7.1 directs that NLRIs without any BGP‐LS Attribute (an attribute‐discard case) be preserved and propagated.
- Multiple experts (Temporal, Scope, Quantitative, Deontic, Structural, CrossRFC, and Boundary) noted that the overlap in these rules creates a normative tension leading to potentially divergent implementations.

**Evidence Snippets:**

- **E1:**

  “If the Sequence Number TLV is not received, then the corresponding NLRI is considered as malformed and MUST be handled as described in Section 7.1.” (Section 5.2.4)

- **E2:**

  “When a BGP speaker receives a BGP Update that does not contain any BGP-LS Attributes, it is most likely an indication of 'Attribute Discard' fault handling, and the BGP speaker SHOULD preserve and propagate the BGP-LS-SPF NLRI… However, NLRIs without the BGP-LS Attribute MUST NOT be used in the SPF calculation (Section 7.1).” (Section 7.1)

**Evidence Summary:**

- (E1) mandates treating missing Sequence Number TLV as a malformed NLRI,
- (E2) instructs preservation and propagation when no BGP-LS Attribute is present.

**Fix Direction:**

Clarify that the malformed NLRI rule in Section 5.2.4 applies only when a BGP-LS Attribute is present but missing a valid Sequence Number TLV, while NLRIs lacking the BGP-LS Attribute due to attribute discard should be preserved and propagated as specified in Section 7.1.


**Severity:** Medium
  *Basis:* The ambiguity can lead to divergent LSDB contents and convergence behavior across implementations.

**Confidence:** High

---

## Report 2: 9815-5-2

**Label:** Incorrect Unnumbered-Link Matching Rule for SPF Bidirectional Connectivity (Missing symmetric check)

**Bug Type:** Inconsistency

**Explanation:**

The SPF algorithm’s matching rule for unnumbered links erroneously repeats the condition on the Remote Identifier without requiring a corresponding check on the Local Identifier, potentially causing incorrect pairing of half-links.

**Justification:**

- The specification requires that the Current-Link's Remote Identifier must match the Remote-Link's Local Identifier twice, omitting the intended symmetric condition that the Current-Link's Local Identifier must equal the Remote-Link's Remote Identifier.
- This error, noted by several experts, can lead to mispaired unnumbered links and incorrect SPF graph construction.

**Evidence Snippets:**

- **E1:**

  “For unnumbered links to match during the IPv4 or IPv6 SPF computation, the Current-Link and Remote-Link's Address Family Link Descriptor TLV must match the address family of the IPv4 or IPv6 SPF computation, the Current-Link's Remote Identifier MUST match the Remote-Link's Local Identifier, and the Current-Link's Remote Identifier MUST match the Remote-Link's Local Identifier.” (Section 6.3, Step 5)

- **E2:**

  The half‐link model (as described in RFC 9552) implies that to fully match, one half‐link’s Local Identifier should equal the other half‐link’s Remote Identifier.

**Evidence Summary:**

- (E1) shows the duplicated condition on Remote Identifier,
- (E2) indicates the intended symmetric matching between Local and Remote Identifiers.

**Fix Direction:**

Replace the second, duplicated condition with 'the Current-Link's Local Identifier MUST match the Remote-Link's Remote Identifier' to enforce the required symmetric check.


**Severity:** High
  *Basis:* Incorrect matching can result in erroneous SPF paths and potential routing miscalculations.

**Confidence:** High

---

## Report 3: 9815-5-3

**Label:** Mis-scoped Requirement for Node and Link NLRI Descriptors Carrying TLVs 512 and 516

**Bug Type:** Inconsistency

**Explanation:**

The specification incorrectly mandates that both Local and Remote Node Descriptors for all NLRI types carry TLVs 512 and 516, even though Node and Prefix NLRIs structurally include only Local Node Descriptors.

**Justification:**

- Section 5.2 requires that 'The Local and Remote Node Descriptors for all NLRI MUST include the BGP Router-ID (TLV 516) and the Autonomous System (TLV 512)', which is impossible for Node and Prefix NLRIs that lack Remote Node Descriptors.
- This mis-scoping leads to potential misinterpretation of NLRI format and capability mismatches with RFC 9552.

**Evidence Snippets:**

- **E1:**

  “For Node NLRI and Link NLRI, the specified Protocol-ID MUST be the direct protocol (4)… The Local and Remote Node Descriptors for all NLRI MUST include the BGP Router-ID (TLV 516) and the Autonomous System (TLV 512) number.” (Section 5.2)

- **E2:**

  In RFC 9552, Node and Prefix NLRIs contain only Local Node Descriptors; only Link NLRIs include both Local and Remote Node Descriptors.

**Evidence Summary:**

- (E1) states the universal requirement for TLVs 512 and 516,
- (E2) clarifies that only Link NLRIs can carry Remote Node Descriptors.

**Fix Direction:**

Revise the requirement so that only the applicable descriptor types are mandated: require Local Node Descriptors in Node and Prefix NLRIs and both Local and Remote Node Descriptors in Link NLRIs.


**Severity:** Medium
  *Basis:* The mis-scoping may cause misimplementation of NLRI structures but does not directly affect SPF decision logic.

**Confidence:** High

---

## Report 4: 9815-5-4

**Label:** Missing Handling Rule for Absent Prefix Metric TLV 1155 in SPF Computation

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that Prefix NLRIs must include the Prefix Metric TLV 1155 to be considered for SPF route calculation, yet it fails to specify how to handle prefixes that do not advertise this TLV.

**Justification:**

- Section 5.2.3 declares that the Prefix Metric (TLV 1155) MUST be advertised to be considered for route calculation.
- The SPF algorithm in Section 6.3 unconditionally uses the Prefix Metric without providing an alternative behavior for NLRIs lacking TLV 1155.

**Evidence Snippets:**

- **E1:**

  “The Prefix Metric (TLV 1155) MUST be advertised to be considered for route calculation.” (Section 5.2.3)

- **E2:**

  Section 6.3’s SPF algorithm uses the Prefix Metric without any branch or error handling for its absence.

**Evidence Summary:**

- (E1) mandates the advertisement of TLV 1155,
- (E2) reveals that the SPF algorithm lacks handling for missing Prefix Metric TLV.

**Fix Direction:**

Specify that if a Prefix NLRI does not carry TLV 1155, then it should either be treated as malformed (and withdrawn) or be explicitly skipped in SPF computation.


**Severity:** Medium
  *Basis:* Ambiguity in prefix handling may result in inconsistent route calculations across implementations.

**Confidence:** High

---

## Report 5: 9815-5-5

**Label:** Ambiguous On-Wire Length for IGP Metric TLV 1095 in BGP-LS-SPF

**Bug Type:** Underspecification

**Explanation:**

Although the specification notes that the BGP SPF metric length is 4 octets, it reuses IGP Metric TLV 1095—which is defined in RFC 9552 as variable-length—without explicitly mandating a 4-octet encoding for BGP-LS-SPF.

**Justification:**

- Section 5.2.2 indicates that the IGP Metric (TLV 1095) MUST be advertised and asserts that the BGP SPF metric length is 4 octets.
- RFC 9552 defines TLV 1095 as having a variable length (1, 2, or 3 octets) dependent on the underlying IGP, leading to potential interoperability issues if not clearly redefined for BGP-LS-SPF.

**Evidence Snippets:**

- **E1:**

  “The IGP Metric (TLV 1095) MUST be advertised. If a BGP speaker receives a Link NLRI without an IGP Metric attribute TLV, then it MUST consider the received NLRI as malformed and is handled as described in Section 7.1. The BGP SPF metric length is 4 octets.” (Section 5.2.2)

- **E2:**

  RFC 9552 defines TLV 1095 as having a variable length, with IS-IS small metrics encoded in 1 octet, OSPF link metrics in 2 octets, and IS-IS wide metrics in 3 octets.

**Evidence Summary:**

- (E1) asserts a 4-octet SPF metric length for TLV 1095,
- (E2) contrasts this with the variable-length encoding defined in RFC 9552.

**Fix Direction:**

Explicitly require that in the context of BGP-LS-SPF, TLV 1095 MUST have a Length of 4 octets; any TLV 1095 with a non-4-octet length must be treated as malformed according to Section 7.1.


**Severity:** Medium
  *Basis:* Unclear TLV encoding could lead to inconsistent metric values and consequently divergent SPF calculations.

**Confidence:** High

---

## Report 6: 9815-5-6

**Label:** Undefined Handling for Malformed or Multiple Sequence Number TLVs in BGP-LS-SPF

**Bug Type:** Underspecification

**Explanation:**

While a single, correct 8-octet Sequence Number TLV is mandatory for BGP-LS-SPF, the specification does not clarify how to handle cases where the TLV is malformed (e.g., incorrect length) or where multiple instances are present.

**Justification:**

- Section 5.2.4 specifies that the Sequence Number TLV must contain an 8-octet sequence number, yet no guidance is provided for error handling if the TLV is malformed or duplicated.
- The absence of explicit error-handling instructions may result in different implementations making incompatible choices when processing such NLRIs.

**Evidence Snippets:**

- **E1:**

  “The BGP-LS Attribute Sequence Number TLV contains an 8-octet sequence number.” (Section 5.2.4)

- **E2:**

  Section 7.1 does not mention TLV 1181 explicitly, leaving receiver behavior for malformed or multiple Sequence Number TLVs undefined.

**Evidence Summary:**

- (E1) establishes the requirement for an 8-octet Sequence Number,
- (E2) reveals that error handling for malformed or duplicate Sequence Number TLVs is not specified.

**Fix Direction:**

Specify that exactly one Sequence Number TLV with a Length of 8 octets must be present per BGP-LS-SPF NLRI; if this condition is not met, the NLRI MUST be considered malformed and treated as withdrawn.


**Severity:** Medium
  *Basis:* Lack of defined error-handling may lead to divergent interpretations of NLRI recency and inconsistent SPF database contents.

**Confidence:** High

---
