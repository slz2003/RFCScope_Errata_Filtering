# Errata Reports

Total reports: 5

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-18-1

**Label:** Temporal Inconsistency in Unicast Handling with Obsoleted Server Unicast/UseMulticast Flows

**Bug Type:** Inconsistency

**Explanation:**

The draft retains a unicast reception procedure that depends on obsolete mechanisms (the Server Unicast option and UseMulticast status code) while simultaneously declaring these mechanisms obsoleted, leading to conflicting normative instructions.

**Justification:**

- The text states that the UseMulticast status code is obsolete and clients should no longer receive it, yet Section 18.4 mandates that servers respond to unicast messages with UseMulticast.
- This contradiction creates two incompatible flows for handling unicast messages, causing ambiguity in client/server behavior.

**Evidence Snippets:**

- **E1:**

  “The UseMulticast status code has been obsoleted and hence clients should no longer send messages to a server's unicast address nor receive the UseMulticast status code.” (Section 16, Message Validation)

- **E2:**

  “If the relay agent … When the server receives a message via unicast from a client to which the server has not sent a Server Unicast option … [it] responds with an Advertise … or Reply message … containing a Status Code option with the value UseMulticast … and no other options.” (Section 18.4, Reception of Unicast Messages)

- **E3:**

  The older 18.2.10 behavior for UseMulticast (client resends original message using multicast) in the referenced RFC 8415 text.

**Evidence Summary:**

- (E1) Section 16 specifies that UseMulticast is obsolete.
- (E2) Section 18.4 still instructs servers to use UseMulticast in responses.
- (E3) The legacy mechanism from RFC 8415 is referenced, highlighting the removed client behavior.

**Severity:** High
  *Basis:* The contradictory instructions may lead to stalled configurations or inconsistent implementations, risking interoperability in corner cases.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-18-2

**Label:** Underspecification in Reconfigure-triggered Renew/Rebind Behavior with T1/T2=0

**Bug Type:** Underspecification

**Explanation:**

The specification is ambiguous about whether Reconfigure-triggered exchanges must adhere to the non-immediate transmission rules for T1/T2=0, resulting in potential discrepancies in client behavior response times.

**Justification:**

- Section 14.2 mandates that when T1 and/or T2 are set to 0, the client MUST not transmit immediately to avoid message storms.
- However, Section 18.2/18.2.11 implies that upon receiving a Reconfigure, a client should respond immediately, creating uncertainty over which rule to apply.

**Evidence Snippets:**

- **E1:**

  “When T1 and/or T2 values are set to 0, the client MUST choose a time to avoid message storms. In particular, it MUST NOT transmit immediately.” (Section 14.2)

- **E2:**

  “Upon receipt of a Reconfigure message… the client responds with a Renew, Rebind, or Information-request message… The client SHOULD treat the Reconfigure message as if the T1 timer had expired.” (Sections 18.2 and 18.2.11)

- **E3:**

  T1/T2=0 semantics in IA_NA and IA_PD options pointing back to Section 14.2.

**Evidence Summary:**

- (E1) Section 14.2 requires delaying transmission when T1/T2 are 0.
- (E2) Sections 18.2 and 18.2.11 state that Reconfigure-triggered responses should behave as if T1 expired.
- (E3) References from IA_NA/IA_PD further link T1/T2=0 behavior to Section 14.2.

**Severity:** Medium
  *Basis:* Inconsistent interpretation may lead to varied client response times, potentially affecting how rapidly configuration changes propagate across the network.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-18-3

**Label:** Ambiguity in Rebind Timer Profiles for Movement Detection with Delegated Prefixes

**Bug Type:** Underspecification

**Explanation:**

The document is unclear about which retransmission parameters should be used when a client with delegated prefixes moves to a new link, mixing T2-driven and confirm-based timer values.

**Justification:**

- Section 18.2.5 describes a standard Rebind procedure using REB_TIMEOUT and REB_MAX_RT, while also noting an alternative for delegated prefixes.
- Section 18.2.12 mandates that when delegated prefixes are present, the Rebind should use confirm-like timers, but does not consolidate the two uses clearly.

**Evidence Snippets:**

- **E1:**

  “At time T2 … the client initiates a Rebind/Reply message exchange … A Rebind is also used to verify delegated prefix bindings but with different retransmission parameters as described in Section 18.2.3.” (Section 18.2.5)

