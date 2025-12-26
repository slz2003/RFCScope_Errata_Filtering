# Errata Reports

Total reports: 2

---

## Report 1: 9755-B-1

**Label:** Incorrect MIME type 'message/rfc' instead of 'message/rfc822' in RFC 9755 Section 6

**Bug Type:** Inconsistency

**Explanation:**

Explanatory text in RFC 9755 Section 6 mistakenly uses 'message/rfc' instead of the correct MIME subtype 'message/rfc822', which can mislead implementers.

**Justification:**

- RFC 9755 Section 6 text indicates that message/global is treated like message/rfc, while the formal ABNF and Section B.2 clearly use 'RFC822' for the corresponding MIME subtype. (E1, E2)
- Additional references from RFC 9051 and RFC 6532 confirm that the correct registered type is 'message/rfc822'. (E3, E4)

**Evidence Snippets:**

- **E1:**

  RFC 9755 Section 6: “[RFC9051], Section 7.5.2 treats message/global like message/rfc, which means that for some messages, the response to FETCH BODYSTRUCTURE varies depending on whether IMAP4rev1 or IMAP4rev2 is in use.”

- **E2:**

  RFC 9755 Section 6 ABNF and related text:  
        media-message   = DQUOTE "MESSAGE" DQUOTE SP DQUOTE ("RFC822" / "GLOBAL") DQUOTE  
        and Section B.2: “[RFC6532] defines a new media type, message/global, which is substantially like message/rfc822 except that the submessage may (also) use the syntax defined in [RFC6532].”

- **E3:**

  RFC 9051 Section 7.5.2 (quoted in the provided excerpt): “Note that headers (part specifiers HEADER or MIME, or the header portion of a MESSAGE/RFC822 or MESSAGE/GLOBAL part) MAY be in UTF‑8.”

- **E4:**

  RFC 6532 Section 3.7: “The content of a message/global part is otherwise identical to that of a message/rfc822 part.”

**Evidence Summary:**

- (E1) RFC 9755 Section 6 uses 'message/rfc' in its descriptive text.
- (E2) The ABNF in Section 6 and the design rationale in Section B.2 clearly define the subtype as 'RFC822'.
- (E3) RFC 9051 confirms through its FETCH response details the use of 'MESSAGE/RFC822'.
- (E4) RFC 6532 establishes that the message/global part is compared against 'message/rfc822'.

**Fix Direction:**

Replace the phrase '[RFC9051], Section 7.5.2 treats message/global like message/rfc,' with '[RFC9051], Section 7.5.2 treats message/global like message/rfc822,' in RFC 9755 Section 6.


**Severity:** High
  *Basis:* The misnaming could lead implementers to use a non-existent MIME subtype, risking interoperability issues.

**Confidence:** High

---

## Report 2: 9755-B-2

**Label:** Inconsistent terminology for APPEND UTF8: 'data item' vs 'data extension'

**Bug Type:** Terminology

**Explanation:**

RFC 9755 Section B.1 inconsistently refers to the mechanism as a 'UTF8 data item' instead of the 'UTF8 data extension' term used in RFC 6855, causing minor editorial ambiguity.

**Justification:**

- RFC 9755 Section B.1 uses the phrasing 'APPEND's UTF8 data item' when explaining the removal of the old mechanism. (E1)
- RFC 6855 Section 4 consistently uses the term 'UTF8 data extension' to describe the same mechanism. (E2)

**Evidence Snippets:**

- **E1:**

  RFC 9755 Section B.1 heading and text:  
          B.1. APPEND UTF8  
          This document removes APPEND's UTF8 data item, making the UTF8-related syntax compatible with IMAP4rev2 as defined by [RFC9051] ...

- **E2:**

  RFC 6855 Section 4 title and wording:  
          4. IMAP UTF8 "APPEND" Data Extension  
          A client that sends a message with UTF-8 headers to the server MUST send them using the "UTF8" data extension to the "APPEND" command.

**Evidence Summary:**

- (E1) RFC 9755 Section B.1 refers to the mechanism as 'UTF8 data item'.
- (E2) RFC 6855 Section 4 defines the mechanism as 'UTF8 data extension' with matching ABNF.


**Severity:** Low
  *Basis:* The issue is primarily editorial; despite the inconsistent terminology, the intended reference is clear and does not impact interoperability.

**Confidence:** High
---
