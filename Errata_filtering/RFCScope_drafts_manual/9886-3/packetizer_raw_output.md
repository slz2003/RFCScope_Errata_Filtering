# Errata Reports

Total reports: 6

---

## Report 1: 9886-3-1

**Label:** Underspecified actor responsibility for DET reverse resolution in DNS

**Bug Type:** Underspecification

**Explanation:**

The normative requirement that DET reverse names MUST resolve to an HHIT RRType is attached to the identifiers without specifying which operator is responsible, creating ambiguity over DNS record management.

**Justification:**

- The text states: DETs, being IPv6 addresses, are to be under ip6.arpa. (nibble reversed per Section 2.5 of RFC 3596 [STD88]) and MUST resolve to an HHIT RRType.
- The normative statement does not assign responsibility to a specific party (e.g., HDA/RAA registry, DIME, or registrar), leading to an underspecification.

**Evidence Snippets:**

- **E1:**

  DETs, being IPv6 addresses, are to be under ip6.arpa. (nibble reversed per Section 2.5 of RFC 3596 [STD88]) and MUST resolve to an HHIT RRType.

- **E2:**

  In the surrounding text, several distinct actors are described: registrants (end users), registrars (agents), registries (zone operators) and DIMEs that “manage registration of and associated lookups from DETs”, with RAAs/HDAs mapped onto “National” and “Local” registries… the requirement is attached to the identifier rather than a specific party.

**Evidence Summary:**

- (E1) states the requirement without specifying an operator
- (E2) explains that the obligation is not explicitly assigned.

**Fix Direction:**

Clarify in the text which entity (e.g., HDA/RAA registry, DIME, or registrar) is responsible for publishing and maintaining the required HHIT RR records.

**Severity:** Medium
  *Basis:* Lack of explicit responsibility can lead to inconsistent implementations and operational confusion.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: NewIssue-1

---

## Report 2: 9886-3-2

**Label:** Implicit shorthand notation for RAA/HDA prefixes lacks explicit bit-level definition

**Bug Type:** Underspecification

**Explanation:**

The shorthand notations for RAAs (2001:3x:xxx0::/44) and HDAs (2001:3x:xxxy:yy00::/56) are presented without explicit bit-level details, forcing implementers to infer the intended mapping from external specifications.

**Justification:**

- The document notes: The shorthand address forms “2001:3x:xxx0::/44” (RAA) and “2001:3x:xxxy:yy00::/56” (HDA) are not formally defined in bit‑level terms here.
- Residual uncertainties mention a reliance on an implicit understanding of how RAA/HDA bits interleave with the DET prefix.

**Evidence Snippets:**

- **E1:**

  The shorthand address forms “2001:3x:xxx0::/44” (RAA) and “2001:3x:xxxy:yy00::/56” (HDA) are not formally defined in bit‑level terms here, and the meaning of the placeholder nibbles x and y has to be inferred from RFC 9374’s HID layout and DET encoding example.

- **E2:**

  ResidualUncertainties: The shorthand forms 2001:3x:xxx0::/44 and 2001:3x:xxxy:yy00::/56 rely on an implicit understanding of how RAA/HDA bits interleave with the DET prefix.

**Evidence Summary:**

- (E1) highlights lack of explicit bit-level definition for shorthand prefixes
- (E2) notes the reliance on implicit understanding.

**Fix Direction:**

Include a detailed bit-level derivation of the RAA/HDA prefixes directly in the document.

**Severity:** Low
  *Basis:* While this ambiguity may increase cognitive load for implementers, referencing external specifications limits the risk of interoperability issues.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: NotedAmbiguities
- Scope Expert: ResidualUncertainties

---

## Report 3: 9886-3-3

**Label:** Cross-RFC DNS model tension between RFC 9886 and RFC 9374

**Bug Type:** Causal Underspecification

**Explanation:**

RFC 9886 mandates ip6.arpa reverse mapping and introduces a new HHIT RRType, while RFC 9374 allows for a more permissive DNS model, creating potential interoperability risks due to cross-document tension.

**Justification:**

- The causal analysis notes a cross‑RFC tension with RFC 9374’s earlier, more permissive DNS-model text.
- The text explains that deployments adhering to RFC 9374’s model (e.g., using the hhit.arpa FQDN) may be non‑interoperable with RFC 9886-compliant systems.

**Evidence Snippets:**

- **E1:**

  The main thing to be aware of is a cross‑RFC tension with RFC 9374’s earlier, more permissive DNS-model text; that’s a policy/interoperability choice rather than a mechanical bug in the Section 3 mapping itself.

- **E2:**

  A deployment that followed RFC 9374 early and chose only the hhit.arpa FQDN model, ignoring ip6.arpa, would be non‑interoperable with 9886‑compliant resolvers that expect reverse ip6.arpa zones.

**Evidence Summary:**

