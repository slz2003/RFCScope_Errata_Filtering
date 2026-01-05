# Errata Reports

Total reports: 1

---


## Report 2: 9905-4-2

**Label:** Ambiguous Actor Role: Undefined 'Operators' in Section 4

**Bug Type:** Underspecification

**Explanation:**

Section 4 employs the term 'Operators' in an ambiguous manner, conflating roles and failing to clearly distinguish which actor is responsible for maintaining SHA‑1 support.

**Justification:**

- The document references 'Operators' without clarifying whether it applies to validating resolver operators, OS/package maintainers, or other parties.
- This ambiguity may lead to inconsistent practices regarding the rebuilding and deployment of software to continue support for SHA‑1-based algorithms.

**Evidence Snippets:**

- **E1:**

  Operators should take care when deploying software packages and operating systems that may have already removed support for the SHA-1 algorithm. In these situations, software may need to be manually built and deployed by an operator to continue supporting the required levels indicated by the 'Use for DNSSEC Validation' and 'Implement for DNSSEC Validation' columns...

- **E2:**

  Operators of validating resolvers MUST treat RSASHA1 and RSASHA1‑NSEC3‑SHA1 DS records as insecure.

**Evidence Summary:**

- (E1) Indicates that the advisory in Section 4 is ambiguous about which 'operator' is responsible for manual software deployment.
- (E2) Reinforces the treatment mandate for validating resolver operators but does not resolve the conflation of different operational roles.

**Fix Direction:**

Clarify the definition of 'Operators' in Section 4 by explicitly distinguishing between validating resolver operators and other parties (such as OS or package maintainers) responsible for ensuring continued support for SHA‑1.

**Severity:** Medium
  *Basis:* A lack of clear role definition may lead to inconsistent implementation and operational practices, possibly impacting the intended security posture of DNSSEC deployments.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: NewIssue-1

---