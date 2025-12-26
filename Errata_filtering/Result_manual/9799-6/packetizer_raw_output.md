# Errata Reports

Total reports: 4

---

## Report 1: 9799-6-1

**Label:** Directory Metadata Field Name Mismatch: inBandOnionCAARequired vs onionCAARequired

**Bug Type:** Inconsistency

**Explanation:**

The specification uses two different JSON field names for the same ACME directory metadata flag, potentially causing interoperability issues.

**Justification:**

- Section 6.4.1 defines the metadata field as inBandOnionCAARequired (including in the JSON example) while Section 7.3 registers the field as onionCAARequired in the IANA registry.
- This discrepancy can lead to clients and servers using different field names, resulting in a failure to properly signal or detect the in‐band CAA requirement.

**Evidence Snippets:**

- **E1:**

  Section 6.4.1: “To support signaling the server's support for fetching CAA record sets over Tor, a new field is defined in the directory ‘meta’ object to signal this.

inBandOnionCAARequired (optional, boolean): If true, the ACME server requires the client to provide the CAA record set in the finalize request. If false or absent, the ACME server does not require the client to provide the CAA record set is this manner.” And the example directory JSON uses "inBandOnionCAARequired": true.

- **E2:**

  Section 7.3: IANA registration table registers the field as:

| Field name       | Field type | Reference |
| onionCAARequired | boolean    | RFC 9799  |

**Evidence Summary:**

- (E1) Section 6.4.1 defines the metadata field as inBandOnionCAARequired with an example JSON.
- (E2) Section 7.3 registers the metadata field as onionCAARequired.

**Fix Direction:**

Align the naming by replacing 'inBandOnionCAARequired' with 'onionCAARequired' in Section 6.4.1 and the JSON example to match the IANA registration.


**Severity:** High
  *Basis:* The mismatch directly affects the discovery of server capabilities and may cause clients to miss the requirement for in‐band CAA, leading to interoperability and issuance errors.

**Confidence:** High

---

## Report 2: 9799-6-2

**Label:** Inconsistent Enforcement and Ordering of caa-critical Flag with In-band CAA

**Bug Type:** Inconsistency

**Explanation:**

The specification contains conflicting requirements for handling the caa-critical flag; it mandates that issuance must wait for decryption of second-layer CAA records while also allowing issuance based solely on an in-band CAA record set.

**Justification:**

- Section 6.3 mandates that if an ACME server encounters the caa-critical flag, it MUST NOT proceed with issuance until second-layer CAA is decrypted and parsed.
- Section 6.4 permits an ACME server to proceed with issuance based solely on a valid in-band CAA record set, even if the caa-critical flag is present, resulting in contradictory requirements.

**Evidence Snippets:**

- **E3:**

  Section 6.3: "If an ACME server encounters this flag, it MUST NOT proceed with issuance until it can decrypt and parse the CAA records from the Second Layer Hidden Service Descriptor."

- **E4:**

  Section 6.4: "If an ACME server receives a validly signed CAA record set in the finalize request, it MAY proceed with issuance on the basis of the client-provided CAA record set only, without checking the CAA set in the Hidden Service Descriptor."

**Evidence Summary:**

- (E3) Section 6.3 enforces waiting for second-layer CAA processing when the caa-critical flag is encountered.
- (E4) Section 6.4 allows issuance based solely on in-band CAA data, creating a conflict.

**Fix Direction:**

Clarify the protocol so that the caa-critical mandate always requires descriptor-based CAA processing, or restrict the in-band CAA shortcut to cases where the caa-critical flag is not present.


**Severity:** Medium
  *Basis:* The conflicting instructions can lead to divergent implementation behaviors, undermining the intended issuance safeguards and resulting in unpredictable certificate issuance.

**Confidence:** High

---

## Report 3: 9799-6-3

**Label:** Ambiguous In-band CAA Expiry Enforcement and Validity Window

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define how the expiry timestamp in the in-band CAA record set should be enforced, leading to ambiguity in handling stale or replayed CAA data.

**Justification:**

- Section 6.4 defines an expiry field for the in-band CAA record set that SHOULD NOT be more than 8 hours in the future, but offers no guidance on rejecting expired records or accounting for clock skew.
- This lack of normative guidance may result in inconsistent interpretations and security weaknesses across implementations.

**Evidence Snippets:**

- **E5:**

  Section 6.4, definition of expiry: "The Unix timestamp at which this CAA record set will expire. This SHOULD NOT be more than 8 hours in the future. ACME servers MUST process this as at least a 64-bit integer to ensure functionality beyond 2038."

- **E6:**

  Temporal Expert T2: "the document never states what the ACME server MUST or SHOULD do with this timestamp in relation to its current time... leaving key aspects of the CAA-checking timeline ambiguous."

**Evidence Summary:**

- (E5) Section 6.4 provides an expiry field without specifying enforcement rules.
- (E6) Temporal Expert points out the lack of guidance on handling the expiry timestamp.

**Fix Direction:**

Introduce explicit normative requirements for expiry validation, including rejecting records past their expiry, defining acceptable clock skew, and clarifying reuse conditions.


**Severity:** Medium
  *Basis:* Ambiguous expiry enforcement can lead to interoperability issues and security vulnerabilities by allowing stale CAA policies to be erroneously accepted or rejected.

**Confidence:** High

---

## Report 4: 9799-6-4

**Label:** Underspecified Keying of onionCAA for Subdomains and Wildcards

**Bug Type:** Underspecification

**Explanation:**

The document does not specify a canonical method for keying the onionCAA dictionary when ACME orders include subdomains or wildcard identifiers, resulting in potential mismatches between provided and expected CAA data.

**Justification:**

- Section 6.4 specifies that the onionCAA field is keyed by the ".onion" service name but does not clarify whether the key should be the base service name or the exact identifier when subdomains or wildcards are used.
- This ambiguity can lead to clients providing CAA data under one key format while servers expect another, causing unnecessary issuance failures.

**Evidence Snippets:**

- **E7:**

  Section 6.4: "A new field is defined in the ACME finalize endpoint to contain the Hidden Service's CAA record set for each ".onion" Special-Use Domain Name in the order. The key is the ".onion" Special-Use Domain Name, and the value is an object [...]"

- **E8:**

  Causal Expert Issue-2.3: "The spec never states whether the onionCAA dictionary key must be the base service name (e.g., bbcweb3....onion) or the exact ACME identifier (e.g., www.bbcweb3....onion or "*.bbcweb3....onion") when subdomains or wildcards are used."

**Evidence Summary:**

- (E7) Section 6.4 defines the onionCAA field as a dictionary keyed by the ".onion" name without further specification.
- (E8) Causal Expert highlights the ambiguity in keying when orders include subdomains or wildcards.

**Fix Direction:**

Specify that the onionCAA dictionary must be keyed by the base ".onion" service name, with all subdomains and wildcard identifiers mapping to that key.


**Severity:** Medium
  *Basis:* Insufficient keying specification can result in mismatches between client-supplied and server-expected CAA data, leading to unnecessary errors and interoperability problems.

**Confidence:** High

---
