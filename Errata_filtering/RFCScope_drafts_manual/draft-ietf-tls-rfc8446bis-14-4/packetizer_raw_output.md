# Errata Reports

Total reports: 2

---


## Report 3: draft-ietf-tls-rfc8446bis-14-4-3

**Label:** ClientHello.extensions Minimum Length Inconsistency

**Bug Type:** Inconsistency

**Explanation:**

The structural definition mandates a minimum length of 8 bytes for ClientHello.extensions, yet a ClientHello containing only a supported_versions extension encodes to 7 bytes, resulting in a conflict between the formal specification and its descriptive text.

**Justification:**

- The struct definition in Section 4.1.2/B.3.1 specifies: Extension extensions<8..2^16-1>; in ClientHello.
- The accompanying description states: “TLS 1.3 ClientHello messages always contain extensions (minimally "supported_versions", otherwise, they will be interpreted as TLS 1.2 ClientHello messages).”
- The supported_versions structure is defined as: struct { ProtocolVersion versions<2..254>; } SupportedVersions; which, with its 1‑byte length and 2 bytes for the version, totals 3 bytes of extension data, plus 4 bytes for the extension header, yielding 7 bytes.

**Evidence Snippets:**

- **E1:**

  Extension extensions<8..2^16-1>; (in ClientHello)

- **E2:**

  TLS 1.3 ClientHello messages always contain extensions (minimally "supported_versions", otherwise, they will be interpreted as TLS 1.2 ClientHello messages).

- **E3:**

  struct { ProtocolVersion versions<2..254>; } SupportedVersions; i.e., versions is a vector with a 1‑byte length and at least 2 bytes of data.

**Evidence Summary:**

- (E1) Specifies the minimum length of the extensions vector as 8 bytes.
- (E2) Indicates that a minimal ClientHello may contain only the supported_versions extension.
- (E3) Shows that a minimal supported_versions extension would be encoded in 7 bytes.

**Fix Direction:**

Either adjust the minimum length of ClientHello.extensions to 7 bytes for the sole supported_versions case or modify the supported_versions extension to meet the 8-byte minimum.

**Severity:** Medium
  *Basis:* This structural mismatch can cause interoperable implementations to either reject valid messages or generate messages that do not conform to the specified bounds.

**Confidence:** High

**Experts mentioning this issue:**

- Quantitative: Issue-1

---


## Report 4: draft-ietf-tls-rfc8446bis-14-4-4

**Label:** Conflicting Requirements in SupportedVersions Handling

**Bug Type:** Inconsistency

**Explanation:**

There are contradictory requirements for handling a ClientHello with a legacy_version of 0x0304 or later when the supported_versions extension is absent, with one clause mandating negotiation of TLS 1.2 and another clause permitting the handshake to be aborted.

**Justification:**

- Section 4.1.2 states: “A server which receives a legacy_version value not equal to 0x0303 MUST abort the handshake with an ‘illegal_parameter’ alert.”
- Section 4.2.1 instructs: “If this extension is not present, servers which are compliant with this specification and which also support TLS 1.2 MUST negotiate TLS 1.2 or prior as specified in [RFC5246], even if ClientHello.legacy_version is 0x0304 or later.”
- Immediately following, Section 4.2.1 adds: “Servers MAY abort the handshake upon receiving a ClientHello with legacy_version 0x0304 or later.”

**Evidence Snippets:**

- **E1:**

  Section 4.1.2: “A server which receives a legacy_version value not equal to 0x0303 MUST abort the handshake with an ‘illegal_parameter’ alert.”

- **E2:**

  Section 4.2.1: “If this extension is not present, servers which are compliant with this specification and which also support TLS 1.2 MUST negotiate TLS 1.2 or prior as specified in [RFC5246], even if ClientHello.legacy_version is 0x0304 or later.”

- **E3:**

  Section 4.2.1: “Servers MAY abort the handshake upon receiving a ClientHello with legacy_version 0x0304 or later.”

**Evidence Summary:**

- (E1) Imposes an unconditional abort for legacy_version values not equal to 0x0303.
- (E2) Requires that servers supporting TLS 1.2 negotiate TLS 1.2 when the supported_versions extension is absent.
- (E3) Provides a general permission to abort the handshake in the same scenario.

**Fix Direction:**

Clarify the text so that the MUST negotiate TLS 1.2 requirement applies exclusively to servers that support TLS 1.2, while the MAY abort option is limited to servers that do not support TLS 1.2, or otherwise harmonize the requirements to remove the contradiction.

**Severity:** Medium
  *Basis:* The conflicting requirements for version negotiation versus outright abortion can result in divergent behaviors among implementations when encountering non‐conformant ClientHello messages.

**Confidence:** High

**Experts mentioning this issue:**

- Causal: Non‑conformant Client Case
- CrossRFC: Issue-3

---