- **E2:**

  “If the client has any valid delegated prefixes obtained from the DHCP server, the client MUST initiate a Rebind/Reply message exchange as described in Section 18.2.5, with the exception that the retransmission parameters should be set as for the Confirm message (see Section 18.2.3).” (Section 18.2.12)

- **E3:**

  Confirm timers (CNF_TIMEOUT, CNF_MAX_RT, CNF_MAX_RD) and their usage in Section 18.2.3.

**Evidence Summary:**

- (E1) Section 18.2.5 defines a standard Rebind with one set of timers.
- (E2) Section 18.2.12 specifies a different timer profile for delegated prefixes.
- (E3) The reference to confirm timers in Section 18.2.3 underscores the intended difference.

**Severity:** Low
  *Basis:* While the ambiguity may cause slower movement detection and recovery, it is unlikely to result in outright interoperability failure.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T3

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-18-4

**Label:** Release Procedure Message Mislabeling: 'Renew message' instead of 'Release message'

**Bug Type:** Inconsistency

**Explanation:**

In the section detailing Release messages, the requirements erroneously refer to the 'Renew message' when specifying the inclusion of the Server Identifier, leading to potential confusion over the correct message format.

**Justification:**

- Section 18.2.7, intended for Release message construction, correctly states the message type is RELEASE but then incorrectly instructs the client to include a Server Identifier option in the Renew message.
- Comparison with the Renew section (18.2.4) and corroboration from multiple experts indicate this is a copy‑and‑paste error that mislabels the target message type.

**Evidence Snippets:**

- **E1:**

  Section 18.2.7: “To release one or more leases, a client sends a Release message to the server.” along with “The client sets the ‘msg-type’ field to RELEASE.”

- **E2:**

  Section 18.2.7: “The client MUST include a Server Identifier option (see Section 21.3) in the Renew message, identifying the server which allocated the lease(s).”

- **E3:**

  Section 18.2.4 (Renew section) correctly states: “The client MUST include a Server Identifier option (see Section 21.3) in the Renew message, identifying the server with which the client most recently communicated.”

**Evidence Summary:**

- (E1) Section 18.2.7 defines the Release message but later refers incorrectly to the Renew message.
- (E2) The misstatement instructs inclusion of the Server Identifier in the Renew message within a section dedicated to Release.
- (E3) The contrasting correct formulation in Section 18.2.4 highlights the inconsistency.

**Fix Direction:**

Replace 'Renew message' with 'Release message' in Section 18.2.7 to align with the intended message type.

**Severity:** Medium
  *Basis:* Mislabelling in normative text risks misinterpretation of client behavior regarding message construction, which is critical for correct protocol implementation.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: Issue-1
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- Terminology Expert: Issue-1

---

## Report 5: draft-ietf-dhc-rfc8415bis-12-18-5

**Label:** Decline Message Processing Actor Error: Incorrectly Naming 'client includes' Instead of 'server includes'

**Bug Type:** Inconsistency

**Explanation:**

The specification for processing Decline messages erroneously states that the client includes certain options in the server-generated Reply, creating a normative contradiction regarding which entity is responsible for the option population.

**Justification:**

- The text in Section 18.3.8 describes server behavior in response to a Decline, yet it incorrectly uses the phrase 'the client includes' when detailing the options to be added to the Reply message.
- Comparisons with the analogous release processing text (Section 18.3.7) and clear contextual cues confirm that the server should, in fact, include the options.

**Evidence Snippets:**

- **E1:**

  Section 18.3.8: “After all the addresses have been processed, the server generates a Reply message by setting the ‘msg-type’ field to REPLY and copying the transaction ID from the Decline message into the ‘transaction-id’ field.  The client includes a Status Code option (see Section 21.13) with the value Success, a Server Identifier option (see Section 21.3) with the server's DUID, and a Client Identifier option (see Section 21.2) with the client's DUID.”

- **E2:**

  Section 18.3.7 (for Release messages) correctly uses “the server includes” when describing option inclusion in the Reply message.

**Evidence Summary:**

- (E1) Section 18.3.8 misstates the actor as ‘the client’ when the server generates the Reply.
- (E2) The correct phrasing in Section 18.3.7 reinforces that the server is responsible for including the options.

**Fix Direction:**

Change 'the client includes' to 'the server includes' in Section 18.3.8.

**Severity:** Medium
  *Basis:* The mistaken attribution of option inclusion to the client can confuse implementers regarding the responsibilities of client and server, potentially leading to implementation errors.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: Issue-2
- Structural Expert: Issue-2
- Terminology Expert: Issue-2

---
