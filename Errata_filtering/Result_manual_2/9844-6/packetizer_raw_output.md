# Errata Reports

Total reports: 1

---

## Report 1: 9844-6-1

**Label:** Over-broad Restatement of RFC 4007 'MUST NOT be sent on the wire' Rule in RFC 9844 §6

**Bug Type:** Inconsistency

**Explanation:**

RFC 9844 Section 6 overgeneralizes RFC 4007’s requirement by omitting its exception that permits transmission of the textual <address>%<zone_id> format when all nodes agree on the semantics, potentially misleading implementers.

**Justification:**

- RFC 9844 §6 states that zone identifiers 'must not be sent on the wire' without including the conditional exception present in RFC 4007 (E1).
- RFC 4007 §11.2 explicitly restricts the prohibition to the textual representation, adding the clause 'unless every node that interprets the format agrees on the semantics' (E2), which is omitted in RFC 9844 (E1) and further contextualized in RFC 4007 §12 (E3).

**Evidence Snippets:**

- **E1:**

  RFC 9844 §6: “As explained in [RFC4007], zone identifiers are of local significance only and must not be sent on the wire. In particular, see the final security consideration of [RFC4007]… Therefore, software that obtains a zone identifier through a UI should not transmit it further.”

- **E2:**

  RFC 4007 §11.2: “It cannot be assumed that indices are common across all nodes in a zone (see Section 6). Hence, the format MUST be used only within a node and MUST NOT be sent on the wire unless every node that interprets the format agrees on the semantics.”

- **E3:**

  RFC 4007 §12: “Since the use of the textual representation of non-global addresses is restricted to use within a single node, it does not create a security vulnerability from outside the node. However, a malicious node might send a packet that contains a textual IPv6 non-global address with a zone index, intending to deceive the receiving node…”

**Evidence Summary:**

- (E1) RFC 9844 §6 instructs that zone identifiers must not be sent on the wire, omitting RFC 4007’s conditional exception.
- (E2) RFC 4007 §11.2 specifies that the prohibition applies only to the textual <address>%<zone_id> format unless semantic agreement exists.
- (E3) RFC 4007 §12 reinforces the local usage of textual representations and notes potential malicious use when transmitted.

**Severity:** Low
  *Basis:* Deontic expert analysis indicates low overall risk since the issue is advisory and unlikely to cause interop failures, despite potential for misinterpretation.

**Confidence:** Medium

**Experts mentioning this issue:**

- DeonticExpert: Issue-1
- CrossRFCExpert: Issue-1

---
