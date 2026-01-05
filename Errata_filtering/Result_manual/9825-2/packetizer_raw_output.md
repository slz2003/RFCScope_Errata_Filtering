# Errata Reports

Total reports: 2

---


## Report 2: 9825-2-2

**Label:** Undefined Semantics for Multiple Administrative Tag sub‑TLV Occurrences

**Bug Type:** Underspecification

**Explanation:**

RFC 9825 does not specify how to handle the occurrence of multiple Administrative Tag sub‑TLVs under the same parent TLV, leaving implementations free to choose incompatible behaviors.

**Justification:**

- Causal Expert explains that the spec is silent on whether multiple sub‑TLVs should be concatenated, if only the first (or last) instance should be used, or if multiple occurrences constitute an error.
- Deontic, Structural, CrossRFC, and Boundary experts all highlight this absence of normative guidance, which may lead to divergent routing policy outcomes.
- RFC 8362 explicitly requires that behavior for multiple occurrences be defined, making the omission in RFC 9825 a significant oversight.

**Evidence Snippets:**

- **E1:**

  Nowhere in Section 2 (or elsewhere in RFC 9825) is there a rule for what to do if more than one Administrative Tag sub‑TLV appears under the same parent TLV (Extended Prefix TLV, Inter‑Area‑Prefix TLV, Intra‑Area‑Prefix TLV, External‑Prefix TLV, or SRv6 Locator TLV).

- **E2:**

  RFC 8362 explicitly requires that specifications define whether the TLV or sub‑TLV is required and the behavior when there are multiple occurrences, yet RFC 9825 remains silent on how to combine or select among several instances of the Administrative Tag sub‑TLV.

**Evidence Summary:**

- (E1) The spec does not address how to process multiple occurrences of the Administrative Tag sub‑TLV under a single parent TLV.
- (E2) RFC 8362 mandates defining behavior for multiple TLV/sub‑TLV instances, highlighting the oversight in RFC 9825.

**Fix Direction:**

Amend RFC 9825 to explicitly state whether multiple Administrative Tag sub‑TLVs are forbidden or, if allowed, to define that either only the first instance is processed or that all tag values must be concatenated in order.


**Severity:** High
  *Basis:* Ambiguity in processing multiple sub‑TLVs can lead to inconsistent tag sets and conflicting routing policies across implementations.

**Confidence:** High

---


## Report 3: 9825-2-3

**Label:** Mismatch in YANG Module Naming: 'ietf-ospf-admin-tag' vs 'ietf-ospf-admin-tags'

**Bug Type:** Inconsistency

**Explanation:**

The Security Considerations section erroneously refers to the YANG module as 'ietf-ospf-admin-tag' (singular) rather than the correct 'ietf-ospf-admin-tags' (plural) as specified in the module header and IANA registration.

**Justification:**

- CrossRFC Expert points out that the YANG module header (Section 7.2) and the IANA considerations register the module as 'ietf-ospf-admin-tags', while the Security Considerations mistakenly omit the trailing 's'.
- This naming inconsistency can cause confusion among implementers searching for the module in official registries or tooling.

**Evidence Snippets:**

- **E1:**

  The YANG module in RFC 9825 Section 7.2 is clearly named `ietf-ospf-admin-tags` in the `module` statement and its namespace is `urn:ietf:params:xml:ns:yang:ietf-ospf-admin-tags`; the IANA Considerations register the module name as `ietf-ospf-admin-tags`. In the Security Considerations, however, the text says: “The ‘ietf-ospf-admin-tag’ YANG module defines a data model …”, omitting the final “s”.

**Evidence Summary:**

- (E1) Discrepancy between the module name 'ietf-ospf-admin-tags' in the YANG header/IANA records and 'ietf-ospf-admin-tag' in the Security Considerations.

**Fix Direction:**

Correct the Security Considerations reference to use 'ietf-ospf-admin-tags' so that it aligns with the module header and IANA registration.


**Severity:** Low
  *Basis:* This is a minor editorial inconsistency that may cause confusion but does not affect the functionality of implementations.

**Confidence:** High

---