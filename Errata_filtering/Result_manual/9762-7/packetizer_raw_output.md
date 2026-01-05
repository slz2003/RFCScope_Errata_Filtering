# Errata Reports

Total reports: 1

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