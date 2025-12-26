# Errata Reports

Total reports: 6

---

## Report 1: 9761-6-1

**Label:** Ambiguous GREASE Handling and Recognized/Unrecognized Terminology for TLS Parameters

**Bug Type:** Underspecification / Terminology

**Explanation:**

The specification ambiguously defines how GREASE values should be treated, allowing them to be classified as either recognized or unrecognized, which may lead to blocking of GREASE values contrary to the intended behavior of ignoring them.

**Justification:**

- Section 5 mandates that MUD (D)TLS profiles MUST NOT include GREASE values, yet Section 6 distinguishes between recognized and unrecognized parameters without clarifying that GREASE values must be treated as unrecognized.
- Multiple expert analyses (Scope, Deontic, CrossRFC, Terminology, Boundary) note that a firewall that is aware of GREASE codepoints may treat these as recognized and thus trigger blocking or alerts, undermining GREASE’s purpose.

**Evidence Snippets:**

- **E1:**

  the (D)TLS profile parameters defined in the YANG module by this document MUST NOT include the GREASE values for extension types, named groups, signature algorithms, (D)TLS versions, pre-shared key exchange modes, cipher suites, and any other TLS parameters defined in future RFCs.

- **E2:**

  If the (D)TLS parameter observed in a (D)TLS session is not specified in the MUD (D)TLS profile and the parameter is recognized by the firewall, it can identify unexpected (D)TLS usage, which can indicate the presence of unauthorized software or malware… The firewall can take several actions, such as blocking the (D)TLS session or raising an alert…

- **E3:**

  This rule also ensures that the network security service will ignore the GREASE values advertised by TLS peers and interoperate with the implementations advertising GREASE values.

**Evidence Summary:**

- (E1) mandates exclusion of GREASE values from the profile.
- (E2) defines behavior for recognized parameters not in the profile.
- (E3) claims that GREASE values will be ignored, creating an ambiguity.

**Fix Direction:**

Amend Section 6 to explicitly state that GREASE values, even if numerically recognized, MUST be treated as unrecognized for the purposes of enforcement, and provide a clear definition of 'recognized' for TLS parameters.


**Severity:** High
  *Basis:* Blocking GREASE values can lead to ossification and interoperability failures, directly contradicting the intended extensibility benefits of GREASE.

**Confidence:** High

---

## Report 2: 9761-6-2

**Label:** Underspecified Enforcement for Profile-Listed but Unrecognized TLS Parameters

**Bug Type:** Underspecification

**Explanation:**

The specification fails to provide normative guidance on how to handle live traffic when a TLS parameter is listed in the MUD (D)TLS profile but is not recognized by the firewall.

**Justification:**

- Section 6 bullet 3 gives an example by triggering an alert for newer profile parameters that the firewall does not recognize but does not state whether the session should be allowed, blocked, or handled differently.
- Both Causal and Deontic experts highlight that this lack of explicit behavior may lead to inconsistent implementations and potential ossification as firewalls might block valid sessions.

**Evidence Snippets:**

- **E1:**

  If the firewall does not recognize the newer parameters [in the updated MUD (D)TLS profile], an alert should be triggered to the firewall vendor and the IoT device owner or administrator. A firewall must be readily updatable so that when new parameters in the MUD (D)TLS profile are discovered that are not recognized by the firewall, it can be updated quickly. … For example, if the cipher suite TLS_AES_128_CCM_8_SHA256 specified in the MUD (D)TLS profile is not recognized by the firewall, an alert will be triggered.

- **E2:**

  it never says whether to allow or block live traffic using those parameters; It does not say “MUST allow” such sessions.

**Evidence Summary:**

- (E1) shows that an alert is triggered for unknown newer parameters without specifying live traffic handling.
- (E2) explicitly points out the lack of normative instruction for sessions using profile-listed but unrecognized parameters.

**Fix Direction:**

Add explicit normative language specifying that if a parameter is present in the MUD (D)TLS profile but not recognized by the firewall, the firewall MUST treat the parameter as unenforceable (i.e., ignore it for admission decisions) while still triggering an alert.


**Severity:** High
  *Basis:* This underspecification can lead to either inadvertent blocking of valid traffic or weak enforcement that undermines the security intent, thereby risking ossification.

**Confidence:** High

---

## Report 3: 9761-6-3

**Label:** Ambiguous Interaction of TLS-Profile Parameter Rules with MUD Default Deny Semantics

**Bug Type:** Both

**Explanation:**

The specification does not clearly reconcile the parameter-level non-blocking rule with RFC 8520’s global default of 'anything not explicitly permitted is denied', potentially resulting in unintended session blocking.

**Justification:**

- RFC 8520 clearly states that any traffic not explicitly permitted must be denied, yet Section 6 suggests that unrecognized parameters should be ignored without clarifying this override.
- Scope Expert notes that without explicit guidance, different implementations may adopt conflicting behaviors that either block or allow traffic inconsistently.

**Evidence Snippets:**

- **E1:**

  RFC 8520 Section 5: “Anything not explicitly permitted is denied. … These are applied AFTER all other explicit rules. Thus, a default behavior can be changed with a ‘drop’ action.”

- **E2:**

  If the (D)TLS parameter observed in a (D)TLS session is not specified in the MUD (D)TLS profile and the (D)TLS parameter is not recognized by the firewall, it can ignore the unrecognized parameter and the correct behavior is not to block the (D)TLS session.

