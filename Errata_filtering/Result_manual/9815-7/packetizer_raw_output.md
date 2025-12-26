# Errata Reports

Total reports: 3

---

## Report 1: 9815-7-1

**Label:** Ambiguous use of 'malformed' for semantic errors in BGP‑LS‑SPF versus RFC 9552

**Bug Type:** Both

**Explanation:**

RFC 9815 mandates treating certain semantic violations (e.g., missing IGP Metric TLV, invalid SPF Status values, missing Sequence Number TLV) as 'malformed' and subjecting them to treat‑as‑withdraw, which conflicts with RFC 9552 where 'malformed' is restricted to syntactic errors.

**Justification:**

- Multiple sections note that RFC 9552 forbids calling semantic issues 'malformed' while RFC 9815 applies that label to semantic failures for SAFI 80.
- This inconsistency creates confusion regarding whether errors should trigger transport-level error handling or only SPF computation rejection.

**Evidence Snippets:**

- **E1:**

  RFC 9552, Fault Management: “A Link-State NLRI MUST NOT be considered malformed or invalid based on the inclusion/exclusion of TLVs or contents of the TLV fields (i.e., semantic errors)… A BGP‑LS Attribute MUST NOT be considered malformed or invalid based on the inclusion/exclusion of TLVs or contents of the TLV fields (i.e., semantic errors)…

- **E2:**

  RFC 9815, Link NLRI usage: “The IGP Metric (TLV 1095) MUST be advertised. If a BGP speaker receives a Link NLRI without an IGP Metric attribute TLV, then it MUST consider the received NLRI as malformed and is handled as described in Section 7.1.”

**Evidence Summary:**

- (E1) Indicates that RFC 9552 restricts the term 'malformed' to syntactic issues.
- (E2) Demonstrates that RFC 9815 applies 'malformed' to semantic violations like a missing IGP Metric TLV.

**Fix Direction:**

Clarify that the 'malformed' designation for SAFI 80 applies only to consumer-side SPF computation (or use distinct terminology for semantic failures) so as not to conflict with RFC 9552's restrictions on semantic validation in BGP‑LS propagation.


**Severity:** High
  *Basis:* This ambiguity may lead to inconsistent error handling and divergent LSDB states across implementations, adversely affecting SPF computation and overall interoperability.

**Confidence:** High

---

## Report 2: 9815-7-2

**Label:** Address Family Link Descriptor TLV mislocated in BGP‑LS Attribute instead of Link NLRI descriptor

**Bug Type:** Inconsistency

**Explanation:**

Section 5.2.2.1 defines the Address Family Link Descriptor TLV (Type 1185) as a Link Descriptor TLV within the NLRI for unnumbered links, but Section 7.1 incorrectly refers to it as residing in the BGP‑LS Attribute.

**Justification:**

- The normative definition in Section 5.2.2.1 places TLV 1185 with the Link Local/Remote Identifiers in the NLRI.
- Section 7.1 mistakenly locates the TLV in the BGP‑LS Attribute, which conflicts with its intended placement and can mislead implementers.

**Evidence Snippets:**

- **E1:**

  Section 5.2.2.1 defines “BGP-LS Link NLRI Address Family Link Descriptor TLV” (Type 1185) and says it “SHOULD be included with the Link Local/Remote Identifiers TLV for unnumbered links” so the link can be used in SPF.

- **E2:**

  Section 7.1: “When a BGP speaker receives a BGP Update containing a malformed Address Family Link Descriptor TLV in the BGP-LS Attribute [RFC9552], the corresponding Link NLRI is considered malformed and MUST be handled as 'treat‑as‑withdraw'.”

**Evidence Summary:**

- (E1) Establishes that TLV 1185 belongs in the NLRI’s Link Descriptor field.
- (E2) Shows the mislocation in Section 7.1 referencing the TLV as part of the BGP‑LS Attribute.

**Fix Direction:**

Amend Section 7.1 to indicate that the Address Family Link Descriptor TLV is located in the Link NLRI’s descriptor set rather than in the BGP‑LS Attribute.


**Severity:** Low
  *Basis:* Although this misreference is unlikely to affect essential SPF functionality, it creates an underspecification that may lead to implementation confusion.

**Confidence:** High

---

## Report 3: 9815-7-3

**Label:** Incorrect reference for 'Loss of LSDB Synchronization' error code in Section 7.4

**Bug Type:** Inconsistency

**Explanation:**

Section 7.4 improperly directs implementers to RFC 4271 Section 3 for the definition of the 'Loss of LSDB Synchronization' error code, whereas this error code is newly defined and allocated (value 9) in RFC 9815.

**Justification:**

- The text in Section 7.4 references RFC 4271 Section 3, which does not define any BGP error codes or a 'Loss of LSDB Synchronization' error.
- RFC 9815 Section 8.6 and the IANA registry clearly assign value 9 to this error code.

**Evidence Snippets:**

- **E1:**

  Section 7.4: “When the session is reset, the BGP speaker MUST send a NOTIFICATION message with the BGP error code 'Loss of LSDB Synchronization' as described in Section 3 of [RFC4271].”

- **E2:**

  Section 8.6: “IANA has assigned value 9 for Loss of LSDB Synchronization in the ‘BGP Error (Notification) Codes’ registry within the ‘Border Gateway Protocol (BGP) Parameters’ registry group.”

**Evidence Summary:**

- (E1) Reveals the incorrect normative reference to RFC 4271 Section 3.
- (E2) Confirms that the error code is defined in RFC 9815 with an assigned value of 9.

**Fix Direction:**

Correct the normative reference in Section 7.4 to point to RFC 9815 Section 8.6 (or the relevant IANA registry) instead of RFC 4271 Section 3.


**Severity:** Low
  *Basis:* While the misreference is primarily a documentation issue, it can cause confusion during implementation if left uncorrected.

**Confidence:** High

---
