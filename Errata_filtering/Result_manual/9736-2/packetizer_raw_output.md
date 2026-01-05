# Errata Reports

Total reports: 1

---


## Report 1: 9736-2-1

**Label:** Ambiguous Scope and Field Naming for String TLVs in RFC 9736 Section 2

**Bug Type:** Underspecification

**Explanation:**

RFC 9736 Section 2 uses generic terminology ("TLV's Length field") to define string TLVs without explicitly mapping it to the concrete "Information Length" field used in BMP TLV diagrams, creating ambiguity about which TLVs are covered and which length field is referenced.

**Justification:**

- Section 2 defines a string TLV using generic language, while the concrete TLV layouts in Sections 3.1, 3.3, and RFC 7854 consistently use 'Information Length' as the length indicator (E1, E4).
- The lack of an explicit statement mapping 'TLV’s Length field' to 'Information Length' leads to uncertainty whether the definition applies universally to all string TLVs or only to the Initiation and Peer Up TLVs that explicitly reference Section 2 (E2, E3, E5).

**Evidence Snippets:**

- **E1:**

  RFC 9736 Section 2: “A string TLV is a free-form sequence of UTF-8 characters whose length in bytes is given by the TLV's Length field. There is no requirement to terminate the string with a null (or any other particular) character -- the Length field gives its termination.”

- **E2:**

  RFC 9736 Section 3.1 (Initiation Information TLV, Type = 0): “The Information field contains a string (Section 2).”

- **E3:**

  RFC 9736 Section 3.3 (Peer Up Information TLV, Type = 0): “The Information field contains a string (Section 2).”

- **E4:**

  All the relevant TLV layouts in RFC 9736 and RFC 7854 (Information TLV in 4.4, Termination TLVs in 4.5, Peer Up Information TLV in 3.3 of 9736) use a field explicitly called “Information Length (2 bytes): The length of the following Information field, in bytes.”

- **E5:**

  RFC 9736, Section 3.3, Type 4: “The Information field contains a free-form UTF-8 string whose byte length is given by the Information Length field. The value is administratively assigned. There is no requirement to terminate the string a with null or any other character.”

**Evidence Summary:**

- (E1) Section 2 defines the string TLV using generic language about a TLV's Length field.
- (E2) The Initiation Information TLV points to Section 2 for its string content.
- (E3) The Peer Up Information TLV similarly uses Section 2 for its string definition.
- (E4) BMP TLV diagrams uniformly define the field as 'Information Length' with explicit details.
- (E5) Additional TLV definitions (e.g., Type 4) reinforce the use of 'Information Length', highlighting the naming discrepancy.

**Fix Direction:**

Revise Section 2 to explicitly state that the 'TLV's Length field' is the 'Information Length' field as used in BMP TLV diagrams and clarify which string TLVs are covered by the definition.


**Severity:** Low
  *Basis:* The ambiguity is primarily editorial and is unlikely to impact interoperability, though it may lead to misinterpretation of the specification's scope.

**Confidence:** High

---