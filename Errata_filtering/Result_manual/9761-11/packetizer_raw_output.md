# Errata Reports

Total reports: 6

---

## Report 1: 9761-11-1

**Label:** Inconsistent Naming and JSON Encoding of YANG Data Nodes

**Bug Type:** Inconsistency

**Explanation:**

The IANA text and JSON example refer to a non-existent node 'client‐profile' and use inconsistent pluralization (e.g. 'tls‐dtls‐profiles' instead of the defined 'client‐profiles' container and 'tls‐dtls‐profile' list), and a Boolean value is improperly encoded as a string.

**Justification:**

- Section 11.1 states that new (D)TLS parameters are added to 'client‐profile' even though the YANG module defines a container 'client‐profiles' with a child list 'tls‐dtls‐profile'.
- The JSON example in Section 7 uses 'ietf‐acl‐tls:client‐profile' with 'tls‐dtls‐profiles' and pluralized leaf names, plus encodes a Boolean as "true" rather than as a JSON boolean.

**Evidence Snippets:**

- **E1:**

  Section 11.1: “IANA has created an IANA-maintained YANG module called "iana-tls-profile" ... which allows for new (D)TLS parameters and (D)TLS versions to be added to "client-profile".”

- **E2:**

  YANG tree: augment adds container "client-profiles" and list "tls-dtls-profile" under "/acl:matches".

- **E3:**

  Example JSON (Section 7): "ietf-acl-tls:client-profile" : { "tls-dtls-profiles" : [ { "name" : "profile1", "supported-tls-versions" : ["tls13"], "cipher-suite" : [4865, 4866], "extension-types" : [10,11,13,16,24], "supported-groups" : [29] } ] }

- **E4:**

  Section 7 JSON: "ietf-mud-tls:is-tls-dtls-profile-supported": "true"

**Evidence Summary:**

- (E1) IANA text incorrectly refers to 'client-profile' rather than the defined 'client-profiles'.
- (E2) The YANG tree defines a container 'client-profiles' with a list 'tls-dtls-profile'.
- (E3) The JSON example uses non-existent node names and pluralizations not found in the YANG model.
- (E4) A Boolean value is encoded as a string, violating the expected RFC 7951 types.

**Fix Direction:**

Update Section 11.1 and the JSON example in Section 7 to reference the correct node names ('client-profiles' and 'tls-dtls-profile') and to encode Boolean values properly as JSON booleans.


**Severity:** Medium
  *Basis:* This inconsistency may lead to confusion for implementers and IANA personnel, as tools relying on the normative YANG model may reject or misinterpret the JSON data.

**Confidence:** High

---

## Report 2: 9761-11-2

**Label:** Misattribution of spki-pin-set and certificate-authority Parameters to IANA-Maintained Module

**Bug Type:** Inconsistency

**Explanation:**

Section 5.3 erroneously claims that the spki-pin-set and certificate-authority parameters belong to the iana-tls-profile module, while in reality they are defined in the ietf-acl-tls module.

**Justification:**

- The text in Section 5.3 lists spki-pin-set and certificate-authority among parameters in the iana-tls-profile module.
- The actual iana-tls-profile module does not define these parameters; they are defined in the ietf-acl-tls module.

**Evidence Snippets:**

- **E1:**

  Section 5.3: “The values for all the parameters in the "iana-tls-profile" YANG module are defined in the TLS and DTLS IANA registries excluding the tls-version, dtls-version, spki-pin-set, and certificate-authority parameters. The values of spki-pin-set and certificate-authority parameters will be specific to the IoT device.”

- **E2:**

  ietf-acl-tls YANG: defines typedef spki-pin-set and typedef certificate-authority; in contrast, the iana-tls-profile module does not include these typedefs.

**Evidence Summary:**

- (E1) Section 5.3 misattributes spki-pin-set and certificate-authority as parts of iana-tls-profile.
- (E2) Actual YANG modules show these parameters are defined in ietf-acl-tls.

**Fix Direction:**

Revise Section 5.3 to correctly attribute spki-pin-set and certificate-authority to the ietf-acl-tls module, or update the iana-tls-profile module if that was the intended design.


**Severity:** Medium
  *Basis:* Misattribution may lead to misunderstandings regarding module boundaries and IANA’s role in maintaining parameter definitions.

**Confidence:** High

---

## Report 3: 9761-11-3

**Label:** Incorrect YANG Update Rule: 'type' vs 'typedef'

**Bug Type:** Inconsistency

**Explanation:**

Section 11.2 directs IANA to add a new 'type' statement with substatements for new parameters, but the normative YANG module uses 'typedef' to define derived types, causing a mismatch with valid YANG syntax.

**Justification:**

- The YANG module in Section 5.3 defines derived types using 'typedef' (e.g., typedef extension-type { type uint16; ... }).
- Section 11.2 instructs the addition of a 'type' statement with substatements (derived type, built-in type, description) that are not valid in YANG.

**Evidence Snippets:**

- **E1:**

  Section 5.3 YANG module: typedef extension-type { type uint16; description "Extension type in the TLS ExtensionType Values registry as defined in Section 7 of RFC 8447."; }

