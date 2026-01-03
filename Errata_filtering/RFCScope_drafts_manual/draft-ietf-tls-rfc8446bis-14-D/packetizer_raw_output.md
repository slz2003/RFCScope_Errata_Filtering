# Errata Reports

Total reports: 1

---

## Report 1: draft-ietf-tls-rfc8446bis-14-D-1

**Label:** No substantive protocol or interoperability error in TLS 1.2 update/renaming text

**Bug Type:** None

**Explanation:**

Section D renames TLS 1.2 terminology (e.g., 'master secret' to 'main secret') and updates the RFC 7627 extension name without affecting on‐the-wire behavior, resulting in no protocol or interoperability error.

**Justification:**

- The candidate issue explicitly notes that there is 'No substantive protocol or interoperability error in TLS 1.2 update/renaming text' (E1).
- The routing summary explains that the section only renames terminology for compatibility and does not change protocol mechanisms (E2).

**Evidence Snippets:**

- **E1:**

  Candidate Issues: 1

  Issue 1:
    Type: None
    Label: No substantive protocol or interoperability error in TLS 1.2 update/renaming text
    Relevant Dimensions: 
    Sketch: Section D only renames the TLS 1.2 “master/premaster” terminology and the RFC 7627 extension name an...

- **E2:**

  Excerpt Summary: Section D normatively renames TLS 1.2 terms from RFC 5246 and RFC 7627 (e.g., “master secret” → “main secret”, “extended_master_secret” extension → “extended_main_secret”), while explicitly keeping the on-the-wire PRF label strings unchanged for compatibility. It is a terminology and registry-update section, not a new protocol mechanism.

**Evidence Summary:**

- (E1) Candidate issue noting that there is no substantive protocol or interoperability error in the renaming text.
- (E2) Summary description outlining that the section only renames terminology without altering protocol behavior.

**Severity:** Low
  *Basis:* All evaluation dimensions (Temporal, Causal, Quantitative, etc.) are rated LOW, indicating no significant adverse impact.

**Confidence:** High

**Experts mentioning this issue:**

- Router: Issue 1

---
