# Errata Reports

Total reports: 1

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