**Evidence Summary:**

- (E1) underscores the default-deny rule from RFC 8520.
- (E2) presents the parameter-level exception without explaining how it interacts with RFC 8520.

**Fix Direction:**

Clarify that for TLS parameters, the non-blocking rule explicitly overrides the default deny rule from RFC 8520 for unrecognized or extra parameters.


**Severity:** Medium
  *Basis:* If not resolved, the conflict may lead to inconsistent firewall behavior and inadvertently block valid sessions.

**Confidence:** High

---

## Report 4: 9761-6-4

**Label:** Misleading Equivalence to RFC 8446 §9.3 Middlebox Invariants

**Bug Type:** Inconsistency

**Explanation:**

The document claims functional equivalence between its handling of unrecognized TLS parameters and the middlebox behavior prescribed in RFC 8446 §9.3, but omits key constraints required by RFC 8446.

**Justification:**

- CrossRFC Expert notes that RFC 8446 §9.3 prohibits further TLS processing after forwarding an unknown ClientHello parameter, a constraint not imposed by RFC 9761.
- This misleading equivalence may cause implementers to design middleboxes that behave non-compliantly with RFC 8446 while believing they are adhering to standards.

**Evidence Snippets:**

- **E1:**

  RFC 8446 §9.3 includes a stronger invariant for non-terminating middleboxes: 'A middlebox which forwards ClientHello parameters it does not understand MUST NOT process any messages beyond that ClientHello. It MUST forward all subsequent traffic unmodified.'

- **E2:**

  RFC 9761’s guidance to ignore unrecognized parameters is stated to be functionally equivalent to the compliant TLS middlebox description in Section 9.3 of [RFC8446] to ignore all unrecognized cipher suites, extensions, and other parameters.

**Evidence Summary:**

- (E1) clarifies the additional constraint in RFC 8446 §9.3.
- (E2) shows the claim of functional equivalence made in RFC 9761 without the additional constraint.

**Fix Direction:**

Revise the text to accurately describe the differences from RFC 8446 §9.3 and remove or qualify the equivalence claim.


**Severity:** Medium
  *Basis:* Incorrect equivalence may lead to design decisions that compromise proper TLS processing and security invariants.

**Confidence:** Medium

---

## Report 5: 9761-6-5

**Label:** JSON MUD Example Naming Inconsistency

**Bug Type:** Inconsistency

**Explanation:**

The JSON example provided in Section 7 does not correctly reflect the YANG node and leaf names defined in the specification, which may cause interoperability and validation issues.

**Justification:**

- Terminology Expert highlights that the JSON example uses incorrect node names such as 'client-profile' instead of 'client-profiles' and 'tls-dtls-profiles' instead of 'tls-dtls-profile'.
- The discrepancy violates the RFC 7951 JSON encoding rules that require exact name matching with the YANG definitions.

**Evidence Snippets:**

- **E1:**

  Section 5.1 tree: +--rw client-profiles {match-on-tls-dtls}? and +--rw tls-dtls-profile* [name]

- **E2:**

  JSON example in Section 7: "ietf-acl-tls:client-profile" : { ... "tls-dtls-profiles" : [ ... ], "supported-tls-versions" : ["tls13"], "extension-types" : [10,11,13,16,24] }

**Evidence Summary:**

- (E1) provides the correct YANG node names.
- (E2) shows the JSON example with naming mismatches that do not align with the YANG definitions.

**Fix Direction:**

Correct the JSON example to use the exact YANG node names: 'client-profiles', 'tls-dtls-profile', 'supported-tls-version', and 'extension-type'.


**Severity:** Medium
  *Basis:* Naming inconsistencies may lead to JSON validation errors and interoperability issues with YANG-based tooling.

**Confidence:** High

---

## Report 6: 9761-6-6

**Label:** Misstatement of Parameter Ownership Between YANG Modules

**Bug Type:** Inconsistency

**Explanation:**

The document incorrectly attributes the spki-pin-set and certificate-authority parameters to the iana-tls-profile module instead of the ietf-acl-tls module, which can cause confusion for implementers.

**Justification:**

- Terminology Expert points out that the introductory text in Section 5.3 incorrectly lists these parameters as part of the iana-tls-profile module even though they are defined in the ietf-acl-tls module.
- This misstatement may mislead implementers regarding where to find the correct definitions for these parameters.

**Evidence Snippets:**

- **E1:**

  Section 5.3 introductory text: “The values for all the parameters in the ‘iana-tls-profile’ YANG module are defined in the TLS and DTLS IANA registries excluding the tls-version, dtls-version, spki-pin-set, and certificate-authority parameters. The values of spki-pin-set and certificate-authority parameters will be specific to the IoT device.”

- **E2:**

  Section 5.3 YANG module iana-tls-profile does not include any typedefs or leafs named spki-pin-set or certificate-authority, whereas Section 5.2 YANG module ietf-acl-tls defines these parameters.

**Evidence Summary:**

- (E1) shows the misstatement in the introductory text regarding parameter ownership.
- (E2) contrasts the actual module definitions between iana-tls-profile and ietf-acl-tls.

**Fix Direction:**

Revise the text in Section 5.3 to correctly attribute the spki-pin-set and certificate-authority parameters to the ietf-acl-tls module.


**Severity:** Low
  *Basis:* The error is primarily editorial and is unlikely to cause significant technical problems, though it may mislead implementers.

**Confidence:** High

---
