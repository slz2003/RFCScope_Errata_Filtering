# Errata Reports

Total reports: 5

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-12-1

**Label:** Misleading Release Guidance for Deprecating IA vs. Actual Release Preconditions

**Bug Type:** Inconsistency

**Explanation:**

Section 12 suggests that a client can deprecate an IA by sending a Release and then using a new IAID, but this contradicts later normative requirements that mandate the client must fully quiesce use of the addresses or prefixes (and for IA_PD, not release while any derived address space is outstanding).

**Justification:**

- Section 12 instructs that releasing an IA initiates deprecation, yet Section 18.2.7 requires that all leases be stopped prior to Release.
- Section 6.3 explicitly forbids a DHCP Release for delegated prefixes when address space is still in use.

**Evidence Snippets:**

- **E1:**

  If the client wishes to obtain a distinctly new address or prefix and deprecate the existing one, the client sends a Release message to the server for the IAs using the original IAID. The client then creates a new IAID, to be used in future messages to obtain leases for the new IA.

- **E2:**

  The client MUST stop using all of the leases being released before the client begins the Release message exchange process. For an address, it MUST already have been removed from the interface. For a delegated prefix, it MUST already have been advertised in Router Advertisements with Preferred and Valid Lifetime of 0 in a Router Advertisement message…

- **E3:**

  A client that has delegated any of the address space received through DHCP Prefix Delegation MUST NOT issue a DHCP Release on the relevant delegated prefix while any of the address space is outstanding.

**Evidence Summary:**

- (E1) Section 12 instructs a Release to initiate deprecation by changing the IAID.
- (E2) Section 18.2.7 requires that all leases be stopped (e.g. removed or set to 0 lifetimes) prior to sending Release.
- (E3) Section 6.3 prohibits the issuance of Release for delegated prefixes when any address space remains in use.

**Fix Direction:**

Modify Section 12 to condition the 'Release + new IAID' process on ensuring that addresses/prefixes are fully quiesced, or explicitly limit this guidance to IA_NA and refer to Section 6.3 for IA_PD.

**Severity:** Medium
  *Basis:* The mismatch can lead to premature release of prefixes or addresses, risking address collisions or routing issues if implementers follow the misleading guidance.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1
- Scope Expert: Issue-1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-12-2

**Label:** Underspecified Lifetime Semantics for Delegated Prefixes Referencing RFC 4862

**Bug Type:** Underspecification

**Explanation:**

Section 12.2 applies RFC 4862’s notions of preferred and valid lifetimes to delegated prefixes even though RFC 4862 defines these semantics only for IPv6 addresses, leaving the intended behavior for prefixes ambiguous.

**Justification:**

- Section 12.1 defines address lifetimes per RFC 4862 but Section 12.2 uses the same phrasing for prefixes despite RFC 4862 not covering prefix-specific behavior.
- This cross-reference forces implementers to infer the intended mapping rather than providing explicit normative guidance for delegated prefix lifetimes.

**Evidence Snippets:**

- **E1:**

  Each address in an IA has a preferred lifetime and a valid lifetime, as defined in [RFC4862]. The lifetimes … apply to the use of addresses; see Section 5.5.4 of [RFC4862].

- **E2:**

  Each delegated prefix in an IA has a preferred lifetime and a valid lifetime, as defined in [RFC4862]. … The lifetimes apply to the use of delegated prefixes; see Section 5.5.4 of [RFC4862].

- **E3:**

  RFC 4862 §5.5.4 is “Address Lifetime Expiry” and discusses preferred/valid lifetimes and deprecation/invalidity of IPv6 unicast addresses on a host interface; it does not define semantics for DHCPv6-delegated prefixes or PD renumbering behavior.

**Evidence Summary:**

- (E1) Section 12.1 ties address lifetimes to RFC 4862 Section 5.5.4.
- (E2) Section 12.2 applies the same definition to delegated prefixes.
- (E3) RFC 4862 only defines these lifetimes for addresses, not prefixes, leaving the prefix behavior ambiguous.

**Fix Direction:**

Revise Section 12.2 to either explicitly define the lifetime semantics for delegated prefixes or reference a more appropriate normative source for prefix lifetimes.

**Severity:** Medium
  *Basis:* The ambiguity may result in divergent implementations regarding the management and renewal of delegated prefixes, potentially affecting interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- CrossRFC Expert: Issue-1

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-12-3

**Label:** Inconsistent Cardinality Description of IA_NA/IA_PD: 'One or More' vs. Empty IA Allowance

**Bug Type:** Inconsistency

**Explanation:**

Section 12 describes the configuration of IA_NA and IA_PD options as having one or more addresses or prefixes, yet later sections require servers to construct and process empty IA options with no addresses or prefixes in error conditions.

**Justification:**

