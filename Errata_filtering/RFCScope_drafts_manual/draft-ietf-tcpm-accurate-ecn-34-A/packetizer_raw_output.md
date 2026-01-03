# Errata Reports

Total reports: 8

---

## Report 1: draft-ietf-tcpm-accurate-ecn-34-A-1

**Label:** LRO/ACK Aggregation vs n ≤ 7 CE Marks Bound (Temporal T1)

**Bug Type:** Both

**Explanation:**

The specification contains a tension between the normative rule that ‘n’ (the number of CE‐marked packets between ACKs) MUST be no greater than 7 and the allowance for LRO/GRO to coalesce many packets into a single ACK, which may allow more than 7 CE marks and compromise the sender’s decoding algorithm.

**Justification:**

- The text mandates that 'n MUST be no greater than 7' to prevent ambiguity (Section 3.2.2.5.1) but also permits LRO/GRO processing that can delay multiple CE marks to a single ACK (Section 3.2.2.5.1).
- This inconsistency creates implementation tension between performance optimizations and maintaining a reliable 3‐bit wrap bound.

**Evidence Snippets:**

- **E1:**

  Increment-Triggered ACKs: An AccECN receiver of a packet MUST emit an ACK if 'n' CE marks have arrived since the previous ACK. … In either case, 'n' MUST be no greater than 7. (Section 3.2.2.5.1)

- **E2:**

  If the arrivals of a number of data packets are all processed as one event, e.g., using large receive offload (LRO) or generic receive offload (GRO), both the above rules SHOULD be interpreted as requiring multiple ACKs to be emitted back-to-back … If this is problematic for high performance, either rule can be interpreted as requiring just a single ACK at the end of the whole receive event. (Section 3.2.2.5.1)

- **E3:**

  The 3-bit ACE field can wrap fairly frequently. … The Data Receiver is not allowed to delay sending an ACK to such an extent that the ACE field would cycle. (Section 2.3)

**Evidence Summary:**

- (E1) establishes the strict upper bound of 7 CE marks per ACK.
- (E2) shows that LRO/GRO may allow a single ACK to summarize many more CE-marked packets.
- (E3) underlines the intent to prevent the ACE field from wrapping.

**Fix Direction:**

Clarify or redefine the LRO/GRO guidance to either require multiple ACKs when more than 7 CE marks occur or adjust the bound for aggregated receive events so that the sender’s decoding remains unambiguous.

**Severity:** Medium
  *Basis:* The inconsistency can lead to divergent sender interpretations of CE count in high-aggregation scenarios, impacting congestion feedback accuracy.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T1

---

## Report 2: draft-ietf-tcpm-accurate-ecn-34-A-2

**Label:** Late/Duplicate Handshake ACKs vs ACE Handshake Encoding (Temporal T2)

**Bug Type:** Underspecification

**Explanation:**

The specification’s treatment of handshake ACKs is ambiguous because it relies on an informally defined notion of 'superseded' ACKs, which may lead to misinterpretation when a late or duplicate handshake ACK (using handshake encoding) is received after the server transitions to ESTABLISHED.

**Justification:**

- The text specifies that handshake encoding is only valid while the server is in SYN-RCVD state, yet also permits the possibility of receiving handshake-encoded pure ACKs post-ESTABLISHED.
- The reliance on a loosely defined 'superseded' criterion introduces ambiguity in whether later ACKs are processed or ignored.

**Evidence Snippets:**

- **E1:**

  This shall be called the handshake encoding of the ACE field, and it is the only exception to the rule that the ACE field carries the 3 least significant bits of the r.cep counter on packets with SYN=0. (Section 3.2.2.1)

- **E2:**

  Once the TCP Server transitions to ESTABLISHED state, it might later receive other pure ACK(s) with the handshake encoding… A Server MAY implement a test for such a case, but it is not required. (Section 3.2.2.2)

- **E3:**

  Whenever the Data Sender receives an ACK with SYN=0 … it first checks whether it has already been superseded (defined in Appendix A.1) by another ACK in which case it ignores the ECN feedback. (Section 3.2.2.2)

**Evidence Summary:**

- (E1) defines the handshake encoding exception for ACKs in SYN-RCVD state.
- (E2) notes that handshake encoding may be received after the server enters ESTABLISHED.
- (E3) shows that the concept of 'superseded' ACKs is used to ignore later feedback, but is informally defined.

**Fix Direction:**

Either establish normative, precise criteria for when a handshake ACK is considered 'superseded' or explicitly mandate that handshake-encoded ACE values be ignored after ESTABLISHED state.

**Severity:** Low
  *Basis:* The risk is mainly limited to transient misalignment in the CE counter interpretation and does not impact connection establishment or wire-level interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T2

---

## Report 3: draft-ietf-tcpm-accurate-ecn-34-A-3

**Label:** Irreversible 'Ignore AccECN Options' Mode After Zeroing Detection (Temporal T3)

**Bug Type:** Underspecification

**Explanation:**

