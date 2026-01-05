# Errata Reports

Total reports: 1

---


## Report 1: 9825-8-1

**Label:** Inconsistent YANG module name in Security Section (ietf-ospf-admin-tag vs ietf-ospf-admin-tags)

**Bug Type:** Inconsistency

**Explanation:**

The Security Considerations section refers to the YANG module as 'ietf-ospf-admin-tag' (singular) while the defined module in Section 7 and its IANA registration in Section 9 use 'ietf-ospf-admin-tags' (plural), leading to a naming inconsistency.

**Justification:**

- The security text in Section 8 uses 'ietf-ospf-admin-tag' whereas the YANG module header and IANA registration clearly specify 'ietf-ospf-admin-tags' (E1, E2, E3).
- Both the Structural and Terminology experts noted that this discrepancy can confuse implementers and automated tools, as the formal name is used elsewhere in the document (E4, E5, E6).

**Evidence Snippets:**

- **E1:**

  The "ietf-ospf-admin-tag" YANG module defines a data model that is designed to be accessed via YANG-based management protocols, such as NETCONF [RFC6241] and RESTCONF [RFC8040].

- **E2:**

  YANG module header in Section 7.2: `module ietf-ospf-admin-tags { ... }` and IANA registration in Section 9: `Name:  ietf-ospf-admin-tags`

- **E3:**

  Section 8 (Security Considerations):  `The "ietf-ospf-admin-tag" YANG module defines a data model that is designed to be accessed via YANG-based management protocols, such as NETCONF [RFC6241] and RESTCONF [RFC8040].`

- **E4:**

  YANG module header in Section 7.2:  <CODE BEGINS> file "ietf-ospf-admin-tags@2025-07-31.yang" module ietf-ospf-admin-tags {

- **E5:**

  YANG tree heading in Section 7.1:  module: ietf-ospf-admin-tags

- **E6:**

  IANA Considerations (Section 9):  Name:  ietf-ospf-admin-tags  Namespace:  urn:ietf:params:xml:ns:yang:ietf-ospf-admin-tags  Prefix:  ospf-admin-tags

**Evidence Summary:**

- (E1) Section 8 refers to the module as 'ietf-ospf-admin-tag'.
- (E2) Section 7.2 and Section 9 use 'ietf-ospf-admin-tags' as the module name.
- (E3) The security text repeats the singular form, creating an inconsistency.
- (E4) The YANG module header file confirms the correct plural module name.
- (E5) The YANG tree heading aligns with the plural naming.
- (E6) IANA registration verifies the use of 'ietf-ospf-admin-tags'.

**Fix Direction:**

In Section 8, replace "ietf-ospf-admin-tag" with "ietf-ospf-admin-tags" to be consistent with the module header and the IANA registration.


**Severity:** Medium
  *Basis:* Terminology Expert rated the inconsistency as Medium due to potential confusion between the informal and formal module names, even though the issue is relatively minor.

**Confidence:** High

---