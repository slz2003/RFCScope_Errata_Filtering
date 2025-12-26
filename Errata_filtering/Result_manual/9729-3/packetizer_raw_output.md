# Errata Reports

Total reports: 1

---

## Report 1: 9729-3-1

**Label:** Mismatch in Context String: 'HTTP Concealed Authentication' vs 'HTTP Signature Authentication'

**Bug Type:** Inconsistency

**Explanation:**

The specification normatively defines the context string as 'HTTP Concealed Authentication' in Section 3.3, yet the hex example in Figure 3 encodes 'HTTP Signature Authentication', which can lead to interoperability issues if implementers copy the example verbatim.

**Justification:**

- The normative text in Section 3.3 mandates the context string to be 'HTTP Concealed Authentication' (E1, E3) while the hex example provides a different string (E2, E4).
- Experts from multiple areas (causal, structural, terminology) have noted that this discrepancy may cause different implementations to use inconsistent constants, risking signature verification failures.

**Evidence Snippets:**

- **E1:**

  Section 3.3 defines the signature input as the concatenation of: “A string that consists of octet 32 (0x20) repeated 64 times; The context string `HTTP Concealed Authentication`; A single 0 byte …; The Signature Input extracted from the key exporter output.”

- **E2:**

  In Figure 3, after the 64 bytes of `0x20` and before the `00` separator, the ASCII portion of the hex is: 48545450205369676E61747572652041757468656E7469636174696F6E, which decodes to `HTTP Signature Authentication`, not `HTTP Concealed Authentication`.

- **E3:**

  Normative prose in Section 3.3: “The signature is computed over the concatenation of: … * The context string "HTTP Concealed Authentication" …”

- **E4:**

  Figure 3 hex example of “content covered by the signature” (excerpted middle lines): 48545450205369676E61747572652041757468656E7469636174696F6E

**Evidence Summary:**

- (E1) The normative definition in Section 3.3 establishes the expected context string.
- (E2) The worked hex example in Figure 3 decodes to a different context string.
- (E3) Terminology analysis confirms the normative string is meant to be 'HTTP Concealed Authentication'.
- (E4) The hex example provided does not match the normative definition.

**Fix Direction:**

Clarify in the specification that the normative context string is 'HTTP Concealed Authentication' and update Figure 3’s hex example so its ASCII portion encodes that exact string.


**Severity:** Medium
  *Basis:* Although the underlying cryptographic mechanism remains sound, the discrepancy may cause interoperability failures if implementers mistakenly use the hex example, leading to mismatched signature inputs.

**Confidence:** High

---
