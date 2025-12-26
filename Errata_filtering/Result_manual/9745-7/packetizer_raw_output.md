# Errata Reports

Total reports: 2

---

## Report 1: 9745-7-1

**Label:** Conflicting guidance on resource behavior after deprecation date in Sections 5 vs. 7

**Bug Type:** Inconsistency

**Explanation:**

The RFC contains contradictory guidance: Section 5 asserts that deprecation does not change resource behavior, while Section 7 mandates that clients must no longer assume behavior remains the same once the deprecation date has passed. This leads to uncertainty about whether the resource’s behavior is guaranteed to be stable over time.

**Justification:**

- Temporal Expert T1 noted tension between the claim of unchanged behavior and the normative requirement that clients must not assume stability after the deprecation date.
- Deontic Expert Issue-1 observed that while the header is treated as a lifecycle hint, the 'MUST no longer assume' language creates a conflict with the descriptive assertion in Section 5.
- Boundary Expert Finding-1 highlighted that the same resource behavior is described in contradictory terms across Section 5 and Section 7, causing potential divergent interpretations by implementers.

**Evidence Snippets:**

- **E1:**

  The act of deprecation does not change any behavior of the resource. The presence of a Deprecation header field in a response is not meant to signal a change in the meaning or function of a resource in the context; consumers can still use the resource in the same way as they did before the resource was declared deprecated.

- **E2:**

  Deprecated resources function as they would have without sending the Deprecation header field, even though non-functional details may be affected (e.g., they have less efficiency and longer response times).

- **E3:**

  In cases where the Deprecation header field value is in the past, the client application developers MUST no longer assume that the behavior of the resource will remain the same as before the deprecation date.

**Evidence Summary:**

- (E1) Section 5 describes that deprecation does not change resource behavior.
- (E2) Section 7 notes that while functionality remains similar, non-functional details may be affected.
- (E3) Section 7 mandates that with a past deprecation date, clients must not assume that behavior remains unchanged.

**Fix Direction:**

Clarify the temporal semantics by explicitly distinguishing between the current behavior observed at deprecation and the future assumptions clients should make beyond the deprecation date.


**Severity:** Medium
  *Basis:* This ambiguity can lead to diverging client implementations and inconsistent risk assessments about resource stability once the deprecation date is passed.

**Confidence:** High

---

## Report 2: 9745-7-2

**Label:** Underspecification at the deprecation date equality boundary

**Bug Type:** Underspecification

**Explanation:**

The specification does not explicitly state how to interpret a deprecation date that is exactly equal to the current time, leaving implementers to decide whether it should be treated as a past or future event.

**Justification:**

- Boundary Expert Finding-2 points out that while the RFC differentiates between future and past deprecation dates, it does not clarify the case when the deprecation date equals the current time.

**Evidence Snippets:**

- **E4:**

  It conveys the deprecation date, which may be in the future (the resource in context will be deprecated at that date) or in the past (the resource in context was deprecated at that date).

- **E5:**

  In cases where the Deprecation header field value is in the past … and In cases where the Deprecation header field value is a date in the future …

**Evidence Summary:**

- (E4) Section 2.1 defines the deprecation date only in terms of being in the future or in the past.
- (E5) Section 7 distinguishes between past and future deprecation dates without explaining the equality scenario.

**Fix Direction:**

Define explicitly how the deprecation header should be interpreted when the deprecation date equals the current time, such as specifying whether it should be treated as expired or not yet effective.


**Severity:** Low
  *Basis:* This ambiguity is mainly interpretative and unlikely to cause severe interoperability issues since the header is advisory.

**Confidence:** High

---
