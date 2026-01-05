# Errata Reports

Total reports: 2

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