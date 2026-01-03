# Errata Reports

Total reports: 4

---

## Report 1: draft-ietf-tls-rfc8446bis-14-2-1

**Label:** Ambiguous Application Data Send Timing in Section 2 vs Section 4.4.4

**Bug Type:** Underspecification

**Explanation:**

Section 2’s overview on when Application Data can be sent may be misinterpreted as sufficient once an endpoint has sent its Finished message, whereas Section 4.4.4 requires both sending the Finished and receiving/validating the peer’s Finished.

**Justification:**

- Section 2 states that Application Data MUST NOT be sent prior to sending the Finished message, presenting it as the single precondition (except for 0‑RTT), which might be read as sufficient.
- Section 4.4.4 clarifies that data may only be sent after an endpoint has sent its own Finished and received and validated the peer’s Finished, with two explicit exceptions.

**Evidence Snippets:**

- **E1:**

  Section 2, after describing the client’s Authentication messages: “At this point, the handshake is complete, and the client and server derive the keying material required by the record layer to exchange application-layer data... Application Data MUST NOT be sent prior to sending the Finished message, except as specified in Section 2.3. Note that while the server may send Application Data prior to receiving the client's Authentication messages, any data sent at that point is, of course, being sent to an unauthenticated peer.”

- **E2:**

  Section 4.4.4 (Finished): “Once a side has sent its Finished message and has received and validated the Finished message from its peer, it may begin to send and receive Application Data over the connection. There are two settings in which it is permitted to send data prior to receiving the peer's Finished: 1. Clients sending 0‑RTT data as described in Section 4.2.10. 2. Servers MAY send data after sending their first flight...”

- **E3:**

  Section 4.4.1 (Transcript Hash / handshake ordering): “Protocol messages MUST be sent in the order defined in Section 4.4.1 and shown in the diagrams in Section 2. A peer which receives a handshake message in an unexpected order MUST abort the handshake with an 'unexpected_message' alert.”

- **E4:**

  Appendix A.1 / A.2 client and server state machines showing that the client receives the server’s Finished before it sends its own Finished, and the server may send application data while still in WAIT_FLIGHT2 (i.e., after its own Finished but before receiving the client’s Finished).

**Evidence Summary:**

- (E1) Section 2 prohibits sending Application Data before sending the Finished message.
- (E2) Section 4.4.4 requires that data transmission only begins after both sending Finished and receiving the peer’s Finished, aside from specific exceptions.
- (E3) Section 4.4.1 reinforces message ordering to ensure proper sequencing.
- (E4) The state machines confirm the intended ordering of these messages.

**Fix Direction:**

Amend Section 2 to reference the complete conditions specified in Section 4.4.4, thereby preventing misinterpretation.

**Severity:** Low
  *Basis:* Although the enforced handshake ordering prevents functional errors, the unclear wording could mislead developers who reference only Section 2.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1

---

## Report 2: draft-ietf-tls-rfc8446bis-14-2-2

**Label:** Conflicting Hash/KDF Provisioning Requirements for External PSKs

**Bug Type:** Inconsistency

**Explanation:**

Section 2.2 demands that externally provisioned PSKs include an explicit KDF hash algorithm, while Section 4.2.11 permits omitting this information by defaulting to SHA‑256, creating a normative conflict.

**Justification:**

- Section 2.2 states: “When PSKs are provisioned externally, the PSK identity and the KDF hash algorithm to be used with the PSK MUST also be provisioned.”
- Section 4.2.11 specifies that for externally established PSKs the Hash algorithm MUST be set or, if not defined, default to SHA‑256.

**Evidence Snippets:**

- **E1:**

  Section 2.2 (Resumption and PSK): “When PSKs are provisioned externally, the PSK identity and the KDF hash algorithm to be used with the PSK MUST also be provisioned.”

