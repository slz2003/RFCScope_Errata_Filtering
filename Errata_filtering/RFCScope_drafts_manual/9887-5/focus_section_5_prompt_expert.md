Let us analyze section 5 of RFC 9887. All references made by section 5 have also been included below.

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
# Referenced Sections from RFC 3365: Strong Security Requirements for Internet Engineering Task Force Standard Protocols

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   The purpose of this document is to document the IETF consensus on
   security requirements for protocols as well as to provide the
   background and motivation for them.

   The Internet is a global network of independently managed networks
   and hosts.  As such there is no central authority responsible for the
   operation of the network.  There is no central authority responsible
   for the provision of security across the network either.

   Security needs to be provided end-to-end or host to host.  The IETF's
   security role is to ensure that IETF standard protocols have the
   necessary features to provide appropriate security for the
   application as it may be used across the Internet.  Mandatory to
   implement mechanisms should provide adequate security to protect
   sensitive business applications.

# Referenced Sections from RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3

The following sections were referenced. Remaining sections are not included.

### 2.3. 0-RTT Data

   When clients and servers share a PSK (either obtained externally or
   via a previous handshake), TLS 1.3 allows clients to send data on the
   first flight ("early data").  The client uses the PSK to authenticate
   the server and to encrypt the early data.

   As shown in Figure 4, the 0-RTT data is just added to the 1-RTT
   handshake in the first flight.  The rest of the handshake uses the
   same messages as for a 1-RTT handshake with PSK resumption. 
         Client                                               Server

         ClientHello
         + early_data
         + key_share*
         + psk_key_exchange_modes
         + pre_shared_key
         (Application Data*)     -------->
                                                         ServerHello
                                                    + pre_shared_key
                                                        + key_share*
                                               {EncryptedExtensions}
                                                       + early_data*
                                                          {Finished}
                                 <--------       [Application Data*]
         (EndOfEarlyData)
         {Finished}              -------->
         [Application Data]      <------->        [Application Data]

               +  Indicates noteworthy extensions sent in the
                  previously noted message.

               *  Indicates optional or situation-dependent
                  messages/extensions that are not always sent.

               () Indicates messages protected using keys
                  derived from a client_early_traffic_secret.

               {} Indicates messages protected using keys
                  derived from a [sender ]_handshake_traffic_secret.

               [] Indicates messages protected using keys
                  derived from [sender ]_application_traffic_secret_N.

               Figure 4: Message Flow for a 0-RTT Handshake 
   IMPORTANT NOTE: The security properties for 0-RTT data are weaker
   than those for other kinds of TLS data.  Specifically:

   1.  This data is not forward secret, as it is encrypted solely under
       keys derived using the offered PSK.

   2.  There are no guarantees of non-replay between connections.
       Protection against replay for ordinary TLS 1.3 1-RTT data is
       provided via the server's Random value, but 0-RTT data does not
       depend on the ServerHello and therefore has weaker guarantees.
       This is especially relevant if the data is authenticated either
       with TLS client authentication or inside the application
       protocol.  The same warnings apply to any use of the
       early_exporter_master_secret.

   0-RTT data cannot be duplicated within a connection (i.e., the server
   will not process the same data twice for the same connection), and an
   attacker will not be able to make 0-RTT data appear to be 1-RTT data
   (because it is protected with different keys).  Appendix E.5 contains
   a description of potential attacks, and Section 8 describes
   mechanisms which the server can use to limit the impact of replay.

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

## 9. Compliance Requirements





### 9.1. Mandatory-to-Implement Cipher Suites

   In the absence of an application profile standard specifying
   otherwise:

   A TLS-compliant application MUST implement the TLS_AES_128_GCM_SHA256
   [GCM ] cipher suite and SHOULD implement the TLS_AES_256_GCM_SHA384
   [GCM ] and TLS_CHACHA20_POLY1305_SHA256 [RFC8439] cipher suites (see Appendix B.4).

   A TLS-compliant application MUST support digital signatures with
   rsa_pkcs1_sha256 (for certificates), rsa_pss_rsae_sha256 (for
   CertificateVerify and certificates), and ecdsa_secp256r1_sha256.  A
   TLS-compliant application MUST support key exchange with secp256r1
   (NIST P-256) and SHOULD support key exchange with X25519 [RFC7748]. 





