# Errata Reports

Total reports: 7

---

## Report 1: 9755-3-1

**Label:** Ambiguous Timing for Client UTF-8 Quoted-Strings Relative to ENABLE

**Bug Type:** Underspecification

**Explanation:**

The specification is unclear whether a client may begin sending UTF-8 quoted-strings immediately upon the server advertising the UTF8=ACCEPT capability or only after a successful ENABLE UTF8=ACCEPT command.

**Justification:**

- One part of the text requires that a client MUST use the ENABLE command to indicate support for UTF-8 quoted-strings, while another part allows the client to use extended quoted syntax if the server advertises support.
- This discrepancy leads to two plausible, incompatible interpretations regarding the effective timing for client-sent UTF-8 quoted-strings.

**Evidence Snippets:**

- **E1:**

  A client MUST use the 'ENABLE' command [RFC5161] with the 'UTF8=ACCEPT' option … to indicate to the server that the client accepts UTF-8 in quoted-strings and supports the 'UTF8=ACCEPT' extension.

- **E2:**

  If the server supports 'UTF8=ACCEPT', the client MAY use extended quoted syntax with any IMAP argument that permits a string…

**Evidence Summary:**

- (E1) The requirement to use ENABLE implies precondition for sending UTF-8 quoted-strings.
- (E2) The allowance based solely on capability advertisement creates ambiguity in the client’s permitted timeline.

**Fix Direction:**

Modify the text to clearly require that clients may only send UTF-8 quoted-strings after successful ENABLE (or explicitly state the intended timeline), in order to resolve the conflict between the MUST and MAY wording.


**Severity:** Medium
  *Basis:* Inconsistent timing may lead to interoperability failures between clients and servers regarding when UTF-8 quoted strings are allowed.

**Confidence:** High

---

## Report 2: 9755-3-2

**Label:** Conflicting APPEND Command Requirements for UTF-8 Headers Pre-ENABLE

**Bug Type:** Inconsistency

**Explanation:**

The APPEND command section contains contradictory instructions by both accepting UTF-8 headers and mandating rejection of 8‑bit header characters when ENABLE UTF8=ACCEPT has not been issued.

**Justification:**

- One sentence states that if the server supports UTF8=ACCEPT, then it accepts UTF-8 headers in the APPEND command message argument.
- The immediately following sentence requires that if the client has not issued ENABLE UTF8=ACCEPT, the server MUST reject any APPEND command containing 8‑bit header characters.

**Evidence Snippets:**

- **E1:**

  If the server supports 'UTF8=ACCEPT', then the server accepts UTF-8 headers in the 'APPEND' command message argument.

- **E2:**

  If an IMAP server supports 'UTF8=ACCEPT' and the IMAP client has not issued the 'ENABLE UTF8=ACCEPT' command, the server MUST reject, with a 'NO' response, an 'APPEND' command that includes any 8-bit character in message header fields.

**Evidence Summary:**

- (E1) The text unconditionally permits UTF-8 headers in APPEND commands.
- (E2) The text then mandates a rejection of such headers if ENABLE has not been issued, resulting in a direct conflict.

**Fix Direction:**

Revise the APPEND command requirement so that acceptance of UTF-8 headers is explicitly conditioned on the client having successfully issued ENABLE UTF8=ACCEPT.


**Severity:** Medium
  *Basis:* The contradiction can result in servers or clients behaving inconsistently, leading to potential interoperability issues.

**Confidence:** High

---

## Report 3: 9755-3-3

**Label:** Ambiguous Timing for SEARCH Charset Restrictions After ENABLE

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state whether a SEARCH command containing a charset specification violates requirements if it is pipelined with the ENABLE command or sent before the client receives an ENABLED response.

**Justification:**

- The text specifies that once a client has enabled UTF-8 support, it MUST NOT issue a SEARCH command with a charset specification.
- However, it remains ambiguous whether issuing a pipelined SEARCH immediately after sending ENABLE, but prior to receiving confirmation, constitutes a violation.

**Evidence Snippets:**

