# Errata Reports

Total reports: 2

---

## Report 1: 9868-9-1

**Label:** Inconsistent Checksum Invariant: Section 8 vs. Section 9 Description

**Bug Type:** Inconsistency

**Explanation:**

The specification contains conflicting descriptions of the checksum invariant: Section 8 claims that the surplus area (including alignment bytes and OCS) sums to zero, while Section 9 defines the OCS as computed over the surplus area plus a 16‑bit surplus-length field, implying that the surplus area alone sums to the one’s‑complement of the length.

**Justification:**

- Section 8 states that alignment bytes and OCS ensure that the one's complement sum of the surplus area is zero.
- Section 9 specifies that OCS is computed over the surplus area including a 16‑bit unsigned length, so the surplus alone sums to ~Length rather than zero.

**Evidence Snippets:**

- **E1:**

  Section 8: “These alignment bytes, coupled with OCS as computed over the remainder of the surplus area, ensure that the one's complement sum of the surplus area is zero.”

- **E2:**

  Section 9: “The OCS consists of a 16-bit Internet checksum [RFC1071], computed over the surplus area and including the length of the surplus area as an unsigned 16-bit value. … Because the OCS is computed over the surplus area and its length and then inverted, the OCS effectively negates the effect that incorrectly including the surplus has on the UDP checksum. As a result, when OCS is non-zero, the UDP checksum is the same in either case.”

**Evidence Summary:**

- (E1) shows the claim that the surplus area (by itself) must sum to zero.
- (E2) demonstrates that Section 9’s definition includes the surplus length in the checksum computation, leading to a different invariant.

**Fix Direction:**

Clarify Section 8 to explicitly state that the invariant applies to the combination of the 16-bit surplus-length field and the surplus area, ensuring that their one’s complement sum equals 0xFFFF (the negative zero), which aligns with the formal definition in Section 9.


**Severity:** High
  *Basis:* This inconsistency may lead implementers to follow different interpretations, resulting in divergent OCS computations and potential interoperability failures, especially regarding middlebox checksum recalculations.

**Confidence:** High

---

## Report 2: 9868-9-2

**Label:** Underspecified Handling for Computed OCS Value of Zero When UDP Checksum is Non-Zero

**Bug Type:** Underspecification

**Explanation:**

The document does not specify how to handle the case when the computed 16-bit Internet checksum for OCS naturally evaluates to 0x0000 while the UDP checksum is non-zero, even though it mandates that OCS MUST be non-zero in such cases.

**Justification:**

- Section 9 requires that 'The OCS MUST be non-zero when the UDP checksum is non-zero' and uses a value of zero to indicate an unused checksum.
- There is no normative rule to remap a naturally computed zero (0x0000) to a non-zero value (such as 0xFFFF), leaving ambiguity in sender behavior and causing potential receiver discrepancies.

**Evidence Snippets:**

- **E1:**

  Section 9: “The OCS consists of a 16-bit Internet checksum [RFC1071]…” and “>> The OCS MUST be non-zero when the UDP checksum is non-zero.”

- **E2:**

  Section 9: “When not used (i.e., containing zero), the OCS is assumed to be ‘correct’ for the purpose of accepting UDP datagrams at a receiver (see Section 14).”

**Evidence Summary:**

- (E1) indicates the requirement for a non-zero OCS when the UDP checksum is non-zero.
- (E2) shows that a zero value signals that OCS is not in use, thus creating ambiguity when the computed checksum is zero.

**Fix Direction:**

Introduce an explicit mapping rule (for example, mapping a computed 0x0000 to an on‑wire value of 0xFFFF) for cases where the checksum naturally evaluates to zero while the UDP checksum is non-zero, analogous to the handling in UDP/TCP protocols.


**Severity:** High
  *Basis:* Without a defined rule, rare packets with a naturally computed zero OCS may be inconsistently handled between different implementations, leading to misinterpretation of OCS usage and potential loss of option processing.

**Confidence:** High

---
