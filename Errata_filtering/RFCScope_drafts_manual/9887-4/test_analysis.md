================================================================================
COMPLETE ANALYSIS RESULT
================================================================================

RFC: Unknown
Section: Unknown
Model: gpt-5.1

================================================================================
ROUTER ANALYSIS
================================================================================
================================================================================
ROUTING SUMMARY
================================================================================

Excerpt Summary: Section 4 of RFC 9887 defines how TACACS+ implementations must treat the legacy TACACS+ “obfuscation” mechanism when operating over TLS, in particular mandating use of the TAC_PLUS_UNENCRYPTED_FLAG and specific error-handling behavior. The referenced text from RFC 8907 defines the original meaning and normative requirements for that flag, obfuscation, and error handling.
Overall Bug Likelihood: Medium

Dimensions:
  - Temporal: LOW - Only simple “initial packet / subsequent packets” ordering; no complex sequencing or timers involved.
  - ActorDirectionality: LOW - Roles (client/server) are clear and consistent; no obvious confusion about who sends or interprets which flag.
  - Scope: MEDIUM - Correctness depends on differentiating *TLS* vs *non‑TLS* connections and when RFC 9887 overrides RFC 8907 behavior.
  - Causal: MEDIUM - How a received flag value is handled (drop vs ERROR vs connection termination) affects interoperability and downgrade/misconfig behavior.
  - Quantitative: LOW - Only single-bit flags, no nontrivial ranges or size calculations.
  - Deontic: HIGH - There are potentially conflicting MUST/MUST NOT requirements between RFC 9887 and RFC 8907 for the same flag, depending on context.
  - Structural: LOW - No ABNF/YANG/ASN.1 to compare; header/flag usage is consistent at a structural level.
  - CrossRFC: HIGH - Section 4 of RFC 9887 explicitly references and alters behavior defined in RFC 8907 Sections 4.1, 4.5, and 10.5.2; correctness depends on reading them together.
  - Terminology: MEDIUM - The same flag name (TAC_PLUS_UNENCRYPTED_FLAG) is reused with effectively inverted operational requirements in the TLS case vs the base spec.
  - Boundary: LOW - Edge behavior when receiving an unexpected flag is described; no unaddressed “empty/limit” corner cases apparent.

Candidate Issues: 2

  Issue 1:
    Type: Inconsistency
    Label: Normative conflict and unclear override of TAC_PLUS_UNENCRYPTED_FLAG semantics between RFC 8907 and RFC 9887 for TLS connections
    Relevant Dimensions: Deontic, CrossRFC, Scope, Causal, Terminology
    Sketch: In RFC 8907, TAC_PLUS_UNENCRYPTED_FLAG has strong prohibitions:  
      • Section 4.1/4.5/10.5.2 col...

  Issue 2:
    Type: Underspecification
    Label: Ambiguous scope of “obfuscation is hereby obsoleted” relative to non‑TLS TACACS+ in RFC 8907
    Relevant Dimensions: Scope, CrossRFC, Deontic
    Sketch: RFC 9887 Section 4 says “obfuscation is hereby obsoleted” and that the section “describes how the TA...

Response ID: resp_07acc0554deb90c3006958c23a0dfc81958a8aa7f1364d2b2c

================================================================================
EXPERT ANALYSES
================================================================================
================================================================================
REASONING EXPERT ANALYSES
================================================================================

## Scope Expert
--------------------------------------------------------------------------------

### Expert Analysis:
--------------------------------------------------------------------------------
ScopeAnalysis:
- ExcerptSummary: Section 4 of RFC 9887 redefines how the TACACS+ “obfuscation” mechanism and the TAC_PLUS_UNENCRYPTED_FLAG are to be used when TACACS+ runs over TLS, while RFC 8907 originally defined those mechanisms only for non‑TLS (plain TCP) TACACS+ connections. There are interactions and apparent conflicts between the global, transport‑agnostic requirements in RFC 8907 and the TLS‑specific behavior in RFC 9887 that hinge on the scope of those rules (TLS vs non‑TLS, production vs test).

- ScopeModel:
  - Targets:
    - TACACS+ connections:
      - Non‑TLS connections as defined in RFC 9887 Section 2 (“connection defined in [RFC8907] … using the unsecure TACACS+ authentication and obfuscation (or the unobfuscated option for testing)”).
      - TLS connections (“TLS TACACS+ connections”) as defined in RFC 9887 Sections 2 and 3.2.
    - TACACS+ sessions (authentication, authorization, accounting) as defined in RFC 8907 Section 3.5.
    - TACACS+ packet header field `flags`, specifically:
      - TAC_PLUS_UNENCRYPTED_FLAG := 0x01 (RFC 8907 Section 4.1 / 4.5 / 10.5.2).
    - TACACS+ clients and servers:
      - Non‑TLS clients/servers (RFC 8907 behaviours).
      - TLS TACACS+ clients/servers (as defined and constrained in RFC 9887 Sections 2, 3, and 4).
  - Conditions:
    - Transport:
      - Whether the connection is a non‑TLS TACACS+ TCP connection vs a TLS 1.3 connection carrying TACACS+ as application data (RFC 9887 Sections 2, 3.1, 3.2).
    - Obfuscation vs non‑obfuscation (RFC 8907 Section 4.5, RFC 9887 Section 4):
      - TAC_PLUS_UNENCRYPTED_FLAG = 0 in RFC 8907 means “obfuscated” and is required for normal operation (“flag field MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set to 0 so that the packet body is obfuscated …”, RFC 8907 Section 4.5).
      - TAC_PLUS_UNENCRYPTED_FLAG = 1 in RFC 8907 signals “entire packet body is in cleartext” and is “deprecated and MUST NOT be used in production. A request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true.” (RFC 8907 Section 4.5) plus additional prohibitions in Section 10.5.2.
      - In RFC 9887 over TLS: peers “MUST NOT use obfuscation with TLS” and the client “MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit” for a TLS connection, with “All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.” (RFC 9887 Section 4).
    - Error conditions:
      - RFC 8907:
        - Key mismatch or invalid packet length after de‑obfuscation ⇒ server MUST return *_STATUS_ERROR and then follow Section 4.4 connection-handling rules (RFC 8907 Sections 4.5 and 4.4).
        - Use of TAC_PLUS_UNENCRYPTED_FLAG or mismatch between configuration and flag in responses ⇒ clients MUST close TCP session and treat as FAIL for AUTHEN/AUTHOR per Section 10.5.2.
      - RFC 9887 over TLS:
        - TLS TACACS+ server receiving a packet over TLS with TAC_PLUS_UNENCRYPTED_FLAG not set to 1 MUST return *_STATUS_ERROR “with the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1, and terminate the session.” (Section 4).
        - TACACS+ client receiving a packet “with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the session, and SHOULD log this error.” (Section 4).
    - Deployment / production vs test:
      - RFC 8907 makes TAC_PLUS_UNENCRYPTED_FLAG a deprecated testing-only option: “This option is deprecated and MUST NOT be used in production.” (Section 4.5) and “Clients MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG … This option MUST NOT be used when the client is in production.” (Section 10.5.2).
      - RFC 9887, by contrast, requires this flag for all TLS TACACS+ connections, including production deployments (Sections 3–4).
  - NotedAmbiguities:
    - RFC 8907’s TAC_PLUS_UNENCRYPTED_FLAG requirements (MUST NOT set in production, MUST reject connections that have it set) are written without any transport qualifier; they implicitly apply to all TACACS+ connections.
    - RFC 9887 requires TAC_PLUS_UNENCRYPTED_FLAG=1 on *all* packets over TLS, in direct tension with those earlier “MUST NOT set” requirements, but does not explicitly say that the earlier rules are scoped to non‑TLS or are superseded for TLS.
    - The phrase “obfuscation is hereby obsoleted” in RFC 9887 Section 4 sounds global, but the operational text that follows is all TLS‑specific, while other parts of RFC 9887 still clearly describe non‑TLS connections that “use … obfuscation (or the unobfuscated option for testing)” and coexistence of TLS and non‑TLS.
    - The final client‑side error-handling rule in Section 4 of RFC 9887 (“A TACACS+ client that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1…”) omits an explicit “over TLS” qualifier, unlike the immediately preceding server‑side rule.

