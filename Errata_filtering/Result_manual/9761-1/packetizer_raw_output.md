# Errata Reports

Total reports: 1

---


## Report 1: 9761-1-1

**Label:** Mis-citation of MALWARE-TLS for Tor/Psiphon/Ultrasurf usage

**Bug Type:** Inconsistency

**Explanation:**

The RFC cites [MALWARE-TLS] as evidence that malware uses privacy-enhancing tools like Tor, Psiphon, and Ultrasurf, yet the referenced work does not specifically analyze these technologies, leading to a misleading bibliographic reference.

**Justification:**

- The Causal Expert analysis notes that Section 1 includes a bullet listing 'Tor, Psiphon, Ultrasurf (see [MALWARE-TLS])', although the summary of MALWARE-TLS indicates that it does not provide evidence for malware using these tools (E1).
- The CrossRFC Expert analysis further explains that readers expecting empirical evidence from [MALWARE-TLS] will be misled, since the paper only addresses general TLS usage beyond browsers without discussing these specific privacy tools (E2).

**Evidence Snippets:**

- **E1:**

  Using privacy enhancing technologies like Tor, Psiphon, Ultrasurf (see [MALWARE-TLS]), and evasion techniques such as ClientHello randomization.

- **E2:**

  Section 1 states that malware uses “privacy enhancing technologies like Tor, Psiphon, Ultrasurf (see [MALWARE-TLS ])”, explicitly using the MALWARE-TLS reference (Anderson & McGrew, “TLS Beyond the Browser”) as support for that specific claim. However, the provided summary of MALWARE-TLS says the paper focuses on general TLS usage patterns beyond browsers and does not provide specific information or analysis on these tools (Tor, Psiphon, Ultrasurf).

**Evidence Summary:**

- (E1) The Section 1 bullet cites [MALWARE-TLS] for the use of Tor, Psiphon, and Ultrasurf, despite the reference not supporting this claim.
- (E2) The CrossRFC analysis highlights that the MALWARE-TLS paper does not discuss these specific privacy technologies, showing a mis-citation.

**Fix Direction:**

Replace [MALWARE-TLS] with a reference that actually documents malware use of Tor, Psiphon, and Ultrasurf, or remove the parenthetical citation to avoid misleading readers.


**Severity:** Low
  *Basis:* The issue is bibliographic and does not impact protocol operation or security; it only affects evidentiary accuracy and scholarly correctness.

**Confidence:** Medium

---