# Errata Reports

Total reports: 3

---

## Report 1: 9830-1-1

**Label:** Ambiguous RT Removal Behavior by Intermediate BGP Nodes

**Bug Type:** Underspecification

**Explanation:**

The specification permits intermediate BGP nodes to filter and/or remove the Route Target extended community before propagation, yet later mandates that an SR Policy update must contain either a Route Target or the NO_ADVERTISE community. This creates an ambiguity regarding how an update can remain valid if RTs are removed.

**Justification:**

- The spec allows configuration to filter and/or remove the Route Target extended community before propagation.
- Normative rules (Section 4.2.1 and 4.2.3) require that an update contains either RT(s) or NO_ADVERTISE, meaning that if all RTs are removed and NO_ADVERTISE cannot be applied for propagated events, the update becomes invalid.

**Evidence Snippets:**

- **E1:**

  By default, a BGP node receiving an SR Policy NLRI SHOULD NOT remove the Route Target extended community before propagation. An implementation MAY provide support for configuration to filter and/or remove the Route Target extended community before propagation.

- **E2:**

  The SR Policy update MUST have either the NO_ADVERTISE community, at least one Route Target extended community in IPv4-address format, or both. If a router supporting this specification receives an SR Policy update with no Route Target extended communities and no NO_ADVERTISE community, the update MUST be considered to be malformed.

- **E3:**

  If a route reflector actually removes all RTs from an update that previously had them and still re-advertises the NLRI, it will be sending an update that violates the validity condition in 4.2.1 and that cannot legally be marked with NO_ADVERTISE (since 4.2.3 bans propagation of such routes).

**Evidence Summary:**

- (E1) Indicates that removal of RTs is allowed by configuration before propagation.
- (E2) Specifies that a valid SR Policy update must have either RT(s) or NO_ADVERTISE.
- (E3) Demonstrates that removing all RTs may result in an update that violates validity conditions.

**Fix Direction:**

Clarify in the specification that if an intermediate BGP node removes the RT extended community, it must either not propagate the update or must apply a compensating action (e.g., adding NO_ADVERTISE) to ensure the update remains valid.


**Severity:** Medium
  *Basis:* Ambiguous handling of RT removal can lead to propagation of malformed SR Policy updates, adversely affecting proper BGP behavior.

**Confidence:** High

---

## Report 2: 9830-1-2

**Label:** Ambiguous Route Target Requirement in Section 1 vs. NO_ADVERTISE Mode in Normative Sections

**Bug Type:** Inconsistency

**Explanation:**

Section 1’s descriptive list implies that SR Policy updates always include one or more Route Target extended communities, which conflicts with later normative sections that permit valid updates using only the NO_ADVERTISE community for direct headend sessions.

**Justification:**

- The bullet list in Section 1 unconditionally lists the RT extended community as part of the SR Policy advertisement.
- Normative sections (4.1, 4.2.1, and 4.2.2) clearly allow an update to be valid when the NO_ADVERTISE community is used in lieu of an RT, highlighting a scope mismatch.

**Evidence Snippets:**

- **E1:**

  One or more IPv4 address-specific format Route Target extended community ... attached to the SR Policy CP advertisement that indicates the intended headend of such an SR Policy CP advertisement.

- **E2:**

  If no route target is attached to the SR Policy NLRI, then it is assumed that the originator sends the SR Policy update directly ... to the intended receiver. In such a case, the NO_ADVERTISE community MUST be attached to the SR Policy update.

- **E3:**

  The SR Policy update MUST have either the NO_ADVERTISE community, at least one Route Target extended community in IPv4-address format, or both.

**Evidence Summary:**

- (E1) Shows that Section 1 lists RT as a mandatory component in the advertisement.
- (E2) Indicates that direct sessions use the NO_ADVERTISE community when RT is absent.
- (E3) Sets out the normative validity rule allowing either RT, NO_ADVERTISE, or both, thereby conflicting with the unqualified description in Section 1.

**Fix Direction:**

Revise Section 1 to explicitly state that the inclusion of Route Target extended communities applies only to scenarios involving propagation (via RRs or across ASes), while direct headend sessions may use the NO_ADVERTISE community exclusively.


**Severity:** Low
  *Basis:* Although the normative sections are clear, the introductory text could mislead implementers by overstating the universal requirement for RTs.

**Confidence:** Medium

---

## Report 3: 9830-1-3

**Label:** Color-Only Type 3 Mismatch: 'Reserved' in Text vs. 'Unassigned' in IANA Table

**Bug Type:** Inconsistency

**Explanation:**

The normative text in Section 3 mandates that Color-Only Type 3 is reserved and must be treated like Type 0, yet the IANA table in Section 6.9 lists it as 'Unassigned', presenting a conflicting description.

**Justification:**

- Section 3 clearly states that Type 3 is 'Reserved for future use and SHOULD NOT be used' and requires receivers to treat it as Type 0.
- The IANA registry (Table 9 in Section 6.9) labels Type 3 as 'Unassigned', which may mislead implementers regarding its proper handling.

**Evidence Snippets:**

- **E1:**

  Section 3 (Color Extended Community):  
“Type 3 (bits 11):  Reserved for future use and SHOULD NOT be used.  Upon reception, an implementation MUST treat it like Type 0.”

- **E2:**

  Section 6.9, IANA registry “Color Extended Community Color-Only Types”, Table 9:
| 3    | Unassigned                            | RFC 9830  |

**Evidence Summary:**

- (E1) Indicates that Type 3 is normatively reserved and must be handled as Type 0.
- (E2) Shows that the IANA table describes Type 3 as unassigned, conflicting with the text.

**Fix Direction:**

Update the IANA table and associated registry text to reflect the normative status of Type 3 as 'Reserved; SHOULD NOT be used. Upon reception, treat as Type 0', ensuring consistency with Section 3.


**Severity:** Low
  *Basis:* While this discrepancy is unlikely to cause direct interoperability issues, it can lead to confusion about the correct handling of Type 3.

**Confidence:** High

---