- (E1) notes the cross-RFC tension regarding DNS model choices
- (E2) explains the potential non‑interoperability arising from legacy implementations.

**Fix Direction:**

Include an explicit statement clarifying that the ip6.arpa reverse mapping model in RFC 9886 is the required interoperable DNS model, thereby superseding the alternative approach referenced in RFC 9374.

**Severity:** Medium
  *Basis:* The conflicting DNS models can lead to divergent implementations and interoperability issues across deployments.

**Confidence:** High

**Experts mentioning this issue:**

- Causal Expert: Section 2.4
- CrossRFC Expert: Issue-1

---

## Report 4: 9886-3-4

**Label:** Misattribution of HHIT records to RFC 9374 instead of RFC 9886

**Bug Type:** Inconsistency

**Explanation:**

The document incorrectly cites RFC 9374 as the source for HHIT records, even though the HHIT RRType is defined in RFC 9886, which may mislead implementers regarding the proper DNS record format.

**Justification:**

- Structural analysis notes that the text states 'They would also provision the HHIT records [RFC9374] for these IP addresses', implying that RFC 9374 defines the HHIT record format, which it does not.
- CrossRFC and Terminology analyses confirm that the correct definition is in Section 5.1 of RFC 9886.

**Evidence Snippets:**

- **E1:**

  For DRIP, blocks of IP addresses could be delegated from the 3.0.0.1.0.0.2.ip6.arpa. domain … They would also provision the HHIT records [RFC9374] for these IP addresses.

- **E2:**

  The HHIT Resource Record (HHIT, RRType 67) is a metadata record… defined in Section 5.1 of this document.

**Evidence Summary:**

- (E1) shows the misattributed reference to RFC 9374 for HHIT records
- (E2) clarifies that the HHIT RRType is defined in RFC 9886.

**Fix Direction:**

Change the reference in Section 3.1 to point to Section 5.1 of RFC 9886 for the HHIT RRType definition instead of RFC 9374.

**Severity:** Medium
  *Basis:* Incorrect cross-referencing may lead implementers to use the wrong specification, thus causing interoperability issues.

**Confidence:** High

**Experts mentioning this issue:**

- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1
- Terminology Expert: Issue-2

---

## Report 5: 9886-3-5

**Label:** Double trailing dot in ip6.arpa reverse domain names

**Bug Type:** Inconsistency

**Explanation:**

Some ip6.arpa reverse domain names are rendered with an extra trailing dot, which may be interpreted as an invalid fully qualified domain name.

**Justification:**

- Terminology analysis highlights instances such as '3.0.0.1.0.0.2.ip6.arpa..' that contain two trailing dots.
- This inconsistent representation contrasts with other parts of the document that correctly use a single trailing dot.

**Evidence Snippets:**

- **E1:**

  Its corresponding domain name for reverse lookups is 3.0.0.1.0.0.2.ip6.arpa.. The IAB has administrative control of this domain name.

- **E2:**

  with respective nibble reverse domains of y.y.y.x.x.x.x.3.0.0.1.0.0.2.ip6.arpa..

**Evidence Summary:**

- (E1) indicates an extra trailing dot in the reverse lookup domain
- (E2) shows a similar extra dot in the nibble reverse domain.

**Fix Direction:**

Replace double trailing dots with a single trailing dot (e.g., '3.0.0.1.0.0.2.ip6.arpa.') consistently throughout the document.

**Severity:** Medium
  *Basis:* Incorrect domain name formatting can lead to errors in DNS configurations.

**Confidence:** High

**Experts mentioning this issue:**

- Terminology Expert: Issue-1

---

## Report 6: 9886-3-6

**Label:** Ambiguous use of the term 'apex'

**Bug Type:** Ambiguity/Terminology

**Explanation:**

The term 'apex' is used ambiguously to refer both to the DNS apex and the DET/PKI apex, which may require cross-referencing with external documents to resolve its intended meaning.

**Justification:**

- Scope analysis mentions that 'apex' is used both for the DNS apex 3.0.0.1.0.0.2.ip6.arpa. (operated by IANA/IAB) and for the DET/PKI 'Apex' entity in other DRIP documents, potentially confusing implementers.

**Evidence Snippets:**

- **E1:**

  The term “apex” is used both for the DNS apex 3.0.0.1.0.0.2.ip6.arpa. (operated by IANA/IAB) and, in other DRIP documents, for the DET/PKI “Apex” entity; section 3 briefly mentions “Apex, RAA or HDA” DIMEs without explicitly re‑defining “apex”.

**Evidence Summary:**

- (E1) highlights the ambiguous usage of the term 'apex'.

**Fix Direction:**

Clarify the nomenclature by defining 'DNS apex' and 'DET apex' distinctly or use alternative unambiguous terminology.

**Severity:** Low
  *Basis:* While primarily affecting clarity and documentation, the ambiguity may lead to minor confusion for implementers.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: NotedAmbiguities

---
