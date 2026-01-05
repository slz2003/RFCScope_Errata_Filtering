# Errata Reports

Total reports: 3

---


## Report 1: 9824-5-1

**Label:** Ambiguous Resolver Handling of CO Flag in Multi-hop Scenarios

**Bug Type:** Underspecification

**Explanation:**

The specification does not clearly define how resolvers should handle the CO flag in both their upstream queries and downstream responses – including caching behavior – which can lead to inconsistent NXDOMAIN restoration in multi‐hop deployments.

**Justification:**

- Section 5.1 instructs that resolvers must reset the response code back to NOERROR when the downstream querier does not set CO but does not clarify whether resolvers should always propagate or mirror the CO flag in their upstream queries.
- The text is vague about how cached NXNAME responses should be keyed or treated in multi-hop scenarios, leaving room for interpretation among different implementations.

**Evidence Snippets:**

- **E1:**

  When this flag is sent in a query by a resolver, it indicates that the resolver will accept a NODATA response with a signed NXNAME for a nonexistent name, together with the response code field set to NXDOMAIN (3).

- **E2:**

  In responses to such queries, an authoritative server implementing both Compact Denial of Existence and this signaling scheme will set the Compact Answers OK EDNS header flag and, for nonexistent names, will additionally set the response code field to NXDOMAIN.

- **E3:**

  EDNS is a hop-by-hop signal, so resolvers will need to record the presence of this flag in associated cache data and respond to downstream DNSSEC-enabled queriers appropriately. If the querier does not set the Compact Answers OK flag, the resolver will need to reset the response code back to NOERROR (0) for an NXNAME response.

**Evidence Summary:**

- (E1) defines the acceptance of a response with NXNAME under CO in queries.
- (E2) describes the authoritative server’s obligation to set RCODE=NXDOMAIN with CO.
- (E3) outlines the resolver’s hop-by-hop signal handling and caching requirement.

**Fix Direction:**

Clarify in Section 5.1 the normative requirements for resolver behavior – including when to set or mirror the CO flag on upstream queries and how to manage cached NXNAME responses – to ensure consistent NXDOMAIN restoration across hops.


**Severity:** Medium
  *Basis:* This ambiguity affects the visible negative response (NXDOMAIN) optimization across resolver chains without impacting DNSSEC validation, potentially leading to divergent behaviors.

**Confidence:** High

---


## Report 2: 9824-5-2

**Label:** Ambiguous Definition of Non‑DNSSEC‑Enabled Queries Excluding Non‑EDNS Queries

**Bug Type:** Underspecification

**Explanation:**

The document defines non‑DNSSEC‑enabled queries solely by the absence of the DO bit in the EDNS0 OPT header, leaving queries that do not include an OPT RR ambiguous.

**Justification:**

- Section 5 defines non‑DNSSEC‑enabled queries as those that do not set the DO bit in the EDNS0 OPT header, implicitly assuming the presence of an OPT RR.
- RFC 3225, however, confines the DO bit to the OPT RR, meaning queries without EDNS lack any DO bit, and their classification is not made explicit.

**Evidence Snippets:**

- **E1:**

  This is generally possible for non-DNSSEC-enabled queries, namely those that do not set the DO bit (‘DNSSEC answer OK’) in the EDNS0 OPT header. For such queries, authoritative servers implementing Compact Denial of Existence could return a normal NXDOMAIN response.

- **E2:**

  RFC 3225 defines the DO bit specifically *within* the EDNS0 OPT RR; queries without EDNS do not have a DO bit at all.

**Evidence Summary:**

- (E1) indicates non‑DNSSEC‑enabled queries are defined by not setting the DO bit.
- (E2) notes that queries without an OPT RR do not have a DO bit, thus remaining undefined.

**Fix Direction:**

Clarify in Section 5 that non‑DNSSEC‑enabled queries include both queries with an OPT RR and DO=0 and queries lacking an OPT RR entirely.


**Severity:** Medium
  *Basis:* This underspecification may lead to inconsistent handling of non‑EDNS queries across implementations.

**Confidence:** High

---


## Report 3: 9824-5-3

**Label:** Terminology Inconsistency: 'NODATA response' with RCODE=NXDOMAIN

**Bug Type:** Inconsistency

**Explanation:**

The specification inconsistently applies the term 'NODATA response' by describing a response with an empty Answer section and a signed NXNAME as a NODATA response, yet it mandates that the response use RCODE=NXDOMAIN, which contradicts established definitions.

**Justification:**

- Section 3.1 defines a NODATA response as having RCODE=NOERROR with an empty Answer section.
- Section 5.1, however, instructs that a resolver will accept a NODATA response with the response code field set to NXDOMAIN, conflicting with both Section 3.1 and RFC 7129’s established terminology.

**Evidence Snippets:**

- **E1:**

  a NODATA response (response code NOERROR, empty Answer section) is generated with a dynamically constructed NSEC record…

- **E2:**

  When this flag is sent in a query by a resolver, it indicates that the resolver will accept a NODATA response with a signed NXNAME for a nonexistent name, together with the response code field set to NXDOMAIN (3).

**Evidence Summary:**

- (E1) defines a NODATA response with RCODE=NOERROR.
- (E2) contradicts that definition by pairing 'NODATA response' with RCODE=NXDOMAIN.

**Fix Direction:**

Revise Section 5.1 to either refrain from using the term 'NODATA response' when RCODE=NXDOMAIN is employed, or introduce a distinct term that accurately describes the Compact Answers response with NXDOMAIN.


**Severity:** Medium
  *Basis:* The conflicting terminology may mislead implementers about the expected header values and the classification of negative responses.

**Confidence:** High

---