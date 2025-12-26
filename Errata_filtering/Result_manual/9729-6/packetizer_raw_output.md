# Errata Reports

Total reports: 4

---

## Report 1: 9729-6-1

**Label:** TLS-version requirement in Section 7 conflicts with split deployment design

**Bug Type:** Both

**Explanation:**

Section 7 unconditionally applies TLS version requirements to any connection using the Concealed scheme, conflicting with split frontend/backend deployments where the backend may receive authentication over non‑TLS or non‑EMS connections.

**Justification:**

- Section 6.2 describes that in split deployments the frontend must send the TLS exporter output via the Concealed-Auth-Export header, even if the backend‐facing connection is not TLS.
- Section 7 mandates that if a connection does not meet TLS 1.3 or TLS 1.2+EMS, the authentication must be ignored, which contradicts the intended backend processing described in Section 6.3.

**Evidence Snippets:**

- **E1:**

  Section 6.2: In split deployments, “the backend won't be able to directly access the TLS keying material exporter from the TLS connection between the client and frontend, so the frontend needs to explicitly send it. This document defines the ‘Concealed-Auth-Export’ request header field for this purpose.” The frontend “SHALL forward the HTTP request to the backend, including the original unmodified Authorization (or Proxy-Authorization) header field and the newly added Concealed-Auth-Export header field.”

- **E2:**

  Section 6.3: “Once the backend receives the Authorization (or Proxy-Authorization) header field and the key exporter output, it looks up the key ID in its database of public keys. The backend SHALL then perform the following checks: … If all of these checks succeed, the backend can consider the request to be properly authenticated…”

- **E3:**

  Section 7: “This authentication scheme is only defined for uses of HTTP with TLS [TLS]…. Clients MUST NOT use the Concealed HTTP authentication scheme on connections that do not meet one of the two properties above. If a server receives a request that uses this authentication scheme on a connection that meets neither of the above properties, the server MUST treat the request as if the authentication were not present.”

**Evidence Summary:**

- (E1) Section 6.2 explains the need to forward the TLS exporter output in split deployments.
- (E2) Section 6.3 details the backend’s reliance on the forwarded exporter output.
- (E3) Section 7 imposes TLS requirements on any connection, creating the conflict.

**Fix Direction:**

Clarify Section 7 so that its TLS requirements apply only to the client‐facing connection (the TLS terminator/frontend) rather than to every connection carrying Concealed credentials.


**Severity:** High
  *Basis:* The conflict forces split deployments into non-conformance, potentially undermining the intended security properties.

**Confidence:** High

---

## Report 2: 9729-6-2

**Label:** Ambiguous forwarding of Proxy-Authorization header in split deployments

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that the frontend forward the original, unmodified Proxy-Authorization header along with the Authorization header, without clarifying that Proxy-Authorization may have different semantics in proxy versus gateway contexts.

**Justification:**

- Section 2 indicates that the client may use either Authorization or Proxy-Authorization for sending the signature.
- Section 6 requires the header to be forwarded unchanged, yet standard HTTP semantics distinguish between proxy and gateway roles, creating ambiguity in split deployments.

**Evidence Snippets:**

- **E1:**

  Section 2: “The client … then sends the signature using the Authorization (or Proxy-Authorization) header field (see Section 11 of [HTTP]).”

- **E2:**

  Section 6: The frontend “will parse the Authorization (or Proxy-Authorization) header field. If the authentication scheme is set to ‘Concealed’…” (6.1); later, “The frontend SHALL forward the HTTP request to the backend, including the original unmodified Authorization (or Proxy-Authorization) header field…” (6.2).

- **E3:**

  Section 3.7 of RFC 9110 (included context): distinguishes “proxy” and “gateway (a.k.a. reverse proxy)” and notes that a gateway “acts as an origin server for the outbound connection,” whereas a proxy is a message-forwarding agent chosen by the client.

**Evidence Summary:**

- (E1) Section 2 shows that both Authorization and Proxy-Authorization are used by the client.
- (E2) Section 6 mandates forwarding these headers unchanged in split deployments.
- (E3) Section 3.7 of RFC 9110 explains the differing roles of proxies and gateways, highlighting the ambiguity.

**Fix Direction:**

Clarify the intended use and forwarding rules for the Proxy-Authorization header, possibly by limiting mandatory forwarding to Authorization or by explicitly defining allowed scenarios for Proxy-Authorization in split deployments.


