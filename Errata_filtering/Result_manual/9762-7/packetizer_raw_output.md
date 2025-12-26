# Errata Reports

Total reports: 9

---

## Report 1: 9762-7-1

**Label:** Underspecified P-list Update on P Flag Clearance

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define how to update the per‐interface P-flagged prefix list when a previously P-flagged prefix is re-advertised with the P flag cleared while its preferred lifetime remains non-zero.

**Justification:**

- The list membership is defined only by having been received with the P flag set and by removal only when the preferred lifetime becomes zero, leaving the behavior on a P flag change ambiguous.
- The security considerations mention oscillation and REBIND triggering when the P flag toggles, which is not explicitly supported by the normative text.

**Evidence Snippets:**

- **E1:**

  For each interface, the client MUST keep a list of every prefix that was received from a PIO with the P flag set and currently has a non-zero preferred lifetime.

- **E2:**

  When a prefix's preferred lifetime becomes zero, either because the preferred lifetime expires or because the client receives a PIO for the prefix with a zero preferred lifetime, the prefix MUST be removed from the list.

- **E3:**

  The attacker might force hosts to oscillate between DHCPv6-PD and PIO-based SLAAC by sending the same set of PIOs with and then without the P flag set. That would cause the clients to issue REBIND requests, increasing the load on the DHCP infrastructure.

**Evidence Summary:**

- (E1) defines the list membership criteria, (E2) specifies removal only on lifetime expiration, and (E3) indicates that toggling the P flag is expected to trigger REBINDs—highlighting the ambiguity.

**Fix Direction:**

Clarify that when a new PIO for a prefix is received with the P flag cleared, the prefix MUST be removed from the list (unless another concurrent PIO with the P flag set applies).


**Severity:** Medium
  *Basis:* This ambiguity affects when DHCPv6-PD is started or stopped and when REBINDs are triggered, potentially leading to divergent behavior and security concerns.

**Confidence:** High

---

## Report 2: 9762-7-2

**Label:** Ambiguous Duration for Disabling P Flag Processing

**Bug Type:** Underspecification

**Explanation:**

The specification does not specify when or if a disabled P flag processing should be re-enabled after a failure to obtain suitable PD prefixes.

**Justification:**

- The normative text allows a client to disable further processing of the P flag if no suitable PD prefixes are obtained but gives no guidance on when this decision may be revisited.
- This may lead to permanently ignored P flag updates even after network conditions improve.

**Evidence Snippets:**

- **E1:**

  If the client does not obtain any suitable prefixes via DHCPv6-PD that are suitable for SLAAC, it MAY choose to disable further processing of the P flag on that interface, allowing the client to fall back to other address assignment mechanisms…

- **E2:**

  If the P flag is set and the node implements RFC 9762, it SHOULD treat the Autonomous flag as if it was unset and use prefix delegation to obtain addresses as described in RFC 9762.

**Evidence Summary:**

- (E1) shows that disabling P flag processing is permitted without timing guidance, and (E2) indicates conflicting expectation for continuous processing.

**Fix Direction:**

Provide normative guidance (e.g., timers or event conditions) for re-enabling P flag processing after it has been disabled.


**Severity:** Low
  *Basis:* While the ambiguity may lead to divergent retry behaviors, it does not break interoperability or core protocol functionality.

**Confidence:** High

---

## Report 3: 9762-7-3

**Label:** Ambiguous REBIND Trigger Condition with Expired Delegated Prefixes

**Bug Type:** Underspecification

**Explanation:**

It is unclear whether the REBIND should be triggered on a P-list change if the client has only historical (and possibly expired) delegated prefixes versus currently valid ones.

**Justification:**

- The REBIND rule is tied to the clause 'if the client has already received delegated prefix(es)', which does not distinguish between current valid prefixes and those that are no longer usable.
- This ambiguity can lead to REBINDs being sent without any valid IA_PD content or not being sent when expected.

**Evidence Snippets:**

- **E1:**

  If the client has already received delegated prefix(es) from one or more servers, then any time one or more prefix(es) are added to or removed from the list, the client MUST consider this to be a change in configuration information as described in Section 18.2.12 of [RFC8415]. In that case, the client MUST perform a REBIND, unless the list is now empty.

**Evidence Summary:**

- (E1) ties REBIND to any change in the P-list without considering whether the delegated prefixes are still valid.

**Fix Direction:**

Clarify that REBIND triggering should depend on the presence of currently valid delegated prefixes rather than solely on their historical receipt.


