# Errata Reports

Total reports: 3

---

## Report 1: 9905-5-1

**Label:** Inconsistency between registry 'Use for DNSSEC Validation: RECOMMENDED' and operational mandate for RSASHA1/RSASHA1‑NSEC3‑SHA1

**Bug Type:** Inconsistency

**Explanation:**

The IANA registry entry for RSASHA1 and RSASHA1‑NSEC3‑SHA1 specifies 'Use for DNSSEC Validation: RECOMMENDED', implying these algorithms should be actively used by validators, while RFC 9905 mandates that operators treat them as unsupported and render responses insecure.

**Justification:**

- Multiple expert analyses note that the registry’s RECOMMENDED value (a positive validation recommendation) conflicts with the operational requirement to treat the algorithms as unsupported/insecure.
- This divergence can lead to inconsistent security behavior across implementations that rely solely on registry guidance versus those following the operational text.

**Evidence Snippets:**

- **E1:**

  RFC 9904, Section 2.1: ‘Use for DNSSEC Validation: Indicates the recommendation for using the algorithm in DNSSEC validators.’

- **E2:**

  RFC 9905 Section 2: ‘Operators of validating resolvers MUST treat DNSSEC signing algorithms RSASHA1 and RSASHA1‑NSEC3‑SHA1 as unsupported, rendering responses insecure if they cannot be validated by other supported signing algorithms.’

- **E3:**

  RFC 9905 Section 5, RSASHA1 row: ‘Use for DNSSEC Validation: RECOMMENDED’

**Evidence Summary:**

- (E1) RFC 9904 defines the validation column as a recommendation for using the algorithm in DNSSEC validators.
- (E2) RFC 9905 mandates that these algorithms be treated as unsupported for security, despite continued implementation.
- (E3) The registry update in Section 5 retains a RECOMMENDED value, creating a direct conflict.

**Fix Direction:**

Update the IANA registry entries for RSASHA1 and RSASHA1‑NSEC3‑SHA1 to change 'Use for DNSSEC Validation' from RECOMMENDED to a lower-strength value (e.g., NOT RECOMMENDED or MUST NOT) that aligns with the operational mandate.

**Severity:** High
  *Basis:* The conflicting guidance risks divergent DNSSEC security assessments, undermining uniform operational behavior.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionality Expert: Issue-1
- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1
- Terminology Expert: Issue-1

---

## Report 2: 9905-5-2

**Label:** Ambiguous deprecation scope for SHA‑1 algorithms (signing vs DS digest)

**Bug Type:** Underspecification

**Explanation:**

The document’s broad reference to 'SHA‑1–based algorithms' creates ambiguity by conflating the deprecation of RSASHA1/RSASHA1‑NSEC3‑SHA1 signing algorithms with the unchanged validation status of the SHA‑1 DS digest algorithm.

**Justification:**

- The RFC narrative broadly criticizes SHA‑1 usage in RRSIG and DS records yet only enforces stricter operational requirements on the signing algorithms.
- Registry values for the SHA‑1 DS digest remain unchanged, leaving operators uncertain if DS digest validation is likewise deprecated.

**Evidence Snippets:**

- **E1:**

  RFC 9905 §1: notes that DNSSEC 'made extensive use of SHA‑1, for example, as a cryptographic hash algorithm in … RRSIG and DS records' and concludes: 'This document thus deprecates the use of RSASHA1 and RSASHA1‑NSEC3‑SHA1 for DNS Security Algorithms.'

- **E2:**

  RFC 9905 §5 restates the SHA‑1 (1) Digest Algorithms row with: 'Use for DNSSEC Delegation: MUST NOT; Use for DNSSEC Validation: RECOMMENDED; Implement for DNSSEC Delegation: MUST NOT; Implement for DNSSEC Validation: MUST'

**Evidence Summary:**

- (E1) The introduction broadly deprecates SHA‑1 without clearly separating signing algorithms from DS digest usage.
- (E2) The unchanged registry for SHA‑1 DS digests (value 1) contrasts with the stricter treatment of RSASHA1 and RSASHA1‑NSEC3‑SHA1, introducing ambiguity.

**Fix Direction:**

Clarify in the text that the deprecation applies exclusively to RSASHA1 and RSASHA1‑NSEC3‑SHA1 signing algorithms, and confirm that the SHA‑1 DS digest algorithm retains its existing validation profile.

**Severity:** Low
  *Basis:* While the ambiguity may confuse operators about deprecation scope, it does not create a normative conflict affecting implementation.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-2
- Terminology Expert: Issue-3

---

## Report 3: 9905-5-3

**Label:** Misnaming of DS registry as 'DNS Security Algorithm Numbers' in RFC 9905 Section 1

**Bug Type:** Inconsistency

**Explanation:**

RFC 9905 Section 1 erroneously refers to both the DNSKEY and DS registries as 'DNS Security Algorithm Numbers' instead of distinguishing the DS registry by its proper title 'Digest Algorithms'.

**Justification:**

- The misnaming conflates two distinct IANA registries, potentially causing confusion for implementers about which registry to consult for DS digest values.
- RFC 9904 clearly differentiates between the 'DNS Security Algorithm Numbers' (for DNSKEY) and 'Digest Algorithms' (for DS), making the error a source of unnecessary ambiguity.

**Evidence Snippets:**

- **E1:**

  RFC 9905, Section 1: ‘Operators are encouraged to consider switching to one of the recommended algorithms listed in the ‘DNS Security Algorithm Numbers’ [DNSKEY-IANA ] and ‘DNS Security Algorithm Numbers’ [DS-IANA ] registries, respectively.’

- **E2:**

  RFC 9904, Section 1.1: ‘the IANA ‘DNS Security Algorithm Numbers’ [DNSKEY-IANA ] and ‘Digest Algorithms’ [DS-IANA ] registries…’

**Evidence Summary:**

- (E1) RFC 9905 Section 1 incorrectly names both registries as 'DNS Security Algorithm Numbers'.
- (E2) RFC 9904 correctly distinguishes the DS registry as 'Digest Algorithms', highlighting the naming error.

**Fix Direction:**

Modify RFC 9905 Section 1 to refer to the DS registry by its correct name, 'Digest Algorithms' [DS-IANA].

**Severity:** Low
  *Basis:* Although the naming error may lead to temporary confusion, it does not impact the normative requirements or functionality of DNSSEC.

**Confidence:** High

**Experts mentioning this issue:**

- Terminology Expert: Issue-2

---
