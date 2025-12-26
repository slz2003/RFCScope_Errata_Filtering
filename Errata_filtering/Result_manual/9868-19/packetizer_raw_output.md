# Errata Reports

Total reports: 2

---

## Report 1: 9868-19-1

**Label:** Section 19 'MUST ignore' Rule Conflicts with Configurable Mandatory Options

**Bug Type:** Inconsistency

**Explanation:**

Section 19 mandates that all non‑UNSAFE UDP Options be ignored on failure without qualification, which conflicts with later sections that allow configuration to mandate option enforcement and drop packets.

**Justification:**

- Section 19 states that non‑UNSAFE options MUST be ignored if not supported or upon failure, implying an unchangeable behavior.
- Other sections (e.g., Section 11.3, Section 14, and Section 15) explicitly permit receivers to override default behavior and treat certain options as mandatory, dropping packets if they are missing or invalid.

**Evidence Snippets:**

- **E1:**

  Section 19: “All UDP Options other than UNSAFE ones MUST be ignored if not supported or upon failure (e.g., APC).” and “All UDP Options that fail MUST result in the UDP data still being sent to the application layer by default to ensure equivalence with legacy devices.”

- **E2:**

  Section 11.3 (APC): “Like all SAFE UDP Options, the APC MUST be silently ignored when failing, unless the receiver has been explicitly configured to do otherwise.”

**Evidence Summary:**

- (E1) Section 19 imposes an unconditional rule to ignore non‑UNSAFE options, suggesting it applies in all cases.
- (E2) Section 11.3 clarifies that the default behavior can be overridden by explicit configuration.

**Fix Direction:**

Revise Section 19 to explicitly state that its rules apply by default and are subject to configuration overrides, aligning its scope with Sections 6, 11, 14, and 15.


**Severity:** Medium
  *Basis:* The conflict may lead to implementations that either disregard necessary security checks or misapply the default behavior, affecting interoperability and security.

**Confidence:** High

---

## Report 2: 9868-19-2

**Label:** Section 19 Over-Broad Failure Clause Does Not Exclude UNSAFE Options

**Bug Type:** Inconsistency & Underspecification

**Explanation:**

Section 19 states that all UDP Options that fail must still result in the UDP data being sent without explicitly excluding UNSAFE options, which contradicts other sections that mandate dropping data when UNSAFE options fail or are misused.

**Justification:**

- Section 19’s clause applies generically to all UDP Options, whereas Sections 12 and 14 specify that failure or misuse of UNSAFE options must lead to the dropping of UDP user data.
- This ambiguous wording may cause implementers to mistakenly deliver user data even when UNSAFE options are unsupported or incorrectly placed.

**Evidence Snippets:**

- **E3:**

  Section 19: “All UDP Options that fail MUST result in the UDP data still being sent to the application layer by default to ensure equivalence with legacy devices.”

- **E4:**

  Section 12: “Receivers supporting UDP Options MUST silently drop the UDP user data of the reassembled datagram if any fragment or the entire datagram includes an UNSAFE Option whose Kind is not supported or if an UNSAFE Option appears outside the context of a fragment or reassembled fragments.”

- **E5:**

  Section 14: “Unless configuration settings direct otherwise, all options except UNSAFE Options MUST result in the UDP user data being passed to the upper layer protocol or application, regardless of whether all options are processed, are supported, or succeed.”

**Evidence Summary:**

- (E3) Section 19 mandates that failing options must still deliver UDP data by default without distinguishing UNSAFE options.
- (E4) Section 12 requires that UDP user data be dropped when an UNSAFE option is unsupported or misused.
- (E5) Section 14 reinforces that the default data delivery rule does not apply to UNSAFE options.

**Fix Direction:**

Modify Section 19 to limit its failure clause to SAFE UDP Options and explicitly refer to Section 12 for the behavior required with UNSAFE options.


**Severity:** Medium
  *Basis:* Ambiguity in the exclusion of UNSAFE options may lead to erroneous implementations that compromise the intended security and reliability of the protocol.

**Confidence:** High

---