- CandidateIssues:
  - Issue-1:
    - BugType: Both
    - ShortLabel: Conflicting MUST/MUST‑NOT semantics for TAC_PLUS_UNENCRYPTED_FLAG between RFC 8907 and RFC 9887 due to missing TLS vs non‑TLS scoping
    - ScopeProblemType: Transport-context scoping (TLS vs non‑TLS) of a globally-defined header flag and its production-use constraints
    - Evidence:
      - RFC 8907, Section 4.5:
        - “The flag field MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set to 0 so that the packet body is obfuscated…” and “TAC_PLUS_UNENCRYPTED_FLAG == 0x1 … This option is deprecated and MUST NOT be used in production. … A request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true.”
      - RFC 8907, Section 10.5.2:
        - “TACACS+ servers MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set.”  
          “TACACS+ clients MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG. Clients MUST be implemented in a way that requires explicit configuration to enable the use of TAC_PLUS_UNENCRYPTED_FLAG. This option MUST NOT be used when the client is in production.”
      - RFC 9887, Section 1 and 3.2:
        - “This document updates the TACACS+ protocol to use TLS 1.3 … and obsoletes the use of TACACS+ obfuscation mechanisms.”  
          “The TACACS+ obfuscation mechanism defined in [RFC8907] MUST NOT be applied when operating over TLS (Section 4).”
      - RFC 9887, Section 4:
        - “Peers MUST NOT use obfuscation with TLS.”  
          “A TACACS+ client initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit… All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.”  
          “A TLS TACACS+ server that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 over a TLS connection MUST return an error … and terminate the session.”
    - DetailedReasoning:
      - In RFC 8907, the TAC_PLUS_UNENCRYPTED_FLAG has a transport-agnostic definition: it indicates a non‑obfuscated packet body and is explicitly characterized as a deprecated, non‑production testing facility. Servers “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set,” and clients “MUST NOT set” it in production deployments (Sections 4.5, 10.5.2). No scope qualifier limits these requirements to any particular transport; at the time, only raw TCP was assumed.
      - RFC 9887 introduces a TLS transport, but Section 3.2 explicitly says that, once TLS is established, “the exchange of TACACS+ data MUST proceed in accordance with the procedures defined in [RFC8907]” except that “the TACACS+ obfuscation mechanism … MUST NOT be applied when operating over TLS.” Section 4 then goes further and requires the opposite flag setting for TLS: the client “MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit” when initiating a TLS connection, and “All subsequent packets MUST have” it set to 1; servers and clients must treat any packet over TLS with the bit not set to 1 as an error.
      - Thus, on TLS connections, RFC 9887 requires exactly what RFC 8907 forbids in general: setting TAC_PLUS_UNENCRYPTED_FLAG to 1 in production, for all packets. At the same time, RFC 8907 says servers “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set” and clients “MUST NOT set” it, also without any mention of TLS.
      - The only way to reconcile these is to infer a change in scope that is not written explicitly: that RFC 8907’s requirements for TAC_PLUS_UNENCRYPTED_FLAG (Sections 4.5 and 10.5.2) are to be interpreted as applying only to *non‑TLS* TACACS+ connections, and that RFC 9887 overrides them for TLS connections by reusing the same flag to mean “no obfuscation because TLS is providing confidentiality/integrity”.
      - However, RFC 9887 never states that the previous “MUST NOT set” and “MUST reject” requirements for TAC_PLUS_UNENCRYPTED_FLAG are limited to non‑TLS or superseded over TLS. Instead, it simply adds TLS‑specific requirements that appear to contradict the unqualified RFC 8907 rules. This is a classic scope mismatch: an originally global rule is implicitly narrowed in the update, but the narrowing is left implicit.
      - An implementer who naively follows both documents might conclude that a production TACACS+ client is *never* allowed to set TAC_PLUS_UNENCRYPTED_FLAG, even on TLS connections, or that a server must still “reject connections” when it sees the flag set, even though 9887 requires it on all TLS packets. This could lead to non‑interoperable behavior or implementations that refuse to conform to 9887’s TLS mode because they view it as violating 8907’s Security Considerations.
      - To remove ambiguity, an erratum or clarification should explicitly state that:
        - RFC 8907’s requirements about TAC_PLUS_UNENCRYPTED_FLAG (especially in Sections 4.5 and 10.5.2) are intended for non‑TLS TACACS+ connections only, and
        - For TLS TACACS+ connections, RFC 9887 Section 4 fully defines the semantics and required usage of TAC_PLUS_UNENCRYPTED_FLAG and supersedes the earlier “MUST NOT set” / “MUST reject” language.

  - Issue-2:
    - BugType: Underspecification
    - ShortLabel: Ambiguous scope of “obfuscation is hereby obsoleted” vs explicit continued use of obfuscation on non‑TLS connections
    - ScopeProblemType: Ambiguous global vs TLS‑only deprecation of a mechanism (obfuscation) and its configuration
    - Evidence:
      - RFC 9887, Introduction:
        - “…updates the TACACS+ protocol to use TLS 1.3 authentication and encryption [RFC8446], and obsoletes the use of TACACS+ obfuscation mechanisms.”
      - RFC 9887, Section 4:
        - “The obfuscation mechanism documented in Section 4.5 of [RFC8907] is weak. The introduction of TLS authentication and encryption to TACACS+ replaces this former mechanism, so obfuscation is hereby obsoleted. This section describes how the TACACS+ client and servers MUST operate regarding the obfuscation mechanism.”
      - RFC 9887, Section 2 (Non-TLS connection definition):
        - “It is a connection without TLS, using the unsecure TACACS+ authentication and obfuscation (or the unobfuscated option for testing).”
      - RFC 9887, Sections 6.1 and 6.2:
        - Explicitly discuss migration where “TLS and non-TLS versions of TACACS+ exist in an organization” and the continued need to support non‑TLS TACACS+ servers and clients.
      - RFC 9887, Section 3.5:
        - “Although this document removes the option of obfuscation (Section 4), it is still possible that the TLS and non-TLS versions of TACACS+ exist in an organization… In such cases, the shared secrets configured for TACACS+ obfuscation clients MUST NOT be the same as the PSKs configured for TLS clients.”
    - DetailedReasoning:
      - The wording “obsoletes the use of TACACS+ obfuscation mechanisms” in the Introduction and “obfuscation is hereby obsoleted” in Section 4 is phrased in a global way: it appears to declare the obfuscation mechanism defined in RFC 8907 Section 4.5 to be obsolete for TACACS+ generally, not just when TLS is in use.
      - However, elsewhere in RFC 9887 the text explicitly describes non‑TLS connections that *continue* to “use the unsecure TACACS+ authentication and obfuscation (or the unobfuscated option for testing)” and devotes whole subsections to migration and coexistence of TLS and non‑TLS TACACS+ servers and clients (Sections 6.1, 6.2). Section 3.5 even explicitly mentions “shared secrets configured for TACACS+ obfuscation clients,” which presupposes continued use of the RFC 8907 obfuscation algorithm in some deployments.
      - The only concrete normative enforcement of “obsolescence” in Section 4 is scoped to TLS: “Peers MUST NOT use obfuscation with TLS,” and the mandatory setting of TAC_PLUS_UNENCRYPTED_FLAG for TLS connections. There is no new normative text that forbids obfuscation on *non‑TLS* connections; for those, all of RFC 8907’s obfuscation rules remain intact by reference.
      - This creates an ambiguity about what “obsoleted” means in practice. Does it:
        - (a) mean that the mechanism is entirely retired and MUST NOT be used even on non‑TLS connections (which would contradict the descriptive and migration text, and leave legacy non‑TLS clients with no specified confidentiality option), or
        - (b) mean that the mechanism is simply not to be used *when TLS is available* and is otherwise still defined and usable, or
        - (c) mean merely “historically discouraged” without introducing new normative prohibitions beyond those already in RFC 8907’s Security Considerations?
      - Because the term “obsoleted” in IETF practice often has a specific document‑level meaning (“Obsoletes: …” in the header), using it in the body text without clear transport‑scope qualifications invites misinterpretation about the intended status of RFC 8907’s obfuscation for non‑TLS TACACS+.
      - For implementers and operators, this affects decisions such as whether new non‑TLS deployments are allowed to use obfuscation at all (given that non‑TLS is itself still being accommodated during migration), and whether support for the obfuscation mode is mandatory, optional, or now forbidden in updated implementations.
      - A clarification is needed to narrow the scope of “obsoleted” to what is actually enforced normatively—namely, that obfuscation MUST NOT be used on TLS TACACS+ connections, while the behavior and (deprecated) status of obfuscation on non‑TLS TACACS+ remains as per RFC 8907 unless and until separately updated.

  - Issue-3:
    - BugType: Underspecification
    - ShortLabel: Client-side error handling for TAC_PLUS_UNENCRYPTED_FLAG not explicitly restricted to TLS, overlapping with non‑TLS rules in RFC 8907
    - ScopeProblemType: Missing “over TLS” condition on a normative client rule, causing overlap with existing non‑TLS behavior
    - Evidence:
      - RFC 9887, Section 4 (server-side over TLS):
        - “A TLS TACACS+ server that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 over a TLS connection MUST return an error of TAC_PLUS_AUTHEN_STATUS_ERROR, TAC_PLUS_AUTHOR_STATUS_ERROR, or TAC_PLUS_ACCT_STATUS_ERROR as appropriate for the TACACS+ message type, with the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1, and terminate the session. This behavior corresponds to that defined in Section 4.5 of [RFC8907] regarding data obfuscation for TAC_PLUS_UNENCRYPTED_FLAG or key mismatches.”
      - Immediately following sentence (client-side, no TLS qualifier):
        - “A TACACS+ client that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the session, and SHOULD log this error.”
      - RFC 8907, Section 10.5.2 (non‑TLS client behavior on flag/config mismatches):
        - When responses have TAC_PLUS_UNENCRYPTED_FLAG inconsistent with expected obfuscation settings, “the TACACS+ client MUST close the TCP session, and process the response in the same way that a TAC_PLUS_AUTHEN_STATUS_FAIL (authentication sessions) or TAC_PLUS_AUTHOR_STATUS_FAIL (authorization sessions) was received.”
    - DetailedReasoning:
      - Section 4 of RFC 9887 is introduced as describing “how the TACACS+ client and servers MUST operate regarding the obfuscation mechanism” after stating that obfuscation is “hereby obsoleted.” The first two normative items in that section clearly refer to TLS cases: “Peers MUST NOT use obfuscation *with TLS*” and a client “initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit… All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.” The next sentence explicitly scopes the server’s reaction to “a packet… over a TLS connection”.
      - However, the final sentence shifts back to generic language: “A TACACS+ client that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the session, and SHOULD log this error,” without saying “over a TLS connection” or otherwise tying this behavior explicitly to TLS.
      - RFC 8907 already specifies what a non‑TLS client must do when TAC_PLUS_UNENCRYPTED_FLAG usage is inconsistent with the negotiated obfuscation configuration (Section 10.5.2): the client “MUST close the TCP session” and *also* “process the response in the same way that” a FAIL status was received. That is, for non‑TLS, such a mismatch is mapped into the AAA result processing as a FAIL, not merely treated as an opaque error and discarded.
      - If an implementer reads the unqualified client rule in RFC 9887 Section 4 as applying to *all* TACACS+ connections, it silently overrides and simplifies the more detailed RFC 8907 non‑TLS behavior: instead of mapping the response to FAIL, the client would just terminate the session and log, leaving application-level handling unspecified. That constitutes a change to the core protocol beyond “the direct implications of deprecating obfuscation,” contradicting RFC 9887’s own statement that it “does not make any changes to the core TACACS+ protocol, other than the direct implications of deprecating obfuscation” (Section 5.1).
      - Conversely, if the intention is that RFC 9887’s client-side rule applies only to TLS sessions (as symmetry with the immediately preceding server‑side TLS rule strongly suggests), then its scope is insufficiently specified: the missing “over a TLS connection” condition makes it unclear that this is a TLS‑only behavior and leaves room for misinterpretation.
      - This is fundamentally a scope issue: a TLS-specific behavioral rule is written in a way that appears global, and it overlaps with an existing global rule in RFC 8907. Clarifying that the “MUST terminate the session” behavior applies only to packets received *on TLS TACACS+ connections* would restore the separation: RFC 9887 governs TLS error handling, while RFC 8907 Section 10.5.2 continues to govern non‑TLS mismatches.

