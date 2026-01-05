# Errata Reports

Total reports: 1

---


## Report 1: draft-ietf-dhc-rfc8415bis-12-17-1

**Label:** Ambiguous Interface Selection for Mixed IA_NA and IA_PD in Section 17

**Bug Type:** Underspecification

**Explanation:**

Section 17 defines interface-selection rules at the message level without explicitly resolving which interface to use when a DHCP message contains both IA_NA and IA_PD options, potentially causing ambiguity in multi-interface or multi-WAN scenarios.

**Justification:**

- Section 17.1 mandates using the interface for which configuration information is requested for IA_NA, while Section 17.2 directs that PD messages be sent on the upstream interface, yet no precedence is given for mixed messages.
- Additional text in Sections 12.2, 18.1, and 6.4 emphasizes per-interface transaction assumptions, which highlights the potential for ambiguity when multiple IA types are included in one message.

**Evidence Snippets:**

- **E1:**

  Section 17.1: “When a client sends a DHCP message … it SHOULD send the message through the interface for which configuration information (including the addresses) is being requested. However, the client MAY send the message through another interface…”

- **E2:**

  Section 17.2: “When a client sends a DHCP message for the purpose of prefix delegation, it SHOULD be sent on the interface associated with the upstream router… This rule applies even in the case where a separate IA_PD is used for each downstream interface.”

- **E3:**

  Section 12.2: IA_PD “can be associated with the client, with a set of interfaces, or with exactly one interface. … It may associate a distinct IA_PD with each of its downstream network interfaces…”

- **E4:**

  Section 18.1: “a client SHOULD use a single transaction for all of the IA options required on an interface… servers MUST return the same T1/T2 values for all IA options in a Reply…”

- **E5:**

  Section 6.4 (CE routers): “This model of operation combines address assignment (see Section 6.2) and prefix delegation (see Section 6.3). In general, this model assumes that a single set of transactions between the client and server will assign or extend the client's non-temporary addresses and delegated prefixes.”

**Evidence Summary:**

- (E1) Section 17.1 defines the outgoing interface for configuration requests (IA_NA).
- (E2) Section 17.2 mandates that DHCP messages for prefix delegation (IA_PD) use the upstream interface.
- (E3) Section 12.2 shows that IA_PD can be associated flexibly, which may be conflicting when mixed with IA_NA.
- (E4) Section 18.1 emphasizes the use of a single transaction per interface, implying a one-to-one mapping.
- (E5) Section 6.4 reinforces the model of combined address assignment and prefix delegation under a single transaction.

**Fix Direction:**

Consider adding an explicit clarification in Section 17.2 stating that for messages containing both IA_NA and IA_PD options, the outgoing interface should default to the upstream interface (especially for CE router scenarios following RFC 7084).

**Severity:** Low
  *Basis:* The potential ambiguity is largely theoretical and primarily affects complex multi-interface or multi-WAN cases, while typical deployments using a single transaction per interface would not encounter interoperability failures.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: Issue-1
- Scope: Issue-1
- Deontic: Issue-1
- Boundary: Finding-1

---