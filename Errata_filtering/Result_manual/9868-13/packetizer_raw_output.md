# Errata Reports

Total reports: 1

---


## Report 2: 9868-13-2

**Label:** Mislabelled FRAG Option Figure Caption for Terminal Fragment

**Bug Type:** Inconsistency

**Explanation:**

Figure 11 is incorrectly captioned as 'UDP Non-Terminal FRAG Option Format' even though the surrounding text explains that it represents the terminal FRAG Option format.

**Justification:**

- The text clearly states that non-terminal fragments use the shorter variant (Figure 10) and terminal fragments use the longer variant (Figure 11).
- Figure 11’s caption repeats 'Non-Terminal' despite later commentary indicating it is the terminal format.
- Although the different lengths (10 vs 12) help disambiguate, the caption error may still confuse readers who rely on the figure titles.

**Evidence Snippets:**

- **E1:**

  The FRAG Option has two formats: non-terminal fragments use the shorter variant (Figure 10) and terminal fragments use the longer (Figure 11).

- **E2:**

  Figure 10:

“Figure 10: UDP Non-Terminal FRAG Option Format”

- **E3:**

  Figure 11:

“Figure 11: UDP Non-Terminal FRAG Option Format”

Immediately after Figure 11: “The terminal FRAG Option format adds a Reassembled Datagram Option Start (RDOS) pointer…”

**Evidence Summary:**

- (E1) The text distinguishes between non-terminal (Figure 10) and terminal (Figure 11) FRAG Option formats.
- (E2) Figure 10 is correctly labelled as non-terminal.
- (E3) Figure 11 is mislabeled as non-terminal despite the subsequent description indicating a terminal format.

**Fix Direction:**

Change the caption under Figure 11 to 'UDP Terminal FRAG Option Format'.


**Severity:** Low
  *Basis:* The mislabel is an editorial error that may cause momentary confusion but is unlikely to impact interoperability owing to other disambiguating details (such as option length).

**Confidence:** High

---