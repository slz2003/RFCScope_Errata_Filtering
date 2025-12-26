# Errata Reports

Total reports: 1

---

## Report 1: 9740-4-1

**Label:** Ambiguity in tcpOptionsFull Semantics for Experimental Options (Kinds 253/254) Relative to ExID Lists

**Bug Type:** Underspecification

**Explanation:**

The specification initially defines tcpOptionsFull with a universal rule for setting bits based on observed TCP options, but later introduces an exception for experimental options (Kinds 253/254) when ExID list IEs are present, creating ambiguous, split semantics.

**Justification:**

- The document first states: For each TCP option, the respective bit is set to 1 if any observed packet contains the option, implying universal application.
- Later, it specifies that in the presence of tcpSharedOptionExID16List or tcpSharedOptionExID32List IEs, bits for Kinds 253/254 MUST NOT be set—even if the corresponding option was observed—leading to two-mode behavior and potential misinterpretation when only tcpOptionsFull is examined.

**Evidence Snippets:**

- **E1:**

  For each TCP option, there is a bit in this set. The bit is set to 1 if any observed packet of this Flow contains the corresponding TCP option. Otherwise, if no observed packet of this Flow contains the respective TCP option, the value of the corresponding bit is 0.

- **E2:**

  The presence of tcpSharedOptionExID16List or tcpSharedOptionExID32List IEs is an indication that a shared TCP option (Kind=253 or 254) is observed in a Flow. The presence of tcpSharedOptionExID16List or tcpSharedOptionExID32List IEs takes precedence over setting the corresponding bits in the tcpOptionsFull IE for the same Flow. … the Exporter MUST NOT set to 1 the shared TCP options (Kind=253 or 254) of the tcpOptionsFull IE that is reported for the same Flow.

**Evidence Summary:**

- (E1) The general tcpOptionsFull rule sets a bit to 1 if any packet contains the corresponding TCP option.
- (E2) An exception is later introduced for experimental options (Kinds 253/254) when ExID list IEs are present, which forces those bits to 0 despite the generic rule.

**Fix Direction:**

Clarify the tcpOptionsFull semantics by explicitly excluding Kinds 253 and 254 from the general 'bit set if present' rule when ExID list IEs are used, and add collector guidance to rely solely on the ExID lists for these options.


**Severity:** Low
  *Basis:* This issue is primarily semantic and editorial; while it does not impede overall protocol operation, it may lead to misinterpretation by collectors that rely only on the tcpOptionsFull bitmap.

**Confidence:** High

---
