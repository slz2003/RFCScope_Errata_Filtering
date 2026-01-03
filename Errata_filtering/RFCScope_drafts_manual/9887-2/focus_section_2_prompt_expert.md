Let us analyze section 2 of RFC 9887. All references made by section 2 have also been included below.

# RFC 9887: Terminal Access Controller Access-Control System Plus (TACACS+) over TLS 1.3

## 1. Introduction

   "The Terminal Access Controller Access-Control System Plus (TACACS+)
   Protocol" [RFC8907] provides device administration for routers,
   network access servers, and other networked computing devices via one
   or more centralized TACACS+ servers.  The protocol provides
   authentication, authorization, and accounting services (AAA) for
   TACACS+ clients within the device administration use case.

   The content of the protocol is highly sensitive and requires secure
   transport to safeguard a deployment.  However, TACACS+ lacks
   effective confidentiality, integrity, and authentication of the
   connection and network traffic between the TACACS+ server and client.
   The security mechanisms as described in Section 4.5 of [RFC8907] are
   extremely weak.

   To address these deficiencies, this document updates the TACACS+
   protocol to use TLS 1.3 authentication and encryption [RFC8446], and
   obsoletes the use of TACACS+ obfuscation mechanisms.  The maturity of
   TLS in version 1.3 and above makes it a suitable choice for the
   TACACS+ protocol.

## 2. Technical Definitions

   The terms defined in Section 3 of [RFC8907] are fully applicable here
   and will not be repeated.  The following terms are also used in this
   document.

   Obfuscation:  TACACS+ was originally intended to incorporate a
      mechanism for securing the body of its packets.  The algorithm is
      categorized as obfuscation in Section 10.5.2 of [RFC8907].  The
      term is used to ensure that the algorithm is not mistaken for
      encryption.  It should not be considered secure.

   Non-TLS connection:  This term refers to the connection defined in
      [RFC8907].  It is a connection without TLS, using the unsecure
      TACACS+ authentication and obfuscation (or the unobfuscated option
      for testing).  The use of well-known TCP/IP host port number 49 is
      specified as the default for non-TLS connections.

   TLS connection:  A TLS connection is a TCP/IP connection with TLS
      authentication and encryption used by TACACS+ for transport.  A
      TLS connection for TACACS+ is always between one TACACS+ client
      and one TACACS+ server.

   TLS TACACS+ server:  This document describes a variant of the TACACS+
      server, introduced in Section 3.2 of [RFC8907], which utilizes TLS
      for transport, and makes some associated protocol optimizations.
      Both server variants respond to TACACS+ traffic, but this document
      specifically defines a TACACS+ server (whether TLS or non-TLS) as
      being bound to a specific port number on a particular IP address
      or hostname.  This definition is important in the context of the
      configuration of TACACS+ clients to ensure they direct their
      traffic to the correct TACACS+ servers.

   Peer:  The peer of a TACACS+ client (or server) in the context of a
      TACACS+ connection, is a TACACS+ server (or client).  Together,
      the ends of a TACACS+ connection are referred to as peers.

### 2.1. Requirements Language

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

## 3. TACACS+ over TLS

   TACACS+ over TLS takes the protocol defined in [RFC8907], removes the
   option for obfuscation, and specifies that TLS 1.3 be used for
   transport (Section 3.1 elaborates on TLS version support).  A new
   well-known default host port number is used.  The next sections
   provide further details and guidance.

   TLS is introduced into TACACS+ to fulfill the following requirements:

   1.  Confidentiality and Integrity: The MD5 algorithm underlying the
       obfuscation mechanism specified in [RFC8907] has been shown to be
       insecure [RFC6151] when used for encryption.  This prevents
       TACACS+ from being used in a deployment compliant with
       [FIPS-140-3].  Securing the TACACS+ protocol with TLS is intended
       to provide confidentiality and integrity without requiring the
       provision of a secured network.

   2.  Peer authentication: The authentication capabilities of TLS
       replace the shared secrets of obfuscation for mutual
       authentication.

   This document adheres to the recommendations in [REQ-TLS13].

### 3.1. Separating TLS Connections

   Peers implementing the TACACS+ protocol variant defined in this
   document MUST apply mutual authentication and encrypt all data
   exchanged between them.  Therefore, when a TCP connection is
   established for the service, a TLS handshake begins immediately.
   Options that upgrade an initial non-TLS connection MUST NOT be used;
   see Section 5.3.

   To ensure clear separation between TACACS+ traffic using TLS and that
   which does not (see Section 5.3), servers supporting TACACS+ over TLS
   MUST listen on a TCP/IP port distinct from that used by non-TLS
   TACACS+ servers.  It is further RECOMMENDED to deploy the TLS and
   non-TLS services on separate hosts, as discussed in Section 5.1.1.

   Given the prevalence of default port usage in existing TACACS+ client
   implementations, this specification assigns the well-known TCP port
   number 300 for TACACS+ over TLS (see Section 7).

   While the use of the designated port number is strongly encouraged,
   deployments with specific requirements MAY use alternative TCP port
   numbers.  In such cases, operators must carefully consider the
   operational implications described in Section 5.3.

### 3.2. TLS Connection

   A TACACS+ client initiates a TLS connection by making a TCP
   connection to a configured TLS TACACS+ server on the TACACS+ TLS port
   number.  Once the TCP connection is established, the client MUST
   immediately begin the TLS negotiation before sending any TACACS+
   protocol data.

   A minimum of TLS 1.3 [RFC8446] MUST be used for transport.  It is
   expected that TACACS+, as described in this document, will work with
   future versions of TLS.  Earlier versions of TLS MUST NOT be used.

   Once the TLS connection has been successfully established, the
   exchange of TACACS+ data MUST proceed in accordance with the
   procedures defined in [RFC8907].  However, all TACACS+ messages SHALL
   be transmitted as TLS application data.  The TACACS+ obfuscation
   mechanism defined in [RFC8907] MUST NOT be applied when operating
   over TLS (Section 4).

   TLS TACACS+ connections are generally not long-lived.  The connection
   will be closed by either a peer if it encounters an error or an
   inactivity timeout.

   For connections not operating in Single Connection Mode (as defined
   in Section 4.3 of [RFC8907]), the TCP session SHALL be closed upon
   completion of the associated TACACS+ session.  Connections operating
   in Single Connection Mode MAY persist for a longer duration but are
   typically subject to timeout and closure after a brief period of
   inactivity.  Consequently, support for transport-layer keepalive
   mechanisms is not required.

   Why a connection is closed has no bearing on TLS resumption, unless
   closed by a TLS error, in which case it is possible that the ticket
   has been invalidated.

   TACACS+ clients and servers widely support IPv6 configuration in
   addition to IPv4.  This document makes no changes to recommendations
   in this area.

