# Errata Reports

Total reports: 1

---

## Report 1: 9762-11-1

**Label:** Malformed Sentence and Oversimplified 'Enable DHCPv6' Wording in Section 11

**Bug Type:** Inconsistency

**Explanation:**

Section 11 of RFC 9762 contains a grammatically malformed sentence that ambiguously states that enabling P flag support will 'enable DHCPv6' on a host, which may mislead implementers about when DHCPv6-PD is triggered.

**Justification:**

- The sentence 'Implementing the P flag support on a host and receiving will enable DHCPv6 on that host if the host receives an RA containing a PIO with the P bit set.' is grammatically malformed with the stray 'and receiving' and creates an oversimplified description compared to the normative text in Section 7.1 (E1).
- The phrasing uses 'will enable DHCPv6' even though the normative behavior is defined with a SHOULD in Section 7.1, potentially misleading about the actual operational behavior (E1, E2).

**Evidence Snippets:**

- **E1:**

  Implementing the P flag support on a host and receiving will enable DHCPv6 on that host if the host receives an RA containing a PIO with the P bit set.

- **E2:**

  Sending DHCPv6 packets may reveal some minor additional information about the host, most prominently the hostname. This is not a new concern and would apply for any network that uses DHCPv6 and sets the M flag in RAs.

**Evidence Summary:**

- (E1) The malformed sentence in Section 11 creates ambiguity in how DHCPv6-PD is triggered.
- (E2) The oversimplified privacy wording may misrepresent the true normative behavior specified elsewhere.

**Fix Direction:**

Revise the problematic sentence to clearly state that implementing P flag support will cause the host to engage in DHCPv6-PD per the normative instructions in Section 7.1; for example, replace it with: 'Implementing support for the P flag on a host will cause the host to start using DHCPv6-PD when it receives an RA containing a PIO with the P bit set, as described in Section 7.1.'


---
