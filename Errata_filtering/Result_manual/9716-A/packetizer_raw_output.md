# Errata Reports

Total reports: 1

---


## Report 3: 9716-A-3

**Label:** Inconsistent TLV inclusion requirements in dynamic return-path echo replies

**Bug Type:** Both

**Explanation:**

There is a discrepancy between the normative mandate that dynamic return-path echo replies include a Reply Path TLV with a Return Code of 0x0006 and examples that allow certain internal nodes to omit the TLV.

**Justification:**

- Section 5.3 requires that when performing dynamic return-path building a TLV (or a new one) must be included with a Return Code of 0x0006 (see E8).
- Section 5.5.1 and Appendix A.1.3 suggest that internal nodes may omit setting the TLV or its return code under certain conditions (see E9 and E10).

**Evidence Snippets:**

- **E8:**

  Section 5.3: “When the node is configured to dynamically create a return path for the next echo request, the procedures described in Section 5.5 MUST be used. The Reply Path Return Code MUST be set to 0x0006, and the same Reply Path TLV or a new Reply Path TLV MUST be included in the echo reply.”

- **E9:**

  Section 5.5.1: “Internal nodes or non-domain border nodes might not set the Reply Path TLV Return Code to 0x0006 in the echo reply message as there is no change in the return path.”

- **E10:**

  Appendix A.1.3: Dynamic example: after P1 returns an echo reply with Return Code 0x0006 and a Reply Path TLV, it says: “This traceroute doesn't need any changes to the Reply Path TLV until it leaves AS1. The same Reply Path TLV that is received may be included in the echo reply by P1 and P2, or no Reply Path TLV is included so that the head-end continues to use the same return path in the echo request.”

**Evidence Summary:**

- (E8) Normative provision from Section 5.3 mandates inclusion of a Reply Path TLV with Return Code 0x0006.
- (E9) Section 5.5.1 notes that internal nodes might not set the return code.
- (E10) Appendix A.1.3 permits omission of the TLV in some echo replies.

**Fix Direction:**

Clarify the dynamic return-path procedure by explicitly specifying which nodes must include the Reply Path TLV and under what conditions omission is acceptable.


**Severity:** High
  *Basis:* The lack of clear guidance can lead to different implementations treating echo replies differently, resulting in interoperability issues.

**Confidence:** High

---