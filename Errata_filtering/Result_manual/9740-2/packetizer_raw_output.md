# Errata Reports

Total reports: 1


---

## Report 2: 9740-2-2

**Label:** Stylistic Naming Inconsistency in IE Names: ipv6ExtensionHeadersChainLength vs ipv6ExtensionHeaderChainLengthList

**Bug Type:** Inconsistency

**Explanation:**

There is a minor stylistic inconsistency where one Information Element name includes an extra 's' ('ipv6ExtensionHeadersChainLength') compared to the other ('ipv6ExtensionHeaderChainLengthList'), even though their semantics remain clear.

**Justification:**

- The analysis notes that outside Section 2, the IE names differ by an extra 's' in 'Headers', representing a naming quirk rather than a substantive error (E3).

**Evidence Snippets:**

- **E3:**

  Elsewhere in the document (outside Section 2), the IE names `ipv6ExtensionHeadersChainLength` (ElementID 518) and `ipv6ExtensionHeaderChainLengthList` (ElementID 519) differ by an extra “s” in “Headers” vs. “Header”. The descriptions and IANA table consistently use these names, and their semantics are clear and distinct. This looks like a stylistic inconsistency in naming, not a conflicting or misleading definition, because the IEs are always referenced with their exact names and unique IDs, and an implementer cannot reasonably confuse one for the other.

**Evidence Summary:**

- (E3) The document displays a naming inconsistency where one IE name includes an extra 's' compared to the other, a minor stylistic issue noted in the expert analysis.

**Fix Direction:**

Consider aligning the naming convention to remove the extra 's' for clarity, though the current naming does not impact functionality.


---
