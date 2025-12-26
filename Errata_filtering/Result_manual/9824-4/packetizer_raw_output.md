# Errata Reports

Total reports: 4

---

## Report 1: 9824-4-1

**Label:** NXNAME Meta‑TYPE bit in NSEC3 Type Bit Maps conflicts with RFC 5155 'MUST be 0' rule

**Bug Type:** Inconsistency

**Explanation:**

RFC 9824 mandates that for responses to nonexistent names the NXNAME bit must be set in the NSEC3 Type Bit Maps, while RFC 5155 requires that all Meta‑TYPE bits be cleared, creating a normative conflict.

**Justification:**

- RFC 9824 §4 states that 'In responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE. In responses to ENT names, the Type Bit Maps field will be empty.' (Evidence: E1)
- RFC 5155 §3.2.1 requires that 'Bits representing Meta-TYPEs or QTYPEs ... MUST be set to 0 ...' (Evidence: E2)
- RFC 9824 §7.1 explicitly updates RFC 4034 to carve out an exception for NXNAME in NSEC records but does not update RFC 5155, leaving a conflicting requirement. (Evidence: E3)

**Evidence Snippets:**

- **E1:**

  RFC 9824 §4 (NSEC3 mode) states: “In responses for nonexistent names, the Type Bit Maps field will contain only the NXNAME Meta-TYPE. In responses to ENT names, the Type Bit Maps field will be empty.”

- **E2:**

  RFC 5155 §3.2.1 (for NSEC3 Type Bit Maps) says: “Bits representing Meta-TYPEs or QTYPEs as specified in Section 3.1 of [RFC2929] or within the range reserved for assignment only to QTYPEs and Meta-TYPEs MUST be set to 0, since they do not appear in zone data. If encountered, they must be ignored upon reading.”

- **E3:**

  RFC 9824 §7.1 explicitly updates RFC 4034 §4.1.2 to allow the NXNAME pseudo-type in responses for nonexistent names, but makes no corresponding update to RFC 5155.

**Evidence Summary:**

- (E1) NXNAME is mandated in the Type Bit Maps for nonexistent names according to RFC 9824 §4.
- (E2) RFC 5155 §3.2.1 states that all Meta‑TYPE bits must be 0.
- (E3) The update in RFC 9824 does not extend to RFC 5155, causing the conflict.

**Fix Direction:**

Update RFC 5155 §3.2.1 to explicitly permit the NXNAME Meta‑TYPE in NSEC3 Type Bit Maps for Compact Denial of Existence, similar to the exception provided for NSEC in RFC 9824 §7.1.


---

## Report 2: 9824-4-2

**Label:** Next Hashed Owner Name 'hash+1' diverges from RFC 5155 chain semantics

**Bug Type:** Inconsistency

**Explanation:**

RFC 9824 specifies that the Next Hashed Owner Name is computed by simply adding one to the binary hash value, which conflicts with RFC 5155’s requirement that it be the hash of the next actual owner in the zone, resulting in a conceptual mismatch.

**Justification:**

- RFC 9824 §4 defines the Next Hashed Owner Name as 'the immediate name successor of the unencoded binary form of the previous hash, which can be computed by adding one to the binary hash value.' (Evidence: E1)
- RFC 5155 §3.1.7 requires that the Next Hashed Owner Name be the hash of the owner name that immediately follows in the zone’s ordered set. (Evidence: E2)
- This approach may yield a Next field that does not correspond to any actual zone owner, undermining the expected NSEC3 chain semantics. (Evidence: E3)

**Evidence Snippets:**

- **E1:**

  RFC 9824 Section 4: “The Next Hashed Owner Name is the immediate name successor of the unencoded binary form of the previous hash, which can be computed by adding one to the binary hash value.”

