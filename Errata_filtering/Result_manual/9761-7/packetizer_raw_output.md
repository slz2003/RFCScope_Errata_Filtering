# Errata Reports

Total reports: 3

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