# Errata Reports

Total reports: 4

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-7-1

**Label:** Conflicting UDP Destination Port Requirements Between Section 7.2 and RFC 8357

**Bug Type:** Normative Conflict / Underspecification

**Explanation:**

Section 7.2 mandates that servers and relay agents send Relay-reply messages to UDP destination port 547, but RFC 8357 requires that, when the Relay Source Port Option is in use, replies must be sent to the source port recorded from the incoming packet, leading to contradictory obligations.

**Justification:**

- Section 7.2 unconditionally requires servers and relay agents to send Relay-reply messages to port 547.
- RFC 8357 explicitly mandates that servers use the UDP source port from the incoming Relay-forward message when applicable.
- The draft only briefly notes that RFC 8357 'changes some of these rules,' without explicitly qualifying the destination port requirement.

**Evidence Snippets:**

- **E1:**

  Clients MUST listen for DHCP messages on UDP port 546. Servers and relay agents MUST listen for DHCP messages on UDP port 547. Therefore, clients MUST send DHCP messages to UDP destination port 547. Servers MUST send Relay-reply messages to UDP destination port 547 and client messages to UDP destination port 546. Relay agents MUST send Relay-forward and Relay-reply messages to UDP destination port 547 and client messages to UDP destination port 546.

- **E2:**

  Please note that the Relay Source Port Option [RFC8357] changes some of these rules for servers and relays agents.

- **E3:**

  When sending back replies, the DHCP server MUST use the UDP port number that the incoming relay agent uses instead of the fixed DHCP port number.

**Evidence Summary:**

- (E1) establishes the fixed port requirements in Section 7.2.
- (E2) indicates that RFC 8357 alters these rules without specifying which ones.
- (E3) details the RFC 8357 requirement to use the incoming relay’s source port.

**Fix Direction:**

Amend Section 7.2 to clearly state that the fixed destination port requirement (UDP port 547) is overridden when the Relay Source Port Option [RFC8357] is in use, thereby requiring servers and relays to send replies to the recorded source port.

**Severity:** Medium
  *Basis:* The conflict between the unconditional fixed port rule and RFC 8357's alternative behavior can lead to interoperability failures in implementations supporting the generalized relay source port logic.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ActorDirectionalityExpert: Issue-1
- ScopeExpert: Issue-1
- QuantitativeExpert: Issue-2
- DeonticExpert: Issue-1
- CrossRFCExpert: Issue-1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-7-2

**Label:** Underspecification of Long-term Behavior for T1/T2 when Set to Infinity (0xffffffff)

**Bug Type:** Underspecification

**Explanation:**

The document defines 0xffffffff as representing 'infinity' for lifetimes and T1/T2, which effectively disables renewal and rebind operations, yet provides no concrete operational guidance on managing such configurations over the long term.

**Justification:**

- Section 7.7 specifies that all time values for lifetimes, T1, and T2 are 32-bit unsigned integers, with 0xffffffff indicating 'infinity'.
- The draft warns operators to 'take care' when using these infinite values but does not provide explicit recommendations or bounds for maintenance, renewal, or renumbering.

**Evidence Snippets:**

- **E1:**

  All time values for lifetimes, T1, and T2 are unsigned 32-bit integers and are expressed in units of seconds. The value 0xffffffff is taken to mean ‘infinity’ when used as a lifetime… or a value for T1 or T2.

- **E2:**

  Care should be taken in setting T1 or T2 to 0xffffffff (‘infinity’). A client will never attempt to extend the lifetimes of any addresses in an IA with T1 set to 0xffffffff. A client will never attempt to use a Rebind message to locate a different server to extend the lifetimes of any addresses in an IA with T2 set to 0xffffffff.

- **E3:**

  If the ‘shortest’ preferred lifetime is 0xffffffff (‘infinity’), the recommended T1 and T2 values are also 0xffffffff.

**Evidence Summary:**

