# Errata Reports

Total reports: 4

---

## Report 1: 9844-3-1

**Label:** Ambiguous RFC7622/8089 Update: Unclear patch instructions for deleting RFC6874 references

**Bug Type:** Underspecification

**Explanation:**

RFC 9844’s update clause for RFC 7622 and RFC 8089 by deleting RFC6874 references is ambiguous because it lacks explicit patch text, leaving the treatment of IPv6 zone identifiers in IP-literal syntax (especially for JIDs) unclear.

**Justification:**

- No explicit patch text is provided to remove notes relying on RFC6874's modifications, which could lead to conflicting interpretations about whether the normative text should revert to RFC3986.
- Both the Scope and Structural analyses highlight uncertainty about whether the change affects only the reference lists or also modifies key ABNF and implementation notes.
- Residual uncertainty remains regarding whether any in-body text in RFC 8089 should be altered.

**Evidence Snippets:**

- **E1:**

  RFC 9844 Section 3: “This document completely obsoletes [RFC6874]… Note that obsoleting [RFC6874] reverts the change that it made to the URI syntax defined by [RFC3986]… This document also updates [RFC7622] and [RFC8089] by deleting their references to [RFC6874].”

- **E2:**

  RFC 7622 excerpt: RFC 6874 appears only in the Normative References list; the domainpart rules describe IPv4/IPv6 addresses and FQDNs but do not mention zone identifiers explicitly or tie domainpart syntax to RFC6874.

- **E3:**

  RFC 8089 excerpt: RFC 6874 similarly appears in the Normative References list; no explicit patch in RFC9844 specifies whether any other occurrences (if any exist in the body of RFC8089) are to be changed.

- **E4:**

  RFC 7622 JID ABNF and note:  domainpart   = IP-literal / IPv4address / ifqdn ; the 'IPv4address' and 'IP-literal' rules are defined in RFCs 3986 and 6874, respectively... Implementation Note: “Reuse of the IP-literal rule from [RFC6874] … Also note that the IP-literal rule was updated between [RFC3986] and [RFC6874] to optionally add a zone identifier to any literal address.”

- **E5:**

  RFC 3986 host ABNF: IP-literal = "[" ( IPv6address / IPv6future ) "]" followed by: “This syntax does not support IPv6 scoped addressing zone identifiers.”

**Evidence Summary:**

- (E1) RFC 9844 Section 3 declares the obsolescence of RFC6874 and states that RFC7622/8089 are updated by deleting its references.
- (E2) RFC 7622 uses RFC6874 only in its Normative References list, leaving its normative text ambiguous.
- (E3) RFC 8089 similarly displays RFC6874 only bibliographically with no explicit in-text patch.
- (E4) The JID ABNF in RFC7622 still includes commentary on RFC6874-based modifications, raising uncertainty about the intended syntax.
- (E5) RFC 3986’s IP-literal syntax does not support scoped zone identifiers, deepening the ambiguity of the update.

**Fix Direction:**

Provide explicit, patch-style instructions in RFC 9844 to revise the ABNF and implementation notes in RFC 7622 (and any affected parts of RFC 8089) so that the intended normative syntax for IPv6 addresses is unambiguous.

**Severity:** Medium
  *Basis:* The lack of precise revision instructions could lead to divergent interpretations among implementers, as noted by both Scope and Structural Experts.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Structural Expert: Issue-1

---

## Report 2: 9844-3-2

**Label:** Ambiguous Scope of UI Requirement Update to RFC4007

**Bug Type:** Underspecification

**Explanation:**

RFC 9844 introduces a UI requirement for entering IPv6 addresses but simultaneously excludes devices lacking human‐readable zone identifiers, creating ambiguity about which RFC4007 implementations must comply.

**Justification:**

- The introduction explicitly excludes devices that do not support human‐readable zone identifiers, yet Section 3 broadly claims an update to RFC4007 by adding a UI requirement.
- This discrepancy can lead implementers to diverge on whether the new UI requirement applies to all RFC4007-based systems or only to those with user interfaces.

**Evidence Snippets:**

- **E1:**

  RFC 9844 Introduction: “Devices whose network stack does not support the model of a human-readable zone identifier described in [RFC4007] are out of scope for this document.”

- **E2:**

  RFC 9844 Section 3: “It also updates [RFC4007] by adding a new requirement that user interfaces support the zone identifier as described in Section 5.”

