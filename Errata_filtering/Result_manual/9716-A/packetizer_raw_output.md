# Errata Reports

Total reports: 6

---

## Report 1: 9716-A-1

**Label:** Reversed segment order in Reply Path TLV examples in Appendix A

**Bug Type:** Inconsistency

**Explanation:**

Some worked examples in Appendix A list the Reply Path TLV segments in reverse order compared to the normative specification in Sections 5.1 and 5.3.

**Justification:**

- Normative text requires that the first segment corresponds to the top MPLS label (see E1 and E2), yet one example in the P3‐failure scenario shows the order reversed (see E4).
- There is a clear conflict between the correct ordering shown in the non‐failure example (E3) and the reversed ordering in the failure example (E4).

**Evidence Snippets:**

- **E1:**

  Section 5.1: “The Reply Path TLV MUST contain the SR Path in the reverse direction encoded as an ordered list of segments. The first segment MUST correspond to the top segment in the MPLS header that the responder MUST use while sending the echo reply.”

- **E2:**

  Section 5.3: “The top label MUST be constructed from the first segment of the Reply Path TLV. The remaining labels MUST be constructed by following the order of the segments from the Reply Path TLV.”

- **E3:**

  Appendix A.1.2.1, after visiting ASBR4: “the return path beyond ASBR4 will be [N‑ASBR4, EPE‑ASBR4‑ASBR1, N‑PE1].”

- **E4:**

  Appendix A.1.2.1, P3‑failure scenario: “the echo reply is sent using the labels in the Reply Path TLV, which is [N‑PE1, EPE‑ASBR4‑ASBR1, N‑ASBR4].”

**Evidence Summary:**

- (E1) Normative ordering requirement as specified in Section 5.1.
- (E2) Normative rule reiterated in Section 5.3.
- (E3) Correct segment order shown in the non‐failure example in Appendix A.1.2.1.
- (E4) Reversed segment order shown in the P3‐failure scenario.

**Fix Direction:**

Correct the segment order in the Appendix A examples so that the first segment in the Reply Path TLV is used to construct the top MPLS label, in accordance with Sections 5.1 and 5.3.


**Severity:** Medium
  *Basis:* Incorrect label ordering can lead to improper MPLS label stack construction and misrouted echo replies, potentially causing interoperability problems.

**Confidence:** High

---

## Report 2: 9716-A-2

**Label:** Reply Path TLV omission with Reply Mode 5 in echo requests

**Bug Type:** Both

**Explanation:**

The examples imply that implementations may omit the Reply Path TLV for early hops while still using Reply Mode 5, which directly contradicts the MUST‐level requirements in Sections 5.1 and 5.2 and in RFC 7110.

**Justification:**

- Section 5.1 and Section 5.2 clearly mandate that when using Reply Mode 5 the echo request must include a Reply Path TLV (see E5 and E6).
- Appendix A.1.2.1 suggests that the TLV may be excluded until the traceroute reaches the first domain border (see E7), creating a contradiction.

**Evidence Snippets:**

- **E5:**

  Section 5.1: “The LSP ping initiator MUST set the Reply Mode of the echo request to 5 (Reply via Specified Path), and a Reply Path TLV MUST be carried in the echo request message correspondingly.”

- **E6:**

  Section 5.2: “when the Reply Mode is set to 5 (Reply via Specified Path), the echo request must contain the Reply Path TLV. The absence of the Reply Path TLV is treated as a malformed echo request.”

- **E7:**

  Appendix A.1.2.1, traceroute example: “Note that implementations may choose to exclude the Reply Path TLV until the traceroute reaches the first domain border as the return IP path to PE1 is expected to be available inside the first domain.”

**Evidence Summary:**

- (E5) Section 5.1 mandates that the echo request include a Reply Path TLV when using Reply Mode 5.
- (E6) Section 5.2 states that the absence of a Reply Path TLV in Reply Mode 5 is treated as a malformed request.
- (E7) Appendix A.1.2.1 implies that the TLV can be omitted until the first domain border.

**Fix Direction:**

Clarify in the text that if Reply Mode 5 is used, a Reply Path TLV MUST always be included. If TLV omission is desired, the Reply Mode must be adjusted accordingly (e.g. to Reply Modes 2 or 3).


**Severity:** High
  *Basis:* Omitting the TLV while using Reply Mode 5 can result in echo requests being treated as malformed, causing traceroute or ping operations to fail.

**Confidence:** High

---

## Report 3: 9716-A-3

**Label:** Inconsistent TLV inclusion requirements in dynamic return-path echo replies

**Bug Type:** Both

**Explanation:**

There is a discrepancy between the normative mandate that dynamic return-path echo replies include a Reply Path TLV with a Return Code of 0x0006 and examples that allow certain internal nodes to omit the TLV.

**Justification:**

- Section 5.3 requires that when performing dynamic return-path building a TLV (or a new one) must be included with a Return Code of 0x0006 (see E8).
- Section 5.5.1 and Appendix A.1.3 suggest that internal nodes may omit setting the TLV or its return code under certain conditions (see E9 and E10).

**Evidence Snippets:**

- **E8:**

  Section 5.3: “When the node is configured to dynamically create a return path for the next echo request, the procedures described in Section 5.5 MUST be used. The Reply Path Return Code MUST be set to 0x0006, and the same Reply Path TLV or a new Reply Path TLV MUST be included in the echo reply.”

