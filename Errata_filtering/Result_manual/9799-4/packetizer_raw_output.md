# Errata Reports

Total reports: 5

---

## Report 1: 9799-4-1

**Label:** Ambiguous Timing for Descriptor Propagation and Challenge Expiration

**Bug Type:** Underspecification

**Explanation:**

The specification lacks clear, concrete timing bounds for the client’s waiting period and the challenge expiration deadline during descriptor propagation, which may lead to mismatches between ACME servers and Hidden Services.

**Justification:**

- The text states: It MUST wait some (indeterminate) amount of time for the new descriptor to propagate, with guidance that it should take no more than a few minutes, while also stating that ACME servers MUST NOT expire challenges before a 'reasonable time'—recommended at least 30 minutes but left to operator preference.
- This vagueness can result in disparate implementations that choose significantly different timeouts, causing validation failures.

**Evidence Snippets:**

- **E1:**

  It MUST wait some (indeterminate) amount of time for the new descriptor to propagate the Tor Hidden Service directory servers before proceeding with responding to the challenge. This should take no more than a few minutes. This specification does not set a fixed time as changes in the operation of the Tor network can affect this propagation time in the future.

- **E2:**

  ACME servers MUST NOT expire challenges before a reasonable time to allow publication of the new descriptor. It is RECOMMENDED the server allow at least 30 minutes; however, it is entirely up to operator preference.

**Evidence Summary:**

- (E1) indicates an indeterminate waiting period for descriptor propagation.
- (E2) provides a vague, operator-dependent requirement for challenge expiration.

**Fix Direction:**

Define explicit quantitative timing requirements or clearly bound the conditions under which these timeouts apply specifically to .onion challenges.


**Severity:** Medium
  *Basis:* Ambiguous timing can cause the server and client to operate with incompatible assumptions, leading to systematic validation failures.

**Confidence:** High

---

## Report 2: 9799-4-2

**Label:** Directory Metadata Field Name Mismatch: inBandOnionCAARequired vs onionCAARequired

**Bug Type:** Inconsistency

**Explanation:**

The specification uses different names for the same directory metadata field in its body and IANA registration, which can cause interoperability issues between ACME servers and clients.

**Justification:**

- The document body and example define the field as inBandOnionCAARequired, while the IANA registration table lists the field as onionCAARequired.
- This naming discrepancy may lead to misinterpretation of whether in-band CAA is required.

**Evidence Snippets:**

- **E1:**

  To support signaling the server's support for fetching CAA record sets over Tor, a new field is defined in the directory ‘meta’ object to signal this. inBandOnionCAARequired (optional, boolean): If true, the ACME server requires the client to provide the CAA record set in the finalize request.

- **E2:**

  IANA registration: | Field name       | Field type | Reference | ... | onionCAARequired | boolean    | RFC 9799  |

**Evidence Summary:**

- (E1) shows the field defined as inBandOnionCAARequired in the document text.
- (E2) shows the IANA registration defines the field as onionCAARequired.

**Fix Direction:**

Align the field name across the document and the IANA registration—either update IANA to use inBandOnionCAARequired or clarify the intended mapping between the two names.


**Severity:** High
  *Basis:* A mismatch in the JSON field names can directly lead to incompatibility between implementations, causing miscommunication of CAA requirements.

**Confidence:** High

---

## Report 3: 9799-4-3

**Label:** Ambiguous Scope of authKey Across ACME Challenge Types

**Bug Type:** Underspecification

**Explanation:**

The document does not clearly specify which ACME challenge types should include the optional authKey field, leading to potential differences in implementation.

**Justification:**

- The onion-csr-01 challenge object explicitly includes the authKey field, but for http-01 and tls-alpn-01 challenges the connection is only mentioned in prose referencing modifications in Sections 4 and 6.
- This lack of explicit guidance may result in divergent interpretations among implementers.

**Evidence Snippets:**

- **E1:**

  Section 3.1.2: http-01 “MAY be used to validate a ".onion" Special-Use Domain Name with the modifications defined in this document, namely those described in Sections 4 and 6.”
