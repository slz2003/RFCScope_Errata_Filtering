# Errata Reports

Total reports: 6

---

## Report 1: 9761-7-1

**Label:** ACL container incorrectly nested under MUD container

**Bug Type:** Inconsistency

**Explanation:**

The JSON example incorrectly nests the ACL container within the MUD container instead of listing them as separate top‐level objects, violating RFC 8520’s required structure.

**Justification:**

- RFC 8520 states that a valid MUD file must contain two root objects—a 'mud' container and an 'acls' container as separate top‐level members.
- The Section 7 JSON example shows 'ietf-access-control-list:acls' nested inside 'ietf-mud:mud', which contradicts the YANG model.

**Evidence Snippets:**

- **E1:**

  RFC 8520 explicitly: “A valid MUD file will contain two root objects: a ‘mud’ container and an ‘acls’ container. Extensions may add additional root objects as required.”  The RFC 8520 example shows:  {
  "ietf-mud:mud": { ... },
  "ietf-access-control-list:acls": { ... }
}

- **E2:**

  In Section 7 of RFC 9761, the example instead nests ACLs inside the MUD object:  {
  "ietf-mud:mud": {
    ...
    "from-device-policy": { ... },
    "ietf-access-control-list:acls": {
      "acl": [ ... ]
    }
  }
}

**Evidence Summary:**

- (E1) RFC 8520 requires separate top-level 'mud' and 'acls' objects.
- (E2) The example in Section 7 nests 'acls' inside 'mud', which is inconsistent.

**Fix Direction:**

Re-arrange the JSON so that 'ietf-access-control-list:acls' is placed as a top-level sibling of 'ietf-mud:mud'.


**Severity:** High
  *Basis:* This violation breaks the fundamental JSON-YANG binding expected by RFC 8520 and will lead to schema validation errors.

**Confidence:** High

---

## Report 2: 9761-7-2

**Label:** Actions container incorrectly placed inside matches

**Bug Type:** Inconsistency

**Explanation:**

The JSON example incorrectly nests the 'actions' container within the 'matches' container of an ACE entry rather than as a separate sibling, causing structural errors.

**Justification:**

- RFC 8520’s example shows 'matches' and 'actions' as sibling members of an ACE, not nested together.
- Embedding 'actions' inside 'matches' violates the YANG model for ACLs and may result in dropped or misinterpreted action data.

**Evidence Snippets:**

- **E1:**

  JSON example fragment:

"ace": [
  {
    "name": "cl0-frdev",
    "matches": {
      "ipv6": { ... },
      "tcp": { ... },
      "ietf-acl-tls:client-profile": { ... },
      "actions": {
         "forwarding": "accept"
      }
    }
  }
]

- **E2:**

  RFC 8520 example ACE:

"ace": [
  {
    "name": "cl0-frdev",
    "matches": { ... },
    "actions": {
      "forwarding": "accept"
    }
  }
]

**Evidence Summary:**

- (E1) The Section 7 JSON shows 'actions' nested within 'matches', which is not supported by the YANG model.
- (E2) The normative RFC 8520 example has 'actions' as a separate sibling of 'matches'.

**Fix Direction:**

Restructure each ACE so that 'actions' is placed as a sibling to 'matches', not within it.


**Severity:** High
  *Basis:* Incorrect nesting of 'actions' leads to validation errors and potentially missing ACL action instructions.

**Confidence:** High

---

## Report 3: 9761-7-3

**Label:** Misnamed ietf-acl-tls JSON members do not match YANG identifiers

**Bug Type:** Inconsistency

**Explanation:**

The JSON example uses incorrect member names for the TLS profile augmentation (e.g., 'client-profile' instead of 'client-profiles' and 'tls-dtls-profiles' instead of 'tls-dtls-profile'), preventing proper matching with the YANG model.

**Justification:**

- The ietf-acl-tls YANG module specifies a container named 'client-profiles' and a list 'tls-dtls-profile', but the JSON example uses singular/plural variations that do not exist in the model.
- Other leaf-list names such as 'supported-tls-version' are mismatched by being pluralized in the JSON ('supported-tls-versions'), which leads to unrecognized nodes.

**Evidence Snippets:**

- **E1:**

  YANG (ietf-acl-tls, Section 5.2):

container client-profiles { ... list tls-dtls-profile { key "name"; ... leaf-list supported-tls-version { ... } leaf-list cipher-suite { ... } leaf-list extension-type { ... } leaf-list supported-group { ... } ... } }

- **E2:**

  JSON example:

"ietf-acl-tls:client-profile" : {
  "tls-dtls-profiles" : [
    {
      "name" : "profile1",
      "supported-tls-versions" : ["tls13"],
      "cipher-suite" : [4865, 4866],
      "extension-types" : [10,11,13,16,24],
      "supported-groups" : [29]
    }
  ]
}

