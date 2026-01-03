# Errata Reports

Total reports: 7

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-16-1

**Label:** Ambiguous server handling of client unicast messages in Section 16

**Bug Type:** Inconsistency

**Explanation:**

Section 16 initially prohibits server acceptance of unicast messages from clients but later permits legacy servers to continue receiving unicast messages, creating ambiguity in the expected behavior.

**Justification:**

- The normative text first states that servers SHOULD NOT accept unicast traffic, yet immediately after it allows upgraded servers to continue processing unicast messages from legacy clients.
- This conflicting guidance may lead to divergent implementations about whether to drop or process such messages.

**Evidence Snippets:**

- **E1:**

  Servers SHOULD NOT accept unicast traffic from clients.  The Server Unicast option (see Section 21.12) and UseMulticast status code (see Section 21.13) have been obsoleted and hence clients should no longer send messages to a server's unicast address nor receive the UseMulticast status code.

- **E2:**

  However, a server that previously supported the Server Unicast option and is upgraded to not support it, MAY continue to receive unicast messages if it previously sent the client the Server Unicast option.  But this causes no harm and the client will eventually switch back to sending multicast messages…

**Evidence Summary:**

- (E1) indicates that servers should not accept unicast traffic, while (E2) permits legacy unicast reception under certain conditions.

**Fix Direction:**

Clarify and prioritize the normative instructions in Section 16 by explicitly stating whether legacy unicast messages must be dropped or can be processed.

**Severity:** Medium
  *Basis:* The conflicting instructions may result in inconsistent implementations across different servers.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: NewIssue-1
- Causal Expert: Issue 1
- Boundary Expert: Finding-2

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-16-2

**Label:** Ambiguity in client multicast message acceptance rule

**Bug Type:** Underspecification

**Explanation:**

The rule 'Clients SHOULD NOT accept multicast messages' is broad and lacks clear scoping, potentially impacting the interpretation for future message types.

**Justification:**

- The normative statement is unconditional despite a following note that tries to confine its scope.
- This lack of explicit scoping could lead to misinterpretation when extensions introduce new message types.

**Evidence Snippets:**

- **E1:**

  Clients SHOULD NOT accept multicast messages.

- **E2:**

  Note: The multicast/unicast rules mentioned above apply to the DHCP messages within this document.  Messages defined in other and future documents may have different rules.

**Evidence Summary:**

- (E1) presents the broad rule, and (E2) attempts to limit it without clear precedence.

**Fix Direction:**

Specify in Section 16 the precise message types or contexts to which the multicast rejection rule applies.

**Severity:** Low
  *Basis:* The issue mainly affects the interpretation for future extensions and is unlikely to impact current conforming implementations.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: NewIssue-2
- Deontic Expert: Issue-2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-16-3

**Label:** Contradictory handling of invalid options in messages

**Bug Type:** Inconsistency and Underspecification

**Explanation:**

The general rule permitting endpoints to extract information from messages with known invalid options conflicts with specific rules that mandate discarding such messages, leaving endpoint behavior ambiguous.

**Justification:**

- The global text allows clients and servers to either salvage information or discard messages that contain known invalid options.
- In contrast, the specific rule for Information-request messages mandates that any message including an IA option MUST be discarded.

**Evidence Snippets:**

- **E1:**

  Clients and servers MAY choose to either (1) extract information from such a message if the information is of use to the recipient or (2) ignore such a message completely and just discard it.

- **E2:**

  Servers MUST discard any received Information-request message that meets any of the following conditions: … the message includes an IA option.

**Evidence Summary:**

- (E1) provides a permissive option while (E2) imposes a strict discard requirement, leading to ambiguity.

**Fix Direction:**

Clarify that the specific per-message rules (e.g., for Information-request) override the general permissive rule.

**Severity:** Low
  *Basis:* The ambiguity is limited to error handling in non-normative cases and is unlikely to affect interoperability among compliant implementations.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: NewIssue-3
- Boundary Expert: Finding-1

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-16-4

**Label:** Underspecified client behavior upon receiving an obsolete UseMulticast status code

**Bug Type:** Underspecification

**Explanation:**

The draft lacks clear normative guidance on how a client should react if it receives a UseMulticast status code, leaving implementations uncertain whether to treat it as a generic failure or mimic legacy behavior.

**Justification:**

- The text states that clients should no longer receive the UseMulticast status code, yet provides no instructions in case one is received.
- Legacy RFC 8415 behavior (resending via multicast) is referenced but not explicitly mandated in the new guidelines.

**Evidence Snippets:**