- **E1:**

  Once an IMAP client has enabled UTF-8 support with the 'ENABLE UTF8=ACCEPT' command, it MUST NOT issue a 'SEARCH' command that contains a charset specification.

- **E2:**

  RFC 5161 explicitly allows pipelining ENABLE with other commands, creating ambiguity about when the restriction applies.

**Evidence Summary:**

- (E1) The SEARCH command is forbidden to include a charset once UTF8=ACCEPT is enabled.
- (E2) Pipelining enables the possibility of a SEARCH command being sent before the client receives confirmation, leading to uncertainty.

**Fix Direction:**

Clarify in the specification whether the prohibition applies from the moment the ENABLE command is sent or only after an ENABLED response is received.


**Severity:** Low
  *Basis:* Although the ambiguity is subtle and likely limited to pipelining scenarios, it may still lead to minor interoperability inconsistencies.

**Confidence:** High

---

## Report 4: 9755-3-4

**Label:** Underspecified Behavior for NAMESPACE and ACL Under UTF8=ACCEPT

**Bug Type:** Underspecification

**Explanation:**

While the document claims that UTF8=ACCEPT 'affects' NAMESPACE and ACL, it fails to precisely define the encoding requirements for mailbox names and prefixes in these extensions.

**Justification:**

- The text states that the capability also affects other IMAP extensions such as NAMESPACE and ACL but does not detail what encoding rules apply.
- This lack of normative guidance may result in some servers providing UTF-8 responses while others continue using legacy modified UTF‑7 for namespace prefixes or ACL mailbox fields.

**Evidence Snippets:**

- **E1:**

  This capability also affects other IMAP extensions that can return mailbox names or their prefixes, such as NAMESPACE [RFC2342] and ACL [RFC4314].

- **E2:**

  Mailbox names MUST comply with the Net-Unicode Definition ([RFC5198], Section 2) with the specific exception that they MUST NOT contain control characters (U+0000 - U+001F and U+0080 - U+009F), a delete character (U+007F), a line separator (U+2028), or a paragraph separator (U+2029).

- **E3:**

  The text does not spell out what 'affects' concretely means for these syntactic constructs.

**Evidence Summary:**

- (E1) The capability is said to affect NAMESPACE and ACL, but the exact changes are not detailed.
- (E2) The global mailbox name rules are provided, yet no explicit guidance is given for the namespace prefixes or ACL mailbox fields.
- (E3) This omission leaves room for divergent implementations.

**Fix Direction:**

Explicitly define the requirements for NAMESPACE prefixes and ACL mailbox arguments, such that these elements must adhere to the same UTF‑8 Net‑Unicode rules as other mailbox names when UTF8=ACCEPT is enabled.


**Severity:** Medium
  *Basis:* Ambiguity in encoding requirements may cause inconsistent implementations across servers and clients, damaging interoperability.

**Confidence:** High

---

## Report 5: 9755-3-5

**Label:** Unclear Scope of Net‑Unicode Mailbox-Name Requirement Relative to Legacy Representations

**Bug Type:** Underspecification

**Explanation:**

The document unconditionally mandates that mailbox names comply with the Net‑Unicode Definition without clarifying whether this applies to all on‐wire representations or just to UTF‑8 representations in the UTF8=ACCEPT mode, leading to potential conflicts with legacy modified UTF‑7 usage.

**Justification:**

- The text requires 'Mailbox names MUST comply with the Net‑Unicode Definition' but also references legacy conventions from RFC 3501 which use modified UTF‑7.
- This creates uncertainty about whether the Net‑Unicode requirement is intended to apply universally or only when a client has fully enabled UTF8=ACCEPT.

**Evidence Snippets:**

- **E1:**

  Mailbox names MUST comply with the Net‑Unicode Definition ([RFC5198], Section 2) with the specific exception that they MUST NOT contain control characters (U+0000 - U+001F and U+0080 - U+009F), a delete character (U+007F), a line separator (U+2028), or a paragraph separator (U+2029).

