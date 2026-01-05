# Errata Reports

Total reports: 2

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