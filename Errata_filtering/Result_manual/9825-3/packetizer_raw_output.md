# Errata Reports

Total reports: 3

---

## Report 1: 9825-3-1

**Label:** Undefined multiplicity handling for Administrative Tag Sub‑TLV instances

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly state whether more than one Administrative Tag Sub‑TLV instance is allowed per parent TLV, nor does it define how duplicates should be handled.

**Justification:**

- RFC 9825 defines the sub‑TLV on a per‐instance basis without limiting the number of instances or specifying duplicate handling, leaving behavior for multiple instances ambiguous (Sections 2 and 3).
- RFC 8362 expects each TLV/sub‑TLV specification to define multiplicity behavior, as seen with other sub‑TLVs where subsequent instances are ignored.

**Evidence Snippets:**

- **E1:**

  RFC 9825 defines the sub‑TLV format and semantics per instance, but does not constrain the number of instances per parent TLV or define behavior for duplicates: “This document creates a new Administrative Tag Sub-TLV… This sub-TLV specifies one or more 32-bit unsigned integers that may be associated with an OSPF advertised prefix.” and “If the length is 0 or not a multiple of 4 octets, the sub-TLV MUST be ignored…” (Section 2). Section 3 only says where it is valid, not how many instances or what to do if there are several: “The Administrative Tag Sub-TLV specified herein will be valid as a sub-TLV of the following TLVs…” and then enumerates Extended Prefix TLV, Inter-Area-Prefix TLV, Intra-Area-Prefix TLV, External-Prefix TLV, and SRv6 Locator TLV. RFC 8362 explicitly calls out that each TLV/sub‑TLV specification should define multiplicity behavior: “the specification should define whether the TLV or sub-TLV is required and the behavior when there are multiple occurrences of the TLV or sub-TLV.” (Section 3). For other sub‑TLVs in RFC 8362, this is done explicitly, e.g., “The sub-TLV is optional and the first specified instance is used as the forwarding address… Instances subsequent to the first MUST be ignored.” (Sections 3.10, 3.11) and “Instances subsequent to the first MUST be ignored.” for Route-Tag sub‑TLV (Section 3.12). The YANG model in RFC 9825 models exactly one container `prefix-admin-tag-sub-tlv` per parent TLV with a `leaf-list admin-tag` (Section 7.2), which implicitly assumes a single logical sub‑TLV with a list of tags, but does not translate into a wire‑format constraint on TLV multiplicity.

- **E2:**

  RFC 8362’s TLV framework explicitly notes that “the specification should define … the behavior when there are multiple occurrences of the TLV or sub-TLV” for Extended-LSA TLVs and sub-TLVs, and its own sub-TLVs (e.g., Route-Tag) explicitly say that only the first instance is used and subsequent instances MUST be ignored. RFC 9825 defines the Administrative Tag Sub-TLV (with its value being a list of one or more 32-bit tags) and makes it valid under the Extended Prefix TLVs of RFC 8362 and the SRv6 Locator TLV of RFC 9513, but it never states what to do if more than one Administrative Tag Sub-TLV of the same type appears under a single parent TLV instance. For example, an External-Prefix TLV in an E-AS-External-LSA or a SRv6 Locator TLV could theoretically carry two Administrative Tag Sub-TLVs; an implementation might (a) use only the first, (b) concatenate/union the tag lists from all of them, or (c) treat this as an error and ignore all occurrences. Because RFC 8362 explicitly expects each extension specification to define duplicate-handling behavior and gives concrete precedent (Route-Tag sub-TLV: “instances subsequent to the first MUST be ignored”), the absence of such a rule in RFC 9825 leaves room for divergent behavior between implementations handling malformed or non-conformant packets.

**Evidence Summary:**

- (E1) Describes the lack of constraints on the number of Administrative Tag Sub‑TLV instances per parent TLV and the absence of duplicate-handling rules.
- (E2) Highlights that, contrary to RFC 8362’s expectations, RFC 9825 does not specify which behavior to follow if multiple Administrative Tag Sub‑TLVs are present.

**Fix Direction:**

Specify explicit duplicate handling behavior in RFC 9825, either by mandating at most one Administrative Tag Sub‑TLV per parent TLV (with subsequent instances ignored) or by defining how multiple instances should be merged.


**Severity:** Medium
  *Basis:* Divergent behavior in handling multiple instances could lead to inconsistent administrative tag sets and interoperability issues across implementations.

