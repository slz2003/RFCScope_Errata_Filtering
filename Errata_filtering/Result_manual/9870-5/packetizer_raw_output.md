# Errata Reports

Total reports: 1

---

## Report 1: 9870-5-1

**Label:** Ambiguous Example 5.2: Unqualified EXP Bit Setting vs. udpSafeExIDList Exception

**Bug Type:** Specification Ambiguity/Underspecification

**Explanation:**

Example 5.2 in Section 5 unconditionally states that the EXP bit in udpSafeOptions is set when SAFE Experimental Options are observed, without accounting for the normative exception in Section 4.1 that mandates the EXP bit must remain unset when udpSafeExIDList is present.

**Justification:**

- Section 4.1 clearly mandates that when udpSafeExIDList is present, the exporter MUST NOT set the EXP flag in udpSafeOptions, thereby taking precedence over the general rule that sets the bit when an option is observed.
- Section 5.2’s unqualified statement about the EXP bit being set conflicts with this requirement, as further illustrated by Section 5.3 where udpSafeOptions is set to 0x05 (omitting the EXP bit) in the presence of udpSafeExIDList.

**Evidence Snippets:**

- **E1:**

  Section 4.1: “Options are mapped to bits… The bit is set to 1 if the corresponding SAFE UDP Option is observed at least once in the Flow. The bit is set to 0 if the option is never observed in the Flow. … The presence of udpSafeExIDList takes precedence over setting the corresponding bit in the udpSafeOptions IE for the same Flow. In order to optimize the use of the reduced-size encoding in the presence of udpSafeExIDList IE, the Exporter MUST NOT set the EXP flag of the udpSafeOptions IE that is reported for the same Flow to 1.”

- **E2:**

  Section 5.2: “Let us now consider a UDP Flow in which SAFE Experimental Options are observed. If a udpSafeOptions IE is exported for this Flow, then that IE will have the EXP bit set to 1 (Figure 4). This example does not make any assumption about the presence of other UDP Options (‘X’ can be set to 0 or 1).”

- **E3:**

  Section 5.3: “Following the guidance in Section 4.1, the reported udpSafeOptions IE will be set to 0x05 even in the presence of EXP Options.”

**Evidence Summary:**

- (E1) Section 4.1 mandates that the presence of udpSafeExIDList takes precedence and forbids setting the EXP bit in udpSafeOptions.
- (E2) Section 5.2 unconditionally states that the EXP bit is set to 1 when EXP options are observed.
- (E3) Section 5.3 clarifies that udpSafeOptions is set to 0x05 (omitting the EXP bit) when udpSafeExIDList is present.

**Fix Direction:**

Revise Example 5.2 to explicitly state that the EXP bit is set only when udpSafeExIDList is not present, thereby aligning the example with the normative rule in Section 4.1.


**Severity:** Low
  *Basis:* While the issue may lead to implementer confusion regarding the correct handling of the EXP bit, it does not cause wire-format or interoperability failures.

**Confidence:** High

---
