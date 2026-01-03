# Errata Reports

Total reports: 6

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-19-1

**Label:** Underspecified multi‐relay Reconfigure chain reconstruction

**Bug Type:** Underspecification

**Explanation:**

The draft clearly specifies the reverse relay chain for client‐initiated exchanges but only describes server‐initiated Reconfigure via a single relay. This leaves the behavior for reconstructing a full multi‐relay chain ambiguous.

**Justification:**

- The text states that a client’s reply must be relayed through the same chain via nested Relay‐reply messages, yet for server‐initiated Reconfigure only the single‐relay case is detailed.
- There is no normative guidance on how to reconstruct a multi‐hop Relay‐reply chain when two or more relays exist between the client and the server.

**Evidence Snippets:**

- **E1:**

  The document precisely specifies how replies to client‑initiated exchanges must traverse the same chain of relay agents in reverse order, by nesting Relay‑reply messages matching the nested Relay‑forward path, and by requiring the server to record the peer-address fields seen on the way in. In contrast, for server‑initiated Reconfigure, the text only clearly describes the case of sending via a single relay and then explicitly treats the use of relays for Reconfigure as an implementation detail. There is no normative guidance on how to reconstruct a multi-hop Relay‑reply chain for Reconfigure, or even whether a server is expected to do so, in a topology with two or more relays between client and server.

- **E2:**

  Client-side handling: “Upon receipt of a valid Reconfigure message, the client responds with a Renew message, a Rebind message, or an Information-request message … While the transaction is in progress, the client discards any Reconfigure messages it receives.”

**Evidence Summary:**

- (E1) Highlights the lack of normative guidance for multi-relay reconstruction for server-initiated Reconfigure.
- (E2) Indicates that client behavior assumes Reconfigure can arrive at any time, underscoring the ambiguity in multi-relay scenarios.

**Fix Direction:**

Clarify normative guidance by explicitly specifying whether and how a full nested Relay‐reply chain must be reconstructed for server‐initiated Reconfigure in multi‐relay topologies.

**Severity:** Low
  *Basis:* The lack of precise instructions may lead to divergent temporal behaviors in multi‐relay deployments, though basic configuration delivery remains possible.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-19-2

**Label:** Inconsistent normative requirements for relay agent versus LDRA in link-address field

**Bug Type:** Underspecification

**Explanation:**

The draft’s generic relay rule (Section 19.1.1) instructs relays to populate the link-address field with a globally scoped address, while RFC6221 mandates that Lightweight DHCPv6 Relay Agents (LDRAs) must set the link-address to the unspecified address (::) and include an Interface-ID. The exception for LDRAs is not explicitly carved out in the draft.

**Justification:**

- Section 19.1.1 directs a relay to 'place a globally scoped unicast address (i.e., GUA or ULA)' in the link-address field, irrespective of Interface-ID inclusion.
- RFC6221 explicitly requires an LDRA to set the link-address field to :: and to include the Interface-ID option, and Section 13.1 instructs the server to ignore a zero link-address.

**Evidence Snippets:**

- **E1:**

  If the relay agent received the message to be relayed from a client, the relay agent places a globally scoped unicast address (i.e., GUA or ULA) from a prefix assigned to the link… into the link-address field. If such an address is not available, the relay agent may set the link-address field to a link-local address…

- **E2:**

  RFC6221 §6.1: “The LDRA MUST set the link-address field of the Relay-Forward message to the Unspecified Address (::) and MUST include the Interface-ID option in all DHCP Relay-Forward messages.”

**Evidence Summary:**

- (E1) Describes the generic behavior for populating the link-address field.
- (E2) Specifies the LDRA-specific requirement that contradicts the generic rule.

**Fix Direction:**

Modify Section 19.1.1 to explicitly exclude LDRAs by stating that the generic rule applies only to non-LDRA relays, and refer to RFC6221 for LDRA-specific behavior.

**Severity:** Low
  *Basis:* Although servers are expected to handle the LDRA case by ignoring zero-valued link-address, the lack of an explicit exception may confuse implementers and lead to inconsistent relay implementations.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: Issue-1
- ScopeExpert: Issue-1

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-19-3

**Label:** Underspecified server-side algorithm for mapping Relay-forward headers in mixed relay chains

**Bug Type:** Underspecification

**Explanation:**

The draft relies on external RFCs for determining which Relay-forward header field value to use when multiple encapsulated headers are present, leaving it ambiguous how a server prioritizes fields in a mixed relay scenario.

**Justification:**

- The draft states that the link-address may come from any encapsulated Relay-forward message, with the most encapsulated (closest to the client) being most useful, but it does not explicitly define the selection algorithm.
- It further relies on RFC6221 by instructing that a zero link-address must be ignored, which implicitly leaves a gap in specifying the precise precedence when multiple relays are involved.

**Evidence Snippets:**

- **E1:**

  The link-address in this case may come from any of the Relay-forward messages encapsulated in the received Relay-forward, and in general the most encapsulated (closest Relay-forward to the client) has the most useful value.

- **E2:**

  According to [RFC6221], the server MUST ignore any link-address field whose value is zero.

**Evidence Summary:**