- **E1:**

  The Server Unicast option (see Section 21.12) and UseMulticast status code (see Section 21.13) have been obsoleted and hence clients should no longer send messages to a server's unicast address nor receive the UseMulticast status code.

- **E2:**

  If the client receives a Reply message with a status code of UseMulticast, the client … sends subsequent messages … using multicast. The client resends the original message using multicast.

**Evidence Summary:**

- (E1) indicates that UseMulticast is obsolete, while (E2) describes legacy behavior that is no longer explicitly mandated.

**Fix Direction:**

Add explicit normative instructions for handling a UseMulticast status code if one is received.

**Severity:** Medium
  *Basis:* Without clear instructions, implementations might adopt inconsistent fallback behaviors in mixed deployment scenarios.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-1

---

## Report 5: draft-ietf-dhc-rfc8415bis-12-16-5

**Label:** Candidate: Obsoletion of Server Unicast and UseMulticast is narrow but consistent in scope

**Bug Type:** None

**Explanation:**

The document deliberately obsoletes the Server Unicast option and UseMulticast status code within a narrow scope; although the guidance is sparse, it is internally consistent.

**Justification:**

- Relevant sections instruct that the obsoleted features should be ignored and not requested, which aligns with the overall move to a multicast-only model.

**Evidence Snippets:**

- **E1:**

  Section 16: “Servers SHOULD NOT accept unicast traffic from clients. The Server Unicast option (see Section 21.12) and UseMulticast status code (see Section 21.13) have been obsoleted and hence clients should no longer send messages to a server's unicast address nor receive the UseMulticast status code. However, a server that previously supported the Server Unicast option and is upgraded to not support it, MAY continue to receive unicast messages…”

- **E2:**

  Section 21.13: UseMulticast status code row updated to “Obsoleted; no longer used.”

**Evidence Summary:**

- (E1) and (E2) illustrate that the obsoletion is applied in a narrow, controlled manner consistent with the revised protocol model.

**Severity:** Low
  *Basis:* Since the guidance is coherent and supports the transition to multicast-only behavior, no corrective action is necessary.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---

## Report 6: draft-ietf-dhc-rfc8415bis-12-16-6

**Label:** Candidate: Broad multicast acceptance rule for clients is contextually coherent

**Bug Type:** None

**Explanation:**

While the directive that 'Clients SHOULD NOT accept multicast messages' appears broad, it is explicitly limited to the DHCP messages defined in this document and does not conflict with the established transmission behaviors.

**Justification:**

- An accompanying note clarifies that the rule applies only to the current DHCP message set, allowing future extensions to specify different rules.

**Evidence Snippets:**

- **E1:**

  Section 16: “Clients SHOULD NOT accept multicast messages. … Note: The multicast/unicast rules mentioned above apply to the DHCP messages within this document. Messages defined in other and future documents may have different rules.”

- **E2:**

  Section 7.1, 14: only defined multicast destination used by clients is All_DHCP_Relay_Agents_and_Servers (ff02::1:2) for client→server/relay transmission.

**Evidence Summary:**

- (E1) and (E2) show that the broad rule is intended only for the defined DHCP messages and is contextually appropriate.

**Severity:** Low
  *Basis:* The scoping note mitigates the broad phrasing, ensuring the rule does not adversely impact current protocol operation.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2

---

## Report 7: draft-ietf-dhc-rfc8415bis-12-16-7

**Label:** Potential asymmetry in Client Identifier validation in Reply messages

**Bug Type:** Inconsistency

**Explanation:**

While servers are required to include the correct Client Identifier in messages, the specification does not require clients to verify a Reply message’s Client Identifier against their own, resulting in an asymmetry in validation.

**Justification:**

- Section 16.10 provides MUST-discard rules for missing Server Identifier and transaction-id mismatches but omits similar requirements for an incorrect Client Identifier in Reply messages.
- This asymmetry might lead to divergent robustness in client implementations.

**Evidence Snippets:**

- **E1:**

  Not a bug but worth noting: Section 16.10 (Reply) gives explicit MUST-discard rules for missing Server Identifier and transaction-id mismatch, but does not say that a client MUST discard a Reply whose Client Identifier option (if present) does not match its own DUID.

**Evidence Summary:**

- (E1) highlights the lack of a normative requirement for clients to validate the Client Identifier in Reply messages.

**Fix Direction:**

Consider adding normative guidance requiring clients to verify the Client Identifier in Reply messages.

**Severity:** Low
  *Basis:* Although the asymmetry may cause variability in client robustness, it is unlikely to lead to significant interoperability issues.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary Expert: Notes

---
