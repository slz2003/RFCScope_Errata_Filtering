# Errata Reports

Total reports: 4

---

## Report 1: 9886-7-1

**Label:** Conflict between mandatory HHIT/BRID presence and 'just-in-time' non-publication recommendation

**Bug Type:** Inconsistency

**Explanation:**

The specification mandates that DET reverse names always resolve to an HHIT RRType (and BRID for UAS RID) while also recommending that no RRTypes be published until needed, creating a temporal and operational conflict.

**Justification:**

- Section 4 requires that DET reverse names MUST resolve to an HHIT RRType and, for UAS RID, the BRID RRType MUST be present.
- Section 7.2 recommends withholding publication of any RRTypes under a DET's domain until required, which would result in the absence of these mandatory records.
- Section 7.1 relies on the continuous availability of HHIT records for certificate walking when DNSSEC is not in use.

**Evidence Snippets:**

- **E1:**

  DETs, being IPv6 addresses, are to be under ip6.arpa. (nibble reversed per Section 2.5 of RFC 3596 [STD88]) and MUST resolve to an HHIT RRType. … For UAS RID, the BRID RRType MUST be present to provide the Broadcast Endorsements (BEs) defined in Section 3.1.2.1 of [RFC9575].

- **E2:**

  When practical, it is RECOMMENDED that no RRTypes under a DET's specific domain name be published unless and until it is required for use by other parties. Such action would cause at least the HHIT RRType to not be in the DNS, protecting the public key in the certificate from being exposed before its needed.

- **E3:**

  When DNSSEC is not in use, … clients MUST ‘walk’ the tree of certificates locating each certificate by performing DNS lookups of HHIT RRTypes for each DET verifying inclusion into the hierarchy.

**Evidence Summary:**

- (E1) Section 4 imposes mandatory publication of HHIT (and BRID for UAS RID).
- (E2) Section 7.2 recommends withholding the RRTypes until required for use.
- (E3) Section 7.1 assumes HHIT records are available for certificate walking in non-DNSSEC scenarios.

**Fix Direction:**

Clarify the temporal and operational boundaries: specify that the mandatory presence applies during active use while just-in-time non-publication is limited to pre-activation or post-retirement phases.

**Severity:** Medium
  *Basis:* The conflict can lead to inconsistent client behavior, failed certificate walking in non-DNSSEC deployments, and potential validation failures.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal: T1
- Scope: Issue-1
- Deontic: Issue-1
- CrossRFC: Issue-1

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

## Report 3: 9886-7-3

**Label:** Unscoped actor requirement for BRID auth field validation

**Bug Type:** Underspecification

**Explanation:**

The mandate to validate the BRID auth field is provided without specifying which entity is responsible, leading to potential inconsistencies in enforcement.

**Justification:**

- Section 7.1 requires that the BRID auth field, which contains endorsements, MUST be validated according to RFC9575 guidelines.
- The text references validation as performed by Broadcast RID Observers but does not clearly assign the responsibility, leaving it open to divergent interpretations.

**Evidence Snippets:**

- **E6:**

  The contents of the BRID RRType auth key, containing Endorsements as described in Section 4.2 of [RFC9575], are a shadow of the X.509 certificate found in the HHIT RRType. The validation of these Endorsements follow the guidelines written in Section 6.4.2 of [RFC9575] for Broadcast RID Observers and when present MUST also be validated.

**Evidence Summary:**

- (E6) Mandates validation of the BRID auth field per RFC9575 but does not specify which actor is responsible for this validation.

**Fix Direction:**

Specify the intended actor (e.g., Broadcast RID Observers) who must perform the BRID auth validation to ensure consistent implementation.

**Severity:** Medium
  *Basis:* Without clear role attribution, different entities may either overlook or over-apply the validation requirement, undermining consistency.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-3

---

## Report 4: 9886-7-4

**Label:** Conflict between certificate walking and Broadcast Endorsements for registration proof

**Bug Type:** Inconsistency

**Explanation:**

The text asserts that, in the absence of DNSSEC, certificate walking is the only method to prove valid registration, conflicting with RFC9575 which allows Broadcast Endorsements as an alternative method.

**Justification:**

- Section 7.1 specifies that clients MUST walk the tree of certificates via HHIT lookups as the only non-DNSSEC method for proving registration.
- RFC9575 defines Broadcast Endorsements (DRIP Link messages) as a valid mechanism for proving registration, implying an alternative to certificate walking.
- This inconsistency may cause divergent implementations depending on which method is relied upon.

**Evidence Snippets:**

- **E7:**

  When DNSSEC is not in use, … clients MUST ‘walk’ the tree of certificates locating each certificate by performing DNS lookups of HHIT RRTypes for each DET verifying inclusion into the hierarchy.

- **E8:**

  RFC 9575 Section 4.2 defines Broadcast Endorsements (DRIP Link messages) and notes that ‘The Endorsement that proves a DET is registered MUST come from its immediate parent in the registration hierarchy,’ and Section 6.4.2 further treats chains of DRIP Links as the mechanism providing trust in the key.

**Evidence Summary:**

- (E7) Asserts that certificate walking is the only method for registration proof in non-DNSSEC scenarios.
- (E8) Describes Broadcast Endorsements as an alternative mechanism for establishing registration.

**Fix Direction:**

Revise the language in Section 7.1 to either acknowledge Broadcast Endorsements as a valid alternative for registration proof or remove the exclusive reliance on certificate walking in non-DNSSEC environments.

**Severity:** Medium
  *Basis:* The conflict may lead to inconsistent registration validation practices, affecting interoperability between implementations that follow different trust proofs.

**Confidence:** Medium

**Experts mentioning this issue:**

- CrossRFC: Issue-2

---
