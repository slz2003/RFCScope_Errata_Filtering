# Errata Reports

Total reports: 1

---

## Report 1: 9729-4-1

**Label:** Inconsistent ABNF for Integer Parameter 's' Excludes Single-Digit Non-Zero Values and Permits Out-of-Range Values

**Bug Type:** Inconsistency

**Explanation:**

The ABNF for the integer authentication parameter 's' enforces that non-zero values must be at least two digits, which conflicts with the prose stating that any integer from 0 to 65535 (including single-digit values) is allowed. This mismatch can result in interoperability differences.

**Justification:**

- The generic prose in Section 4 states that the integer is encoded without a minus and without leading zeroes, implying that a lone non-zero digit (1–9) is valid (E1).
- The ABNF definition 'concealed-integer-param-value =  %x31-39 1*4( DIGIT ) / "0"' forces non-zero integers to be written with at least two digits, thereby excluding values such as '1' through '9' (E2).
- The 's' parameter is defined as an integer between 0 and 65535, meaning single-digit values are within its valid range, while the ABNF also permits 5‑digit numbers that may exceed the allowed range, creating further ambiguity (E3).

**Evidence Snippets:**

- **E1:**

  The integer below is encoded without a minus and without leading zeroes. In other words, the value of this integer authentication parameter MUST NOT include any characters other than digits and MUST NOT start with a zero unless the full value is '0'.

- **E2:**

  concealed-integer-param-value =  %x31-39 1*4( DIGIT ) / "0"

- **E3:**

  The REQUIRED 's' (signature scheme) Parameter is an integer that specifies the signature scheme used ... Its value is an integer between 0 and 65535 inclusive from the IANA 'TLS SignatureScheme' registry…

**Evidence Summary:**

- (E1) Prose specifies an integer with no minus and no leading zeroes, implicitly allowing single-digit non-zero values.
- (E2) ABNF requires a leading digit from 1–9 followed by at least one additional DIGIT, disallowing single-digit values.
- (E3) The 's' parameter definition mandates a numeric range of 0–65535, which includes values like 1–9.

**Fix Direction:**

Adjust the ABNF to allow a non-zero digit to be followed by zero to four additional digits (for example, change '1*4(DIGIT)' to '*4(DIGIT)' or '0*4(DIGIT)'), so that single-digit non-zero integers are accepted while still enforcing no leading zeros.


**Severity:** Medium
  *Basis:* Interoperability issues may arise if implementations interpret the syntax differently, especially if future TLS SignatureScheme assignments include single-digit values or if out-of-range 5-digit numbers are accepted without extra checking.

**Confidence:** High

---
