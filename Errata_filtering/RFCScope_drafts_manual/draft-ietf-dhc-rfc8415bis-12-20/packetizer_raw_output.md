# Errata Reports

Total reports: 1

---


## Report 4: draft-ietf-dhc-rfc8415bis-12-20-4

**Label:** Misstated Mitigation for RKAP Key Disclosure via RFC 7610/SAVI

**Bug Type:** Inconsistency

**Explanation:**

The security discussion inaccurately implies that deploying RFC 7610 (DHCPv6‑Shield) and RFC 7513 (SAVI) can prevent RKAP reconfigure key disclosure, overstating their protective guarantees.

**Justification:**

- The text suggests that RKAP key exposure over public networks can be easily prevented by using these mechanisms.
- In reality, RFC 7610 provides filtering of rogue DHCP messages and RFC 7513 focuses on validating source bindings, neither of which encrypts or confidentially protects legitimate DHCPv6 exchanges.

**Evidence Snippets:**

- **E1:**

  The privacy/security discussion states that an attacker on a public (e.g., Wi‑Fi) network could observe the RKAP reconfigure key during the initial exchange and that “this is relatively easily prevented by disallowing direct client‑to‑client communication on these networks or using [RFC7610] and [RFC7513].”

- **E2:**

  RFC 7610, DHCPv6‑Shield is explicitly defined as a filter to block DHCPv6 server‑to‑client messages that arrive on untrusted L2 ports, in order to protect against rogue DHCPv6 servers; it does not provide confidentiality for legitimate DHCPv6 traffic and does not prevent a host from listening on a port that is allowed to receive such traffic.

- **E3:**

  Thus, saying that use of RFC 7610 and RFC 7513 “relatively easily” prevents an attacker from learning the RKAP key by eavesdropping over a shared medium overstates what those RFCs actually guarantee and misrepresents their scope.

**Evidence Summary:**

- (E1) Indicates the claimed mitigation for RKAP key exposure using RFC 7610 and RFC 7513.
- (E2) Explains that these RFCs only filter or validate DHCPv6 messages and do not ensure confidentiality.
- (E3) Concludes that the stated mitigation is an overstatement of the RFCs' capabilities.

**Fix Direction:**

Revise the text to accurately reflect that RFC 7610 and RFC 7513 provide filtering and validation, not confidentiality, thereby not fully preventing passive RKAP key disclosure.

**Severity:** Medium
  *Basis:* The misrepresentation could lead operators to rely on insufficient protection measures, posing a potential security risk.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1

---