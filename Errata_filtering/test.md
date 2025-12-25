# Errata Reports

Total reports: 4

---

## Report 1: 9810-4-1

**Label:** Ambiguous 'next update' validity rule for 'new with new' certificate

**Bug Type:** Underspecification

**Explanation:**

The specification requires the 'new with new' certificate to have a notAfter time that is after the notBefore time of the next update, but the term 'next update of this certificate' is undefined and reverses the clearer rule from RFC 4210.

**Justification:**

- The text states: The ‘new with new’ certificate must have a validity period with a notBefore time that is before the notAfter time of the ‘old with old’ certificate and a notAfter time that is after the notBefore time of the next update of this certificate.
- For comparison, RFC 4210 clearly required the validity to end at or before the CA’s next key update event.

**Evidence Snippets:**

- **E1:**

  The ‘new with new’ certificate must have a validity period with a notBefore time that is before the notAfter time of the ‘old with old’ certificate and a notAfter time that is after the notBefore time of the next update of this certificate.

- **E2:**

  RFC 4210 wording (for comparison): “The ‘new with new’ certificate must have a validity period starting at the generation time of the new key pair and ending at or before the time by which the CA will next update its key pair.”

**Evidence Summary:**

- (E1) specifies an ambiguous validity period requirement referencing an undefined 'next update'.
- (E2) shows the clear requirement from RFC 4210 that is not maintained in the current text.

**Fix Direction:**

Replace the ambiguous phrase with a directive tied to the CA’s next key-update event (e.g., 'ending at or before the time by which the CA will next update its key pair') and clearly state the intended inequality.

**Severity:** Low
  *Basis:* The undefined term 'next update of this certificate' forces operators to guess at a concrete validity interval, potentially leading to inconsistent rollover semantics without breaking on‐the‐wire interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1
- Quantitative Expert: Issue-1
- Deontic Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 2: 9810-4-2

**Label:** Optional link certificates omitted vs. verification procedures assuming their presence

**Bug Type:** Both

**Explanation:**

While the specification marks the link certificates (newWithOld and oldWithNew) as OPTIONAL, the verification procedures for Cases 2 and 3 assume their presence, leaving no guidance for implementations that legitimately omit them.

**Justification:**

- The document notes that 'RootCaKeyUpdateContent is updated to specify these link certificates as optional' and states that their usage is use-case specific.
- However, the verification procedures instruct verifiers to always 'get the ‘new with new’ and ‘new with old’ certificates' (or 'get the ‘old with new’ certificate') without describing a fallback when they are absent.

**Evidence Snippets:**

- **E1:**

  Note: The usage of link certificates has been shown to be very specific for each use case, and no assumptions are done on this aspect. RootCaKeyUpdateContent is updated to specify these link certificates as optional.

- **E2:**

  In case 2, the verifier must get access to the new public key of the CA. The verifier does the following: 1. Get the ‘new with new’ and ‘new with old’ certificates.

**Evidence Summary:**

- (E1) establishes that link certificates are optional and not to be assumed.
- (E2) shows that the verification procedure unconditionally requires these certificates.

**Fix Direction:**

Either condition the verification procedures on the presence of the link certificates or provide alternative trust-anchor update mechanisms when they are omitted.

**Severity:** Medium
  *Basis:* Omitting link certificates is permitted by the data model; however, the fixed verification steps force an implementation error or fallback with no clear normative guidance.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T2
- Scope Expert: Issue-1
- Causal Expert: Issue-2
- Deontic Expert: Issue-2
- Structural Expert: Issue-2

---

## Report 3: 9810-4-3

**Label:** Misnamed LDAP/X.509 CA certificate attribute ('caCertificate' vs 'cACertificate')

**Bug Type:** Inconsistency

**Explanation:**

The specification inconsistently refers to the LDAP attribute for CA certificates; it is first identified using the standard 'cACertificate' but later referred to as 'caCertificate', which does not match the X.500/LDAP standard.

**Justification:**

- The document correctly states that certificates are stored as cACertificate attribute values when published in an LDAP directory.
- Later, instructions to 'look up the certificates in the caCertificate attribute' conflict with the standardized attribute name.

**Evidence Snippets:**

- **E1:**

  When an LDAP directory is used to publish root CA updates, the old and new root CA certificates together with the two link certificates are stored as cACertificate attribute values.

- **E2:**

  If a repository is available, look up the certificates in the caCertificate attribute.

**Evidence Summary:**

- (E1) shows the correct, standard attribute name 'cACertificate'.
- (E2) demonstrates the later inconsistent use of 'caCertificate'.

**Fix Direction:**

Replace 'caCertificate' with 'cACertificate' in all procedural instructions to align with the X.500/LDAP standard and earlier text.

**Severity:** Low
  *Basis:* Although LDAP attribute names are typically treated case-insensitively, the inconsistency can mislead implementers and documentation authors.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-1
- Terminology Expert: Issue-1
- CrossRFC Expert: Issue-2

---

## Report 4: 9810-4-4

**Label:** Case 2 signature verification error: using old CA key for 'new with new' certificate

**Bug Type:** Causal Inconsistency

**Explanation:**

In the Case 2 verification procedure, the specification instructs verifiers to verify the 'new with new' certificate using the old CA key, which is incorrect because 'new with new' is signed with the new key.

**Justification:**

- The procedure states to 'Verify the signatures using the old root CA key (which the verifier has locally)' even though the 'new with new' certificate is self‐signed by the new key.
- This error leads to a situation where the verification process will always fail for honest deployments.

**Evidence Snippets:**

- **E1:**

  In case 2, the verifier must get access to the new public key of the CA. The verifier does the following: 1. Get the ‘new with new’ and ‘new with old’ certificates. … Verify the signatures using the old root CA key (which the verifier has locally).

**Evidence Summary:**

- (E1) illustrates that the verification instruction incorrectly applies the old key to the 'new with new' certificate.

**Fix Direction:**

Revise the verification algorithm so that the old CA key is used solely for verifying the 'new with old' certificate, while the 'new with new' certificate is verified using the new CA public key or appropriate self-signature checks.

**Severity:** Medium
  *Basis:* Using the wrong key for signature verification directly prevents the certificate chain from being established in normal operations, causing valid updates to be rejected.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Bug 1

---
