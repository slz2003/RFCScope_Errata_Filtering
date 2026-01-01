# Errata Reports

Total reports: 1

---

## Report 1: 9748-1-1

**Label:** Wrong / over‑broad IANA location for NTS registries

**Bug Type:** Inconsistency

**Explanation:**

The document inaccurately states that all NTS registries are located at https://www.iana.org/assignments/nts, whereas the correct dedicated NTS registry page is at nts-parameters and some NTS-related values reside in generic IANA registries.

**Justification:**

- The analysis explains that the claim is doubly inaccurate by both pointing to the wrong URL and implying that all NTS registry actions occur at that location (E1).

**Evidence Snippets:**

- **E1:**

  The introduction claims that “the NTS registries can all be found at <https://www.iana.org/assignments/nts>”. RFC 8915 creates several *NTS‑specific* registries (“Network Time Security Key Establishment Record Types”, “Network Time Security Next Protocols”, “Network Time Security Error Codes”, and “Network Time Security Warning Codes”) and also allocates NTS‑related values in generic IANA registries (service port, ALPN ID, TLS exporter label, NTP Extension Field Types and Kiss‑o'-Death Codes) in Sections 7.1–7.5. Those generic registries are not hosted under a dedicated NTS page at all; they live on their own, long‑standing registry pages (Service Name and Port, ALPN Protocol IDs, TLS Exporter Labels, NTP Parameters). Separately, IANA’s convention for protocol‑specific parameter pages is to use a “‑parameters” slug (e.g., `ntp-parameters`, `tls-parameters`), and the NTS registries defined in RFC 8915 are grouped under a “Network Time Security (NTS) Parameters” page whose canonical path is `nts-parameters`, not a bare `/nts`. As a result, the statement “can all be found at … /nts” is doubly inaccurate: it (a) points to the wrong URL for the NTS parameter page and (b) suggests all NTS‑related registry actions reside there, which is not true for the allocations made into existing generic registries. An implementer who follows the text literally may either hit a non‑existent page or overlook the generic registries where some NTS values actually live, causing confusion when trying to locate or verify codepoints. This is best corrected by changing the URL to the actual NTS parameter page and softening the wording to clarify that only the dedicated NTS parameter registries (not all NTS‑related allocations) are on that page.

**Evidence Summary:**

- (E1) The text claims that all NTS registries are at https://www.iana.org/assignments/nts, but clarifies that the dedicated NTS page is actually at nts-parameters and that some allocations appear in separate generic registries.

**Fix Direction:**

Change the URL to the dedicated NTS parameters page (nts-parameters) and modify the wording to indicate that only dedicated registries are listed there.

**Severity:** Medium
  *Basis:* The potential for implementer confusion due to referencing an incorrect URL and over-broad registry inclusion leads to a medium severity rating.

**Confidence:** Medium

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1

---
