# Errata Reports

Total reports: 2

---

## Report 1: 9803-2-1

**Label:** XSD uniqueness on @for blocks multiple custom RR types in Policy Mode

**Bug Type:** Inconsistency

**Explanation:**

The XML Schema’s uniqueness constraint on the @for attribute restricts the inclusion of more than one <ttl:ttl> element with for="custom", which conflicts with the normative requirement to list all supported DNS record types (including multiple custom types) in Policy Mode.

**Justification:**

- The specification’s prose indicates that servers may support various custom DNS RR types via the custom attribute, yet the XSD enforces uniqueness based solely on the for attribute.
- Multiple expert analyses note that this technical inconsistency forces implementers to choose between schema validity and meeting the protocol’s requirement to advertise all supported types.

**Evidence Snippets:**

- **E1:**

  Section 1.2.1.2: “this document does not enumerate or restrict the DNS record types that can be included in the ‘custom’ attribute of the <ttl:ttl> element” and servers “MUST restrict the DNS record types that are accepted in <create> and <update> commands, and included in <info> responses, allowing only those types that are (a) registered in [IANA-RRTYPES] and (b) appropriate for use above a zone cut.”

- **E2:**

  Section 2.1.1.2: “the EPP response MUST contain <ttl:ttl> records for all supported DNS record types, irrespective of whether those record types are actually in use by the object in question.”

- **E3:**

  XSD in Section 8: <simpleType name="rrType"> enumerates "NS", "DS", "DNAME", "A", "AAAA", and "custom". Each of <element name="create">, <element name="update">, and <element name="infData"> has a <unique> constraint with <selector xpath="ttl:ttl"/> and <field xpath="@for"/>, enforcing uniqueness of @for within each container.

**Evidence Summary:**

- (E1) indicates that the document does not restrict the custom types, (E2) mandates that all supported types be reported, and (E3) shows that the schema enforces uniqueness solely on the @for attribute.

**Fix Direction:**

Modify the uniqueness constraint to include the @custom attribute for elements where for="custom", or revise the data model so that the @for attribute directly represents the DNS RR type, eliminating the need for a separate custom attribute.


**Severity:** High
  *Basis:* This inconsistency forces a choice between schema validity and fulfilling the protocol’s normative requirements, potentially breaking interoperability.

**Confidence:** High

---

## Report 2: 9803-2-2

**Label:** Undefined behavior for omitted policy attribute on <ttl:info>

**Bug Type:** Underspecification

**Explanation:**

The normative sections do not explicitly define how a server should behave when a <ttl:info> element is provided without a policy attribute, leading to potential divergent interpretations despite the XSD default.

**Justification:**

- Although the XSD and Section 1.2.1.3 specify a default value of 'false' for the policy attribute, the operative descriptions in Sections 2.1.1.1 and 2.1.1.2 only cover cases when the policy attribute is explicitly present.
- Experts warn that this gap may result in inconsistent behavior between implementations that apply schema defaulting and those that do not.

**Evidence Snippets:**

- **E1:**

  Section 1.2.1.3: “The <ttl:info> element is used by clients to request that the server include additional information in <info> responses for domain and host objects. It has a single OPTIONAL ‘policy’ attribute, which takes a boolean value with a default value of ‘false’.”

- **E2:**

  XSD (Section 8): <element name="info"><complexType><attribute name="policy" type="boolean" default="false"/></complexType></element>

- **E3:**

  Section 2.1.1.1: “If a server receives an <info> command for a domain or host object that includes a <ttl:info> element with a ‘policy’ attribute that is ‘0’ or ‘false’, then the EPP response MUST contain <ttl:ttl> records …”

**Evidence Summary:**

- (E1) describes the optional policy attribute with a default value, (E2) shows the XSD declaration of this default, and (E3) specifies behavior only for explicitly set values.

**Fix Direction:**

Insert an explicit normative statement that an omitted policy attribute must be treated as if its value were 'false' (i.e., Default Mode applies).


**Severity:** Medium
  *Basis:* While the gap does not render the extension unimplementable, it may cause inconsistent server behavior and interoperability issues among implementations that do or do not apply XML Schema defaulting.

**Confidence:** High

---