**Confidence:** Medium

---

## Report 2: 9825-3-2

**Label:** Ambiguous relationship between Route-Tag and Administrative Tag Sub‑TLVs on External-Prefix TLVs

**Bug Type:** Underspecification

**Explanation:**

RFC 9825 does not clearly define how the new Administrative Tag Sub‑TLV interacts with the pre-existing Route-Tag sub‑TLV on External-Prefix TLVs, leading to potential inconsistencies in tag interpretation.

**Justification:**

- Section 4 of RFC 9825 mandates use of the existing tag for the first administrative tag while allowing additional tags via the new sub‑TLV, but it never clarifies if the existing tag refers to the Route-Tag sub‑TLV value or the first element in the new sub‑TLV list.
- This ambiguity may cause divergent implementations, with some treating the two sub‑TLVs as separate namespaces and others expecting them to be consistent.

**Evidence Snippets:**

- **E1:**

  RFC 8362 defines the Route-Tag sub-TLV (type 3 in the “OSPFv3 Extended-LSA Sub-TLVs” registry) for the External-Prefix TLV, stating that it has “identical semantics to the optional External Route Tag” of RFC 5340 and is the way to carry that single 32-bit external route tag in Extended AS-External and NSSA LSAs. RFC 9825 then introduces a new Administrative Tag Sub-TLV in the same “OSPFv3 Extended-LSA Sub-TLVs” registry and explicitly allows it under the External-Prefix TLV in both the E-AS-External-LSA and E-NSSA-LSA. Section 4 of RFC 9825 says that “when tags are advertised for AS External or NSSA LSA prefixes, the existing tag in the OSPFv2 and OSPFv3 AS-External-LSA and NSSA-LSA encodings MUST be utilized for the first tag; additional tags MAY be advertised using the Administrative Tag Sub-TLV.” However, RFC 9825 never explicitly states that, for Extended LSAs, this “existing tag” is the Route-Tag sub-TLV of RFC 8362, nor does it state whether the value of that Route-Tag sub-TLV MUST be equal to the first element in the Administrative Tag Sub-TLV list (i.e., are they logically the same thing, or two distinct tag namespaces that may diverge?).

**Evidence Summary:**

- (E1) Explains the lack of explicit instructions regarding the mapping between the Route-Tag sub‑TLV and the Administrative Tag Sub‑TLV on External-Prefix TLVs, leading to ambiguity in which tag should be considered primary.

**Fix Direction:**

Include explicit clarification in RFC 9825 on how the Route-Tag sub‑TLV relates to the Administrative Tag Sub‑TLV on External-Prefix TLVs, and whether their values must match or how they should interact.


**Severity:** Medium
  *Basis:* The ambiguity may result in inconsistent tag interpretation and non-interoperable behavior in systems that rely on a consistent notion of primary administrative tags.

**Confidence:** Medium

---

## Report 3: 9825-3-3

**Label:** No explicit instruction for handling Administrative Tag Sub‑TLV under unexpected parent TLVs

**Bug Type:** Underspecification

**Explanation:**

The specification does not state what should occur if an Administrative Tag Sub‑TLV appears under a TLV that is not one of the designated valid parent TLVs.

**Justification:**

- The analysis notes that there is no explicit instruction on handling an Administrative Tag Sub‑TLV attached to an unexpected TLV (e.g., OSPFv3 TLVs other than the three prefix TLVs).
- Although by analogy with RFC 8362 such sub‑TLVs would likely be ignored, the absence of explicit guidance can lead to inconsistent implementation behavior.

**Evidence Snippets:**

- **E1:**

  There is no explicit instruction on what to do if an Administrative Tag Sub‑TLV appears under an unexpected TLV (e.g., OSPFv3 TLVs other than the three prefix TLVs), although by analogy with RFC 8362 this would presumably just be ignored and not treated as an error.

**Evidence Summary:**

- (E1) Indicates that the specification lacks explicit guidance on how to handle Administrative Tag Sub‑TLVs received under non-designated parent TLVs.

**Fix Direction:**

Add explicit guidance in RFC 9825 for handling cases where an Administrative Tag Sub‑TLV is attached to an unexpected parent TLV, such as by mandating it be ignored.


**Severity:** Low
  *Basis:* While unlikely to affect normal operation since such cases might be ignored, the omission may cause minor implementation inconsistencies.

**Confidence:** Inferred

---