### 9.2. Mandatory-to-Implement Extensions

   In the absence of an application profile standard specifying
   otherwise, a TLS-compliant application MUST implement the following
   TLS extensions:

   -  Supported Versions ("supported_versions"; Section 4.2.1)

   -  Cookie ("cookie"; Section 4.2.2)

   -  Signature Algorithms ("signature_algorithms"; Section 4.2.3)

   -  Signature Algorithms Certificate ("signature_algorithms_cert";Section 4.2.3)

   -  Negotiated Groups ("supported_groups"; Section 4.2.7)

   -  Key Share ("key_share"; Section 4.2.8)

   -  Server Name Indication ("server_name"; Section 3 of [RFC6066])

   All implementations MUST send and use these extensions when offering
   applicable features:

   -  "supported_versions" is REQUIRED for all ClientHello, ServerHello,
      and HelloRetryRequest messages.

   -  "signature_algorithms" is REQUIRED for certificate authentication.

   -  "supported_groups" is REQUIRED for ClientHello messages using DHE
      or ECDHE key exchange.

   -  "key_share" is REQUIRED for DHE or ECDHE key exchange.

   -  "pre_shared_key" is REQUIRED for PSK key agreement.

   -  "psk_key_exchange_modes" is REQUIRED for PSK key agreement. 
   A client is considered to be attempting to negotiate using this
   specification if the ClientHello contains a "supported_versions"
   extension with 0x0304 contained in its body.  Such a ClientHello
   message MUST meet the following requirements:

   -  If not containing a "pre_shared_key" extension, it MUST contain
      both a "signature_algorithms" extension and a "supported_groups"
      extension.

   -  If containing a "supported_groups" extension, it MUST also contain
      a "key_share" extension, and vice versa.  An empty
      KeyShare.client_shares vector is permitted.

   Servers receiving a ClientHello which does not conform to these
   requirements MUST abort the handshake with a "missing_extension"
   alert.

   Additionally, all implementations MUST support the use of the
   "server_name" extension with applications capable of using it.
   Servers MAY require clients to send a valid "server_name" extension.
   Servers requiring this extension SHOULD respond to a ClientHello
   lacking a "server_name" extension by terminating the connection with
   a "missing_extension" alert.

### 9.3. Protocol Invariants

   This section describes invariants that TLS endpoints and middleboxes
   MUST follow.  It also applies to earlier versions of TLS.

   TLS is designed to be securely and compatibly extensible.  Newer
   clients or servers, when communicating with newer peers, should
   negotiate the most preferred common parameters.  The TLS handshake
   provides downgrade protection: Middleboxes passing traffic between a
   newer client and newer server without terminating TLS should be
   unable to influence the handshake (see Appendix E.1).  At the same
   time, deployments update at different rates, so a newer client or
   server MAY continue to support older parameters, which would allow it
   to interoperate with older endpoints. 
   For this to work, implementations MUST correctly handle extensible
   fields:

   -  A client sending a ClientHello MUST support all parameters
      advertised in it.  Otherwise, the server may fail to interoperate
      by selecting one of those parameters.

   -  A server receiving a ClientHello MUST correctly ignore all
      unrecognized cipher suites, extensions, and other parameters.
      Otherwise, it may fail to interoperate with newer clients.  In
      TLS 1.3, a client receiving a CertificateRequest or
      NewSessionTicket MUST also ignore all unrecognized extensions.

   -  A middlebox which terminates a TLS connection MUST behave as a
      compliant TLS server (to the original client), including having a
      certificate which the client is willing to accept, and also as a
      compliant TLS client (to the original server), including verifying
      the original server's certificate.  In particular, it MUST
      generate its own ClientHello containing only parameters it
      understands, and it MUST generate a fresh ServerHello random
      value, rather than forwarding the endpoint's value.

      Note that TLS's protocol requirements and security analysis only
      apply to the two connections separately.  Safely deploying a TLS
      terminator requires additional security considerations which are
      beyond the scope of this document.

   -  A middlebox which forwards ClientHello parameters it does not
      understand MUST NOT process any messages beyond that ClientHello.
      It MUST forward all subsequent traffic unmodified.  Otherwise, it
      may fail to interoperate with newer clients and servers.

      Forwarded ClientHellos may contain advertisements for features not
      supported by the middlebox, so the response may include future TLS
      additions the middlebox does not recognize.  These additions MAY
      change any message beyond the ClientHello arbitrarily.  In
      particular, the values sent in the ServerHello might change, the
      ServerHello format might change, and the TLSCiphertext format
      might change.

   The design of TLS 1.3 was constrained by widely deployed
   non-compliant TLS middleboxes (see Appendix D.4); however, it does
   not relax the invariants.  Those middleboxes continue to be
   non-compliant. 





