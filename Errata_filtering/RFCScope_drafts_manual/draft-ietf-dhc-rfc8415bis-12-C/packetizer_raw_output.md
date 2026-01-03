# Errata Reports

Total reports: 5

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-C-1

**Label:** IA_LL T1/T2=0 behavior underspecified in Section 14.2

**Bug Type:** Underspecification

**Explanation:**

When T1/T2 values are set to 0 for IA_LL, Section 14.2—written only for IA_NA and IA_PD—leaves the renewal behavior for IA_LL undefined.

**Justification:**

- Section 14.2 explicitly handles only IA_NA and IA_PD, while the normative text for IA_LL from RFC 8947 mandates following Section 14.2, resulting in ambiguity.
- This gap may lead an implementer to choose arbitrary renewal times for IA_LL when T1/T2 are zero.

**Evidence Snippets:**

- **E1:**

  “This document defines three IA types: IA_NA, IA_TA (obsoleted), and IA_PD. Another IA type (IA_LL) was defined in [RFC8947] and more may be defined.” (Section 4.2)

- **E2:**

  “In certain cases, T1 and/or T2 values may be set to 0. Currently, there are two such cases: 1. a client received an IA_NA option… 2. a client received an IA_PD option…” (Section 14.2)

- **E3:**

  IA_LL definition: T1/T2 fields, recommended T1/T2 selection, and “If the time at which the addresses in an IA_LL are to be renewed is to be left to the discretion of the client, the server sets T1 and T2 to 0. The client MUST follow the rules defined in Section 14.2 of [RFC8415].”

**Evidence Summary:**

- (E1) identifies IA_LL as an additional IA type alongside IA_NA and IA_PD.
- (E2) shows that Section 14.2 only specifies behavior for IA_NA and IA_PD when T1/T2 are 0.
- (E3) indicates that IA_LL is expected to follow Section 14.2, leading to an underspecified renewal behavior.

**Fix Direction:**

Clarify Section 14.2 to explicitly include IA_LL or provide a separate renewal guidance for IA_LL when T1/T2 are set to 0.

**Severity:** Medium
  *Basis:* An unclear renewal timer strategy for IA_LL can lead to unintended renewal storms or inconsistent behavior across IA types.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-C-2

**Label:** IA_LL moved-to-new-link semantics underspecified

**Bug Type:** Underspecification

**Explanation:**

When a client detects a link change, the specification does not define how to handle IA_LL, leaving ambiguity over whether to trigger Renew, Rebind, or another exchange for IA_LL state.

**Justification:**

- Section 18.2.12 details the behavior for IA_NA and IA_PD during link changes but omits any guidance for IA_LL.
- The omission forces implementers to invent ad hoc behaviors for IA_LL during link transitions.

**Evidence Snippets:**

- **E1:**

  “This document defines three IA types: IA_NA, IA_TA (obsoleted), and IA_PD. Another IA type (IA_LL) was defined in [RFC8947] and more may be defined.” (Section 4.2)

- **E2:**

  The detailed rules in Section 18.2.12 for when a client may have moved to a new link enumerate behaviors for addresses and prefixes but do not mention IA_LL.

**Evidence Summary:**

- (E1) establishes IA_LL as a recognized IA type.
- (E2) demonstrates that Section 18.2.12 does not account for IA_LL, causing underspecification.

**Fix Direction:**

Explicitly define how IA_LL bindings are to be handled on link changes in Section 18.2.12 or refer to RFC8947 for the intended behavior.

**Severity:** Low
  *Basis:* While divergent implementations might emerge, the impact on overall protocol correctness is limited.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-C-3

**Label:** Narrowing of 'IA option(s)' term excludes IA_LL from generic renewal rules

**Bug Type:** Inconsistency and Underspecification

**Explanation:**

The draft now defines 'IA option(s)' narrowly as including only IA_NA, IA_TA, and IA_PD, which may inadvertently exclude IA_LL and future IA types from the generic T1/T2 timer selection and combined Renew/Rebind procedures.

**Justification:**

- RFC 8415 originally allowed for future IA types by defining 'IA option(s)' in an open-ended manner, but the revised text restricts it to specific types.
- As a result, normative rules for T1/T2 and joint renewal may not apply to IA_LL, leading to potential inconsistencies.

**Evidence Snippets:**

- **E1:**

  RFC 8415 defined “IA” and “IA option(s)” with explicit allowance for future IA types.

- **E2:**

  The draft now defines “IA option(s)” narrowly as “one or more IA_NA, IA_TA (obsoleted), and/or IA_PD”, while only mentioning IA_LL in a side note.

- **E3:**

  T1/T2 are still defined only in terms of addresses via IA_NA and prefixes via IA_PD, with no mention that they also govern IA_LL timers.

**Evidence Summary:**

- (E1) highlights the original open-ended intent regarding IA types.
- (E2) shows the narrowed definition in the revised draft.
- (E3) demonstrates that the timer rules do not explicitly cover IA_LL, creating a gap in applicability.

**Fix Direction:**

Broaden the definition of 'IA option(s)' to explicitly include IA_LL or add a clarifying statement that generic T1/T2 and renewal rules apply to all IA types unless otherwise specified.

**Severity:** Medium
  *Basis:* Excluding IA_LL from renewal rules may lead to divergent timing behavior across IA types and interoperability issues.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1

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

## Report 5: draft-ietf-dhc-rfc8415bis-12-C-5

**Label:** Inconsistencies in Appendix B/C option tables regarding IA_TA and IA_LL/LLADDR

**Bug Type:** Editorial/Clarity Issue

**Explanation:**

The informational tables in Appendices B and C include outdated entries like IA_TA and omit details for IA_LL/LLADDR, potentially confusing readers about the normative option scope.

**Justification:**

- The tables are labeled as informational and note that conflicts with earlier text should defer to the normative sections.
- Despite the normative text defining IA_LL/LLADDR in RFC8947, the tables do not reflect these updates and continue to show legacy entries, which could lead to misinterpretation.

**Evidence Snippets:**

- **E1:**

  “These tables are informational. If they conflict with text earlier in this document, that text should be considered authoritative.”

- **E2:**

  IA_LL/LLADDR scoping: “An IA_LL option may only appear in the options area of a DHCP message. … The Link-Layer Addresses option … must be encapsulated in the IA_LL-options field of an IA_LL option.”

**Evidence Summary:**

- (E1) emphasizes that the tables are informational and may not reflect current normative definitions.
- (E2) shows that the normative scoping for IA_LL/LLADDR is defined elsewhere, highlighting the inconsistency with the tables.

**Fix Direction:**

Revise the Appendix B/C tables to accurately include current normative definitions for IA_LL/LLADDR and remove or clearly mark legacy entries such as IA_TA.

**Severity:** Low
  *Basis:* Although it may cause confusion, the mismatches in the informational tables do not impact the core protocol operation.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-2

---
