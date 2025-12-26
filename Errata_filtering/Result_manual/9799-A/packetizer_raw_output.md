# Errata Reports

Total reports: 4

---

## Report 1: 9799-A-1

**Label:** Ambiguous Actor Role for Hidden Service Descriptor Re-Signing and Challenge Delay

**Bug Type:** Underspecification

**Explanation:**

Section 4 uses the term 'client' ambiguously, making it unclear whether the ACME client or the Hidden Service (Tor) component is responsible for re-signing and republishing the Hidden Service Descriptor and delaying the challenge response.

**Justification:**

- The text indicates that if the Ed25519 public key is novel, the 'client' must resign and republish its descriptor and then wait for propagation, but the responsibilities of re-signing versus delaying the challenge are assigned to different operational roles without clear distinction.

**Evidence Snippets:**

- **E1:**

  In the case in which the Ed25519 public key is novel to the client, it will have to resign and republish its Hidden Service Descriptor. It MUST wait some (indeterminate) amount of time for the new descriptor to propagate the Tor Hidden Service directory servers before proceeding with responding to the challenge.

- **E2:**

  Within this RFC, “client” generally refers to the ACME client, which is the software sending ACME requests. However, the act of re-signing and republishing a Hidden Service Descriptor is functionally performed by the Hidden Service / Tor component (triggered by the Hidden Service operator), not by the ACME client per se.

**Evidence Summary:**

- (E1) Describes the requirement for re-signing and waiting, highlighting ambiguous actor responsibilities.
- (E2) Explains the overloaded use of the term 'client' and the resulting role ambiguity.

**Fix Direction:**

Clarify the text by explicitly assigning the re-signing and republishing of the Hidden Service Descriptor to the Hidden Service (or its operator) and specifying that the ACME client is separately responsible for delaying its challenge response.


**Severity:** Medium
  *Basis:* While in practical deployments the same operator controls both roles, the ambiguity may lead to divergent implementations and confusion.

**Confidence:** High

---

## Report 2: 9799-A-2

**Label:** Directory metadata field name mismatch (inBandOnionCAARequired vs onionCAARequired)

**Bug Type:** Inconsistency

**Explanation:**

The RFC inconsistently defines the JSON field name used to signal in-band CAA requirements by using 'inBandOnionCAARequired' in the normative text and examples, while registering 'onionCAARequired' in the IANA registry.

**Justification:**

- Section 6.4.1 defines and exemplifies the directory meta field as 'inBandOnionCAARequired', whereas Section 7.3 registers the field name as 'onionCAARequired', leading to potential mismatches in implementation.
- This inconsistency can cause clients and servers to use different field names, resulting in erroneous conclusions about whether in-band CAA data is required.

**Evidence Snippets:**

- **E1:**

  Section 6.4.1 explicitly defines a new directory meta field and gives an example using the name inBandOnionCAARequired: “To support signaling the server's support for fetching CAA record sets over Tor, a new field is defined in the directory "meta" object to signal this. inBandOnionCAARequired (optional, boolean): If true, the ACME server requires the client to provide the CAA record set in the finalize request. If false or absent, the ACME server does not require the client to provide the CAA record set is this manner.” and the directory example uses "inBandOnionCAARequired": true

- **E2:**

  Section 7.3 registers a different field name in the IANA “ACME Directory Metadata Fields” registry: | Field name       | Field type | Reference | | onionCAARequired | boolean    | RFC 9799  |

**Evidence Summary:**

- (E1) Normative text and examples define and use 'inBandOnionCAARequired' as the directory meta field.
- (E2) The IANA registration lists the field name as 'onionCAARequired', creating a naming conflict.

**Fix Direction:**

Align the IANA registration in Section 7.3 with Section 6.4.1 by using the same field name (e.g., change the registered name to 'inBandOnionCAARequired') to ensure a unique, canonical JSON member name.