Section 3.2: onion-csr-01 challenge object explicitly includes authKey (optional, object): The ACME server's Ed25519 public key encoded as per [RFC8037]. This is further defined in Section 4.
Section 4: “an additional field in the challenge object is defined … authKey (optional, object): The ACME server's Ed25519 public key encoded as per [RFC8037].”

**Evidence Summary:**

- (E1) shows that while authKey is clearly defined for onion-csr-01, its application to http-01 and tls-alpn-01 challenges remains ambiguous.

**Fix Direction:**

Explicitly state in the specification which challenge types (e.g., http-01 and tls-alpn-01) must or may include the authKey field when validating .onion domains.


**Severity:** Medium
  *Basis:* Ambiguity in which challenge objects include authKey could lead to non-interoperable implementations.

**Confidence:** High

---

## Report 4: 9799-4-4

**Label:** Mis-scoped Test for Hidden Service Client Authentication Requirement

**Bug Type:** Both

**Explanation:**

The specification relies on a per-CA CLIENT-ID match against auth-client lines to infer whether a Hidden Service requires client authentication, which may misrepresent the service-wide requirement.

**Justification:**

- Section 4 directs the ACME server to assume that if no auth-client line matches its computed CLIENT-ID then the Hidden Service does not require client authentication.
- However, later sections indicate that even if the ACME server is not authorized, the service may still require client authentication to access its descriptors.

**Evidence Snippets:**

- **E1:**

  Section 4: “There is no method to communicate to the CA that client authentication is necessary; instead, the ACME server MUST attempt to calculate its CLIENT-ID as per the ‘Client behavior’ section of [tor-spec]. If no auth-client line in the First Layer Hidden Service Descriptor matches the computed client-id, then the server MUST assume that the Hidden Service does not require client authentication and proceed accordingly.”
Section 6.2: “If the Hidden Service has client authentication enabled, then it will be impossible for the ACME server to decrypt the Second Layer Hidden Service Descriptor to read the CAA records until the ACME server's public key has been added to the First Layer Hidden Service Descriptor.”

**Evidence Summary:**

- (E1) combines the directive to use CLIENT-ID matching with an assumption that may ignore the overall client authentication requirement of the Hidden Service.

**Fix Direction:**

Revise the test so that a non-matching CLIENT-ID indicates that the ACME server is not yet authorized, rather than that the service does not require client authentication at all.


**Severity:** Medium
  *Basis:* This mis-scoping may lead to premature assumptions about service configuration, resulting in failed descriptor decryption and validation.

**Confidence:** High

---

## Report 5: 9799-4-5

**Label:** Misleading Tor-spec Reference for authKey Mechanism

**Bug Type:** Inconsistency

**Explanation:**

The specification incorrectly cites the 'Authentication during the introduction phase' section of tor-spec to justify the use of authKey for descriptor retrieval, causing potential misinterpretation of the underlying Tor mechanisms.

**Justification:**

- Section 4 instructs that the ACME server advertise an Ed25519 public key as per the 'Authentication during the introduction phase' section of tor-spec for the purpose of retrieving the Hidden Service Descriptor.
- The provided tor-spec summary, however, describes an INTRODUCE1-based authentication mechanism intended for the introduction phase and not for descriptor retrieval, leading to ambiguity in CLIENT-ID computation and authorization.

**Evidence Snippets:**

- **E1:**

  Section 4: “To this end, an additional field in the challenge object is defined to allow the ACME server to advertise the Ed25519 public key it will use (as per the 'Authentication during the introduction phase' section of [tor-spec]) to authenticate itself when retrieving the Hidden Service Descriptor.”

According to the provided tor-spec summary, 'Authentication during the introduction phase' is a mechanism for the INTRODUCE1 message and does not govern descriptor retrieval.

**Evidence Summary:**

- (E1) contrasts the intended use of authKey for descriptor retrieval with the tor-spec mechanism, which is meant for introduction-phase authentication.

**Fix Direction:**

Revise the reference to point to the correct tor-spec sections governing descriptor-level client authorization and CLIENT-ID computation, or clarify how the current reference maps to descriptor retrieval.


**Severity:** High
  *Basis:* An incorrect tor-spec reference may lead implementers to adopt the wrong authentication mechanism, resulting in failures to retrieve or decrypt Hidden Service Descriptors.

**Confidence:** Medium

---
