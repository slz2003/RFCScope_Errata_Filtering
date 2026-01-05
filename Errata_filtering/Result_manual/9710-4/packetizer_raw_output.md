# Errata Reports

Total reports: 1

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