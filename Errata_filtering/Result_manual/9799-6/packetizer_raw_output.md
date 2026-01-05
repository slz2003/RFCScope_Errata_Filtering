# Errata Reports

Total reports: 2

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