### 3.3. TLS Authentication Options

   Implementations MUST support certificate-based mutual authentication,
   to provide a core option for interoperability between deployments.
   This authentication option is specified in Section 3.4.

   In addition to certificate-based TLS authentication, implementations
   MAY support the following alternative authentication mechanisms:

   *  Pre-Shared Keys (PSKs) (Section 3.5), also known as external PSKs
      in TLS 1.3.

   *  Raw Public Keys (RPKs).  The details of RPKs are considered out of
      scope for this document.  Refer to [RFC7250] and Section 4.4.2 of
      [RFC8446] for implementation, deployment, and security
      considerations.

### 3.4. TLS Certificate-Based Authentication

   TLS certificate authentication is the primary authentication option
   for TACACS+ over TLS.  This section covers certificate-based
   authentication only.

   Deploying TLS certificate-based authentication correctly will
   considerably improve the security of TACACS+ deployments.  It is
   essential for implementers and operators to understand the
   implications of a TLS certificate-based authentication solution,
   including the correct handling of certificates, Certificate
   Authorities (CAs), and all elements of TLS configuration.  For
   guidance, start with [BCP195].

   Each peer MUST validate the certificate path of its remote peer,
   including revocation checking, as described in Section 3.4.1.

   If the verification succeeds, the authentication is successful and
   the connection is permitted.  Policy may impose further constraints
   upon the peer, allowing or denying the connection based on
   certificate fields or any other parameters exposed by the
   implementation.

   Unless disabled by configuration, a peer MUST NOT permit connection
   of any peer that presents an invalid TLS certificate.

#### 3.4.1. TLS Certificate Path Verification

   The implementation of certificate-based mutual authentication MUST
   support certificate path validation as described in Section 6 of
   [RFC5280].

   In some deployments, a peer may be isolated from a remote peer's CA.
   Implementations for these deployments MUST support certificate chains
   (aka bundles or chains of trust), where the entire chain of the
   remote peer's certificate is stored on the local peer.

   TLS Cached Information Extension [RFC7924] SHOULD be implemented.
   This MAY be augmented with RPKs [RFC7250], though revocation must be
   handled as it is not part of that specification.

   Other approaches may be used for loading the intermediate
   certificates onto the client, but they MUST include support for
   revocation checking.  For example, [RFC5280] details the Authority
   Information Access (AIA) extension to provide information about the
   issuer of the certificate in which the extension appears.  It can be
   used to provide the address of the Online Certificate Status Protocol
   (OCSP) responder from where the revocation status of the certificate
   (which includes the extension) can be checked.

#### 3.4.2. TLS Certificate Identification

   For the client-side validation of presented TLS TACACS+ server
   identities, implementations MUST follow the validation techniques
   defined in [RFC9525].  Identifier types DNS-ID, IP-ID, or SRV-ID are
   applicable for use with the TLS TACACS+ protocol; they are selected
   by operators depending upon the deployment design.  TLS TACACS+ does
   not use URI-IDs for TLS TACACS+ server identity verification.

   Wildcards in TLS TACACS+ server identities simplify certificate
   management by allowing a single certificate to secure multiple
   servers in a deployment.  However, this introduces security risks, as
   compromising the private key of a wildcard certificate impacts all
   servers using it.  To address these risks, the guidelines in Section 6.3 of [RFC9525] MUST be followed, and the wildcard SHOULD be
   confined to a subdomain dedicated solely to TACACS+ servers.

   For the TLS TACACS+ server-side validation of client identities,
   implementations MUST support the ability to configure which fields of
   a certificate are used for client identification to verify that the
   client is a valid source for the received certificate and that it is
   permitted access to TACACS+. Implementations MUST support either:

   *  Network-address-based validation methods as described in Section 5.2 of [RFC5425] or

   *  Client Identity validation of a shared identity in the certificate
      subjectAltName.  This is applicable in deployments where the
      client securely supports an identity which is shared with the TLS
      TACACS+ server.  Matching of dNSName and iPAddress MUST be
      supported.  Other options defined in Section 4.2.1.6 of [RFC5280]      MAY be supported.  This approach allows a client's network
      location to be reconfigured without issuing a new client
      certificate.

   Implementations MUST support the TLS Server Name Indication (SNI)
   extension (Section 3 of [RFC6066]).  TLS TACACS+ clients MUST support
   the ability to configure the TLS TACACS+ server's domain name, so
   that it may be included in the SNI "server_name" extension of the
   client hello (This is distinct from the IP Address or hostname
   configuration used for the TCP connection).  Refer to Section 5.1.5   for security related operator considerations.

   Certificate provisioning is out of scope of this document.

#### 3.4.3. Cipher Suites Requirements

   Implementations MUST support the TLS 1.3 mandatory cipher suites
   (Section 9.1 of [RFC8446]).  Readers should refer to [BCP195].  The
   cipher suites offered or accepted SHOULD be configurable so that
   operators can adapt.

