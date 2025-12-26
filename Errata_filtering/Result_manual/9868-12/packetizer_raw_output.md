# Errata Reports

Total reports: 2

---

## Report 1: 9868-12-1

**Label:** Underspecified handling of supported UNSAFE option failures

**Bug Type:** Underspecification

**Explanation:**

The framework lacks a clear, normative rule specifying what to do when a supported UNSAFE option (e.g., UENC, UCMP) fails internally, which may lead to divergent behavior in implementation.

**Justification:**

- Section 11.4 indicates that UNSAFE options must discard the UDP datagram on failure, yet Section 12 only mandates dropping user data for unsupported kinds or out‐of-context usage, leaving internal failures unaddressed.
- Multiple expert analyses (Temporal, Scope, Deontic, and Boundary) note that the normative language does not define behavior for supported UNSAFE option failures such as decryption/MAC or compression errors.

**Evidence Snippets:**

- **E1:**

  Per-datagram UDP Options… Processing of those options occurs after reassembly is complete. This enables the safe use of UNSAFE Options, which are required to result in discarding the entire UDP datagram if they are unknown to the receiver or otherwise fail (see Section 11.4).

- **E2:**

  Receivers supporting UDP Options MUST silently drop the UDP user data of the reassembled datagram if any fragment or the entire datagram includes an UNSAFE Option whose Kind is not supported or if an UNSAFE Option appears outside the context of a fragment or reassembled fragments.

- **E3:**

  The framework never states, with BCP 14 language, what must happen when an UNSAFE option of a *supported* Kind fails according to its own specification (e.g., UENC decryption/MAC failure, UCMP decompression failure, option-specific internal error).

**Evidence Summary:**

- (E1) Section 11.4 indicates that UNSAFE options should cause discarding of the UDP datagram when they fail, but does not cover failures in supported options.
- (E2) Section 12 mandates dropping user data only when the UNSAFE option is unsupported or misused.
- (E3) There is no normative language detailing actions for internal failure of a supported UNSAFE option.

**Fix Direction:**

In Section 12 (or in related sections such as 14/19), add a clear, BCP‑14–style normative rule stating that any internal failure of a supported UNSAFE option (e.g., decryption, MAC, or decompression failure) MUST result in the UDP user data being dropped (or delivered as a zero-length datagram).


**Severity:** Medium
  *Basis:* Multiple experts highlighted that the underspecification could lead to divergent implementations, posing interoperability and security risks.

**Confidence:** High

---

## Report 2: 9868-12-2

**Label:** Contradictory rules on failed UNSAFE options: drop user data vs. deliver UDP data

**Bug Type:** Inconsistency

**Explanation:**

There is a conflict between sections: while Section 12 mandates that user data be dropped when an UNSAFE option is unsupported or misused, Section 19 instructs that failed options still result in the UDP data being sent, creating ambiguity in receiver behavior.

**Justification:**

- Deontic analysis shows that Section 12 requires dropping the UDP user data for UNSAFE options in error cases, whereas Section 19's broad statement mandates delivering UDP data even on option failure.
- Boundary analysis emphasizes that the global statement in Section 19 contradicts the explicit exclusion of UNSAFE options from the 'always deliver data' rule found in Section 14 and the intended fate-sharing semantics.

**Evidence Snippets:**

- **E1:**

  UNSAFE Options MUST be used only as part of UDP fragments, used either per-fragment or after reassembly. Receivers supporting UDP Options MUST silently drop the UDP user data of the reassembled datagram if any fragment or the entire datagram includes an UNSAFE Option whose Kind is not supported or if an UNSAFE Option appears outside the context of a fragment or reassembled fragments.

- **E2:**

  This enables the safe use of UNSAFE Options, which are required to result in discarding the entire UDP datagram if they are unknown to the receiver or otherwise fail (see Section 11.4).

- **E3:**

  All UDP Options other than UNSAFE ones MUST be ignored if not supported or upon failure (e.g., APC). All UDP Options that fail MUST result in the UDP data still being sent to the application layer by default to ensure equivalence with legacy devices.

**Evidence Summary:**

- (E1) Section 12 mandates dropping UDP user data when an UNSAFE option is unsupported or out of context.
- (E2) Section 11.4 describes the design intent for failure to result in discarding the datagram.
- (E3) Section 19 requires that, in all failure cases, UDP data is still delivered, contradicting the dropping requirement.

**Fix Direction:**

Harmonize the text in Sections 12, 11.4, and 19 so that all UNSAFE option failures—whether due to unsupported kinds, misplacement, or internal processing errors—consistently result in dropping the UDP user data (or delivering a zero-length datagram) in line with the intended fate-sharing semantics.


**Severity:** High
  *Basis:* The conflicting MUST statements can lead to divergent, non‑compliant implementation behavior with potential security and interoperability consequences.

**Confidence:** High

---
