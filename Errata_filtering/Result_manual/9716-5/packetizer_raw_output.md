# Errata Reports

Total reports: 6

---

## Report 1: 9716-5-1

**Label:** Missing Reset of Reply Path Return Code When Reusing TLV in Next Echo Request

**Bug Type:** Underspecification

**Explanation:**

When the initiator copies the Reply Path TLV from an echo reply into the next echo request, the specification does not explicitly require that the internal Reply Path return code field be reset to zero as mandated by RFC 7110.

**Justification:**

- RFC 7110 clearly requires the Reply Path return code to be set to zero in any echo request.
- RFC 9716 instructs the initiator to reuse the TLV without explicitly stating that the return code must be cleared.

**Evidence Snippets:**

- **E1:**

  “The Reply Path return code field… MUST be set to zero in a Reply Path TLV carried on an echo request message and MUST be ignored on receipt.” (RFC 7110 §4.2)

- **E2:**

  “if the Reply Path Return Code is ‘Use Reply Path TLV from this echo reply for building the next echo request’ … the Reply Path TLV from the echo reply MUST be sent in the next echo request with the TTL incremented by 1.” (RFC 9716 §5.4)

**Evidence Summary:**

- (E1) mandates that echo requests include a zeroed Reply Path return code.
- (E2) instructs reusing the TLV without mentioning the need to reset the field.

**Fix Direction:**

Add an explicit normative requirement that when the Reply Path TLV is reused for an echo request, its internal Reply Path return code field MUST be set to 0x0000.


**Severity:** Medium
  *Basis:* Non‐conformance with RFC 7110 may lead to inconsistent behavior or confusion during conformance testing.

**Confidence:** High

---

## Report 2: 9716-5-2

**Label:** Inconsistent Inclusion of Reply Path TLV in Echo Replies

**Bug Type:** Inconsistency

**Explanation:**

There is a divergence between the mandatory inclusion of the Reply Path TLV in echo replies as required by RFC 7110 and RFC 9716’s examples that permit its omission.

**Justification:**

- RFC 7110 mandates that if a Reply Path TLV appears in an echo request, it MUST be echoed in the corresponding reply.
- RFC 9716 Appendix A.1.3 shows examples where responders may omit the TLV.

**Evidence Snippets:**

- **E1:**

  “if a Reply Path (RP) TLV is included in an echo request message, a Reply Path (RP) TLV MUST be included in the corresponding echo reply message sent by an implementation that is conformant to this specification.” (RFC 7110 §4.2)

- **E2:**

  “The same Reply Path TLV that is received may be included in the echo reply by P1 and P2, or no Reply Path TLV is included so that the head-end continues to use the same return path in the echo request that it used to send the previous echo request.” (RFC 9716 Appendix A.1.3, dynamic Reply Path TLV example)

**Evidence Summary:**

- (E1) establishes that a TLV must be echoed if present in the request.
- (E2) presents an example where the TLV is omitted to reuse an old path.

**Fix Direction:**

Clarify the normative text to require that an echo reply MUST include a Reply Path TLV whenever it was present in the echo request, and adjust the examples accordingly.


**Severity:** Medium
  *Basis:* The inconsistent treatment may lead to divergent implementations and confusion in diagnostic tools relying on a one‐to‐one correspondence.

**Confidence:** High

---

## Report 3: 9716-5-3

**Label:** Ambiguous Terminology: 'Return Code' Versus 'Reply Path Return Code'

**Bug Type:** Both

**Explanation:**

The document inconsistently uses the unqualified term 'Return Code' for values 0x0006/0x0007, which are intended solely for the Reply Path TLV’s internal field, potentially confusing them with the MPLS echo header Return Code defined in RFC 8029.

**Justification:**

- Some sections explicitly refer to 'Reply Path Return Code' while others drop the qualifier.
- The unqualified usage risks misinterpreting the new codes as part of the header Return Code, where value 7 is reserved.

**Evidence Snippets:**

- **E1:**

  “The Reply Path Return Code is set as described in Section 7.4 of [RFC7110]… When the node is configured to dynamically create a return path for the next echo request, the procedures described in Section 5.5 MUST be used. The Reply Path Return Code MUST be set to 0x0006, and the same Reply Path TLV or a new Reply Path TLV MUST be included in the echo reply.” (RFC 9716 §5.3)

- **E2:**

  “If the initiator node does not support the Return Code ‘Use Reply Path TLV from this echo reply for building the next echo request’…” (RFC 9716 §5.4)

**Evidence Summary:**

- (E1) assigns 0x0006 specifically as a Reply Path Return Code.
- (E2) uses the unqualified term 'Return Code', creating ambiguity with the header field.

**Fix Direction:**

Revise the document to consistently use 'Reply Path Return Code' when discussing 0x0006/0x0007 and explicitly state that these codes apply only to the TLV’s internal field, not to the MPLS echo header Return Code.


**Severity:** Medium
  *Basis:* This ambiguity can lead to implementations misusing reserved header values and adversely affecting interoperability.

