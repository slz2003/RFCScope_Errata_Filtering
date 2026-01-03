Let us analyze section 3 of RFC 9887. All references made by section 3 have also been included below.

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

### 3.4. Connection

   TACACS+ uses TCP for its transport.  TCP Server port 49 is allocated
   by IANA for TACACS+ traffic.

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

### 10.4. Security of Accounting Sessions

   Accounting sessions SHOULD be used via a secure transport (see
   "TACACS+ Best Practices" (Section 10.5)).  Although Accounting
   sessions are not directly involved in authentication or authorizing
   operations on the device, man-in-the-middle attackers may do any of
   the following:

   *  Replace accounting data with new valid values or garbage that can
      confuse auditors or hide information related to their
      authentication and/or authorization attack attempts.

   *  Try and poison an accounting log with entries designed to make
      systems behave in unintended ways (these systems could be TACACS+
      servers and any other systems that would manage accounting
      entries).

   In addition to these direct manipulations, different client
   implementations pass a different fidelity of accounting data.  Some
   vendors have been observed in the wild that pass sensitive data like
   passwords, encryption keys, and the like as part of the accounting
   log.  Due to a lack of strong encryption with perfect forward
   secrecy, this data may be revealed in the future, leading to a
   security incident.

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

# Referenced Sections from RFC 6151: Updated Security Considerations for the MD5 Message-Digest and the HMAC-MD5 Algorithms

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   MD5 [MD5] is a message digest algorithm that takes as input a message
   of arbitrary length and produces as output a 128-bit "fingerprint" or
   "message digest" of the input.  The published attacks against MD5
   show that it is not prudent to use MD5 when collision resistance is
   required.  This document replaces the security considerations in RFC 
   1321 [MD5].

   [HMAC ] defined a mechanism for message authentication using
   cryptographic hash functions.  Any message digest algorithm can be
   used, but the cryptographic strength of HMAC depends on the
   properties of the underlying hash function.  [HMAC-MD5] defined test
   cases for HMAC-MD5.  This document updates the security
   considerations in [HMAC ], which [HMAC-MD5] points to for its security
   considerations.

   [HASH-Attack ] summarizes the use of hashes in many protocols and
   discusses how attacks against a message digest algorithm's one-way
   and collision-free properties affect and do not affect Internet
   protocols.  Familiarity with [HASH-Attack ] is assumed.  One of the
   uses of message digest algorithms in [HASH-Attack ] was integrity
   protection.  Where the MD5 checksum is used inline with the protocol
   solely to protect against errors, an MD5 checksum is still an
   acceptable use.  Applications and protocols need to clearly state in
   their security considerations what security services, if any, are
   expected from the MD5 checksum.  In fact, any application and
   protocol that employs MD5 for any purpose needs to clearly state the
   expected security services from their use of MD5.

## 2. Security Considerations

   MD5 was published in 1992 as an Informational RFC.  Since that time,
   MD5 has been extensively studied and new cryptographic attacks have
   been discovered.  Message digest algorithms are designed to provide
   collision, pre-image, and second pre-image resistance.  In addition,
   message digest algorithms are used with a shared secret value for
   message authentication in HMAC, and in this context, some people may
   find the guidance for key lengths and algorithm strengths in
   [SP800-57] and [SP800-131] useful.

   MD5 is no longer acceptable where collision resistance is required
   such as digital signatures.  It is not urgent to stop using MD5 in
   other ways, such as HMAC-MD5; however, since MD5 must not be used for
   digital signatures, new protocol designs should not employ HMAC-MD5.
   Alternatives to HMAC-MD5 include HMAC-SHA256 [HMAC ] [HMAC-SHA256] and
   [AES-CMAC ] when AES is more readily available than a hash function. 





### 2.1. Collision Resistance

   Pseudo-collisions for the compress function of MD5 were first
   described in 1993 [denBBO1993].  In 1996, [DOB1995] demonstrated a
   collision pair for the MD5 compression function with a chosen initial
   value.  The first paper that demonstrated two collision pairs for MD5
   was published in 2004 [WFLY2004].  The detailed attack techniques for
   MD5 were published at EUROCRYPT 2005 [WAYU2005].  Since then, a lot
   of research results have been published to improve collision attacks
   on MD5. The attacks presented in [KLIM2006] can find MD5 collision in
   about one minute on a standard notebook PC (Intel Pentium, 1.6GHz).
   [STEV2007] claims that it takes 10 seconds or less on a 2.6Ghz
   Pentium4 to find collisions.  In [STEV2007], [SLdeW2007],
   [SSALMOdeW2009], and [SLdeW2009], the collision attacks on MD5 were
   successfully applied to X.509 certificates.

   Notice that the collision attack on MD5 can also be applied to
   password-based challenge-and-response authentication protocols such
   as the APOP (Authenticated Post Office Protocol) option in POP [POP ]
   used in post office authentication as presented in [LEUR2007].

   In fact, more delicate attacks on MD5 to improve the speed of finding
   collisions have been published recently.  However, the aforementioned
   results have provided sufficient reason to eliminate MD5 usage in
   applications where collision resistance is required such as digital
   signatures.

### 2.2. Pre-Image and Second Pre-Image Resistance

   Even though the best result can find a pre-image attack of MD5 faster
   than exhaustive search, as presented in [SAAO2009], the complexity
   2^123.4 is still pretty high.

