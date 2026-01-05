# Errata Reports

Total reports: 3

---


## Report 2: 9859-4-2

**Label:** Ambiguous DNSSEC Validation Criteria for DSYNC Endpoint Discovery

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state which DNSSEC validation outcomes (Secure, Insecure, Bogus, or Indeterminate) qualify as a 'positive DSYNC answer', leaving room for divergent implementations.

**Justification:**

- RFC 9859 Section 4.1 instructs a sender to validate the DSYNC lookup if DNSSEC is enabled and to return the result if it constitutes a positive answer, but it does not define which DNSSEC validation states are acceptable.
- Multiple experts note that the lack of guidance on handling outcomes such as Bogus or Indeterminate leaves room for different security policies and behaviors.

**Evidence Snippets:**

- **E1:**

  RFC 9859 Section 4.1 step 2: “Perform a lookup of type DSYNC for the lookup name, and validate the response if DNSSEC is enabled.  If this results in a positive DSYNC answer, return it.”

- **E2:**

  It is RECOMMENDED that zones containing DSYNC records be secured with DNSSEC. … If so, the sender may choose to ignore unsigned DSYNC records.

**Evidence Summary:**

- (E1) outlines the DSYNC lookup and validation step without detailing acceptable validation outcomes.
- (E2) indicates the recommendation for DNSSEC without providing normative guidance on handling unsigned or failed validations.

**Fix Direction:**

Add explicit normative guidelines that specify which DNSSEC validation outcomes (e.g., Secure only, or Secure plus optionally Insecure) count as a positive DSYNC answer and how to handle Bogus or Indeterminate responses.

**Severity:** Medium
  *Basis:* Ambiguity in DNSSEC handling can lead to inconsistent endpoint selection, resulting either in missed notifications or potential exposure to spoofed targets.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2
- Causal: Issue 2
- Deontic: Issue-1
- Boundary: Finding-1

---


## Report 3: 9859-4-3

**Label:** Ambiguous Enforcement of Agent-Domain NS-Subordination for EDNS0 Report-Channel

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly specify which party (sender or receiver) is responsible for enforcing that the agent domain in the EDNS0 Report-Channel option is subordinate or equal to one of the child’s NS hostnames, and what action should be taken if the requirement is not met.

**Justification:**

- RFC 9859 Section 4.2.1 mandates that the agent domain in the Report-Channel option MUST be subordinate or equal to one of the NS hostnames, but Section 4.3 only states that if this condition is met, error reporting SHOULD occur.
- The ambiguity leaves uncertainty about whether the sender bears the responsibility to choose an appropriate agent domain or if the receiver must enforce the check and possibly reject or ignore the option if the condition is not fulfilled.

**Evidence Snippets:**

- **E1:**

  RFC 9859 Section 4.2.1: “When including this EDNS0 option, the second label (QTYPE) of the report query name is equal to the qtype received in the NOTIFY message.  Its agent domain MUST be subordinate or equal to one of the NS hostnames, as listed in the child's delegation in the parent zone.  This is to prevent malicious senders from causing the NOTIFY recipient to send unsolicited report queries to unrelated third parties.”

- **E2:**

  RFC 9859 Section 4.3: “If the NOTIFY message contains an EDNS0 Report-Channel option … with an agent domain subordinate or equal to one of the NS hostnames listed in the delegation, the processing party SHOULD report any errors occurring during CDS/CDNSKEY/CSYNC processing by sending a report query…”

**Evidence Summary:**

- (E1) indicates the requirement for the agent domain but does not assign responsibility for enforcement.
- (E2) shows that receivers are only conditionally advised to send a report query, leaving the enforcement role ambiguous.

**Fix Direction:**

Clarify the text to specify whether the sender must select an agent domain that meets the requirement and/or if the receiver must ignore or reject the option when the agent domain does not satisfy the subordinate condition.

**Severity:** Medium
  *Basis:* Lack of clear enforcement could lead to unintended error-report traffic being sent to unrelated parties, undermining the intended security benefits.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-3

---


## Report 4: 9859-4-4

**Label:** Undefined Error Handling and Fallback Behavior in DSYNC Endpoint Discovery

**Bug Type:** Underspecification

**Explanation:**

The DSYNC endpoint discovery procedure does not define how to handle lookup errors (e.g., SERVFAIL, REFUSED, or timeouts) or how to proceed when a DSYNC RRset is returned that contains only unusable (no-op) records, leading to potential inconsistencies in fallback behavior.

**Justification:**

- Boundary Expert Finding-2 points out that the current text only distinguishes between a 'positive DSYNC answer' and a 'negative response', leaving ambiguous cases such as SERVFAIL or other non-standard error responses unaddressed.
- Boundary Expert Finding-3 highlights that if a DSYNC RRset exists but all records are no-op (e.g., Scheme=0 or Port=0), the procedure does not specify whether this should be treated as a positive answer or trigger fallback.

**Evidence Snippets:**

- **E1:**

  Boundary Expert Finding-2: “Perform a lookup of type DSYNC … If this results in a positive DSYNC answer, return it.” (Section 4.1, step 2) and “If the query resulted in a negative response: … Otherwise, return null (no notification target available).” (Section 4.1, step 3)

- **E2:**

  Boundary Expert Finding-3: “No behavior defined when DSYNC RRset exists but all RRs are unusable. Records with value 0 (null scheme) are ignored by consumers.” (Sections 2.1 and 4.1)

**Evidence Summary:**

- (E1) shows that the discovery algorithm only defines paths for clearly positive or negative responses, omitting error conditions such as SERVFAIL or non-standard responses.
- (E2) indicates the absence of guidance for handling DSYNC RRsets that contain only no-op records.

**Fix Direction:**

Define normative fallback procedures for DSYNC lookup errors and specify that, if a DSYNC RRset is returned but contains only unusable records (e.g., all with Scheme=0 or Port=0), it should be treated as equivalent to a negative response to trigger fallback.

**Severity:** Medium
  *Basis:* Ambiguous error handling may result in different implementations choosing incompatible fallback behaviors, potentially affecting notification reliability and security.

**Confidence:** High

**Experts mentioning this issue:**

- Boundary: Finding-2
- Boundary: Finding-3

---