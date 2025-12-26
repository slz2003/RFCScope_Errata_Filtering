# Errata Reports

Total reports: 1

---

## Report 1: 9815-2-1

**Label:** Ambiguous scope of SPF Decision Process and optional-attribute advertisement rule

**Bug Type:** Underspecification

**Explanation:**

Section 2’s wording on altering the Decision Process and suppressing optional path attributes can be read as applying globally, whereas later sections limit these changes strictly to BGP-LS-SPF, creating ambiguity for implementers.

**Justification:**

- Section 2 describes the BGP SPF extensions as leveraging base BGP unchanged (except for the Decision Process) and mandates that optional attributes SHOULD NOT be advertised, without clearly restricting the scope.
- Subsequent sections (Sections 1, 5.1, and 6.4) explicitly restrict the SPF Decision Process and attribute rules to the BGP-LS-SPF SAFI, which conflicts with the broader reading of Section 2.

**Evidence Snippets:**

- **E1:**

  With the exception of the Decision Process, BGP SPF extensions leverage the BGP protocol [RFC4271] without change. This includes the BGP protocol Finite State Machine, BGP messages and their encodings, the processing of BGP messages, BGP attributes and path attributes, BGP NLRI encodings, and any error handling defined in [RFC4271], [RFC4760], and [RFC7606]. (Section 2)

- **E2:**

  Due to changes in the Decision Process, there are mechanisms and encodings that are no longer applicable. Unless explicitly specified in the context of BGP SPF, all optional path attributes SHOULD NOT be advertised. If received, all path attributes MUST be accepted, validated, and propagated consistently with the BGP protocol [RFC4271], even if not needed by BGP SPF. (Section 2)

- **E3:**

  The modifications to [RFC4271] for BGP SPF described herein only apply to IPv4 and IPv6 as underlay unicast Subsequent Address Family Identifiers (SAFIs). Operations for any other BGP SAFIs are outside the scope of this document. (Section 1, Introduction)

- **E4:**

  The SPF-based Decision Process (Section 6) applies only to the BGP-LS-SPF SAFI and MUST NOT be used with other combinations of the BGP-LS AFI (16388). (Section 5.1)

- **E5:**

  While the BGP-LS-SPF address family and the BGP unicast address families may install routes into the routing tables of the same device, they operate independently (i.e., ‘ships-in-the-night’ mode). There is no implicit route redistribution between the BGP-LS-SPF address family and the BGP unicast address families. (Section 6.4)

**Evidence Summary:**

- (E1) Section 2 describes BGP SPF extensions as using base BGP with an exception for the Decision Process.
- (E2) Section 2 mandates suppression of optional path attributes without per-SAFI qualification.
- (E3) The Introduction limits modifications to IPv4/IPv6 underlay SAFIs.
- (E4) Section 5.1 confines the SPF Decision Process to the BGP-LS-SPF SAFI.
- (E5) Section 6.4 reinforces independent operation between BGP-LS-SPF and unicast families.

**Fix Direction:**

Modify Section 2 to clearly state that both the altered Decision Process and the optional attribute suppression apply only to BGP-LS-SPF NLRI, ensuring that other BGP SAFIs retain standard RFC4271 behavior.


**Severity:** Low
  *Basis:* The ambiguity is mostly editorial; although a misinterpretation could mislead implementers, the full document provides sufficient constraints to prevent protocol breakage if read in entirety.

**Confidence:** High

---
