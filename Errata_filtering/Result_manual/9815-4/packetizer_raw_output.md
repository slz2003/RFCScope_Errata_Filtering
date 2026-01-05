# Errata Reports

Total reports: 1

---


## Report 1: 9815-4-1

**Label:** Ambiguous BGP-LS vs BGP-LS-SPF terminology in Section 4.3

**Bug Type:** Ambiguity/Inconsistency

**Explanation:**

Section 4.3 uses inconsistent terminology by mixing clear references to BGP-LS-SPF (e.g. 'BGP-LS-SPF Link NLRI') with ambiguous terms (e.g. 'BGP-LS peers' and 'BGP-LS Link NLRI'), which may lead implementers to misinterpret which SAFI is intended.

**Justification:**

- The CrossRFC analysis notes that while the section initially ties NLRI advertisement to BGP-LS-SPF, later references to 'BGP-LS peers' create ambiguity about whether the legacy or the SPF SAFI is intended (E1, E2).
- The Terminology analysis reinforces that this switch in terminology could cause readers to erroneously design dual-SAFI deployments, leading to potential interoperability issues (E1, E2).

**Evidence Snippets:**

- **E1:**

  BGP-LS-SPF Link NLRI is advertised as long as the corresponding link is considered up and available as per the chosen liveness detection mechanism (thus, the BFD protocol [RFC5880] is RECOMMENDED).

- **E2:**

  The controller may use constraints to determine when to advertise BGP-LS-SPF NLRI for BGP-LS peers. For example, a controller may delay advertisement of a link between two peers the until the EoR marker Section 5.3 has been received from both BGP peers and the BGP-LS Link NLRI for the link(s) between the two nodes has been received from both BGP peers.

**Evidence Summary:**

- (E1) Section 4.3 first states that BGP-LS-SPF Link NLRI is advertised based on link availability.
- (E2) Later in the same section, ambiguous terms 'BGP-LS peers' and 'BGP-LS Link NLRI' are used, creating confusion over the intended SAFI.

**Fix Direction:**

In Section 4.3, revise the text to explicitly refer to BGP-LS-SPF peers and the BGP-LS-SPF Link NLRI, ensuring consistency and clear differentiation from the legacy BGP-LS SAFI.


**Severity:** Medium
  *Basis:* The ambiguity can result in divergent interpretations and implementation choices, potentially affecting convergence, controller behavior, and interoperability.

**Confidence:** Medium

---