- **E2:**

  RFC 3501 Section 5.1.3 defines the 'Mailbox International Naming Convention' based on modified UTF‑7, and servers 'SHOULD not return 8‑bit mailbox names in LIST or LSUB.'

**Evidence Summary:**

- (E1) The strict Net‑Unicode requirement is imposed unconditionally on mailbox names.
- (E2) Legacy mailbox naming specifies modified UTF‑7, creating ambiguity over which rule applies in mixed environments.

**Fix Direction:**

Clarify that the Net‑Unicode requirement applies only to mailbox names represented in UTF‑8 under the UTF8=ACCEPT extension while allowing modified UTF‑7 representations for legacy client compatibility, or clearly delineate the transition boundary.


**Severity:** Medium
  *Basis:* Without clear scoping, implementers may make divergent choices for mailbox name encoding, risking interoperability with legacy systems.

**Confidence:** High

---

## Report 6: 9755-3-6

**Label:** Incorrect Media Type Name 'message/rfc' Instead of 'message/rfc822'

**Bug Type:** Inconsistency

**Explanation:**

The text erroneously refers to the media type 'message/rfc' instead of the registered and correct 'message/rfc822', potentially causing confusion.

**Justification:**

- The excerpt states that RFC9051 treats message/global like 'message/rfc', yet elsewhere the correct type 'message/rfc822' is used.
- Since MIME standards define the registered type as 'message/rfc822', this misnaming is inconsistent.

**Evidence Snippets:**

- **E1:**

  [RFC9051], Section 7.5.2 treats message/global like message/rfc, which means that for some messages, the response to FETCH BODYSTRUCTURE varies depending on whether IMAP4rev1 or IMAP4rev2 is in use.

- **E2:**

  When IMAP4rev1 and UTF8=ACCEPT has been enabled, the server MAY treat message/global like message/rfc822 when computing the body structure, but MAY also treat it as described in [RFC3501].

**Evidence Summary:**

- (E1) The text incorrectly uses 'message/rfc' in a normative context.
- (E2) The presence of 'message/rfc822' elsewhere indicates that 'message/rfc' is a typographical error.

**Fix Direction:**

Change all instances of 'message/rfc' to 'message/rfc822' to align with the proper registered media type.


**Severity:** Medium
  *Basis:* Incorrect media type naming can lead to confusion in MIME handling and interoperability.

**Confidence:** High

---

## Report 7: 9755-3-7

**Label:** Misreferenced Section for 'UTF8=ACCEPT' ENABLE Option Definition

**Bug Type:** Inconsistency

**Explanation:**

The document incorrectly indicates that the 'UTF8=ACCEPT' option is defined in Section 4, while Section 4 deals only with APPEND command semantics, leading to an internal reference error.

**Justification:**

- The text states: 'A client MUST use the ‘ENABLE’ command [RFC5161] with the ‘UTF8=ACCEPT’ option (defined in Section 4 below)...' but Section 4 does not define this option.
- This misreferencing may confuse readers about where the capability and its usage are formally specified.

**Evidence Snippets:**

- **E1:**

  A client MUST use the 'ENABLE' command [RFC5161] with the 'UTF8=ACCEPT' option (defined in Section 4 below) to indicate to the server that the client accepts UTF-8 in quoted-strings and supports the 'UTF8=ACCEPT' extension.

- **E2:**

  Section 4 in this excerpt is titled 'APPEND Command' and only discusses behavior of APPEND when 'UTF8=ACCEPT' is supported and/or enabled; it does not 'define' the ENABLE option syntax or name.

**Evidence Summary:**

- (E1) The specification incorrectly refers to Section 4 as containing the definition of the 'UTF8=ACCEPT' option.
- (E2) Section 4 is concerned with APPEND command behavior, not the formal definition of the ENABLE option.

**Fix Direction:**

Revise the reference to indicate that the 'UTF8=ACCEPT' capability is defined in Section 3 or remove the erroneous section reference entirely.


**Severity:** Low
  *Basis:* This is an editorial error that may mislead readers but is unlikely to affect the implementation of the protocol.

**Confidence:** High

---
