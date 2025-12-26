# Errata Reports

Total reports: 2

---

## Report 1: 9868-18-1

**Label:** Ambiguity in UDP Length Permissibility Wording (Section 18 vs ROHC Requirements)

**Bug Type:** Ambiguity

**Explanation:**

Section 18’s statement that it has 'always been permissible' for the UDP Length to be inconsistent with the IP payload length may mislead implementers by implying universal permissiveness, which contrasts with the ROHC profile’s 'MUST match' requirement.

**Justification:**

- Section 18 explicitly states 'It has always been permissible for the UDP Length to be inconsistent with the IP transport payload length [RFC0768].' (E1)
- The RouterSketch assessment notes that the hypothesis focused on whether 'always been permissible' is too strong given ROHC’s 'MUST match' wording. (E2)

**Evidence Snippets:**

- **E1:**

  RFC 9868 Section 18 states, “It has always been permissible for the UDP Length to be inconsistent with the IP transport payload length [RFC0768].” There is no text in the base UDP spec (RFC 768) or the host requirements (RFC 1122) that requires the UDP Length to equal the IP payload length; RFC 1122 even explicitly notes there are “no known errors in the specification of UDP” .

- **E2:**

  RouterSketch assessment: The router’s hypothesis focused on whether “always been permissible” is too strong given ROHC’s “MUST match” wording, and whether the embedded device comment is fully supported.

**Evidence Summary:**

- (E1) RFC 9868 Section 18 permissiveness clause regarding UDP Length.
- (E2) RouterSketch assessment questioning the strength of the 'always been permissible' wording relative to ROHC requirements.

**Fix Direction:**

Revise the wording in Section 18 to clarify that the permissibility of a UDP Length and IP payload length mismatch is context-specific and does not override specific requirements such as those in ROHC profiles.


**Severity:** Low
  *Basis:* The issue is principally an editorial ambiguity that could mislead implementers, though it does not constitute a normative error.

**Confidence:** High

---

## Report 2: 9868-18-2

**Label:** Ambiguity in Embedded Device Comment Regarding Full IP Datagram Delivery

**Bug Type:** Ambiguity

**Explanation:**

The document comments that an embedded device passing the entire IP datagram to the UDP application layer is inconsistent with the UDP application interface, but this is ambiguous since RFC 1122 does not explicitly forbid exposing an additional raw datagram API.

**Justification:**

- Section 18 describes a device that “passes the entire IP datagram to the UDP application layer” and notes this as inconsistent with the UDP application interface. (E1)
- The analysis clarifies that there is no explicit prohibition in RFC 1122 against a host OS offering both a standard UDP receive API and an additional raw datagram API. (E2)

**Evidence Snippets:**

- **E1:**

  Section 18 describes a device that “passes the entire IP datagram to the UDP application layer” and says this “feature is also inconsistent with the UDP application interface [RFC0768] [RFC1122].”

- **E2:**

  There is no explicit prohibition in RFC 1122 against a host OS offering an additional, raw “receive the entire IP datagram” API alongside the normal UDP receive interface.

**Evidence Summary:**

- (E1) Section 18's description of an embedded device delivering the full IP datagram to the application layer.
- (E2) Analysis noting that RFC 1122 does not expressly forbid an alternative raw datagram API.

**Fix Direction:**

Clarify the embedded device comment in Section 18 to distinguish normative UDP API requirements from optional raw datagram delivery features, ensuring consistency with RFC 1122.


**Severity:** Low
  *Basis:* The issue arises from ambiguous wording that may cause interpretative confusion but does not conflict with explicit normative specifications.

**Confidence:** High

---
