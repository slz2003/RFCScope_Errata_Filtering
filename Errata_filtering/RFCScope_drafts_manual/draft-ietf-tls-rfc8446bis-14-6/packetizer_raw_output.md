# Errata Reports

Total reports: 5

---

## Report 1: draft-ietf-tls-rfc8446bis-14-6-1

**Label:** Ambiguity in 'user_canceled' vs 'close_notify' Ordering for Handshake Termination

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state whether receiving a 'user_canceled' alert should immediately terminate the handshake or require waiting for the subsequent 'close_notify', which can lead to different shutdown sequences.

**Justification:**

- The text does not explicitly mandate that 'user_canceled' alone must terminate the TLS connection, leaving implementations free to decide whether to wait for a 'close_notify' or to close immediately.
- This ambiguity may result in inconsistent logging, alert ordering, and observable shutdown behavior between different implementations.

**Evidence Snippets:**

- **E1:**

  §1.2: “Clarify behavior around ‘user_canceled’, requiring that ‘close_notify’ be sent and that ‘user_canceled’ should be ignored.” Section 1.2.
§6 (intro): “The ‘close_notify’ alert is used to indicate orderly closure of one direction of the connection.”
§6.1 (close_notify): “Any data received after a closure alert has been received MUST be ignored. This alert MUST be sent with AlertLevel=warning.”
§6.1 (user_canceled): “This alert notifies the recipient that the sender is canceling the handshake for some reason unrelated to a protocol failure. … This alert MUST be followed by a ‘close_notify’. This alert generally has AlertLevel=warning. Receiving implementations SHOULD continue to read data from the peer until a ‘close_notify’ is received, though they MAY log or otherwise record them.”
§6.1 (general closure rule): “Each party MUST send a ‘close_notify’ alert before closing its write side of the connection, unless it has already sent some error alert.”
§5.1: “Alert messages … MUST NOT be fragmented across records, and multiple alert messages MUST NOT be coalesced into a single TLSPlaintext record. In other words, a record with an Alert type MUST contain exactly one message.”

**Evidence Summary:**

- (E1) References from Sections 1.2, 6, 6.1, and 5.1 indicate that the ordering and termination effect of 'user_canceled' versus 'close_notify' is not crisply defined.

**Fix Direction:**

Specify in a dedicated statement that receiving a 'user_canceled' alert does not by itself terminate the handshake and that the state change to a closed connection is to be triggered only upon receipt or sending of a 'close_notify'.

**Severity:** Low
  *Basis:* Although differences in shutdown ordering may affect logging and error reporting, all interpretations eventually lead to termination without impacting security.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1

---

## Report 2: draft-ietf-tls-rfc8446bis-14-6-2

**Label:** Ambiguous Scope of 'Closure Alert' Rule for close_notify and user_canceled

**Bug Type:** Inconsistency

**Explanation:**

The specification uses the generic term 'closure alert' in a way that could inadvertently force implementations to ignore data after either a 'close_notify' or a 'user_canceled' alert, which contradicts the intended exception for 'user_canceled'.

**Justification:**

- Section 6.1 mandates that any closure alert should cause subsequent data to be ignored, yet it also specifies that 'user_canceled' must be followed by a 'close_notify' and does not itself trigger immediate termination of data processing.
- The change summary in Section 1.2 reinforces that 'user_canceled' should be effectively ignored in terms of connection closure, highlighting a conflict in using an over-broad class term.

**Evidence Snippets:**

- **E2:**

  Alerts are classified: “Alerts are divided into two classes: closure alerts and error alerts.” Section 6.
`close_notify` description: “Any data received after a closure alert has been received MUST be ignored. This alert MUST be sent with AlertLevel=warning.” Section 6.1.
`user_canceled` description: “This alert … MUST be followed by a ‘close_notify’. This alert generally has AlertLevel=warning. Receiving implementations SHOULD continue to read data from the peer until a ‘close_notify’ is received, though they MAY log or otherwise record them.” Section 6.1.
Change summary: “Clarify behavior around ‘user_canceled’, requiring that ‘close_notify’ be sent and that ‘user_canceled’ should be ignored.” Section 1.2.

**Evidence Summary:**

- (E2) The evidence shows that while 'close_notify' requires immediate data ignoring, 'user_canceled' is meant to be an informational precursor, leading to a conflicting application of the term 'closure alert'.

**Fix Direction:**

Rescope the data-ignoring rule so that it applies explicitly to 'close_notify' only, thereby preserving the requirement for 'user_canceled' to be followed by a 'close_notify' without preemptively halting data processing.

**Severity:** Low
  *Basis:* The issue is primarily an editorial inconsistency that could result in varied implementations, though it does not compromise overall security or connectivity.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

---

## Report 3: draft-ietf-tls-rfc8446bis-14-6-3

**Label:** Conflict Between general_error Usage and Mandatory Specific Error Alerts

