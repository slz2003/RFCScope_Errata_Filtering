# Errata Reports

Total reports: 1

---


## Report 1: 9868-17-1

**Label:** Misidentification of UDP-Lite's Checksum Coverage field as 'UDP Length'

**Bug Type:** Inconsistency

**Explanation:**

RFC 9868 erroneously refers to a 'UDP Length' field in UDP-Lite instead of its properly defined 'Checksum Coverage' field, creating a conflict with RFC 3828.

**Justification:**

- Section 17 of RFC 9868 describes the UDP-Lite header by stating it interprets the 'UDP Length field' for checksum purposes, which contradicts RFC 3828 that replaces the UDP Length field with a Checksum Coverage field.
- Multiple experts (Scope, Structural, CrossRFC, and Terminology) noted that the misnaming can mislead implementers regarding the actual header layout and semantics.

**Evidence Snippets:**

- **E1:**

  Section 17 repeatedly refers to a “UDP Length field” in the context of UDP-Lite (“interpret the UDP Length field as the prefix covered by the UDP checksum”, “The UDP Length field in UDP-Lite is already used to indicate the boundary between user data covered by the checksum and user data not covered.”) even though RFC 3828 explicitly states that UDP-Lite replaces the UDP Length field with a distinct Checksum Coverage field and derives total length from IP.

- **E2:**

  “It uses a different transport protocol number (136) than UDP (17) to interpret the UDP Length field as the prefix covered by the UDP checksum.” … “UDP-Lite cannot support UDP Options, either as proposed here or in any other form, because the entire payload of the UDP packet is already defined as user data and there is no additional field in which to indicate a surplus area for options. The UDP Length field in UDP-Lite is already used to indicate the boundary between user data covered by the checksum and user data not covered.”

- **E3:**

  The UDP-Lite header is shown in figure 1. Its format differs from UDP in that the Length field has been replaced with a Checksum Coverage field. Checksum Coverage is the number of octets, counting from the first octet of the UDP-Lite header, that are covered by the checksum.

**Evidence Summary:**

- (E1) Section 17 of RFC 9868 misnames the UDP-Lite field as 'UDP Length' despite RFC 3828 replacing it with 'Checksum Coverage'.
- (E2) RFC 9868 describes UDP-Lite as using the UDP Length field for checksum coverage, furthering the misidentification.
- (E3) RFC 3828 clearly defines the replacement of the Length field with a Checksum Coverage field.

**Fix Direction:**

Replace references to 'UDP Length field' with 'Checksum Coverage field' in UDP-Lite contexts, particularly in Section 17.


**Severity:** Medium
  *Basis:* The misnaming introduces conceptual confusion about UDP-Lite's header structure, which may mislead implementers even though it does not directly affect core functionality.

**Confidence:** High

---