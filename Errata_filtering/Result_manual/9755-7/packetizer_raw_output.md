# Errata Reports

Total reports: 6

---

## Report 1: 9755-7-1

**Label:** Temporal ambiguity in UTF8=ONLY quoted-string transmission pre-ENABLE

**Bug Type:** Inconsistency

**Explanation:**

There is a temporal inconsistency between Sections 3 and 7 regarding when a UTF8=ONLY server may send UTF-8 in quoted-strings.

**Justification:**

- Section 3 prohibits sending UTF-8 in quoted-strings until the client has issued ENABLE UTF8=ACCEPT, yet Section 7 asserts that a UTF8=ONLY server will send UTF-8 in quoted-strings as a capability property.
- This conflicting temporal requirement creates uncertainty about the server’s behavior in the early session phase.

**Evidence Snippets:**

- **E1:**

  “The IMAP server MUST NOT send UTF-8 in quoted-strings to the client unless the client has indicated support for that syntax by using the 'ENABLE UTF8=ACCEPT' command.” (Section 3)

- **E2:**

  “The 'UTF8=ONLY' capability indicates that the server supports 'UTF8=ACCEPT' … and that it requires support for UTF‑8 from clients. In particular, this means that the server will send UTF-8 in quoted-strings, and it will not accept the older international mailbox name convention (modified UTF‑7 [RFC3501]).” (Section 7)

- **E3:**

  “Because these are incompatible changes to IMAP, explicit server announcement and client confirmation are necessary: clients MUST use the 'ENABLE UTF8=ACCEPT' command before using this server.” (Section 7)

**Evidence Summary:**

- (E1) Section 3 mandates that UTF-8 in quoted-strings is not sent until after ENABLE UTF8=ACCEPT is issued.
- (E2) Section 7 describes UTF8=ONLY as unconditionally sending UTF-8 in quoted-strings.
- (E3) The requirement for client confirmation before using the server is stressed, creating a temporal inconsistency.

**Fix Direction:**

Clarify that a UTF8=ONLY server shall only send UTF-8 in quoted-strings after the client has successfully issued ENABLE UTF8=ACCEPT.


**Severity:** Medium
  *Basis:* The conflicting instructions could cause servers to prematurely send UTF-8, potentially breaking legacy client behavior.

**Confidence:** High

---

## Report 2: 9755-7-2

**Label:** Inappropriate use of [CANNOT] response code for recoverable commands

**Bug Type:** Inconsistency

**Explanation:**

The specification requires sending a 'NO [CANNOT]' response for pre-ENABLE commands that might require UTF-8 support, yet RFC 5530 reserves [CANNOT] for operations that can never succeed.

**Justification:**

- Section 7 mandates rejection with 'NO [CANNOT]' for any command that might require UTF-8 support when ENABLE has not been issued.
- RFC 5530 defines [CANNOT] to mean that an operation violates an invariant and can never succeed, which contradicts the temporary nature of this precondition.

**Evidence Snippets:**

- **E4:**

  “A server that advertises 'UTF8=ONLY' will reject, with a 'NO [CANNOT]' response [RFC5530], any command that might require UTF‑8 support and is not preceded by an 'ENABLE UTF8=ACCEPT' command.” (Section 7)

- **E5:**

  “If an IMAP server supports 'UTF8=ACCEPT' and the IMAP client has not issued the 'ENABLE UTF8=ACCEPT' command, the server MUST reject, with a 'NO' response, an 'APPEND' command that includes any 8-bit character in message header fields.” (Section 4)

- **E6:**

  RFC 5530 defines: “CANNOT — The operation violates some invariant of the server and can never succeed.”

**Evidence Summary:**

- (E4) shows the mandated use of NO [CANNOT] in Section 7 for commands sent before ENABLE.
- (E5) illustrates that similar commands are expected to be rejected pre-ENABLE.
- (E6) establishes that [CANNOT] should indicate a permanent failure, conflicting with the temporary precondition.

**Fix Direction:**

Either adopt a different response code for transient pre-ENABLE failures or adjust the definition of [CANNOT] so that its use is consistent with RFC 5530.


**Severity:** Medium
  *Basis:* Misusing a permanent failure code for a temporary condition may lead clients to improperly cease retrying recoverable commands, harming interoperability.

**Confidence:** High

---

## Report 3: 9755-7-3

**Label:** Undefined scope of commands requiring UTF‑8 support

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly define which commands 'might require UTF‑8 support', leaving it open to divergent interpretations across implementations.

**Justification:**

- The specification uses the phrase 'any command that might require UTF‑8 support' without providing a normative list or detailed criteria.
- This vagueness can lead to different servers rejecting different sets of commands in the pre-ENABLE state.

**Evidence Snippets:**

- **E7:**

  “A server that advertises 'UTF8=ONLY' will reject, with a 'NO [CANNOT]' response, any command that might require UTF‑8 support and is not preceded by an 'ENABLE UTF8=ACCEPT' command.” (Section 7)

- **E8:**

  The specification does not concretely state which commands fall into 'might require UTF‑8 support' (as noted by the Scope Expert in Issue-1).