- **E9:**

  Section 5.5.1: “Internal nodes or non-domain border nodes might not set the Reply Path TLV Return Code to 0x0006 in the echo reply message as there is no change in the return path.”

- **E10:**

  Appendix A.1.3: Dynamic example: after P1 returns an echo reply with Return Code 0x0006 and a Reply Path TLV, it says: “This traceroute doesn't need any changes to the Reply Path TLV until it leaves AS1. The same Reply Path TLV that is received may be included in the echo reply by P1 and P2, or no Reply Path TLV is included so that the head-end continues to use the same return path in the echo request.”

**Evidence Summary:**

- (E8) Normative provision from Section 5.3 mandates inclusion of a Reply Path TLV with Return Code 0x0006.
- (E9) Section 5.5.1 notes that internal nodes might not set the return code.
- (E10) Appendix A.1.3 permits omission of the TLV in some echo replies.

**Fix Direction:**

Clarify the dynamic return-path procedure by explicitly specifying which nodes must include the Reply Path TLV and under what conditions omission is acceptable.


**Severity:** High
  *Basis:* The lack of clear guidance can lead to different implementations treating echo replies differently, resulting in interoperability issues.

**Confidence:** High

---

## Report 4: 9716-A-4

**Label:** Ambiguous actor for outgoing interface selection in dynamic return-path building

**Bug Type:** Underspecification

**Explanation:**

The text ambiguously attributes the responsibility for selecting the outgoing interface for the echo reply to both the local ASBR and a 'remote ASBR', causing confusion.

**Justification:**

- The description first states that the ASBR should locally decide the outgoing interface and then refers to a remote ASBR in the decision process (see E11), making it unclear which entity is responsible.

**Evidence Snippets:**

- **E11:**

  ASBR should locally decide the outgoing interface for the echo reply packet. Generally, remote ASBR will choose the interface on which the incoming OAM packet was received to send the echo reply out.

**Evidence Summary:**

- (E11) The text ambiguously refers to both local and remote ASBR for selecting the outgoing interface.

**Fix Direction:**

Revise the text to clearly state that the local ASBR is solely responsible for selecting the outgoing interface based on the incoming OAM packet.


**Severity:** Low
  *Basis:* Although the intended behavior may be inferred, the ambiguity can lead to implementation confusion.

**Confidence:** High

---

## Report 5: 9716-A-5

**Label:** Hybrid MPLS+IP return path example conflicts with Reply Mode 5 TTL semantics

**Bug Type:** Inconsistency

**Explanation:**

The hybrid MPLS+IP return path examples imply that, after MPLS label removal, the echo reply may be forwarded based on its IP header, which contradicts RFC 7110’s requirements for Reply Mode 5.

**Justification:**

- RFC 7110 mandates that in Reply Mode 5 the echo reply’s IP TTL is set to 1 and the destination IP address (typically in the 127/8 range) must not be used for forwarding (see E12).
- In contrast, the hybrid example suggests a return path where, after label popping, IP forwarding is used to reach PE1 (see E13).

**Evidence Snippets:**

- **E12:**

  RFC 7110 Reply Mode 5 requirements: the echo reply’s destination IP address MUST be set to a 127/8 address, the IP Time to Live (TTL) MUST be set to 1, and the destination IP address MUST never be used in a forwarding decision.

- **E13:**

  Appendix A.1.1 hybrid example: “An example return path for this case could be [N‑ASBR4, EPE‑ASBR4‑ASBR1]” which implies that IP forwarding is used after the MPLS labels are removed.

**Evidence Summary:**

- (E12) RFC 7110 specifies that echo replies must have TTL=1 and not use the destination IP for forwarding.
- (E13) The hybrid example in Appendix A implies that IP forwarding occurs after label removal.

**Fix Direction:**

Revise the hybrid MPLS+IP examples to either fully conform to MPLS-only Reply Mode 5 semantics or clearly define the conditions under which IP forwarding is permitted.


**Severity:** Medium
  *Basis:* If implemented as shown, echo replies may be misrouted or dropped because the IP header conditions required for MPLS forwarding would be violated.

**Confidence:** Medium

---

## Report 6: 9716-A-6

**Label:** Ambiguous precedence between A‑Flag and explicit SID in Type‑C/Type‑D segments

**Bug Type:** Underspecification

**Explanation:**

The specification does not clarify which mechanism should be used when both the A‑Flag is set and an explicit SID is provided in Type‑C/Type‑D segments.

**Justification:**

- The text mandates that when the A‑Flag is set the SR Algorithm is used to derive the label, and separately that an explicit SID must be used when present; however, it does not state how to resolve cases where both are provided (see E14).

**Evidence Snippets:**

- **E14:**

  For Type‑C/Type‑D segments: 'When the A‑Flag is present, this specifies the SR Algorithm… The SID field, when present, MUST be used for constructing the Reply Path.' The specification does not state how to proceed when both are provided.

**Evidence Summary:**

- (E14) The text is ambiguous on which value to use when both the A‑Flag (and associated SR Algorithm) and an explicit SID are provided in a Type‑C/Type‑D segment.

**Fix Direction:**

Define clear precedence rules to specify whether the explicit SID should override the algorithm‐derived label or vice versa.


**Severity:** Medium
  *Basis:* This ambiguity may lead to inconsistent label derivation across implementations, particularly in multi‐algorithm environments.

**Confidence:** High

---
