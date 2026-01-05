# Errata Reports

Total reports: 4

---


## Report 1: 9815-6-1

**Label:** Unnumbered-link bidirectional matching rule error in Section 6.3

**Bug Type:** Inconsistency

**Explanation:**

The SPF algorithm’s unnumbered link matching rule repeats the same identifier check, omitting the necessary verification of the Current-Link’s Local Identifier against the Remote-Link’s Remote Identifier.

**Justification:**

- Multiple experts noted that the rule is duplicated, leaving out the required cross-check.
- The duplicated clause forces implementations to rely on a single comparison which can cause mis‐association of parallel unnumbered links.

**Evidence Snippets:**

- **E1:**

  For unnumbered links to match during the IPv4 or IPv6 SPF computation, the Current-Link and Remote-Link's Address Family Link Descriptor TLV must match the address family of the IPv4 or IPv6 SPF computation, the Current-Link's Remote Identifier MUST match the Remote-Link's Local Identifier, and the Current-Link's Remote Identifier MUST match the Remote-Link's Local Identifier. Since the Link's Remote Identifier may not be known, a value of 0 is considered a wildcard and will match any Current or Remote Link's Local Identifier (see TLV 258 [RFC9552]).

**Evidence Summary:**

- (E1) The rule duplicates the condition 'Current-Link's Remote Identifier MUST match the Remote-Link's Local Identifier', neglecting to check the Current-Link's Local Identifier against the Remote-Link's Remote Identifier.

**Fix Direction:**

Replace the duplicated clause with a symmetric check: verify that the Current-Link’s Local Identifier matches the Remote-Link’s Remote Identifier as well as ensuring Current-Link’s Remote Identifier matches Remote-Link’s Local Identifier.


**Severity:** Medium
  *Basis:* An incorrect bidirectional match can lead to mis-pairing of unnumbered links and incorrect SPF topology computation, especially under misconfiguration or parallel link scenarios.

**Confidence:** High

---


## Report 6: 9815-6-6

**Label:** Undefined term 'Node-ID' in SPF NLRI recency determination

**Bug Type:** Inconsistency

**Explanation:**

The term 'Node-ID' is used in the context of NLRI recency determination without a clear definition, creating ambiguity over whether it refers to the full Node Descriptor set or just a subset such as the BGP Router-ID.

**Justification:**

- The specification instructs that recency is determined by examining the 'Node-ID' and sequence number, but does not define 'Node-ID'.
- This ambiguity can lead to inconsistent comparisons of self-originated NLRIs across implementations.

**Evidence Snippets:**

- **E1:**

  When BGP-LS-SPF NLRI is received, all that is required is to determine whether it is the most recent by examining the Node-ID and sequence number as described in Section 6.1.

- **E2:**

  Nodes, Links, or Prefix NLRIs with Node Descriptors matching the local BGP speaker are considered self-originated.

**Evidence Summary:**

- (E1) uses the term 'Node-ID' without definition, while (E2) refers to Node Descriptors.
- The lack of a clear definition may cause different interpretations in NLRI recency comparisons.

**Fix Direction:**

Replace 'Node-ID' with a clearly defined term, such as 'Node Descriptors', or provide an explicit definition of 'Node-ID' in the document.


**Severity:** Medium
  *Basis:* Ambiguity in key identifiers can lead to mismatches in NLRI selection and misidentification of self-originated routes.

**Confidence:** High

---


## Report 7: 9815-6-7

**Label:** Incorrect use of 'Node Descriptors' instead of 'Link Descriptors' for link NLRI fields

**Bug Type:** Inconsistency

**Explanation:**

The document incorrectly refers to link-specific TLVs (e.g., TLV 259–262) as 'Node Descriptors' rather than using the proper term 'Link Descriptors', which may mislead implementers about NLRI structure.

**Justification:**

- Section 5.2.2 describes the advertisement of link NLRI with unique Local and Remote Node Descriptors, but the TLVs mentioned are defined as Link Descriptor TLVs in RFC9552.
- This misuse of terminology can confuse the distinction between node identity and link identity.

**Evidence Snippets:**

- **E1:**

  Link NLRI is advertised with unique Local and Remote Node Descriptors dependent on the IP addressing. For IPv4 links, the link's local IPv4 interface address (TLV 259) and remote IPv4 neighbor address (TLV 260) are used. For IPv6 links, the local IPv6 interface address (TLV 261) and remote IPv6 neighbor address (TLV 262) are used.

**Evidence Summary:**

- (E1) incorrectly labels TLVs that are actually defined for link descriptors as 'Node Descriptors', causing potential confusion.

**Fix Direction:**

Change references from 'Node Descriptors' to 'Link Descriptors' in the context of link NLRI advertisement.


**Severity:** Medium
  *Basis:* This terminology error can lead to incorrect encoding or interpretation of NLRI contents in BGP-LS-SPF implementations.

**Confidence:** High

---


## Report 8: 9815-6-8

**Label:** Inconsistent naming of prefix NLRIs: 'BGP-LS-Prefix NLRI' vs 'BGP-LS-SPF Prefix NLRI'

**Bug Type:** Inconsistency

**Explanation:**

The specification inconsistently refers to prefix NLRIs using both 'BGP-LS-Prefix NLRI' and 'BGP-LS-SPF Prefix NLRI', which may confuse implementers about the intended use and scope of these NLRIs.

**Justification:**

- One section describes the prefix NLRI advertised with SPF Status using the term 'BGP-LS-Prefix NLRI', while another mandates that updates must be for 'BGP-LS-SPF Prefix NLRI'.
- Such inconsistent naming can lead to misinterpretation regarding the applicability of SPF-specific behaviors.

**Evidence Snippets:**

- **E1:**

  the BGP-LS-Prefix NLRI is advertised with SPF Status indicating the prefix is unreachable prior to withdrawal.

- **E2:**

  the BGP-LS-SPF Prefix NLRI MUST advertise a more recent version of the BGP-LS-SPF Prefix NLRI without the SPF Status TLV...

**Evidence Summary:**

- (E1) and (E2) show conflicting usage of the naming conventions for prefix NLRIs.
- The inconsistency may result in ambiguity over which NLRIs the SPF procedures apply to.

**Fix Direction:**

Standardize all references to use 'BGP-LS-SPF Prefix NLRI' consistently throughout the document.


**Severity:** Low
  *Basis:* The naming inconsistency is primarily cosmetic but could lead to minor implementation confusion.

**Confidence:** High

---