- **E2:**

  Section 11.2: “When a (D)TLS parameter is added to the ‘ACL (D)TLS Parameters’ registry, a new ‘type’ statement must be added to the iana-tls-profile YANG module. The following ‘type’ statement, and substatements thereof, should be defined: ‘derived type’: ... ‘built-in type’: ... ‘description’: ...”

**Evidence Summary:**

- (E1) The YANG module defines derived types using the 'typedef' construct.
- (E2) Section 11.2 erroneously mandates a 'type' statement with substatements not supported by YANG.

**Fix Direction:**

Modify Section 11.2 so that it instructs the use of a new 'typedef' statement (with an inner 'type' and description) in order to align with standard YANG syntax.


**Severity:** Medium
  *Basis:* This normative inconsistency may cause confusion during module updates, potentially leading to invalid YANG if interpreted literally.

**Confidence:** High

---

## Report 4: 9761-11-4

**Label:** Wrong YANG Prefix in IANA Module Names Registry for ietf-acl-tls

**Bug Type:** Inconsistency

**Explanation:**

The ietf-acl-tls YANG module declares its prefix as 'acl-tls', but the IANA registry entry in Section 11.1 incorrectly lists its prefix as 'ietf-acl-tls'.

**Justification:**

- The YANG module clearly sets its prefix with: prefix acl-tls;
- Section 11.1 of the IANA registration text lists the prefix for ietf-acl-tls as 'ietf-acl-tls', which conflicts with the actual module declaration.

**Evidence Snippets:**

- **E1:**

  YANG module definition: module ietf-acl-tls { ... prefix acl-tls; ... }

- **E2:**

  IANA registration text (Section 11.1): Name: ietf-acl-tls ... Prefix: ietf-acl-tls

**Evidence Summary:**

- (E1) The module defines its prefix as 'acl-tls'.
- (E2) The IANA registry entry incorrectly records the prefix as 'ietf-acl-tls'.

**Fix Direction:**

Update the IANA registry entry in Section 11.1 to reflect the correct YANG prefix 'acl-tls' for the ietf-acl-tls module.


**Severity:** Medium
  *Basis:* This mismatch can confuse tool developers and implementers who rely on the registry for accurate module metadata.

**Confidence:** High

---

## Report 5: 9761-11-5

**Label:** Missing NACM 'default-deny-write' Extension in YANG Modules

**Bug Type:** Inconsistency

**Explanation:**

Although Sections 9.3 and 9.4 claim that the NACM extension 'default-deny-write' is set for all data nodes, the provided YANG modules lack the required import of ietf-netconf-acm and do not include any nacm:default-deny-write extensions.

**Justification:**

- Section 9.3/9.4 explicitly state that the NACM 'default-deny-write' extension has been set on all data nodes in the modules.
- Review of the YANG modules shows neither an import of ietf-netconf-acm nor any use of the nacm:default-deny-write extension.

**Evidence Snippets:**

- **E1:**

  Section 9.3: “For this reason, the NACM extension "default-deny-write" has been set for all data nodes defined in this module.”

- **E2:**

  The shown YANG modules do not import ietf-netconf-acm and contain no nacm:default-deny-write extensions.

**Evidence Summary:**

- (E1) The text claims that the NACM 'default-deny-write' extension is applied to all nodes.
- (E2) The actual YANG modules lack the necessary import and extension use for NACM.

**Fix Direction:**

Either update the YANG modules to import ietf-netconf-acm and attach the nacm:default-deny-write extension to all appropriate nodes, or modify the descriptive text to remove the incorrect claim.


**Severity:** Medium
  *Basis:* Operators might assume enhanced security controls are enforced by the schema when they are not, leading to misconfigurations.

**Confidence:** High

---

## Report 6: 9761-11-6

**Label:** Underspecification in TLS/DTLS Version Registry Cross-Reference

**Bug Type:** Underspecification

**Explanation:**

Section 11.5 does not explicitly clarify that the tls-version and dtls-version values are to be sourced from the new ACL TLS/DTLS Version Codes registries rather than from the TLS/DTLS Parameters registry, leading to potential ambiguity.

**Justification:**

- A ResidualUncertainties note indicates that though the interaction is mostly clear when read together, Section 11.5 omits explicit reference to the registry source for tls-version and dtls-version values.
- This omission may leave room for misinterpretation about where the version values should be drawn from.

**Evidence Snippets:**

- **E1:**

  ResidualUncertainties: The interaction between Section 11.5’s description of the “ACL (D)TLS Parameters” registry and Sections 11.3/11.4 (version-code registries) is mostly clear when read together, but 11.5 does not explicitly say that tls-version/dtls-version values come from the new version registries.

**Evidence Summary:**

- (E1) Section 11.5 lacks an explicit statement that tls-version and dtls-version values derive from the new ACL TLS/DTLS Version Codes registries.

**Fix Direction:**

Clarify in Section 11.5 that the tls-version and dtls-version values are specifically drawn from the ACL TLS/DTLS Version Codes registries.


**Severity:** Low
  *Basis:* While the ambiguity is minor and likely inferrable by experienced implementers, explicit clarification would help prevent potential misunderstandings.

**Confidence:** Medium

---