**Confidence:** High

---

## Report 4: 9716-5-4

**Label:** Unclear TTL Increment Reference in Dynamic Traceroute

**Bug Type:** Underspecification

**Explanation:**

The instruction to increment 'the TTL' in dynamic traceroute is ambiguous, leaving it unclear whether the MPLS header TTL or the TTL in Type‑A segment sub‑TLVs should be incremented.

**Justification:**

- RFC 9716 §5.4 calls for the TLV to be sent with the TTL incremented by 1 without clarifying which TTL is subject to change.
- RFC 8029 and the Type‑A sub‑TLV definition provide different contexts for TTL usage.

**Evidence Snippets:**

- **E1:**

  “if the Reply Path Return Code is ‘Use Reply Path TLV from this echo reply for building the next echo request’ … the Reply Path TLV from the echo reply MUST be sent in the next echo request with the TTL incremented by 1. … If the TTL is already 255, the traceroute procedure MUST be ended with an appropriate log message.” (RFC 9716 §5.4)

- **E2:**

  “TTL: 1 octet of TTL. If the originator wants the receiver to choose the TTL value, it MUST set the TTL field to 255.” (RFC 9716 §4.1, Type‑A Segment sub‑TLV)

**Evidence Summary:**

- (E1) mandates a TTL increment but does not specify its target.
- (E2) defines a TTL within a sub‑TLV, contributing to the ambiguity.

**Fix Direction:**

Clarify that the TTL increment applies exclusively to the outer MPLS echo request TTL as defined in RFC 8029.


**Severity:** Low
  *Basis:* While unlikely to cause major interoperation failures, the ambiguity may lead to minor misinterpretations of the probe progression.

**Confidence:** High

---

## Report 5: 9716-5-5

**Label:** Undefined Behavior for Empty Reply Path TLV

**Bug Type:** Underspecification

**Explanation:**

RFC 9716 requires that the Reply Path TLV carry the SR path as an ordered list of segments but does not define the behavior when this TLV contains zero segment sub‑TLVs, even though RFC 7110 permits an empty list.

**Justification:**

- RFC 9716 mandates that the Reply Path TLV include at least one segment to construct the echo reply’s MPLS header.
- RFC 7110 allows for a Reply Path TLV with zero, one, or more sub‑TLVs, creating a boundary condition that is not resolved by RFC 9716.

**Evidence Snippets:**

- **E1:**

  The Reply Path TLV MUST contain the SR Path in the reverse direction encoded as an ordered list of segments. The first segment MUST correspond to the top segment in the MPLS header that the responder MUST use while sending the echo reply. (Section 5.1 and 5.3)

- **E2:**

  Reply Path: … can contain zero, one or more Target FEC sub‑TLVs. (RFC 7110 Section 4.2)

**Evidence Summary:**

- (E1) implies that a non‑empty segment list is expected for proper operation.
- (E2) indicates that an empty list is allowed, creating an undefined boundary condition.

**Fix Direction:**

Define that for Reply Mode 5 the Reply Path TLV MUST contain at least one segment, or explicitly describe the expected behavior if the TLV is empty.


**Severity:** Medium
  *Basis:* This underspecification may lead to divergent behaviors in implementations affecting OAM diagnostics.

**Confidence:** Medium

---

## Report 6: 9716-5-6

**Label:** Conflicting Rules for Dynamic Return Path on Unusable SR Path

**Bug Type:** Inconsistency

**Explanation:**

There is a conflict between the requirement to set the Reply Path Return Code to 0x0006 for dynamic return path building and the alternative instruction to send an error code when the top label is unreachable.

**Justification:**

- RFC 9716 mandates that nodes configured for dynamic return path construction set the Reply Path Return Code to 0x0006.
- However, it also advises that if the top label is unreachable, the responder SHOULD send an appropriate error code per RFC 7110, creating contradictory instructions.

**Evidence Snippets:**

- **E1:**

  The responder MAY check the reachability of the top label in its own Label Forwarding Information Base (LFIB) before sending the echo reply. If the top label is unreachable, the responder SHOULD send the appropriate Return Code and follow the procedures as per Section 5.2 of [RFC7110]. (RFC 9716 Section 5.3)

- **E2:**

  When the node is configured to dynamically create a return path for the next echo request, the procedures described in Section 5.5 MUST be used. The Reply Path Return Code MUST be set to 0x0006, and the same Reply Path TLV or a new Reply Path TLV MUST be included in the echo reply. (RFC 9716 Section 5.3)

**Evidence Summary:**

- (E1) instructs an alternative error handling when the top label is unreachable.
- (E2) unconditionally mandates the use of Reply Path Return Code 0x0006 for dynamic path building.

**Fix Direction:**

Clarify the normative behavior for the scenario when the dynamic return path is enabled but the specified SR path is unreachable, indicating which rule takes precedence.


**Severity:** Medium
  *Basis:* This conflict may cause implementations to diverge in handling failure scenarios, impairing diagnostic accuracy.

**Confidence:** Medium

---
