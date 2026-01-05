# Errata Reports

Total reports: 2

---


## Report 4: 9868-11-4

**Label:** Contradictory Requirements for Overlong Option Lengths

**Bug Type:** Inconsistency

**Explanation:**

The document provides conflicting instructions by requiring that options with lengths overrunning the surplus area MUST trigger discarding of all options, while elsewhere such options SHOULD be silently ignored.

**Justification:**

- Section 10 states that options indicating overruns MUST lead to the discard of all UDP Options.
- Section 14, on the other hand, suggests that options that exceed the available UDP packet length SHOULD be silently ignored.

**Evidence Snippets:**

- **E1:**

  Options with inherently invalid Length field values, i.e., those that indicate underruns of the option itself or overruns of the surplus area (pointing past the end of the IP payload), MUST be treated as an indication of a malformed surplus area, and all options MUST silently be discarded.

- **E2:**

  Any options whose length exceeds that of the UDP packet (i.e., intending to use data that would have been beyond the surplus area) SHOULD be silently ignored (again to model legacy behavior).

**Evidence Summary:**

- (E1) Strict directive from Section 10 to discard all options on length overrun.
- (E2) Section 14 suggesting a softer approach by ignoring only the offending option.

**Fix Direction:**

Harmonize the treatment of overlong option lengths by specifying a single consistent rule for all cases.


**Severity:** Medium
  *Basis:* This discrepancy can lead to divergent receiver behaviors when processing malformed packets.

**Confidence:** High

---


## Report 5: 9868-11-5

**Label:** FRAG Terminal Format Figure Caption Mislabelled as Non-Terminal

**Bug Type:** Inconsistency

**Explanation:**

The caption for Figure 11 incorrectly labels the terminal FRAG Option format as non-terminal, which may mislead implementers regarding the correct format and length.

**Justification:**

- The prose clearly distinguishes that non-terminal fragments use the shorter format (Figure 10) and terminal fragments use the longer format (Figure 11).
- However, Figure 11’s caption incorrectly states 'UDP Non-Terminal FRAG Option Format' despite showing a 12-byte format with an additional RDOS field.

**Evidence Snippets:**

- **E1:**

  The FRAG Option has two formats: non-terminal fragments use the shorter variant (Figure 10) and terminal fragments use the longer (Figure 11).

- **E2:**

  Figure 11: UDP Non-Terminal FRAG Option Format

**Evidence Summary:**

- (E1) Text introducing the two distinct FRAG formats.
- (E2) The incorrect caption of Figure 11 that mislabels the terminal format.

**Fix Direction:**

Correct the caption of Figure 11 to read 'UDP Terminal FRAG Option Format' (optionally noting Len=12 and inclusion of the RDOS field).


**Severity:** Low
  *Basis:* This labeling error is unlikely to impact protocol interoperability but may cause temporary confusion during implementation.

**Confidence:** High

---