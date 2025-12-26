# Errata Reports

Total reports: 1

---

## Report 1: 9870-3-1

**Label:** Over‐generalized ExID size statement in RFC 9870 Section 3

**Bug Type:** Inconsistency

**Explanation:**

RFC 9870 states that only 16‑bit ExIDs are supported in [RFC9868], which may mislead readers into thinking that 32‑bit ExIDs are not supported at all, even though RFC 9868 allows 32‑bit ExIDs for TCP options.

**Justification:**

- RFC 9870 Section 3 includes the statement 'Only 16‑bit ExIDs are supported in [RFC9868]', which can be interpreted too broadly (E1).
- RFC 9868’s IANA considerations clarify that while UDP options use 16‑bit ExIDs, 32‑bit ExIDs are available for TCP, with UDP options constrained to 16‑bit values (E2).

**Evidence Snippets:**

- **E1:**

  Section 3 of RFC 9870:
“For both options, Experiment Identifiers (ExIDs) are used to differentiate concurrent use of these options. Known ExIDs are expected to be registered within IANA. … Also, Section 4.5 specifies a new IPFIX IE to export observed ExIDs in the UEXP Options. Only 16‑bit ExIDs are supported in [RFC9868].”

- **E2:**

  RFC 9868, Section 26 (IANA Considerations), as provided:
“16‑bit ExIDs can be used with either TCP or UDP; 32‑bit ExIDs can be used with TCP or their first 16 bits can be used with UDP. … TCP/UDP ExIDs used for UDP are always 16 bits because their use in EXP and UEXP Options is required …”

**Evidence Summary:**

- (E1) RFC 9870 Section 3 states the limitation to 16‑bit ExIDs.
- (E2) RFC 9868 clarifies that while UDP options are limited to 16‑bit ExIDs, 32‑bit ExIDs are supported for TCP.

**Fix Direction:**

In Section 3 of RFC 9870, rephrase the statement to clarify that only UDP Options use 16‑bit ExIDs. For example: 'For UDP Options, [RFC9868] defines the use of only 16‑bit ExIDs (even though the shared TCP/UDP ExID registry also supports 32‑bit ExIDs for TCP options).'



**Severity:** Low
  *Basis:* The issue is editorial; it may cause misinterpretation of ExID scope but does not affect the correct encoding or interoperability of UDP Options.

**Confidence:** High