- (E1) Indicates the intended use of the most encapsulated link-address but without normative direction.
- (E2) Emphasizes that a zero link-address should be ignored, leaving the selection process underspecified.

**Fix Direction:**

Provide explicit normative guidance in Section 13.1 to define the precedence rules for selecting the appropriate link-address from among multiple Relay-forward headers.

**Severity:** Low
  *Basis:* While external RFCs clarify much of the behavior, the lack of explicit, self-contained instructions may lead to divergent interpretations across implementations.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: NewIssue-1

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-19-4

**Label:** Missing error-handling for Relay-reply with absent Interface-Id and zero link-address

**Bug Type:** Underspecification

**Explanation:**

The specification does not define what a relay should do if it receives a Relay-reply message that contains neither an Interface-Id option nor a non-zero link-address, leaving error-handling behavior ambiguous.

**Justification:**

- Section 19.2 explains that if an Interface-Id is present the message is relayed on that link, or if link-address is non-zero it is used; however, it does not state the required behavior if both conditions fail.
- This gap could lead to inconsistent handling of such malformed messages among implementations.

**Evidence Snippets:**

- **E1:**

  If the Relay-reply message includes an Interface-Id option … the relay agent relays the message … on the link identified by the Interface-Id option. Otherwise, if the link-address field is not set to 0, the relay agent relays the message on the link identified by the link-address field.

- **E2:**

  This leaves one explicit gap: the case where both conditions fail—i.e., the Relay-reply has no Interface-Id option and link-address == 0.

**Evidence Summary:**

- (E1) Describes the two valid branches for processing a Relay-reply.
- (E2) Points out the missing specification for when neither valid field is present.

**Fix Direction:**

Specify that a Relay-reply lacking both a non-zero link-address and an Interface-Id option must be dropped (or handled by a defined error procedure) to ensure deterministic behavior.

**Severity:** Low
  *Basis:* While this case is unlikely in compliant deployments, the absence of explicit error-handling instructions may lead to divergent behaviors if nonconformant messages are encountered.

**Confidence:** High

**Experts mentioning this issue:**

- ScopeExpert: Issue-2
- BoundaryExpert: Finding-1

---

## Report 5: draft-ietf-dhc-rfc8415bis-12-19-5

**Label:** Inconsistent hop-count initialization in Relay-forward messages

**Bug Type:** Inconsistency

**Explanation:**

The definition of hop-count as the number of relays that have relayed the message conflicts with the instruction to set the hop-count to 0 for the first relay, creating an off-by-one discrepancy.

**Justification:**

- The Relay message format defines hop-count as the number of relay agents that have already relayed the message.
- However, Section 19.1.1 prescribes setting hop-count to 0 when the first relay converts a client message, which contradicts the field’s definition.

**Evidence Snippets:**

- **E1:**

  The Relay message format defines: “hop-count: Number of relay agents that have already relayed this message. A 1‑octet field.” (Section 9.1/9.2). When a relay receives a message from a client, “The hop-count value in the Relay-forward message is set to 0.” (Section 19.1.1).

**Evidence Summary:**

- (E1) Illustrates the discrepancy between the formal definition of hop-count and its initialization in the protocol.

**Fix Direction:**

Revise the initialization rule so that the hop-count accurately reflects the number of relay agents that have processed the message, for example by setting it to 1 at the first relay.

**Severity:** Low
  *Basis:* This inconsistency can potentially lead to misinterpretation of the relay chain depth and variations in implementations under complex topologies.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-1

---

## Report 6: draft-ietf-dhc-rfc8415bis-12-19-6

**Label:** HOP_COUNT_LIMIT off-by-one mismatch

**Bug Type:** Inconsistency

**Explanation:**

Although HOP_COUNT_LIMIT is defined as 8, the relay processing rule discards messages when the hop-count is greater than or equal to 8, effectively allowing a maximum hop-count of 7, which conflicts with the description of the constant.

**Justification:**

- Table 1 defines HOP_COUNT_LIMIT as 8, described as the 'Max hop count in a Relay-forward message'.
- Section 19.1.2 states that a Relay-forward message is discarded if its hop-count is greater than or equal to HOP_COUNT_LIMIT, thereby permitting values only up to 7.

**Evidence Snippets:**

- **E1:**

  Table 1 defines “HOP_COUNT_LIMIT | 8 | Max hop count in a Relay-forward message” (Section 7.6).

- **E2:**

  The relay-chain rule says: “If the hop-count value in the message is greater than or equal to HOP_COUNT_LIMIT, the relay agent discards the received message.” (Section 19.1.2)

**Evidence Summary:**

- (E1) Provides the defined value and description of HOP_COUNT_LIMIT.
- (E2) Demonstrates that the operational rule limits accepted hop-count values to 7.

**Fix Direction:**

Either adjust the discarding rule to match the defined maximum of 8 or redefine HOP_COUNT_LIMIT to clearly indicate that 7 is the maximum allowed hop-count value.

**Severity:** Low
  *Basis:* This discrepancy may lead to diverging interpretations of the maximum relay chain depth, potentially affecting interoperability in complex relay deployments.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-2

---
