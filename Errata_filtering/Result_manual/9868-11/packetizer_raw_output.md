# Errata Reports

Total reports: 8

---

## Report 1: 9868-11-1

**Label:** APC Option Length Handling Conflict: Global Minimum-Length vs. APC 'Unrecognized Lengths' Inconsistency

**Bug Type:** Both

**Explanation:**

There is an ambiguity in how APC option lengths are handled, where the global rule treats lengths below the minimum as fatal (discarding all options), but the APC‐specific rule instructs that unrecognized lengths be treated as a checksum failure without invalidating other options.

**Justification:**

- The global rule in Section 10 mandates that option lengths smaller than the minimum for the corresponding Kind MUST result in discarding all UDP Options.
- Section 11.3 instructs that UDP packets with unrecognized APC lengths MUST be treated similarly to those with incorrect checksum fields.
- This contradiction could lead to divergent receiver behaviors in error cases.

**Evidence Snippets:**

- **E1:**

  Option Lengths (or Extended Lengths, where applicable) smaller than the minimum for the corresponding Kind MUST be treated as an error. Such errors call into question the remainder of the surplus area and thus MUST result in all UDP Options being silently discarded.

- **E2:**

  UDP packets with unrecognized APC lengths MUST receive the same treatment as UDP packets with incorrect APC Option checksum fields.

- **E3:**

  Receivers cannot generally treat unexpected Option Lengths as invalid, as this would unnecessarily limit future revision of options (e.g., defining a new APC that is defined by having a different length).

**Evidence Summary:**

- (E1) Global rule requiring discarding options when lengths are below minimum.
- (E2) APC‐specific rule calling for treatment of unrecognized lengths as checksum failures.
- (E3) A remark on allowing future APC revisions which adds to the ambiguity.

**Fix Direction:**

Clarify that the APC‐specific unrecognized lengths rule applies only to lengths greater than or equal to the defined minimum (6 bytes), and that lengths below the minimum must trigger the global error handling.


**Severity:** Medium
  *Basis:* The inconsistency affects error recovery and interoperability, potentially leading to divergent implementations in handling malformed APC options.

**Confidence:** High

---

## Report 2: 9868-11-2

**Label:** Ambiguous Fragment Size Calculation (S) with Additional Per-Fragment Options

**Bug Type:** Underspecification

**Explanation:**

The fragmentation procedure does not explicitly define how to account for additional per‐fragment options when computing the desired fragment size (S), which may result in fragments that exceed the path MTU.

**Justification:**

- Section 11.4 instructs to choose S accounting for per‐fragment options, yet the subsequent S–12/S–14 formulas subtract only the fixed overhead for FRAG and OCS.
- Different interpretations of S could produce fragments that are oversized if extra per‐fragment options are present.

**Evidence Snippets:**

- **E1:**

  Identify the desired fragment size, which we will call 'S'. This value is calculated to take into account the path MTU (if known) and to allow space for per-fragment options.

- **E2:**

  Fragment 'D' into chunks of size no larger than 'S'-12 each (10 for the non-terminal FRAG Option and 2 for OCS)...

- **E3:**

  The simple S–12/S–14 formulas only account for OCS and FRAG, and do not change when other options are present.

**Evidence Summary:**

- (E1) Instruction to compute S by considering both the PMTU and space for per-fragment options.
- (E2) The chunking rule subtracts only 12 or 14 bytes for fixed options.
- (E3) The text highlights that additional per-fragment options are not explicitly accounted for.

**Fix Direction:**

Provide an explicit formula or guidance for subtracting the size of all additional per‐fragment options when determining S.


**Severity:** Low
  *Basis:* Although the underspecification could lead to oversize fragments, competent implementers are likely to adjust S to account for extra options, minimizing critical failures.

**Confidence:** High

---

## Report 3: 9868-11-3

**Label:** Misleading 'Transport Payload Contained in a FRAG Option' Statement

**Bug Type:** Underspecification

**Explanation:**

The specification misleadingly states that the transport payload is contained in a FRAG Option, whereas in reality the payload resides in the fragment data area following the UDP Options.

**Justification:**

- Section 10 instructs that when UNSAFE Options are present, any transport payload MUST be contained in a FRAG Option.
- Section 11.4, however, defines that the fragment data (carrying the payload) follows the FRAG Option in the surplus area.

**Evidence Snippets:**

