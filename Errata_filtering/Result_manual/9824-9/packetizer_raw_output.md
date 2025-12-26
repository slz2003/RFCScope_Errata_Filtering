# Errata Reports

Total reports: 2

---

## Report 1: 9824-9-1

**Label:** Misnumbered CO EDNS Header Flag Bit in Table 2

**Bug Type:** Inconsistency

**Explanation:**

Section 5.1 describes the CO flag as occupying the second most significant bit of the 16‑bit EDNS header flags field (implying bit 14), but Section 9’s Table 2 incorrectly lists it as Bit 1, which conflicts with the established IANA numbering convention.

**Justification:**

- Section 5.1 clearly states that the CO flag is in the second most significant bit (implying its value should be 0x4000 when DO is at bit 15).
- Section 9’s Table 2 assigns CO to Bit 1, which under the IANA registry would mean the second least significant bit (0x0002), leading to divergent implementations.
- Multiple experts (Causal, Quantitative, Structural, CrossRFC, and Terminology) noted that this inconsistency can cause resolvers and authoritative servers to disagree on the CO flag’s presence, silently disabling its intended functionality.

**Evidence Snippets:**

- **E1:**

  Section 5.1: “A new EDNS0 [RFC6891] header flag is defined in the second most significant bit of the flags field in the EDNS0 OPT header. This flag is referred to as the Compact Answers OK (CO) flag.” and the diagram shows: 2: |DO|CO|                 Z                       |

- **E2:**

  Section 9, Table 2: “IANA has also allocated the following flag in the ‘EDNS Header Flags (16 bits)’ registry… | Bit 1 | CO   | Compact Answers OK | RFC 9824”

**Evidence Summary:**

- (E1) Section 5.1 specifies that CO is the second most significant bit, meaning it should be in the position immediately below DO (bit 14).
- (E2) Section 9 Table 2 incorrectly designates CO as Bit 1, conflicting with the intended bit numbering.

**Fix Direction:**

Update Section 9 Table 2 to list the CO flag as Bit 14, aligning it with the description in Section 5.1 and the established IANA EDNS header flags numbering (DO = bit 15, CO = bit 14).


**Severity:** High
  *Basis:* This misnumbering can lead to differing wire encodings between implementations, potentially causing the CO signaling feature to be silently ignored and breaking interoperability.

**Confidence:** High

---

## Report 2: 9824-9-2

**Label:** NXNAME Meta-TYPE in NSEC3 Bit Maps Conflicts with RFC 5155 Rules

**Bug Type:** Both

**Explanation:**

RFC 9824 requires the NXNAME Meta-TYPE (value 128) to be used in NSEC3 Type Bit Maps for indicating nonexistent names, but RFC 5155 mandates that Meta-TYPE bits be set to 0 and ignored. This normative conflict creates ambiguity in how NXNAME should be processed in NSEC3 responses.

**Justification:**

- RFC 9824 Sections 2 and 4 specify that for responses concerning nonexistent names the NSEC3 Type Bit Maps field will contain only the NXNAME Meta-TYPE, thereby signaling NXDOMAIN conditions.
- RFC 5155 Section 3.2.1 unambiguously states that bits corresponding to Meta-TYPEs must be set to 0 and ignored, and RFC 9824 does not offer an explicit exception for NSEC3 as it does for NSEC via its update to RFC 4034.
- Multiple experts (Causal, Quantitative, Deontic, Structural, and CrossRFC) emphasized that this conflict can lead to validators either ignoring the NXNAME bit or treating its presence as an error, undermining the intended compact denial of existence mechanism.

**Evidence Snippets:**

- **E1:**

  RFC 9824 Section 2: “This specification defines the use of NXNAME (128), a synthetic RR type to signal the presence of a nonexistent name. … This RR type is added to the NSEC Type Bit Maps field for responses to nonexistent names…”

- **E2:**

  RFC 9824 Section 4: “In responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE. In responses to ENT names, the Type Bit Maps field will be empty.”

- **E3:**

  RFC 5155 Section 3.2.1: “Bits representing Meta-TYPEs or QTYPEs as specified in Section 3.1 of [RFC2929] or within the range reserved for assignment only to QTYPEs and Meta-TYPEs MUST be set to 0, since they do not appear in zone data. If encountered, they must be ignored upon reading.”

**Evidence Summary:**

- (E1) RFC 9824 mandates the use of the NXNAME Meta-TYPE in NSEC3 bitmaps for signaling nonexistent names.
- (E2) RFC 9824 further specifies that for nonexistent names, the NXNAME bit is the only one set in the Type Bit Maps field.
- (E3) RFC 5155 contradicts this by requiring that all Meta-TYPE bits be zero and ignored, with no provided exception for NXNAME in NSEC3.

**Fix Direction:**

Explicitly update RFC 5155 (or add a clarifying exception in RFC 9824) to exempt NXNAME from the 'MUST be 0' rule in NSEC3 Type Bit Maps, ensuring that NXNAME is honored as intended in responses for nonexistent names.


**Severity:** High
  *Basis:* The conflicting normative requirements may lead to inconsistent handling of NXNAME in NSEC3 records, which can undermine the mechanism designed to distinguish between NXDOMAIN and NODATA responses.

**Confidence:** High

---
