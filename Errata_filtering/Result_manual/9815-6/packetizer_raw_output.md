# Errata Reports

Total reports: 2

---





## Report 6: 9815-6-6

**Label:** Undefined term 'Node-ID' in SPF NLRI recency determination

**Bug Type:** Inconsistency

**Explanation:**

The term 'Node-ID' is used in the context of NLRI recency determination without a clear definition, creating ambiguity over whether it refers to the full Node Descriptor set or just a subset such as the BGP Router-ID.

**Justification:**

- The specification instructs that recency is determined by examining the 'Node-ID' and sequence number, but does not define 'Node-ID'.
- This ambiguity can lead to inconsistent comparisons of self-originated NLRIs across implementations.

**Evidence Snippets:**

- **E1:**

  When BGP-LS-SPF NLRI is received, all that is required is to determine whether it is the most recent by examining the Node-ID and sequence number as described in Section 6.1.

- **E2:**

  Nodes, Links, or Prefix NLRIs with Node Descriptors matching the local BGP speaker are considered self-originated.

**Evidence Summary:**

- (E1) uses the term 'Node-ID' without definition, while (E2) refers to Node Descriptors.
- The lack of a clear definition may cause different interpretations in NLRI recency comparisons.

**Fix Direction:**

Replace 'Node-ID' with a clearly defined term, such as 'Node Descriptors', or provide an explicit definition of 'Node-ID' in the document.


**Severity:** Medium
  *Basis:* Ambiguity in key identifiers can lead to mismatches in NLRI selection and misidentification of self-originated routes.

**Confidence:** High

---


## Report 7: 9815-6-7

**Label:** Incorrect use of 'Node Descriptors' instead of 'Link Descriptors' for link NLRI fields

**Bug Type:** Inconsistency

**Explanation:**

The document incorrectly refers to link-specific TLVs (e.g., TLV 259–262) as 'Node Descriptors' rather than using the proper term 'Link Descriptors', which may mislead implementers about NLRI structure.

**Justification:**

- Section 5.2.2 describes the advertisement of link NLRI with unique Local and Remote Node Descriptors, but the TLVs mentioned are defined as Link Descriptor TLVs in RFC9552.
- This misuse of terminology can confuse the distinction between node identity and link identity.

**Evidence Snippets:**

- **E1:**

  Link NLRI is advertised with unique Local and Remote Node Descriptors dependent on the IP addressing. For IPv4 links, the link's local IPv4 interface address (TLV 259) and remote IPv4 neighbor address (TLV 260) are used. For IPv6 links, the local IPv6 interface address (TLV 261) and remote IPv6 neighbor address (TLV 262) are used.

**Evidence Summary:**

- (E1) incorrectly labels TLVs that are actually defined for link descriptors as 'Node Descriptors', causing potential confusion.

**Fix Direction:**

Change references from 'Node Descriptors' to 'Link Descriptors' in the context of link NLRI advertisement.


**Severity:** Medium
  *Basis:* This terminology error can lead to incorrect encoding or interpretation of NLRI contents in BGP-LS-SPF implementations.

**Confidence:** High

---



**Confidence:** High

---