The document defines two modes regarding AccECN Options—one reversible when options are absent and one irreversible when options are zeroed—but does not clarify if or when a host can exit the 'ignore options' mode triggered by zeroing.

**Justification:**

- The text describes a reversible mode when options are assumed absent but provides no exit condition for the mode that ignores options after zero detection.
- This inconsistency leaves implementers uncertain if a recovered option stream should be reintegrated into normal feedback processing.

**Evidence Snippets:**

- **E1:**

  If it runs a test and either initial value is zero, the Server will switch into a mode that ignores AccECN Options for this half connection. (Server case, Section 3.2.3.2.4)

- **E2:**

  While a host is in the mode that ignores AccECN Options it MUST adopt the conservative interpretation of the ACE field discussed in Section 3.2.2.5. (Section 3.2.3.2.4)

- **E3:**

  If a host is in the mode that assumes incoming AccECN Options are not available, but it receives an AccECN Option at any later point during the connection… (Section 3.2.3.2.3)

**Evidence Summary:**

- (E1) shows the irreversible switch into an 'ignore options' mode upon detecting zeroed options.
- (E2) mandates conservative interpretation once in that mode.
- (E3) contrasts with the reversible 'assume not available' mode, highlighting the ambiguity.

**Fix Direction:**

Clarify whether the 'ignore options' mode is permanent or provide normative criteria for reverting to normal behavior when valid AccECN Options are subsequently received.

**Severity:** Low
  *Basis:* The ambiguity affects the long-term behavior of the AccECN feedback mechanism in scenarios where path conditions change, though it does not prevent correct negotiation.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T3

---

## Report 4: draft-ietf-tcpm-accurate-ecn-34-A-4

**Label:** Contradiction Between General Feedback Obligation and Specific Handshake Exception (Deontic Issue-1)

**Bug Type:** Inconsistency

**Explanation:**

The specification mandates that a host in AccECN mode is generally obliged to respond to ECN feedback; however, it also explicitly instructs a server in SYN-RCVD state receiving a handshake ACK with an ACE value of zero to never respond to ECN feedback, resulting in a normative contradiction.

**Justification:**

- Section 3.1.5 states that an AccECN-mode host is obliged to respond appropriately to congestion feedback.
- In contrast, Section 3.2.2.1 (Note 1) specifies that if a server in SYN-RCVD receives a zero value on a pure ACK, it MUST NOT respond to AccECN feedback for the rest of the connection.

**Evidence Snippets:**

- **E1:**

  A host in AccECN mode: - is obliged to respond appropriately to AccECN feedback that indicates there were ECN marks on packets it had previously sent… (Section 3.1.5)

- **E2:**

  If the Server is in AccECN mode and in SYN-RCVD state, and if it receives a value of zero on a pure ACK with SYN=0 and no SACK blocks, for the rest of the connection the Server MUST NOT set ECT on outgoing packets and MUST NOT respond to AccECN feedback. (Section 3.2.2.1, Note 1)

**Evidence Summary:**

- (E1) lays out a general obligation to react to ECN feedback.
- (E2) creates an exception by mandating no response when a specific handshake condition occurs.

**Fix Direction:**

Modify the general obligation to clearly carve out the handshake ACE=0 case as a normative exception, or remove the contradiction by rephrasing one of the statements.

**Severity:** Medium
  *Basis:* The contradiction could lead to divergent implementations, where some servers might respond to CE feedback and others might not, affecting congestion control behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic: Issue-1

---

## Report 5: draft-ietf-tcpm-accurate-ecn-34-A-5

**Label:** Ambiguous Normative Status for Sender Behavior Under Suspected ECN Mangling (Deontic Issue-2)

**Bug Type:** Underspecification

**Explanation:**

The document offers only non-normative advice on how the Data Sender should behave when ECN mangling or ACE/AccECN-option zeroing is suspected, resulting in an ambiguous normative standard that may lead to divergent congestion control responses.

**Justification:**

- The text advises that the Data Sender 'ought to send non-ECN-capable packets' and 'is advised not to respond to any feedback of CE markings' upon detecting continuous CE-marking or a zero ACE field.
- However, the general normative obligation still requires reaction to congestion feedback, creating uncertainty about which behavior is compliant.

**Evidence Snippets:**

- **E1:**

  A host in AccECN mode: … is still obliged to respond appropriately to congestion feedback, even when it is solely sending non-ECN-capable packets. (Section 3.1.5)

- **E2:**

  If continuous CE-marking is detected, for the remainder of the half-connection, the Data Sender ought to send non-ECN-capable packets and it is advised not to respond to any feedback of CE markings. … This advice is not stated normatively. (Section 3.2.2.3)

- **E3:**

  If the first post-handshake ACE is zero, the Data Sender … ought to send non-ECN-capable packets and it is advised not to respond to any feedback of CE markings. … This advice is not stated normatively. (Section 3.2.2.4)

**Evidence Summary:**

- (E1) establishes the general obligation to respond to congestion feedback.
- (E2) and (E3) provide non-normative advice for handling suspected mangling, resulting in ambiguity.

**Fix Direction:**

