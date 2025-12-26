# Errata Reports

Total reports: 1

---

## Report 1: 9774-7-1

**Label:** Ambiguous 'obsoleting' vs 'deprecating' AS_SET/AS_CONFED_SET in RFC 9774

**Bug Type:** Inconsistency

**Explanation:**

The document inconsistently refers to AS_SET/AS_CONFED_SET by using 'deprecating' in Section 3 and 'obsoleting' in Section 7, which may mislead implementers about whether the segments should be merely treated as withdraw or completely stripped from processing.

**Justification:**

- Section 3 mandates deprecation and require treat‑as‑withdraw on reception, while Section 7 suggests that obsoleting these segments and removing related code could decrease the BGP attack surface.
- This difference in terminology and implied treatment might lead to conflicts with RFC 6811’s expectations that such segments remain recognizable for origin validation.

**Evidence Snippets:**

- **E1:**

  Earlier in RFC 9774, Section 3 explicitly says that this document *updates* RFC 4271 and RFC 5065 by deprecating AS_SET and AS_CONFED_SET and requiring “treat‑as‑withdraw” on reception; the title and body consistently talk about deprecation and prohibition of use, not a change to the underlying definitions or codepoints.

- **E2:**

  Section 7 then states: “Obsoleting these path segment types from BGP and the removal of the related code from implementations would potentially decrease the attack surface for BGP.”

- **E3:**

  the current wording is therefore somewhat inconsistent and could merit an editorial clarification, but it does not, by itself, create a hard cross‑RFC protocol contradiction.

**Evidence Summary:**

- (E1) Section 3 uses 'deprecating' for AS_SET/AS_CONFED_SET with a clear mandate for treat‑as‑withdraw.
- (E2) Section 7 uses 'obsoleting' suggesting complete removal of handling for these segments.
- (E3) This inconsistency is noted as an editorial concern that could mislead implementers.

**Fix Direction:**

Clarify Section 7 to align with Section 3 by avoiding the term 'obsoleting' and consistently describing the requirement to deprecate and treat as withdraw.


**Severity:** Low
  *Basis:* The issue is primarily editorial and causes potential confusion rather than a direct protocol violation.

**Confidence:** Medium

---
