# Errata Reports

Total reports: 2

---

## Report 1: draft-ietf-tcpm-accurate-ecn-34-4-1

**Label:** Ambiguous Update of RFC 3168 §6.1.1: Entire Section Claimed Updated Yet Exempting Control Packet Prohibitions

**Bug Type:** Inconsistency

**Explanation:**

Section 4 contradicts itself by claiming that the whole of RFC 3168 §6.1.1 is updated while later stating that the prohibition on ECN‐capable control packets (e.g., SYN/SYN‐ACK) remains unchanged.

**Justification:**

- The first bullet in Section 4 asserts a complete update of RFC 3168 §6.1.1 by Section 3.1, implying all rules are replaced.
- A later bullet in the same section excludes the control packet and retransmission prohibitions from the update, creating clear normative ambiguity for implementers.

**Evidence Snippets:**

- **E1:**

  The whole of ‘6.1.1 TCP Initialization’ of [RFC3168] is updated by Section 3.1 of the present specification.

- **E2:**

  Sections 5.2, 6.1.1, 6.1.4, 6.1.5 and 6.1.6 of [RFC3168] prohibit use of ECN on TCP control packets and retransmissions. The present specification does not update that aspect of RFC3168, but it does say what feedback an AccECN Data Receiver ought to provide if it receives an ECN‑capable control packet or retransmission.

- **E3:**

  A host MUST NOT set ECT on SYN or SYN‑ACK packets.

**Evidence Summary:**

- (E1) Section 4 first bullet claims a complete update of RFC 3168 §6.1.1 by Section 3.1.
- (E2) A later bullet clarifies that RFC3168 prohibitions on ECN‐capable control packets are not updated.
- (E3) RFC 3168 §6.1.1 includes the prohibition ‘A host MUST NOT set ECT on SYN or SYN‑ACK packets’.

**Fix Direction:**

Revise Section 4 to clearly specify that only the ECN negotiation and initialization parts of RFC 3168 §6.1.1 are updated (for AccECN mode) while the prohibitions on ECN‐capable control packets and retransmissions remain governed by RFC 3168 (as modified by RFC 8311).

**Severity:** High
  *Basis:* The ambiguity could lead to critical misinterpretations of TCP initialization behavior, potentially compromising interoperability between AccECN-enabled and Classic ECN implementations.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 2: draft-ietf-tcpm-accurate-ecn-34-4-2

**Label:** Unqualified Update of RFC 3168 §6.1.2 Without Limiting to AccECN Mode

**Bug Type:** Both

**Explanation:**

Section 4 unconditionally updates RFC 3168 §6.1.2 by removing the CWR requirement and redefining ECE ACK semantics, even though these changes should apply only to connections operating in AccECN mode.

**Justification:**

- The second bullet in Section 4 states that the responses to an ECN‑Echo (ECE) ACK packet are updated and that setting the CWR flag no longer applies, without restricting this update to AccECN-mode connections.
- Elsewhere, Section 3.1.2 specifies that if a host falls back to Classic ECN, it must continue to comply with the original RFC 3168 rules, which includes the CWR behavior.

**Evidence Snippets:**

- **E1:**

  In ‘6.1.2.  The TCP Sender’ of [RFC3168], all mentions of a congestion response to an ECN‑Echo (ECE) ACK packet are updated by Section 3.2 of the present specification to mean an increment to the sender’s count of CE‑marked packets, s.cep. And the requirements to set the CWR flag no longer apply, as specified in Section 3.1.5 of the present specification. Otherwise, the remaining requirements in ‘6.1.2.  The TCP Sender’ still stand.

- **E2:**

  When an AccECN‑capable host falls back to Classic ECN, “it MUST set both its half connections into the feedback mode shown … If the TCP Client has set itself into Classic ECN feedback mode it MUST then comply with [RFC3168].”

- **E3:**

  RFC 3168 §6.1.2 requires that an ECN‑capable sender set CWR “on the first new data packet sent after the window reduction.”

**Evidence Summary:**

- (E1) Section 4’s second bullet unconditionally replaces the ECN‑Echo handling and CWR flag requirement.
- (E2) Section 3.1.2 mandates that Classic ECN behavior, including CWR setting, must be followed in fallback scenarios.
- (E3) RFC 3168 §6.1.2 explicitly requires the setting of CWR after a window reduction.

**Fix Direction:**

Amend Section 4’s mapping for RFC 3168 §6.1.2 to explicitly state that the update (removal of CWR and change in ECE ACK semantics) applies only to connections operating in AccECN mode, while connections in Classic ECN mode must adhere to the original RFC 3168 requirements.

**Severity:** High
  *Basis:* This underspecification risks improper handling of TCP sender behavior under Classic ECN, leading to potential interoperability issues between different ECN modes.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2

---