### 3.5. TLS PSK Authentication

   As an alternative to certificate-based authentication,
   implementations MAY support PSKs, also known as external PSKs in TLS
   1.3 [RFC8446].  These should not be confused with resumption PSKs.

   The use of external PSKs is less well established than certificate-
   based authentication.  It is RECOMMENDED that systems follow the
   directions of [RFC9257] and Section 4 of [RFC8446].

   Where PSK authentication is implemented, PSK lengths of at least 16
   octets MUST be supported.

   PSK identity MUST follow recommendations of Section 6.1 of [RFC9257].
   Implementations MUST support PSK identities of at least 16 octets.

   Although this document removes the option of obfuscation (Section 4),
   it is still possible that the TLS and non-TLS versions of TACACS+
   exist in an organization, for example, during migration
   (Section 6.1).  In such cases, the shared secrets configured for
   TACACS+ obfuscation clients MUST NOT be the same as the PSKs
   configured for TLS clients.

### 3.6. TLS Resumption

   TLS Resumption [RFC8446] can minimize the number of round trips
   required during the handshake process.  If a TLS client holds a
   ticket previously extracted from a NewSessionTicket message from the
   TLS TACACS+ server, it can use the PSK identity tied to that ticket.
   If the TLS TACACS+ server consents, the resumed session is
   acknowledged as authenticated and securely linked to the initial
   session.

   The client SHOULD use resumption when it holds a valid unused ticket
   from the TLS TACACS+ server, as each ticket is intended for a single
   use only and will be refreshed during resumption.  The TLS TACACS+
   server can reject a resumption request, but the TLS TACACS+ server
   SHOULD allow resumption if the ticket in question has not expired and
   has not been used before.

   When a TLS TACACS+ server is presented with a resumption request from
   the TLS client, it MAY still choose to require a full handshake.  In
   this case, the negotiation proceeds as if the session was a new
   authentication, and the resumption attempt is ignored.  As described
   in Appendix C.4 of [RFC8446], reuse of a ticket allows passive
   observers to correlate different connections.  TLS TACACS+ clients
   and servers SHOULD follow the client tracking preventions in Appendix C.4 of [RFC8446].

   When processing TLS resumption, certificates must be verified to
   check for revocation during the period since the last
   NewSessionTicket Message.

   The resumption ticket_lifetime SHOULD be configurable, including a
   zero seconds lifetime.  Refer to Section 4.6.1 of [RFC8446] for
   guidance on ticket lifetime.

## 4. Obsolescence of TACACS+ Obfuscation

   The obfuscation mechanism documented in Section 4.5 of [RFC8907] is
   weak.

   The introduction of TLS authentication and encryption to TACACS+
   replaces this former mechanism, so obfuscation is hereby obsoleted.
   This section describes how the TACACS+ client and servers MUST
   operate regarding the obfuscation mechanism.

   Peers MUST NOT use obfuscation with TLS.

   A TACACS+ client initiating a TACACS+ TLS connection MUST set the
   TAC_PLUS_UNENCRYPTED_FLAG bit, thereby asserting that obfuscation is
   not used for the session.  All subsequent packets MUST have the
   TAC_PLUS_UNENCRYPTED_FLAG bit set to 1.

   A TLS TACACS+ server that receives a packet with the
   TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 over a TLS connection MUST
   return an error of TAC_PLUS_AUTHEN_STATUS_ERROR,
   TAC_PLUS_AUTHOR_STATUS_ERROR, or TAC_PLUS_ACCT_STATUS_ERROR as
   appropriate for the TACACS+ message type, with the
   TAC_PLUS_UNENCRYPTED_FLAG bit set to 1, and terminate the session.
   This behavior corresponds to that defined in Section 4.5 of [RFC8907]   regarding data obfuscation for TAC_PLUS_UNENCRYPTED_FLAG or key
   mismatches.

   A TACACS+ client that receives a packet with the
   TAC_PLUS_UNENCRYPTED_FLAG bit not set to 1 MUST terminate the
   session, and SHOULD log this error.

## 5. Security Considerations





### 5.1. TLS

   This document improves the confidentiality, integrity, and
   authentication of the connection and network traffic between TACACS+
   peers by adding TLS support.

   Simply adding TLS support to the protocol does not guarantee the
   protection of the TLS TACACS+ server and clients.  It is essential
   for the operators and equipment vendors to adhere to the latest best
   practices for ensuring the integrity of network devices and selecting
   secure TLS key and encryption algorithms.

   [BCP195] offers substantial guidance for implementing and deploying
   protocols that use TLS.  Those implementing and deploying Secure
   TACACS+ must adhere to the recommendations relevant to TLS 1.3
   outlined in [BCP195] or its subsequent versions.

   This document outlines additional restrictions permissible under
   [BCP195].  For example, any recommendations referring to TLS 1.2,
   including the mandatory support, are not relevant for Secure TACACS+,
   as TLS 1.3 or above is mandated.

   This document concerns the use of TLS as transport for TACACS+ and
   does not make any changes to the core TACACS+ protocol, other than
   the direct implications of deprecating obfuscation.  Operators MUST
   be cognizant of the security implications of the TACACS+ protocol
   itself.  Further documents are planned, for example, to address the
   security implications of password-based authentication and enhance
   the protocol to accommodate alternative schemes.

#### 5.1.1. TLS Use

   New TACACS+ production deployments SHOULD use TLS authentication and
   encryption.  Also see [RFC3365].

   TLS TACACS+ servers (as defined in Section 2) MUST NOT allow non-TLS
   connections, because of the threat of downgrade attacks or
   misconfiguration described in Section 5.3.  Instead, separate non-TLS
   TACACS+ servers SHOULD be set up to cater for these clients.

   It is NOT RECOMMENDED that TLS TACACS+ servers and non-TLS TACACS+
   servers be deployed on the same host, for reasons discussed in Section 5.3.  Non-TLS connections would be better served by deploying
   the required non-TLS TACACS+ servers on separate hosts.

   TACACS+ clients MUST NOT fail back to a non-TLS connection if a TLS
   connection fails.  This prohibition includes during the migration of
   a deployment (Section 6.1).

#### 5.1.2. TLS 0-RTT

   TLS 1.3 resumption and PSK techniques make it possible to send early
   data, aka 0-RTT data, data that is sent before the TLS handshake
   completes.  Replay of this data is a risk.  Given the sensitivity of
   TACACS+ data, clients MUST NOT send data until the full TLS handshake
   completes; that is, clients MUST NOT send 0-RTT data and TLS TACACS+
   servers MUST abruptly disconnect clients that do.

   TLS TACACS+ clients and servers MUST NOT include the "early_data"
   extension.  See Sections 2.3 and 4.2.10 of [RFC8446] for security
   concerns.