## 10. Security Considerations

   Security issues are discussed throughout this memo, especially in
   Appendices C, D, and E.

# Referenced Sections from RFC 6066: Transport Layer Security (TLS) Extensions: Extension Definitions

The following sections were referenced. Remaining sections are not included.

### 1.1. Specific Extensions Covered

   The extensions described here focus on extending the functionality
   provided by the TLS protocol message formats.  Other issues, such as
   the addition of new cipher suites, are deferred.

   The extension types defined in this document are:

      enum {
          server_name(0), max_fragment_length(1),
          client_certificate_url(2), trusted_ca_keys(3),
          truncated_hmac(4), status_request(5), (65535)
      } ExtensionType;

   Specifically, the extensions described in this document:

   -  Allow TLS clients to provide to the TLS server the name of the
      server they are contacting.  This functionality is desirable in
      order to facilitate secure connections to servers that host
      multiple 'virtual' servers at a single underlying network address.

   -  Allow TLS clients and servers to negotiate the maximum fragment
      length to be sent.  This functionality is desirable as a result of
      memory constraints among some clients, and bandwidth constraints
      among some access networks.

   -  Allow TLS clients and servers to negotiate the use of client
      certificate URLs.  This functionality is desirable in order to
      conserve memory on constrained clients. 
   -  Allow TLS clients to indicate to TLS servers which certification
      authority (CA) root keys they possess.  This functionality is
      desirable in order to prevent multiple handshake failures
      involving TLS clients that are only able to store a small number
      of CA root keys due to memory limitations.

   -  Allow TLS clients and servers to negotiate the use of truncated
      Message Authentication Codes (MACs).  This functionality is
      desirable in order to conserve bandwidth in constrained access
      networks.

   -  Allow TLS clients and servers to negotiate that the server sends
      the client certificate status information (e.g., an Online
      Certificate Status Protocol (OCSP) [RFC2560] response) during a
      TLS handshake.  This functionality is desirable in order to avoid
      sending a Certificate Revocation List (CRL) over a constrained
      access network and therefore saving bandwidth.

   TLS clients and servers may use the extensions described in this
   document.  The extensions are designed to be backwards compatible,
   meaning that TLS clients that support the extensions can talk to TLS
   servers that do not support the extensions, and vice versa.

   Note that any messages associated with these extensions that are sent
   during the TLS handshake MUST be included in the hash calculations
   involved in "Finished" messages.

   Note also that all the extensions defined in this document are
   relevant only when a session is initiated.  A client that requests
   session resumption does not in general know whether the server will
   accept this request, and therefore it SHOULD send the same extensions
   as it would send if it were not attempting resumption.  When a client
   includes one or more of the defined extension types in an extended
   client hello while requesting session resumption:

   -  The server name indication extension MAY be used by the server
      when deciding whether or not to resume a session as described in Section 3.

   -  If the resumption request is denied, the use of the extensions is
      negotiated as normal.

   -  If, on the other hand, the older session is resumed, then the
      server MUST ignore the extensions and send a server hello
      containing none of the extension types.  In this case, the
      functionality of these extensions negotiated during the original
      session initiation is applied to the resumed session. 





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

