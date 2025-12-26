# Errata Reports

Total reports: 3

---

## Report 1: 9868-7-1

**Label:** UDP Length Field Inconsistency: ≥8 vs 0 for IPv6 Jumbograms

**Bug Type:** Inconsistency

**Explanation:**

The document mandates that the UDP Length field must be at least 8 bytes, yet also permits UDP Length to be 0 for IPv6 Jumbograms, resulting in a direct semantic conflict.

**Justification:**

- Section 10 enforces that the UDP Length is at least 8 bytes, while Section 22 explicitly describes the valid case of UDP Length==0 in IPv6 Jumbograms, leading to inconsistent rules (E1, E2).

**Evidence Snippets:**

- **E1:**

  Section 10:  >> The UDP Length MUST be at least as large as the UDP header (8) and no larger than the IP transport payload. Datagrams with length values outside this range MUST be silently dropped as invalid and logged.

- **E2:**

  Section 22 (interaction with Jumbograms):  UDP Options cannot be supported when a UDP packet has no independent UDP Length. One such case is when UDP Length==0 in IPv6, intended for (but not limited to) IPv6 Jumbograms [RFC2675]. Note that although this technique is ‘Standard’, the specification did not ‘update’ UDP [RFC0768].

**Evidence Summary:**

- (E1) Section 10 mandates UDP Length to be at least 8 bytes.
- (E2) Section 22 permits UDP Length==0 for IPv6 Jumbograms, creating a conflict.

**Fix Direction:**

Specify separate normative conditions for UDP Length when handling IPv6 Jumbograms to reconcile the general minimum length requirement with the special case of UDP Length==0.


**Severity:** High
  *Basis:* The inconsistency may lead to valid UDP Jumbogram packets being dropped, resulting in interoperability failures.

**Confidence:** High

---

## Report 2: 9868-7-2

**Label:** Underspecification in IPv6 UDP Length Calculation for Jumbo Payloads

**Bug Type:** Underspecification

**Explanation:**

The formula for the UDP Length upper bound in IPv6 does not account for the Jumbo Payload option, leaving the computation of the IP transport payload ambiguous for such packets.

**Justification:**

- Section 7 uses the IPv6 Payload Length field (which is 0 when a Jumbo Payload option is used) to derive UDP Length, while Section 22 restricts UDP Options for cases when UDP Length==0, without clarifying how to treat Jumbo Payload packets (E1, E2).

**Evidence Snippets:**

- **E1:**

  Section 7 (IPv6 formula):  For IPv6, the IP Payload Length field indicates the transport payload after the base IPv6 header… so the upper bound for UDP Length is given by:  UDP_Length <= IPv6_Payload_Length - sum(extension header lengths)

- **E2:**

  Section 22:  UDP Options cannot be supported when a UDP packet has no independent UDP Length. One such case is when UDP Length==0 in IPv6, intended for (but not limited to) IPv6 Jumbograms [RFC2675].

**Evidence Summary:**

- (E1) Section 7 derives UDP Length from the IPv6 Payload Length without distinguishing Jumbo Payload cases.
- (E2) Section 22 forbids UDP Options for UDP Length==0, highlighting an oversight when Jumbo Payload options are used.

**Fix Direction:**

Clarify in Section 7 that the IPV6 payload length formula applies only when no Jumbo Payload option is present or provide a separate computation rule for Jumbo Payload scenarios.


**Severity:** Medium
  *Basis:* The ambiguity may result in miscalculation of UDP Length in implementations handling Jumbo Payloads, leading to potential validation errors.

**Confidence:** High

---

## Report 3: 9868-7-3

**Label:** Underspecification in IPv6 IP Transport Payload Calculation with IPComp

**Bug Type:** Underspecification

**Explanation:**

The method for calculating the 'IP transport payload' in IPv6 ignores non-extension shim headers such as IPComp, leaving the UDP Options processing undefined in these scenarios.

**Justification:**

- While the IPv4 discussion accounts for shim headers (e.g., IPComp), the IPv6 formula in Section 7 only considers extension headers, thereby not clarifying how to handle UDP when IPComp is present (E1, E2).

**Evidence Snippets:**

- **E1:**

  For IPv6, however, Section 7 only discusses “IPv6 extension headers” as defined in RFC 8200 and gives the formula UDP_Length <= IPv6_Payload_Length - sum(extension header lengths), with no mention of non‑extension “shim” headers such as IPComp.

- **E2:**

  Thus, in a valid IPv6+IPComp configuration, the outer IPv6 Payload Length covers an IPComp header plus compressed payload, and the first Next Header visible at the IPv6 level is 108 (IPComp), not 17 (UDP). The RFC 9868 formula for deriving the “IP transport payload” from IPv6 Payload Length minus extension-header lengths does not say how to handle this case: it neither clarifies that UDP Options are not defined when UDP is carried under IPComp, nor gives a rule analogous to the IPv4 shim‑header discussion.

**Evidence Summary:**

- (E1) Section 7’s IPv6 payload calculation only accounts for extension headers, omitting IPComp.
- (E2) The text does not specify how to exclude the IPComp header when computing the IP transport payload for UDP.

**Fix Direction:**

Provide clear guidance on computing the IP transport payload in IPv6 when IPComp (or other shim headers) is present, or explicitly disallow UDP Options in such configurations.


**Severity:** Medium
  *Basis:* The lack of guidance may lead to inconsistent implementation regarding UDP Options when IPComp is involved, causing ambiguity in UDP processing.

**Confidence:** Medium

---
