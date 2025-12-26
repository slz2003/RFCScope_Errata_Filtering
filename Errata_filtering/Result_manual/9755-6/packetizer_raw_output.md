# Errata Reports

Total reports: 2

---

## Report 1: 9755-6-1

**Label:** IMAP4rev2 message/global BODYSTRUCTURE underspecification: Scoped to UTF8=ACCEPT

**Bug Type:** Underspecification

**Explanation:**

The specification conditions IMAP4rev2 servers to follow RFC 9051’s message/global BODYSTRUCTURE rules only when UTF8=ACCEPT is in use, which may lead implementers to erroneously believe that RFC 9051’s mandatory behavior does not apply to legacy sessions.

**Justification:**

- RFC 9755 Section 6 states that 'When IMAP4rev2 and UTF8=ACCEPT are in use, the server MUST behave as described in [RFC9051]', thereby tying the behavior to the UTF8=ACCEPT extension.
- The conditional phrasing could mislead implementers into thinking that non-UTF8 sessions are exempt from RFC 9051’s requirements, despite RFC 9051 being a core part of IMAP4rev2.

**Evidence Snippets:**

- **E1:**

  RFC 9755 Section 6: “When IMAP4rev2 and UTF8=ACCEPT are in use, the server MUST behave as described in [RFC9051].”

- **E2:**

  Because the “MUST behave as described in [RFC9051]” clause is explicitly conditioned on both “IMAP4rev2” and “UTF8=ACCEPT are in use”, an implementer could reasonably infer that the RFC 9051 behavior for message/global BODYSTRUCTURE is only mandated in sessions where the UTF8=ACCEPT extension is active.

**Evidence Summary:**

- (E1) Shows the conditional requirement tied to UTF8=ACCEPT.
- (E2) Explains how this phrasing could be misinterpreted as limiting RFC 9051’s scope.

**Fix Direction:**

Clarify the text to state that for IMAP4rev2, RFC 9051’s requirements for message/global BODYSTRUCTURE apply in all cases, regardless of UTF8=ACCEPT.


**Severity:** Medium
  *Basis:* The conditional language may cause inconsistent implementations and interoperability issues by implying a narrower scope than intended.

**Confidence:** High

---

## Report 2: 9755-6-2

**Label:** Incorrect media type reference: 'message/rfc' instead of 'message/rfc822'

**Bug Type:** Inconsistency

**Explanation:**

RFC 9755 mistakenly references the non-existent media type 'message/rfc' rather than using the standardized and widely recognized 'message/rfc822', leading to potential confusion in implementation.

**Justification:**

- The document’s Section 6 states that [RFC9051] treats message/global like message/rfc, which is inconsistent with established MIME standards.
- Both the ABNF in RFC 9755 and references in RFC 9051 correctly use 'message/rfc822', highlighting the error in the initial reference.

**Evidence Snippets:**

- **E1:**

  RFC 9755 Section 6: “[RFC9051], Section 7.5.2 treats message/global like message/rfc, which means that for some messages, the response to FETCH BODYSTRUCTURE varies depending on whether IMAP4rev1 or IMAP4rev2 is in use.”

- **E2:**

  RFC 9051 ABNF and text only define MESSAGE/RFC822 and MESSAGE/GLOBAL as special message types, e.g. media-message = DQUOTE "MESSAGE" DQUOTE SP DQUOTE ("RFC822" / "GLOBAL") DQUOTE and “A body type of type MESSAGE/RFC822 or MESSAGE/GLOBAL also has nested part numbers…”

- **E3:**

  Section 6 of RFC 9755 later correctly refers to message/rfc822 in its comparison with message/global, reinforcing that the proper reference should be message/rfc822.

**Evidence Summary:**

- (E1) Highlights the incorrect use of 'message/rfc' in Section 6.
- (E2) Demonstrates that the ABNF and other texts define only MESSAGE/RFC822 and MESSAGE/GLOBAL.
- (E3) Indicates that the inconsistency is localized to one part of the document, as later text uses the correct term.

**Fix Direction:**

Replace 'message/rfc' with 'message/rfc822' throughout RFC 9755, particularly in Section 6, to ensure consistency with MIME standards and related RFCs.


**Severity:** Medium
  *Basis:* The misnaming introduces ambiguity that can lead to misinterpretation and potential incompatibilities in implementations.

**Confidence:** High

---