**Bug Type:** Inconsistency

**Explanation:**

The specification introduces a 'general_error' alert for cases lacking a more specific error, yet elsewhere it mandates that specific alerts (such as decode_error or illegal_parameter) must be used, leading to conflicting requirements.

**Justification:**

- Section 6.2 states that in fatal error conditions an implementation MUST send a specific alert as prescribed, while the 'general_error' description permits concealing the specific error code.
- This conflict creates ambiguity regarding whether 'general_error' can substitute for a mandated alert in contexts where a specific error is required.

**Evidence Snippets:**

- **E3:**

  “Whenever an implementation encounters a fatal error condition, it SHOULD send an appropriate fatal alert and MUST close the connection … The phrases ‘terminate the connection with an X alert’ and ‘abort the handshake with an X alert’ mean that the implementation MUST send alert X if it sends any alert.” (Section 6.2)
“general_error: Sent to indicate an error condition in cases when either no more specific error is available or the sender wishes to conceal the specific error code. Implementations SHOULD use more specific errors when available.” (Section 6.2)

**Evidence Summary:**

- (E3) The evidence contrasts the mandate to send a specific alert in fatal error conditions with the permissive use of 'general_error' to obscure specific error details.

**Fix Direction:**

Clarify that the 'general_error' alert may only be used in scenarios where no specific, mandated error alert is defined, preventing its use as a substitute in contexts that require a particular alert code.

**Severity:** Medium
  *Basis:* This inconsistency may lead to different implementations signaling errors differently, potentially affecting interoperability and error handling.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-1

---

## Report 4: draft-ietf-tls-rfc8446bis-14-6-4

**Label:** Ambiguity in Handling Unknown and RESERVED Alert Codes Across TLS Versions

**Bug Type:** Editorial

**Explanation:**

The guidance on treating unknown alert types as fatal in TLS 1.3, together with references to RESERVED alert codes from earlier TLS versions, may confuse implementers regarding how to handle these alerts in mixed-version deployments.

**Justification:**

- Section 6 instructs that unknown alert types MUST be treated as error alerts in TLS 1.3, yet it does not explicitly delineate this behavior from TLS 1.2, where RFC 5246 applies.
- The inclusion of RESERVED alert codes in Appendix B.2 without clear scoping further adds to potential misinterpretation in multi-version contexts.

**Evidence Snippets:**

- **E4:**

  “All the alerts listed in Section 6.2 MUST be sent with AlertLevel=fatal and MUST be treated as error alerts when received regardless of the AlertLevel in the message. Unknown Alert types MUST be treated as error alerts.” (Section 6)
“Values listed as ‘_RESERVED’ were used in previous versions of TLS and are listed here for completeness. TLS 1.3 implementations MUST NOT send them but might receive them from older TLS implementations.” (Appendix B.2)

**Evidence Summary:**

- (E4) The evidence highlights that unknown alerts are mandated to be fatal in TLS 1.3 while referencing RESERVED codes from earlier versions, without explicit clarification on the differences in handling.

**Fix Direction:**

Add explicit clarification distinguishing the handling of unknown and RESERVED alert codes in TLS 1.3 from the rules that apply under RFC 5246 for TLS 1.2.

**Severity:** Low
  *Basis:* The potential confusion is mainly editorial and is unlikely to result in critical interoperability failures.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- Deontic Expert: Issue-2
- Boundary Expert: Finding-2

---

## Report 5: draft-ietf-tls-rfc8446bis-14-6-5

**Label:** Clarification Needed for 'user_canceled' Alert Usage During Handshake vs Post-Handshake

**Bug Type:** None

**Explanation:**

Although the specification defines 'user_canceled' as a closure alert that must be followed by a 'close_notify', its recommendations for post-handshake scenarios are less explicit, potentially leading to varied interpretations.

**Justification:**

- Section 6.1 advises that if 'user_canceled' is used after the handshake is complete, simply sending a 'close_notify' is more appropriate.
- The guidance to continue reading data until a 'close_notify' is received may be interpreted differently during active versus post-handshake phases.

**Evidence Snippets:**

- **E5:**

  “user_canceled: This alert notifies the recipient that the sender is canceling the handshake for some reason unrelated to a protocol failure. If a user cancels an operation after the handshake is complete, just closing the connection by sending a ‘close_notify’ is more appropriate. This alert MUST be followed by a ‘close_notify’. This alert generally has AlertLevel=warning. Receiving implementations SHOULD continue to read data from the peer until a ‘close_notify’ is received, though they MAY log or otherwise record them.” (Section 6.1)

**Evidence Summary:**

- (E5) The evidence indicates that while 'user_canceled' must be followed by a 'close_notify', the recommended behavior differs depending on whether the alert occurs during or after the handshake.

**Severity:** Low
  *Basis:* The ambiguity is primarily about editorial clarity and is unlikely to disrupt the overall connection termination process.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary Expert: Finding-1

---