- ResidualUncertainties:
  - It is not fully clear whether the authors intended “obfuscation is hereby obsoleted” to be interpreted as a strictly TLS‑scoped deprecation (i.e., “obfuscation MUST NOT be used when TLS is used as transport”), or as a broader statement that new non‑TLS deployments should avoid obfuscation entirely while still leaving the mechanism specified for legacy interoperability. The current text does not clearly distinguish between document‑status obsolescence and feature‑level prohibition across transports.
  - The interplay between RFC 8907 Section 4.4’s connection-level handling of ERROR for key/obfuscation problems and RFC 9887’s TLS‑side “terminate the session” wording is not fully disentangled here; a more explicit statement in RFC 9887 about whether 4.4’s connection‑reuse rules still apply unchanged over TLS would further clarify the error-handling scope.

[Used vector stores: vs_6958c1299d1881919236f07c8d11bc8e]

## Causal Expert
--------------------------------------------------------------------------------

### Expert Analysis:
--------------------------------------------------------------------------------
1. **Summary**

Following RFC 9887 and RFC 8907 together, *as written*, yields a coherent and executable behavior for TAC_PLUS_UNENCRYPTED_FLAG on TLS vs non‑TLS connections. There is a textual normative tension (“MUST NOT set” vs “MUST set”), but it is cleanly resolved by scope (non‑TLS vs TLS) and by the explicit “updates / obsoletes obfuscation” language, so nothing becomes unimplementable or fundamentally broken. The only real gap is that the client‑side AAA decision after detecting a wrong flag over TLS is not explicitly spelled out.