**Severity:** Low
  *Basis:* The potential for triggering unnecessary REBIND exchanges can lead to inefficient exchanges but does not prevent protocol operation.

**Confidence:** Medium

---

## Report 4: 9762-7-4

**Label:** Ambiguous SLAAC-Suitable Prefix Length Thresholds

**Bug Type:** Underspecification

**Explanation:**

The document uses qualitative terms such as 'short enough for SLAAC' and 'suitable for SLAAC' without providing explicit numerical thresholds, leaving implementers to infer the correct prefix lengths.

**Justification:**

- Terms like 'too long', 'short enough', and 'shorter than required' are not precisely defined in the document.
- This ambiguity may lead to varying interpretations and potential interop issues, especially on links with non-standard interface identifier lengths.

**Evidence Snippets:**

- **E1:**

  When a client requests a prefix via DHCPv6‑PD, it MUST use the prefix length hint ... to request a prefix that is short enough to form addresses via SLAAC.

- **E2:**

  If the delegated prefix is too long to be used for SLAAC, the client MUST ignore it, as Section 7 of [RFC9663] requires the network to provide a SLAAC‑suitable prefix to clients. If the prefix is shorter than required for SLAAC, the client SHOULD accept it, allocate one or more longer prefixes suitable for SLAAC, and use the prefixes as described below.

**Evidence Summary:**

- (E1) and (E2) illustrate that the SLAAC suitability criteria rely on qualitative language without explicit numeric boundaries.

**Fix Direction:**

Define explicit numerical guidelines for SLAAC suitability, for example by referencing RFC 4862’s requirement that the sum of the prefix length and the interface identifier length equals 128 bits.


**Severity:** Medium
  *Basis:* The lack of explicit thresholds can result in inconsistent PD prefix processing across different implementations.

**Confidence:** High

---

## Report 5: 9762-7-5

**Label:** Vacuous 'All PIOs Have P Set' Condition with Zero PIOs

**Bug Type:** Underspecification

**Explanation:**

The rule to suppress IA_NA requests when all processed PIOs have the P bit set is ambiguous when there are zero PIOs, leading to potentially inconsistent behavior.

**Justification:**

- Mathematically, a universal condition over an empty set is vacuously true, yet operationally it is unclear if the rule should apply when no PIOs are present.
- This could cause different implementations to either suppress or not suppress IA_NA requests under the same RA conditions.

**Evidence Snippets:**

- **E1:**

  Similarly, if all PIOs processed by the client have the P bit set, the client SHOULD NOT request individual IPv6 addresses from DHCPv6, i.e., it SHOULD NOT include any IA_NA options in Solicit messages [RFC8415].

- **E2:**

  The P bit is purely a positive indicator... The absence of any PIOs with the P bit does not carry any kind of signal to the opposite and MUST NOT be processed to mean that DHCPv6‑PD is absent.

**Evidence Summary:**

- (E1) specifies suppression of IA_NA when PIOs are present with P set, while (E2) indicates that absence of PIOs should not be interpreted as an opposite signal, creating ambiguity in the zero‐PIO case.

**Fix Direction:**

Clarify the condition to require that the rule applies only when at least one PIO is present.


**Severity:** Medium
  *Basis:* This ambiguity can lead to observable differences in address assignment behavior across implementations.

**Confidence:** High

---

## Report 6: 9762-7-6

**Label:** Conflicting Interface Association for Delegated-Prefix Addresses

**Bug Type:** Inconsistency

**Explanation:**

The specification mandates that delegated-prefix addresses be treated as if assigned to different interfaces for Neighbor Discovery/Duplicate Address Detection (ND/DAD) and for source address selection, conflicting with the traditional single-interface binding model.

**Justification:**

- Section 7.2 requires that if a delegated prefix is advertised, addresses are to be treated as if assigned to the advertising (downstream) interface for ND/DAD.
- Section 7.5 instructs that for source address selection, the same addresses should be treated as if assigned to the interface on which the prefix was received (upstream), creating a conflict.

**Evidence Snippets:**

- **E1:**

  If the client advertises the prefix on an interface and it has formed addresses from the prefix, then it MUST act as though the addresses were assigned to that interface for the purposes of Neighbor Discovery and Duplicate Address Detection.

- **E2:**

  For the purpose of source address selection [RFC6724], if the host creates any addresses from a delegated prefix, it SHOULD treat those addresses as if they were assigned to the interface on which the prefix was received.

**Evidence Summary:**

- (E1) mandates ND/DAD treatment on the downstream interface, while (E2) directs source address selection to use the upstream interface, leading to conflicting interface associations.

