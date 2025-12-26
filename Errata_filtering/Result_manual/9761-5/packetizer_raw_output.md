# Errata Reports

Total reports: 7

---

## Report 1: 9761-5-1

**Label:** Mis-scoped spki-pin-set and certificate-authority parameters between iana-tls-profile and ietf-acl-tls

**Bug Type:** Inconsistency

**Explanation:**

Section 5.3 incorrectly attributes spki-pin-set and certificate-authority parameters to the iana-tls-profile module, while these parameters are defined (or intended to be device‐specific) in the ietf-acl-tls module.

**Justification:**

- Section 5.3 states: The values for all the parameters in the 'iana-tls-profile' YANG module are defined in the TLS and DTLS IANA registries excluding the tls-version, dtls-version, spki-pin-set, and certificate-authority parameters. The values of spki-pin-set and certificate-authority parameters will be specific to the IoT device.
- The iana-tls-profile YANG module as given defines typedefs only for extension-type, supported-group, signature-algorithm, psk-key-exchange-mode, application-protocol, cert-compression-algorithm, cipher-algorithm, tls-version, and dtls-version; no spki-pin-set or certificate-authority typedefs appear in it.
- In contrast, the ietf-acl-tls module defines: typedef spki-pin-set { type binary; ... } and typedef certificate-authority { type string; ... } and uses certificate-authority as the type of the certificate-authorities leaf-list.

**Evidence Snippets:**

- **E1:**

  Section 5.3: “The values for all the parameters in the `iana-tls-profile` YANG module are defined in the TLS and DTLS IANA registries excluding the tls-version, dtls-version, spki-pin-set, and certificate-authority parameters. The values of spki-pin-set and certificate-authority parameters will be specific to the IoT device.”

- **E2:**

  The `iana-tls-profile` YANG module as given defines typedefs only for `extension-type`, `supported-group`, `signature-algorithm`, `psk-key-exchange-mode`, `application-protocol`, `cert-compression-algorithm`, `cipher-algorithm`, `tls-version`, and `dtls-version`; no `spki-pin-set` or `certificate-authority` typedefs appear in it.

- **E3:**

  The `ietf-acl-tls` module defines: typedef spki-pin-set { type binary; ... } typedef certificate-authority { type string; ... } and uses `certificate-authority` as the type of the `certificate-authorities` leaf-list (gated by `{tls13 or dtls13}`?).

**Evidence Summary:**

- (E1) Section 5.3 describes excluded parameters for the iana-tls-profile module.
- (E2) The iana-tls-profile module lacks any spki-pin-set or certificate-authority typedefs.
- (E3) The ietf-acl-tls module defines these typedefs, indicating a mis-scoping.

**Fix Direction:**

Clarify in Section 5.3 that spki-pin-set and certificate-authority are not part of the iana-tls-profile module; update the text and associated IANA registry descriptions to reflect that these parameters are device-specific and defined in the ietf-acl-tls module, or remove unused references.


**Severity:** High
  *Basis:* This mis-scoping can lead to serious interoperability issues and confusion for both implementers and IANA maintainers regarding parameter ownership.

**Confidence:** High

---

## Report 2: 9761-5-2

**Label:** Ambiguous attribution of match-on-tls-dtls feature support between device/MUD and ACL server

**Bug Type:** Underspecification

**Explanation:**

The specification ambiguously attributes the implementation of the match-on-tls-dtls feature to the IoT device or its MUD file instead of clearly limiting it to the ACL server as defined in the YANG model.

**Justification:**

- YANG in Section 5.2 defines the feature match-on-tls-dtls as an ACL server capability with an augment that applies to the ACL model.
- Section 5.4 text states that if the ietf-mud-tls extension is supported by the device, the MUD file is assumed to implement the match-on-tls-dtls feature, conflating roles and creating ambiguity.

**Evidence Snippets:**

- **E1:**

  YANG in Section 5.2: feature match-on-tls-dtls { description "The networking device can support matching on (D)TLS parameters."; } and an augment: ... 

- **E2:**

  Section 5.4 text: “If the `ietf-mud-tls` extension is supported by the device, MUD file is assumed to implement the `match-on-tls-dtls` ACL model feature defined in this specification.”

**Evidence Summary:**