---

2. **Causal Analysis**

### 2.1. Baseline behavior in RFC 8907 (non‑TLS)

For the original, non‑TLS TACACS+:

- Obfuscation is the normal mode:
  - “The flag field MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set to 0 so that the packet body is obfuscated”  .
- The “unencrypted” flag is a deprecated test/debug mode:
  - `TAC_PLUS_UNENCRYPTED_FLAG == 0x1 ... MUST NOT be used in production. … A request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true.`  
- Security section:
  - Servers “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set.”  
  - Clients “MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG” and it “MUST NOT be used when the client is in production.”  

So, in the *non‑TLS* world:

- Normal, secure-ish operation: obfuscated body, flag = 0, shared secret used.
- Flag = 1 is explicitly forbidden and causes the server to drop/reject.

### 2.2. New behavior in RFC 9887 (TLS variant)

RFC 9887 is explicit about the scope:

- It introduces “TACACS+ over TLS” as a variant: TACACS+ “over TLS takes the protocol defined in [RFC8907], removes the option for obfuscation, and specifies that TLS 1.3 be used for transport.”  
- It repeatedly narrows its changes to **TLS connections** only:
  - “This document concerns the use of TLS as transport for TACACS+ and does not make any changes to the core TACACS+ protocol, other than the direct implications of deprecating obfuscation.”  

Section 4 then gives *new, explicit* rules for the flag on TLS connections:

- “Peers MUST NOT use obfuscation with TLS.”  
- “A TACACS+ client initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit… All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.”  
- Server behavior if it sees flag ≠ 1 **over TLS**:
  - “MUST return an error of TAC_PLUS_AUTHEN_STATUS_ERROR, TAC_PLUS_AUTHOR_STATUS_ERROR, or TAC_PLUS_ACCT_STATUS_ERROR … with the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1, and terminate the session.”  
- Client behavior if it sees flag ≠ 1:
  - “MUST terminate the session, and SHOULD log this error.”  

This is the exact opposite of the non‑TLS semantics: for TLS, the flag is *required* to be 1 at all times, and 0 is treated as a hard error.

Causally, if you implement this literally with a clear “is this a TLS connection?” switch:

- **Non‑TLS connection (RFC 8907 only):**
  - Client: MUST NOT set flag = 1; normal operation requires flag = 0.
  - Server: MUST reject flag = 1 and require a shared secret for obfuscation.
- **TLS connection (RFC 9887):**
  - Client: MUST set flag = 1 on every TACACS+ packet; MUST NOT apply obfuscation.
  - Server: MUST require flag = 1; seeing flag ≠ 1 → send *_STATUS_ERROR, close TLS/TCP.
  - Client seeing flag ≠ 1 from server → close connection and log.

No step is impossible: for every incoming TACACS+ packet, the role knows whether the connection is TLS or not (it either did a TLS handshake or it didn’t), and can apply the appropriate per‑connection rule.

### 2.3. Resolution of the apparent “MUST NOT” vs “MUST” conflict

The apparent contradiction is:

- RFC 8907: clients “MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG” and servers “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set”  .
- RFC 9887 (for TLS): clients “MUST set” that flag on all packets; servers “MUST” error if it is **not** set  .

From a protocol execution standpoint, this is not a logical deadlock because:

- RFC 9887 is explicitly an **update** and explicitly says it “removes the option for obfuscation” and “obsoletes the use of TACACS+ obfuscation mechanisms”  .
- The 8907 MUST‑NOTs clearly apply to the **obfuscation / non‑TLS** model. 9887 introduces a **new usage** of the same bit in the “TLS over TACACS+” context, where obfuscation is forbidden and the only meaningful value of the bit is “not obfuscated”.

So a correct implementation simply branches:

```text
if connection_is_tls:
    # RFC 9887 rules
    always send flag=1; require peer to send flag=1; treat flag!=1 as ERROR+close
else:
    # RFC 8907 rules
    send flag=0 in normal production; never send flag=1 in production;
    reject peers that send flag=1.
```

