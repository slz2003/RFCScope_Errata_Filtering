# Errata Reports

Total reports: 3


---

## Report 2: 9811-6-2

**Label:** Inconsistent CoAP Content-Formats Reference to HTTP-only RFC 9811 (Section 6 CoAP bullet)

**Bug Type:** Inconsistency

**Explanation:**

The CoAP Content-Formats registry entry for application/pkixcmp is updated to reference RFC 9811, an HTTP-specific binding, even though the normative CoAP transfer specifications are defined in RFC 9482.

**Justification:**

- RFC 9811 instructs that the CoAP registry entry should refer to this document instead of RFC 4210; however, CoAP transfer for CMP is defined in RFC 9482 (E2).
- The Lightweight CMP Profile mandates that if CoAP transfer is used, RFC 9482 must be followed, making the reference to an HTTP-only specification misleading (E2).

**Evidence Snippets:**

- **E2:**

  RFC 9811 tells IANA that the reference for application/pkixcmp in the CoAP Content‑Formats registry should “refer to this document, instead of [RFC4210]”. However, CoAP transfer for CMP is normatively defined in CMP over CoAP, RFC 9482, not in RFC 9811. The Lightweight CMP Profile states that if CoAP transfer is used, “the specifications described in CMP over CoAP [RFC9482] MUST be followed” and that CoAP URIs and semantics are taken from RFC 9482 . It also explicitly ties use of the application/pkixcmp media type in CoAP (and HTTP) to the transfer definitions in RFC 6712 and RFC 9482 . RFC 9811 is strictly an HTTP binding and does not define CoAP behavior. If IANA updates the CoAP Content‑Formats entry to reference only RFC 9811, implementers who follow the registry will be directed to an HTTP‑only spec for a CoAP content‑format, while the actual CoAP mapping is in RFC 9482. This cross‑document mismatch can mislead CoAP implementers and obscures the correct normative source for CoAP transfer semantics.

**Evidence Summary:**

- (E2) RFC 9811 directs the CoAP registry to reference itself despite RFC 9482 defining CoAP transfer, creating a cross-document mismatch.


**Severity:** High
  *Basis:* Directing CoAP implementers to an HTTP-specific document can mislead implementation by obscuring the normative source for CoAP behavior.

**Confidence:** High
---

## Report 3: 9811-6-3

**Label:** Incorrect Prior Reference for Well-Known cmp URI (Section 6 Well-Known URIs bullet)

**Bug Type:** Inconsistency

**Explanation:**

RFC 9811 directs IANA to update the well-known cmp URI entry by replacing a reference to RFC 4210, yet RFC 4210 never defined this URI; the entry was actually established by RFC 9480 and RFC 9482.

**Justification:**

- The analysis shows that RFC 4210 is incorrectly cited as the prior reference when the well-known cmp URI was originally registered under RFC 9480 and RFC 9482 (E3).
- This discrepancy may confuse implementers tracking the history and accuracy of the registry entry (E3).

**Evidence Snippets:**

- **E3:**

  RFC 9811 instructs IANA that “the reference for ‘cmp’ in the ‘Well-Known URIs’ registry … refers to this document instead of [RFC4210]”. But RFC 4210 never defined or registered a well‑known “cmp” URI; that registry entry was created and defined later by CMP Updates, RFC 9480, which says: “IANA has registered the following new entry in the ‘Well-Known URIs’ registry … URI Suffix: cmp … Reference: [RFC9480] [RFC9482]” . Thus, the current defining references for the well‑known cmp URI are RFC 9480 and RFC 9482, not RFC 4210. From IANA’s and implementers’ point of view, RFC 9811’s text implies it is replacing a reference to RFC 4210 that does not actually exist; the true prior reference is [RFC9480] [RFC9482]. While IANA can still interpret the instruction as “set the reference to RFC 9811”, the mismatch between 9811 and 9480 over what document previously owned the registration is a cross‑RFC inconsistency and can cause confusion when tracking the change history of that registry entry.

**Evidence Summary:**

- (E3) RFC 9811 replaces a non-existent RFC 4210 reference for the well-known cmp URI, where the actual definition lies in RFC 9480 and RFC 9482, leading to potential confusion.


**Severity:** Medium
  *Basis:* While the misidentification does not directly break functionality, it can lead to confusion regarding the correct historical references of the registry entry.

**Confidence:** Medium
---

## Report 4: 9811-6-4

**Label:** Media Type Registry Points Solely to HTTP Binding, Obscuring Core CMP Semantics (Section 6 Media Types bullet)

**Bug Type:** Underspecification

**Explanation:**

The media type registry entry for application/pkixcmp is updated to reference RFC 9811, which is focused on an HTTP binding, potentially obscuring the fact that the core CMP semantics are defined in RFC 9810 for other transports.

**Justification:**

- RFC 9811 updates the media type reference to itself instead of RFC 2510, while the actual payload semantics of PKIMessage are defined in RFC 9810 (E4).
- This creates a layering concern where non-HTTP implementations (such as MIME email or MQTT) might be misdirected to an HTTP‑centric document (E4).

**Evidence Snippets:**

- **E4:**

  RFC 9811 tells IANA that the Media Types registry entry for application/pkixcmp should now reference “this document, instead of [RFC2510]”. RFC 2510 originally defined the MIME usage of application/pkixcmp for email and HTTP , while the current core CMP protocol semantics (the structure and meaning of PKIMessage/PKIMessages) are defined in RFC 9810 . RFC 9810 explicitly treats transfer protocols such as email, HTTP, MQTT, and CoAP as “beyond the scope of this document and must be specified separately”, citing RFC 9811 for HTTP and RFC 9482 for CoAP . Pointing the generic media type entry solely at RFC 9811 therefore makes the registry’s primary reference be an HTTP‑specific binding rather than the document that defines the media type’s payload semantics across all transports. This does not create a direct contradiction (since RFC 9811 in turn references RFC 9810), but it leaves the relationship between the media type, the core CMP specification, and the various transfer bindings implicit. Implementers consulting the Media Types registry for non‑HTTP use of application/pkixcmp (e.g., MIME email or MQTT as mentioned in RFC 9810 ) will be directed first to the HTTP‑centric document rather than the protocol core, which is a layering/clarity issue rather than a strict interoperability break.

**Evidence Summary:**

- (E4) The media type registry now points solely to RFC 9811, an HTTP binding, potentially obscuring the core CMP semantics found in RFC 9810 for non-HTTP transports.


**Severity:** Low
  *Basis:* The issue is principally one of clarity and layering rather than an interoperability error, affecting non-HTTP implementations less directly.

**Confidence:** Low
---
