# Errata Reports

Total reports: 3

---

## Report 1: 9844-5-1

**Label:** Underspecified Handling of Alternative UI Delimiter for IPv6 Zone Identifiers

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state that alternative delimiter forms (e.g., using '-' instead of '%') are intended solely as a UI input convention and must be normalized to the RFC 4007 canonical '<address>%<zone_id>' form before they are used in APIs or stored in configuration.

**Justification:**

- RFC 9844 Section 5 allows an alternative delimiter (e.g., ‘fe80::1-eth0’) without explicitly clarifying if it is to be used only for UI convenience, leading to potential normalization ambiguities (Scope Expert Issue-1, CrossRFC Expert Issue-2).
- The lack of explicit guidance may lead to inconsistent behavior across implementations, where the alternative delimiter might mistakenly be propagated as a standardized textual representation.

**Evidence Snippets:**

- **E1:**

  RFC 9844 Section 5: “the UI SHOULD support the complete format specified by [RFC4007] (e.g., ‘fe80::1%eth0’). If this is impossible for practical reasons, the UI MAY support an alternative delimiter in place of ‘%’. The hyphen (‘-’) is suggested (e.g., ‘fe80::1-eth0’).”

- **E2:**

  The text that “the zone identifier is considered independently of the IPv6 address itself” hints that the intended model is to separate address and zone internally, but it does not spell out that the alternative-delimiter syntax is purely a UI-facing shorthand that must be normalized (e.g., split into <address> and <zone_id> and then converted to an interface index or the canonical ‘%’ form) before being used with APIs or persisted.

- **E3:**

  RFC 9844 never clearly states that such alternative‑delimiter strings are purely UI‑local and MUST be normalized back into the RFC 4007 ‘%’ form (or to binary indices) before being passed to APIs or reused as textual IPv6 literals.

**Evidence Summary:**

- (E1) shows that an alternative delimiter is allowed without clear normalization instructions.
- (E2) indicates the lack of explicit guidance on the UI-only nature of the alternative delimiter.
- (E3) reinforces that the spec does not mandate normalization to the standardized RFC 4007 format.

**Fix Direction:**

Add explicit language stating that any alternative delimiter is solely a UI convenience and MUST be normalized to the RFC 4007 '<address>%<zone_id>' form (or converted directly to binary representations) before further processing.

**Severity:** Low
  *Basis:* The issue is primarily a clarificatory ambiguity; while it may lead to inconsistent interpretations across implementations, it does not directly impair on‐the‐wire protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-1
- Causal: Alternative Delimiter Discussion
- CrossRFC: Issue-2

---

## Report 2: 9844-5-2

**Label:** Ambiguous Trigger Condition for Zone Identifier Requirement ('address other than a global unicast address')

**Bug Type:** Underspecification

**Explanation:**

The phrase 'address other than a global unicast address' is ambiguous, potentially leading to inconsistent interpretations regarding the inclusion of certain address types such as Unique Local Addresses (ULAs) in the UI requirement for zone identifiers.

**Justification:**

- RFC 9844 Section 5 uses the trigger condition without explicitly linking it to RFC 4007’s distinction between global and non‑global scope, leaving it open to misinterpretation (Scope Expert Issue-2, CrossRFC Expert Issue-1).
- Ambiguity in the terminology may cause implementers to erroneously require zone identifier support for ULA addresses if they misinterpret 'global unicast' as excluding ULAs (Causal Expert Section 2.1).

**Evidence Snippets:**

- **E1:**

  RFC 9844 Section 5: “A user interface (UI) that allows or requires the user to enter an IPv6 address other than a global unicast address MUST provide a means for entering a link-local address or a scoped multicast address and selecting a zone identifier as specified by [RFC4007]….”

- **E2:**

  RFC 4007 Section 4 / 11.1: defines scopes (link-local vs global) and says the <address>%<zone_id> format “applies to all kinds of unicast and multicast addresses of non-global scope … [and] should not be used for global addresses…”

- **E3:**

  IPv6 architecture treats Unique Local Addresses (ULAs) as having global scope, yet some implementers might misinterpret ‘global unicast address’ to exclude ULAs, thereby erroneously applying the UI zone identifier requirement.

**Evidence Summary:**

- (E1) presents the trigger condition wording in RFC 9844 Section 5.
- (E2) outlines the intended scope per RFC 4007 for the textual zone identifier format.
- (E3) highlights the potential for misinterpretation regarding ULAs and the global scope definition.

**Fix Direction:**

Clarify the trigger condition by specifying that the requirement applies to all IPv6 addresses of non-global scope as defined in RFC 4007, and explicitly note that Unique Local Addresses are to be treated as global for the purpose of this requirement.

**Severity:** Low
  *Basis:* While the ambiguity may lead to over-implementation of zone identifier support in some cases, it does not disrupt on-the-wire behavior or API functionality.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2
- Causal: Issue-2.1
- CrossRFC: Issue-1

---

## Report 3: 9844-5-3

**Label:** Ambiguity in the Scope of 'URIs Fetched by Web Browsers' Exemption

**Bug Type:** Ambiguity

**Explanation:**

The exemption for 'URIs fetched by web browsers' is insufficiently defined, leading to uncertainty about which browser-based UIs are exempt from the requirement to support zone identifiers.

**Justification:**

- The document states that the recommendations do not apply to 'URIs fetched by web browsers', but it does not specify whether this exemption covers only the URL bar or all browser-hosted dialogs (Scope Expert ResidualUncertainties).
- This ambiguity may result in inconsistencies in how browser interfaces handle the entry of IPv6 literals with zone identifiers.

**Evidence Snippets:**

- **E1:**

  The carve‑out “URIs fetched by web browsers” is partly orthogonal to the earlier UI‑centric requirement. It is not entirely clear how to treat browser‑provided UIs that allow entry of raw IPv6 literals (without scheme) or browser configuration dialogs that are not strictly URI fetch paths.

**Evidence Summary:**

- (E1) highlights the lack of clarity regarding which browser UIs are covered by the exemption.

**Fix Direction:**

Clarify the exemption by defining the specific browser contexts (e.g., only the URL bar for navigational purposes) that are not subject to the zone identifier requirement.

**Severity:** Low
  *Basis:* This ambiguity is a deployment nuance and is unlikely to have a significant impact on interoperability, but it may lead to varied implementations in browser-related UIs.

**Confidence:** Medium

**Experts mentioning this issue:**

- Scope: Residual Uncertainties

---
