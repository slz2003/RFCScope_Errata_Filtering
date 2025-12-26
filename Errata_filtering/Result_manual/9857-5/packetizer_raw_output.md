# Errata Reports

Total reports: 7

---

## Report 1: 9857-5-1

**Label:** SR Policy Name TLV Scope Ambiguity (Per-Candidate vs. Per-Policy)

**Bug Type:** Underspecification

**Explanation:**

The SR Policy Name TLV is encoded per candidate path even though it is conceptually a single attribute for the entire SR Policy, and the specification does not define what to do when conflicting names appear.

**Justification:**

- Evidence shows the TLV is advertised per candidate path (“Only a single instance … for a given candidate path.” (5.4)) even though a policy is logically unique.
- It states that the Candidate Path Descriptor identifies an SR Policy candidate path and that the associated TLVs relate to that candidate path, leaving inter‑instance consistency undefined.

**Evidence Snippets:**

- **E1:**

  The SR Policy Name TLV is an optional TLV that is used to carry the symbolic name associated with the SR Policy. Only a single instance of this TLV is advertised for a given candidate path. (5.4)

- **E2:**

  The SR Policy Candidate Path Descriptor TLV… identifies an SR Policy candidate path… and the BGP‑LS Attribute TLVs are “associated with the SR Policy Candidate Path NLRI type.” (4, 5)

**Evidence Summary:**

- (E1) Shows the per‐candidate path advertisement of the SR Policy Name TLV.
- (E2) Indicates the association of TLVs with candidate path NLRI without reconciliation rules.

**Fix Direction:**

Either require that all candidate paths for the same SR Policy carry identical SR Policy Name TLVs or redefine the TLV to be explicitly scoped per candidate path.


**Severity:** Medium
  *Basis:* Mismatch between encoding and conceptual scope can lead to inconsistent policy views across controllers.

**Confidence:** High

---

## Report 2: 9857-5-2

**Label:** Inconsistent Dataplane and Algorithm/Topology Flags Across TLV Scopes

**Bug Type:** Both

**Explanation:**

Dataplane selection flags and algorithm/topology parameters are encoded in the candidate path constraints, per-segment list, and BSID TLVs without any normative requirement for consistency, creating ambiguity if values differ.

**Justification:**

- Candidate path constraints define a D-flag for SRv6 versus SR/MPLS (5.6), while the segment list TLV independently defines a D-flag (5.7).
- Additionally, the SR Binding SID TLV (5.1) uses a similar flag without clarifying cross-TLV precedence, leaving implementations free to interpret conflicting signals.

**Evidence Snippets:**

- **E1:**

  Candidate path constraints D‑flag: “Indicates that the candidate path uses an SRv6 data plane when set and an SR/MPLS data plane when clear.” (5.6)

- **E2:**

  Segment list D‑flag: “Indicates that the SID list consists of SRv6 SIDs when set and SR/MPLS labels when clear.” (5.7)

- **E3:**

  SR Binding SID D‑flag: “Indicates… if a BSID is a 16‑octet SRv6 SID (when set) or 4‑octet SR/MPLS label value (when clear).” (5.1)

**Evidence Summary:**

- (E1) Defines the data plane at candidate path level.
- (E2) Defines the data plane at segment list level.
- (E3) Shows similar functionality at the BSID level.

**Fix Direction:**

Specify that all flags representing dataplane and algorithm/topology information must agree, or provide a clear rule for resolving differences between candidate path, segment list, and BSID TLVs.


**Severity:** Medium
  *Basis:* Inconsistent flag values can lead to diverging interpretations by BGP‑LS consumers and undermine interoperability.

**Confidence:** High

---

## Report 3: 9857-5-3

**Label:** Ambiguous Scope of SR Binding SID TLV 1201 for SRv6 After Deprecation

**Bug Type:** Underspecification

**Explanation:**

The document deprecates using SR Binding SID TLV 1201 for SRv6 BSIDs but does not clearly limit its scope, leaving ambiguity as to whether new implementations may transmit it with the D-flag set.

**Justification:**

- The TLV is initially defined to support both MPLS and SRv6 based on the D-flag. (5.1)
- Later text deprecates its use for SRv6, yet leaves it as ‘SHOULD NOT’ rather than ‘MUST NOT’, and the IANA registry does not restrict 1201 to MPLS only.

**Evidence Snippets:**

- **E1:**

  SR Binding SID TLV definition: “D‑Flag: Indicates the data plane for the BSIDs and if a BSID is a 16‑octet SRv6 SID (when set) or 4‑octet SR/MPLS label value (when clear).” (5.1)

