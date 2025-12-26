# Errata Reports

Total reports: 3

---

## Report 1: 9745-2-1

**Label:** Underspecification: Handling of Multiple Deprecation Header Values and Evolving Deprecation Dates

**Bug Type:** Underspecification

**Explanation:**

The specification defines Deprecation as a singleton Structured Field with a single Date but does not specify how to process multiple header instances or conflicting deprecation dates over time, which may lead to inconsistent interpretations.

**Justification:**

- The spec states that 'Deprecation is an Item Structured Header Field; its value MUST be a Date as per Section 3.3.7 of [RFC9651]' but does not explain how to behave if multiple values are present.
- It also advises authors to document behavior when multiple members are present, yet RFC 9745 does not clarify if the earliest, latest, or a specific date should be used.

**Evidence Snippets:**

- **E1:**

  Deprecation is an Item Structured Header Field; its value MUST be a Date as per Section 3.3.7 of [RFC9651].

- **E2:**

  Authors of fields with a singleton value (see Section 5.5) are additionally advised to document how to treat messages where the multiple members are present (a sensible default would be to ignore the field, but this might not always be the right choice).

- **E3:**

  9745 does not specify whether a recipient should: (a) reject the header as invalid, (b) pick the earliest date, (c) pick the latest date, or (d) apply some other rule, nor does it say anything about reconciling changing deprecation dates across multiple responses over time.

**Evidence Summary:**

- (E1) Defines Deprecation as a singleton Date field.
- (E2) Advises documenting handling of multiple values but does not specify it.
- (E3) Lists the ambiguity in reconciling multiple or evolving deprecation dates.

**Fix Direction:**

Clarify the processing rules for when multiple Deprecation header values are present and specify how conflicting deprecation dates across responses should be handled.


**Severity:** Low
  *Basis:* Temporal analysis indicates that although ambiguous, the field is advisory and does not affect protocol correctness.

**Confidence:** High

---

## Report 2: 9745-2-2

**Label:** Ambiguity in Deprecation Header Resource Scope (Target vs. Representation/Content-Location)

**Bug Type:** Underspecification

**Explanation:**

The specification is ambiguous about which resource is deprecated, leaving it unclear whether the Deprecation header applies to the target resource that returns the response or to the resource identified via Content-Location.

**Justification:**

- RFC 9745 states that the header describes the deprecation of 'the resource identified with the response it occurred within (see Section 6.4.2 of [HTTP])', but Section 6.4.2 defines resource mapping based on content representation which can lead to different interpretations.
- The use of non‑standard phrases like 'resource in the context of the message' further deepens the ambiguity when a response includes a Content-Location differing from the request target.

**Evidence Snippets:**

- **E1:**

  The Deprecation HTTP response header field describes the deprecation of the resource identified with the response it occurred within (see Section 6.4.2 of [HTTP ]).

- **E2:**

  HTTP 9110 §6.4.2 “Identifying Content” defines rules for mapping a response’s content to a resource, distinguishing: (a) representations of the target resource and (b) representations of a different resource indicated by Content-Location.

- **E3:**

  Ambiguous identification of *which* resource is deprecated (confusing HTTP terminology and scope)

- **E4:**

  If an implementer reads the specification literally, a response with a differing Content-Location could lead to deprecating either the target resource or the Content-Location resource, resulting in divergent behaviors.

**Evidence Summary:**

- (E1) Indicates deprecation applies to the 'resource identified with the response'.
- (E2) Shows that HTTP §6.4.2 distinguishes between target and Content-Location resources.
- (E3) Highlights the ambiguous terminology used in the spec regarding which resource is deprecated.
- (E4) Explains the potential for divergent interpretation in edge cases.

**Fix Direction:**

Revise the Deprecation header description to clearly state that it applies to the target resource that returns the response, irrespective of any Content-Location header.


**Severity:** Medium
  *Basis:* Multiple expert analyses indicate that the ambiguity can lead to inconsistent migration decisions in edge cases.

**Confidence:** Medium

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
