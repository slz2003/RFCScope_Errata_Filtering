# Errata Reports

Total reports: 10

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

## Report 2: 9815-6-2

**Label:** Ambiguous NLRI selection rule ordering affecting convergence

**Bug Type:** Underspecification

**Explanation:**

The document does not explicitly state that the NLRI selection rules, particularly preferring self-originated NLRIs over those with higher sequence numbers, must be applied in a strict order, creating potential divergence in convergence behavior.

**Justification:**

- Experts highlighted that the intended precedence between rule #1 (self-originated preference) and rule #3 (highest sequence number) is not clearly mandated.
- This ambiguity could result in some routers erroneously retaining stale NLRIs after a restart.

**Evidence Snippets:**

- **E1:**

  1.  NLRIs self‑originated from directly connected BGP SPF peers are preferred… This rule assures that a stale NLRI is updated even if a BGP SPF router loses its sequence number state due to a cold start.

- **E2:**

  3.  The NLRI with the most recent Sequence Number TLV, i.e., the highest sequence number is selected.

**Evidence Summary:**

- (E1) Indicates a preference for self-originated NLRIs, while (E2) uses highest sequence number, but the relative ordering is not explicitly defined.
- (E1, E2) The lack of a strict tie-break order may lead to different implementations resolving conflicts inconsistently.

**Fix Direction:**

Add explicit language stating that the NLRI selection rules are to be applied in the listed order, with rule (1) (self-originated) taking precedence over rule (3) (highest sequence number).


**Severity:** High
  *Basis:* Incorrect rule ordering can force routers to keep stale routes, undermining the convergence guarantee after sequence number resets.

**Confidence:** High

---

## Report 3: 9815-6-3

**Label:** Ambiguous handling of prefix NLRI missing Prefix Metric TLV (TLV 1155)

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define whether a Prefix NLRI lacking the mandatory Prefix Metric TLV should be treated as malformed, preserved but ignored for SPF, or assigned a default metric, which could lead to divergent implementation behavior.

**Justification:**

- The text mandates that the Prefix Metric MUST be advertised for route calculation, yet references to RFC 9552 suggest its absence implies no reachability.
- Experts worry that the lack of explicit error-handling could cause inconsistencies in SPF computation.

**Evidence Snippets:**

- **E1:**

  The Prefix Metric (TLV 1155) MUST be advertised to be considered for route calculation.

- **E2:**

  The cost for each prefix is the metric advertised in the Prefix Attribute Prefix Metric (TLV 1155) added to the cost to reach the Current-Node.

**Evidence Summary:**

- (E1) Specifies that TLV 1155 is essential for route calculation, while (E2) shows that SPF calculation depends on its value.
- (E1, E2) The document does not specify the behavior if TLV 1155 is missing, leading to ambiguity.

**Fix Direction:**

Clarify in Section 5.2.3 or 7.1 that a Prefix NLRI missing the Prefix Metric TLV must either be treated as malformed (triggering a withdrawal) or preserved solely for LS propagation while being excluded from SPF calculations.


**Severity:** Low
  *Basis:* While divergent treatments do not affect basic route installation, they can result in inconsistent SPF topology views across different implementations.

**Confidence:** High

---

## Report 4: 9815-6-4

**Label:** Inconsistent scope definition for the SPF-based Decision Process

**Bug Type:** Both

**Explanation:**

The document provides conflicting statements about whether the SPF-based Decision Process applies solely to the BGP-LS-SPF SAFI or more broadly to IPv4/IPv6 unicast, leading to potential misimplementation of routing behavior.

**Justification:**

- The Introduction suggests the modifications apply to IPv4/IPv6 as underlay unicast SAFIs, while Section 5.1 restricts the process to the BGP-LS-SPF SAFI.
- Section 6 unconditionally states that the SPF-based Decision Process replaces the standard BGP process, which may mislead some implementers.

**Evidence Snippets:**

- **E1:**

  The modifications to [RFC4271] for BGP SPF described herein only apply to IPv4 and IPv6 as underlay unicast Subsequent Address Family Identifiers (SAFIs).

- **E2:**

  The SPF-based Decision Process (Section 6) applies only to the BGP-LS-SPF SAFI and MUST NOT be used with other combinations of the BGP-LS AFI (16388).

- **E3:**

  The SPF-based Decision Process replaces the BGP Decision Process described in [RFC4271].

**Evidence Summary:**

- (E1) implies a scope limited to certain unicast SAFIs, while (E2) confines SPF to BGP-LS-SPF.
- (E3) suggests a global replacement of the BGP process, creating a conflict.

**Fix Direction:**

Clarify that the SPF-based Decision Process strictly applies only to BGP-LS-SPF NLRI and does not affect the decision process for standard IPv4/IPv6 unicast SAFIs.