Every MUST is satisfiable: you never have to satisfy “MUST set” and “MUST NOT set” simultaneously on the same connection.

### 2.4. “Corresponds to Section 4.5” language

9887 says the TLS‑error behavior “corresponds to that defined in Section 4.5 of [RFC8907] regarding data obfuscation for TAC_PLUS_UNENCRYPTED_FLAG or key mismatches.”  

- For key mismatches, 8907 indeed requires the server to return ERROR and then complete/close the connection according to 4.4  .
- For `TAC_PLUS_UNENCRYPTED_FLAG == 1`, 8907 says “A request MUST be dropped”  , not “return ERROR”. So 9887 slightly changes behavior in that case by *mandating an explicit *_STATUS_ERROR reply* before closing.

This is a small semantic change in an error/misconfiguration path, but it is fully implementable and consistent with the general 8907 model that ERROR means “processing did not complete; behave as if server could not be contacted”  . It does not create an impossible situation for either peer.

### 2.5. Client’s AAA decision on wrong-flag over TLS

One place where behavior is not completely spelled out:

- For TLS, 9887 says only: client “MUST terminate the session, and SHOULD log this error” when it receives a packet with the flag not set to 1  .
- It does **not** say explicitly whether the client should treat this as:
  - equivalent to TAC_PLUS_*_STATUS_ERROR (server unavailable / try backup), or
  - equivalent to a FAIL (deny the user outright), or
  - implementation-specific.

Given 8907’s model that ERROR means “behave as if that host could not be contacted”  , the most natural reading is “treat it like a transport/protocol error / server unreachable”. Still, different implementations could reasonably make different choices here.

This is an underspecification of behavior in an error/misconfiguration case, but it does not break the protocol machinery: the client can always close the connection and either deny or retry according to local policy.

---

3. **Problem Classification**

- **Not a Causal Inconsistency.**  
  There is no unimplementable requirement or state-machine deadlock. A compliant implementation can satisfy all MUSTs by conditioning on “TLS vs non‑TLS” at the connection level, which is a clear and observable state.

- **Mild Causal Underspecification (error handling semantics only).**  
  For TLS clients detecting an incorrect flag from the server, the AAA decision (treat as FAIL vs ERROR/server‑unreachable) is not spelled out, so different implementations may react differently in that error path. This affects how access decisions are made in the face of a misbehaving peer, but does not affect normal interoperable operation between compliant peers.

---

4. **Minimal Fix Shape (Optional)**

If the RFC is revised, the most targeted clarifications would be:

- Explicitly state that, **for TLS connections**, RFC 9887 overrides the TAC_PLUS_UNENCRYPTED_FLAG requirements in Sections 4.5 and 10.5.2 of RFC 8907, and define the bit as “MUST be 1 to indicate that TACACS+ obfuscation is not in use on a TLS connection”.
- In Section 4 of RFC 9887, specify that when a TLS client terminates the session due to receiving a packet with the flag not set to 1, it **MUST treat this as equivalent to a *_STATUS_ERROR / server-unreachable** for AAA decision purposes (or explicitly say it is implementation-specific).

---

5. **Non-Issues**

- The fact that 8907’s text says “MUST NOT set” for the flag, and 9887 says “MUST set” for TLS, is not a protocol bug once the “over TLS” scope and “updates/obsoletes obfuscation” language are taken into account. It is a normal evolution: the same bit now has opposite meaning depending on whether the underlying transport is the legacy non‑TLS model or the new TLS variant.  
- The requirement for the TLS server to send *_STATUS_ERROR and then close is implementable for all three TACACS+ message types and fits cleanly into the existing state machines for Authentication, Authorization, and Accounting.

[Used vector stores: vs_6958c1299d1881919236f07c8d11bc8e]

## Deontic Expert
--------------------------------------------------------------------------------

### Expert Analysis:
--------------------------------------------------------------------------------
DeonticAnalysis:
- ExcerptSummary:  
  RFC 9887 Section 4 defines how TACACS+ over TLS must handle the legacy obfuscation mechanism and, in particular, how the TAC_PLUS_UNENCRYPTED_FLAG bit is to be used and what error-handling behavior is required. RFC 8907 defines the original semantics of that flag and strong prohibitions around its use, including connection rejection and request dropping. The two documents must fit together coherently for implementations that support both TLS and non‑TLS TACACS+.

- OverallDeonticRisk: Medium

