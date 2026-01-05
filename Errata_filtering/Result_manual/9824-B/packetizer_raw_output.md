# Errata Reports

Total reports: 1

---


## Report 1: 9824-B-1

**Label:** Misnamed Oracle Cloud Initiative Instead of Oracle Cloud Infrastructure in Historical Note

**Bug Type:** Inconsistency

**Explanation:**

The RFC text refers to 'Oracle Cloud Initiative' rather than the widely recognized 'Oracle Cloud Infrastructure (OCI)', which could mislead readers correlating the RFC with real-world deployments.

**Justification:**

- The text includes the sentence: Oracle Cloud Initiative implemented Compact Denial of Existence using NSEC3 in October 2024.
- Context notes highlight that no other part of the RFC uses 'Oracle Cloud Initiative', while all other service names are correctly spelled.

**Evidence Snippets:**

- **E1:**

  Oracle Cloud Initiative implemented Compact Denial of Existence using NSEC3 in October 2024.

- **E2:**

  No other mention of “Oracle Cloud Initiative” or Oracle’s platform elsewhere in the provided RFC 9824 text.
        All other proper names in Section B (“Cloudflare”, “NS1”, “Amazon Route53”, “Knot DNS”) are recognizable DNS service or software names and appear to be spelled correctly.

**Evidence Summary:**

- (E1) Oracle Cloud Initiative implemented Compact Denial of Existence using NSEC3 in October 2024.
- (E2) No other mention of “Oracle Cloud Initiative” or Oracle’s platform elsewhere in the provided RFC 9824 text. All other proper names in Section B (“Cloudflare”, “NS1”, “Amazon Route53”, “Knot DNS”) are recognizable DNS service or software names and appear to be spelled correctly.

**Fix Direction:**

In Section B (Historical Implementation Notes), replace ‘Oracle Cloud Initiative implemented Compact Denial of Existence using NSEC3 in October 2024.’ with ‘Oracle Cloud Infrastructure (OCI) implemented Compact Denial of Existence using NSEC3 in October 2024.’


**Severity:** Low
  *Basis:* The error is editorial in nature and does not affect protocol behavior, though it may cause confusion when correlating RFC content with industry-standard naming.

**Confidence:** High

---