# Errata Reports

Total reports: 1

---

## Report 1: 9815-1-1

**Label:** Ambiguous RFC 4271 Modification Scope: IPv4/IPv6 Underlay Unicast vs. BGP‑LS‑SPF SAFI

**Bug Type:** Ambiguity/Underspecification

**Explanation:**

The document ambiguously defines the scope of the RFC 4271 modifications by stating in Section 1 that they apply to IPv4/IPv6 underlay unicast SAFIs, while later sections explicitly restrict the SPF‐based Decision Process to the BGP‑LS‑SPF SAFI (AFI 16388/SAFI 80).

**Justification:**

- Section 1’s text indicates that the modifications apply to IPv4 and IPv6 as underlay unicast SAFIs (E1), which could be misinterpreted as affecting the classic BGP unicast address families.
- Sections 5.1 and 6.4 clarify that the SPF‐based Decision Process is to be used exclusively with the BGP‑LS‑SPF SAFI, creating an inconsistency in the overall scope (E2, E3).

**Evidence Snippets:**

- **E1:**

  Section 1: “The modifications to [RFC4271] for BGP SPF described herein only apply to IPv4 and IPv6 as underlay unicast Subsequent Address Family Identifiers (SAFIs). Operations for any other BGP SAFIs are outside the scope of this document.”

- **E2:**

  Section 5.1: “This document introduces the BGP‑LS‑SPF SAFI with a value of 80. The SPF‑based Decision Process (Section 6) applies only to the BGP‑LS‑SPF SAFI and MUST NOT be used with other combinations of the BGP‑LS AFI (16388). … The BGP‑LS‑SPF SAFI is used to advertise IPv4 and IPv6 prefix information in a format facilitating an SPF‑based Decision Process.”

- **E3:**

  Section 6.4: “While the BGP‑LS‑SPF address family and the BGP unicast address families may install routes into the routing tables of the same device, they operate independently (i.e., ‘ships‑in‑the‑night’ mode). There is no implicit route redistribution between the BGP‑LS‑SPF address family and the BGP unicast address families. … [T]he BGP‑LS‑SPF SAFI is considered an underlay SAFI.”

**Evidence Summary:**

- (E1) Section 1 states that modifications apply to IPv4/IPv6 underlay unicast SAFIs.
- (E2) Section 5.1 specifies that the SPF‐based Decision Process applies only to the BGP‑LS‑SPF SAFI (AFI 16388/SAFI 80).
- (E3) Section 6.4 reinforces the separation by noting independent operation between BGP‑LS‑SPF and the standard unicast address families.

**Fix Direction:**

Revise Section 1 (and Section 9, if applicable) to explicitly state that the modifications to RFC 4271 apply solely to the BGP‑LS‑SPF SAFI (AFI 16388/SAFI 80) used for underlay topology and prefix information, and clarify that the standard IPv4/IPv6 unicast SAFIs remain unaffected.


**Severity:** Medium
  *Basis:* Misinterpretation of the document’s scope could lead implementers to erroneously apply the SPF-based Decision Process to traditional IPv4/IPv6 unicast routing, potentially causing significant interoperability issues.

**Confidence:** High

---
