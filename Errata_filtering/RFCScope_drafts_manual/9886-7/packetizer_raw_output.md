# Errata Reports

Total reports: 1

---


## Report 2: 9886-7-2

**Label:** Ambiguous DNSSEC requirements and fallback certificate-walking procedures

**Bug Type:** Underspecification

**Explanation:**

The document ambiguously specifies that apex/self‑signed entities MUST use DNSSEC while also allowing for exceptions and fallback certificate-walking, leaving unclear which deployments can forgo DNSSEC.

**Justification:**

- Section 4 mandates that DNSSEC MUST be used for apex entities and is recommended for others.
- Section 7.1 introduces guidance with 'except when national regulations prevent it' and provides a fallback mechanism (certificate walking) for non-DNSSEC environments.
- This creates uncertainty about whether regulatory restrictions can override the DNSSEC MUST or if apex entities must always be deployed with DNSSEC.

**Evidence Snippets:**

- **E4:**

  DNSSEC MUST be used for apex entities (those which use a self-signed Canonical Registration Certificate) and is RECOMMENDED for other entities. When a DIME decides to use DNSSEC, they SHOULD define a framework for cryptographic algorithms and key management [RFC6841].

- **E5:**

  These components … thus operation SHOULD follow best common practices, specifically in security (such as running DNSSEC) as appropriate except when national regulations prevent it. … A self-signed entity (i.e., an entity that self-signed its certificate as part of the HHIT RRType) MUST use DNSSEC.

**Evidence Summary:**

- (E4) Establishes a strict DNSSEC requirement for apex entities along with recommendations for others.
- (E5) Introduces a regulatory exception and fallback to certificate walking, creating ambiguity in the application of DNSSEC.

**Fix Direction:**

Clarify the intended scope and priority rules: delineate precisely when DNSSEC is mandatory and when fallback certificate-walking is acceptable, especially in the context of regulatory restrictions.

**Severity:** Medium
  *Basis:* Ambiguous instructions may lead to divergent deployments and weaken security assurances if apex entities operate without DNSSEC.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2
- Deontic: Issue-2

---