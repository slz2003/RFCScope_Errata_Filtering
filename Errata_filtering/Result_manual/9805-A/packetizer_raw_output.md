# Errata Reports

Total reports: 3

---

## Report 1: 9805-A-1

**Label:** Ambiguous Dependence on Appendix A as 'Exhaustive List' vs. Actual RAO Usage

**Bug Type:** Underspecification

**Explanation:**

The text ambiguously ties the permission to continue using the IPv6 Router Alert option either to a protocol's actual use or to inclusion in the 'exhaustive list' in Appendix A, creating uncertainty about which protocols are grandfathered.

**Justification:**

- Section 4 states that any protocol using the IPv6 RAO may continue to use it, while Appendix A is described as an exhaustive list, implying that only listed protocols qualify.
- The grouping of protocol families (e.g., RSVP and NSIS) in the table raises questions on whether unlisted RAO users might be excluded.

**Evidence Snippets:**

- **E1:**

  Section 4: “This document deprecates the IPv6 Router Alert option. Protocols that use the IPv6 Router Alert option MAY continue to do so, even in future versions. However, new protocols that are standardized in the future MUST NOT use the IPv6 Router Alert option. Appendix A contains an exhaustive list of protocols that MAY continue to use the IPv6 Router Alert option.”

- **E2:**

  Appendix A: “Table 1 contains an exhaustive list of protocols that use the IPv6 Router Alert option.”

- **E3:**

  The table then groups whole protocol families under a single row, e.g., “Resource Reservation Protocol (RSVP): Both IPv4 and IPv6 implementations” with a handful of extension RFCs cited, and “Next Steps in Signaling (NSIS)” with only RFC 5979 and 5971 cited even though the NSIS suite uses per‑NSLP RAO values.

**Evidence Summary:**

- (E1) Section 4 deprecates the IPv6 RAO but permits its continued use, referencing an 'exhaustive list'.
- (E2) Appendix A claims to be an exhaustive list of protocols using the IPv6 RAO.
- (E3) The grouping in the table introduces ambiguity about which protocols are actually covered.

**Fix Direction:**

Revise the text to clarify that the normative permission to use the IPv6 Router Alert option is based on actual RAO usage rather than solely on inclusion in Appendix A, and explain that the appendix is a snapshot rather than a binding enumeration.


**Severity:** Medium
  *Basis:* Ambiguous phrasing could lead to misinterpretation by implementers regarding which protocols are allowed to continue using the IPv6 RAO, potentially resulting in interoperability issues.

**Confidence:** High

---

## Report 2: 9805-A-2

**Label:** Tension between Global RAO Allowance and MPLS Ping Specific Deprecation

**Bug Type:** Underspecification

**Explanation:**

The global rule permitting continued use of the IPv6 Router Alert option conflicts with the specific deprecation note for MPLS Ping, creating mixed guidance for future implementations.

**Justification:**

- Section 4 provides a blanket permission for protocols using the RAO to continue its use, while Appendix A notes that MPLS Ping's use of the RAO is deprecated.
- RFC 9570 removes the strict RAO requirement for MPLS Ping and marks RAO value 69 as deprecated, further contrasting with the global allowance.

**Evidence Snippets:**

- **E1:**

  Section 4’s global rule: “Protocols that use the IPv6 Router Alert option MAY continue to do so, even in future versions.”

- **E2:**

  Appendix A’s MPLS Ping row: “MPLS Ping (Use of the IPv6 Router Alert option is deprecated)” with references [RFC7506][RFC8029][RFC9570].

- **E3:**

  RFC 9570 explicitly *removes* that RAO requirement and instructs IANA to mark RAO value 69 as “DEPRECATED”.

**Evidence Summary:**

- (E1) A global permission for continued RAO usage is provided in Section 4.
- (E2) MPLS Ping is noted in Appendix A as having its RAO usage deprecated.
- (E3) RFC 9570’s removal of the RAO requirement and deprecation of value 69 complicate the guidance for MPLS Ping.

**Fix Direction:**

Amend the guidance to indicate that MPLS Ping must adhere to the deprecation established by RFC 9570, thereby exempting it from the generic 'MAY continue' rule applied to other protocols.


**Severity:** Medium
  *Basis:* The contradiction could mislead protocol designers regarding whether future MPLS Ping implementations can legitimately use the IPv6 RAO, risking inconsistent deployments.

**Confidence:** High

---

## Report 3: 9805-A-3

**Label:** PGM Listed as IPv6 RAO User Despite Lack of Defined IPv6 RAO Usage

**Bug Type:** Inconsistency

**Explanation:**

RFC 9805 classifies PGM as a protocol that uses the IPv6 Router Alert option, even though its specification only mandates the use of the IP Router Alert Option (pertaining to IPv4) and does not define an IPv6 equivalent.

**Justification:**

- The PGM specification explicitly states that 'SPMs, NCFs, and RDATA MUST be transmitted with the IP Router Alert Option' and references RFC 2113, which is for IPv4.
- Despite this, RFC 9805 Appendix A lists PGM as a protocol allowed to continue using the IPv6 Router Alert option, creating a cross-RFC inconsistency.

**Evidence Snippets:**

- **E1:**

  In the PGM specification, the only explicit Router Alert requirement is “SPMs, NCFs, and RDATA MUST be transmitted with the IP Router Alert Option [6]. This option gives network elements a network-layer indication that a packet should be extracted from IP switching for more detailed processing.” The reference [6] is RFC 2113 (IPv4 IP Router Alert)...

- **E2:**

  RFC 9805 Appendix A claims that Pragmatic General Multicast (PGM) is one of the “protocols that use the IPv6 Router Alert option” and therefore MAY continue to use it.

**Evidence Summary:**

- (E1) PGM’s specification mandates only the IPv4 Router Alert option as per RFC 2113.
- (E2) RFC 9805 lists PGM as an IPv6 RAO user despite the lack of a defined IPv6 mechanism.

**Fix Direction:**

Either revise Appendix A to remove PGM from the list of IPv6 RAO users or update the PGM specification to include a defined IPv6 RAO usage.


**Severity:** Medium
  *Basis:* This inconsistency may mislead implementers into expecting IPv6 RAO behavior in PGM, potentially leading to interoperability problems when the IPv6 mechanism is not actually defined.

**Confidence:** Medium

---
