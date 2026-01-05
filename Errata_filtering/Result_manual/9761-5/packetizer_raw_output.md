# Errata Reports

Total reports: 2

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