- **E2:**

  Section 4.2.11 (Pre‑Shared Key Extension): “Each PSK is associated with a single Hash algorithm. For PSKs established via the ticket mechanism (Section 4.6.1), this is the KDF Hash algorithm on the connection where the ticket was established. For externally established PSKs, the Hash algorithm MUST be set when the PSK is established or default to SHA‑256 if no such algorithm is defined.”

**Evidence Summary:**

- (E1) Section 2.2 requires that the KDF hash algorithm is explicitly provisioned for external PSKs.
- (E2) Section 4.2.11 allows the hash algorithm to default to SHA‑256 if not explicitly provided.

**Fix Direction:**

Revise Section 2.2 to align with Section 4.2.11 by permitting a default value or by uniformly mandating an explicit hash algorithm.

**Severity:** Medium
  *Basis:* The inconsistency alters the allowed configuration space for external PSKs and may confuse implementers regarding normalization of PSK parameters.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---

## Report 3: draft-ietf-tls-rfc8446bis-14-2-3

**Label:** Ambiguous Definition of Handshake Completion in Section 2

**Bug Type:** Ambiguity

**Explanation:**

The wording in Section 2 stating that the handshake is 'complete' immediately after sending the Finished message may be misinterpreted as indicating that key derivation is finalized at that point, even though later sections require both Finished messages to be exchanged and validated.

**Justification:**

- The phrase in Section 2 suggests handshake completion immediately after the client sends its Finished message.
- Later sections, such as Section 4.4.4, imply that a handshake is only effectively complete after both sides have exchanged and validated Finished messages.

**Evidence Snippets:**

- **E1:**

  Section 2, after describing the client’s Authentication messages: “At this point, the handshake is complete, and the client and server derive the keying material required by the record layer to exchange application-layer data... Application Data MUST NOT be sent prior to sending the Finished message, except as specified in Section 2.3.”

**Evidence Summary:**

- (E1) Section 2’s text implies that handshake completion coincides with sending the Finished message, which may lead to confusion regarding the actual timing of key derivation and handshake finalization.

**Fix Direction:**

Clarify in Section 2 that handshake completion occurs only after both parties have exchanged and validated their Finished messages.

**Severity:** Low
  *Basis:* This is primarily a clarity issue that might mislead implementers, despite having no negative impact on interoperability due to enforced message ordering.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1
- Scope Expert: ResidualUncertainties

---

## Report 4: draft-ietf-tls-rfc8446bis-14-2-4

**Label:** Inaccurate 0‑RTT Handshake Overview in Section 2.3 Missing EndOfEarlyData

**Bug Type:** Editorial

**Explanation:**

The overview in Section 2.3 implies that a 0‑RTT handshake uses the same messages as a 1‑RTT handshake with PSK resumption, but it omits mention of the mandatory EndOfEarlyData message, which could mislead implementers.

**Justification:**

- Section 2.3 states that 'the rest of the handshake uses the same messages as for a 1‑RTT handshake with PSK resumption' despite the fact that the 0‑RTT flow requires an extra EndOfEarlyData message.
- Later normative sections (such as Section 4.5) clarify the inclusion of the EndOfEarlyData message, indicating an editorial oversight in the overview.

**Evidence Snippets:**

- **E1:**

  Not a bug: Section 2.3 informally says that “the rest of the handshake uses the same messages as for a 1‑RTT handshake with PSK resumption,” even though in the 0‑RTT case there is an extra EndOfEarlyData message before the client’s Finished.

**Evidence Summary:**

- (E1) The 0‑RTT handshake overview in Section 2.3 omits the EndOfEarlyData message, which is required in the formal 0‑RTT flow.

**Fix Direction:**

Update Section 2.3 to explicitly mention the EndOfEarlyData message in the 0‑RTT handshake flow.

**Severity:** Low
  *Basis:* This is a minor editorial oversight that may confuse readers who rely solely on the overview without consulting later sections.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary Expert: Finding-1

---