- Section 12.1 and 12.2 state that an IA contains one or more addresses or prefixes.
- Normative sections (e.g., Section 18.3.2 for IA_NA and Section 18.3.9 for IA_PD) mandate that empty IAs with appropriate status codes must be returned when no addresses or prefixes can be assigned.

**Evidence Snippets:**

- **E1:**

  The configuration information in an IA_NA option consists of one or more IPv6 addresses along with the T1 and T2 values for the IA.

- **E2:**

  The configuration information in an IA_PD option consists of one or more prefixes along with the T1 and T2 values for the IA_PD.

- **E3:**

  If the server does not send the NotOnLink status code but it cannot assign any IP addresses to an IA, the server MUST return the IA option in the Reply message with no addresses in the IA and a Status Code option … NoAddrsAvail in the IA.

**Evidence Summary:**

- (E1) Section 12.1 specifies that an IA_NA must contain one or more addresses.
- (E2) Section 12.2 specifies that an IA_PD must contain one or more prefixes.
- (E3) Later sections require construction of empty IA options with status codes when resource allocation fails.

**Fix Direction:**

Adjust the language in Section 12 to indicate that an IA may contain zero or more addresses/prefixes, explicitly accounting for cases when a status code like NoAddrsAvail or NoPrefixAvail is returned.

**Severity:** Low
  *Basis:* While the normative behavior is clear from later sections, the wording in Section 12 may mislead implementers and result in interoperability issues if empty IAs are rejected.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Issue 1
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- Boundary Expert: Finding-1

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-12-4

**Label:** Ambiguous IAID Persistence Requirement for Devices Without Non-Volatile Storage

**Bug Type:** Underspecification

**Explanation:**

Section 12 mandates that the IAID be consistent across client restarts but also acknowledges that devices without non-volatile storage and with changing hardware configurations may not meet this requirement, leaving the fallback behavior unclear.

**Justification:**

- The text states that for any given use of an IA, the IAID MUST be consistent across restarts, yet it concedes that devices without non-volatile storage might not be able to do so.
- It suggests using a well-known IAID (e.g., zero) only for single-IA cases, thereby not covering scenarios where multiple IAs are present on constrained devices.

**Evidence Snippets:**

- **E1:**

  For any given use of an IA by the client, the IAID for that IA MUST be consistent across restarts of the DHCP client.

- **E2:**

  There may be no way for a client to maintain consistency of the IAIDs if it does not have non‑volatile storage and the client's hardware configuration changes.

- **E3:**

  If the client uses only one IAID, it can use a well‑known value, e.g., zero.

**Evidence Summary:**

- (E1) The IAID must remain consistent across restarts for a given use.
- (E2) The text acknowledges that without non-volatile storage and with changing hardware, consistency may be impossible.
- (E3) A well-known IAID is suggested only for the single-IA case.

**Fix Direction:**

Clarify the IAID persistence requirement by conditioning the MUST on a stable client configuration or by providing explicit fallback behavior for multi-IA clients lacking non-volatile storage.

**Severity:** Low
  *Basis:* Although the protocol remains operational (with potential resource management implications), the ambiguity may lead to inconsistent implementations and unintended server-side resource allocation.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Issue 2
- Deontic Expert: Issue-2
- Boundary Expert: Finding-2

---

## Report 5: draft-ietf-dhc-rfc8415bis-12-12-5

**Label:** Ambiguous IA_PD Association to Interfaces

**Bug Type:** Ambiguity

**Explanation:**

Section 12.2 describes IA_PD as being associated with the client, a set of interfaces, or a single interface, yet the on-the-wire binding is only based on <DUID, IA-type, IAID>, leaving the intended operational scope of IA_PD unclear.

**Justification:**

- The text states that an IA_PD can be associated with various interface scopes but does not explain how this association is reflected or enforced in protocol behavior.
- This ambiguity may lead to differences in interpretation regarding whether certain rules should apply per-interface or per-IA_PD.

**Evidence Snippets:**

- **E1:**

  An IA_PD is different from an IA for address assignment in that it does not need to be associated with exactly one interface. One IA_PD can be associated with the client, with a set of interfaces, or with exactly one interface.

- **E2:**

  The IA_PD’s 'association' to 'the client, a set of interfaces, or one interface' is purely conceptual; the on-the-wire binding tuple <DUID, IA-type, IAID> has no explicit interface notion, which may leave some ambiguity about which statements later in the document are per-interface vs per-IA_PD.

**Evidence Summary:**

- (E1) Section 12.2 describes multiple possible association models for IA_PD.
- (E2) The binding tuple does not include interface information, resulting in ambiguity about the intended scope.

**Fix Direction:**

Provide clarification in Section 12.2 on how the conceptual association of IA_PD to interfaces is mapped onto the protocol’s binding and operational behavior.

**Severity:** Low
  *Basis:* This ambiguity does not affect the core protocol operation but could lead to varying interpretations among implementers.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope Expert: NotedAmbiguities
- Boundary Expert: (implied ambiguity)

---