### 11.6. Security Considerations for status_request

   If a client requests an OCSP response, it must take into account that
   an attacker's server using a compromised key could (and probably
   would) pretend not to support the extension.  In this case, a client
   that requires OCSP validation of certificates SHOULD either contact
   the OCSP server directly or abort the handshake.

   Use of the OCSP nonce request extension (id-pkix-ocsp-nonce) may
   improve security against attacks that attempt to replay OCSP
   responses; see Section 4.4.1 of [RFC2560] for further details.

# Referenced Sections from RFC 9525: Service Identity in TLS

The following sections were referenced. Remaining sections are not included.

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

### 7.1. Wildcard Certificates

   Wildcard certificates automatically vouch for any single-label
   hostnames within their domain, but not multiple levels of domains.
   This can be convenient for administrators but also poses the risk of
   vouching for rogue or buggy hosts.  For example, see [Defeating-SSL ]
   (beginning at slide 91) and [HTTPSbytes ] (slides 38-40).

   As specified in Section 6.3, restricting the presented identifiers in
   certificates to only one wildcard character (e.g.,
   "*.bigcompany.example" but not "*.*.bigcompany.example") and
   restricting the use of wildcards to only the left-most domain label
   can help to mitigate certain aspects of the attack described in
   [Defeating-SSL ].

   That same attack also relies on the initial use of a cleartext HTTP
   connection, which is hijacked by an active on-path attacker and
   subsequently upgraded to HTTPS.  In order to mitigate such an attack,
   administrators and software developers are advised to follow the
   strict TLS guidelines provided in [TLS-REC ], Section 3.2.

   Because the attack described in [HTTPSbytes ] relies on an underlying
   cross-site scripting (XSS) attack, web browsers and applications are
   advised to follow best practices to prevent XSS attacks; for example,
   see [XSS ], which was published by the Open Web Application Security
   Project (OWASP).

   Protection against a wildcard that identifies a public suffix
   [Public-Suffix ], such as *.co.uk or *.com, is beyond the scope of
   this document.

   As noted in Section 3, application protocols can disallow the use of
   wildcard certificates entirely as a more foolproof mitigation.

# Referenced Sections from RFC 9257: Guidance for External Pre-Shared Key (PSK) Usage in TLS

The following sections were referenced. Remaining sections are not included.

## 4. PSK Security Properties

   The use of a previously established PSK allows TLS nodes to
   authenticate the endpoint identities.  It also offers other benefits,
   including resistance to attacks in the presence of quantum computers;
   see Section 4.2 for related discussion.  However, these keys do not
   provide privacy protection of endpoint identities, nor do they
   provide non-repudiation (one endpoint in a connection can deny the
   conversation); see Section 7 for related discussion.

   PSK authentication security implicitly assumes one fundamental
   property: each PSK is known to exactly one client and one server and
   they never switch roles.  If this assumption is violated, then the
   security properties of TLS are severely weakened as discussed below.