- Issues:

  - Issue-1:
    - BugType: Both
    - Title: Unclear normative override of TAC_PLUS_UNENCRYPTED_FLAG rules for TLS vs. non‑TLS, leading to apparent MUST vs. MUST NOT conflicts
    - Description:  
      RFC 8907 gives very strong, apparently unconditional rules about TAC_PLUS_UNENCRYPTED_FLAG: the flag “MUST NOT be used in production”; the flag field “MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set to 0” for normal operation; “A request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true”; servers “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set”; and “TACACS+ clients MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG” (with an exception only via explicit test configuration).   At the same time, RFC 9887 Section 4 states that, for TLS, “Peers MUST NOT use obfuscation with TLS” and that “A TACACS+ client initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit … All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1,” while a TLS server seeing a packet over TLS with the bit not set to 1 “MUST” send a *_STATUS_ERROR and terminate the session.    
      Logically, without scoping, the earlier “MUST NOT set” / “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set” from RFC 8907 and the later “MUST set … and all subsequent packets MUST have … set to 1” from RFC 9887 are incompatible for the same actors in production TLS deployments. RFC 9887 does say that obfuscation “is hereby obsoleted” and that obfuscation “MUST NOT be applied when operating over TLS”, and that TACACS+ over TLS otherwise follows RFC 8907, but it does not explicitly say that the specific UNENCRYPTED_FLAG rules of RFC 8907 (Sections 4.1, 4.5, and 10.5.2) are limited to non‑TLS connections or are overridden for TLS.    
      A careful reader can infer that the old UNENCRYPTED_FLAG prohibitions were written only for the non‑TLS, MD5‑obfuscation context and that RFC 9887 is redefining the operational requirements for TLS connections; however, this is not spelled out. Because RFC 9887 §3.2 also says that “the exchange of TACACS+ data MUST proceed in accordance with the procedures defined in [RFC8907]” except for obfuscation, the scope of which parts of RFC 8907 are still binding for TLS is left ambiguous.   The result is a normative structure where, taken literally, clients and servers are simultaneously under a “MUST NOT set” from RFC 8907 and a “MUST set” from RFC 9887 when operating over TLS, and servers are both required to “reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set” and to treat packets with the flag unset over TLS as protocol errors.  
      The intended resolution is almost certainly that: (a) the original UNENCRYPTED_FLAG constraints in RFC 8907 apply only to non‑TLS TACACS+ connections using the obfuscation mechanism; and (b) for TLS TACACS+ connections, they are replaced by the rules in RFC 9887 Section 4. But because this override and its scope are never stated explicitly, there is both an apparent inconsistency (MUST vs. MUST NOT on the same behavior) and underspecification (implementers must guess that the later text wins for TLS and that all earlier UNENCRYPTED_FLAG prohibitions are non‑TLS‑only).
    - KeyTextSnippets:
      - RFC 8907, header flags: “TAC_PLUS_UNENCRYPTED_FLAG := 0x01 … This option MUST NOT be used in production. … This flag SHOULD be clear in all deployments.”  
      - RFC 8907, Section 4.5: “The flag field MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set to 0 so that the packet body is obfuscated… TAC_PLUS_UNENCRYPTED_FLAG == 0x1 … This option is deprecated and MUST NOT be used in production. … A request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true.”  
      - RFC 8907, Section 10.5.2: “TACACS+ servers MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set.” / “TACACS+ clients MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG. Clients MUST be implemented in a way that requires explicit configuration to enable the use of TAC_PLUS_UNENCRYPTED_FLAG. This option MUST NOT be used when the client is in production.”  
      - RFC 9887, Section 3.2: “Once the TLS connection has been successfully established, the exchange of TACACS+ data MUST proceed in accordance with the procedures defined in [RFC8907]. However, … The TACACS+ obfuscation mechanism defined in [RFC8907] MUST NOT be applied when operating over TLS (Section 4).”  
      - RFC 9887, Section 4: “obfuscation is hereby obsoleted. This section describes how the TACACS+ client and servers MUST operate regarding the obfuscation mechanism. Peers MUST NOT use obfuscation with TLS.”  
      - RFC 9887, Section 4: “A TACACS+ client initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit… All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.”  
      - RFC 9887, Section 4: “A TLS TACACS+ server that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 over a TLS connection MUST return an error … with the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1, and terminate the session.”  
    - Impact:  
      Implementers who mechanically apply RFC 8907’s MUST/MUST NOT rules alongside RFC 9887 can be left unsure whether TLS clients are allowed or required to set TAC_PLUS_UNENCRYPTED_FLAG, and whether TLS servers should reject or accept such packets. This can lead to non‑interoperable behavior (e.g., servers treating correctly flagged TLS packets as violations per RFC 8907) or to implementers informally “deciding” that 9887 overrides 8907 without a clear textual basis. A small, explicit clarification that 8907’s UNENCRYPTED_FLAG restrictions apply only to non‑TLS TACACS+ connections and are superseded by RFC 9887 Section 4 for TLS would remove this ambiguity.

  - Issue-2:
    - BugType: Underspecification
    - Title: Ambiguous scope of “obfuscation is hereby obsoleted” and of the client’s error-handling rule
    - Description:  
      RFC 9887 states that “obfuscation is hereby obsoleted” and that the document “does not make any changes to the core TACACS+ protocol, other than the direct implications of deprecating obfuscation,” but the concrete normative text that follows in Section 4 is framed almost entirely in terms of TLS: “Peers MUST NOT use obfuscation with TLS”; a client “initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit”; and a TLS server receiving a packet with the bit not set over a TLS connection MUST send an error and terminate the session.   At the same time, the definitions explicitly describe a “Non‑TLS connection” as “using the unsecure TACACS+ authentication and obfuscation (or the unobfuscated option for testing),” and Section 6.2 describes continued operation of non‑TLS TACACS+ clients and servers.    
      This makes it unclear whether “obsoleted” is intended as a global prohibition on obfuscation (which would contradict the ongoing non‑TLS usage described), or only as a statement about the secure TLS profile. The last sentence of Section 4 compounds this by saying, without an explicit TLS scope, that “A TACACS+ client that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the session, and SHOULD log this error.”   For TLS, this is consistent with the earlier requirement that all packets on a TLS connection have the bit set, but read literally across both TLS and non‑TLS it would require clients to terminate *all* sessions when receiving packets with the flag cleared—which is the normal, obfuscated mode in RFC 8907—and would contradict the non‑TLS behavior that RFC 8907 specifies. The likely intent is “when operating over TLS,” but that condition is not actually written.  
      As a result, implementers must infer that (1) “obsoleted” is scoped to TACACS+ over TLS and does not ban obfuscation on legacy non‑TLS transports, and (2) the client‑side “MUST terminate” rule applies only to TLS connections. Those inferences are not guaranteed by the text, leaving a normative gap around the exact scope of obfuscation’s deprecation and the applicability of the new error‑handling rule.
    - KeyTextSnippets:
      - RFC 9887, Section 2 (Non‑TLS connection): “This term refers to the connection defined in [RFC8907]. It is a connection without TLS, using the unsecure TACACS+ authentication and obfuscation (or the unobfuscated option for testing).”  
      - RFC 9887, Section 4: “The introduction of TLS authentication and encryption to TACACS+ replaces this former mechanism, so obfuscation is hereby obsoleted. This section describes how the TACACS+ client and servers MUST operate regarding the obfuscation mechanism. Peers MUST NOT use obfuscation with TLS.”  
      - RFC 9887, Section 4 (client behavior): “A TACACS+ client that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the session, and SHOULD log this error.”  
      - RFC 9887, Sections 6.1–6.2: discussion of migration and “Some TACACS+ client devices in a deployment may not implement TLS. These devices will require access to non‑TLS TACACS+ servers.”  
      - RFC 8907, Section 4.5: normal non‑TLS obfuscated operation requires TAC_PLUS_UNENCRYPTED_FLAG set to 0.  
    - Impact:  
      The lack of explicit scoping invites misinterpretation: some readers might conclude that obfuscation must not be used at all any more, even on legacy non‑TLS transports that RFC 9887 explicitly anticipates, or that the “client MUST terminate” rule applies to any packet with the flag cleared, which would make standard 8907‑compliant non‑TLS operation impossible. While most implementers will likely understand from context that Section 4’s rules are meant for TLS connections, the spec would be more robust and interoperable if it clearly restricted those requirements to TLS and, if desired, explicitly stated whether obfuscation remains allowed (albeit discouraged) on non‑TLS TACACS+ connections.

[Used vector stores: vs_6958c1299d1881919236f07c8d11bc8e]

## CrossRFC Expert
--------------------------------------------------------------------------------