- **E2:**

  RFC 5155 Section 3.1.7: “The Next Hashed Owner Name field contains the next hashed owner name in hash order. Given the ordered set of all hashed owner names, the Next Hashed Owner Name field contains the hash of an owner name that immediately follows the owner name of the given NSEC3 RR.”

- **E3:**

  The RFC 9824 example shows a Next value computed as a numeric increment (e.g., …JQ → …JR), which may not match any actual hashed owner name in the zone.

**Evidence Summary:**

- (E1) RFC 9824 defines Next Hashed Owner Name as computed by hash+1.
- (E2) RFC 5155 requires the Next field to be an actual successor from the zone’s ordered hashed owner names.
- (E3) The numerical increment may not correspond to any real owner, illustrating the inconsistency.

**Fix Direction:**

Clarify in RFC 9824 that for Compact Denial of Existence the Next Hashed Owner Name may be computed as hash+1 (using modular arithmetic) and that it is exempt from forming a complete chain as defined in RFC 5155.


---

## Report 3: 9824-4-3

**Label:** Undefined wrap-around behavior for Next Hashed Owner Name 'hash+1' computation

**Bug Type:** Underspecification

**Explanation:**

RFC 9824 instructs that the Next Hashed Owner Name is computed by adding one to the binary hash value without specifying how to handle an arithmetic overflow when the hash value is maximal.

**Justification:**

- RFC 9824 §4 directs that Next Hashed Owner Name is computed by 'adding one to the binary hash value,' but does not define behavior at the maximum hash value. (Evidence: E1)
- RFC 5155’s description of a complete NSEC3 chain implies a wrap-around where the last record’s Next field equals the first record’s hash, suggesting that modular arithmetic should apply. (Evidence: E2)

**Evidence Snippets:**

- **E1:**

  RFC 9824 §4: “The Next Hashed Owner Name is the immediate name successor of the unencoded binary form of the previous hash, which can be computed by adding one to the binary hash value.”

- **E2:**

  RFC 5155 §3.1.7: “The value of the Next Hashed Owner Name field in the last NSEC3 RR in the zone is the same as the hashed owner name of the first NSEC3 RR in the zone in hash order,” implying wrap-around behavior.

**Evidence Summary:**

- (E1) 'hash+1' is specified without clarification of overflow handling.
- (E2) RFC 5155 implies a wrap-around in the NSEC3 chain that is not defined for the hash+1 computation in RFC 9824.

**Fix Direction:**

Define the hash+1 operation as modular arithmetic (mod 2^N, where N is the hash length in bits) to ensure a well-defined wrap-around behavior.


---

## Report 4: 9824-4-4

**Label:** Inconsistent use of 'pseudo-type' and 'Meta-TYPE' terminology for NXNAME

**Bug Type:** Inconsistency

**Explanation:**

RFC 9824 alternately uses the terms 'pseudo-type' and 'Meta-TYPE' when referring to NXNAME, which may lead to minor interpretative inconsistencies.

**Justification:**

- RFC 9824 Section 2 defines NXNAME as a Meta-TYPE, aligning with RFC 6895. (Evidence: E1)
- RFC 9824 §7.1 later refers to NXNAME as a pseudo-type when describing the exception for Compact Denial of Existence. (Evidence: E2)
- The mixed terminology could cause slight confusion among implementers despite not affecting functionality.

**Evidence Snippets:**

- **E1:**

  RFC 9824 Section 2: “This specification defines the use of NXNAME (128), a synthetic RR type to signal the presence of a nonexistent name. It is a ‘Meta-TYPE’, as defined in [RFC6895]…”

- **E2:**

  RFC 9824 §7.1: “… There is one exception to this rule for Compact Denial of Existence (RFC 9824), where the NXNAME pseudo-type is allowed to appear in responses to nonexistent names.”

**Evidence Summary:**

- (E1) NXNAME is defined as a Meta‑TYPE in Section 2.
- (E2) NXNAME is later referred to as a pseudo-type in Section 7.1.


---