### 4.1. Shared PSKs

   As discussed in Section 5.1, to demonstrate their attack, [AASS19]
   describes scenarios where multiple clients or multiple servers share
   a PSK.  If this is done naively by having all members share a common
   key, then TLS authenticates only group membership, and the security
   of the overall system is inherently rather brittle.  There are a
   number of obvious weaknesses here:

   1.  Any group member can impersonate any other group member.

   2.  If a PSK is combined with the result of a fresh ephemeral key
       exchange, then compromise of a group member that knows the
       resulting shared secret will enable the attacker to passively
       read traffic (and actively modify it).

   3.  If a PSK is not combined with the result of a fresh ephemeral key
       exchange, then compromise of any group member allows the attacker
       to passively read all traffic (and actively modify it), including
       past traffic.

   Additionally, a malicious non-member can reroute handshakes between
   honest group members to connect them in unintended ways, as described
   below.  Note that a partial mitigation for this class of attack is
   available: each group member includes the Server Name Indication
   (SNI) extension [RFC6066] and terminates the connection on mismatch
   between the presented SNI value and the receiving member's known
   identity.  See [Selfie ] for details.

   To illustrate the rerouting attack, consider three peers, A, B, and
   C, who all know the PSK.  The attack proceeds as follows:

   1.  A sends a ClientHello to B.

   2.  The attacker intercepts the message and redirects it to C.

   3.  C responds with a second flight (ServerHello, ...) to A.

   4.  A sends a Finished message to B.  A has completed the handshake,
       ostensibly with B.

   5.  The attacker redirects the Finished message to C.  C has
       completed the handshake with A.

   In this attack, peer authentication is not provided.  Also, if C
   supports a weaker set of ciphersuites than B, cryptographic algorithm
   downgrade attacks might be possible.  This rerouting is a type of
   identity misbinding attack [Krawczyk ] [Sethi ].  Selfie attack
   [Selfie ] is a special case of the rerouting attack against a group
   member that can act as both a TLS server and a client.  In the Selfie
   attack, a malicious non-member reroutes a connection from the client
   to the server on the same endpoint.

   Finally, in addition to these weaknesses, sharing a PSK across nodes
   may negatively affect deployments.  For example, revocation of
   individual group members is not possible without establishing a new
   PSK for all of the members that have not been revoked.

### 4.2. PSK Entropy

   Entropy properties of external PSKs may also affect TLS security
   properties.  For example, if a high-entropy PSK is used, then PSK-
   only key establishment modes provide expected security properties for
   TLS, including establishment of the same session keys between peers,
   secrecy of session keys, peer authentication, and downgrade
   protection.  See Appendix E.1 of [RFC8446] for an explanation of
   these properties.  However, these modes lack forward security.
   Forward security may be achieved by using a PSK-DH mode or by using
   PSKs with short lifetimes.

   In contrast, if a low-entropy PSK is used, then PSK-only key
   establishment modes are subject to passive exhaustive search attacks,
   which will reveal the traffic keys.  PSK-DH modes are subject to
   active attacks in which the attacker impersonates one side.  The
   exhaustive search phase of these attacks can be mounted offline if
   the attacker captures a single handshake using the PSK, but those
   attacks will not lead to compromise of the traffic keys for that
   connection because those also depend on the Diffie-Hellman (DH)
   exchange.  Low-entropy keys are only secure against active attack if
   a Password-Authenticated Key Exchange (PAKE) is used with TLS.  At
   the time of writing, the Crypto Forum Research Group (CFRG) is
   working on specifying recommended PAKEs (see [CPACE ] and [OPAQUE ] for
   the symmetric and asymmetric cases, respectively).

## 7. Privacy Considerations

   PSK privacy properties are orthogonal to security properties
   described in Section 4.  TLS does little to keep PSK identity
   information private.  For example, an adversary learns information
   about the external PSK or its identifier by virtue of the identifier
   appearing in cleartext in a ClientHello.  As a result, a passive
   adversary can link two or more connections together that use the same
   external PSK on the wire.  Depending on the PSK identity, a passive
   attacker may also be able to identify the device, person, or
   enterprise running the TLS client or TLS server.  An active attacker
   can also use the PSK identity to suppress handshakes or application
   data from a specific device by blocking, delaying, or rate-limiting
   traffic.  Techniques for mitigating these risks require further
   analysis and are out of scope for this document.

   In addition to linkability in the network, external PSKs are
   intrinsically linkable by PSK receivers.  Specifically, servers can
   link successive connections that use the same external PSK together.
   Preventing this type of linkability is out of scope.

