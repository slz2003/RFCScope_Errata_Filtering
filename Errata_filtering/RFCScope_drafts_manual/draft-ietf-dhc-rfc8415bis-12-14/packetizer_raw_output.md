# Errata Reports

Total reports: 3

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-14-1

**Label:** Underspecified Client Scheduling for Renew/Rebind when T1/T2=0

**Bug Type:** Underspecification

**Explanation:**

The specification leaves the client’s decision for scheduling renew/rebind when T1/T2 are 0 ambiguous, without clear numerical bounds or randomization guidance.

**Justification:**

- The spec requires that when T1 and/or T2 are 0, the client MUST choose a time to avoid message storms and MUST NOT transmit immediately, yet it does not define a minimum delay or tie the timing to valid lifetimes.
- This vagueness can lead to widely differing client behaviors, such as synchronized bursts or excessively delayed renewals, even though they technically meet the requirements.

**Evidence Snippets:**

- **E1:**

  When T1 and/or T2 values are set to 0, the client MUST choose a time to avoid message storms. In particular, it MUST NOT transmit immediately. … The client MUST choose renew and rebind times to not violate rate-limiting restrictions as defined in Section 14.2

- **E2:**

  In a message sent by a server to a client, the client MUST use the values in the T1 and T2 fields for the T1 and T2 times, unless values in those fields are 0. … If the time at which the addresses in an IA_NA are to be renewed is to be left to the discretion of the client, the server sets the T1 and T2 values to 0. The client MUST follow the rules defined in Section 14.2.

**Evidence Summary:**

- (E1) Specifies that when T1/T2 are 0 the client must avoid immediate transmission but leaves the delay undefined.
- (E2) Indicates that the timing for renewal is left to the client without concrete guidelines.

**Fix Direction:**

Consider adding explicit normative timing requirements or recommendations (for example, minimum delay values and randomization guidelines) for clients when T1/T2 are 0.

**Severity:** Medium
  *Basis:* The underspecification may lead to significant variation in client behavior affecting network load and testing reproducibility.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1
- Causal Expert: underspecification

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-14-2

**Label:** Ambiguity in MRD Calculation for Renew with T2=0

**Bug Type:** Underspecification

**Explanation:**

The MRD parameter for Renew messages is defined as the remaining time until the earliest T2, but when T2 is 0 the specification does not indicate how to compute MRD, leading to inconsistent client behavior.

**Justification:**

- When all IA_NA/IA_PD options have T2 set to 0, there is no clear 'earliest T2' from which to calculate MRD.
- This gap forces implementations to choose between treating MRD as zero, infinite, or deriving it from other parameters, potentially causing divergent behavior in renewal retransmission timing.

**Evidence Snippets:**

- **E1:**

  The client transmits the [Renew] message according to Section 15, using the following parameters: … MRD = Remaining time until earliest T2. The message exchange is terminated when the earliest time T2 is reached…

- **E2:**

  If T1 or T2 had been set to 0 by the server … the client may, at its discretion, send a Renew or Rebind message, respectively. The client MUST follow the rules defined in Section 14.2.

**Evidence Summary:**

- (E1) Defines MRD based on the earliest T2 time without accounting for T2=0 cases.
- (E2) Highlights that when T2 is 0 the client operates under discretionary timing, leaving MRD computation ambiguous.

**Fix Direction:**

Explicitly specify how MRD should be computed or applied when T2 is set to 0, such as linking it to the valid lifetime or stating that the time-based cutoff does not apply.

**Severity:** Low
  *Basis:* While the ambiguity could lead to differing client renewal behaviors, it does not fundamentally break protocol correctness.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-14-3

**Label:** Inconsistent Server Identifier Option Semantics

**Bug Type:** Inconsistency

**Explanation:**

Section 14 provides a generic description of the Server Identifier option that conflicts with later message-type-specific rules, potentially leading to misinterpretation by implementers.

**Justification:**

- Section 14 states that omitting the Server Identifier option indicates a broadcast to all servers and including it targets an individual server, but later sections mandate its inclusion or omission based on the message type (e.g., Solicit messages must not include it).
- This scope mismatch can cause devices to send messages incorrectly, for instance, sending a Request without a Server Identifier when the protocol requires it.

**Evidence Snippets:**

- **E1:**

  Section 14: “A client uses multicast to reach all servers or an individual server. An individual server is indicated by specifying that server's DUID in a Server Identifier option (see Section 21.3) in the client's message. (All servers will receive this message, but only the indicated server will respond.) All servers are indicated when this option is not supplied.”

- **E2:**

  Solicit validation (Section 16.2): “Servers MUST discard any Solicit messages that … do include a Server Identifier option.”

**Evidence Summary:**

- (E1) Provides a global, unqualified definition of the Server Identifier option from Section 14.
- (E2) Demonstrates a message-specific rule that contradicts the generic definition by disallowing the option in Solicit messages.

**Fix Direction:**

Clarify the Server Identifier option semantics by explicitly limiting the global description to those message types where the option is permitted and distinguishing it from cases with strict presence or absence requirements.

**Severity:** Medium
  *Basis:* The inconsistency may lead to critical implementation errors, such as incorrect targeting of messages, impacting lease management and server discovery.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---
