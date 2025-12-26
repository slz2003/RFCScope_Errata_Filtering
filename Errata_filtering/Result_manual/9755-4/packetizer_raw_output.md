# Errata Reports

Total reports: 2

---

## Report 1: 9755-4-1

**Label:** Ambiguous ENABLE Condition in APPEND UTF‑8 Acceptance

**Bug Type:** Ambiguity / Underspecification

**Explanation:**

Section 4 is ambiguous because it states that a server accepts UTF‑8 headers based solely on advertised support, while later mandating rejection if the client has not issued ENABLE UTF8=ACCEPT, leaving the precise acceptance conditions unclear.

**Justification:**

- The text states: If the server supports ‘UTF8=ACCEPT’, then the server accepts UTF‑8 headers in the ‘APPEND’ command message argument, which omits any mention of an ENABLE requirement. (E1)
- Immediately following, it requires that if the client has not issued the ENABLE command, the server MUST reject any APPEND containing 8‑bit header fields. (E2)
- Deontic analysis notes that while the negative case (rejection) is normatively specified, the corresponding positive case (acceptance after ENABLE) is only implied and not expressed in BCP 14 language.

**Evidence Snippets:**

- **E1:**

  If the server supports ‘UTF8=ACCEPT’, then the server accepts UTF‑8 headers in the ‘APPEND’ command message argument.

- **E2:**

  If an IMAP server supports ‘UTF8=ACCEPT’ and the IMAP client has not issued the ‘ENABLE UTF8=ACCEPT’ command, the server MUST reject, with a ‘NO’ response, an ‘APPEND’ command that includes any 8‑bit character in message header fields.

**Evidence Summary:**

- (E1) The acceptance statement is unconditional, lacking reference to client ENABLE.
- (E2) The rejection rule explicitly requires the ENABLE state, creating an ambiguity between acceptance and rejection conditions.

**Fix Direction:**

Revise the first sentence of Section 4 to explicitly condition acceptance on the client having issued ENABLE UTF8=ACCEPT, for example by specifying: 'If the server supports UTF8=ACCEPT and the client has issued ENABLE UTF8=ACCEPT, then the server MUST accept UTF‑8 headers in the APPEND command message argument.'


**Severity:** Low
  *Basis:* The ambiguity could lead to divergent implementations regarding when UTF‑8 headers must be accepted, but it does not create an unimplementable or self‑contradictory behavior.

**Confidence:** High

---

## Report 2: 9755-4-2

**Label:** Ambiguous Scope of 'Message Header Fields' in APPEND Literal

**Bug Type:** Underspecification

**Explanation:**

The rejection rule in Section 4 for APPEND commands is ambiguous because it does not clearly define what constitutes 'message header fields' when the APPEND literal does not conform to a standard RFC 5322 layout.

**Justification:**

- Section 4 mandates rejection of an APPEND command if any 8‑bit character is found in message header fields, but does not clarify which part of the literal qualifies as headers. (E1)
- RFC 3501 specifies that the APPEND argument SHOULD be in RFC‑2822 format and permits deviations, while RFC 5322 defines headers in a structured manner, leading to potential uncertainty. (E2, E3)
- This lack of clarity may allow different implementations to interpret the boundaries of header fields inconsistently.

**Evidence Snippets:**

- **E1:**

  …the server MUST reject, with a ‘NO’ response, an ‘APPEND’ command that includes any 8‑bit character in message header fields.

- **E2:**

  RFC 3501 §6.3.11 (APPEND): “This argument SHOULD be in the format of an [RFC-2822] message. 8‑bit characters are permitted in the message. A server implementation that is unable to preserve 8‑bit data properly MUST be able to reversibly convert 8‑bit APPEND data to 7‑bit…” and note: “There MAY be exceptions, e.g., draft messages, in which required [RFC-2822] header lines are omitted in the message literal argument to APPEND.”

- **E3:**

  RFC 5322 §3.5/§3.6: defines a “message” as “(fields / obs-fields) [CRLF body]” and describes headers as a block of “header fields” followed by the optional body.

**Evidence Summary:**

- (E1) The rule mandates rejection based on 8‑bit characters in 'message header fields'.
- (E2) RFC 3501 permits non‑conforming message formats, which may lack a clear header section.
- (E3) The standard RFC 5322 definition does not necessarily apply to all APPEND literals, creating ambiguity.

**Fix Direction:**

Provide a clear definition of 'message header fields' for APPEND literals, and specify how servers should handle cases where the literal does not strictly conform to RFC 5322 formatting.


**Severity:** Low
  *Basis:* The ambiguity could lead to interoperability issues as different servers might interpret the boundaries of header fields differently, but it is not a critical contradiction.

**Confidence:** High

---
