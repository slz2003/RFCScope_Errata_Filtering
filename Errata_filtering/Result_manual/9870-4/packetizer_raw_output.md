# Errata Reports

Total reports: 2

---


## Report 1: 9870-4-1

**Label:** Unclear Flow-Aggregation and Deduplication Semantics for udpSafeExIDList/udpUnsafeExIDList

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define whether udpSafeExIDList/udpUnsafeExIDList should aggregate all distinct ExIDs observed in a Flow or simply report a single occurrence, nor does it specify ordering or deduplication behavior.

**Justification:**

- The IE definitions only state that the lists report 'Observed ExIDs' without clarifying if they are to be aggregated over the entire Flow or if duplicates are allowed.
- There is no normative guidance on whether multiple occurrences of the same ExID should be collapsed into one element or retained as separate entries.

**Evidence Snippets:**

- **E1:**

  udpExID: “Observed ExID in an Experimental (EXP, Kind=127) Option or an UNSAFE Experimental (UEXP, Kind=254) Option. A basicList of udpExID is used to report udpSafeExIDList and udpUnsafeExIDList values.”

- **E2:**

  udpSafeExIDList: “Observed ExIDs in the Experimental (EXP, Kind=127) Option. A basicList of udpExID Information Elements in which each udpExID Information Element carries the ExID observed in an EXP Option.”

**Evidence Summary:**

- (E1) shows that the IE simply reports observed ExIDs without specifying aggregation semantics.
- (E2) reinforces the lack of guidance regarding deduplication or ordering.

**Fix Direction:**

Clarify the IE definitions to state that each list MUST contain each distinct ExID observed in a Flow exactly once and, if desired, define a canonical ordering.


**Severity:** Medium
  *Basis:* Ambiguity in aggregation and deduplication may lead to inconsistent Flow metrics across implementations.

**Confidence:** High

---


## Report 3: 9870-4-3

**Label:** Incorrect '24 Octets' Example for Multiplexing IE

**Bug Type:** Inconsistency

**Explanation:**

The documentation claims that reporting mandatory SAFE Options using a single multiplexing IE would require at least 24 octets, which does not align with the reduced-size encoding calculations based on the mandatory option range.

**Justification:**

- Mandatory SAFE options are defined for Kinds 0–7, which would require only 1 octet when encoded with reduced-size encoding.
- The stated 24 octets correspond to a full 192-bit field, an interpretation that does not match any realistic scenario for mandatory options.

**Evidence Snippets:**

- **E1:**

  Section 4 says: “using one single IE that would multiplex both types of options will limit the benefits of reduced-size encoding in the presence of UNSAFE Options. For example, at least 24 octets would be needed to report mandatory SAFE Options that are observed in a Flow. … As further detailed in Section 5.1, only one octet is needed to report mandatory SAFE Options.”

- **E2:**

  Mandatory SAFE options (‘must‑support’ marked with ‘*’) in RFC 9868 are Kinds 0–7 (EOL, NOP, APC, FRAG, MDS, MRDS, REQ, RES), not the entire 0–191 SAFE range.

**Evidence Summary:**

- (E1) provides the conflicting statements regarding field length, with one part claiming 24 octets and another stating only one octet is needed.
- (E2) clarifies that mandatory options only cover Kinds 0–7, leading to a different expected encoding length.

**Fix Direction:**

Correct the explanatory text to accurately reflect the reduced-size encoding calculations based on the mandatory option range.


**Severity:** Low
  *Basis:* This error is confined to the explanatory text and does not affect the normative wire format, though it may confuse implementers about overhead expectations.

**Confidence:** High

---