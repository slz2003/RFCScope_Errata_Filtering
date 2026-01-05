# Errata Reports

Total reports: 1

---


## Report 1: draft-ietf-dhc-rfc8415bis-12-15-1

**Label:** Ambiguous Interpretation of MRC (Maximum Retransmission Count): Retransmissions vs Total Transmissions

**Bug Type:** Ambiguous Semantics

**Explanation:**

The specification’s language regarding MRC is ambiguous, mixing a definition based on retransmissions with a failure condition based on total transmissions, which leads to two plausible interpretations.

**Justification:**

- Section 15 states that MRC is the upper bound on the number of times a client may retransmit a message but then says the exchange fails once the message has been transmitted MRC times, creating a conflict.
- Table 1 describes values such as REQ_MAX_RC as 'Max Request retry attempts', which implies counting only retransmissions, not the initial send.

**Evidence Snippets:**

- **E1:**

  MRC specifies an upper bound on the number of times a client may retransmit a message. Unless MRC is zero, the message exchange fails once the client has transmitted the message MRC times. (Section 15)

- **E2:**

  Table 1 entries: “REQ_MAX_RC – Max Request retry attempts”, “REL_MAX_RC – Max Release retry attempts”, “DEC_MAX_RC – Max Decline retry attempts” (Section 7.6)

**Evidence Summary:**

- (E1) Section 15 defines MRC in conflicting terms: first as counting retransmissions only and then as counting total transmissions.
- (E2) The retransmission parameters in Table 1 suggest that the intended meaning is retry attempts (i.e., retransmissions only).

**Fix Direction:**

Clarify the description of MRC in Section 15 by explicitly stating that it counts only retransmissions and that the total number of transmissions is 1 + MRC.

**Severity:** Low
  *Basis:* The ambiguity produces only an off‑by‑one discrepancy which affects timing but does not break interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T1
- Quantitative: Issue-1
- Deontic: Issue-1
- Terminology: Issue-1
- Boundary: Finding-1

---