#### 5.1.3. TLS Options

   Recommendations in [BCP195] MUST be followed to determine which TLS
   versions and algorithms should be supported, deprecated, obsoleted,
   or abandoned.

   Also, Section 9 of [RFC8446] prescribes mandatory supported options.

#### 5.1.4. Unreachable Certificate Authority (CA)

   Operators should be cognizant of the potential of TLS TACACS+ server
   and/or client isolation from their peer's CA by network failures.
   Isolation from a public key certificate's CA will cause the
   verification of the certificate to fail and thus TLS authentication
   of the peer to fail.  The approach mentioned in Section 3.4.1 can
   help address this and should be considered.

#### 5.1.5. TLS Server Name Indicator (SNI)

   Operators should be aware that the TLS SNI extension is part of the
   TLS client hello, which is sent in cleartext.  It is, therefore,
   subject to eavesdropping.  Also see Section 11.1 of [RFC6066].

#### 5.1.6. Server Identity Wildcards

   The use of wildcards in TLS server identities creates a single point
   of failure: a compromised private key of a wildcard certificate
   impacts all servers using it.  Their use MUST follow the
   recommendations of Section 7.1 of [RFC9525].  Operators MUST ensure
   that the wildcard is limited to a subdomain dedicated solely to TLS
   TACACS+ servers.  Further, operators MUST ensure that the TLS TACACS+
   servers covered by a wildcard certificate are impervious to
   redirection of traffic to a different server (for example, due to on-
   path attacks or DNS cache poisoning).

### 5.2. TACACS+ Configuration

   Implementors must ensure that the configuration scheme introduced for
   enabling TLS is straightforward and leaves no room for ambiguity
   regarding whether TLS or non-TLS will be used between the TACACS+
   client and the TACACS+ server.

   This document recommends the use of a separate port number that TLS
   TACACS+ servers will listen to.  Where deployments have not
   overridden the defaults explicitly, TACACS+ client implementations
   MUST use the correct port values:

   *  49: for non-TLS connection TACACS+

   *  300: for TLS connection TACACS+

   Implementors may offer a single option for TACACS+ clients and
   servers to disable all non-TLS TACACS+ operations.  When enabled on a
   TACACS+ server, it will not respond to any requests from non-TLS
   TACACS+ client connections.  When enabled on a TACACS+ client, it
   will not establish any non-TLS TACACS+ server connections.

### 5.3. Well-Known TCP/IP Port Number

   A new port number is considered appropriate (rather than a mechanism
   that negotiates an upgrade from an initial non-TLS TACACS+
   connection) because it allows:

   *  ease of blocking the unobfuscated or obfuscated connections by the
      TCP/IP port number,

   *  passive Intrusion Detection Systems (IDSs) monitoring the
      unobfuscated to be unaffected by the introduction of TLS,

   *  avoidance of on-path attacks that can interfere with upgrade, and

   *  prevention of the accidental exposure of sensitive information due
      to misconfiguration.

   However, the coexistence of inferior authentication and obfuscation,
   whether a non-TLS connection or deprecated parts that compose TLS,
   also presents an opportunity for downgrade attacks.  Causing failure
   of connections to the TLS-enabled service or the negotiation of
   shared algorithm support are two such downgrade attacks.

   The simplest mitigation exposure from non-TLS connection methods is
   to refuse non-TLS connections at the host entirely, perhaps using
   separate hosts for non-TLS connections and TLS.

   Another approach is mutual configuration that requires TLS.  TACACS+
   clients and servers SHOULD support configuration that requires peers,
   globally and individually, to use TLS.  Furthermore, peers SHOULD be
   configurable to limit offered or recognized TLS versions and
   algorithms to those recommended by standards bodies and implementers.

## 6. Operational Considerations

   Operational and deployment considerations are spread throughout the
   document.  While avoiding repetition, it is useful for the impatient
   to direct particular attention to Sections 5.2 and 5.1.5.  However,
   it is important that the entire Section 5 is observed.

   It is essential for operators to understand the implications of a TLS
   certificate-based authentication solution, including the correct
   handling of certificates, CAs, and all elements of TLS configuration.
   Refer to [BCP195] for guidance.  Attention is drawn to the
   provisioning of certificates to all peers, including TACACS+ TLS
   clients, to permit the mandatory mutual authentication.

### 6.1. Migration



   Section 5.2 mentions that for an optimal deployment of TLS TACACS+,
   TLS should be universally applied throughout the deployment.
   However, during the migration process from a non-TLS TACACS+
   deployment, operators may need to support both TLS and non-TLS
   TACACS+ servers.  This migration phase allows operators to gradually
   transition their deployments from an insecure state to a more secure
   one, but it is important to note that it is vulnerable to downgrade
   attacks.  Therefore, the migration phase should be considered
   insecure until it is fully completed.  To mitigate this hazard:

   *  The period where any client is configured with both TLS and non-
      TLS TACACS+ servers should be minimized.

   *  The operator must consider the security impact of supporting both
      TLS and non-TLS connections, as mentioned above.

### 6.2. Maintaining Non-TLS TACACS+ Clients

   Some TACACS+ client devices in a deployment may not implement TLS.
   These devices will require access to non-TLS TACACS+ servers.
   Operators must follow the recommendation of Section 5.1.1 and deploy
   separate non-TLS TACACS+ servers for these non-TLS clients from those
   used for the TLS clients.

### 6.3. YANG Model for TACACS+ Clients

   [TACACS-YANG ] specifies a YANG model for managing TACACS+ clients,
   including TLS support.

