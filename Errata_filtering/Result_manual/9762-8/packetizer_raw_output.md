# Errata Reports

Total reports: 2

---

## Report 1: 9762-8-1

**Label:** Ambiguous Mapping Between RFC 6724 'Router' Identity and DHCPv6 Server/Relay

**Bug Type:** Underspecification / Role Confusion

**Explanation:**

Section 8 instructs hosts to associate delegated prefixes with the link‑local address of the DHCPv6 server or relay, conflating it with the router that advertised the prefix per RFC 6724 Rule 5.5, which leads to ambiguity when the server/relay is not the actual router used for forwarding.

**Justification:**

- The text first states that the prefix should be associated with the router that advertised it, but then instructs using the DHCPv6 server or relay's link‑local address without explaining its relationship to the default router.
- Multiple experts noted that this conflation is problematic in topologies where the DHCPv6 server or relay is not co-located with the RA‑advertising default router, leading to inconsistent source address selection.

**Evidence Snippets:**

- **E1:**

  In multi-prefix multihoming, the host generally needs to associate the prefix with the router that advertised it (for example, see Rule 5.5 in [RFC6724]).

- **E2:**

  If the host supports Rule 5.5, then it SHOULD associate each prefix with the link-local address of the DHCPv6 server or relay from which it received the REPLY packet.

- **E3:**

  When receiving multiple REPLYs carrying the same prefix from distinct link-local addresses, the host SHOULD associate that prefix with all of these addresses.

**Evidence Summary:**

- (E1) defines the original expectation to associate a prefix with the advertising router as per RFC 6724.
- (E2) instructs associating the prefix with the DHCPv6 server/relay's link‑local address, creating a potential role conflict.
- (E3) reinforces the approach for handling multiple REPLYs, which further highlights the ambiguity in the mapping.

**Fix Direction:**

Clarify the text to explicitly require that the delegated prefix be associated with the actual default router (as determined via RA advertisements) or clearly define under which conditions the DHCPv6 server/relay's link‑local address may be used for Rule 5.5.


**Severity:** Medium
  *Basis:* Incorrect or inconsistent association of prefixes with router identities in multihomed deployments can lead to suboptimal source address selection, even though overall connectivity remains unaffected.

**Confidence:** High

---

## Report 2: 9762-8-2

**Label:** Undefined Behavior for Non-Link-Local DHCPv6 REPLY Sources

**Bug Type:** Underspecification

**Explanation:**

Section 8 mandates associating each delegated prefix with the link‑local address of the DHCPv6 REPLY sender but does not define how to proceed when the REPLY is received from a non-link‑local address.

**Justification:**

- The normative instruction assumes that the REPLY always originates from a link‑local address, leaving the behavior undefined if a non-link‑local source is encountered.
- Experts noted that in deployments where a DHCPv6 REPLY is sent from a global or ULA address, the lack of fallback rules can lead to divergent implementations and unpredictable source address selection.

**Evidence Snippets:**

- **E1:**

  If the host supports Rule 5.5, then it SHOULD associate each prefix with the link-local address of the DHCPv6 server or relay from which it received the REPLY packet.

- **E2:**

  In an edge or non-conforming deployment where the client actually receives REPLYs from a non-link-local source address (e.g., server using a global or ULA source), the phrase 'link-local address … from which it received the REPLY' no longer matches reality: the visible source is not link-local, and the spec does not say whether the client should either use the non-link-local address for association or skip the association entirely.

**Evidence Summary:**

- (E1) establishes the requirement to use the link‑local address for associating delegated prefixes.
- (E2) highlights the gap in specification when the REPLY’s source address is not link‑local, leaving the intended behavior ambiguous.

**Fix Direction:**

Define the fallback behavior when a DHCPv6 REPLY is received from a non-link‑local address, either by mandating that REPLYs must use a link‑local source or by specifying how to derive an appropriate association when they do not.


**Severity:** Medium
  *Basis:* This underspecification can result in inconsistent implementations in environments where DHCPv6 REPLYs may originate from non-link‑local addresses, thus affecting the intended source address selection mechanism.

**Confidence:** High

---
