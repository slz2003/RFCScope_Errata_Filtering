# Errata Reports

Total reports: 2

---

## Report 1: 9869-5-1

**Label:** Ambiguous PTB Token Validation Requirements (Section 5.2 vs Section 8)

**Bug Type:** Both

**Explanation:**

The RFC ambiguously defines 'this validation' in the PTB processing section, making it unclear whether the token check is optional (a SHOULD) or mandatory (a MUST), which conflicts with the later security requirements stated in Section 8.

**Justification:**

- Section 5.2 describes two checks—a mandatory protocol/port validation and a token check labeled as SHOULD—then uses the phrase 'this validation' ambiguously.
- Section 8 unambiguously requires that the token be used in combination with port validation, effectively upgrading the token check to a MUST requirement.

**Evidence Snippets:**

- **E1:**

  Before processing an ICMP PTB message, the DPLPMTUD method needs to perform two checks…
* The receiver MUST validate the protocol information in the quoted packet carried in an ICMP PTB message payload…
* Specifically, a UDP Options receiver SHOULD confirm that the token contained in the UDP REQ Option of the quoted packet has a value that corresponds to a probe packet that was recently sent by the current endpoint.
… An implementation unable to support this validation MUST ignore received ICMP PTB messages.

- **E2:**

  A UDP Options sender that utilizes ICMP PTB messages received to a probe packet MUST use the quoted packet to validate the UDP port information in combination with the token contained in the UDP Option before processing the packet using the DPLPMTUD method.

- **E3:**

  Support for receiving ICMP PTB messages is OPTIONAL for DPLPMTUD. A UDP Options sender can therefore ignore received ICMP PTB messages.

**Evidence Summary:**

- (E1) Lists the two validation checks and shows the ambiguous use of ‘this validation’.
- (E2) Provides the clear MUST requirement from Section 8 that ties token usage to PTB processing.
- (E3) Emphasizes that PTB reception is optional, highlighting the inconsistency in how token validation is treated.

**Fix Direction:**

Clarify and align the validation requirements by explicitly stating whether the token check is mandatory or optional, and consistently refer to the same scope of 'this validation' in both Section 5.2 and Section 8.


**Severity:** Medium
  *Basis:* The ambiguity may lead to divergent interpretations and implementations, potentially reducing security robustness against spoofed PTB messages.

**Confidence:** High

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
