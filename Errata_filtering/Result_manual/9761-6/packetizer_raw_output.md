# Errata Reports

Total reports: 3

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