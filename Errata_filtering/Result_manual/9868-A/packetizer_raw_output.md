# Errata Reports

Total reports: 1

---

## Report 1: 9868-A-1

**Label:** Inconsistent socket option naming in Appendix A, Table 3 (spaces vs underscores)

**Bug Type:** Inconsistency

**Explanation:**

Some socket option identifiers in Table 3 are rendered with embedded spaces (e.g., UDP OPT MDS) while others use underscores (e.g., UDP_OPT_MDS), which undermines the goal of consistent naming and may mislead implementers.

**Justification:**

- Structural Expert analysis notes that while most entries follow the UDP_OPT_* pattern, several entries use spaces, creating an inconsistency in the same table (E1).
- Terminology Expert analysis reinforces that the mixed usage between underscore-based and space-containing names contradicts both the intended style and the corresponding sysctl table entries, indicating a typographical error (E2).

**Evidence Snippets:**

- **E1:**

  In Table 3, some entries are given as UDP_OPT, UDP_OPT_OCS, UDP_OPT_APC, UDP_OPT_FRAG, UDP_OPT_TIME, UDP_OPT_UCMP, UDP_OPT_UENC. In the same table, other entries are written with embedded spaces: UDP OPT MDS, UDP OPT MRDS, UDP OPT REQ, UDP OPT RES, UDP OPT AUTH, UDP OPT EXP, UDP OPT UEXP. Earlier in Table 2 all sysctl names consistently use underscores, e.g., net.ipv4.udp_opt_mds, net.ipv4.udp_opt_mrds, etc.

- **E2:**

  Appendix A, socket options table (Table 3):

          | Name         | Meaning                     |
          ...
          | UDP_OPT      | Enable UDP Options (at all) |
          ...
          | UDP_OPT_OCS  | Use UDP OCS                 |
          ...
          | UDP_OPT_APC  | Enable UDP APC Option       |
          ...
          | UDP_OPT_FRAG | Enable UDP fragmentation    |
          +--------------+-----------------------------+
          | UDP OPT MDS  | Enable UDP MDS Option       |
          +--------------+-----------------------------+
          | UDP OPT MRDS | Enable UDP MRDS Option      |
          +--------------+-----------------------------+
          | UDP OPT REQ  | Enable UDP REQ Option       |
          +--------------+-----------------------------+
          | UDP OPT RES  | Enable UDP RES Option       |
          ...
          | UDP_OPT_TIME | Enable UDP TIME Option      |
          +--------------+-----------------------------+
          | UDP OPT AUTH | Enable UDP AUTH Option      |
          +--------------+-----------------------------+
          | UDP OPT EXP  | Enable UDP EXP Option       |
          ...
          | UDP_OPT_UCMP | Enable UDP UCMP Option      |
          +--------------+-----------------------------+
          | UDP_OPT_UENC | Enable UDP UENC Option      |
          +--------------+-----------------------------+
          | UDP OPT UEXP | Enable UDP UEXP Option      |

**Evidence Summary:**

- (E1) Structural Expert highlights that Table 3 mixes underscore-based names with space-separated names, contrasting with the consistent sysctl names in Table 2.
- (E2) Terminology Expert provides the full excerpt from Appendix A, Table 3, showing the inconsistency between entries such as UDP_OPT_* and UDP OPT *.

**Fix Direction:**

Clarify and normalize the socket option names in Table 3 so that all entries follow the underscore-based naming. Replace 'UDP OPT MDS' with 'UDP_OPT_MDS', 'UDP OPT MRDS' with 'UDP_OPT_MRDS', 'UDP OPT REQ' with 'UDP_OPT_REQ', 'UDP OPT RES' with 'UDP_OPT_RES', 'UDP OPT AUTH' with 'UDP_OPT_AUTH', 'UDP OPT EXP' with 'UDP_OPT_EXP', and 'UDP OPT UEXP' with 'UDP_OPT_UEXP'.


**Severity:** Medium
  *Basis:* The issue is primarily editorial affecting API clarity; while the inconsistency may mislead implementers, it does not impact on‐the‐wire interoperability. Terminology Expert explicitly rated it as Medium, and Structural Expert’s high likelihood relates to the consistency goal rather than critical operational failure.

**Confidence:** High

**Severity:** Medium
  *Basis:* The issue is primarily editorial affecting API clarity; while the inconsistency may mislead implementers, it does not impact on‐the‐wire interoperability. Terminology Expert explicitly rated it as Medium, and Structural Expert’s high likelihood relates to the consistency goal rather than critical operational failure.

**Confidence:** High


---
