# Errata Reports

Total reports: 2

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