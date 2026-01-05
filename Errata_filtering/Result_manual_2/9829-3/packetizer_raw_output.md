# Errata Reports

Total reports: 1

---


## Report 1: 9829-3-1

**Label:** Ambiguous Definition of 'Issuer’s Current Manifest' and CRL Selection

**Bug Type:** Underspecification/Inconsistency

**Explanation:**

The specification does not clearly define which manifest (and thus which CRL) is considered 'current' in cases such as after the nextUpdate time, during repository update races, or when multiple candidate manifests are present.

**Justification:**

- Temporal analysis (T1) questions whether an expired-but-cached manifest remains 'current' for validation.
- Scope and Causal analyses note that the term 'issuer’s current manifest' is introduced without a definitive RP‐side algorithm, leading to potential divergent interpretations.
- Boundary analysis highlights that when no unique current CRL (via manifest plus CRLDP) can be determined, it is ambiguous how the RP should proceed.

**Evidence Snippets:**

- **E1:**

  RFC 9829’s update to the path-validation rule (step 5) says that revocation status must be checked against “the issuer’s current CRL, as identified by the issuer’s current manifest and the CRLDP of the certificate,” and Section 2 adds that “a resource certificate cannot be validated without consulting the current manifest of the certificate’s issuer.”

- **E2:**

  Scope Analysis: The term “issuer’s current manifest” is introduced in RFC 9829’s update but is not explicitly defined, especially in cases with multiple manifests or repository inconsistencies.

- **E3:**

  Boundary Analysis: … when no unique 'issuer’s current CRL' can be determined via manifest + CRLDP, the updated step 5 depends on such a CRL existing, but does not say what the RP MUST or SHOULD do when it cannot identify one.

**Evidence Summary:**

- (E1) Describes the reliance on an ambiguously defined 'current' manifest for CRL selection.
- (E2) Points out that the specification lacks precise guidance for choosing the current manifest in edge cases.
- (E3) Highlights that ambiguous conditions (e.g., missing or multiple manifests) are not normatively resolved.

**Fix Direction:**

Add explicit normative text defining the RP algorithm for selecting the 'issuer’s current manifest' and associated CRL, including guidance for handling stale, missing, or ambiguous manifests (for example, by reference to RFC 9286’s RP behavior rules).

**Severity:** Low
  *Basis:* The ambiguity affects edge-case scenarios and may lead to inconsistent revocation decisions across implementations, though it is unlikely to cause direct security compromise.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1
- ScopeExpert: Issue-1
- CausalExpert: Issue 2
- BoundaryExpert: Finding-2

---