# Errata Reports

Total reports: 1

---


## Report 1: draft-ietf-tcpm-accurate-ecn-34-1-1

**Label:** Inconsistent RFC references for 'Reno' (RFC6582 vs RFC5681)

**Bug Type:** Inconsistency

**Explanation:**

The draft inconsistently references the Reno congestion control algorithm by citing RFC6582—which actually defines NewReno—in one part and RFC5681 in another, potentially misleading readers about the intended baseline behavior.

**Justification:**

- The CrossRFC expert analysis notes that the introductory paragraph cites 'Reno [RFC6582]' while a later sentence correctly associates standard Reno with RFC5681, creating a cross-RFC inconsistency (E1, E2).
- The Terminology expert analysis reinforces that this mislabelling may confuse implementers or readers regarding the actual reference for Reno, suggesting the citation should be standardized (E1, E2).

**Evidence Snippets:**

- **E1:**

  This is sufficient for congestion control scheme like Reno [RFC6582] and Cubic [RFC9438]… (Section 1, first paragraph)

- **E2:**

  Like Classic ECN feedback, AccECN can be used by standard Reno or CUBIC congestion control [RFC5681] [RFC9438]… (later in Section 1)

**Evidence Summary:**

- (E1) shows the initial use of RFC6582 for Reno; (E2) shows the later correct use of RFC5681 for standard Reno.

**Fix Direction:**

In Section 1, first paragraph, modify the citation to use either 'Reno [RFC5681]' for standard Reno or 'NewReno [RFC6582]' if the intent is to reference the NewReno modification, ensuring consistent RFC references throughout the document.

**Severity:** Low
  *Basis:* Both expert analyses highlight that while the inconsistency could cause confusion, its technical impact is minimal.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC: Issue-1
- Terminology: Issue-1

---