# Errata Reports

Total reports: 1

---


## Report 3: 9745-2-3

**Label:** Invalid Sunset Header Example Timezone ('UTC' instead of 'GMT')

**Bug Type:** Inconsistency

**Explanation:**

The example provided for the Sunset header in RFC 9745 uses 'UTC' as the timezone, which does not conform to the HTTP-date specification that requires the use of 'GMT'.

**Justification:**

- RFC 8594 and the HTTP-date specification mandate that the Sunset header use the IMF-fixdate format with 'GMT' as the timezone.
- The example 'Sunset: Sun, 30 Jun 2024 23:59:59 UTC' directly conflicts with the normative example provided in RFC 8594.

**Evidence Snippets:**

- **E1:**

  Sunset: Sun, 30 Jun 2024 23:59:59 UTC

- **E2:**

  For example:  Sunset: Sat, 31 Dec 2018 23:59:59 GMT

**Evidence Summary:**

- (E1) Shows the RFC 9745 example using 'UTC' as the timezone.
- (E2) Provides the normative example from RFC 8594 using 'GMT'.

**Fix Direction:**

Update the Sunset header example in Section 4 of RFC 9745 to replace 'UTC' with 'GMT' so that it conforms to the HTTP-date syntax.


**Severity:** Medium
  *Basis:* The inconsistency between the spec example and the normative HTTP-date format may mislead implementers and cause parsing errors.

**Confidence:** High

---