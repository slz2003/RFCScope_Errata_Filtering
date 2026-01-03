# Errata Reports

Total reports: 3

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-21-1

**Label:** Conflicting ORO Semantics between Sections 21.7 and 18.3

**Bug Type:** Inconsistency

**Explanation:**

The specification contradicts itself by requiring that top‐level option codes MUST be present in the Option Request option (Section 21.7) while simultaneously permitting servers to send additional unsolicited options (Section 18.3).

**Justification:**

- Section 21.7 mandates that 'Other top-level option codes MUST appear in the Option Request option or they will not be sent by the server,' implying a hard prohibition on unsolicited top-level options.
- Section 18.3 explicitly states that 'The server MAY return additional options to the client if it has been configured to do so,' which permits sending options that were not requested.

**Evidence Snippets:**

- **E1:**

  Section 21.7 (Option Request option) after listing codes that MUST NOT appear in ORO:  “Other top-level option codes MUST appear in the Option Request option or they will not be sent by the server. Only top-level option codes MAY appear in the Option Request option.”  It then points to [IANA-OPTION-DETAILS] for which codes are “required, permitted or forbidden.”

- **E2:**

  Section 18.3 (Server Behavior):  “If the client included an Option Request option (see Section 21.7) in its message, the server includes options in the response message containing configuration parameters for all of the options identified in the Option Request option that the server has been configured to return to the client. The server MAY return additional options to the client if it has been configured to do so.”

**Evidence Summary:**

- (E1) Section 21.7 requires that other top-level option codes MUST appear in the Option Request option or they will not be sent.
- (E2) Section 18.3 permits the server to return additional options beyond those requested.

**Fix Direction:**

Clarify the text by either narrowing the scope of the 'MUST' in Section 21.7 to apply only to options explicitly requiring presence in the ORO (per the IANA 'Client ORO' registry) or by restricting the 'MAY return additional options' in Section 18.3 to exclude unsolicited top-level options.

**Severity:** Medium
  *Basis:* The contradictory instructions could lead to divergent server behaviors and interoperability issues, especially for options such as DNS and NTP that are commonly sent even when not requested.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-21-2

**Label:** Ambiguity in IA_TA Handling within IA Option Validation

**Bug Type:** Underspecification

**Explanation:**

The document instructs that the obsoleted IA_TA option should be ignored (Section 21.5), while Section 16’s message validation rules require discarding any message containing an IA option, creating conflicting guidance.

**Justification:**

- Section 21.5 advises that if a server receives an IA_TA option, it SHOULD ignore it and continue processing.
- Section 16.12 mandates that a server MUST discard any Information-request message that includes an IA option, leaving ambiguity over the treatment of IA_TA.

**Evidence Snippets:**

- **E1:**

  Section 21.5: “The Identity Association for Temporary Addresses (IA_TA) option is obsoleted by this document. … The client SHOULD NOT send this option. The server SHOULD NOT send this option. When the server receives IA_TA option, the option SHOULD be ignored and the message processing should continue as usual.”

- **E2:**

  Section 16.12: “Servers MUST discard any received Information-request message that meets any of the following conditions: … * the message includes an IA option.”

**Evidence Summary:**

- (E1) Section 21.5 instructs servers to ignore IA_TA and proceed normally.
- (E2) Section 16.12 requires discarding messages that contain any IA option.

**Fix Direction:**

Specify explicitly in Section 16 (or in a clarifying note) that the message validation rules do not apply to the obsoleted IA_TA option, or adjust Section 21.5 to align with the validation rules.

**Severity:** Low
  *Basis:* Since IA_TA is obsoleted and not widely used, the ambiguity mainly affects legacy and edge-case implementations rather than core functionality.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-21-3

**Label:** Incorrect Cross-Reference for IA Address Encapsulation

**Bug Type:** Inconsistency

**Explanation:**

The encapsulated option definition in Section 4.2 incorrectly references Section 21.5 for the IA Address option, even though IA_NA is defined in Section 21.4 and IA Address is in Section 21.6, leading to reader confusion.

**Justification:**

- Section 4.2 states: “For example, the IA Address option is contained in IA_NA options (see Section 21.5),” which is misleading.
- In the current draft, Section 21.5 covers the obsoleted IA_TA option, while IA_NA and IA Address are properly defined in Sections 21.4 and 21.6, respectively.

**Evidence Snippets:**

- **E1:**

  Snippet1 (4.2): “encapsulated option … For example, the IA Address option is contained in IA_NA options (see Section 21.5).”

- **E2:**

  Snippet2 (Section 21 layout): Section 21.4 defines the IA_NA option; Section 21.5 is “Identity Association for Temporary Addresses Option” (IA_TA, obsoleted); Section 21.6 defines the “IA Address Option”.

**Evidence Summary:**

- (E1) The definition in Section 4.2 incorrectly points to Section 21.5 for the IA Address option.
- (E2) The actual definitions are in Sections 21.4 (IA_NA) and 21.6 (IA Address), while Section 21.5 pertains to the obsoleted IA_TA option.

**Fix Direction:**

Update the cross-reference in Section 4.2 to correctly refer to Section 21.4 and/or Section 21.6 instead of Section 21.5.

**Severity:** Low
  *Basis:* This error is primarily editorial and affects clarity rather than protocol operation.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-2
- CrossRFC Expert: Issue-2

---
