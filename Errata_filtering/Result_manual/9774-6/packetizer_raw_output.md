# Errata Reports

Total reports: 1

---

## Report 1: 9774-6-1

**Label:** Ambiguous Scope of ATOMIC_AGGREGATE Requirement in Consistent Brief Aggregation (Sections 5.2 vs 6.1)

**Bug Type:** Both

**Explanation:**

The document presents ambiguity regarding whether the ATOMIC_AGGREGATE attribute must be attached conditionally—only when AS_PATH truncation occurs as per Section 5.2—or unconditionally included as described in Section 6.1, which may confuse implementers and operators.

**Justification:**

- Section 5.2 mandates that if truncating the AS_PATH actually removes AS numbers compared to the expected RFC4271 aggregation, the ATOMIC_AGGREGATE attribute SHOULD be attached, implying a conditional application.
- In contrast, Section 6.1 states that in consistent brief aggregation, the AGGREGATOR and ATOMIC_AGGREGATE attributes are included without reiterating the condition, creating an ambiguity in operational interpretation.

**Evidence Snippets:**

- **E1:**

  Section 5.2: “the implementation MUST be configured to truncate the AS_PATH after the right‑most instance of the desired origin AS for the aggregate… This form of brief aggregation is referred to as ‘consistent brief’ BGP aggregation.” Followed by: “If the resulting AS_PATH would be truncated from the otherwise expected result of BGP AS_PATH aggregation (an AS_SET would not be generated and possibly some ASes are removed from the ‘longest leading sequence’ of ASes), the ATOMIC_AGGREGATE Path Attribute SHOULD be attached.”

- **E2:**

  Section 6.1: “When aggregating prefixes, network operators MUST use consistent brief aggregation as described in Section 5.2. In consistent brief aggregation, the AGGREGATOR and ATOMIC_AGGREGATE Path Attributes are included, but the AS_PATH does not have AS_SET or AS_CONFED_SET path segment types.”

- **E3:**

  RFC 4271 Section 5.1.6 (quoted in Section 5): “If an aggregate excludes at least some of the AS numbers present in the AS_PATH of the routes that are aggregated as a result of dropping the AS_SET, the aggregated route, when advertised to the peer, SHOULD include the ATOMIC_AGGREGATE attribute.”

**Evidence Summary:**

- (E1) Section 5.2 defines a conditional attachment of ATOMIC_AGGREGATE based on whether truncation actually removes AS numbers.
- (E2) Section 6.1 describes consistent brief aggregation as including ATOMIC_AGGREGATE unconditionally, leading to interpretative tension.
- (E3) RFC 4271 provides guidance that supports the conditional usage as described in Section 5.2.

**Fix Direction:**

Clarify the text in Section 6.1 to explicitly state that the ATOMIC_AGGREGATE attribute is attached only when the AS_PATH is truncated as specified in Section 5.2, or alternatively adjust Section 5.2 to reflect an unconditional attachment if that is the intended behavior.


**Severity:** Low
  *Basis:* The ambiguity affects clarity and may lead to divergent interpretations but does not impact interoperability or security.

**Confidence:** High

---