## 8. Security Considerations

   Security considerations are provided throughout this document.  It
   bears repeating that there are concerns related to the use of
   external PSKs regarding proper identification of TLS 1.3 endpoints
   and additional risks when external PSKs are known to a group.

   It is NOT RECOMMENDED to share the same PSK between more than one
   client and server.  However, as discussed in Section 5.1, there are
   application scenarios that may rely on sharing the same PSK among
   multiple nodes.  [RFC9258] helps in mitigating rerouting and Selfie-
   style reflection attacks when the PSK is shared among multiple nodes.
   This is achieved by correctly using the node identifiers in the
   ImportedIdentity.context construct specified in [RFC9258].  One
   solution would be for each endpoint to select one globally unique
   identifier to use in all PSK handshakes.  The unique identifier can,
   for example, be one of its Media Access Control (MAC) addresses, a
   32-byte random number, or its Universally Unique IDentifier (UUID)
   [RFC4122].  Note that such persistent, global identifiers have
   privacy implications; see Section 7.

   Each endpoint SHOULD know the identifier of the other endpoint with
   which it wants to connect and SHOULD compare it with the other
   endpoint's identifier used in ImportedIdentity.context.  However, it
   is important to remember that endpoints sharing the same group PSK
   can always impersonate each other.

   Considerations for external PSK usage extend beyond proper
   identification.  When early data is used with an external PSK, the
   random value in the ClientHello is the only source of entropy that
   contributes to key diversity between sessions.  As a result, when an
   external PSK is used more than one time, the random number source on
   the client has a significant role in the protection of the early
   data.

# Referenced Sections from RFC 8907: The Terminal Access Controller Access-Control System Plus (TACACS+) Protocol

The following sections were referenced. Remaining sections are not included.

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

### 10.2. Security of Authentication Sessions

   Authentication sessions SHOULD be used via a secure transport (see
   "TACACS+ Best Practices" (Section 10.5)) as the man-in-the-middle
   attack may completely subvert them.  Even CHAP, which may be
   considered resistant to password interception, is unsafe as it does
   not protect the username from a trivial man-in-the-middle attack.

   This document deprecates the redirection mechanism using the
   TAC_PLUS_AUTHEN_STATUS_FOLLOW option, which was included in "The
   Draft".  As part of this process, the secret key for a new server was
   sent to the client.  This public exchange of secret keys means that
   once one session is broken, it may be possible to leverage that key
   to attacking connections to other servers.  This mechanism MUST NOT
   be used in modern deployments.  It MUST NOT be used outside a secured
   deployment.

### 10.3. Security of Authorization Sessions

   Authorization sessions SHOULD be used via a secure transport (see
   "TACACS+ Best Practices" (Section 10.5)) as it's trivial to execute a
   successful man-in-the-middle attack that changes well-known plaintext
   in either requests or responses.

   As an example, take the field "authen_method".  It's not unusual in
   actual deployments to authorize all commands received via the device
   local serial port (a console port), as that one is usually considered
   secure by virtue of the device located in a physically secure
   location.  If an administrator would configure the authorization
   system to allow all commands entered by the user on a local console
   to aid in troubleshooting, that would give all access to all commands
   to any attacker that would be able to change the "authen_method" from
   TAC_PLUS_AUTHEN_METH_TACACSPLUS to TAC_PLUS_AUTHEN_METH_LINE.  In
   this regard, the obfuscation provided by the protocol itself wouldn't
   help much, because:

   *  A lack of integrity means that any byte in the payload may be
      changed without either side detecting the change.

   *  Known plaintext means that an attacker would know with certainty
      which octet is the target of the attack (in this case, first octet
      after the header).

   *  In combination with known plaintext, the attacker can determine
      with certainty the value of the crypto-pad octet used to obfuscate
      the original octet.

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





