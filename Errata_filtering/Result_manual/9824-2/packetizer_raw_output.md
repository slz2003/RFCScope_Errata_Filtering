# Errata Reports

Total reports: 1

---


## Report 1: 9824-2-1

**Label:** NXNAME in NSEC3 Bit Maps Conflicts with RFC 5155 Meta-TYPE Rule

**Bug Type:** Inconsistency

**Explanation:**

RFC 9824 mandates that the NXNAME Meta-TYPE bit be set in NSEC3 responses for nonexistent names, while RFC 5155 requires that all Meta-TYPE bits be 0 and ignored, creating a normative conflict.

**Justification:**

- RFC 9824 explicitly states that when NSEC3 is used, the NXNAME bit must be the sole entry in the Type Bit Maps field for nonexistent names.
- RFC 5155 Section 3.2.1 mandates that bits representing Meta-TYPEs must be set to 0 and ignored upon reading.

**Evidence Snippets:**

- **E1:**

  RFC 9824 Section 2: This specification defines the use of NXNAME (128), a synthetic RR type to signal the presence of a nonexistent name. … This RR type is added to the NSEC Type Bit Maps field for responses to nonexistent names, … If NSEC3 is being used, this RR type is the sole entry in the Type Bit Maps field. It is a ‘Meta-TYPE’, as defined in [RFC6895]…

- **E2:**

  RFC 5155 Section 3.2.1: Bits representing Meta-TYPEs or QTYPEs as specified in Section 3.1 of [RFC2929] or within the range reserved for assignment only to QTYPEs and Meta-TYPEs MUST be set to 0, since they do not appear in zone data. If encountered, they must be ignored upon reading.

- **E3:**

  RFC 9824 Section 4: In responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE. In responses to ENT names, the Type Bit Maps field will be empty.

**Evidence Summary:**

- (E1) Indicates that NXNAME is to be set in the NSEC3 Type Bit Maps for nonexistent names.
- (E2) Mandates that Meta-TYPE bits (including NXNAME) must be 0 and ignored.
- (E3) Confirms NXNAME is intended as the sole bit in NSEC3 responses.

**Fix Direction:**

Update RFC 5155 to include an exception for NXNAME similar to the update made for RFC 4034, allowing the NXNAME bit to be set and interpreted in NSEC3 responses.


**Severity:** High
  *Basis:* This conflict creates mutually exclusive normative requirements that can force implementations to choose between two standards, potentially undermining the intended NXNAME signaling mechanism.

**Confidence:** High

---