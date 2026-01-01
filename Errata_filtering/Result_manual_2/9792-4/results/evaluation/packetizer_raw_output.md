# Errata Reports

Total reports: 2

---

## Report 1: 9792-4-1

**Label:** OSPFv3 Registry Text Refers to OSPFv2 sub-TLV in Section 4.2.2

**Bug Type:** Inconsistency

**Explanation:**

Section 4.2.2 mistakenly states that the OSPFv3 Prefix Extended Flags registry defines bits in the OSPFv2 Prefix Extended Flags sub-TLV, which conflicts with the intended protocol-specific binding.

**Justification:**

- Section 4.2.2’s text uses 'OSPFv2 Prefix Extended Flags sub-TLV' even though the registry is clearly meant for OSPFv3, as the registry name and surrounding context indicate (E1, E2).
- Multiple experts (Scope, Deontic, CrossRFC, and Terminology) note that this copy‑and‑paste error confuses the mapping between registries and protocol sub-TLVs, potentially impacting interoperability (E3).

**Evidence Snippets:**

- **E1:**

  Section 4.2.2 (OSPFv3): “The registry defines the bits in the Prefix Extended Flags field in the OSPFv2 Prefix Extended Flags sub-TLV as specified in Section 2.”

- **E2:**

  Section 2: “This document defines a variable-length Prefix Extended Flags sub-TLV for OSPFv2 and OSPFv3.” and later distinguishes “The OSPFv2 Prefix Extended Flags sub-TLV…” from “The OSPFv3 Prefix Extended Flags sub-TLV…”.

- **E3:**

  Deontic Analysis: “IANA has created the ‘OSPFv3 Prefix Extended Flags’ registry within the ‘Open Shortest Path First v3 (OSPFv3) Parameters’ registry group.”

**Evidence Summary:**

- (E1) Shows that Section 4.2.2 incorrectly refers to the OSPFv2 sub-TLV in the context of OSPFv3.
- (E2) Indicates that Section 2 explicitly distinguishes between the OSPFv2 and OSPFv3 sub-TLVs.
- (E3) Confirms the intended creation of a separate OSPFv3 registry.

**Fix Direction:**

In Section 4.2.2, replace 'OSPFv2 Prefix Extended Flags sub-TLV' with 'OSPFv3 Prefix Extended Flags sub-TLV'.

**Severity:** Medium
  *Basis:* The error could lead to misinterpretation of the registry's scope and result in interoperability/documentation issues.

**Confidence:** High

**Experts mentioning this issue:**

- ScopeExpert: Issue-1
- DeonticExpert: Issue-1
- CrossRFCExpert: Issue-1
- TerminologyExpert: Issue-1

---

## Report 2: 9792-4-2

**Label:** Misleading Phrase 'flags defined in this document' in Registry Sections

**Bug Type:** Underspecification

**Explanation:**

Both registry sections (4.1.2 and 4.2.2) include the phrase 'flags defined in this document' even though no bits or flags are defined in the RFC.

**Justification:**

- Each registry section states 'No bits are currently defined' and then immediately follows with a reference to 'the flags defined in this document', creating a contradictory message (E1, E2).
- The intended guidance appears to be for future allocations in the registry rather than implying that the RFC itself defines flags.

**Evidence Snippets:**

- **E1:**

  Section 4.1.2 (OSPFv2 registry): “No bits are currently defined. Bits 0-31 are to be initially marked as ‘Unassigned’.’ Immediately followed by: “The flags defined in this document may use either a single bit or multiple bits to represent a state, as determined by the specific requirements of the document defining them.”

- **E2:**

  Section 4.2.2 (OSPFv3 registry) uses the same pair of sentences: “No bits are currently defined. Bits 0-31 are to be initially marked as ‘Unassigned’.’ and “The flags defined in this document may use either a single bit or multiple bits to represent a state, as determined by the specific requirements of the document defining them.”

**Evidence Summary:**

- (E1) Demonstrates the contradictory wording in Section 4.1.2 regarding flag definitions.
- (E2) Shows that Section 4.2.2 replicates the same misleading phrase.

**Fix Direction:**

Replace 'the flags defined in this document' with 'Flags defined in this registry' in both Sections 4.1.2 and 4.2.2.

**Severity:** Low
  *Basis:* This is an editorial/terminology issue that may cause confusion but does not affect current implementations.

**Confidence:** High

**Experts mentioning this issue:**

- TerminologyExpert: Issue-2

---
