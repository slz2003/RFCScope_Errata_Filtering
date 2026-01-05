# Errata Reports

Total reports: 1

---


## Report 2: 9799-7-2

**Label:** Ambiguous environmental scope of directory metadata flag (inBandOnionCAARequired)

**Bug Type:** Underspecification

**Explanation:**

The introductory text suggests that the flag signals the server's support for fetching CAA record sets over Tor, but its formal definition dictates that a true value requires the client to supply CAA data in-band, causing ambiguity about its intended use.

**Justification:**

- Section 6.4.1 introduces the flag with wording that implies it signals Tor-based fetching capability, yet defines the field so that a true value means the server mandates in-band CAA data.
- This conflict in description may lead to different interpretations by implementers, potentially resulting in misconfiguration.

**Evidence Snippets:**

- **E3:**

  Section 6.4.1 introduces the field with:
“To support signaling the server's support for fetching CAA record sets over Tor, a new field is defined in the directory "meta" object to signal this.”
But the field itself is defined as:
 inBandOnionCAARequired (optional, boolean): If true, the ACME server requires the client to provide the CAA record set in the finalize request. If false or absent, the ACME server does not require the client to provide the CAA record set in this manner.
Earlier in 6.4.1: “If an ACME server does not support fetching a service's CAA record set from its Hidden Service Descriptor, and the ACME client does not provide an "onionCAA" object in its finalize request, the ACME server MUST respond with an "onionCAARequired" error…”

**Evidence Summary:**

- (E3) Section 6.4.1 uses contradictory language by first suggesting the flag supports Tor-based fetching and then defining it such that a true value requires in-band CAA, leading to ambiguity.

**Fix Direction:**

Clarify the flag's intended scope by either revising the introductory text to indicate that a true value denotes an in-band requirement or by adjusting the definition to consistently indicate support for Tor-based fetching.


**Severity:** Medium
  *Basis:* The conflicting descriptions may cause confusion among implementers, potentially resulting in divergent behaviors, though the risk is less immediately severe than the naming mismatch.

**Confidence:** High

---