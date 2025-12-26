# Errata Reports

Total reports: 2

---

## Report 1: 9803-8-1

**Label:** Uniqueness Constraint on @for Prevents Multiple Custom RRTYPE TTLs

**Bug Type:** Inconsistency

**Explanation:**

The XML Schema enforces uniqueness solely on the @for attribute, which prevents including more than one TTL entry for distinct custom DNS record types, even though the prose expects that multiple custom types can be represented.

**Justification:**

- The schema defines a uniqueness constraint on <ttl:ttl> elements based only on the @for attribute (e.g., in the <ttl:create>, <ttl:update>, and <ttl:infData> containers), which means that even if the actual DNS record type is provided via the 'custom' attribute, only one element per @for value ('custom') is allowed (E1).
- Conversely, the normative text requires that in Policy Mode the server MUST include TTL entries for all supported DNS record types, including multiple custom types defined by different 'custom' attribute values (E2).

**Evidence Snippets:**

- **E1:**

  <element name="create" type="ttl:commandContainer">
  <unique name="uniqueRRTypeForCreate">
    <selector xpath="ttl:ttl"/>
    <field xpath="@for"/>
  </unique>
</element>

<element name="update" type="ttl:commandContainer">
  <unique name="uniqueRRTypeForUpdate">
    <selector xpath="ttl:ttl"/>
    <field xpath="@for"/>
  </unique>
</element>

<element name="infData" type="ttl:responseContainer">
  <unique name="uniqueRRTypeForInfo">
    <selector xpath="ttl:ttl"/>
    <field xpath="@for"/>
  </unique>
</element>

- **E2:**

  Section 1.2.1 and 1.2.1.2 of the prose state that when using a custom DNS record type, the <ttl:ttl> element MUST include a 'custom' attribute containing a DNS record type that conforms to future extensibility. In Policy Mode (Section 2.1.1.2), the response MUST contain <ttl:ttl> records for all supported DNS record types, implying that multiple custom types should be allowed.

**Evidence Summary:**

- (E1) The schema uniquely keys <ttl:ttl> elements on the @for attribute only.
- (E2) The normative text requires representing TTLs for all supported DNS record types, including several custom types.

**Fix Direction:**

Modify the XML Schema uniqueness constraints so that for built-in types uniqueness is enforced on @for, while for custom types uniqueness is enforced on the combination of @for and @custom (or by separating the constraints), thereby allowing multiple custom TTL entries.


**Severity:** High
  *Basis:* This issue directly conflicts with the forward‐compatibility design, as it prevents a server from representing TTLs for more than one custom DNS record type per object, potentially leading to schema-validation failures and interoperability problems.

**Confidence:** High

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
