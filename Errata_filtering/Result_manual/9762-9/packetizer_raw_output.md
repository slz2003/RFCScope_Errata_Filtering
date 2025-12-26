# Errata Reports

Total reports: 3

---

## Report 1: 9762-9-1

**Label:** Ambiguous Boolean Condition in RA 'No DHCPv6 info' Note Mixing RA Header and PIO P Flag

**Bug Type:** Inconsistency

**Explanation:**

The updated RA note in RFC 9762 ambiguously aggregates the RA header flags (M and O) with the per-PIO P flag using an 'or' condition, which may cause valid RA advertisements to be misinterpreted as carrying no DHCPv6 information.

**Justification:**

- The original RFC 4861 note states that 'if neither M nor O flags are set' there is no DHCPv6 information, whereas RFC 9762 replaces it with a condition that is true whenever any of the M, O, or P flags is not set.
- The P flag is defined as a 1‑bit flag in the Prefix Information Option rather than as an RA header flag, creating a scope mix-up between header-level and per-prefix indicators.

**Evidence Snippets:**

- **E1:**

  Original RFC 4861 note: “If neither M nor O flags are set, this indicates that no information is available via DHCPv6.”

- **E2:**

  RFC 9762 Section 9.1 replacement: “If the M, O, or P (RFC 9762) flags are not set, this indicates that no information is available via DHCPv6.”

- **E3:**

  P is defined earlier in RFC 9762 as a 1‑bit flag in the PIO format (per prefix) after the R flag, not as an RA‑header flag alongside M and O.

**Evidence Summary:**

- (E1) shows the original condition based only on M and O,
- (E2) shows the new ambiguous condition that includes P,
- (E3) clarifies that the P flag is a per-PIO flag.

**Fix Direction:**

Replace the note with explicit language that states: 'If both the M and O flags in the Router Advertisement header are zero and no Prefix Information Option has the P flag set, then the advertisement carries no DHCPv6 information.'


**Severity:** High
  *Basis:* The ambiguity could lead to misinterpretation whereby valid DHCPv6 advertisements (with either M or a properly set P flag) may be incorrectly treated as carrying no information, potentially disabling DHCPv6 functionality.

**Confidence:** High

---

## Report 2: 9762-9-2

**Label:** Conflict Between RA Note Negative Interpretation and P Flag's Purely Positive Semantics

**Bug Type:** Inconsistency

**Explanation:**

The RA note in RFC 9762 treats the absence of the P flag as indicating no DHCPv6 information, which conflicts with Section 7.3 that stipulates the P flag is a purely positive indicator and its absence must not be interpreted as negative evidence for DHCPv6-PD.

**Justification:**

- RFC 9762 Section 7.3 explicitly states that the P flag is solely a positive indicator and that the absence of any PIO with the P flag set should not be taken to mean DHCPv6-PD is absent.
- By including the P flag in the RA note's condition, the text may mislead implementers into disabling DHCPv6-PD when an RA lacks a P indication, contrary to the normative behavior outlined in Section 7.3.

**Evidence Snippets:**

- **E1:**

  RFC 9762 Section 7.3: “The P bit is purely a positive indicator, telling nodes that DHCPv6 prefix delegation is available… The absence of any PIOs with the P bit does not carry any kind of signal to the opposite and MUST NOT be processed to mean that DHCPv6-PD is absent. In particular, nodes… MUST NOT disable DHCPv6-PD on the absence of PIOs with the P bit set.”

- **E2:**

  RFC 9762 Section 9.1 (new text): “Note: If the M, O, or P (RFC 9762) flags are not set, this indicates that no information is available via DHCPv6.”

**Evidence Summary:**

- (E1) specifies the intended positive-only use of the P flag,
- (E2) shows the RA note that incorrectly uses the absence of P to indicate no DHCPv6 information.

**Fix Direction:**

Clarify that the RA note describes only the content of the advertisement and that the absence of the P flag does not imply the absence of DHCPv6-PD, thereby aligning with the normative guidance in Section 7.3.


**Severity:** High
  *Basis:* If misinterpreted, the conflict may lead to implementations disabling DHCPv6-PD, adversely affecting network behavior in environments that rely on DHCPv6 prefix delegation.

**Confidence:** High

---

## Report 3: 9762-9-3

**Label:** Inconsistent Naming of Prefix Information Option

**Bug Type:** Inconsistency 

**Explanation:**

There exists a naming discrepancy between 'Prefix-Information option' and 'Prefix Information Option' in the text, which may cause minor editorial confusion.

**Justification:**

- The old text in RFC 4862 uses 'Prefix-Information option', while the updated text in RFC 9762 uses 'Prefix Information Option', matching IANA registry conventions.
- Although the underlying functionality remains unchanged, such inconsistencies can be confusing for readers comparing the texts.

**Evidence Snippets:**

- **E1:**

  Section 9.2 OLD TEXT (from RFC 4862 Section 5.5.3): “For each Prefix-Information option in the Router Advertisement:”

- **E2:**

  Section 9.2 NEW TEXT (in RFC 9762): “For each Prefix Information Option in the Router Advertisement:”

**Evidence Summary:**

- (E1) documents the hyphenated form,
- (E2) shows the non-hyphenated form used in the updated text.

**Fix Direction:**

Standardize the naming throughout the document to 'Prefix Information Option' to ensure textual consistency with the IANA registry and RFC 9762.


**Severity:** Low
  *Basis:* This issue is purely editorial with no impact on interoperability or protocol semantics.

**Confidence:** High

---