- **E9:**

  The document never gives a criterion or list for determining which IMAP commands should be rejected pre-ENABLE, leading to potential boundary ambiguities (Boundary Expert: Finding-1).

**Evidence Summary:**

- (E7) highlights the ambiguous language used in Section 7.
- (E8) points out the lack of concrete criteria for determining the command set.
- (E9) emphasizes that the absence of guidelines can result in inconsistent behavior across server implementations.

**Fix Direction:**

Provide a clear, normative list or specific criteria for which commands are considered to require UTF‑8 support, or specify that the gating decision is made on a per-invocation basis.


**Severity:** Low
  *Basis:* Although the ambiguity may lead to some interoperability differences, compliant clients are expected to issue ENABLE UTF8=ACCEPT immediately, minimizing practical impact.

**Confidence:** High

---

## Report 4: 9755-7-4

**Label:** Ambiguity in 'before using this server' relative to authentication

**Bug Type:** Underspecification

**Explanation:**

The instruction that clients must use ENABLE UTF8=ACCEPT 'before using this server' is ambiguous because ENABLE is only valid in the authenticated state, leaving unclear what commands may be issued between authentication and enabling.

**Justification:**

- Section 3 restricts ENABLE UTF8=ACCEPT to the authenticated state, yet Section 7 requires it before 'using the server', suggesting a broader timeframe.
- This discrepancy creates uncertainty about whether pre-ENABLE commands post-authentication are allowable.

**Evidence Snippets:**

- **E10:**

  “The 'ENABLE UTF8=ACCEPT' command is only valid in the authenticated state.” (Section 3)

- **E11:**

  “clients MUST use the 'ENABLE UTF8=ACCEPT' command before using this server.” (Section 7)

**Evidence Summary:**

- (E10) shows that ENABLE is only allowed after authentication.
- (E11) mandates using ENABLE before any other server usage, leading to a conflict that is not clearly resolved.

**Fix Direction:**

Clarify that 'using the server' refers specifically to post-ENABLE operations and that the client should issue ENABLE UTF8=ACCEPT immediately after authentication.


**Severity:** Low
  *Basis:* The potential ambiguity affects only a narrow transition period and is unlikely to cause significant interoperability issues if clients follow recommended practices.

**Confidence:** High

---

## Report 5: 9755-7-5

**Label:** Incorrect cross-reference for UTF8=ACCEPT definition

**Bug Type:** Inconsistency

**Explanation:**

The document mistakenly refers to the definition of the UTF8=ACCEPT option as being in Section 4, even though Section 4 covers only the APPEND Command, which may mislead implementers.

**Justification:**

- The cross-reference in Section 3 directs the reader to Section 4 for the definition of UTF8=ACCEPT, but Section 4 does not provide that definition.
- This creates confusion about where the authoritative description of the UTF8=ACCEPT capability is located.

**Evidence Snippets:**

- **E12:**

  “A client MUST use the ‘ENABLE’ command [RFC5161] with the ‘UTF8=ACCEPT’ option (defined in Section 4 below) to indicate to the server that the client accepts UTF-8 in quoted-strings and supports the ‘UTF8=ACCEPT’ extension.” (Section 3)

- **E13:**

  “Section 4 heading: ‘APPEND Command’” (as noted by the Terminology Expert)

**Evidence Summary:**

- (E12) shows the erroneous reference claiming that UTF8=ACCEPT is defined in Section 4.
- (E13) confirms that Section 4 does not contain the definition, leading to internal inconsistency.

**Fix Direction:**

Correct the cross-reference in Section 3 so that it points to the section where UTF8=ACCEPT is actually defined (or modify the text to state it is defined in the current section).


**Severity:** Medium
  *Basis:* Incorrect cross-references can hinder implementers’ ability to correctly interpret the specification.

**Confidence:** High

---

## Report 6: 9755-7-6

**Label:** Incorrect media type 'message/rfc' instead of 'message/rfc822'

**Bug Type:** Inconsistency

**Explanation:**

The document incorrectly uses the non-existent media type 'message/rfc' where 'message/rfc822' is the recognized standard, which may lead to confusion in MIME handling.

**Justification:**

- One part of the text states that message/global is treated like 'message/rfc', while another part correctly uses 'message/rfc822', revealing a typographical error.
- This discrepancy may mislead implementers regarding the correct media type for email messages.

**Evidence Snippets:**

- **E14:**

  “Section 6, first paragraph: [RFC9051], Section 7.5.2 treats message/global like message/rfc, which means that for some messages, the response to FETCH BODYSTRUCTURE varies depending on whether IMAP4rev1 or IMAP4rev2 is in use.”

- **E15:**

  “Section 6, later paragraph: When IMAP4rev1 and UTF8=ACCEPT has been enabled, the server MAY treat message/global like message/rfc822 when computing the body structure…”

**Evidence Summary:**

- (E14) shows the erroneous usage of 'message/rfc', while (E15) demonstrates the intended correct usage of 'message/rfc822'.

**Fix Direction:**

Replace all instances of 'message/rfc' with 'message/rfc822' in the document to align with standard media type definitions.


**Severity:** Medium
  *Basis:* Using an incorrect media type can cause confusion when interfacing with MIME systems and referencing external standards.

**Confidence:** High

---
