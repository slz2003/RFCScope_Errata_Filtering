# Errata Reports

Total reports: 4

---

## Report 1: 9824-7-1

**Label:** Ambiguous NXNAME Exemption from 'MUST be ignored' Rule in NSEC/NSEC3 Bitmaps

**Bug Type:** Inconsistency

**Explanation:**

RFC 9824 permits the NXNAME bit to be set in responses but does not clearly exempt it from the 'MUST be ignored upon being read' rule, resulting in conflicting normative instructions for receivers.

**Justification:**

- The updated RFC 4034 §4.1.2 text allows NXNAME to appear in the bitmap yet retains the requirement that pseudo‐type bits be ignored upon reading.
- Multiple expert analyses (Scope, Causal, Deontic, Structural, CrossRFC, and Terminology) point out that this ambiguity can lead to implementations either ignoring the NXNAME bit or using it as a signal, undermining the intended function.

**Evidence Snippets:**

- **E1:**

  Bits representing pseudo-types MUST be clear, as they do not appear in zone data. If encountered, they MUST be ignored upon being read. There is one exception to this rule for Compact Denial of Existence (RFC 9824), where the NXNAME pseudo-type is allowed to appear in responses to nonexistent names.

- **E2:**

  This specification defines the use of NXNAME (128), a synthetic RR type to signal the presence of a nonexistent name. … This RR type is added to the NSEC Type Bit Maps field for responses to nonexistent names …

**Evidence Summary:**

- (E1) The updated text permits NXNAME to appear but does not explicitly exempt it from being ignored.
- (E2) RFC 9824 defines NXNAME as a signal, implying it should be interpreted rather than simply ignored.

**Fix Direction:**

Clarify RFC 9824 §7.1 to explicitly state that the 'MUST be ignored upon being read' rule does not apply to the NXNAME bit when used for Compact Denial of Existence.


**Severity:** Medium
  *Basis:* This normatively ambiguous exemption undermines the intended use of NXNAME as a signaling mechanism, potentially leading to inconsistent resolver behavior, even though it does not affect DNSSEC cryptographic validation.

**Confidence:** High

---

## Report 2: 9824-7-2

**Label:** NSEC3 Type Bit Maps Conflict with NXNAME Exception

**Bug Type:** Inconsistency and Underspecification

**Explanation:**

RFC 5155’s rules still require that Meta‑TYPE bits be clear and ignored, conflicting with RFC 9824’s use of NXNAME as an informative signal in NSEC3 responses.

**Justification:**

- RFC 5155 §3.2.1 mandates that Meta‑TYPE bits be set to zero and ignored on reading, without any exception for NXNAME.
- RFC 9824 specifies that, in Compact Denial responses, the NSEC3 bitmap should contain only the NXNAME Meta‑TYPE, creating a conflict between the two specifications.

**Evidence Snippets:**

- **E1:**

  RFC 5155 §3.2.1: “The encoding of the Type Bit Maps field is the same as that used by the NSEC RR, described in [RFC4034]… Bits representing Meta-TYPEs or QTYPEs … MUST be set to 0, since they do not appear in zone data. If encountered, they must be ignored upon reading.”

- **E2:**

  RFC 9824 §4 specifies for NSEC3 that: “In responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE. In responses to ENT names, the Type Bit Maps field will be empty.”

**Evidence Summary:**

- (E1) RFC 5155 requires Meta‑TYPE bits to be zero and ignored.
- (E2) RFC 9824 intends for NXNAME to be present in NSEC3 responses, which conflicts with RFC 5155’s mandate.

**Fix Direction:**

Modify RFC 5155 to explicitly incorporate an NXNAME exception, or state that the NXNAME exception in RFC 9824 applies to both NSEC and NSEC3 Type Bit Maps.


**Severity:** Medium
  *Basis:* The conflict forces implementers to choose between two incompatible normative requirements, which may lead to interoperability issues in the handling of Compact Denial responses.

**Confidence:** High

---

## Report 3: 9824-7-3

**Label:** Inadequate Scoping of NSEC Type Bit Maps 'RRset Existence' Semantics for NXNAME

**Bug Type:** Underspecification

**Explanation:**

The general rule in RFC 4034 that a set bit indicates an RRset exists does not exclude NXNAME, which is intended solely as a signaling mechanism and does not represent an actual RRset.

**Justification:**

- RFC 4034 §4.1.2 states that a set bit signifies the presence of an RRset at the NSEC owner name, a semantic that does not apply to NXNAME.
- NXNAME is defined in RFC 9824 as a Meta‑TYPE that stores no data and is used only to signal nonexistent names, leading to a semantic mismatch.

**Evidence Snippets:**

- **E1:**

  RFC 4034 §4.1.2 begins: “The Type Bit Maps field identifies the RRset types that exist at the NSEC RR’s owner name.” It further states: “If a bit is set, it indicates that an RRset of that type is present for the NSEC RR's owner name.”

- **E2:**

  NXNAME in RFC 9824 is a Meta‑TYPE that “stores no data in a DNS zone and cannot be usefully queried,” and is only added to the type bitmap as an indicator for nonexistent names.

**Evidence Summary:**

- (E1) RFC 4034 defines set bits as denoting the existence of an RRset.
- (E2) NXNAME is meant only as a signal and does not imply an actual RRset.

**Fix Direction:**

Revise RFC 4034 §4.1.2 to explicitly exclude NXNAME from implying the presence of an RRset, clarifying its role as a non-data-bearing signal in Compact Denial responses.


**Severity:** Medium
  *Basis:* This underspecification may lead to inconsistent interpretations of NXNAME, causing downstream tools and validators to misrepresent its purpose.

**Confidence:** High

---

## Report 4: 9824-7-4

**Label:** NXNAME Terminology Inconsistency: Meta‑TYPE vs Pseudo‑type

**Bug Type:** Inconsistency

**Explanation:**

NXNAME is inconsistently referred to as both a Meta‑TYPE and a pseudo‑type within RFC 9824, which could confuse implementers regarding its proper classification and behavior.

**Justification:**

- RFC 9824 Section 2 defines NXNAME as a Meta‑TYPE that stores no data, while Section 7.1 refers to it as a pseudo‑type allowed in responses.
- This mixed terminology might lead to uncertainty in how to map the concept of pseudo‑types from RFC 4034 to the Meta‑TYPE definition in RFC 6895.

**Evidence Snippets:**

- **E1:**

  From RFC 9824 Section 2: “This specification defines the use of NXNAME (128), a synthetic RR type to signal the presence of a nonexistent name. ... It is a 'Meta-TYPE', as defined in [RFC6895], and it stores no data in a DNS zone and cannot be usefully queried.”

- **E2:**

  From RFC 9824 Section 7.1: “There is one exception to this rule for Compact Denial of Existence (RFC 9824), where the NXNAME pseudo-type is allowed to appear in responses to nonexistent names.”

**Evidence Summary:**

- (E1) NXNAME is defined as a Meta‑TYPE in Section 2.
- (E2) Section 7.1 alternatively refers to NXNAME as a pseudo‑type.

**Fix Direction:**

Harmonize the terminology by either consistently referring to NXNAME as a Meta‑TYPE or by clarifying that it is a Meta‑TYPE used in a pseudo‑type context as per RFC 4034.


**Severity:** Low
  *Basis:* While the naming discrepancy may cause confusion among implementers, it does not affect the underlying protocol behavior.

**Confidence:** High

---
