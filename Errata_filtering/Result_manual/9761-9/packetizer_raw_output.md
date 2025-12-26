# Errata Reports

Total reports: 1

---

## Report 1: 9761-9-1

**Label:** NACM default-deny-write Claimed but Not Implemented in ietf-acl-tls and ietf-mud-tls Modules

**Bug Type:** Inconsistency

**Explanation:**

Sections 9.3 and 9.4 assert that the NACM extension 'default-deny-write' is applied to all writable data nodes in the ietf-acl-tls and ietf-mud-tls modules, yet neither module imports the required NACM module nor applies any nacm:default-deny-write annotations.

**Justification:**

- The ietf-acl-tls module imports iana-tls-profile, ietf-crypto-types, and ietf-access-control-list but does not import ietf-netconf-acm or include any nacm:default-deny-write annotations, leaving its sensitive nodes unprotected (E1).
- The ietf-mud-tls module imports only ietf-mud and defines a single writable leaf without any NACM extensions, contradicting the claimed security behavior (E2).
- The security text in Sections 9.3 and 9.4 explicitly states that 'the NACM extension "default-deny-write" has been set for all data nodes defined in this module,' which is inconsistent with the actual YANG module definitions (E3).

**Evidence Snippets:**

- **E1:**

  ietf-acl-tls module imports iana-tls-profile, ietf-crypto-types, and ietf-access-control-list, but not ietf-netconf-acm, and contains no nacm:-prefixed extensions in any data definition; the augment defines container client-profiles and list tls-dtls-profile with multiple config true leaves/leaf-lists (e.g., supported-tls-version, cipher-suite, accept-list-ta-cert)  .

- **E2:**

  ietf-mud-tls module imports only ietf-mud and defines one config true leaf is-tls-dtls-profile-supported, again with no NACM import or nacm: extensions  .

- **E3:**

  Section 9.3: “For instance, the addition or removal of references to trusted anchors, (D)TLS versions, cipher suites, etc., can dramatically alter the implemented security policy. For this reason, the NACM extension "default-deny-write" has been set for all data nodes defined in this module.” and Section 9.4: “For instance, update that the device does not support (D)TLS profile can dramatically alter the implemented security policy. For this reason, the NACM extension "default-deny-write" has been set for all data nodes defined in this module.”  

**Evidence Summary:**

- (E1) Evidence from ietf-acl-tls module showing absence of NACM annotations.
- (E2) Evidence from ietf-mud-tls module showing absence of NACM annotations.
- (E3) Security text claims that the NACM default-deny-write extension is applied to all data nodes.

**Fix Direction:**

Either update the YANG modules to import ietf-netconf-acm and annotate the relevant data definitions with nacm:default-deny-write, or modify the security text in Sections 9.3 and 9.4 to accurately reflect the implemented behavior.


**Severity:** High
  *Basis:* The mismatch could lead operators to assume stronger, node-specific write protections than are actually enforced, risking unintended access control failures.

**Confidence:** High

---
