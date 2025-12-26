# Errata Reports

Total reports: 1

---

## Report 1: 9868-25-1

**Label:** Ambiguous Wording in Section 25.3: Conflation of 'Required Options' with Must-Support Options and Inconsistent TLV Terminology

**Bug Type:** Inconsistency

**Explanation:**

Section 25.3 uses imprecise language by stating that 'required options come first and at most once each' and by referring to 'non-zero TLVs', which conflates the defined must‑support options (with their precise ordering and repetition rules in Section 10) with API-required options and introduces inconsistent terminology.

**Justification:**

- The candidate issue notes that Section 25.3’s blanket statement is overbroad compared to the precise ordering and duplicate‐handling rules defined in Section 10, and it ambiguously uses the term 'required options' without distinguishing it from 'must‑support options' (Scope Expert Issue-1).
- The terminology analysis further highlights that the use of 'required options' and 'non-zero TLVs' in Section 25.3 is inconsistent with earlier definitions (e.g., 'must‑support options' and 'non‐padding options') and may mislead implementers about which options are subject to ordering and repetition constraints.

**Evidence Snippets:**

- **E1:**

  Section 10:
- Must-support definition and ordering: “An endpoint supporting UDP Options MUST support those marked with an ‘*’ above… These are called ‘must-support’ options.” and “‘Must-support’ options other than NOP and EOL MUST be placed by the transmitter before other SAFE UDP Options.”
- Repetition rules: “Other than FRAG, NOP, EXP, and UEXP, each option SHOULD NOT occur more than once in a single UDP datagram. If an option other than these four occurs more than once, a receiver MUST interpret only the first instance of that option and MUST ignore later instances.”
- FRAG multiple-use rule: “If FRAG occurs more than once, the options area MUST be considered malformed and MUST NOT be processed.”

- **E2:**

  Section 25.3: “Because required options come first and at most once each (with the exception of NOPs, which never need to come in sequences of more than seven in a row), their DoS impact is limited.”

- **E3:**

  Section 25.3: 
“Implementations concerned with the potential for UDP Options introducing a vulnerability MAY implement only the required UDP Options and SHOULD also limit processing of TLVs, in number of non-padding options, total length, or both. The number of non-zero TLVs allowed in such cases MUST be at least as many as the number of concurrent options supported with an additional few to account for unexpected unknown options …”

**Evidence Summary:**

- (E1) Section 10 provides the normative definitions for must‑support options, including their required ordering and duplicate handling.
- (E2) Section 25.3 states that 'required options come first and at most once each', which is an overbroad summary that may be misinterpreted.
- (E3) Section 25.3 further uses ambiguous phrases such as 'required UDP Options' and 'non-zero TLVs' that conflict with earlier defined terminology.

**Fix Direction:**

In Section 25.3, replace the phrase 'required options' with 'must‑support options (other than NOP and EOL)' and use consistent terminology by replacing 'non-zero TLVs' with 'non‑padding TLVs'. This clarification will align the security guidance with the precise normative rules in Section 10.


**Severity:** Medium
  *Basis:* While the normative requirements remain clear in Section 10, the ambiguous language in Section 25.3 could mislead implementers and result in over-restrictive DoS mitigations.

**Confidence:** High

---
