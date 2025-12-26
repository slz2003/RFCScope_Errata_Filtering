# Errata Reports

Total reports: 3

---

## Report 1: 9811-3-1

**Label:** Announcement Retry Timing Underspecification in CMP-over-HTTP

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define key timing parameters for announcement retries, such as the starting point for measuring delays, the maximum number or duration of retries, and the handling of continuous 202 responses.

**Justification:**

- The text leaves implicit when the 'appropriate delay/time span' should be measured (e.g. from the original send, receipt of a 202, or the last retry).
- It does not specify any upper bounds on the number or duration of retries, nor how to handle a receiver that persistently returns 202.

**Evidence Snippets:**

- **E1:**

  The push/announcement mechanism defines a time-based retry strategy driven by HTTP 202 (Accepted) and 201 (Created), but leaves several temporal aspects implicit: exactly when retry timers are measured from, how many retries are reasonable, and how to behave if 202 persists or if responses are unreliable. However, the text does not specify: * From which event the "appropriate delay/time span" is counted (the original send vs. receipt of 202 vs. last retry). * Whether there is any upper bound on the number and duration of retries (which affects how long a sender keeps a given announcement “live”). * How to handle a receiver that continues to respond with 202 indefinitely (no clear transition from "accepted" to "failed"). * How the retry behavior interacts with the security guidance that, absent authenticated HTTP, "information regarding the announcement's processing state may not be trusted" and the overall PKI design "must not depend on the announcements being reliably received and processed by their destination" 5, item 4.

**Evidence Summary:**

- (E1) Describes the unspecified timing details for announcement retries, including the measurement start point, upper bounds, and handling of persistent 202 responses.

**Fix Direction:**

Clarify the announcement retry semantics by explicitly defining the start event for timing, setting any upper bounds on retries, and detailing the behavior when 202 responses continue to be received.


---

## Report 2: 9811-3-2

**Label:** Ambiguous Scope for Forwarding CMP Messages on HTTP Error Responses

**Bug Type:** Inconsistency

**Explanation:**

The normative requirement to 'MUST forward CMP messages when an HTTP error status code occurs' is unscoped, which conflicts with the mandate for announcement responses to have empty content, creating ambiguity over actor responsibilities.

**Justification:**

- Section 1.2 instructs that implementations MUST forward CMP messages on HTTP error statuses without clarifying which actor is responsible.
- In contrast, Section 3.5 requires that announcement acknowledgments contain empty content, thereby forbidding any CMP response in that context.

**Evidence Snippets:**

- **E2:**

  Implementations MUST forward CMP messages when an HTTP error status code occurs; see Section 3.1.

- **E3:**

  CMP announcement messages do not require any CMP response. However, the recipient MUST acknowledge receipt with an HTTP response having an appropriate status code and empty content.

**Evidence Summary:**

- (E2) Shows the forwarding requirement on HTTP error statuses.
- (E3) States that announcement responses must have empty content, conflicting with a universal forwarding mandate.

**Fix Direction:**

Either scope the forwarding requirement to exclude announcement acknowledgments or add an explicit note clarifying that the empty-content rule in Section 3.5 overrides the general forwarding rule.