- (E1) defines the use of 0xffffffff to denote 'infinity' for time values.
- (E2) explains that infinite T1/T2 values preclude renewal and rebind actions.
- (E3) shows that the recommendation for setting T1 and T2 to infinity reinforces this static behavior.

**Fix Direction:**

Provide explicit operational guidance for deployments using infinite values, including best practices for lease maintenance, renumbering, or manual reconfiguration over long timescales.

**Severity:** Low
  *Basis:* Although the use of infinity is internally consistent, the lack of concrete guidance may lead to operational complexities in long-term network management.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-7-3

**Label:** INF_MAX_RT Option Inconsistency Between Advertise Message Processing and Appendix B Table

**Bug Type:** Inconsistency

**Explanation:**

The normative text mandates that clients process the INF_MAX_RT option when it appears in Advertise messages, yet Appendix B’s summary table marks INF_MAX_RT as valid only in Reply messages, risking misinterpretation.

**Justification:**

- Section 18.2.9 explicitly requires that clients process the INF_MAX_RT option in Advertise messages.
- Appendix B’s table contradicts this by displaying INF_MAX_RT only in the context of Reply messages, which could mislead implementers using the table as a quick reference.

**Evidence Snippets:**

- **E1:**

  Table 1 defines INF_MAX_RT with default 3600 secs and describes it as a “Max Information-request timeout value”.

- **E2:**

  Section 18.2.9 says: “The client MUST process any SOL_MAX_RT option (see Section 21.24) and INF_MAX_RT option (see Section 21.25) present in an Advertise message, even if the message contains a Status Code option …”.

- **E3:**

  Appendix B, third table:

                   SOL_MAX_RT  INF_MAX_RT
           Solicit
           Advert.    *
           ...
           Reply      *           *

INF_MAX_RT is marked only for Reply, not Advert.

**Evidence Summary:**

- (E1) describes the INF_MAX_RT option and its default value.
- (E2) mandates processing of INF_MAX_RT in Advertise messages.
- (E3) shows that Appendix B’s table restricts INF_MAX_RT to Reply messages only.

**Fix Direction:**

Revise Appendix B to include INF_MAX_RT for Advertise messages or modify the normative text so that it aligns with the table summary.

**Severity:** Medium
  *Basis:* This inconsistency can lead to divergent interpretations, with some implementations possibly ignoring INF_MAX_RT in Advertise messages, thereby affecting protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-1

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-7-4

**Label:** Ambiguity in the Use of 'Client Messages' Terminology in Relay Agent Rules

**Bug Type:** Editorial / Ambiguity

**Explanation:**

The term 'client messages' in the relay agent rules is ambiguous, potentially leading to confusion about whether it refers solely to messages sent from the relay to the client or has a broader meaning.

**Justification:**

- The text states: 'Relay agents MUST send Relay-forward and Relay-reply messages to UDP destination port 547 and client messages to UDP destination port 546.'
- The lack of explicit directional language creates a momentary ambiguity about the role of relays when sending messages to clients.

**Evidence Snippets:**

- **E1:**

  Relay agents MUST send Relay-forward and Relay-reply messages to UDP destination port 547 and client messages to UDP destination port 546.

- **E2:**

  In the surrounding text, relays serve as intermediaries: they send Relay‑forward/Relay‑reply upstream to servers and send decapsulated “client messages” downstream to clients. Although the intended meaning is inferable, making this explicit would avoid ambiguity.

**Evidence Summary:**

- (E1) provides the normative statement regarding ports for relay agents.
- (E2) explains the contextual ambiguity about the term 'client messages'.

**Fix Direction:**

Clarify the text to explicitly state that 'client messages' indicates messages decapsulated by the relay destined for DHCP clients.

**Severity:** Low
  *Basis:* This issue is primarily editorial and may cause temporary confusion, but it does not impact the fundamental protocol operation.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionalityExpert: NewIssue-1

---
