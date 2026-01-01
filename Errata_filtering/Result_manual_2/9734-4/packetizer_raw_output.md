# Errata Reports

Total reports: 3

---

## Report 1: 9734-4-1

**Label:** Underspecification in EKU Combination Guidance for id-kp-imUri Usage (Missing anyExtendedKeyUsage Considerations)

**Bug Type:** Underspecification

**Explanation:**

The specification only prohibits combining id-kp-imUri with id-kp-clientAuth or id-kp-serverAuth but fails to address combinations with anyExtendedKeyUsage or other broad EKUs, which may allow certificates to be misused across protocols.

**Justification:**

- Evidence shows that RFC 9734 only advises against mixing id-kp-imUri with id-kp-clientAuth or id-kp-serverAuth, while RFC 5280 permits inclusion of anyExtendedKeyUsage, leaving a gap in normative guidance (ActorDirectionality Issue-1, Scope Candidate Issue-1).
- Experts across multiple analyses note that without rules against combining id-kp-imUri with broad EKUs, the intended cross‑protocol risk reduction is undermined (Deontic Issue-1, CrossRFC Issue-1, Causal analysis).

**Evidence Snippets:**

- **E1:**

  RFC 9734 Section 4: “Issuers SHOULD NOT set the id‑kp‑imUri extended key purpose and an id‑kp‑clientAuth or id‑kp‑serverAuth extended key purpose: that would defeat the improved specificity offered by having an id‑kp‑imUri extended key purpose.”

- **E2:**

  RFC 5280 Section 4.2.1.12: “Key purposes may be defined by any organization with a need. … If a CA includes extended key usages to satisfy such applications, but does not wish to restrict usages of the key, the CA can include the special KeyPurposeId anyExtendedKeyUsage in addition to the particular key purposes required by the applications. … Certificate using applications MAY require that the extended key usage extension be present and that a particular purpose be indicated in order for the certificate to be acceptable to that application.”

- **E3:**

  RFC 9734 Section 4 (security claim): “The id-kp-imUri extended key purpose does not introduce new security risks but instead reduces existing security risks by providing means to identify if the certificate is generated to sign IM identity credentials.”

**Evidence Summary:**

- (E1) Provides the limited normative prohibition focused on id-kp-clientAuth and id-kp-serverAuth.
- (E2) Details RFC 5280’s permissive stance on using anyExtendedKeyUsage alongside specific EKUs.
- (E3) States the security claim that is weakened by the absence of broader EKU restrictions.

**Fix Direction:**

Extend normative guidance to explicitly prohibit or discourage combining id-kp-imUri with anyExtendedKeyUsage and other broad EKUs that could negate the intended specificity.

**Severity:** Medium
  *Basis:* Without comprehensive EKU combination rules, certificates may inadvertently support multiple protocols, compromising the intended security benefit.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: Issue-1
- Scope: Issue-1
- Deontic: Issue-1
- CrossRFC: Issue-1
- Causal: Underspecification

---

## Report 2: 9734-4-2

**Label:** Absence of Relying-Party Guidance for Interpreting id-kp-imUri in Certificates

**Bug Type:** Underspecification

**Explanation:**

The document directs normative guidance solely to certificate issuers without specifying how IM applications or other relying parties should interpret certificates that include id-kp-imUri, leading to potential inconsistencies.

**Justification:**

- ActorDirectionality points out that while id-kp-imUri is defined and linked to reduced security risks, no requirements are provided for IM clients or verifiers (NewIssue-1).
- Scope and Deontic analyses highlight that leaving relying-party decisions to local policy can undermine the intended risk reduction and lead to interoperability issues.

**Evidence Snippets:**

- **E1:**

  RFC 9734 defines id-kp-imUri and says it “may be included in certificates used to prove the identity of an IM client” but gives no normative text aimed at relying parties (IM clients/servers or MLS verifiers).

- **E2:**

  RFC 9734 Section 4: “The id-kp-imUri extended key purpose does not introduce new security risks but instead reduces existing security risks…” and then only gives guidance to “Issuers”.

**Evidence Summary:**

- (E1) Highlights the absence of any guidance for relying parties regarding the use of id-kp-imUri.
- (E2) Emphasizes that the security risk reduction claim is not supported by corresponding relying-party requirements.

**Fix Direction:**

Add normative recommendations for relying-party behavior, such as clear criteria for accepting or rejecting certificates that carry id-kp-imUri in combination with other EKUs.

**Severity:** Medium
  *Basis:* The lack of relying-party guidance may lead to inconsistent security policies and interoperability problems, preventing the intended risk reduction from being realized.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality: NewIssue-1
- Scope: Issue-2
- Deontic: Issue-2
- Causal: Underspecification

---

## Report 3: 9734-4-3

**Label:** Incorrect RFC Reference for XMPP URI Definition

**Bug Type:** Inconsistency

**Explanation:**

RFC 9734 incorrectly cites RFC 6121 as the source for the XMPP URI definition, which can mislead implementers in locating the normative specification for XMPP URI syntax.

**Justification:**

- CrossRFC analysis notes that while RFC 9734’s Introduction mentions XMPP URI with a reference to RFC 6121, the actual syntactical definition of XMPP URI/IRI is provided in RFC 5122 and updated by RFC 7712.
- This cross‑document reference error creates ambiguity for implementers seeking accurate normative guidance for XMPP URIs.

**Evidence Snippets:**

- **E1:**

  In RFC 9734’s Introduction, the text says that a subjectAltName “can be an IM URI [RFC3860] or Extensible Messaging and Presence Protocol (XMPP) URI [RFC6121]”. However, the XMPP URI scheme is not defined in RFC 6121; the XMPP URI/IRI syntax is defined in RFC 5122 and later updated/obsoleted by RFC 7712.

**Evidence Summary:**

- (E1) Describes the misreference where RFC 6121 is incorrectly cited for the XMPP URI definition instead of RFC 5122/RFC 7712.

**Fix Direction:**

Update the reference to correctly cite RFC 5122 and RFC 7712 as the defining documents for the XMPP URI syntax, or clarify the intended use of RFC 6121 if applicable.

**Severity:** High
  *Basis:* An incorrect RFC reference can mislead implementers and result in inconsistent application of the URI syntax, impacting interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC: Issue-2

---
