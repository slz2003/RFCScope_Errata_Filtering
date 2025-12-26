# Errata Reports

Total reports: 5

---

## Report 1: 9815-10-1

**Label:** Ambiguous EoR max-wait timer expiration behavior for advertising Link NLRI

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define the action required when the maximum wait timer for an EoR expires, which may lead to different implementations taking divergent actions.

**Justification:**

- Section 10.4 mandates that a timer specifying the maximum time to wait may be configured, yet it does not normatively specify what should happen when the timer expires.
- The lack of explicit post-timer expiry instructions could result in implementations either advertising Link NLRIs immediately, resetting the session, or continuing to wait indefinitely.

**Evidence Snippets:**

- **E1:**

  An End-of-RIB (EoR) marker (Section 5.3) for the BGP-LS-SPF SAFI MAY be required from a peer prior to advertising the BGP-LS-SPF Link NLRI for the corresponding link to that peer. When required, the default is to wait indefinitely for the EoR marker prior to advertising the BGP-LS-SPF Link NLRI. Refer to Section 10.4.

- **E2:**

  The usage of the EoR marker [RFC4724] with the BGP-LS-SPF SAFI is somewhat different than the other BGP SAFIs. Reception of the EoR marker MAY optionally be expected prior to advertising a Link NLRI for a given peer.

- **E3:**

  Depending on the peering model, topology, and convergence requirements, an EoR marker (Section 5.3) for the BGP-LS-SPF SAFI MAY be required from the peer prior to advertising a BGP-LS Link NLRI for the peer. If configuration is supported, this MUST be configurable at the BGP SPF instance level and MUST be configured consistently throughout the BGP SPF routing domain.

- **E4:**

  When this configuration is provided, the default MUST be to wait indefinitely prior to advertising a BGP-LS Link NLRI. Configuration of a timer specifying the maximum time to wait prior to advertisement MAY be provided.

**Evidence Summary:**

- (E1) Section 4.1 outlines that waiting indefinitely for an EoR marker is the default behavior for BGP-LS-SPF Link NLRI.
- (E2) Section 5.3 establishes that the EoR marker is optionally expected prior to advertisement.
- (E3) Section 10.4 describes configuration requirements but does not define the mandatory action after timer expiry.
- (E4) Section 10.4 reiterates the indefinite wait without prescribing a post-timer expiry action.

**Fix Direction:**

Clarify in Section 10.4 with normative language what action must be taken upon expiry of the maximum wait timer when no EoR is received.


**Severity:** Low
  *Basis:* Although the ambiguity might lead to different convergence behaviors, its limited scope reduces the overall impact.

**Confidence:** High

---

## Report 2: 9815-10-2

**Label:** Mislabeling NLRI type: BGP-LS Link NLRI vs BGP-LS-SPF Link NLRI in Section 10.4

**Bug Type:** Inconsistency

**Explanation:**

Section 10.4 inconsistently refers to the NLRI type by using 'BGP-LS Link NLRI' instead of the consistently used 'BGP-LS-SPF Link NLRI', which could confuse implementers about the intended behavior.

**Justification:**

- Sections 4.1 and 5.3 clearly refer to 'BGP-LS-SPF Link NLRI' for the EoR-gated advertisement, establishing the correct nomenclature.
- The use of 'BGP-LS Link NLRI' in Section 10.4 creates a normative inconsistency that might lead to gating of the wrong NLRI type.

**Evidence Snippets:**

- **E1:**

  An End-of-RIB (EoR) marker (Section 5.3) for the BGP-LS-SPF SAFI MAY be required from a peer prior to advertising the BGP-LS-SPF Link NLRI for the corresponding link to that peer. When required, the default is to wait indefinitely for the EoR marker prior to advertising the BGP-LS-SPF Link NLRI. Refer to Section 10.4.

- **E2:**

  Depending on the peering model, topology, and convergence requirements, an EoR marker (Section 5.3) for the BGP-LS-SPF SAFI MAY be required from the peer prior to advertising a BGP-LS Link NLRI for the peer. If configuration is supported, this MUST be configurable at the BGP SPF instance level and MUST be configured consistently throughout the BGP SPF routing domain. When this configuration is provided, the default MUST be to wait indefinitely prior to advertising a BGP-LS Link NLRI. Configuration of a timer specifying the maximum time to wait prior to advertisement MAY be provided.

**Evidence Summary:**

- (E1) Section 4.1 uses 'BGP-LS-SPF Link NLRI' indicating the intended NLRI type for EoR-gated advertisement.
- (E2) Section 10.4 erroneously uses 'BGP-LS Link NLRI', resulting in conflicting normative instructions.

**Fix Direction:**

Replace all occurrences of 'BGP-LS Link NLRI' in Section 10.4 with 'BGP-LS-SPF Link NLRI' to maintain consistency.


