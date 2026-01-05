# Errata Reports

Total reports: 1

---


## Report 3: draft-ietf-dhc-rfc8415bis-12-7-3

**Label:** INF_MAX_RT Option Inconsistency Between Advertise Message Processing and Appendix B Table

**Bug Type:** Inconsistency

**Explanation:**

The normative text mandates that clients process the INF_MAX_RT option when it appears in Advertise messages, yet Appendix B’s summary table marks INF_MAX_RT as valid only in Reply messages, risking misinterpretation.

**Justification:**

- Section 18.2.9 explicitly requires that clients process the INF_MAX_RT option in Advertise messages.
- Appendix B’s table contradicts this by displaying INF_MAX_RT only in the context of Reply messages, which could mislead implementers using the table as a quick reference.

**Evidence Snippets:**

- **E1:**

  Table 1 defines INF_MAX_RT with default 3600 secs and describes it as a “Max Information-request timeout value”.

- **E2:**

  Section 18.2.9 says: “The client MUST process any SOL_MAX_RT option (see Section 21.24) and INF_MAX_RT option (see Section 21.25) present in an Advertise message, even if the message contains a Status Code option …”.

- **E3:**

  Appendix B, third table:

                   SOL_MAX_RT  INF_MAX_RT
           Solicit
           Advert.    *
           ...
           Reply      *           *

INF_MAX_RT is marked only for Reply, not Advert.

**Evidence Summary:**

- (E1) describes the INF_MAX_RT option and its default value.
- (E2) mandates processing of INF_MAX_RT in Advertise messages.
- (E3) shows that Appendix B’s table restricts INF_MAX_RT to Reply messages only.

**Fix Direction:**

Revise Appendix B to include INF_MAX_RT for Advertise messages or modify the normative text so that it aligns with the table summary.

**Severity:** Medium
  *Basis:* This inconsistency can lead to divergent interpretations, with some implementations possibly ignoring INF_MAX_RT in Advertise messages, thereby affecting protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- QuantitativeExpert: Issue-1

---