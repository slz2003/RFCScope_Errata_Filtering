# Errata Reports

Total reports: 2

---

## Report 1: 9870-2-1

**Label:** Misreferenced RFC9868 'datagram' Terminology in RFC9870

**Bug Type:** Inconsistency

**Explanation:**

RFC9870 inaccurately claims that a standalone 'datagram' term is defined in RFC9868 Section 3, whereas RFC9868 only defines specific terms like 'IP datagram' and 'User datagram', creating potential ambiguity.

**Justification:**

- RFC 9870 Section 2 states that the document uses the terms defined in Section 3 of [RFC9868], especially 'datagram' and 'surplus area', yet the referenced section does not define a standalone 'datagram' (CrossRFC Expert Issue-1).
- The Terminology Expert analysis confirms that only compound terms ('IP datagram' and 'User datagram') are defined in RFC9868, not a single term 'datagram' (Terminology Expert Issue-1).

**Evidence Snippets:**

- **E1:**

  RFC 9870, Section 2:  Also, this document uses the terms defined in Section 3 of [RFC9868], especially "datagram" and "surplus area".

- **E2:**

  In the provided Section 3 of RFC 9868, there is **no standalone defined term** labeled simply “datagram”; only “IP datagram” and “User datagram” (and related “UDP packet”) are explicitly defined.

- **E3:**

  To fill that void, [RFC9868] extends UDP with a mechanism to insert extensions in datagrams. The document focuses exclusively on exporting observed UDP Options in datagrams.

**Evidence Summary:**

- (E1) RFC9870 Section 2 claims to use 'datagram' as defined in RFC9868.
- (E2) RFC9868 Section 3 does not include a standalone definition for 'datagram' but instead defines 'IP datagram' and 'User datagram'.
- (E3) The usage of the term 'datagrams' implies ambiguity in the intended reference.

**Fix Direction:**

Revise RFC9870 Section 2 to reference the precise terms 'IP datagram' and/or 'User datagram' as defined in RFC9868.


**Severity:** Low
  *Basis:* The inconsistency is minor and mostly editorial in nature, with little risk of causing implementation issues.

**Confidence:** High

---

## Report 2: 9870-2-2

**Label:** Inconsistent IE Naming: 'udpExID' Deviates from RFC7012 Lowercase Acronym Guidance

**Bug Type:** Inconsistency

**Explanation:**

While RFC9870 claims adherence to RFC7012 naming conventions, the IE name 'udpExID' uses an uppercase acronym, deviating from the guideline that all letters after the first should be lowercase, making it a stylistic inconsistency.

**Justification:**

- RFC9870 Section 2 asserts that the document adheres to the naming conventions for Information Elements per RFC7012, yet the IANA table defines the IE 'udpExID' in a form inconsistent with the lowercase acronym guideline (Terminology Expert Issue-2).
- RFC7012 Section 2.3 specifies that except for the initial letter, all letters should be lowercase, which 'udpExID' does not follow.

**Evidence Snippets:**

- **E1:**

  RFC 9870 Section 2:  The document adheres to the naming conventions for Information Elements per Section 2.3 of [RFC7012].

- **E2:**

  Defined IE names in RFC 9870 / IANA table: udpSafeOptions, udpUnsafeOptions, udpExID, udpSafeExIDList, udpUnsafeExIDList.

- **E3:**

  RFC 7012 Section 2.3:  Names of Information Elements MUST start with lowercase letters. Composed names MUST use capital letters for the first letter of each component (except for the first one). All other letters are lowercase, even for acronyms. ... Examples are "sourceMacAddress" and "destinationIPv4Address".

**Evidence Summary:**

- (E1) RFC9870 claims adherence to RFC7012 IE naming conventions.
- (E2) The IANA table in RFC9870 includes 'udpExID', which does not conform to the expected lowercase style.
- (E3) RFC7012 mandates that all letters after the first of each component should be lowercase, highlighting the deviation.


**Severity:** Low
  *Basis:* The stylistic deviation is minor, does not impact functionality, and is primarily an editorial issue.

**Confidence:** High
---
