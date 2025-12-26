# Errata Reports

Total reports: 1

---

## Report 2: 9710-5-2

**Label:** Informal Registry Naming in Section 5 Additional Information Table

**Bug Type:** Inconsistency

**Explanation:**

The registry names in the Additional Information table are presented in a slightly informal or abbreviated manner compared to the exact IANA registry titles, which might cause minor confusion for pedantic readers despite the URLs providing unambiguous guidance.

**Justification:**

- CrossRFC Expert noted a minor editorial quirk regarding icmpCodeIPv4 being directed to the 'ICMP Type Numbers' registry name, which emphasizes types rather than codes.
- Terminology Expert observed that the registry names (e.g., 'IGMP Type Numbers' and 'Private Enterprise Numbers (PENs)') are not exactly as per the official titles, even though the URLs remove any ambiguity regarding the intended registry.

**Evidence Snippets:**

- **E2:**

  Possible minor editorial quirks: One could argue that pointing icmpCodeIPv4 to the “ICMP Type Numbers” registry name is slightly imprecise verbally, since that registry title highlights types rather than codes; however, the URL is to the overall ICMP parameters page that includes the code tables, so an implementer will land on the correct registry content.

- **E3:**

  |32 | icmpTypeCodeIPv4 | See the "ICMP Type Numbers" registry [https://www.iana.org/assignments/icmp-parameters]
|33 | igmpType | See the "IGMP Type Numbers" registry [https://www.iana.org/assignments/igmp-type-numbers]
|139| icmpTypeCodeIPv6 | See the "ICMPv6 'type' Numbers" and "ICMPv6 'Code' Fields" registries [https://www.iana.org/assignments/icmpv6-parameters]
|346| privateEnterpriseNumber | See the "Private Enterprise Numbers (PENs)" registry [https://www.iana.org/assignments/enterprise-numbers]
The table caption: Table 1: Cite an IANA Registry Under Additional Information

**Evidence Summary:**

- (E2) CrossRFC Expert highlights the imprecision in using the label 'ICMP Type Numbers' for icmpCodeIPv4, which focuses on types rather than codes.
- (E3) Terminology Expert provides evidence of the abbreviated registry names used in the table, showing that they differ slightly from the full, official IANA registry titles.


**Severity:** Low
  *Basis:* The issue is purely editorial, affecting only stylistic presentation without impacting interoperability or data encoding.

**Confidence:** High
---