- **E3:**

  RFC 9844 Section 5: “A user interface (UI) that allows or requires the user to enter an IPv6 address other than a global unicast address MUST provide a means for entering a link-local address or a scoped multicast address and selecting a zone identifier as specified by [RFC4007]…”

**Evidence Summary:**

- (E1) The introduction limits the document’s scope to devices that support human-readable zone identifiers.
- (E2) Section 3 states an update to RFC4007 by adding a UI requirement without reiterating the scope limitation.
- (E3) Section 5 defines the UI requirement as mandatory for certain user interfaces, without reference to the exclusion in the introduction.

**Fix Direction:**

Clarify in the text of RFC 9844 that the UI requirement update to RFC4007 applies only to systems that both support human-readable zone identifiers and provide a user interface for address entry.

**Severity:** Medium
  *Basis:* The conflicting scope statements may lead to divergent interpretations among implementers regarding which devices or systems are subject to the new UI requirement.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2

---

## Report 3: 9844-3-3

**Label:** Editorial Inconsistency in Numeric Zone Identifier Phrasing

**Bug Type:** Editorial

**Explanation:**

The phrasing in RFC 9844 regarding the use of a decimal integer as a replacement for a human‐readable zone identifier is informal and could be misinterpreted compared to the normative language of RFC4007 Section 11.2.

**Justification:**

- RFC 9844 informally describes using the underlying interface number as a decimal integer, potentially leading to confusion about whether it is equivalent to the normative 'non-negative decimal integer' requirement in RFC4007.
- The disparity in tone between the documents may result in inconsistencies in interpretation, even though the intended behavior remains aligned.

**Evidence Snippets:**

- **E1:**

  RFC 9844 Section 3: “in most operating systems, it is possible to use the underlying interface number, represented as a decimal integer, as an equivalent to the human-readable string. This is recommended by Section 11.2 of [RFC4007], but it is not required.”

- **E2:**

  RFC 4007 Section 11.2: “For example, to represent link index 2, the implementation can simply use ‘2’ as <zone_id>… An implementation SHOULD support at least numerical indices that are non-negative decimal integers as <zone_id>.”

**Evidence Summary:**

- (E1) RFC 9844 suggests the use of a decimal integer for the underlying interface number as an equivalent to a human-readable zone identifier.
- (E2) RFC 4007 Section 11.2 normatively specifies that implementations SHOULD support non-negative decimal integers as zone identifiers.

**Fix Direction:**

Update the language in RFC 9844 to adopt a more normative tone that clearly aligns with RFC4007’s specification regarding numeric zone identifiers.

**Severity:** Low
  *Basis:* The issue is editorial in nature and does not affect interoperability, but clarifying the language would prevent potential misinterpretation.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-2

---

## Report 4: 9844-3-4

**Label:** Meta Clarification: Normative Relationship Declarations as Process Statements

**Bug Type:** None

**Explanation:**

The statements in RFC 9844 that declare it obsoletes and updates previous RFCs serve as meta-process declarations and do not impose additional normative requirements.

**Justification:**

- Deontic analysis indicates that the obsoletes and updates statements align with the established normative content and are intended as process/meta declarations.
- The excerpts from RFC 7622 and RFC 8089 further support that these changes are largely bibliographic and do not affect protocol operations.

**Evidence Snippets:**

- **E1:**

  “This document completely obsoletes [RFC6874] … and replaces it with a generic UI requirement.”

- **E2:**

  “Note that obsoleting [RFC6874] reverts the change that it made to the URI syntax defined by [RFC3986], so [RFC3986] is no longer updated by [RFC6874].”

- **E3:**

  “This document also updates [RFC7622] and [RFC8089] by deleting their references to [RFC6874].”

- **E4:**

  “It also updates [RFC4007] by adding a new requirement that user interfaces support the zone identifier as described in Section 5.”

- **E5:**

  RFC 7622 §3.2 plus its Normative References list including RFC 3986 and RFC 6874.

- **E6:**

  RFC 8089 §7.1 Normative References including RFC 3986 and RFC 6874.

**Evidence Summary:**

- (E1-E4) Section 3 of RFC 9844 makes declarative statements about obsoleting and updating prior RFCs.
- (E5-E6) The excerpts from RFC 7622 and RFC 8089 indicate that these changes are mostly bibliographic and do not alter the underlying protocol behavior.

**Severity:** Low
  *Basis:* The statements are intended as meta-process declarations and, as clarified by the Deontic Expert, do not introduce contradictory normative obligations.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-1

---