- **E1:**

  When UNSAFE Options are present, the UDP user data MUST be empty, and any transport payload MUST be contained in a FRAG Option (see Section 11.4).

- **E2:**

  The Frag. Start field indicates the location of the beginning of the fragment data, measured from the beginning of the UDP header of the fragment. The fragment data follows the remainder of the UDP Options and continues to the end of the IP datagram...

**Evidence Summary:**

- (E1) Statement from Section 10 on transport payload containment in a FRAG Option.
- (E2) Description from Section 11.4 clarifying that the payload is actually located in the fragment data area.

**Fix Direction:**

Reword the UNSAFE options rule to indicate that the transport payload is carried in the fragment data area associated with the FRAG Option, rather than within its TLV body.


**Severity:** Low
  *Basis:* This is primarily a terminology issue that is unlikely to cause major interoperability problems.

**Confidence:** High

---

## Report 4: 9868-11-4

**Label:** Contradictory Requirements for Overlong Option Lengths

**Bug Type:** Inconsistency

**Explanation:**

The document provides conflicting instructions by requiring that options with lengths overrunning the surplus area MUST trigger discarding of all options, while elsewhere such options SHOULD be silently ignored.

**Justification:**

- Section 10 states that options indicating overruns MUST lead to the discard of all UDP Options.
- Section 14, on the other hand, suggests that options that exceed the available UDP packet length SHOULD be silently ignored.

**Evidence Snippets:**

- **E1:**

  Options with inherently invalid Length field values, i.e., those that indicate underruns of the option itself or overruns of the surplus area (pointing past the end of the IP payload), MUST be treated as an indication of a malformed surplus area, and all options MUST silently be discarded.

- **E2:**

  Any options whose length exceeds that of the UDP packet (i.e., intending to use data that would have been beyond the surplus area) SHOULD be silently ignored (again to model legacy behavior).

**Evidence Summary:**

- (E1) Strict directive from Section 10 to discard all options on length overrun.
- (E2) Section 14 suggesting a softer approach by ignoring only the offending option.

**Fix Direction:**

Harmonize the treatment of overlong option lengths by specifying a single consistent rule for all cases.


**Severity:** Medium
  *Basis:* This discrepancy can lead to divergent receiver behaviors when processing malformed packets.

**Confidence:** High

---

## Report 5: 9868-11-5

**Label:** FRAG Terminal Format Figure Caption Mislabelled as Non-Terminal

**Bug Type:** Inconsistency

**Explanation:**

The caption for Figure 11 incorrectly labels the terminal FRAG Option format as non-terminal, which may mislead implementers regarding the correct format and length.

**Justification:**

- The prose clearly distinguishes that non-terminal fragments use the shorter format (Figure 10) and terminal fragments use the longer format (Figure 11).
- However, Figure 11’s caption incorrectly states 'UDP Non-Terminal FRAG Option Format' despite showing a 12-byte format with an additional RDOS field.

**Evidence Snippets:**

- **E1:**

  The FRAG Option has two formats: non-terminal fragments use the shorter variant (Figure 10) and terminal fragments use the longer (Figure 11).

- **E2:**

  Figure 11: UDP Non-Terminal FRAG Option Format

**Evidence Summary:**

- (E1) Text introducing the two distinct FRAG formats.
- (E2) The incorrect caption of Figure 11 that mislabels the terminal format.

**Fix Direction:**

Correct the caption of Figure 11 to read 'UDP Terminal FRAG Option Format' (optionally noting Len=12 and inclusion of the RDOS field).


**Severity:** Low
  *Basis:* This labeling error is unlikely to impact protocol interoperability but may cause temporary confusion during implementation.

**Confidence:** High

---

## Report 6: 9868-11-6

**Label:** Inconsistent Handling of OCS Failure in Fragmented Datagrams

**Bug Type:** Inconsistency

**Explanation:**

For fragments experiencing an OCS failure, the generic processing rules imply delivery of zero-length user data, while the FRAG-specific reassembly rules require that no individual fragment be delivered, leading to ambiguous behavior.

**Justification:**

- Section 14’s pseudocode indicates that failures may result in delivering UDP user data (even if zero-length).
- Section 11.4 clearly mandates that individual UDP fragments MUST NOT be forwarded and that reassembly failures SHOULD NOT generate zero-length frames.

**Evidence Snippets:**