**Severity:** High
  *Basis:* Misinterpreting the scope could lead to applying SPF semantics to inappropriate SAFIs, causing severe routing anomalies.

**Confidence:** High

---

## Report 5: 9815-6-5

**Label:** Inconsistent mandatory TLV requirements versus attr-discard handling

**Bug Type:** Both

**Explanation:**

There is a conflict between the requirement for mandatory TLVs (such as the Sequence Number and IGP Metric) for BGP-LS-SPF NLRIs and the recommendation to preserve and propagate NLRI lacking the BGP-LS Attribute, leading to divergent error-handling behaviors.

**Justification:**

- Section 5.2.2 and 5.2.4 mandate that certain TLVs must be present, otherwise the NLRI is considered malformed.
- Section 7.1 instructs that NLRI without the BGP-LS Attribute, which would lack these TLVs, must not be used in SPF but should be preserved for propagation.

**Evidence Snippets:**

- **E1:**

  The IGP Metric (TLV 1095) MUST be advertised. If a BGP speaker receives a Link NLRI without an IGP Metric attribute TLV, then it MUST consider the received NLRI as malformed and is handled as described in Section 7.1.

- **E2:**

  When a BGP speaker receives a BGP Update that does not contain any BGP-LS Attributes, it is most likely an indication of ‘Attribute Discard’ fault handling, and the BGP speaker SHOULD preserve and propagate the BGP-LS-SPF NLRI as described in Section 8.2.2 of [RFC9552].

**Evidence Summary:**

- (E1) enforces mandatory presence of TLVs for normal operation, while (E2) calls for preservation of NLRI even if the attribute is missing.
- (E1, E2) This discrepancy creates ambiguity on how to treat NLRI resulting from attribute discard.

**Fix Direction:**

Revise the language to specify that TLVs are mandatory only for NLRIs that participate in SPF computation, and that attr-discarded NLRI may be preserved without these TLVs.


**Severity:** Medium
  *Basis:* Divergent interpretations may lead to inconsistencies in LSDB contents and SPF calculation, affecting interoperability.

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

## Report 9: 9815-6-9

**Label:** Ambiguous terminology for 'Local-RIB' and 'GLOBAL-RIB' conflicting with RFC 4271

**Bug Type:** Inconsistency

**Explanation:**

The document introduces the terms 'Local-RIB' and 'GLOBAL-RIB' in a manner that conflicts with the standard RFC 4271 terminology, potentially causing confusion about the roles of these routing tables.

**Justification:**

- The specification redefines these terms in the context of SPF computation without clearly distinguishing them from RFC 4271’s Loc-RIB and RIB.
- This overlap in terminology can lead to misinterpretation of routing table interactions within BGP implementations.

**Evidence Snippets:**

- **E1:**

  Local Route Information Base (Local-RIB): A routing table that contains reachability information ... GLOBAL-RIB: The RIB containing the current routes that are installed in the router's forwarding plane.

**Evidence Summary:**

- (E1) demonstrates that the document’s usage of 'Local-RIB' and 'GLOBAL-RIB' may be confused with the standard terms defined in RFC 4271.

**Fix Direction:**

Include a clarifying note to distinguish the algorithm-specific 'Local-RIB' and 'GLOBAL-RIB' from the standard Loc-RIB and RIB as defined in RFC 4271.


**Severity:** Low
  *Basis:* This issue is mainly a clarity concern and is unlikely to cause major interoperability problems if addressed.

**Confidence:** High

---

## Report 10: 9815-6-10

**Label:** Undefined handling for self-originated NLRI with lower sequence number

**Bug Type:** Underspecification

**Explanation:**

The specification does not define the behavior when a self-originated NLRI is received with a sequence number lower than the local copy, leaving a gap in error-handling for this common boundary condition.

**Justification:**

- Boundary Expert noted that while special processing is described for received self-originated NLRIs with higher sequence numbers or attribute differences, the case where the sequence number is lower is not addressed.
- This undefined behavior can result in divergent implementations regarding stale NLRI treatment.

**Evidence Snippets:**

- **E1:**

  No rule is given for the case where the received self-originated NLRI has a smaller sequence number than the local node's sequence number (or where seq is smaller and attributes differ).

**Evidence Summary:**

- (E1) highlights the absence of defined behavior for self-originated NLRIs with lower sequence numbers, which is a common boundary condition.

**Fix Direction:**

Define explicit behavior for self-originated NLRI reception when the sequence number is lower than the local value, such as treating the update as stale and ignoring it.




**Severity:** Medium
  *Basis:* Ambiguity in this case may lead to inconsistencies in LSDB state and slow or erratic convergence in the network.

**Confidence:** High
---
