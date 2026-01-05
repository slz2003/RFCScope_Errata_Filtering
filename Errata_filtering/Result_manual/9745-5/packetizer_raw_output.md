# Errata Reports

Total reports: 1

---


## Report 1: 9745-5-1

**Label:** Inconsistent Temporal Guidance on Deprecated Resource Behavior

**Bug Type:** Inconsistency

**Explanation:**

The specification offers conflicting guidance regarding resource behavior after deprecation: Section 5 implies that deprecation does not change behavior, while Section 7 mandates that clients must not assume the resource continues to behave as before once the deprecation date is past.

**Justification:**

- Section 5 provides descriptive language that deprecation does not alter the resource’s behavior, suggesting stable operational semantics.
- Section 7, however, imposes a normative requirement that once the deprecation date is in the past, clients must no longer assume unchanged behavior, creating a temporal and actor-role conflict.

**Evidence Snippets:**

- **E1:**

  Section 5: “The act of deprecation does not change any behavior of the resource. The presence of a Deprecation header field in a response is not meant to signal a change in the meaning or function of a resource in the context; consumers can still use the resource in the same way as they did before the resource was declared deprecated.”

- **E2:**

  Section 7: “In cases where the Deprecation header field value is in the past, the client application developers MUST no longer assume that the behavior of the resource will remain the same as before the deprecation date.”

**Evidence Summary:**

- (E1) Section 5 describes deprecation as having no impact on resource behavior.
- (E2) Section 7 instructs clients to drop any assumption of unchanged behavior after the deprecation date.

**Fix Direction:**

Clarify that Section 5’s statement applies only at the point of signaling deprecation and explicitly define the transition to the guidance in Section 7 regarding client assumptions post-deprecation.


**Severity:** Medium
  *Basis:* Multiple expert analyses highlight that this conflicting temporal guidance could lead to divergent client interpretations and implementation strategies, affecting interoperability.

**Confidence:** High

---