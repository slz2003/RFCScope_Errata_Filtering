# Errata Reports

Total reports: 4

---

## Report 1: 9868-10-1

**Label:** Ambiguous responsibilities for concurrent use of AUTH and UENC options

**Bug Type:** Underspecification

**Explanation:**

The specification states that AUTH and UENC MUST NOT be used concurrently but fails to clearly assign responsibilities to senders versus receivers, leaving ambiguity in error handling when both options appear in a packet.

**Justification:**

- The excerpt only specifies the prohibition without detailing whether senders must omit one option or how receivers should react if both are present.
- This underspecification may lead to divergent behaviors across implementations.

**Evidence Snippets:**

- **E1:**

  >> AUTH and UENC MUST NOT be used concurrently.

- **E2:**

  The subject of the requirement is “AUTH and UENC” rather than a concrete actor. For conformance, an implementer needs to know at least: (a) that *senders* MUST NOT construct a UDP packet containing both AUTH and UENC options, and (b) what a *receiver* MUST do if it nonetheless receives such a packet (e.g., ignore one option, ignore both, or drop the entire datagram and/or its user data).

**Evidence Summary:**

- (E1) The prohibition is stated without specifying actor responsibilities.
- (E2) The text does not indicate what a receiver should do when both options are present.

**Fix Direction:**

Clarify the normative text by specifying separate requirements for senders (prohibiting construction of packets with both options) and receivers (defining the precise response when both options are encountered).


**Severity:** Low
  *Basis:* The ambiguity is limited to error handling for malformed packets and is unlikely to affect well‐formed traffic.

**Confidence:** High

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

## Report 3: 9868-10-3

**Label:** Overlapping and conflicting safe option length handling rules

**Bug Type:** Inconsistency/Underspecification

**Explanation:**

The specification presents overlapping rules for handling safe options with invalid lengths—such as lengths that are too short—resulting in contradictory mandates to either discard the entire options area or ignore only the malformed option.

**Justification:**

- Section 10 requires that option lengths smaller than the minimum for the corresponding kind trigger a global error and lead to discarding all options.
- Simultaneously, the text mandates that malformed safe options should be silently ignored on a per-option basis, and the APC example further complicates this by equating unrecognized lengths with a soft failure similar to a checksum error.

**Evidence Snippets:**

- **E1:**

  >> Option Lengths (or Extended Lengths, where applicable) smaller than the minimum for the corresponding Kind MUST be treated as an error. Such errors call into question the remainder of the surplus area and thus MUST result in all UDP Options being silently discarded.

- **E2:**

  >> Receivers supporting UDP Options MUST silently ignore unknown or malformed SAFE Options (i.e., in the same way a legacy receiver would ignore all UDP Options). An option is malformed when its length does not indicate (one of) the value(s) stated in the option's specification. A malformed FRAG Option is an exception to this rule; it SHALL be treated as an unsupported UNSAFE Option.

- **E3:**

  >> UDP packets with unrecognized APC lengths MUST receive the same treatment as UDP packets with incorrect APC Option checksum fields.

**Evidence Summary:**

- (E1) The rule for lengths smaller than the minimum mandates a global discard of all options.
- (E2) The rule for malformed safe options calls for ignoring only the offending option.
- (E3) The APC example demonstrates a specific case where these overlapping directives conflict.

**Fix Direction:**

Clarify and separate the error-handling criteria by defining distinct, non-overlapping categories for safe option length errors and by explicitly stating the precedence among global and per-option error treatments.


**Severity:** High
  *Basis:* The overlapping rules can lead to unpredictable behavior and interoperability failures, particularly affecting processing of safe options such as APC in malformed packets.

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