**Severity:** Medium
  *Basis:* Ambiguity in how headers are forwarded can lead to inconsistent implementations and unintended exposure of hop-by-hop credentials.

**Confidence:** High

---

## Report 3: 9729-6-3

**Label:** Undefined backend handling when key exporter output is missing or untrusted

**Bug Type:** Underspecification

**Explanation:**

The specification does not explicitly require the backend to treat the request as unauthenticated when a Concealed-Auth-Export header is absent, malformed, or untrusted, leaving backend behavior undefined.

**Justification:**

- Section 6.2 instructs that servers must ignore the Concealed-Auth-Export header unless the sender is trusted, but does not specify that the accompanying Authorization header must be disregarded if the exporter output is missing or invalid.
- Section 6.3 mandates backend checks that assume the presence of a valid key exporter output without clarifying the behavior when it is not available.

**Evidence Snippets:**

- **E1:**

  If the roles are split between two separate HTTP servers, then the backend won't be able to directly access the TLS keying material exporter from the TLS connection between the client and frontend, so the frontend needs to explicitly send it. This document defines the ‘Concealed-Auth-Export’ request header field for this purpose. (Section 6.2)

- **E2:**

  HTTP servers that parse the Concealed-Auth-Export header field MUST ignore it unless they have already established that they trust the sender. (Section 6.2)

- **E3:**

  Once the backend receives the Authorization (or Proxy-Authorization) header field and the key exporter output, it looks up the key ID in its database of public keys. The backend SHALL then perform the following checks: … If any of the above checks fail, the backend MUST treat it as if the Authorization (or Proxy-Authorization) header field was missing. (Section 6.3)

**Evidence Summary:**

- (E1) Section 6.2 explains the need for the frontend to send the key exporter output in split deployments.
- (E2) Section 6.2 requires that the export header be ignored if the sender isn’t trusted.
- (E3) Section 6.3 mandates backend validation that assumes a valid exporter output, without addressing the absent or untrusted case.

**Fix Direction:**

Require explicitly that if a valid and trusted key exporter output is not present, the backend MUST treat the request as if the Authorization header were missing.


**Severity:** High
  *Basis:* Undefined handling in the absence of a proper exporter output may lead to inconsistent or insecure authentication processing.

**Confidence:** High

---

## Report 4: 9729-6-4

**Label:** Mismatch between ABNF and numeric range for 's' parameter

**Bug Type:** Inconsistency

**Explanation:**

The ABNF definition for the integer parameter 's' only permits 2-5 digit numbers (or '0') while the prose allows single-digit numbers and requires a value between 0 and 65535, leading to acceptance of out-of-range values and rejection of valid ones.

**Justification:**

- Figure 4’s ABNF specifies: concealed-integer-param-value =  %x31-39 1*4( DIGIT ) / "0", which disallows single-digit integers like '1'.
- The accompanying prose states that the 's' parameter is an integer between 0 and 65535 with no leading zeroes, creating a conflict between the grammar and the intended numerical range.

**Evidence Snippets:**

- **E1:**

  concealed-integer-param-value =  %x31-39 1*4( DIGIT ) / "0" (Figure 4: Authentication Parameter Value ABNF)

- **E2:**

  Prose just above the ABNF: “The integer below is encoded without a minus and without leading zeroes. In other words, the value of this integer authentication parameter MUST NOT include any characters other than digits and MUST NOT start with a zero unless the full value is "0".” Definition of the s parameter: “Its value is an integer between 0 and 65535 inclusive from the IANA 'TLS SignatureScheme' registry…” (Section 4.4) Frontend/backend behavior: both MUST “validate that all the required authentication parameters are present and can be parsed correctly as defined in Section 4” (Sections 6.1 and 6.3)

**Evidence Summary:**

- (E1) The ABNF in Figure 4 allows only a non-zero digit followed by 1-4 digits or the value "0".
- (E2) The prose requires the 's' parameter to be between 0 and 65535 with no leading zeroes, which conflicts with the ABNF by disallowing valid single-digit values and accepting numbers up to 99999.

**Fix Direction:**

Revise the ABNF to allow single-digit non-zero integers and restrict accepted values through either the grammar or an additional numeric check enforcing the 0–65535 range.


**Severity:** High
  *Basis:* This inconsistency can lead to interoperability issues where valid authentication credentials might be rejected or out-of-range values accepted.

**Confidence:** High

---
