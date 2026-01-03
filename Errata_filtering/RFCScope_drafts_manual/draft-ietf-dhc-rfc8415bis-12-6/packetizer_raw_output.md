# Errata Reports

Total reports: 5

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-6-1

**Label:** Timing and state for adding extra IA_NA/IA_PD via Request/Renew is underspecified

**Bug Type:** Underspecification

**Explanation:**

Section 6.5 permits clients to add extra IA_NA/IA_PD options via an extra Request or by including them in a Renew, but it does not clearly specify the timing and state‐machine interactions for merging new IAs with existing ones.

**Justification:**

- Section 6.5 advises that a client may send an extra Request with additional IA_NA options but omits a detailed timeline for how these new IAs integrate with the existing T1/T2 timers.
- References to Sections 18.2.2 and 18.1 imply requirements (e.g. all IA options must share the same T1/T2) without clarifying how late Requests affect renewal scheduling.

**Evidence Snippets:**

- **E1:**

  Alternatively, it can send an extra Request message with additional new IA_NA options (or include them in a Renew message). (Section 6.5)

- **E2:**

  Section 18.1 requires servers to return the same T1/T2 values for all IA options in a Reply, to simplify renewal scheduling.

- **E3:**

  Section 18.2.2 describes Request as the message used to “populate IAs with leases and obtain other configuration information” during the normal four‑message Solicit/Advertise/Request/Reply exchange.

**Evidence Summary:**

- (E1) shows the option for an extra Request or Renew to add new IA options, (E2) mandates uniform T1/T2 across IA options, and (E3) defines the role of the Request message, leaving the timing integration underspecified.

**Fix Direction:**

Specify explicit timing rules and client state‐machine adjustments when extra IA options are added via Request or Renew messages.

**Severity:** Low
  *Basis:* Although the underspecification could lead to divergent implementations in timing behavior, it does not undermine core protocol interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-6-2

**Label:** Ambiguous lifecycle timing for self‐generated address registration

**Bug Type:** Underspecification

**Explanation:**

Section 6.6’s description of self‐generated address registration implies that the lifecycle behaves like a typical DHCP lease with T1/T2-based renewal, which can mislead implementers since RFC 9686 uses a separate timing model.

**Justification:**

- The text states that ‘a lease is created by the server’ and that the device performs periodic renewals, which may lead readers to incorrectly assume that T1/T2 timers apply.
- In reality, RFC 9686 mandates a registration mechanism based on the address’s Valid Lifetime and its own retry logic, distinct from the IA_NA/IA_PD timing model.

**Evidence Snippets:**

- **E1:**

  The most of the lifecycle remains the same in principle: a lease is created by the server, and the device performs periodic actions to get the lease renewed, and eventually the lease can expire. (Section 6.6)

**Evidence Summary:**

- (E1) illustrates the phrasing that can be read as implying that self‐generated address registration follows the same T1/T2 renewal mechanism as server‐assigned leases.

**Fix Direction:**

Clarify the text in Section 6.6 to indicate that self‐generated address registration follows the RFC 9686 retry mechanism based on the Valid Lifetime, not the T1/T2 renewal schedule.

**Severity:** Low
  *Basis:* The ambiguous description may confuse implementers about the timing model without compromising protocol functionality.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-6-3

**Label:** Misattributed normative reference for DHCP Release behavior in PD

**Bug Type:** Inconsistency

**Explanation:**

Section 6.3 cites RFC 9096 WPD‑9 to justify a strict prohibition on sending DHCP Release when any delegated address space is outstanding, but RFC 9096 actually only advises against automatic release on restart events.

**Justification:**

- The draft states that a client MUST NOT issue a DHCP Release while any delegated address space (including IA_NA, IA_PD, and SLAAC‑derived addresses) is outstanding, citing RFC 9096 WPD‑9 as Best Current Practice.
- However, RFC 9096 WPD‑9 merely advises that CE routers SHOULD NOT automatically send DHCPv6‑PD RELEASE messages upon restart events, not a comprehensive prohibition on releasing when any addresses are in use.

