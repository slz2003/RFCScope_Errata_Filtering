# Errata Reports

Total reports: 1

---

## Report 1: 9859-3-1

**Label:** Ambiguous uniqueness scope for DSYNC records (global vs per owner name)

**Bug Type:** Underspecification

**Explanation:**

The specification’s statement 'There MUST NOT be more than one DSYNC record for each combination of RRtype and Scheme' is ambiguous regarding whether it applies globally across the _dsync subtree or is limited to each owner name.

**Justification:**

- Section 3 mandates the uniqueness constraint without clarifying its granularity, leaving it open to a global interpretation (E1).
- Child-specific publication examples in Section 3.2 imply that the intended constraint should apply per owner name, not globally (E2).

**Evidence Snippets:**

- **E1:**

  Section 3: “Parent operators … MUST publish endpoint information using the record type defined in Section 2 under the _dsync subdomain of the parent zone, as described in the following subsections.  
        There MUST NOT be more than one DSYNC record for each combination of RRtype and Scheme.”

- **E2:**

  Section 3.2 (Child-specific): “It is also possible to publish child-specific records where the parent zone's labels are stripped from the child's Fully Qualified Domain Name (FQDN), and the result is used in place of the wildcard label,” with an example `child._dsync.example.  IN DSYNC  CDS NOTIFY 5300 rr-endpoint.example.`

**Evidence Summary:**

- (E1) Normative uniqueness clause in Section 3 does not specify the granularity of the constraint.
- (E2) The child-specific example in Section 3.2 implies a per owner name interpretation.

**Fix Direction:**

Clarify the uniqueness constraint in Section 3 by specifying that it applies at each owner name under the _dsync subtree (e.g., 'At any given owner name, there MUST NOT be more than one DSYNC record for any given combination of RRtype and Scheme').

**Severity:** Medium
  *Basis:* Multiple expert analyses (Deontic and Terminology) indicate that a global reading would unnecessarily restrict deployment flexibility, while the intended use case supports a per owner name constraint.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Causal Expert: Issue-1
- Deontic Expert: Issue-1
- Terminology Expert: Issue-1

---
