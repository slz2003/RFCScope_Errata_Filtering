# Errata Reports

Total reports: 2

---

## Report 1: 9710-4-1

**Label:** collectorTransportPort future-protocol clause uses incorrect port direction

**Bug Type:** Inconsistency

**Explanation:**

The collectorTransportPort Information Element (IE) incorrectly references '16‑bit source port identifiers' for future transport protocols, even though its definition clearly specifies it as a destination port identifier.

**Justification:**

- The NEW description in Section 4.4.2 states the field is the destination port identifier but then mentions future protocols having 16‑bit source port identifiers, which conflicts with its intended role.
- Parallel IEs such as destinationTransportPort correctly refer to '16‑bit destination port identifiers', highlighting the editorial inconsistency.

**Evidence Snippets:**

- **E1:**

  Section 4.4.2 NEW description for collectorTransportPort: “The destination port identifier to which the Exporting Process sends Flow information. For transport protocols such as UDP, TCP, and SCTP, this is the destination port number. This field MAY also be used for future transport protocols that have 16-bit source port identifiers.”

- **E2:**

  Section 4.2.2 NEW description for destinationTransportPort: “This field MAY also be used for future transport protocols that have 16-bit destination port identifiers.”

- **E3:**

  Terminology Expert noted the same issue where the use of 'source' in collectorTransportPort is a clear copy‑paste error given its designation as a destination port.

**Evidence Summary:**

- (E1) shows the incorrect use of 'source port identifiers' in the collectorTransportPort description.
- (E2) demonstrates the correct reference in a similar IE (destinationTransportPort).
- (E3) confirms the issue as an editorial copy‑paste error.

**Fix Direction:**

Replace '16-bit source port identifiers' with '16-bit destination port identifiers' in the collectorTransportPort description.


**Severity:** Low
  *Basis:* This is an editorial inconsistency that does not affect the underlying operational or encoding behavior, but could cause confusion during implementation.

**Confidence:** High

---

## Report 2: 9710-4-2

**Label:** forwardingStatus IE ambiguous handling of high-order bits (bits 8–31)

**Bug Type:** Underspecification

**Explanation:**

The forwardingStatus IE update specifies that only the least-significant byte is structured and should be exported using reduced‑size encoding, but it fails to clarify how the remaining bits of the 32‑bit unsigned integer are to be handled.

**Justification:**

- The text explains that a structure is associated only with the least-significant byte and that future versions may define meanings for the remaining bits, yet does not instruct whether exporters should set bits 8–31 to zero or if collectors must ignore them.
- The phrase 'exported as unsigned8' conflates the abstract type with the reduced-size encoding, creating a semantic grey area regarding the treatment of the high-order bits.

**Evidence Snippets:**

- **E4:**

  NotedAmbiguities: For forwardingStatus, the text notes that only the least-significant byte is structured and that future versions may be defined to associate meanings with the remaining bits. However, it does not explicitly state what current exporters/collectors must do with bits 8–31 (set to zero vs. ignore), leaving a small semantic grey area.

- **E5:**

  Causal Analysis observed: 'The wording “exported as unsigned8” is a mild causal underspecification in wording (conflating encoding size with abstract type) but not enough to cause real breakage', implying ambiguity in the handling of the full 32-bit value.

**Evidence Summary:**

- (E4) highlights the lack of explicit instructions for the handling of bits 8–31 in the forwardingStatus IE.
- (E5) points out that the ambiguous phrase 'exported as unsigned8' creates potential misinterpretation regarding the full 32‑bit value.

**Fix Direction:**

Clarify the forwardingStatus specification by explicitly stating that exporters SHOULD set bits 8–31 to zero and that collectors SHALL ignore these bits.


**Severity:** Low
  *Basis:* The ambiguity is limited to wording and does not impact interoperability in compliance with IPFIX reduced‑size encoding rules, though it may lead to minor implementation differences.

**Confidence:** High

---