- **E2:**

  Later in the same section: “In the case of an SRv6, the SR Binding SID sub‑TLV does not have the ability to signal the SRv6 Endpoint behavior… Therefore, the SR Binding SID sub‑TLV SHOULD NOT be used for the advertisement of an SRv6 Binding SID. Instead, the SRv6 Binding SID TLV defined in Section 5.2 SHOULD be used… The use of the SR Binding SID sub‑TLV for advertisement of the SRv6 Binding SID has been deprecated, and it is documented here only for backward compatibility with implementations that followed early draft versions of this specification.” (5.1, last paragraph)

- **E3:**

  IANA Table 4 assigns code point 1201 as “SR Binding SID”, without restricting it to MPLS‑only.

**Evidence Summary:**

- (E1) Shows a generic definition of the SR Binding SID TLV with support for both dataplanes.
- (E2) Indicates the deprecation of its use for SRv6, but only as advisory language.
- (E3) Confirms that the IANA registry does not enforce MPLS-only usage.

**Fix Direction:**

Clearly state that new BGP‑LS Producers MUST NOT originate SR Binding SID TLV 1201 with the D-flag set for SRv6 and should use the SRv6 Binding SID TLV (1212) instead.


**Severity:** Medium
  *Basis:* Ambiguity in the deprecation policy may lead to inconsistent encoding of SRv6 BSIDs across implementations.

**Confidence:** High

---

## Report 4: 9857-5-4

**Label:** Ambiguous Handling of Mutually Exclusive P-Flag and U-Flag in Candidate Path Constraints

**Bug Type:** Underspecification

**Explanation:**

Although the specification states that the P-Flag and U-Flag for protection preferences cannot be set simultaneously, it does not provide a normative rule or guidance on receiver behavior when both are set, which can lead to inconsistent handling.

**Justification:**

- The spec defines the P-Flag: “Indicates that the candidate path prefers the use of only protected SIDs … mutually exclusive with the U-Flag.”
- It similarly defines the U-Flag as mutually exclusive with the P-Flag, yet provides no error handling or precedence rule.

**Evidence Snippets:**

- **E1:**

  P-Flag: Indicates that the candidate path prefers the use of only protected SIDs when set… This flag is mutually exclusive with the U-flag (i.e., both of these flags cannot be set at the same time).

- **E2:**

  U-Flag: Indicates that the candidate path prefers the use of only unprotected SIDs when set… This flag is mutually exclusive with the P-Flag (i.e., both of these flags cannot be set at the same time).

**Evidence Summary:**

- (E1) Specifies the mutual exclusivity of the P-Flag.
- (E2) Reinforces that the U-Flag should not be set with the P-Flag.

**Fix Direction:**

Include explicit normative language (e.g., a MUST NOT clause) forbidding simultaneous setting of P-Flag and U-Flag, and specify the consumer interpretation if both are encountered.


**Severity:** Medium
  *Basis:* Without clear guidance, differing interpretations of contradictory flag settings could lead to inconsistent policy computations across networks.

**Confidence:** High

---

## Report 5: 9857-5-5

**Label:** Underspecified Drop-Upon-Invalid Flag Combinations (I-Flag and U-Flag)

**Bug Type:** Underspecification

**Explanation:**

The mapping of the I-Flag and U-Flag to the Drop-Upon-Invalid behavior does not define all allowed combinations or their exact relationship with other candidate path attributes, leaving ambiguity for consumers.

**Justification:**

- The I-Flag is defined to indicate that the candidate path is to perform Drop-Upon-Invalid, and the U-Flag indicates that the candidate path is actively dropping traffic.
- The spec does not clarify if U=1 must always accompany I=1, nor does it define allowed combinations when Drop-Upon-Invalid is enabled or disabled.

**Evidence Snippets:**

- **E1:**

  I-Flag: Indicates that the candidate path is to perform the ‘Drop‑Upon‑Invalid’ behavior when no other valid candidate path is available for this SR Policy when the flag is set.… When clear, it indicates that the candidate path is not enabled for the ‘Drop‑Upon‑Invalid’ behavior.

- **E2:**

  U-Flag: Indicates that the candidate path is reported as active and is dropping traffic as a result of the ‘Drop‑Upon‑Invalid’ behavior being activated for the SR Policy when set. When clear, it indicates that the candidate path is not dropping traffic as a result of the ‘Drop‑Upon‑Invalid’ behavior.

**Evidence Summary:**

- (E1) Describes the intended role of the I-Flag for Drop‑Upon‑Invalid behavior.
- (E2) Describes the role of the U-Flag without clarifying its dependency on I-Flag or active state.