## 7. IANA Considerations

   IANA has allocated the following new well-known system in the
   "Service Name and Transport Protocol Port Number Registry" (see
   <https://www.iana.org/assignments/service-names-port-numbers >).  The
   service name "tacacss" follows the common practice of appending an
   "s" to the name given to the non-TLS well-known port name.  See the
   justification for the allocation in Section 5.3.

   Service Name:  tacacss
   Port Number:  300
   Transport Protocol:  TCP
   Description:  TLS Secure Login Host Protocol (TACACSS)
   Assignee:  IESG
   Contact:  IETF Chair
   Reference:  RFC 9887   Considerations about service discovery are out of scope of this
   document.

## 8. Acknowledgments

   The author(s) would like to thank Russ Housley, Steven M. Bellovin,
   Stephen Farrell, Alan DeKok, Warren Kumari, Tom Petch, Tirumal Reddy,
   Valery Smyslov, and Mohamed Boucadair for their support, insightful
   review, and/or comments.  [RFC5425] was also used as a basis for the
   general approach to TLS.  [RFC9190] was used as a basis for TLS
   resumption recommendations.  Although still in draft form at the time
   of writing, [RFC9813] was used as a model for PSK recommendations.


---
# Referenced Sections from RFC 8907: The Terminal Access Controller Access-Control System Plus (TACACS+) Protocol

The following sections were referenced. Remaining sections are not included.

## 3. Technical Definitions

   This section provides a few basic definitions that are applicable to
   this document.

### 3.1. Client

   The client is any device that initiates TACACS+ protocol requests to
   mediate access, mainly for the Device Administration use case.

### 3.2. Server

   The server receives TACACS+ protocol requests and replies according
   to its business model in accordance with the flows defined in this
   document.

### 3.3. Packet

   All uses of the word packet in this document refer to TACACS+
   protocol data units unless explicitly noted otherwise.  The informal
   term "packet" has become an established part of the definition.

### 3.4. Connection

   TACACS+ uses TCP for its transport.  TCP Server port 49 is allocated
   by IANA for TACACS+ traffic.

### 3.5. Session

   The concept of a session is used throughout this document.  A TACACS+
   session is a single authentication sequence, a single authorization
   exchange, or a single accounting exchange.

   An accounting and authorization session will consist of a single pair
   of packets (the request and its reply).  An authentication session
   may involve an arbitrary number of packets being exchanged.  The
   session is an operational concept that is maintained between the
   TACACS+ client and server.  It does not necessarily correspond to a
   given user or user action.

### 3.6. Treatment of Enumerated Protocol Values

   This document describes various enumerated values in the packet
   header and the headers for specific packet types.  For example, in
   the authentication start packet type, this document defines the
   action field with three values: TAC_PLUS_AUTHEN_LOGIN,
   TAC_PLUS_AUTHEN_CHPASS, and TAC_PLUS_AUTHEN_SENDAUTH.

   If the server does not implement one of the defined options in a
   packet that it receives, or it encounters an option that is not
   listed in this document for a header field, then it should respond
   with an ERROR and terminate the session.  This will allow the client
   to try a different option.

   If an error occurs but the type of the incoming packet cannot be
   determined, a packet with the identical cleartext header but with a
   sequence number incremented by one and the length set to zero MUST be
   returned to indicate an error.

### 3.7. Treatment of Text Strings

   The TACACS+ protocol makes extensive use of text strings.  "The
   Draft" intended that these strings would be treated as byte arrays
   where each byte would represent a US-ASCII character.

   More recently, server implementations have been extended to interwork
   with external identity services, and so a more nuanced approach is
   needed.  Usernames MUST be encoded and handled using the
   UsernameCasePreserved Profile specified in [RFC8265].  The security
   considerations in Section 8 of [RFC8265] apply.

   Where specifically mentioned, data fields contain arrays of arbitrary
   bytes as required for protocol processing.  These are not intended to
   be made visible through user interface to users.

   All other text fields in TACACS+ MUST be treated as printable byte
   arrays of US-ASCII as defined by [RFC0020].  The term "printable"
   used here means the fields MUST exclude the "Control Characters"
   defined in Section 5.2 of [RFC0020].

## 4. TACACS+ Packets and Sessions





### 4.1. The TACACS+ Packet Header

   All TACACS+ packets begin with the following 12-byte header.  The
   header describes the remainder of the packet:

    1 2 3 4 5 6 7 8  1 2 3 4 5 6 7 8  1 2 3 4 5 6 7 8  1 2 3 4 5 6 7 8
   +----------------+----------------+----------------+----------------+
   |major  | minor  |                |                |                |
   |version| version|      type      |     seq_no     |   flags        |
   +----------------+----------------+----------------+----------------+
   |                                                                   |
   |                            session_id                             |
   +----------------+----------------+----------------+----------------+
   |                                                                   |
   |                              length                               |
   +----------------+----------------+----------------+----------------+

   The following general rules apply to all TACACS+ packet types:

   *  To signal that any variable-length data fields are unused, the
      corresponding length values are set to zero.  Such fields MUST be
      ignored, and treated as if not present.

   *  The lengths of data and message fields in a packet are specified
      by their corresponding length field (and are not null terminated).

   *  All length values are unsigned and in network byte order.

   major_version

      This is the major TACACS+ version number.

      TAC_PLUS_MAJOR_VER := 0xc

   minor_version

      This is the minor TACACS+ version number.

      TAC_PLUS_MINOR_VER_DEFAULT := 0x0

      TAC_PLUS_MINOR_VER_ONE := 0x1

   type

      This is the packet type.

      Options are:

      TAC_PLUS_AUTHEN := 0x01 (Authentication)

      TAC_PLUS_AUTHOR := 0x02 (Authorization)

      TAC_PLUS_ACCT := 0x03 (Accounting)

   seq_no

      This is the sequence number of the current packet.  The first
      packet in a session MUST have the sequence number 1, and each
      subsequent packet will increment the sequence number by one.
      TACACS+ clients only send packets containing odd sequence numbers,
      and TACACS+ servers only send packets containing even sequence
      numbers.

      The sequence number must never wrap, i.e., if the sequence number
      2^(8)-1 is ever reached, that session must terminate and be
      restarted with a sequence number of 1.

   flags

      This field contains various bitmapped flags.

      The flag bit:

      TAC_PLUS_UNENCRYPTED_FLAG := 0x01

      This flag indicates that the sender did not obfuscate the body of
      the packet.  This option MUST NOT be used in production.  The
      application of this flag will be covered in "Security
      Considerations" (Section 10).

      This flag SHOULD be clear in all deployments.  Modern network
      traffic tools support encrypted traffic when configured with the
      shared secret (see "Shared Secrets" (Section 10.5.1)), so
      obfuscated mode can and SHOULD be used even during test.

      The single-connection flag:

      TAC_PLUS_SINGLE_CONNECT_FLAG := 0x04

      This flag is used to allow a client and server to negotiate
      "Single Connection Mode" (Section 4.3).

      All other bits MUST be ignored when reading, and SHOULD be set to
      zero when writing.

   session_id

      The Id for this TACACS+ session.  This field does not change for
      the duration of the TACACS+ session.  This number MUST be
      generated by a cryptographically strong random number generation
      method.  Failure to do so will compromise security of the session.
      For more details, refer to [RFC4086].

   length

      The total length of the packet body (not including the header).
      Implementations MUST allow control over maximum packet sizes
      accepted by TACACS+ Servers.  The recommended maximum packet size
      is 2^(16).

### 4.2. The TACACS+ Packet Body

   The TACACS+ body types are defined in the packet header.  The next
   sections of this document will address the contents of the different
   TACACS+ bodies.

### 4.3. Single Connection Mode

   Single Connection Mode is intended to improve performance where there
   is a lot of traffic between a client and a server by allowing the
   client to multiplex multiple sessions on a single TCP connection.

   The packet header contains the TAC_PLUS_SINGLE_CONNECT_FLAG used by
   the client and server to negotiate the use of Single Connection Mode.

   The client sets this flag to indicate that it supports multiplexing
   TACACS+ sessions over a single TCP connection.  The client MUST NOT
   send a second packet on a connection until single-connect status has
   been established.

   To indicate it will support Single Connection Mode, the server sets
   this flag in the first reply packet in response to the first request
   from a client.  The server may set this flag even if the client does
   not set it, but the client may ignore the flag and close the
   connection after the session completes.

   The flag is only relevant for the first two packets on a connection,
   to allow the client and server to establish Single Connection Mode.
   No provision is made for changing Single Connection Mode after the
   first two packets; the client and server MUST ignore the flag after
   the second packet on a connection.

   If Single Connection Mode has not been established in the first two
   packets of a TCP connection, then both the client and the server
   close the connection at the end of the first session.

   The client negotiates Single Connection Mode to improve efficiency.
   The server may refuse to allow Single Connection Mode for the client.
   For example, it may not be appropriate to allocate a long-lasting TCP
   connection to a specific client in some deployments.  Even if the
   server is configured to permit Single Connection Mode for a specific
   client, the server may close the connection.  For example, a server
   MUST be configured to time out a Single Connection Mode TCP
   connection after a specific period of inactivity to preserve its
   resources.  The client MUST accommodate such closures on a TCP
   session even after Single Connection Mode has been established.

   The TCP connection underlying the Single Connection Mode will close
   eventually either because of the timeout from the server or from an
   intermediate link.  If a session is in progress when the client
   detects disconnect, then the client should handle it as described in
   "Session Completion" (Section 4.4).  If a session is not in progress,
   then the client will need to detect this and restart the Single
   Connection Mode when it initiates the next session.

### 4.4. Session Completion

   The REPLY packets defined for the packet types in the sections below
   (Authentication, Authorization, and Accounting) contain a status
   field.  The complete set of options for this field depend upon the
   packet type, but all three REPLY packet types define values
   representing PASS, ERROR, and FAIL, which indicate the last packet of
   a regular session (one that is not aborted).

   The server responds with a PASS or a FAIL to indicate that the
   processing of the request completed and that the client can apply the
   result (PASS or FAIL) to control the execution of the action that
   prompted the request to be sent to the server.

   The server responds with an ERROR to indicate that the processing of
   the request did not complete.  The client cannot apply the result,
   and it MUST behave as if the server could not be connected to.  For
   example, the client tries alternative methods, if they are available,
   such as sending the request to a backup server or using local
   configuration to determine whether the action that prompted the
   request should be executed.

   Refer to "Aborting an Authentication Session" (Section 5.4.3) for
   details on handling additional status options.

   When the session is complete, the TCP connection should be handled as
   follows, according to whether Single Connection Mode was negotiated:

   *  If Single Connection Mode was not negotiated, then the connection
      should be closed.

   *  If Single Connection Mode was enabled, then the connection SHOULD
      be left open (see "Single Connection Mode" (Section 4.3)) but may
      still be closed after a timeout period to preserve deployment
      resources.

   *  If Single Connection Mode was enabled, but an ERROR occurred due
      to connection issues (such as an incorrect secret (see Section 4.5)), then any further new sessions MUST NOT be accepted
      on the connection.  If there are any sessions that have already
      been established, then they MAY be completed.  Once all active
      sessions are completed, then the connection MUST be closed.

   It is recommended that client implementations provide robust schemes
   for dealing with servers that cannot be connected to.  Options
   include providing a list of servers for redundancy and an option for
   a local fallback configuration if no servers can be reached.  Details
   will be implementation specific.

   The client should manage connections and handle the case of a server
   that establishes a connection but does not respond.  The exact
   behavior is implementation specific.  It is recommended that the
   client close the connection after a configurable timeout.

### 4.5. Data Obfuscation

   The body of packets may be obfuscated.  The following sections
   describe the obfuscation method that is supported in the protocol.
   In "The Draft", this process was actually referred to as Encryption,
   but the algorithm would not meet modern standards and so will not be
   termed as encryption in this document.

   The obfuscation mechanism relies on a secret key, a shared secret
   value that is known to both the client and the server.  The secret
   keys MUST remain secret.

   Server implementations MUST allow a unique secret key to be
   associated with each client.  It is a site-dependent decision as to
   whether or not the use of separate keys is appropriate.

   The flag field MUST be configured with TAC_PLUS_UNENCRYPTED_FLAG set
   to 0 so that the packet body is obfuscated by XORing it bytewise with
   a pseudo-random pad:

      ENCRYPTED {data} = data ^(pseudo_pad)

   The packet body can then be de-obfuscated by XORing it bytewise with
   a pseudo-random pad.

      data = ENCRYPTED {data} ^(pseudo_pad)

   The pad is generated by concatenating a series of MD5 hashes (each 16
   bytes long) and truncating it to the length of the input data.
   Whenever used in this document, MD5 refers to the "RSA Data Security,
   Inc.  MD5 Message-Digest Algorithm" as specified in [RFC1321].

      pseudo_pad = {MD5_1 [,MD5_2 [ ... ,MD5_n]]} truncated to len(data)

   The first MD5 hash is generated by concatenating the session_id, the
   secret key, the version number, and the sequence number, and then
   running MD5 over that stream.  All of those input values are
   available in the packet header, except for the secret key, which is a
   shared secret between the TACACS+ client and server.

   The version number and session_id are extracted from the header.

   Subsequent hashes are generated by using the same input stream but
   concatenating the previous hash value at the end of the input stream.

      MD5_1 = MD5{session_id, key, version, seq_no} MD5_2 =
      MD5{session_id, key, version, seq_no, MD5_1} ....  MD5_n =
      MD5{session_id, key, version, seq_no, MD5_n-1}

   When a server detects that the secrets it has configured for the
   device do not match, it MUST return ERROR.  For details of TCP
   connection handling on ERROR, refer to "Session Completion"
   (Section 4.4).

      TAC_PLUS_UNENCRYPTED_FLAG == 0x1

   This option is deprecated and MUST NOT be used in production.  In
   this case, the entire packet body is in cleartext.  A request MUST be
   dropped if TAC_PLUS_UNENCRYPTED_FLAG is set to true.

   After a packet body is de-obfuscated, the lengths of the component
   values in the packet are summed.  If the sum is not identical to the
   cleartext datalength value from the header, the packet MUST be
   discarded and an ERROR signaled.  For details of TCP connection
   handling on ERROR, refer to "Session Completion" (Section 4.4).

   Commonly, such failures are seen when the keys are mismatched between
   the client and the TACACS+ server.

### 10.1. General Security of the Protocol

   The TACACS+ protocol does not include a security mechanism that would
   meet modern-day requirements.  These security mechanisms would be
   best referred to as "obfuscation" and not "encryption", since they
   provide no meaningful integrity, privacy, or replay protection.  An
   attacker with access to the data stream should be assumed to be able
   to read and modify all TACACS+ packets.  Without mitigation, a range
   of risks such as the following are possible:

   *  Accounting information may be modified by the man-in-the-middle
      attacker, making such logs unsuitable and not trustable for
      auditing purposes.

   *  Invalid or misleading values may be inserted by the man-in-the-
      middle attacker in various fields at known offsets to try and
      circumvent the authentication or authorization checks even inside
      the obfuscated body.

   While the protocol provides some measure of transport privacy, it is
   vulnerable to at least the following attacks:

   *  Brute-force attacks exploiting increased efficiency of MD5 digest
      computation.

   *  Known plaintext attacks that may decrease the cost of brute-force
      attacks.

   *  Chosen plaintext attacks that may decrease the cost of a brute-
      force attacks.

   *  No forward secrecy.

   Even though, to the best knowledge of the authors, this method of
   encryption wasn't rigorously tested, enough information is available
   that it is best referred to as "obfuscation" and not "encryption".

   For these reasons, users deploying the TACACS+ protocol in their
   environments MUST limit access to known clients and MUST control the
   security of the entire transmission path.  Attackers who can guess
   the key or otherwise break the obfuscation will gain unrestricted and
   undetected access to all TACACS+ traffic.  Ensuring that a
   centralized AAA system like TACACS+ is deployed on a secured
   transport is essential to managing the security risk of such an
   attack.

   The following parts of this section enumerate only the session-
   specific risks that are in addition to general risk associated with
   bare obfuscation and lack of integrity checking.

### 10.5. TACACS+ Best Practices

   With respect to the observations about the security issues described
   above, a network administrator MUST NOT rely on the obfuscation of
   the TACACS+ protocol.  TACACS+ MUST be used within a secure
   deployment; TACACS+ MUST be deployed over networks that ensure
   privacy and integrity of the communication and MUST be deployed over
   a network that is separated from other traffic.  Failure to do so
   will impact overall network security.

   The following recommendations impose restrictions on how the protocol
   is applied.  These restrictions were not imposed in "The Draft".  New
   implementations, and upgrades of current implementations, MUST
   implement these recommendations.  Vendors SHOULD provide mechanisms
   to assist the administrator to achieve these best practices.

#### 10.5.1. Shared Secrets

   TACACS+ servers and clients MUST treat shared secrets as sensitive
   data to be managed securely, as would be expected for other sensitive
   data such as identity credential information.  TACACS+ servers MUST
   NOT leak sensitive data.

   For example:

   *  TACACS+ servers MUST NOT expose shared secrets in logs.

   *  TACACS+ servers MUST allow a dedicated secret key to be defined
      for each client.

   *  TACACS+ server management systems MUST provide a mechanism to
      track secret key lifetimes and notify administrators to update
      them periodically.  TACACS+ server administrators SHOULD change
      secret keys at regular intervals.

   *  TACACS+ servers SHOULD warn administrators if secret keys are not
      unique per client.

   *  TACACS+ server administrators SHOULD always define a secret for
      each client.

   *  TACACS+ servers and clients MUST support shared keys that are at
      least 32 characters long.

   *  TACACS+ servers MUST support policy to define minimum complexity
      for shared keys.

   *  TACACS+ clients SHOULD NOT allow servers to be configured without
      a shared secret key or shared key that is less than 16 characters
      long.

   *  TACACS+ server administrators SHOULD configure secret keys of a
      minimum of 16 characters in length.

#### 10.5.2. Connections and Obfuscation

   TACACS+ servers MUST allow the definition of individual clients.  The
   servers MUST only accept network connection attempts from these
   defined known clients.

   TACACS+ servers MUST reject connections that have
   TAC_PLUS_UNENCRYPTED_FLAG set.  There MUST always be a shared secret
   set on the server for the client requesting the connection.

   If an invalid shared secret is detected when processing packets for a
   client, TACACS+ servers MUST NOT accept any new sessions on that
   connection.  TACACS+ servers MUST terminate the connection on
   completion of any sessions that were previously established with a
   valid shared secret on that connection.

   TACACS+ clients MUST NOT set TAC_PLUS_UNENCRYPTED_FLAG.  Clients MUST
   be implemented in a way that requires explicit configuration to
   enable the use of TAC_PLUS_UNENCRYPTED_FLAG.  This option MUST NOT be
   used when the client is in production.

   When a TACACS+ client receives responses from servers where:

   *  the response packet was received from the server configured with a
      shared key, but the packet has TAC_PLUS_UNENCRYPTED_FLAG set, and

   *  the response packet was received from the server configured not to
      use obfuscation, but the packet has TAC_PLUS_UNENCRYPTED_FLAG not
      set,

   the TACACS+ client MUST close the TCP session, and process the
   response in the same way that a TAC_PLUS_AUTHEN_STATUS_FAIL
   (authentication sessions) or TAC_PLUS_AUTHOR_STATUS_FAIL
   (authorization sessions) was received.

#### 10.5.3. Authentication

   To help TACACS+ administrators select stronger authentication
   options, TACACS+ servers MUST allow the administrator to configure
   the server to only accept challenge/response options for
   authentication (TAC_PLUS_AUTHEN_TYPE_CHAP or
   TAC_PLUS_AUTHEN_TYPE_MSCHAP or TAC_PLUS_AUTHEN_TYPE_MSCHAPV2 for
   authen_type).

   TACACS+ server administrators SHOULD enable the option mentioned in
   the previous paragraph.  TACACS+ server deployments SHOULD only
   enable other options (such as TAC_PLUS_AUTHEN_TYPE_ASCII or
   TAC_PLUS_AUTHEN_TYPE_PAP) when unavoidable due to requirements of
   identity/password systems.

   TACACS+ server administrators SHOULD NOT allow the same credentials
   to be applied in challenge-based (TAC_PLUS_AUTHEN_TYPE_CHAP or
   TAC_PLUS_AUTHEN_TYPE_MSCHAP or TAC_PLUS_AUTHEN_TYPE_MSCHAPV2) and
   non-challenge-based authen_type options, as the insecurity of the
   latter will compromise the security of the former.

   TAC_PLUS_AUTHEN_SENDAUTH and TAC_PLUS_AUTHEN_SENDPASS options
   mentioned in "The Draft" SHOULD NOT be used due to their security
   implications.  TACACS+ servers SHOULD NOT implement them.  If they
   must be implemented, the servers MUST default to the options being
   disabled and MUST warn the administrator that these options are not
   secure.

#### 10.5.4. Authorization

   The authorization and accounting features are intended to provide
   extensibility and flexibility.  There is a base dictionary defined in
   this document, but it may be extended in deployments by using new
   argument names.  The cost of the flexibility is that administrators
   and implementers MUST ensure that the argument and value pairs shared
   between the clients and servers have consistent interpretation.

   TACACS+ clients that receive an unrecognized mandatory argument MUST
   evaluate server response as if they received
   TAC_PLUS_AUTHOR_STATUS_FAIL.

#### 10.5.5. Redirection Mechanism

   "The Draft" described a redirection mechanism
   (TAC_PLUS_AUTHEN_STATUS_FOLLOW).  This feature is difficult to
   secure.  The option to send secret keys in the server list is
   particularly insecure, as it can reveal client shared secrets.

   TACACS+ servers MUST deprecate the redirection mechanism.

   If the redirection mechanism is implemented, then TACACS+ servers
   MUST disable it by default and MUST warn TACACS+ server
   administrators that it must only be enabled within a secure
   deployment due to the risks of revealing shared secrets.

   TACACS+ clients SHOULD deprecate this feature by treating
   TAC_PLUS_AUTHEN_STATUS_FOLLOW as TAC_PLUS_AUTHEN_STATUS_FAIL.

# Referenced Sections from RFC 2119: Key words for use in RFCs to Indicate Requirement Levels

The following sections were referenced. Remaining sections are not included.

## 9. Author's Address

      Scott Bradner
      Harvard University
      1350 Mass. Ave.
      Cambridge, MA 02138

      phone - +1 617 495 3864

      email - sob@harvard.edu










































# Referenced Sections from RFC 8174: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words

The following sections were referenced. Remaining sections are not included.

## 2. Clarifying Capitalization of Key Words

   The following change is made to [RFC2119]:

   === OLD ===
   In many standards track documents several words are used to signify
   the requirements in the specification.  These words are often
   capitalized.  This document defines these words as they should be
   interpreted in IETF documents.  Authors who follow these guidelines
   should incorporate this phrase near the beginning of their document:

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in RFC 2119.


   === NEW ===
   In many IETF documents, several words, when they are in all capitals
   as shown below, are used to signify the requirements in the
   specification.  These capitalized words can bring significant clarity
   and consistency to documents because their meanings are well defined.
   This document defines how those words are interpreted in IETF
   documents when the words are in all capitals.

   o  These words can be used as defined here, but using them is not
      required.  Specifically, normative text does not require the use
      of these key words.  They are used for clarity and consistency
      when that is what's wanted, but a lot of normative text does not
      use them and is still normative.

   o  The words have the meanings specified herein only when they are in
      all capitals.

   o  When these words are not capitalized, they have their normal
      English meanings and are not affected by this document.

   Authors who follow these guidelines should incorporate this phrase
   near the beginning of their document:

      The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
      NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
      "MAY", and "OPTIONAL" in this document are to be interpreted as
      described in BCP 14 [RFC2119] [RFC8174] when, and only when, they
      appear in all capitals, as shown here.

   === END ===