**Evidence Summary:**

- (E1) The YANG definition requires the use of 'client-profiles' and 'tls-dtls-profile' with specific leaf names.
- (E2) The JSON example uses misnamed members such as 'client-profile' and 'tls-dtls-profiles', which do not match the definition.

**Fix Direction:**

Rename the JSON members to exactly match the YANG identifiers (e.g., use 'ietf-acl-tls:client-profiles' with a 'tls-dtls-profile' list and correct leaf names).


**Severity:** High
  *Basis:* Misnaming leads to nonconformance with the YANG model and causes TLS profile information to be dropped or ignored.

**Confidence:** High

---

## Report 4: 9761-7-4

**Label:** Boolean leaf misencoded as JSON string instead of boolean

**Bug Type:** Inconsistency

**Explanation:**

The boolean leaf 'ietf-mud-tls:is-tls-dtls-profile-supported' is encoded as the string "true" instead of the JSON boolean literal true, violating RFC 7951 requirements.

**Justification:**

- RFC 7951 requires boolean values to be encoded as true or false without quotes.
- Using a string value leads to a type mismatch that may cause the leaf to be rejected or defaulted incorrectly.

**Evidence Snippets:**

- **E1:**

  ietf-mud-tls defines:

leaf is-tls-dtls-profile-supported {
  type boolean;
  default "false";
}

- **E2:**

  JSON example uses:

"ietf-mud-tls:is-tls-dtls-profile-supported": "true",

**Evidence Summary:**

- (E1) The YANG model defines the leaf as boolean with a default of "false".
- (E2) The JSON example incorrectly encodes the value as a string "true".

**Fix Direction:**

Encode the boolean value without quotes (e.g., "ietf-mud-tls:is-tls-dtls-profile-supported": true).


**Severity:** High
  *Basis:* This error violates the JSON encoding rules for booleans and may lead to misinterpretation of device capabilities.

**Confidence:** High

---

## Report 5: 9761-7-5

**Label:** IANA text refers to non-existent 'client-profile' node

**Bug Type:** Inconsistency

**Explanation:**

The IANA documentation mistakenly refers to a 'client-profile' node, while the correct YANG model defines the container as 'client-profiles', which could mislead readers.

**Justification:**

- Terminology Expert notes that Section 11.1 uses the term 'client-profile', but no such node exists in the ietf-acl-tls or iana-tls-profile modules.
- This misnaming may cause confusion in registry maintenance and in implementers’ interpretations of the model.

**Evidence Snippets:**

- **E1:**

  Section 11.1:

“IANA has created an IANA-maintained YANG module called ‘iana-tls-profile’ based on the contents of Section 5.3, which allows for new (D)TLS parameters and (D)TLS versions to be added to ‘client- profile’.”

**Evidence Summary:**

- (E1) The IANA text refers to adding parameters to ‘client-profile’, a node that does not exist in the YANG modules.

**Fix Direction:**

Update the IANA documentation to refer to the correct node name ('client-profiles').


**Severity:** Low
  *Basis:* This is primarily an editorial issue that may cause minor confusion, but does not affect the structural conformance of implementations.

**Confidence:** High

---

## Report 6: 9761-7-6

**Label:** Misattribution of spki-pin-set and certificate-authority to the iana-tls-profile module

**Bug Type:** Inconsistency

**Explanation:**

The documentation erroneously attributes the spki-pin-set and certificate-authority typedefs to the iana-tls-profile module, when they are actually defined in the ietf-acl-tls module.

**Justification:**

- Terminology Expert indicates that the prose in Section 5.3 wrongly states these parameters belong to the iana-tls-profile module.
- The actual YANG modules show that spki-pin-set and certificate-authority are defined in ietf-acl-tls and are device-specific.

**Evidence Snippets:**

- **E1:**

  Section 5.3 (first paragraph):  “The values for all the parameters in the ‘iana-tls-profile’ YANG module are defined in the TLS and DTLS IANA registries excluding the tls-version, dtls-version, spki-pin-set, and certificate-authority parameters.  The values of spki-pin-set and certificate-authority parameters will be specific to the IoT device.”

**Evidence Summary:**

- (E1) The text misattributes spki-pin-set and certificate-authority to the iana-tls-profile module, despite them being defined in ietf-acl-tls.

**Fix Direction:**

Revise the documentation to correctly state that spki-pin-set and certificate-authority are defined in the ietf-acl-tls module.


**Severity:** Low
  *Basis:* This is a documentation error that may confuse implementers or IANA, but does not affect actual protocol operation.

**Confidence:** High

---