- (E1) The YANG model defines match-on-tls-dtls as an ACL server capability.
- (E2) Section 5.4 ambiguously assigns this feature to the device/MUD file.

**Fix Direction:**

Revise Section 5.4 to clarify that match-on-tls-dtls is solely an ACL server capability, and that the presence of the ietf-mud-tls extension indicates that the network security service must support this feature.


**Severity:** Medium
  *Basis:* Incorrect feature attribution may lead to configuration mismatches, though it is less likely to completely break interoperability.

**Confidence:** High

---

## Report 3: 9761-5-3

**Label:** Inconsistent certificate-authority encoding: DER-encoded requirement vs free-form string in YANG

**Bug Type:** Both

**Explanation:**

The specification mandates that certificate authority distinguished names be provided in a DER-encoded format, yet the YANG model defines the certificate-authority type as a free-form string without constraints, leading to potential mismatches between implementations.

**Justification:**

- Section 5 specifies the certificate authorities as 'represented in DER-encoded format [X690] (defined in Section 4.2.4 of RFC8446)', aligning with TLS 1.3 requirements.
- The ietf-acl-tls module, however, defines a typedef for certificate-authority as 'type string' without specifying how DER encoding is to be represented.
- This inconsistency may lead to different interpretations (e.g., base64, hex, or textual formats) and incompatibility in matching certificate authorities during TLS handshakes.

**Evidence Snippets:**

- **E1:**

  Section 5 describes CA names as DER‑encoded distinguished names (defined in Section 4.2.4 of RFC8446).

- **E2:**

  In `ietf-acl-tls`: typedef certificate-authority { type string; description "Distinguished Name of Certificate authority as discussed in Section 4.2.4 of RFC 8446."; }

- **E3:**

  TLS 1.3 defines DistinguishedName as an opaque field carrying DER‑encoded X.501 Name according to Section 4.2.4.

**Evidence Summary:**

- (E1) The prose mandates DER-encoded CA values.
- (E2) The YANG model uses a free-form string for certificate-authority.
- (E3) TLS 1.3 requires that DistinguishedName be DER-encoded.

**Fix Direction:**

Either change the certificate-authority typedef to 'type binary' to carry DER-encoded data (with appropriate RFC 7951 encoding rules) or update the prose to specify a canonical textual representation such as RFC 4514, ensuring both prose and model are aligned.


**Severity:** High
  *Basis:* This inconsistency can critically affect security policy enforcement by causing mismatches in certificate authority comparisons across different implementations.

**Confidence:** High

---

## Report 4: 9761-5-4

**Label:** Profile name restrictions not enforced between description and YANG schema

**Bug Type:** Inconsistency

**Explanation:**

The profile name is described as prohibiting spaces and special characters, yet the YANG schema only enforces a length constraint, allowing any characters.

**Justification:**

- In the ietf-acl-tls list definition, the leaf 'name' is defined with a type string that restricts length to 1..64 but does not include a pattern to forbid spaces and special characters.
- The description explicitly states 'space and special characters are not allowed,' creating a divergence between the normative text and the enforced schema.

**Evidence Snippets:**

- **E1:**

  leaf name { type string { length "1..64"; } description "The name of (D)TLS profile; space and special characters are not allowed."; }

**Evidence Summary:**

- (E1) The YANG leaf 'name' only constrains length while its description prohibits spaces and special characters.

**Fix Direction:**

Either add a pattern restriction to the YANG type to enforce the prohibition on spaces and special characters or update the description to reflect that only the length is constrained.


**Severity:** Low
  *Basis:* This issue affects configuration consistency and interoperability of profile identifiers, but its impact is limited to management-plane validation.

**Confidence:** High

---

## Report 5: 9761-5-5

**Label:** Naming mismatches between tree diagram, JSON example, and normative YANG modules

**Bug Type:** Inconsistency

**Explanation:**

There are multiple discrepancies in node naming across the tree diagram, JSON example, and the YANG modules (e.g., use of 'supported-groups' versus 'supported-group', and 'client-profile' versus 'client-profiles'), which can lead to confusion and validation errors.

**Justification:**

- The tree diagram in Section 5.1 shows '+--rw supported-groups*' while the YANG module defines the leaf-list as 'supported-group'.
- The JSON example uses node names such as 'ietf-acl-tls:client-profile' and 'tls-dtls-profiles', which do not match the normative YANG identifiers ('client-profiles' and 'tls-dtls-profile').

