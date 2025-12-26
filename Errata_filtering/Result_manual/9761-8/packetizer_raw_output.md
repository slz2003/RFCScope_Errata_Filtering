# Errata Reports

Total reports: 2

---

## Report 1: 9761-8-1

**Label:** Ambiguous Ordering of Proxy Interception vs Two-Stage ACL Matching

**Bug Type:** Underspecification

**Explanation:**

The document mixes the ordering of handshake interception, decryption of handshake messages, and ACL application without clearly defining which ACL checks are enforcement points and when they occur.

**Justification:**

- The text states: the goal is for the proxy to intercept the (D)TLS handshake before applying any ACL rules. (E1)
- It then indicates that MUD (D)TLS ACL matching would need to occur after decrypting the encrypted TLS handshake messages within the proxy. (E2)
- It further describes ACL matching as performed in two stages—first on cleartext and second after decryption—leading to an ambiguous temporal ordering. (E3)

**Evidence Snippets:**

- **E1:**

  the goal is for the proxy to intercept the (D)TLS handshake before applying any ACL rules. (Section 8)

- **E2:**

  This implies that MUD (D)TLS ACL matching would need to occur after decrypting the encrypted TLS handshake messages within the proxy. (Section 8)

- **E3:**

  ACL matching would be performed in two stages: first, by filtering clear-text TLS handshake message and second, by filtering after decrypting the TLS handshake messages. (Section 8)

**Evidence Summary:**

- (E1) indicates that interception should occur before any ACL rules are applied, (E2) requires decryption before ACL matching, and (E3) describes a two-stage process—together these create ambiguity regarding the order of enforcement.

**Fix Direction:**

Clarify in Section 8 whether ACL matching is to be enforced in a pre-decryption stage versus a post-decryption stage and specify the exact point at which ACL decisions must be rendered.


**Severity:** Medium
  *Basis:* Different interpretations of the ordering may lead to inconsistent behavior between implementations, affecting security enforcement timing.

**Confidence:** High

---

## Report 2: 9761-8-2

**Label:** Underspecified Timing for Completion of Second-Stage ACL Checks Relative to Finished and Application Data

**Bug Type:** Underspecification

**Explanation:**

The document fails to specify when the second-stage ACL checks must be completed relative to the receipt of Finished messages and the forwarding of application data (including 0‑RTT), leading to potential implementation differences.

**Justification:**

- While the text describes a two-stage ACL process, it does not anchor the completion of second-stage checks to a defined point in the TLS handshake. (E4)
- This omission leaves open the possibility for some implementations to forward application data (or 0‑RTT data) before fully completing ACL validations. (E5)

**Evidence Snippets:**

- **E4:**

  ACL matching would be performed in two stages: first, by filtering clear-text TLS handshake message and second, by filtering after decrypting the TLS handshake messages. (Section 8)

- **E5:**

  The document describes a two-stage ACL process in a (D)TLS 1.3 proxy—on cleartext handshake data, then on decrypted handshake messages—but never states at what point in the handshake all necessary checks must be completed before forwarding application data, including 0‑RTT data.

**Evidence Summary:**

- (E4) outlines the two-stage ACL matching process, while (E5) highlights the absence of a defined cut‑off in the handshake for completing these checks before application data is sent.

**Fix Direction:**

Add a clarifying sentence in Section 8 that explicitly requires all ACL checks—including those based on decrypted handshake messages—to be completed before any application data (including 0‑RTT) is forwarded.


**Severity:** Medium
  *Basis:* Without a defined point for ACL completion, implementations may vary in when they enforce policy, potentially allowing data flow before full validation and creating inconsistent security postures.

**Confidence:** High

---
