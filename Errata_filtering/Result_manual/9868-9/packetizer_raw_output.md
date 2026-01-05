# Errata Reports

Total reports: 1

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