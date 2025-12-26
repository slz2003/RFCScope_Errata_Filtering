# Errata Reports

Total reports: 2

---

## Report 1: 9805-4-1

**Label:** Ambiguous classification between 'future versions' and new protocols for RAO use

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly define the criteria for distinguishing a 'future version' of an existing RAO‐using protocol from an entirely new protocol, which could lead to inconsistent application of RAO usage policies.

**Justification:**

- The normative text permits continued RAO use for existing protocols in their 'future versions' while prohibiting new protocols, but it fails to specify how to decide which category a protocol belongs to.
- Multiple expert analyses (Scope, Deontic, Terminology, and Boundary) highlight this as an underspecified boundary that may affect future protocol revisions such as MLDv3 or RSVPv3.

**Evidence Snippets:**

- **E1:**

  Protocols that use the IPv6 Router Alert option MAY continue to do so, even in future versions. However, new protocols that are standardized in the future MUST NOT use the IPv6 Router Alert option. Appendix A contains an exhaustive list of protocols that MAY continue to use the IPv6 Router Alert option. (RFC 9805, Section 4)

- **E2:**

  Table 1 contains an exhaustive list of protocols that use the IPv6 Router Alert option. (RFC 9805, Appendix A)

- **E3:**

  It is left for future work to develop new versions of MLDv2 and MRD that do not rely on the IPv6 Router Alert option. That task is out of scope for this document. (RFC 9805, Section 5)

**Evidence Summary:**

- (E1) Presents the dual normative statements regarding RAO use for 'future versions' versus new protocols.
- (E2) Lists the protocols in Appendix A without clarifying versioning criteria.
- (E3) Acknowledges future work on protocol revisions without providing normative classification rules.

**Fix Direction:**

Clarify in Section 4 the criteria that determine when a protocol revision is considered a 'future version' of an existing protocol versus when it qualifies as a 'new protocol', for example by linking the classification to protocol identity or continuity markers.


**Severity:** Medium
  *Basis:* The lack of clear criteria creates a normative ambiguity that may lead to inconsistent decisions in future standardization efforts, impacting security policy enforcement.

**Confidence:** High

---

## Report 2: 9805-4-2

**Label:** MPLS Ping entry inconsistent: Global permission versus deprecation note

**Bug Type:** Inconsistency

**Explanation:**

The MPLS Ping entry in Appendix A includes a note that its use of the IPv6 Router Alert option is deprecated, which conflicts with the global statement that all protocols in Appendix A may continue to use the option.

**Justification:**

- Section 4 asserts that 'Appendix A contains an exhaustive list of protocols that MAY continue to use the IPv6 Router Alert option', suggesting a blanket permission.
- The MPLS Ping row, however, specifically notes '(Use of the IPv6 Router Alert option is deprecated)', creating an internal textual conflict about its status.

**Evidence Snippets:**

- **E1:**

  Appendix A contains an exhaustive list of protocols that MAY continue to use the IPv6 Router Alert option. (Section 4)

- **E2:**

  Table 1: Protocols That Use the IPv6 Router Alert Option (Appendix A table heading)

- **E3:**

  MPLS Ping (Use of the IPv6 Router Alert option is deprecated) (Appendix A, Table 1, MPLS Ping row)

**Evidence Summary:**

- (E1) Establishes the global permission for protocols in Appendix A to continue using the RAO.
- (E2) Provides the context of the list as defined in Appendix A.
- (E3) Shows the specific deprecation note for MPLS Ping, which contradicts the global statement.

**Fix Direction:**

Revise either the global statement in Section 4 or the MPLS Ping entry in Appendix A to consistently reflect the intended status regarding RAO use.


**Severity:** Low
  *Basis:* Although the inconsistency is textual and may not affect current implementation, it can cause confusion among protocol designers regarding the intended deprecation status.

**Confidence:** High

---

