# Errata Reports

Total reports: 3

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