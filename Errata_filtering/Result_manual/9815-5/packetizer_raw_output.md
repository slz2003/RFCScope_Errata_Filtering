# Errata Reports

Total reports: 1

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