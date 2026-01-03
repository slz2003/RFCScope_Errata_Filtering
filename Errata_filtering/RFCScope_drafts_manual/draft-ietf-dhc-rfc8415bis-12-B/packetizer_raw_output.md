# Errata Reports

Total reports: 3

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-B-1

**Label:** INF_MAX_RT Omission in Advertise Row of Appendix B

**Bug Type:** Inconsistency

**Explanation:**

The SOL_MAX_RT/INF_MAX_RT table in Appendix B marks only SOL_MAX_RT with a star for the Advertise row, despite normative text in Sections 18.2.9 and 21.25 allowing INF_MAX_RT in Advertise messages and requiring clients to process it.

**Justification:**

- Normative text states that a server MAY include INF_MAX_RT in any response and that clients MUST process it in an Advertise message (as seen in evidence snippets), yet the table for Advertise shows only a '*' for SOL_MAX_RT.
- This misalignment between the summary table and the normative sections could lead implementers to erroneously believe that INF_MAX_RT is not permitted in Advertise messages.

**Evidence Snippets:**

- **E1:**

  The client MUST process any SOL_MAX_RT option (see Section 21.24) and INF_MAX_RT option (see Section 21.25) present in an Advertise message, even if the message contains a Status Code option...

- **E2:**

  Appendix B table: 
           SOL_MAX_RT  INF_MAX_RT
             ...
             Advert.    *
             ...
             Reply      *           *

- **E3:**

  The DHCP server MAY include the INF_MAX_RT option in any response it sends to a client that has included the INF_MAX_RT option code in an Option Request option.

**Evidence Summary:**

- (E1) Normative processing requires both SOL_MAX_RT and INF_MAX_RT to be processed in Advertise.
- (E2) The table displays a '*' for SOL_MAX_RT in Advertise but leaves INF_MAX_RT blank.
- (E3) Normative text explicitly permits INF_MAX_RT in responses such as Advertise.

**Fix Direction:**

In Appendix B’s SOL_MAX_RT/INF_MAX_RT table, add a '*' in the INF_MAX_RT column for the Advertise row to align with the normative text.

**Severity:** Medium
  *Basis:* Reliance on the table by implementers may lead to misinterpretation of the allowed options, impacting behavior related to retransmission timing.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: Issue-1
- Scope: Issue-2
- Deontic: Issue-1
- Structural: Issue-1
- Causal: INF_MAX_RT in Advertise

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-B-2

**Label:** Ambiguous Information-request Row in Appendix B Regarding Client and Server Identifier Options

**Bug Type:** Inconsistency

**Explanation:**

The Information-request row in Appendix B is ambiguously formatted with '* (see note)' which may lead readers to misinterpret which identifier (Client or Server) is allowed, potentially omitting the Client Identifier and restricting the Server Identifier incorrectly.

**Justification:**

- The table row appears as 'Inform.   * (see note)              *           *' followed by a note that ties the allowed use of the Server Identifier exclusively to Reconfigure responses.
- Normative text, however, instructs that the client SHOULD include a Client Identifier and that a Server Identifier, while required for Reconfigure-triggered requests, may also be used in other cases to direct the request.

**Evidence Snippets:**

- **E1:**

  Inform.   * (see note)              *           *
NOTE: The Server Identifier option (see Section 21.3) is only included in Information-request messages that are sent in response to a Reconfigure (see Section 18.2.6).

- **E2:**

  Normative text for Information‑request messages: “The client SHOULD include a Client Identifier option (see Section 21.2)… When responding to a Reconfigure, the client MUST include a Server Identifier option (see Section 21.3)…

**Evidence Summary:**

- (E1) The table’s formatting and accompanying note create ambiguity over which identifier columns are marked.
- (E2) Normative sections require the inclusion of Client Identifier and allow Server Identifier beyond just Reconfigure responses.

**Fix Direction:**

Revise the table formatting to clearly assign a '*' to the Client Identifier column and to the Server Identifier column (with a separate footnote indicating its conditional requirement in Reconfigure responses), eliminating ambiguity.

**Severity:** Medium
  *Basis:* Ambiguity in the table may mislead implementers who rely on the summary, potentially impacting correct message formatting and interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: Issue-2
- Scope: Issue-1
- Deontic: Issue-2
- Structural: Issue-2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-B-3

**Label:** Lack of Clear Directionality in Reconfigure Row of Appendix B

**Bug Type:** Inconsistency

**Explanation:**

The Reconfigure row in Appendix B does not explicitly indicate that it applies only to server-generated messages, which might mislead readers into thinking the table summarizes bidirectional behavior.

**Justification:**

- The Reconfigure row is presented as 'Reconf.   *      *                                          *' without explicit directionality.
- Normative text (e.g., Section 18.3.11) clearly establishes that only the server sends Reconfigure messages, so the table should unambiguously reflect this.

**Evidence Snippets:**

- **E1:**

  Reconf.   *      *                                          *

- **E2:**

  The server sets the 'msg-type' field to RECONFIGURE... The server includes a Server Identifier option ... and a Client Identifier option ... in the Reconfigure message.

**Evidence Summary:**

- (E1) The table row for Reconfigure lacks explicit indication of the originating actor.
- (E2) Normative text clarifies that Reconfigure messages are exclusively server-originated.

**Fix Direction:**

Update Appendix B to explicitly denote that the Reconfigure row describes server-to-client messages only.

**Severity:** Low
  *Basis:* Although the normative text is clear, the ambiguous presentation in the table could mislead casual readers without affecting protocol operation.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: NewIssue-1

---
