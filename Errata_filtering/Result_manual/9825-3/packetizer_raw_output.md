# Errata Reports

Total reports: 1

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