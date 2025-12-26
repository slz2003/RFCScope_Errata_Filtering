# Errata Reports

Total reports: 1

---

## Report 1: 9805-6-1

**Label:** Overbroad Mitigation Claim for IPv6 RAO Security Considerations in RFC 9805 Section 6

**Bug Type:** Inconsistency

**Explanation:**

RFC 9805 Section 6 asserts that the document mitigates all security considerations associated with the IPv6 Router Alert option, yet it only deprecates RAO for new protocols while permitting existing protocols to continue using it, leaving inherent security risks unaddressed.

**Justification:**

- Section 4 of RFC 9805 clearly states that protocols already using RAO may continue to do so, contrasting with the blanket mitigation claim in Section 6 (see E1 and E2).
- RFC 6398 details residual risks intrinsic to RAO usage, such as an inability to reliably filter unwanted RAO packets, indicating that complete mitigation is not achieved (see E3).
- Multiple expert analyses (Scope, Deontic, CrossRFC, and Boundary) have flagged this overbroad claim as misleading and potentially harmful by suggesting that no further RAO-specific protections are needed.

**Evidence Snippets:**

- **E1:**

  RFC 9805 Section 4: “This document deprecates the IPv6 Router Alert option.  Protocols that use the IPv6 Router Alert option MAY continue to do so, even in future versions.  However, new protocols that are standardized in the future MUST NOT use the IPv6 Router Alert option.”

- **E2:**

  RFC 9805 Section 6: “This document mitigates all security considerations associated with the IPv6 Router Alert option.  These security considerations can be found in [RFC2711], [RFC6192], and [RFC6398].”

- **E3:**

  RFC 6398 Section 3: IP Router Alert “does not provide a convenient universal mechanism to accurately and reliably distinguish between IP Router Alert packets of interest and unwanted IP Router Alert packets.  This, in turn, creates a security concern when the IP Router Alert Option is used…”

**Evidence Summary:**

- (E1) Demonstrates that existing RAO-using protocols are allowed to continue operation.
- (E2) Contains the sweeping claim of complete security mitigation.
- (E3) Highlights the inherent risks of RAO usage that remain even with mitigations.

**Fix Direction:**

Revise Section 6 to clarify that the mitigation claim applies only to new protocols by deprecating RAO usage, and explicitly state that existing RAO-related security risks must continue to be addressed using the standard mitigation practices outlined in RFC 6398 and RFC 6192.


**Severity:** Medium
  *Basis:* The misleading claim could result in operators and implementers neglecting necessary protections, leaving systems vulnerable to denial-of-service attacks via continued RAO usage.

**Confidence:** High

---