**Severity:** Medium
  *Basis:* Being in a normative section, the inconsistency may lead to misinterpretation of which NLRI type is subject to EoR gating, affecting operational behavior.

**Confidence:** High

---

## Report 3: 9815-10-3

**Label:** Misnamed SPF back-off parameter: TIME_TO_LEARN vs TIME_TO_LEARN_INTERVAL in Section 10.5

**Bug Type:** Inconsistency

**Explanation:**

Section 10.5 refers to the SPF back-off parameter as 'TIME_TO_LEARN' rather than the RFC 8405 standard 'TIME_TO_LEARN_INTERVAL', which may lead to nomenclature confusion.

**Justification:**

- The parameter is listed as TIME_TO_LEARN in Section 10.5 even though RFC 8405 consistently uses TIME_TO_LEARN_INTERVAL.
- This mismatch can cause discrepancies in configuration, documentation, or tooling across implementations.

**Evidence Snippets:**

- **E1:**

  If supported, configuration of the INITIAL_SPF_DELAY, SHORT_SPF_DELAY, LONG_SPF_DELAY, TIME_TO_LEARN, and HOLDDOWN_INTERVAL MUST be supported [RFC8405].

- **E2:**

  RFC 8405 Section 6 specifies the delays as INITIAL_SPF_DELAY, SHORT_SPF_DELAY, LONG_SPF_DELAY, TIME_TO_LEARN_INTERVAL, and HOLDDOWN_INTERVAL.

**Evidence Summary:**

- (E1) Section 10.5 uses TIME_TO_LEARN as the parameter name.
- (E2) RFC 8405 clearly denotes the parameter as TIME_TO_LEARN_INTERVAL.

**Fix Direction:**

Replace 'TIME_TO_LEARN' with 'TIME_TO_LEARN_INTERVAL' in Section 10.5 for consistency with RFC 8405.


**Severity:** Low
  *Basis:* The issue is minor and editorial in nature, with limited risk of operational impact.

**Confidence:** High

---

## Report 4: 9815-10-4

**Label:** Ambiguous term 'BGP SPF instance level' in Section 10.4 configuration guidance

**Bug Type:** Underspecification

**Explanation:**

The phrase 'BGP SPF instance level' is used in Section 10.4 without a clear definition, leading to uncertainty about the intended scope for configuration.

**Justification:**

- The specification does not clarify whether 'instance level' refers to the BGP process, AFI/SAFI, BGP‑LS Instance‑ID, or VRF, which can cause inconsistent configuration interpretation.

**Evidence Snippets:**

- **E1:**

  The term “BGP SPF instance level” is not defined elsewhere in RFC 9815, so the exact object to which “instance‑level” configuration and consistency requirements apply (per BGP process, per AFI/SAFI, per BGP‑LS Instance‑ID, per VRF, etc.) is not explicit in the specification.

**Evidence Summary:**

- (E1) The undefined nature of 'BGP SPF instance level' creates an ambiguity regarding its intended configuration boundary.

**Fix Direction:**

Provide a clear definition of 'BGP SPF instance level' in the specification, specifying whether it applies at the BGP process, AFI/SAFI, BGP‑LS Instance‑ID, or VRF level.


**Severity:** Low
  *Basis:* While primarily affecting configuration clarity, the ambiguity may lead to minor interoperability or management issues.

**Confidence:** Medium

---

## Report 5: 9815-10-5

**Label:** Ambiguity in Section 10.2: Mixing prefix and link metric configuration

**Bug Type:** Underspecification

**Explanation:**

Section 10.2 conflates prefix and link metrics by discussing loopback/non–loopback prefixes alongside link IGP metrics, potentially confusing operators about the intended scope of metric configuration.

**Justification:**

- The section is titled 'Link Metric Configuration', yet its first paragraph discusses loopback and non‑loopback prefixes (governed by Prefix Metric TLV 1155), while later guidance deals with link IGP metrics (TLV 1095).
- This mix creates ambiguity regarding whether both types of metrics are subject to the same domain-wide rules.

**Evidence Snippets:**

- **E1:**

  Section 10.2 is titled “Link Metric Configuration”, but its first paragraph talks explicitly about loopback and non‑loopback prefixes, which are governed by Prefix Metric TLV 1155 rather than the IGP Metric TLV 1095 on links; the section then ends with guidance that is explicitly about link metrics. The exact scope of “metric” in the first paragraph vs “IGP metrics for all advertised links” could be clearer.

**Evidence Summary:**

- (E1) The title and content of Section 10.2 mix prefix metrics and link IGP metrics, creating an unclear scope for the configuration recommendations.

**Fix Direction:**

Revise Section 10.2 to clearly differentiate and separately address prefix metric recommendations and link IGP metric configuration.


**Severity:** Low
  *Basis:* This is an editorial clarity issue which may cause confusion but has limited impact on protocol behavior.

**Confidence:** Medium

---