**Fix Direction:**

Revise the language to clearly differentiate between the physical address assignment used for ND/DAD and the candidate-set association used for source address selection, ensuring these do not conflict.


**Severity:** Medium
  *Basis:* The conflicting instructions violate the typical one-interface-per-address model and can result in inconsistent source address selection behavior.

**Confidence:** High

---

## Report 7: 9762-7-7

**Label:** REBIND vs RENEW Inconsistency with RFC 8415

**Bug Type:** Inconsistency

**Explanation:**

The document mandates that any change in the P-list triggers a REBIND, which conflicts with RFC 8415's guidance that configuration changes not associated with link movement should trigger a Renew.

**Justification:**

- RFC 9762 requires a REBIND on any change to the P-list (unless the list is empty), while RFC 8415 normally expects a Renew for significant configuration changes that are not due to mobility.
- This discrepancy may force clients to send REBIND messages unnecessarily, altering the expected DHCPv6 behavior.

**Evidence Snippets:**

- **E1:**

  If the client has already received delegated prefix(es) from one or more servers, then any time one or more prefix(es) are added to or removed from the list, the client MUST consider this to be a change in configuration information as described in Section 18.2.12 of [RFC8415]. In that case, the client MUST perform a REBIND, unless the list is now empty.

- **E2:**

  RFC 8415 Section 18.2.12 describes using a Renew/Reply exchange for configuration changes that are not associated with link movement.

**Evidence Summary:**

- (E1) dictates a REBIND on any P-list change, whereas (E2) shows that RFC 8415 would normally expect a Renew for non-mobility related configuration changes.

**Fix Direction:**

Clarify whether the P-list change should always trigger a REBIND as specified or if the appropriate RFC 8415 mechanism (Renew) should be used under certain conditions.


**Severity:** Medium
  *Basis:* This inconsistency may lead to different DHCPv6 message behaviors, affecting interoperability and server load.

**Confidence:** Medium

---

## Report 8: 9762-7-8

**Label:** Misleading Reference for 'Prefix Length Hint'

**Bug Type:** Underspecification

**Explanation:**

The specification refers to RFC 8415 Section 18.2.4 for the prefix length hint, which may mislead implementers since that section mainly deals with Renew message behavior rather than initial prefix requests.

**Justification:**

- The cross-reference could cause confusion as to whether the prefix length hint applies to initial Solicit/Request exchanges or only to Renew messages.

**Evidence Snippets:**

- **E1:**

  When a client requests a prefix via DHCPv6-PD, it MUST use the prefix length hint (Section 18.2.4 of [RFC8415])…

**Evidence Summary:**

- (E1) shows that the specification directs implementers to a section focused on Renew messages, potentially misleading those seeking the general definition of the prefix length hint.

**Fix Direction:**

Amend the reference to encompass the sections of RFC 8415 that govern both initial prefix requests and renewal, ensuring clarity on how the prefix length hint is to be used.


**Severity:** Low
  *Basis:* This is primarily a documentation clarity issue and does not affect the core protocol operation.

**Confidence:** Low

---

## Report 9: 9762-7-9

**Label:** Ambiguous Usage of 'Client' vs 'Host' in Processing P Flag

**Bug Type:** Underspecification

**Explanation:**

The document inconsistently uses the terms 'client' and 'host', creating ambiguity regarding which requirements apply to router-based PD clients versus non-router hosts.

**Justification:**

- Key normative requirements, such as processing of redirects and source address selection, are sometimes specified for 'hosts' only, while the overall document applies to 'clients' that may be either.
- This inconsistency leads to uncertainty whether router-based PD clients (e.g., CE routers) are expected to implement the 'host' behaviors.

**Evidence Snippets:**

- **E1:**

  Client: a node that connects to a network and acquires addresses. ... It may be either a host or a router as defined by [RFC8200].

- **E2:**

  Host: any node that is not a router [RFC8200]

- **E3:**

  This specification only applies to clients that support DHCPv6 prefix delegation.

**Evidence Summary:**

- (E1) and (E2) provide differing definitions, while (E3) shows that the document’s normative scope is not consistently applied between 'client' and 'host'.

**Fix Direction:**

Clarify and consistently define the roles of 'client' and 'host', specifying explicitly which requirements apply to router-based PD clients and which apply to non-router hosts.


**Severity:** Medium
  *Basis:* Ambiguity in terminology can lead to inconsistent implementations and misinterpretation of which behaviors are mandatory.

**Confidence:** High

---