Define a clear, normative fallback behavior for the Data Sender when suspected ECN mangling or option zeroing is detected, reducing the discretion left to implementers.

**Severity:** Medium
  *Basis:* This ambiguity may lead to significant differences in congestion response under adversarial or degraded network conditions, undermining the intended security and robustness.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic: Issue-2

---

## Report 6: draft-ietf-tcpm-accurate-ecn-34-A-6

**Label:** Undefined Receiver Counters 'r.ec0b' Versus Defined 'r.e0b' (Structural/Terminology Issue)

**Bug Type:** Inconsistency

**Explanation:**

The specification inconsistently refers to the receiver’s byte counters using the names 'r.ec0b' and 'r.ec1b' in one section while elsewhere the defined names are 'r.e0b' and 'r.e1b', potentially confusing implementers.

**Justification:**

- Section 3.2 defines the receiver counters as r.e0b and r.e1b, but later in Section 3.2.3.3 the text refers to r.ec0b and r.ec1b.
- This discrepancy is also noted in the Terminology analysis, where the misnaming could lead to uncertainty regarding which counters are to be used.

**Evidence Snippets:**

- **E1:**

  Each Data Receiver of each half connection maintains four counters, r.cep, r.ceb, r.e0b and r.e1b. (Section 3.2)

- **E2:**

  It SHOULD always include an AccECN Option if the r.ceb counter is incrementing and it MAY include an AccECN Option if r.ec0b or r.ec1b is incrementing. (Section 3.2.3.3)

**Evidence Summary:**

- (E1) specifies the defined counter names as r.e0b and r.e1b.
- (E2) shows the inconsistent use of r.ec0b and r.ec1b in a later section.

**Fix Direction:**

Replace the incorrect references 'r.ec0b' and 'r.ec1b' with the defined 'r.e0b' and 'r.e1b' in Section 3.2.3.3.

**Severity:** Medium
  *Basis:* Inconsistent naming in normative text can lead to confusion in implementation and misinterpretation of the protocol state.

**Confidence:** High

**Experts mentioning this issue:**

- Structural: Issue-1
- Terminology: Issue-1

---

## Report 7: draft-ietf-tcpm-accurate-ecn-34-A-7

**Label:** Misnamed Option Field 'EEB0' Versus Defined 'EE0B' (Terminology Issue)

**Bug Type:** Inconsistency

**Explanation:**

An example in the document misnames an option field as 'EEB0' instead of the defined and consistently used 'EE0B', which could confuse readers and implementers.

**Justification:**

- Figures, tables, and normative text define the option field as 'EE0B', yet an example in Section 3.2.3.3 incorrectly refers to it as 'EEB0'.
- Such a typographical error in an example may lead to uncertainty when correlating field names with the official definitions.

**Evidence Snippets:**

- **E1:**

  As a second example, if the first packet to arrive happens to be CE-marked, the Data Receiver will have to arbitrarily choose whether to precede the ECEB field with an EE0B field or an EE1B field.  If it chooses, say, EEB0 but it turns out never to receive ECT(0), it can start sending EE1B and ECEB instead… (Section 3.2.3.3)

**Evidence Summary:**

- (E1) shows the erroneous use of 'EEB0' instead of the defined 'EE0B'.

**Fix Direction:**

Replace 'EEB0' with 'EE0B' in the example text of Section 3.2.3.3.

**Severity:** Low
  *Basis:* This appears to be a minor typographical error in an example and is unlikely to affect interoperable behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Terminology: Issue-2

---

## Report 8: draft-ietf-tcpm-accurate-ecn-34-A-8

**Label:** Mis-citing RFC 3168’s Definition of 'Invalid' ECN Transitions (CrossRFC Issue)

**Bug Type:** Inconsistency

**Explanation:**

The document asserts that the invalid transitions of the IP-ECN field are defined in RFC 3168 Section 18 and repeats them verbatim, but the list does not precisely match the detailed analysis found in RFC 3168, risking confusion for readers cross-referencing the RFC.

**Justification:**

- Section 3.2.2.3 lists three invalid transitions as a repetition from RFC 3168, but RFC 3168’s treatment is more nuanced and subdivided into multiple classes.
- This mis-citation could mislead implementers expecting an exact reproduction of the RFC’s classification.

**Evidence Snippets:**

- **E1:**

  Invalid transitions of the IP-ECN field are defined in section 18 of the Classic ECN specification [RFC3168] and repeated here for convenience: * the not-ECT codepoint changes; * either ECT codepoint transitions to not-ECT; * the CE codepoint changes. (Section 3.2.2.3)

**Evidence Summary:**

- (E1) shows the claimed repetition of RFC 3168’s invalid transitions, which does not fully capture the nuance of the original specification.

**Fix Direction:**

Clarify that the list is an editorial summary and, if necessary, adjust the wording to accurately reflect the original treatment in RFC 3168.

**Severity:** Low
  *Basis:* This issue is primarily editorial and is unlikely to impact actual protocol behavior or interoperability.

**Confidence:** Medium

**Experts mentioning this issue:**

- CrossRFC: Issue-1

---
