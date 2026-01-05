# Errata Reports

Total reports: 1

---


## Report 4: draft-ietf-dhc-rfc8415bis-12-C-4

**Label:** Misnamed Server Identifier requirement in Release message construction

**Bug Type:** Causal Inconsistency and Terminology Error

**Explanation:**

Section 18.2.7 incorrectly specifies that a Server Identifier option must be included in the Renew message while describing Release message construction, which may cause implementers to omit the necessary Server Identifier in Release messages.

**Justification:**

- The section discussing Release message construction states “The client MUST include a Server Identifier option ... in the Renew message” even though it is dedicated to Release messages.
- Server-side validation in Section 16.9 mandates that Release messages include a Server Identifier, and the correct text for Renew is already given in Section 18.2.4.
- Multiple experts (Causal, Structural, and Terminology) have identified this misreference as a normative error.

**Evidence Snippets:**

- **E1:**

  Section 18.2.7: “The client MUST include a Server Identifier option (see Section 21.3) in the Renew message, identifying the server which allocated the lease(s).”

- **E2:**

  Section 16.9: “Servers MUST discard any received Release message that meets any of the following conditions: … the message does not include a Server Identifier option (see Section 21.3).”

- **E3:**

  Section 18.2.4 correctly states: “The client MUST include a Server Identifier option (see Section 21.3) in the Renew message, identifying the server with which the client most recently communicated.”

**Evidence Summary:**

- (E1) shows the misnamed requirement in Section 18.2.7 for Release messages.
- (E2) confirms that Release messages are required to include a Server Identifier option.
- (E3) contrasts with the proper text for Renew, highlighting the error.

**Fix Direction:**

Replace the phrase 'in the Renew message' with 'in the Release message' in Section 18.2.7 to align with normative requirements.

**Severity:** Medium
  *Basis:* If implemented literally, Release messages lacking a Server Identifier may be silently discarded by servers, affecting lease release behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Issue in Section 18.2.7
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- Terminology Expert: Issue-1

---