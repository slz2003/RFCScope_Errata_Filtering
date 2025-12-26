# Errata Reports

Total reports: 3

---

## Report 1: 9857-3-1

**Label:** Misuse of TLVs 1028/1029 as Node Descriptor TLVs in Local Node Descriptors TLV

**Bug Type:** Both

**Explanation:**

RFC 9857 mandates that TLVs 1028 and 1029 be included as Node Descriptor TLVs in the Local Node Descriptors TLV, yet these TLVs are defined in RFC 9552 as Node/Link Attribute TLVs, creating a classification conflict.

**Justification:**

- RFC 9857 Section 3 requires the Local Node Descriptors TLV to include TLVs 1028 and 1029 as Node Descriptor TLVs.
- RFC 9552 defines TLVs 1028/1029 solely as Node/Link Attribute TLVs (with valid Node Descriptor sub‐TLVs limited to types 512–515), leading to potential misinterpretation of the NLRI key.

**Evidence Snippets:**

- **E1:**

  The Local Node Descriptors TLV MUST include at least one of the following Node Descriptor TLVs: * IPv4 Router-ID of Local Node (TLV 1028) [RFC9552] … * IPv6 Router-ID of Local Node (TLV 1029) [RFC9552]

- **E2:**

  In RFC 9552, the only Node Descriptor sub‑TLV types are 512–515 (Table “Node Descriptor Sub-TLVs”), and TLVs 1028/1029 are defined as Node/Link *Attribute* TLVs, not descriptor sub‑TLVs, e.g. “IPv4 Router-ID of Local Node (1028)” and “IPv6 Router-ID of Local Node (1029)” under Link/Node Attribute TLVs.

**Evidence Summary:**

- (E1) RFC 9857 mandates the inclusion of TLVs 1028/1029 as Node Descriptor TLVs in the Local Node Descriptors TLV.
- (E2) RFC 9552 defines these TLVs as Node/Link Attribute TLVs rather than as part of the Node Descriptor sub‑TLV set.

**Fix Direction:**

Clarify in RFC 9857 that, for NLRI Type 5, either new Node Descriptor sub‑TLV types should be defined for IPv4/IPv6 headend identifiers or explicitly state that TLVs 1028 and 1029 are valid as Node Descriptor sub‑TLVs despite their original attribute definition.


**Severity:** Medium
  *Basis:* The conflicting TLV classifications may lead to interoperability issues as different implementations might treat these TLVs differently when constructing the NLRI key.

**Confidence:** High

---

## Report 2: 9857-3-2

**Label:** Ambiguous Scope of the Identifier (BGP-LS Instance-ID) for SR Policy Candidate Path NLRI

**Bug Type:** Underspecification

**Explanation:**

The Identifier field for NLRI Type 5 is defined as an 8‑octet BGP‑LS Instance-ID per RFC 9552, but RFC 9857 does not clarify which IGP or policy domain this identifier should represent when SR Policies span multiple domains.

**Justification:**

- RFC 9857 Section 3 states: Identifier: An 8‑octet value as defined in Section 5.2 of [RFC9552].
- RFC 9552 ties the Identifier field to the IGP routing domain, yet SR Policy candidate paths can be generated in multi‑domain or PCE scenarios without clear guidance on which domain’s Instance-ID to use.

**Evidence Snippets:**

- **E1:**

  Identifier: An 8-octet value as defined in Section 5.2 of [RFC9552].

- **E2:**

  In RFC 9552 Section 5.2: “The Identifier field carries an 8‑octet BGP‑LS Instance Identifier (Instance-ID) number that is used to identify the IGP routing domain where the NLRI belongs. …”

**Evidence Summary:**

- (E1) RFC 9857 defines the Identifier field simply as an 8‑octet value per RFC 9552.
- (E2) RFC 9552 associates the Identifier with an IGP routing domain, which does not clearly map to SR Policy candidate paths in multi‑domain scenarios.

**Fix Direction:**

Specify in RFC 9857 which domain or entity (e.g., the headend’s IGP instance or a dedicated policy domain) the Identifier must correspond to, especially in PCE or multi‑domain deployments.


**Severity:** Medium
  *Basis:* Ambiguity in the intended scope of the Instance-ID may lead to inconsistent correlation of SR Policy state with topological information across different implementations.

**Confidence:** Medium

---

## Report 3: 9857-3-3

**Label:** Confusing Description of BGP Confederation Member TLV 517 Contents

**Bug Type:** Inconsistency

**Explanation:**

RFC 9857 describes TLV 517 as containing the headend node of the SR Policy, yet it actually carries the Member-AS Number, leading to misleading terminology that may confuse implementers.

**Justification:**

- The document states that TLV 517 'contains the ASN of the confederation member (i.e., Member-AS Number); if BGP confederations are used, it contains the headend node of the SR Policy,' conflating an ASN with a node identifier.
- This description is at odds with the standard definitions in RFC 5065 and RFC 9086, where TLV 517 is intended solely to carry the Member-AS Number.

**Evidence Snippets:**

- **E1:**

  BGP Confederation Member (TLV 517) [RFC9086], which contains the ASN of the confederation member (i.e., Member-AS Number); if BGP confederations are used, it contains the headend node of the SR Policy.

**Evidence Summary:**

- (E1) The text specifies that TLV 517 both carries the Member-AS Number and, if BGP confederations are used, 'contains the headend node,' which creates ambiguity.

**Fix Direction:**

Revise the description for TLV 517 in RFC 9857 to state unambiguously that it carries the Member-AS Number associated with the headend’s confederation membership, not a node identifier.


**Severity:** Low
  *Basis:* The misdescription is primarily a terminology issue and is unlikely to cause significant interoperability problems, though it may confuse implementers.

**Confidence:** Medium

---
