# Errata Reports

Total reports: 1

---


## Report 2: 9803-8-2

**Label:** CustomRRType Pattern Mismatch with RFC 6895 Regular Expression

**Bug Type:** Inconsistency

**Explanation:**

The XML Schema’s definition of customRRType uses the pattern 'A|[A-Z][A-Z0-9\-]*[A-Z0-9]', which diverges from the strict RFC 6895 regex by explicitly allowing the single‐letter mnemonic 'A' and by not excluding names matching patterns like '(TYPE|CLASS)[0-9]*'.

**Justification:**

- The normative prose (Section 1.2.1) mandates that when the 'for' attribute is 'custom', the accompanying 'custom' attribute must contain a DNS record type conforming with the regex defined in RFC 6895 (i.e. [A-Z][A-Z0-9\-]*[A-Z0-9] with additional exclusions) (E1).
- However, the XML Schema defines customRRType with the pattern 'A|[A-Z][A-Z0-9\-]*[A-Z0-9]', which both permits 'A' as a valid custom value and omits the exclusion of mnemonics such as 'TYPE123', leading to a mismatch between prose and schema (E2).

**Evidence Snippets:**

- **E1:**

  Section 1.2.1: "If the value of the 'for' attribute is 'custom', then the <ttl:ttl> element MUST also have a 'custom' attribute containing a DNS record type conforming with the regular expression in Section 3.1 of [RFC6895]."

- **E2:**

  <simpleType name="customRRType">
  <restriction base="token">
    <pattern value="A|[A-Z][A-Z0-9\-]*[A-Z0-9]"/>
  </restriction>
</simpleType>

**Evidence Summary:**

- (E1) Normative text requires a DNS record type matching the RFC 6895 regex for custom types.
- (E2) The schema pattern 'A|[A-Z][A-Z0-9\-]*[A-Z0-9]' allows 'A' and does not enforce the exclusion of values like 'TYPE123'.

**Fix Direction:**

Either update the XML Schema to enforce a pattern that fully matches the RFC 6895 requirements (including the exclusion of patterns like '(TYPE|CLASS)[0-9]*') or modify the normative text to clarify that the schema pattern is the authoritative constraint.


**Severity:** Low
  *Basis:* The discrepancy affects lexical validation of the custom RRTYPE mnemonics; although it might lead to minor interoperability inconsistencies, the IANA registration requirement mitigates the risk.

**Confidence:** High

---