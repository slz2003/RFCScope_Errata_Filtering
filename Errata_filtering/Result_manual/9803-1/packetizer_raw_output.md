# Errata Reports

Total reports: 4

---

## Report 1: 9803-1-1

**Label:** XML Schema uniqueness constraint on '@for' prevents multiple custom RR types in one container

**Bug Type:** Inconsistency

**Explanation:**

The XML Schema’s uniqueness constraint on the '@for' attribute forces all <ttl:ttl> elements to have unique values, which prevents representing multiple custom DNS record types (e.g., CAA and MX) as intended by the prose.

**Justification:**

- The normative text requires a Policy Mode response to include TTL records for all supported DNS record types – including multiple custom entries – but the schema only allows one element with for='custom' per container.
- This conflict forces implementations to choose between supporting multiple custom RR types and remaining schema‐valid.

**Evidence Snippets:**

- **E1:**

  For each <ttl:create>, <ttl:update>, and <ttl:infData>, XML Schema enforces that all child <ttl:ttl> elements have a unique @for value within that container:
  - <unique name="uniqueRRTypeForCreate"><selector xpath="ttl:ttl"/><field xpath="@for"/></unique>
  - Same pattern for update and infData.

- **E2:**

  Since @for is an enumeration that includes 'custom' as a single value, this uniqueness rule allows at most one <ttl:ttl for="custom" ...> element per container, which prevents including multiple custom RR types as required by Policy Mode.

**Evidence Summary:**

- (E1) The schema defines a uniqueness constraint on the @for attribute for all <ttl:ttl> elements within a container.
- (E2) Because 'custom' is a single enumerated value, multiple custom RR type entries cannot coexist in a single command or response.

**Fix Direction:**

Change the uniqueness constraint so that for entries with for="custom" the constraint is applied to the pair (@for, @custom) rather than to @for alone, while keeping uniqueness on @for for built‐in types.


**Severity:** High
  *Basis:* It directly prevents encoding the intended support for multiple custom DNS record types, causing serious interoperability issues.

**Confidence:** High

---

## Report 2: 9803-1-2

**Label:** Custom attribute regex mismatch: RFC 6895 versus XML Schema

**Bug Type:** Inconsistency

**Explanation:**

The normative text requires that the 'custom' attribute conform to the RFC 6895 regular expression (which excludes single‐letter mnemonics), while the XML Schema explicitly permits the value 'A', resulting in a mismatch.

**Justification:**

- The prose states that when for="custom" the value MUST conform to the RFC 6895 regex ([A-Z][A-Z0-9\-]*[A-Z0-9]), which does not allow single-letter mnemonics.
- In contrast, the XML Schema defines the type ttl:customRRType with the pattern 'A|[A-Z][A-Z0-9\-]*[A-Z0-9]', thereby allowing 'A' and creating a divergence between textual and schema validation.

**Evidence Snippets:**

- **E1:**

  If the value of the 'for' attribute is 'custom', then the <ttl:ttl> element MUST also have a 'custom' attribute containing a DNS record type conforming with the regular expression in Section 3.1 of [RFC6895].

- **E2:**

  <simpleType name="customRRType">
  <restriction base="token">
    <pattern value="A|[A-Z][A-Z0-9\-]*[A-Z0-9]"/>
  </restriction>
</simpleType>

**Evidence Summary:**

- (E1) Normatively, the custom attribute is required to conform to the RFC 6895 regex, which would disallow a single-letter value like 'A'.
- (E2) The XML Schema, however, explicitly adds an 'A' alternative to the pattern, extending the allowed set.

**Fix Direction:**

Either remove the 'A' special case from the XML Schema to enforce the RFC 6895 regex, or modify the normative text to state that the allowed pattern is that defined by the schema.


**Severity:** Medium
  *Basis:* This divergence might lead to varying validation outcomes between implementations that rely on textual interpretation versus schema validation.

**Confidence:** High

---

## Report 3: 9803-1-3

**Label:** Underspecified behavior for omitted 'policy' attribute in <ttl:info>

**Bug Type:** Underspecification

**Explanation:**

The specification relies on XML Schema defaulting to treat a missing policy attribute as 'false' but does not explicitly mandate this behavior, leaving ambiguity in how Default Mode should be determined.

**Justification:**

- The text mentions that <ttl:info> has an optional 'policy' attribute with a default of 'false', yet it does not explicitly require implementations to treat its absence as equivalent to 'false'.
- This reliance on XML Schema defaulting, which many EPP implementations do not apply at runtime, creates a potential for inconsistent behavior.

**Evidence Snippets:**

- **E1:**

  The <ttl:info> element … has a single OPTIONAL ‘policy’ attribute, which takes a boolean value with a default value of ‘false’.

- **E2:**

  If a server receives an <info> command … that includes a <ttl:info> element with a ‘policy’ attribute that is ‘0’ or ‘false’, then … Default Mode…

**Evidence Summary:**

- (E1) Indicates that the policy attribute has a default value of 'false'.
- (E2) Describes behavior for when the policy attribute is explicitly set to 'false', but does not clarify what should happen if it is omitted.

**Fix Direction:**

Add explicit language stating that if the 'policy' attribute is absent, it MUST be interpreted as 'false'.


**Severity:** Medium
  *Basis:* Ambiguity in default behavior can lead to interoperability issues if different implementations handle the absence of the attribute inconsistently.

**Confidence:** High

---

## Report 4: 9803-1-4

**Label:** Underspecified handling of 'custom' attribute when 'for' is not 'custom'

**Bug Type:** Underspecification

**Explanation:**

The specification does not clarify how to interpret a <ttl:ttl> element that includes a 'custom' attribute when the 'for' attribute does not equal 'custom'.

**Justification:**

- While the normative rules describe the use of the 'custom' attribute only when for="custom", there is no guidance on behavior if a 'custom' attribute is provided with any other value.
- This lack of clarification might result in inconsistent processing across different implementations.

**Evidence Snippets:**

- **E1:**

  the document does not specify how to handle a <ttl:ttl> where for != "custom" but a custom attribute is present; the most likely intended behavior is to ignore custom in that case.

**Evidence Summary:**

- (E1) Points out the absence of guidance for handling a 'custom' attribute when the 'for' attribute is not set to 'custom'.

**Fix Direction:**

Include a clarifying statement indicating that any 'custom' attribute should be ignored if the 'for' attribute is not 'custom'.


**Severity:** Low
  *Basis:* Although this ambiguity is less likely to cause major issues, it may still lead to minor interoperability differences.

**Confidence:** High

---
