# Errata Reports

Total reports: 1

---

## Report 1: 9745-4-1

**Label:** Invalid HTTP-date Timezone in Sunset Header Example (UTC instead of GMT)

**Bug Type:** Inconsistency

**Explanation:**

The RFC 9745 example for the Sunset header uses 'UTC' as the timezone token, which is non‐conformant with the HTTP-date syntax defined in RFC 8594 and RFC 7231/RFC 9110 that requires 'GMT'.

**Justification:**

- RFC 9745 Section 4 shows the example 'Sunset: Sun, 30 Jun 2024 23:59:59 UTC' which violates the normative HTTP-date syntax.
- RFC 8594 and the HTTP core specifications mandate using 'GMT' as the timezone token, as evidenced by the provided examples.

**Evidence Snippets:**

- **E1:**

  RFC 9745 Section 4 example:  
        `Deprecation: @1688169599`  
        `Sunset: Sun, 30 Jun 2024 23:59:59 UTC`

- **E2:**

  RFC 8594 defines Sunset as an HTTP-date and references RFC 7231’s HTTP-date syntax:  
        “The Sunset value is an HTTP-date timestamp, as defined in Section 7.1.1.1 of [RFC7231]. … Sunset = HTTP-date”  
        and gives only `GMT` examples:  
        `Sunset: Sat, 31 Dec 2018 23:59:59 GMT` and `Sunset: Wed, 11 Nov 2026 11:11:11 GMT`.

**Evidence Summary:**

- (E1) Shows the example from RFC 9745 Section 4 where the Sunset header uses 'UTC' as the timezone token.
- (E2) Demonstrates that the normative definition of HTTP-date requires the use of 'GMT' as evidenced by the examples in RFC 8594.

**Fix Direction:**

Change the example in RFC 9745 Section 4 to use 'GMT' instead of 'UTC', e.g., 'Sunset: Sun, 30 Jun 2024 23:59:59 GMT'.



**Severity:** High
  *Basis:* Using an invalid timezone token may lead to non-conformant HTTP-date values, resulting in interoperability issues with strict HTTP parser implementations.

**Confidence:** High