**Severity:** High
  *Basis:* Because JSON field names are used literally in implementations, this mismatch can cause clients and servers to misinterpret the in-band CAA requirement, leading to interoperability failures.

**Confidence:** High

---

## Report 3: 9799-A-3

**Label:** Conflicting issuance requirements: caa-critical flag versus in-band CAA bypass

**Bug Type:** Inconsistency

**Explanation:**

The RFC presents conflicting rules by mandating that issuance must not proceed when a caa-critical flag is encountered (requiring descriptor-based CAA checks) while also permitting issuance based solely on a valid in-band CAA record set.

**Justification:**

- Section 6.3 requires that if a caa-critical flag is present, the server MUST NOT issue until the descriptor CAA records are decrypted and parsed.
- Section 6.4 allows the server to proceed with issuance based solely on a valid in-band CAA record set, potentially permitting bypass of the descriptor check, even when caa-critical is set.

**Evidence Snippets:**

- **E1:**

  If an ACME server encounters this flag, it MUST NOT proceed with issuance until it can decrypt and parse the CAA records from the Second Layer Hidden Service Descriptor.

- **E2:**

  If an ACME server receives a validly signed CAA record set in the finalize request, it MAY proceed with issuance on the basis of the client-provided CAA record set only, without checking the CAA set in the Hidden Service Descriptor.

**Evidence Summary:**

- (E1) Enforces a strict requirement to check descriptor-based CAA when caa-critical is present.
- (E2) Permits bypassing the descriptor check if in-band CAA data is provided, creating a conflict.

**Fix Direction:**

Clarify the normative text so that the in-band CAA bypass does not apply when a caa-critical flag is present, ensuring that descriptor-based CAA checks remain mandatory in such cases.


**Severity:** High
  *Basis:* This conflict may lead to divergent behaviors in issuance, undermining the intended security guarantees provided by the caa-critical flag.

**Confidence:** High

---

## Report 4: 9799-A-4

**Label:** Incorrect CA/B Forum BR reference in Appendix A for HTTP/tls-alpn Validation Methods

**Bug Type:** Inconsistency

**Explanation:**

The RFC misreferences CA/B Forum Baseline Requirements by citing Appendix B.2.a.ii as defining allowed http-01 or tls-alpn-01 validation methods, while that section actually deals with certificate policy object identifiers.

**Justification:**

- Appendix A states that “Appendix B.2.a.ii of [cabf-br] defines, and this document allows, using the http-01 or tls-alpn-01 validation methods”, which suggests these methods are justified by CA/B Forum rules.
- However, summaries of cabf-br indicate that Section B.2.a.ii pertains to the inclusion of policy identifiers in certificates, not the validation methods relevant to .onion services, leading to confusion for implementers and auditors.

**Evidence Snippets:**

- **E1:**

  Appendix A states: “Appendix B.2.a.ii of [cabf-br ] defines, and this document allows, using the http-01 or tls-alpn-01 validation methods already present in ACME (with some considerations).”

- **E2:**

  The provided summary of [cabf-br] says: “Section B.2.a.ii of this document pertains to the "Certificate Policy Object Identifier" within the "Certificate Profile" section. This section specifies the requirements for the inclusion of policy identifiers in certificates, ensuring that they accurately reflect the CA's adherence to the Baseline Requirements.”

**Evidence Summary:**

- (E1) RFC 9799 Appendix A incorrectly cites B.2.a.ii as defining validation methods.
- (E2) The CA/B Forum summary indicates that B.2.a.ii is actually about certificate policy object identifiers.

**Fix Direction:**

Correct the cross-reference in Appendix A by either updating the subsection reference to the proper part of the CA/B Forum Baseline Requirements that addresses allowed validation methods or by revising the descriptive text to accurately reflect the content of B.2.a.ii.


**Severity:** Low
  *Basis:* This misreference is informational and does not affect protocol operation, but it may mislead implementers and compliance auditors regarding the applicable CA/B Forum guidelines.

**Confidence:** Medium

---