- **E1:**

  if the UDP checksum passes or is zero then … deliver the UDP user data but ignore other options (this is required to emulate legacy behavior)

- **E2:**

  Individual UDP fragments MUST NOT be forwarded to the user. The reassembled datagram is received only after complete reassembly, checksum validation, and continued processing of the remaining UDP Options.

**Evidence Summary:**

- (E1) Generic processing rule that can lead to delivery of zero-length data on failure.
- (E2) FRAG-specific rule prohibiting the delivery of individual fragments.

**Fix Direction:**

Clarify that OCS failures in a fragmented datagram must result in reassembly failure without delivering any zero-length fragments to the user.


**Severity:** High
  *Basis:* This inconsistency can cause observable differences in packet delivery, impacting application behavior and protocol robustness.

**Confidence:** High

---

## Report 7: 9868-11-7

**Label:** Conflicting Requirements for FRAG with Non-Empty User Data and UNSAFE Options

**Bug Type:** Inconsistency

**Explanation:**

The specification contains contradictory requirements for packets that include a FRAG option yet have non-empty user data, especially when UNSAFE options are present, leading to ambiguity over whether the data should be delivered or dropped.

**Justification:**

- Section 11.4 requires that if FRAG is present and user data is not empty, all UDP Options be ignored and the user data delivered.
- Section 12 mandates that UDP user data MUST be dropped if an UNSAFE Option appears outside the proper fragmented context.
- This divergence creates uncertainty in how a receiver should process such packets.

**Evidence Snippets:**

- **E1:**

  When FRAG is present, the UDP user data MUST be empty. If the user data is not empty, all UDP Options MUST be silently ignored and the user data received MUST be sent to the user.

- **E2:**

  Receivers supporting UDP Options MUST silently drop the UDP user data of the reassembled datagram if any fragment or the entire datagram includes an UNSAFE Option whose Kind is not supported or if an UNSAFE Option appears outside the context of a fragment or reassembled fragments.

**Evidence Summary:**

- (E1) Directive from Section 11.4 to deliver user data when FRAG is present despite non-empty data.
- (E2) Rule from Section 12 requiring the dropping of user data when UNSAFE options are detected outside their proper context.

**Fix Direction:**

Specify a clear precedence or revised rule that resolves how to handle packets with FRAG options, non-empty user data, and UNSAFE options.


**Severity:** High
  *Basis:* This conflict is security-sensitive and may lead to widely divergent behaviors among implementations.

**Confidence:** High

---

## Report 8: 9868-11-8

**Label:** Underspecified Behavior for Out-of-Range FRAG Pointers (RDOS/Frag.Start)

**Bug Type:** Underspecification

**Explanation:**

The specification does not define receiver behavior when the FRAG pointer fields (Frag.Start and RDOS) contain out-of-range values, leaving reassembly and option processing undefined in error or malicious conditions.

**Justification:**

- Section 11.4 defines the Frag.Start and RDOS fields without providing explicit bounds or handling instructions if the values fall outside expected ranges.
- Lack of validation guidance for these pointers may lead to inconsistent handling, impacting reassembly correctness and security.

**Evidence Snippets:**

- **E1:**

  The Frag. Start field indicates the location of the beginning of the fragment data, measured from the beginning of the UDP header of the fragment. The fragment data follows the remainder of the UDP Options and continues to the end of the IP datagram...

- **E2:**

  The terminal FRAG Option format adds a Reassembled Datagram Option Start (RDOS) pointer, measured from the start of the original UDP datagram header, indicating the end of the reassembled data and the start of the surplus area within the original UDP datagram.

- **E3:**

  Options with inherently invalid Length field values, i.e., those that indicate underruns of the option itself or overruns of the surplus area (pointing past the end of the IP payload), MUST be treated as an indication of a malformed surplus area, and all options MUST silently be discarded.

**Evidence Summary:**

- (E1) Description of the Frag.Start field and its role in locating fragment data.
- (E2) Definition of the RDOS pointer in the terminal FRAG Option format.
- (E3) General rule for handling inherently invalid option lengths, implying the need for similar validation for pointer fields.

**Fix Direction:**

Define explicit receiver behavior and validation checks for FRAG pointer fields, specifying actions for out-of-range values.


**Severity:** High
  *Basis:* Improper handling of pointer values is critical for correct reassembly and may open security vulnerabilities if not uniformly addressed.

**Confidence:** High

---
