# Errata Reports

Total reports: 2

---


## Report 2: 9868-10-2

**Label:** Conflicting handling for options with length overruns

**Bug Type:** Inconsistency

**Explanation:**

The specification mandates two contradictory behaviors for options whose declared length overruns the valid surplus area: Section 10 requires discarding all options, while Section 14 instructs that only the offending option should be silently ignored.

**Justification:**

- Section 10 states that options with inherently invalid Length values (indicating an overrun) MUST result in discarding the entire options area.
- Section 14, however, recommends that options whose length exceeds that of the UDP packet SHOULD be ignored on an individual basis, leading to conflicting error-handling directives.

**Evidence Snippets:**

- **E1:**

  >> Options with inherently invalid Length field values, i.e., those that indicate underruns of the option itself or overruns of the surplus area (pointing past the end of the IP payload), MUST be treated as an indication of a malformed surplus area, and all options MUST silently be discarded.

- **E2:**

  >> Any options whose length exceeds that of the UDP packet (i.e., intending to use data that would have been beyond the surplus area) SHOULD be silently ignored (again to model legacy behavior).

**Evidence Summary:**

- (E1) Section 10 mandates that an overrun condition causes all options to be discarded.
- (E2) Section 14 instructs that an overrun option should be ignored individually.

**Fix Direction:**

Revise the text to resolve the conflict by explicitly defining a single consistent behavior for handling options with length overruns, for example by having Section 14 defer to the stricter rule in Section 10.


**Severity:** High
  *Basis:* Divergent implementations based on this conflict could lead to significant interoperability and security issues during error handling.

**Confidence:** High

---


## Report 4: 9868-10-4

**Label:** FRAG Option diagram caption mislabeling

**Bug Type:** Inconsistency

**Explanation:**

The caption of Figure 11 incorrectly labels the diagram as 'UDP Non-Terminal FRAG Option Format' even though the surrounding text and figure content indicate it should represent the terminal FRAG option format.

**Justification:**

- The text differentiates between the non-terminal FRAG option (depicted in Figure 10) and the terminal FRAG option, yet the caption for Figure 11 is misnamed.
- This mislabeling may lead readers to misinterpret the correct packet format for terminal fragments.

**Evidence Snippets:**

- **E1:**

  The FRAG Option has two formats: non-terminal fragments use the shorter variant (Figure 10) and terminal fragments use the longer (Figure 11).

- **E2:**

  Figure 11: UDP Non-Terminal FRAG Option Format

**Evidence Summary:**

- (E1) The text indicates a distinction between non-terminal and terminal FRAG options.
- (E2) The caption of Figure 11 incorrectly labels the terminal format as non-terminal.

**Fix Direction:**

Correct the caption for Figure 11 to read 'UDP Terminal FRAG Option Format' so that it aligns with the descriptive text and intended packet layout.


**Severity:** Low
  *Basis:* This is a documentation error that may cause confusion but does not affect the actual protocol processing.

**Confidence:** High

---