### Expert Analysis:
--------------------------------------------------------------------------------
CrossRFCReport:
- ExcerptSummary: Section 4 of RFC 9887 reuses the legacy TAC_PLUS_UNENCRYPTED_FLAG from RFC 8907 to signal “no TACACS+ obfuscation” on TLS connections, and defines new error-handling when the flag is not set, while RFC 8907 originally tied this flag tightly to the obfuscation mechanism and strongly discouraged its use. The two RFCs must therefore be read together to understand when TAC_PLUS_UNENCRYPTED_FLAG is required, forbidden, or obsolete, and how ERROR vs drop/Fail behaviors apply.

- OverallCrossRFCLikelihood: Medium

- Issues:
  - Issue-1:
    - BugType: Underspecification
    - ShortLabel: TLS use of TAC_PLUS_UNENCRYPTED_FLAG vs RFC 8907 “MUST NOT set” rules
    - Description: RFC 8907 defines TAC_PLUS_UNENCRYPTED_FLAG as indicating that the packet body is not obfuscated and says this option “MUST NOT be used in production” and “SHOULD be clear in all deployments” in the common header definition . In the obfuscation section, it further specifies that when TAC_PLUS_UNENCRYPTED_FLAG == 0x1 the option is deprecated, MUST NOT be used in production, and that “a request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true” , and in the security best-practices text it tightens this by requiring that servers “MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set” and that clients “MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG” except via explicit configuration, again with “MUST NOT … in production” . RFC 9887 Section 4, for TLS connections, flips this: a TLS TACACS+ client initiating a TLS connection “MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit” and “All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1”, and a TLS server on TLS “MUST” treat packets where the bit is not 1 as an *_STATUS_ERROR and terminate the session. While this is clearly intended to be an update limited to TLS, the document never explicitly says that it overrides RFC 8907 Section 10.5.2 and the “MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG” rule when operating over TLS. Implementers must therefore infer, from context, that 8907’s prohibitions apply only to non‑TLS/obfuscation connections and are superseded for TLS connections; the lack of an explicit “when operating over TLS as defined in this document, the requirements of RFC 8907 Sections 4.1, 4.5, and 10.5.2 regarding TAC_PLUS_UNENCRYPTED_FLAG are updated as follows …” creates room for confusion (e.g., an implementer trying to honor both contradictory MUSTs simultaneously). This is not a hard contradiction—9887 is clearly an update—but the cross-RFC scoping of the flag’s semantics over TLS vs non‑TLS is left implicit rather than normative.
    - EntitiesInvolved: ["TAC_PLUS_UNENCRYPTED_FLAG", "RFC 8907 Section 4.1 (flags)", "RFC 8907 Section 4.5 (Data Obfuscation)", "RFC 8907 Section 10.5.2 (Connections and Obfuscation)", "RFC 9887 Section 4 (Obsolescence of TACACS+ Obfuscation)"]
    - CrossRefsUsed: ["RFC 8907 header flag definition with MUST NOT use of TAC_PLUS_UNENCRYPTED_FLAG in production ", "RFC 8907 Section 4.5 rules for TAC_PLUS_UNENCRYPTED_FLAG == 1 and dropping requests ", "RFC 8907 Section 10.5.2 MUST NOT set / MUST reject connections requirements "]
    - Confidence: High

  - Issue-2:
    - BugType: Underspecification
    - ShortLabel: Ambiguous scope of “obsolescence” of obfuscation vs ongoing non‑TLS use
    - Description: RFC 9887 Section 4 says that the MD5-based obfuscation mechanism “is weak” and that “obfuscation is hereby obsoleted,” and the section title is “Obsolescence of TACACS+ Obfuscation,” which could be read as a protocol‑wide prohibition. However, elsewhere in RFC 9887, “Non‑TLS connection” is explicitly defined as the RFC 8907 TACACS+ connection “using the unsecure TACACS+ authentication and obfuscation (or the unobfuscated option for testing),” and migration sections describe continued operation of non‑TLS TACACS+ servers for legacy clients, without stating that obfuscation itself is now a MUST‑NOT . At the same time, RFC 8907 still contains detailed normative behavior for obfuscation (MD5 pad generation, ERROR handling on key mismatch, and the requirement that TAC_PLUS_UNENCRYPTED_FLAG be 0 for normal operation) and its security section explicitly frames these mechanisms as “obfuscation” rather than real encryption, but does not mark them “Historic” or “MUST NOT” overall; instead it gives best‑practice guidance that TACACS+ must run over a secure transport . The combination leaves it unclear whether RFC 9887 is merely recommending against new use of obfuscation (while allowing it for installed non‑TLS base as in RFC 8907), or is formally updating RFC 8907 to prohibit obfuscation in all future deployments while still documenting it for legacy interop. That ambiguity in the cross‑RFC relationship could lead to differing interpretations about whether a compliant non‑TLS RFC 8907 implementation deployed today is still allowed to support obfuscation, or whether it is now formally out of spec.
    - EntitiesInvolved: ["TACACS+ obfuscation mechanism", "RFC 8907 Section 4.5 (Data Obfuscation)", "RFC 8907 Section 10.1 and 10.5 (Security and Best Practices)", "RFC 9887 Section 2 (Non-TLS connection definition)", "RFC 9887 Section 4 (Obsolescence of TACACS+ Obfuscation)"]
    - CrossRefsUsed: ["RFC 8907 Section 4.5 description of the obfuscation algorithm and related ERROR behavior ", "RFC 8907 Section 10.1 and 10.5 best-practice framing of obfuscation as insecure and requiring secure transport ", "RFC 9887’s own definition of Non‑TLS connection and reference to continued use of TACACS+ obfuscation/unobfuscated modes (from the user-provided excerpt)"]
    - Confidence: Medium

- IfNoIssues:

[Used vector stores: vs_6958c1299d1881919236f07c8d11bc8e]

## Terminology Expert
--------------------------------------------------------------------------------

