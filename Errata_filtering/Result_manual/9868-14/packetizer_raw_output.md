# Errata Reports

Total reports: 3

---

## Report 1: 9868-14-1

**Label:** Conflicting handling of overlong option Length (Section 10 vs Section 14)

**Bug Type:** Inconsistency

**Explanation:**

The specification defines two contradictory behaviors for options whose Length overruns the available surplus area: Section 10 mandates that any such overrun renders the entire options area malformed (all options must be discarded), while Section 14 instructs that offending options should be silently ignored.

**Justification:**

- Section 10 requires that options indicating overruns of the surplus area result in a malformed surplus area and all options must be discarded.
- Section 14, by contrast, recommends that any option whose length exceeds that of the UDP packet (i.e. would use bytes beyond the surplus area) SHOULD be silently ignored, implying that only the offending option is dropped.
- This divergence can lead to different implementations processing malformed packets in incompatible ways.

**Evidence Snippets:**

- **E1:**

  Options with inherently invalid Length field values, i.e., those that indicate underruns of the option itself or overruns of the surplus area (pointing past the end of the IP payload), MUST be treated as an indication of a malformed surplus area, and all options MUST silently be discarded.

- **E2:**

  Any options whose length exceeds that of the UDP packet (i.e., intending to use data that would have been beyond the surplus area) SHOULD be silently ignored (again to model legacy behavior).

**Evidence Summary:**

- (E1) Section 10 mandates complete discard of the options area for any overrun condition.
- (E2) Section 14 suggests that only the offending TLV should be ignored.

**Fix Direction:**

Revise the text in Section 14 to explicitly align with Section 10 by either requiring that an option that overruns the surplus area causes the entire options area to be discarded or by amending Section 10 so that only the offending option is skipped. The normative language (MUST vs SHOULD) must be reconciled.


**Severity:** High
  *Basis:* This ambiguity affects the fundamental parsing behavior of the options, potentially leading to divergent implementations and interoperability issues as well as impacting security assumptions when processing malformed packets.

**Confidence:** High

---

## Report 2: 9868-14-2

**Label:** Pseudocode omits UNSAFE-option exception vs. UNSAFE drop requirements

**Bug Type:** Both

**Explanation:**

The pseudocode in Section 14 unconditionally mandates delivery of UDP user data after option processing, without explicitly excluding UNSAFE options, which contradicts Section 12’s requirement to drop user data when an UNSAFE option is unsupported or out-of-context.

**Justification:**

- The introductory text in Section 14 states that all options except UNSAFE Options must result in data delivery, but the pseudocode that follows does not differentiate between SAFE and UNSAFE options.
- Section 12 clearly mandates that if any UNSAFE option is unsupported or mislocated, the UDP user data must be dropped.
- This inconsistency can lead to implementations delivering data in scenarios where UNSAFE options should force the datagram to be discarded.

**Evidence Snippets:**

- **E3:**

  Unless configuration settings direct otherwise, all options except UNSAFE Options MUST result in the UDP user data being passed to the upper layer protocol or application, regardless of whether all options are processed, are supported, or succeed.

- **E4:**

  Receivers supporting UDP Options MUST silently drop the UDP user data of the reassembled datagram if any fragment or the entire datagram includes an UNSAFE Option whose Kind is not supported or if an UNSAFE Option appears outside the context of a fragment or reassembled fragments.

**Evidence Summary:**

- (E3) Section 14’s general rule mandates delivery for all options except UNSAFE, yet the pseudocode does not reflect this exception.
- (E4) Section 12 explicitly requires dropping UDP data when an UNSAFE option is unsupported or mis-contextual.

**Fix Direction:**

Adjust the pseudocode in Section 14 so that it explicitly excludes UNSAFE options from its 'deliver data regardless of support' clause and mandates a separate check that drops the UDP user data in accordance with Section 12.


**Severity:** High
  *Basis:* Failing to properly account for UNSAFE options can lead to a violation of the intended safety properties and security guarantees of the protocol, resulting in potentially hazardous and divergent behavior in implementations.

**Confidence:** High

---

## Report 3: 9868-14-3

**Label:** OCS failure versus FRAG forwarding ambiguity

**Bug Type:** Both

**Explanation:**

There is an ambiguity in the specification regarding packets that contain FRAG options: when the OCS fails, the pseudo‐code instructs the receiver to deliver the UDP user data (often as a zero-length datagram), yet Section 11.4 prohibits forwarding individual UDP fragments.

**Justification:**

- The algorithm in Section 14 directs that if OCS fails, the UDP user data is delivered while options are ignored, emulating legacy behavior.
- However, Section 11.4 mandates that individual UDP fragments MUST NOT be forwarded, creating a conflict when a fragment with OCS failure would result in a zero-length datagram.
- This conflicting guidance creates ambiguity about whether an OCS-failed fragment should be delivered as legacy zero-length data or dropped entirely.

**Evidence Snippets:**

- **E5:**

  Individual UDP fragments MUST NOT be forwarded to the user.  The reassembled datagram is received only after complete reassembly, checksum validation, and continued processing of the remaining UDP Options.

- **E6:**

  if (OCS != 0 and OCS fails) or (OCS == 0 and UDP CS != 0) then deliver the UDP user data but ignore other options (this is required to emulate legacy behavior)

**Evidence Summary:**

- (E5) Section 11.4 prohibits the forwarding of individual UDP fragments.
- (E6) The pseudo-code in Section 14, in the event of an OCS failure, instructs delivery of UDP user data, resulting in a zero-length packet.

**Fix Direction:**

Clarify the intended behavior for FRAG-carrying packets when OCS fails by explicitly specifying that such packets, due to their fragment status, must not be delivered—even as zero-length datagrams—in order to conform with the mandate against forwarding individual UDP fragments.


**Severity:** Low
  *Basis:* Although the ambiguity applies only in error conditions and may not affect well-formed packets, it could still lead to inconsistent handling of fragments across implementations.

**Confidence:** High

---
