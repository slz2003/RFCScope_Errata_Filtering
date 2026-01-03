# Errata Reports

Total reports: 2

---

## Report 1: draft-ietf-tcpm-accurate-ecn-34-7-1

**Label:** No Structural or Syntactic Issues in IANA Actions

**Bug Type:** None

**Explanation:**

The IANA Considerations section is fully consistent with the draft’s internal definitions and exhibits no structural or syntactic bugs.

**Justification:**

- The expert confirmed that bit 7 is consistently used as AE, and all procedural and administrative instructions to IANA match the technical details without introducing ambiguity (E1).

**Evidence Snippets:**

- **E1:**

  The IANA Considerations section cleanly matches the technical definitions elsewhere in the draft: bit 7 is consistently used as AE (Accurate ECN), aligning with the diagrams and negotiation rules, and the text correctly notes its prior use as NS by RFC 3540 and its historic status per RFC 8311. The two AccECN option kinds (172 and 174) are used consistently throughout (figures, normative text, and IANA table), and their variable-length nature (with specific allowed lengths defined earlier) is compatible with using “N” in the registry. The instructions to IANA about updating registry references and notes are procedural/administrative rather than wire-format- or grammar‑affecting. They do not introduce ambiguity about bit positions, field sizes, option formats, or encodings, and therefore do not constitute structural or syntactic bugs from an implementability standpoint.

**Evidence Summary:**

- (E1) The section is consistent in its use of bit 7 as AE and in the handling of option kinds, with no ambiguity in technical definitions or IANA instructions.

**Severity:** Low
  *Basis:* The issue is determined to have no impact on implementability as all technical details are consistent.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-1

---

## Report 2: draft-ietf-tcpm-accurate-ecn-34-7-2

**Label:** Omission of 'Updates: 8311' Reference in IANA Considerations

**Bug Type:** Editorial

**Explanation:**

The draft omits an explicit 'Updates: 8311' reference which might be expected given the change in the status of TCP header flag bit 7.

**Justification:**

- The expert noted that while the omission does not cause implementer-visible ambiguity, it is an editorial matter that could benefit from explicit cross-referencing for clarity (E2).

**Evidence Snippets:**

- **E2:**

  Finally, while one could argue that this document “logically” updates RFC 8311 (since it changes the status of bit 7 from Reserved to AE), RFC 8311’s only normative action on that bit is via IANA, and there is no ongoing behavioral requirement that would be contradicted; omitting “Updates: 8311” is at most an editorial matter and does not create implementer-visible ambiguity or registry inconsistency.

**Evidence Summary:**

- (E2) The text explains that omitting “Updates: 8311” is an editorial matter, even though it might be logically expected given the reclassification of bit 7.

**Severity:** Low
  *Basis:* The issue is editorial; the lack of an explicit update reference does not affect technical functionality.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Editorial comment on omission of Updates: 8311

---
