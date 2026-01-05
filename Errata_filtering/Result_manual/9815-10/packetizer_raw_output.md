# Errata Reports

Total reports: 2

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