# Errata Reports

Total reports: 1

---


## Report 1: 9716-4-1

**Label:** Underspecified error handling for Segment sub-TLVs 46–48 in TLVs 1 or 16

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that if Segment sub-TLVs 46–48 appear in TLVs 1 or 16, 'appropriate error codes' must be returned, but it fails to specify which error codes or procedures to use, causing ambiguity.

**Justification:**

- Multiple expert analyses note that RFC 9716 delegates error handling to RFC8029 without clarifying the exact Return Code, Subcode, or diagnostic TLV details.
- This underspecification may lead to divergent implementations and inconsistent error reporting in operational diagnostics.

**Evidence Snippets:**

- **E1:**

  If these sub-TLVs appear in TLVs 1 or 16, appropriate error codes MUST be returned as defined in [RFC8029].

- **E2:**

  RFC 8029 defines mechanisms for malformed echo requests and “TLVs not understood” at the top-level, but does not specify a procedure for unrecognized sub-TLV types within TLVs 1 or 16.

- **E3:**

  This document defines the usage and processing of the Type-A, Type-C, and Type-D Segment sub-TLVs when they appear in TLV 21 (Reply Path TLV). If these sub-TLVs appear in TLVs 1 or 16, appropriate error codes MUST be returned as defined in [RFC8029].

**Evidence Summary:**

- (E1) Mandates return of 'appropriate error codes' when sub-TLVs 46–48 are present in TLVs 1 or 16.
- (E2) Highlights that RFC8029 does not define specific error procedures for such sub-TLVs.
- (E3) Reinforces the requirement from RFC9716 without providing further clarification.

**Fix Direction:**

Clarify the exact error handling by specifying the precise Return Code/Subcode and diagnostic behavior (e.g., inclusion of an Errored TLVs TLV) for cases where sub-TLVs 46–48 appear in TLVs 1 or 16.


**Severity:** Medium
  *Basis:* Ambiguity in error handling can result in inconsistent diagnostics and interoperability issues between different implementations.

**Confidence:** High

---