**Fix Direction:**

Define a clear set of rules for how I-Flag and U-Flag relate (e.g., that U=1 implies I=1 and requires the candidate path to be active), and specify allowed combinations.


**Severity:** Medium
  *Basis:* Ambiguities in these flag combinations can lead to divergent interpretations of candidate path state across different consumers.

**Confidence:** High

---

## Report 6: 9857-5-6

**Label:** Inconsistent Terminology: SR Binding SID Referred to as TLV and Sub-TLV

**Bug Type:** Inconsistency

**Explanation:**

The SR Binding SID (code point 1201) is inconsistently described as both a top-level TLV and as a sub‑TLV, which may confuse implementers as to its correct placement in the BGP‑LS Attribute.

**Justification:**

- Section 5.1 describes the object as the “SR Binding SID TLV” with a defined format and code point.
- Later, the same object is referred to as the “SR Binding SID sub‑TLV” when discussing SRv6 deprecation, while Table 4 registers it as a top-level TLV.

**Evidence Snippets:**

- **E1:**

  Section 5.1 titles and describes “SR Binding SID TLV” (Type 1201) as: “The SR Binding Segment Identifier (BSID) is an optional TLV … The TLV has the following format… Type: 1201 … Length: Variable (valid values are 12 or 36 octets).”

- **E2:**

  The closing paragraph of Section 5.1 states: “In the case of an SRv6, the SR Binding SID sub‑TLV does not have the ability to signal the SRv6 Endpoint behavior … Therefore, the SR Binding SID sub‑TLV SHOULD NOT be used for the advertisement of an SRv6 Binding SID. Instead, the SRv6 Binding SID TLV defined in Section 5.2 SHOULD be used…”

- **E3:**

  Section 8.3 (Table 4) lists code point 1201 as “SR Binding SID” under “BGP‑LS NLRI and Attribute TLVs”, i.e., as a top-level TLV, not a sub‑TLV.

**Evidence Summary:**

- (E1) Introduces SR Binding SID as a TLV with a specific format.
- (E2) Later, refers to the same object as a sub-TLV.
- (E3) Confirms in the registry that it is a top-level TLV.

**Fix Direction:**

Revise Section 5.1 to consistently refer to code point 1201 as a top-level TLV and remove or clearly distinguish any legacy sub‑TLV terminology.


**Severity:** Low
  *Basis:* Although the inconsistency may cause confusion, it does not affect on‐wire encoding.

**Confidence:** High

---

## Report 7: 9857-5-7

**Label:** Ambiguous Combination of Disjointness Status Flags in SR Disjoint Group Constraint sub‑TLV

**Bug Type:** Underspecification

**Explanation:**

The status flags F, I, and X within the SR Disjoint Group Constraint sub‑TLV are intended to represent mutually exclusive disjointness outcomes, but the specification does not state whether these flags are mutually exclusive or provide precedence rules when multiple are set.

**Justification:**

- The sub‑TLV defines F-Flag, I-Flag, and X-Flag with distinct meanings regarding disjointness fallback and invalidation.
- It does not specify that these flags must be mutually exclusive or how to interpret conflicting combinations, leaving room for divergent implementation interpretations.

**Evidence Snippets:**

- **E1:**

  For the SR Disjoint Group Constraint sub‑TLV (Type 1211): • Request Flags F and I describe permission to “fall back to a lower level of disjointness” and to “fall back to the default best path (e.g., an IGP path).” • Status Flags are defined as: – “F-Flag: Indicates that the computation has fallen back to a lower level of disjointness than requested when set and that there has been no fallback to a lower level of disjointness when clear.” – “I-Flag: Indicates that the computation has fallen back to the best path (e.g., an IGP path) and disjointness has not been achieved when set and that there has been no fallback to the best path when clear.” – “X-Flag: Indicates that the disjointness constraint could not be achieved and hence the path has been invalidated when set and that the path has not been invalidated due to unmet disjointness constraints when clear.”

- **E2:**

  The intro to 5.6.4 also says: “The computation is expected to achieve the highest level of disjointness requested; when that is not possible, then fall back to a lesser level progressively based on the levels indicated.”

**Evidence Summary:**

- (E1) Lists the three status flags and their intended meanings without specifying exclusivity.
- (E2) Implies a fallback order without detailing flag precedence.

**Fix Direction:**

Add normative language to specify that F, I, and X are mutually exclusive and define the precedence order if more than one flag is set.


**Severity:** Medium
  *Basis:* Ambiguous flag combinations can lead to inconsistent disjointness state interpretations among consumers.

**Confidence:** High

---
