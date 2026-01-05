# Errata Reports

Total reports: 1

---


## Report 1: 9868-14-1

**Label:** Conflicting handling of overlong option Length (Section 10 vs Section 14)

**Bug Type:** Inconsistency

**Explanation:**

The specification defines two contradictory behaviors for options whose Length overruns the available surplus area: Section 10 mandates that any such overrun renders the entire options area malformed (all options must be discarded), while Section 14 instructs that offending options should be silently ignored.

**Justification:**

- Section 10 requires that options indicating overruns of the surplus area result in a malformed surplus area and all options must be discarded.
- Section 14, by contrast, recommends that any option whose length exceeds that of the UDP packet (i.e. would use bytes beyond the surplus area) SHOULD be silently ignored, implying that only the offending option is dropped.
- This divergence can lead to different implementations processing malformed packets in incompatible ways.

**Evidence Snippets:**

- **E1:**

  Options with inherently invalid Length field values, i.e., those that indicate underruns of the option itself or overruns of the surplus area (pointing past the end of the IP payload), MUST be treated as an indication of a malformed surplus area, and all options MUST silently be discarded.

- **E2:**

  Any options whose length exceeds that of the UDP packet (i.e., intending to use data that would have been beyond the surplus area) SHOULD be silently ignored (again to model legacy behavior).

**Evidence Summary:**

- (E1) Section 10 mandates complete discard of the options area for any overrun condition.
- (E2) Section 14 suggests that only the offending TLV should be ignored.

**Fix Direction:**

Revise the text in Section 14 to explicitly align with Section 10 by either requiring that an option that overruns the surplus area causes the entire options area to be discarded or by amending Section 10 so that only the offending option is skipped. The normative language (MUST vs SHOULD) must be reconciled.


**Severity:** High
  *Basis:* This ambiguity affects the fundamental parsing behavior of the options, potentially leading to divergent implementations and interoperability issues as well as impacting security assumptions when processing malformed packets.

**Confidence:** High

---