### 2.3. HMAC

   The cryptanalysis of HMAC-MD5 is usually conducted together with NMAC
   (Nested MAC) since they are closely related.  NMAC uses two
   independent keys K1 and K2 such that NMAC(K1, K2, M) = H(K1, H(K2,
   M), where K1 and K2 are used as secret initialization vectors (IVs)
   for hash function H(IV, M).  If we re-write the HMAC equation using
   two secret IVs such that IV2 = H(K Xor ipad) and IV1 = H(K Xor opad),
   then HMAC(K, M) = NMAC(IV1, IV2, M).  Here it is very important to
   notice that IV1 and IV2 are not independently selected.

   The first analysis was explored on NMAC-MD5 using related keys in
   [COYI2006].  The partial key recovery attack cannot be extended to
   HMAC-MD5, since for HMAC, recovering partial secret IVs can hardly
   lead to recovering (partial) key K.  Another paper presented at 
   Crypto 2007 [FLN2007] extended results of [COYI2006] to a full key
   recovery attack on NMAC-MD5.  Since it also uses related key attack,
   it does not seem applicable to HMAC-MD5.

   A EUROCRYPT 2009 paper presented a distinguishing attack on HMAC-MD5
   [WYWZZ2009] without using related keys.  It can distinguish an
   instantiation of HMAC with MD5 from an instantiation with a random
   function with 2^97 queries with probability 0.87.  This is called
   distinguishing-H.  Using the distinguishing attack, it can recover
   some bits of the intermediate status of the second block.  However,
   as it is pointed out in [WYWZZ2009], it cannot be used to recover the
   (partial) inner key H(K Xor ipad).  It is not obvious how the attack
   can be used to form a forgery attack either.

   The attacks on HMAC-MD5 do not seem to indicate a practical
   vulnerability when used as a message authentication code.
   Considering that the distinguishing-H attack is different from a
   distinguishing-R attack, which distinguishes an HMAC from a random
   function, the practical impact on HMAC usage as a pseudorandom
   function (PRF) such as in a key derivation function is not well
   understood.

   Therefore, it may not be urgent to remove HMAC-MD5 from the existing
   protocols.  However, since MD5 must not be used for digital
   signatures, for a new protocol design, a ciphersuite with HMAC-MD5
   should not be included.  Options include HMAC-SHA256 [HMAC ]
   [HMAC-SHA256] and [AES-CMAC ] when AES is more readily available than
   a hash function.

# Referenced Sections from RFC draft-ietf-uta-require-tls13-12: Updates: 9325 (if approved)                                    N. Aviram

The following sections were referenced. Remaining sections are not included.

## 8. References





### 8.1. Normative References






   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997,
              <https://www.rfc-editor.org/rfc/rfc2119>.

   [RFC8174]  Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 
              2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174,
              May 2017, <https://www.rfc-editor.org/rfc/rfc8174>.

   [RFC9325]  Sheffer, Y., Saint-Andre, P., and T. Fossati,
              "Recommendations for Secure Use of Transport Layer
              Security (TLS) and Datagram Transport Layer Security
              (DTLS)", BCP 195, RFC 9325, DOI 10.17487/RFC9325, November
              2022, <https://www.rfc-editor.org/rfc/rfc9325>.

   [TLS12]    Dierks, T. and E. Rescorla, "The Transport Layer Security
              (TLS) Protocol Version 1.2", RFC 5246,
              DOI 10.17487/RFC5246, August 2008,
              <https://www.rfc-editor.org/rfc/rfc5246>.

   [TLS12FROZEN ]
              Salz, R. and N. Aviram, "TLS 1.2 is in Feature Freeze",
              Work in Progress, Internet-Draft, draft-ietf-tls-tls12-
              frozen-08, 3 April 2025,
              <https://datatracker.ietf.org/doc/html/draft-ietf-tls-
              tls12-frozen-08>.

   [TLS13]    Rescorla, E., "The Transport Layer Security (TLS) Protocol
              Version 1.3", Work in Progress, Internet-Draft, draft-
              ietf-tls-rfc8446bis-12, 17 February 2025,
              <https://datatracker.ietf.org/doc/html/draft-ietf-tls-
              rfc8446bis-12>.

### 8.2. Informative References

   [BEAST ]    Duong, T. and J. Rizzo, "Here come the xor ninjas", n.d.,
              <http://www.hpcc.ecs.soton.ac.uk/dan/talks/bullrun/
              Beast.pdf >.

   [CBCSCANNING ]
              Merget, R., Somorovsky, J., Aviram, N., Young, C.,
              Fliegenschmidt, J., Schwenk, J., and Y. Shavitt, "Scalable
              Scanning and Automatic Classification of TLS Padding
              Oracle Vulnerabilities", n.d.,
              <https://www.usenix.org/system/files/sec19-merget.pdf >. 
   [DNSTLS ]   Dickinson, S., Gillmor, D., and T. Reddy, "Usage Profiles
              for DNS over TLS and DNS over DTLS", RFC 8310,
              DOI 10.17487/RFC8310, March 2018,
              <https://www.rfc-editor.org/rfc/rfc8310>.

   [FREAK ]    Beurdouche, B., Bhargavan, K., Delignat-Lavaud, A.,
              Fournet, C., Kohlweiss, M., Pironti, A., Strub, P.-Y., and
              J. K. Zinzindohoue, "A messy state of the union: Taming
              the composite state machines of TLS", n.d.,
              <https://inria.hal.science/hal-01114250/file/messy-state-
              of-the-union-oakland15.pdf >.

   [I-D.ietf-pquip-pqc-engineers ]
              Banerjee, A., Reddy.K, T., Schoinianakis, D., Hollebeek,
              T., and M. Ounsworth, "Post-Quantum Cryptography for
              Engineers", Work in Progress, Internet-Draft, draft-ietf-
              pquip-pqc-engineers-09, 13 February 2025,
              <https://datatracker.ietf.org/doc/html/draft-ietf-pquip-
              pqc-engineers-09>.

   [I-D.ietf-tls-deprecate-obsolete-kex ]
              Bartle, C. and N. Aviram, "Deprecating Obsolete Key
              Exchange Methods in TLS 1.2", Work in Progress, Internet-
              Draft, draft-ietf-tls-deprecate-obsolete-kex-05, 3
              September 2024, <https://datatracker.ietf.org/doc/html/
              draft-ietf-tls-deprecate-obsolete-kex-05>.

   [LUCKY13]  Al Fardan, N. J. and K. G. Paterson, "Lucky Thirteen:
              Breaking the TLS and DTLS record protocols", n.d.,
              <http://www.isg.rhul.ac.uk/tls/TLStiming.pdf >.

   [LUCKY13FIX ]
              Somorovsky, J., "Systematic fuzzing and testing of TLS
              libraries", n.d., <https://nds.rub.de/media/nds/
              veroeffentlichungen/2016/10/19/tls-attacker-ccs16.pdf >.

   [PQC ]      "What Is Post-Quantum Cryptography?", August 2024,
              <https://www.nist.gov/cybersecurity/what-post-quantum-
              cryptography >.

   [QUICTLS ]  Thomson, M., Ed. and S. Turner, Ed., "Using TLS to Secure
              QUIC", RFC 9001, DOI 10.17487/RFC9001, May 2021,
              <https://www.rfc-editor.org/rfc/rfc9001>. 
   [RENEG1]   Rescorla, E., "Understanding the TLS Renegotiation
              Attack", n.d.,
              <https://web.archive.org/web/20091231034700/
              http://www.educatedguesswork.org/2009/11/
              understanding_the_tls_renegoti .html>.

   [RENEG2]   Ray, M., "Authentication Gap in TLS Renegotiation", n.d.,
              <https://web.archive.org/web/20091228061844/
              http://extendedsubset.com/?p=8>.

   [RFC5746]  Rescorla, E., Ray, M., Dispensa, S., and N. Oskov,
              "Transport Layer Security (TLS) Renegotiation Indication
              Extension", RFC 5746, DOI 10.17487/RFC5746, February 2010,
              <https://www.rfc-editor.org/rfc/rfc5746>.

   [RFC7465]  Popov, A., "Prohibiting RC4 Cipher Suites", RFC 7465,
              DOI 10.17487/RFC7465, February 2015,
              <https://www.rfc-editor.org/rfc/rfc7465>.

   [SLOTH ]    Bhargavan, K. and G. Leurent, "Transcript collision
              attacks: Breaking authentication in TLS, IKE, and SSH",
              n.d., <https://inria.hal.science/hal-01244855/file/
              SLOTH_NDSS16.pdf >.

   [TRIPLESHAKE ]
              "Triple Handshakes Considered Harmful Breaking and Fixing
              Authentication over TLS", n.d.,
              <https://mitls.org/pages/attacks/3SHAKE >.

   [WEAKDH ]   Adrian, D., Bhargavan, K., Durumeric, Z., Gaudry, P.,
              Green, M., Halderman, J. A., Heninger, N., Springall, D.,
              ThomÃ©, E., Valenta, L., and B. VanderSloot, "Imperfect
              forward secrecy: How Diffie-Hellman fails in practice",
              n.d.,
              <https://dl.acm.org/doi/pdf/10.1145/2810103.2813707>.


# Referenced Sections from RFC 9325: Recommendations for Secure Use of Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS)

The following sections were referenced. Remaining sections are not included.

## 4. Recommendations: Cipher Suites

   TLS 1.2 provided considerable flexibility in the selection of cipher
   suites.  Unfortunately, the security of some of these cipher suites
   has degraded over time to the point where some are known to be
   insecure (this is one reason why TLS 1.3 restricted such
   flexibility).  Incorrectly configuring a server leads to no or
   reduced security.  This section includes recommendations on the
   selection and negotiation of cipher suites.

### 4.1. General Guidelines

   Cryptographic algorithms weaken over time as cryptanalysis improves:
   algorithms that were once considered strong become weak.
   Consequently, cipher suites using weak algorithms need to be phased
   out and replaced with more secure cipher suites.  This helps to
   ensure that the desired security properties still hold.  SSL/TLS has
   been in existence for well over 20 years and many of the cipher
   suites that have been recommended in various versions of SSL/TLS are
   now considered weak or at least not as strong as desired.  Therefore,
   this section modernizes the recommendations concerning cipher suite
   selection.

   *  Implementations MUST NOT negotiate the cipher suites with NULL
      encryption.

      Rationale: The NULL cipher suites do not encrypt traffic and so
      provide no confidentiality services.  Any entity in the network
      with access to the connection can view the plaintext of contents
      being exchanged by the client and server.  Nevertheless, this
      document does not discourage software from implementing NULL
      cipher suites, since they can be useful for testing and debugging.

   *  Implementations MUST NOT negotiate RC4 cipher suites.

      Rationale: The RC4 stream cipher has a variety of cryptographic
      weaknesses, as documented in [RFC7465].  Note that DTLS
      specifically forbids the use of RC4 already.

   *  Implementations MUST NOT negotiate cipher suites offering less
      than 112 bits of security, including so-called "export-level"
      encryption (which provides 40 or 56 bits of security).

      Rationale: Based on [RFC3766], at least 112 bits of security is
      needed.  40-bit and 56-bit security (found in so-called "export
      ciphers") are considered insecure today.

   *  Implementations SHOULD NOT negotiate cipher suites that use
      algorithms offering less than 128 bits of security.

      Rationale: Cipher suites that offer 112 or more bits but less than
      128 bits of security are not considered weak at this time;
      however, it is expected that their useful lifespan is short enough
      to justify supporting stronger cipher suites at this time.
      128-bit ciphers are expected to remain secure for at least several
      years and 256-bit ciphers until the next fundamental technology
      breakthrough.  Note that, because of so-called "meet-in-the-
      middle" attacks [Multiple-Encryption ], some legacy cipher suites
      (e.g., 168-bit Triple DES (3DES)) have an effective key length
      that is smaller than their nominal key length (112 bits in the
      case of 3DES).  Such cipher suites should be evaluated according
      to their effective key length.

   *  Implementations SHOULD NOT negotiate cipher suites based on RSA
      key transport, a.k.a. "static RSA".

      Rationale: These cipher suites, which have assigned values
      starting with the string "TLS_RSA_WITH_*", have several drawbacks,
      especially the fact that they do not support forward secrecy.

   *  Implementations SHOULD NOT negotiate cipher suites based on non-
      ephemeral (static) finite-field Diffie-Hellman (DH) key agreement.
      Similarly, implementations SHOULD NOT negotiate non-ephemeral
      Elliptic Curve DH key agreement.

      Rationale: The former cipher suites, which have assigned values
      prefixed by "TLS_DH_*", have several drawbacks, especially the
      fact that they do not support forward secrecy.  The latter
      ("TLS_ECDH_*") also lack forward secrecy and are subject to
      invalid curve attacks [Jager2015].

   *  Implementations MUST support and prefer to negotiate cipher suites
      offering forward secrecy.  However, TLS 1.2 implementations SHOULD
      NOT negotiate cipher suites based on ephemeral finite-field
      Diffie-Hellman key agreement (i.e., "TLS_DHE_*" suites).  This is
      justified by the known fragility of the construction (see
      [RACCOON ]) and the limitation around negotiation, including using
      [RFC7919], which has seen very limited uptake.

      Rationale: Forward secrecy (sometimes called "perfect forward
      secrecy") prevents the recovery of information that was encrypted
      with older session keys, thus limiting how far back in time data
      can be decrypted when an attack is successful.  See Sections 7.3      and 7.4 for a detailed discussion.

### 4.2. Cipher Suites for TLS 1.2

   Given the foregoing considerations, implementation and deployment of
   the following cipher suites is RECOMMENDED:

   *  TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

   *  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

   *  TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256

   *  TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384

   As these are Authenticated Encryption with Associated Data (AEAD)
   algorithms [RFC5116], these cipher suites are supported only in TLS
   1.2 and not in earlier protocol versions.

   Typically, to prefer these suites, the order of suites needs to be
   explicitly configured in server software.  It would be ideal if
   server software implementations were to prefer these suites by
   default.

   Some devices have hardware support for AES Counter Mode with CBC-MAC
   (AES-CCM) but not AES Galois/Counter Mode (AES-GCM), so they are
   unable to follow the foregoing recommendations regarding cipher
   suites.  There are even devices that do not support public key
   cryptography at all, but these are out of scope entirely.

   A cipher suite that operates in CBC (cipher block chaining) mode
   (e.g., TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256) SHOULD NOT be used
   unless the encrypt_then_mac extension [RFC7366] is also successfully
   negotiated.  This requirement applies to both client and server
   implementations.

   When using ECDSA signatures for authentication of TLS peers, it is
   RECOMMENDED that implementations use the NIST curve P-256.  In
   addition, to avoid predictable or repeated nonces (which could reveal
   the long-term signing key), it is RECOMMENDED that implementations
   implement "deterministic ECDSA" as specified in [RFC6979] and in line
   with the recommendations in [RFC8446].

   Note that implementations of "deterministic ECDSA" may be vulnerable
   to certain side-channel and fault injection attacks precisely because
   of their determinism.  While most fault injection attacks described
   in the literature assume physical access to the device (and therefore
   are more relevant in Internet of Things (IoT) deployments with poor
   or non-existent physical security), some can be carried out remotely
   [Poddebniak2017], e.g., as Rowhammer [Kim2014] variants.  In
   deployments where side-channel attacks and fault injection attacks
   are a concern, implementation strategies combining both randomness
   and determinism (for example, as described in [CFRG-DET-SIGS ]) can be
   used to avoid the risk of successful extraction of the signing key.

#### 4.2.1. Implementation Details

   Clients SHOULD include TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 as the
   first proposal to any server.  Servers MUST prefer this cipher suite
   over weaker cipher suites whenever it is proposed, even if it is not
   the first proposal.  Clients are of course free to offer stronger
   cipher suites, e.g., using AES-256; when they do, the server SHOULD
   prefer the stronger cipher suite unless there are compelling reasons
   (e.g., seriously degraded performance) to choose otherwise.

   The previous version of the TLS recommendations [RFC7525] implicitly
   allowed the old RFC 5246 mandatory-to-implement cipher suite,
   TLS_RSA_WITH_AES_128_CBC_SHA.  At the time of writing, this cipher
   suite does not provide additional interoperability, except with very
   old clients.  As with other cipher suites that do not provide forward
   secrecy, implementations SHOULD NOT support this cipher suite.  Other
   application protocols specify other cipher suites as mandatory to
   implement (MTI).

   [RFC8422] allows clients and servers to negotiate ECDH parameters
   (curves).  Both clients and servers SHOULD include the "Supported
   Elliptic Curves Extension" [RFC8422].  Clients and servers SHOULD
   support the NIST P-256 (secp256r1) [RFC8422] and X25519 (x25519)
   [RFC7748] curves.  Note that [RFC8422] deprecates all but the
   uncompressed point format.  Therefore, if the client sends an
   ec_point_formats extension, the ECPointFormatList MUST contain a
   single element, "uncompressed".

### 4.3. Cipher Suites for TLS 1.3

   This document does not specify any cipher suites for TLS 1.3.
   Readers are referred to Section 9.1 of [RFC8446] for cipher suite
   recommendations.

### 4.4. Limits on Key Usage

   All ciphers have an upper limit on the amount of traffic that can be
   securely protected with any given key.  In the case of AEAD cipher
   suites, two separate limits are maintained for each key:

   1.  Confidentiality limit (CL), i.e., the number of records that can
       be encrypted.

   2.  Integrity limit (IL), i.e., the number of records that are
       allowed to fail authentication.

   The latter applies to DTLS (and also to QUIC) but not to TLS itself,
   since TLS connections are torn down on the first decryption failure.

   When a sender is approaching CL, the implementation SHOULD initiate a
   new handshake (in TLS 1.3, this can be achieved by sending a
   KeyUpdate message on the established session) to rotate the session
   key.  When a receiver has reached IL, the implementation SHOULD close
   the connection.  Although these recommendations are a best practice,
   implementers need to be aware that it is not always easy to
   accomplish them in protocols that are built on top of TLS/DTLS
   without introducing coordination across layer boundaries.  See Section 6 of [RFC9001] for an example of the cooperation that was
   necessary in QUIC between the crypto and transport layers to support
   key updates.  Note that in general, application protocols might not
   be able to emulate that method given their more constrained
   interaction with TLS/DTLS.  As a result of these complexities, these
   recommendations are not mandatory.

   For all TLS 1.3 cipher suites, readers are referred to Section 5.5 of
   [RFC8446] for the values of CL and IL.  For all DTLS 1.3 cipher
   suites, readers are referred to Section 4.5.3 of [RFC9147].

   For all AES-GCM cipher suites recommended for TLS 1.2 and DTLS 1.2 in
   this document, CL can be derived by plugging the corresponding
   parameters into the inequalities in Section 6.1 of [AEAD-LIMITS ] that
   apply to random, partially implicit nonces, i.e., the nonce
   construction used in TLS 1.2.  Although the obtained figures are
   slightly higher than those for TLS 1.3, it is RECOMMENDED that the
   same limit of 2^24.5 records is used for both versions.

   For all AES-GCM cipher suites recommended for DTLS 1.2, IL (obtained
   from the same inequalities referenced above) is 2^28.

### 4.5. Public Key Length

   When using the cipher suites recommended in this document, two public
   keys are normally used in the TLS handshake: one for the Diffie-
   Hellman key agreement and one for server authentication.  Where a
   client certificate is used, a third public key is added.

   With a key exchange based on modular exponential (MODP) Diffie-
   Hellman groups ("DHE" cipher suites), DH key lengths of at least 2048
   bits are REQUIRED.

   Rationale: For various reasons, in practice, DH keys are typically
   generated in lengths that are powers of two (e.g., 2^10 = 1024 bits,
   2^11 = 2048 bits, 2^12 = 4096 bits).  Because a DH key of 1228 bits
   would be roughly equivalent to only an 80-bit symmetric key
   [RFC3766], it is better to use keys longer than that for the "DHE"
   family of cipher suites.  A DH key of 1926 bits would be roughly
   equivalent to a 100-bit symmetric key [RFC3766].  A DH key of 2048
   bits (equivalent to a 112-bit symmetric key) is the minimum allowed
   by the latest revision of [NIST.SP.800-56A ] as of this writing (see
   in particular Appendix D  of that document).

   As noted in [RFC3766], correcting for the emergence of The Weizmann
   Institute Relation Locator (TWIRL) machine [TWIRL ] would imply that
   1024-bit DH keys yield about 61 bits of equivalent strength and that
   a 2048-bit DH key would yield about 92 bits of equivalent strength.
   The Logjam attack [Logjam ] further demonstrates that 1024-bit Diffie-
   Hellman parameters should be avoided.

   With regard to ECDH keys, implementers are referred to the IANA "TLS
   Supported Groups" registry (formerly known as the "EC Named Curve
   Registry") within the "Transport Layer Security (TLS) Parameters"
   registry [IANA_TLS ] and in particular to the "recommended" groups.
   Curves of less than 224 bits MUST NOT be used.  This recommendation
   is in line with the latest revision of [NIST.SP.800-56A ].

   When using RSA, servers MUST authenticate using certificates with at
   least a 2048-bit modulus for the public key.  In addition, the use of
   the SHA-256 hash algorithm is RECOMMENDED and SHA-1 or MD5 MUST NOT
   be used [RFC9155] (for more details, see also [CAB-Baseline ], for
   which the current version at the time of writing is 1.8.4).  Clients
   MUST indicate to servers that they request SHA-256 by using the
   "Signature Algorithms" extension defined in TLS 1.2.  For TLS 1.3,
   the same requirement is already specified by [RFC8446].

### 4.6. Truncated HMAC

   Implementations MUST NOT use the Truncated HMAC Extension, defined in Section 7 of [RFC6066].

   Rationale: The extension does not apply to the AEAD cipher suites
   recommended above.  However, it does apply to most other TLS cipher
   suites.  Its use has been shown to be insecure in [PatersonRS11].

# Referenced Sections from RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3

The following sections were referenced. Remaining sections are not included.

## 4. Handshake Protocol

   The handshake protocol is used to negotiate the security parameters
   of a connection.  Handshake messages are supplied to the TLS record
   layer, where they are encapsulated within one or more TLSPlaintext or
   TLSCiphertext structures which are processed and transmitted as
   specified by the current active connection state. 
      enum {
          client_hello(1),
          server_hello(2),
          new_session_ticket(4),
          end_of_early_data(5),
          encrypted_extensions(8),
          certificate(11),
          certificate_request(13),
          certificate_verify(15),
          finished(20),
          key_update(24),
          message_hash(254),
          (255)
      } HandshakeType;

      struct {
          HandshakeType msg_type;    /* handshake type */
          uint24 length;             /* remaining bytes in message */
          select (Handshake.msg_type) {
              case client_hello:          ClientHello;
              case server_hello:          ServerHello;
              case end_of_early_data:     EndOfEarlyData;
              case encrypted_extensions:  EncryptedExtensions;
              case certificate_request:   CertificateRequest;
              case certificate:           Certificate;
              case certificate_verify:    CertificateVerify;
              case finished:              Finished;
              case new_session_ticket:    NewSessionTicket;
              case key_update:            KeyUpdate;
          };
      } Handshake;

   Protocol messages MUST be sent in the order defined in Section 4.4.1   and shown in the diagrams in Section 2.  A peer which receives a
   handshake message in an unexpected order MUST abort the handshake
   with an "unexpected_message" alert.

   New handshake message types are assigned by IANA as described in Section 11.

### 4.1. Key Exchange Messages

   The key exchange messages are used to determine the security
   capabilities of the client and the server and to establish shared
   secrets, including the traffic keys used to protect the rest of the
   handshake and the data. 





#### 4.1.1. Cryptographic Negotiation

   In TLS, the cryptographic negotiation proceeds by the client offering
   the following four sets of options in its ClientHello:

   -  A list of cipher suites which indicates the AEAD algorithm/HKDF
      hash pairs which the client supports.

   -  A "supported_groups" (Section 4.2.7) extension which indicates the
      (EC)DHE groups which the client supports and a "key_share"
      (Section 4.2.8) extension which contains (EC)DHE shares for some
      or all of these groups.

   -  A "signature_algorithms" (Section 4.2.3) extension which indicates
      the signature algorithms which the client can accept.  A
      "signature_algorithms_cert" extension (Section 4.2.3) may also be
      added to indicate certificate-specific signature algorithms.

   -  A "pre_shared_key" (Section 4.2.11) extension which contains a
      list of symmetric key identities known to the client and a
      "psk_key_exchange_modes" (Section 4.2.9) extension which indicates
      the key exchange modes that may be used with PSKs.

   If the server does not select a PSK, then the first three of these
   options are entirely orthogonal: the server independently selects a
   cipher suite, an (EC)DHE group and key share for key establishment,
   and a signature algorithm/certificate pair to authenticate itself to
   the client.  If there is no overlap between the received
   "supported_groups" and the groups supported by the server, then the
   server MUST abort the handshake with a "handshake_failure" or an
   "insufficient_security" alert.

   If the server selects a PSK, then it MUST also select a key
   establishment mode from the set indicated by the client's
   "psk_key_exchange_modes" extension (at present, PSK alone or with
   (EC)DHE).  Note that if the PSK can be used without (EC)DHE, then
   non-overlap in the "supported_groups" parameters need not be fatal,
   as it is in the non-PSK case discussed in the previous paragraph.

   If the server selects an (EC)DHE group and the client did not offer a
   compatible "key_share" extension in the initial ClientHello, the
   server MUST respond with a HelloRetryRequest (Section 4.1.4) message. 
   If the server successfully selects parameters and does not require a
   HelloRetryRequest, it indicates the selected parameters in the
   ServerHello as follows:

   -  If PSK is being used, then the server will send a "pre_shared_key"
      extension indicating the selected key.

   -  When (EC)DHE is in use, the server will also provide a "key_share"
      extension.  If PSK is not being used, then (EC)DHE and
      certificate-based authentication are always used.

   -  When authenticating via a certificate, the server will send the
      Certificate (Section 4.4.2) and CertificateVerify (Section 4.4.3)
      messages.  In TLS 1.3 as defined by this document, either a PSK or
      a certificate is always used, but not both.  Future documents may
      define how to use them together.

   If the server is unable to negotiate a supported set of parameters
   (i.e., there is no overlap between the client and server parameters),
   it MUST abort the handshake with either a "handshake_failure" or
   "insufficient_security" fatal alert (see Section 6).

#### 4.1.2. Client Hello

   When a client first connects to a server, it is REQUIRED to send the
   ClientHello as its first TLS message.  The client will also send a
   ClientHello when the server has responded to its ClientHello with a
   HelloRetryRequest.  In that case, the client MUST send the same
   ClientHello without modification, except as follows:

   -  If a "key_share" extension was supplied in the HelloRetryRequest,
      replacing the list of shares with a list containing a single
      KeyShareEntry from the indicated group.

   -  Removing the "early_data" extension (Section 4.2.10) if one was
      present.  Early data is not permitted after a HelloRetryRequest.

   -  Including a "cookie" extension if one was provided in the
      HelloRetryRequest. 
   -  Updating the "pre_shared_key" extension if present by recomputing
      the "obfuscated_ticket_age" and binder values and (optionally)
      removing any PSKs which are incompatible with the server's
      indicated cipher suite.

   -  Optionally adding, removing, or changing the length of the
      "padding" extension [RFC7685].

   -  Other modifications that may be allowed by an extension defined in
      the future and present in the HelloRetryRequest.

   Because TLS 1.3 forbids renegotiation, if a server has negotiated
   TLS 1.3 and receives a ClientHello at any other time, it MUST
   terminate the connection with an "unexpected_message" alert.

   If a server established a TLS connection with a previous version of
   TLS and receives a TLS 1.3 ClientHello in a renegotiation, it MUST
   retain the previous protocol version.  In particular, it MUST NOT
   negotiate TLS 1.3.

   Structure of this message:

      uint16 ProtocolVersion;
      opaque Random[32];

      uint8 CipherSuite[2];    /* Cryptographic suite selector */

      struct {
          ProtocolVersion legacy_version = 0x0303;    /* TLS v1.2 */
          Random random;
          opaque legacy_session_id<0..32>;
          CipherSuite cipher_suites<2..2^16-2>;
          opaque legacy_compression_methods<1..2^8-1>;
          Extension extensions<8..2^16-1>;
      } ClientHello;
   legacy_version:  In previous versions of TLS, this field was used for
      version negotiation and represented the highest version number
      supported by the client.  Experience has shown that many servers
      do not properly implement version negotiation, leading to "version
      intolerance" in which the server rejects an otherwise acceptable
      ClientHello with a version number higher than it supports.  In
      TLS 1.3, the client indicates its version preferences in the
      "supported_versions" extension (Section 4.2.1) and the
      legacy_version field MUST be set to 0x0303, which is the version
      number for TLS 1.2.  TLS 1.3 ClientHellos are identified as having
      a legacy_version of 0x0303 and a supported_versions extension
      present with 0x0304 as the highest version indicated therein.
      (See Appendix D  for details about backward compatibility.)

   random:  32 bytes generated by a secure random number generator.  See Appendix C  for additional information.

   legacy_session_id:  Versions of TLS before TLS 1.3 supported a
      "session resumption" feature which has been merged with pre-shared
      keys in this version (see Section 2.2).  A client which has a
      cached session ID set by a pre-TLS 1.3 server SHOULD set this
      field to that value.  In compatibility mode (see Appendix D.4),
      this field MUST be non-empty, so a client not offering a
      pre-TLS 1.3 session MUST generate a new 32-byte value.  This value
      need not be random but SHOULD be unpredictable to avoid
      implementations fixating on a specific value (also known as
      ossification).  Otherwise, it MUST be set as a zero-length vector
      (i.e., a zero-valued single byte length field).

   cipher_suites:  A list of the symmetric cipher options supported by
      the client, specifically the record protection algorithm
      (including secret key length) and a hash to be used with HKDF, in
      descending order of client preference.  Values are defined in Appendix B.4.  If the list contains cipher suites that the server
      does not recognize, support, or wish to use, the server MUST
      ignore those cipher suites and process the remaining ones as
      usual.  If the client is attempting a PSK key establishment, it
      SHOULD advertise at least one cipher suite indicating a Hash
      associated with the PSK. 
   legacy_compression_methods:  Versions of TLS before 1.3 supported
      compression with the list of supported compression methods being
      sent in this field.  For every TLS 1.3 ClientHello, this vector
      MUST contain exactly one byte, set to zero, which corresponds to
      the "null" compression method in prior versions of TLS.  If a
      TLS 1.3 ClientHello is received with any other value in this
      field, the server MUST abort the handshake with an
      "illegal_parameter" alert.  Note that TLS 1.3 servers might
      receive TLS 1.2 or prior ClientHellos which contain other
      compression methods and (if negotiating such a prior version) MUST
      follow the procedures for the appropriate prior version of TLS.

   extensions:  Clients request extended functionality from servers by
      sending data in the extensions field.  The actual "Extension"
      format is defined in Section 4.2.  In TLS 1.3, the use of certain
      extensions is mandatory, as functionality has moved into
      extensions to preserve ClientHello compatibility with previous
      versions of TLS.  Servers MUST ignore unrecognized extensions.

   All versions of TLS allow an extensions field to optionally follow
   the compression_methods field.  TLS 1.3 ClientHello messages always
   contain extensions (minimally "supported_versions", otherwise, they
   will be interpreted as TLS 1.2 ClientHello messages).  However,
   TLS 1.3 servers might receive ClientHello messages without an
   extensions field from prior versions of TLS.  The presence of
   extensions can be detected by determining whether there are bytes
   following the compression_methods field at the end of the
   ClientHello.  Note that this method of detecting optional data
   differs from the normal TLS method of having a variable-length field,
   but it is used for compatibility with TLS before extensions were
   defined.  TLS 1.3 servers will need to perform this check first and
   only attempt to negotiate TLS 1.3 if the "supported_versions"
   extension is present.  If negotiating a version of TLS prior to 1.3,
   a server MUST check that the message either contains no data after
   legacy_compression_methods or that it contains a valid extensions
   block with no data following.  If not, then it MUST abort the
   handshake with a "decode_error" alert.

   In the event that a client requests additional functionality using
   extensions and this functionality is not supplied by the server, the
   client MAY abort the handshake.

   After sending the ClientHello message, the client waits for a
   ServerHello or HelloRetryRequest message.  If early data is in use,
   the client may transmit early Application Data (Section 2.3) while
   waiting for the next handshake message. 





#### 4.1.3. Server Hello

   The server will send this message in response to a ClientHello
   message to proceed with the handshake if it is able to negotiate an
   acceptable set of handshake parameters based on the ClientHello.

   Structure of this message:

      struct {
          ProtocolVersion legacy_version = 0x0303;    /* TLS v1.2 */
          Random random;
          opaque legacy_session_id_echo<0..32>;
          CipherSuite cipher_suite;
          uint8 legacy_compression_method = 0;
          Extension extensions<6..2^16-1>;
      } ServerHello;

   legacy_version:  In previous versions of TLS, this field was used for
      version negotiation and represented the selected version number
      for the connection.  Unfortunately, some middleboxes fail when
      presented with new values.  In TLS 1.3, the TLS server indicates
      its version using the "supported_versions" extension
      (Section 4.2.1), and the legacy_version field MUST be set to
      0x0303, which is the version number for TLS 1.2.  (See Appendix D       for details about backward compatibility.)

   random:  32 bytes generated by a secure random number generator.  See Appendix C  for additional information.  The last 8 bytes MUST be
      overwritten as described below if negotiating TLS 1.2 or TLS 1.1,
      but the remaining bytes MUST be random.  This structure is
      generated by the server and MUST be generated independently of the
      ClientHello.random.

   legacy_session_id_echo:  The contents of the client's
      legacy_session_id field.  Note that this field is echoed even if
      the client's value corresponded to a cached pre-TLS 1.3 session
      which the server has chosen not to resume.  A client which
      receives a legacy_session_id_echo field that does not match what
      it sent in the ClientHello MUST abort the handshake with an
      "illegal_parameter" alert.

   cipher_suite:  The single cipher suite selected by the server from
      the list in ClientHello.cipher_suites.  A client which receives a
      cipher suite that was not offered MUST abort the handshake with an
      "illegal_parameter" alert.

   legacy_compression_method:  A single byte which MUST have the
      value 0. 
   extensions:  A list of extensions.  The ServerHello MUST only include
      extensions which are required to establish the cryptographic
      context and negotiate the protocol version.  All TLS 1.3
      ServerHello messages MUST contain the "supported_versions"
      extension.  Current ServerHello messages additionally contain
      either the "pre_shared_key" extension or the "key_share"
      extension, or both (when using a PSK with (EC)DHE key
      establishment).  Other extensions (see Section 4.2) are sent
      separately in the EncryptedExtensions message.

   For reasons of backward compatibility with middleboxes (see Appendix D.4), the HelloRetryRequest message uses the same structure
   as the ServerHello, but with Random set to the special value of the
   SHA-256 of "HelloRetryRequest":

     CF 21 AD 74 E5 9A 61 11 BE 1D 8C 02 1E 65 B8 91
     C2 A2 11 16 7A BB 8C 5E 07 9E 09 E2 C8 A8 33 9C

   Upon receiving a message with type server_hello, implementations MUST
   first examine the Random value and, if it matches this value, process
   it as described in Section 4.1.4).

   TLS 1.3 has a downgrade protection mechanism embedded in the server's
   random value.  TLS 1.3 servers which negotiate TLS 1.2 or below in
   response to a ClientHello MUST set the last 8 bytes of their Random
   value specially in their ServerHello.

   If negotiating TLS 1.2, TLS 1.3 servers MUST set the last 8 bytes of
   their Random value to the bytes:

     44 4F 57 4E 47 52 44 01

   If negotiating TLS 1.1 or below, TLS 1.3 servers MUST, and TLS 1.2
   servers SHOULD, set the last 8 bytes of their ServerHello.Random
   value to the bytes:

     44 4F 57 4E 47 52 44 00

   TLS 1.3 clients receiving a ServerHello indicating TLS 1.2 or below
   MUST check that the last 8 bytes are not equal to either of these
   values.  TLS 1.2 clients SHOULD also check that the last 8 bytes are
   not equal to the second value if the ServerHello indicates TLS 1.1 or
   below.  If a match is found, the client MUST abort the handshake with
   an "illegal_parameter" alert.  This mechanism provides limited
   protection against downgrade attacks over and above what is provided
   by the Finished exchange: because the ServerKeyExchange, a message
   present in TLS 1.2 and below, includes a signature over both random
   values, it is not possible for an active attacker to modify the 
   random values without detection as long as ephemeral ciphers are
   used.  It does not provide downgrade protection when static RSA
   is used.

   Note: This is a change from [RFC5246], so in practice many TLS 1.2
   clients and servers will not behave as specified above.

   A legacy TLS client performing renegotiation with TLS 1.2 or prior
   and which receives a TLS 1.3 ServerHello during renegotiation MUST
   abort the handshake with a "protocol_version" alert.  Note that
   renegotiation is not possible when TLS 1.3 has been negotiated.

#### 4.1.4. Hello Retry Request

   The server will send this message in response to a ClientHello
   message if it is able to find an acceptable set of parameters but the
   ClientHello does not contain sufficient information to proceed with
   the handshake.  As discussed in Section 4.1.3, the HelloRetryRequest
   has the same format as a ServerHello message, and the legacy_version,
   legacy_session_id_echo, cipher_suite, and legacy_compression_method
   fields have the same meaning.  However, for convenience we discuss
   "HelloRetryRequest" throughout this document as if it were a distinct
   message.

   The server's extensions MUST contain "supported_versions".
   Additionally, it SHOULD contain the minimal set of extensions
   necessary for the client to generate a correct ClientHello pair.  As
   with the ServerHello, a HelloRetryRequest MUST NOT contain any
   extensions that were not first offered by the client in its
   ClientHello, with the exception of optionally the "cookie" (see Section 4.2.2) extension.

   Upon receipt of a HelloRetryRequest, the client MUST check the
   legacy_version, legacy_session_id_echo, cipher_suite, and
   legacy_compression_method as specified in Section 4.1.3 and then
   process the extensions, starting with determining the version using
   "supported_versions".  Clients MUST abort the handshake with an
   "illegal_parameter" alert if the HelloRetryRequest would not result
   in any change in the ClientHello.  If a client receives a second
   HelloRetryRequest in the same connection (i.e., where the ClientHello
   was itself in response to a HelloRetryRequest), it MUST abort the
   handshake with an "unexpected_message" alert. 
   Otherwise, the client MUST process all extensions in the
   HelloRetryRequest and send a second updated ClientHello.  The
   HelloRetryRequest extensions defined in this specification are:

   -  supported_versions (see Section 4.2.1)

   -  cookie (see Section 4.2.2)

   -  key_share (see Section 4.2.8)

   A client which receives a cipher suite that was not offered MUST
   abort the handshake.  Servers MUST ensure that they negotiate the
   same cipher suite when receiving a conformant updated ClientHello (if
   the server selects the cipher suite as the first step in the
   negotiation, then this will happen automatically).  Upon receiving
   the ServerHello, clients MUST check that the cipher suite supplied in
   the ServerHello is the same as that in the HelloRetryRequest and
   otherwise abort the handshake with an "illegal_parameter" alert.

   In addition, in its updated ClientHello, the client SHOULD NOT offer
   any pre-shared keys associated with a hash other than that of the
   selected cipher suite.  This allows the client to avoid having to
   compute partial hash transcripts for multiple hashes in the second
   ClientHello.

   The value of selected_version in the HelloRetryRequest
   "supported_versions" extension MUST be retained in the ServerHello,
   and a client MUST abort the handshake with an "illegal_parameter"
   alert if the value changes. 





### 4.2. Extensions

   A number of TLS messages contain tag-length-value encoded extensions
   structures.

    struct {
        ExtensionType extension_type;
        opaque extension_data<0..2^16-1>;
    } Extension;

    enum {
        server_name(0),                             /* RFC 6066 */
        max_fragment_length(1),                     /* RFC 6066 */
        status_request(5),                          /* RFC 6066 */
        supported_groups(10),                       /* RFC 8422, 7919 */
        signature_algorithms(13),                   /* RFC 8446 */
        use_srtp(14),                               /* RFC 5764 */
        heartbeat(15),                              /* RFC 6520 */
        application_layer_protocol_negotiation(16), /* RFC 7301 */
        signed_certificate_timestamp(18),           /* RFC 6962 */
        client_certificate_type(19),                /* RFC 7250 */
        server_certificate_type(20),                /* RFC 7250 */
        padding(21),                                /* RFC 7685 */
        pre_shared_key(41),                         /* RFC 8446 */
        early_data(42),                             /* RFC 8446 */
        supported_versions(43),                     /* RFC 8446 */
        cookie(44),                                 /* RFC 8446 */
        psk_key_exchange_modes(45),                 /* RFC 8446 */
        certificate_authorities(47),                /* RFC 8446 */
        oid_filters(48),                            /* RFC 8446 */
        post_handshake_auth(49),                    /* RFC 8446 */
        signature_algorithms_cert(50),              /* RFC 8446 */
        key_share(51),                              /* RFC 8446 */
        (65535)
    } ExtensionType;
   Here:

   -  "extension_type" identifies the particular extension type.

   -  "extension_data" contains information specific to the particular
      extension type.

   The list of extension types is maintained by IANA as described in Section 11.

   Extensions are generally structured in a request/response fashion,
   though some extensions are just indications with no corresponding
   response.  The client sends its extension requests in the ClientHello
   message, and the server sends its extension responses in the
   ServerHello, EncryptedExtensions, HelloRetryRequest, and Certificate
   messages.  The server sends extension requests in the
   CertificateRequest message which a client MAY respond to with a
   Certificate message.  The server MAY also send unsolicited extensions
   in the NewSessionTicket, though the client does not respond directly
   to these.

   Implementations MUST NOT send extension responses if the remote
   endpoint did not send the corresponding extension requests, with the
   exception of the "cookie" extension in the HelloRetryRequest.  Upon
   receiving such an extension, an endpoint MUST abort the handshake
   with an "unsupported_extension" alert.

   The table below indicates the messages where a given extension may
   appear, using the following notation: CH (ClientHello),
   SH (ServerHello), EE (EncryptedExtensions), CT (Certificate),
   CR (CertificateRequest), NST (NewSessionTicket), and
   HRR (HelloRetryRequest).  If an implementation receives an extension
   which it recognizes and which is not specified for the message in
   which it appears, it MUST abort the handshake with an
   "illegal_parameter" alert. 
   +--------------------------------------------------+-------------+
   | Extension                                        |     TLS 1.3 |
   +--------------------------------------------------+-------------+
   | server_name [RFC6066]                            |      CH, EE |
   |                                                  |             |
   | max_fragment_length [RFC6066]                    |      CH, EE |
   |                                                  |             |
   | status_request [RFC6066]                         |  CH, CR, CT |
   |                                                  |             |
   | supported_groups [RFC7919]                       |      CH, EE |
   |                                                  |             |
   | signature_algorithms (RFC 8446)                  |      CH, CR |
   |                                                  |             |
   | use_srtp [RFC5764]                               |      CH, EE |
   |                                                  |             |
   | heartbeat [RFC6520]                              |      CH, EE |
   |                                                  |             |
   | application_layer_protocol_negotiation [RFC7301] |      CH, EE |
   |                                                  |             |
   | signed_certificate_timestamp [RFC6962]           |  CH, CR, CT |
   |                                                  |             |
   | client_certificate_type [RFC7250]                |      CH, EE |
   |                                                  |             |
   | server_certificate_type [RFC7250]                |      CH, EE |
   |                                                  |             |
   | padding [RFC7685]                                |          CH |
   |                                                  |             |
   | key_share (RFC 8446)                             | CH, SH, HRR |
   |                                                  |             |
   | pre_shared_key (RFC 8446)                        |      CH, SH |
   |                                                  |             |
   | psk_key_exchange_modes (RFC 8446)                |          CH |
   |                                                  |             |
   | early_data (RFC 8446)                            | CH, EE, NST |
   |                                                  |             |
   | cookie (RFC 8446)                                |     CH, HRR |
   |                                                  |             |
   | supported_versions (RFC 8446)                    | CH, SH, HRR |
   |                                                  |             |
   | certificate_authorities (RFC 8446)               |      CH, CR |
   |                                                  |             |
   | oid_filters (RFC 8446)                           |          CR |
   |                                                  |             |
   | post_handshake_auth (RFC 8446)                   |          CH |
   |                                                  |             |
   | signature_algorithms_cert (RFC 8446)             |      CH, CR |
   +--------------------------------------------------+-------------+
   When multiple extensions of different types are present, the
   extensions MAY appear in any order, with the exception of
   "pre_shared_key" (Section 4.2.11) which MUST be the last extension in
   the ClientHello (but can appear anywhere in the ServerHello
   extensions block).  There MUST NOT be more than one extension of the
   same type in a given extension block.

   In TLS 1.3, unlike TLS 1.2, extensions are negotiated for each
   handshake even when in resumption-PSK mode.  However, 0-RTT
   parameters are those negotiated in the previous handshake; mismatches
   may require rejecting 0-RTT (see Section 4.2.10).

   There are subtle (and not so subtle) interactions that may occur in
   this protocol between new features and existing features which may
   result in a significant reduction in overall security.  The following
   considerations should be taken into account when designing new
   extensions:

   -  Some cases where a server does not agree to an extension are error
      conditions (e.g., the handshake cannot continue), and some are
      simply refusals to support particular features.  In general, error
      alerts should be used for the former and a field in the server
      extension response for the latter.

   -  Extensions should, as far as possible, be designed to prevent any
      attack that forces use (or non-use) of a particular feature by
      manipulation of handshake messages.  This principle should be
      followed regardless of whether the feature is believed to cause a
      security problem.  Often the fact that the extension fields are
      included in the inputs to the Finished message hashes will be
      sufficient, but extreme care is needed when the extension changes
      the meaning of messages sent in the handshake phase.  Designers
      and implementors should be aware of the fact that until the
      handshake has been authenticated, active attackers can modify
      messages and insert, remove, or replace extensions. 





#### 4.2.1. Supported Versions

      struct {
          select (Handshake.msg_type) {
              case client_hello:
                   ProtocolVersion versions<2..254>;

              case server_hello: /* and HelloRetryRequest */
                   ProtocolVersion selected_version;
          };
      } SupportedVersions;

   The "supported_versions" extension is used by the client to indicate
   which versions of TLS it supports and by the server to indicate which
   version it is using.  The extension contains a list of supported
   versions in preference order, with the most preferred version first.
   Implementations of this specification MUST send this extension in the
   ClientHello containing all versions of TLS which they are prepared to
   negotiate (for this specification, that means minimally 0x0304, but
   if previous versions of TLS are allowed to be negotiated, they MUST
   be present as well).

   If this extension is not present, servers which are compliant with
   this specification and which also support TLS 1.2 MUST negotiate
   TLS 1.2 or prior as specified in [RFC5246], even if
   ClientHello.legacy_version is 0x0304 or later.  Servers MAY abort the
   handshake upon receiving a ClientHello with legacy_version 0x0304 or
   later.

   If this extension is present in the ClientHello, servers MUST NOT use
   the ClientHello.legacy_version value for version negotiation and MUST
   use only the "supported_versions" extension to determine client
   preferences.  Servers MUST only select a version of TLS present in
   that extension and MUST ignore any unknown versions that are present
   in that extension.  Note that this mechanism makes it possible to
   negotiate a version prior to TLS 1.2 if one side supports a sparse
   range.  Implementations of TLS 1.3 which choose to support prior
   versions of TLS SHOULD support TLS 1.2.  Servers MUST be prepared to
   receive ClientHellos that include this extension but do not include
   0x0304 in the list of versions.

   A server which negotiates a version of TLS prior to TLS 1.3 MUST set
   ServerHello.version and MUST NOT send the "supported_versions"
   extension.  A server which negotiates TLS 1.3 MUST respond by sending
   a "supported_versions" extension containing the selected version
   value (0x0304).  It MUST set the ServerHello.legacy_version field to
   0x0303 (TLS 1.2).  Clients MUST check for this extension prior to
   processing the rest of the ServerHello (although they will have to 
   parse the ServerHello in order to read the extension).  If this
   extension is present, clients MUST ignore the
   ServerHello.legacy_version value and MUST use only the
   "supported_versions" extension to determine the selected version.  If
   the "supported_versions" extension in the ServerHello contains a
   version not offered by the client or contains a version prior to
   TLS 1.3, the client MUST abort the handshake with an
   "illegal_parameter" alert.

#### 4.2.2. Cookie

      struct {
          opaque cookie<1..2^16-1>;
      } Cookie;

   Cookies serve two primary purposes:

   -  Allowing the server to force the client to demonstrate
      reachability at their apparent network address (thus providing a
      measure of DoS protection).  This is primarily useful for
      non-connection-oriented transports (see [RFC6347] for an example
      of this).

   -  Allowing the server to offload state to the client, thus allowing
      it to send a HelloRetryRequest without storing any state.  The
      server can do this by storing the hash of the ClientHello in the
      HelloRetryRequest cookie (protected with some suitable integrity
      protection algorithm).

   When sending a HelloRetryRequest, the server MAY provide a "cookie"
   extension to the client (this is an exception to the usual rule that
   the only extensions that may be sent are those that appear in the
   ClientHello).  When sending the new ClientHello, the client MUST copy
   the contents of the extension received in the HelloRetryRequest into
   a "cookie" extension in the new ClientHello.  Clients MUST NOT use
   cookies in their initial ClientHello in subsequent connections.

   When a server is operating statelessly, it may receive an unprotected
   record of type change_cipher_spec between the first and second
   ClientHello (see Section 5).  Since the server is not storing any
   state, this will appear as if it were the first message to be
   received.  Servers operating statelessly MUST ignore these records. 





#### 4.2.3. Signature Algorithms

   TLS 1.3 provides two extensions for indicating which signature
   algorithms may be used in digital signatures.  The
   "signature_algorithms_cert" extension applies to signatures in
   certificates, and the "signature_algorithms" extension, which
   originally appeared in TLS 1.2, applies to signatures in
   CertificateVerify messages.  The keys found in certificates MUST also
   be of appropriate type for the signature algorithms they are used
   with.  This is a particular issue for RSA keys and PSS signatures, as
   described below.  If no "signature_algorithms_cert" extension is
   present, then the "signature_algorithms" extension also applies to
   signatures appearing in certificates.  Clients which desire the
   server to authenticate itself via a certificate MUST send the
   "signature_algorithms" extension.  If a server is authenticating via
   a certificate and the client has not sent a "signature_algorithms"
   extension, then the server MUST abort the handshake with a
   "missing_extension" alert (see Section 9.2).

   The "signature_algorithms_cert" extension was added to allow
   implementations which supported different sets of algorithms for
   certificates and in TLS itself to clearly signal their capabilities.
   TLS 1.2 implementations SHOULD also process this extension.
   Implementations which have the same policy in both cases MAY omit the
   "signature_algorithms_cert" extension. 
   The "extension_data" field of these extensions contains a
   SignatureSchemeList value:

      enum {
          /* RSASSA-PKCS1-v1_5 algorithms */
          rsa_pkcs1_sha256(0x0401),
          rsa_pkcs1_sha384(0x0501),
          rsa_pkcs1_sha512(0x0601),

          /* ECDSA algorithms */
          ecdsa_secp256r1_sha256(0x0403),
          ecdsa_secp384r1_sha384(0x0503),
          ecdsa_secp521r1_sha512(0x0603),

          /* RSASSA-PSS algorithms with public key OID rsaEncryption */
          rsa_pss_rsae_sha256(0x0804),
          rsa_pss_rsae_sha384(0x0805),
          rsa_pss_rsae_sha512(0x0806),

          /* EdDSA algorithms */
          ed25519(0x0807),
          ed448(0x0808),

          /* RSASSA-PSS algorithms with public key OID RSASSA-PSS */
          rsa_pss_pss_sha256(0x0809),
          rsa_pss_pss_sha384(0x080a),
          rsa_pss_pss_sha512(0x080b),

          /* Legacy algorithms */
          rsa_pkcs1_sha1(0x0201),
          ecdsa_sha1(0x0203),

          /* Reserved Code Points */
          private_use(0xFE00..0xFFFF),
          (0xFFFF)
      } SignatureScheme;

      struct {
          SignatureScheme supported_signature_algorithms<2..2^16-2>;
      } SignatureSchemeList;

   Note: This enum is named "SignatureScheme" because there is already a
   "SignatureAlgorithm" type in TLS 1.2, which this replaces.  We use
   the term "signature algorithm" throughout the text. 
   Each SignatureScheme value lists a single signature algorithm that
   the client is willing to verify.  The values are indicated in
   descending order of preference.  Note that a signature algorithm
   takes as input an arbitrary-length message, rather than a digest.
   Algorithms which traditionally act on a digest should be defined in
   TLS to first hash the input with a specified hash algorithm and then
   proceed as usual.  The code point groups listed above have the
   following meanings:

   RSASSA-PKCS1-v1_5 algorithms:  Indicates a signature algorithm using
      RSASSA-PKCS1-v1_5 [RFC8017] with the corresponding hash algorithm
      as defined in [SHS ].  These values refer solely to signatures
      which appear in certificates (see Section 4.4.2.2) and are not
      defined for use in signed TLS handshake messages, although they
      MAY appear in "signature_algorithms" and
      "signature_algorithms_cert" for backward compatibility with
      TLS 1.2.

   ECDSA algorithms:  Indicates a signature algorithm using ECDSA
      [ECDSA ], the corresponding curve as defined in ANSI X9.62 [ECDSA ]
      and FIPS 186-4 [DSS ], and the corresponding hash algorithm as
      defined in [SHS ].  The signature is represented as a DER-encoded
      [X690] ECDSA-Sig-Value structure.

   RSASSA-PSS RSAE algorithms:  Indicates a signature algorithm using
      RSASSA-PSS [RFC8017] with mask generation function 1.  The digest
      used in the mask generation function and the digest being signed
      are both the corresponding hash algorithm as defined in [SHS ].
      The length of the Salt MUST be equal to the length of the output
      of the digest algorithm.  If the public key is carried in an X.509
      certificate, it MUST use the rsaEncryption OID [RFC5280].

   EdDSA algorithms:  Indicates a signature algorithm using EdDSA as
      defined in [RFC8032] or its successors.  Note that these
      correspond to the "PureEdDSA" algorithms and not the "prehash"
      variants.

   RSASSA-PSS PSS algorithms:  Indicates a signature algorithm using
      RSASSA-PSS [RFC8017] with mask generation function 1.  The digest
      used in the mask generation function and the digest being signed
      are both the corresponding hash algorithm as defined in [SHS ].
      The length of the Salt MUST be equal to the length of the digest
      algorithm.  If the public key is carried in an X.509 certificate,
      it MUST use the RSASSA-PSS OID [RFC5756].  When used in
      certificate signatures, the algorithm parameters MUST be DER
      encoded.  If the corresponding public key's parameters are
      present, then the parameters in the signature MUST be identical to
      those in the public key. 
   Legacy algorithms:  Indicates algorithms which are being deprecated
      because they use algorithms with known weaknesses, specifically
      SHA-1 which is used in this context with either (1) RSA using
      RSASSA-PKCS1-v1_5 or (2) ECDSA.  These values refer solely to
      signatures which appear in certificates (see Section 4.4.2.2) and
      are not defined for use in signed TLS handshake messages, although
      they MAY appear in "signature_algorithms" and
      "signature_algorithms_cert" for backward compatibility with
      TLS 1.2.  Endpoints SHOULD NOT negotiate these algorithms but are
      permitted to do so solely for backward compatibility.  Clients
      offering these values MUST list them as the lowest priority
      (listed after all other algorithms in SignatureSchemeList).
      TLS 1.3 servers MUST NOT offer a SHA-1 signed certificate unless
      no valid certificate chain can be produced without it (see Section 4.4.2.2).

   The signatures on certificates that are self-signed or certificates
   that are trust anchors are not validated, since they begin a
   certification path (see [RFC5280], Section 3.2).  A certificate that
   begins a certification path MAY use a signature algorithm that is not
   advertised as being supported in the "signature_algorithms"
   extension.

   Note that TLS 1.2 defines this extension differently.  TLS 1.3
   implementations willing to negotiate TLS 1.2 MUST behave in
   accordance with the requirements of [RFC5246] when negotiating that
   version.  In particular:

   -  TLS 1.2 ClientHellos MAY omit this extension.

   -  In TLS 1.2, the extension contained hash/signature pairs.  The
      pairs are encoded in two octets, so SignatureScheme values have
      been allocated to align with TLS 1.2's encoding.  Some legacy
      pairs are left unallocated.  These algorithms are deprecated as of
      TLS 1.3.  They MUST NOT be offered or negotiated by any
      implementation.  In particular, MD5 [SLOTH ], SHA-224, and DSA
      MUST NOT be used.

   -  ECDSA signature schemes align with TLS 1.2's ECDSA hash/signature
      pairs.  However, the old semantics did not constrain the signing
      curve.  If TLS 1.2 is negotiated, implementations MUST be prepared
      to accept a signature that uses any curve that they advertised in
      the "supported_groups" extension.

   -  Implementations that advertise support for RSASSA-PSS (which is
      mandatory in TLS 1.3) MUST be prepared to accept a signature using
      that scheme even when TLS 1.2 is negotiated.  In TLS 1.2,
      RSASSA-PSS is used with RSA cipher suites. 





#### 4.2.4. Certificate Authorities

   The "certificate_authorities" extension is used to indicate the
   certificate authorities (CAs) which an endpoint supports and which
   SHOULD be used by the receiving endpoint to guide certificate
   selection.

   The body of the "certificate_authorities" extension consists of a
   CertificateAuthoritiesExtension structure.

      opaque DistinguishedName<1..2^16-1>;

      struct {
          DistinguishedName authorities<3..2^16-1>;
      } CertificateAuthoritiesExtension;

   authorities:  A list of the distinguished names [X501] of acceptable
      certificate authorities, represented in DER-encoded [X690] format.
      These distinguished names specify a desired distinguished name for
      a trust anchor or subordinate CA; thus, this message can be used
      to describe known trust anchors as well as a desired authorization
      space.

   The client MAY send the "certificate_authorities" extension in the
   ClientHello message.  The server MAY send it in the
   CertificateRequest message.

   The "trusted_ca_keys" extension [RFC6066], which serves a similar
   purpose but is more complicated, is not used in TLS 1.3 (although it
   may appear in ClientHello messages from clients which are offering
   prior versions of TLS).

#### 4.2.5. OID Filters

   The "oid_filters" extension allows servers to provide a set of
   OID/value pairs which it would like the client's certificate to
   match.  This extension, if provided by the server, MUST only be sent
   in the CertificateRequest message.

      struct {
          opaque certificate_extension_oid<1..2^8-1>;
          opaque certificate_extension_values<0..2^16-1>;
      } OIDFilter;

      struct {
          OIDFilter filters<0..2^16-1>;
      } OIDFilterExtension;
   filters:  A list of certificate extension OIDs [RFC5280] with their
      allowed value(s) and represented in DER-encoded [X690] format.
      Some certificate extension OIDs allow multiple values (e.g.,
      Extended Key Usage).  If the server has included a non-empty
      filters list, the client certificate included in the response MUST
      contain all of the specified extension OIDs that the client
      recognizes.  For each extension OID recognized by the client, all
      of the specified values MUST be present in the client certificate
      (but the certificate MAY have other values as well).  However, the
      client MUST ignore and skip any unrecognized certificate extension
      OIDs.  If the client ignored some of the required certificate
      extension OIDs and supplied a certificate that does not satisfy
      the request, the server MAY at its discretion either continue the
      connection without client authentication or abort the handshake
      with an "unsupported_certificate" alert.  Any given OID MUST NOT
      appear more than once in the filters list.

   PKIX RFCs define a variety of certificate extension OIDs and their
   corresponding value types.  Depending on the type, matching
   certificate extension values are not necessarily bitwise-equal.  It
   is expected that TLS implementations will rely on their PKI libraries
   to perform certificate selection using certificate extension OIDs.

   This document defines matching rules for two standard certificate
   extensions defined in [RFC5280]:

   -  The Key Usage extension in a certificate matches the request when
      all key usage bits asserted in the request are also asserted in
      the Key Usage certificate extension.

   -  The Extended Key Usage extension in a certificate matches the
      request when all key purpose OIDs present in the request are also
      found in the Extended Key Usage certificate extension.  The
      special anyExtendedKeyUsage OID MUST NOT be used in the request.

   Separate specifications may define matching rules for other
   certificate extensions. 





#### 4.2.6. Post-Handshake Client Authentication

   The "post_handshake_auth" extension is used to indicate that a client
   is willing to perform post-handshake authentication (Section 4.6.2).
   Servers MUST NOT send a post-handshake CertificateRequest to clients
   which do not offer this extension.  Servers MUST NOT send this
   extension.

      struct {} PostHandshakeAuth;

   The "extension_data" field of the "post_handshake_auth" extension is
   zero length.

#### 4.2.7. Supported Groups

   When sent by the client, the "supported_groups" extension indicates
   the named groups which the client supports for key exchange, ordered
   from most preferred to least preferred.

   Note: In versions of TLS prior to TLS 1.3, this extension was named
   "elliptic_curves" and only contained elliptic curve groups.  See
   [RFC8422] and [RFC7919].  This extension was also used to negotiate
   ECDSA curves.  Signature algorithms are now negotiated independently
   (see Section 4.2.3).

   The "extension_data" field of this extension contains a
   "NamedGroupList" value:

      enum {

          /* Elliptic Curve Groups (ECDHE) */
          secp256r1(0x0017), secp384r1(0x0018), secp521r1(0x0019),
          x25519(0x001D), x448(0x001E),

          /* Finite Field Groups (DHE) */
          ffdhe2048(0x0100), ffdhe3072(0x0101), ffdhe4096(0x0102),
          ffdhe6144(0x0103), ffdhe8192(0x0104),

          /* Reserved Code Points */
          ffdhe_private_use(0x01FC..0x01FF),
          ecdhe_private_use(0xFE00..0xFEFF),
          (0xFFFF)
      } NamedGroup;

      struct {
          NamedGroup named_group_list<2..2^16-1>;
      } NamedGroupList;
   Elliptic Curve Groups (ECDHE):  Indicates support for the
      corresponding named curve, defined in either FIPS 186-4 [DSS ] or
      [RFC7748].  Values 0xFE00 through 0xFEFF are reserved for
      Private Use [RFC8126].

   Finite Field Groups (DHE):  Indicates support for the corresponding
      finite field group, defined in [RFC7919].  Values 0x01FC through
      0x01FF are reserved for Private Use.

   Items in named_group_list are ordered according to the sender's
   preferences (most preferred choice first).

   As of TLS 1.3, servers are permitted to send the "supported_groups"
   extension to the client.  Clients MUST NOT act upon any information
   found in "supported_groups" prior to successful completion of the
   handshake but MAY use the information learned from a successfully
   completed handshake to change what groups they use in their
   "key_share" extension in subsequent connections.  If the server has a
   group it prefers to the ones in the "key_share" extension but is
   still willing to accept the ClientHello, it SHOULD send
   "supported_groups" to update the client's view of its preferences;
   this extension SHOULD contain all groups the server supports,
   regardless of whether they are currently supported by the client.

#### 4.2.8. Key Share

   The "key_share" extension contains the endpoint's cryptographic
   parameters.

   Clients MAY send an empty client_shares vector in order to request
   group selection from the server, at the cost of an additional round
   trip (see Section 4.1.4).

      struct {
          NamedGroup group;
          opaque key_exchange<1..2^16-1>;
      } KeyShareEntry;

   group:  The named group for the key being exchanged.

   key_exchange:  Key exchange information.  The contents of this field
      are determined by the specified group and its corresponding
      definition.  Finite Field Diffie-Hellman [DH76] parameters are
      described in Section 4.2.8.1; Elliptic Curve Diffie-Hellman
      parameters are described in Section 4.2.8.2. 
   In the ClientHello message, the "extension_data" field of this
   extension contains a "KeyShareClientHello" value:

      struct {
          KeyShareEntry client_shares<0..2^16-1>;
      } KeyShareClientHello;

   client_shares:  A list of offered KeyShareEntry values in descending
      order of client preference.

   This vector MAY be empty if the client is requesting a
   HelloRetryRequest.  Each KeyShareEntry value MUST correspond to a
   group offered in the "supported_groups" extension and MUST appear in
   the same order.  However, the values MAY be a non-contiguous subset
   of the "supported_groups" extension and MAY omit the most preferred
   groups.  Such a situation could arise if the most preferred groups
   are new and unlikely to be supported in enough places to make
   pregenerating key shares for them efficient.

   Clients can offer as many KeyShareEntry values as the number of
   supported groups it is offering, each representing a single set of
   key exchange parameters.  For instance, a client might offer shares
   for several elliptic curves or multiple FFDHE groups.  The
   key_exchange values for each KeyShareEntry MUST be generated
   independently.  Clients MUST NOT offer multiple KeyShareEntry values
   for the same group.  Clients MUST NOT offer any KeyShareEntry values
   for groups not listed in the client's "supported_groups" extension.
   Servers MAY check for violations of these rules and abort the
   handshake with an "illegal_parameter" alert if one is violated.

   In a HelloRetryRequest message, the "extension_data" field of this
   extension contains a KeyShareHelloRetryRequest value:

      struct {
          NamedGroup selected_group;
      } KeyShareHelloRetryRequest;

   selected_group:  The mutually supported group the server intends to
      negotiate and is requesting a retried ClientHello/KeyShare for.

   Upon receipt of this extension in a HelloRetryRequest, the client
   MUST verify that (1) the selected_group field corresponds to a group
   which was provided in the "supported_groups" extension in the
   original ClientHello and (2) the selected_group field does not
   correspond to a group which was provided in the "key_share" extension
   in the original ClientHello.  If either of these checks fails, then
   the client MUST abort the handshake with an "illegal_parameter"
   alert.  Otherwise, when sending the new ClientHello, the client MUST 
   replace the original "key_share" extension with one containing only a
   new KeyShareEntry for the group indicated in the selected_group field
   of the triggering HelloRetryRequest.

   In a ServerHello message, the "extension_data" field of this
   extension contains a KeyShareServerHello value:

      struct {
          KeyShareEntry server_share;
      } KeyShareServerHello;

   server_share:  A single KeyShareEntry value that is in the same group
      as one of the client's shares.

   If using (EC)DHE key establishment, servers offer exactly one
   KeyShareEntry in the ServerHello.  This value MUST be in the same
   group as the KeyShareEntry value offered by the client that the
   server has selected for the negotiated key exchange.  Servers
   MUST NOT send a KeyShareEntry for any group not indicated in the
   client's "supported_groups" extension and MUST NOT send a
   KeyShareEntry when using the "psk_ke" PskKeyExchangeMode.  If using
   (EC)DHE key establishment and a HelloRetryRequest containing a
   "key_share" extension was received by the client, the client MUST
   verify that the selected NamedGroup in the ServerHello is the same as
   that in the HelloRetryRequest.  If this check fails, the client MUST
   abort the handshake with an "illegal_parameter" alert.

##### 4.2.8.1. Diffie-Hellman Parameters

   Diffie-Hellman [DH76] parameters for both clients and servers are
   encoded in the opaque key_exchange field of a KeyShareEntry in a
   KeyShare structure.  The opaque value contains the Diffie-Hellman
   public value (Y = g^X mod p) for the specified group (see [RFC7919]
   for group definitions) encoded as a big-endian integer and padded to
   the left with zeros to the size of p in bytes.

   Note: For a given Diffie-Hellman group, the padding results in all
   public keys having the same length.

   Peers MUST validate each other's public key Y by ensuring that 1 < Y
   < p-1.  This check ensures that the remote peer is properly behaved
   and isn't forcing the local system into a small subgroup. 





##### 4.2.8.2. ECDHE Parameters

   ECDHE parameters for both clients and servers are encoded in the
   opaque key_exchange field of a KeyShareEntry in a KeyShare structure.

   For secp256r1, secp384r1, and secp521r1, the contents are the
   serialized value of the following struct:

      struct {
          uint8 legacy_form = 4;
          opaque X[coordinate_length];
          opaque Y[coordinate_length];
      } UncompressedPointRepresentation;

   X and Y, respectively, are the binary representations of the x and y
   values in network byte order.  There are no internal length markers,
   so each number representation occupies as many octets as implied by
   the curve parameters.  For P-256, this means that each of X and Y use
   32 octets, padded on the left by zeros if necessary.  For P-384, they
   take 48 octets each.  For P-521, they take 66 octets each.

   For the curves secp256r1, secp384r1, and secp521r1, peers MUST
   validate each other's public value Q by ensuring that the point is a
   valid point on the elliptic curve.  The appropriate validation
   procedures are defined in Section 4.3.7 of [ECDSA ] and alternatively
   in Section 5.6.2.3 of [KEYAGREEMENT ].  This process consists of three
   steps: (1) verify that Q is not the point at infinity (O), (2) verify
   that for Q = (x, y) both integers x and y are in the correct
   interval, and (3) ensure that (x, y) is a correct solution to the
   elliptic curve equation.  For these curves, implementors do not need
   to verify membership in the correct subgroup.

   For X25519 and X448, the contents of the public value are the byte
   string inputs and outputs of the corresponding functions defined in
   [RFC7748]: 32 bytes for X25519 and 56 bytes for X448.

   Note: Versions of TLS prior to 1.3 permitted point format
   negotiation; TLS 1.3 removes this feature in favor of a single point
   format for each curve.

#### 4.2.9. Pre-Shared Key Exchange Modes

   In order to use PSKs, clients MUST also send a
   "psk_key_exchange_modes" extension.  The semantics of this extension
   are that the client only supports the use of PSKs with these modes,
   which restricts both the use of PSKs offered in this ClientHello and
   those which the server might supply via NewSessionTicket. 
   A client MUST provide a "psk_key_exchange_modes" extension if it
   offers a "pre_shared_key" extension.  If clients offer
   "pre_shared_key" without a "psk_key_exchange_modes" extension,
   servers MUST abort the handshake.  Servers MUST NOT select a key
   exchange mode that is not listed by the client.  This extension also
   restricts the modes for use with PSK resumption.  Servers SHOULD NOT
   send NewSessionTicket with tickets that are not compatible with the
   advertised modes; however, if a server does so, the impact will just
   be that the client's attempts at resumption fail.

   The server MUST NOT send a "psk_key_exchange_modes" extension.

      enum { psk_ke(0), psk_dhe_ke(1), (255) } PskKeyExchangeMode;

      struct {
          PskKeyExchangeMode ke_modes<1..255>;
      } PskKeyExchangeModes;

   psk_ke:  PSK-only key establishment.  In this mode, the server
      MUST NOT supply a "key_share" value.

   psk_dhe_ke:  PSK with (EC)DHE key establishment.  In this mode, the
      client and server MUST supply "key_share" values as described in Section 4.2.8.

   Any future values that are allocated must ensure that the transmitted
   protocol messages unambiguously identify which mode was selected by
   the server; at present, this is indicated by the presence of the
   "key_share" in the ServerHello.

#### 4.2.10. Early Data Indication

   When a PSK is used and early data is allowed for that PSK, the client
   can send Application Data in its first flight of messages.  If the
   client opts to do so, it MUST supply both the "pre_shared_key" and
   "early_data" extensions.

   The "extension_data" field of this extension contains an
   "EarlyDataIndication" value. 
      struct {} Empty;

      struct {
          select (Handshake.msg_type) {
              case new_session_ticket:   uint32 max_early_data_size;
              case client_hello:         Empty;
              case encrypted_extensions: Empty;
          };
      } EarlyDataIndication;

   See Section 4.6.1 for details regarding the use of the
   max_early_data_size field.

   The parameters for the 0-RTT data (version, symmetric cipher suite,
   Application-Layer Protocol Negotiation (ALPN) [RFC7301] protocol,
   etc.) are those associated with the PSK in use.  For externally
   provisioned PSKs, the associated values are those provisioned along
   with the key.  For PSKs established via a NewSessionTicket message,
   the associated values are those which were negotiated in the
   connection which established the PSK.  The PSK used to encrypt the
   early data MUST be the first PSK listed in the client's
   "pre_shared_key" extension.

   For PSKs provisioned via NewSessionTicket, a server MUST validate
   that the ticket age for the selected PSK identity (computed by
   subtracting ticket_age_add from PskIdentity.obfuscated_ticket_age
   modulo 2^32) is within a small tolerance of the time since the ticket
   was issued (see Section 8).  If it is not, the server SHOULD proceed
   with the handshake but reject 0-RTT, and SHOULD NOT take any other
   action that assumes that this ClientHello is fresh.

   0-RTT messages sent in the first flight have the same (encrypted)
   content types as messages of the same type sent in other flights
   (handshake and application_data) but are protected under different
   keys.  After receiving the server's Finished message, if the server
   has accepted early data, an EndOfEarlyData message will be sent to
   indicate the key change.  This message will be encrypted with the
   0-RTT traffic keys. 
   A server which receives an "early_data" extension MUST behave in one
   of three ways:

   -  Ignore the extension and return a regular 1-RTT response.  The
      server then skips past early data by attempting to deprotect
      received records using the handshake traffic key, discarding
      records which fail deprotection (up to the configured
      max_early_data_size).  Once a record is deprotected successfully,
      it is treated as the start of the client's second flight and the
      server proceeds as with an ordinary 1-RTT handshake.

   -  Request that the client send another ClientHello by responding
      with a HelloRetryRequest.  A client MUST NOT include the
      "early_data" extension in its followup ClientHello.  The server
      then ignores early data by skipping all records with an external
      content type of "application_data" (indicating that they are
      encrypted), up to the configured max_early_data_size.

   -  Return its own "early_data" extension in EncryptedExtensions,
      indicating that it intends to process the early data.  It is not
      possible for the server to accept only a subset of the early data
      messages.  Even though the server sends a message accepting early
      data, the actual early data itself may already be in flight by the
      time the server generates this message.

   In order to accept early data, the server MUST have accepted a PSK
   cipher suite and selected the first key offered in the client's
   "pre_shared_key" extension.  In addition, it MUST verify that the
   following values are the same as those associated with the
   selected PSK:

   -  The TLS version number

   -  The selected cipher suite

   -  The selected ALPN [RFC7301] protocol, if any

   These requirements are a superset of those needed to perform a 1-RTT
   handshake using the PSK in question.  For externally established
   PSKs, the associated values are those provisioned along with the key.
   For PSKs established via a NewSessionTicket message, the associated
   values are those negotiated in the connection during which the ticket
   was established.

   Future extensions MUST define their interaction with 0-RTT. 
   If any of these checks fail, the server MUST NOT respond with the
   extension and must discard all the first-flight data using one of the
   first two mechanisms listed above (thus falling back to 1-RTT or
   2-RTT).  If the client attempts a 0-RTT handshake but the server
   rejects it, the server will generally not have the 0-RTT record
   protection keys and must instead use trial decryption (either with
   the 1-RTT handshake keys or by looking for a cleartext ClientHello in
   the case of a HelloRetryRequest) to find the first non-0-RTT message.

   If the server chooses to accept the "early_data" extension, then it
   MUST comply with the same error-handling requirements specified for
   all records when processing early data records.  Specifically, if the
   server fails to decrypt a 0-RTT record following an accepted
   "early_data" extension, it MUST terminate the connection with a
   "bad_record_mac" alert as per Section 5.2.

   If the server rejects the "early_data" extension, the client
   application MAY opt to retransmit the Application Data previously
   sent in early data once the handshake has been completed.  Note that
   automatic retransmission of early data could result in incorrect
   assumptions regarding the status of the connection.  For instance,
   when the negotiated connection selects a different ALPN protocol from
   what was used for the early data, an application might need to
   construct different messages.  Similarly, if early data assumes
   anything about the connection state, it might be sent in error after
   the handshake completes.

   A TLS implementation SHOULD NOT automatically resend early data;
   applications are in a better position to decide when retransmission
   is appropriate.  A TLS implementation MUST NOT automatically resend
   early data unless the negotiated connection selects the same ALPN
   protocol.

#### 4.2.11. Pre-Shared Key Extension

   The "pre_shared_key" extension is used to negotiate the identity of
   the pre-shared key to be used with a given handshake in association
   with PSK key establishment. 
   The "extension_data" field of this extension contains a
   "PreSharedKeyExtension" value:

      struct {
          opaque identity<1..2^16-1>;
          uint32 obfuscated_ticket_age;
      } PskIdentity;

      opaque PskBinderEntry<32..255>;

      struct {
          PskIdentity identities<7..2^16-1>;
          PskBinderEntry binders<33..2^16-1>;
      } OfferedPsks;

      struct {
          select (Handshake.msg_type) {
              case client_hello: OfferedPsks;
              case server_hello: uint16 selected_identity;
          };
      } PreSharedKeyExtension;

   identity:  A label for a key.  For instance, a ticket (as defined in Appendix B.3.4) or a label for a pre-shared key established
      externally.

   obfuscated_ticket_age:  An obfuscated version of the age of the key. Section 4.2.11.1 describes how to form this value for identities
      established via the NewSessionTicket message.  For identities
      established externally, an obfuscated_ticket_age of 0 SHOULD be
      used, and servers MUST ignore the value.

   identities:  A list of the identities that the client is willing to
      negotiate with the server.  If sent alongside the "early_data"
      extension (see Section 4.2.10), the first identity is the one used
      for 0-RTT data.

   binders:  A series of HMAC values, one for each value in the
      identities list and in the same order, computed as described
      below.

   selected_identity:  The server's chosen identity expressed as a
      (0-based) index into the identities in the client's list.

   Each PSK is associated with a single Hash algorithm.  For PSKs
   established via the ticket mechanism (Section 4.6.1), this is the KDF
   Hash algorithm on the connection where the ticket was established.
   For externally established PSKs, the Hash algorithm MUST be set when 
   the PSK is established or default to SHA-256 if no such algorithm is
   defined.  The server MUST ensure that it selects a compatible PSK
   (if any) and cipher suite.

   In TLS versions prior to TLS 1.3, the Server Name Identification
   (SNI) value was intended to be associated with the session (Section 3
   of [RFC6066]), with the server being required to enforce that the SNI
   value associated with the session matches the one specified in the
   resumption handshake.  However, in reality the implementations were
   not consistent on which of two supplied SNI values they would use,
   leading to the consistency requirement being de facto enforced by the
   clients.  In TLS 1.3, the SNI value is always explicitly specified in
   the resumption handshake, and there is no need for the server to
   associate an SNI value with the ticket.  Clients, however, SHOULD
   store the SNI with the PSK to fulfill the requirements of Section 4.6.1.

   Implementor's note: When session resumption is the primary use case
   of PSKs, the most straightforward way to implement the PSK/cipher
   suite matching requirements is to negotiate the cipher suite first
   and then exclude any incompatible PSKs.  Any unknown PSKs (e.g., ones
   not in the PSK database or encrypted with an unknown key) SHOULD
   simply be ignored.  If no acceptable PSKs are found, the server
   SHOULD perform a non-PSK handshake if possible.  If backward
   compatibility is important, client-provided, externally established
   PSKs SHOULD influence cipher suite selection.

   Prior to accepting PSK key establishment, the server MUST validate
   the corresponding binder value (see Section 4.2.11.2 below).  If this
   value is not present or does not validate, the server MUST abort the
   handshake.  Servers SHOULD NOT attempt to validate multiple binders;
   rather, they SHOULD select a single PSK and validate solely the
   binder that corresponds to that PSK.  See Section 8.2 and Appendix E.6 for the security rationale for this requirement.  In
   order to accept PSK key establishment, the server sends a
   "pre_shared_key" extension indicating the selected identity.

   Clients MUST verify that the server's selected_identity is within the
   range supplied by the client, that the server selected a cipher suite
   indicating a Hash associated with the PSK, and that a server
   "key_share" extension is present if required by the ClientHello
   "psk_key_exchange_modes" extension.  If these values are not
   consistent, the client MUST abort the handshake with an
   "illegal_parameter" alert. 
   If the server supplies an "early_data" extension, the client MUST
   verify that the server's selected_identity is 0.  If any other value
   is returned, the client MUST abort the handshake with an
   "illegal_parameter" alert.

   The "pre_shared_key" extension MUST be the last extension in the
   ClientHello (this facilitates implementation as described below).
   Servers MUST check that it is the last extension and otherwise fail
   the handshake with an "illegal_parameter" alert.

##### 4.2.11.1. Ticket Age

   The client's view of the age of a ticket is the time since the
   receipt of the NewSessionTicket message.  Clients MUST NOT attempt to
   use tickets which have ages greater than the "ticket_lifetime" value
   which was provided with the ticket.  The "obfuscated_ticket_age"
   field of each PskIdentity contains an obfuscated version of the
   ticket age formed by taking the age in milliseconds and adding the
   "ticket_age_add" value that was included with the ticket (see Section 4.6.1), modulo 2^32.  This addition prevents passive
   observers from correlating connections unless tickets are reused.
   Note that the "ticket_lifetime" field in the NewSessionTicket message
   is in seconds but the "obfuscated_ticket_age" is in milliseconds.
   Because ticket lifetimes are restricted to a week, 32 bits is enough
   to represent any plausible age, even in milliseconds.

##### 4.2.11.2. PSK Binder

   The PSK binder value forms a binding between a PSK and the current
   handshake, as well as a binding between the handshake in which the
   PSK was generated (if via a NewSessionTicket message) and the current
   handshake.  Each entry in the binders list is computed as an HMAC
   over a transcript hash (see Section 4.4.1) containing a partial
   ClientHello up to and including the PreSharedKeyExtension.identities
   field.  That is, it includes all of the ClientHello but not the
   binders list itself.  The length fields for the message (including
   the overall length, the length of the extensions block, and the
   length of the "pre_shared_key" extension) are all set as if binders
   of the correct lengths were present.

   The PskBinderEntry is computed in the same way as the Finished
   message (Section 4.4.4) but with the BaseKey being the binder_key
   derived via the key schedule from the corresponding PSK which is
   being offered (see Section 7.1). 
   If the handshake includes a HelloRetryRequest, the initial
   ClientHello and HelloRetryRequest are included in the transcript
   along with the new ClientHello.  For instance, if the client sends
   ClientHello1, its binder will be computed over:

      Transcript-Hash(Truncate(ClientHello1))

   Where Truncate() removes the binders list from the ClientHello.

   If the server responds with a HelloRetryRequest and the client then
   sends ClientHello2, its binder will be computed over:

      Transcript-Hash(ClientHello1,
                      HelloRetryRequest,
                      Truncate(ClientHello2))

   The full ClientHello1/ClientHello2 is included in all other handshake
   hash computations.  Note that in the first flight,
   Truncate(ClientHello1) is hashed directly, but in the second flight,
   ClientHello1 is hashed and then reinjected as a "message_hash"
   message, as described in Section 4.4.1.

##### 4.2.11.3. Processing Order

   Clients are permitted to "stream" 0-RTT data until they receive the
   server's Finished, only then sending the EndOfEarlyData message,
   followed by the rest of the handshake.  In order to avoid deadlocks,
   when accepting "early_data", servers MUST process the client's
   ClientHello and then immediately send their flight of messages,
   rather than waiting for the client's EndOfEarlyData message before
   sending its ServerHello.

### 4.3. Server Parameters

   The next two messages from the server, EncryptedExtensions and
   CertificateRequest, contain information from the server that
   determines the rest of the handshake.  These messages are encrypted
   with keys derived from the server_handshake_traffic_secret. 





#### 4.3.1. Encrypted Extensions

   In all handshakes, the server MUST send the EncryptedExtensions
   message immediately after the ServerHello message.  This is the first
   message that is encrypted under keys derived from the
   server_handshake_traffic_secret.

   The EncryptedExtensions message contains extensions that can be
   protected, i.e., any which are not needed to establish the
   cryptographic context but which are not associated with individual
   certificates.  The client MUST check EncryptedExtensions for the
   presence of any forbidden extensions and if any are found MUST abort
   the handshake with an "illegal_parameter" alert.

   Structure of this message:

      struct {
          Extension extensions<0..2^16-1>;
      } EncryptedExtensions;

   extensions:  A list of extensions.  For more information, see the
      table in Section 4.2.

#### 4.3.2. Certificate Request

   A server which is authenticating with a certificate MAY optionally
   request a certificate from the client.  This message, if sent, MUST
   follow EncryptedExtensions.

   Structure of this message:

      struct {
          opaque certificate_request_context<0..2^8-1>;
          Extension extensions<2..2^16-1>;
      } CertificateRequest;
   certificate_request_context:  An opaque string which identifies the
      certificate request and which will be echoed in the client's
      Certificate message.  The certificate_request_context MUST be
      unique within the scope of this connection (thus preventing replay
      of client CertificateVerify messages).  This field SHALL be zero
      length unless used for the post-handshake authentication exchanges
      described in Section 4.6.2.  When requesting post-handshake
      authentication, the server SHOULD make the context unpredictable
      to the client (e.g., by randomly generating it) in order to
      prevent an attacker who has temporary access to the client's
      private key from pre-computing valid CertificateVerify messages.

   extensions:  A set of extensions describing the parameters of the
      certificate being requested.  The "signature_algorithms" extension
      MUST be specified, and other extensions may optionally be included
      if defined for this message.  Clients MUST ignore unrecognized
      extensions.

   In prior versions of TLS, the CertificateRequest message carried a
   list of signature algorithms and certificate authorities which the
   server would accept.  In TLS 1.3, the former is expressed by sending
   the "signature_algorithms" and optionally "signature_algorithms_cert"
   extensions.  The latter is expressed by sending the
   "certificate_authorities" extension (see Section 4.2.4).

   Servers which are authenticating with a PSK MUST NOT send the
   CertificateRequest message in the main handshake, though they MAY
   send it in post-handshake authentication (see Section 4.6.2) provided
   that the client has sent the "post_handshake_auth" extension (see Section 4.2.6).

### 4.4. Authentication Messages

   As discussed in Section 2, TLS generally uses a common set of
   messages for authentication, key confirmation, and handshake
   integrity: Certificate, CertificateVerify, and Finished.  (The PSK
   binders also perform key confirmation, in a similar fashion.)  These
   three messages are always sent as the last messages in their
   handshake flight.  The Certificate and CertificateVerify messages are
   only sent under certain circumstances, as defined below.  The
   Finished message is always sent as part of the Authentication Block.
   These messages are encrypted under keys derived from the
   [sender ]_handshake_traffic_secret. 
   The computations for the Authentication messages all uniformly take
   the following inputs:

   -  The certificate and signing key to be used.

   -  A Handshake Context consisting of the set of messages to be
      included in the transcript hash.

   -  A Base Key to be used to compute a MAC key.

   Based on these inputs, the messages then contain:

   Certificate:  The certificate to be used for authentication, and any
      supporting certificates in the chain.  Note that certificate-based
      client authentication is not available in PSK handshake flows
      (including 0-RTT).

   CertificateVerify:  A signature over the value
      Transcript-Hash(Handshake Context, Certificate).

   Finished:  A MAC over the value Transcript-Hash(Handshake Context,
      Certificate, CertificateVerify) using a MAC key derived from the
      Base Key.

   The following table defines the Handshake Context and MAC Base Key
   for each scenario:

   +-----------+-------------------------+-----------------------------+
   | Mode      | Handshake Context       | Base Key                    |
   +-----------+-------------------------+-----------------------------+
   | Server    | ClientHello ... later   | server_handshake_traffic_   |
   |           | of EncryptedExtensions/ | secret                      |
   |           | CertificateRequest      |                             |
   |           |                         |                             |
   | Client    | ClientHello ... later   | client_handshake_traffic_   |
   |           | of server               | secret                      |
   |           | Finished/EndOfEarlyData |                             |
   |           |                         |                             |
   | Post-     | ClientHello ... client  | client_application_traffic_ |
   | Handshake | Finished +              | secret_N                    |
   |           | CertificateRequest      |                             |
   +-----------+-------------------------+-----------------------------+





#### 4.4.1. The Transcript Hash

   Many of the cryptographic computations in TLS make use of a
   transcript hash.  This value is computed by hashing the concatenation
   of each included handshake message, including the handshake message
   header carrying the handshake message type and length fields, but not
   including record layer headers.  I.e.,

    Transcript-Hash(M1, M2, ... Mn) = Hash(M1 || M2 || ... || Mn)

   As an exception to this general rule, when the server responds to a
   ClientHello with a HelloRetryRequest, the value of ClientHello1 is
   replaced with a special synthetic handshake message of handshake type
   "message_hash" containing Hash(ClientHello1).  I.e.,

  Transcript-Hash(ClientHello1, HelloRetryRequest, ... Mn) =
      Hash(message_hash ||        /* Handshake type */
           00 00 Hash.length  ||  /* Handshake message length (bytes) */
           Hash(ClientHello1) ||  /* Hash of ClientHello1 */
           HelloRetryRequest  || ... || Mn)

   The reason for this construction is to allow the server to do a
   stateless HelloRetryRequest by storing just the hash of ClientHello1
   in the cookie, rather than requiring it to export the entire
   intermediate hash state (see Section 4.2.2).

   For concreteness, the transcript hash is always taken from the
   following sequence of handshake messages, starting at the first
   ClientHello and including only those messages that were sent:
   ClientHello, HelloRetryRequest, ClientHello, ServerHello,
   EncryptedExtensions, server CertificateRequest, server Certificate,
   server CertificateVerify, server Finished, EndOfEarlyData, client
   Certificate, client CertificateVerify, client Finished.

   In general, implementations can implement the transcript by keeping a
   running transcript hash value based on the negotiated hash.  Note,
   however, that subsequent post-handshake authentications do not
   include each other, just the messages through the end of the main
   handshake. 





#### 4.4.2. Certificate

   This message conveys the endpoint's certificate chain to the peer.

   The server MUST send a Certificate message whenever the agreed-upon
   key exchange method uses certificates for authentication (this
   includes all key exchange methods defined in this document
   except PSK).

   The client MUST send a Certificate message if and only if the server
   has requested client authentication via a CertificateRequest message
   (Section 4.3.2).  If the server requests client authentication but no
   suitable certificate is available, the client MUST send a Certificate
   message containing no certificates (i.e., with the "certificate_list"
   field having length 0).  A Finished message MUST be sent regardless
   of whether the Certificate message is empty.

   Structure of this message:

      enum {
          X509(0),
          RawPublicKey(2),
          (255)
      } CertificateType;

      struct {
          select (certificate_type) {
              case RawPublicKey:
                /* From RFC 7250 ASN.1_subjectPublicKeyInfo */
                opaque ASN1_subjectPublicKeyInfo<1..2^24-1>;

              case X509:
                opaque cert_data<1..2^24-1>;
          };
          Extension extensions<0..2^16-1>;
      } CertificateEntry;

      struct {
          opaque certificate_request_context<0..2^8-1>;
          CertificateEntry certificate_list<0..2^24-1>;
      } Certificate;
   certificate_request_context:  If this message is in response to a
      CertificateRequest, the value of certificate_request_context in
      that message.  Otherwise (in the case of server authentication),
      this field SHALL be zero length.

   certificate_list:  A sequence (chain) of CertificateEntry structures,
      each containing a single certificate and set of extensions.

   extensions:  A set of extension values for the CertificateEntry.  The
      "Extension" format is defined in Section 4.2.  Valid extensions
      for server certificates at present include the OCSP Status
      extension [RFC6066] and the SignedCertificateTimestamp extension
      [RFC6962]; future extensions may be defined for this message as
      well.  Extensions in the Certificate message from the server MUST
      correspond to ones from the ClientHello message.  Extensions in
      the Certificate message from the client MUST correspond to
      extensions in the CertificateRequest message from the server.  If
      an extension applies to the entire chain, it SHOULD be included in
      the first CertificateEntry.

   If the corresponding certificate type extension
   ("server_certificate_type" or "client_certificate_type") was not
   negotiated in EncryptedExtensions, or the X.509 certificate type was
   negotiated, then each CertificateEntry contains a DER-encoded X.509
   certificate.  The sender's certificate MUST come in the first
   CertificateEntry in the list.  Each following certificate SHOULD
   directly certify the one immediately preceding it.  Because
   certificate validation requires that trust anchors be distributed
   independently, a certificate that specifies a trust anchor MAY be
   omitted from the chain, provided that supported peers are known to
   possess any omitted certificates.

   Note: Prior to TLS 1.3, "certificate_list" ordering required each
   certificate to certify the one immediately preceding it; however,
   some implementations allowed some flexibility.  Servers sometimes
   send both a current and deprecated intermediate for transitional
   purposes, and others are simply configured incorrectly, but these
   cases can nonetheless be validated properly.  For maximum
   compatibility, all implementations SHOULD be prepared to handle
   potentially extraneous certificates and arbitrary orderings from any
   TLS version, with the exception of the end-entity certificate which
   MUST be first.

   If the RawPublicKey certificate type was negotiated, then the
   certificate_list MUST contain no more than one CertificateEntry,
   which contains an ASN1_subjectPublicKeyInfo value as defined in [RFC7250], Section 3. 
   The OpenPGP certificate type [RFC6091] MUST NOT be used with TLS 1.3.

   The server's certificate_list MUST always be non-empty.  A client
   will send an empty certificate_list if it does not have an
   appropriate certificate to send in response to the server's
   authentication request.

##### 4.4.2.1. OCSP Status and SCT Extensions

   [RFC6066] and [RFC6961] provide extensions to negotiate the server
   sending OCSP responses to the client.  In TLS 1.2 and below, the
   server replies with an empty extension to indicate negotiation of
   this extension and the OCSP information is carried in a
   CertificateStatus message.  In TLS 1.3, the server's OCSP information
   is carried in an extension in the CertificateEntry containing the
   associated certificate.  Specifically, the body of the
   "status_request" extension from the server MUST be a
   CertificateStatus structure as defined in [RFC6066], which is
   interpreted as defined in [RFC6960].

   Note: The status_request_v2 extension [RFC6961] is deprecated.
   TLS 1.3 servers MUST NOT act upon its presence or information in it
   when processing ClientHello messages; in particular, they MUST NOT
   send the status_request_v2 extension in the EncryptedExtensions,
   CertificateRequest, or Certificate messages.  TLS 1.3 servers MUST be
   able to process ClientHello messages that include it, as it MAY be
   sent by clients that wish to use it in earlier protocol versions.

   A server MAY request that a client present an OCSP response with its
   certificate by sending an empty "status_request" extension in its
   CertificateRequest message.  If the client opts to send an OCSP
   response, the body of its "status_request" extension MUST be a
   CertificateStatus structure as defined in [RFC6066].

   Similarly, [RFC6962] provides a mechanism for a server to send a
   Signed Certificate Timestamp (SCT) as an extension in the ServerHello
   in TLS 1.2 and below.  In TLS 1.3, the server's SCT information is
   carried in an extension in the CertificateEntry. 





##### 4.4.2.2. Server Certificate Selection

   The following rules apply to the certificates sent by the server:

   -  The certificate type MUST be X.509v3 [RFC5280], unless explicitly
      negotiated otherwise (e.g., [RFC7250]).

   -  The server's end-entity certificate's public key (and associated
      restrictions) MUST be compatible with the selected authentication
      algorithm from the client's "signature_algorithms" extension
      (currently RSA, ECDSA, or EdDSA).

   -  The certificate MUST allow the key to be used for signing (i.e.,
      the digitalSignature bit MUST be set if the Key Usage extension is
      present) with a signature scheme indicated in the client's
      "signature_algorithms"/"signature_algorithms_cert" extensions (see Section 4.2.3).

   -  The "server_name" [RFC6066] and "certificate_authorities"
      extensions are used to guide certificate selection.  As servers
      MAY require the presence of the "server_name" extension, clients
      SHOULD send this extension, when applicable.

   All certificates provided by the server MUST be signed by a signature
   algorithm advertised by the client if it is able to provide such a
   chain (see Section 4.2.3).  Certificates that are self-signed or
   certificates that are expected to be trust anchors are not validated
   as part of the chain and therefore MAY be signed with any algorithm.

   If the server cannot produce a certificate chain that is signed only
   via the indicated supported algorithms, then it SHOULD continue the
   handshake by sending the client a certificate chain of its choice
   that may include algorithms that are not known to be supported by the
   client.  This fallback chain SHOULD NOT use the deprecated SHA-1 hash
   algorithm in general, but MAY do so if the client's advertisement
   permits it, and MUST NOT do so otherwise.

   If the client cannot construct an acceptable chain using the provided
   certificates and decides to abort the handshake, then it MUST abort
   the handshake with an appropriate certificate-related alert (by
   default, "unsupported_certificate"; see Section 6.2 for more
   information).

   If the server has multiple certificates, it chooses one of them based
   on the above-mentioned criteria (in addition to other criteria, such
   as transport-layer endpoint, local configuration, and preferences). 





##### 4.4.2.3. Client Certificate Selection

   The following rules apply to certificates sent by the client:

   -  The certificate type MUST be X.509v3 [RFC5280], unless explicitly
      negotiated otherwise (e.g., [RFC7250]).

   -  If the "certificate_authorities" extension in the
      CertificateRequest message was present, at least one of the
      certificates in the certificate chain SHOULD be issued by one of
      the listed CAs.

   -  The certificates MUST be signed using an acceptable signature
      algorithm, as described in Section 4.3.2.  Note that this relaxes
      the constraints on certificate-signing algorithms found in prior
      versions of TLS.

   -  If the CertificateRequest message contained a non-empty
      "oid_filters" extension, the end-entity certificate MUST match the
      extension OIDs that are recognized by the client, as described in Section 4.2.5.

##### 4.4.2.4. Receiving a Certificate Message

   In general, detailed certificate validation procedures are out of
   scope for TLS (see [RFC5280]).  This section provides TLS-specific
   requirements.

   If the server supplies an empty Certificate message, the client MUST
   abort the handshake with a "decode_error" alert.

   If the client does not send any certificates (i.e., it sends an empty
   Certificate message), the server MAY at its discretion either
   continue the handshake without client authentication or abort the
   handshake with a "certificate_required" alert.  Also, if some aspect
   of the certificate chain was unacceptable (e.g., it was not signed by
   a known, trusted CA), the server MAY at its discretion either
   continue the handshake (considering the client unauthenticated) or
   abort the handshake.

   Any endpoint receiving any certificate which it would need to
   validate using any signature algorithm using an MD5 hash MUST abort
   the handshake with a "bad_certificate" alert.  SHA-1 is deprecated,
   and it is RECOMMENDED that any endpoint receiving any certificate
   which it would need to validate using any signature algorithm using a
   SHA-1 hash abort the handshake with a "bad_certificate" alert.  For
   clarity, this means that endpoints can accept these algorithms for
   certificates that are self-signed or are trust anchors. 
   All endpoints are RECOMMENDED to transition to SHA-256 or better as
   soon as possible to maintain interoperability with implementations
   currently in the process of phasing out SHA-1 support.

   Note that a certificate containing a key for one signature algorithm
   MAY be signed using a different signature algorithm (for instance, an
   RSA key signed with an ECDSA key).

#### 4.4.3. Certificate Verify

   This message is used to provide explicit proof that an endpoint
   possesses the private key corresponding to its certificate.  The
   CertificateVerify message also provides integrity for the handshake
   up to this point.  Servers MUST send this message when authenticating
   via a certificate.  Clients MUST send this message whenever
   authenticating via a certificate (i.e., when the Certificate message
   is non-empty).  When sent, this message MUST appear immediately after
   the Certificate message and immediately prior to the Finished
   message.

   Structure of this message:

      struct {
          SignatureScheme algorithm;
          opaque signature<0..2^16-1>;
      } CertificateVerify;

   The algorithm field specifies the signature algorithm used (see Section 4.2.3 for the definition of this type).  The signature is a
   digital signature using that algorithm.  The content that is covered
   under the signature is the hash output as described in Section 4.4.1,
   namely:

      Transcript-Hash(Handshake Context, Certificate)

   The digital signature is then computed over the concatenation of:

   -  A string that consists of octet 32 (0x20) repeated 64 times

   -  The context string

   -  A single 0 byte which serves as the separator

   -  The content to be signed 
   This structure is intended to prevent an attack on previous versions
   of TLS in which the ServerKeyExchange format meant that attackers
   could obtain a signature of a message with a chosen 32-byte prefix
   (ClientHello.random).  The initial 64-byte pad clears that prefix
   along with the server-controlled ServerHello.random.

   The context string for a server signature is
   "TLS 1.3, server CertificateVerify".  The context string for a
   client signature is "TLS 1.3, client CertificateVerify".  It is
   used to provide separation between signatures made in different
   contexts, helping against potential cross-protocol attacks.

   For example, if the transcript hash was 32 bytes of 01 (this length
   would make sense for SHA-256), the content covered by the digital
   signature for a server CertificateVerify would be:

      2020202020202020202020202020202020202020202020202020202020202020
      2020202020202020202020202020202020202020202020202020202020202020
      544c5320312e332c207365727665722043657274696669636174655665726966
      79
      00
      0101010101010101010101010101010101010101010101010101010101010101

   On the sender side, the process for computing the signature field of
   the CertificateVerify message takes as input:

   -  The content covered by the digital signature

   -  The private signing key corresponding to the certificate sent in
      the previous message

   If the CertificateVerify message is sent by a server, the signature
   algorithm MUST be one offered in the client's "signature_algorithms"
   extension unless no valid certificate chain can be produced without
   unsupported algorithms (see Section 4.2.3).

   If sent by a client, the signature algorithm used in the signature
   MUST be one of those present in the supported_signature_algorithms
   field of the "signature_algorithms" extension in the
   CertificateRequest message.

   In addition, the signature algorithm MUST be compatible with the key
   in the sender's end-entity certificate.  RSA signatures MUST use an
   RSASSA-PSS algorithm, regardless of whether RSASSA-PKCS1-v1_5
   algorithms appear in "signature_algorithms".  The SHA-1 algorithm
   MUST NOT be used in any signatures of CertificateVerify messages. 
   All SHA-1 signature algorithms in this specification are defined
   solely for use in legacy certificates and are not valid for
   CertificateVerify signatures.

   The receiver of a CertificateVerify message MUST verify the signature
   field.  The verification process takes as input:

   -  The content covered by the digital signature

   -  The public key contained in the end-entity certificate found in
      the associated Certificate message

   -  The digital signature received in the signature field of the
      CertificateVerify message

   If the verification fails, the receiver MUST terminate the handshake
   with a "decrypt_error" alert.

#### 4.4.4. Finished

   The Finished message is the final message in the Authentication
   Block.  It is essential for providing authentication of the handshake
   and of the computed keys.

   Recipients of Finished messages MUST verify that the contents are
   correct and if incorrect MUST terminate the connection with a
   "decrypt_error" alert.

   Once a side has sent its Finished message and has received and
   validated the Finished message from its peer, it may begin to send
   and receive Application Data over the connection.  There are two
   settings in which it is permitted to send data prior to receiving the
   peer's Finished:

   1.  Clients sending 0-RTT data as described in Section 4.2.10.

   2.  Servers MAY send data after sending their first flight, but
       because the handshake is not yet complete, they have no assurance
       of either the peer's identity or its liveness (i.e., the
       ClientHello might have been replayed). 
   The key used to compute the Finished message is computed from the
   Base Key defined in Section 4.4 using HKDF (see Section 7.1).
   Specifically:

   finished_key =
       HKDF-Expand-Label(BaseKey, "finished", "", Hash.length)

   Structure of this message:

      struct {
          opaque verify_data[Hash.length];
      } Finished;

   The verify_data value is computed as follows:

      verify_data =
          HMAC(finished_key,
               Transcript-Hash(Handshake Context,
                               Certificate*, CertificateVerify*))

      * Only included if present.

   HMAC [RFC2104] uses the Hash algorithm for the handshake.  As noted
   above, the HMAC input can generally be implemented by a running hash,
   i.e., just the handshake hash at this point.

   In previous versions of TLS, the verify_data was always 12 octets
   long.  In TLS 1.3, it is the size of the HMAC output for the Hash
   used for the handshake.

   Note: Alerts and any other non-handshake record types are not
   handshake messages and are not included in the hash computations.

   Any records following a Finished message MUST be encrypted under the
   appropriate application traffic key as described in Section 7.2.  In
   particular, this includes any alerts sent by the server in response
   to client Certificate and CertificateVerify messages.

### 4.5. End of Early Data

      struct {} EndOfEarlyData;

   If the server sent an "early_data" extension in EncryptedExtensions,
   the client MUST send an EndOfEarlyData message after receiving the
   server Finished.  If the server does not send an "early_data"
   extension in EncryptedExtensions, then the client MUST NOT send an
   EndOfEarlyData message.  This message indicates that all 0-RTT
   application_data messages, if any, have been transmitted and that the 
   following records are protected under handshake traffic keys.
   Servers MUST NOT send this message, and clients receiving it MUST
   terminate the connection with an "unexpected_message" alert.  This
   message is encrypted under keys derived from the
   client_early_traffic_secret.

### 4.6. Post-Handshake Messages

   TLS also allows other messages to be sent after the main handshake.
   These messages use a handshake content type and are encrypted under
   the appropriate application traffic key.

#### 4.6.1. New Session Ticket Message

   At any time after the server has received the client Finished
   message, it MAY send a NewSessionTicket message.  This message
   creates a unique association between the ticket value and a secret
   PSK derived from the resumption master secret (see Section 7).

   The client MAY use this PSK for future handshakes by including the
   ticket value in the "pre_shared_key" extension in its ClientHello
   (Section 4.2.11).  Servers MAY send multiple tickets on a single
   connection, either immediately after each other or after specific
   events (see Appendix C.4).  For instance, the server might send a new
   ticket after post-handshake authentication in order to encapsulate
   the additional client authentication state.  Multiple tickets are
   useful for clients for a variety of purposes, including:

   -  Opening multiple parallel HTTP connections.

   -  Performing connection racing across interfaces and address
      families via (for example) Happy Eyeballs [RFC8305] or related
      techniques.

   Any ticket MUST only be resumed with a cipher suite that has the same
   KDF hash algorithm as that used to establish the original connection.

   Clients MUST only resume if the new SNI value is valid for the server
   certificate presented in the original session and SHOULD only resume
   if the SNI value matches the one used in the original session.  The
   latter is a performance optimization: normally, there is no reason to
   expect that different servers covered by a single certificate would
   be able to accept each other's tickets; hence, attempting resumption
   in that case would waste a single-use ticket.  If such an indication
   is provided (externally or by any other means), clients MAY resume
   with a different SNI value. 
   On resumption, if reporting an SNI value to the calling application,
   implementations MUST use the value sent in the resumption ClientHello
   rather than the value sent in the previous session.  Note that if a
   server implementation declines all PSK identities with different SNI
   values, these two values are always the same.

   Note: Although the resumption master secret depends on the client's
   second flight, a server which does not request client authentication
   MAY compute the remainder of the transcript independently and then
   send a NewSessionTicket immediately upon sending its Finished rather
   than waiting for the client Finished.  This might be appropriate in
   cases where the client is expected to open multiple TLS connections
   in parallel and would benefit from the reduced overhead of a
   resumption handshake, for example.

      struct {
          uint32 ticket_lifetime;
          uint32 ticket_age_add;
          opaque ticket_nonce<0..255>;
          opaque ticket<1..2^16-1>;
          Extension extensions<0..2^16-2>;
      } NewSessionTicket;

   ticket_lifetime:  Indicates the lifetime in seconds as a 32-bit
      unsigned integer in network byte order from the time of ticket
      issuance.  Servers MUST NOT use any value greater than
      604800 seconds (7 days).  The value of zero indicates that the
      ticket should be discarded immediately.  Clients MUST NOT cache
      tickets for longer than 7 days, regardless of the ticket_lifetime,
      and MAY delete tickets earlier based on local policy.  A server
      MAY treat a ticket as valid for a shorter period of time than what
      is stated in the ticket_lifetime.

   ticket_age_add:  A securely generated, random 32-bit value that is
      used to obscure the age of the ticket that the client includes in
      the "pre_shared_key" extension.  The client-side ticket age is
      added to this value modulo 2^32 to obtain the value that is
      transmitted by the client.  The server MUST generate a fresh value
      for each ticket it sends.

   ticket_nonce:  A per-ticket value that is unique across all tickets
      issued on this connection. 
   ticket:  The value of the ticket to be used as the PSK identity.  The
      ticket itself is an opaque label.  It MAY be either a database
      lookup key or a self-encrypted and self-authenticated value.

   extensions:  A set of extension values for the ticket.  The
      "Extension" format is defined in Section 4.2.  Clients MUST ignore
      unrecognized extensions.

   The sole extension currently defined for NewSessionTicket is
   "early_data", indicating that the ticket may be used to send 0-RTT
   data (Section 4.2.10).  It contains the following value:

   max_early_data_size:  The maximum amount of 0-RTT data that the
      client is allowed to send when using this ticket, in bytes.  Only
      Application Data payload (i.e., plaintext but not padding or the
      inner content type byte) is counted.  A server receiving more than
      max_early_data_size bytes of 0-RTT data SHOULD terminate the
      connection with an "unexpected_message" alert.  Note that servers
      that reject early data due to lack of cryptographic material will
      be unable to differentiate padding from content, so clients
      SHOULD NOT depend on being able to send large quantities of
      padding in early data records.

   The PSK associated with the ticket is computed as:

       HKDF-Expand-Label(resumption_master_secret,
                        "resumption", ticket_nonce, Hash.length)

   Because the ticket_nonce value is distinct for each NewSessionTicket
   message, a different PSK will be derived for each ticket.

   Note that in principle it is possible to continue issuing new tickets
   which indefinitely extend the lifetime of the keying material
   originally derived from an initial non-PSK handshake (which was most
   likely tied to the peer's certificate).  It is RECOMMENDED that
   implementations place limits on the total lifetime of such keying
   material; these limits should take into account the lifetime of the
   peer's certificate, the likelihood of intervening revocation, and the
   time since the peer's online CertificateVerify signature.

#### 4.6.2. Post-Handshake Authentication

   When the client has sent the "post_handshake_auth" extension (see Section 4.2.6), a server MAY request client authentication at any
   time after the handshake has completed by sending a
   CertificateRequest message.  The client MUST respond with the
   appropriate Authentication messages (see Section 4.4).  If the client
   chooses to authenticate, it MUST send Certificate, CertificateVerify,
   and Finished.  If it declines, it MUST send a Certificate message
   containing no certificates followed by Finished.  All of the client's
   messages for a given response MUST appear consecutively on the wire
   with no intervening messages of other types.

   A client that receives a CertificateRequest message without having
   sent the "post_handshake_auth" extension MUST send an
   "unexpected_message" fatal alert.

   Note: Because client authentication could involve prompting the user,
   servers MUST be prepared for some delay, including receiving an
   arbitrary number of other messages between sending the
   CertificateRequest and receiving a response.  In addition, clients
   which receive multiple CertificateRequests in close succession MAY
   respond to them in a different order than they were received (the
   certificate_request_context value allows the server to disambiguate
   the responses).

#### 4.6.3. Key and Initialization Vector Update

   The KeyUpdate handshake message is used to indicate that the sender
   is updating its sending cryptographic keys.  This message can be sent
   by either peer after it has sent a Finished message.  Implementations
   that receive a KeyUpdate message prior to receiving a Finished
   message MUST terminate the connection with an "unexpected_message"
   alert.  After sending a KeyUpdate message, the sender SHALL send all
   its traffic using the next generation of keys, computed as described
   in Section 7.2.  Upon receiving a KeyUpdate, the receiver MUST update
   its receiving keys.

      enum {
          update_not_requested(0), update_requested(1), (255)
      } KeyUpdateRequest;

      struct {
          KeyUpdateRequest request_update;
      } KeyUpdate;

   request_update:  Indicates whether the recipient of the KeyUpdate
      should respond with its own KeyUpdate.  If an implementation
      receives any other value, it MUST terminate the connection with an
      "illegal_parameter" alert.

   If the request_update field is set to "update_requested", then the
   receiver MUST send a KeyUpdate of its own with request_update set to
   "update_not_requested" prior to sending its next Application Data
   record.  This mechanism allows either side to force an update to the
   entire connection, but causes an implementation which receives 
   multiple KeyUpdates while it is silent to respond with a single
   update.  Note that implementations may receive an arbitrary number of
   messages between sending a KeyUpdate with request_update set to
   "update_requested" and receiving the peer's KeyUpdate, because those
   messages may already be in flight.  However, because send and receive
   keys are derived from independent traffic secrets, retaining the
   receive traffic secret does not threaten the forward secrecy of data
   sent before the sender changed keys.

   If implementations independently send their own KeyUpdates with
   request_update set to "update_requested" and they cross in flight,
   then each side will also send a response, with the result that each
   side increments by two generations.

   Both sender and receiver MUST encrypt their KeyUpdate messages with
   the old keys.  Additionally, both sides MUST enforce that a KeyUpdate
   with the old key is received before accepting any messages encrypted
   with the new key.  Failure to do so may allow message truncation
   attacks.

### B.4. Cipher Suites

   A symmetric cipher suite defines the pair of the AEAD algorithm and
   hash algorithm to be used with HKDF.  Cipher suite names follow the
   naming convention:

      CipherSuite TLS_AEAD_HASH = VALUE;

      +-----------+------------------------------------------------+
      | Component | Contents                                       |
      +-----------+------------------------------------------------+
      | TLS       | The string "TLS"                               |
      |           |                                                |
      | AEAD      | The AEAD algorithm used for record protection  |
      |           |                                                |
      | HASH      | The hash algorithm used with HKDF              |
      |           |                                                |
      | VALUE     | The two-byte ID assigned for this cipher suite |
      +-----------+------------------------------------------------+

   This specification defines the following cipher suites for use with
   TLS 1.3.

              +------------------------------+-------------+
              | Description                  | Value       |
              +------------------------------+-------------+
              | TLS_AES_128_GCM_SHA256       | {0x13,0x01} |
              |                              |             |
              | TLS_AES_256_GCM_SHA384       | {0x13,0x02} |
              |                              |             |
              | TLS_CHACHA20_POLY1305_SHA256 | {0x13,0x03} |
              |                              |             |
              | TLS_AES_128_CCM_SHA256       | {0x13,0x04} |
              |                              |             |
              | TLS_AES_128_CCM_8_SHA256     | {0x13,0x05} |
              +------------------------------+-------------+
   The corresponding AEAD algorithms AEAD_AES_128_GCM, AEAD_AES_256_GCM,
   and AEAD_AES_128_CCM are defined in [RFC5116].
   AEAD_CHACHA20_POLY1305 is defined in [RFC8439].  AEAD_AES_128_CCM_8
   is defined in [RFC6655].  The corresponding hash algorithms are
   defined in [SHS ].

   Although TLS 1.3 uses the same cipher suite space as previous
   versions of TLS, TLS 1.3 cipher suites are defined differently, only
   specifying the symmetric ciphers, and cannot be used for TLS 1.2.
   Similarly, cipher suites for TLS 1.2 and lower cannot be used with
   TLS 1.3.

   New cipher suite values are assigned by IANA as described in Section 11.

### C.2. Certificates and Authentication

   Implementations are responsible for verifying the integrity of
   certificates and should generally support certificate revocation
   messages.  Absent a specific indication from an application profile,
   certificates should always be verified to ensure proper signing by a
   trusted certificate authority (CA).  The selection and addition of
   trust anchors should be done very carefully.  Users should be able to
   view information about the certificate and trust anchor.
   Applications SHOULD also enforce minimum and maximum key sizes.  For
   example, certification paths containing keys or signatures weaker
   than 2048-bit RSA or 224-bit ECDSA are not appropriate for secure
   applications.

### C.4. Client Tracking Prevention

   Clients SHOULD NOT reuse a ticket for multiple connections.  Reuse of
   a ticket allows passive observers to correlate different connections.
   Servers that issue tickets SHOULD offer at least as many tickets as
   the number of connections that a client might use; for example, a web
   browser using HTTP/1.1 [RFC7230] might open six connections to a
   server.  Servers SHOULD issue new tickets with every connection.
   This ensures that clients are always able to use a new ticket when
   creating a new connection.

# Referenced Sections from RFC 7250: Using Raw Public Keys in Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS)

The following sections were referenced. Remaining sections are not included.

## 6. Security Considerations

   The transmission of raw public keys, as described in this document,
   provides benefits by lowering the over-the-air transmission overhead
   since raw public keys are naturally smaller than an entire
   certificate.  There are also advantages from a code-size point of
   view for parsing and processing these keys.  The cryptographic
   procedures for associating the public key with the possession of a
   private key also follows standard procedures.

   However, the main security challenge is how to associate the public
   key with a specific entity.  Without a secure binding between
   identifier and key, the protocol will be vulnerable to man-in-the-
   middle attacks.  This document assumes that such binding can be made
   out-of-band, and we list a few examples in Section 1.  DANE [RFC6698]
   offers one such approach.  In order to address these vulnerabilities,
   specifications that make use of the extension need to specify how the
   identifier and public key are bound.  In addition to ensuring the
   binding is done out-of-band, an implementation also needs to check
   the status of that binding.

   If public keys are obtained using DANE, these public keys are
   authenticated via DNSSEC.  Using pre-configured keys is another out-
   of-band method for authenticating raw public keys.  While pre-
   configured keys are not suitable for a generic Web-based e-commerce
   environment, such keys are a reasonable approach for many smart
   object deployments where there is a close relationship between the
   software running on the device and the server-side communication
   endpoint.  Regardless of the chosen mechanism for out-of-band public
   key validation, an assessment of the most suitable approach has to be
   made prior to the start of a deployment to ensure the security of the
   system.

   An attacker might try to influence the handshake exchange to make the
   parties select different certificate types than they would normally
   choose.

   For this attack, an attacker must actively change one or more
   handshake messages.  If this occurs, the client and server will
   compute different values for the handshake message hashes.  As a
   result, the parties will not accept each others' Finished messages.
   Without the master_secret, the attacker cannot repair the Finished
   messages, so the attack will be discovered. 





# Referenced Sections from RFC 5280: Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile

The following sections were referenced. Remaining sections are not included.

### 3.4. Operational Protocols

   Operational protocols are required to deliver certificates and CRLs
   (or status information) to certificate-using client systems.
   Provisions are needed for a variety of different means of certificate
   and CRL delivery, including distribution procedures based on LDAP,
   HTTP, FTP, and X.500.  Operational protocols supporting these
   functions are defined in other PKIX specifications.  These
   specifications may include definitions of message formats and
   procedures for supporting all of the above operational environments,
   including definitions of or references to appropriate MIME content
   types.

##### 4.2.1.6. Subject Alternative Name

   The subject alternative name extension allows identities to be bound
   to the subject of the certificate.  These identities may be included
   in addition to or in place of the identity in the subject field of
   the certificate.  Defined options include an Internet electronic mail 
   address, a DNS name, an IP address, and a Uniform Resource Identifier
   (URI).  Other options exist, including completely local definitions.
   Multiple name forms, and multiple instances of each name form, MAY be
   included.  Whenever such identities are to be bound into a
   certificate, the subject alternative name (or issuer alternative
   name) extension MUST be used; however, a DNS name MAY also be
   represented in the subject field using the domainComponent attribute
   as described in Section 4.1.2.4.  Note that where such names are
   represented in the subject field implementations are not required to
   convert them into DNS names.

   Because the subject alternative name is considered to be definitively
   bound to the public key, all parts of the subject alternative name
   MUST be verified by the CA.

   Further, if the only subject identity included in the certificate is
   an alternative name form (e.g., an electronic mail address), then the
   subject distinguished name MUST be empty (an empty sequence), and the
   subjectAltName extension MUST be present.  If the subject field
   contains an empty sequence, then the issuing CA MUST include a
   subjectAltName extension that is marked as critical.  When including
   the subjectAltName extension in a certificate that has a non-empty
   subject distinguished name, conforming CAs SHOULD mark the
   subjectAltName extension as non-critical.

   When the subjectAltName extension contains an Internet mail address,
   the address MUST be stored in the rfc822Name.  The format of an
   rfc822Name is a "Mailbox" as defined in Section 4.1.2 of [RFC2821].
   A Mailbox has the form "Local-part@Domain".  Note that a Mailbox has
   no phrase (such as a common name) before it, has no comment (text
   surrounded in parentheses) after it, and is not surrounded by "<" and
   ">".  Rules for encoding Internet mail addresses that include
   internationalized domain names are specified in Section 7.5.

   When the subjectAltName extension contains an iPAddress, the address
   MUST be stored in the octet string in "network byte order", as
   specified in [RFC791].  The least significant bit (LSB) of each octet
   is the LSB of the corresponding byte in the network address.  For IP
   version 4, as specified in [RFC791], the octet string MUST contain
   exactly four octets.  For IP version 6, as specified in
   [RFC2460], the octet string MUST contain exactly sixteen octets.

   When the subjectAltName extension contains a domain name system
   label, the domain name MUST be stored in the dNSName (an IA5String).
   The name MUST be in the "preferred name syntax", as specified by Section 3.5 of [RFC1034] and as modified by Section 2.1 of
   [RFC1123].  Note that while uppercase and lowercase letters are
   allowed in domain names, no significance is attached to the case.  In 
   addition, while the string " " is a legal domain name, subjectAltName
   extensions with a dNSName of " " MUST NOT be used.  Finally, the use
   of the DNS representation for Internet mail addresses
   (subscriber.example.com instead of subscriber@example.com) MUST NOT
   be used; such identities are to be encoded as rfc822Name.  Rules for
   encoding internationalized domain names are specified in Section 7.2.

   When the subjectAltName extension contains a URI, the name MUST be
   stored in the uniformResourceIdentifier (an IA5String).  The name
   MUST NOT be a relative URI, and it MUST follow the URI syntax and
   encoding rules specified in [RFC3986].  The name MUST include both a
   scheme (e.g., "http" or "ftp") and a scheme-specific-part.  URIs that
   include an authority ([RFC3986], Section 3.2) MUST include a fully
   qualified domain name or IP address as the host.  Rules for encoding
   Internationalized Resource Identifiers (IRIs) are specified in Section 7.4.

   As specified in [RFC3986], the scheme name is not case-sensitive
   (e.g., "http" is equivalent to "HTTP").  The host part, if present,
   is also not case-sensitive, but other components of the scheme-
   specific-part may be case-sensitive.  Rules for comparing URIs are
   specified in Section 7.4.

   When the subjectAltName extension contains a DN in the directoryName,
   the encoding rules are the same as those specified for the issuer
   field in Section 4.1.2.4.  The DN MUST be unique for each subject
   entity certified by the one CA as defined by the issuer field.  A CA
   MAY issue more than one certificate with the same DN to the same
   subject entity.

   The subjectAltName MAY carry additional name types through the use of
   the otherName field.  The format and semantics of the name are
   indicated through the OBJECT IDENTIFIER in the type-id field.  The
   name itself is conveyed as value field in otherName.  For example,
   Kerberos [RFC4120] format names can be encoded into the otherName,
   using a Kerberos 5 principal name OID and a SEQUENCE of the Realm and
   the PrincipalName.

   Subject alternative names MAY be constrained in the same manner as
   subject distinguished names using the name constraints extension as
   described in Section 4.2.1.10.

   If the subjectAltName extension is present, the sequence MUST contain
   at least one entry.  Unlike the subject field, conforming CAs MUST
   NOT issue certificates with subjectAltNames containing empty
   GeneralName fields.  For example, an rfc822Name is represented as an
   IA5String.  While an empty string is a valid IA5String, such an
   rfc822Name is not permitted by this profile.  The behavior of clients 
   that encounter such a certificate when processing a certification
   path is not defined by this profile.

   Finally, the semantics of subject alternative names that include
   wildcard characters (e.g., as a placeholder for a set of names) are
   not addressed by this specification.  Applications with specific
   requirements MAY use such names, but they must define the semantics.

   id-ce-subjectAltName OBJECT IDENTIFIER ::=  { id-ce 17 }

   SubjectAltName ::= GeneralNames

   GeneralNames ::= SEQUENCE SIZE (1..MAX) OF GeneralName

   GeneralName ::= CHOICE {
        otherName                       [0]     OtherName,
        rfc822Name                      [1]     IA5String,
        dNSName                         [2]     IA5String,
        x400Address                     [3]     ORAddress,
        directoryName                   [4]     Name,
        ediPartyName                    [5]     EDIPartyName,
        uniformResourceIdentifier       [6]     IA5String,
        iPAddress                       [7]     OCTET STRING,
        registeredID                    [8]     OBJECT IDENTIFIER }

   OtherName ::= SEQUENCE {
        type-id    OBJECT IDENTIFIER,
        value      [0] EXPLICIT ANY DEFINED BY type-id }

   EDIPartyName ::= SEQUENCE {
        nameAssigner            [0]     DirectoryString OPTIONAL,
        partyName               [1]     DirectoryString }

### 6.1. Basic Path Validation

   This text describes an algorithm for X.509 path processing.  A
   conforming implementation MUST include an X.509 path processing
   procedure that is functionally equivalent to the external behavior of
   this algorithm.  However, support for some of the certificate
   extensions processed in this algorithm are OPTIONAL for compliant
   implementations.  Clients that do not support these extensions MAY
   omit the corresponding steps in the path validation algorithm.

   For example, clients are not required to support the policy mappings
   extension.  Clients that do not support this extension MAY omit the
   path validation steps where policy mappings are processed.  Note that
   clients MUST reject the certificate if it contains an unsupported
   critical extension.

   While the certificate and CRL profiles specified in Sections 4 and 5   of this document specify values for certificate and CRL fields and
   extensions that are considered to be appropriate for the Internet
   PKI, the algorithm presented in this section is not limited to
   accepting certificates and CRLs that conform to these profiles.
   Therefore, the algorithm only includes checks to verify that the
   certification path is valid according to X.509 and does not include
   checks to verify that the certificates and CRLs conform to this
   profile.  While the algorithm could be extended to include checks for
   conformance to the profiles in Sections 4 and 5, this profile
   RECOMMENDS against including such checks.

   The algorithm presented in this section validates the certificate
   with respect to the current date and time.  A conforming
   implementation MAY also support validation with respect to some point
   in the past.  Note that mechanisms are not available for validating a
   certificate with respect to a time outside the certificate validity
   period. 
   The trust anchor is an input to the algorithm.  There is no
   requirement that the same trust anchor be used to validate all
   certification paths.  Different trust anchors MAY be used to validate
   different paths, as discussed further in Section 6.2.

   The primary goal of path validation is to verify the binding between
   a subject distinguished name or a subject alternative name and
   subject public key, as represented in the target certificate, based
   on the public key of the trust anchor.  In most cases, the target
   certificate will be an end entity certificate, but the target
   certificate may be a CA certificate as long as the subject public key
   is to be used for a purpose other than verifying the signature on a
   public key certificate.  Verifying the binding between the name and
   subject public key requires obtaining a sequence of certificates that
   support that binding.  The procedure performed to obtain this
   sequence of certificates is outside the scope of this specification.

   To meet this goal, the path validation process verifies, among other
   things, that a prospective certification path (a sequence of n
   certificates) satisfies the following conditions:

      (a)  for all x in {1, ..., n-1}, the subject of certificate x is
           the issuer of certificate x+1;

      (b)  certificate 1 is issued by the trust anchor;

      (c)  certificate n is the certificate to be validated (i.e., the
           target certificate); and

      (d)  for all x in {1, ..., n}, the certificate was valid at the
           time in question.

   A certificate MUST NOT appear more than once in a prospective
   certification path.

   When the trust anchor is provided in the form of a self-signed
   certificate, this self-signed certificate is not included as part of
   the prospective certification path.  Information about trust anchors
   is provided as inputs to the certification path validation algorithm
   (Section 6.1.1).

   A particular certification path may not, however, be appropriate for
   all applications.  Therefore, an application MAY augment this
   algorithm to further limit the set of valid paths.  The path
   validation process also determines the set of certificate policies
   that are valid for this path, based on the certificate policies
   extension, policy mappings extension, policy constraints extension,
   and inhibit anyPolicy extension.  To achieve this, the path 
   validation algorithm constructs a valid policy tree.  If the set of
   certificate policies that are valid for this path is not empty, then
   the result will be a valid policy tree of depth n, otherwise the
   result will be a null valid policy tree.

   A certificate is self-issued if the same DN appears in the subject
   and issuer fields (the two DNs are the same if they match according
   to the rules specified in Section 7.1).  In general, the issuer and
   subject of the certificates that make up a path are different for
   each certificate.  However, a CA may issue a certificate to itself to
   support key rollover or changes in certificate policies.  These
   self-issued certificates are not counted when evaluating path length
   or name constraints.

   This section presents the algorithm in four basic steps: (1)
   initialization, (2) basic certificate processing, (3) preparation for
   the next certificate, and (4) wrap-up.  Steps (1) and (4) are
   performed exactly once.  Step (2) is performed for all certificates
   in the path.  Step (3) is performed for all certificates in the path
   except the final certificate.  Figure 2 provides a high-level
   flowchart of this algorithm. 
                           +-------+
                           | START |
                           +-------+
                               |
                               V
                       +----------------+
                       | Initialization |
                       +----------------+
                               |
                               +<--------------------+
                               |                     |
                               V                     |
                       +----------------+            |
                       |  Process Cert  |            |
                       +----------------+            |
                               |                     |
                               V                     |
                       +================+            |
                       |  IF Last Cert  |            |
                       |    in Path     |            |
                       +================+            |
                         |            |              |
                    THEN |            | ELSE         |
                         V            V              |
              +----------------+ +----------------+  |
              |    Wrap up     | |  Prepare for   |  |
              +----------------+ |   Next Cert    |  |
                      |          +----------------+  |
                      V               |              |
                  +-------+           +--------------+
                  | STOP  |
                  +-------+

         Figure 2.  Certification Path Processing Flowchart

#### 6.1.1. Inputs

   This algorithm assumes that the following nine inputs are provided to
   the path processing logic:

      (a)  a prospective certification path of length n.

      (b)  the current date/time. 
      (c)  user-initial-policy-set:  A set of certificate policy
           identifiers naming the policies that are acceptable to the
           certificate user.  The user-initial-policy-set contains the
           special value any-policy if the user is not concerned about
           certificate policy.

      (d)  trust anchor information, describing a CA that serves as a
           trust anchor for the certification path.  The trust anchor
           information includes:

         (1)  the trusted issuer name,

         (2)  the trusted public key algorithm,

         (3)  the trusted public key, and

         (4)  optionally, the trusted public key parameters associated
              with the public key.

      The trust anchor information may be provided to the path
      processing procedure in the form of a self-signed certificate.
      When the trust anchor information is provided in the form of a
      certificate, the name in the subject field is used as the trusted
      issuer name and the contents of the subjectPublicKeyInfo field is
      used as the source of the trusted public key algorithm and the
      trusted public key.  The trust anchor information is trusted
      because it was delivered to the path processing procedure by some
      trustworthy out-of-band procedure.  If the trusted public key
      algorithm requires parameters, then the parameters are provided
      along with the trusted public key.

      (e)  initial-policy-mapping-inhibit, which indicates if policy
           mapping is allowed in the certification path.

      (f)  initial-explicit-policy, which indicates if the path must be
           valid for at least one of the certificate policies in the
           user-initial-policy-set.

      (g)  initial-any-policy-inhibit, which indicates whether the
           anyPolicy OID should be processed if it is included in a
           certificate.

      (h)  initial-permitted-subtrees, which indicates for each name
           type (e.g., X.500 distinguished names, email addresses, or IP
           addresses) a set of subtrees within which all subject names
           in every certificate in the certification path MUST fall.
           The initial-permitted-subtrees input includes a set for each
           name type.  For each name type, the set may consist of a 
           single subtree that includes all names of that name type or
           one or more subtrees that each specifies a subset of the
           names of that name type, or the set may be empty.  If the set
           for a name type is empty, then the certification path will be
           considered invalid if any certificate in the certification
           path includes a name of that name type.

      (i)  initial-excluded-subtrees, which indicates for each name type
           (e.g., X.500 distinguished names, email addresses, or IP
           addresses) a set of subtrees within which no subject name in
           any certificate in the certification path may fall.  The
           initial-excluded-subtrees input includes a set for each name
           type.  For each name type, the set may be empty or may
           consist of one or more subtrees that each specifies a subset
           of the names of that name type.  If the set for a name type
           is empty, then no names of that name type are excluded.

   Conforming implementations are not required to support the setting of
   all of these inputs.  For example, a conforming implementation may be
   designed to validate all certification paths using a value of FALSE
   for initial-any-policy-inhibit.

#### 6.1.2. Initialization

   This initialization phase establishes eleven state variables based
   upon the nine inputs:

      (a)  valid_policy_tree:  A tree of certificate policies with their
           optional qualifiers; each of the leaves of the tree
           represents a valid policy at this stage in the certification
           path validation.  If valid policies exist at this stage in
           the certification path validation, the depth of the tree is
           equal to the number of certificates in the chain that have
           been processed.  If valid policies do not exist at this stage
           in the certification path validation, the tree is set to
           NULL.  Once the tree is set to NULL, policy processing
           ceases.

           Each node in the valid_policy_tree includes three data
           objects: the valid policy, a set of associated policy
           qualifiers, and a set of one or more expected policy values.
           If the node is at depth x, the components of the node have
           the following semantics:

         (1)  The valid_policy is a single policy OID representing a
              valid policy for the path of length x. 
         (2)  The qualifier_set is a set of policy qualifiers associated
              with the valid policy in certificate x.

         (3)  The expected_policy_set contains one or more policy OIDs
              that would satisfy this policy in the certificate x+1.

      The initial value of the valid_policy_tree is a single node with
      valid_policy anyPolicy, an empty qualifier_set, and an
      expected_policy_set with the single value anyPolicy.  This node is
      considered to be at depth zero.

      Figure 3 is a graphic representation of the initial state of the
      valid_policy_tree.  Additional figures will use this format to
      describe changes in the valid_policy_tree during path processing.

              +----------------+
              |   anyPolicy    |   <---- valid_policy
              +----------------+
              |       {}       |   <---- qualifier_set
              +----------------+
              |  {anyPolicy}   |   <---- expected_policy_set
              +----------------+

      Figure 3.  Initial Value of the valid_policy_tree State Variable

      (b)  permitted_subtrees:  a set of root names for each name type
           (e.g., X.500 distinguished names, email addresses, or IP
           addresses) defining a set of subtrees within which all
           subject names in subsequent certificates in the certification
           path MUST fall.  This variable includes a set for each name
           type, and the initial value is initial-permitted-subtrees.

      (c)  excluded_subtrees:  a set of root names for each name type
           (e.g., X.500 distinguished names, email addresses, or IP
           addresses) defining a set of subtrees within which no subject
           name in subsequent certificates in the certification path may
           fall.  This variable includes a set for each name type, and
           the initial value is initial-excluded-subtrees.

      (d)  explicit_policy:  an integer that indicates if a non-NULL
           valid_policy_tree is required.  The integer indicates the
           number of non-self-issued certificates to be processed before
           this requirement is imposed.  Once set, this variable may be
           decreased, but may not be increased.  That is, if a
           certificate in the path requires a non-NULL
           valid_policy_tree, a later certificate cannot remove this
           requirement.  If initial-explicit-policy is set, then the
           initial value is 0, otherwise the initial value is n+1. 
      (e)  inhibit_anyPolicy:  an integer that indicates whether the
           anyPolicy policy identifier is considered a match.  The
           integer indicates the number of non-self-issued certificates
           to be processed before the anyPolicy OID, if asserted in a
           certificate other than an intermediate self-issued
           certificate, is ignored.  Once set, this variable may be
           decreased, but may not be increased.  That is, if a
           certificate in the path inhibits processing of anyPolicy, a
           later certificate cannot permit it.  If initial-any-policy-
           inhibit is set, then the initial value is 0, otherwise the
           initial value is n+1.

      (f)  policy_mapping:  an integer that indicates if policy mapping
           is permitted.  The integer indicates the number of non-self-
           issued certificates to be processed before policy mapping is
           inhibited.  Once set, this variable may be decreased, but may
           not be increased.  That is, if a certificate in the path
           specifies that policy mapping is not permitted, it cannot be
           overridden by a later certificate.  If initial-policy-
           mapping-inhibit is set, then the initial value is 0,
           otherwise the initial value is n+1.

      (g)  working_public_key_algorithm:  the digital signature
           algorithm used to verify the signature of a certificate.  The
           working_public_key_algorithm is initialized from the trusted
           public key algorithm provided in the trust anchor
           information.

      (h)  working_public_key:  the public key used to verify the
           signature of a certificate.  The working_public_key is
           initialized from the trusted public key provided in the trust
           anchor information.

      (i)  working_public_key_parameters:  parameters associated with
           the current public key that may be required to verify a
           signature (depending upon the algorithm).  The
           working_public_key_parameters variable is initialized from
           the trusted public key parameters provided in the trust
           anchor information.

      (j)  working_issuer_name:  the issuer distinguished name expected
           in the next certificate in the chain.  The
           working_issuer_name is initialized to the trusted issuer name
           provided in the trust anchor information. 
      (k)  max_path_length:  this integer is initialized to n, is
           decremented for each non-self-issued certificate in the path,
           and may be reduced to the value in the path length constraint
           field within the basic constraints extension of a CA
           certificate.

   Upon completion of the initialization steps, perform the basic
   certificate processing steps specified in 6.1.3.

#### 6.1.3. Basic Certificate Processing

   The basic path processing actions to be performed for certificate i
   (for all i in [1..n]) are listed below.

      (a)  Verify the basic certificate information.  The certificate
           MUST satisfy each of the following:

         (1)  The signature on the certificate can be verified using
              working_public_key_algorithm, the working_public_key, and
              the working_public_key_parameters.

         (2)  The certificate validity period includes the current time.

         (3)  At the current time, the certificate is not revoked.  This
              may be determined by obtaining the appropriate CRL
              (Section 6.3), by status information, or by out-of-band
              mechanisms.

         (4)  The certificate issuer name is the working_issuer_name.

      (b)  If certificate i is self-issued and it is not the final
           certificate in the path, skip this step for certificate i.
           Otherwise, verify that the subject name is within one of the
           permitted_subtrees for X.500 distinguished names, and verify
           that each of the alternative names in the subjectAltName
           extension (critical or non-critical) is within one of the
           permitted_subtrees for that name type.

      (c)  If certificate i is self-issued and it is not the final
           certificate in the path, skip this step for certificate i.
           Otherwise, verify that the subject name is not within any of
           the excluded_subtrees for X.500 distinguished names, and
           verify that each of the alternative names in the
           subjectAltName extension (critical or non-critical) is not
           within any of the excluded_subtrees for that name type. 
      (d)  If the certificate policies extension is present in the
           certificate and the valid_policy_tree is not NULL, process
           the policy information by performing the following steps in
           order:

         (1)  For each policy P not equal to anyPolicy in the
              certificate policies extension, let P-OID denote the OID
              for policy P and P-Q denote the qualifier set for policy
              P.  Perform the following steps in order:

            (i)   For each node of depth i-1 in the valid_policy_tree
                  where P-OID is in the expected_policy_set, create a
                  child node as follows: set the valid_policy to P-OID,
                  set the qualifier_set to P-Q, and set the
                  expected_policy_set to
                  {P-OID}.

                  For example, consider a valid_policy_tree with a node
                  of depth i-1 where the expected_policy_set is {Gold,
                  White}.  Assume the certificate policies Gold and
                  Silver appear in the certificate policies extension of
                  certificate i.  The Gold policy is matched, but the
                  Silver policy is not.  This rule will generate a child
                  node of depth i for the Gold policy.  The result is
                  shown as Figure 4.

                             +-----------------+
                             |       Red       |
                             +-----------------+
                             |       {}        |
                             +-----------------+   node of depth i-1
                             |  {Gold, White}  |
                             +-----------------+
                                      |
                                      |
                                      |
                                      V
                             +-----------------+
                             |      Gold       |
                             +-----------------+
                             |       {}        |
                             +-----------------+   node of depth i
                             |     {Gold}      |
                             +-----------------+

                    Figure 4.  Processing an Exact Match 
            (ii)  If there was no match in step (i) and the
                  valid_policy_tree includes a node of depth i-1 with
                  the valid_policy anyPolicy, generate a child node with
                  the following values: set the valid_policy to P-OID,
                  set the qualifier_set to P-Q, and set the
                  expected_policy_set to  {P-OID}.

                  For example, consider a valid_policy_tree with a node
                  of depth i-1 where the valid_policy is anyPolicy.
                  Assume the certificate policies Gold and Silver appear
                  in the certificate policies extension of certificate
                  i.  The Gold policy does not have a qualifier, but the
                  Silver policy has the qualifier Q-Silver.  If Gold and
                  Silver were not matched in (i) above, this rule will
                  generate two child nodes of depth i, one for each
                  policy.  The result is shown as Figure 5.

                                   +-----------------+
                                   |    anyPolicy    |
                                   +-----------------+
                                   |       {}        |
                                   +-----------------+ node of depth i-1
                                   |   {anyPolicy}   |
                                   +-----------------+
                                      /           \
                                     /             \
                                    /               \
                                   /                 \
                     +-----------------+          +-----------------+
                     |      Gold       |          |     Silver      |
                     +-----------------+          +-----------------+
                     |       {}        |          |   {Q-Silver}    |
                     +-----------------+ nodes of +-----------------+
                     |     {Gold}      | depth i  |    {Silver}     |
                     +-----------------+          +-----------------+

                  Figure 5.  Processing Unmatched Policies when a
                  Leaf Node Specifies anyPolicy

         (2)  If the certificate policies extension includes the policy
              anyPolicy with the qualifier set AP-Q and either (a)
              inhibit_anyPolicy is greater than 0 or (b) i<n and the
              certificate is self-issued, then:

              For each node in the valid_policy_tree of depth i-1, for
              each value in the expected_policy_set (including
              anyPolicy) that does not appear in a child node, create a
              child node with the following values: set the valid_policy 
              to the value from the expected_policy_set in the parent
              node, set the qualifier_set to AP-Q, and set the
              expected_policy_set to the value in the valid_policy from
              this node.

              For example, consider a valid_policy_tree with a node of
              depth i-1 where the expected_policy_set is {Gold, Silver}.
              Assume anyPolicy appears in the certificate policies
              extension of certificate i with no policy qualifiers, but
              Gold and Silver do not appear.  This rule will generate
              two child nodes of depth i, one for each policy.  The
              result is shown below as Figure 6.

                               +-----------------+
                               |      Red        |
                               +-----------------+
                               |       {}        |
                               +-----------------+ node of depth i-1
                               |  {Gold, Silver} |
                               +-----------------+
                                  /           \
                                 /             \
                                /               \
                               /                 \
                 +-----------------+          +-----------------+
                 |      Gold       |          |     Silver      |
                 +-----------------+          +-----------------+
                 |       {}        |          |       {}        |
                 +-----------------+ nodes of +-----------------+
                 |     {Gold}      | depth i  |    {Silver}     |
                 +-----------------+          +-----------------+

              Figure 6.  Processing Unmatched Policies When the
              Certificate Policies Extension Specifies anyPolicy

         (3)  If there is a node in the valid_policy_tree of depth i-1
              or less without any child nodes, delete that node.  Repeat
              this step until there are no nodes of depth i-1 or less
              without children.

              For example, consider the valid_policy_tree shown in
              Figure 7 below.  The two nodes at depth i-1 that are
              marked with an 'X' have no children, and they are deleted.
              Applying this rule to the resulting tree will cause the
              node at depth i-2 that is marked with a 'Y' to be deleted.
              In the resulting tree, there are no nodes of depth i-1 or
              less without children, and this step is complete. 
      (e)  If the certificate policies extension is not present, set the
           valid_policy_tree to NULL.

      (f)  Verify that either explicit_policy is greater than 0 or the
           valid_policy_tree is not equal to NULL;

   If any of steps (a), (b), (c), or (f) fails, the procedure
   terminates, returning a failure indication and an appropriate reason.

   If i is not equal to n, continue by performing the preparatory steps
   listed in Section 6.1.4.  If i is equal to n, perform the wrap-up
   steps listed in Section 6.1.5.

                                 +-----------+
                                 |           | node of depth i-3
                                 +-----------+
                                 /     |     \
                                /      |      \
                               /       |       \
                   +-----------+ +-----------+ +-----------+
                   |           | |           | |     Y     | nodes of
                   +-----------+ +-----------+ +-----------+ depth i-2
                   /   \               |             |
                  /     \              |             |
                 /       \             |             |
      +-----------+ +-----------+ +-----------+ +-----------+ nodes of
      |           | |     X     | |           | |    X      |  depth
      +-----------+ +-----------+ +-----------+ +-----------+   i-1
            |                      /    |    \
            |                     /     |     \
            |                    /      |      \
      +-----------+ +-----------+ +-----------+ +-----------+ nodes of
      |           | |           | |           | |           |  depth
      +-----------+ +-----------+ +-----------+ +-----------+   i

             Figure 7.  Pruning the valid_policy_tree

#### 6.1.4. Preparation for Certificate i+1

      To prepare for processing of certificate i+1, perform the
      following steps for certificate i:

      (a)  If a policy mappings extension is present, verify that the
           special value anyPolicy does not appear as an
           issuerDomainPolicy or a subjectDomainPolicy.

      (b)  If a policy mappings extension is present, then for each
           issuerDomainPolicy ID-P in the policy mappings extension:
         (1)  If the policy_mapping variable is greater than 0, for each
              node in the valid_policy_tree of depth i where ID-P is the
              valid_policy, set expected_policy_set to the set of
              subjectDomainPolicy values that are specified as
              equivalent to ID-P by the policy mappings extension.

              If no node of depth i in the valid_policy_tree has a
              valid_policy of ID-P but there is a node of depth i with a
              valid_policy of anyPolicy, then generate a child node of
              the node of depth i-1 that has a valid_policy of anyPolicy
              as follows:

            (i)    set the valid_policy to ID-P;

            (ii)   set the qualifier_set to the qualifier set of the
                   policy anyPolicy in the certificate policies
                   extension of certificate i; and

            (iii)  set the expected_policy_set to the set of
                   subjectDomainPolicy values that are specified as
                   equivalent to ID-P by the policy mappings extension.

         (2)  If the policy_mapping variable is equal to 0:

            (i)    delete each node of depth i in the valid_policy_tree
                   where ID-P is the valid_policy.

            (ii)   If there is a node in the valid_policy_tree of depth
                   i-1 or less without any child nodes, delete that
                   node.  Repeat this step until there are no nodes of
                   depth i-1 or less without children.

      (c)  Assign the certificate subject name to working_issuer_name.

      (d)  Assign the certificate subjectPublicKey to
           working_public_key.

      (e)  If the subjectPublicKeyInfo field of the certificate contains
           an algorithm field with non-null parameters, assign the
           parameters to the working_public_key_parameters variable.

           If the subjectPublicKeyInfo field of the certificate contains
           an algorithm field with null parameters or parameters are
           omitted, compare the certificate subjectPublicKey algorithm
           to the working_public_key_algorithm.  If the certificate
           subjectPublicKey algorithm and the
           working_public_key_algorithm are different, set the
           working_public_key_parameters to null. 
      (f)  Assign the certificate subjectPublicKey algorithm to the
           working_public_key_algorithm variable.

      (g)  If a name constraints extension is included in the
           certificate, modify the permitted_subtrees and
           excluded_subtrees state variables as follows:

         (1)  If permittedSubtrees is present in the certificate, set
              the permitted_subtrees state variable to the intersection
              of its previous value and the value indicated in the
              extension field.  If permittedSubtrees does not include a
              particular name type, the permitted_subtrees state
              variable is unchanged for that name type.  For example,
              the intersection of example.com and foo.example.com is
              foo.example.com.  And the intersection of example.com and
              example.net is the empty set.

         (2)  If excludedSubtrees is present in the certificate, set the
              excluded_subtrees state variable to the union of its
              previous value and the value indicated in the extension
              field.  If excludedSubtrees does not include a particular
              name type, the excluded_subtrees state variable is
              unchanged for that name type.  For example, the union of
              the name spaces example.com and foo.example.com is
              example.com.  And the union of example.com and example.net
              is both name spaces.

      (h)  If certificate i is not self-issued:

         (1)  If explicit_policy is not 0, decrement explicit_policy by
              1.

         (2)  If policy_mapping is not 0, decrement policy_mapping by 1.

         (3)  If inhibit_anyPolicy is not 0, decrement inhibit_anyPolicy
              by 1.

      (i)  If a policy constraints extension is included in the
           certificate, modify the explicit_policy and policy_mapping
           state variables as follows:

         (1)  If requireExplicitPolicy is present and is less than
              explicit_policy, set explicit_policy to the value of
              requireExplicitPolicy.

         (2)  If inhibitPolicyMapping is present and is less than
              policy_mapping, set policy_mapping to the value of
              inhibitPolicyMapping. 
      (j)  If the inhibitAnyPolicy extension is included in the
           certificate and is less than inhibit_anyPolicy, set
           inhibit_anyPolicy to the value of inhibitAnyPolicy.

      (k)  If certificate i is a version 3 certificate, verify that the
           basicConstraints extension is present and that cA is set to
           TRUE.  (If certificate i is a version 1 or version 2
           certificate, then the application MUST either verify that
           certificate i is a CA certificate through out-of-band means
           or reject the certificate.  Conforming implementations may
           choose to reject all version 1 and version 2 intermediate
           certificates.)

      (l)  If the certificate was not self-issued, verify that
           max_path_length is greater than zero and decrement
           max_path_length by 1.

      (m)  If pathLenConstraint is present in the certificate and is
           less than max_path_length, set max_path_length to the value
           of pathLenConstraint.

      (n)  If a key usage extension is present, verify that the
           keyCertSign bit is set.

      (o)  Recognize and process any other critical extension present in
           the certificate.  Process any other recognized non-critical
           extension present in the certificate that is relevant to path
           processing.

   If check (a), (k), (l), (n), or (o) fails, the procedure terminates,
   returning a failure indication and an appropriate reason.

   If (a), (k), (l), (n), and (o) have completed successfully, increment
   i and perform the basic certificate processing specified in Section 
   6.1.3.

#### 6.1.5. Wrap-Up Procedure

   To complete the processing of the target certificate, perform the
   following steps for certificate n:

      (a)  If explicit_policy is not 0, decrement explicit_policy by 1.

      (b)  If a policy constraints extension is included in the
           certificate and requireExplicitPolicy is present and has a
           value of 0, set the explicit_policy state variable to 0. 
      (c)  Assign the certificate subjectPublicKey to
           working_public_key.

      (d)  If the subjectPublicKeyInfo field of the certificate contains
           an algorithm field with non-null parameters, assign the
           parameters to the working_public_key_parameters variable.

           If the subjectPublicKeyInfo field of the certificate contains
           an algorithm field with null parameters or parameters are
           omitted, compare the certificate subjectPublicKey algorithm
           to the working_public_key_algorithm.  If the certificate
           subjectPublicKey algorithm and the
           working_public_key_algorithm are different, set the
           working_public_key_parameters to null.

      (e)  Assign the certificate subjectPublicKey algorithm to the
           working_public_key_algorithm variable.

      (f)  Recognize and process any other critical extension present in
           the certificate n.  Process any other recognized non-critical
           extension present in certificate n that is relevant to path
           processing.

      (g)  Calculate the intersection of the valid_policy_tree and the
           user-initial-policy-set, as follows:

         (i)    If the valid_policy_tree is NULL, the intersection is
                NULL.

         (ii)   If the valid_policy_tree is not NULL and the user-
                initial-policy-set is any-policy, the intersection is
                the entire valid_policy_tree.

         (iii)  If the valid_policy_tree is not NULL and the user-
                initial-policy-set is not any-policy, calculate the
                intersection of the valid_policy_tree and the user-
                initial-policy-set as follows:

             1.  Determine the set of policy nodes whose parent nodes
                 have a valid_policy of anyPolicy.  This is the
                 valid_policy_node_set.

             2.  If the valid_policy of any node in the
                 valid_policy_node_set is not in the user-initial-
                 policy-set and is not anyPolicy, delete this node and
                 all its children. 
             3.  If the valid_policy_tree includes a node of depth n
                 with the valid_policy anyPolicy and the user-initial-
                 policy-set is not any-policy, perform the following
                 steps:

               a.  Set P-Q to the qualifier_set in the node of depth n
                   with valid_policy anyPolicy.

               b.  For each P-OID in the user-initial-policy-set that is
                   not the valid_policy of a node in the
                   valid_policy_node_set, create a child node whose
                   parent is the node of depth n-1 with the valid_policy
                   anyPolicy.  Set the values in the child node as
                   follows: set the valid_policy to P-OID, set the
                   qualifier_set to P-Q, and set the expected_policy_set
                   to {P-OID}.

               c.  Delete the node of depth n with the valid_policy
                   anyPolicy.

             4.  If there is a node in the valid_policy_tree of depth
                 n-1 or less without any child nodes, delete that node.
                 Repeat this step until there are no nodes of depth n-1
                 or less without children.

   If either (1) the value of explicit_policy variable is greater than
   zero or (2) the valid_policy_tree is not NULL, then path processing
   has succeeded.

#### 6.1.6. Outputs

   If path processing succeeds, the procedure terminates, returning a
   success indication together with final value of the
   valid_policy_tree, the working_public_key, the
   working_public_key_algorithm, and the working_public_key_parameters.

## A. Pseudo-ASN.1 Structures and OIDs

   This appendix describes data objects used by conforming PKI
   components in an "ASN.1-like" syntax.  This syntax is a hybrid of the
   1988 and 1993 ASN.1 syntaxes.  The 1988 ASN.1 syntax is augmented
   with 1993 UNIVERSAL Types UniversalString, BMPString, and UTF8String.

   The ASN.1 syntax does not permit the inclusion of type statements in
   the ASN.1 module, and the 1993 ASN.1 standard does not permit use of
   the new UNIVERSAL types in modules using the 1988 syntax.  As a
   result, this module does not conform to either version of the ASN.1
   standard.

   This appendix may be converted into 1988 ASN.1 by replacing the
   definitions for the UNIVERSAL Types with the 1988 catch-all "ANY".

### A.1. Explicitly Tagged Module, 1988 Syntax

PKIX1Explicit88 { iso(1) identified-organization(3) dod(6) internet(1)
  security(5) mechanisms(5) pkix(7) id-mod(0) id-pkix1-explicit(18) }

DEFINITIONS EXPLICIT TAGS ::=

BEGIN

-- EXPORTS ALL --

-- IMPORTS NONE --

-- UNIVERSAL Types defined in 1993 and 1998 ASN.1
-- and required by this specification

UniversalString ::= [UNIVERSAL 28] IMPLICIT OCTET STRING
        -- UniversalString is defined in ASN.1:1993

BMPString ::= [UNIVERSAL 30] IMPLICIT OCTET STRING
      -- BMPString is the subtype of UniversalString and models
      -- the Basic Multilingual Plane of ISO/IEC 10646

UTF8String ::= [UNIVERSAL 12] IMPLICIT OCTET STRING
      -- The content of this type conforms to RFC 3629.

-- PKIX specific OIDs

id-pkix  OBJECT IDENTIFIER  ::=
         { iso(1) identified-organization(3) dod(6) internet(1)
                    security(5) mechanisms(5) pkix(7) }
-- PKIX arcs

id-pe OBJECT IDENTIFIER ::= { id-pkix 1 }
        -- arc for private certificate extensions
id-qt OBJECT IDENTIFIER ::= { id-pkix 2 }
        -- arc for policy qualifier types
id-kp OBJECT IDENTIFIER ::= { id-pkix 3 }
        -- arc for extended key purpose OIDS
id-ad OBJECT IDENTIFIER ::= { id-pkix 48 }
        -- arc for access descriptors

-- policyQualifierIds for Internet policy qualifiers

id-qt-cps      OBJECT IDENTIFIER ::=  { id-qt 1 }
      -- OID for CPS qualifier
id-qt-unotice  OBJECT IDENTIFIER ::=  { id-qt 2 }
      -- OID for user notice qualifier

-- access descriptor definitions

id-ad-ocsp         OBJECT IDENTIFIER ::= { id-ad 1 }
id-ad-caIssuers    OBJECT IDENTIFIER ::= { id-ad 2 }
id-ad-timeStamping OBJECT IDENTIFIER ::= { id-ad 3 }
id-ad-caRepository OBJECT IDENTIFIER ::= { id-ad 5 }

-- attribute data types

Attribute               ::= SEQUENCE {
      type             AttributeType,
      values    SET OF AttributeValue }
            -- at least one value is required

AttributeType           ::= OBJECT IDENTIFIER

AttributeValue          ::= ANY -- DEFINED BY AttributeType

AttributeTypeAndValue   ::= SEQUENCE {
        type    AttributeType,
        value   AttributeValue }

-- suggested naming attributes: Definition of the following
--   information object set may be augmented to meet local
--   requirements.  Note that deleting members of the set may
--   prevent interoperability with conforming implementations.
-- presented in pairs: the AttributeType followed by the
--   type definition for the corresponding AttributeValue 
-- Arc for standard naming attributes

id-at OBJECT IDENTIFIER ::= { joint-iso-ccitt(2) ds(5) 4 }

-- Naming attributes of type X520name

id-at-name                AttributeType ::= { id-at 41 }
id-at-surname             AttributeType ::= { id-at  4 }
id-at-givenName           AttributeType ::= { id-at 42 }
id-at-initials            AttributeType ::= { id-at 43 }
id-at-generationQualifier AttributeType ::= { id-at 44 }

-- Naming attributes of type X520Name:
--   X520name ::= DirectoryString (SIZE (1..ub-name))
--
-- Expanded to avoid parameterized type:
X520name ::= CHOICE {
      teletexString     TeletexString   (SIZE (1..ub-name)),
      printableString   PrintableString (SIZE (1..ub-name)),
      universalString   UniversalString (SIZE (1..ub-name)),
      utf8String        UTF8String      (SIZE (1..ub-name)),
      bmpString         BMPString       (SIZE (1..ub-name)) }

-- Naming attributes of type X520CommonName

id-at-commonName        AttributeType ::= { id-at 3 }

-- Naming attributes of type X520CommonName:
--   X520CommonName ::= DirectoryName (SIZE (1..ub-common-name))
--
-- Expanded to avoid parameterized type:
X520CommonName ::= CHOICE {
      teletexString     TeletexString   (SIZE (1..ub-common-name)),
      printableString   PrintableString (SIZE (1..ub-common-name)),
      universalString   UniversalString (SIZE (1..ub-common-name)),
      utf8String        UTF8String      (SIZE (1..ub-common-name)),
      bmpString         BMPString       (SIZE (1..ub-common-name)) }
-- Naming attributes of type X520LocalityName

id-at-localityName      AttributeType ::= { id-at 7 }

-- Naming attributes of type X520LocalityName:
--   X520LocalityName ::= DirectoryName (SIZE (1..ub-locality-name))
--
-- Expanded to avoid parameterized type:
X520LocalityName ::= CHOICE {
      teletexString     TeletexString   (SIZE (1..ub-locality-name)),
      printableString   PrintableString (SIZE (1..ub-locality-name)),
      universalString   UniversalString (SIZE (1..ub-locality-name)),
      utf8String        UTF8String      (SIZE (1..ub-locality-name)),
      bmpString         BMPString       (SIZE (1..ub-locality-name)) }

-- Naming attributes of type X520StateOrProvinceName

id-at-stateOrProvinceName AttributeType ::= { id-at 8 }

-- Naming attributes of type X520StateOrProvinceName:
--   X520StateOrProvinceName ::= DirectoryName (SIZE (1..ub-state-name))
--
-- Expanded to avoid parameterized type:
X520StateOrProvinceName ::= CHOICE {
      teletexString     TeletexString   (SIZE (1..ub-state-name)),
      printableString   PrintableString (SIZE (1..ub-state-name)),
      universalString   UniversalString (SIZE (1..ub-state-name)),
      utf8String        UTF8String      (SIZE (1..ub-state-name)),
      bmpString         BMPString       (SIZE (1..ub-state-name)) }
-- Naming attributes of type X520OrganizationName

id-at-organizationName  AttributeType ::= { id-at 10 }

-- Naming attributes of type X520OrganizationName:
--   X520OrganizationName ::=
--          DirectoryName (SIZE (1..ub-organization-name))
--
-- Expanded to avoid parameterized type:
X520OrganizationName ::= CHOICE {
      teletexString     TeletexString
                          (SIZE (1..ub-organization-name)),
      printableString   PrintableString
                          (SIZE (1..ub-organization-name)),
      universalString   UniversalString
                          (SIZE (1..ub-organization-name)),
      utf8String        UTF8String
                          (SIZE (1..ub-organization-name)),
      bmpString         BMPString
                          (SIZE (1..ub-organization-name))  }

-- Naming attributes of type X520OrganizationalUnitName

id-at-organizationalUnitName AttributeType ::= { id-at 11 }

-- Naming attributes of type X520OrganizationalUnitName:
--   X520OrganizationalUnitName ::=
--          DirectoryName (SIZE (1..ub-organizational-unit-name))
--
-- Expanded to avoid parameterized type:
X520OrganizationalUnitName ::= CHOICE {
      teletexString     TeletexString
                          (SIZE (1..ub-organizational-unit-name)),
      printableString   PrintableString
                          (SIZE (1..ub-organizational-unit-name)),
      universalString   UniversalString
                          (SIZE (1..ub-organizational-unit-name)),
      utf8String        UTF8String
                          (SIZE (1..ub-organizational-unit-name)),
      bmpString         BMPString
                          (SIZE (1..ub-organizational-unit-name)) }
-- Naming attributes of type X520Title

id-at-title             AttributeType ::= { id-at 12 }

-- Naming attributes of type X520Title:
--   X520Title ::= DirectoryName (SIZE (1..ub-title))
--
-- Expanded to avoid parameterized type:
X520Title ::= CHOICE {
      teletexString     TeletexString   (SIZE (1..ub-title)),
      printableString   PrintableString (SIZE (1..ub-title)),
      universalString   UniversalString (SIZE (1..ub-title)),
      utf8String        UTF8String      (SIZE (1..ub-title)),
      bmpString         BMPString       (SIZE (1..ub-title)) }

-- Naming attributes of type X520dnQualifier

id-at-dnQualifier       AttributeType ::= { id-at 46 }

X520dnQualifier ::=     PrintableString

-- Naming attributes of type X520countryName (digraph from IS 3166)

id-at-countryName       AttributeType ::= { id-at 6 }

X520countryName ::=     PrintableString (SIZE (2))

-- Naming attributes of type X520SerialNumber

id-at-serialNumber      AttributeType ::= { id-at 5 }

X520SerialNumber ::=    PrintableString (SIZE (1..ub-serial-number))

-- Naming attributes of type X520Pseudonym

id-at-pseudonym         AttributeType ::= { id-at 65 }

-- Naming attributes of type X520Pseudonym:
--   X520Pseudonym ::= DirectoryName (SIZE (1..ub-pseudonym))
--
-- Expanded to avoid parameterized type:
X520Pseudonym ::= CHOICE {
   teletexString     TeletexString   (SIZE (1..ub-pseudonym)),
   printableString   PrintableString (SIZE (1..ub-pseudonym)),
   universalString   UniversalString (SIZE (1..ub-pseudonym)),
   utf8String        UTF8String      (SIZE (1..ub-pseudonym)),
   bmpString         BMPString       (SIZE (1..ub-pseudonym)) }
-- Naming attributes of type DomainComponent (from RFC 4519)

id-domainComponent   AttributeType ::= { 0 9 2342 19200300 100 1 25 }

DomainComponent ::=  IA5String

-- Legacy attributes

pkcs-9 OBJECT IDENTIFIER ::=
       { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1) 9 }

id-emailAddress      AttributeType ::= { pkcs-9 1 }

EmailAddress ::=     IA5String (SIZE (1..ub-emailaddress-length))

-- naming data types --

Name ::= CHOICE { -- only one possibility for now --
      rdnSequence  RDNSequence }

RDNSequence ::= SEQUENCE OF RelativeDistinguishedName

DistinguishedName ::=   RDNSequence

RelativeDistinguishedName ::= SET SIZE (1..MAX) OF AttributeTypeAndValue

-- Directory string type --

DirectoryString ::= CHOICE {
      teletexString       TeletexString   (SIZE (1..MAX)),
      printableString     PrintableString (SIZE (1..MAX)),
      universalString     UniversalString (SIZE (1..MAX)),
      utf8String          UTF8String      (SIZE (1..MAX)),
      bmpString           BMPString       (SIZE (1..MAX)) }

-- certificate and CRL specific structures begin here

Certificate  ::=  SEQUENCE  {
     tbsCertificate       TBSCertificate,
     signatureAlgorithm   AlgorithmIdentifier,
     signature            BIT STRING  }
TBSCertificate  ::=  SEQUENCE  {
     version         [0]  Version DEFAULT v1,
     serialNumber         CertificateSerialNumber,
     signature            AlgorithmIdentifier,
     issuer               Name,
     validity             Validity,
     subject              Name,
     subjectPublicKeyInfo SubjectPublicKeyInfo,
     issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL,
                          -- If present, version MUST be v2 or v3
     subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                          -- If present, version MUST be v2 or v3
     extensions      [3]  Extensions OPTIONAL
                          -- If present, version MUST be v3 --  }

Version  ::=  INTEGER  {  v1(0), v2(1), v3(2)  }

CertificateSerialNumber  ::=  INTEGER

Validity ::= SEQUENCE {
     notBefore      Time,
     notAfter       Time  }

Time ::= CHOICE {
     utcTime        UTCTime,
     generalTime    GeneralizedTime }

UniqueIdentifier  ::=  BIT STRING

SubjectPublicKeyInfo  ::=  SEQUENCE  {
     algorithm            AlgorithmIdentifier,
     subjectPublicKey     BIT STRING  }

Extensions  ::=  SEQUENCE SIZE (1..MAX) OF Extension

Extension  ::=  SEQUENCE  {
     extnID      OBJECT IDENTIFIER,
     critical    BOOLEAN DEFAULT FALSE,
     extnValue   OCTET STRING
                 -- contains the DER encoding of an ASN.1 value
                 -- corresponding to the extension type identified
                 -- by extnID
     }
-- CRL structures

CertificateList  ::=  SEQUENCE  {
     tbsCertList          TBSCertList,
     signatureAlgorithm   AlgorithmIdentifier,
     signature            BIT STRING  }

TBSCertList  ::=  SEQUENCE  {
     version                 Version OPTIONAL,
                                   -- if present, MUST be v2
     signature               AlgorithmIdentifier,
     issuer                  Name,
     thisUpdate              Time,
     nextUpdate              Time OPTIONAL,
     revokedCertificates     SEQUENCE OF SEQUENCE  {
          userCertificate         CertificateSerialNumber,
          revocationDate          Time,
          crlEntryExtensions      Extensions OPTIONAL
                                   -- if present, version MUST be v2
                               }  OPTIONAL,
     crlExtensions           [0] Extensions OPTIONAL }
                                   -- if present, version MUST be v2

-- Version, Time, CertificateSerialNumber, and Extensions were
-- defined earlier for use in the certificate structure

AlgorithmIdentifier  ::=  SEQUENCE  {
     algorithm               OBJECT IDENTIFIER,
     parameters              ANY DEFINED BY algorithm OPTIONAL  }
                                -- contains a value of the type
                                -- registered for use with the
                                -- algorithm object identifier value

-- X.400 address syntax starts here

ORAddress ::= SEQUENCE {
   built-in-standard-attributes BuiltInStandardAttributes,
   built-in-domain-defined-attributes
                   BuiltInDomainDefinedAttributes OPTIONAL,
   -- see also teletex-domain-defined-attributes
   extension-attributes ExtensionAttributes OPTIONAL }
-- Built-in Standard Attributes

BuiltInStandardAttributes ::= SEQUENCE {
   country-name                  CountryName OPTIONAL,
   administration-domain-name    AdministrationDomainName OPTIONAL,
   network-address           [0] IMPLICIT NetworkAddress OPTIONAL,
     -- see also extended-network-address
   terminal-identifier       [1] IMPLICIT TerminalIdentifier OPTIONAL,
   private-domain-name       [2] PrivateDomainName OPTIONAL,
   organization-name         [3] IMPLICIT OrganizationName OPTIONAL,
     -- see also teletex-organization-name
   numeric-user-identifier   [4] IMPLICIT NumericUserIdentifier
                                 OPTIONAL,
   personal-name             [5] IMPLICIT PersonalName OPTIONAL,
     -- see also teletex-personal-name
   organizational-unit-names [6] IMPLICIT OrganizationalUnitNames
                                 OPTIONAL }
     -- see also teletex-organizational-unit-names

CountryName ::= [APPLICATION 1] CHOICE {
   x121-dcc-code         NumericString
                           (SIZE (ub-country-name-numeric-length)),
   iso-3166-alpha2-code  PrintableString
                           (SIZE (ub-country-name-alpha-length)) }

AdministrationDomainName ::= [APPLICATION 2] CHOICE {
   numeric   NumericString   (SIZE (0..ub-domain-name-length)),
   printable PrintableString (SIZE (0..ub-domain-name-length)) }

NetworkAddress ::= X121Address  -- see also extended-network-address

X121Address ::= NumericString (SIZE (1..ub-x121-address-length))

TerminalIdentifier ::= PrintableString (SIZE (1..ub-terminal-id-length))

PrivateDomainName ::= CHOICE {
   numeric   NumericString   (SIZE (1..ub-domain-name-length)),
   printable PrintableString (SIZE (1..ub-domain-name-length)) }

OrganizationName ::= PrintableString
                            (SIZE (1..ub-organization-name-length))
  -- see also teletex-organization-name

NumericUserIdentifier ::= NumericString
                            (SIZE (1..ub-numeric-user-id-length))
PersonalName ::= SET {
   surname     [0] IMPLICIT PrintableString
                    (SIZE (1..ub-surname-length)),
   given-name  [1] IMPLICIT PrintableString
                    (SIZE (1..ub-given-name-length)) OPTIONAL,
   initials    [2] IMPLICIT PrintableString
                    (SIZE (1..ub-initials-length)) OPTIONAL,
   generation-qualifier [3] IMPLICIT PrintableString
                    (SIZE (1..ub-generation-qualifier-length))
                    OPTIONAL }
  -- see also teletex-personal-name

OrganizationalUnitNames ::= SEQUENCE SIZE (1..ub-organizational-units)
                             OF OrganizationalUnitName
  -- see also teletex-organizational-unit-names

OrganizationalUnitName ::= PrintableString (SIZE
                    (1..ub-organizational-unit-name-length))

-- Built-in Domain-defined Attributes

BuiltInDomainDefinedAttributes ::= SEQUENCE SIZE
                    (1..ub-domain-defined-attributes) OF
                    BuiltInDomainDefinedAttribute

BuiltInDomainDefinedAttribute ::= SEQUENCE {
   type PrintableString (SIZE
                   (1..ub-domain-defined-attribute-type-length)),
   value PrintableString (SIZE
                   (1..ub-domain-defined-attribute-value-length)) }

-- Extension Attributes

ExtensionAttributes ::= SET SIZE (1..ub-extension-attributes) OF
               ExtensionAttribute

ExtensionAttribute ::=  SEQUENCE {
   extension-attribute-type [0] IMPLICIT INTEGER
                   (0..ub-extension-attributes),
   extension-attribute-value [1]
                   ANY DEFINED BY extension-attribute-type }

-- Extension types and attribute values

common-name INTEGER ::= 1

CommonName ::= PrintableString (SIZE (1..ub-common-name-length))
teletex-common-name INTEGER ::= 2

TeletexCommonName ::= TeletexString (SIZE (1..ub-common-name-length))

teletex-organization-name INTEGER ::= 3

TeletexOrganizationName ::=
                TeletexString (SIZE (1..ub-organization-name-length))

teletex-personal-name INTEGER ::= 4

TeletexPersonalName ::= SET {
   surname     [0] IMPLICIT TeletexString
                    (SIZE (1..ub-surname-length)),
   given-name  [1] IMPLICIT TeletexString
                    (SIZE (1..ub-given-name-length)) OPTIONAL,
   initials    [2] IMPLICIT TeletexString
                    (SIZE (1..ub-initials-length)) OPTIONAL,
   generation-qualifier [3] IMPLICIT TeletexString
                    (SIZE (1..ub-generation-qualifier-length))
                    OPTIONAL }

teletex-organizational-unit-names INTEGER ::= 5

TeletexOrganizationalUnitNames ::= SEQUENCE SIZE
      (1..ub-organizational-units) OF TeletexOrganizationalUnitName

TeletexOrganizationalUnitName ::= TeletexString
                  (SIZE (1..ub-organizational-unit-name-length))

pds-name INTEGER ::= 7

PDSName ::= PrintableString (SIZE (1..ub-pds-name-length))

physical-delivery-country-name INTEGER ::= 8

PhysicalDeliveryCountryName ::= CHOICE {
   x121-dcc-code NumericString (SIZE (ub-country-name-numeric-length)),
   iso-3166-alpha2-code PrintableString
                               (SIZE (ub-country-name-alpha-length)) }

postal-code INTEGER ::= 9

PostalCode ::= CHOICE {
   numeric-code   NumericString (SIZE (1..ub-postal-code-length)),
   printable-code PrintableString (SIZE (1..ub-postal-code-length)) }

physical-delivery-office-name INTEGER ::= 10
PhysicalDeliveryOfficeName ::= PDSParameter

physical-delivery-office-number INTEGER ::= 11

PhysicalDeliveryOfficeNumber ::= PDSParameter

extension-OR-address-components INTEGER ::= 12

ExtensionORAddressComponents ::= PDSParameter

physical-delivery-personal-name INTEGER ::= 13

PhysicalDeliveryPersonalName ::= PDSParameter

physical-delivery-organization-name INTEGER ::= 14

PhysicalDeliveryOrganizationName ::= PDSParameter

extension-physical-delivery-address-components INTEGER ::= 15

ExtensionPhysicalDeliveryAddressComponents ::= PDSParameter

unformatted-postal-address INTEGER ::= 16

UnformattedPostalAddress ::= SET {
   printable-address SEQUENCE SIZE (1..ub-pds-physical-address-lines)
        OF PrintableString (SIZE (1..ub-pds-parameter-length)) OPTIONAL,
   teletex-string TeletexString
        (SIZE (1..ub-unformatted-address-length)) OPTIONAL }

street-address INTEGER ::= 17

StreetAddress ::= PDSParameter

post-office-box-address INTEGER ::= 18

PostOfficeBoxAddress ::= PDSParameter

poste-restante-address INTEGER ::= 19

PosteRestanteAddress ::= PDSParameter

unique-postal-name INTEGER ::= 20

UniquePostalName ::= PDSParameter

local-postal-attributes INTEGER ::= 21
LocalPostalAttributes ::= PDSParameter

PDSParameter ::= SET {
   printable-string PrintableString
                (SIZE(1..ub-pds-parameter-length)) OPTIONAL,
   teletex-string TeletexString
                (SIZE(1..ub-pds-parameter-length)) OPTIONAL }

extended-network-address INTEGER ::= 22

ExtendedNetworkAddress ::= CHOICE {
   e163-4-address SEQUENCE {
      number      [0] IMPLICIT NumericString
                       (SIZE (1..ub-e163-4-number-length)),
      sub-address [1] IMPLICIT NumericString
                       (SIZE (1..ub-e163-4-sub-address-length))
                       OPTIONAL },
   psap-address   [0] IMPLICIT PresentationAddress }

PresentationAddress ::= SEQUENCE {
    pSelector     [0] EXPLICIT OCTET STRING OPTIONAL,
    sSelector     [1] EXPLICIT OCTET STRING OPTIONAL,
    tSelector     [2] EXPLICIT OCTET STRING OPTIONAL,
    nAddresses    [3] EXPLICIT SET SIZE (1..MAX) OF OCTET STRING }

terminal-type  INTEGER ::= 23

TerminalType ::= INTEGER {
   telex        (3),
   teletex      (4),
   g3-facsimile (5),
   g4-facsimile (6),
   ia5-terminal (7),
   videotex     (8) } (0..ub-integer-options)

-- Extension Domain-defined Attributes

teletex-domain-defined-attributes INTEGER ::= 6

TeletexDomainDefinedAttributes ::= SEQUENCE SIZE
   (1..ub-domain-defined-attributes) OF TeletexDomainDefinedAttribute

TeletexDomainDefinedAttribute ::= SEQUENCE {
        type TeletexString
               (SIZE (1..ub-domain-defined-attribute-type-length)),
        value TeletexString
               (SIZE (1..ub-domain-defined-attribute-value-length)) }
--  specifications of Upper Bounds MUST be regarded as mandatory
--  from Annex B of ITU-T X.411 Reference Definition of MTS Parameter
--  Upper Bounds

-- Upper Bounds
ub-name INTEGER ::= 32768
ub-common-name INTEGER ::= 64
ub-locality-name INTEGER ::= 128
ub-state-name INTEGER ::= 128
ub-organization-name INTEGER ::= 64
ub-organizational-unit-name INTEGER ::= 64
ub-title INTEGER ::= 64
ub-serial-number INTEGER ::= 64
ub-match INTEGER ::= 128
ub-emailaddress-length INTEGER ::= 255
ub-common-name-length INTEGER ::= 64
ub-country-name-alpha-length INTEGER ::= 2
ub-country-name-numeric-length INTEGER ::= 3
ub-domain-defined-attributes INTEGER ::= 4
ub-domain-defined-attribute-type-length INTEGER ::= 8
ub-domain-defined-attribute-value-length INTEGER ::= 128
ub-domain-name-length INTEGER ::= 16
ub-extension-attributes INTEGER ::= 256
ub-e163-4-number-length INTEGER ::= 15
ub-e163-4-sub-address-length INTEGER ::= 40
ub-generation-qualifier-length INTEGER ::= 3
ub-given-name-length INTEGER ::= 16
ub-initials-length INTEGER ::= 5
ub-integer-options INTEGER ::= 256
ub-numeric-user-id-length INTEGER ::= 32
ub-organization-name-length INTEGER ::= 64
ub-organizational-unit-name-length INTEGER ::= 32
ub-organizational-units INTEGER ::= 4
ub-pds-name-length INTEGER ::= 16
ub-pds-parameter-length INTEGER ::= 30
ub-pds-physical-address-lines INTEGER ::= 6
ub-postal-code-length INTEGER ::= 16
ub-pseudonym INTEGER ::= 128
ub-surname-length INTEGER ::= 40
ub-terminal-id-length INTEGER ::= 24
ub-unformatted-address-length INTEGER ::= 180
ub-x121-address-length INTEGER ::= 16

-- Note - upper bounds on string types, such as TeletexString, are
-- measured in characters.  Excepting PrintableString or IA5String, a
-- significantly greater number of octets will be required to hold
-- such a value.  As a minimum, 16 octets, or twice the specified
-- upper bound, whichever is the larger, should be allowed for 
-- TeletexString.  For UTF8String or UniversalString at least four
-- times the upper bound should be allowed.

END

### A.2. Implicitly Tagged Module, 1988 Syntax

PKIX1Implicit88 { iso(1) identified-organization(3) dod(6) internet(1)
  security(5) mechanisms(5) pkix(7) id-mod(0) id-pkix1-implicit(19) }

DEFINITIONS IMPLICIT TAGS ::=

BEGIN

-- EXPORTS ALL --

IMPORTS
      id-pe, id-kp, id-qt-unotice, id-qt-cps,
      -- delete following line if "new" types are supported --
      BMPString, UTF8String,  -- end "new" types --
      ORAddress, Name, RelativeDistinguishedName,
      CertificateSerialNumber, Attribute, DirectoryString
      FROM PKIX1Explicit88 { iso(1) identified-organization(3)
            dod(6) internet(1) security(5) mechanisms(5) pkix(7)
            id-mod(0) id-pkix1-explicit(18) };

-- ISO arc for standard certificate and CRL extensions

id-ce OBJECT IDENTIFIER  ::=  {joint-iso-ccitt(2) ds(5) 29}

-- authority key identifier OID and syntax

id-ce-authorityKeyIdentifier OBJECT IDENTIFIER ::=  { id-ce 35 }

AuthorityKeyIdentifier ::= SEQUENCE {
    keyIdentifier             [0] KeyIdentifier            OPTIONAL,
    authorityCertIssuer       [1] GeneralNames             OPTIONAL,
    authorityCertSerialNumber [2] CertificateSerialNumber  OPTIONAL }
    -- authorityCertIssuer and authorityCertSerialNumber MUST both
    -- be present or both be absent

KeyIdentifier ::= OCTET STRING 
-- subject key identifier OID and syntax

id-ce-subjectKeyIdentifier OBJECT IDENTIFIER ::=  { id-ce 14 }

SubjectKeyIdentifier ::= KeyIdentifier

-- key usage extension OID and syntax

id-ce-keyUsage OBJECT IDENTIFIER ::=  { id-ce 15 }

KeyUsage ::= BIT STRING {
     digitalSignature        (0),
     nonRepudiation          (1),  -- recent editions of X.509 have
                                -- renamed this bit to contentCommitment
     keyEncipherment         (2),
     dataEncipherment        (3),
     keyAgreement            (4),
     keyCertSign             (5),
     cRLSign                 (6),
     encipherOnly            (7),
     decipherOnly            (8) }

-- private key usage period extension OID and syntax

id-ce-privateKeyUsagePeriod OBJECT IDENTIFIER ::=  { id-ce 16 }

PrivateKeyUsagePeriod ::= SEQUENCE {
     notBefore       [0]     GeneralizedTime OPTIONAL,
     notAfter        [1]     GeneralizedTime OPTIONAL }
     -- either notBefore or notAfter MUST be present

-- certificate policies extension OID and syntax

id-ce-certificatePolicies OBJECT IDENTIFIER ::=  { id-ce 32 }

anyPolicy OBJECT IDENTIFIER ::= { id-ce-certificatePolicies 0 }

CertificatePolicies ::= SEQUENCE SIZE (1..MAX) OF PolicyInformation

PolicyInformation ::= SEQUENCE {
     policyIdentifier   CertPolicyId,
     policyQualifiers   SEQUENCE SIZE (1..MAX) OF
             PolicyQualifierInfo OPTIONAL }

CertPolicyId ::= OBJECT IDENTIFIER 
PolicyQualifierInfo ::= SEQUENCE {
     policyQualifierId  PolicyQualifierId,
     qualifier          ANY DEFINED BY policyQualifierId }

-- Implementations that recognize additional policy qualifiers MUST
-- augment the following definition for PolicyQualifierId

PolicyQualifierId ::= OBJECT IDENTIFIER ( id-qt-cps | id-qt-unotice )

-- CPS pointer qualifier

CPSuri ::= IA5String

-- user notice qualifier

UserNotice ::= SEQUENCE {
     noticeRef        NoticeReference OPTIONAL,
     explicitText     DisplayText OPTIONAL }

NoticeReference ::= SEQUENCE {
     organization     DisplayText,
     noticeNumbers    SEQUENCE OF INTEGER }

DisplayText ::= CHOICE {
     ia5String        IA5String      (SIZE (1..200)),
     visibleString    VisibleString  (SIZE (1..200)),
     bmpString        BMPString      (SIZE (1..200)),
     utf8String       UTF8String     (SIZE (1..200)) }

-- policy mapping extension OID and syntax

id-ce-policyMappings OBJECT IDENTIFIER ::=  { id-ce 33 }

PolicyMappings ::= SEQUENCE SIZE (1..MAX) OF SEQUENCE {
     issuerDomainPolicy      CertPolicyId,
     subjectDomainPolicy     CertPolicyId }

-- subject alternative name extension OID and syntax

id-ce-subjectAltName OBJECT IDENTIFIER ::=  { id-ce 17 }

SubjectAltName ::= GeneralNames

GeneralNames ::= SEQUENCE SIZE (1..MAX) OF GeneralName 
GeneralName ::= CHOICE {
     otherName                 [0]  AnotherName,
     rfc822Name                [1]  IA5String,
     dNSName                   [2]  IA5String,
     x400Address               [3]  ORAddress,
     directoryName             [4]  Name,
     ediPartyName              [5]  EDIPartyName,
     uniformResourceIdentifier [6]  IA5String,
     iPAddress                 [7]  OCTET STRING,
     registeredID              [8]  OBJECT IDENTIFIER }

-- AnotherName replaces OTHER-NAME ::= TYPE-IDENTIFIER, as
-- TYPE-IDENTIFIER is not supported in the '88 ASN.1 syntax

AnotherName ::= SEQUENCE {
     type-id    OBJECT IDENTIFIER,
     value      [0] EXPLICIT ANY DEFINED BY type-id }

EDIPartyName ::= SEQUENCE {
     nameAssigner              [0]  DirectoryString OPTIONAL,
     partyName                 [1]  DirectoryString }

-- issuer alternative name extension OID and syntax

id-ce-issuerAltName OBJECT IDENTIFIER ::=  { id-ce 18 }

IssuerAltName ::= GeneralNames

id-ce-subjectDirectoryAttributes OBJECT IDENTIFIER ::=  { id-ce 9 }

SubjectDirectoryAttributes ::= SEQUENCE SIZE (1..MAX) OF Attribute

-- basic constraints extension OID and syntax

id-ce-basicConstraints OBJECT IDENTIFIER ::=  { id-ce 19 }

BasicConstraints ::= SEQUENCE {
     cA                      BOOLEAN DEFAULT FALSE,
     pathLenConstraint       INTEGER (0..MAX) OPTIONAL }
-- name constraints extension OID and syntax

id-ce-nameConstraints OBJECT IDENTIFIER ::=  { id-ce 30 }

NameConstraints ::= SEQUENCE {
     permittedSubtrees       [0]     GeneralSubtrees OPTIONAL,
     excludedSubtrees        [1]     GeneralSubtrees OPTIONAL }

GeneralSubtrees ::= SEQUENCE SIZE (1..MAX) OF GeneralSubtree

GeneralSubtree ::= SEQUENCE {
     base                    GeneralName,
     minimum         [0]     BaseDistance DEFAULT 0,
     maximum         [1]     BaseDistance OPTIONAL }

BaseDistance ::= INTEGER (0..MAX)

-- policy constraints extension OID and syntax

id-ce-policyConstraints OBJECT IDENTIFIER ::=  { id-ce 36 }

PolicyConstraints ::= SEQUENCE {
     requireExplicitPolicy   [0]     SkipCerts OPTIONAL,
     inhibitPolicyMapping    [1]     SkipCerts OPTIONAL }

SkipCerts ::= INTEGER (0..MAX)

-- CRL distribution points extension OID and syntax

id-ce-cRLDistributionPoints     OBJECT IDENTIFIER  ::=  {id-ce 31}

CRLDistributionPoints ::= SEQUENCE SIZE (1..MAX) OF DistributionPoint

DistributionPoint ::= SEQUENCE {
     distributionPoint       [0]     DistributionPointName OPTIONAL,
     reasons                 [1]     ReasonFlags OPTIONAL,
     cRLIssuer               [2]     GeneralNames OPTIONAL }

DistributionPointName ::= CHOICE {
     fullName                [0]     GeneralNames,
     nameRelativeToCRLIssuer [1]     RelativeDistinguishedName }
ReasonFlags ::= BIT STRING {
     unused                  (0),
     keyCompromise           (1),
     cACompromise            (2),
     affiliationChanged      (3),
     superseded              (4),
     cessationOfOperation    (5),
     certificateHold         (6),
     privilegeWithdrawn      (7),
     aACompromise            (8) }

-- extended key usage extension OID and syntax

id-ce-extKeyUsage OBJECT IDENTIFIER ::= {id-ce 37}

ExtKeyUsageSyntax ::= SEQUENCE SIZE (1..MAX) OF KeyPurposeId

KeyPurposeId ::= OBJECT IDENTIFIER

-- permit unspecified key uses

anyExtendedKeyUsage OBJECT IDENTIFIER ::= { id-ce-extKeyUsage 0 }

-- extended key purpose OIDs

id-kp-serverAuth             OBJECT IDENTIFIER ::= { id-kp 1 }
id-kp-clientAuth             OBJECT IDENTIFIER ::= { id-kp 2 }
id-kp-codeSigning            OBJECT IDENTIFIER ::= { id-kp 3 }
id-kp-emailProtection        OBJECT IDENTIFIER ::= { id-kp 4 }
id-kp-timeStamping           OBJECT IDENTIFIER ::= { id-kp 8 }
id-kp-OCSPSigning            OBJECT IDENTIFIER ::= { id-kp 9 }

-- inhibit any policy OID and syntax

id-ce-inhibitAnyPolicy OBJECT IDENTIFIER ::=  { id-ce 54 }

InhibitAnyPolicy ::= SkipCerts

-- freshest (delta)CRL extension OID and syntax

id-ce-freshestCRL OBJECT IDENTIFIER ::=  { id-ce 46 }

FreshestCRL ::= CRLDistributionPoints 
-- authority info access

id-pe-authorityInfoAccess OBJECT IDENTIFIER ::= { id-pe 1 }

AuthorityInfoAccessSyntax  ::=
        SEQUENCE SIZE (1..MAX) OF AccessDescription

AccessDescription  ::=  SEQUENCE {
        accessMethod          OBJECT IDENTIFIER,
        accessLocation        GeneralName  }

-- subject info access

id-pe-subjectInfoAccess OBJECT IDENTIFIER ::= { id-pe 11 }

SubjectInfoAccessSyntax  ::=
        SEQUENCE SIZE (1..MAX) OF AccessDescription

-- CRL number extension OID and syntax

id-ce-cRLNumber OBJECT IDENTIFIER ::= { id-ce 20 }

CRLNumber ::= INTEGER (0..MAX)

-- issuing distribution point extension OID and syntax

id-ce-issuingDistributionPoint OBJECT IDENTIFIER ::= { id-ce 28 }

IssuingDistributionPoint ::= SEQUENCE {
     distributionPoint          [0] DistributionPointName OPTIONAL,
     onlyContainsUserCerts      [1] BOOLEAN DEFAULT FALSE,
     onlyContainsCACerts        [2] BOOLEAN DEFAULT FALSE,
     onlySomeReasons            [3] ReasonFlags OPTIONAL,
     indirectCRL                [4] BOOLEAN DEFAULT FALSE,
     onlyContainsAttributeCerts [5] BOOLEAN DEFAULT FALSE }
     -- at most one of onlyContainsUserCerts, onlyContainsCACerts,
     -- and onlyContainsAttributeCerts may be set to TRUE.

id-ce-deltaCRLIndicator OBJECT IDENTIFIER ::= { id-ce 27 }

BaseCRLNumber ::= CRLNumber 
-- reason code extension OID and syntax

id-ce-cRLReasons OBJECT IDENTIFIER ::= { id-ce 21 }

CRLReason ::= ENUMERATED {
     unspecified             (0),
     keyCompromise           (1),
     cACompromise            (2),
     affiliationChanged      (3),
     superseded              (4),
     cessationOfOperation    (5),
     certificateHold         (6),
     removeFromCRL           (8),
     privilegeWithdrawn      (9),
     aACompromise           (10) }

-- certificate issuer CRL entry extension OID and syntax

id-ce-certificateIssuer OBJECT IDENTIFIER ::= { id-ce 29 }

CertificateIssuer ::= GeneralNames

-- hold instruction extension OID and syntax

id-ce-holdInstructionCode OBJECT IDENTIFIER ::= { id-ce 23 }

HoldInstructionCode ::= OBJECT IDENTIFIER

-- ANSI x9 arc holdinstruction arc

holdInstruction OBJECT IDENTIFIER ::=
          {joint-iso-itu-t(2) member-body(2) us(840) x9cm(10040) 2}

-- ANSI X9 holdinstructions

id-holdinstruction-none OBJECT IDENTIFIER  ::=
                                      {holdInstruction 1} -- deprecated

id-holdinstruction-callissuer OBJECT IDENTIFIER ::= {holdInstruction 2}

id-holdinstruction-reject OBJECT IDENTIFIER ::= {holdInstruction 3}
-- invalidity date CRL entry extension OID and syntax

id-ce-invalidityDate OBJECT IDENTIFIER ::= { id-ce 24 }

InvalidityDate ::=  GeneralizedTime

END

# Referenced Sections from RFC 7924: Transport Layer Security (TLS) Cached Information Extension

The following sections were referenced. Remaining sections are not included.

## 3. Cached Information Extension

   This document defines a new extension type (cached_info(25)), which
   is used in ClientHello and ServerHello messages.  The extension type
   is specified as follows.

         enum {
              cached_info(25), (65535)
         } ExtensionType;
   The extension_data field of this extension, when included in the
   ClientHello, MUST contain the CachedInformation structure.  The
   client MAY send multiple CachedObjects of the same
   CachedInformationType.  This may, for example, be the case when the
   client has cached multiple certificates from a server.

         enum {
              cert(1), cert_req(2) (255)
         } CachedInformationType;

         struct {
              select (type) {
                case client:
                  CachedInformationType type;
                  opaque hash_value<1..255>;
                case server:
                  CachedInformationType type;
              } body;
         } CachedObject;

         struct {
              CachedObject cached_info<1..2^16-1>;
         } CachedInformation;

   This document defines the following two types:

   'cert' type for not sending the complete server certificate message:

      With the type field set to 'cert', the client MUST include the
      fingerprint of the Certificate message in the hash_value field.
      For this type, the fingerprint MUST be calculated using the
      procedure described in Section 5 with the Certificate message as
      input data.

   'cert_req' Type for not sending the complete CertificateRequest
      Message:

      With the type set to 'cert_req', the client MUST include the
      fingerprint of the CertificateRequest message in the hash_value
      field.  For this type, the fingerprint MUST be calculated using
      the procedure described in Section 5 with the CertificateRequest
      message as input data.

   New cached info types can be added following the policy described in
   the IANA Considerations (Section 8).  New message digest algorithms
   for use with these types can also be added by registering a new type
   that makes use of the updated message digest algorithm.  For
   practical reasons, we recommend reusing hash algorithms already 
   available with TLS ciphersuites.  To avoid additional code and to
   keep the collision probability low, new hash algorithms MUST NOT have
   a collision resistance worse than SHA-256.

# Referenced Sections from RFC 9525: Service Identity in TLS

The following sections were referenced. Remaining sections are not included.

### 1.1. Motivation

   The visible face of the Internet largely consists of services that
   employ a client-server architecture in which a client communicates
   with an application service.  When a client communicates with an
   application service using [TLS ], [DTLS ], or a protocol built on those
   ([QUIC ] being a notable example), it has some notion of the server's
   identity (e.g., "the website at bigcompany.example") while attempting
   to establish secure communication.  Likewise, during TLS negotiation,
   the server presents its notion of the service's identity in the form
   of a public key certificate that was issued by a certification
   authority (CA) in the context of the Internet Public Key
   Infrastructure using X.509 [PKIX ].  Informally, we can think of these
   identities as the client's "reference identity" and the server's
   "presented identity"; more formal definitions are given later.  A
   client needs to verify that the server's presented identity matches
   its reference identity so it can deterministically and automatically
   authenticate the communication.

   This document defines procedures for how clients perform this
   verification.  It therefore defines requirements on other parties,
   such as the certification authorities that issue certificates, the
   service administrators requesting them, and the protocol designers
   defining interactions between clients and servers.

   This document obsoletes RFC 6125 [VERIFY ].  Changes from RFC 6125   [VERIFY ] are described under Appendix A .

## 6. Verifying Service Identity

   At a high level, the client verifies the application service's
   identity by performing the following actions:

   1.  The client constructs a list of reference identifiers it would
       find acceptable based on the source domain and, if applicable,
       the type of service to which the client is connecting.

   2.  The server provides its presented identifiers in the form of a
       PKIX certificate.

   3.  The client checks each of its reference identifiers against the
       server's presented identifiers for the purpose of finding a
       match.  When checking a reference identifier against a presented
       identifier, the client matches the source domain of the
       identifiers and, optionally, their application service type.

   Naturally, in addition to checking identifiers, a client should
   perform further checks, such as expiration and revocation, to ensure
   that the server is authorized to provide the requested service.
   Because such checking is not a matter of verifying the application
   service identity presented in a certificate, methods for doing so are
   out of scope for this document.

### 6.1. Constructing a List of Reference Identifiers





#### 6.1.1. Rules

   The client MUST construct a list of acceptable reference identifiers
   and MUST do so independently of the identifiers presented by the
   server.

   The inputs used by the client to construct its list of reference
   identifiers might be a URI that a user has typed into an interface
   (e.g., an HTTPS URL for a website), configured account information
   (e.g., the domain name of a host for retrieving email, which might be
   different from the DNS domain name portion of a username), a
   hyperlink in a web page that triggers a browser to retrieve a media
   object or script, or some other combination of information that can
   yield a source domain and an application service type.

   This document does not precisely define how reference identifiers are
   generated.  Defining reference identifiers is the responsibility of
   applications or protocols that use this document.  Because the
   security of a system that uses this document will depend on how
   reference identifiers are generated, great care should be taken in
   this process.  For example, a protocol or application could specify
   that the application service type is obtained through a one-to-one
   mapping of URI schemes to service types or that the protocol or
   application supports only a restricted set of URI schemes.
   Similarly, it could specify that a domain name or an IP address taken
   as input to the reference identifier must be obtained in a secure
   context such as a hyperlink embedded in a web page that was delivered
   over an authenticated and encrypted channel (for instance, see
   [SECURE-CONTEXTS ] with regard to the web platform).

   Naturally, if the inputs themselves are invalid or corrupt (e.g., a
   user has clicked a hyperlink provided by a malicious entity in a
   phishing attack), then the client might end up communicating with an
   unexpected application service.

   During the course of processing, a client might be exposed to
   identifiers that look like, but are not, reference identifiers.  For
   example, DNS resolution that starts at a DNS-ID reference identifier
   might produce intermediate domain names that need to be further
   resolved.  Unless an application defines a process for authenticating
   intermediate identifiers in a way that then allows them to be used as
   a reference identifier (for example, see [SMTP-TLS ]), any
   intermediate values are not reference identifiers and MUST NOT be
   treated as such.  In the DNS case, not treating intermediate domain
   names as reference identifiers removes DNS and DNS resolution from
   the attack surface.

   As one example of the process of generating a reference identifier,
   from the user input of the URI <sip:alice@voice.college.example>, a
   client could derive the application service type sip from the URI
   scheme and parse the domain name college.example from the "host"
   component.

   Using the combination of one or more FQDNs or IP addresses, plus
   optionally an application service type, the client MUST construct its
   list of reference identifiers in accordance with the following rules:

   *  If a server for the application service type is typically
      associated with a URI for security purposes (i.e., a formal
      protocol document specifies the use of URIs in server
      certificates), the reference identifier SHOULD be a URI-ID.

   *  If a server for the application service type is typically
      discovered by means of DNS SRV records, the reference identifier
      SHOULD be an SRV-ID.

   *  If the reference identifier is an IP address, the reference
      identifier is an IP-ID.

   *  In the absence of more specific identifiers, the reference
      identifier is a DNS-ID.  A reference identifier of type DNS-ID can
      be directly constructed from an FQDN that is (a) contained in or
      securely derived from the inputs or (b) explicitly associated with
      the source domain by means of user configuration.

   Which identifier types a client includes in its list of reference
   identifiers, and their priority, is a matter of local policy.  For
   example, a client that is built to connect only to a particular kind
   of service might be configured to accept as valid only certificates
   that include an SRV-ID for that application service type.  By
   contrast, a more lenient client, even if built to connect only to a
   particular kind of service, might include SRV-IDs, DNS-IDs, and IP-
   IDs in its list of reference identifiers.

#### 6.1.2. Examples

   The following examples are for illustrative purposes only and are not
   intended to be comprehensive.

   1.  A web browser that is connecting via HTTPS to the website at
       <https://www.bigcompany.example/> would have a single reference
       identifier: a DNS-ID of www.bigcompany.example.

   2.  A web browser connecting to <https://192.0.2.107/> would have a
       single IP-ID reference identifier of 192.0.2.107.  Likewise, if
       connecting to <https://[2001:db8::abcd]>, it would have a single
       IP-ID reference identifier of 2001:db8::abcd.

   3.  A mail user agent that is connecting via IMAPS to the email
       service at isp.example (resolved as mail.isp.example) might have
       three reference identifiers: an SRV-ID of _imaps.isp.example (see
       [EMAIL-SRV ]) and DNS-IDs of isp.example and mail.isp.example.  An
       email user agent that does not support [EMAIL-SRV ] would probably
       be explicitly configured to connect to mail.isp.example, whereas
       an SRV-aware user agent would derive isp.example from an email
       address of the form user@isp.example but might also accept
       mail.isp.example as the DNS domain name portion of reference
       identifiers for the service.

   4.  A VoIP user agent that is connecting via SIP to the voice service
       at voice.college.example might have only one reference
       identifier: a URI-ID of sip:voice.college.example (see
       [SIP-CERTS ]).

   5.  An IM client that is connecting via XMPP to the IM service at
       messenger.example might have three reference identifiers: an SRV-
       ID of _xmpp-client.messenger.example (see [XMPP ]), a DNS-ID of
       messenger.example, and an XMPP-specific XmppAddr of
       messenger.example (see [XMPP ]).

   In all these cases, presented identifiers that do not match the
   reference identifier(s) would be rejected; for instance:

   *  With regard to the first example, a DNS-ID of
      web.bigcompany.example would be rejected because the DNS domain
      name portion does not match www.bigcompany.example.

   *  With regard to the third example, a URI-ID of
      <sip:www.college.example> would be rejected because the DNS domain
      name portion does not match "voice.college.example", and a DNS-ID
      of "voice.college.example" would be rejected because it lacks the
      appropriate application service type portion (i.e., it does not
      specify a "sip:" URI).

### 6.2. Preparing to Seek a Match

   Once the client has constructed its list of reference identifiers and
   has received the server's presented identifiers, the client checks
   its reference identifiers against the presented identifiers for the
   purpose of finding a match.  The search fails if the client exhausts
   its list of reference identifiers without finding a match.  The
   search succeeds if any presented identifier matches one of the
   reference identifiers, at which point the client SHOULD stop the
   search.

   Before applying the comparison rules provided in the following
   sections, the client might need to split the reference identifier
   into components.  Each reference identifier produces either a domain
   name or an IP address and optionally an application service type as
   follows:

   *  A DNS-ID reference identifier MUST be used directly as the DNS
      domain name, and there is no application service type.

   *  An IP-ID reference identifier MUST exactly match the value of an
      iPAddress entry in subjectAltName, with no partial (e.g., network-
      level) matching.  There is no application service type.

   *  For an SRV-ID reference identifier, the DNS domain name portion is
      the Name and the application service type portion is the Service.
      For example, an SRV-ID of _imaps.isp.example has a DNS domain name
      portion of isp.example and an application service type portion of
      imaps, which maps to the IMAP application protocol as explained in
      [EMAIL-SRV ].

   *  For a reference identifier of type URI-ID, the DNS domain name
      portion is the "reg-name" part of the "host" component and the
      application service type portion is the scheme, as defined above.
      Matching only the "reg-name" rule from [URI ] limits the additional
      domain name validation (Section 6.3) to DNS domain names or non-IP
      hostnames.  A URI that contains an IP address might be matched
      against an IP-ID in place of a URI-ID by some lenient clients.
      This document does not describe how a URI that contains no "host"
      component can be matched.  Note that extraction of the "reg-name"
      might necessitate normalization of the URI (as explained in
      Section 6 of [URI ]).  For example, a URI-ID of
      <sip:voice.college.example> would be split into a DNS domain name
      portion of voice.college.example and an application service type
      of sip (associated with an application protocol of SIP as
      explained in [SIP-CERTS ]).

   If the reference identifier produces a domain name, the client MUST
   match the DNS name; see Section 6.3.  If the reference identifier
   produces an IP address, the client MUST match the IP address; see Section 6.4.  If an application service type is present, it MUST also
   match the service type; see Section 6.5.

### 6.3. Matching the DNS Domain Name Portion

   This section describes how the client must determine if the presented
   DNS name matches the reference DNS name.  The rules differ depending
   on whether the domain to be checked is an internationalized domain
   name, as defined in Section 2, or not.  For clients that support
   presented identifiers containing the wildcard character "*", this
   section also specifies a supplemental rule for such "wildcard
   certificates".  This section uses the description of labels and
   domain names in [DNS-CONCEPTS ].

   If the DNS domain name portion of a reference identifier is not an
   internationalized domain name (i.e., an FQDN that conforms to
   "preferred name syntax" as described in Section 3.5 of
   [DNS-CONCEPTS ]), then the matching of the reference identifier
   against the presented identifier MUST be performed by comparing the
   set of domain name labels using a case-insensitive ASCII comparison,
   as clarified by [DNS-CASE ].  For example, WWW.BigCompany.Example
   would be lower-cased to www.bigcompany.example for comparison
   purposes.  Each label MUST match in order for the names to be
   considered a match, except as supplemented by the rule about checking
   wildcard labels in presented identifiers given below.

   If the DNS domain name portion of a reference identifier is an
   internationalized domain name, then the client MUST convert any
   U-labels [IDNA-DEFS ] in the domain name to A-labels before checking
   the domain name or comparing it with others.  In accordance with
   [IDNA-PROTO ], A-labels MUST be compared as case-insensitive ASCII.
   Each label MUST match in order for the domain names to be considered
   to match, except as supplemented by the rule about checking wildcard
   labels in presented identifiers given below.

   If the technology specification supports wildcards in presented
   identifiers, then the client MUST match the reference identifier
   against a presented identifier whose DNS domain name portion contains
   the wildcard character "*" in a label, provided these requirements
   are met:

   1.  There is only one wildcard character.

   2.  The wildcard character appears only as the complete content of
       the left-most label.

   If the requirements are not met, the presented identifier is invalid
   and MUST be ignored.

   A wildcard in a presented identifier can only match one label in a
   reference identifier.  This specification covers only wildcard
   characters in presented identifiers, not wildcard characters in
   reference identifiers or in DNS domain names more generally.
   Therefore, the use of wildcard characters as described herein is not
   to be confused with DNS wildcard matching, where the "*" label always
   matches at least one whole label and sometimes more; see
   [DNS-CONCEPTS ], Section 4.3.3 and [DNS-WILDCARDS ].  In particular, it
   also deviates from [DNS-WILDCARDS ], Section 2.1.3.

   For information regarding the security characteristics of wildcard
   certificates, see Section 7.1.

### 6.4. Matching an IP Address Portion

   Matching of an IP-ID is based on an octet-for-octet comparison of the
   bytes of the reference identity with the bytes contained in the
   iPAddress subjectAltName.

   For an IP address that appears in a URI-ID, the "host" component of
   both the reference identity and the presented identifier must match.
   These are parsed as either an "IPv6address" (following [URI ],
   Section 3.2.2) or an "IPv4address" (following [IPv4]).  If the
   resulting octets are equal, the IP address matches.

   This document does not specify how an SRV-ID reference identity can
   include an IP address, as [SRVNAME ] only defines string names, not
   octet identifiers such as an IP address.

### 6.5. Matching the Application Service Type Portion

   The rules for matching the application service type depend on whether
   the identifier is an SRV-ID or a URI-ID.

   These identifiers provide an application service type portion to be
   checked, but that portion is combined only with the DNS domain name
   portion of the SRV-ID or URI-ID itself.  Consider the example of a
   messaging client that has two reference identifiers: (1) an SRV-ID of
   _xmpp-client.messenger.example and (2) a DNS-ID of app.example.  The
   client MUST check (1) the combination of (a) an application service
   type of xmpp-client and (b) a DNS domain name of messenger.example as
   well as (2) a DNS domain name of app.example.  However, the client
   MUST NOT check the combination of an application service type of
   xmpp-client and a DNS domain name of app.example because it does not
   have an SRV-ID of _xmpp-client.app.example in its list of reference
   identifiers.

   If the identifier is an SRV-ID, then the application service name
   MUST be matched in a case-insensitive manner, in accordance with
   [DNS-SRV ].  Note that per [SRVNAME ], the underscore "_" is part of
   the service name in DNS SRV records and in SRV-IDs.

   If the identifier is a URI-ID, then the scheme name portion MUST be
   matched in a case-insensitive manner, in accordance with [URI ].  Note
   that the colon ":" is a separator between the scheme name and the
   rest of the URI and thus does not need to be included in any
   comparison.

### 6.6. Outcome

   If the client has found a presented identifier that matches a
   reference identifier, then the service identity check has succeeded.
   In this case, the client MUST use the matched reference identifier as
   the validated identity of the application service.

   If the client does not find a presented identifier matching any of
   the reference identifiers, then the client MUST proceed as follows.

   If the client is an automated application, then it SHOULD terminate
   the communication attempt with a bad certificate error and log the
   error appropriately.  The application MAY provide a configuration
   setting to disable this behavior, but it MUST NOT disable this
   security control by default.

   If the client is one that is directly controlled by a human user,
   then it SHOULD inform the user of the identity mismatch and
   automatically terminate the communication attempt with a bad
   certificate error in order to prevent users from inadvertently
   bypassing security protections in hostile situations.  Such clients
   MAY give advanced users the option of proceeding with acceptance
   despite the identity mismatch.  Although this behavior can be
   appropriate in certain specialized circumstances, it needs to be
   handled with extreme caution, for example by first encouraging even
   an advanced user to terminate the communication attempt and, if they
   choose to proceed anyway, by forcing the user to view the entire
   certification path before proceeding.

   The application MAY also present the user with the ability to accept
   the presented certificate as valid for subsequent connections.  Such
   ad hoc "pinning" SHOULD NOT restrict future connections to just the
   pinned certificate.  Local policy that statically enforces a given
   certificate for a given peer SHOULD be made available only as prior
   configuration rather than a just-in-time override for a failed
   connection.

# Referenced Sections from RFC 5425: Transport Layer Security (TLS) Transport Mapping for Syslog

The following sections were referenced. Remaining sections are not included.

### 5.2. Subject Name Authorization

   Implementations MUST support certification path validation [RFC5280].
   In addition, they MUST support specifying the authorized peers using
   locally configured host names and matching the name against the
   certificate as follows.

   o  Implementations MUST support matching the locally configured host
      name against a dNSName in the subjectAltName extension field and
      SHOULD support checking the name against the common name portion
      of the subject distinguished name.

   o  The '*' (ASCII 42) wildcard character is allowed in the dNSName of
      the subjectAltName extension (and in common name, if used to store
      the host name), but only as the left-most (least significant) DNS
      label in that value.  This wildcard matches any left-most DNS
      label in the server name.  That is, the subject *.example.com
      matches the server names a.example.com and b.example.com, but does
      not match example.com or a.b.example.com.  Implementations MUST
      support wildcards in certificates as specified above, but MAY
      provide a configuration option to disable them.

   o  Locally configured names MAY contain the wildcard character to
      match a range of values.  The types of wildcards supported MAY be
      more flexible than those allowed in subject names, making it
      possible to support various policies for different environments.
      For example, a policy could allow for a trust-root-based
      authorization where all credentials issued by a particular CA
      trust root are authorized.

   o  If the locally configured name is an internationalized domain
      name, conforming implementations MUST convert it to the ASCII
      Compatible Encoding (ACE) format for performing comparisons, as
      specified in Section 7 of [RFC5280].

   o  Implementations MAY support matching a locally configured IP
      address against an iPAddress stored in the subjectAltName
      extension.  In this case, the locally configured IP address is
      converted to an octet string as specified in [RFC5280], Section 
      4.2.1.6.  A match occurs if this octet string is equal to the
      value of iPAddress in the subjectAltName extension.

# Referenced Sections from RFC 6066: Transport Layer Security (TLS) Extensions: Extension Definitions

The following sections were referenced. Remaining sections are not included.

## 3. Server Name Indication

   TLS does not provide a mechanism for a client to tell a server the
   name of the server it is contacting.  It may be desirable for clients
   to provide this information to facilitate secure connections to
   servers that host multiple 'virtual' servers at a single underlying
   network address.

   In order to provide any of the server names, clients MAY include an
   extension of type "server_name" in the (extended) client hello.  The
   "extension_data" field of this extension SHALL contain
   "ServerNameList" where:

      struct {
          NameType name_type;
          select (name_type) {
              case host_name: HostName;
          } name;
      } ServerName;

      enum {
          host_name(0), (255)
      } NameType;

      opaque HostName<1..2^16-1>;

      struct {
          ServerName server_name_list<1..2^16-1>
      } ServerNameList;

   The ServerNameList MUST NOT contain more than one name of the same
   name_type.  If the server understood the ClientHello extension but
   does not recognize the server name, the server SHOULD take one of two
   actions: either abort the handshake by sending a fatal-level
   unrecognized_name(112) alert or continue the handshake.  It is NOT
   RECOMMENDED to send a warning-level unrecognized_name(112) alert,
   because the client's behavior in response to warning-level alerts is
   unpredictable.  If there is a mismatch between the server name used
   by the client application and the server name of the credential
   chosen by the server, this mismatch will become apparent when the
   client application performs the server endpoint identification, at
   which point the client application will have to decide whether to
   proceed with the communication.  TLS implementations are encouraged
   to make information available to application callers about warning-
   level alerts that were received or sent during a TLS handshake.  Such
   information can be useful for diagnostic purposes. 
      Note: Earlier versions of this specification permitted multiple
      names of the same name_type.  In practice, current client
      implementations only send one name, and the client cannot
      necessarily find out which name the server selected.  Multiple
      names of the same name_type are therefore now prohibited.

   Currently, the only server names supported are DNS hostnames;
   however, this does not imply any dependency of TLS on DNS, and other
   name types may be added in the future (by an RFC that updates this
   document).  The data structure associated with the host_name NameType
   is a variable-length vector that begins with a 16-bit length.  For
   backward compatibility, all future data structures associated with
   new NameTypes MUST begin with a 16-bit length field.  TLS MAY treat
   provided server names as opaque data and pass the names and types to
   the application.

   "HostName" contains the fully qualified DNS hostname of the server,
   as understood by the client.  The hostname is represented as a byte
   string using ASCII encoding without a trailing dot.  This allows the
   support of internationalized domain names through the use of A-labels
   defined in [RFC5890].  DNS hostnames are case-insensitive.  The
   algorithm to compare hostnames is described in [RFC5890], Section 
   2.3.2.4.

   Literal IPv4 and IPv6 addresses are not permitted in "HostName".

   It is RECOMMENDED that clients include an extension of type
   "server_name" in the client hello whenever they locate a server by a
   supported name type.

   A server that receives a client hello containing the "server_name"
   extension MAY use the information contained in the extension to guide
   its selection of an appropriate certificate to return to the client,
   and/or other aspects of security policy.  In this event, the server
   SHALL include an extension of type "server_name" in the (extended)
   server hello.  The "extension_data" field of this extension SHALL be
   empty.

   When the server is deciding whether or not to accept a request to
   resume a session, the contents of a server_name extension MAY be used
   in the lookup of the session in the session cache.  The client SHOULD
   include the same server_name extension in the session resumption
   request as it did in the full handshake that established the session.
   A server that implements this extension MUST NOT accept the request
   to resume the session if the server_name extension contains a
   different name.  Instead, it proceeds with a full handshake to
   establish a new session.  When resuming a session, the server MUST
   NOT include a server_name extension in the server hello. 
   If an application negotiates a server name using an application
   protocol and then upgrades to TLS, and if a server_name extension is
   sent, then the extension SHOULD contain the same name that was
   negotiated in the application protocol.  If the server_name is
   established in the TLS session handshake, the client SHOULD NOT
   attempt to request a different server name at the application layer.

# Referenced Sections from RFC 9257: Guidance for External Pre-Shared Key (PSK) Usage in TLS

The following sections were referenced. Remaining sections are not included.

### 6.1. Stack Interfaces

   Most major TLS implementations support external PSKs.  Stacks
   supporting external PSKs provide interfaces that applications may use
   when configuring PSKs for individual connections.  Details about some
   existing stacks at the time of writing are below.

   *  OpenSSL and BoringSSL: Applications can specify support for
      external PSKs via distinct ciphersuites in TLS 1.2 and below.
      Also, they can then configure callbacks that are invoked for PSK
      selection during the handshake.  These callbacks must provide a
      PSK identity and key.  The exact format of the callback depends on
      the negotiated TLS protocol version, with new callback functions
      added specifically to OpenSSL for TLS 1.3 [RFC8446] PSK support.
      The PSK length is validated to be between 1-256 bytes (inclusive).
      The PSK identity may be up to 128 bytes long.

   *  mbedTLS: Client applications configure PSKs before creating a
      connection by providing the PSK identity and value inline.
      Servers must implement callbacks similar to that of OpenSSL.  Both
      PSK identity and key lengths may be between 1-16 bytes long
      (inclusive).

   *  gnuTLS: Applications configure PSK values as either raw byte
      strings or hexadecimal strings.  The PSK identity and key size are
      not validated.

   *  wolfSSL: Applications configure PSKs with callbacks similar to
      OpenSSL.

#### 6.1.1. PSK Identity Encoding and Comparison



   Section 5.1 of [RFC4279] mandates that the PSK identity should be
   first converted to a character string and then encoded to octets
   using UTF-8.  This was done to avoid interoperability problems
   (especially when the identity is configured by human users).  On the
   other hand, [RFC7925] advises implementations against assuming any
   structured format for PSK identities and recommends byte-by-byte
   comparison for any operation.  When PSK identities are configured
   manually, it is important to be aware that visually identical strings
   may, in fact, differ due to encoding issues.

   TLS 1.3 [RFC8446] follows the same practice of specifying the PSK
   identity as a sequence of opaque bytes (shown as opaque
   identity<1..2^16-1> in the specification) that thus is compared on a
   byte-by-byte basis.  [RFC8446] also requires that the PSK identities
   are at least 1 byte and at the most 65535 bytes in length.  Although
   [RFC8446] does not place strict requirements on the format of PSK
   identities, note that the format of PSK identities can vary depending
   on the deployment:

   *  The PSK identity MAY be a user-configured string when used in
      protocols like Extensible Authentication Protocol (EAP) [RFC3748].
      For example, gnuTLS treats PSK identities as usernames.

   *  PSK identities MAY have a domain name suffix for roaming and
      federation.  In applications and settings where the domain name
      suffix is privacy sensitive, this practice is NOT RECOMMENDED.

   *  Deployments should take care that the length of the PSK identity
      is sufficient to avoid collisions.

#### 6.1.2. PSK Identity Collisions

   It is possible, though unlikely, that an external PSK identity may
   clash with a resumption PSK identity.  The TLS stack implementation
   and sequencing of PSK callbacks influences the application's behavior
   when identity collisions occur.  When a server receives a PSK
   identity in a TLS 1.3 ClientHello, some TLS stacks execute the
   application's registered callback function before checking the
   stack's internal session resumption cache.  This means that if a PSK
   identity collision occurs, the application's external PSK usage will
   typically take precedence over the internal session resumption path.

   Because resumption PSK identities are assigned by the TLS stack
   implementation, it is RECOMMENDED that these identifiers be assigned
   in a manner that lets resumption PSKs be distinguished from external
   PSKs to avoid concerns with collisions altogether.

# Summary of reference from NIST, "Security Requirements for Cryptographic Modules", NIST FIPS 140-3, DOI 10.6028/NIST.FIPS.140-3, March 2019, <https://nvlpubs.nist.gov/nistpubs/FIPS/ NIST.FIPS.140-3.pdf >. (FIPS-140-3)

The National Institute of Standards and Technology (NIST) published the Federal Information Processing Standard (FIPS) 140-3, titled "Security Requirements for Cryptographic Modules," in March 2019. This standard outlines the security requirements for cryptographic modules used within security systems that protect sensitive but unclassified information. 

FIPS 140-3 defines four increasing levels of security—Levels 1 through 4—designed to accommodate a wide range of applications and environments. Each level builds upon the previous one, adding more stringent requirements to enhance the security of cryptographic modules. 

The standard specifies security requirements across several areas related to the secure design, implementation, and operation of cryptographic modules, including:

- **Cryptographic Module Specification**: Detailed description of the module and its security functions.

- **Cryptographic Module Interfaces**: Definition of all entry and exit points and their functions.

- **Roles, Services, and Authentication**: Identification of authorized roles and the corresponding services, along with authentication methods.

- **Software/Firmware Security**: Measures to ensure the integrity and security of software and firmware components.

- **Operating Environment**: Requirements for the module's operational context, including the operating system and other software.

- **Physical Security**: Protections against physical tampering and environmental conditions.

- **Non-Invasive Security**: Safeguards against non-invasive attacks that could compromise the module.

- **Sensitive Security Parameter Management**: Procedures for handling critical security parameters, such as cryptographic keys.

- **Self-Tests**: Mechanisms for the module to perform self-tests to ensure proper functioning.

- **Life-Cycle Assurance**: Controls over the development, distribution, and maintenance of the module.

- **Mitigation of Other Attacks**: Strategies to address other potential vulnerabilities not covered in the above areas.

FIPS 140-3 is applicable to all federal agencies that use cryptographic-based security systems to protect sensitive information in computer and telecommunication systems. The standard is also available for adoption by private and commercial organizations. 

For more detailed information, you can access the full document here:  

