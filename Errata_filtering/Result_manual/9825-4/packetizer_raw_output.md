# Errata Reports

Total reports: 4

---

## Report 1: 9825-4-1

**Label:** ECMP vs. Summary Tag Propagation Conflict

**Bug Type:** Both

**Explanation:**

There is an ambiguity between the ECMP rule in Section 4.1 and the exclusion rule for summaries in Section 4, resulting in conflicting instructions on whether component route tags should be propagated in aggregated routes.

**Justification:**

- The ECMP rule mandates that an ABR must associate tags from one of the contributing LSAs when multiple LSAs contribute to a route.
- In contrast, the summarization section explicitly states that tags from component routes SHOULD NOT be propagated to the summary.

**Evidence Snippets:**

- **E1:**

  ECMP rule in 4.1: “When multiple LSAs contribute to an OSPF route, it is possible that these LSAs will all have different tags. In this situation, the OSPF ABR propagating the route to other areas with inter‑area LSAs MUST associate the tags from one of the LSAs contributing a path and, if the implementation supports multiple tags, MAY associate tags from multiple contributing LSAs up to the maximum number of tags supported.”

- **E2:**

  Aggregation exception in Section 4: “For configured area ranges, NSSA ranges, and configured aggregation of redistributed routes, tags from component routes SHOULD NOT be propagated to the summary. Implementations SHOULD provide a mechanism to configure multiple tags for area ranges, NSSA ranges, and redistributed route summaries.”

**Evidence Summary:**

- (E1) ECMP rule mandates association of tags from one contributing LSA in ECMP scenarios.
- (E2) Aggregation exception forbids propagating component route tags to summaries.

**Fix Direction:**

Clarify Section 4.1 to apply only to ECMP of identical prefixes and explicitly exempt summary/aggregated routes, or provide guidance on reconciling the conflicting rules.


**Severity:** Medium
  *Basis:* The conflict between a MUST requirement and a SHOULD NOT recommendation in overlapping conditions could lead to inconsistent tag propagation and policy discrepancies across implementations.

**Confidence:** High

---

## Report 2: 9825-4-2

**Label:** Ambiguous Scope of 'All Types of Prefixes' Requirement

**Bug Type:** Underspecification

**Explanation:**

The requirement that an OSPF router must advertise and interpret a tag for 'all types of prefixes' is ambiguous because it is not clear whether this applies to every OSPF prefix or only to those carried in the specific TLVs described in Section 3.

**Justification:**

- Section 4 states the normative requirement for all types of prefixes, while Section 3 limits the Administrative Tag Sub‑TLV to a specific set of TLVs.
- This discrepancy could lead to different interpretations about which prefix types require tag support.

**Evidence Snippets:**

- **E1:**

  Section 4 opening sentence: “An OSPF router supporting this specification MUST be able to advertise and interpret at least one tag for all types of prefixes.”

- **E2:**

  Section 3 limits applicability of the Administrative Tag Sub‑TLV: “The Administrative Tag Sub‑TLV specified herein will be valid as a sub‑TLV of the following TLVs… (OSPFv2 Extended Prefix TLV; OSPFv3 Inter‑Area‑Prefix, Intra‑Area‑Prefix, and External‑Prefix TLVs; SRv6 Locator TLV).”

**Evidence Summary:**

- (E1) Section 4 mandates tag support for all types of prefixes.
- (E2) Section 3 restricts the usage of the Administrative Tag Sub‑TLV to specific TLVs.

**Fix Direction:**

Amend the requirement to clarify that 'all types of prefixes' refers only to those prefixes carried in the TLVs defined in Section 3 and for AS‑External/NSSA prefixes using the existing External Route Tag field.


**Severity:** Medium
  *Basis:* The ambiguity in the scope of the normative requirement could lead to divergent implementations and interoperability issues.

**Confidence:** High

---

## Report 3: 9825-4-3

**Label:** Underspecified Tag Truncation Subset Selection

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that the order of tags be preserved when propagating multiple tags, but does not specify which subset of tags should be retained when a router supports fewer tags than are present.

**Justification:**

- The requirement to preserve tag order is intended to provide a consistent view for routers with limited capacity, yet it does not define whether the first N tags, or some other subset, should be propagated.
- This underspecification may result in different routers selecting different subsets of tags for the same prefix.

**Evidence Snippets:**

- **E1:**

  Section 4: “When propagating multiple tags between areas as previously described, the order of the tags MUST be preserved so that implementations supporting fewer tags will have a consistent view across areas.”

- **E2:**

  Section 6: “However, the default behavior SHOULD be to advertise or propagate the lesser number of all the tags associated with the prefix or the maximum number of tags supported by the implementation.”

**Evidence Summary:**

- (E1) Requires preserving tag order when truncating.
- (E2) Specifies default propagation based on maximum supported tags without defining the selection criteria.

**Fix Direction:**

Define a normative rule that, by default, routers MUST retain the first N tags in the received order when their capacity is exceeded.


**Severity:** Medium
  *Basis:* Without a specific rule for tag subset selection, different implementations may propagate inconsistent tag sets, undermining policy consistency across areas.

**Confidence:** High

---

## Report 4: 9825-4-4

**Label:** Ambiguous Interaction Between Local Policy and ECMP Tag Propagation Rule

**Bug Type:** Both

**Explanation:**

The specification allows local policy to control tag propagation, yet Section 4.1 unconditionally requires that an ABR must associate tags from a contributing LSA in ECMP scenarios, creating a potential conflict between local policy settings and mandatory behavior.

**Justification:**

- Section 4 indicates that administrative tag propagation is subject to local policy, allowing for inhibition of tag advertisement.
- Conversely, Section 4.1 imposes a MUST requirement for associating tags in ECMP cases with no exception for local policy, leading to ambiguity in how local policy should be applied.

**Evidence Snippets:**

- **E1:**

  Section 4: “An OSPF router supporting this specification SHOULD propagate administrative tags when acting as an Area Border Router (ABR) and when originating summary advertisements into other areas (unless inhibited by local policy (Section 6)).”

- **E2:**

  Section 4.1: “When multiple LSAs contribute to an OSPF route, it is possible that these LSAs will all have different tags. In this situation, the OSPF ABR propagating the route to other areas with inter‑area LSAs MUST associate the tags from one of the LSAs contributing a path…”

**Evidence Summary:**

- (E1) Indicates that tag propagation can be inhibited by local policy.
- (E2) Imposes an unconditional MUST requirement for associating tags in ECMP scenarios.

**Fix Direction:**

Clarify whether local policy may override the ECMP requirement or if the ECMP rule applies regardless, and update the text to resolve the conflicting directives.


**Severity:** Medium
  *Basis:* This ambiguity may cause inconsistent behavior across implementations, leading to operational surprises in environments where local policies are used to control tag propagation.

**Confidence:** High

---
