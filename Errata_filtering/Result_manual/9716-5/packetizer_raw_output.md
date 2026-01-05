# Errata Reports

Total reports: 2

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