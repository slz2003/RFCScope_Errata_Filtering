# Errata Reports

Total reports: 2

---

## Report 1: 9755-9-1

**Label:** Ambiguity in Definition of 'Header Information Transmitted over the Wire'

**Bug Type:** Documentation Ambiguity / Underspecification

**Explanation:**

The specification does not explicitly enumerate which IMAP data items count as header information, which may leave implementers uncertain about what must be rendered as 7‑bit ASCII.

**Justification:**

- The Scope Expert notes that the phrase 'all header information transmitted over the wire' is not spelled out in terms of specific IMAP data items (such as whether ENVELOPE fields or encapsulated MESSAGE/RFC822 parts are included).
- Residual uncertainties are acknowledged in the document, indicating that the complete set of header items is not enumerated in one place, even though this appears to be an intentional design choice.

**Evidence Snippets:**

- **E1:**

  NotedAmbiguities:
    - “All header information transmitted over the wire” is not spelled out in terms of specific IMAP data items (e.g., whether this phrase explicitly includes:
      - `ENVELOPE` fields (subject, address personal names, etc.),
      - headers of encapsulated `MESSAGE/RFC822` and `message/global` parts).

- **E2:**

  ResidualUncertainties:
  - The document does not enumerate, in one place, the complete set of IMAP data items that are considered “header information” (e.g., ENVELOPE fields, headers of `message/global` parts), so implementers must combine RFC 9755 with RFC 3501/5322/6532 to derive that set. However, this appears to be by design and does not conflict with the existing normative scopes.

**Evidence Summary:**

- (E1) The Scope Expert’s NotedAmbiguities points out that the term 'all header information transmitted over the wire' lacks a precise definition in terms of specific IMAP fields.
- (E2) The ResidualUncertainties section clarifies that a complete enumeration of header items is missing, requiring implementers to cross-reference other RFCs.


**Severity:** Low
  *Basis:* The ambiguity is unlikely to break protocol compliance since implementers can derive the intended set using RFC 3501/5322, but it may lead to inconsistent implementations.

**Confidence:** High
---

## Report 2: 9755-9-2

**Label:** Unspecified Downgrade/Hiding Strategy for Legacy Clients

**Bug Type:** Underspecification

**Explanation:**

The document leaves the method for ensuring 7‑bit header output for legacy clients intentionally unspecified, which may lead to divergent user experiences across different server implementations.

**Justification:**

- Section 9 mandates that all header information sent to non‑UTF8=ACCEPT clients conform to RFC 3501 and RFC 5322, but defers the strategy for handling non‑ASCII headers to Section 8 without prescribing a unique solution.
- Multiple experts note that acceptable strategies include hiding problematic messages, generating surrogates, or performing downgrading, resulting in an expected divergence in behavior.

**Evidence Snippets:**

- **E1:**

  DeonticAnalysis:
“When a message that requires SMTPUTF8 is encountered and the client does not enable UTF-8 capability, choices available to the server include hiding the problematic message(s), creating in‑band or out‑of‑band notifications or error messages, or somehow trying to create a surrogate of the message…” and “Implementations that choose to perform downgrading SHOULD use one of the standardized algorithms provided in [RFC6857] or [RFC6858].”

- **E2:**

  BoundaryAnalysis ExcerptEvidence:
      • Section 8 of RFC 9755 explicitly notes multiple possible strategies: hiding messages, creating surrogate/downgraded messages, using RFC 6857/6858 algorithms, and concludes that the “best (or ‘least bad’) approach” depends on local conditions.

- **E3:**

  CausalAnalysis:
“The result is divergent user experience, but not a fundamental interoperability or correctness failure at the IMAP protocol level.”

**Evidence Summary:**

- (E1) Deontic Expert analysis details the multiple acceptable strategies for handling headers when the client does not enable UTF8, without mandating a single approach.
- (E2) Boundary Expert evidence underscores that Section 8 lays out several options, emphasizing local discretion in choosing a downgrade method.
- (E3) Causal Expert remarks that the resulting divergent user experience is an anticipated outcome of this design choice.


**Severity:** Low
  *Basis:* The open-ended downgrade/hiding strategy is a deliberate design trade-off that does not impair protocol interoperability, though it may result in variable client experiences.

**Confidence:** High
---
