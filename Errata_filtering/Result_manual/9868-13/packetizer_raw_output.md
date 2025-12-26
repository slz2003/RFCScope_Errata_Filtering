# Errata Reports

Total reports: 3

---

## Report 1: 9868-13-1

**Label:** Ambiguous and conflicting rules for modifying UDP packet content by UNSAFE options in Section 13

**Bug Type:** Inconsistency

**Explanation:**

Section 13 ambiguously prohibits modifying UDP packet content outside an option’s own field while simultaneously allowing UNSAFE options (e.g., UCMP/UENC) to modify UDP user data and later options, creating a potential conflict in interpretation.

**Justification:**

- Section 10 permits options to modify later options (e.g., for compression as expected for UCMP) while Section 13 lists UDP user data and surplus area as areas that must remain unmodified.
- Sections 12.1/12.2 describe UNSAFE options as expected to operate on UDP user data and later options, which conflicts with the blanket prohibition in Section 13.
- There is residual uncertainty on whether reserved options such as AUTH/UENC/UCMP are intended to be exempt from Section 13’s dependency/modification restrictions.

**Evidence Snippets:**

- **E1:**

  Section 10: “This does not prohibit options that modify later options (in order of appearance within a packet), such as would typically be the case for compression (UCMP).”

- **E2:**

  Section 12.1: “The UNSAFE Compression (UCMP, Kind=192) Option is reserved for all UDP compression mechanisms. UCMP is expected to cover the UDP user data and some (e.g., later, in sequence) UDP Options.”

- **E3:**

  Section 13: “At the sender, new options MUST NOT modify UDP packet content anywhere outside their option field, excepting only UNSAFE Options; areas that need to remain unmodified include the IP header, IP options, UDP user data, and surplus area (i.e., other options).”

- **E4:**

  Section 13 (later): “UNSAFE Options MAY modify the UDP user data, e.g., by encryption, compression, or other transformations. All other (SAFE) options MUST NOT modify the UDP user data.”

**Evidence Summary:**

- (E1) Section 10 allows modification of later options as with UCMP.
- (E2) Section 12.1 describes UNSAFE options (e.g., UCMP) as expected to cover UDP user data and later options.
- (E3) Section 13 prohibits modifying UDP packet content outside the option field except for UNSAFE options, listing UDP user data and surplus area as off‐limits.
- (E4) Section 13 later explicitly permits UNSAFE options to modify UDP user data, which creates ambiguity.

**Fix Direction:**

Revise Section 13 to explicitly differentiate between SAFE and UNSAFE options—for example, restrict the unmodifiable areas clause to SAFE options and clearly state that UNSAFE options may modify UDP user data and later options.


**Severity:** Medium
  *Basis:* The ambiguity may lead to misinterpretation by future option designers and could result in inconsistent implementations if UNSAFE options are improperly constrained.

**Confidence:** High

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
