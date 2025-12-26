# Errata Reports

Total reports: 1

---

## Report 1: 9869-8-1

**Label:** Conflicting Normative Requirements for ICMP PTB Token Validation (Section 5.2 vs Section 8)

**Bug Type:** Inconsistency

**Explanation:**

RFC 9869 provides conflicting normative requirements for validating ICMP PTB messages: Section 5.2 recommends using the REQ token (SHOULD) while Section 8 mandates its use (MUST) when processing probe-related PTB messages.

**Justification:**

- Section 5.2 specifies that the receiver SHOULD confirm the REQ token against a recently sent probe and that an implementation unable to support 'this validation' MUST ignore PTB messages.
- Section 8, however, requires that a UDP Options sender MUST use the quoted packet to validate UDP port information in combination with the token before processing the message, effectively upgrading the recommendation into a mandatory check.

**Evidence Snippets:**

- **E1:**

  Section 5.2 bullets:  - The receiver MUST validate the protocol information in the quoted packet carried in an ICMP PTB message payload to validate the message originated from the sending node (see Section 4.6.1 of [RFC8899]).  - The receiver SHOULD utilize information that is not simple for an off-path attacker to determine (see Section 4.6.1 of [RFC8899]). Specifically, a UDP Options receiver SHOULD confirm that the token contained in the UDP REQ Option of the quoted packet has a value that corresponds to a probe packet that was recently sent by the current endpoint.  - An implementation unable to support this validation MUST ignore received ICMP PTB messages.

- **E2:**

  Section 8: A UDP Options sender that utilizes ICMP PTB messages received to a probe packet MUST use the quoted packet to validate the UDP port information in combination with the token contained in the UDP Option before processing the packet using the DPLPMTUD method.

**Evidence Summary:**

- (E1) Section 5.2 prescribes a MUST for validating protocol information and a SHOULD for token validation, with a fallback to ignore PTB if the validation cannot be supported.
- (E2) Section 8 mandates that both the UDP port and token from the quoted packet be used for validation before processing the message.

**Fix Direction:**

Clarify the normative language in Sections 5.2 and 8 by either explicitly scoping the requirement on the REQ token (e.g., making it optional or mandatory uniformly) to eliminate the ambiguity.


**Severity:** Medium
  *Basis:* The inconsistency can lead to divergent interpretations in implementations and varied security behavior, potentially weakening off-path PTB spoofing protection under some interpretations.

**Confidence:** Medium

---
