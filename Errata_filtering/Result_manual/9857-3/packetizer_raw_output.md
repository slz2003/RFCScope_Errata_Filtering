# Errata Reports

Total reports: 1

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