**Evidence Snippets:**

- **E1:**

  A client that has delegated any of the address space received through DHCP Prefix Delegation MUST NOT issue a DHCP Release on the relevant delegated prefix while any of the address space is outstanding. Requirement WPD‑9 in [RFC9096] makes this the Best Current Practice.

- **E2:**

  RFC 9096 WPD‑9 actually says “CE routers SHOULD NOT automatically send DHCPv6‑PD RELEASE messages upon restart events” and does not include a blanket prohibition.

**Evidence Summary:**

- (E1) documents the draft’s strict no‐Release requirement citing WPD‑9, while (E2) contrasts this by showing the actual, more limited guidance in RFC 9096.

**Fix Direction:**

Revise Section 6.3 to accurately reflect RFC 9096 by limiting the no‐Release rule to what is stated in WPD‑9 and avoiding an overbroad prohibition.

**Severity:** High
  *Basis:* Misattributing normative requirements can lead to divergent implementations and misinterpretation of best practices.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC: Issue-1

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-6-4

**Label:** Inconsistent use of 'lease' for self‐generated address registration

**Bug Type:** Inconsistency

**Explanation:**

Section 6.6 refers to the creation of a ‘lease’ for self‐generated address registration, which conflicts with the document’s definition of 'lease' and with RFC 9686, where the mechanism is a registration of a self-selected address rather than a server‐granted contract.

**Justification:**

- The document’s definition in Section 4.2 describes a lease as a contract by which the server grants the use of an address; this does not apply to self‐generated addresses.
- Using ‘lease’ in Section 6.6 conflates the concept of server-assigned addresses with client-registered ones, potentially leading to design and implementation confusion.

**Evidence Snippets:**

- **E1:**

  lease  A contract by which the server grants the use of an address or delegated prefix to the client for a specified period of time. (Section 4.2)

- **E2:**

  The most of the lifecycle remains the same in principle: a lease is created by the server, and the device performs periodic actions to get the lease renewed, and eventually the lease can expire. (Section 6.6)

**Evidence Summary:**

- (E1) lays out the formal definition of 'lease', while (E2) shows its inconsistent usage in the context of self‐generated address registration.

**Fix Direction:**

Replace the term 'lease' with a term such as 'binding' in Section 6.6 to accurately describe the RFC 9686 registration mechanism.

**Severity:** Medium
  *Basis:* Terminology inconsistencies may lead to confusion in system design even though the on-wire protocol remains unaffected.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC: Issue-2
- Terminology: Issue-1

---

## Report 5: draft-ietf-dhc-rfc8415bis-12-6-5

**Label:** Ambiguous scope terms in operational model descriptions

**Bug Type:** Ambiguity

**Explanation:**

Section 6 uses phrases such as 'a client that has delegated any of the address space received through DHCP Prefix Delegation' and assumes a 'single provisioning domain', both of which can be interpreted more broadly than intended.

**Justification:**

- The phrase regarding delegated address space might be read narrowly to indicate only clients that sub-delegate addresses or broadly to include any client that uses the delegated prefix.
- Similarly, the statement about a single provisioning domain could be misinterpreted as a global protocol assumption rather than an operational model simplification.

**Evidence Snippets:**

- **E1:**

  A client that has delegated any of the address space received through DHCP Prefix Delegation (Section 6.3)

- **E2:**

  This document assumes that the DHCP servers and the client, communicating with the servers via a specific interface, belong to a single provisioning domain. (Section 6)

**Evidence Summary:**

- (E1) and (E2) highlight the ambiguous language which might cause readers to adopt broader interpretations of delegated address space and provisioning domain assumptions.

**Fix Direction:**

Clarify in Section 6 that the delegated address space term applies only to clients that further sub-delegate addresses, and note that the single provisioning domain assumption is an illustrative operational model rather than a strict protocol requirement.

**Severity:** Low
  *Basis:* While ambiguous wording may lead to misinterpretation, it does not affect the protocol’s core functionality.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: NotedAmbiguity

---
