# Errata Reports

Total reports: 1

---


## Report 2: 9869-5-2

**Label:** Inconsistent Actor Role Assignment for PTB Validation

**Bug Type:** Inconsistency

**Explanation:**

The RFC inconsistently assigns the responsibility for PTB message validation between the UDP Options sender and receiver, which can mislead implementers as to which endpoint should perform the token-based validation.

**Justification:**

- Section 5.2 suggests that the token check is performed by a UDP Options receiver even though ICMP PTB messages are sent to the UDP Options sender.
- Other sections, particularly in the security considerations, correctly assign the validation responsibility to the UDP Options sender.

**Evidence Snippets:**

- **E4:**

  Section 5.2 assigns the token-based PTB validation step to a “UDP Options receiver”, even though ICMP PTB messages are sent back to the original UDP sender and must therefore be processed by the UDP Options sender. Elsewhere (notably in the security considerations) the RFC correctly assigns PTB validation responsibilities to the UDP Options sender, so this is a clear role/direction inversion.

**Evidence Summary:**

- (E4) Highlights the actor role inconsistency between sections by showing that the token-based PTB validation is incorrectly attributed to the UDP Options receiver in one part of the RFC.

**Fix Direction:**

Clearly define and consistently assign the PTB validation responsibility to the UDP Options sender across all sections of the RFC to eliminate any role/direction inversion.


**Severity:** Medium
  *Basis:* Misassignment of roles can confuse implementers and may lead to insecure behavior if the wrong endpoint processes critical PTB validations.

**Confidence:** High

---