# Errata Reports

Total reports: 5

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

## Report 2: 9824-2-2

**Label:** Ambiguous Resolver Handling for NXNAME

**Bug Type:** Underspecification

**Explanation:**

The document ambiguously states that resolvers require no special handling for NXNAME, yet later mandates specific behavior for queries and optional response code restoration, causing unclear resolver responsibilities.

**Justification:**

- Section 2 implies that resolvers can use standard processing with no special handling for NXNAME.
- Subsequent sections (e.g., Section 3.5 and Section 5.1) introduce mandatory actions for explicit NXNAME queries and optional behavior for signaling, creating contradictory guidance.

**Evidence Snippets:**

- **E4:**

  Section 2: 'No special handling of this RR type is required on the part of DNS resolvers. However, resolvers may optionally implement the behavior described in Section 5.1 ('Signaled Response Code Restoration')...'

- **E5:**

  Section 3.5: 'If an explicit query for the NXNAME RR type is received, the DNS server MUST return a Format Error (response code FORMERR). A resolver MUST NOT forward these queries upstream or attempt iterative resolution.'

**Evidence Summary:**

- (E4) Suggests that resolvers are not required to perform any special processing for NXNAME.
- (E5) Mandates specific processing for explicit NXNAME queries, leading to ambiguity.

**Fix Direction:**

Clarify the intended resolver behavior by explicitly distinguishing between standard processing for ordinary responses and mandatory handling for explicit NXNAME queries.


**Severity:** Medium
  *Basis:* Ambiguous resolver instructions may result in divergent implementations, though core DNS functionality is maintained.

**Confidence:** High

---

## Report 3: 9824-2-3

**Label:** Ambiguity in Section 2: Descriptive Pre-NXNAME vs. Normative NXNAME Behavior

**Bug Type:** Underspecification

**Explanation:**

The initial part of Section 2 describes a pre-NXNAME behavior for Compact Answers that conflicts with later normative requirements incorporating NXNAME, potentially confusing implementers about the intended operational mode.

**Justification:**

- The first paragraph of Section 2 describes nonexistent name responses as containing only NSEC and RRSIG, with empty bitmaps for NSEC3 and identical patterns for ENTs.
- Immediately following this, NXNAME is introduced and required to appear in responses, creating an unclear transition between historical description and new normative requirements.

**Evidence Snippets:**

- **E6:**

  RFC 9824 Section 2: 'This method generates NODATA responses for nonexistent names that don't match a DNS wildcard. Since there are clearly no record types for such names, the NSEC Type Bit Maps field in the response will only contain the NSEC and RRSIG types (and in the case of NSEC3, the Type Bit Maps field will be empty). Tools that need to accurately identify nonexistent names in responses cannot rely on this specific type bitmap because Empty Non-Terminal (ENT) names … will return exactly the same Type Bit Maps field.'

- **E7:**

  Immediately following: 'This specification defines the use of NXNAME (128), a synthetic RR type to signal the presence of a nonexistent name. … This RR type is added to the NSEC Type Bit Maps field for responses to nonexistent names… If NSEC3 is being used, this RR type is the sole entry in the Type Bit Maps field.'

**Evidence Summary:**

- (E6) Describes the pre-NXNAME behavior for responses, implying identical treatment for ENTs and nonexistent names.
- (E7) Introduces NXNAME to differentiate responses, conflicting with the earlier description.

**Fix Direction:**

Revise Section 2 to clearly separate historical behavior (pre-NXNAME) from the updated normative requirements that include NXNAME.


**Severity:** Low
  *Basis:* This ambiguity is primarily editorial and may mislead a reader who skims the section, though full normative details are provided later.

**Confidence:** High

---

## Report 4: 9824-2-4

**Label:** Inconsistent Scope: NXNAME Restricted to NSEC Type Bitmap vs. Use in NSEC3

**Bug Type:** Both

**Explanation:**

Section 3.5 restricts NXNAME to the NSEC type bitmap, while Section 4 explicitly requires its use in NSEC3 type bitmaps, resulting in conflicting guidance on where NXNAME is permitted to appear.

**Justification:**

- Section 3.5 states that NXNAME SHOULD NOT appear anywhere except in the NSEC type bitmap of a Compact Answer.
- Section 4 mandates that in NSEC3 responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE.

**Evidence Snippets:**

- **E8:**

  Section 3.5: 'NXNAME is a Meta-TYPE that SHOULD NOT appear anywhere in a DNS message apart from the NSEC type bitmap of a Compact Answer response for a nonexistent name.'

- **E9:**

  Section 4: 'In responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE. In responses to ENT names, the Type Bit Maps field will be empty.'

**Evidence Summary:**

- (E8) Limits the appearance of NXNAME to NSEC bitmaps.
- (E9) Specifies the required use of NXNAME in NSEC3 bitmaps, creating a scope discrepancy.

**Fix Direction:**

Modify Section 3.5 to refer to the 'Type Bit Maps field of an NSEC or NSEC3 record' rather than exclusively the NSEC type bitmap.


**Severity:** Medium
  *Basis:* The terminology discrepancy may lead to inconsistent implementations regarding where NXNAME should appear.

**Confidence:** High

---

## Report 5: 9824-2-5

**Label:** Incomplete Update to RFC 4034: Pseudo-type Bits Still Mandated to be Ignored

**Bug Type:** Inconsistency

**Explanation:**

RFC 9824 updates RFC 4034 to allow NXNAME to appear in responses for nonexistent names, but the updated text still includes language requiring that pseudo-type bits be ignored, which undermines the intended NXNAME semantics.

**Justification:**

- The updated RFC 4034 text in RFC 9824 provides an exception for NXNAME to appear, yet retains the mandate that pseudo-type bits must be cleared and ignored.
- This creates a normative tension where validators might still ignore the NXNAME signal, contrary to the purpose of its introduction.

**Evidence Snippets:**

- **E10:**

  RFC 9824 Update (Section 7.1): '... Bits representing pseudo-types MUST be clear... There is one exception to this rule for Compact Denial of Existence (RFC 9824), where the NXNAME pseudo-type is allowed to appear in responses to nonexistent names.'

- **E11:**

  RFC 4034 Original: 'Bits representing pseudo-types MUST be clear, as they do not appear in zone data. If encountered, they MUST be ignored upon being read.'

**Evidence Summary:**

- (E10) Shows the intended exception for NXNAME in RFC 9824.
- (E11) Confirms that the original text mandates that pseudo-type bits be ignored, a requirement not sufficiently overridden for NXNAME.

**Fix Direction:**

Explicitly exempt NXNAME from the 'must be ignored' requirement in the updated RFC 4034 text to ensure validators can interpret the NXNAME signal.


**Severity:** Medium
  *Basis:* The incomplete update may lead to validators disregarding the NXNAME signal, thereby nullifying its distinguishing purpose in responses.

**Confidence:** High

---
