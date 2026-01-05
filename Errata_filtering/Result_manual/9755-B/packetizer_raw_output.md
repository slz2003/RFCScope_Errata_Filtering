# Errata Reports

Total reports: 1

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