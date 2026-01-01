# Errata Reports

Total reports: 2

---

## Report 1: 9906-1-1

**Label:** Inconsistent RFC Metadata: Header lists 'Unknown' vs. Summary references RFC 9906

**Bug Type:** Metadata Inconsistency

**Explanation:**

The header metadata indicates 'RFC: Unknown' and 'Section: Unknown' while the excerpt summary clearly references Section 1 of RFC 9906, creating an internal inconsistency.

**Justification:**

- The header block shows 'RFC: Unknown' and 'Section: Unknown' (E1).
- The summary states 'Section 1 of RFC 9906 gives background…' (E2).

**Evidence Snippets:**

- **E1:**

  RFC: Unknown
Section: Unknown
Model: gpt-5.1

- **E2:**

  Excerpt Summary: Section 1 of RFC 9906 gives background for deprecating the older GOST R 34.10-2001 / 34.11-94 algorithms in DNSSEC, points to RFC 5933 and RFC 9558 for technical details, and states that the newer 2012 GOST algorithms’ requirement levels are unchanged. Section 1.1 provides standard BCP 14 keyword boilerplate.

**Evidence Summary:**

- (E1) The header metadata lists 'RFC: Unknown' and 'Section: Unknown'.
- (E2) The excerpt summary references 'Section 1 of RFC 9906'.

**Fix Direction:**

Update the header metadata to correctly reflect the RFC and section referenced in the summary, or adjust the summary to match the header metadata.

**Severity:** Low
  *Basis:* The issue is confined to metadata inconsistency and does not affect the technical or normative content.

**Confidence:** High

**Experts mentioning this issue:**

- Router: Metadata Inconsistency

---

## Report 2: 9906-1-2

**Label:** Underspecified Candidate Issue: Missing details in Candidate Issues block

**Bug Type:** Underspecified Candidate Issue

**Explanation:**

A candidate issue is referenced in the candidate issues block without any additional details such as type, label, or relevant dimensions, leaving its intent unclear.

**Justification:**

- The candidate issues block notes a single issue with 'Type: None', and provides an empty label and no relevant dimensions (E3).

**Evidence Snippets:**

- **E3:**

  Candidate Issues: 1

  Issue 1:
    Type: None
    Label: 
    Relevant Dimensions: 
    Sketch: ...

**Evidence Summary:**

- (E3) The candidate issues block shows an issue entry with missing type, label, and relevant dimensions.

**Fix Direction:**

Provide detailed information for the candidate issue, including a clear type, descriptive label, and specification of relevant dimensions or sketch.

**Severity:** Low
  *Basis:* Since the candidate issue is underspecified, it is not clear if it poses a technical risk; however, clarification is needed.

**Confidence:** Medium

**Experts mentioning this issue:**

- Router: Issue 1

---
