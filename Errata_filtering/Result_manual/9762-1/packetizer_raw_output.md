# Errata Reports

Total reports: 2

---

## Report 1: 9762-1-1

**Label:** Ambiguous /60→/64 Capacity Example and Implicit Reserved /64

**Bug Type:** Underspecification

**Explanation:**

The RFC text uses a /60 example that implies only 15 available /64 prefixes without stating that one prefix is implicitly reserved, which creates ambiguity when compared with standard IPv6 arithmetic.

**Justification:**

- The introduction states that on a home network receiving a /60, the network runs out of prefixes after 15 devices have been connected (E1).
- Pure IPv6 arithmetic indicates a /60 contains 16 /64s, so the figure of 15 implies an unstated reservation, causing potential confusion (E1).

**Evidence Snippets:**

- **E1:**

  The introduction states that on “a home network that only receives a /60 from the ISP and each client obtains a /64 prefix, then the network will run out of prefixes after 15 devices have been connected.” Pure IPv6 prefix arithmetic – and the capacity reasoning in RFC 9663 Section 8, which discusses how many /64s fit in a given aggregate – implies that a /60 contains 2^(64‑60)=16 distinct /64s, so if *all* 16 were usable for per‑device PD, exhaustion would occur only when trying to serve the 17th device  . The “15 devices” figure is only correct if one of the 16 /64s from the /60 is implicitly reserved for some other use (most plausibly a shared on‑link /64 advertised via a PIO for non‑PD clients), but that assumption is not stated in Section 1 and RFC 9663’s per‑device model does not intrinsically require such a shared on‑link GUA prefix. A careful reader comparing the /60 example in RFC 9762 with the more general capacity discussion in RFC 9663 Section 8 might be confused about whether per‑device PD is expected to coexist *by definition* with a separate on‑link /64, or whether those 16 /64s are, in principle, all available for PD. This is non‑normative background text and does not create an interoperability failure, but the missing assumption makes the cross‑document prefix‑capacity story slightly opaque.

**Evidence Summary:**

- (E1) The excerpt indicates that the /60 example implies only 15 usable /64 prefixes, although standard arithmetic would allow 16, leaving ambiguity regarding an unstated reserved prefix.


---

## Report 2: 9762-1-2

**Label:** Ambiguous P Flag Location: Inconsistent Placement in RA Note vs. PIO Definition

**Bug Type:** Inconsistency

**Explanation:**

The modified RFC 4861 note groups the P flag with the RA header flags (M and O) despite its explicit definition as a Prefix Information Option flag, which can mislead implementers about its intended location.

**Justification:**

- Section 5 explicitly defines the P flag as a 1-bit PIO flag located after the R flag (E1).
- The IANA Considerations register P as 'PIO Option Bit 3', confirming its place in the PIO (E2).
- Section 9.1’s new text groups M, O, and P flags together, obscuring the fact that P belongs in the PIO (E3).

**Evidence Snippets:**

- **E1:**

  Section 5 (definition and location of P flag):  “The P flag (also called the DHCPv6-PD Preferred Flag) is a 1-bit PIO flag, located after the R flag [RFC6275].”  Figure 1 immediately above shows the Prefix Information Option with the flags field “|L|A|R|P| Rsvd1 |”.

- **E2:**

  IANA Considerations (scope of registry entry):  “IANA has made the following allocation in the ‘IPv6 Neighbor Discovery Prefix Information Option Flags’ registry [RFC8425]:  … `PIO Option Bit 3 | P - DHCPv6-PD Preferred Flag | RFC 9762`.”

- **E3:**

  Section 9.1 (change to RFC 4861 Section 4.2 note):  “NEW TEXT:  |  Note: If the M, O, or P (RFC 9762) flags are not set, this  indicates that no information is available via DHCPv6.”

**Evidence Summary:**

- (E1) The P flag is defined as a PIO flag following the R flag.
- (E2) The IANA registry confirms the P flag as part of the PIO with Option Bit 3.
- (E3) The new note in Section 9.1 groups the P flag with M and O flags, creating ambiguity about its location.

**Fix Direction:**

Clarify Section 9.1 by explicitly stating: 'If neither the M nor O flags in the Router Advertisement header are set and no Prefix Information Option in the RA has the P flag set, then no DHCPv6 information is available.'


---