### Expert Analysis:
--------------------------------------------------------------------------------
TerminologyAnalysis:
- OverallBugLikelihood: None
- Issues:
  - Issue-1:
    - BugType: None
    - Severity: Low
    - ShortLabel: TAC_PLUS_UNENCRYPTED_FLAG semantics for TLS vs non‑TLS are scoped, not terminologically conflicting
    - Evidence:
      - ExcerptSnippets:
        - RFC 8907 defines the flag bit and its meaning:  
          “TAC_PLUS_UNENCRYPTED_FLAG := 0x01  
          This flag indicates that the sender did not obfuscate the body of the packet.” (Section 4.1)
        - RFC 8907 further constrains its use for the classic (non‑TLS) protocol:  
          “The flag field MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set to 0 so that the packet body is obfuscated…” (Section 4.5)  
          “TAC_PLUS_UNENCRYPTED_FLAG == 0x1 … This option is deprecated and MUST NOT be used in production. … A request MUST be dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true.” (Section 4.5)  
          “TACACS+ servers MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set.”  
          “TACACS+ clients MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG. Clients MUST be implemented in a way that requires explicit configuration to enable the use of TAC_PLUS_UNENCRYPTED_FLAG. This option MUST NOT be used when the client is in production.” (Section 10.5.2)
        - RFC 9887 defines distinct connection types and scope:  
          “Non-TLS connection:  This term refers to the connection defined in [RFC8907]. It is a connection without TLS…”  
          “TLS connection:  A TLS connection is a TCP/IP connection with TLS authentication and encryption used by TACACS+ for transport.” (Section 2)
        - RFC 9887 obsoletes obfuscation *for TLS* and mandates use of the flag there:  
          “The obfuscation mechanism documented in Section 4.5 of [RFC8907] is weak. … obfuscation is hereby obsoleted. This section describes how the TACACS+ client and servers MUST operate regarding the obfuscation mechanism.  
          Peers MUST NOT use obfuscation with TLS.  
          A TACACS+ client initiating a TACACS+ TLS connection MUST set the TAC_PLUS_UNENCRYPTED_FLAG bit, thereby asserting that obfuscation is not used for the session. All subsequent packets MUST have the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.” (Section 4)
        - RFC 9887’s error handling over TLS using the same flag:  
          “A TLS TACACS+ server that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 over a TLS connection MUST return an error of TAC_PLUS_AUTHEN_STATUS_ERROR, TAC_PLUS_AUTHOR_STATUS_ERROR, or TAC_PLUS_ACCT_STATUS_ERROR … with the TAC_PLUS_UNENCRYPTED_FLAG bit set to 1, and terminate the session. This behavior corresponds to that defined in Section 4.5 of [RFC8907] regarding data obfuscation for TAC_PLUS_UNENCRYPTED_FLAG or key mismatches.” (Section 4)  
          “A TACACS+ client that receives a packet with the TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the session, and SHOULD log this error.” (Section 4)
      - ContextSnippets:
        - RFC 9887 explicitly states the overall intent for TLS:  
          “TACACS+ over TLS takes the protocol defined in [RFC8907], removes the option for obfuscation, and specifies that TLS 1.3 be used for transport.” (Section 3)  
          “The TACACS+ obfuscation mechanism defined in [RFC8907] MUST NOT be applied when operating over TLS (Section 4).” (Section 3.2)
    - Reasoning:
      - The underlying *semantic meaning* of `TAC_PLUS_UNENCRYPTED_FLAG` is the same in both RFC 8907 and RFC 9887: “the sender did not obfuscate the body of the packet.” RFC 9887’s text (“thereby asserting that obfuscation is not used for the session”) is fully consistent with that meaning; no new or conflicting *definition* of the flag is introduced.
      - The apparent conflict arises at the level of normative behavior, not terminology:
        - RFC 8907, written for the original non‑TLS protocol, strongly discourages use of the flag in production and requires servers/clients to reject connections or requests where the flag is set.
        - RFC 9887, which updates RFC 8907, introduces a *new transport context* (“TLS connection” vs “Non‑TLS connection” as defined in Section 2) and then mandates that, in the TLS context, all packets MUST carry `TAC_PLUS_UNENCRYPTED_FLAG = 1` to indicate that the inner TACACS+ body is not obfuscated (because obfuscation is forbidden over TLS).
      - From a terminology and naming standpoint, this is a deliberate reuse of the same bit and the same name for the same condition (“no obfuscation”), applied in a different transport regime:
        - Non‑TLS connections (the RFC 8907 world) continue to treat that condition as deprecated and normally disallowed.
        - TLS connections (the RFC 9887 world) *require* that condition because obfuscation is prohibited.
      - RFC 9887 clearly scopes its new requirements to TLS by repeatedly qualifying behavior with “over a TLS connection” and by separately defining “Non‑TLS connection” as the RFC 8907 case. The document as a whole is about “TACACS+ over TLS”, so a careful implementer will naturally apply Section 4’s rules specifically to TLS sessions and continue to use RFC 8907’s rules for non‑TLS sessions.
      - There is no inconsistent renaming, no use of different names for the same flag, and no reference to a non‑existent flag or status code. The status constants `TAC_PLUS_AUTHEN_STATUS_ERROR`, `TAC_PLUS_AUTHOR_STATUS_ERROR`, and `TAC_PLUS_ACCT_STATUS_ERROR` are correctly named and match their definitions in RFC 8907.
      - The sentence “This behavior corresponds to that defined in Section 4.5 of [RFC8907]…” is somewhat loose (RFC 8907’s behavior for `TAC_PLUS_UNENCRYPTED_FLAG == 1` is “drop” rather than “send *_STATUS_ERROR”), but this is commentary, not the normative rule; the actual required behavior for TLS is explicitly described just before that sentence and is unambiguous.
      - Because the underlying flag meaning is unchanged and the new usage is clearly scoped to TLS, implementers are not misled about what the flag *is*; they simply must apply different policy for TLS vs non‑TLS connections, which is clearly explained. This is not a terminology/naming bug but a deliberate update of normative behavior in a new context.
    - PatchSuggestion:
      - None. A possible future clarity improvement (not necessary for an erratum) would be to add an explicit sentence such as: “This document updates Sections 4.5 and 10.5.2 of [RFC8907] for TLS connections: when operating over TLS, the requirements that clients ‘MUST NOT set’ TAC_PLUS_UNENCRYPTED_FLAG and that servers ‘MUST reject connections that have TAC_PLUS_UNENCRYPTED_FLAG set’ are replaced by the requirements in this Section 4.” But the current text is already sufficiently clear for implementation.

- Notes:
  - UsedRouterIssues: Considered the router’s candidate issue about a normative conflict/unclear override of `TAC_PLUS_UNENCRYPTED_FLAG` semantics between RFC 8907 and RFC 9887. Concluded that, from a terminology and naming perspective, there is no actual bug: the flag name and meaning are consistent, and the differing requirements are properly scoped to different connection types (non‑TLS vs TLS) in the updating RFC.
  - NewIssuesFromExpert: false
  - Limitations:
    - Analysis is based solely on the provided excerpts. I assume (consistently with the text) that RFC 9887 is formally marked as “Updates: 8907” in its front matter. If that were not the case, one might want a more explicit statement of its updating relationship, but that would still be a document‑level editorial concern rather than a terminology bug with the flag or status names themselves.

[Used vector stores: vs_6958c1299d1881919236f07c8d11bc8e]


Vector Stores Used: vs_6958c1299d1881919236f07c8d11bc8e