**Evidence Snippets:**

- **E1:**

  Tree in 5.1: '+--rw supported-groups* |       ianatp:supported-group'

- **E2:**

  YANG in 5.2: 'leaf-list supported-group { type ianatp:supported-group; ... }'

- **E3:**

  JSON example in Section 7 uses 'ietf-acl-tls:client-profile' and 'tls-dtls-profiles' instead of the YANG-defined names.

**Evidence Summary:**

- (E1) The tree diagram uses 'supported-groups*'.
- (E2) The YANG module defines it as 'supported-group'.
- (E3) The JSON example employs inconsistent naming such as 'client-profile' instead of 'client-profiles'.

**Fix Direction:**

Update the tree diagram and JSON examples to precisely match the YANG node names (e.g., using 'supported-group' and 'client-profiles') as dictated by the normative model.


**Severity:** Medium
  *Basis:* While these naming mismatches mainly affect documentation and example correctness, they can lead to implementation errors if developers rely on the provided examples.

**Confidence:** High

---

## Report 6: 9761-5-6

**Label:** JSON MUD example does not conform to the defined YANG models

**Bug Type:** Inconsistency

**Explanation:**

The JSON example provided in Section 7 contains several errors—such as boolean values encoded as strings, misnamed nodes, and incorrect nesting—that cause it not to conform to the YANG model definitions.

**Justification:**

- The JSON example encodes the boolean 'ietf-mud-tls:is-tls-dtls-profile-supported' as "true" (a string) instead of as a boolean literal.
- It also misnames nodes, using 'ietf-acl-tls:client-profile' and 'tls-dtls-profiles' rather than the correct 'client-profiles' and 'tls-dtls-profile', and nests 'actions' improperly under 'matches'.

**Evidence Snippets:**

- **E1:**

  "ietf-mud-tls:is-tls-dtls-profile-supported": "true",

- **E2:**

  JSON example uses 'ietf-acl-tls:client-profile' and 'tls-dtls-profiles' instead of the correct names as defined in YANG.

**Evidence Summary:**

- (E1) Boolean value is encoded as a string rather than a boolean.
- (E2) Node names in the JSON example do not match the normative YANG definitions.

**Fix Direction:**

Revise the JSON example to use proper boolean literals and to exactly match the YANG structure and node names per RFC 7951, ensuring that nodes such as 'client-profiles' and 'tls-dtls-profile' are correctly named and placed.


**Severity:** High
  *Basis:* Non-conformance in the example can mislead implementers and result in parsing or validation errors, directly impacting interoperability.

**Confidence:** High

---

## Report 7: 9761-5-7

**Label:** NACM default-deny-write claimed in security considerations but not present in YANG modules

**Bug Type:** Inconsistency

**Explanation:**

The security considerations claim that the NACM default-deny-write extension has been applied to all data nodes, yet the YANG modules for ietf-acl-tls and ietf-mud-tls do not import or include any NACM-related annotations.

**Justification:**

- Sections 9.3 and 9.4 assert that 'the NACM extension "default-deny-write" has been set for all data nodes defined in this module.'
- However, the ietf-acl-tls and ietf-mud-tls YANG modules do not import ietf-netconf-acm and contain no nacm:default-deny-write statements, contradicting the security claims.

**Evidence Snippets:**

- **E1:**

  Section 9.3: “For this reason, the NACM extension "default-deny-write" has been set for all data nodes defined in this module.”

- **E2:**

  The ietf-acl-tls and ietf-mud-tls YANG modules, as shown, import no ietf-netconf-acm and contain no nacm:default-deny-write statements.

**Evidence Summary:**

- (E1) Security considerations claim NACM default-deny-write for all nodes.
- (E2) The YANG modules do not include any NACM annotations.

**Fix Direction:**

Either add the NACM annotations (import ietf-netconf-acm and add nacm:default-deny-write statements) to the YANG modules or update the security considerations to accurately describe the current schema behavior.


**Severity:** Medium
  *Basis:* The lack of NACM extensions could lead implementers to a false sense of access-control security, potentially impacting secure deployments.

**Confidence:** High

---
