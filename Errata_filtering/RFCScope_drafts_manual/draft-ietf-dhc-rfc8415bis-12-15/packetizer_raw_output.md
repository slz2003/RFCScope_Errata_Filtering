# Errata Reports

Total reports: 3

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

## Report 2: draft-ietf-dhc-rfc8415bis-12-15-2

**Label:** Underspecified Interaction of MRD with RT, T2, and Lease Expiry

**Bug Type:** Underspecification

**Explanation:**

The text does not clearly specify how the retransmission timer (RT) should interact with the hard deadline (MRD) and related timers like T2 or lease expiry, leaving open how to handle a retransmission that would occur after MRD.

**Justification:**

- Section 15 defines MRD as a hard upper bound on retransmission time, but does not state if a computed RT that extends beyond MRD should be suppressed or adjusted.
- The treatment of replies that arrive slightly after MRD (or T2/lease expiry) is not explicitly described, leading to potential differences in client behavior.

**Evidence Snippets:**

- **E1:**

  MRD specifies an upper bound on the length of time a client may retransmit a message. Unless MRD is zero, the message exchange fails once MRD seconds have elapsed since the client first transmitted the message. (Section 15)

- **E2:**

  Renew: “MRD = Remaining time until earliest T2 … The message exchange is terminated when the earliest time T2 is reached, at which point the client begins the Rebind message exchange.” (Section 18.2.4)

**Evidence Summary:**

- (E1) Section 15 provides an upper bound for retransmissions using MRD without detailing RT and MRD interaction.
- (E2) Section 18.2.4 illustrates how MRD is instantiated for Renew, yet leaves unresolved how to align RT intervals when nearing T2.

**Fix Direction:**

Explicitly define how RT should be adjusted or suppressed when the next retransmission would exceed the MRD, and specify how late-arriving replies are to be handled.

**Severity:** Low
  *Basis:* While the ambiguity may cause slight differences in timing or retransmission behavior, it does not undermine essential protocol interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-15-3

**Label:** Ambiguity in the Scope of 'Message Exchange' and Elapsed Time Measurement

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly delineate whether each client‐initiated message constitutes a separate exchange with its own elapsed time counter or if the Elapsed Time spans a multi‐message lifecycle.

**Justification:**

- Section 15 and Section 21.9 refer to the 'current DHCP message exchange' but do not clarify whether this resets with each new client message (e.g., Request or Renew).
- The per‐message requirements in Section 18 specify inclusion of an Elapsed Time option without explicitly defining the exchange boundaries, leading to possible inconsistencies in timer resets.

**Evidence Snippets:**

- **E1:**

  The client begins the message exchange by transmitting a message to the server. The message exchange terminates when either (1) the client successfully receives the appropriate response or responses from a server or servers or (2) the message exchange is considered to have failed according to the retransmission mechanism described below. (Section 15)

- **E2:**

  The elapsed time is measured from the time at which the client sent the first message in the message exchange, and the elapsed-time field is set to 0 in the first message in the message exchange. (Section 21.9)

**Evidence Summary:**

- (E1) Section 15 defines the initiation and termination of a 'message exchange' without specifying its granularity.
- (E2) Section 21.9 sets the context for elapsed time measurement, but does not clarify if it resets for each distinct client-initiated message exchange.

**Fix Direction:**

Clarify that each client-initiated message (e.g., Solicit, Request, Renew) constitutes a separate message exchange with an independent elapsed time counter that resets to 0 for its first transmission.

**Severity:** Low
  *Basis:* Although the ambiguity might lead to differences in elapsed time reporting, implementers can generally infer the intended per-message behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T3

---
