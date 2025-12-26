# Errata Reports

Total reports: 2

---

## Report 1: 9761-2-1

**Label:** Conflicting DTLS References in (D)TLS Terminology: RFC6347 vs RFC9147

**Bug Type:** Inconsistency

**Explanation:**

The document’s definition of (D)TLS in Section 2 exclusively ties DTLS to RFC6347, while later parts (e.g., the Introduction and YANG modules) reference DTLS 1.3 via RFC9147, which can mislead implementers.

**Justification:**

- Section 2 defines (D)TLS with reference only to RFC6347, despite other sections clearly including DTLS 1.3 as per RFC9147 (E1, E3).
- The discrepancy between the normative references may cause readers to apply DTLS rules incorrectly.

**Evidence Snippets:**

- **E1:**

  Section 2 defines “(D)TLS” as “used for statements that apply to both Transport Layer Security [RFC8446] and Datagram Transport Layer Security [RFC6347]”, explicitly tying “TLS” to RFC 8446 (TLS 1.3) and “DTLS” to RFC 6347 (DTLS 1.2). Elsewhere in RFC 9761, DTLS 1.3 is clearly in scope: the Introduction refers generically to “DTLS [RFC9147]” as one of the dominant protocols, and the YANG modules and IANA registries define explicit support for both `dtls12` (RFC 6347) and `dtls13` (RFC 9147).

- **E2:**

  Section 2 definition:  (D)TLS:  Used for statements that apply to both Transport Layer Security [RFC8446] and Datagram Transport Layer Security [RFC6347].  Specific terms "TLS" and "DTLS" are used for any statement that applies to either protocol alone.

- **E3:**

  Introduction:  TLS [RFC8446] and DTLS [RFC9147] are the dominant protocols (counting all (D)TLS versions) that provide encryption for IoT device traffic.

**Evidence Summary:**

- (E1) Highlights the discrepancy between the Section 2 definition and later references to DTLS 1.3.
- (E2) Provides the verbatim Section 2 definition that cites only RFC6347 for DTLS.
- (E3) Shows that the Introduction refers to DTLS via RFC9147.

**Fix Direction:**

Revise the (D)TLS definition in Section 2 to include both RFC6347 and RFC9147 (or otherwise clarify that both DTLS 1.2 and 1.3 are in scope).


**Severity:** Medium
  *Basis:* Expert analyses indicate that using two different normative references for DTLS creates ambiguity that may mislead implementers.

**Confidence:** Medium

---

## Report 2: 9761-2-2

**Label:** Inconsistent Middlebox Definition: TLS-only reference vs. (D)TLS Usage

**Bug Type:** Inconsistency

**Explanation:**

The document’s definition of 'Middlebox' in Section 2 restricts its scope to TLS traffic, while subsequent sections discuss middleboxes in the context of (D)TLS (including DTLS), leading to a potential terminological mismatch.

**Justification:**

- Section 2 provides a TLS-only definition of Middlebox, yet later sections refer to (D)TLS profiles and proxies, implying applicability to both TLS and DTLS traffic.
- This definitional gap could cause implementers to question whether DTLS-interacting devices fall under the term 'middlebox'.

**Evidence Snippets:**

- **E4:**

  Section 2 definition:  Middlebox:  A middlebox that interacts with TLS traffic can either act as a TLS proxy, intercepting and decrypting the traffic for inspection, or inspect the traffic between TLS peers without terminating the TLS session.

- **E5:**

  Section 3: If the IoT device supports a MUD (D)TLS profile, the (D)TLS profile parameters of the IoT device can be used by a middlebox to detect and block malware communication... Section 4: In (D)TLS 1.3, full (D)TLS handshake inspection is not possible ... the middlebox can block malicious flows without acting as a (D)TLS 1.3 proxy. Section 4.1: To obtain more visibility into negotiated TLS 1.3 parameters, a middlebox can act as a (D)TLS 1.3 proxy.

**Evidence Summary:**

- (E4) Offers the TLS-only definition of Middlebox in Section 2.
- (E5) Demonstrates that later sections refer to middlebox functionality in the context of (D)TLS, including DTLS.

**Fix Direction:**

Modify the Section 2 Middlebox definition to reference (D)TLS traffic instead of TLS only, for example by stating 'interacts with (D)TLS traffic.'



**Severity:** Low
  *Basis:* The issue is minor and terminological, unlikely to lead to interoperability problems but may cause temporary confusion.

**Confidence:** Inferred
