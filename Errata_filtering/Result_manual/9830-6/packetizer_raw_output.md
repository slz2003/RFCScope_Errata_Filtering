# Errata Reports

Total reports: 2

---

## Report 1: 9830-6-1

**Label:** Color-Only Type 3 Inconsistency: Reserved Semantics vs Unassigned in IANA Registry

**Bug Type:** Inconsistency

**Explanation:**

The document defines Color-Only Type 3 as reserved with mandatory receiver behavior in Section 3, yet the IANA registry in Section 6.9 labels it 'Unassigned', creating a contradictory specification.

**Justification:**

- Section 3 explicitly states: Type 3 is 'Reserved for future use and SHOULD NOT be used. Upon reception, an implementation MUST treat it like Type 0.' (E1)
- Section 6.9 lists the same code point as 'Unassigned' in the Color Extended Community registry, conflicting with the normative text (E2)

**Evidence Snippets:**

- **E1:**

  Section 3 defines the Color-Only Type field in the Color Extended Community flags and explicitly gives semantics for Type 3: “Type 3 (bits 11): Reserved for future use and SHOULD NOT be used. Upon reception, an implementation MUST treat it like Type 0.” (Section 3, Color Extended Community)

- **E2:**

  Section 6.9 creates the “Color Extended Community Color-Only Types” registry and Table 9 lists Type 3 as “Unassigned | RFC 9830” (Section 6.9, Table 9)

**Evidence Summary:**

- (E1) Section 3 specifies that Type 3 is reserved and mandates treatment as Type 0.
- (E2) Section 6.9’s IANA registry table marks Type 3 as Unassigned.

**Fix Direction:**

Update the IANA registry in Section 6.9 to indicate that Type 3 is reserved, for example by changing its entry to 'Reserved; on reception treat as Type 0', thereby aligning the registry with the normative text in Section 3.


**Severity:** Medium
  *Basis:* Multiple experts noted that while the normative text clearly defines receiver behavior, the 'Unassigned' label in the registry may mislead implementers and future authors regarding the code point's intended use.

**Confidence:** High

---

## Report 2: 9830-6-2

**Label:** SR Policy Segment Flags Inconsistency: Conflicting B-Flag Bit Position

**Bug Type:** Inconsistency

**Explanation:**

There is a conflict in the bit assignment for the B-Flag: the flag diagram in Section 2.4.4.2.3 indicates one bit position while the IANA registry table in Section 6.8 assigns it to a different bit, potentially leading to misinterpretation of the flag.

**Justification:**

- The diagram in Section 2.4.4.2.3 (Figure 13) shows the B-Flag positioned as implied by the flag layout (bit positioned in the third slot alongside V-Flag) (E1).
- In contrast, the IANA registry in Section 6.8 (Table 8) assigns the B-Flag to bit 3, with bits 1–2 marked as unassigned, creating an encoding conflict (E2).

**Evidence Snippets:**

- **E1:**

  Section 2.4.4.2.3 (SR Policy Segment Flags), Figure 13:

          0 1 2 3 4 5 6 7  
          +-+-+-+-+-+-+-+-+  
          |V|   |B|       |  
          +-+-+-+-+-+-+-+-+

          and the following text:
          The Segment Type sub-TLVs described above may contain the following SR Policy Segment Flags in their Flags field. Also refer to Section 6.8: …
          * When the B-Flag is set, it indicates the presence of the "SRv6 Endpoint Behavior & SID Structure" encoding specified in Section 2.4.4.2.4.

- **E2:**

  Section 6.8 (IANA registry “SR Policy Segment Flags”), Table 8:

          +=====+====================================+===========+
          | Bit | Description                        | Reference |
          +=====+====================================+===========+
          | 0   | Segment Verification Flag (V-Flag) | RFC 9830  |
          | 1-2 | Unassigned                         |           |
          | 3   | SRv6 Endpoint Behavior & SID       | RFC 9830  |
          |     | Structure Flag (B-Flag)            |           |
          | 4-7 | Unassigned                         |           |
          +-----+------------------------------------------------+

**Evidence Summary:**

- (E1) The flag diagram in Section 2.4.4.2.3 positions the B-Flag in a different bit location.
- (E2) The IANA registry table in Section 6.8 assigns the B-Flag to bit 3, conflicting with the diagram.

**Fix Direction:**

Align the IANA registry table in Section 6.8 with the flag diagram in Section 2.4.4.2.3 by reassigning the B-Flag to the appropriate bit (e.g., bit 2) to ensure consistent wire encoding.


**Severity:** High
  *Basis:* This inconsistency directly impacts wire-level encoding and decoding, causing potential misinterpretation of the presence of the SRv6 Endpoint Behavior & SID Structure.

**Confidence:** High

**Severity:** High
  *Basis:* This inconsistency directly impacts wire-level encoding and decoding, causing potential misinterpretation of the presence of the SRv6 Endpoint Behavior & SID Structure.

**Confidence:** High


---
