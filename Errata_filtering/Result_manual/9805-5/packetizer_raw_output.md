# Errata Reports

Total reports: 3

---

## Report 1: 9805-5-1

**Label:** Ambiguity in New Protocol vs Future Version Classification for Router Alert Usage

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly define the boundary between 'new protocols' and 'future versions', making it ambiguous whether a successor to MLDv2 or MRD should be allowed to continue using the IPv6 Router Alert option.

**Justification:**

- Multiple sections provide conflicting guidance: Section 4 permits continued use in 'future versions' while prohibiting use in 'new protocols'.
- Appendix A is declared exhaustive, yet Section 5 encourages developing new versions without Router Alert, leaving the classification criteria unclear.

**Evidence Snippets:**

- **E1:**

  Protocols that use the IPv6 Router Alert option MAY continue to do so, even in future versions. However, new protocols that are standardized in the future MUST NOT use the IPv6 Router Alert option. Appendix A contains an exhaustive list of protocols that MAY continue to use the IPv6 Router Alert option.

- **E2:**

  The only protocols in Appendix A that have widespread deployment are Multicast Listener Discovery Version 2 (MLDv2) [RFC9777] and Multicast Router Discovery (MRD) [RFC4286]. … It is left for future work to develop new versions of MLDv2 and MRD that do not rely on the IPv6 Router Alert option. That task is out of scope for this document.

- **E3:**

  All MLDv2 messages described in this document MUST be sent with … an IPv6 Router Alert option and All MRD messages are sent with … Hop Limit of 1 and contain the Router Alert Option [4] [5].

**Evidence Summary:**

- (E1) Shows conflicting normative language regarding continued use versus prohibition for new protocols.
- (E2) Highlights the non-normative future work note for MLDv2 and MRD, adding to the ambiguity.
- (E3) Demonstrates that existing protocols still mandate the use of Router Alert, underscoring the classification dilemma.

**Fix Direction:**

Clarify the criteria by explicitly defining what constitutes a 'new protocol' versus a 'future version' and explain how each category should handle Router Alert usage.


**Severity:** Medium
  *Basis:* Ambiguous classification can lead to inconsistent interpretations and divergent implementations in future revisions, potentially affecting interoperability and security.

**Confidence:** High

---

## Report 2: 9805-5-2

**Label:** Overstated Security Mitigation Claim in RFC 9805 Section 6

**Bug Type:** Inconsistency

**Explanation:**

RFC 9805 claims to mitigate all security considerations related to the IPv6 Router Alert option while still permitting existing protocols to mandate its use, creating a misleading impression about the overall security posture.

**Justification:**

- Section 6 asserts that all security considerations are mitigated, yet the document allows continued mandatory use of Router Alert in widely deployed protocols like MLDv2 and MRD.
- This mismatch can cause operators to underestimate residual risks, as the attack surfaces associated with these protocols remain unchanged.

**Evidence Snippets:**

- **E1:**

  This document mitigates all security considerations associated with the IPv6 Router Alert option. These security considerations can be found in [RFC2711], [RFC6192], and [RFC6398].

- **E2:**

  Protocols that use the IPv6 Router Alert option MAY continue to do so, even in future versions. However, new protocols that are standardized in the future MUST NOT use the IPv6 Router Alert option. Appendix A contains an exhaustive list of protocols that MAY continue to use the IPv6 Router Alert option.

- **E3:**

  All MLDv2 messages described in this document MUST be sent with … an IPv6 Router Alert option [RFC2711] in a Hop-by-Hop Options header. All MRD messages are sent with … Hop Limit of 1 and contain the Router Alert Option [4] [5].

**Evidence Summary:**

- (E1) Indicates the blanket security mitigation claim made in Section 6.
- (E2) Confirms that existing protocols are still permitted to use Router Alert.
- (E3) Provides evidence of mandatory Router Alert usage in protocols like MLDv2 and MRD, highlighting the inconsistency.

**Fix Direction:**

Revise Section 6 to restrict the security mitigation claim to new protocols and clarify that existing RA-dependent protocols continue to retain their inherent security risks.


**Severity:** Medium
  *Basis:* The overly broad security statement may lead to misinterpretation by implementers and network operators, resulting in insufficient mitigation measures for known vulnerabilities.

**Confidence:** High

---

## Report 3: 9805-5-3

**Label:** Residual Uncertainty in Pre-RFC 9805 IANA Router Alert Registry Details

**Bug Type:** Underspecification

**Explanation:**

The document omits details of the pre-RFC 9805 IPv6 Router Alert Option Values registry, creating uncertainty over how experimental values are treated for protocols listed in Appendix A.

**Justification:**

- The text notes that the exact content of the registry is not included, which may affect protocols that previously depended on experimental Router Alert values now marked as 'Reserved'.
- Additional clarifying text in RFC 9805 or the IANA registry could resolve potential scope interactions and prevent future misinterpretation.

**Evidence Snippets:**

- **E1:**

  The exact content of the “IPv6 Router Alert Option Values” registry prior to RFC 9805 is not included here. If any Appendix A protocols depended on the “experimental” Router Alert values that were converted to “Reserved”, there could be additional subtle scope interactions between the IANA actions and the claim that those protocols “MAY continue to use” Router Alert. However, from the given text, it appears the intent is to forbid *new* experiments rather than to invalidate existing assignments, and clarifying text in the registry or an additional note in RFC 9805 might be needed to remove any doubt.

**Evidence Summary:**

- (E1) Highlights the omission of detailed pre-RFC 9805 registry content and the resulting uncertainty regarding the treatment of experimental Router Alert values.

**Fix Direction:**

Add clarifying details or an explanatory note in RFC 9805 to specify how experimental Router Alert values are handled in the context of the IANA registry.


**Severity:** Low
  *Basis:* While not directly affecting current protocol operation, the lack of clarity could lead to confusion in future interpretations of the registry status and its implications.

**Confidence:** Medium

---
