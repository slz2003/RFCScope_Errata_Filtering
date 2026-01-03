Let us analyze section 4 of draft draft-ietf-tls-rfc8446bis-14. All references made by section 4 have also been included below.

# Draft draft-ietf-tls-rfc8446bis-14: Obsoletes: 8446 (if approved)                          13 September 2025

## 1. Introduction

   RFC EDITOR: PLEASE REMOVE THE FOLLOWING PARAGRAPH The source for this
   draft is maintained in GitHub.  Suggested changes should be submitted
   as pull requests at https://github.com/ekr/tls13-spec .  Instructions
   are on that page as well.

   The primary goal of TLS is to provide a secure channel between two
   communicating peers; the only requirement from the underlying
   transport is a reliable, in-order data stream.  Specifically, the
   secure channel should provide the following properties:

   *  Authentication: The server side of the channel is always
      authenticated; the client side is optionally authenticated.
      Authentication can happen via asymmetric cryptography (e.g., RSA
      [RSA ], the Elliptic Curve Digital Signature Algorithm (ECDSA)
      [DSS ], or the Edwards-Curve Digital Signature Algorithm (EdDSA)
      [RFC8032]) or a symmetric pre-shared key (PSK).

   *  Confidentiality: Data sent over the channel after establishment is
      only visible to the endpoints.  TLS does not hide the length of
      the data it transmits, though endpoints are able to pad TLS
      records to obscure lengths and improve protection against traffic
      analysis techniques.

   *  Integrity: Data sent over the channel after establishment cannot
      be modified by attackers without detection.

   These properties should be true even in the face of an attacker who
   has complete control of the network, as described in [RFC3552].  See Appendix F  for a more complete statement of the relevant security
   properties.

   TLS consists of two primary components:
   *  A handshake protocol (Section 4) that authenticates the
      communicating parties, negotiates cryptographic algorithms and
      parameters, and establishes shared keying material.  The handshake
      protocol is designed to resist tampering; an active attacker
      should not be able to force the peers to negotiate different
      parameters than they would if the connection were not under
      attack.

   *  A record protocol (Section 5) that uses the parameters established
      by the handshake protocol to protect traffic between the
      communicating peers.  The record protocol divides traffic up into
      a series of records, each of which is independently protected
      using the traffic keys.

   TLS is application protocol independent; higher-level protocols can
   layer on top of TLS transparently.  The TLS standard, however, does
   not specify how protocols add security with TLS; how to initiate TLS
   handshaking and how to interpret the authentication certificates
   exchanged are left to the judgment of the designers and implementors
   of protocols that run on top of TLS.  Application protocols using TLS
   MUST specify how TLS works with their application protocol, including
   how and when handshaking occurs, and how to do identity verification.
   [RFC9525] provides useful guidance on integrating TLS with
   application protocols.

   This document defines TLS version 1.3.  While TLS 1.3 is not directly
   compatible with previous versions, all versions of TLS incorporate a
   versioning mechanism which allows clients and servers to
   interoperably negotiate a common version if one is supported by both
   peers.

   This document supersedes and obsoletes previous versions of TLS,
   including version 1.2 [RFC5246].  It also obsoletes the TLS ticket
   mechanism defined in [RFC5077] and replaces it with the mechanism
   defined in Section 2.2.  Because TLS 1.3 changes the way keys are
   derived, it updates [RFC5705] as described in Section 7.5.  It also
   changes how Online Certificate Status Protocol (OCSP) messages are
   carried and therefore updates [RFC6066] and obsoletes [RFC6961] as
   described in Section 4.4.2.1.

### 1.1. Conventions and Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 
   14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here. 
   The following terms are used:

   client: The endpoint initiating the TLS connection.

   connection: A transport-layer connection between two endpoints.

   endpoint: Either the client or server of the connection.

   handshake: An initial negotiation between client and server that
   establishes the parameters of their subsequent interactions within
   TLS.

   peer: An endpoint.  When discussing a particular endpoint, "peer"
   refers to the endpoint that is not the primary subject of discussion.

   receiver: An endpoint that is receiving records.

   sender: An endpoint that is transmitting records.

   server: The endpoint that did not initiate the TLS connection.

### 1.2. Relationship to RFC 8446

   TLS 1.3 was originally specified in [RFC8446].  This document is a
   minor update to TLS 1.3 that retains the same version number and is
   backward compatible.  It tightens some requirements and contains
   updated text in areas which were found to be unclear as well as other
   editorial improvements.  In addition, it removes the use of the term
   "master" as applied to secrets in favor of the term "main" or shorter
   names where no term was necessary.  This document makes the following
   specific technical changes:

   *  Forbid negotiating TLS 1.0 and 1.1 as they are now deprecated by
      [RFC8996].

   *  Removes ambiguity around which hash is used with PreSharedKeys and
      HelloRetryRequest.

   *  Require that clients ignore NewSessionTicket if they do not
      support resumption.

   *  Upgrade the requirement to initiate key update before exceeding
      key usage limits to MUST.

   *  Limit the number of permitted KeyUpdate messages.

   *  Restore text defining the level of "close_notify" to "warning". 
   *  Clarify behavior around "user_canceled", requiring that
      "close_notify" be sent and that "user_canceled" should be ignored.

   *  Add a "general_error" generic alert.

   *  Corrected the lower bound on CertificateRequest.extensions to be 0
      bytes.  This was an error in the syntax as it is possible to send
      no extensions, which results in length 0.

   In addition, there have been some improvements to the security
   considerations, especially around privacy.

### 1.3. Major Differences from TLS 1.2

   The following is a list of the major functional differences between
   TLS 1.2 and TLS 1.3.  It is not intended to be exhaustive, and there
   are many minor differences.

   *  The list of supported symmetric encryption algorithms has been
      pruned of all algorithms that are considered legacy.  Those that
      remain are all Authenticated Encryption with Associated Data
      (AEAD) algorithms.  The cipher suite concept has been changed to
      separate the authentication and key exchange mechanisms from the
      record protection algorithm (including secret key length) and a
      hash to be used with both the key derivation function and
      handshake message authentication code (MAC).

   *  A zero round-trip time (0-RTT) mode was added, saving a round trip
      at connection setup for some application data, at the cost of
      certain security properties.

   *  Static RSA and Diffie-Hellman cipher suites have been removed; all
      public-key based key exchange mechanisms now provide forward
      secrecy.

   *  All handshake messages after the ServerHello are now encrypted.
      The newly introduced EncryptedExtensions message allows various
      extensions previously sent in the clear in the ServerHello to also
      enjoy confidentiality protection.

   *  The key derivation function has been redesigned.  The new design
      allows easier analysis by cryptographers due to their improved key
      separation properties.  The HMAC-based Extract-and-Expand Key
      Derivation Function (HKDF) is used as an underlying primitive.

   *  The handshake state machine has been significantly restructured to
      be more consistent and to remove superfluous messages such as
      ChangeCipherSpec (except when needed for middlebox compatibility). 
   *  Elliptic curve algorithms are now in the base spec and new
      signature algorithms, such as EdDSA, are included.  TLS 1.3
      removed point format negotiation in favor of a single point format
      for each curve.

   *  Other cryptographic improvements were made, including changing the
      RSA padding to use the RSA Probabilistic Signature Scheme (RSASSA-
      PSS), and the removal of compression, the Digital Signature
      Algorithm (DSA), and custom Ephemeral Diffie-Hellman (DHE) groups.

   *  The TLS 1.2 version negotiation mechanism has been deprecated in
      favor of a version list in an extension.  This increases
      compatibility with existing servers that incorrectly implemented
      version negotiation.

   *  Session resumption with and without server-side state as well as
      the PSK-based cipher suites of earlier TLS versions have been
      replaced by a single new PSK exchange.

   *  References have been updated to point to the updated versions of
      RFCs, as appropriate (e.g., RFC 5280 rather than RFC 3280).

### 1.4. Updates Affecting TLS 1.2

   This document defines several changes that optionally affect
   implementations of TLS 1.2, including those which do not also support
   TLS 1.3:

   *  A version downgrade protection mechanism is described in Section 4.1.3.

   *  RSASSA-PSS signature schemes are defined in Section 4.2.3.

   *  The "supported_versions" ClientHello extension can be used to
      negotiate the version of TLS to use, in preference to the
      legacy_version field of the ClientHello.

   *  The "signature_algorithms_cert" extension allows a client to
      indicate which signature algorithms it can validate in X.509
      certificates.

   *  The term "master" as applied to secrets has been removed, and the
      "extended_master_secret" extension [RFC7627] has been renamed to
      "extended_main_secret".

   Additionally, this document clarifies some compliance requirements
   for earlier versions of TLS; see Section 9.3. 





## 2. Protocol Overview

   The cryptographic parameters used by the secure channel are produced
   by the TLS handshake protocol.  This sub-protocol of TLS is used by
   the client and server when first communicating with each other.  The
   handshake protocol allows peers to negotiate a protocol version,
   select cryptographic algorithms, authenticate each other (with client
   authentication being optional), and establish shared secret keying
   material.  Once the handshake is complete, the peers use the
   established keys to protect the application-layer traffic.

   A failure of the handshake or other protocol error triggers the
   termination of the connection, optionally preceded by an alert
   message (Section 6).

   TLS supports three basic key exchange modes:

   *  (EC)DHE (Diffie-Hellman over either finite fields or elliptic
      curves)

   *  PSK-only

   *  PSK with (EC)DHE

   Figure 1 below shows the basic full TLS handshake:
       Client                                              Server

Key  ^ ClientHello
Exch | + key_share*
     | + signature_algorithms*
     | + psk_key_exchange_modes*
     v + pre_shared_key*         -------->
                                                       ServerHello  ^ Key
                                                      + key_share*  | Exch
                                                 + pre_shared_key*  v
                                             {EncryptedExtensions}  ^  Server
                                             {CertificateRequest*}  v  Params
                                                    {Certificate*}  ^
                                              {CertificateVerify*}  | Auth
                                                        {Finished}  v
                                 <--------     [Application Data*]
     ^ {Certificate*}
Auth | {CertificateVerify*}
     v {Finished}                -------->
       [Application Data]        <------->      [Application Data]

              +  Indicates noteworthy extensions sent in the
                 previously noted message.

              *  Indicates optional or situation-dependent
                 messages/extensions that are not always sent.

              {} Indicates messages protected using keys
                 derived from a [sender ]_handshake_traffic_secret.

              [] Indicates messages protected using keys
                 derived from [sender ]_application_traffic_secret_N.

            Figure 1: Message Flow for Full TLS Handshake

   The handshake can be thought of as having three phases (indicated in
   the diagram above):

   *  Key Exchange: Establish shared keying material and select the
      cryptographic parameters.  Everything after this phase is
      encrypted.

   *  Server Parameters: Establish other handshake parameters (whether
      the client is authenticated, application-layer protocol support,
      etc.).

   *  Authentication: Authenticate the server (and, optionally, the
      client) and provide key confirmation and handshake integrity. 
   In the Key Exchange phase, the client sends the ClientHello
   (Section 4.1.2) message, which contains a random nonce
   (ClientHello.random); its offered protocol versions; a list of
   symmetric cipher/hash pairs; either a list of Diffie-Hellman key
   shares (in the "key_share" (Section 4.2.8) extension), a list of pre-
   shared key labels (in the "pre_shared_key" (Section 4.2.11)
   extension), or both; and potentially additional extensions.
   Additional fields and/or messages may also be present for middlebox
   compatibility.

   The server processes the ClientHello and determines the appropriate
   cryptographic parameters for the connection.  It then responds with
   its own ServerHello (Section 4.1.3), which indicates the negotiated
   connection parameters.  The combination of the ClientHello and the
   ServerHello determines the shared keys.  If (EC)DHE key establishment
   is in use, then the ServerHello contains a "key_share" extension with
   the server's ephemeral Diffie-Hellman share; the server's share MUST
   be in the same group as one of the client's shares.  If PSK key
   establishment is in use, then the ServerHello contains a
   "pre_shared_key" extension indicating which of the client's offered
   PSKs was selected.  Note that implementations can use (EC)DHE and PSK
   together, in which case both extensions will be supplied.

   The server then sends two messages to establish the Server
   Parameters:

   EncryptedExtensions:  responses to ClientHello extensions that are
      not required to determine the cryptographic parameters, other than
      those that are specific to individual certificates.
      [Section 4.3.1]

   CertificateRequest:  if certificate-based client authentication is
      desired, the desired parameters for that certificate.  This
      message is omitted if client authentication is not desired.
      [Section 4.3.2]

   Finally, the client and server exchange Authentication messages.  TLS
   uses the same set of messages every time that certificate-based
   authentication is needed.  (PSK-based authentication happens as a
   side effect of key exchange.)  Specifically:

   Certificate:  The certificate of the endpoint and any per-certificate 
      extensions.  This message is omitted by the server if not
      authenticating with a certificate and by the client if the server
      did not send CertificateRequest (thus indicating that the client
      should not authenticate with a certificate).  Note that if raw
      public keys [RFC7250] or the cached information extension
      [RFC7924] are in use, then this message will not contain a
      certificate but rather some other value corresponding to the
      server's long-term key.  [Section 4.4.2]

   CertificateVerify:  A signature over the entire handshake using the
      private key corresponding to the public key in the Certificate
      message.  This message is omitted if the endpoint is not
      authenticating via a certificate.  [Section 4.4.3]

   Finished:  A MAC (Message Authentication Code) over the entire
      handshake.  This message provides key confirmation for the shared
      secrets established in the handshake binds the endpoint's identity
      to the exchanged keys, and in PSK mode also authenticates the
      handshake.  [Section 4.4.4]

   Upon receiving the server's messages, the client responds with its
   Authentication messages, namely Certificate and CertificateVerify (if
   requested), and Finished.

   At this point, the handshake is complete, and the client and server
   derive the keying material required by the record layer to exchange
   application-layer data protected through authenticated encryption.
   Application Data MUST NOT be sent prior to sending the Finished
   message, except as specified in Section 2.3.  Note that while the
   server may send Application Data prior to receiving the client's
   Authentication messages, any data sent at that point is, of course,
   being sent to an unauthenticated peer.

### 2.1. Incorrect DHE Share

   If the client has not provided a sufficient "key_share" extension
   (e.g., it includes only DHE or ECDHE groups unacceptable to or
   unsupported by the server), the server corrects the mismatch with a
   HelloRetryRequest and the client needs to restart the handshake with
   an appropriate "key_share" extension, as shown in Figure 2.  If no
   common cryptographic parameters can be negotiated, the server MUST
   abort the handshake with an appropriate alert. 
            Client                                               Server

            ClientHello
            + key_share             -------->
                                                      HelloRetryRequest
                                    <--------               + key_share
            ClientHello
            + key_share             -------->
                                                            ServerHello
                                                            + key_share
                                                  {EncryptedExtensions}
                                                  {CertificateRequest*}
                                                         {Certificate*}
                                                   {CertificateVerify*}
                                                             {Finished}
                                    <--------       [Application Data*]
            {Certificate*}
            {CertificateVerify*}
            {Finished}              -------->
            [Application Data]      <------->        [Application Data]

        Figure 2: Message Flow for a Full Handshake with Mismatched
                                 Parameters

   Note: The handshake transcript incorporates the initial ClientHello/
   HelloRetryRequest exchange; it is not reset with the new ClientHello.

   TLS also allows several optimized variants of the basic handshake, as
   described in the following sections.

### 2.2. Resumption and Pre-Shared Key (PSK)

   Although TLS PSKs can be established externally, PSKs can also be
   established in a previous connection and then used to establish a new
   connection ("session resumption" or "resuming" with a PSK).  Once a
   handshake has completed, the server can send the client a PSK
   identity that corresponds to a unique key derived from the initial
   handshake (see Section 4.6.1).  The client can then use that PSK
   identity in future handshakes to negotiate the use of the associated
   PSK.  If the server accepts the PSK, then the security context of the
   new connection is cryptographically tied to the original connection
   and the key derived from the initial handshake is used to bootstrap
   the cryptographic state instead of a full handshake.  In TLS 1.2 and
   below, this functionality was provided by "session IDs" and "session
   tickets" [RFC5077].  Both mechanisms are obsoleted in TLS 1.3. 
   PSKs can be used with (EC)DHE key exchange to provide forward secrecy
   in combination with shared keys, or can be used alone, at the cost of
   losing forward secrecy for the application data.

   Figure 3 shows a pair of handshakes in which the first handshake
   establishes a PSK and the second handshake uses it:

          Client                                               Server

   Initial Handshake:
          ClientHello
          + key_share               -------->
                                                          ServerHello
                                                          + key_share
                                                {EncryptedExtensions}
                                                {CertificateRequest*}
                                                       {Certificate*}
                                                 {CertificateVerify*}
                                                           {Finished}
                                    <--------     [Application Data*]
          {Certificate*}
          {CertificateVerify*}
          {Finished}                -------->
                                    <--------      [NewSessionTicket]
          [Application Data]        <------->      [Application Data]


   Subsequent Handshake:
          ClientHello
          + key_share*
          + psk_key_exchange_modes
          + pre_shared_key          -------->
                                                          ServerHello
                                                     + pre_shared_key
                                                         + key_share*
                                                {EncryptedExtensions}
                                                           {Finished}
                                    <--------     [Application Data*]
          {Finished}                -------->
          [Application Data]        <------->      [Application Data]

               Figure 3: Message Flow for Resumption and PSK

   As the server is authenticating via a PSK, it does not send a
   Certificate or a CertificateVerify message.  When a client offers
   resumption via a PSK, it SHOULD also supply a "key_share" extension
   to the server to allow the server to decline resumption and fall back
   to a full handshake, if needed.  The server responds with a 
   "pre_shared_key" extension to negotiate the use of PSK key
   establishment and can (as shown here) respond with a "key_share"
   extension to do (EC)DHE key establishment, thus providing forward
   secrecy.

   When PSKs are provisioned externally, the PSK identity and the KDF
   hash algorithm to be used with the PSK MUST also be provisioned.

   Note:  When using an externally provisioned pre-shared secret, a
      critical consideration is using sufficient entropy during the key
      generation, as discussed in [RFC4086].  Deriving a shared secret
      from a password or other low-entropy sources is not secure.  A
      low-entropy secret, or password, is subject to dictionary attacks
      based on the PSK binder.  The specified PSK authentication is not
      a strong password-based authenticated key exchange even when used
      with Diffie-Hellman key establishment.  Specifically, it does not
      prevent an attacker that can observe the handshake from performing
      a brute-force attack on the password/pre-shared key.

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

   1.  The protocol does not provide any forward secrecy guarantees for
       this data.  The server's behavior determines what forward secrecy
       guarantees, if any, apply (see Section 8.1).  This behavior is
       not communicated to the client as part of the protocol.
       Therefore, absent out-of-band knowledge of the server's behavior,
       the client should assume that this data is not forward secret. 
   2.  There are no guarantees of non-replay between connections.
       Protection against replay for ordinary TLS 1.3 1-RTT data is
       provided via the server's Random value, but 0-RTT data does not
       depend on the ServerHello and therefore has weaker guarantees.
       This is especially relevant if the data is authenticated either
       with TLS client authentication or inside the application
       protocol.  The same warnings apply to any use of the
       early_exporter_secret.

   0-RTT data cannot be duplicated within a connection (i.e., the server
   will not process the same data twice for the same connection), and an
   attacker will not be able to make 0-RTT data appear to be 1-RTT data
   (because it is protected with different keys).  Appendix F.5 contains
   a description of potential attacks, and Section 8 describes
   mechanisms which the server can use to limit the impact of replay.

## 3. Presentation Language

   This document deals with the formatting of data in an external
   representation.  The following very basic and somewhat casually
   defined presentation syntax will be used.

   In the definitions below, optional components of this syntax are
   denoted by enclosing them in "[[ ]]" (double brackets).

### 3.1. Basic Block Size

   The representation of all data items is explicitly specified.  The
   basic data block size is one byte (i.e., 8 bits).  Multiple-byte data
   items are concatenations of bytes, from left to right, from top to
   bottom.  From the byte stream, a multi-byte item (a numeric in the
   following example) is formed (using C notation) by:

      value = (byte[0] << 8*(n-1)) | (byte[1] << 8*(n-2)) |
              ... | byte[n-1];

   This byte ordering for multi-byte values is the commonplace network
   byte order or big-endian format.

### 3.2. Miscellaneous

   Comments begin with "/*" and end with "*/".

   Single-byte entities containing uninterpreted data are of type
   opaque.

   A type alias T' for an existing type T is defined by:
      T T';

### 3.3. Numbers

   The basic numeric data type is an unsigned byte (uint8).  All larger
   numeric data types are constructed from a fixed-length series of
   bytes concatenated as described in Section 3.1 and are also unsigned.
   The following numeric types are predefined.

      uint8 uint16[2];
      uint8 uint24[3];
      uint8 uint32[4];
      uint8 uint64[8];

   All values, here and elsewhere in the specification, are transmitted
   in network byte (big-endian) order; the uint32 represented by the hex
   bytes 01 02 03 04 is equivalent to the decimal value 16909060.

### 3.4. Vectors

   A vector (single-dimensioned array) is a stream of homogeneous data
   elements.  For presentation purposes, this specification refers to
   vectors as lists.  The size of the vector may be specified at
   documentation time or left unspecified until runtime.  In either
   case, the length declares the number of bytes, not the number of
   elements, in the vector.  The syntax for specifying a new type, T',
   that is a fixed-length vector of type T is

      T T'[n];

   Here, T' occupies n bytes in the data stream, where n is a multiple
   of the size of T.  The length of the vector is not included in the
   encoded stream.

   In the following example, Datum is defined to be three consecutive
   bytes that the protocol does not interpret, while Data is three
   consecutive Datum, consuming a total of nine bytes.

      opaque Datum[3];      /* three uninterpreted bytes */
      Datum Data[9];        /* three consecutive 3-byte vectors */

   Variable-length vectors are defined by specifying a subrange of legal
   lengths, inclusively, using the notation <floor..ceiling>.  When
   these are encoded, the actual length precedes the vector's contents
   in the byte stream.  The length will be in the form of a number
   consuming as many bytes as required to hold the vector's specified
   maximum (ceiling) length.  A variable-length vector with an actual
   length field of zero is referred to as an empty vector. 
      T T'<floor..ceiling>;

   In the following example, "mandatory" is a vector that must contain
   between 300 and 400 bytes of type opaque.  It can never be empty.
   The actual length field consumes two bytes, a uint16, which is
   sufficient to represent the value 400 (see Section 3.3).  Similarly,
   "longer" can represent up to 800 bytes of data, or 400 uint16
   elements, and it may be empty.  Its encoding will include a two-byte
   actual length field prepended to the vector.  The length of an
   encoded vector must be an exact multiple of the length of a single
   element (e.g., a 17-byte vector of uint16 would be illegal).

      opaque mandatory<300..400>;
            /* length field is two bytes, cannot be empty */
      uint16 longer<0..800>;
            /* zero to 400 16-bit unsigned integers */

### 3.5. Enumerateds

   An additional sparse data type, called "enum" or "enumerated", is
   available.  Each definition is a different type.  Only enumerateds of
   the same type may be assigned or compared.  Every element of an
   enumerated must be assigned a value, as demonstrated in the following
   example.  Since the elements of the enumerated are not ordered, they
   can be assigned any unique value, in any order.

      enum { e1(v1), e2(v2), ... , en(vn) [[, (n)]] } Te;

   Future extensions or additions to the protocol may define new values.
   Implementations need to be able to parse and ignore unknown values
   unless the definition of the field states otherwise.

   An enumerated occupies as much space in the byte stream as would its
   maximal defined ordinal value.  The following definition would cause
   one byte to be used to carry fields of type Color.

      enum { red(3), blue(5), white(7) } Color;

   One may optionally specify a value without its associated tag to
   force the width definition without defining a superfluous element.

   In the following example, Taste will consume two bytes in the data
   stream but can only assume the values 1, 2, or 4 in the current
   version of the protocol.

      enum { sweet(1), sour(2), bitter(4), (32000) } Taste;
   The names of the elements of an enumeration are scoped within the
   defined type.  In the first example, a fully qualified reference to
   the second element of the enumeration would be Color.blue.  Such
   qualification is not required if the target of the assignment is well
   specified.

      Color color = Color.blue;     /* overspecified, legal */
      Color color = blue;           /* correct, type implicit */

   The names assigned to enumerateds do not need to be unique.  The
   numerical value can describe a range over which the same name
   applies.  The value includes the minimum and maximum inclusive values
   in that range, separated by two period characters.  This is
   principally useful for reserving regions of the space.

      enum { sad(0), meh(1..254), happy(255) } Mood;

### 3.6. Constructed Types

   Structure types may be constructed from primitive types for
   convenience.  Each specification declares a new, unique type.  The
   syntax used for definitions is much like that of C.

      struct {
          T1 f1;
          T2 f2;
          ...
          Tn fn;
      } T;

   Fixed- and variable-length list (vector) fields are allowed using the
   standard list syntax.  Structures V1 and V2 in the variants example
   (Section 3.8) demonstrate this.

   The fields within a structure may be qualified using the type's name,
   with a syntax much like that available for enumerateds.  For example,
   T.f2 refers to the second field of the previous declaration.

### 3.7. Constants

   Fields and variables may be assigned a fixed value using "=", as in:

      struct {
          T1 f1 = 8;  /* T.f1 must always be 8 */
          T2 f2;
      } T;





### 3.8. Variants

   Defined structures may have variants based on some knowledge that is
   available within the environment.  The selector must be an enumerated
   type that defines the possible variants the structure defines.  Each
   arm of the select (below) specifies the type of that variant's field
   and an optional field label.  The mechanism by which the variant is
   selected at runtime is not prescribed by the presentation language.

      struct {
          T1 f1;
          T2 f2;
          ....
          Tn fn;
          select (E) {
              case e1: Te1 [[fe1]];
              case e2: Te2 [[fe2]];
              ....
              case en: Ten [[fen]];
          };
      } Tv;

   For example:

      enum { apple(0), orange(1) } VariantTag;

      struct {
          uint16 number;
          opaque string<0..10>; /* variable length */
      } V1;

      struct {
          uint32 number;
          opaque string[10];    /* fixed length */
      } V2;

      struct {
          VariantTag type;
          select (VariantRecord.type) {
              case apple:  V1;
              case orange: V2;
          };
      } VariantRecord;





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

   *  A list of cipher suites which indicates the AEAD algorithm/HKDF
      hash pairs which the client supports.

   *  A "supported_groups" (Section 4.2.7) extension which indicates the
      (EC)DHE groups which the client supports and a "key_share"
      (Section 4.2.8) extension which contains (EC)DHE shares for some
      or all of these groups.

   *  A "signature_algorithms" (Section 4.2.3) extension which indicates
      the signature algorithms which the client can accept.  A
      "signature_algorithms_cert" extension (Section 4.2.3) may also be
      added to indicate certificate-specific signature algorithms.

   *  A "pre_shared_key" (Section 4.2.11) extension which contains a
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
   establishment mode from the list indicated by the client's
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

   *  If PSK is being used, then the server will send a "pre_shared_key"
      extension indicating the selected key.

   *  When (EC)DHE is in use, the server will also provide a "key_share"
      extension.  If PSK is not being used, then (EC)DHE and
      certificate-based authentication are always used.

   *  When authenticating via a certificate, the server will send the
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

   *  If a "key_share" extension was supplied in the HelloRetryRequest,
      replacing the list of shares with a list containing a single
      KeyShareEntry from the indicated group.

   *  Removing the "early_data" extension (Section 4.2.10) if one was
      present.  Early data is not permitted after a HelloRetryRequest.

   *  Including a "cookie" extension if one was provided in the
      HelloRetryRequest.

   *  Updating the "pre_shared_key" extension if present by recomputing
      the "obfuscated_ticket_age" and binder values and (optionally)
      removing any PSKs which are incompatible with the server's
      indicated cipher suite. 
   *  Optionally adding, removing, or changing the length of the
      "padding" extension [RFC7685].

   *  Other modifications that may be allowed by an extension defined in
      the future and present in the HelloRetryRequest.

   Because TLS 1.3 forbids renegotiation, if a server has negotiated TLS
   1.3 and receives a ClientHello at any other time, it MUST terminate
   the connection with an "unexpected_message" alert.

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
      ClientHello with a version number higher than it supports.  In TLS
      1.3, the client indicates its version preferences in the
      "supported_versions" extension (Section 4.2.1) and the
      legacy_version field MUST be set to 0x0303, which is the version
      number for TLS 1.2.  TLS 1.3 ClientHellos are identified as having
      a legacy_version of 0x0303 and a supported_versions extension
      present with 0x0304 as the highest version indicated therein.
      (See Appendix E  for details about backward compatibility.)  A
      server which receives a legacy_version value not equal to 0x0303
      MUST abort the handshake with an "illegal_parameter" alert.

   random:  32 bytes generated by a secure random number generator.  See 



      Appendix C  for additional information.

   legacy_session_id:  Versions of TLS before TLS 1.3 supported a
      "session resumption" feature which has been merged with pre-shared
      keys in this version (see Section 2.2).  A client which has a
      cached session ID set by a pre-TLS 1.3 server SHOULD set this
      field to that value.  In compatibility mode (see Appendix E.4),
      this field MUST be non-empty, so a client not offering a pre-TLS
      1.3 session MUST generate a new 32-byte value.  This value need
      not be random but SHOULD be unpredictable to avoid implementations
      fixating on a specific value (also known as ossification).
      Otherwise, it MUST be set as a zero-length list (i.e., a zero-
      valued single byte length field).

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
      sent in this field.  For every TLS 1.3 ClientHello, this list MUST
      contain exactly one byte, set to zero, which corresponds to the
      "null" compression method in prior versions of TLS.  If a TLS 1.3
      ClientHello is received with any other value in this field, the
      server MUST abort the handshake with an "illegal_parameter" alert.
      Note that TLS 1.3 servers might receive TLS 1.2 or prior
      ClientHellos which contain other compression methods and (if
      negotiating such a prior version) MUST follow the procedures for
      the appropriate prior version of TLS.

   extensions:  Clients request extended functionality from servers by
      sending data in the extensions field.  The actual "Extension"
      format is defined in Section 4.2.  In TLS 1.3, the use of certain
      extensions is mandatory, as functionality has moved into
      extensions to preserve ClientHello compatibility with previous
      versions of TLS.  Servers MUST ignore unrecognized extensions.

   All versions of TLS allow an extensions field to optionally follow
   the compression_methods field.  TLS 1.3 ClientHello messages always
   contain extensions (minimally "supported_versions", otherwise, they
   will be interpreted as TLS 1.2 ClientHello messages).  However, TLS 
   1.3 servers might receive ClientHello messages without an extensions
   field from prior versions of TLS.  The presence of extensions can be
   detected by determining whether there are bytes following the
   compression_methods field at the end of the ClientHello.  Note that
   this method of detecting optional data differs from the normal TLS
   method of having a variable-length field, but it is used for
   compatibility with TLS before extensions were defined.  TLS 1.3
   servers will need to perform this check first and only attempt to
   negotiate TLS 1.3 if the "supported_versions" extension is present.
   If negotiating a version of TLS prior to 1.3, a server MUST check
   that the message either contains no data after
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
      0x0303, which is the version number for TLS 1.2.  (See Appendix E 




      for details about backward compatibility.)  A client which
      receives a TLS 1.3 Server Hello with a legacy_version value not
      equal to 0x0303 MUST abort the handshake with an
      "illegal_parameter" alert.

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
      the ClientHello.cipher_suites list.  A client which receives a
      cipher suite that was not offered MUST abort the handshake with an
      "illegal_parameter" alert.

   legacy_compression_method:  A single byte which MUST have the value
      0.  If a TLS 1.3 ServerHello is received with any other value in
      this field, the client MUST abort the handshake with an
      "illegal_parameter" alert.

   extensions:  A list of extensions.  The ServerHello MUST only include
      extensions which are required to establish the cryptographic
      context and negotiate the protocol version.  All TLS 1.3
      ServerHello messages MUST contain the "supported_versions"
      extension.  Current ServerHello messages additionally contain
      either the "pre_shared_key" extension or the "key_share"
      extension, or both (when using a PSK with (EC)DHE key
      establishment).  Other extensions (see Section 4.2) are sent
      separately in the EncryptedExtensions message.

   For reasons of backward compatibility with middleboxes (see Appendix E.4), the HelloRetryRequest message uses the same structure
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

   [RFC8996] and Appendix E.5 forbid the negotiation of TLS versions
   below 1.2.  However, server implementations which do not follow that
   guidance MUST set the last 8 bytes of their ServerHello.random value
   to the bytes:

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
   used.  It does not provide downgrade protection when static RSA is
   used.

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
   necessary for the client to generate a correct ClientHello pair.  A
   HelloRetryRequest MUST NOT contain any extensions that were not first
   offered by the client in its ClientHello, with the exception of
   optionally the "cookie" (see Section 4.2.2) extension.

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

   *  supported_versions (see Section 4.2.1)

   *  cookie (see Section 4.2.2)

   *  key_share (see Section 4.2.8)

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
   *  "extension_type" identifies the particular extension type.

   *  "extension_data" contains information specific to the particular
      extension type.

   The contents of the "extension_data" field are typically defined by
   an extension-specific structure defined in the TLS presentation
   language.  Unless otherwise specified, trailing data is forbidden.
   That is, senders MUST NOT include data after the structure in the
   "extension_data" field.  When processing an extension, receivers MUST
   abort the handshake with a "decode_error" alert if there is data left
   over after parsing the structure.  This does not apply if the
   receiver does not implement or is configured to ignore an extension.

   The list of extension types is maintained by IANA as described in Section 11.

   Extensions are generally structured in a request/response fashion,
   though some extensions are just requests with no corresponding
   response (i.e., indications).  The client sends its extension
   requests in the ClientHello message, and the server sends its
   extension responses in the ServerHello, EncryptedExtensions,
   HelloRetryRequest, and Certificate messages.  The server sends
   extension requests in the CertificateRequest message which a client
   MAY respond to with a Certificate message.  The server MAY also send
   unsolicited extensions in the NewSessionTicket, though the client
   does not respond directly to these.

   Implementations MUST NOT send extension responses (i.e., in the
   ServerHello, EncryptedExtensions, HelloRetryRequest, and Certificate
   messages) if the remote endpoint did not send the corresponding
   extension requests, with the exception of the "cookie" extension in
   the HelloRetryRequest.  Upon receiving such an extension, an endpoint
   MUST abort the handshake with an "unsupported_extension" alert.

   The table below indicates the messages where a given extension may
   appear, using the following notation: CH (ClientHello), SH
   (ServerHello), EE (EncryptedExtensions), CT (Certificate), CR
   (CertificateRequest), NST (NewSessionTicket), and HRR
   (HelloRetryRequest).  If an implementation receives an extension
   which it recognizes and which is not specified for the message in
   which it appears, it MUST abort the handshake with an
   "illegal_parameter" alert. 
    +==================================================+=============+
    | Extension                                        |     TLS 1.3 |
    +==================================================+=============+
    | server_name [RFC6066]                            |      CH, EE |
    +--------------------------------------------------+-------------+
    | max_fragment_length [RFC6066]                    |      CH, EE |
    +--------------------------------------------------+-------------+
    | status_request [RFC6066]                         |  CH, CR, CT |
    +--------------------------------------------------+-------------+
    | supported_groups [RFC7919]                       |      CH, EE |
    +--------------------------------------------------+-------------+
    | signature_algorithms [RFC8446]                   |      CH, CR |
    +--------------------------------------------------+-------------+
    | use_srtp [RFC5764]                               |      CH, EE |
    +--------------------------------------------------+-------------+
    | heartbeat [RFC6520]                              |      CH, EE |
    +--------------------------------------------------+-------------+
    | application_layer_protocol_negotiation [RFC7301] |      CH, EE |
    +--------------------------------------------------+-------------+
    | signed_certificate_timestamp [RFC6962]           |  CH, CR, CT |
    +--------------------------------------------------+-------------+
    | client_certificate_type [RFC7250]                |      CH, EE |
    +--------------------------------------------------+-------------+
    | server_certificate_type [RFC7250]                |      CH, EE |
    +--------------------------------------------------+-------------+
    | padding [RFC7685]                                |          CH |
    +--------------------------------------------------+-------------+
    | cached_info [RFC7924]                            |      CH, EE |
    +--------------------------------------------------+-------------+
    | compress_certificate [RFC8879]                   |      CH, CR |
    +--------------------------------------------------+-------------+
    | record_size_limit [RFC8849]                      |      CH, EE |
    +--------------------------------------------------+-------------+
    | delegated_credentials [RFC9345]                  |  CH, CR, CT |
    +--------------------------------------------------+-------------+
    | supported_ekt_ciphers [RFC8870]                  |      CH, EE |
    +--------------------------------------------------+-------------+
    | pre_shared_key [RFC8446]                         |      CH, SH |
    +--------------------------------------------------+-------------+
    | early_data [RFC8446]                             | CH, EE, NST |
    +--------------------------------------------------+-------------+
    | psk_key_exchange_modes [RFC8446]                 |          CH |
    +--------------------------------------------------+-------------+
    | cookie [RFC8446]                                 |     CH, HRR |
    +--------------------------------------------------+-------------+
    | supported_versions [RFC8446]                     | CH, SH, HRR |
    +--------------------------------------------------+-------------+
    | certificate_authorities [RFC8446]                |      CH, CR |
    +--------------------------------------------------+-------------+
    | oid_filters [RFC8446]                            |          CR |
    +--------------------------------------------------+-------------+
    | post_handshake_auth [RFC8446]                    |          CH |
    +--------------------------------------------------+-------------+
    | signature_algorithms_cert [RFC8446]              |      CH, CR |
    +--------------------------------------------------+-------------+
    | key_share [RFC8446]                              | CH, SH, HRR |
    +--------------------------------------------------+-------------+
    | transparency_info [RFC9162]                      |  CH, CR, CT |
    +--------------------------------------------------+-------------+
    | connection_id [RFC9146]                          |      CH, SH |
    +--------------------------------------------------+-------------+
    | external_id_hash [RFC8844]                       |      CH, EE |
    +--------------------------------------------------+-------------+
    | external_session_id [RFC8844]                    |      CH, EE |
    +--------------------------------------------------+-------------+
    | quic_transport_parameters [RFC9001]              |      CH, EE |
    +--------------------------------------------------+-------------+
    | ticket_request [RFC9149]                         |      CH, EE |
    +--------------------------------------------------+-------------+

                         Table 1: TLS Extensions

   Note: this table includes only extensions marked "Recommended" at the
   time of this writing.

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
   *  Some cases where a server does not agree to an extension are error
      conditions (e.g., the handshake cannot continue), and some are
      simply refusals to support particular features.  In general, error
      alerts should be used for the former and a field in the server
      extension response for the latter.

   *  Extensions should, as far as possible, be designed to prevent any
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
   this specification and which also support TLS 1.2 MUST negotiate TLS
   1.2 or prior as specified in [RFC5246], even if
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
   0x0303 (TLS 1.2).

   After checking ServerHello.random to determine if the server
   handshake message is a ServerHello or HelloRetryRequest, clients MUST
   check for this extension prior to processing the rest of the
   ServerHello.  This will require clients to parse the ServerHello to
   read the extension.  If this extension is present, clients MUST
   ignore the ServerHello.legacy_version value and MUST use only the
   "supported_versions" extension to determine the selected version.  If
   the "supported_versions" extension in the ServerHello contains a
   version not offered by the client or contains a version prior to TLS
   1.3, the client MUST abort the handshake with an "illegal_parameter"
   alert.

#### 4.2.2. Cookie

      struct {
          opaque cookie<1..2^16-1>;
      } Cookie;

   Cookies serve two primary purposes:

   *  Allowing the server to force the client to demonstrate
      reachability at their apparent network address (thus providing a
      measure of DoS protection).  This is primarily useful for non-
      connection-oriented transports (see [RFC6347] for an example of
      this). 
   *  Allowing the server to offload state to the client, thus allowing
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
      "signature_algorithms_cert" for backward compatibility with TLS
      1.2.

   ECDSA algorithms:  Indicates a signature algorithm using ECDSA [DSS ],
      the corresponding curve as defined in NIST SP 800-186 [ECDP ], and
      the corresponding hash algorithm as defined in [SHS ].  The
      signature is represented as a DER-encoded [X690] ECDSA-Sig-Value
      structure as defined in [RFC4492].

   RSASSA-PSS RSAE algorithms:  Indicates a signature algorithm using
      RSASSA-PSS with a mask generation function of MGF1, as defined in
      [RFC8017].  The digest used in MGF1 and the digest being signed
      are both the corresponding hash algorithm as defined in [SHS ].
      The length of the Salt MUST be equal to the length of the output
      of the digest algorithm.  If the public key is carried in an X.509
      certificate, it MUST use the rsaEncryption OID [RFC5280].

   EdDSA algorithms:  Indicates a signature algorithm using EdDSA as
      defined in [RFC8032] or its successors.  Note that these
      correspond to the "PureEdDSA" algorithms and not the "prehash"
      variants.

   RSASSA-PSS PSS algorithms:  Indicates a signature algorithm using
      RSASSA-PSS with a mask generation function of MGF1, as defined in
      [RFC8017].  The digest used in MGF1 and the digest being signed
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
      "signature_algorithms_cert" for backward compatibility with TLS
      1.2.  Endpoints SHOULD NOT negotiate these algorithms but are
      permitted to do so solely for backward compatibility.  Clients
      offering these values MUST list them as the lowest priority
      (listed after all other algorithms in SignatureSchemeList).  TLS
      1.3 servers MUST NOT offer a SHA-1 signed certificate unless no
      valid certificate chain can be produced without it (see Section 4.4.2.2).

   The signatures on certificates that are self-signed or certificates
   that are trust anchors are not validated, since they begin a
   certification path (see [RFC5280], Section 3.2).  A certificate that
   begins a certification path MAY use a signature algorithm that is not
   advertised as being supported in the "signature_algorithms" and
   "signature_algorithms_cert" extensions.

   Note that TLS 1.2 defines this extension differently.  TLS 1.3
   implementations willing to negotiate TLS 1.2 MUST behave in
   accordance with the requirements of [RFC5246] when negotiating that
   version.  In particular:

   *  TLS 1.2 ClientHellos MAY omit this extension.

   *  In TLS 1.2, the extension contained hash/signature pairs.  The
      pairs are encoded in two octets, so SignatureScheme values have
      been allocated to align with TLS 1.2's encoding.  Some legacy
      pairs are left unallocated.  These algorithms are deprecated as of
      TLS 1.3.  They MUST NOT be offered or negotiated by any
      implementation.  In particular, MD5 [SLOTH ], SHA-224, and DSA MUST
      NOT be used.

   *  ECDSA signature schemes align with TLS 1.2's ECDSA hash/signature
      pairs.  However, the old semantics did not constrain the signing
      curve.  If TLS 1.2 is negotiated, implementations MUST be prepared
      to accept a signature that uses any curve that they advertised in
      the "supported_groups" extension.

   *  Implementations that advertise support for RSASSA-PSS (which is
      mandatory in TLS 1.3) MUST be prepared to accept a signature using
      that scheme even when TLS 1.2 is negotiated.  In TLS 1.2, RSASSA-
      PSS is used with RSA cipher suites. 





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
   purpose, but is more complicated, is not used in TLS 1.3 (although it
   may appear in ClientHello messages from clients which are offering
   prior versions of TLS).

#### 4.2.5. OID Filters

   The "oid_filters" extension allows servers to provide a list of OID/
   value pairs which it would like the client's certificate to match.
   This extension, if provided by the server, MUST only be sent in the
   CertificateRequest message.

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

   *  The Key Usage extension in a certificate matches the request when
      all key usage bits asserted in the request are also asserted in
      the Key Usage certificate extension.

   *  The Extended Key Usage extension in a certificate matches the
      request when all key purpose OIDs present in the request are also
      found in the Extended Key Usage certificate extension.  The
      special anyExtendedKeyUsage OID MUST NOT be used in the request.

   Separate specifications may define matching rules for other
   certificate extensions.

#### 4.2.6. Post-Handshake Certificate-Based Client Authentication

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
      corresponding named curve, defined in either NIST SP 800-186
      [ECDP ] or in [RFC7748].  Values 0xFE00 through 0xFEFF are reserved
      for Private Use [RFC8126].

   Finite Field Groups (DHE):  Indicates support for the corresponding
      finite field group, defined in [RFC7919].  Values 0x01FC through
      0x01FF are reserved for Private Use. 
   Items in "named_group_list" are ordered according to the sender's
   preferences (most preferred choice first).  The "named_group_list"
   MUST NOT contain any duplicate entries.  A recipient MAY abort a
   connection with a fatal illegal_parameter alert if it detects a
   duplicate entry.

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

   Clients MAY send an empty client_shares list to request group
   selection from the server, at the cost of an additional round trip
   (see Section 4.1.4).

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

   This list MAY be empty if the client is requesting a
   HelloRetryRequest.  Each KeyShareEntry value MUST correspond to a
   group offered in the "supported_groups" extension and MUST appear in
   the same order.  However, the values MAY be a non-contiguous subset
   of the "supported_groups" extension and MAY omit the most preferred
   groups.  Such a situation could arise if the most preferred groups
   are new and unlikely to be supported in enough places to make
   pregenerating key shares for them efficient.

   For this reason, the omission of a share for group A and inclusion of
   one for group B does not mean that the client prefers B to A.
   Selecting a group based on KeyShareEntry may result in the use of a
   less preferred group than the client and server mutually support,
   though saving the round trip of HelloRetryRequest.  Servers that wish
   to respect the client's group preferences SHOULD first select a group
   based on "supported_groups" and then either send a ServerHello or a
   HelloRetryRequest depending on the contents of KeyshareClienthello.

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
   server has selected for the negotiated key exchange.  Servers MUST
   NOT send a KeyShareEntry for any group not indicated in the client's
   "supported_groups" extension and MUST NOT send a KeyShareEntry when
   using the "psk_ke" PskKeyExchangeMode.  If using (EC)DHE key
   establishment and a HelloRetryRequest containing a "key_share"
   extension was received by the client, the client MUST verify that the
   selected NamedGroup in the ServerHello is the same as that in the
   HelloRetryRequest.  If this check fails, the client MUST abort the
   handshake with an "illegal_parameter" alert.

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
   procedures are defined in Appendix D.1 of [ECDP ] and alternatively in
   Section 5.6.2.3 of [KEYAGREEMENT ].  This process consists of three
   steps: (1) verify that Q is not the point at infinity (O), (2) verify
   that for Q = (x, y) both integers x and y are in the correct
   interval, and (3) ensure that (x, y) is a correct solution to the
   elliptic curve equation.  For these curves, implementors do not need
   to verify membership in the correct subgroup.

   For X25519 and X448, the content of the public value is the K_A or
   K_B value described in Section 6 of [RFC7748].  This is 32 bytes for
   X25519 and 56 bytes for X448.

   Note: Versions of TLS prior to 1.3 permitted point format
   negotiation; TLS 1.3 removes this feature in favor of a single point
   format for each curve.

#### 4.2.9. Pre-Shared Key Exchange Modes

   To use PSKs, clients MUST also send a "psk_key_exchange_modes"
   extension.  The semantics of this extension are that the client only
   supports the use of PSKs with these modes, which restricts both the
   use of PSKs offered in this ClientHello and those which the server
   might supply via NewSessionTicket.

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

   psk_ke:  PSK-only key establishment.  In this mode, the server MUST
      NOT supply a "key_share" value.

   psk_dhe_ke:  PSK with (EC)DHE key establishment.  In this mode, the
      client and server MUST supply "key_share" values as described in Section 4.2.8.

   Any future values that are allocated must ensure that the transmitted
   protocol messages unambiguously identify which mode was selected by
   the server; at present, this is indicated by the presence of the
   "key_share" in the ServerHello.

#### 4.2.10. Early Data Indication

   When a PSK is used and early data is allowed for that PSK (see for
   instance Appendix B.3.4), the client can send Application Data in its
   first flight of messages.  If the client opts to do so, it MUST
   supply both the "pre_shared_key" and "early_data" extensions.

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

   *  Ignore the extension and return a regular 1-RTT response.  The
      server then skips past early data by attempting to deprotect
      received records using the handshake traffic key, discarding
      records which fail deprotection (up to the configured
      max_early_data_size).  Once a record is deprotected successfully,
      it is treated as the start of the client's second flight and the
      server proceeds as with an ordinary 1-RTT handshake.

   *  Request that the client send another ClientHello by responding
      with a HelloRetryRequest.  A client MUST NOT include the
      "early_data" extension in its followup ClientHello.  The server
      then ignores early data by skipping all records with an external
      content type of "application_data" (indicating that they are
      encrypted), up to the configured max_early_data_size. 
   *  Return its own "early_data" extension in EncryptedExtensions,
      indicating that it intends to process the early data.  It is not
      possible for the server to accept only a subset of the early data
      messages.  Even though the server sends a message accepting early
      data, the actual early data itself may already be in flight by the
      time the server generates this message.

   In order to accept early data, the server MUST have selected the
   first key offered in the client's "pre_shared_key" extension.  In
   addition, it MUST verify that the following values are the same as
   those associated with the selected PSK:

   *  The selected TLS version number

   *  The selected cipher suite

   *  The selected ALPN [RFC7301] protocol, if any

   These requirements are a superset of those needed to perform a 1-RTT
   handshake using the PSK in question.

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
      (0-based) index into the identities in the client's
      "OfferedPsks.identities" list.

   Each PSK is associated with a single Hash algorithm.  For PSKs
   established via the ticket mechanism (Section 4.6.1), this is the KDF
   Hash algorithm on the connection where the ticket was established.
   For externally established PSKs, the Hash algorithm MUST be set when
   the PSK is established or default to SHA-256 if no such algorithm is
   defined.  The server MUST ensure that it selects a compatible PSK (if
   any) and cipher suite.

   In TLS versions prior to TLS 1.3, the Server Name Indication (SNI)
   value was intended to be associated with the session (Section 3 of
   [RFC6066]), with the server being required to enforce that the SNI
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
   binder that corresponds to that PSK.  See Section 8.2 and Appendix F.6 for the security rationale for this requirement.  To
   accept PSK key establishment, the server sends a "pre_shared_key"
   extension indicating the selected identity.

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
   observers from correlating connections unless tickets or key shares
   are reused.  Note that the "ticket_lifetime" field in the
   NewSessionTicket message is in seconds but the
   "obfuscated_ticket_age" is in milliseconds.  Because ticket lifetimes
   are restricted to a week, 32 bits is enough to represent any
   plausible age, even in milliseconds.

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

   Where Truncate() removes the binders list from the ClientHello.  Note
   that this hash will be computed using the hash associated with the
   PSK, as the client does not know which cipher suite the server will
   select.

   If the server responds with a HelloRetryRequest and the client then
   sends ClientHello2, its binder will be computed over:

      Transcript-Hash(ClientHello1,
                      HelloRetryRequest,
                      Truncate(ClientHello2))

   The full ClientHello1/ClientHello2 is included in all other handshake
   hash computations.  Note that in the first flight,
   Truncate(ClientHello1) is hashed directly, but in the second flight,
   ClientHello1 is hashed and then reinjected as a "message_hash"
   message, as described in Section 4.4.1.  Note that the "message_hash"
   will be hashed with the negotiated function, which may or may not
   match the hash associated with the PSK.  This is consistent with how
   the transcript is calculated for the rest of the handshake.

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
          Extension extensions<0..2^16-1>;
      } CertificateRequest;

   certificate_request_context:  An opaque string which identifies the
      certificate request and which will be echoed in the client's
      Certificate message.  The certificate_request_context MUST be
      unique within the scope of this connection (thus preventing replay
      of client CertificateVerify messages).  This field SHALL be zero
      length unless used for the post-handshake authentication exchanges
      described in Section 4.6.2.  When requesting post-handshake
      authentication, the server SHOULD make the context unpredictable
      to the client (e.g., by randomly generating it) to prevent an
      attacker who has temporary access to the client's private key from
      pre-computing valid CertificateVerify messages.

   extensions:  A list of extensions describing the parameters of the
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

   Servers which are authenticating with a resumption PSK MUST NOT send
   the CertificateRequest message in the main handshake, though they MAY
   send it in post-handshake authentication (see Section 4.6.2) provided
   that the client has sent the "post_handshake_auth" extension (see Section 4.2.6).  In the absence of some other specification to the
   contrary, servers which are authenticating with an external PSK MUST
   NOT send the CertificateRequest message either in the main handshake
   or request post-handshake authentication.  [RFC8773] provides an
   extension to permit this, but has received less analysis than this
   specification. 





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
   [sender ]_handshake_traffic_secret, except for post-handshake
   authentication.

   The computations for the Authentication messages all uniformly take
   the following inputs:

   *  The certificate and signing key to be used.

   *  A Handshake Context consisting of the list of messages to be
      included in the transcript hash.

   *  A Base Key to be used to compute a MAC key.

   Based on these inputs, the messages then contain:

   Certificate  The certificate to be used for authentication, and any
      supporting certificates in the chain.  Note that certificate-based
      client authentication is not available in PSK handshake flows
      (including 0-RTT).

   CertificateVerify:  A signature over the value Transcript-
      Hash(Handshake Context, Certificate)

   Finished:  A MAC over the value Transcript-Hash(Handshake Context,
      Certificate, CertificateVerify) using a MAC key derived from the
      Base Key.

   The following table defines the Handshake Context and MAC Base Key
   for each scenario:
   +=========+====================+=====================================+
   |Mode     |Handshake Context   |Base Key                             |
   +=========+====================+=====================================+
   |Server   |ClientHello ...     |server_handshake_traffic_secret      |
   |         |later of            |                                     |
   |         |EncryptedExtensions/|                                     |
   |         |CertificateRequest  |                                     |
   +---------+--------------------+-------------------------------------+
   |Client   |ClientHello ...     |client_handshake_traffic_secret      |
   |         |later of server     |                                     |
   |         |Finished/           |                                     |
   |         |EndOfEarlyData      |                                     |
   +---------+--------------------+-------------------------------------+
   |Post-    |ClientHello ...     |[sender ]_application_traffic_secret_N|
   |Handshake|client Finished +   |                                     |
   |         |CertificateRequest  |                                     |
   +---------+--------------------+-------------------------------------+

                       Table 2: Authentication Inputs

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
          00 00 Hash.length  ||   /* Handshake message length (bytes) */
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
   Certificate, client CertificateVerify, and client Finished.

   In general, implementations can implement the transcript by keeping a
   running transcript hash value based on the negotiated hash.  Note,
   however, that subsequent post-handshake authentications do not
   include each other, just the messages through the end of the main
   handshake.

#### 4.4.2. Certificate

   This message conveys the endpoint's certificate chain to the peer.

   The server MUST send a Certificate message whenever the agreed-upon
   key exchange method uses certificates for authentication (this
   includes all key exchange methods defined in this document except
   PSK).

   The client MUST send a Certificate message if and only if the server
   has requested certificate-based client authentication via a
   CertificateRequest message (Section 4.3.2).  If the server requests
   certificate-based client authentication but no suitable certificate
   is available, the client MUST send a Certificate message containing
   no certificates (i.e., with the "certificate_list" field having
   length 0).  A Finished message MUST be sent regardless of whether the
   Certificate message is empty.

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

   certificate_list:  A list (chain) of CertificateEntry structures,
      each containing a single certificate and list of extensions.

   extensions:  A list of extension values for the CertificateEntry.
      The "Extension" format is defined in Section 4.2.  Valid
      extensions for server certificates at present include the OCSP
      Status extension [RFC6066] and the SignedCertificateTimestamp
      extension [RFC6962]; future extensions may be defined for this
      message as well.  Extensions in the Certificate message from the
      server MUST correspond to ones from the ClientHello message.
      Extensions in the Certificate message from the client MUST
      correspond to extensions in the CertificateRequest message from
      the server.  If an extension applies to the entire chain, it
      SHOULD be included in the first CertificateEntry.

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
   Note: The status_request_v2 extension [RFC6961] is deprecated.  TLS
   1.3 servers MUST NOT act upon its presence or information in it when
   processing ClientHello messages; in particular, they MUST NOT send
   the status_request_v2 extension in the EncryptedExtensions,
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

##### 4.4.2.2. Certificate Selection

   The following rules apply to the certificates sent by the client or
   server:

   *  The certificate type MUST be X.509v3 [RFC5280], unless explicitly
      negotiated otherwise (e.g., [RFC7250]).

   *  The end-entity certificate MUST allow the key to be used for
      signing with a signature scheme indicated in the peer's
      "signature_algorithms" extension (see Section 4.2.3).  That is,
      the digitalSignature bit MUST be set if the Key Usage extension is
      present, and the public key (with associated restrictions) MUST be
      compatible with some supported signature scheme.

   *  If the peer sent a "certificate_authorities" extension, at least
      one of the certificates in the certificate chain SHOULD be issued
      by one of the listed CAs.

   The following rule additionally applies to certificates sent by the
   client:

   *  If the CertificateRequest message contained a non-empty
      "oid_filters" extension, the end-entity certificate MUST match the
      extension OIDs that are recognized by the client, as described in Section 4.2.5.

   The following rule additionally applies to certificates sent by the
   server:
   *  The "server_name" [RFC6066] extension is used to guide certificate
      selection.  As servers MAY require the presence of the
      "server_name" extension, clients SHOULD send this extension when
      the server is identified by name.

   All certificates provided by the sender MUST be signed by a signature
   algorithm advertised by the peer, if it is able to provide such a
   chain (see Section 4.2.3).  Certificates that are self-signed or
   certificates that are expected to be trust anchors are not validated
   as part of the chain and therefore MAY be signed with any algorithm.

   If the sender is the server, and the server cannot produce a
   certificate chain that is signed only via the indicated supported
   algorithms, then it SHOULD continue the handshake by sending a
   certificate chain of its choice that may include algorithms that are
   not known to be supported by the client.  This fallback chain MUST
   NOT use the deprecated SHA-1 hash, unless the client specifically
   advertises that it is willing to accept SHA-1.

   If the sender is the client, the client MAY use a fallback chain as
   above, or continue the handshake anonymously.

   If the receiver cannot construct an acceptable chain using the
   provided certificates and decides to abort the handshake, then it
   MUST abort the handshake with an appropriate certificate-related
   alert (by default, "unsupported_certificate"; see Section 6.2 for
   more information).

   If the sender has multiple certificates, it chooses one of them based
   on the above-mentioned criteria (in addition to other criteria, such
   as transport-layer endpoint, local configuration, and preferences).

##### 4.4.2.3. Receiving a Certificate Message

   In general, detailed certificate validation procedures are out of
   scope for TLS (see [RFC5280]).  This section provides TLS-specific
   requirements.

   If the server supplies an empty Certificate message, the client MUST
   abort the handshake with a "decode_error" alert. 
   If the client does not send any certificates (i.e., it sends an empty
   Certificate message), the server MAY at its discretion either
   continue the handshake without client authentication, or abort the
   handshake with a "certificate_required" alert.  Also, if some aspect
   of the certificate chain was unacceptable (e.g., it was not signed by
   a known, trusted CA), the server MAY at its discretion either
   continue the handshake (considering the client unauthenticated) or
   abort the handshake.

   Any endpoint receiving any certificate which it would need to
   validate using any signature algorithm using an MD5 hash MUST abort
   the handshake with a "bad_certificate" alert.  SHA-1 is deprecated
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

   *  A string that consists of octet 32 (0x20) repeated 64 times

   *  The context string (defined below)

   *  A single 0 byte which serves as the separator

   *  The content to be signed

   This structure is intended to prevent an attack on previous versions
   of TLS in which the ServerKeyExchange format meant that attackers
   could obtain a signature of a message with a chosen 32-byte prefix
   (ClientHello.random).  The initial 64-byte pad clears that prefix
   along with the server-controlled ServerHello.random.

   The context string for a server signature is "TLS 1.3, server
   CertificateVerify" The context string for a client signature is "TLS
   1.3, client CertificateVerify" It is used to provide separation
   between signatures made in different contexts, helping against
   potential cross-protocol attacks.

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

   *  The content covered by the digital signature

   *  The private signing key corresponding to the certificate sent in
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

   *  The content covered by the digital signature

   *  The public key contained in the end-entity certificate found in
      the associated Certificate message

   *  The digital signature received in the signature field of the
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

   If the client's hello contained a suitable "psk_key_exchange_modes"
   extension, at any time after the server has received the client
   Finished message, it MAY send a NewSessionTicket message.  This
   message creates a unique association between the ticket value and a
   secret PSK derived from the resumption secret (see Section 7).

   The client MAY use this PSK for future handshakes by including the
   ticket value in the "pre_shared_key" extension in its ClientHello
   (Section 4.2.11).  Clients which receive a NewSessionTicket message
   but do not support resumption MUST silently ignore this message.
   Resumption MAY be done while the original connection is still open.
   Servers MAY send multiple tickets on a single connection, either
   immediately after each other or after specific events (see Appendix C.4).  For instance, the server might send a new ticket
   after post-handshake authentication thus encapsulating the additional
   client authentication state.  Multiple tickets are useful for clients
   for a variety of purposes, including:

   *  Opening multiple parallel HTTP connections.

   *  Performing connection racing across interfaces and address
      families via (for example) Happy Eyeballs [RFC8305] or related
      techniques. 
   Any ticket MUST only be resumed with a cipher suite that has the same
   KDF hash algorithm as that used to establish the original connection.

   Clients MUST only resume if the new SNI value is valid for the server
   certificate presented in the original session, and SHOULD only resume
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

   Note: Although the resumption secret depends on the client's second
   flight, a server which does not request certificate-based client
   authentication MAY compute the remainder of the transcript
   independently and then send a NewSessionTicket immediately upon
   sending its Finished rather than waiting for the client Finished.
   This might be appropriate in cases where the client is expected to
   open multiple TLS connections in parallel and would benefit from the
   reduced overhead of a resumption handshake, for example.

      struct {
          uint32 ticket_lifetime;
          uint32 ticket_age_add;
          opaque ticket_nonce<0..255>;
          opaque ticket<1..2^16-1>;
          Extension extensions<0..2^16-1>;
      } NewSessionTicket;

   ticket_lifetime:  Indicates the lifetime in seconds as a 32-bit
      unsigned integer in network byte order from the time of ticket
      issuance.  Servers MUST NOT use any value greater than 604800
      seconds (7 days).  The value of zero indicates that the ticket
      should be discarded immediately.  Clients MUST NOT use tickets for
      longer than 7 days after issuance, regardless of the
      ticket_lifetime, and MAY delete tickets earlier based on local
      policy.  A server MAY treat a ticket as valid for a shorter period
      of time than what is stated in the ticket_lifetime.

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

   extensions:  A list of extension values for the ticket.  The
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
      be unable to differentiate padding from content, so clients SHOULD
      NOT depend on being able to send large quantities of padding in
      early data records.

   The PSK associated with the ticket is computed as:

       HKDF-Expand-Label(resumption_secret,
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

   When the client has sent the "post_handshake_auth" extension (see Section 4.2.6), a server MAY request certificate-based client
   authentication at any time after the handshake has completed by
   sending a CertificateRequest message.  The client MUST respond with
   the appropriate Authentication messages (see Section 4.4).  If the
   client chooses to authenticate, it MUST send Certificate,
   CertificateVerify, and Finished.  If it declines, it MUST send a
   Certificate message containing no certificates followed by Finished.
   All of the client's messages for a given response MUST appear
   consecutively on the wire with no intervening messages of other type
   or from other responses.

   A client that receives a CertificateRequest message without having
   sent the "post_handshake_auth" extension MUST send an
   "unexpected_message" fatal alert.

   Note: Because certificate-based client authentication could involve
   prompting the user, servers MUST be prepared for some delay,
   including receiving an arbitrary number of other messages between
   sending the CertificateRequest and receiving a response.  In
   addition, clients which receive multiple CertificateRequests in close
   succession MAY respond to them in a different order than they were
   received (the certificate_request_context value allows the server to
   disambiguate the responses).

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
   update.  Until receiving a subsequent KeyUpdate from the peer, the
   sender MUST NOT send another KeyUpdate with request_update set to
   "update_requested".

   Note that implementations may receive an arbitrary number of messages
   between sending a KeyUpdate with request_update set to
   "update_requested" and receiving the peer's KeyUpdate, including
   unrelated KeyUpdates, because those messages may already be in
   flight.  However, because send and receive keys are derived from
   independent traffic secrets, retaining the receive traffic secret
   does not threaten the forward secrecy of data sent before the sender
   changed keys.

   If implementations independently send their own KeyUpdates with
   request_update set to "update_requested", and they cross in flight,
   then each side will also send a response, with the result that each
   side increments by two generations.

   Both sender and receiver MUST encrypt their KeyUpdate messages with
   the old keys.  Additionally, both sides MUST enforce that a KeyUpdate
   with the old key is received before accepting any messages encrypted
   with the new key.  Failure to do so may allow message truncation
   attacks.

   With a 128-bit key as in AES-128, rekeying 2^64 times has a high
   probability of key reuse within a given connection.  Note that even
   if the key repeats, the IV is also independently generated, so the
   chance of a joint key/IV collision is much lower.  To provide an
   extra margin of security, sending implementations MUST NOT allow the
   epoch -- and hence the number of key updates -- to exceed 2^48-1.  In
   order to allow this value to be changed later -- for instance for
   ciphers with more than 128-bit keys -- receiving implementations MUST
   NOT enforce this rule.  If a sending implementation receives a
   KeyUpdate with request_update set to "update_requested", it MUST NOT
   send its own KeyUpdate if that would cause it to exceed these limits
   and SHOULD instead ignore the "update_requested" flag.  This may
   result in an eventual need to terminate the connection when the
   limits in Section 5.5 are reached. 





## 5. Record Protocol

   The TLS record protocol takes messages to be transmitted, fragments
   the data into manageable blocks, protects the records, and transmits
   the result.  Received data is verified, decrypted, reassembled, and
   then delivered to higher-level clients.

   TLS records are typed, which allows multiple higher-level protocols
   to be multiplexed over the same record layer.  This document
   specifies four content types: handshake, application_data, alert, and
   change_cipher_spec.  The change_cipher_spec record is used only for
   compatibility purposes (see Appendix E.4).

   An implementation may receive an unencrypted record of type
   change_cipher_spec consisting of the single byte value 0x01 at any
   time after the first ClientHello message has been sent or received
   and before the peer's Finished message has been received and MUST
   simply drop it without further processing.  Note that this record may
   appear at a point at the handshake where the implementation is
   expecting protected records, and so it is necessary to detect this
   condition prior to attempting to deprotect the record.  An
   implementation which receives any other change_cipher_spec value or
   which receives a protected change_cipher_spec record MUST abort the
   handshake with an "unexpected_message" alert.  If an implementation
   detects a change_cipher_spec record received before the first
   ClientHello message or after the peer's Finished message, it MUST be
   treated as an unexpected record type (though stateless servers may
   not be able to distinguish these cases from allowed cases).

   Implementations MUST NOT send record types not defined in this
   document unless negotiated by some extension.  If a TLS
   implementation receives an unexpected record type, it MUST terminate
   the connection with an "unexpected_message" alert.  New record
   content type values are assigned by IANA in the TLS ContentType
   registry as described in Section 11.

### 5.1. Record Layer

   The record layer fragments information blocks into TLSPlaintext
   records carrying data in chunks of 2^14 bytes or less.  Message
   boundaries are handled differently depending on the underlying
   ContentType.  Any future content types MUST specify appropriate
   rules.  Note that these rules are stricter than what was enforced in
   TLS 1.2.

   Handshake messages MAY be coalesced into a single TLSPlaintext record
   or fragmented across several records, provided that:
   *  Handshake messages MUST NOT be interleaved with other record
      types.  That is, if a handshake message is split over two or more
      records, there MUST NOT be any other records between them.

   *  Handshake messages MUST NOT span key changes.  Implementations
      MUST verify that all messages immediately preceding a key change
      align with a record boundary; if not, then they MUST terminate the
      connection with an "unexpected_message" alert.  Because the
      ClientHello, EndOfEarlyData, ServerHello, Finished, and KeyUpdate
      messages can immediately precede a key change, implementations
      MUST send these messages in alignment with a record boundary.

   Implementations MUST NOT send zero-length fragments of Handshake
   types, even if those fragments contain padding.

   Alert messages (Section 6) MUST NOT be fragmented across records, and
   multiple alert messages MUST NOT be coalesced into a single
   TLSPlaintext record.  In other words, a record with an Alert type
   MUST contain exactly one message.

   Application Data messages contain data that is opaque to TLS.
   Application Data messages are always protected.  Zero-length
   fragments of Application Data (i.e., TLSInnerPlaintext records of
   type application_data with zero-length content) MAY be sent, as they
   are potentially useful as a traffic analysis countermeasure.
   Application Data fragments MAY be split across multiple records or
   coalesced into a single record.

      enum {
          invalid(0),
          change_cipher_spec(20),
          alert(21),
          handshake(22),
          application_data(23),
          (255)
      } ContentType;

      struct {
          ContentType type;
          ProtocolVersion legacy_record_version;
          uint16 length;
          opaque fragment[TLSPlaintext.length];
      } TLSPlaintext;

   type:  The higher-level protocol used to process the enclosed
      fragment.

   legacy_record_version:  MUST be set to 0x0303 for all records 
      generated by a TLS 1.3 implementation other than an initial
      ClientHello (i.e., one not generated after a HelloRetryRequest),
      where it MAY also be 0x0301 for compatibility purposes.  This
      field is deprecated and MUST be ignored for all purposes.
      Previous versions of TLS would use other values in this field
      under some circumstances.

   length:  The length (in bytes) of the following
      TLSPlaintext.fragment.  The length MUST NOT exceed 2^14 bytes.  An
      endpoint that receives a record that exceeds this length MUST
      terminate the connection with a "record_overflow" alert.

   fragment  The data being transmitted.  This value is transparent and
      is treated as an independent block to be dealt with by the higher-
      level protocol specified by the type field.

   This document describes TLS 1.3, which uses the version 0x0304.  This
   version value is historical, deriving from the use of 0x0301 for TLS
   1.0 and 0x0300 for SSL 3.0.  To maximize backward compatibility, a
   record containing an initial ClientHello SHOULD have version 0x0301
   (reflecting TLS 1.0) and a record containing a second ClientHello or
   a ServerHello MUST have version 0x0303 (reflecting TLS 1.2).  When
   negotiating prior versions of TLS, endpoints follow the procedure and
   requirements provided in Appendix E .

   When record protection has not yet been engaged, TLSPlaintext
   structures are written directly onto the wire.  Once record
   protection has started, TLSPlaintext records are protected and sent
   as described in the following section.  Note that Application Data
   records MUST NOT be written to the wire unprotected (see Section 2   for details).

### 5.2. Record Payload Protection

   The record protection functions translate a TLSPlaintext structure
   into a TLSCiphertext structure.  The deprotection functions reverse
   the process.  In TLS 1.3, as opposed to previous versions of TLS, all
   ciphers are modeled as "Authenticated Encryption with Associated
   Data" (AEAD) [RFC5116].  AEAD functions provide a unified encryption
   and authentication operation which turns plaintext into authenticated
   ciphertext and back again.  Each encrypted record consists of a
   plaintext header followed by an encrypted body, which itself contains
   a type and optional padding. 
      struct {
          opaque content[TLSPlaintext.length];
          ContentType type;
          uint8 zeros[length_of_padding];
      } TLSInnerPlaintext;

      struct {
          ContentType opaque_type = application_data; /* 23 */
          ProtocolVersion legacy_record_version = 0x0303; /* TLS v1.2 */
          uint16 length;
          opaque encrypted_record[TLSCiphertext.length];
      } TLSCiphertext;

   content:  The TLSPlaintext.fragment value, containing the byte
      encoding of a handshake or an alert message, or the raw bytes of
      the application's data to send.

   type:  The TLSPlaintext.type value containing the content type of the
      record.

   zeros:  An arbitrary-length run of zero-valued bytes may appear in
      the cleartext after the type field.  This provides an opportunity
      for senders to pad any TLS record by a chosen amount as long as
      the total stays within record size limits.  See Section 5.4 for
      more details.

   opaque_type:  The outer opaque_type field of a TLSCiphertext record
      is always set to the value 23 (application_data) for outward
      compatibility with middleboxes accustomed to parsing previous
      versions of TLS.  The actual content type of the record is found
      in TLSInnerPlaintext.type after decryption.

   legacy_record_version:  The legacy_record_version field is always
      0x0303.  TLS 1.3 TLSCiphertexts are not generated until after TLS
      1.3 has been negotiated, so there are no historical compatibility
      concerns where other values might be received.  Note that the
      handshake protocol, including the ClientHello and ServerHello
      messages, authenticates the protocol version, so this value is
      redundant.

   length:  The length (in bytes) of the following
      TLSCiphertext.encrypted_record, which is the sum of the lengths of
      the content and the padding, plus one for the inner content type,
      plus any expansion added by the AEAD algorithm.  The length MUST
      NOT exceed 2^14 + 256 bytes.  An endpoint that receives a record
      that exceeds this length MUST terminate the connection with a
      "record_overflow" alert. 
   encrypted_record:  The AEAD-encrypted form of the serialized
      TLSInnerPlaintext structure.

   AEAD algorithms take as input a single key, a nonce, a plaintext, and
   "additional data" to be included in the authentication check, as
   described in Section 2.1 of [RFC5116].  The key is either the
   client_write_key or the server_write_key, the nonce is derived from
   the sequence number and the client_write_iv or server_write_iv (see Section 5.3), and the additional data input is the record header.
   I.e.,

      additional_data = TLSCiphertext.opaque_type ||
                        TLSCiphertext.legacy_record_version ||
                        TLSCiphertext.length

   The plaintext input to the AEAD algorithm is the encoded
   TLSInnerPlaintext structure.  Derivation of traffic keys is defined
   in Section 7.3.

   The AEAD output consists of the ciphertext output from the AEAD
   encryption operation.  The length of the plaintext is greater than
   the corresponding TLSPlaintext.length due to the inclusion of
   TLSInnerPlaintext.type and any padding supplied by the sender.  The
   length of the AEAD output will generally be larger than the
   plaintext, but by an amount that varies with the AEAD algorithm.
   Since the ciphers might incorporate padding, the amount of overhead
   could vary with different lengths of plaintext.  Symbolically,

      AEADEncrypted =
          AEAD-Encrypt(write_key, nonce, additional_data, plaintext)

   The encrypted_record field of TLSCiphertext is set to AEADEncrypted.

   To decrypt and verify, the cipher takes as input the key, nonce,
   additional data, and the AEADEncrypted value.  The output is either
   the plaintext or an error indicating that the decryption failed.
   There is no separate integrity check.  Symbolically,

   plaintext of encrypted_record =
       AEAD-Decrypt(peer_write_key, nonce, additional_data, AEADEncrypted)

   If the decryption fails, the receiver MUST terminate the connection
   with a "bad_record_mac" alert. 
   An AEAD algorithm used in TLS 1.3 MUST NOT produce an expansion
   greater than 255 octets.  An endpoint that receives a record from its
   peer with TLSCiphertext.length larger than 2^14 + 256 octets MUST
   terminate the connection with a "record_overflow" alert.  This limit
   is derived from the maximum TLSInnerPlaintext length of 2^14 octets +
   1 octet for ContentType + the maximum AEAD expansion of 255 octets.

### 5.3. Per-Record Nonce

   A 64-bit sequence number is maintained separately for reading and
   writing records.  The appropriate sequence number is incremented by
   one after reading or writing each record.  Each sequence number is
   set to zero at the beginning of a connection and whenever the key is
   changed; the first record transmitted under a particular traffic key
   MUST use sequence number 0.

   Because the size of sequence numbers is 64-bit, they should not wrap.
   If a TLS implementation would need to wrap a sequence number, it MUST
   either rekey (Section 4.6.3) or terminate the connection.

   Each AEAD algorithm will specify a range of possible lengths for the
   per-record nonce, from N_MIN bytes to N_MAX bytes of input [RFC5116].
   The length of the TLS per-record nonce (iv_length) is set to the
   larger of 8 bytes and N_MIN for the AEAD algorithm (see [RFC5116],
   Section 4).  An AEAD algorithm where N_MAX is less than 8 bytes MUST
   NOT be used with TLS.  The per-record nonce for the AEAD construction
   is formed as follows:

   1.  The 64-bit record sequence number is encoded in network byte
       order and padded to the left with zeros to iv_length.

   2.  The padded sequence number is XORed with either the static
       client_write_iv or server_write_iv (depending on the role).

   The resulting quantity (of length iv_length) is used as the per-
   record nonce.

   Note: This is a different construction from that in TLS 1.2, which
   specified a partially explicit nonce.

### 5.4. Record Padding

   All encrypted TLS records can be padded to inflate the size of the
   TLSCiphertext.  This allows the sender to hide the size of the
   traffic from an observer. 
   When generating a TLSCiphertext record, implementations MAY choose to
   pad.  An unpadded record is just a record with a padding length of
   zero.  Padding is a string of zero-valued bytes appended to the
   ContentType field before encryption.  Implementations MUST set the
   padding octets to all zeros before encrypting.

   Application Data records may contain a zero-length
   TLSInnerPlaintext.content if the sender desires.  This permits
   generation of plausibly sized cover traffic in contexts where the
   presence or absence of activity may be sensitive.  Implementations
   MUST NOT send Handshake and Alert records that have a zero-length
   TLSInnerPlaintext.content; if such a message is received, the
   receiving implementation MUST terminate the connection with an
   "unexpected_message" alert.

   The padding sent is automatically verified by the record protection
   mechanism; upon successful decryption of a
   TLSCiphertext.encrypted_record, the receiving implementation scans
   the field from the end toward the beginning until it finds a non-zero
   octet.  This non-zero octet is the content type of the message.  This
   padding scheme was selected because it allows padding of any
   encrypted TLS record by an arbitrary size (from zero up to TLS record
   size limits) without introducing new content types.  The design also
   enforces all-zero padding octets, which allows for quick detection of
   padding errors.

   Implementations MUST limit their scanning to the cleartext returned
   from the AEAD decryption.  If a receiving implementation does not
   find a non-zero octet in the cleartext, it MUST terminate the
   connection with an "unexpected_message" alert.

   The presence of padding does not change the overall record size
   limitations: the full encoded TLSInnerPlaintext MUST NOT exceed 2^14
   + 1 octets.  If the maximum fragment length is reduced -- as for
   example by the record_size_limit extension from [RFC8449] -- then the
   reduced limit applies to the full plaintext, including the content
   type and padding.

   Selecting a padding policy that suggests when and how much to pad is
   a complex topic and is beyond the scope of this specification.  If
   the application-layer protocol on top of TLS has its own padding, it
   may be preferable to pad Application Data TLS records within the
   application layer.  Padding for encrypted Handshake or Alert records
   must still be handled at the TLS layer, though.  Later documents may
   define padding selection algorithms or define a padding policy
   request mechanism through TLS extensions or some other means. 





### 5.5. Limits on Key Usage

   There are cryptographic limits on the amount of plaintext which can
   be safely encrypted under a given set of keys.  [AEAD-LIMITS ]
   provides an analysis of these limits under the assumption that the
   underlying primitive (AES or ChaCha20) has no weaknesses.
   Implementations MUST either close the connection or do a key update
   as described in Section 4.6.3 prior to reaching these limits.  Note
   that it is not possible to perform a KeyUpdate for early data and
   therefore implementations MUST NOT exceed the limits when sending
   early data.  Receiving implementations SHOULD NOT enforce these
   limits, as future analyses may result in updated values.

   For AES-GCM, up to 2^24.5 full-size records (about 24 million) may be
   encrypted on a given connection while keeping a safety margin of
   approximately 2^-57 for Authenticated Encryption (AE) security.  For
   ChaCha20/Poly1305, the record sequence number would wrap before the
   safety limit is reached.

## 6. Alert Protocol

   TLS provides an Alert content type to indicate closure information
   and errors.  Like other messages, alert messages are encrypted as
   specified by the current connection state.

   Alert messages convey a description of the alert and a legacy field
   that conveyed the severity level of the message in previous versions
   of TLS.  Alerts are divided into two classes: closure alerts and
   error alerts.  In TLS 1.3, the severity is implicit in the type of
   alert being sent, and the "level" field can safely be ignored.  The
   "close_notify" alert is used to indicate orderly closure of one
   direction of the connection.  Upon receiving such an alert, the TLS
   implementation SHOULD indicate end-of-data to the application.

   Error alerts indicate abortive closure of the connection (see Section 6.2).  Upon receiving an error alert, the TLS implementation
   SHOULD indicate an error to the application and MUST NOT allow any
   further data to be sent or received on the connection.  Servers and
   clients MUST forget the secret values and keys established in failed
   connections, with the exception of the PSKs associated with session
   tickets, which SHOULD be discarded if possible.

   All the alerts listed in Section 6.2 MUST be sent with
   AlertLevel=fatal and MUST be treated as error alerts when received
   regardless of the AlertLevel in the message.  Unknown Alert types
   MUST be treated as error alerts. 
   Note: TLS defines two generic alerts (see Section 6) to use upon
   failure to parse a message.  Peers which receive a message which
   cannot be parsed according to the syntax (e.g., have a length
   extending beyond the message boundary or contain an out-of-range
   length) MUST terminate the connection with a "decode_error" alert.
   Peers which receive a message which is syntactically correct but
   semantically invalid (e.g., a DHE share of p - 1, or an invalid enum)
   MUST terminate the connection with an "illegal_parameter" alert.

      enum { warning(1), fatal(2), (255) } AlertLevel;

      enum {
          close_notify(0),
          unexpected_message(10),
          bad_record_mac(20),
          record_overflow(22),
          handshake_failure(40),
          bad_certificate(42),
          unsupported_certificate(43),
          certificate_revoked(44),
          certificate_expired(45),
          certificate_unknown(46),
          illegal_parameter(47),
          unknown_ca(48),
          access_denied(49),
          decode_error(50),
          decrypt_error(51),
          protocol_version(70),
          insufficient_security(71),
          internal_error(80),
          inappropriate_fallback(86),
          user_canceled(90),
          missing_extension(109),
          unsupported_extension(110),
          unrecognized_name(112),
          bad_certificate_status_response(113),
          unknown_psk_identity(115),
          certificate_required(116),
          general_error(117),
          no_application_protocol(120),
          (255)
      } AlertDescription;

      struct {
          AlertLevel level;
          AlertDescription description;
      } Alert;





### 6.1. Closure Alerts

   The client and the server must share knowledge that the connection is
   ending in order to avoid a truncation attack.

   close_notify:  This alert notifies the recipient that the sender will
      not send any more messages on this connection.  Any data received
      after a closure alert has been received MUST be ignored.  This
      alert MUST be sent with AlertLevel=warning.

   user_canceled:  This alert notifies the recipient that the sender is
      canceling the handshake for some reason unrelated to a protocol
      failure.  If a user cancels an operation after the handshake is
      complete, just closing the connection by sending a "close_notify"
      is more appropriate.  This alert MUST be followed by a
      "close_notify".  This alert generally has AlertLevel=warning.
      Receiving implementations SHOULD continue to read data from the
      peer until a "close_notify" is received, though they MAY log or
      otherwise record them.

   Either party MAY initiate a close of its write side of the connection
   by sending a "close_notify" alert.  If a transport-level close is
   received prior to a "close_notify", the receiver cannot know that all
   the data that was sent has been received.

   Each party MUST send a "close_notify" alert before closing its write
   side of the connection, unless it has already sent some error alert.
   This does not have any effect on its read side of the connection.
   Note that this is a change from versions of TLS prior to TLS 1.3 in
   which implementations were required to react to a "close_notify" by
   discarding pending writes and sending an immediate "close_notify"
   alert of their own.  That previous requirement could cause truncation
   in the read side.  Both parties need not wait to receive a
   "close_notify" alert before closing their read side of the
   connection, though doing so would introduce the possibility of
   truncation.

   Application protocols MAY choose to flush their send buffers and
   immediately send a close_notify upon receiving a close_notify, but
   this allows an attacker to influence the data that the peer receives
   by delaying the close_notify or by delaying the transport level
   delivery of the application's packets.  These issues can be addressed
   at the application layer, for instance by ignoring packets received
   after transmitting the close_notify.

   If the application protocol using TLS provides that any data may be
   carried over the underlying transport after the TLS connection is
   closed, the TLS implementation MUST receive a "close_notify" alert 
   before indicating end-of-data to the application layer.  No part of
   this standard should be taken to dictate the manner in which a usage
   profile for TLS manages its data transport, including when
   connections are opened or closed.

   Note: It is assumed that closing the write side of a connection
   reliably delivers pending data before destroying the transport.

### 6.2. Error Alerts

   Error handling in TLS is very simple.  When an error is detected, the
   detecting party sends a message to its peer.  Upon transmission or
   receipt of a fatal alert message, both parties MUST immediately close
   the connection.

   Whenever an implementation encounters a fatal error condition, it
   SHOULD send an appropriate fatal alert and MUST close the connection
   without sending or receiving any additional data.  Throughout this
   specification, when the phrases "terminate the connection" and "abort
   the handshake" are used without a specific alert it means that the
   implementation SHOULD send the alert indicated by the descriptions
   below.  The phrases "terminate the connection with an X alert" and
   "abort the handshake with an X alert" mean that the implementation
   MUST send alert X if it sends any alert.  All alerts defined below in
   this section, as well as all unknown alerts, are universally
   considered fatal as of TLS 1.3 (see Section 6).  The implementation
   SHOULD provide a way to facilitate logging the sending and receiving
   of alerts.

   The following error alerts are defined:

   unexpected_message:  An inappropriate message (e.g., the wrong
      handshake message, premature Application Data, etc.) was received.
      This alert should never be observed in communication between
      proper implementations.

   bad_record_mac:  This alert is returned if a record is received which
      cannot be deprotected.  Because AEAD algorithms combine decryption
      and verification, and also to avoid side-channel attacks, this
      alert is used for all deprotection failures.  This alert should
      never be observed in communication between proper implementations,
      except when messages were corrupted in the network.

   record_overflow:  A TLSCiphertext record was received that had a 
      length more than 2^14 + 256 bytes, or a record decrypted to a
      TLSPlaintext record with more than 2^14 bytes (or some other
      negotiated limit).  This alert should never be observed in
      communication between proper implementations, except when messages
      were corrupted in the network.

   handshake_failure:  Receipt of a "handshake_failure" alert message
      indicates that the sender was unable to negotiate an acceptable
      set of security parameters given the options available.

   bad_certificate:  A certificate was corrupt, contained signatures
      that did not verify correctly, etc.

   unsupported_certificate:  A certificate was of an unsupported type.

   certificate_revoked:  A certificate was revoked by its signer.

   certificate_expired:  A certificate has expired or is not currently
      valid.

   certificate_unknown:  Some other (unspecified) issue arose in
      processing the certificate, rendering it unacceptable.

   illegal_parameter:  A field in the handshake was incorrect or
      inconsistent with other fields.  This alert is used for errors
      which conform to the formal protocol syntax but are otherwise
      incorrect.

   unknown_ca:  A valid certificate chain or partial chain was received,
      but the certificate was not accepted because the CA certificate
      could not be located or could not be matched with a known trust
      anchor.

   access_denied:  A valid certificate or PSK was received, but when
      access control was applied, the sender decided not to proceed with
      negotiation.

   decode_error:  A message could not be decoded because some field was
      out of the specified range or the length of the message was
      incorrect.  This alert is used for errors where the message does
      not conform to the formal protocol syntax.  This alert should
      never be observed in communication between proper implementations,
      except when messages were corrupted in the network.

   decrypt_error:  A handshake (not record layer) cryptographic
      operation failed, including being unable to correctly verify a
      signature or validate a Finished message or a PSK binder. 
   protocol_version:  The protocol version the peer has attempted to
      negotiate is recognized but not supported (see Appendix E ).

   insufficient_security:  Returned instead of "handshake_failure" when
      a negotiation has failed specifically because the server requires
      parameters more secure than those supported by the client.

   internal_error:  An internal error unrelated to the peer or the
      correctness of the protocol (such as a memory allocation failure)
      makes it impossible to continue.

   inappropriate_fallback:  Sent by a server in response to an invalid
      connection retry attempt from a client (see [RFC7507]).

   missing_extension:  Sent by endpoints that receive a handshake
      message not containing an extension that is mandatory to send for
      the offered TLS version or other negotiated parameters.

   unsupported_extension:  Sent by endpoints receiving any handshake
      message containing an extension in a ServerHello,
      HelloRetryRequest, EncryptedExtensions, or Certificate not first
      offered in the corresponding ClientHello or CertificateRequest.

   unrecognized_name:  Sent by servers when no server exists identified
      by the name provided by the client via the "server_name" extension
      (see [RFC6066]).

   bad_certificate_status_response:  Sent by clients when an invalid or
      unacceptable OCSP response is provided by the server via the
      "status_request" extension (see [RFC6066]).

   unknown_psk_identity:  Sent by servers when PSK key establishment is
      desired but no acceptable PSK identity is provided by the client.
      Sending this alert is OPTIONAL; servers MAY instead choose to send
      a "decrypt_error" alert to merely indicate an invalid PSK
      identity.

   certificate_required:  Sent by servers when a client certificate is
      desired but none was provided by the client.

   general_error:  Sent to indicate an error condition in cases when
      either no more specific error is available or the senders wishes
      to conceal the specific error code.  Implementations SHOULD use
      more specific errors when available.

   no_application_protocol:  Sent by servers when a client
      "application_layer_protocol_negotiation" extension advertises only
      protocols that the server does not support (see [RFC7301]). 
   New Alert values are assigned by IANA as described in Section 11.

## 7. Cryptographic Computations

   The TLS handshake establishes one or more input secrets which are
   combined to create the actual working keying material, as detailed
   below.  The key derivation process incorporates both the input
   secrets and the handshake transcript.  Note that because the
   handshake transcript includes the random values from the Hello
   messages, any given handshake will have different traffic secrets,
   even if the same input secrets are used, as is the case when the same
   PSK is used for multiple connections.

### 7.1. Key Schedule

   The key derivation process makes use of the HKDF-Extract and HKDF-
   Expand functions as defined for HKDF [RFC5869], as well as the
   functions defined below:

       HKDF-Expand-Label(Secret, Label, Context, Length) =
            HKDF-Expand(Secret, HkdfLabel, Length)

       Where HkdfLabel is specified as:

       struct {
           uint16 length = Length;
           opaque label<7..255> = "tls13 " + Label;
           opaque context<0..255> = Context;
       } HkdfLabel;

       Derive-Secret(Secret, Label, Messages) =
            HKDF-Expand-Label(Secret, Label,
                              Transcript-Hash(Messages), Hash.length)

   The Hash function used by Transcript-Hash and HKDF is the cipher
   suite hash algorithm.  Hash.length is its output length in bytes.
   Messages is the concatenation of the indicated handshake messages,
   including the handshake message type and length fields, but not
   including record layer headers.  Note that in some cases a zero-
   length Context (indicated by "") is passed to HKDF-Expand-Label.  The
   labels specified in this document are all ASCII strings and do not
   include a trailing NUL byte.

   Any extensions to TLS which use "HKDF-Expand-Label" use the HkdfLabel
   definition associated with the version of TLS with which they are
   being used.  When used with this specification, that means using
   HkdfLabel as defined above; when used with DTLS [RFC9147] that means
   using the version defined in [RFC9147], Section 5.9. 
   Note: With common hash functions, any label longer than 12 characters
   requires an additional iteration of the hash function to compute.
   The labels in this specification have all been chosen to fit within
   this limit.

   Keys are derived from two input secrets using the HKDF-Extract and
   Derive-Secret functions.  The general pattern for adding a new secret
   is to use HKDF-Extract with the Salt being the current secret state
   and the Input Keying Material (IKM) being the new secret to be added.
   In this version of TLS 1.3, the two input secrets are:

   *  PSK (a pre-shared key established externally or derived from the
      resumption_secret value from a previous connection)

   *  (EC)DHE shared secret (Section 7.4)

   This produces the key schedule shown in the diagram below (Figure 5.
   In this diagram, the following formatting conventions apply:

   *  HKDF-Extract is drawn as taking the Salt argument from the top and
      the IKM argument from the left, with its output to the bottom and
      the name of the output on the right.

   *  Derive-Secret's Secret argument is indicated by the incoming
      arrow.  For instance, the Early Secret is the Secret for
      generating the client_early_traffic_secret.

   *  "0" indicates a string of Hash.length bytes set to zero.

   Note: the key derivation labels use the string "master" even though
   the values are referred to as "main" secrets.  This mismatch is a
   result of renaming the values while retaining compatibility.

   Note: this does not show all the leaf keys such as the separate AEAD
   and IV keys but rather the first set of secrets derived from the
   handshake.

                    0
                    |
                    v
        PSK -->  HKDF-Extract = Early Secret
                    |
                    +-----> Derive-Secret(.,
                    |                     "ext binder" |
                    |                     "res binder",
                    |                     "")
                    |              = binder_key
                    |
                    +-----> Derive-Secret(., "c e traffic",
                    |                     ClientHello)
                    |              = client_early_traffic_secret
                    |
                    +-----> Derive-Secret(., "e exp master",
                    |                     ClientHello)
                    |              = early_exporter_secret
                    v
               Derive-Secret(., "derived", "")
                    |
                    v
     (EC)DHE --> HKDF-Extract = Handshake Secret
                    |
                    +-----> Derive-Secret(., "c hs traffic",
                    |                     ClientHello...ServerHello)
                    |              = client_handshake_traffic_secret
                    |
                    +-----> Derive-Secret(., "s hs traffic",
                    |                     ClientHello...ServerHello)
                    |              = server_handshake_traffic_secret
                    v
               Derive-Secret(., "derived", "")
                    |
                    v
           0 --> HKDF-Extract = Main Secret
                    |
                    +-----> Derive-Secret(., "c ap traffic",
                    |                     ClientHello...server Finished)
                    |              = client_application_traffic_secret_0
                    |
                    +-----> Derive-Secret(., "s ap traffic",
                    |                     ClientHello...server Finished)
                    |              = server_application_traffic_secret_0
                    |
                    +-----> Derive-Secret(., "exp master",
                    |                     ClientHello...server Finished)
                    |              = exporter_secret
                    |
                    +-----> Derive-Secret(., "res master",
                                          ClientHello...client Finished)
                                   = resumption_secret

                    Figure 5: Main TLS 1.3 Key Schedule

   The general pattern here is that the secrets shown down the left side
   of the diagram are just raw entropy without context, whereas the
   secrets down the right side include Handshake Context and therefore
   can be used to derive working keys without additional context.  Note 
   that the different calls to Derive-Secret may take different Messages
   arguments, even with the same secret.  In a 0-RTT exchange, Derive-
   Secret is called with four distinct transcripts; in a 1-RTT-only
   exchange, it is called with three distinct transcripts.

   If a given secret is not available, then the 0-value consisting of a
   string of Hash.length bytes set to zeros is used.  Note that this
   does not mean skipping rounds, so if PSK is not in use, Early Secret
   will still be HKDF-Extract(0, 0).  For the computation of the
   binder_key, the label is "ext binder" for external PSKs (those
   provisioned outside of TLS) and "res binder" for resumption PSKs
   (those provisioned as the resumption secret of a previous handshake).
   The different labels prevent the substitution of one type of PSK for
   the other.

   There are multiple potential Early Secret values, depending on which
   PSK the server ultimately selects.  The client will need to compute
   one for each potential PSK; if no PSK is selected, it will then need
   to compute the Early Secret corresponding to the zero PSK.

   Once all the values which are to be derived from a given secret have
   been computed, that secret SHOULD be erased.

### 7.2. Updating Traffic Secrets

   Once the handshake is complete, it is possible for either side to
   update its sending traffic keys using the KeyUpdate handshake message
   defined in Section 4.6.3.  The next generation of traffic keys is
   computed by generating client_/server_application_traffic_secret_N+1
   from client_/server_application_traffic_secret_N as described in this
   section and then re-deriving the traffic keys as described in Section 7.3.

   The next-generation application_traffic_secret is computed as:

       application_traffic_secret_N+1 =
           HKDF-Expand-Label(application_traffic_secret_N,
                             "traffic upd", "", Hash.length)

   Once client_/server_application_traffic_secret_N+1 and its associated
   traffic keys have been computed, implementations SHOULD delete
   client_/server_application_traffic_secret_N and its associated
   traffic keys.

### 7.3. Traffic Key Calculation

   The traffic keying material is generated from the following input
   values:
   *  A secret value

   *  A purpose value indicating the specific value being generated

   *  The length of the key being generated

   The traffic keying material is generated from an input traffic secret
   value using:

    [sender ]_write_key = HKDF-Expand-Label(Secret, "key", "", key_length)
    [sender ]_write_iv  = HKDF-Expand-Label(Secret, "iv", "", iv_length)

   [sender ] denotes the sending side.  The value of Secret for each
   category of data is shown in the table below.

      +====================+=======================================+
      | Data Type          | Secret                                |
      +====================+=======================================+
      | 0-RTT Application  | client_early_traffic_secret           |
      | and EndOfEarlyData |                                       |
      +--------------------+---------------------------------------+
      | Initial Handshake  | [sender ]_handshake_traffic_secret     |
      +--------------------+---------------------------------------+
      | Post-Handshake and | [sender ]_application_traffic_secret_N |
      | Application Data   |                                       |
      +--------------------+---------------------------------------+

                    Table 3: Secrets for Traffic Keys

   Alerts are sent with the then current sending key (or as plaintext if
   no such key has been established.)  All the traffic keying material
   is recomputed whenever the underlying Secret changes (e.g., when
   changing from the handshake to Application Data keys or upon a key
   update).

### 7.4. (EC)DHE Shared Secret Calculation





#### 7.4.1. Finite Field Diffie-Hellman

   For finite field groups, a conventional Diffie-Hellman [KEYAGREEMENT ]
   computation is performed.  The negotiated key (Z) is converted to a
   byte string by encoding in big-endian form and left-padded with zeros
   up to the size of the prime.  This byte string is used as the shared
   secret in the key schedule as specified above.

   Note that this construction differs from previous versions of TLS
   which remove leading zeros. 





#### 7.4.2. Elliptic Curve Diffie-Hellman

   For secp256r1, secp384r1 and secp521r1, ECDH calculations (including
   key generation and shared secret calculation) are performed according
   to Sections 5.6.1.2 and 5.7.1.2 of [KEYAGREEMENT ] using the Elliptic
   Curve Cryptography Cofactor Diffie-Hellman Primitive.  The shared
   secret Z is the x-coordinate of the ECDH shared secret elliptic curve
   point represented as an octet string.  Note that the octet string Z
   as output by the Field-Element-to-Byte String Conversion specified in Appendix C.2 of [KEYAGREEMENT ] has constant length for any given
   field; leading zeros found in this octet string MUST NOT be
   truncated.  See Section 4.2.8.2 for requirements on public-key
   validation.

   For X25519 and X448, the ECDH calculations are as follows:

   *  The public key to put into the KeyShareEntry.key_exchange
      structure is the result of applying the ECDH scalar multiplication
      function to the secret key of appropriate length (into scalar
      input) and the standard public basepoint (into u-coordinate point
      input).

   *  The ECDH shared secret is the result of applying the ECDH scalar
      multiplication function to the secret key (into scalar input) and
      the peer's public key (into u-coordinate point input).  The output
      is used raw, with no processing.

   For these curves, implementations SHOULD use the approach specified
   in [RFC7748] to calculate the Diffie-Hellman shared secret.
   Implementations MUST check whether the computed Diffie-Hellman shared
   secret is the all-zero value and abort if so, as described in Section 6 of [RFC7748].  If implementors use an alternative
   implementation of these elliptic curves, they SHOULD perform the
   additional checks specified in Section 7 of [RFC7748].

### 7.5. Exporters

   [RFC5705] defines keying material exporters for TLS in terms of the
   TLS pseudorandom function (PRF).  This document replaces the PRF with
   HKDF, thus requiring a new construction.  The exporter interface
   remains the same.

   The exporter value is computed as:

   TLS-Exporter(label, context_value, key_length) =
       HKDF-Expand-Label(Derive-Secret(Secret, label, ""),
                         "exporter", Hash(context_value), key_length)
   Where Secret is either the early_exporter_secret or the
   exporter_secret.  Implementations MUST use the exporter_secret unless
   explicitly specified by the application.  The early_exporter_secret
   is defined for use in settings where an exporter is needed for 0-RTT
   data.  A separate interface for the early exporter is RECOMMENDED;
   this avoids the exporter user accidentally using an early exporter
   when a regular one is desired or vice versa.

   If no context is provided, the context_value is zero length.
   Consequently, providing no context computes the same value as
   providing an empty context.  This is a change from previous versions
   of TLS where an empty context produced a different output than an
   absent context.  As of this document's publication, no allocated
   exporter label is used both with and without a context.  Future
   specifications MUST NOT define a use of exporters that permit both an
   empty context and no context with the same label.  New uses of
   exporters SHOULD provide a context in all exporter computations,
   though the value could be empty.

   Requirements for the format of exporter labels are defined in Section 4 of [RFC5705].

## 8. 0-RTT and Anti-Replay

   As noted in Section 2.3 and Appendix F.5, TLS does not provide
   inherent replay protections for 0-RTT data.  There are two potential
   threats to be concerned with:

   *  Network attackers who mount a replay attack by simply duplicating
      a flight of 0-RTT data.

   *  Network attackers who take advantage of client retry behavior to
      arrange for the server to receive multiple copies of an
      application message.  This threat already exists to some extent
      because clients that value robustness respond to network errors by
      attempting to retry requests.  However, 0-RTT adds an additional
      dimension for any server system which does not maintain globally
      consistent server state.  Specifically, if a server system has
      multiple zones where tickets from zone A will not be accepted in
      zone B, then an attacker can duplicate a ClientHello and early
      data intended for A to both A and B.  At A, the data will be
      accepted in 0-RTT, but at B the server will reject 0-RTT data and
      instead force a full handshake.  If the attacker blocks the
      ServerHello from A, then the client will complete the handshake
      with B and probably retry the request, leading to duplication on
      the server system as a whole. 
   The first class of attack can be prevented by sharing state to
   guarantee that the 0-RTT data is accepted at most once.  Servers
   SHOULD provide that level of replay safety by implementing one of the
   methods described in this section or by equivalent means.  It is
   understood, however, that due to operational concerns not all
   deployments will maintain state at that level.  Therefore, in normal
   operation, clients will not know which, if any, of these mechanisms
   servers actually implement and hence MUST only send early data which
   they deem safe to be replayed.

   In addition to the direct effects of replays, there is a class of
   attacks where even operations normally considered idempotent could be
   exploited by a large number of replays (timing attacks, resource
   limit exhaustion and others, as described in Appendix F.5).  Those
   can be mitigated by ensuring that every 0-RTT payload can be replayed
   only a limited number of times.  The server MUST ensure that any
   instance of it (be it a machine, a thread, or any other entity within
   the relevant serving infrastructure) would accept 0-RTT for the same
   0-RTT handshake at most once; this limits the number of replays to
   the number of server instances in the deployment.  Such a guarantee
   can be accomplished by locally recording data from recently received
   ClientHellos and rejecting repeats, or by any other method that
   provides the same or a stronger guarantee.  The "at most once per
   server instance" guarantee is a minimum requirement; servers SHOULD
   limit 0-RTT replays further when feasible.

   The second class of attack cannot be prevented at the TLS layer and
   MUST be dealt with by any application.  Note that any application
   whose clients implement any kind of retry behavior already needs to
   implement some sort of anti-replay defense.

### 8.1. Single-Use Tickets

   The simplest form of anti-replay defense is for the server to only
   allow each session ticket to be used once.  For instance, the server
   can maintain a database of all outstanding valid tickets, deleting
   each ticket from the database as it is used.  If an unknown ticket is
   provided, the server would then fall back to a full handshake.

   If the tickets are not self-contained but rather are database keys,
   and the corresponding PSKs are deleted upon use, then connections
   established using PSKs enjoy not only anti-replay protection, but
   also forward secrecy once all copies of the PSK from the database
   entry have been deleted.  This mechanism also improves security for
   PSK usage when PSK is used without (EC)DHE. 
   Because this mechanism requires sharing the session database between
   server nodes in environments with multiple distributed servers, it
   may be hard to achieve high rates of successful PSK 0-RTT connections
   when compared to self-encrypted tickets.  Unlike session databases,
   session tickets can successfully do PSK-based session establishment
   even without consistent storage, though when 0-RTT is allowed they
   still require consistent storage for anti-replay of 0-RTT data, as
   detailed in the following section.

### 8.2. Client Hello Recording

   An alternative form of anti-replay is to record a unique value
   derived from the ClientHello (generally either the random value or
   the PSK binder) and reject duplicates.  Recording all ClientHellos
   causes state to grow without bound, but a server can instead record
   ClientHellos within a given time window and use the
   "obfuscated_ticket_age" to ensure that tickets aren't reused outside
   that window.

   To implement this, when a ClientHello is received, the server first
   verifies the PSK binder as described in Section 4.2.11.  It then
   computes the expected_arrival_time as described in the next section
   and rejects 0-RTT if it is outside the recording window, falling back
   to the 1-RTT handshake.

   If the expected_arrival_time is in the window, then the server checks
   to see if it has recorded a matching ClientHello.  If one is found,
   it either aborts the handshake with an "illegal_parameter" alert or
   accepts the PSK but rejects 0-RTT.  If no matching ClientHello is
   found, then it accepts 0-RTT and then stores the ClientHello for as
   long as the expected_arrival_time is inside the window.  Servers MAY
   also implement data stores with false positives, such as Bloom
   filters, in which case they MUST respond to apparent replay by
   rejecting 0-RTT but MUST NOT abort the handshake. 
   The server MUST derive the storage key only from validated sections
   of the ClientHello.  If the ClientHello contains multiple PSK
   identities, then an attacker can create multiple ClientHellos with
   different binder values for the less-preferred identity on the
   assumption that the server will not verify it (as recommended by Section 4.2.11).  I.e., if the client sends PSKs A and B but the
   server prefers A, then the attacker can change the binder for B
   without affecting the binder for A.  If the binder for B is part of
   the storage key, then this ClientHello will not appear as a
   duplicate, which will cause the ClientHello to be accepted, and may
   cause side effects such as replay cache pollution, although any 0-RTT
   data will not be decryptable because it will use different keys.  If
   the validated binder or the ClientHello.random is used as the storage
   key, then this attack is not possible.

   Because this mechanism does not require storing all outstanding
   tickets, it may be easier to implement in distributed systems with
   high rates of resumption and 0-RTT, at the cost of potentially weaker
   anti-replay defense because of the difficulty of reliably storing and
   retrieving the received ClientHello messages.  In many such systems,
   it is impractical to have globally consistent storage of all the
   received ClientHellos.  In this case, the best anti-replay protection
   is provided by having a single storage zone be authoritative for a
   given ticket and refusing 0-RTT for that ticket in any other zone.
   This approach prevents simple replay by the attacker because only one
   zone will accept 0-RTT data.  A weaker design is to implement
   separate storage for each zone but allow 0-RTT in any zone.  This
   approach limits the number of replays to once per zone.  Application
   message duplication of course remains possible with either design.

   When implementations are freshly started, they SHOULD reject 0-RTT as
   long as any portion of their recording window overlaps the startup
   time.  Otherwise, they run the risk of accepting replays which were
   originally sent during that period.

   Note: If the client's clock is running much faster than the server's,
   then a ClientHello may be received that is outside the window in the
   future, in which case it might be accepted for 1-RTT, causing a
   client retry, and then acceptable later for 0-RTT.  This is another
   variant of the second form of attack described in Section 8. 





### 8.3. Freshness Checks

   Because the ClientHello indicates the time at which the client sent
   it, it is possible to efficiently determine whether a ClientHello was
   likely sent reasonably recently and only accept 0-RTT for such a
   ClientHello, otherwise falling back to a 1-RTT handshake.  This is
   necessary for the ClientHello storage mechanism described in Section 8.2 because otherwise the server needs to store an unlimited
   number of ClientHellos, and is a useful optimization for self-
   contained single-use tickets because it allows efficient rejection of
   ClientHellos which cannot be used for 0-RTT.

   To implement this mechanism, a server needs to store the time that
   the server generated the session ticket, offset by an estimate of the
   round-trip time between client and server.  I.e.,

       adjusted_creation_time = creation_time + estimated_RTT

   This value can be encoded in the ticket, thus avoiding the need to
   keep state for each outstanding ticket.  The server can determine the
   client's view of the age of the ticket by subtracting the ticket's
   "ticket_age_add" value from the "obfuscated_ticket_age" parameter in
   the client's "pre_shared_key" extension.  The server can determine
   the expected_arrival_time of the ClientHello as:

     expected_arrival_time = adjusted_creation_time + clients_ticket_age

   When a new ClientHello is received, the expected_arrival_time is then
   compared against the current server wall clock time and if they
   differ by more than a certain amount, 0-RTT is rejected, though the
   1-RTT handshake can be allowed to complete.

   There are several potential sources of error that might cause
   mismatches between the expected_arrival_time and the measured time.
   Variations in client and server clock rates are likely to be minimal,
   though potentially the absolute times may be off by large values.
   Network propagation delays are the most likely causes of a mismatch
   in legitimate values for elapsed time.  Both the NewSessionTicket and
   ClientHello messages might be retransmitted and therefore delayed,
   which might be hidden by TCP.  For clients on the Internet, this
   implies windows on the order of ten seconds to account for errors in
   clocks and variations in measurements; other deployment scenarios may
   have different needs.  Clock skew distributions are not symmetric, so
   the optimal tradeoff may involve an asymmetric range of permissible
   mismatch values. 
   Note that freshness checking alone is not sufficient to prevent
   replays because it does not detect them during the error window,
   which -- depending on bandwidth and system capacity -- could include
   billions of replays in real-world settings.  In addition, this
   freshness checking is only done at the time the ClientHello is
   received, and not when subsequent early Application Data records are
   received.  After early data is accepted, records may continue to be
   streamed to the server over a longer time period.

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

   *  Supported Versions ("supported_versions"; Section 4.2.1)

   *  Cookie ("cookie"; Section 4.2.2)

   *  Signature Algorithms ("signature_algorithms"; Section 4.2.3)

   *  Signature Algorithms Certificate ("signature_algorithms_cert";Section 4.2.3)

   *  Negotiated Groups ("supported_groups"; Section 4.2.7)

   *  Key Share ("key_share"; Section 4.2.8)

   *  Server Name Indication ("server_name"; Section 3 of [RFC6066])
   All implementations MUST send and use these extensions when offering
   applicable features:

   *  "supported_versions" is REQUIRED for all ClientHello, ServerHello,
      and HelloRetryRequest messages.

   *  "signature_algorithms" is REQUIRED for certificate authentication.

   *  "supported_groups" is REQUIRED for ClientHello messages using DHE
      or ECDHE key exchange.

   *  "key_share" is REQUIRED for DHE or ECDHE key exchange.

   *  "pre_shared_key" is REQUIRED for PSK key agreement.

   *  "psk_key_exchange_modes" is REQUIRED for PSK key agreement.

   A client is considered to be attempting to negotiate using this
   specification if the ClientHello contains a "supported_versions"
   extension with 0x0304 contained in its body.  Such a ClientHello
   message MUST meet the following requirements:

   *  If not containing a "pre_shared_key" extension, it MUST contain
      both a "signature_algorithms" extension and a "supported_groups"
      extension.

   *  If containing a "supported_groups" extension, it MUST also contain
      a "key_share" extension, and vice versa.  An empty
      KeyShare.client_shares list is permitted.

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
   unable to influence the handshake (see Appendix F.1).  At the same
   time, deployments update at different rates, so a newer client or
   server MAY continue to support older parameters, which would allow it
   to interoperate with older endpoints.

   For this to work, implementations MUST correctly handle extensible
   fields:

   *  A client sending a ClientHello MUST support all parameters
      advertised in it.  Otherwise, the server may fail to interoperate
      by selecting one of those parameters.

   *  A server receiving a ClientHello MUST correctly ignore all
      unrecognized cipher suites, extensions, and other parameters.
      Otherwise, it may fail to interoperate with newer clients.  In TLS
      1.3, a client receiving a CertificateRequest or NewSessionTicket
      MUST also ignore all unrecognized extensions.

   *  A middlebox which terminates a TLS connection MUST behave as a
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

   *  A middlebox which forwards ClientHello parameters it does not
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

   The design of TLS 1.3 was constrained by widely deployed non-
   compliant TLS middleboxes (see Appendix E.4); however, it does not
   relax the invariants.  Those middleboxes continue to be non-
   compliant.

## 10. Security Considerations

   Security issues are discussed throughout this memo, especially in Appendix C , Appendix E , and Appendix F .

## 11. IANA Considerations

   This document uses several registries that were originally created in
   [RFC4346] and updated in [RFC8446] and [RFC8447].  The changes
   between [RFC8446] and [RFC8447] this document are described in Section 11.1.  IANA has updated these to reference this document.

   The registries and their allocation policies are below:

   *  TLS Cipher Suites registry: values with the first byte in the
      range 0-254 (decimal) are assigned via Specification Required
      [RFC8126].  Values with the first byte 255 (decimal) are reserved
      for Private Use [RFC8126].

      IANA has added the cipher suites listed in Appendix B.4 to the
      registry.  The "Value" and "Description" columns are taken from
      the table.  The "DTLS-OK" and "Recommended" columns are both
      marked as "Y" for each new cipher suite.

   *  TLS ContentType registry: Future values are allocated via
      Standards Action [RFC8126].

   *  TLS Alerts registry: Future values are allocated via Standards
      Action [RFC8126].  IANA [is requested to/has] populated this
      registry with the values from Appendix B.2.  The "DTLS-OK" column
      is marked as "Y" for all such values.  Values marked as
      "_RESERVED" have comments describing their previous usage. 
   *  TLS HandshakeType registry: Future values are allocated via
      Standards Action [RFC8126].  IANA has updated this registry to
      rename item 4 from "NewSessionTicket" to "new_session_ticket" and
      populated this registry with the values from Appendix B.3.  The
      "DTLS-OK" column is marked as "Y" for all such values.  Values
      marked "_RESERVED" have comments describing their previous or
      temporary usage.

   This document also uses the TLS ExtensionType Values registry
   originally created in [RFC4366].  IANA has updated it to reference
   this document.  Changes to the registry follow:

   *  IANA has updated the registration policy as follows:

      Values with the first byte in the range 0-254 (decimal) are
      assigned via Specification Required [RFC8126].  Values with the
      first byte 255 (decimal) are reserved for Private Use [RFC8126].

   *  IANA has updated this registry to include the "key_share",
      "pre_shared_key", "psk_key_exchange_modes", "early_data",
      "cookie", "supported_versions", "certificate_authorities",
      "oid_filters", "post_handshake_auth", and
      "signature_algorithms_cert" extensions with the values defined in
      this document and the "Recommended" value of "Y".

   *  IANA has updated this registry to include a "TLS 1.3" column which
      lists the messages in which the extension may appear.  This column
      has been initially populated from the table in Section 4.2, with
      any extension not listed there marked as "-" to indicate that it
      is not used by TLS 1.3.

   This document updates two entries in the TLS Certificate Types
   registry originally created in [RFC6091] and updated in [RFC8447].
   IANA has updated the entry for value 1 to have the name
   "OpenPGP_RESERVED", "Recommended" value "N", and comment "Used in TLS
   versions prior to 1.3."  IANA has updated the entry for value 0 to
   have the name "X509", "Recommended" value "Y", and comment "Was X.509
   before TLS 1.3".

   This document updates an entry in the TLS Certificate Status Types
   registry originally created in [RFC6961].  IANA has updated the entry
   for value 2 to have the name "ocsp_multi_RESERVED" and comment "Used
   in TLS versions prior to 1.3". 
   This document updates two entries in the TLS Supported Groups
   registry (created under a different name by [RFC4492]; now maintained
   by [RFC8422]) and updated by [RFC7919] and [RFC8447].  The entries
   for values 29 and 30 (x25519 and x448) have been updated to also
   refer to this document.

   In addition, this document defines two new registries that are
   maintained by IANA:

   *  TLS SignatureScheme registry: Values with the first byte in the
      range 0-253 (decimal) are assigned via Specification Required
      [RFC8126].  Values with the first byte 254 or 255 (decimal) are
      reserved for Private Use [RFC8126].  Values with the first byte in
      the range 0-6 or with the second byte in the range 0-3 that are
      not currently allocated are reserved for backward compatibility.
      This registry has a "Recommended" column.  The registry has been
      initially populated with the values described in Section 4.2.3.
      The following values are marked as "Recommended":
      ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384,
      rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512,
      rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, and
      ed25519.  The "Recommended" column is assigned a value of "N"
      unless explicitly requested, and adding a value with a
      "Recommended" value of "Y" requires Standards Action [RFC8126].
      IESG Approval is REQUIRED for a Y->N transition.

   *  TLS PskKeyExchangeMode registry: Values in the range 0-253
      (decimal) are assigned via Specification Required [RFC8126].  The
      values 254 and 255 (decimal) are reserved for Private Use
      [RFC8126].  This registry has a "Recommended" column.  The
      registry has been initially populated with psk_ke (0) and
      psk_dhe_ke (1).  Both are marked as "Recommended".  The
      "Recommended" column is assigned a value of "N" unless explicitly
      requested, and adding a value with a "Recommended" value of "Y"
      requires Standards Action [RFC8126].  IESG Approval is REQUIRED
      for a Y->N transition.

### 11.1. Changes for this RFC

   IANA [shall update/has updated] all references to RFC 8446 in the
   IANA registries with references to this document.

   IANA [shall rename/has renamed] the "extended_master_secret" value in
   the TLS ExtensionType Values registry to "extended_main_secret".

   IANA [shall create/has created] a value for the "general_error" alert
   in the TLS Alerts Registry with the value given in Section 6. 





## A. State Machine

   This appendix provides a summary of the legal state transitions for
   the client and server handshakes.  State names (in all capitals,
   e.g., START) have no formal meaning but are provided for ease of
   comprehension.  Actions which are taken only in certain circumstances
   are indicated in [].  The notation "K_{send,recv} = foo" means "set
   the send/recv key to the given key".

### A.1. Client





















                              START <----+
               Send ClientHello |        | Recv HelloRetryRequest
          [K_send = early data] |        |
                                v        |
           +->               WAIT_SH ----+
           |                    | Recv ServerHello
           |                    | K_recv = handshake
       Can |                    V
      send |                 WAIT_EE
     early |                    | Recv EncryptedExtensions
      data |          +---------+-------+
           |    Using |                 | Using certificate
           |      PSK |                 v
           |          |            WAIT_CERT_CR
           |          |        Recv |       | Recv CertificateRequest
           |          | Certificate |       v
           |          |             |    WAIT_CERT
           |          |             |       | Recv Certificate
           |          |             v       v
           |          |              WAIT_CV
           |          |                   | Recv CertificateVerify
           |          +-> WAIT_FINISHED <-+
           |                  | Recv Finished
           +->                | [Send EndOfEarlyData]
                              | K_send = handshake
                              | [Send Certificate [+ CertificateVerify]]
    Can send                  | Send Finished
    app data   -->            | K_send = K_recv = application
    after here                v
                          CONNECTED

   Note that with the transitions as shown above, clients may send
   alerts that derive from post-ServerHello messages in the clear or
   with the early data keys.  If clients need to send such alerts, they
   SHOULD first rekey to the handshake keys if possible.

### A.2. Server

















                              START <-----+
               Recv ClientHello |         | Send HelloRetryRequest
                                v         |
                             RECVD_CH ----+
                                | Select parameters
                                v
                             NEGOTIATED
                                | Send ServerHello
                                | K_send = handshake
                                | Send EncryptedExtensions
                                | [Send CertificateRequest]
 Can send                       | [Send Certificate + CertificateVerify]
 app data                       | Send Finished
 after   -->                    | K_send = application
 here                  +--------+--------+
              No 0-RTT |                 | 0-RTT
                       |                 |
   K_recv = handshake  |                 | K_recv = early data
 [Skip decrypt errors] |    +------> WAIT_EOED --+
                       |    |       Recv |       | Recv EndOfEarlyData
                       |    | early data |       | K_recv = handshake
                       |    +------------+       |
                       |                         |
                       +-> WAIT_FLIGHT2 <--------+
                                |
                       +--------+--------+
               No auth |                 | Cert-based client auth
                       |                 |
                       |                 v
                       |             WAIT_CERT
                       |        Recv |       | Recv Certificate
                       |       empty |       v
                       | Certificate |    WAIT_CV
                       |             |       | Recv
                       |             v       | CertificateVerify
                       +-> WAIT_FINISHED <---+
                                | Recv Finished
                                | K_recv = application
                                v
                            CONNECTED

## B. Protocol Data Structures and Constant Values

   This appendix provides the normative protocol types and the
   definitions for constants.  Values listed as "_RESERVED" were used in
   previous versions of TLS and are listed here for completeness.  TLS
   1.3 implementations MUST NOT send them but might receive them from
   older TLS implementations. 





### B.1. Record Layer

      enum {
          invalid(0),
          change_cipher_spec(20),
          alert(21),
          handshake(22),
          application_data(23),
          (255)
      } ContentType;

      struct {
          ContentType type;
          ProtocolVersion legacy_record_version;
          uint16 length;
          opaque fragment[TLSPlaintext.length];
      } TLSPlaintext;

      struct {
          opaque content[TLSPlaintext.length];
          ContentType type;
          uint8 zeros[length_of_padding];
      } TLSInnerPlaintext;

      struct {
          ContentType opaque_type = application_data; /* 23 */
          ProtocolVersion legacy_record_version = 0x0303; /* TLS v1.2 */
          uint16 length;
          opaque encrypted_record[TLSCiphertext.length];
      } TLSCiphertext;

### B.2. Alert Messages






















      enum { warning(1), fatal(2), (255) } AlertLevel;

      enum {
          close_notify(0),
          unexpected_message(10),
          bad_record_mac(20),
          decryption_failed_RESERVED(21),
          record_overflow(22),
          decompression_failure_RESERVED(30),
          handshake_failure(40),
          no_certificate_RESERVED(41),
          bad_certificate(42),
          unsupported_certificate(43),
          certificate_revoked(44),
          certificate_expired(45),
          certificate_unknown(46),
          illegal_parameter(47),
          unknown_ca(48),
          access_denied(49),
          decode_error(50),
          decrypt_error(51),
          export_restriction_RESERVED(60),
          protocol_version(70),
          insufficient_security(71),
          internal_error(80),
          inappropriate_fallback(86),
          user_canceled(90),
          no_renegotiation_RESERVED(100),
          missing_extension(109),
          unsupported_extension(110),
          certificate_unobtainable_RESERVED(111),
          unrecognized_name(112),
          bad_certificate_status_response(113),
          bad_certificate_hash_value_RESERVED(114),
          unknown_psk_identity(115),
          certificate_required(116),
          general_error(117),
          no_application_protocol(120),
          (255)
      } AlertDescription;

      struct {
          AlertLevel level;
          AlertDescription description;
      } Alert;

### B.3. Handshake Protocol







      enum {
          hello_request_RESERVED(0),
          client_hello(1),
          server_hello(2),
          hello_verify_request_RESERVED(3),
          new_session_ticket(4),
          end_of_early_data(5),
          hello_retry_request_RESERVED(6),
          encrypted_extensions(8),
          certificate(11),
          server_key_exchange_RESERVED(12),
          certificate_request(13),
          server_hello_done_RESERVED(14),
          certificate_verify(15),
          client_key_exchange_RESERVED(16),
          finished(20),
          certificate_url_RESERVED(21),
          certificate_status_RESERVED(22),
          supplemental_data_RESERVED(23),
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

#### B.3.1. Key Exchange Messages












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

    struct {
        ProtocolVersion legacy_version = 0x0303;    /* TLS v1.2 */
        Random random;
        opaque legacy_session_id_echo<0..32>;
        CipherSuite cipher_suite;
        uint8 legacy_compression_method = 0;
        Extension extensions<6..2^16-1>;
    } ServerHello;

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

    struct {
        NamedGroup group;
        opaque key_exchange<1..2^16-1>;
    } KeyShareEntry;

    struct {
        KeyShareEntry client_shares<0..2^16-1>;
    } KeyShareClientHello;

    struct {
        NamedGroup selected_group;
    } KeyShareHelloRetryRequest;

    struct {
        KeyShareEntry server_share;
    } KeyShareServerHello;

    struct {
        uint8 legacy_form = 4;
        opaque X[coordinate_length];
        opaque Y[coordinate_length];
    } UncompressedPointRepresentation;

    enum { psk_ke(0), psk_dhe_ke(1), (255) } PskKeyExchangeMode;

    struct {
        PskKeyExchangeMode ke_modes<1..255>;
    } PskKeyExchangeModes;

    struct {} Empty;

    struct {
        select (Handshake.msg_type) {
            case new_session_ticket:   uint32 max_early_data_size;
            case client_hello:         Empty;
            case encrypted_extensions: Empty;
        };
    } EarlyDataIndication;

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

##### B.3.1.1. Version Extension

      struct {
          select (Handshake.msg_type) {
              case client_hello:
                   ProtocolVersion versions<2..254>;

              case server_hello: /* and HelloRetryRequest */
                   ProtocolVersion selected_version;
          };
      } SupportedVersions;

##### B.3.1.2. Cookie Extension

      struct {
          opaque cookie<1..2^16-1>;
      } Cookie;

##### B.3.1.3. Signature Algorithm Extension



















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
          obsolete_RESERVED(0x0000..0x0200),
          dsa_sha1_RESERVED(0x0202),
          obsolete_RESERVED(0x0204..0x0400),
          dsa_sha256_RESERVED(0x0402),
          obsolete_RESERVED(0x0404..0x0500),
          dsa_sha384_RESERVED(0x0502),
          obsolete_RESERVED(0x0504..0x0600),
          dsa_sha512_RESERVED(0x0602),
          obsolete_RESERVED(0x0604..0x06FF),
          private_use(0xFE00..0xFFFF),
          (0xFFFF)
      } SignatureScheme;

      struct {
          SignatureScheme supported_signature_algorithms<2..2^16-2>;
      } SignatureSchemeList;





##### B.3.1.4. Supported Groups Extension

      enum {
          unallocated_RESERVED(0x0000),

          /* Elliptic Curve Groups (ECDHE) */
          obsolete_RESERVED(0x0001..0x0016),
          secp256r1(0x0017), secp384r1(0x0018), secp521r1(0x0019),
          obsolete_RESERVED(0x001A..0x001C),
          x25519(0x001D), x448(0x001E),

          /* Finite Field Groups (DHE) */
          ffdhe2048(0x0100), ffdhe3072(0x0101), ffdhe4096(0x0102),
          ffdhe6144(0x0103), ffdhe8192(0x0104),

          /* Reserved Code Points */
          ffdhe_private_use(0x01FC..0x01FF),
          ecdhe_private_use(0xFE00..0xFEFF),
          obsolete_RESERVED(0xFF01..0xFF02),
          (0xFFFF)
      } NamedGroup;

      struct {
          NamedGroup named_group_list<2..2^16-1>;
      } NamedGroupList;

   Values within "obsolete_RESERVED" ranges are used in previous
   versions of TLS and MUST NOT be offered or negotiated by TLS 1.3
   implementations.  The obsolete curves have various known/theoretical
   weaknesses or have had very little usage, in some cases only due to
   unintentional server configuration issues.  They are no longer
   considered appropriate for general use and should be assumed to be
   potentially unsafe.  The set of curves specified here is sufficient
   for interoperability with all currently deployed and properly
   configured TLS implementations.

#### B.3.2. Server Parameters Messages

















      opaque DistinguishedName<1..2^16-1>;

      struct {
          DistinguishedName authorities<3..2^16-1>;
      } CertificateAuthoritiesExtension;

      struct {
          opaque certificate_extension_oid<1..2^8-1>;
          opaque certificate_extension_values<0..2^16-1>;
      } OIDFilter;

      struct {
          OIDFilter filters<0..2^16-1>;
      } OIDFilterExtension;

      struct {} PostHandshakeAuth;

      struct {
          Extension extensions<0..2^16-1>;
      } EncryptedExtensions;

      struct {
          opaque certificate_request_context<0..2^8-1>;
          Extension extensions<0..2^16-1>;
      } CertificateRequest;

#### B.3.3. Authentication Messages



























      enum {
          X509(0),
          OpenPGP_RESERVED(1),
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

      struct {
          SignatureScheme algorithm;
          opaque signature<0..2^16-1>;
      } CertificateVerify;

      struct {
          opaque verify_data[Hash.length];
      } Finished;

#### B.3.4. Ticket Establishment

      struct {
          uint32 ticket_lifetime;
          uint32 ticket_age_add;
          opaque ticket_nonce<0..255>;
          opaque ticket<1..2^16-1>;
          Extension extensions<0..2^16-1>;
      } NewSessionTicket;

#### B.3.5. Updating Keys










      struct {} EndOfEarlyData;

      enum {
          update_not_requested(0), update_requested(1), (255)
      } KeyUpdateRequest;

      struct {
          KeyUpdateRequest request_update;
      } KeyUpdate;

### B.4. Cipher Suites

   A cipher suite defines the pair of the AEAD algorithm and hash
   algorithm to be used with HKDF.  Cipher suite names follow the naming
   convention:

      CipherSuite TLS_AEAD_HASH = VALUE;

   +===========+=======================================================+
   | Component | Contents                                              |
   +===========+=======================================================+
   | TLS       | The string "TLS"                                      |
   +-----------+-------------------------------------------------------+
   | AEAD      | The AEAD algorithm used for record protection         |
   +-----------+-------------------------------------------------------+
   | HASH      | The hash algorithm used with HKDF and                 |
   |           | Transcript-Hash                                       |
   +-----------+-------------------------------------------------------+
   | VALUE     | The two byte ID assigned for this cipher              |
   |           | suite                                                 |
   +-----------+-------------------------------------------------------+

                    Table 4: Cipher Suite Name Structure

   This specification defines the following cipher suites for use with
   TLS 1.3. 
              +==============================+=============+
              | Description                  | Value       |
              +==============================+=============+
              | TLS_AES_128_GCM_SHA256       | {0x13,0x01} |
              +------------------------------+-------------+
              | TLS_AES_256_GCM_SHA384       | {0x13,0x02} |
              +------------------------------+-------------+
              | TLS_CHACHA20_POLY1305_SHA256 | {0x13,0x03} |
              +------------------------------+-------------+
              | TLS_AES_128_CCM_SHA256       | {0x13,0x04} |
              +------------------------------+-------------+
              | TLS_AES_128_CCM_8_SHA256     | {0x13,0x05} |
              +------------------------------+-------------+

                        Table 5: Cipher Suite List

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

## C. Implementation Notes

   The TLS protocol cannot prevent many common security mistakes.  This
   appendix provides several recommendations to assist implementors.
   [RFC8448] provides test vectors for TLS 1.3 handshakes.

### C.1. Random Number Generation and Seeding

   TLS requires a cryptographically secure pseudorandom number generator
   (CSPRNG).  A performant and appropriately-secure CSPRNG is provided
   by most operating systems or can be sourced from a cryptographic
   library.  It is RECOMMENDED to use an existing CSPRNG implementation
   in preference to crafting a new one.  Many adequate cryptographic
   libraries are already available under favorable license terms.
   Should those prove unsatisfactory, [RFC4086] provides guidance on the
   generation of random values. 
   TLS uses random values (1) in public protocol fields such as the
   public Random values in the ClientHello and ServerHello and (2) to
   generate keying material.  With a properly functioning CSPRNG, this
   does not present a security problem, as it is not feasible to
   determine the CSPRNG state from its output.  However, with a broken
   CSPRNG, it may be possible for an attacker to use the public output
   to determine the CSPRNG internal state and thereby predict the keying
   material, as documented in [CHECKOWAY ] and [DSA-1571-1].

   Implementations can provide extra security against this form of
   attack by using separate CSPRNGs to generate public and private
   values.

   [RFC8937] describes a way for security protocol implementations to
   augment their (pseudo)random number generators using a long-term
   private key and a deterministic signature function.  This improves
   randomness from broken or otherwise subverted random number
   generators.

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

   Note that it is common practice in some protocols to use the same
   certificate in both client and server modes.  This setting has not
   been extensively analyzed and it is the responsibility of the higher
   level protocol to ensure there is no ambiguity in this case about the
   higher-level semantics.

### C.3. Implementation Pitfalls

   Implementation experience has shown that certain parts of earlier TLS
   specifications are not easy to understand and have been a source of
   interoperability and security problems.  Many of these areas have
   been clarified in this document but this appendix contains a short
   list of the most important things that require special attention from
   implementors. 
   TLS protocol issues:

   *  Do you correctly handle handshake messages that are fragmented to
      multiple TLS records (see Section 5.1)?  Do you correctly handle
      corner cases like a ClientHello that is split into several small
      fragments?  Do you fragment handshake messages that exceed the
      maximum fragment size?  In particular, the Certificate and
      CertificateRequest handshake messages can be large enough to
      require fragmentation.  Certificate compression as defined in
      [RFC8879] can be used to reduce the risk of fragmentation.

   *  Do you ignore the TLS record layer version number in all
      unencrypted TLS records (see Appendix E )?

   *  Have you ensured that all support for SSL, RC4, EXPORT ciphers,
      and MD5 (via the "signature_algorithms" extension) is completely
      removed from all possible configurations that support TLS 1.3 or
      later, and that attempts to use these obsolete capabilities fail
      correctly? (see Appendix E )?

   *  Do you handle TLS extensions in ClientHellos correctly, including
      unknown extensions?

   *  When the server has requested a client certificate but no suitable
      certificate is available, do you correctly send an empty
      Certificate message, instead of omitting the whole message (see Section 4.4.2)?

   *  When processing the plaintext fragment produced by AEAD-Decrypt
      and scanning from the end for the ContentType, do you avoid
      scanning past the start of the cleartext in the event that the
      peer has sent a malformed plaintext of all zeros?

   *  Do you properly ignore unrecognized cipher suites (Section 4.1.2),
      hello extensions (Section 4.2), named groups (Section 4.2.7), key
      shares (Section 4.2.8), supported versions (Section 4.2.1), and
      signature algorithms (Section 4.2.3) in the ClientHello?

   *  As a server, do you send a HelloRetryRequest to clients which
      support a compatible (EC)DHE group but do not predict it in the
      "key_share" extension?  As a client, do you correctly handle a
      HelloRetryRequest from the server?

   Cryptographic details:

   *  What countermeasures do you use to prevent timing attacks
      [TIMING ]?
   *  When using Diffie-Hellman key exchange, do you correctly preserve
      leading zero bytes in the negotiated key (see Section 7.4.1)?

   *  Does your TLS client check that the Diffie-Hellman parameters sent
      by the server are acceptable (see Section 4.2.8.1)?

   *  Do you use a strong and, most importantly, properly seeded random
      number generator (see Appendix C.1) when generating Diffie-Hellman
      private values, the ECDSA "k" parameter, and other security-
      critical values?  It is RECOMMENDED that implementations implement
      "deterministic ECDSA" as specified in [RFC6979].  Note that purely
      deterministic ECC signatures such as deterministic ECDSA and EdDSA
      may be vulnerable to certain side-channel and fault injection
      attacks in easily accessible IoT devices.

   *  Do you zero-pad Diffie-Hellman public key values and shared
      secrets to the group size (see Section 4.2.8.1 and Section 7.4.1)?

   *  Do you verify signatures after making them, to protect against
      RSA-CRT key leaks [FW15]?

### C.4. Client and Server Tracking Prevention

   Clients SHOULD NOT reuse a ticket for multiple connections.  Reuse of
   a ticket allows passive observers to correlate different connections.
   Servers that issue tickets SHOULD offer at least as many tickets as
   the number of connections that a client might use; for example, a web
   browser using HTTP/1.1 [RFC9112] might open six connections to a
   server.  Servers SHOULD issue new tickets with every connection.
   This ensures that clients are always able to use a new ticket when
   creating a new connection.

   Offering a ticket to a server additionally allows the server to
   correlate different connections.  This is possible independent of
   ticket reuse.  Client applications SHOULD NOT offer tickets across
   connections that are meant to be uncorrelated.  For example, [FETCH ]
   defines network partition keys to separate cache lookups in web
   browsers.

   Clients and Servers SHOULD NOT reuse a key share for multiple
   connections.  Reuse of a key share allows passive observers to
   correlate different connections.  Reuse of a client key share to the
   same server additionally allows the server to correlate different
   connections.

   It is RECOMMENDED that the labels for external identities be selected
   so that they do not provide additional information about the identity
   of the user.  For instance, if the label includes an e-mail address,
   then this trivially identifies the user to a passive attacker, unlike
   the client's Certificate, which is encrypted.  There are a number of
   potential ways to avoid this risk, including (1) using random
   identity labels (2) pre-encrypting the identity under a key known to
   the server or (3) using the Encrypted Client Hello
   [I-D.ietf-tls-esni ] extension.

   If an external PSK identity is used for multiple connections, then it
   will generally be possible for an external observer to track clients
   and/or servers across connections.  Use of the Encrypted Client Hello
   [I-D.ietf-tls-esni ] extension can mitigate this risk, as can
   mechanisms external to TLS that rotate or encrypt the PSK identity.

### C.5. Unauthenticated Operation

   Previous versions of TLS offered explicitly unauthenticated cipher
   suites based on anonymous Diffie-Hellman.  These modes have been
   deprecated in TLS 1.3.  However, it is still possible to negotiate
   parameters that do not provide verifiable server authentication by
   several methods, including:

   *  Raw public keys [RFC7250].

   *  Using a public key contained in a certificate but without
      validation of the certificate chain or any of its contents.

   Either technique used alone is vulnerable to man-in-the-middle
   attacks and therefore unsafe for general use.  However, it is also
   possible to bind such connections to an external authentication
   mechanism via out-of-band validation of the server's public key,
   trust on first use, or a mechanism such as channel bindings (though
   the channel bindings described in [RFC5929] are not defined for TLS
   1.3).  If no such mechanism is used, then the connection has no
   protection against active man-in-the-middle attack; applications MUST
   NOT use TLS in such a way absent explicit configuration or a specific
   application profile.

## D. Updates to TLS 1.2

   To align with the names used this document, the following terms from
   [RFC5246] are renamed:

   *  The master secret, computed in Section 8.1 of [RFC5246], is
      renamed to the main secret.  It is referred to as main_secret in
      formulas and structures, instead of master_secret.  However, the
      label parameter to the PRF function is left unchanged for
      compatibility. 
   *  The premaster secret is renamed to the preliminary secret.  It is
      referred to as preliminary_secret in formulas and structures,
      instead of pre_master_secret.

   *  The PreMasterSecret and EncryptedPreMasterSecret structures,
      defined in Section 7.4.7.1 of [RFC5246], are renamed to
      PreliminarySecret and EncryptedPreliminarySecret, respectively.

   Correspondingly, the extension defined in [RFC7627] is renamed to the
   "Extended Main Secret" extension.  The extension code point is
   renamed to "extended_main_secret".  The label parameter to the PRF
   function in Section 4 of [RFC7627] is left unchanged for
   compatibility.

## E. Backward Compatibility

   The TLS protocol provides a built-in mechanism for version
   negotiation between endpoints potentially supporting different
   versions of TLS.

   TLS 1.x and SSL 3.0 use compatible ClientHello messages.  Servers can
   also handle clients trying to use future versions of TLS as long as
   the ClientHello format remains compatible and there is at least one
   protocol version supported by both the client and the server.

   Prior versions of TLS used the record layer version number
   (TLSPlaintext.legacy_record_version and
   TLSCiphertext.legacy_record_version) for various purposes.  As of TLS
   1.3, this field is deprecated.  The value of
   TLSPlaintext.legacy_record_version MUST be ignored by all
   implementations.  The value of TLSCiphertext.legacy_record_version is
   included in the additional data for deprotection but MAY otherwise be
   ignored or MAY be validated to match the fixed constant value.
   Version negotiation is performed using only the handshake versions
   (ClientHello.legacy_version and ServerHello.legacy_version, as well
   as the ClientHello, HelloRetryRequest, and ServerHello
   "supported_versions" extensions).  To maximize interoperability with
   older endpoints, implementations that negotiate the use of TLS
   1.0-1.2 SHOULD set the record layer version number to the negotiated
   version for the ServerHello and all records thereafter.

   For maximum compatibility with previously non-standard behavior and
   misconfigured deployments, all implementations SHOULD support
   validation of certification paths based on the expectations in this
   document, even when handling prior TLS versions' handshakes (see Section 4.4.2.2). 
   TLS 1.2 and prior supported an "Extended Main Secret" [RFC7627]
   extension which digested large parts of the handshake transcript into
   the secret and derived keys.  Note this extension was renamed in Appendix D .  Because TLS 1.3 always hashes in the transcript up to
   the server Finished, implementations which support both TLS 1.3 and
   earlier versions SHOULD indicate the use of the Extended Main Secret
   extension in their APIs whenever TLS 1.3 is used.

### E.1. Negotiating with an Older Server

   A TLS 1.3 client who wishes to negotiate with servers that do not
   support TLS 1.3 will send a normal TLS 1.3 ClientHello containing
   0x0303 (TLS 1.2) in ClientHello.legacy_version but with the correct
   version(s) in the "supported_versions" extension.  If the server does
   not support TLS 1.3, it will respond with a ServerHello containing an
   older version number.  If the client agrees to use this version, the
   negotiation will proceed as appropriate for the negotiated protocol.
   A client using a ticket for resumption SHOULD initiate the connection
   using the version that was previously negotiated.

   Note that 0-RTT data is not compatible with older servers and SHOULD
   NOT be sent absent knowledge that the server supports TLS 1.3.  See Appendix E.3.

   If the version chosen by the server is not supported by the client
   (or is not acceptable), the client MUST abort the handshake with a
   "protocol_version" alert.

   Some legacy server implementations are known to not implement the TLS
   specification properly and might abort connections upon encountering
   TLS extensions or versions which they are not aware of.
   Interoperability with buggy servers is a complex topic beyond the
   scope of this document.  Multiple connection attempts may be required
   to negotiate a backward-compatible connection; however, this practice
   is vulnerable to downgrade attacks and is NOT RECOMMENDED.

### E.2. Negotiating with an Older Client

   A TLS server can also receive a ClientHello indicating a version
   number smaller than its highest supported version.  If the
   "supported_versions" extension is present, the server MUST negotiate
   using that extension as described in Section 4.2.1.  If the
   "supported_versions" extension is not present, the server MUST
   negotiate the minimum of ClientHello.legacy_version and TLS 1.2.  For
   example, if the server supports TLS 1.0, 1.1, and 1.2, and
   legacy_version is TLS 1.0, the server will proceed with a TLS 1.0
   ServerHello.  If the "supported_versions" extension is absent and the
   server only supports versions greater than 
   ClientHello.legacy_version, the server MUST abort the handshake with
   a "protocol_version" alert.

   Note that earlier versions of TLS did not clearly specify the record
   layer version number value in all cases
   (TLSPlaintext.legacy_record_version).  Servers will receive various
   TLS 1.x versions in this field, but its value MUST always be ignored.

### E.3. 0-RTT Backward Compatibility

   0-RTT data is not compatible with older servers.  An older server
   will respond to the ClientHello with an older ServerHello, but it
   will not correctly skip the 0-RTT data and will fail to complete the
   handshake.  This can cause issues when a client attempts to use
   0-RTT, particularly against multi-server deployments.  For example, a
   deployment could deploy TLS 1.3 gradually with some servers
   implementing TLS 1.3 and some implementing TLS 1.2, or a TLS 1.3
   deployment could be downgraded to TLS 1.2.

   A client that attempts to send 0-RTT data MUST fail a connection if
   it receives a ServerHello with TLS 1.2 or older.  It can then retry
   the connection with 0-RTT disabled.  To avoid a downgrade attack, the
   client SHOULD NOT disable TLS 1.3, only 0-RTT.

   To avoid this error condition, multi-server deployments SHOULD ensure
   a uniform and stable deployment of TLS 1.3 without 0-RTT prior to
   enabling 0-RTT.

### E.4. Middlebox Compatibility Mode

   Field measurements [Ben17a ] [Ben17b ] [Res17a ] [Res17b ] have found
   that a significant number of middleboxes misbehave when a TLS client/
   server pair negotiates TLS 1.3.  Implementations can increase the
   chance of making connections through those middleboxes by making the
   TLS 1.3 handshake look more like a TLS 1.2 handshake:

   *  The client always provides a non-empty session ID in the
      ClientHello, as described in the legacy_session_id section of Section 4.1.2.

   *  If not offering early data, the client sends a dummy
      change_cipher_spec record (see the third paragraph of Section 5)
      immediately before its second flight.  This may either be before
      its second ClientHello or before its encrypted handshake flight.
      If offering early data, the record is placed immediately after the
      first ClientHello. 
   *  The server sends a dummy change_cipher_spec record immediately
      after its first handshake message.  This may either be after a
      ServerHello or a HelloRetryRequest.

   When put together, these changes make the TLS 1.3 handshake resemble
   TLS 1.2 session resumption, which improves the chance of successfully
   connecting through middleboxes.  This "compatibility mode" is
   partially negotiated: the client can opt to provide a session ID or
   not, and the server has to echo it.  Either side can send
   change_cipher_spec at any time during the handshake, as they must be
   ignored by the peer, but if the client sends a non-empty session ID,
   the server MUST send the change_cipher_spec as described in this
   appendix.

### E.5. Security Restrictions Related to Backward Compatibility

   Implementations negotiating the use of older versions of TLS SHOULD
   prefer forward secret and AEAD cipher suites, when available.

   The security of RC4 cipher suites is considered insufficient for the
   reasons cited in [RFC7465].  Implementations MUST NOT offer or
   negotiate RC4 cipher suites for any version of TLS for any reason.

   Old versions of TLS permitted the use of very low strength ciphers.
   Ciphers with a strength less than 112 bits MUST NOT be offered or
   negotiated for any version of TLS for any reason.

   The security of SSL 2.0 [SSL2], SSL 3.0 [RFC6101], TLS 1.0 [RFC2246],
   and TLS 1.1 [RFC4346] are considered insufficient for the reasons
   enumerated in [RFC6176], [RFC7568], and [RFC8996] and they MUST NOT
   be negotiated for any reason.

   Implementations MUST NOT send an SSL version 2.0 compatible CLIENT-
   HELLO.  Implementations MUST NOT negotiate TLS 1.3 or later using an
   SSL version 2.0 compatible CLIENT-HELLO.  Implementations are NOT
   RECOMMENDED to accept an SSL version 2.0 compatible CLIENT-HELLO to
   negotiate older versions of TLS.

   Implementations MUST NOT send a ClientHello.legacy_version or
   ServerHello.legacy_version set to 0x0300 or less.  Any endpoint
   receiving a Hello message with ClientHello.legacy_version or
   ServerHello.legacy_version set to 0x0300 MUST abort the handshake
   with a "protocol_version" alert.

   Implementations MUST NOT send any records with a version less than
   0x0300.  Implementations SHOULD NOT accept any records with a version
   less than 0x0300 (but may inadvertently do so if the record version
   number is ignored completely). 
   Implementations MUST NOT use the Truncated HMAC extension, defined in Section 7 of [RFC6066], as it is not applicable to AEAD algorithms
   and has been shown to be insecure in some scenarios.

## F. Overview of Security Properties

   A complete security analysis of TLS is outside the scope of this
   document.  In this appendix, we provide an informal description of
   the desired properties as well as references to more detailed work in
   the research literature which provides more formal definitions.

   We cover properties of the handshake separately from those of the
   record layer.

### F.1. Handshake

   The TLS handshake is an Authenticated Key Exchange (AKE) protocol
   which is intended to provide both one-way authenticated (server-only)
   and mutually authenticated (client and server) functionality.  At the
   completion of the handshake, each side outputs its view of the
   following values:

   *  A set of "session keys" (the various secrets derived from the main
      secret) from which can be derived a set of working keys.  Note
      that when early data is in use, secrets are also derived from the
      early secret.  These enjoy somewhat weaker properties than those
      derived from the main secret, as detailed below.

   *  A set of cryptographic parameters (algorithms, etc.).

   *  The identities of the communicating parties.

   We assume the attacker to be an active network attacker, which means
   it has complete control over the network used to communicate between
   the parties [RFC3552].  Even under these conditions, the handshake
   should provide the properties listed below.  Note that these
   properties are not necessarily independent, but reflect the protocol
   consumers' needs.

   Establishing the same session keys:  The handshake needs to output
      the same set of session keys on both sides of the handshake,
      provided that it completes successfully on each endpoint (see
      [CK01], Definition 1, part 1).

   Secrecy of the session keys:  The shared session keys should be known 
      only to the communicating parties and not to the attacker (see
      [CK01]; Definition 1, part 2).  Note that in a unilaterally
      authenticated connection, the attacker can establish its own
      session keys with the server, but those session keys are distinct
      from those established by the client.

   Peer Authentication:  The client's view of the peer identity should
      reflect the server's identity.  If the client is authenticated,
      the server's view of the peer identity should match the client's
      identity.

   Uniqueness of the session keys:  Any two distinct handshakes should
      produce distinct, unrelated session keys.  Individual session keys
      produced by a handshake should also be distinct and independent.

   Downgrade Protection:  The cryptographic parameters should be the
      same on both sides and should be the same as if the peers had been
      communicating in the absence of an attack (see [BBFGKZ16];
      Definitions 8 and 9).

   Forward secret with respect to long-term keys:  If the long-term
      keying material (in this case the signature keys in certificate-
      based authentication modes or the external/resumption PSK in PSK
      with (EC)DHE modes) is compromised after the handshake is
      complete, this does not compromise the security of the session key
      (see [DOW92]), as long as the session key itself (and all material
      that could be used to recreate the session key) has been erased.
      In particular, private keys corresponding to key shares, shared
      secrets, and keys derived in the TLS Key Schedule other than
      binder_key, resumption_secret, and PSKs derived from the
      resumption_secret also need to be erased.  The forward secrecy
      property is not satisfied when PSK is used in the "psk_ke"
      PskKeyExchangeMode.  Failing to erase keys or secrets intended to
      be ephemeral or connection-specific in effect creates additional
      long-term keys that must be protected.  Compromise of those long-
      term keys (even after the handshake is complete) can result in
      loss of protection for the connection's traffic.

   Key Compromise Impersonation (KCI) resistance:  In a mutually
      authenticated connection with certificates, compromising the long-
      term secret of one actor should not break that actor’s
      authentication of their peer in the given connection (see
      [HGFS15]).  For example, if a client's signature key is
      compromised, it should not be possible to impersonate arbitrary
      servers to that client in subsequent handshakes.

   Protection of endpoint identities:  The server's identity 
      (certificate) should be protected against passive attackers.  The
      client's identity (certificate) should be protected against both
      passive and active attackers.  This property does not hold for
      cipher suites without confidentiality; while this specification
      does not define any such cipher suites, other documents may do so.

   Informally, the signature-based modes of TLS 1.3 provide for the
   establishment of a unique, secret, shared key established by an
   (EC)DHE key exchange and authenticated by the server's signature over
   the handshake transcript, as well as tied to the server's identity by
   a MAC.  If the client is authenticated by a certificate, it also
   signs over the handshake transcript and provides a MAC tied to both
   identities.  [SIGMA ] describes the design and analysis of this type
   of key exchange protocol.  If fresh (EC)DHE keys are used for each
   connection, then the output keys are forward secret.

   The external PSK and resumption PSK bootstrap from a long-term shared
   secret into a unique per-connection set of short-term session keys.
   This secret may have been established in a previous handshake.  If
   PSK with (EC)DHE key establishment is used, these session keys will
   also be forward secret.  The resumption PSK has been designed so that
   the resumption secret computed by connection N and needed to form
   connection N+1 is separate from the traffic keys used by connection
   N, thus providing forward secrecy between the connections.  In
   addition, if multiple tickets are established on the same connection,
   they are associated with different keys, so compromise of the PSK
   associated with one ticket does not lead to the compromise of
   connections established with PSKs associated with other tickets.
   This property is most interesting if tickets are stored in a database
   (and so can be deleted) rather than if they are self-encrypted. 
   Forward secrecy limits the effect of key leakage in one direction
   (compromise of a key at time T2 does not compromise some key at time
   T1 where T1 < T2).  Protection in the other direction (compromise at
   time T1 does not compromise keys at time T2) can be achieved by
   rerunning (EC)DHE.  If a long-term authentication key has been
   compromised, a full handshake with (EC)DHE gives protection against
   passive attackers.  If the resumption_secret has been compromised, a
   resumption handshake with (EC)DHE gives protection against passive
   attackers and a full handshake with (EC)DHE gives protection against
   active attackers.  If a traffic secret has been compromised, any
   handshake with (EC)DHE gives protection against active attackers.
   Using the terms in [RFC7624], forward secrecy without rerunning
   (EC)DHE does not stop an attacker from doing static key exfiltration.
   After key exfiltration of application_traffic_secret_N, an attacker
   can e.g., passively eavesdrop on all future data sent on the
   connection including data encrypted with
   application_traffic_secret_N+1, application_traffic_secret_N+2, etc.
   Frequently rerunning (EC)DHE forces an attacker to do dynamic key
   exfiltration (or content exfiltration).

   The PSK binder value forms a binding between a PSK and the current
   handshake, as well as between the session where the PSK was
   established and the current session.  This binding transitively
   includes the original handshake transcript, because that transcript
   is digested into the values which produce the resumption secret.
   This requires that both the KDF used to produce the resumption secret
   and the MAC used to compute the binder be collision resistant.  See Appendix F.1.1 for more on this.  Note: The binder does not cover the
   binder values from other PSKs, though they are included in the
   Finished MAC.

   Note: This specification does not currently permit the server to send
   a certificate_request message in non-certificate-based handshakes
   (e.g., PSK).  If this restriction were to be relaxed in future, the
   client's signature would not cover the server's certificate directly.
   However, if the PSK was established through a NewSessionTicket, the
   client's signature would transitively cover the server's certificate
   through the PSK binder.  [PSK-FINISHED ] describes a concrete attack
   on constructions that do not bind to the server's certificate (see
   also [Kraw16]).  It is unsafe to use certificate-based client
   authentication when the client might potentially share the same PSK/
   key-id pair with two different endpoints.  In the absence of some
   other specification to the contrary, implementations MUST NOT combine
   external PSKs with certificate-based authentication of either the
   client or server.  [RFC8773] provides an extension to permit this,
   but has not received the level of analysis as this specification. 
   If an exporter is used, then it produces values which are unique and
   secret (because they are generated from a unique session key).
   Exporters computed with different labels and contexts are
   computationally independent, so it is not feasible to compute one
   from another or the session secret from the exported value.  Note:
   Exporters can produce arbitrary-length values; if exporters are to be
   used as channel bindings, the exported value MUST be large enough to
   provide collision resistance.  The exporters provided in TLS 1.3 are
   derived from the same Handshake Contexts as the early traffic keys
   and the application traffic keys, respectively, and thus have similar
   security properties.  Note that they do not include the client's
   certificate; future applications which wish to bind to the client's
   certificate may need to define a new exporter that includes the full
   handshake transcript.

   For all handshake modes, the Finished MAC (and, where present, the
   signature) prevents downgrade attacks.  In addition, the use of
   certain bytes in the random nonces as described in Section 4.1.3   allows the detection of downgrade to previous TLS versions.  See
   [BBFGKZ16] for more details on TLS 1.3 and downgrade.

   As soon as the client and the server have exchanged enough
   information to establish shared keys, the remainder of the handshake
   is encrypted, thus providing protection against passive attackers,
   even if the computed shared key is not authenticated.  Because the
   server authenticates before the client, the client can ensure that if
   it authenticates to the server, it only reveals its identity to an
   authenticated server.  Note that implementations must use the
   provided record-padding mechanism during the handshake to avoid
   leaking information about the identities due to length.  The client's
   proposed PSK identities are not encrypted, nor is the one that the
   server selects.

#### F.1.1. Key Derivation and HKDF

   Key derivation in TLS 1.3 uses HKDF as defined in [RFC5869] and its
   two components, HKDF-Extract and HKDF-Expand.  The full rationale for
   the HKDF construction can be found in [Kraw10] and the rationale for
   the way it is used in TLS 1.3 in [KW16].  Throughout this document,
   each application of HKDF-Extract is followed by one or more
   invocations of HKDF-Expand.  This ordering should always be followed
   (including in future revisions of this document); in particular, one
   SHOULD NOT use an output of HKDF-Extract as an input to another
   application of HKDF-Extract without an HKDF-Expand in between.
   Multiple applications of HKDF-Expand to some of the same inputs are
   allowed as long as these are differentiated via the key and/or the
   labels. 
   Note that HKDF-Expand implements a pseudorandom function (PRF) with
   both inputs and outputs of variable length.  In some of the uses of
   HKDF in this document (e.g., for generating exporters and the
   resumption_secret), it is necessary that the application of HKDF-
   Expand be collision resistant; namely, it should be infeasible to
   find two different inputs to HKDF-Expand that output the same value.
   This requires the underlying hash function to be collision resistant
   and the output length from HKDF-Expand to be of size at least 256
   bits (or as much as needed for the hash function to prevent finding
   collisions).

#### F.1.2. Certificate-Based Client Authentication

   A client that has sent certificate-based authentication data to a
   server, either during the handshake or in post-handshake
   authentication, cannot be sure whether the server afterwards
   considers the client to be authenticated or not.  If the client needs
   to determine if the server considers the connection to be
   unilaterally or mutually authenticated, this has to be provisioned by
   the application layer.  See [CHHSV17] for details.  In addition, the
   analysis of post-handshake authentication from [Kraw16] shows that
   the client identified by the certificate sent in the post-handshake
   phase possesses the traffic key.  This party is therefore the client
   that participated in the original handshake or one to whom the
   original client delegated the traffic key (assuming that the traffic
   key has not been compromised).

#### F.1.3. 0-RTT

   The 0-RTT mode of operation generally provides security properties
   similar to those of 1-RTT data, with the two exceptions that the
   0-RTT encryption keys do not provide full forward secrecy and that
   the server is not able to guarantee uniqueness of the handshake (non-
   replayability) without keeping potentially undue amounts of state.
   See Section 8 for mechanisms to limit the exposure to replay.

#### F.1.4. Exporter Independence

   The exporter_secret and early_exporter_secret are derived to be
   independent of the traffic keys and therefore do not represent a
   threat to the security of traffic encrypted with those keys.
   However, because these secrets can be used to compute any exporter
   value, they SHOULD be erased as soon as possible.  If the total set
   of exporter labels is known, then implementations SHOULD pre-compute
   the inner Derive-Secret stage of the exporter computation for all
   those labels, then erase the [early_]exporter_secret, followed by
   each inner values as soon as it is known that it will not be needed
   again. 





#### F.1.5. Post-Compromise Security

   TLS does not provide security for handshakes which take place after
   the peer's long-term secret (signature key or external PSK) is
   compromised.  It therefore does not provide post-compromise security
   [CCG16], sometimes also referred to as backwards or future secrecy.
   This is in contrast to KCI resistance, which describes the security
   guarantees that a party has after its own long-term secret has been
   compromised.

#### F.1.6. External References

   The reader should refer to the following references for analysis of
   the TLS handshake: [DFGS15], [CHSV16], [DFGS16], [KW16], [Kraw16],
   [FGSW16], [LXZFH16], [FG17], and [BBK17].

### F.2. Record Layer

   The record layer depends on the handshake producing strong traffic
   secrets which can be used to derive bidirectional encryption keys and
   nonces.  Assuming that is true, and the keys are used for no more
   data than indicated in Section 5.5, then the record layer should
   provide the following guarantees:

   Confidentiality:  An attacker should not be able to determine the
      plaintext contents of a given record.

   Integrity:  An attacker should not be able to craft a new record
      which is different from an existing record which will be accepted
      by the receiver.

   Order protection/non-replayability:  An attacker should not be able
      to cause the receiver to accept a record which it has already
      accepted or cause the receiver to accept record N+1 without having
      first processed record N.

   Length concealment:  Given a record with a given external length, the
      attacker should not be able to determine the amount of the record
      that is content versus padding.

   Forward secrecy after key change:  If the traffic key update
      mechanism described in Section 4.6.3 has been used and the
      previous generation key is deleted, an attacker who compromises
      the endpoint should not be able to decrypt traffic encrypted with
      the old key. 
   Informally, TLS 1.3 provides these properties by AEAD-protecting the
   plaintext with a strong key.  AEAD encryption [RFC5116] provides
   confidentiality and integrity for the data.  Non-replayability is
   provided by using a separate nonce for each record, with the nonce
   being derived from the record sequence number (Section 5.3), with the
   sequence number being maintained independently at both sides; thus
   records which are delivered out of order result in AEAD deprotection
   failures.  In order to prevent mass cryptanalysis when the same
   plaintext is repeatedly encrypted by different users under the same
   key (as is commonly the case for HTTP), the nonce is formed by mixing
   the sequence number with a secret per-connection initialization
   vector derived along with the traffic keys.  See [BT16] for analysis
   of this construction.

   The rekeying technique in TLS 1.3 (see Section 7.2) follows the
   construction of the serial generator as discussed in [REKEY ], which
   shows that rekeying can allow keys to be used for a larger number of
   encryptions than without rekeying.  This relies on the security of
   the HKDF-Expand-Label function as a pseudorandom function (PRF).  In
   addition, as long as this function is truly one way, it is not
   possible to compute traffic keys from prior to a key change (forward
   secrecy).

   TLS does not provide security for data which is communicated on a
   connection after a traffic secret of that connection is compromised.
   That is, TLS does not provide post-compromise security/future
   secrecy/backward secrecy with respect to the traffic secret.  Indeed,
   an attacker who learns a traffic secret can compute all future
   traffic secrets on that connection.  Systems which want such
   guarantees need to do a fresh handshake and establish a new
   connection with an (EC)DHE exchange.

#### F.2.1. External References

   The reader should refer to the following references for analysis of
   the TLS record layer: [BMMRT15], [BT16], [BDFKPPRSZZ16], [BBK17], and
   [PS18].

### F.3. Traffic Analysis

   TLS is susceptible to a variety of traffic analysis attacks based on
   observing the length and timing of encrypted packets [CLINIC ]
   [HCJC16].  This is particularly easy when there is a small set of
   possible messages to be distinguished, such as for a video server
   hosting a fixed corpus of content, but still provides usable
   information even in more complicated scenarios. 
   TLS does not provide any specific defenses against this form of
   attack but does include a padding mechanism for use by applications:
   The plaintext protected by the AEAD function consists of content plus
   variable-length padding, which allows the application to produce
   arbitrary-length encrypted records as well as padding-only cover
   traffic to conceal the difference between periods of transmission and
   periods of silence.  Because the padding is encrypted alongside the
   actual content, an attacker cannot directly determine the length of
   the padding, but may be able to measure it indirectly by the use of
   timing channels exposed during record processing (i.e., seeing how
   long it takes to process a record or trickling in records to see
   which ones elicit a response from the server).  In general, it is not
   known how to remove all of these channels because even a constant-
   time padding removal function will likely feed the content into data-
   dependent functions.  At minimum, a fully constant-time server or
   client would require close cooperation with the application-layer
   protocol implementation, including making that higher-level protocol
   constant time.

   Note: Robust traffic analysis defenses will likely lead to inferior
   performance due to delays in transmitting packets and increased
   traffic volume.

### F.4. Side Channel Attacks

   In general, TLS does not have specific defenses against side-channel
   attacks (i.e., those which attack the communications via secondary
   channels such as timing), leaving those to the implementation of the
   relevant cryptographic primitives.  However, certain features of TLS
   are designed to make it easier to write side-channel resistant code:

   *  Unlike previous versions of TLS which used a composite MAC-then-
      encrypt structure, TLS 1.3 only uses AEAD algorithms, allowing
      implementations to use self-contained constant-time
      implementations of those primitives.

   *  TLS uses a uniform "bad_record_mac" alert for all decryption
      errors, which is intended to prevent an attacker from gaining
      piecewise insight into portions of the message.  Additional
      resistance is provided by terminating the connection on such
      errors; a new connection will have different cryptographic
      material, preventing attacks against the cryptographic primitives
      that require multiple trials. 
   Information leakage through side channels can occur at layers above
   TLS, in application protocols and the applications that use them.
   Resistance to side-channel attacks depends on applications and
   application protocols separately ensuring that confidential
   information is not inadvertently leaked.

### F.5. Replay Attacks on 0-RTT

   Replayable 0-RTT data presents a number of security threats to TLS-
   using applications, unless those applications are specifically
   engineered to be safe under replay (minimally, this means idempotent,
   but in many cases may also require other stronger conditions, such as
   constant-time response).  Potential attacks include:

   *  Duplication of actions which cause side effects (e.g., purchasing
      an item or transferring money) to be duplicated, thus harming the
      site or the user.

   *  Attackers can store and replay 0-RTT messages to reorder them with
      respect to other messages (e.g., moving a delete to after a
      create).

   *  Amplifying existing information leaks caused by side effects like
      caching.  An attacker could learn information about the content of
      a 0-RTT message by replaying it to some cache node that has not
      cached some resource of interest, and then using a separate
      connection to check whether that resource has been added to the
      cache.  This could be repeated with different cache nodes as often
      as the 0-RTT message is replayable.

   If data can be replayed a large number of times, additional attacks
   become possible, such as making repeated measurements of the speed of
   cryptographic operations.  In addition, they may be able to overload
   rate-limiting systems.  For a further description of these attacks,
   see [Mac17].

   Ultimately, servers have the responsibility to protect themselves
   against attacks employing 0-RTT data replication.  The mechanisms
   described in Section 8 are intended to prevent replay at the TLS
   layer but do not provide complete protection against receiving
   multiple copies of client data.  TLS 1.3 falls back to the 1-RTT
   handshake when the server does not have any information about the
   client, e.g., because it is in a different cluster which does not
   share state or because the ticket has been deleted as described in Section 8.1.  If the application-layer protocol retransmits data in
   this setting, then it is possible for an attacker to induce message
   duplication by sending the ClientHello to both the original cluster
   (which processes the data immediately) and another cluster which will 
   fall back to 1-RTT and process the data upon application-layer
   replay.  The scale of this attack is limited by the client's
   willingness to retry transactions and therefore only allows a limited
   amount of duplication, with each copy appearing as a new connection
   at the server.

   If implemented correctly, the mechanisms described in Section 8.1 and Section 8.2 prevent a replayed ClientHello and its associated 0-RTT
   data from being accepted multiple times by any cluster with
   consistent state; for servers which limit the use of 0-RTT to one
   cluster for a single ticket, then a given ClientHello and its
   associated 0-RTT data will only be accepted once.  However, if state
   is not completely consistent, then an attacker might be able to have
   multiple copies of the data be accepted during the replication
   window.  Because clients do not know the exact details of server
   behavior, they MUST NOT send messages in early data which are not
   safe to have replayed and which they would not be willing to retry
   across multiple 1-RTT connections.

   Application protocols MUST NOT use 0-RTT data without a profile that
   defines its use.  That profile needs to identify which messages or
   interactions are safe to use with 0-RTT and how to handle the
   situation when the server rejects 0-RTT and falls back to 1-RTT.

   In addition, to avoid accidental misuse, TLS implementations MUST NOT
   enable 0-RTT (either sending or accepting) unless specifically
   requested by the application and MUST NOT automatically resend 0-RTT
   data if it is rejected by the server unless instructed by the
   application.  Server-side applications may wish to implement special
   processing for 0-RTT data for some kinds of application traffic
   (e.g., abort the connection, request that data be resent at the
   application layer, or delay processing until the handshake
   completes).  In order to allow applications to implement this kind of
   processing, TLS implementations MUST provide a way for the
   application to determine if the handshake has completed.

#### F.5.1. Replay and Exporters

   Replays of the ClientHello produce the same early exporter, thus
   requiring additional care by applications which use these exporters.
   In particular, if these exporters are used as an authentication
   channel binding (e.g., by signing the output of the exporter) an
   attacker who compromises the PSK can transplant authenticators
   between connections without compromising the authentication key. 
   In addition, the early exporter SHOULD NOT be used to generate
   server-to-client encryption keys because that would entail the reuse
   of those keys.  This parallels the use of the early application
   traffic keys only in the client-to-server direction.

### F.6. PSK Identity Exposure

   Because implementations respond to an invalid PSK binder by aborting
   the handshake, it may be possible for an attacker to verify whether a
   given PSK identity is valid.  Specifically, if a server accepts both
   external-PSK and certificate-based handshakes, a valid PSK identity
   will result in a failed handshake, whereas an invalid identity will
   just be skipped and result in a successful certificate handshake.
   Servers which solely support PSK handshakes may be able to resist
   this form of attack by treating the cases where there is no valid PSK
   identity and where there is an identity but it has an invalid binder
   identically.

### F.7. Sharing PSKs Across Protocol Versions

   TLS 1.3 takes a conservative approach to PSKs by binding them to a
   specific KDF.  By contrast, TLS 1.2 allows PSKs to be used with any
   hash function and the TLS 1.2 PRF.  Thus, any PSK which is used with
   both TLS 1.2 and TLS 1.3 must be used with only one hash in TLS 1.3,
   which is less than optimal if users want to provision a single PSK.
   The constructions in TLS 1.2 and TLS 1.3 are different, although they
   are both based on HMAC.  While there is no known way in which the
   same PSK might produce related output in both versions, only limited
   analysis has been done.  Implementations can ensure safety from
   cross-protocol related output by not reusing PSKs between TLS 1.3 and
   TLS 1.2.

### F.8. External PSKs and Rerouting

   External PSKs in TLS are designed to be known to exactly one client
   and one server.  However, as noted in [RFC9257], there are use cases
   where PSKs are shared between more than two entities.  In such
   scenarios, in addition to the expected security weakness where a
   compromised group member can impersonate any other member, a
   malicious non-member can reroute handshakes between honest group
   members to connect them in unintended ways [Selfie ].  [RFC9257]
   provides recommendations for external PSK usage, including the use of
   external PSK importers as defined in [RFC9258], that prevent such
   malicious rerouting of messages 





### F.9. Misbinding when using Self-Signed Certificates or Raw Public Keys

   When TLS 1.3 is used with self-signed certificates without useful
   identities (as in DTLS-SRTP [RFC5763]) or raw public keys [RFC7250]
   for peer authentication, it may be vulnerable to misbinding attacks
   [MM24].  This risk can be mitigated by using the "external_id_hash"
   extension [RFC8844] or, if only the server is being authenticated, by
   the server verifying that the "server_name" extension matches its
   expected identity.

### F.10. Attacks on Static RSA

   Although TLS 1.3 does not use RSA key transport and so is not
   directly susceptible to Bleichenbacher-type attacks [Blei98] if TLS
   1.3 servers also support static RSA in the context of previous
   versions of TLS, then it may be possible to impersonate the server
   for TLS 1.3 connections [JSS15].  TLS 1.3 implementations can prevent
   this attack by disabling support for static RSA across all versions
   of TLS.  In principle, implementations might also be able to separate
   certificates with different keyUsage bits for static RSA decryption
   and RSA signature, but this technique relies on clients refusing to
   accept signatures using keys in certificates that do not have the
   digitalSignature bit set, and many clients do not enforce this
   restriction.

## G. Change Log

   [[RFC EDITOR: Please remove in final RFC.]] Since -06 - Updated text
   about differences from RFC 8446. - Clarify which parts of IANA
   considerations are new to this document. - Upgrade the requirement to
   initiate key update before exceeding key usage limits to MUST. - Add
   some text around use of the same cert for client and server.

   Since -05

   *  Port in text on key update limits from RFC 9147 (Issue 1257)

   *  Clarify that you need to ignore NST if you don't do resumption
      (Issue 1280)

   *  Discuss the privacy implications of external key reuse (Issue
      1287)

   *  Advice on key deletion (PR 1282)

   *  Clarify what unsolicited extensions means (PR 1275)

   *  close_notify should be warning (PR 1290)
   *  Reference RFC 8773 (PR 1296)

   *  Add some more information about application bindings and cite RFC 
      9525 (PR 1297)

   Since -04

   *  Update the extension table (Issue 1241)

   *  Clarify user_canceled (Issue 1208)

   *  Clarify 0-RTT cache side channels (Issue 1225)

   *  Require that message reinjection be done with the current hash.
      Potentially a clarification and potentially a wire format change
      depending on previous interpretation (Issue 1227)

   Changelog not updated between -00 and -03

   Since -00

   *  Update TLS 1.2 terminology

   *  Specify "certificate-based" client authentication

   *  Clarify that privacy guarantees don't apply when you have null
      encryption

   *  Shorten some names

   *  Address tracking implications of resumption



---
# Referenced Sections from RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   The primary goal of TLS is to provide a secure channel between two
   communicating peers; the only requirement from the underlying
   transport is a reliable, in-order data stream.  Specifically, the
   secure channel should provide the following properties:

   -  Authentication: The server side of the channel is always
      authenticated; the client side is optionally authenticated.
      Authentication can happen via asymmetric cryptography (e.g., RSA
      [RSA ], the Elliptic Curve Digital Signature Algorithm (ECDSA)
      [ECDSA ], or the Edwards-Curve Digital Signature Algorithm (EdDSA)
      [RFC8032]) or a symmetric pre-shared key (PSK).

   -  Confidentiality: Data sent over the channel after establishment is
      only visible to the endpoints.  TLS does not hide the length of
      the data it transmits, though endpoints are able to pad TLS
      records in order to obscure lengths and improve protection against
      traffic analysis techniques.

   -  Integrity: Data sent over the channel after establishment cannot
      be modified by attackers without detection.

   These properties should be true even in the face of an attacker who
   has complete control of the network, as described in [RFC3552].  See Appendix E  for a more complete statement of the relevant security
   properties.

   TLS consists of two primary components:

   -  A handshake protocol (Section 4) that authenticates the
      communicating parties, negotiates cryptographic modes and
      parameters, and establishes shared keying material.  The handshake
      protocol is designed to resist tampering; an active attacker
      should not be able to force the peers to negotiate different
      parameters than they would if the connection were not under
      attack.

   -  A record protocol (Section 5) that uses the parameters established
      by the handshake protocol to protect traffic between the
      communicating peers.  The record protocol divides traffic up into
      a series of records, each of which is independently protected
      using the traffic keys. 
   TLS is application protocol independent; higher-level protocols can
   layer on top of TLS transparently.  The TLS standard, however, does
   not specify how protocols add security with TLS; how to initiate TLS
   handshaking and how to interpret the authentication certificates
   exchanged are left to the judgment of the designers and implementors
   of protocols that run on top of TLS.

   This document defines TLS version 1.3.  While TLS 1.3 is not directly
   compatible with previous versions, all versions of TLS incorporate a
   versioning mechanism which allows clients and servers to
   interoperably negotiate a common version if one is supported by both
   peers.

   This document supersedes and obsoletes previous versions of TLS,
   including version 1.2 [RFC5246].  It also obsoletes the TLS ticket
   mechanism defined in [RFC5077] and replaces it with the mechanism
   defined in Section 2.2.  Because TLS 1.3 changes the way keys are
   derived, it updates [RFC5705] as described in Section 7.5.  It also
   changes how Online Certificate Status Protocol (OCSP) messages are
   carried and therefore updates [RFC6066] and obsoletes [RFC6961] as
   described in Section 4.4.2.1.

### 1.1. Conventions and Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

   The following terms are used:

   client:  The endpoint initiating the TLS connection.

   connection:  A transport-layer connection between two endpoints.

   endpoint:  Either the client or server of the connection.

   handshake:  An initial negotiation between client and server that
      establishes the parameters of their subsequent interactions
      within TLS.

   peer:  An endpoint.  When discussing a particular endpoint, "peer"
      refers to the endpoint that is not the primary subject of
      discussion. 
   receiver:  An endpoint that is receiving records.

   sender:  An endpoint that is transmitting records.

   server:  The endpoint that did not initiate the TLS connection.

### 1.2. Major Differences from TLS 1.2

   The following is a list of the major functional differences between
   TLS 1.2 and TLS 1.3.  It is not intended to be exhaustive, and there
   are many minor differences.

   -  The list of supported symmetric encryption algorithms has been
      pruned of all algorithms that are considered legacy.  Those that
      remain are all Authenticated Encryption with Associated Data
      (AEAD) algorithms.  The cipher suite concept has been changed to
      separate the authentication and key exchange mechanisms from the
      record protection algorithm (including secret key length) and a
      hash to be used with both the key derivation function and
      handshake message authentication code (MAC).

   -  A zero round-trip time (0-RTT) mode was added, saving a round trip
      at connection setup for some application data, at the cost of
      certain security properties.

   -  Static RSA and Diffie-Hellman cipher suites have been removed; all
      public-key based key exchange mechanisms now provide forward
      secrecy.

   -  All handshake messages after the ServerHello are now encrypted.
      The newly introduced EncryptedExtensions message allows various
      extensions previously sent in the clear in the ServerHello to also
      enjoy confidentiality protection.

   -  The key derivation functions have been redesigned.  The new design
      allows easier analysis by cryptographers due to their improved key
      separation properties.  The HMAC-based Extract-and-Expand Key
      Derivation Function (HKDF) is used as an underlying primitive.

   -  The handshake state machine has been significantly restructured to
      be more consistent and to remove superfluous messages such as
      ChangeCipherSpec (except when needed for middlebox compatibility).

   -  Elliptic curve algorithms are now in the base spec, and new
      signature algorithms, such as EdDSA, are included.  TLS 1.3
      removed point format negotiation in favor of a single point format
      for each curve. 
   -  Other cryptographic improvements were made, including changing the
      RSA padding to use the RSA Probabilistic Signature Scheme
      (RSASSA-PSS), and the removal of compression, the Digital
      Signature Algorithm (DSA), and custom Ephemeral Diffie-Hellman
      (DHE) groups.

   -  The TLS 1.2 version negotiation mechanism has been deprecated in
      favor of a version list in an extension.  This increases
      compatibility with existing servers that incorrectly implemented
      version negotiation.

   -  Session resumption with and without server-side state as well as
      the PSK-based cipher suites of earlier TLS versions have been
      replaced by a single new PSK exchange.

   -  References have been updated to point to the updated versions of
      RFCs, as appropriate (e.g., RFC 5280 rather than RFC 3280).

### 1.3. Updates Affecting TLS 1.2

   This document defines several changes that optionally affect
   implementations of TLS 1.2, including those which do not also support
   TLS 1.3:

   -  A version downgrade protection mechanism is described in Section 4.1.3.

   -  RSASSA-PSS signature schemes are defined in Section 4.2.3.

   -  The "supported_versions" ClientHello extension can be used to
      negotiate the version of TLS to use, in preference to the
      legacy_version field of the ClientHello.

   -  The "signature_algorithms_cert" extension allows a client to
      indicate which signature algorithms it can validate in X.509
      certificates.

   Additionally, this document clarifies some compliance requirements
   for earlier versions of TLS; see Section 9.3. 





## 2. Protocol Overview

   The cryptographic parameters used by the secure channel are produced
   by the TLS handshake protocol.  This sub-protocol of TLS is used by
   the client and server when first communicating with each other.  The
   handshake protocol allows peers to negotiate a protocol version,
   select cryptographic algorithms, optionally authenticate each other,
   and establish shared secret keying material.  Once the handshake is
   complete, the peers use the established keys to protect the
   application-layer traffic.

   A failure of the handshake or other protocol error triggers the
   termination of the connection, optionally preceded by an alert
   message (Section 6).

   TLS supports three basic key exchange modes:

   -  (EC)DHE (Diffie-Hellman over either finite fields or elliptic
      curves)

   -  PSK-only

   -  PSK with (EC)DHE 
   Figure 1 below shows the basic full TLS handshake:

       Client                                           Server

Key  ^ ClientHello
Exch | + key_share*
     | + signature_algorithms*
     | + psk_key_exchange_modes*
     v + pre_shared_key*       -------->
                                                  ServerHello  ^ Key
                                                 + key_share*  | Exch
                                            + pre_shared_key*  v
                                        {EncryptedExtensions}  ^  Server
                                        {CertificateRequest*}  v  Params
                                               {Certificate*}  ^
                                         {CertificateVerify*}  | Auth
                                                   {Finished}  v
                               <--------  [Application Data*]
     ^ {Certificate*}
Auth | {CertificateVerify*}
     v {Finished}              -------->
       [Application Data]      <------->  [Application Data]

              +  Indicates noteworthy extensions sent in the
                 previously noted message.

              *  Indicates optional or situation-dependent
                 messages/extensions that are not always sent.

              {} Indicates messages protected using keys
                 derived from a [sender ]_handshake_traffic_secret.

              [] Indicates messages protected using keys
                 derived from [sender ]_application_traffic_secret_N.

               Figure 1: Message Flow for Full TLS Handshake

   The handshake can be thought of as having three phases (indicated in
   the diagram above):

   -  Key Exchange: Establish shared keying material and select the
      cryptographic parameters.  Everything after this phase is
      encrypted.

   -  Server Parameters: Establish other handshake parameters
      (whether the client is authenticated, application-layer protocol
      support, etc.). 
   -  Authentication: Authenticate the server (and, optionally, the
      client) and provide key confirmation and handshake integrity.

   In the Key Exchange phase, the client sends the ClientHello
   (Section 4.1.2) message, which contains a random nonce
   (ClientHello.random); its offered protocol versions; a list of
   symmetric cipher/HKDF hash pairs; either a set of Diffie-Hellman key
   shares (in the "key_share" (Section 4.2.8) extension), a set of
   pre-shared key labels (in the "pre_shared_key" (Section 4.2.11)
   extension), or both; and potentially additional extensions.
   Additional fields and/or messages may also be present for middlebox
   compatibility.

   The server processes the ClientHello and determines the appropriate
   cryptographic parameters for the connection.  It then responds with
   its own ServerHello (Section 4.1.3), which indicates the negotiated
   connection parameters.  The combination of the ClientHello and the
   ServerHello determines the shared keys.  If (EC)DHE key establishment
   is in use, then the ServerHello contains a "key_share" extension with
   the server's ephemeral Diffie-Hellman share; the server's share MUST
   be in the same group as one of the client's shares.  If PSK key
   establishment is in use, then the ServerHello contains a
   "pre_shared_key" extension indicating which of the client's offered
   PSKs was selected.  Note that implementations can use (EC)DHE and PSK
   together, in which case both extensions will be supplied.

   The server then sends two messages to establish the Server
   Parameters:

   EncryptedExtensions:  responses to ClientHello extensions that are
      not required to determine the cryptographic parameters, other than
      those that are specific to individual certificates.
      [Section 4.3.1]

   CertificateRequest:  if certificate-based client authentication is
      desired, the desired parameters for that certificate.  This
      message is omitted if client authentication is not desired.
      [Section 4.3.2]
   Finally, the client and server exchange Authentication messages.  TLS
   uses the same set of messages every time that certificate-based
   authentication is needed.  (PSK-based authentication happens as a
   side effect of key exchange.)  Specifically:

   Certificate:  The certificate of the endpoint and any per-certificate
      extensions.  This message is omitted by the server if not
      authenticating with a certificate and by the client if the server
      did not send CertificateRequest (thus indicating that the client
      should not authenticate with a certificate).  Note that if raw
      public keys [RFC7250] or the cached information extension
      [RFC7924] are in use, then this message will not contain a
      certificate but rather some other value corresponding to the
      server's long-term key.  [Section 4.4.2]

   CertificateVerify:  A signature over the entire handshake using the
      private key corresponding to the public key in the Certificate
      message.  This message is omitted if the endpoint is not
      authenticating via a certificate.  [Section 4.4.3]

   Finished:  A MAC (Message Authentication Code) over the entire
      handshake.  This message provides key confirmation, binds the
      endpoint's identity to the exchanged keys, and in PSK mode also
      authenticates the handshake.  [Section 4.4.4]

   Upon receiving the server's messages, the client responds with its
   Authentication messages, namely Certificate and CertificateVerify (if
   requested), and Finished.

   At this point, the handshake is complete, and the client and server
   derive the keying material required by the record layer to exchange
   application-layer data protected through authenticated encryption.
   Application Data MUST NOT be sent prior to sending the Finished
   message, except as specified in Section 2.3.  Note that while the
   server may send Application Data prior to receiving the client's
   Authentication messages, any data sent at that point is, of course,
   being sent to an unauthenticated peer. 





### 2.1. Incorrect DHE Share

   If the client has not provided a sufficient "key_share" extension
   (e.g., it includes only DHE or ECDHE groups unacceptable to or
   unsupported by the server), the server corrects the mismatch with a
   HelloRetryRequest and the client needs to restart the handshake with
   an appropriate "key_share" extension, as shown in Figure 2.  If no
   common cryptographic parameters can be negotiated, the server MUST
   abort the handshake with an appropriate alert.

        Client                                               Server

        ClientHello
        + key_share             -------->
                                                  HelloRetryRequest
                                <--------               + key_share
        ClientHello
        + key_share             -------->
                                                        ServerHello
                                                        + key_share
                                              {EncryptedExtensions}
                                              {CertificateRequest*}
                                                     {Certificate*}
                                               {CertificateVerify*}
                                                         {Finished}
                                <--------       [Application Data*]
        {Certificate*}
        {CertificateVerify*}
        {Finished}              -------->
        [Application Data]      <------->        [Application Data]

             Figure 2: Message Flow for a Full Handshake with
                           Mismatched Parameters

   Note: The handshake transcript incorporates the initial
   ClientHello/HelloRetryRequest exchange; it is not reset with the
   new ClientHello.

   TLS also allows several optimized variants of the basic handshake, as
   described in the following sections. 





### 2.2. Resumption and Pre-Shared Key (PSK)

   Although TLS PSKs can be established out of band, PSKs can also be
   established in a previous connection and then used to establish a new
   connection ("session resumption" or "resuming" with a PSK).  Once a
   handshake has completed, the server can send the client a PSK
   identity that corresponds to a unique key derived from the initial
   handshake (see Section 4.6.1).  The client can then use that PSK
   identity in future handshakes to negotiate the use of the associated
   PSK.  If the server accepts the PSK, then the security context of the
   new connection is cryptographically tied to the original connection
   and the key derived from the initial handshake is used to bootstrap
   the cryptographic state instead of a full handshake.  In TLS 1.2 and
   below, this functionality was provided by "session IDs" and "session
   tickets" [RFC5077].  Both mechanisms are obsoleted in TLS 1.3.

   PSKs can be used with (EC)DHE key exchange in order to provide
   forward secrecy in combination with shared keys, or can be used
   alone, at the cost of losing forward secrecy for the application
   data. 
   Figure 3 shows a pair of handshakes in which the first handshake
   establishes a PSK and the second handshake uses it:

          Client                                               Server

   Initial Handshake:
          ClientHello
          + key_share               -------->
                                                          ServerHello
                                                          + key_share
                                                {EncryptedExtensions}
                                                {CertificateRequest*}
                                                       {Certificate*}
                                                 {CertificateVerify*}
                                                           {Finished}
                                    <--------     [Application Data*]
          {Certificate*}
          {CertificateVerify*}
          {Finished}                -------->
                                    <--------      [NewSessionTicket]
          [Application Data]        <------->      [Application Data]


   Subsequent Handshake:
          ClientHello
          + key_share*
          + pre_shared_key          -------->
                                                          ServerHello
                                                     + pre_shared_key
                                                         + key_share*
                                                {EncryptedExtensions}
                                                           {Finished}
                                    <--------     [Application Data*]
          {Finished}                -------->
          [Application Data]        <------->      [Application Data]

               Figure 3: Message Flow for Resumption and PSK

   As the server is authenticating via a PSK, it does not send a
   Certificate or a CertificateVerify message.  When a client offers
   resumption via a PSK, it SHOULD also supply a "key_share" extension
   to the server to allow the server to decline resumption and fall back
   to a full handshake, if needed.  The server responds with a
   "pre_shared_key" extension to negotiate the use of PSK key
   establishment and can (as shown here) respond with a "key_share"
   extension to do (EC)DHE key establishment, thus providing forward
   secrecy. 
   When PSKs are provisioned out of band, the PSK identity and the KDF
   hash algorithm to be used with the PSK MUST also be provisioned.

   Note:  When using an out-of-band provisioned pre-shared secret, a
      critical consideration is using sufficient entropy during the key
      generation, as discussed in [RFC4086].  Deriving a shared secret
      from a password or other low-entropy sources is not secure.  A
      low-entropy secret, or password, is subject to dictionary attacks
      based on the PSK binder.  The specified PSK authentication is not
      a strong password-based authenticated key exchange even when used
      with Diffie-Hellman key establishment.  Specifically, it does not
      prevent an attacker that can observe the handshake from performing
      a brute-force attack on the password/pre-shared key.

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

## 3. Presentation Language

   This document deals with the formatting of data in an external
   representation.  The following very basic and somewhat casually
   defined presentation syntax will be used.

### 3.1. Basic Block Size

   The representation of all data items is explicitly specified.  The
   basic data block size is one byte (i.e., 8 bits).  Multiple-byte data
   items are concatenations of bytes, from left to right, from top to
   bottom.  From the byte stream, a multi-byte item (a numeric in the
   following example) is formed (using C notation) by:

      value = (byte[0] << 8*(n-1)) | (byte[1] << 8*(n-2)) |
              ... | byte[n-1];

   This byte ordering for multi-byte values is the commonplace network
   byte order or big-endian format. 





### 3.2. Miscellaneous

   Comments begin with "/*" and end with "*/".

   Optional components are denoted by enclosing them in "[[ ]]" (double
   brackets).

   Single-byte entities containing uninterpreted data are of
   type opaque.

   A type alias T' for an existing type T is defined by:

      T T';

### 3.3. Numbers

   The basic numeric data type is an unsigned byte (uint8).  All larger
   numeric data types are constructed from a fixed-length series of
   bytes concatenated as described in Section 3.1 and are also unsigned.
   The following numeric types are predefined.

      uint8 uint16[2];
      uint8 uint24[3];
      uint8 uint32[4];
      uint8 uint64[8];

   All values, here and elsewhere in the specification, are transmitted
   in network byte (big-endian) order; the uint32 represented by the hex
   bytes 01 02 03 04 is equivalent to the decimal value 16909060.

### 3.4. Vectors

   A vector (single-dimensioned array) is a stream of homogeneous data
   elements.  The size of the vector may be specified at documentation
   time or left unspecified until runtime.  In either case, the length
   declares the number of bytes, not the number of elements, in the
   vector.  The syntax for specifying a new type, T', that is a fixed-
   length vector of type T is

      T T'[n];

   Here, T' occupies n bytes in the data stream, where n is a multiple
   of the size of T.  The length of the vector is not included in the
   encoded stream. 
   In the following example, Datum is defined to be three consecutive
   bytes that the protocol does not interpret, while Data is three
   consecutive Datum, consuming a total of nine bytes.

      opaque Datum[3];      /* three uninterpreted bytes */
      Datum Data[9];        /* three consecutive 3-byte vectors */

   Variable-length vectors are defined by specifying a subrange of legal
   lengths, inclusively, using the notation <floor..ceiling>.  When
   these are encoded, the actual length precedes the vector's contents
   in the byte stream.  The length will be in the form of a number
   consuming as many bytes as required to hold the vector's specified
   maximum (ceiling) length.  A variable-length vector with an actual
   length field of zero is referred to as an empty vector.

      T T'<floor..ceiling>;

   In the following example, "mandatory" is a vector that must contain
   between 300 and 400 bytes of type opaque.  It can never be empty.
   The actual length field consumes two bytes, a uint16, which is
   sufficient to represent the value 400 (see Section 3.3).  Similarly,
   "longer" can represent up to 800 bytes of data, or 400 uint16
   elements, and it may be empty.  Its encoding will include a two-byte
   actual length field prepended to the vector.  The length of an
   encoded vector must be an exact multiple of the length of a single
   element (e.g., a 17-byte vector of uint16 would be illegal).

      opaque mandatory<300..400>;
            /* length field is two bytes, cannot be empty */
      uint16 longer<0..800>;
            /* zero to 400 16-bit unsigned integers */

### 3.5. Enumerateds

   An additional sparse data type, called "enum" or "enumerated", is
   available.  Each definition is a different type.  Only enumerateds of
   the same type may be assigned or compared.  Every element of an
   enumerated must be assigned a value, as demonstrated in the following
   example.  Since the elements of the enumerated are not ordered, they
   can be assigned any unique value, in any order.

      enum { e1(v1), e2(v2), ... , en(vn) [[, (n)]] } Te;

   Future extensions or additions to the protocol may define new values.
   Implementations need to be able to parse and ignore unknown values
   unless the definition of the field states otherwise. 
   An enumerated occupies as much space in the byte stream as would its
   maximal defined ordinal value.  The following definition would cause
   one byte to be used to carry fields of type Color.

      enum { red(3), blue(5), white(7) } Color;

   One may optionally specify a value without its associated tag to
   force the width definition without defining a superfluous element.

   In the following example, Taste will consume two bytes in the data
   stream but can only assume the values 1, 2, or 4 in the current
   version of the protocol.

      enum { sweet(1), sour(2), bitter(4), (32000) } Taste;

   The names of the elements of an enumeration are scoped within the
   defined type.  In the first example, a fully qualified reference to
   the second element of the enumeration would be Color.blue.  Such
   qualification is not required if the target of the assignment is well
   specified.

      Color color = Color.blue;     /* overspecified, legal */
      Color color = blue;           /* correct, type implicit */

   The names assigned to enumerateds do not need to be unique.  The
   numerical value can describe a range over which the same name
   applies.  The value includes the minimum and maximum inclusive values
   in that range, separated by two period characters.  This is
   principally useful for reserving regions of the space.

      enum { sad(0), meh(1..254), happy(255) } Mood;

### 3.6. Constructed Types

   Structure types may be constructed from primitive types for
   convenience.  Each specification declares a new, unique type.  The
   syntax used for definitions is much like that of C.

      struct {
          T1 f1;
          T2 f2;
          ...
          Tn fn;
      } T;

   Fixed- and variable-length vector fields are allowed using the
   standard vector syntax.  Structures V1 and V2 in the variants example
   (Section 3.8) demonstrate this. 
   The fields within a structure may be qualified using the type's name,
   with a syntax much like that available for enumerateds.  For example,
   T.f2 refers to the second field of the previous declaration.

### 3.7. Constants

   Fields and variables may be assigned a fixed value using "=", as in:

      struct {
          T1 f1 = 8;  /* T.f1 must always be 8 */
          T2 f2;
      } T;

### 3.8. Variants

   Defined structures may have variants based on some knowledge that is
   available within the environment.  The selector must be an enumerated
   type that defines the possible variants the structure defines.  Each
   arm of the select (below) specifies the type of that variant's field
   and an optional field label.  The mechanism by which the variant is
   selected at runtime is not prescribed by the presentation language.

      struct {
          T1 f1;
          T2 f2;
          ....
          Tn fn;
          select (E) {
              case e1: Te1 [[fe1]];
              case e2: Te2 [[fe2]];
              ....
              case en: Ten [[fen]];
          };
      } Tv;
   For example:

      enum { apple(0), orange(1) } VariantTag;

      struct {
          uint16 number;
          opaque string<0..10>; /* variable length */
      } V1;

      struct {
          uint32 number;
          opaque string[10];    /* fixed length */
      } V2;

      struct {
          VariantTag type;
          select (VariantRecord.type) {
              case apple:  V1;
              case orange: V2;
          };
      } VariantRecord;

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

### 6.2. Error Alerts

   Error handling in TLS is very simple.  When an error is detected, the
   detecting party sends a message to its peer.  Upon transmission or
   receipt of a fatal alert message, both parties MUST immediately close
   the connection.

   Whenever an implementation encounters a fatal error condition, it
   SHOULD send an appropriate fatal alert and MUST close the connection
   without sending or receiving any additional data.  In the rest of
   this specification, when the phrases "terminate the connection" and
   "abort the handshake" are used without a specific alert it means that
   the implementation SHOULD send the alert indicated by the
   descriptions below.  The phrases "terminate the connection with an X
   alert" and "abort the handshake with an X alert" mean that the
   implementation MUST send alert X if it sends any alert.  All alerts
   defined below in this section, as well as all unknown alerts, are
   universally considered fatal as of TLS 1.3 (see Section 6).  The
   implementation SHOULD provide a way to facilitate logging the sending
   and receiving of alerts.

   The following error alerts are defined:

   unexpected_message:  An inappropriate message (e.g., the wrong
      handshake message, premature Application Data, etc.) was received.
      This alert should never be observed in communication between
      proper implementations.

   bad_record_mac:  This alert is returned if a record is received which
      cannot be deprotected.  Because AEAD algorithms combine decryption
      and verification, and also to avoid side-channel attacks, this
      alert is used for all deprotection failures.  This alert should
      never be observed in communication between proper implementations,
      except when messages were corrupted in the network.

   record_overflow:  A TLSCiphertext record was received that had a
      length more than 2^14 + 256 bytes, or a record decrypted to a
      TLSPlaintext record with more than 2^14 bytes (or some other
      negotiated limit).  This alert should never be observed in
      communication between proper implementations, except when messages
      were corrupted in the network.

   handshake_failure:  Receipt of a "handshake_failure" alert message
      indicates that the sender was unable to negotiate an acceptable
      set of security parameters given the options available.

   bad_certificate:  A certificate was corrupt, contained signatures
      that did not verify correctly, etc. 
   unsupported_certificate:  A certificate was of an unsupported type.

   certificate_revoked:  A certificate was revoked by its signer.

   certificate_expired:  A certificate has expired or is not currently
      valid.

   certificate_unknown:  Some other (unspecified) issue arose in
      processing the certificate, rendering it unacceptable.

   illegal_parameter:  A field in the handshake was incorrect or
      inconsistent with other fields.  This alert is used for errors
      which conform to the formal protocol syntax but are otherwise
      incorrect.

   unknown_ca:  A valid certificate chain or partial chain was received,
      but the certificate was not accepted because the CA certificate
      could not be located or could not be matched with a known trust
      anchor.

   access_denied:  A valid certificate or PSK was received, but when
      access control was applied, the sender decided not to proceed with
      negotiation.

   decode_error:  A message could not be decoded because some field was
      out of the specified range or the length of the message was
      incorrect.  This alert is used for errors where the message does
      not conform to the formal protocol syntax.  This alert should
      never be observed in communication between proper implementations,
      except when messages were corrupted in the network.

   decrypt_error:  A handshake (not record layer) cryptographic
      operation failed, including being unable to correctly verify a
      signature or validate a Finished message or a PSK binder.

   protocol_version:  The protocol version the peer has attempted to
      negotiate is recognized but not supported (see Appendix D ).

   insufficient_security:  Returned instead of "handshake_failure" when
      a negotiation has failed specifically because the server requires
      parameters more secure than those supported by the client.

   internal_error:  An internal error unrelated to the peer or the
      correctness of the protocol (such as a memory allocation failure)
      makes it impossible to continue.

   inappropriate_fallback:  Sent by a server in response to an invalid
      connection retry attempt from a client (see [RFC7507]). 
   missing_extension:  Sent by endpoints that receive a handshake
      message not containing an extension that is mandatory to send for
      the offered TLS version or other negotiated parameters.

   unsupported_extension:  Sent by endpoints receiving any handshake
      message containing an extension known to be prohibited for
      inclusion in the given handshake message, or including any
      extensions in a ServerHello or Certificate not first offered in
      the corresponding ClientHello or CertificateRequest.

   unrecognized_name:  Sent by servers when no server exists identified
      by the name provided by the client via the "server_name" extension
      (see [RFC6066]).

   bad_certificate_status_response:  Sent by clients when an invalid or
      unacceptable OCSP response is provided by the server via the
      "status_request" extension (see [RFC6066]).

   unknown_psk_identity:  Sent by servers when PSK key establishment is
      desired but no acceptable PSK identity is provided by the client.
      Sending this alert is OPTIONAL; servers MAY instead choose to send
      a "decrypt_error" alert to merely indicate an invalid PSK
      identity.

   certificate_required:  Sent by servers when a client certificate is
      desired but none was provided by the client.

   no_application_protocol:  Sent by servers when a client
      "application_layer_protocol_negotiation" extension advertises only
      protocols that the server does not support (see [RFC7301]).

   New Alert values are assigned by IANA as described in Section 11.

### 7.1. Key Schedule

   The key derivation process makes use of the HKDF-Extract and
   HKDF-Expand functions as defined for HKDF [RFC5869], as well as the
   functions defined below:

       HKDF-Expand-Label(Secret, Label, Context, Length) =
            HKDF-Expand(Secret, HkdfLabel, Length)

       Where HkdfLabel is specified as:

       struct {
           uint16 length = Length;
           opaque label<7..255> = "tls13 " + Label;
           opaque context<0..255> = Context;
       } HkdfLabel;

       Derive-Secret(Secret, Label, Messages) =
            HKDF-Expand-Label(Secret, Label,
                              Transcript-Hash(Messages), Hash.length)

   The Hash function used by Transcript-Hash and HKDF is the cipher
   suite hash algorithm.  Hash.length is its output length in bytes.
   Messages is the concatenation of the indicated handshake messages,
   including the handshake message type and length fields, but not
   including record layer headers.  Note that in some cases a zero-
   length Context (indicated by "") is passed to HKDF-Expand-Label.  The
   labels specified in this document are all ASCII strings and do not
   include a trailing NUL byte.

   Note: With common hash functions, any label longer than 12 characters
   requires an additional iteration of the hash function to compute.
   The labels in this specification have all been chosen to fit within
   this limit. 
   Keys are derived from two input secrets using the HKDF-Extract and
   Derive-Secret functions.  The general pattern for adding a new secret
   is to use HKDF-Extract with the Salt being the current secret state
   and the Input Keying Material (IKM) being the new secret to be added.
   In this version of TLS 1.3, the two input secrets are:

   -  PSK (a pre-shared key established externally or derived from the
      resumption_master_secret value from a previous connection)

   -  (EC)DHE shared secret (Section 7.4)

   This produces a full key derivation schedule shown in the diagram
   below.  In this diagram, the following formatting conventions apply:

   -  HKDF-Extract is drawn as taking the Salt argument from the top and
      the IKM argument from the left, with its output to the bottom and
      the name of the output on the right.

   -  Derive-Secret's Secret argument is indicated by the incoming
      arrow.  For instance, the Early Secret is the Secret for
      generating the client_early_traffic_secret.

   -  "0" indicates a string of Hash.length bytes set to zero. 
             0
             |
             v
   PSK ->  HKDF-Extract = Early Secret
             |
             +-----> Derive-Secret(., "ext binder" | "res binder", "")
             |                     = binder_key
             |
             +-----> Derive-Secret(., "c e traffic", ClientHello)
             |                     = client_early_traffic_secret
             |
             +-----> Derive-Secret(., "e exp master", ClientHello)
             |                     = early_exporter_master_secret
             v
       Derive-Secret(., "derived", "")
             |
             v
   (EC)DHE -> HKDF-Extract = Handshake Secret
             |
             +-----> Derive-Secret(., "c hs traffic",
             |                     ClientHello...ServerHello)
             |                     = client_handshake_traffic_secret
             |
             +-----> Derive-Secret(., "s hs traffic",
             |                     ClientHello...ServerHello)
             |                     = server_handshake_traffic_secret
             v
       Derive-Secret(., "derived", "")
             |
             v
   0 -> HKDF-Extract = Master Secret
             |
             +-----> Derive-Secret(., "c ap traffic",
             |                     ClientHello...server Finished)
             |                     = client_application_traffic_secret_0
             |
             +-----> Derive-Secret(., "s ap traffic",
             |                     ClientHello...server Finished)
             |                     = server_application_traffic_secret_0
             |
             +-----> Derive-Secret(., "exp master",
             |                     ClientHello...server Finished)
             |                     = exporter_master_secret
             |
             +-----> Derive-Secret(., "res master",
                                   ClientHello...client Finished)
                                   = resumption_master_secret 
   The general pattern here is that the secrets shown down the left side
   of the diagram are just raw entropy without context, whereas the
   secrets down the right side include Handshake Context and therefore
   can be used to derive working keys without additional context.  Note
   that the different calls to Derive-Secret may take different Messages
   arguments, even with the same secret.  In a 0-RTT exchange,
   Derive-Secret is called with four distinct transcripts; in a
   1-RTT-only exchange, it is called with three distinct transcripts.

   If a given secret is not available, then the 0-value consisting of a
   string of Hash.length bytes set to zeros is used.  Note that this
   does not mean skipping rounds, so if PSK is not in use, Early Secret
   will still be HKDF-Extract(0, 0).  For the computation of the
   binder_key, the label is "ext binder" for external PSKs (those
   provisioned outside of TLS) and "res binder" for resumption PSKs
   (those provisioned as the resumption master secret of a previous
   handshake).  The different labels prevent the substitution of one
   type of PSK for the other.

   There are multiple potential Early Secret values, depending on which
   PSK the server ultimately selects.  The client will need to compute
   one for each potential PSK; if no PSK is selected, it will then need
   to compute the Early Secret corresponding to the zero PSK.

   Once all the values which are to be derived from a given secret have
   been computed, that secret SHOULD be erased.

# Referenced Sections from RFC 5246: The Transport Layer Security (TLS) Protocol Version 1.2

The following sections were referenced. Remaining sections are not included.

### 1.2. Major Differences from TLS 1.1

   This document is a revision of the TLS 1.1 [TLS1.1] protocol which
   contains improved flexibility, particularly for negotiation of
   cryptographic algorithms.  The major changes are:

   -  The MD5/SHA-1 combination in the pseudorandom function (PRF) has
      been replaced with cipher-suite-specified PRFs.  All cipher suites
      in this document use P_SHA256.

   -  The MD5/SHA-1 combination in the digitally-signed element has been
      replaced with a single hash.  Signed elements now include a field
      that explicitly specifies the hash algorithm used.

   -  Substantial cleanup to the client's and server's ability to
      specify which hash and signature algorithms they will accept.
      Note that this also relaxes some of the constraints on signature
      and hash algorithms from previous versions of TLS.

   -  Addition of support for authenticated encryption with additional
      data modes.

   -  TLS Extensions definition and AES Cipher Suites were merged in
      from external [TLSEXT ] and [TLSAES ].

   -  Tighter checking of EncryptedPreMasterSecret version numbers.

   -  Tightened up a number of requirements.

   -  Verify_data length now depends on the cipher suite (default is
      still 12).

   -  Cleaned up description of Bleichenbacher/Klima attack defenses. 
   -  Alerts MUST now be sent in many cases.

   -  After a certificate_request, if no certificates are available,
      clients now MUST send an empty certificate list.

   -  TLS_RSA_WITH_AES_128_CBC_SHA is now the mandatory to implement
      cipher suite.

   -  Added HMAC-SHA256 cipher suites.

   -  Removed IDEA and DES cipher suites.  They are now deprecated and
      will be documented in a separate document.

   -  Support for the SSLv2 backward-compatible hello is now a MAY, not
      a SHOULD, with sending it a SHOULD NOT.  Support will probably
      become a SHOULD NOT in the future.

   -  Added limited "fall-through" to the presentation language to allow
      multiple case arms to have the same encoding.

   -  Added an Implementation Pitfalls sections

   -  The usual clarifications and editorial work.

#### 7.2.2. Error Alerts

   Error handling in the TLS Handshake protocol is very simple.  When an
   error is detected, the detecting party sends a message to the other
   party.  Upon transmission or receipt of a fatal alert message, both
   parties immediately close the connection.  Servers and clients MUST
   forget any session-identifiers, keys, and secrets associated with a
   failed connection.  Thus, any connection terminated with a fatal
   alert MUST NOT be resumed.

   Whenever an implementation encounters a condition which is defined as
   a fatal alert, it MUST send the appropriate alert prior to closing
   the connection.  For all errors where an alert level is not
   explicitly specified, the sending party MAY determine at its
   discretion whether to treat this as a fatal error or not.  If the
   implementation chooses to send an alert but intends to close the
   connection immediately afterwards, it MUST send that alert at the
   fatal alert level.

   If an alert with a level of warning is sent and received, generally
   the connection can continue normally.  If the receiving party decides
   not to proceed with the connection (e.g., after having received a
   no_renegotiation alert that it is not willing to accept), it SHOULD
   send a fatal alert to terminate the connection.  Given this, the
   sending party cannot, in general, know how the receiving party will
   behave.  Therefore, warning alerts are not very useful when the
   sending party wants to continue the connection, and thus are
   sometimes omitted.  For example, if a peer decides to accept an
   expired certificate (perhaps after confirming this with the user) and
   wants to continue the connection, it would not generally send a
   certificate_expired alert.

   The following error alerts are defined:

   unexpected_message
      An inappropriate message was received.  This alert is always fatal
      and should never be observed in communication between proper
      implementations. 
   bad_record_mac
      This alert is returned if a record is received with an incorrect
      MAC.  This alert also MUST be returned if an alert is sent because
      a TLSCiphertext decrypted in an invalid way: either it wasn't an
      even multiple of the block length, or its padding values, when
      checked, weren't correct.  This message is always fatal and should
      never be observed in communication between proper implementations
      (except when messages were corrupted in the network).

   decryption_failed_RESERVED
      This alert was used in some earlier versions of TLS, and may have
      permitted certain attacks against the CBC mode [CBCATT ].  It MUST
      NOT be sent by compliant implementations.

   record_overflow
      A TLSCiphertext record was received that had a length more than
      2^14+2048 bytes, or a record decrypted to a TLSCompressed record
      with more than 2^14+1024 bytes.  This message is always fatal and
      should never be observed in communication between proper
      implementations (except when messages were corrupted in the
      network).

   decompression_failure
      The decompression function received improper input (e.g., data
      that would expand to excessive length).  This message is always
      fatal and should never be observed in communication between proper
      implementations.

   handshake_failure
      Reception of a handshake_failure alert message indicates that the
      sender was unable to negotiate an acceptable set of security
      parameters given the options available.  This is a fatal error.

   no_certificate_RESERVED
      This alert was used in SSLv3 but not any version of TLS.  It MUST
      NOT be sent by compliant implementations.

   bad_certificate
      A certificate was corrupt, contained signatures that did not
      verify correctly, etc.

   unsupported_certificate
      A certificate was of an unsupported type.

   certificate_revoked
      A certificate was revoked by its signer. 
   certificate_expired
      A certificate has expired or is not currently valid.

   certificate_unknown
      Some other (unspecified) issue arose in processing the
      certificate, rendering it unacceptable.

   illegal_parameter
      A field in the handshake was out of range or inconsistent with
      other fields.  This message is always fatal.

   unknown_ca
      A valid certificate chain or partial chain was received, but the
      certificate was not accepted because the CA certificate could not
      be located or couldn't be matched with a known, trusted CA.  This
      message is always fatal.

   access_denied
      A valid certificate was received, but when access control was
      applied, the sender decided not to proceed with negotiation.  This
      message is always fatal.

   decode_error
      A message could not be decoded because some field was out of the
      specified range or the length of the message was incorrect.  This
      message is always fatal and should never be observed in
      communication between proper implementations (except when messages
      were corrupted in the network).

   decrypt_error
      A handshake cryptographic operation failed, including being unable
      to correctly verify a signature or validate a Finished message.
      This message is always fatal.

   export_restriction_RESERVED
      This alert was used in some earlier versions of TLS.  It MUST NOT
      be sent by compliant implementations.

   protocol_version
      The protocol version the client has attempted to negotiate is
      recognized but not supported.  (For example, old protocol versions
      might be avoided for security reasons.)  This message is always
      fatal. 
   insufficient_security
      Returned instead of handshake_failure when a negotiation has
      failed specifically because the server requires ciphers more
      secure than those supported by the client.  This message is always
      fatal.

   internal_error
      An internal error unrelated to the peer or the correctness of the
      protocol (such as a memory allocation failure) makes it impossible
      to continue.  This message is always fatal.

   user_canceled
      This handshake is being canceled for some reason unrelated to a
      protocol failure.  If the user cancels an operation after the
      handshake is complete, just closing the connection by sending a
      close_notify is more appropriate.  This alert should be followed
      by a close_notify.  This message is generally a warning.

   no_renegotiation
      Sent by the client in response to a hello request or by the server
      in response to a client hello after initial handshaking.  Either
      of these would normally lead to renegotiation; when that is not
      appropriate, the recipient should respond with this alert.  At
      that point, the original requester can decide whether to proceed
      with the connection.  One case where this would be appropriate is
      where a server has spawned a process to satisfy a request; the
      process might receive security parameters (key length,
      authentication, etc.) at startup, and it might be difficult to
      communicate changes to these parameters after that point.  This
      message is always a warning.

   unsupported_extension
      sent by clients that receive an extended server hello containing
      an extension that they did not put in the corresponding client
      hello.  This message is always fatal.

   New Alert values are assigned by IANA as described in Section 12.

### 7.4. Handshake Protocol

   The TLS Handshake Protocol is one of the defined higher-level clients
   of the TLS Record Protocol.  This protocol is used to negotiate the
   secure attributes of a session.  Handshake messages are supplied to
   the TLS record layer, where they are encapsulated within one or more
   TLSPlaintext structures, which are processed and transmitted as
   specified by the current active session state.

      enum {
          hello_request(0), client_hello(1), server_hello(2),
          certificate(11), server_key_exchange (12),
          certificate_request(13), server_hello_done(14),
          certificate_verify(15), client_key_exchange(16),
          finished(20), (255)
      } HandshakeType;

      struct {
          HandshakeType msg_type;    /* handshake type */
          uint24 length;             /* bytes in message */
          select (HandshakeType) {
              case hello_request:       HelloRequest;
              case client_hello:        ClientHello;
              case server_hello:        ServerHello;
              case certificate:         Certificate;
              case server_key_exchange: ServerKeyExchange;
              case certificate_request: CertificateRequest;
              case server_hello_done:   ServerHelloDone;
              case certificate_verify:  CertificateVerify;
              case client_key_exchange: ClientKeyExchange;
              case finished:            Finished;
          } body;
      } Handshake;
   The handshake protocol messages are presented below in the order they
   MUST be sent; sending handshake messages in an unexpected order
   results in a fatal error.  Unneeded handshake messages can be
   omitted, however.  Note one exception to the ordering: the
   Certificate message is used twice in the handshake (from server to
   client, then from client to server), but described only in its first
   position.  The one message that is not bound by these ordering rules
   is the HelloRequest message, which can be sent at any time, but which
   SHOULD be ignored by the client if it arrives in the middle of a
   handshake.

   New handshake message types are assigned by IANA as described in Section 12.

#### 7.4.1. Hello Messages

   The hello phase messages are used to exchange security enhancement
   capabilities between the client and server.  When a new session
   begins, the record layer's connection state encryption, hash, and
   compression algorithms are initialized to null.  The current
   connection state is used for renegotiation messages.

##### 7.4.1.1. Hello Request

   When this message will be sent:

      The HelloRequest message MAY be sent by the server at any time.

   Meaning of this message:

      HelloRequest is a simple notification that the client should begin
      the negotiation process anew.  In response, the client should send
      a ClientHello message when convenient.  This message is not
      intended to establish which side is the client or server but
      merely to initiate a new negotiation.  Servers SHOULD NOT send a
      HelloRequest immediately upon the client's initial connection.  It
      is the client's job to send a ClientHello at that time.

      This message will be ignored by the client if the client is
      currently negotiating a session.  This message MAY be ignored by
      the client if it does not wish to renegotiate a session, or the
      client may, if it wishes, respond with a no_renegotiation alert.
      Since handshake messages are intended to have transmission
      precedence over application data, it is expected that the
      negotiation will begin before no more than a few records are
      received from the client.  If the server sends a HelloRequest but
      does not receive a ClientHello in response, it may close the
      connection with a fatal alert. 
      After sending a HelloRequest, servers SHOULD NOT repeat the
      request until the subsequent handshake negotiation is complete.

   Structure of this message:

      struct { } HelloRequest;

   This message MUST NOT be included in the message hashes that are
   maintained throughout the handshake and used in the Finished messages
   and the certificate verify message.

##### 7.4.1.2. Client Hello

   When this message will be sent:

      When a client first connects to a server, it is required to send
      the ClientHello as its first message.  The client can also send a
      ClientHello in response to a HelloRequest or on its own initiative
      in order to renegotiate the security parameters in an existing
      connection.

   Structure of this message:

      The ClientHello message includes a random structure, which is used
      later in the protocol.

         struct {
             uint32 gmt_unix_time;
             opaque random_bytes[28];
         } Random;

      gmt_unix_time
         The current time and date in standard UNIX 32-bit format
         (seconds since the midnight starting Jan 1, 1970, UTC, ignoring
         leap seconds) according to the sender's internal clock.  Clocks
         are not required to be set correctly by the basic TLS protocol;
         higher-level or application protocols may define additional
         requirements.  Note that, for historical reasons, the data
         element is named using GMT, the predecessor of the current
         worldwide time base, UTC.

      random_bytes
         28 bytes generated by a secure random number generator.

   The ClientHello message includes a variable-length session
   identifier.  If not empty, the value identifies a session between the
   same client and server whose security parameters the client wishes to
   reuse.  The session identifier MAY be from an earlier connection,
   this connection, or from another currently active connection.  The
   second option is useful if the client only wishes to update the
   random structures and derived values of a connection, and the third
   option makes it possible to establish several independent secure
   connections without repeating the full handshake protocol.  These
   independent connections may occur sequentially or simultaneously; a
   SessionID becomes valid when the handshake negotiating it completes
   with the exchange of Finished messages and persists until it is
   removed due to aging or because a fatal error was encountered on a
   connection associated with the session.  The actual contents of the
   SessionID are defined by the server.

      opaque SessionID<0..32>;

   Warning: Because the SessionID is transmitted without encryption or
   immediate MAC protection, servers MUST NOT place confidential
   information in session identifiers or let the contents of fake
   session identifiers cause any breach of security.  (Note that the
   content of the handshake as a whole, including the SessionID, is
   protected by the Finished messages exchanged at the end of the
   handshake.)

   The cipher suite list, passed from the client to the server in the
   ClientHello message, contains the combinations of cryptographic
   algorithms supported by the client in order of the client's
   preference (favorite choice first).  Each cipher suite defines a key
   exchange algorithm, a bulk encryption algorithm (including secret key
   length), a MAC algorithm, and a PRF.  The server will select a cipher
   suite or, if no acceptable choices are presented, return a handshake
   failure alert and close the connection.  If the list contains cipher
   suites the server does not recognize, support, or wish to use, the
   server MUST ignore those cipher suites, and process the remaining
   ones as usual.

      uint8 CipherSuite[2];    /* Cryptographic suite selector */

   The ClientHello includes a list of compression algorithms supported
   by the client, ordered according to the client's preference.

      enum { null(0), (255) } CompressionMethod;
      struct {
          ProtocolVersion client_version;
          Random random;
          SessionID session_id;
          CipherSuite cipher_suites<2..2^16-2>;
          CompressionMethod compression_methods<1..2^8-1>;
          select (extensions_present) {
              case false:
                  struct {};
              case true:
                  Extension extensions<0..2^16-1>;
          };
      } ClientHello;

   TLS allows extensions to follow the compression_methods field in an
   extensions block.  The presence of extensions can be detected by
   determining whether there are bytes following the compression_methods
   at the end of the ClientHello.  Note that this method of detecting
   optional data differs from the normal TLS method of having a
   variable-length field, but it is used for compatibility with TLS
   before extensions were defined.

   client_version
      The version of the TLS protocol by which the client wishes to
      communicate during this session.  This SHOULD be the latest
      (highest valued) version supported by the client.  For this
      version of the specification, the version will be 3.3 (see Appendix E  for details about backward compatibility).

   random
      A client-generated random structure.

   session_id
      The ID of a session the client wishes to use for this connection.
      This field is empty if no session_id is available, or if the
      client wishes to generate new security parameters.

   cipher_suites
      This is a list of the cryptographic options supported by the
      client, with the client's first preference first.  If the
      session_id field is not empty (implying a session resumption
      request), this vector MUST include at least the cipher_suite from
      that session.  Values are defined in Appendix A.5.

   compression_methods
      This is a list of the compression methods supported by the client,
      sorted by client preference.  If the session_id field is not empty
      (implying a session resumption request), it MUST include the 
      compression_method from that session.  This vector MUST contain,
      and all implementations MUST support, CompressionMethod.null.
      Thus, a client and server will always be able to agree on a
      compression method.

   extensions
      Clients MAY request extended functionality from servers by sending
      data in the extensions field.  The actual "Extension" format is
      defined in Section 7.4.1.4.

   In the event that a client requests additional functionality using
   extensions, and this functionality is not supplied by the server, the
   client MAY abort the handshake.  A server MUST accept ClientHello
   messages both with and without the extensions field, and (as for all
   other messages) it MUST check that the amount of data in the message
   precisely matches one of these formats; if not, then it MUST send a
   fatal "decode_error" alert.

   After sending the ClientHello message, the client waits for a
   ServerHello message.  Any handshake message returned by the server,
   except for a HelloRequest, is treated as a fatal error.

##### 7.4.1.3. Server Hello

   When this message will be sent:

      The server will send this message in response to a ClientHello
      message when it was able to find an acceptable set of algorithms.
      If it cannot find such a match, it will respond with a handshake
      failure alert.

   Structure of this message:

      struct {
          ProtocolVersion server_version;
          Random random;
          SessionID session_id;
          CipherSuite cipher_suite;
          CompressionMethod compression_method;
          select (extensions_present) {
              case false:
                  struct {};
              case true:
                  Extension extensions<0..2^16-1>;
          };
      } ServerHello;
   The presence of extensions can be detected by determining whether
   there are bytes following the compression_method field at the end of
   the ServerHello.

   server_version
      This field will contain the lower of that suggested by the client
      in the client hello and the highest supported by the server.  For
      this version of the specification, the version is 3.3.  (See Appendix E  for details about backward compatibility.)

   random
      This structure is generated by the server and MUST be
      independently generated from the ClientHello.random.

   session_id
      This is the identity of the session corresponding to this
      connection.  If the ClientHello.session_id was non-empty, the
      server will look in its session cache for a match.  If a match is
      found and the server is willing to establish the new connection
      using the specified session state, the server will respond with
      the same value as was supplied by the client.  This indicates a
      resumed session and dictates that the parties must proceed
      directly to the Finished messages.  Otherwise, this field will
      contain a different value identifying the new session.  The server
      may return an empty session_id to indicate that the session will
      not be cached and therefore cannot be resumed.  If a session is
      resumed, it must be resumed using the same cipher suite it was
      originally negotiated with.  Note that there is no requirement
      that the server resume any session even if it had formerly
      provided a session_id.  Clients MUST be prepared to do a full
      negotiation -- including negotiating new cipher suites -- during
      any handshake.

   cipher_suite
      The single cipher suite selected by the server from the list in
      ClientHello.cipher_suites.  For resumed sessions, this field is
      the value from the state of the session being resumed.

   compression_method
      The single compression algorithm selected by the server from the
      list in ClientHello.compression_methods.  For resumed sessions,
      this field is the value from the resumed session state.

   extensions
      A list of extensions.  Note that only extensions offered by the
      client can appear in the server's list. 





##### 7.4.1.4. Hello Extensions

   The extension format is:

      struct {
          ExtensionType extension_type;
          opaque extension_data<0..2^16-1>;
      } Extension;

      enum {
          signature_algorithms(13), (65535)
      } ExtensionType;

   Here:

   -  "extension_type" identifies the particular extension type.

   -  "extension_data" contains information specific to the particular
      extension type.

   The initial set of extensions is defined in a companion document
   [TLSEXT ].  The list of extension types is maintained by IANA as
   described in Section 12.

   An extension type MUST NOT appear in the ServerHello unless the same
   extension type appeared in the corresponding ClientHello.  If a
   client receives an extension type in ServerHello that it did not
   request in the associated ClientHello, it MUST abort the handshake
   with an unsupported_extension fatal alert.

   Nonetheless, "server-oriented" extensions may be provided in the
   future within this framework.  Such an extension (say, of type x)
   would require the client to first send an extension of type x in a
   ClientHello with empty extension_data to indicate that it supports
   the extension type.  In this case, the client is offering the
   capability to understand the extension type, and the server is taking
   the client up on its offer.

   When multiple extensions of different types are present in the
   ClientHello or ServerHello messages, the extensions MAY appear in any
   order.  There MUST NOT be more than one extension of the same type.

   Finally, note that extensions can be sent both when starting a new
   session and when requesting session resumption.  Indeed, a client
   that requests session resumption does not in general know whether the
   server will accept this request, and therefore it SHOULD send the
   same extensions as it would send if it were not attempting
   resumption. 
   In general, the specification of each extension type needs to
   describe the effect of the extension both during full handshake and
   session resumption.  Most current TLS extensions are relevant only
   when a session is initiated: when an older session is resumed, the
   server does not process these extensions in Client Hello, and does
   not include them in Server Hello.  However, some extensions may
   specify different behavior during session resumption.

   There are subtle (and not so subtle) interactions that may occur in
   this protocol between new features and existing features which may
   result in a significant reduction in overall security.  The following
   considerations should be taken into account when designing new
   extensions:

   -  Some cases where a server does not agree to an extension are error
      conditions, and some are simply refusals to support particular
      features.  In general, error alerts should be used for the former,
      and a field in the server extension response for the latter.

   -  Extensions should, as far as possible, be designed to prevent any
      attack that forces use (or non-use) of a particular feature by
      manipulation of handshake messages.  This principle should be
      followed regardless of whether the feature is believed to cause a
      security problem.

      Often the fact that the extension fields are included in the
      inputs to the Finished message hashes will be sufficient, but
      extreme care is needed when the extension changes the meaning of
      messages sent in the handshake phase.  Designers and implementors
      should be aware of the fact that until the handshake has been
      authenticated, active attackers can modify messages and insert,
      remove, or replace extensions.

   -  It would be technically possible to use extensions to change major
      aspects of the design of TLS; for example the design of cipher
      suite negotiation.  This is not recommended; it would be more
      appropriate to define a new version of TLS -- particularly since
      the TLS handshake algorithms have specific protection against
      version rollback attacks based on the version number, and the
      possibility of version rollback should be a significant
      consideration in any major design change.

###### 7.4.1.4.1. Signature Algorithms

   The client uses the "signature_algorithms" extension to indicate to
   the server which signature/hash algorithm pairs may be used in
   digital signatures.  The "extension_data" field of this extension
   contains a "supported_signature_algorithms" value. 
      enum {
          none(0), md5(1), sha1(2), sha224(3), sha256(4), sha384(5),
          sha512(6), (255)
      } HashAlgorithm;

      enum { anonymous(0), rsa(1), dsa(2), ecdsa(3), (255) }
        SignatureAlgorithm;

      struct {
            HashAlgorithm hash;
            SignatureAlgorithm signature;
      } SignatureAndHashAlgorithm;

      SignatureAndHashAlgorithm
        supported_signature_algorithms<2..2^16-2>;

   Each SignatureAndHashAlgorithm value lists a single hash/signature
   pair that the client is willing to verify.  The values are indicated
   in descending order of preference.

   Note: Because not all signature algorithms and hash algorithms may be
   accepted by an implementation (e.g., DSA with SHA-1, but not
   SHA-256), algorithms here are listed in pairs.

   hash
      This field indicates the hash algorithm which may be used.  The
      values indicate support for unhashed data, MD5 [MD5], SHA-1,
      SHA-224, SHA-256, SHA-384, and SHA-512 [SHS ], respectively.  The
      "none" value is provided for future extensibility, in case of a
      signature algorithm which does not require hashing before signing.

   signature
      This field indicates the signature algorithm that may be used.
      The values indicate anonymous signatures, RSASSA-PKCS1-v1_5
      [PKCS1] and DSA [DSS ], and ECDSA [ECDSA ], respectively.  The
      "anonymous" value is meaningless in this context but used in Section 7.4.3.  It MUST NOT appear in this extension.

   The semantics of this extension are somewhat complicated because the
   cipher suite indicates permissible signature algorithms but not hash
   algorithms.  Sections 7.4.2 and 7.4.3 describe the appropriate rules.

   If the client supports only the default hash and signature algorithms
   (listed in this section), it MAY omit the signature_algorithms
   extension.  If the client does not support the default algorithms, or
   supports other hash and signature algorithms (and it is willing to
   use them for verifying messages sent by the server, i.e., server
   certificates and server key exchange), it MUST send the 
   signature_algorithms extension, listing the algorithms it is willing
   to accept.

   If the client does not send the signature_algorithms extension, the
   server MUST do the following:

   -  If the negotiated key exchange algorithm is one of (RSA, DHE_RSA,
      DH_RSA, RSA_PSK, ECDH_RSA, ECDHE_RSA), behave as if client had
      sent the value {sha1,rsa}.

   -  If the negotiated key exchange algorithm is one of (DHE_DSS,
      DH_DSS), behave as if the client had sent the value {sha1,dsa}.

   -  If the negotiated key exchange algorithm is one of (ECDH_ECDSA,
      ECDHE_ECDSA), behave as if the client had sent value {sha1,ecdsa}.

   Note: this is a change from TLS 1.1 where there are no explicit
   rules, but as a practical matter one can assume that the peer
   supports MD5 and SHA-1.

   Note: this extension is not meaningful for TLS versions prior to 1.2.
   Clients MUST NOT offer it if they are offering prior versions.
   However, even if clients do offer it, the rules specified in [TLSEXT ]
   require servers to ignore extensions they do not understand.

   Servers MUST NOT send this extension.  TLS servers MUST support
   receiving this extension.

   When performing session resumption, this extension is not included in
   Server Hello, and the server ignores the extension in Client Hello
   (if present).

#### 7.4.2. Server Certificate

   When this message will be sent:

      The server MUST send a Certificate message whenever the agreed-
      upon key exchange method uses certificates for authentication
      (this includes all key exchange methods defined in this document
      except DH_anon).  This message will always immediately follow the
      ServerHello message.

   Meaning of this message:

      This message conveys the server's certificate chain to the client.

      The certificate MUST be appropriate for the negotiated cipher
      suite's key exchange algorithm and any negotiated extensions. 
   Structure of this message:

      opaque ASN.1Cert<1..2^24-1>;

      struct {
          ASN.1Cert certificate_list<0..2^24-1>;
      } Certificate;

   certificate_list
      This is a sequence (chain) of certificates.  The sender's
      certificate MUST come first in the list.  Each following
      certificate MUST directly certify the one preceding it.  Because
      certificate validation requires that root keys be distributed
      independently, the self-signed certificate that specifies the root
      certificate authority MAY be omitted from the chain, under the
      assumption that the remote end must already possess it in order to
      validate it in any case.

   The same message type and structure will be used for the client's
   response to a certificate request message.  Note that a client MAY
   send no certificates if it does not have an appropriate certificate
   to send in response to the server's authentication request.

   Note: PKCS #7 [PKCS7] is not used as the format for the certificate
   vector because PKCS #6 [PKCS6] extended certificates are not used.
   Also, PKCS #7 defines a SET rather than a SEQUENCE, making the task
   of parsing the list more difficult.

   The following rules apply to the certificates sent by the server:

   -  The certificate type MUST be X.509v3, unless explicitly negotiated
      otherwise (e.g., [TLSPGP ]).

   -  The end entity certificate's public key (and associated
      restrictions) MUST be compatible with the selected key exchange
      algorithm.

      Key Exchange Alg.  Certificate Key Type

      RSA                RSA public key; the certificate MUST allow the
      RSA_PSK            key to be used for encryption (the
                         keyEncipherment bit MUST be set if the key
                         usage extension is present).
                         Note: RSA_PSK is defined in [TLSPSK ]. 
      DHE_RSA            RSA public key; the certificate MUST allow the
      ECDHE_RSA          key to be used for signing (the
                         digitalSignature bit MUST be set if the key
                         usage extension is present) with the signature
                         scheme and hash algorithm that will be employed
                         in the server key exchange message.
                         Note: ECDHE_RSA is defined in [TLSECC ].

      DHE_DSS            DSA public key; the certificate MUST allow the
                         key to be used for signing with the hash
                         algorithm that will be employed in the server
                         key exchange message.

      DH_DSS             Diffie-Hellman public key; the keyAgreement bit
      DH_RSA             MUST be set if the key usage extension is
                         present.

      ECDH_ECDSA         ECDH-capable public key; the public key MUST
      ECDH_RSA           use a curve and point format supported by the
                         client, as described in [TLSECC ].

      ECDHE_ECDSA        ECDSA-capable public key; the certificate MUST
                         allow the key to be used for signing with the
                         hash algorithm that will be employed in the
                         server key exchange message.  The public key
                         MUST use a curve and point format supported by
                         the client, as described in  [TLSECC ].

   -  The "server_name" and "trusted_ca_keys" extensions [TLSEXT ] are
      used to guide certificate selection.

   If the client provided a "signature_algorithms" extension, then all
   certificates provided by the server MUST be signed by a
   hash/signature algorithm pair that appears in that extension.  Note
   that this implies that a certificate containing a key for one
   signature algorithm MAY be signed using a different signature
   algorithm (for instance, an RSA key signed with a DSA key).  This is
   a departure from TLS 1.1, which required that the algorithms be the
   same.  Note that this also implies that the DH_DSS, DH_RSA,
   ECDH_ECDSA, and ECDH_RSA key exchange algorithms do not restrict the
   algorithm used to sign the certificate.  Fixed DH certificates MAY be
   signed with any hash/signature algorithm pair appearing in the
   extension.  The names DH_DSS, DH_RSA, ECDH_ECDSA, and ECDH_RSA are
   historical. 
   If the server has multiple certificates, it chooses one of them based
   on the above-mentioned criteria (in addition to other criteria, such
   as transport layer endpoint, local configuration and preferences,
   etc.).  If the server has a single certificate, it SHOULD attempt to
   validate that it meets these criteria.

   Note that there are certificates that use algorithms and/or algorithm
   combinations that cannot be currently used with TLS.  For example, a
   certificate with RSASSA-PSS signature key (id-RSASSA-PSS OID in
   SubjectPublicKeyInfo) cannot be used because TLS defines no
   corresponding signature algorithm.

   As cipher suites that specify new key exchange methods are specified
   for the TLS protocol, they will imply the certificate format and the
   required encoded keying information.

#### 7.4.3. Server Key Exchange Message

   When this message will be sent:

      This message will be sent immediately after the server Certificate
      message (or the ServerHello message, if this is an anonymous
      negotiation).

      The ServerKeyExchange message is sent by the server only when the
      server Certificate message (if sent) does not contain enough data
      to allow the client to exchange a premaster secret.  This is true
      for the following key exchange methods:

         DHE_DSS
         DHE_RSA
         DH_anon

      It is not legal to send the ServerKeyExchange message for the
      following key exchange methods:

         RSA
         DH_DSS
         DH_RSA

      Other key exchange algorithms, such as those defined in [TLSECC ],
      MUST specify whether the ServerKeyExchange message is sent or not;
      and if the message is sent, its contents. 
   Meaning of this message:

      This message conveys cryptographic information to allow the client
      to communicate the premaster secret: a Diffie-Hellman public key
      with which the client can complete a key exchange (with the result
      being the premaster secret) or a public key for some other
      algorithm.

   Structure of this message:

      enum { dhe_dss, dhe_rsa, dh_anon, rsa, dh_dss, dh_rsa
            /* may be extended, e.g., for ECDH -- see [TLSECC ] */
           } KeyExchangeAlgorithm;

      struct {
          opaque dh_p<1..2^16-1>;
          opaque dh_g<1..2^16-1>;
          opaque dh_Ys<1..2^16-1>;
      } ServerDHParams;     /* Ephemeral DH parameters */

      dh_p
         The prime modulus used for the Diffie-Hellman operation.

      dh_g
         The generator used for the Diffie-Hellman operation.

      dh_Ys
         The server's Diffie-Hellman public value (g^X mod p). 
      struct {
          select (KeyExchangeAlgorithm) {
              case dh_anon:
                  ServerDHParams params;
              case dhe_dss:
              case dhe_rsa:
                  ServerDHParams params;
                  digitally-signed struct {
                      opaque client_random[32];
                      opaque server_random[32];
                      ServerDHParams params;
                  } signed_params;
              case rsa:
              case dh_dss:
              case dh_rsa:
                  struct {} ;
                 /* message is omitted for rsa, dh_dss, and dh_rsa */
              /* may be extended, e.g., for ECDH -- see [TLSECC ] */
          };
      } ServerKeyExchange;

      params
         The server's key exchange parameters.

      signed_params
         For non-anonymous key exchanges, a signature over the server's
         key exchange parameters.

   If the client has offered the "signature_algorithms" extension, the
   signature algorithm and hash algorithm MUST be a pair listed in that
   extension.  Note that there is a possibility for inconsistencies
   here.  For instance, the client might offer DHE_DSS key exchange but
   omit any DSA pairs from its "signature_algorithms" extension.  In
   order to negotiate correctly, the server MUST check any candidate
   cipher suites against the "signature_algorithms" extension before
   selecting them.  This is somewhat inelegant but is a compromise
   designed to minimize changes to the original cipher suite design.

   In addition, the hash and signature algorithms MUST be compatible
   with the key in the server's end-entity certificate.  RSA keys MAY be
   used with any permitted hash algorithm, subject to restrictions in
   the certificate, if any.

   Because DSA signatures do not contain any secure indication of hash
   algorithm, there is a risk of hash substitution if multiple hashes
   may be used with any key.  Currently, DSA [DSS ] may only be used with
   SHA-1.  Future revisions of DSS [DSS-3] are expected to allow the use
   of other digest algorithms with DSA, as well as guidance as to which 
   digest algorithms should be used with each key size.  In addition,
   future revisions of [PKIX ] may specify mechanisms for certificates to
   indicate which digest algorithms are to be used with DSA.

   As additional cipher suites are defined for TLS that include new key
   exchange algorithms, the server key exchange message will be sent if
   and only if the certificate type associated with the key exchange
   algorithm does not provide enough information for the client to
   exchange a premaster secret.

#### 7.4.4. Certificate Request

   When this message will be sent:

       A non-anonymous server can optionally request a certificate from
       the client, if appropriate for the selected cipher suite.  This
       message, if sent, will immediately follow the ServerKeyExchange
       message (if it is sent; otherwise, this message follows the
       server's Certificate message).

   Structure of this message:

      enum {
          rsa_sign(1), dss_sign(2), rsa_fixed_dh(3), dss_fixed_dh(4),
          rsa_ephemeral_dh_RESERVED(5), dss_ephemeral_dh_RESERVED(6),
          fortezza_dms_RESERVED(20), (255)
      } ClientCertificateType;

      opaque DistinguishedName<1..2^16-1>;

      struct {
          ClientCertificateType certificate_types<1..2^8-1>;
          SignatureAndHashAlgorithm
            supported_signature_algorithms<2^16-1>;
          DistinguishedName certificate_authorities<0..2^16-1>;
      } CertificateRequest;

   certificate_types
      A list of the types of certificate types that the client may
      offer.

         rsa_sign        a certificate containing an RSA key
         dss_sign        a certificate containing a DSA key
         rsa_fixed_dh    a certificate containing a static DH key.
         dss_fixed_dh    a certificate containing a static DH key 
   supported_signature_algorithms
      A list of the hash/signature algorithm pairs that the server is
      able to verify, listed in descending order of preference.

   certificate_authorities
      A list of the distinguished names [X501] of acceptable
      certificate_authorities, represented in DER-encoded format.  These
      distinguished names may specify a desired distinguished name for a
      root CA or for a subordinate CA; thus, this message can be used to
      describe known roots as well as a desired authorization space.  If
      the certificate_authorities list is empty, then the client MAY
      send any certificate of the appropriate ClientCertificateType,
      unless there is some external arrangement to the contrary.

   The interaction of the certificate_types and
   supported_signature_algorithms fields is somewhat complicated.
   certificate_types has been present in TLS since SSLv3, but was
   somewhat underspecified.  Much of its functionality is superseded by
   supported_signature_algorithms.  The following rules apply:

   -  Any certificates provided by the client MUST be signed using a
      hash/signature algorithm pair found in
      supported_signature_algorithms.

   -  The end-entity certificate provided by the client MUST contain a
      key that is compatible with certificate_types.  If the key is a
      signature key, it MUST be usable with some hash/signature
      algorithm pair in supported_signature_algorithms.

   -  For historical reasons, the names of some client certificate types
      include the algorithm used to sign the certificate.  For example,
      in earlier versions of TLS, rsa_fixed_dh meant a certificate
      signed with RSA and containing a static DH key.  In TLS 1.2, this
      functionality has been obsoleted by the
      supported_signature_algorithms, and the certificate type no longer
      restricts the algorithm used to sign the certificate.  For
      example, if the server sends dss_fixed_dh certificate type and
      {{sha1, dsa}, {sha1, rsa}} signature types, the client MAY reply
      with a certificate containing a static DH key, signed with RSA-
      SHA1.

   New ClientCertificateType values are assigned by IANA as described in Section 12.

   Note: Values listed as RESERVED may not be used.  They were used in
   SSLv3. 
   Note: It is a fatal handshake_failure alert for an anonymous server
   to request client authentication.

#### 7.4.5. Server Hello Done

   When this message will be sent:

      The ServerHelloDone message is sent by the server to indicate the
      end of the ServerHello and associated messages.  After sending
      this message, the server will wait for a client response.

   Meaning of this message:

      This message means that the server is done sending messages to
      support the key exchange, and the client can proceed with its
      phase of the key exchange.

      Upon receipt of the ServerHelloDone message, the client SHOULD
      verify that the server provided a valid certificate, if required,
      and check that the server hello parameters are acceptable.

   Structure of this message:

      struct { } ServerHelloDone;

#### 7.4.6. Client Certificate

   When this message will be sent:

      This is the first message the client can send after receiving a
      ServerHelloDone message.  This message is only sent if the server
      requests a certificate.  If no suitable certificate is available,
      the client MUST send a certificate message containing no
      certificates.  That is, the certificate_list structure has a
      length of zero.  If the client does not send any certificates, the
      server MAY at its discretion either continue the handshake without
      client authentication, or respond with a fatal handshake_failure
      alert.  Also, if some aspect of the certificate chain was
      unacceptable (e.g., it was not signed by a known, trusted CA), the
      server MAY at its discretion either continue the handshake
      (considering the client unauthenticated) or send a fatal alert.

      Client certificates are sent using the Certificate structure
      defined in Section 7.4.2. 
   Meaning of this message:

      This message conveys the client's certificate chain to the server;
      the server will use it when verifying the CertificateVerify
      message (when the client authentication is based on signing) or
      calculating the premaster secret (for non-ephemeral Diffie-
      Hellman).  The certificate MUST be appropriate for the negotiated
      cipher suite's key exchange algorithm, and any negotiated
      extensions.

   In particular:

   -  The certificate type MUST be X.509v3, unless explicitly negotiated
      otherwise (e.g., [TLSPGP ]).

   -  The end-entity certificate's public key (and associated
      restrictions) has to be compatible with the certificate types
      listed in CertificateRequest:

      Client Cert. Type   Certificate Key Type

      rsa_sign            RSA public key; the certificate MUST allow the
                          key to be used for signing with the signature
                          scheme and hash algorithm that will be
                          employed in the certificate verify message.

      dss_sign            DSA public key; the certificate MUST allow the
                          key to be used for signing with the hash
                          algorithm that will be employed in the
                          certificate verify message.

      ecdsa_sign          ECDSA-capable public key; the certificate MUST
                          allow the key to be used for signing with the
                          hash algorithm that will be employed in the
                          certificate verify message; the public key
                          MUST use a curve and point format supported by
                          the server.

      rsa_fixed_dh        Diffie-Hellman public key; MUST use the same
      dss_fixed_dh        parameters as server's key.

      rsa_fixed_ecdh      ECDH-capable public key; MUST use the
      ecdsa_fixed_ecdh    same curve as the server's key, and MUST use a
                          point format supported by the server.

   -  If the certificate_authorities list in the certificate request
      message was non-empty, one of the certificates in the certificate
      chain SHOULD be issued by one of the listed CAs. 
   -  The certificates MUST be signed using an acceptable hash/
      signature algorithm pair, as described in Section 7.4.4.  Note
      that this relaxes the constraints on certificate-signing
      algorithms found in prior versions of TLS.

   Note that, as with the server certificate, there are certificates
   that use algorithms/algorithm combinations that cannot be currently
   used with TLS.

#### 7.4.7. Client Key Exchange Message

   When this message will be sent:

      This message is always sent by the client.  It MUST immediately
      follow the client certificate message, if it is sent.  Otherwise,
      it MUST be the first message sent by the client after it receives
      the ServerHelloDone message.

   Meaning of this message:

      With this message, the premaster secret is set, either by direct
      transmission of the RSA-encrypted secret or by the transmission of
      Diffie-Hellman parameters that will allow each side to agree upon
      the same premaster secret.

      When the client is using an ephemeral Diffie-Hellman exponent,
      then this message contains the client's Diffie-Hellman public
      value.  If the client is sending a certificate containing a static
      DH exponent (i.e., it is doing fixed_dh client authentication),
      then this message MUST be sent but MUST be empty.

   Structure of this message:

      The choice of messages depends on which key exchange method has
      been selected.  See Section 7.4.3 for the KeyExchangeAlgorithm
      definition. 
      struct {
          select (KeyExchangeAlgorithm) {
              case rsa:
                  EncryptedPreMasterSecret;
              case dhe_dss:
              case dhe_rsa:
              case dh_dss:
              case dh_rsa:
              case dh_anon:
                  ClientDiffieHellmanPublic;
          } exchange_keys;
      } ClientKeyExchange;

##### 7.4.7.1. RSA-Encrypted Premaster Secret Message

   Meaning of this message:

      If RSA is being used for key agreement and authentication, the
      client generates a 48-byte premaster secret, encrypts it using the
      public key from the server's certificate, and sends the result in
      an encrypted premaster secret message.  This structure is a
      variant of the ClientKeyExchange message and is not a message in
      itself.

   Structure of this message:

      struct {
          ProtocolVersion client_version;
          opaque random[46];
      } PreMasterSecret;

      client_version
         The latest (newest) version supported by the client.  This is
         used to detect version rollback attacks.

      random
         46 securely-generated random bytes.

      struct {
          public-key-encrypted PreMasterSecret pre_master_secret;
      } EncryptedPreMasterSecret;

      pre_master_secret
         This random value is generated by the client and is used to
         generate the master secret, as specified in Section 8.1. 
   Note: The version number in the PreMasterSecret is the version
   offered by the client in the ClientHello.client_version, not the
   version negotiated for the connection.  This feature is designed to
   prevent rollback attacks.  Unfortunately, some old implementations
   use the negotiated version instead, and therefore checking the
   version number may lead to failure to interoperate with such
   incorrect client implementations.

   Client implementations MUST always send the correct version number in
   PreMasterSecret.  If ClientHello.client_version is TLS 1.1 or higher,
   server implementations MUST check the version number as described in
   the note below.  If the version number is TLS 1.0 or earlier, server
   implementations SHOULD check the version number, but MAY have a
   configuration option to disable the check.  Note that if the check
   fails, the PreMasterSecret SHOULD be randomized as described below.

   Note: Attacks discovered by Bleichenbacher [BLEI ] and Klima et al.
   [KPR03] can be used to attack a TLS server that reveals whether a
   particular message, when decrypted, is properly PKCS#1 formatted,
   contains a valid PreMasterSecret structure, or has the correct
   version number.

   As described by Klima [KPR03], these vulnerabilities can be avoided
   by treating incorrectly formatted message blocks and/or mismatched
   version numbers in a manner indistinguishable from correctly
   formatted RSA blocks.  In other words:

      1. Generate a string R of 46 random bytes

      2. Decrypt the message to recover the plaintext M

      3. If the PKCS#1 padding is not correct, or the length of message
         M is not exactly 48 bytes:
            pre_master_secret = ClientHello.client_version || R
         else If ClientHello.client_version <= TLS 1.0, and version
         number check is explicitly disabled:
            pre_master_secret = M
         else:
            pre_master_secret = ClientHello.client_version || M[2..47]

   Note that explicitly constructing the pre_master_secret with the
   ClientHello.client_version produces an invalid master_secret if the
   client has sent the wrong version in the original pre_master_secret.

   An alternative approach is to treat a version number mismatch as a
   PKCS-1 formatting error and randomize the premaster secret
   completely:
      1. Generate a string R of 48 random bytes

      2. Decrypt the message to recover the plaintext M

      3. If the PKCS#1 padding is not correct, or the length of message
         M is not exactly 48 bytes:
            pre_master_secret = R
         else If ClientHello.client_version <= TLS 1.0, and version
         number check is explicitly disabled:
            premaster secret = M
         else If M[0..1] != ClientHello.client_version:
            premaster secret = R
         else:
            premaster secret = M

   Although no practical attacks against this construction are known,
   Klima et al. [KPR03] describe some theoretical attacks, and therefore
   the first construction described is RECOMMENDED.

   In any case, a TLS server MUST NOT generate an alert if processing an
   RSA-encrypted premaster secret message fails, or the version number
   is not as expected.  Instead, it MUST continue the handshake with a
   randomly generated premaster secret.  It may be useful to log the
   real cause of failure for troubleshooting purposes; however, care
   must be taken to avoid leaking the information to an attacker
   (through, e.g., timing, log files, or other channels.)

   The RSAES-OAEP encryption scheme defined in [PKCS1] is more secure
   against the Bleichenbacher attack.  However, for maximal
   compatibility with earlier versions of TLS, this specification uses
   the RSAES-PKCS1-v1_5 scheme.  No variants of the Bleichenbacher
   attack are known to exist provided that the above recommendations are
   followed.

   Implementation note: Public-key-encrypted data is represented as an
   opaque vector <0..2^16-1> (see Section 4.7).  Thus, the RSA-encrypted
   PreMasterSecret in a ClientKeyExchange is preceded by two length
   bytes.  These bytes are redundant in the case of RSA because the
   EncryptedPreMasterSecret is the only data in the ClientKeyExchange
   and its length can therefore be unambiguously determined.  The SSLv3
   specification was not clear about the encoding of public-key-
   encrypted data, and therefore many SSLv3 implementations do not
   include the length bytes -- they encode the RSA-encrypted data
   directly in the ClientKeyExchange message.

   This specification requires correct encoding of the
   EncryptedPreMasterSecret complete with length bytes.  The resulting
   PDU is incompatible with many SSLv3 implementations.  Implementors 
   upgrading from SSLv3 MUST modify their implementations to generate
   and accept the correct encoding.  Implementors who wish to be
   compatible with both SSLv3 and TLS should make their implementation's
   behavior dependent on the protocol version.

   Implementation note: It is now known that remote timing-based attacks
   on TLS are possible, at least when the client and server are on the
   same LAN.  Accordingly, implementations that use static RSA keys MUST
   use RSA blinding or some other anti-timing technique, as described in
   [TIMING ].

##### 7.4.7.2. Client Diffie-Hellman Public Value

   Meaning of this message:

      This structure conveys the client's Diffie-Hellman public value
      (Yc) if it was not already included in the client's certificate.
      The encoding used for Yc is determined by the enumerated
      PublicValueEncoding.  This structure is a variant of the client
      key exchange message, and not a message in itself.

   Structure of this message:

      enum { implicit, explicit } PublicValueEncoding;

      implicit
         If the client has sent a certificate which contains a suitable
         Diffie-Hellman key (for fixed_dh client authentication), then
         Yc is implicit and does not need to be sent again.  In this
         case, the client key exchange message will be sent, but it MUST
         be empty.

      explicit
         Yc needs to be sent.

      struct {
          select (PublicValueEncoding) {
              case implicit: struct { };
              case explicit: opaque dh_Yc<1..2^16-1>;
          } dh_public;
      } ClientDiffieHellmanPublic;

      dh_Yc
         The client's Diffie-Hellman public value (Yc). 





#### 7.4.8. Certificate Verify

   When this message will be sent:

      This message is used to provide explicit verification of a client
      certificate.  This message is only sent following a client
      certificate that has signing capability (i.e., all certificates
      except those containing fixed Diffie-Hellman parameters).  When
      sent, it MUST immediately follow the client key exchange message.

   Structure of this message:

      struct {
           digitally-signed struct {
               opaque handshake_messages[handshake_messages_length];
           }
      } CertificateVerify;

      Here handshake_messages refers to all handshake messages sent or
      received, starting at client hello and up to, but not including,
      this message, including the type and length fields of the
      handshake messages.  This is the concatenation of all the
      Handshake structures (as defined in Section 7.4) exchanged thus
      far.  Note that this requires both sides to either buffer the
      messages or compute running hashes for all potential hash
      algorithms up to the time of the CertificateVerify computation.
      Servers can minimize this computation cost by offering a
      restricted set of digest algorithms in the CertificateRequest
      message.

      The hash and signature algorithms used in the signature MUST be
      one of those present in the supported_signature_algorithms field
      of the CertificateRequest message.  In addition, the hash and
      signature algorithms MUST be compatible with the key in the
      client's end-entity certificate.  RSA keys MAY be used with any
      permitted hash algorithm, subject to restrictions in the
      certificate, if any.

      Because DSA signatures do not contain any secure indication of
      hash algorithm, there is a risk of hash substitution if multiple
      hashes may be used with any key.  Currently, DSA [DSS ] may only be
      used with SHA-1.  Future revisions of DSS [DSS-3] are expected to
      allow the use of other digest algorithms with DSA, as well as
      guidance as to which digest algorithms should be used with each
      key size.  In addition, future revisions of [PKIX ] may specify
      mechanisms for certificates to indicate which digest algorithms
      are to be used with DSA. 





#### 7.4.9. Finished

   When this message will be sent:

      A Finished message is always sent immediately after a change
      cipher spec message to verify that the key exchange and
      authentication processes were successful.  It is essential that a
      change cipher spec message be received between the other handshake
      messages and the Finished message.

   Meaning of this message:

      The Finished message is the first one protected with the just
      negotiated algorithms, keys, and secrets.  Recipients of Finished
      messages MUST verify that the contents are correct.  Once a side
      has sent its Finished message and received and validated the
      Finished message from its peer, it may begin to send and receive
      application data over the connection.

   Structure of this message:

      struct {
          opaque verify_data[verify_data_length];
      } Finished;

      verify_data
         PRF(master_secret, finished_label, Hash(handshake_messages))
            [0..verify_data_length-1];

      finished_label
         For Finished messages sent by the client, the string
         "client finished".  For Finished messages sent by the server,
         the string "server finished".

      Hash denotes a Hash of the handshake messages.  For the PRF
      defined in Section 5, the Hash MUST be the Hash used as the basis
      for the PRF.  Any cipher suite which defines a different PRF MUST
      also define the Hash to use in the Finished computation.

      In previous versions of TLS, the verify_data was always 12 octets
      long.  In the current version of TLS, it depends on the cipher
      suite.  Any cipher suite which does not explicitly specify
      verify_data_length has a verify_data_length equal to 12.  This
      includes all existing cipher suites.  Note that this
      representation has the same encoding as with previous versions.
      Future cipher suites MAY specify other lengths but such length
      MUST be at least 12 bytes. 
      handshake_messages
         All of the data from all messages in this handshake (not
         including any HelloRequest messages) up to, but not including,
         this message.  This is only data visible at the handshake layer
         and does not include record layer headers.  This is the
         concatenation of all the Handshake structures as defined in Section 7.4, exchanged thus far.

   It is a fatal error if a Finished message is not preceded by a
   ChangeCipherSpec message at the appropriate point in the handshake.

   The value handshake_messages includes all handshake messages starting
   at ClientHello up to, but not including, this Finished message.  This
   may be different from handshake_messages in Section 7.4.8 because it
   would include the CertificateVerify message (if sent).  Also, the
   handshake_messages for the Finished message sent by the client will
   be different from that for the Finished message sent by the server,
   because the one that is sent second will include the prior one.

   Note: ChangeCipherSpec messages, alerts, and any other record types
   are not handshake messages and are not included in the hash
   computations.  Also, HelloRequest messages are omitted from handshake
   hashes.

## 12. IANA Considerations

   This document uses several registries that were originally created in
   [TLS1.1].  IANA has updated these to reference this document.  The
   registries and their allocation policies (unchanged from [TLS1.1])
   are listed below. 
   -  TLS ClientCertificateType Identifiers Registry: Future values in
      the range 0-63 (decimal) inclusive are assigned via Standards
      Action [RFC2434].  Values in the range 64-223 (decimal) inclusive
      are assigned via Specification Required [RFC2434].  Values from
      224-255 (decimal) inclusive are reserved for Private Use
      [RFC2434].

   -  TLS Cipher Suite Registry: Future values with the first byte in
      the range 0-191 (decimal) inclusive are assigned via Standards
      Action [RFC2434].  Values with the first byte in the range 192-254
      (decimal) are assigned via Specification Required [RFC2434].
      Values with the first byte 255 (decimal) are reserved for Private
      Use [RFC2434].

   -  This document defines several new HMAC-SHA256-based cipher suites,
      whose values (in Appendix A.5) have been allocated from the TLS
      Cipher Suite registry.

   -  TLS ContentType Registry: Future values are allocated via
      Standards Action [RFC2434].

   -  TLS Alert Registry: Future values are allocated via Standards
      Action [RFC2434].

   -  TLS HandshakeType Registry: Future values are allocated via
      Standards Action [RFC2434].

   This document also uses a registry originally created in [RFC4366].
   IANA has updated it to reference this document.  The registry and its
   allocation policy (unchanged from [RFC4366]) is listed below:

   -  TLS ExtensionType Registry: Future values are allocated via IETF
      Consensus [RFC2434].  IANA has updated this registry to include
      the signature_algorithms extension and its corresponding value
      (see Section 7.4.1.4).

   In addition, this document defines two new registries to be
   maintained by IANA:

   -  TLS SignatureAlgorithm Registry: The registry has been initially
      populated with the values described in Section 7.4.1.4.1.  Future
      values in the range 0-63 (decimal) inclusive are assigned via
      Standards Action [RFC2434].  Values in the range 64-223 (decimal)
      inclusive are assigned via Specification Required [RFC2434].
      Values from 224-255 (decimal) inclusive are reserved for Private
      Use [RFC2434]. 
   -  TLS HashAlgorithm Registry: The registry has been initially
      populated with the values described in Section 7.4.1.4.1.  Future
      values in the range 0-63 (decimal) inclusive are assigned via
      Standards Action [RFC2434].  Values in the range 64-223 (decimal)
      inclusive are assigned via Specification Required [RFC2434].
      Values from 224-255 (decimal) inclusive are reserved for Private
      Use [RFC2434].

      This document also uses the TLS Compression Method Identifiers
      Registry, defined in [RFC3749].  IANA has allocated value 0 for
      the "null" compression method. 





# Referenced Sections from RFC 2119: Key words for use in RFCs to Indicate Requirement Levels

The following sections were referenced. Remaining sections are not included.

## 2. MUST NOT

 This phrase, or the phrase "SHALL NOT", mean that the
   definition is an absolute prohibition of the specification.

# Referenced Sections from RFC 7685: A Transport Layer Security (TLS) ClientHello Padding Extension

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Successive TLS [RFC5246] versions have added support for more cipher
   suites and, over time, more TLS extensions have been defined.  This
   has caused the size of the TLS ClientHello to grow, and the
   additional size has caused some implementation bugs to come to light.
   At least one of these implementation bugs can be ameliorated by
   making the ClientHello even larger.  This is desirable given that
   fully comprehensive patching of affected implementations is difficult
   to achieve.

   This memo describes a TLS extension that can be used to pad a
   ClientHello to a desired size in order to avoid implementation bugs
   caused by certain ClientHello sizes.

## 5. Security Considerations

   The contents of the padding extension could be used as a covert
   channel.  In order to prevent this, the contents are required to be
   all zeros, although the length of the extension can still be used as
   a much smaller covert channel. 





# Referenced Sections from RFC 8996: Deprecating TLS 1.0 and TLS 1.1

The following sections were referenced. Remaining sections are not included.

## 5. Do Not Use TLS 1.1

   TLS 1.1 MUST NOT be used.  Negotiation of TLS 1.1 from any version of
   TLS MUST NOT be permitted.

   Pragmatically, clients MUST NOT send a ClientHello with
   ClientHello.client_version set to {03,02}.  Similarly, servers MUST
   NOT send a ServerHello with ServerHello.server_version set to
   {03,02}.  Any party receiving a Hello message with the protocol
   version set to {03,02} MUST respond with a "protocol_version" alert
   message and close the connection.

   Any newer version of TLS is more secure than TLS 1.1.  While TLS 1.1
   can be configured to prevent some types of interception, using the
   highest version available is preferred.  Support for TLS 1.1 is
   dwindling in libraries and will impact security going forward if
   mitigations for attacks cannot be easily addressed and supported in
   older libraries.

   Historically, TLS specifications were not clear on what the record
   layer version number (TLSPlaintext.version) could contain when
   sending a ClientHello message.  Appendix E of [RFC5246] notes that
   TLSPlaintext.version could be selected to maximize interoperability,
   though no definitive value is identified as ideal.  That guidance is
   still applicable; therefore, TLS servers MUST accept any value
   {03,XX} (including {03,00}) as the record layer version number for
   ClientHello, but they MUST NOT negotiate TLS 1.1.

# Referenced Sections from RFC 6066: Transport Layer Security (TLS) Extensions: Extension Definitions

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   The Transport Layer Security (TLS) Protocol Version 1.2 is specified
   in [RFC5246].  That specification includes the framework for
   extensions to TLS, considerations in designing such extensions (see Section 7.4.1.4 of [RFC5246]), and IANA Considerations for the
   allocation of new extension code points; however, it does not specify
   any particular extensions other than Signature Algorithms (see Section 7.4.1.4.1 of [RFC5246]).

   This document provides the specifications for existing TLS
   extensions.  It is, for the most part, the adaptation and editing of
   material from RFC 4366, which covered TLS extensions for TLS 1.0 (RFC 
   2246) and TLS 1.1 (RFC 4346).

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





### 1.2. Conventions Used in This Document

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in
   [RFC2119].

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

## 4. Maximum Fragment Length Negotiation

   Without this extension, TLS specifies a fixed maximum plaintext
   fragment length of 2^14 bytes.  It may be desirable for constrained
   clients to negotiate a smaller maximum fragment length due to memory
   limitations or bandwidth limitations.

   In order to negotiate smaller maximum fragment lengths, clients MAY
   include an extension of type "max_fragment_length" in the (extended)
   client hello.  The "extension_data" field of this extension SHALL
   contain:

      enum{
          2^9(1), 2^10(2), 2^11(3), 2^12(4), (255)
      } MaxFragmentLength;

   whose value is the desired maximum fragment length.  The allowed
   values for this field are: 2^9, 2^10, 2^11, and 2^12.

   Servers that receive an extended client hello containing a
   "max_fragment_length" extension MAY accept the requested maximum
   fragment length by including an extension of type
   "max_fragment_length" in the (extended) server hello.  The
   "extension_data" field of this extension SHALL contain a
   "MaxFragmentLength" whose value is the same as the requested maximum
   fragment length.

   If a server receives a maximum fragment length negotiation request
   for a value other than the allowed values, it MUST abort the
   handshake with an "illegal_parameter" alert.  Similarly, if a client
   receives a maximum fragment length negotiation response that differs
   from the length it requested, it MUST also abort the handshake with
   an "illegal_parameter" alert.

   Once a maximum fragment length other than 2^14 has been successfully
   negotiated, the client and server MUST immediately begin fragmenting
   messages (including handshake messages) to ensure that no fragment
   larger than the negotiated length is sent.  Note that TLS already
   requires clients and servers to support fragmentation of handshake
   messages. 
   The negotiated length applies for the duration of the session
   including session resumptions.

   The negotiated length limits the input that the record layer may
   process without fragmentation (that is, the maximum value of
   TLSPlaintext.length; see [RFC5246], Section 6.2.1).  Note that the
   output of the record layer may be larger.  For example, if the
   negotiated length is 2^9=512, then, when using currently defined
   cipher suites (those defined in [RFC5246] and [RFC2712]) and null
   compression, the record-layer output can be at most 805 bytes: 5
   bytes of headers, 512 bytes of application data, 256 bytes of
   padding, and 32 bytes of MAC.  This means that in this event a TLS
   record-layer peer receiving a TLS record-layer message larger than
   805 bytes MUST discard the message and send a "record_overflow"
   alert, without decrypting the message.  When this extension is used
   with Datagram Transport Layer Security (DTLS), implementations SHOULD
   NOT generate record_overflow alerts unless the packet passes message
   authentication.

## 5. Client Certificate URLs

   Without this extension, TLS specifies that when client authentication
   is performed, client certificates are sent by clients to servers
   during the TLS handshake.  It may be desirable for constrained
   clients to send certificate URLs in place of certificates, so that
   they do not need to store their certificates and can therefore save
   memory.

   In order to negotiate sending certificate URLs to a server, clients
   MAY include an extension of type "client_certificate_url" in the
   (extended) client hello.  The "extension_data" field of this
   extension SHALL be empty.

   (Note that it is necessary to negotiate the use of client certificate
   URLs in order to avoid "breaking" existing TLS servers.)

   Servers that receive an extended client hello containing a
   "client_certificate_url" extension MAY indicate that they are willing
   to accept certificate URLs by including an extension of type
   "client_certificate_url" in the (extended) server hello.  The
   "extension_data" field of this extension SHALL be empty.

   After negotiation of the use of client certificate URLs has been
   successfully completed (by exchanging hellos including
   "client_certificate_url" extensions), clients MAY send a
   "CertificateURL" message in place of a "Certificate" message as
   follows (see also Section 2):
      enum {
          individual_certs(0), pkipath(1), (255)
      } CertChainType;

      struct {
          CertChainType type;
          URLAndHash url_and_hash_list<1..2^16-1>;
      } CertificateURL;

      struct {
          opaque url<1..2^16-1>;
          unint8 padding;
          opaque SHA1Hash[20];
      } URLAndHash;

   Here, "url_and_hash_list" contains a sequence of URLs and hashes.
   Each "url" MUST be an absolute URI reference according to [RFC3986]
   that can be immediately used to fetch the certificate(s).

   When X.509 certificates are used, there are two possibilities:

   -  If CertificateURL.type is "individual_certs", each URL refers to a
      single DER-encoded X.509v3 certificate, with the URL for the
      client's certificate first.

   -  If CertificateURL.type is "pkipath", the list contains a single
      URL referring to a DER-encoded certificate chain, using the type
      PkiPath described in Section 10.1.

   When any other certificate format is used, the specification that
   describes use of that format in TLS should define the encoding format
   of certificates or certificate chains, and any constraint on their
   ordering.

   The "padding" byte MUST be 0x01.  It is present to make the structure
   backwards compatible.

   The hash corresponding to each URL is the SHA-1 hash of the
   certificate or certificate chain (in the case of X.509 certificates,
   the DER-encoded certificate or the DER-encoded PkiPath).

   Note that when a list of URLs for X.509 certificates is used, the
   ordering of URLs is the same as that used in the TLS Certificate
   message (see [RFC5246], Section 7.4.2), but opposite to the order in
   which certificates are encoded in PkiPath.  In either case, the self-
   signed root certificate MAY be omitted from the chain, under the
   assumption that the server must already possess it in order to
   validate it. 
   Servers receiving "CertificateURL" SHALL attempt to retrieve the
   client's certificate chain from the URLs and then process the
   certificate chain as usual.  A cached copy of the content of any URL
   in the chain MAY be used, provided that the SHA-1 hash matches the
   hash of the cached copy.

   Servers that support this extension MUST support the 'http' URI
   scheme for certificate URLs and MAY support other schemes.  Use of
   other schemes than 'http', 'https', or 'ftp' may create unexpected
   problems.

   If the protocol used is HTTP, then the HTTP server can be configured
   to use the Cache-Control and Expires directives described in
   [RFC2616] to specify whether and for how long certificates or
   certificate chains should be cached.

   The TLS server MUST NOT follow HTTP redirects when retrieving the
   certificates or certificate chain.  The URLs used in this extension
   MUST NOT be chosen to depend on such redirects.

   If the protocol used to retrieve certificates or certificate chains
   returns a MIME-formatted response (as HTTP does), then the following
   MIME Content-Types SHALL be used: when a single X.509v3 certificate
   is returned, the Content-Type is "application/pkix-cert" [RFC2585],
   and when a chain of X.509v3 certificates is returned, the Content-
   Type is "application/pkix-pkipath" (Section 10.1).

   The server MUST check that the SHA-1 hash of the contents of the
   object retrieved from that URL (after decoding any MIME Content-
   Transfer-Encoding) matches the given hash.  If any retrieved object
   does not have the correct SHA-1 hash, the server MUST abort the
   handshake with a bad_certificate_hash_value(114) alert.  This alert
   is always fatal.

   Clients may choose to send either "Certificate" or "CertificateURL"
   after successfully negotiating the option to send certificate URLs.
   The option to send a certificate is included to provide flexibility
   to clients possessing multiple certificates.

   If a server is unable to obtain certificates in a given
   CertificateURL, it MUST send a fatal certificate_unobtainable(111)
   alert if it requires the certificates to complete the handshake.  If
   the server does not require the certificates, then the server
   continues the handshake.  The server MAY send a warning-level alert
   in this case.  Clients receiving such an alert SHOULD log the alert
   and continue with the handshake if possible. 





## 6. Trusted CA Indication

   Constrained clients that, due to memory limitations, possess only a
   small number of CA root keys may wish to indicate to servers which
   root keys they possess, in order to avoid repeated handshake
   failures.

   In order to indicate which CA root keys they possess, clients MAY
   include an extension of type "trusted_ca_keys" in the (extended)
   client hello.  The "extension_data" field of this extension SHALL
   contain "TrustedAuthorities" where:

      struct {
          TrustedAuthority trusted_authorities_list<0..2^16-1>;
      } TrustedAuthorities;

      struct {
          IdentifierType identifier_type;
          select (identifier_type) {
              case pre_agreed: struct {};
              case key_sha1_hash: SHA1Hash;
              case x509_name: DistinguishedName;
              case cert_sha1_hash: SHA1Hash;
          } identifier;
      } TrustedAuthority;

      enum {
          pre_agreed(0), key_sha1_hash(1), x509_name(2),
          cert_sha1_hash(3), (255)
      } IdentifierType;

      opaque DistinguishedName<1..2^16-1>;

   Here, "TrustedAuthorities" provides a list of CA root key identifiers
   that the client possesses.  Each CA root key is identified via
   either:

   -  "pre_agreed": no CA root key identity supplied.

   -  "key_sha1_hash": contains the SHA-1 hash of the CA root key.  For
      Digital Signature Algorithm (DSA) and Elliptic Curve Digital
      Signature Algorithm (ECDSA) keys, this is the hash of the
      "subjectPublicKey" value.  For RSA keys, the hash is of the big-
      endian byte string representation of the modulus without any
      initial zero-valued bytes.  (This copies the key hash formats
      deployed in other environments.)
   -  "x509_name": contains the DER-encoded X.509 DistinguishedName of
      the CA.

   -  "cert_sha1_hash": contains the SHA-1 hash of a DER-encoded
      Certificate containing the CA root key.

   Note that clients may include none, some, or all of the CA root keys
   they possess in this extension.

   Note also that it is possible that a key hash or a Distinguished Name
   alone may not uniquely identify a certificate issuer (for example, if
   a particular CA has multiple key pairs).  However, here we assume
   this is the case following the use of Distinguished Names to identify
   certificate issuers in TLS.

   The option to include no CA root keys is included to allow the client
   to indicate possession of some pre-defined set of CA root keys.

   Servers that receive a client hello containing the "trusted_ca_keys"
   extension MAY use the information contained in the extension to guide
   their selection of an appropriate certificate chain to return to the
   client.  In this event, the server SHALL include an extension of type
   "trusted_ca_keys" in the (extended) server hello.  The
   "extension_data" field of this extension SHALL be empty.

# Referenced Sections from RFC 7919: Negotiated Finite Field Diffie-Hellman Ephemeral Parameters for Transport Layer Security (TLS)

The following sections were referenced. Remaining sections are not included.

### 6.1. Preference Ordering

   The ordering of named groups in the Supported Groups extension may
   contain some ECDHE groups and some FFDHE groups.  These SHOULD be
   ranked in the order preferred by the client.

   However, the ClientHello also contains a list of desired cipher
   suites, also ranked in preference order.  This presents the
   possibility of conflicted preferences.  For example, if the
   ClientHello contains a cipher_suite field with two choices in order
   <TLS_DHE_RSA_WITH_AES_128_CBC_SHA,
   TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA> and the Supported Groups
   extension contains two choices in order <secp256r1,ffdhe3072>, then
   there is a clear contradiction.  Clients SHOULD NOT present such a
   contradiction since it does not represent a sensible ordering.  A
   server that encounters such a contradiction when selecting between an
   ECDHE or FFDHE key exchange mechanism while trying to respect client
   preferences SHOULD give priority to the Supported Groups extension
   (in the example case, it should select
   TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA with secp256r1) but MAY resolve
   the contradiction any way it sees fit.

   More subtly, clients MAY interleave preferences between ECDHE and
   FFDHE groups; for example, if stronger groups are preferred
   regardless of cost, but weaker groups are acceptable, the Supported
   Groups extension could consist of
   <ffdhe8192,secp384p1,ffdhe3072,secp256r1>.  In this example, with the
   same cipher_suite field offered as the previous example, a server
   configured to respect client preferences and with support for all
   listed groups SHOULD select TLS_DHE_RSA_WITH_AES_128_CBC_SHA with
   ffdhe8192.  A server configured to respect client preferences and
   with support for only secp384p1 and ffdhe3072 SHOULD select
   TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA with secp384p1. 





### 8.1. Negotiation Resistance to Active Attacks

   Because the contents of the Supported Groups extension are hashed in
   the Finished message, an active Man in the Middle (MITM) that tries
   to filter or omit groups will cause the handshake to fail, but
   possibly not before getting the peer to do something it would not
   otherwise have done.

   An attacker who impersonates the server can try to do any of the
   following:

   o  Pretend that a non-compatible server is actually capable of this
      extension and select a group from the client's list, causing the
      client to select a group it is willing to negotiate.  It is
      unclear how this would be an effective attack.

   o  Pretend that a compatible server is actually non-compatible by
      negotiating a non-FFDHE cipher suite.  This is no different than
      MITM cipher suite filtering.

   o  Pretend that a compatible server is actually non-compatible by
      negotiating a DHE cipher suite with a custom (perhaps weak) group
      selected by the attacker.  This is no worse than the current
      scenario and would require the attacker to be able to sign the
      ServerDHParams, which should not be possible without access to the
      server's secret key.

   An attacker who impersonates the client can try to do the following:

   o  Pretend that a compatible client is not compatible (e.g., by not
      offering the Supported Groups extension or by replacing the
      Supported Groups extension with one that includes no FFDHE
      groups).  This could cause the server to negotiate a weaker DHE
      group during the handshake or to select a non-FFDHE cipher suite,
      but it would fail to complete during the final check of the
      Finished message.

   o  Pretend that a non-compatible client is compatible (e.g., by
      adding the Supported Groups extension or by adding FFDHE groups to
      the extension).  This could cause the server to select a
      particular named group in the ServerKeyExchange or to avoid
      selecting an FFDHE cipher suite.  The peers would fail to compute
      the final check of the Finished message. 
   o  Change the list of groups offered by the client (e.g., by removing
      the stronger of the set of groups offered).  This could cause the
      server to negotiate a weaker group than desired, but again, should
      be caught by the check in the Finished message.

### 8.3. Finite Field DHE Only

   Note that this document specifically targets only finite field
   Diffie-Hellman ephemeral key exchange mechanisms.  It does not cover
   the non-ephemeral DH key exchange mechanisms, nor does it address
   ECDHE key exchange, which is defined in [RFC4492].

   Measured by computational cost to the TLS peers, ECDHE appears today
   to offer a much stronger key exchange mechanism than FFDHE. 





### A.1. ffdhe2048

   The 2048-bit group has registry value 256 and is calculated from the
   following formula:

   The modulus is:

   p = 2^2048 - 2^1984 + {[2^1918 * e] + 560316 } * 2^64 - 1
   The hexadecimal representation of p is:

    FFFFFFFF FFFFFFFF ADF85458 A2BB4A9A AFDC5620 273D3CF1
    D8B9C583 CE2D3695 A9E13641 146433FB CC939DCE 249B3EF9
    7D2FE363 630C75D8 F681B202 AEC4617A D3DF1ED5 D5FD6561
    2433F51F 5F066ED0 85636555 3DED1AF3 B557135E 7F57C935
    984F0C70 E0E68B77 E2A689DA F3EFE872 1DF158A1 36ADE735
    30ACCA4F 483A797A BC0AB182 B324FB61 D108A94B B2C8E3FB
    B96ADAB7 60D7F468 1D4F42A3 DE394DF4 AE56EDE7 6372BB19
    0B07A7C8 EE0A6D70 9E02FCE1 CDF7E2EC C03404CD 28342F61
    9172FE9C E98583FF 8E4F1232 EEF28183 C3FE3B1B 4C6FAD73
    3BB5FCBC 2EC22005 C58EF183 7D1683B2 C6F34A26 C1B2EFFA
    886B4238 61285C97 FFFFFFFF FFFFFFFF

   The generator is: g = 2

   The group size is: q = (p-1)/2

   The hexadecimal representation of q is:

    7FFFFFFF FFFFFFFF D6FC2A2C 515DA54D 57EE2B10 139E9E78
    EC5CE2C1 E7169B4A D4F09B20 8A3219FD E649CEE7 124D9F7C
    BE97F1B1 B1863AEC 7B40D901 576230BD 69EF8F6A EAFEB2B0
    9219FA8F AF833768 42B1B2AA 9EF68D79 DAAB89AF 3FABE49A
    CC278638 707345BB F15344ED 79F7F439 0EF8AC50 9B56F39A
    98566527 A41D3CBD 5E0558C1 59927DB0 E88454A5 D96471FD
    DCB56D5B B06BFA34 0EA7A151 EF1CA6FA 572B76F3 B1B95D8C
    8583D3E4 770536B8 4F017E70 E6FBF176 601A0266 941A17B0
    C8B97F4E 74C2C1FF C7278919 777940C1 E1FF1D8D A637D6B9
    9DDAFE5E 17611002 E2C778C1 BE8B41D9 6379A513 60D977FD
    4435A11C 30942E4B FFFFFFFF FFFFFFFF

   The estimated symmetric-equivalent strength of this group is 103
   bits.

   Peers using ffdhe2048 that want to optimize their key exchange with a
   short exponent (Section 5.2) should choose a secret key of at least
   225 bits.

# Referenced Sections from RFC 5764: Datagram Transport Layer Security (DTLS) Extension to Establish Keys for the Secure Real-time Transport Protocol (SRTP)

The following sections were referenced. Remaining sections are not included.

#### 4.1.1. use_srtp Extension Definition

   The client MUST fill the extension_data field of the "use_srtp"
   extension with an UseSRTPData value (see Section 9 for the
   registration):

      uint8 SRTPProtectionProfile[2];

      struct {
         SRTPProtectionProfiles SRTPProtectionProfiles;
         opaque srtp_mki<0..255>;
      } UseSRTPData;

      SRTPProtectionProfile SRTPProtectionProfiles<2..2^16-1>;

   The SRTPProtectionProfiles list indicates the SRTP protection
   profiles that the client is willing to support, listed in descending
   order of preference.  The srtp_mki value contains the SRTP Master Key
   Identifier (MKI) value (if any) that the client will use for his SRTP
   packets.  If this field is of zero length, then no MKI will be used.

   Note: for those unfamiliar with TLS syntax, "srtp_mki<0..255>"
   indicates a variable-length value with a length between 0 and 255
   (inclusive).  Thus, the MKI may be up to 255 bytes long.

   If the server is willing to accept the use_srtp extension, it MUST
   respond with its own "use_srtp" extension in the ExtendedServerHello.
   The extension_data field MUST contain a UseSRTPData value with a
   single SRTPProtectionProfile value that the server has chosen for use
   with this connection.  The server MUST NOT select a value that the
   client has not offered.  If there is no shared profile, the server
   SHOULD NOT return the use_srtp extension at which point the
   connection falls back to the negotiated DTLS cipher suite.  If that
   is not acceptable, the server SHOULD return an appropriate DTLS
   alert. 





# Referenced Sections from RFC 6520: Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS) Heartbeat Extension

The following sections were referenced. Remaining sections are not included.

## 4. Heartbeat Request and Response Messages

   The Heartbeat protocol messages consist of their type and an
   arbitrary payload and padding.

   struct {
      HeartbeatMessageType type;
      uint16 payload_length;
      opaque payload[HeartbeatMessage.payload_length];
      opaque padding[padding_length];
   } HeartbeatMessage;

   The total length of a HeartbeatMessage MUST NOT exceed 2^14 or
   max_fragment_length when negotiated as defined in [RFC6066].

   type:  The message type, either heartbeat_request or
      heartbeat_response.

   payload_length:  The length of the payload.

   payload:  The payload consists of arbitrary content.

   padding:  The padding is random content that MUST be ignored by the
      receiver.  The length of a HeartbeatMessage is TLSPlaintext.length
      for TLS and DTLSPlaintext.length for DTLS.  Furthermore, the
      length of the type field is 1 byte, and the length of the
      payload_length is 2.  Therefore, the padding_length is
      TLSPlaintext.length - payload_length - 3 for TLS and
      DTLSPlaintext.length - payload_length - 3 for DTLS.  The
      padding_length MUST be at least 16.

   The sender of a HeartbeatMessage MUST use a random padding of at
   least 16 bytes.  The padding of a received HeartbeatMessage message
   MUST be ignored.

   If the payload_length of a received HeartbeatMessage is too large,
   the received HeartbeatMessage MUST be discarded silently. 
   When a HeartbeatRequest message is received and sending a
   HeartbeatResponse is not prohibited as described elsewhere in this
   document, the receiver MUST send a corresponding HeartbeatResponse
   message carrying an exact copy of the payload of the received
   HeartbeatRequest.

   If a received HeartbeatResponse message does not contain the expected
   payload, the message MUST be discarded silently.  If it does contain
   the expected payload, the retransmission timer MUST be stopped.

# Referenced Sections from RFC 7301: Transport Layer Security (TLS) Application-Layer Protocol Negotiation Extension

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Increasingly, application-layer protocols are encapsulated in the TLS
   protocol [RFC5246].  This encapsulation enables applications to use
   the existing, secure communications links already present on port 443
   across virtually the entire global IP infrastructure.

   When multiple application protocols are supported on a single server-
   side port number, such as port 443, the client and the server need to
   negotiate an application protocol for use with each connection.  It
   is desirable to accomplish this negotiation without adding network
   round-trips between the client and the server, as each round-trip
   will degrade an end-user's experience.  Further, it would be
   advantageous to allow certificate selection based on the negotiated
   application protocol. 
   This document specifies a TLS extension that permits the application
   layer to negotiate protocol selection within the TLS handshake.  This
   work was requested by the HTTPbis WG to address the negotiation of
   HTTP/2 ([HTTP2]) over TLS; however, ALPN facilitates negotiation of
   arbitrary application-layer protocols.

   With ALPN, the client sends the list of supported application
   protocols as part of the TLS ClientHello message.  The server chooses
   a protocol and sends the selected protocol as part of the TLS
   ServerHello message.  The application protocol negotiation can thus
   be accomplished within the TLS handshake, without adding network
   round-trips, and allows the server to associate a different
   certificate with each application protocol, if desired.

## 3. Application-Layer Protocol Negotiation





### 3.1. The Application-Layer Protocol Negotiation Extension

   A new extension type ("application_layer_protocol_negotiation(16)")
   is defined and MAY be included by the client in its "ClientHello"
   message.

   enum {
       application_layer_protocol_negotiation(16), (65535)
   } ExtensionType;

   The "extension_data" field of the
   ("application_layer_protocol_negotiation(16)") extension SHALL
   contain a "ProtocolNameList" value.

   opaque ProtocolName<1..2^8-1>;

   struct {
       ProtocolName protocol_name_list<2..2^16-1>
   } ProtocolNameList;

   "ProtocolNameList" contains the list of protocols advertised by the
   client, in descending order of preference.  Protocols are named by
   IANA-registered, opaque, non-empty byte strings, as described further
   in Section 6 ("IANA Considerations") of this document.  Empty strings
   MUST NOT be included and byte strings MUST NOT be truncated. 
   Servers that receive a ClientHello containing the
   "application_layer_protocol_negotiation" extension MAY return a
   suitable protocol selection response to the client.  The server will
   ignore any protocol name that it does not recognize.  A new
   ServerHello extension type
   ("application_layer_protocol_negotiation(16)") MAY be returned to the
   client within the extended ServerHello message.  The "extension_data"
   field of the ("application_layer_protocol_negotiation(16)") extension
   is structured the same as described above for the client
   "extension_data", except that the "ProtocolNameList" MUST contain
   exactly one "ProtocolName".

   Therefore, a full handshake with the
   "application_layer_protocol_negotiation" extension in the ClientHello
   and ServerHello messages has the following flow (contrast with Section 7.3 of [RFC5246]):

   Client                                              Server

   ClientHello                     -------->       ServerHello
     (ALPN extension &                               (ALPN extension &
      list of protocols)                              selected protocol)
                                                   Certificate*
                                                   ServerKeyExchange*
                                                   CertificateRequest*
                                   <--------       ServerHelloDone
   Certificate*
   ClientKeyExchange
   CertificateVerify*
   [ChangeCipherSpec]
   Finished                        -------->
                                                   [ChangeCipherSpec]
                                   <--------       Finished
   Application Data                <------->       Application Data

                                 Figure 1

   * Indicates optional or situation-dependent messages that are not
   always sent. 
   An abbreviated handshake with the
   "application_layer_protocol_negotiation" extension has the following
   flow:

   Client                                              Server

   ClientHello                     -------->       ServerHello
     (ALPN extension &                               (ALPN extension &
      list of protocols)                              selected protocol)
                                                   [ChangeCipherSpec]
                                   <--------       Finished
   [ChangeCipherSpec]
   Finished                        -------->
   Application Data                <------->       Application Data

                                 Figure 2

   Unlike many other TLS extensions, this extension does not establish
   properties of the session, only of the connection.  When session
   resumption or session tickets [RFC5077] are used, the previous
   contents of this extension are irrelevant, and only the values in the
   new handshake messages are considered.

### 3.2. Protocol Selection

   It is expected that a server will have a list of protocols that it
   supports, in preference order, and will only select a protocol if the
   client supports it.  In that case, the server SHOULD select the most
   highly preferred protocol that it supports and that is also
   advertised by the client.  In the event that the server supports no
   protocols that the client advertises, then the server SHALL respond
   with a fatal "no_application_protocol" alert.

   enum {
       no_application_protocol(120),
       (255)
   } AlertDescription;

   The protocol identified in the
   "application_layer_protocol_negotiation" extension type in the
   ServerHello SHALL be definitive for the connection, until
   renegotiated.  The server SHALL NOT respond with a selected protocol
   and subsequently use a different protocol for application data
   exchange. 





# Referenced Sections from RFC 6962: Certificate Transparency

The following sections were referenced. Remaining sections are not included.

### 3.3. Including the Signed Certificate Timestamp in the TLS Handshake

   The SCT data corresponding to the end-entity certificate from at
   least one log must be included in the TLS handshake, either by using
   an X509v3 certificate extension as described below, by using a TLS
   extension (Section 7.4.1.4 of [RFC5246]) with type
   "signed_certificate_timestamp", or by using Online Certificate Status
   Protocol (OCSP) Stapling (also known as the "Certificate Status 
   Request" TLS extension; see [RFC6066]), where the response includes
   an OCSP extension with OID 1.3.6.1.4.1.11129.2.4.5 (see [RFC2560])
   and body:

       SignedCertificateTimestampList ::= OCTET STRING

   At least one SCT MUST be included.  Server operators MAY include more
   than one SCT.

   Similarly, a certificate authority MAY submit a Precertificate to
   more than one log, and all obtained SCTs can be directly embedded in
   the final certificate, by encoding the SignedCertificateTimestampList
   structure as an ASN.1 OCTET STRING and inserting the resulting data
   in the TBSCertificate as an X.509v3 certificate extension (OID
   1.3.6.1.4.1.11129.2.4.2).  Upon receiving the certificate, clients
   can reconstruct the original TBSCertificate to verify the SCT
   signature.

   The contents of the ASN.1 OCTET STRING embedded in an OCSP extension
   or X509v3 certificate extension are as follows:

        opaque SerializedSCT<1..2^16-1>;

        struct {
            SerializedSCT sct_list <1..2^16-1>;
        } SignedCertificateTimestampList;

   Here, "SerializedSCT" is an opaque byte string that contains the
   serialized TLS structure.  This encoding ensures that TLS clients can
   decode each SCT individually (i.e., if there is a version upgrade,
   out-of-date clients can still parse old SCTs while skipping over new
   SCTs whose versions they don't understand).

   Likewise, SCTs can be embedded in a TLS extension.  See below for
   details.

   TLS clients MUST implement all three mechanisms.  Servers MUST
   implement at least one of the three mechanisms.  Note that existing
   TLS servers can generally use the certificate extension mechanism
   without modification.

   TLS servers should send SCTs from multiple logs in case one or more
   logs are not acceptable to the client (for example, if a log has been
   struck off for misbehavior or has had a key compromise). 





#### 3.3.1. TLS Extension

   The SCT can be sent during the TLS handshake using a TLS extension
   with type "signed_certificate_timestamp".

   Clients that support the extension SHOULD send a ClientHello
   extension with the appropriate type and empty "extension_data".

   Servers MUST only send SCTs to clients who have indicated support for
   the extension in the ClientHello, in which case the SCTs are sent by
   setting the "extension_data" to a "SignedCertificateTimestampList".

   Session resumption uses the original session information: clients
   SHOULD include the extension type in the ClientHello, but if the
   session is resumed, the server is not expected to process it or
   include the extension in the ServerHello.

## 6. IANA Considerations

   IANA has allocated an RFC 5246 ExtensionType value (18) for the SCT
   TLS extension.  The extension name is "signed_certificate_timestamp".

# Referenced Sections from RFC 7250: Using Raw Public Keys in Transport Layer Security (TLS) and Datagram Transport Layer Security (DTLS)

The following sections were referenced. Remaining sections are not included.

## 3. Structure of the Raw Public Key Extension

   This section defines the two TLS extensions client_certificate_type
   and server_certificate_type, which can be used as part of an extended
   TLS handshake when raw public keys are used.  Section 4 defines the
   behavior of the TLS client and the TLS server using these extensions.

   This specification uses raw public keys whereby the already available
   encoding used in a PKIX certificate in the form of a
   SubjectPublicKeyInfo structure is reused.  To carry the raw public
   key within the TLS handshake, the Certificate payload is used as a
   container, as shown in Figure 1.  The shown Certificate structure is
   an adaptation of its original form [RFC5246]. 
   opaque ASN.1Cert<1..2^24-1>;

   struct {
       select(certificate_type){

            // certificate type defined in this document.
            case RawPublicKey:
              opaque ASN.1_subjectPublicKeyInfo<1..2^24-1>;

           // X.509 certificate defined in RFC 5246           case X.509:
             ASN.1Cert certificate_list<0..2^24-1>;

           // Additional certificate type based on
           // "TLS Certificate Types" subregistry
       };
   } Certificate;

    Figure 1: Certificate Payload as a Container for the Raw Public Key

   The SubjectPublicKeyInfo structure is defined in Section 4.1 of RFC 
   5280 [PKIX ] and not only contains the raw keys, such as the public
   exponent and the modulus of an RSA public key, but also an algorithm
   identifier.  The algorithm identifier can also include parameters.
   The SubjectPublicKeyInfo value in the Certificate payload MUST
   contain the DER encoding [X.690] of the SubjectPublicKeyInfo.  The
   structure, as shown in Figure 2, therefore also contains length
   information.  An example is provided in Appendix A .

      SubjectPublicKeyInfo  ::=  SEQUENCE  {
           algorithm               AlgorithmIdentifier,
           subjectPublicKey        BIT STRING  }

      AlgorithmIdentifier   ::=  SEQUENCE  {
           algorithm               OBJECT IDENTIFIER,
           parameters              ANY DEFINED BY algorithm OPTIONAL  }

              Figure 2: SubjectPublicKeyInfo ASN.1 Structure

   The algorithm identifiers are Object Identifiers (OIDs).  RFC 3279   [RFC3279] and RFC 5480 [RFC5480], for example, define the OIDs shown
   in Figure 3.  Note that this list is not exhaustive, and more OIDs
   may be defined in future RFCs. 
   Key Type            | Document                   | OID
   --------------------+----------------------------+-------------------
   RSA                 | Section 2.3.1 of RFC 3279  | 1.2.840.113549.1.1
   ....................|............................|...................
   Digital Signature   |                            |
   Algorithm (DSA)     | Section 2.3.2 of RFC 3279  | 1.2.840.10040.4.1
   ....................|............................|...................
   Elliptic Curve      |                            |
   Digital Signature   |                            |
   Algorithm (ECDSA)   | Section 2 of RFC 5480      | 1.2.840.10045.2.1
   --------------------+----------------------------+-------------------

              Figure 3: Example Algorithm Object Identifiers

   The extension format for extended client and server hellos, which
   uses the "extension_data" field, is used to carry the
   ClientCertTypeExtension and the ServerCertTypeExtension structures.
   These two structures are shown in Figure 4.  The CertificateType
   structure is an enum with values taken from the "TLS Certificate
   Types" subregistry of the "Transport Layer Security (TLS) Extensions"
   registry [TLS-Ext-Registry ].

   struct {
           select(ClientOrServerExtension) {
               case client:
                 CertificateType client_certificate_types<1..2^8-1>;
               case server:
                 CertificateType client_certificate_type;
           }
   } ClientCertTypeExtension;

   struct {
           select(ClientOrServerExtension) {
               case client:
                 CertificateType server_certificate_types<1..2^8-1>;
               case server:
                 CertificateType server_certificate_type;
           }
   } ServerCertTypeExtension;

                   Figure 4: CertTypeExtension Structure 





### 4.2. Server Hello

   If the server receives a client hello that contains the
   client_certificate_type extension and/or the server_certificate_type
   extension, then three outcomes are possible:

   1.  The server does not support the extension defined in this
       document.  In this case, the server returns the server hello
       without the extensions defined in this document.

   2.  The server supports the extension defined in this document, but
       it does not have any certificate type in common with the client.
       Then, the server terminates the session with a fatal alert of
       type "unsupported_certificate".

   3.  The server supports the extensions defined in this document and
       has at least one certificate type in common with the client.  In
       this case, the processing rules described below are followed.

   The client_certificate_type extension in the client hello indicates
   the certificate types the client is able to provide to the server,
   when requested using a certificate_request message.  If the TLS 
   server wants to request a certificate from the client (via the
   certificate_request message), it MUST include the
   client_certificate_type extension in the server hello.  This
   client_certificate_type extension in the server hello then indicates
   the type of certificates the client is requested to provide in a
   subsequent certificate payload.  The value conveyed in the
   client_certificate_type extension MUST be selected from one of the
   values provided in the client_certificate_type extension sent in the
   client hello.  The server MUST also include a certificate_request
   payload in the server hello message.

   If the server does not send a certificate_request payload (for
   example, because client authentication happens at the application
   layer or no client authentication is required) or none of the
   certificates supported by the client (as indicated in the
   client_certificate_type extension in the client hello) match the
   server-supported certificate types, then the client_certificate_type
   payload in the server hello MUST be omitted.

   The server_certificate_type extension in the client hello indicates
   the types of certificates the client is able to process when provided
   by the server in a subsequent certificate payload.  If the client
   hello indicates support of raw public keys in the
   server_certificate_type extension and the server chooses to use raw
   public keys, then the TLS server MUST place the SubjectPublicKeyInfo
   structure into the Certificate payload.  With the
   server_certificate_type extension in the server hello, the TLS server
   indicates the certificate type carried in the Certificate payload.
   This additional indication enables avoiding parsing ambiguities since
   the Certificate payload may contain either the X.509 certificate or a
   SubjectPublicKeyInfo structure.  Note that only a single value is
   permitted in the server_certificate_type extension when carried in
   the server hello.

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





# Referenced Sections from RFC 7924: Transport Layer Security (TLS) Cached Information Extension

The following sections were referenced. Remaining sections are not included.

## 4. Exchange Specification

   Clients supporting this extension MAY include the "cached_info"
   extension in the (extended) ClientHello.  If the client includes the
   extension, then it MUST contain one or more CachedObject attributes.

   A server supporting this extension MAY include the "cached_info"
   extension in the (extended) ServerHello.  By returning the
   "cached_info" extension, the server indicates that it supports the
   cached info types.  For each indicated cached info type, the server
   MUST alter the transmission of respective payloads, according to the
   rules outlined with each type.  If the server includes the extension,
   it MUST only include CachedObjects of a type also supported by the
   client (as expressed in the ClientHello).  For example, if a client
   indicates support for 'cert' and 'cert_req', then the server cannot
   respond with a "cached_info" attribute containing support for
   ('foo-bar').

   Since the client includes a fingerprint of information it cached (for
   each indicated type), the server is able to determine whether cached
   information is stale.  If the server supports this specification and
   notices a mismatch between the data cached by the client and its own
   information, then the server MUST include the information in full and
   MUST NOT list the respective type in the "cached_info" extension.

   Note: If a server is part of a hosting environment, then the client
   may have cached multiple data items for a single server.  To allow
   the client to select the appropriate information from the cache, it
   is RECOMMENDED that the client utilizes the Server Name Indication
   (SNI) extension [RFC6066].

   Following a successful exchange of the "cached_info" extension in the
   ClientHello and ServerHello, the server alters sending the
   corresponding handshake message.  How information is altered from the
   handshake messages and for the types defined in this specification is
   defined in Sections 4.1 and 4.2, respectively. Appendix A  shows an example hash calculation, and Section 6   illustrates an example protocol exchange. 





### 4.1. Server Certificate Message

   When a ClientHello message contains the "cached_info" extension with
   a type set to 'cert', then the server MAY send the Certificate
   message shown in Figure 1 under the following conditions:

   o  The server software implements the "cached_info" extension defined
      in this specification.

   o  The 'cert' "cached_info" extension is enabled (for example, a
      policy allows the use of this extension).

   o  The server compared the value in the hash_value field of the
      client-provided "cached_info" extension with the fingerprint of
      the Certificate message it normally sends to clients.  This check
      ensures that the information cached by the client is current.  The
      procedure for calculating the fingerprint is described in Section 5.

   The original certificate handshake message syntax is defined in
   [RFC5246] and has been extended with [RFC7250].  RFC 7250 allows the
   certificate payload to contain only the SubjectPublicKeyInfo instead
   of the full information typically found in a certificate.  Hence,
   when this specification is used in combination with [RFC7250] and the
   negotiated certificate type is a raw public key, then the TLS server
   omits sending a certificate payload that contains an ASN.1
   certificate structure with the included SubjectPublicKeyInfo rather
   than the full certificate chain.  As such, this extension is
   compatible with the raw public key extension defined in RFC 7250.
   Note: We assume that the server implementation is able to select the
   appropriate certificate or SubjectPublicKeyInfo from the received
   hash value.  If the SNI extension is used by the client, then the
   server has additional information to guide the selection of the
   appropriate cached info.

   When the cached info specification is used, then a modified version
   of the Certificate message is exchanged.  The modified structure is
   shown in Figure 1.

         struct {
             opaque hash_value<1..255>;
         } Certificate;

                 Figure 1: Cached Info Certificate Message 





### 4.2. CertificateRequest Message

   When a fingerprint for an object of type 'cert_req' is provided in
   the ClientHello, the server MAY send the CertificateRequest message
   shown in Figure 2 under the following conditions:

   o  The server software implements the "cached_info" extension defined
      in this specification.

   o  The 'cert_req' "cached_info" extension is enabled (for example, a
      policy allows the use of this extension).

   o  The server compared the value in the hash_value field of the
      client-provided "cached_info" extension with the fingerprint of
      the CertificateRequest message it normally sends to clients.  This
      check ensures that the information cached by the client is
      current.  The procedure for calculating the fingerprint is
      described in Section 5.

   o  The server wants to request a certificate from the client.

   The original CertificateRequest handshake message syntax is defined
   in [RFC5246].  The modified structure of the CertificateRequest
   message is shown in Figure 2.

         struct {
             opaque hash_value<1..255>;
         } CertificateRequest;

             Figure 2: Cached Info CertificateRequest Message

   The CertificateRequest payload is the input parameter to the
   fingerprint calculation described in Section 5.

# Referenced Sections from RFC 8879: TLS Certificate Compression

The following sections were referenced. Remaining sections are not included.

## 4. Compressed Certificate Message

   If the peer has indicated that it supports compression, server and
   client MAY compress their corresponding Certificate messages
   (Section 4.4.2 of [RFC8446]) and send them in the form of the
   CompressedCertificate message (replacing the Certificate message).

   The CompressedCertificate message is formed as follows:

       struct {
            CertificateCompressionAlgorithm algorithm;
            uint24 uncompressed_length;
            opaque compressed_certificate_message<1..2^24-1>;
       } CompressedCertificate;

   algorithm:  The algorithm used to compress the certificate.  The
      algorithm MUST be one of the algorithms listed in the peer's
      compress_certificate extension.

   uncompressed_length:  The length of the Certificate message once it
      is uncompressed.  If, after decompression, the specified length
      does not match the actual length, the party receiving the invalid
      message MUST abort the connection with the "bad_certificate"
      alert.  The presence of this field allows the receiver to
      preallocate the buffer for the uncompressed Certificate message
      and enforce limits on the message size before performing
      decompression.

   compressed_certificate_message:  The result of applying the indicated
      compression algorithm to the encoded Certificate message that
      would have been sent if certificate compression was not in use.
      The compression algorithm defines how the bytes in the
      compressed_certificate_message field are converted into the
      Certificate message.

   If the specified compression algorithm is zlib, then the Certificate
   message MUST be compressed with the ZLIB compression algorithm, as
   defined in [RFC1950].  If the specified compression algorithm is
   brotli, the Certificate message MUST be compressed with the Brotli
   compression algorithm, as defined in [RFC7932].  If the specified
   compression algorithm is zstd, the Certificate message MUST be
   compressed with the Zstandard compression algorithm, as defined in
   [RFC8478].

   It is possible to define a certificate compression algorithm that
   uses a preshared dictionary to achieve a higher compression ratio.
   This document does not define any such algorithms, but additional
   codepoints may be allocated for such use per the policy in Section 7.3.

   If the received CompressedCertificate message cannot be decompressed,
   the connection MUST be terminated with the "bad_certificate" alert.

   If the format of the Certificate message is altered using the
   server_certificate_type or client_certificate_type extensions
   [RFC7250], the resulting altered message is compressed instead.

# Referenced Sections from RFC 8849: Mapping RTP Streams to Controlling Multiple Streams for Telepresence (CLUE) Media Captures

The following sections were referenced. Remaining sections are not included.

## 9. Security Considerations

   The security considerations of the RTP specification, the RTP/SAVPF
   profile, and the various RTP/RTCP extensions and RTP payload formats
   that form the complete protocol suite described in this memo apply.
   It is believed that there are no new security considerations
   resulting from the combination of these various protocol extensions.

   The "Extended Secure RTP Profile for Real-time Transport Control
   Protocol (RTCP)-Based Feedback (RTP/SAVPF)" document [RFC5124]
   provides the handling of fundamental issues by offering
   confidentiality, integrity, and partial source authentication.  A
   mandatory-to-implement and use media security solution is created by
   combining this secured RTP profile and DTLS-SRTP keying [RFC5764] as
   defined in the communication security section of this memo
   (Section 7).

   RTCP packets convey a CNAME identifier that is used to associate RTP
   packet streams that need to be synchronized across related RTP
   sessions.  Inappropriate choice of CNAME values can be a privacy
   concern, since long-term persistent CNAME identifiers can be used to
   track users across multiple calls.  The communication security
   section of this memo (Section 7) mandates the generation of short-
   term persistent RTCP CNAMEs, as specified in [RFC7022], so they can't
   be used for long-term tracking of the users.

   Some potential denial-of-service attacks exist if the RTCP reporting
   interval is configured to an inappropriate value.  This could be done
   by configuring the RTCP bandwidth fraction to an excessively large or
   small value using the SDP "b=RR:" or "b=RS:" lines [RFC3556], or some
   similar mechanism, or by choosing an excessively large or small value
   for the RTP/AVPF minimal receiver report interval (if using SDP, this
   is the "a=rtcp-fb:... trr-int" parameter) [RFC4585].  The risks are
   as follows:

   1.  The RTCP bandwidth could be configured to make the regular
       reporting interval so large that effective congestion control
       cannot be maintained, potentially leading to denial of service
       due to congestion caused by the media traffic;

   2.  The RTCP interval could be configured to a very small value,
       causing endpoints to generate high-rate RTCP traffic, which
       potentially leads to denial of service due to the non-congestion-
       controlled RTCP traffic; and

   3.  RTCP parameters could be configured differently for each
       endpoint, with some of the endpoints using a large reporting
       interval and some using a smaller interval, leading to denial of
       service due to premature participant timeouts, which are due to
       mismatched timeout periods that are based on the reporting
       interval (this is a particular concern if endpoints use a small
       but non-zero value for the RTP/AVPF minimal receiver report
       interval (trr-int) [RFC4585], as discussed in [RFC8108]).

   Premature participant timeout can be avoided by using the fixed (non-
   reduced) minimum interval when calculating the participant timeout
   [RFC8108].  To address the other concerns, endpoints SHOULD ignore
   parameters that configure the RTCP reporting interval to be
   significantly longer than the default five-second interval specified
   in [RFC3550] (unless the media data rate is so low that the longer
   reporting interval roughly corresponds to 5% of the media data rate)
   or that configure the RTCP reporting interval small enough that the
   RTCP bandwidth would exceed the media bandwidth.

   The guidelines in [RFC6562] apply when using variable bit rate (VBR)
   audio codecs such as Opus.

   Encryption of the header extensions is RECOMMENDED, unless there are
   known reasons, like RTP middleboxes performing voice-activity-based
   source selection or third-party monitoring that will greatly benefit
   from the information, and this has been expressed using API or
   signaling.  If further evidence is produced to show that information
   leakage is significant from audio level indications, then the use of
   encryption needs to be mandated at that time.

   In multi-party communication scenarios using RTP middleboxes, the
   middleboxes are REQUIRED, by this protocol, to not weaken the
   sessions' security.  The middlebox SHOULD maintain confidentiality,
   maintain integrity, and perform source authentication.  The middlebox
   MAY perform checks that prevent any endpoint participating in a
   conference to impersonate another.  Some additional security
   considerations regarding multi-party topologies can be found in
   [RFC7667].

   The CaptureID is created as part of the CLUE protocol.  The CaptId
   SDES item is used to convey the same CaptureID value in the SDES
   item.  When sending the SDES item, the security considerations
   specified in Section 6 of [RFC7941] and in the communication security
   section of this memo (see Section 7) are applicable.  Note that since
   the CaptureID is also carried in CLUE protocol messages, it is
   RECOMMENDED that this SDES item use at least similar protection
   profiles as the CLUE protocol messages carried in the CLUE data
   channel.

# Referenced Sections from RFC 9345: Delegated Credentials for TLS and DTLS

The following sections were referenced. Remaining sections are not included.

## 7. Security Considerations

   The security considerations documented in this section should be
   taken into consideration when using delegated credentials.

### 7.1. Security of Delegated Credential's Private Key

   Delegated credentials limit the exposure of the private key used in a
   (D)TLS connection by limiting its validity period.  An attacker who
   compromises the private key of a delegated credential cannot create
   new delegated credentials, but they can impersonate the compromised
   party in new TLS connections until the delegated credential expires.

   Thus, delegated credentials should not be used to send a delegation
   to an untrusted party.  Rather, they are meant to be used between
   parties that have some trust relationship with each other.  The
   secrecy of the delegated credential's private key is thus important,
   and access control mechanisms SHOULD be used to protect it, including
   file system controls, physical security, or hardware security
   modules.

### 7.2. Re-use of Delegated Credentials in Multiple Contexts

   It is not possible to use the same delegated credential for both
   client and server authentication because issuing parties compute the
   corresponding signature using a context string unique to the intended
   role (client or server).

### 7.3. Revocation of Delegated Credentials

   Delegated credentials do not provide any additional form of early
   revocation.  Since it is short-lived, the expiry of the delegated
   credential revokes the credential.  Revocation of the long-term
   private key that signs the delegated credential (from the end-entity
   certificate) also implicitly revokes the delegated credential.

### 7.4. Interactions with Session Resumption

   If a peer decides to cache the certificate chain and re-validate it
   when resuming a connection, they SHOULD also cache the associated
   delegated credential and re-validate it.  Failing to do so may result
   in resuming connections for which the delegated credential has
   expired.

### 7.5. Privacy Considerations

   Delegated credentials can be valid for 7 days (by default), and it is
   much easier for a service to create delegated credentials than a
   certificate signed by a CA.  A service could determine the client
   time and clock skew by creating several delegated credentials with
   different expiry timestamps and observing which credentials the
   client accepts.  Since client time can be unique to a particular
   client, privacy-sensitive clients who do not trust the service, such
   as browsers in incognito mode, might not want to advertise support
   for delegated credentials, or might limit the number of probes that a
   server can perform.

### 7.6. The Impact of Signature Forgery Attacks

   Delegated credentials are only used in (D)TLS 1.3 connections.
   However, the certificate that signs a delegated credential may be
   used in other contexts such as (D)TLS 1.2.  Using a certificate in
   multiple contexts opens up a potential cross-protocol attack against
   delegated credentials in (D)TLS 1.3.

   When (D)TLS 1.2 servers support RSA key exchange, they may be
   vulnerable to attacks that allow forging an RSA signature over an
   arbitrary message [BLEI ].  The TLS 1.2 specification describes a
   strategy for preventing these attacks that requires careful
   implementation of timing-resistant countermeasures.  (See Section 7.4.7.1 of [RFC5246].)

   Experience shows that, in practice, server implementations may fail
   to fully stop these attacks due to the complexity of this mitigation
   [ROBOT ].  For (D)TLS 1.2 servers that support RSA key exchange using
   a DC-enabled end-entity certificate, a hypothetical signature forgery
   attack would allow forging a signature over a delegated credential.
   The forged delegated credential could then be used by the attacker as
   the equivalent of an on-path attacker, valid for a maximum of 7 days
   (if the default valid_time is used).

   Server operators should therefore minimize the risk of using DC-
   enabled end-entity certificates where a signature forgery oracle may
   be present.  If possible, server operators may choose to use DC-
   enabled certificates only for signing credentials and not for serving
   non-DC (D)TLS traffic.  Furthermore, server operators may use
   elliptic curve certificates for DC-enabled traffic, while using RSA
   certificates without the DelegationUsage certificate extension for
   non-DC traffic; this completely prevents such attacks.

   Note that if a signature can be forged over an arbitrary credential,
   the attacker can choose any value for the valid_time field.  Repeated
   signature forgeries therefore allow the attacker to create multiple
   delegated credentials that can cover the entire validity period of
   the certificate.  Temporary exposure of the key or a signing oracle
   may allow the attacker to impersonate a server for the lifetime of
   the certificate.

# Referenced Sections from RFC 8870: Encrypted Key Transport for DTLS and Secure RTP

The following sections were referenced. Remaining sections are not included.

### 7.3. TLS Extensions

   IANA has added supported_ekt_ciphers as a new extension name to the
   "TLS ExtensionType Values" table of the "Transport Layer Security
   (TLS) Extensions" registry:

   Value:  39

   Extension Name:  supported_ekt_ciphers

   TLS 1.3:  CH, EE

   Recommended:  Y

   Reference:  RFC 8870



# Referenced Sections from RFC 9162: Certificate Transparency Version 2.0

The following sections were referenced. Remaining sections are not included.

### 6.5. transparency_info TLS Extension

   Provided that a TLS client includes the transparency_info extension
   type in the ClientHello and the TLS server supports the
   transparency_info extension:

   *  The TLS server MUST verify that the received extension_data is
      empty.

   *  The TLS server MUST construct a TransItemList of relevant
      TransItems (see Section 6.4), which SHOULD omit any TransItems
      that are already embedded in the server certificate or the stapled
      OCSP response (see Section 7.1).  If the constructed TransItemList
      is not empty, then the TLS server MUST include the
      transparency_info extension with the extension_data set to this
      TransItemList.  If the list is empty, then the server SHOULD omit
      the extension_data element but MAY send it with an empty array.

   TLS servers MUST only include this extension in the following
   messages:

   *  the ServerHello message (for TLS 1.2 or earlier)

   *  the Certificate or CertificateRequest message (for TLS 1.3)

   TLS servers MUST NOT process or include this extension when a TLS
   session is resumed, since session resumption uses the original
   session information.

# Referenced Sections from RFC 9146: Connection Identifier for DTLS 1.2

The following sections were referenced. Remaining sections are not included.

## 3. The "connection_id" Extension

   This document defines the "connection_id" extension, which is used in
   ClientHello and ServerHello messages.

   The extension type is specified as follows.

     enum {
        connection_id(54), (65535)
     } ExtensionType;

   The extension_data field of this extension, when included in the
   ClientHello, MUST contain the ConnectionId structure.  This structure
   contains the CID value the client wishes the server to use when
   sending messages to the client.  A zero-length CID value indicates
   that the client is prepared to send using a CID but does not wish the
   server to use one when sending.

     struct {
         opaque cid<0..2^8-1>;
     } ConnectionId;

   A server willing to use CIDs will respond with a "connection_id"
   extension in the ServerHello, containing the CID it wishes the client
   to use when sending messages towards it.  A zero-length value
   indicates that the server will send using the client's CID but does
   not wish the client to include a CID when sending.

   Because each party sends the value in the "connection_id" extension
   it wants to receive as a CID in encrypted records, it is possible for
   an endpoint to use a deployment-specific constant length for such
   connection identifiers.  This can in turn ease parsing and connection
   lookup -- for example, by having the length in question be a compile-
   time constant.  Such implementations MUST still be able to send CIDs
   of different lengths to other parties.  Since the CID length
   information is not included in the record itself, implementations
   that want to use variable-length CIDs are responsible for
   constructing the CID in such a way that its length can be determined
   on reception.

   In DTLS 1.2, CIDs are exchanged at the beginning of the DTLS session
   only.  There is no dedicated "CID update" message that allows new
   CIDs to be established mid-session, because DTLS 1.2 in general does
   not allow TLS 1.3-style post-handshake messages that do not
   themselves begin other handshakes.  When a DTLS session is resumed or
   renegotiated, the "connection_id" extension is negotiated afresh.

   If DTLS peers have not negotiated the use of CIDs, or a zero-length
   CID has been advertised for a given direction, then the record format
   and content type defined in RFC 6347 MUST be used to send in the
   indicated direction(s).

   If DTLS peers have negotiated the use of a non-zero-length CID for a
   given direction, then once encryption is enabled, they MUST send with
   the record format defined in Figure 3 (see Section 4) with the new
   Message Authentication Code (MAC) computation defined in Section 5   and the content type tls12_cid.  Plaintext payloads never use the new
   record format or the CID content type.

   When receiving, if the tls12_cid content type is set, then the CID is
   used to look up the connection and the security association.  If the
   tls12_cid content type is not set, then the connection and the
   security association are looked up by the 5-tuple and a check MUST be
   made to determine whether a non-zero-length CID is expected.  If a
   non-zero-length CID is expected for the retrieved association, then
   the datagram MUST be treated as invalid, as described in Section 4.1.2.1 of [RFC6347].

   When receiving a datagram with the tls12_cid content type, the new
   MAC computation defined in Section 5 MUST be used.  When receiving a
   datagram with the record format defined in RFC 6347, the MAC
   calculation defined in Section 4.1.2 of [RFC6347] MUST be used.

# Referenced Sections from RFC 8844: Unknown Key-Share Attacks on Uses of TLS with the Session Description Protocol (SDP)

The following sections were referenced. Remaining sections are not included.

#### 3.2.1. Calculating "external_id_hash" for WebRTC Identity

   A WebRTC identity assertion (Section 7 of [WEBRTC-SEC ]) is provided
   as a JSON [JSON ] object that is encoded into a JSON text.  The JSON
   text is encoded using UTF-8 [UTF8] as described by Section 8.1 of
   [JSON ].  The content of the "external_id_hash" extension is produced
   by hashing the resulting octets with SHA-256 [SHA ].  This produces
   the 32 octets of the "binding_hash" parameter, which is the sole
   contents of the extension.

   The SDP "identity" attribute includes the base64 [BASE64] encoding of
   the UTF-8 encoding of the same JSON text.  The "external_id_hash"
   extension is validated by performing base64 decoding on the value of
   the SDP "identity" attribute, hashing the resulting octets using
   SHA-256, and comparing the results with the content of the extension.
   In pseudocode form, using the "identity-assertion-value" field from
   the SDP "identity" attribute grammar as defined in [WEBRTC-SEC ]:

   external_id_hash = SHA-256(b64decode(identity-assertion-value))

   Note:  The base64 of the SDP "identity" attribute is decoded to avoid
      capturing variations in padding.  The base64-decoded identity
      assertion could include leading or trailing whitespace octets.
      WebRTC identity assertions are not canonicalized; all octets are
      hashed.

### 4.3. The external_session_id TLS Extension

   The "external_session_id" TLS extension carries the unique identifier
   that an endpoint selects.  When used with SDP, the value MUST include
   the "tls-id" attribute from the SDP that the endpoint generated when
   negotiating the session.  This document only defines use of this
   extension for SDP; other methods of external session negotiation can
   use this extension to include a unique session identifier.

   The "extension_data" for the "external_session_id" extension contains
   an ExternalSessionId struct, described below using the syntax defined
   in [TLS13]:

      struct {
         opaque session_id<20..255>;
      } ExternalSessionId;

   For SDP, the "session_id" field of the extension includes the value
   of the "tls-id" SDP attribute as defined in [DTLS-SDP ] (that is, the
   "tls-id-value" ABNF production).  The value of the "tls-id" attribute
   is encoded using ASCII [ASCII ].

   Where RTP and RTCP [RTP ] are not multiplexed, it is possible that the
   two separate DTLS connections carrying RTP and RTCP can be switched.
   This is considered benign since these protocols are designed to be
   distinguishable as SRTP [SRTP ] provides key separation.  Using RTP/
   RTCP multiplexing [RTCP-MUX ] further avoids this problem.

   The "external_session_id" extension is included in a ClientHello, and
   if the extension is present in the ClientHello, either ServerHello
   (for TLS and DTLS versions older than 1.3) or EncryptedExtensions
   (for TLS 1.3).

   Endpoints MUST check that the "session_id" parameter in the extension
   that they receive includes the "tls-id" attribute value that they
   received in their peer's session description.  Endpoints can perform
   string comparison by ASCII decoding the TLS extension value and
   comparing it to the SDP attribute value or by comparing the encoded
   TLS extension octets with the encoded SDP attribute value.  An
   endpoint that receives an "external_session_id" extension that is not
   identical to the value that it expects MUST abort the connection with
   a fatal "illegal_parameter" alert.

   The endpoint performs the validation of the "external_id_hash"
   extension in addition to the validation required by [FINGERPRINT ].

   If an endpoint communicates with a peer that does not support this
   extension, it will receive a ClientHello, ServerHello, or
   EncryptedExtensions message that does not include this extension.  An
   endpoint MAY choose to continue a session without this extension in
   order to interoperate with peers that do not implement this
   specification.

   In TLS 1.3, an "external_session_id" extension sent by a server MUST
   be sent in the EncryptedExtensions message.

   This defense is not effective if an attacker can rewrite "tls-id"
   values in signaling.  Only the mechanism in "external_id_hash" is
   able to defend against an attacker that can compromise session
   integrity.

# Referenced Sections from RFC 9001: Using TLS to Secure QUIC

The following sections were referenced. Remaining sections are not included.

### 8.2. QUIC Transport Parameters Extension

   QUIC transport parameters are carried in a TLS extension.  Different
   versions of QUIC might define a different method for negotiating
   transport configuration.

   Including transport parameters in the TLS handshake provides
   integrity protection for these values.

      enum {
         quic_transport_parameters(0x39), (65535)
      } ExtensionType;

   The extension_data field of the quic_transport_parameters extension
   contains a value that is defined by the version of QUIC that is in
   use.

   The quic_transport_parameters extension is carried in the ClientHello
   and the EncryptedExtensions messages during the handshake.  Endpoints
   MUST send the quic_transport_parameters extension; endpoints that
   receive ClientHello or EncryptedExtensions messages without the
   quic_transport_parameters extension MUST close the connection with an
   error of type 0x016d (equivalent to a fatal TLS missing_extension
   alert, see Section 4.8).

   Transport parameters become available prior to the completion of the
   handshake.  A server might use these values earlier than handshake
   completion.  However, the value of transport parameters is not
   authenticated until the handshake completes, so any use of these
   parameters cannot depend on their authenticity.  Any tampering with
   transport parameters will cause the handshake to fail.

   Endpoints MUST NOT send this extension in a TLS connection that does
   not use QUIC (such as the use of TLS with TCP defined in [TLS13]).  A
   fatal unsupported_extension alert MUST be sent by an implementation
   that supports this extension if the extension is received when the
   transport is not QUIC.

   Negotiating the quic_transport_parameters extension causes the
   EndOfEarlyData to be removed; see Section 8.3.

# Referenced Sections from RFC 9149: TLS Ticket Requests

The following sections were referenced. Remaining sections are not included.

## 3. Ticket Requests

   As discussed in Section 1, clients may want different numbers of
   tickets for new or resumed connections.  Clients may indicate to
   servers their desired number of tickets to receive on a single
   connection, in the case of a new or resumed connection, via the
   following "ticket_request" extension:

   enum {
       ticket_request(58), (65535)
   } ExtensionType;

   Clients MAY send this extension in ClientHello.  It contains the
   following structure:

   struct {
       uint8 new_session_count;
       uint8 resumption_count;
   } ClientTicketRequest;

   new_session_count:  The number of tickets desired by the client if
      the server chooses to negotiate a new connection.

   resumption_count:  The number of tickets desired by the client if the
      server is willing to resume using a ticket presented in this
      ClientHello.

   A client starting a new connection SHOULD set new_session_count to
   the desired number of session tickets and resumption_count to 0.
   Once a client's ticket cache is primed, a resumption_count of 1 is a
   good choice that allows the server to replace each ticket with a new
   ticket without over-provisioning the client with excess tickets.
   However, clients that race multiple connections and place a separate
   ticket in each will ultimately end up with just the tickets from a
   single resumed session.  In that case, clients can send a
   resumption_count equal to the number of connections they are
   attempting in parallel.  (Clients that send a resumption_count less
   than the number of parallel connection attempts might end up with
   zero tickets.)

   When a client presenting a previously obtained ticket finds that the
   server nevertheless negotiates a new connection, the client SHOULD
   assume that any other tickets associated with the same session as the
   presented ticket are also no longer valid for resumption.  This
   includes tickets obtained during the initial (new) connection and all
   tickets subsequently obtained as part of subsequent resumptions.
   Requesting more than one ticket when servers complete a new
   connection helps keep the session cache primed.

   Servers SHOULD NOT send more tickets than requested for the
   connection type selected by the server (new or resumed connection).
   Moreover, servers SHOULD place a limit on the number of tickets they
   are willing to send, whether for new or resumed connections, to save
   resources.  Therefore, the number of NewSessionTicket messages sent
   will typically be the minimum of the server's self-imposed limit and
   the number requested.  Servers MAY send additional tickets, typically
   using the same limit, if the tickets that are originally sent are
   somehow invalidated.

   A server that supports and uses a client "ticket_request" extension
   MUST also send the "ticket_request" extension in the
   EncryptedExtensions message.  It contains the following structure:

   struct {
       uint8 expected_count;
   } ServerTicketRequestHint;

   expected_count:  The number of tickets the server expects to send in
      this connection.

   Servers MUST NOT send the "ticket_request" extension in any handshake
   message, including ServerHello or HelloRetryRequest messages.  A
   client MUST abort the connection with an "illegal_parameter" alert if
   the "ticket_request" extension is present in any server handshake
   message.

   If a client receives a HelloRetryRequest, the presence (or absence)
   of the "ticket_request" extension MUST be maintained in the second
   ClientHello message.  Moreover, if this extension is present, a
   client MUST NOT change the value of ClientTicketRequest in the second
   ClientHello message.

# Referenced Sections from RFC 6347: Datagram Transport Layer Security Version 1.2

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   TLS [TLS ] is the most widely deployed protocol for securing network
   traffic.  It is widely used for protecting Web traffic and for e-mail
   protocols such as IMAP [IMAP ] and POP [POP ].  The primary advantage
   of TLS is that it provides a transparent connection-oriented channel.
   Thus, it is easy to secure an application protocol by inserting TLS
   between the application layer and the transport layer.  However, TLS
   must run over a reliable transport channel -- typically TCP [TCP ].
   Therefore, it cannot be used to secure unreliable datagram traffic.

   An increasing number of application layer protocols have been
   designed that use UDP transport.  In particular, protocols such as
   the Session Initiation Protocol (SIP) [SIP ] and electronic gaming
   protocols are increasingly popular.  (Note that SIP can run over both
   TCP and UDP, but that there are situations in which UDP is
   preferable.)  Currently, designers of these applications are faced
   with a number of unsatisfactory choices.  First, they can use IPsec
   [RFC4301].  However, for a number of reasons detailed in [WHYIPSEC ],
   this is only suitable for some applications.  Second, they can design
   a custom application layer security protocol.  Unfortunately,
   although application layer security protocols generally provide
   superior security properties (e.g., end-to-end security in the case
   of S/MIME), they typically require a large amount of effort to design
   -- in contrast to the relatively small amount of effort required to
   run the protocol over TLS.

   In many cases, the most desirable way to secure client/server
   applications would be to use TLS; however, the requirement for
   datagram semantics automatically prohibits use of TLS.  This memo
   describes a protocol for this purpose: Datagram Transport Layer
   Security (DTLS).  DTLS is deliberately designed to be as similar to
   TLS as possible, both to minimize new security invention and to
   maximize the amount of code and infrastructure reuse.

   DTLS 1.0 [DTLS1] was originally defined as a delta from [TLS11].
   This document introduces a new version of DTLS, DTLS 1.2, which is
   defined as a series of deltas to TLS 1.2 [TLS12].  There is no DTLS
   1.1; that version number was skipped in order to harmonize version
   numbers with TLS.  This version also clarifies some confusing points
   in the DTLS 1.0 specification.

   Implementations that speak both DTLS 1.2 and DTLS 1.0 can
   interoperate with those that speak only DTLS 1.0 (using DTLS 1.0 of
   course), just as TLS 1.2 implementations can interoperate with
   previous versions of TLS (see Appendix E.1 of [TLS12] for details),
   with the exception that there is no DTLS version of SSLv2 or SSLv3,
   so backward compatibility issues for those protocols do not apply. 





### 1.1. Requirements Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in RFC 2119 [REQ ].

# Referenced Sections from RFC 8017: PKCS #1: RSA Cryptography Specifications Version 2.2

The following sections were referenced. Remaining sections are not included.

## 2. Notation

   The notation in this document includes:

      c              ciphertext representative, an integer between 0 and
                     n-1

      C              ciphertext, an octet string

      d              RSA private exponent

      d_i            additional factor r_i's CRT exponent,
                     a positive integer such that

                       e * d_i == 1 (mod (r_i-1)), i = 3, ..., u

      dP             p's CRT exponent, a positive integer such that

                       e * dP == 1 (mod (p-1))

      dQ             q's CRT exponent, a positive integer such that

                       e * dQ == 1 (mod (q-1))

      e              RSA public exponent

      EM             encoded message, an octet string

      emBits         (intended) length in bits of an encoded message EM

      emLen          (intended) length in octets of an encoded message
                     EM

      GCD(. , .)     greatest common divisor of two nonnegative integers

      Hash           hash function

      hLen           output length in octets of hash function Hash

      k              length in octets of the RSA modulus n

      K              RSA private key

      L              optional RSAES-OAEP label, an octet string

      LCM(., ..., .) least common multiple of a list of nonnegative
                     integers 
      m              message representative, an integer between 0 and
                     n-1

      M              message, an octet string

      mask           MGF output, an octet string

      maskLen        (intended) length of the octet string mask

      MGF            mask generation function

      mgfSeed        seed from which mask is generated, an octet string

      mLen           length in octets of a message M

      n              RSA modulus, n = r_1 * r_2 * ... * r_u , u >= 2

      (n, e)         RSA public key

      p, q           first two prime factors of the RSA modulus n

      qInv           CRT coefficient, a positive integer less than
                     p such that q * qInv == 1 (mod p)

      r_i            prime factors of the RSA modulus n, including
                     r_1 = p, r_2 = q, and additional factors if any

      s              signature representative, an integer between 0 and
                     n-1

      S              signature, an octet string

      sLen           length in octets of the EMSA-PSS salt

      t_i            additional prime factor r_i's CRT coefficient, a
                     positive integer less than r_i such that

                       r_1 * r_2 * ... * r_(i-1) * t_i == 1 (mod r_i) ,

                     i = 3, ... , u

      u              number of prime factors of the RSA modulus, u >= 2

      x              a nonnegative integer

      X              an octet string corresponding to x

      xLen           (intended) length of the octet string X 
      0x             indicator of hexadecimal representation of an octet
                     or an octet string: "0x48" denotes the octet with
                     hexadecimal value 48; "(0x)48 09 0e" denotes the
                     string of three consecutive octets with hexadecimal
                     values 48, 09, and 0e, respectively

      \lambda(n)     LCM(r_1-1, r_2-1, ... , r_u-1)

      \xor           bit-wise exclusive-or of two octet strings

      \ceil(.)       ceiling function; \ceil(x) is the smallest integer
                     larger than or equal to the real number x

      ||             concatenation operator

      ==             congruence symbol; a == b (mod n) means that the
                     integer n divides the integer a - b

   Note: The Chinese Remainder Theorem (CRT) can be applied in a non-
   recursive as well as a recursive way.  In this document, a recursive
   approach following Garner's algorithm [GARNER ] is used.  See also
   Note 1 in Section 3.2.

#### A.2.4. RSASSA-PKCS-v1_5

   The object identifier for RSASSA-PKCS1-v1_5 SHALL be one of the
   following.  The choice of OID depends on the choice of hash
   algorithm: MD2, MD5, SHA-1, SHA-224, SHA-256, SHA-384, SHA-512,
   SHA-512/224, or SHA-512/256.  Note that if either MD2 or MD5 is used,
   then the OID is just as in PKCS #1 v1.5.  For each OID, the
   parameters field associated with this OID in a value of type
   AlgorithmIdentifier SHALL have a value of type NULL.  The OID should
   be chosen in accordance with the following table:

         Hash algorithm   OID
         ------------------------------------------------------------
         MD2              md2WithRSAEncryption        ::= {pkcs-1 2}
         MD5              md5WithRSAEncryption        ::= {pkcs-1 4}
         SHA-1            sha1WithRSAEncryption       ::= {pkcs-1 5}
         SHA-256          sha224WithRSAEncryption     ::= {pkcs-1 14}
         SHA-256          sha256WithRSAEncryption     ::= {pkcs-1 11}
         SHA-384          sha384WithRSAEncryption     ::= {pkcs-1 12}
         SHA-512          sha512WithRSAEncryption     ::= {pkcs-1 13}
         SHA-512/224      sha512-224WithRSAEncryption ::= {pkcs-1 15}
         SHA-512/256      sha512-256WithRSAEncryption ::= {pkcs-1 16}

   The EMSA-PKCS1-v1_5 encoding method includes an ASN.1 value of type
   DigestInfo, where the type DigestInfo has the syntax

       DigestInfo ::= SEQUENCE {
           digestAlgorithm DigestAlgorithm,
           digest OCTET STRING
       }

   digestAlgorithm identifies the hash function and SHALL be an
   algorithm ID with an OID in the set PKCS1-v1-5DigestAlgorithms.  For
   a discussion of supported hash functions, see Appendix B.1. 
       DigestAlgorithm ::= AlgorithmIdentifier {
          {PKCS1-v1-5DigestAlgorithms}
       }

       PKCS1-v1-5DigestAlgorithms    ALGORITHM-IDENTIFIER ::= {
           { OID id-md2        PARAMETERS NULL }|
           { OID id-md5        PARAMETERS NULL }|
           { OID id-sha1       PARAMETERS NULL }|
           { OID id-sha224     PARAMETERS NULL }|
           { OID id-sha256     PARAMETERS NULL }|
           { OID id-sha384     PARAMETERS NULL }|
           { OID id-sha512     PARAMETERS NULL }|
           { OID id-sha512-224 PARAMETERS NULL }|
           { OID id-sha512-256 PARAMETERS NULL }
       }

# Referenced Sections from RFC 4492: Elliptic Curve Cryptography (ECC) Cipher Suites for Transport Layer Security (TLS)

The following sections were referenced. Remaining sections are not included.

## 7. Security Considerations

   Security issues are discussed throughout this memo.

   For TLS handshakes using ECC cipher suites, the security
   considerations in appendices D.2 and D.3 of [2] and [3] apply
   accordingly.

   Security discussions specific to ECC can be found in [6] and [7].
   One important issue that implementers and users must consider is
   elliptic curve selection.  Guidance on selecting an appropriate
   elliptic curve size is given in Table 1.

   Beyond elliptic curve size, the main issue is elliptic curve
   structure.  As a general principle, it is more conservative to use
   elliptic curves with as little algebraic structure as possible.
   Thus, random curves are more conservative than special curves such as
   Koblitz curves, and curves over F_p with p random are more
   conservative than curves over F_p with p of a special form (and
   curves over F_p with p random might be considered more conservative
   than curves over F_2^m as there is no choice between multiple fields
   of similar size for characteristic 2).  Note, however, that algebraic
   structure can also lead to implementation efficiencies, and
   implementers and users may, therefore, need to balance conservatism
   against a need for efficiency.  Concrete attacks are known against
   only very few special classes of curves, such as supersingular
   curves, and these classes are excluded from the ECC standards that
   this document references [6], [7].

   Another issue is the potential for catastrophic failures when a
   single elliptic curve is widely used.  In this case, an attack on the
   elliptic curve might result in the compromise of a large number of
   keys.  Again, this concern may need to be balanced against efficiency
   and interoperability improvements associated with widely-used curves.
   Substantial additional information on elliptic curve choice can be
   found in [5], [6], [7], and [11]. 
   Implementers and users must also consider whether they need forward
   secrecy.  Forward secrecy refers to the property that session keys
   are not compromised if the static, certified keys belonging to the
   server and client are compromised.  The ECDHE_ECDSA and ECDHE_RSA key
   exchange algorithms provide forward secrecy protection in the event
   of server key compromise, while ECDH_ECDSA and ECDH_RSA do not.
   Similarly, if the client is providing a static, certified key,
   ECDSA_sign client authentication provides forward secrecy protection
   in the event of client key compromise, while ECDSA_fixed_ECDH and
   RSA_fixed_ECDH do not.  Thus, to obtain complete forward secrecy
   protection, ECDHE_ECDSA or ECDHE_RSA must be used for key exchange,
   with ECDSA_sign used for client authentication if necessary.  Here
   again the security benefits of forward secrecy may need to be
   balanced against the improved efficiency offered by other options.

# Referenced Sections from RFC 5280: Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile

The following sections were referenced. Remaining sections are not included.

### 3.1. X.509 Version 3 Certificate

   Users of a public key require confidence that the associated private
   key is owned by the correct remote subject (person or system) with
   which an encryption or digital signature mechanism will be used.
   This confidence is obtained through the use of public key
   certificates, which are data structures that bind public key values
   to subjects.  The binding is asserted by having a trusted CA
   digitally sign each certificate.  The CA may base this assertion upon
   technical means (a.k.a., proof of possession through a challenge-
   response protocol), presentation of the private key, or on an
   assertion by the subject.  A certificate has a limited valid
   lifetime, which is indicated in its signed contents.  Because a
   certificate's signature and timeliness can be independently checked
   by a certificate-using client, certificates can be distributed via 
   untrusted communications and server systems, and can be cached in
   unsecured storage in certificate-using systems.

   ITU-T X.509 (formerly CCITT X.509) or ISO/IEC 9594-8, which was first
   published in 1988 as part of the X.500 directory recommendations,
   defines a standard certificate format [X.509].  The certificate
   format in the 1988 standard is called the version 1 (v1) format.
   When X.500 was revised in 1993, two more fields were added, resulting
   in the version 2 (v2) format.

   The Internet Privacy Enhanced Mail (PEM) RFCs, published in 1993,
   include specifications for a public key infrastructure based on X.509
   v1 certificates [RFC1422].  The experience gained in attempts to
   deploy RFC 1422 made it clear that the v1 and v2 certificate formats
   were deficient in several respects.  Most importantly, more fields
   were needed to carry information that PEM design and implementation
   experience had proven necessary.  In response to these new
   requirements, the ISO/IEC, ITU-T, and ANSI X9 developed the X.509
   version 3 (v3) certificate format.  The v3 format extends the v2
   format by adding provision for additional extension fields.
   Particular extension field types may be specified in standards or may
   be defined and registered by any organization or community.  In June
   1996, standardization of the basic v3 format was completed [X.509].

   ISO/IEC, ITU-T, and ANSI X9 have also developed standard extensions
   for use in the v3 extensions field [X.509][X9.55].  These extensions
   can convey such data as additional subject identification
   information, key attribute information, policy information, and
   certification path constraints.

   However, the ISO/IEC, ITU-T, and ANSI X9 standard extensions are very
   broad in their applicability.  In order to develop interoperable
   implementations of X.509 v3 systems for Internet use, it is necessary
   to specify a profile for use of the X.509 v3 extensions tailored for
   the Internet.  It is one goal of this document to specify a profile
   for Internet WWW, electronic mail, and IPsec applications.
   Environments with additional requirements may build on this profile
   or may replace it.

### 3.2. Certification Paths and Trust

   A user of a security service requiring knowledge of a public key
   generally needs to obtain and validate a certificate containing the
   required public key.  If the public key user does not already hold an
   assured copy of the public key of the CA that signed the certificate,
   the CA's name, and related information (such as the validity period
   or name constraints), then it might need an additional certificate to
   obtain that public key.  In general, a chain of multiple certificates 
   may be needed, comprising a certificate of the public key owner (the
   end entity) signed by one CA, and zero or more additional
   certificates of CAs signed by other CAs.  Such chains, called
   certification paths, are required because a public key user is only
   initialized with a limited number of assured CA public keys.

   There are different ways in which CAs might be configured in order
   for public key users to be able to find certification paths.  For
   PEM, RFC 1422 defined a rigid hierarchical structure of CAs.  There
   are three types of PEM certification authority:

      (a)  Internet Policy Registration Authority (IPRA):  This
           authority, operated under the auspices of the Internet
           Society, acts as the root of the PEM certification hierarchy
           at level 1.  It issues certificates only for the next level
           of authorities, PCAs.  All certification paths start with the
           IPRA.

      (b)  Policy Certification Authorities (PCAs):  PCAs are at level 2
           of the hierarchy, each PCA being certified by the IPRA.  A
           PCA shall establish and publish a statement of its policy
           with respect to certifying users or subordinate certification
           authorities.  Distinct PCAs aim to satisfy different user
           needs.  For example, one PCA (an organizational PCA) might
           support the general electronic mail needs of commercial
           organizations, and another PCA (a high-assurance PCA) might
           have a more stringent policy designed for satisfying legally
           binding digital signature requirements.

      (c)  Certification Authorities (CAs):  CAs are at level 3 of the
           hierarchy and can also be at lower levels.  Those at level 3
           are certified by PCAs.  CAs represent, for example,
           particular organizations, particular organizational units
           (e.g., departments, groups, sections), or particular
           geographical areas. RFC 1422 furthermore has a name subordination rule, which requires
   that a CA can only issue certificates for entities whose names are
   subordinate (in the X.500 naming tree) to the name of the CA itself.
   The trust associated with a PEM certification path is implied by the
   PCA name.  The name subordination rule ensures that CAs below the PCA
   are sensibly constrained as to the set of subordinate entities they
   can certify (e.g., a CA for an organization can only certify entities
   in that organization's name tree).  Certificate user systems are able
   to mechanically check that the name subordination rule has been
   followed. 



   RFC 1422 uses the X.509 v1 certificate format.  The limitations of
   X.509 v1 required imposition of several structural restrictions to
   clearly associate policy information or restrict the utility of
   certificates.  These restrictions included:

      (a)  a pure top-down hierarchy, with all certification paths
           starting from IPRA;

      (b)  a naming subordination rule restricting the names of a CA's
           subjects; and

      (c)  use of the PCA concept, which requires knowledge of
           individual PCAs to be built into certificate chain
           verification logic.  Knowledge of individual PCAs was
           required to determine if a chain could be accepted.

   With X.509 v3, most of the requirements addressed by RFC 1422 can be
   addressed using certificate extensions, without a need to restrict
   the CA structures used.  In particular, the certificate extensions
   relating to certificate policies obviate the need for PCAs and the
   constraint extensions obviate the need for the name subordination
   rule.  As a result, this document supports a more flexible
   architecture, including:

      (a)  Certification paths start with a public key of a CA in a
           user's own domain, or with the public key of the top of a
           hierarchy.  Starting with the public key of a CA in a user's
           own domain has certain advantages.  In some environments, the
           local domain is the most trusted.

      (b)  Name constraints may be imposed through explicit inclusion of
           a name constraints extension in a certificate, but are not
           required.

      (c)  Policy extensions and policy mappings replace the PCA
           concept, which permits a greater degree of automation.  The
           application can determine if the certification path is
           acceptable based on the contents of the certificates instead
           of a priori knowledge of PCAs.  This permits automation of
           certification path processing.

   X.509 v3 also includes an extension that identifies the subject of a
   certificate as being either a CA or an end entity, reducing the
   reliance on out-of-band information demanded in PEM.

   This specification covers two classes of certificates: CA
   certificates and end entity certificates.  CA certificates may be
   further divided into three classes: cross-certificates, self-issued 
   certificates, and self-signed certificates.  Cross-certificates are
   CA certificates in which the issuer and subject are different
   entities.  Cross-certificates describe a trust relationship between
   the two CAs.  Self-issued certificates are CA certificates in which
   the issuer and subject are the same entity.  Self-issued certificates
   are generated to support changes in policy or operations.  Self-
   signed certificates are self-issued certificates where the digital
   signature may be verified by the public key bound into the
   certificate.  Self-signed certificates are used to convey a public
   key for use to begin certification paths.  End entity certificates
   are issued to subjects that are not authorized to issue certificates.

### 4.2. Certificate Extensions

   The extensions defined for X.509 v3 certificates provide methods for
   associating additional attributes with users or public keys and for
   managing relationships between CAs.  The X.509 v3 certificate format
   also allows communities to define private extensions to carry
   information unique to those communities.  Each extension in a
   certificate is designated as either critical or non-critical.  A
   certificate-using system MUST reject the certificate if it encounters
   a critical extension it does not recognize or a critical extension
   that contains information that it cannot process.  A non-critical
   extension MAY be ignored if it is not recognized, but MUST be
   processed if it is recognized.  The following sections present
   recommended extensions used within Internet certificates and standard
   locations for information.  Communities may elect to use additional
   extensions; however, caution ought to be exercised in adopting any
   critical extensions in certificates that might prevent use in a
   general context.

   Each extension includes an OID and an ASN.1 structure.  When an
   extension appears in a certificate, the OID appears as the field
   extnID and the corresponding ASN.1 DER encoded structure is the value
   of the octet string extnValue.  A certificate MUST NOT include more
   than one instance of a particular extension.  For example, a
   certificate may contain only one authority key identifier extension
   (Section 4.2.1.1).  An extension includes the boolean critical, with
   a default value of FALSE.  The text for each extension specifies the
   acceptable values for the critical field for CAs conforming to this
   profile.

   Conforming CAs MUST support key identifiers (Sections 4.2.1.1 and
   4.2.1.2), basic constraints (Section 4.2.1.9), key usage (Section 
   4.2.1.3), and certificate policies (Section 4.2.1.4) extensions.  If
   the CA issues certificates with an empty sequence for the subject
   field, the CA MUST support the subject alternative name extension
   (Section 4.2.1.6).  Support for the remaining extensions is OPTIONAL.
   Conforming CAs MAY support extensions that are not identified within 
   this specification; certificate issuers are cautioned that marking
   such extensions as critical may inhibit interoperability.

   At a minimum, applications conforming to this profile MUST recognize
   the following extensions: key usage (Section 4.2.1.3), certificate
   policies (Section 4.2.1.4), subject alternative name (Section 
   4.2.1.6), basic constraints (Section 4.2.1.9), name constraints
   (Section 4.2.1.10), policy constraints (Section 4.2.1.11), extended
   key usage (Section 4.2.1.12), and inhibit anyPolicy (Section 
   4.2.1.14).

   In addition, applications conforming to this profile SHOULD recognize
   the authority and subject key identifier (Sections 4.2.1.1 and
   4.2.1.2) and policy mappings (Section 4.2.1.5) extensions.

#### 4.2.1. Standard Extensions

   This section identifies standard certificate extensions defined in
   [X.509] for use in the Internet PKI.  Each extension is associated
   with an OID defined in [X.509].  These OIDs are members of the id-ce
   arc, which is defined by the following:

   id-ce   OBJECT IDENTIFIER ::=  { joint-iso-ccitt(2) ds(5) 29 }

##### 4.2.1.1. Authority Key Identifier

   The authority key identifier extension provides a means of
   identifying the public key corresponding to the private key used to
   sign a certificate.  This extension is used where an issuer has
   multiple signing keys (either due to multiple concurrent key pairs or
   due to changeover).  The identification MAY be based on either the
   key identifier (the subject key identifier in the issuer's
   certificate) or the issuer name and serial number.

   The keyIdentifier field of the authorityKeyIdentifier extension MUST
   be included in all certificates generated by conforming CAs to
   facilitate certification path construction.  There is one exception;
   where a CA distributes its public key in the form of a "self-signed"
   certificate, the authority key identifier MAY be omitted.  The
   signature on a self-signed certificate is generated with the private
   key associated with the certificate's subject public key.  (This
   proves that the issuer possesses both the public and private keys.)
   In this case, the subject and authority key identifiers would be
   identical, but only the subject key identifier is needed for
   certification path building.

   The value of the keyIdentifier field SHOULD be derived from the
   public key used to verify the certificate's signature or a method 
   that generates unique values.  Two common methods for generating key
   identifiers from the public key are described in Section 4.2.1.2.
   Where a key identifier has not been previously established, this
   specification RECOMMENDS use of one of these methods for generating
   keyIdentifiers or use of a similar method that uses a different hash
   algorithm.  Where a key identifier has been previously established,
   the CA SHOULD use the previously established identifier.

   This profile RECOMMENDS support for the key identifier method by all
   certificate users.

   Conforming CAs MUST mark this extension as non-critical.

   id-ce-authorityKeyIdentifier OBJECT IDENTIFIER ::=  { id-ce 35 }

   AuthorityKeyIdentifier ::= SEQUENCE {
      keyIdentifier             [0] KeyIdentifier           OPTIONAL,
      authorityCertIssuer       [1] GeneralNames            OPTIONAL,
      authorityCertSerialNumber [2] CertificateSerialNumber OPTIONAL  }

   KeyIdentifier ::= OCTET STRING

##### 4.2.1.2. Subject Key Identifier

   The subject key identifier extension provides a means of identifying
   certificates that contain a particular public key.

   To facilitate certification path construction, this extension MUST
   appear in all conforming CA certificates, that is, all certificates
   including the basic constraints extension (Section 4.2.1.9) where the
   value of cA is TRUE.  In conforming CA certificates, the value of the
   subject key identifier MUST be the value placed in the key identifier
   field of the authority key identifier extension (Section 4.2.1.1) of
   certificates issued by the subject of this certificate.  Applications
   are not required to verify that key identifiers match when performing
   certification path validation.

   For CA certificates, subject key identifiers SHOULD be derived from
   the public key or a method that generates unique values.  Two common
   methods for generating key identifiers from the public key are:

      (1) The keyIdentifier is composed of the 160-bit SHA-1 hash of the
           value of the BIT STRING subjectPublicKey (excluding the tag,
           length, and number of unused bits). 
      (2) The keyIdentifier is composed of a four-bit type field with
           the value 0100 followed by the least significant 60 bits of
           the SHA-1 hash of the value of the BIT STRING
           subjectPublicKey (excluding the tag, length, and number of
           unused bits).

   Other methods of generating unique numbers are also acceptable.

   For end entity certificates, the subject key identifier extension
   provides a means for identifying certificates containing the
   particular public key used in an application.  Where an end entity
   has obtained multiple certificates, especially from multiple CAs, the
   subject key identifier provides a means to quickly identify the set
   of certificates containing a particular public key.  To assist
   applications in identifying the appropriate end entity certificate,
   this extension SHOULD be included in all end entity certificates.

   For end entity certificates, subject key identifiers SHOULD be
   derived from the public key.  Two common methods for generating key
   identifiers from the public key are identified above.

   Where a key identifier has not been previously established, this
   specification RECOMMENDS use of one of these methods for generating
   keyIdentifiers or use of a similar method that uses a different hash
   algorithm.  Where a key identifier has been previously established,
   the CA SHOULD use the previously established identifier.

   Conforming CAs MUST mark this extension as non-critical.

   id-ce-subjectKeyIdentifier OBJECT IDENTIFIER ::=  { id-ce 14 }

   SubjectKeyIdentifier ::= KeyIdentifier

##### 4.2.1.3. Key Usage

   The key usage extension defines the purpose (e.g., encipherment,
   signature, certificate signing) of the key contained in the
   certificate.  The usage restriction might be employed when a key that
   could be used for more than one operation is to be restricted.  For
   example, when an RSA key should be used only to verify signatures on
   objects other than public key certificates and CRLs, the
   digitalSignature and/or nonRepudiation bits would be asserted.
   Likewise, when an RSA key should be used only for key management, the
   keyEncipherment bit would be asserted. 
   Conforming CAs MUST include this extension in certificates that
   contain public keys that are used to validate digital signatures on
   other public key certificates or CRLs.  When present, conforming CAs
   SHOULD mark this extension as critical.

      id-ce-keyUsage OBJECT IDENTIFIER ::=  { id-ce 15 }

      KeyUsage ::= BIT STRING {
           digitalSignature        (0),
           nonRepudiation          (1), -- recent editions of X.509 have
                                -- renamed this bit to contentCommitment
           keyEncipherment         (2),
           dataEncipherment        (3),
           keyAgreement            (4),
           keyCertSign             (5),
           cRLSign                 (6),
           encipherOnly            (7),
           decipherOnly            (8) }

   Bits in the KeyUsage type are used as follows:

      The digitalSignature bit is asserted when the subject public key
      is used for verifying digital signatures, other than signatures on
      certificates (bit 5) and CRLs (bit 6), such as those used in an
      entity authentication service, a data origin authentication
      service, and/or an integrity service.

      The nonRepudiation bit is asserted when the subject public key is
      used to verify digital signatures, other than signatures on
      certificates (bit 5) and CRLs (bit 6), used to provide a non-
      repudiation service that protects against the signing entity
      falsely denying some action.  In the case of later conflict, a
      reliable third party may determine the authenticity of the signed
      data.  (Note that recent editions of X.509 have renamed the
      nonRepudiation bit to contentCommitment.)

      The keyEncipherment bit is asserted when the subject public key is
      used for enciphering private or secret keys, i.e., for key
      transport.  For example, this bit shall be set when an RSA public
      key is to be used for encrypting a symmetric content-decryption
      key or an asymmetric private key.

      The dataEncipherment bit is asserted when the subject public key
      is used for directly enciphering raw user data without the use of
      an intermediate symmetric cipher.  Note that the use of this bit
      is extremely uncommon; almost all applications use key transport
      or key agreement to establish a symmetric key. 
      The keyAgreement bit is asserted when the subject public key is
      used for key agreement.  For example, when a Diffie-Hellman key is
      to be used for key management, then this bit is set.

      The keyCertSign bit is asserted when the subject public key is
      used for verifying signatures on public key certificates.  If the
      keyCertSign bit is asserted, then the cA bit in the basic
      constraints extension (Section 4.2.1.9) MUST also be asserted.

      The cRLSign bit is asserted when the subject public key is used
      for verifying signatures on certificate revocation lists (e.g.,
      CRLs, delta CRLs, or ARLs).

      The meaning of the encipherOnly bit is undefined in the absence of
      the keyAgreement bit.  When the encipherOnly bit is asserted and
      the keyAgreement bit is also set, the subject public key may be
      used only for enciphering data while performing key agreement.

      The meaning of the decipherOnly bit is undefined in the absence of
      the keyAgreement bit.  When the decipherOnly bit is asserted and
      the keyAgreement bit is also set, the subject public key may be
      used only for deciphering data while performing key agreement.

   If the keyUsage extension is present, then the subject public key
   MUST NOT be used to verify signatures on certificates or CRLs unless
   the corresponding keyCertSign or cRLSign bit is set.  If the subject
   public key is only to be used for verifying signatures on
   certificates and/or CRLs, then the digitalSignature and
   nonRepudiation bits SHOULD NOT be set.  However, the digitalSignature
   and/or nonRepudiation bits MAY be set in addition to the keyCertSign
   and/or cRLSign bits if the subject public key is to be used to verify
   signatures on certificates and/or CRLs as well as other objects.

   Combining the nonRepudiation bit in the keyUsage certificate
   extension with other keyUsage bits may have security implications
   depending on the context in which the certificate is to be used.
   Further distinctions between the digitalSignature and nonRepudiation
   bits may be provided in specific certificate policies.

   This profile does not restrict the combinations of bits that may be
   set in an instantiation of the keyUsage extension.  However,
   appropriate values for keyUsage extensions for particular algorithms
   are specified in [RFC3279], [RFC4055], and [RFC4491].  When the
   keyUsage extension appears in a certificate, at least one of the bits
   MUST be set to 1. 





##### 4.2.1.4. Certificate Policies

   The certificate policies extension contains a sequence of one or more
   policy information terms, each of which consists of an object
   identifier (OID) and optional qualifiers.  Optional qualifiers, which
   MAY be present, are not expected to change the definition of the
   policy.  A certificate policy OID MUST NOT appear more than once in a
   certificate policies extension.

   In an end entity certificate, these policy information terms indicate
   the policy under which the certificate has been issued and the
   purposes for which the certificate may be used.  In a CA certificate,
   these policy information terms limit the set of policies for
   certification paths that include this certificate.  When a CA does
   not wish to limit the set of policies for certification paths that
   include this certificate, it MAY assert the special policy anyPolicy,
   with a value of { 2 5 29 32 0 }.

   Applications with specific policy requirements are expected to have a
   list of those policies that they will accept and to compare the
   policy OIDs in the certificate to that list.  If this extension is
   critical, the path validation software MUST be able to interpret this
   extension (including the optional qualifier), or MUST reject the
   certificate.

   To promote interoperability, this profile RECOMMENDS that policy
   information terms consist of only an OID.  Where an OID alone is
   insufficient, this profile strongly recommends that the use of
   qualifiers be limited to those identified in this section.  When
   qualifiers are used with the special policy anyPolicy, they MUST be
   limited to the qualifiers identified in this section.  Only those
   qualifiers returned as a result of path validation are considered.

   This specification defines two policy qualifier types for use by
   certificate policy writers and certificate issuers.  The qualifier
   types are the CPS Pointer and User Notice qualifiers.

   The CPS Pointer qualifier contains a pointer to a Certification
   Practice Statement (CPS) published by the CA.  The pointer is in the
   form of a URI.  Processing requirements for this qualifier are a
   local matter.  No action is mandated by this specification regardless
   of the criticality value asserted for the extension.

   User notice is intended for display to a relying party when a
   certificate is used.  Only user notices returned as a result of path
   validation are intended for display to the user.  If a notice is 
   duplicated, only one copy need be displayed.  To prevent such
   duplication, this qualifier SHOULD only be present in end entity
   certificates and CA certificates issued to other organizations.

   The user notice has two optional fields: the noticeRef field and the
   explicitText field.  Conforming CAs SHOULD NOT use the noticeRef
   option.

      The noticeRef field, if used, names an organization and
      identifies, by number, a particular textual statement prepared by
      that organization.  For example, it might identify the
      organization "CertsRUs" and notice number 1.  In a typical
      implementation, the application software will have a notice file
      containing the current set of notices for CertsRUs; the
      application will extract the notice text from the file and display
      it.  Messages MAY be multilingual, allowing the software to select
      the particular language message for its own environment.

      An explicitText field includes the textual statement directly in
      the certificate.  The explicitText field is a string with a
      maximum size of 200 characters.  Conforming CAs SHOULD use the
      UTF8String encoding for explicitText, but MAY use IA5String.
      Conforming CAs MUST NOT encode explicitText as VisibleString or
      BMPString.  The explicitText string SHOULD NOT include any control
      characters (e.g., U+0000 to U+001F and U+007F to U+009F).  When
      the UTF8String encoding is used, all character sequences SHOULD be
      normalized according to Unicode normalization form C (NFC) [NFC ].

   If both the noticeRef and explicitText options are included in the
   one qualifier and if the application software can locate the notice
   text indicated by the noticeRef option, then that text SHOULD be
   displayed; otherwise, the explicitText string SHOULD be displayed.

   Note: While the explicitText has a maximum size of 200 characters,
   some non-conforming CAs exceed this limit.  Therefore, certificate
   users SHOULD gracefully handle explicitText with more than 200
   characters. 
   id-ce-certificatePolicies OBJECT IDENTIFIER ::=  { id-ce 32 }

   anyPolicy OBJECT IDENTIFIER ::= { id-ce-certificatePolicies 0 }

   certificatePolicies ::= SEQUENCE SIZE (1..MAX) OF PolicyInformation

   PolicyInformation ::= SEQUENCE {
        policyIdentifier   CertPolicyId,
        policyQualifiers   SEQUENCE SIZE (1..MAX) OF
                                PolicyQualifierInfo OPTIONAL }

   CertPolicyId ::= OBJECT IDENTIFIER

   PolicyQualifierInfo ::= SEQUENCE {
        policyQualifierId  PolicyQualifierId,
        qualifier          ANY DEFINED BY policyQualifierId }

   -- policyQualifierIds for Internet policy qualifiers

   id-qt          OBJECT IDENTIFIER ::=  { id-pkix 2 }
   id-qt-cps      OBJECT IDENTIFIER ::=  { id-qt 1 }
   id-qt-unotice  OBJECT IDENTIFIER ::=  { id-qt 2 }

   PolicyQualifierId ::= OBJECT IDENTIFIER ( id-qt-cps | id-qt-unotice )

   Qualifier ::= CHOICE {
        cPSuri           CPSuri,
        userNotice       UserNotice }

   CPSuri ::= IA5String

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





##### 4.2.1.5. Policy Mappings

   This extension is used in CA certificates.  It lists one or more
   pairs of OIDs; each pair includes an issuerDomainPolicy and a
   subjectDomainPolicy.  The pairing indicates the issuing CA considers
   its issuerDomainPolicy equivalent to the subject CA's
   subjectDomainPolicy.

   The issuing CA's users might accept an issuerDomainPolicy for certain
   applications.  The policy mapping defines the list of policies
   associated with the subject CA that may be accepted as comparable to
   the issuerDomainPolicy.

   Each issuerDomainPolicy named in the policy mappings extension SHOULD
   also be asserted in a certificate policies extension in the same
   certificate.  Policies MUST NOT be mapped either to or from the
   special value anyPolicy (Section 4.2.1.4).

   In general, certificate policies that appear in the
   issuerDomainPolicy field of the policy mappings extension are not
   considered acceptable policies for inclusion in subsequent
   certificates in the certification path.  In some circumstances, a CA
   may wish to map from one policy (p1) to another (p2), but still wants
   the issuerDomainPolicy (p1) to be considered acceptable for inclusion
   in subsequent certificates.  This may occur, for example, if the CA
   is in the process of transitioning from the use of policy p1 to the
   use of policy p2 and has valid certificates that were issued under
   each of the policies.  A CA may indicate this by including two policy
   mappings in the CA certificates that it issues.  Each policy mapping
   would have an issuerDomainPolicy of p1; one policy mapping would have
   a subjectDomainPolicy of p1 and the other would have a
   subjectDomainPolicy of p2.

   This extension MAY be supported by CAs and/or applications.
   Conforming CAs SHOULD mark this extension as critical.

   id-ce-policyMappings OBJECT IDENTIFIER ::=  { id-ce 33 }

   PolicyMappings ::= SEQUENCE SIZE (1..MAX) OF SEQUENCE {
        issuerDomainPolicy      CertPolicyId,
        subjectDomainPolicy     CertPolicyId }

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

##### 4.2.1.7. Issuer Alternative Name

   As with Section 4.2.1.6, this extension is used to associate Internet
   style identities with the certificate issuer.  Issuer alternative
   name MUST be encoded as in 4.2.1.6.  Issuer alternative names are not
   processed as part of the certification path validation algorithm in Section 6.  (That is, issuer alternative names are not used in name
   chaining and name constraints are not enforced.)

   Where present, conforming CAs SHOULD mark this extension as non-
   critical.

   id-ce-issuerAltName OBJECT IDENTIFIER ::=  { id-ce 18 }

   IssuerAltName ::= GeneralNames 





##### 4.2.1.8. Subject Directory Attributes

   The subject directory attributes extension is used to convey
   identification attributes (e.g., nationality) of the subject.  The
   extension is defined as a sequence of one or more attributes.
   Conforming CAs MUST mark this extension as non-critical.

   id-ce-subjectDirectoryAttributes OBJECT IDENTIFIER ::=  { id-ce 9 }

   SubjectDirectoryAttributes ::= SEQUENCE SIZE (1..MAX) OF Attribute

##### 4.2.1.9. Basic Constraints

   The basic constraints extension identifies whether the subject of the
   certificate is a CA and the maximum depth of valid certification
   paths that include this certificate.

   The cA boolean indicates whether the certified public key may be used
   to verify certificate signatures.  If the cA boolean is not asserted,
   then the keyCertSign bit in the key usage extension MUST NOT be
   asserted.  If the basic constraints extension is not present in a
   version 3 certificate, or the extension is present but the cA boolean
   is not asserted, then the certified public key MUST NOT be used to
   verify certificate signatures.

   The pathLenConstraint field is meaningful only if the cA boolean is
   asserted and the key usage extension, if present, asserts the
   keyCertSign bit (Section 4.2.1.3).  In this case, it gives the
   maximum number of non-self-issued intermediate certificates that may
   follow this certificate in a valid certification path.  (Note: The
   last certificate in the certification path is not an intermediate
   certificate, and is not included in this limit.  Usually, the last
   certificate is an end entity certificate, but it can be a CA
   certificate.)  A pathLenConstraint of zero indicates that no non-
   self-issued intermediate CA certificates may follow in a valid
   certification path.  Where it appears, the pathLenConstraint field
   MUST be greater than or equal to zero.  Where pathLenConstraint does
   not appear, no limit is imposed.

   Conforming CAs MUST include this extension in all CA certificates
   that contain public keys used to validate digital signatures on
   certificates and MUST mark the extension as critical in such
   certificates.  This extension MAY appear as a critical or non-
   critical extension in CA certificates that contain public keys used
   exclusively for purposes other than validating digital signatures on
   certificates.  Such CA certificates include ones that contain public
   keys used exclusively for validating digital signatures on CRLs and
   ones that contain key management public keys used with certificate 
   enrollment protocols.  This extension MAY appear as a critical or
   non-critical extension in end entity certificates.

   CAs MUST NOT include the pathLenConstraint field unless the cA
   boolean is asserted and the key usage extension asserts the
   keyCertSign bit.

   id-ce-basicConstraints OBJECT IDENTIFIER ::=  { id-ce 19 }

   BasicConstraints ::= SEQUENCE {
        cA                      BOOLEAN DEFAULT FALSE,
        pathLenConstraint       INTEGER (0..MAX) OPTIONAL }

##### 4.2.1.10. Name Constraints

   The name constraints extension, which MUST be used only in a CA
   certificate, indicates a name space within which all subject names in
   subsequent certificates in a certification path MUST be located.
   Restrictions apply to the subject distinguished name and apply to
   subject alternative names.  Restrictions apply only when the
   specified name form is present.  If no name of the type is in the
   certificate, the certificate is acceptable.

   Name constraints are not applied to self-issued certificates (unless
   the certificate is the final certificate in the path).  (This could
   prevent CAs that use name constraints from employing self-issued
   certificates to implement key rollover.)

   Restrictions are defined in terms of permitted or excluded name
   subtrees.  Any name matching a restriction in the excludedSubtrees
   field is invalid regardless of information appearing in the
   permittedSubtrees.  Conforming CAs MUST mark this extension as
   critical and SHOULD NOT impose name constraints on the x400Address,
   ediPartyName, or registeredID name forms.  Conforming CAs MUST NOT
   issue certificates where name constraints is an empty sequence.  That
   is, either the permittedSubtrees field or the excludedSubtrees MUST
   be present.

   Applications conforming to this profile MUST be able to process name
   constraints that are imposed on the directoryName name form and
   SHOULD be able to process name constraints that are imposed on the
   rfc822Name, uniformResourceIdentifier, dNSName, and iPAddress name
   forms.  If a name constraints extension that is marked as critical
   imposes constraints on a particular name form, and an instance of
   that name form appears in the subject field or subjectAltName
   extension of a subsequent certificate, then the application MUST
   either process the constraint or reject the certificate. 
   Within this profile, the minimum and maximum fields are not used with
   any name forms, thus, the minimum MUST be zero, and maximum MUST be
   absent.  However, if an application encounters a critical name
   constraints extension that specifies other values for minimum or
   maximum for a name form that appears in a subsequent certificate, the
   application MUST either process these fields or reject the
   certificate.

   For URIs, the constraint applies to the host part of the name.  The
   constraint MUST be specified as a fully qualified domain name and MAY
   specify a host or a domain.  Examples would be "host.example.com" and
   ".example.com".  When the constraint begins with a period, it MAY be
   expanded with one or more labels.  That is, the constraint
   ".example.com" is satisfied by both host.example.com and
   my.host.example.com.  However, the constraint ".example.com" is not
   satisfied by "example.com".  When the constraint does not begin with
   a period, it specifies a host.  If a constraint is applied to the
   uniformResourceIdentifier name form and a subsequent certificate
   includes a subjectAltName extension with a uniformResourceIdentifier
   that does not include an authority component with a host name
   specified as a fully qualified domain name (e.g., if the URI either
   does not include an authority component or includes an authority
   component in which the host name is specified as an IP address), then
   the application MUST reject the certificate.

   A name constraint for Internet mail addresses MAY specify a
   particular mailbox, all addresses at a particular host, or all
   mailboxes in a domain.  To indicate a particular mailbox, the
   constraint is the complete mail address.  For example,
   "root@example.com" indicates the root mailbox on the host
   "example.com".  To indicate all Internet mail addresses on a
   particular host, the constraint is specified as the host name.  For
   example, the constraint "example.com" is satisfied by any mail
   address at the host "example.com".  To specify any address within a
   domain, the constraint is specified with a leading period (as with
   URIs).  For example, ".example.com" indicates all the Internet mail
   addresses in the domain "example.com", but not Internet mail
   addresses on the host "example.com".

   DNS name restrictions are expressed as host.example.com.  Any DNS
   name that can be constructed by simply adding zero or more labels to
   the left-hand side of the name satisfies the name constraint.  For
   example, www.host.example.com would satisfy the constraint but
   host1.example.com would not.

   Legacy implementations exist where an electronic mail address is
   embedded in the subject distinguished name in an attribute of type
   emailAddress (Section 4.1.2.6).  When constraints are imposed on the 
   rfc822Name name form, but the certificate does not include a subject
   alternative name, the rfc822Name constraint MUST be applied to the
   attribute of type emailAddress in the subject distinguished name.
   The ASN.1 syntax for emailAddress and the corresponding OID are
   supplied in Appendix A .

   Restrictions of the form directoryName MUST be applied to the subject
   field in the certificate (when the certificate includes a non-empty
   subject field) and to any names of type directoryName in the
   subjectAltName extension.  Restrictions of the form x400Address MUST
   be applied to any names of type x400Address in the subjectAltName
   extension.

   When applying restrictions of the form directoryName, an
   implementation MUST compare DN attributes.  At a minimum,
   implementations MUST perform the DN comparison rules specified in Section 7.1.  CAs issuing certificates with a restriction of the form
   directoryName SHOULD NOT rely on implementation of the full ISO DN
   name comparison algorithm.  This implies name restrictions MUST be
   stated identically to the encoding used in the subject field or
   subjectAltName extension.

   The syntax of iPAddress MUST be as described in Section 4.2.1.6 with
   the following additions specifically for name constraints.  For IPv4
   addresses, the iPAddress field of GeneralName MUST contain eight (8)
   octets, encoded in the style of RFC 4632 (CIDR) to represent an
   address range [RFC4632].  For IPv6 addresses, the iPAddress field
   MUST contain 32 octets similarly encoded.  For example, a name
   constraint for "class C" subnet 192.0.2.0 is represented as the
   octets C0 00 02 00 FF FF FF 00, representing the CIDR notation
   192.0.2.0/24 (mask 255.255.255.0).

   Additional rules for encoding and processing name constraints are
   specified in Section 7.

   The syntax and semantics for name constraints for otherName,
   ediPartyName, and registeredID are not defined by this specification,
   however, syntax and semantics for name constraints for other name
   forms may be specified in other documents.

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

##### 4.2.1.11. Policy Constraints

   The policy constraints extension can be used in certificates issued
   to CAs.  The policy constraints extension constrains path validation
   in two ways.  It can be used to prohibit policy mapping or require
   that each certificate in a path contain an acceptable policy
   identifier.

   If the inhibitPolicyMapping field is present, the value indicates the
   number of additional certificates that may appear in the path before
   policy mapping is no longer permitted.  For example, a value of one
   indicates that policy mapping may be processed in certificates issued
   by the subject of this certificate, but not in additional
   certificates in the path.

   If the requireExplicitPolicy field is present, the value of
   requireExplicitPolicy indicates the number of additional certificates
   that may appear in the path before an explicit policy is required for
   the entire path.  When an explicit policy is required, it is
   necessary for all certificates in the path to contain an acceptable
   policy identifier in the certificate policies extension.  An
   acceptable policy identifier is the identifier of a policy required
   by the user of the certification path or the identifier of a policy
   that has been declared equivalent through policy mapping.

   Conforming applications MUST be able to process the
   requireExplicitPolicy field and SHOULD be able to process the
   inhibitPolicyMapping field.  Applications that support the
   inhibitPolicyMapping field MUST also implement support for the
   policyMappings extension.  If the policyConstraints extension is
   marked as critical and the inhibitPolicyMapping field is present,
   applications that do not implement support for the
   inhibitPolicyMapping field MUST reject the certificate.

   Conforming CAs MUST NOT issue certificates where policy constraints
   is an empty sequence.  That is, either the inhibitPolicyMapping field
   or the requireExplicitPolicy field MUST be present.  The behavior of
   clients that encounter an empty policy constraints field is not
   addressed in this profile.

   Conforming CAs MUST mark this extension as critical. 
   id-ce-policyConstraints OBJECT IDENTIFIER ::=  { id-ce 36 }

   PolicyConstraints ::= SEQUENCE {
        requireExplicitPolicy           [0] SkipCerts OPTIONAL,
        inhibitPolicyMapping            [1] SkipCerts OPTIONAL }

   SkipCerts ::= INTEGER (0..MAX)

##### 4.2.1.12. Extended Key Usage

   This extension indicates one or more purposes for which the certified
   public key may be used, in addition to or in place of the basic
   purposes indicated in the key usage extension.  In general, this
   extension will appear only in end entity certificates.  This
   extension is defined as follows:

   id-ce-extKeyUsage OBJECT IDENTIFIER ::= { id-ce 37 }

   ExtKeyUsageSyntax ::= SEQUENCE SIZE (1..MAX) OF KeyPurposeId

   KeyPurposeId ::= OBJECT IDENTIFIER

   Key purposes may be defined by any organization with a need.  Object
   identifiers used to identify key purposes MUST be assigned in
   accordance with IANA or ITU-T Recommendation X.660 [X.660].

   This extension MAY, at the option of the certificate issuer, be
   either critical or non-critical.

   If the extension is present, then the certificate MUST only be used
   for one of the purposes indicated.  If multiple purposes are
   indicated the application need not recognize all purposes indicated,
   as long as the intended purpose is present.  Certificate using
   applications MAY require that the extended key usage extension be
   present and that a particular purpose be indicated in order for the
   certificate to be acceptable to that application.

   If a CA includes extended key usages to satisfy such applications,
   but does not wish to restrict usages of the key, the CA can include
   the special KeyPurposeId anyExtendedKeyUsage in addition to the
   particular key purposes required by the applications.  Conforming CAs
   SHOULD NOT mark this extension as critical if the anyExtendedKeyUsage
   KeyPurposeId is present.  Applications that require the presence of a
   particular purpose MAY reject certificates that include the
   anyExtendedKeyUsage OID but not the particular OID expected for the
   application. 
   If a certificate contains both a key usage extension and an extended
   key usage extension, then both extensions MUST be processed
   independently and the certificate MUST only be used for a purpose
   consistent with both extensions.  If there is no purpose consistent
   with both extensions, then the certificate MUST NOT be used for any
   purpose.

   The following key usage purposes are defined:

   anyExtendedKeyUsage OBJECT IDENTIFIER ::= { id-ce-extKeyUsage 0 }

   id-kp OBJECT IDENTIFIER ::= { id-pkix 3 }

   id-kp-serverAuth             OBJECT IDENTIFIER ::= { id-kp 1 }
   -- TLS WWW server authentication
   -- Key usage bits that may be consistent: digitalSignature,
   -- keyEncipherment or keyAgreement

   id-kp-clientAuth             OBJECT IDENTIFIER ::= { id-kp 2 }
   -- TLS WWW client authentication
   -- Key usage bits that may be consistent: digitalSignature
   -- and/or keyAgreement

   id-kp-codeSigning             OBJECT IDENTIFIER ::= { id-kp 3 }
   -- Signing of downloadable executable code
   -- Key usage bits that may be consistent: digitalSignature

   id-kp-emailProtection         OBJECT IDENTIFIER ::= { id-kp 4 }
   -- Email protection
   -- Key usage bits that may be consistent: digitalSignature,
   -- nonRepudiation, and/or (keyEncipherment or keyAgreement)

   id-kp-timeStamping            OBJECT IDENTIFIER ::= { id-kp 8 }
   -- Binding the hash of an object to a time
   -- Key usage bits that may be consistent: digitalSignature
   -- and/or nonRepudiation

   id-kp-OCSPSigning            OBJECT IDENTIFIER ::= { id-kp 9 }
   -- Signing OCSP responses
   -- Key usage bits that may be consistent: digitalSignature
   -- and/or nonRepudiation

##### 4.2.1.13. CRL Distribution Points

   The CRL distribution points extension identifies how CRL information
   is obtained.  The extension SHOULD be non-critical, but this profile
   RECOMMENDS support for this extension by CAs and applications.
   Further discussion of CRL management is contained in Section 5. 
   The cRLDistributionPoints extension is a SEQUENCE of
   DistributionPoint.  A DistributionPoint consists of three fields,
   each of which is optional: distributionPoint, reasons, and cRLIssuer.
   While each of these fields is optional, a DistributionPoint MUST NOT
   consist of only the reasons field; either distributionPoint or
   cRLIssuer MUST be present.  If the certificate issuer is not the CRL
   issuer, then the cRLIssuer field MUST be present and contain the Name
   of the CRL issuer.  If the certificate issuer is also the CRL issuer,
   then conforming CAs MUST omit the cRLIssuer field and MUST include
   the distributionPoint field.

   When the distributionPoint field is present, it contains either a
   SEQUENCE of general names or a single value, nameRelativeToCRLIssuer.
   If the DistributionPointName contains multiple values, each name
   describes a different mechanism to obtain the same CRL.  For example,
   the same CRL could be available for retrieval through both LDAP and
   HTTP.

   If the distributionPoint field contains a directoryName, the entry
   for that directoryName contains the current CRL for the associated
   reasons and the CRL is issued by the associated cRLIssuer.  The CRL
   may be stored in either the certificateRevocationList or
   authorityRevocationList attribute.  The CRL is to be obtained by the
   application from whatever directory server is locally configured.
   The protocol the application uses to access the directory (e.g., DAP
   or LDAP) is a local matter.

   If the DistributionPointName contains a general name of type URI, the
   following semantics MUST be assumed: the URI is a pointer to the
   current CRL for the associated reasons and will be issued by the
   associated cRLIssuer.  When the HTTP or FTP URI scheme is used, the
   URI MUST point to a single DER encoded CRL as specified in
   [RFC2585].  HTTP server implementations accessed via the URI SHOULD
   specify the media type application/pkix-crl in the content-type
   header field of the response.  When the LDAP URI scheme [RFC4516] is
   used, the URI MUST include a <dn> field containing the distinguished
   name of the entry holding the CRL, MUST include a single <attrdesc>
   that contains an appropriate attribute description for the attribute
   that holds the CRL [RFC4523], and SHOULD include a <host>
   (e.g., <ldap://ldap.example.com/cn=example%20CA,dc=example,dc=com?
   certificateRevocationList;binary>).  Omitting the <host> (e.g.,
   <ldap:///cn=CA,dc=example,dc=com?authorityRevocationList;binary>) has
   the effect of relying on whatever a priori knowledge the client might
   have to contact an appropriate server.  When present,
   DistributionPointName SHOULD include at least one LDAP or HTTP URI.

   If the DistributionPointName contains the single value
   nameRelativeToCRLIssuer, the value provides a distinguished name 
   fragment.  The fragment is appended to the X.500 distinguished name
   of the CRL issuer to obtain the distribution point name.  If the
   cRLIssuer field in the DistributionPoint is present, then the name
   fragment is appended to the distinguished name that it contains;
   otherwise, the name fragment is appended to the certificate issuer
   distinguished name.  Conforming CAs SHOULD NOT use
   nameRelativeToCRLIssuer to specify distribution point names.  The
   DistributionPointName MUST NOT use the nameRelativeToCRLIssuer
   alternative when cRLIssuer contains more than one distinguished name.

   If the DistributionPoint omits the reasons field, the CRL MUST
   include revocation information for all reasons.  This profile
   RECOMMENDS against segmenting CRLs by reason code.  When a conforming
   CA includes a cRLDistributionPoints extension in a certificate, it
   MUST include at least one DistributionPoint that points to a CRL that
   covers the certificate for all reasons.

   The cRLIssuer identifies the entity that signs and issues the CRL.
   If present, the cRLIssuer MUST only contain the distinguished name
   (DN) from the issuer field of the CRL to which the DistributionPoint
   is pointing.  The encoding of the name in the cRLIssuer field MUST be
   exactly the same as the encoding in issuer field of the CRL.  If the
   cRLIssuer field is included and the DN in that field does not
   correspond to an X.500 or LDAP directory entry where CRL is located,
   then conforming CAs MUST include the distributionPoint field.

   id-ce-cRLDistributionPoints OBJECT IDENTIFIER ::=  { id-ce 31 }

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

##### 4.2.1.14. Inhibit anyPolicy

   The inhibit anyPolicy extension can be used in certificates issued to
   CAs.  The inhibit anyPolicy extension indicates that the special
   anyPolicy OID, with the value { 2 5 29 32 0 }, is not considered an
   explicit match for other certificate policies except when it appears
   in an intermediate self-issued CA certificate.  The value indicates
   the number of additional non-self-issued certificates that may appear
   in the path before anyPolicy is no longer permitted.  For example, a
   value of one indicates that anyPolicy may be processed in
   certificates issued by the subject of this certificate, but not in
   additional certificates in the path.

   Conforming CAs MUST mark this extension as critical.

   id-ce-inhibitAnyPolicy OBJECT IDENTIFIER ::=  { id-ce 54 }

   InhibitAnyPolicy ::= SkipCerts

   SkipCerts ::= INTEGER (0..MAX)

##### 4.2.1.15. Freshest CRL (a.k.a. Delta CRL Distribution Point)

   The freshest CRL extension identifies how delta CRL information is
   obtained.  The extension MUST be marked as non-critical by conforming
   CAs.  Further discussion of CRL management is contained in Section 5.

   The same syntax is used for this extension and the
   cRLDistributionPoints extension, and is described in Section 
   4.2.1.13.  The same conventions apply to both extensions.

   id-ce-freshestCRL OBJECT IDENTIFIER ::=  { id-ce 46 }

   FreshestCRL ::= CRLDistributionPoints 





#### 4.2.2. Private Internet Extensions

   This section defines two extensions for use in the Internet Public
   Key Infrastructure.  These extensions may be used to direct
   applications to on-line information about the issuer or the subject.
   Each extension contains a sequence of access methods and access
   locations.  The access method is an object identifier that indicates
   the type of information that is available.  The access location is a
   GeneralName that implicitly specifies the location and format of the
   information and the method for obtaining the information.

   Object identifiers are defined for the private extensions.  The
   object identifiers associated with the private extensions are defined
   under the arc id-pe within the arc id-pkix.  Any future extensions
   defined for the Internet PKI are also expected to be defined under
   the arc id-pe.

      id-pkix  OBJECT IDENTIFIER  ::=
               { iso(1) identified-organization(3) dod(6) internet(1)
                       security(5) mechanisms(5) pkix(7) }

      id-pe  OBJECT IDENTIFIER  ::=  { id-pkix 1 }

##### 4.2.2.1. Authority Information Access

   The authority information access extension indicates how to access
   information and services for the issuer of the certificate in which
   the extension appears.  Information and services may include on-line
   validation services and CA policy data.  (The location of CRLs is not
   specified in this extension; that information is provided by the
   cRLDistributionPoints extension.)  This extension may be included in
   end entity or CA certificates.  Conforming CAs MUST mark this
   extension as non-critical.

   id-pe-authorityInfoAccess OBJECT IDENTIFIER ::= { id-pe 1 }

   AuthorityInfoAccessSyntax  ::=
           SEQUENCE SIZE (1..MAX) OF AccessDescription

   AccessDescription  ::=  SEQUENCE {
           accessMethod          OBJECT IDENTIFIER,
           accessLocation        GeneralName  }

   id-ad OBJECT IDENTIFIER ::= { id-pkix 48 }

   id-ad-caIssuers OBJECT IDENTIFIER ::= { id-ad 2 }

   id-ad-ocsp OBJECT IDENTIFIER ::= { id-ad 1 }
   Each entry in the sequence AuthorityInfoAccessSyntax describes the
   format and location of additional information provided by the issuer
   of the certificate in which this extension appears.  The type and
   format of the information are specified by the accessMethod field;
   the accessLocation field specifies the location of the information.
   The retrieval mechanism may be implied by the accessMethod or
   specified by accessLocation.

   This profile defines two accessMethod OIDs: id-ad-caIssuers and
   id-ad-ocsp.

   In a public key certificate, the id-ad-caIssuers OID is used when the
   additional information lists certificates that were issued to the CA
   that issued the certificate containing this extension.  The
   referenced CA issuers description is intended to aid certificate
   users in the selection of a certification path that terminates at a
   point trusted by the certificate user.

   When id-ad-caIssuers appears as accessMethod, the accessLocation
   field describes the referenced description server and the access
   protocol to obtain the referenced description.  The accessLocation
   field is defined as a GeneralName, which can take several forms.

   When the accessLocation is a directoryName, the information is to be
   obtained by the application from whatever directory server is locally
   configured.  The entry for the directoryName contains CA certificates
   in the crossCertificatePair and/or cACertificate attributes as
   specified in [RFC4523].  The protocol that application uses to access
   the directory (e.g., DAP or LDAP) is a local matter.

   Where the information is available via LDAP, the accessLocation
   SHOULD be a uniformResourceIdentifier.  The LDAP URI [RFC4516] MUST
   include a <dn> field containing the distinguished name of the entry
   holding the certificates, MUST include an <attributes> field that
   lists appropriate attribute descriptions for the attributes that hold
   the DER encoded certificates or cross-certificate pairs [RFC4523],
   and SHOULD include a <host> (e.g., <ldap://ldap.example.com/cn=CA,
   dc=example,dc=com?cACertificate;binary,crossCertificatePair;binary>).
   Omitting the <host> (e.g., <ldap:///cn=exampleCA,dc=example,dc=com?
   cACertificate;binary>) has the effect of relying on whatever a priori
   knowledge the client might have to contact an appropriate server.

   Where the information is available via HTTP or FTP, accessLocation
   MUST be a uniformResourceIdentifier and the URI MUST point to either
   a single DER encoded certificate as specified in [RFC2585] or a
   collection of certificates in a BER or DER encoded "certs-only" CMS
   message as specified in [RFC2797]. 
   Conforming applications that support HTTP or FTP for accessing
   certificates MUST be able to accept individual DER encoded
   certificates and SHOULD be able to accept "certs-only" CMS messages.

   HTTP server implementations accessed via the URI SHOULD specify the
   media type application/pkix-cert [RFC2585] in the content-type header
   field of the response for a single DER encoded certificate and SHOULD
   specify the media type application/pkcs7-mime [RFC2797] in the
   content-type header field of the response for "certs-only" CMS
   messages.  For FTP, the name of a file that contains a single DER
   encoded certificate SHOULD have a suffix of ".cer" [RFC2585] and the
   name of a file that contains a "certs-only" CMS message SHOULD have a
   suffix of ".p7c" [RFC2797].  Consuming clients may use the media type
   or file extension as a hint to the content, but should not depend
   solely on the presence of the correct media type or file extension in
   the server response.

   The semantics of other id-ad-caIssuers accessLocation name forms are
   not defined.

   An authorityInfoAccess extension may include multiple instances of
   the id-ad-caIssuers accessMethod.  The different instances may
   specify different methods for accessing the same information or may
   point to different information.  When the id-ad-caIssuers
   accessMethod is used, at least one instance SHOULD specify an
   accessLocation that is an HTTP [RFC2616] or LDAP [RFC4516] URI.

   The id-ad-ocsp OID is used when revocation information for the
   certificate containing this extension is available using the Online
   Certificate Status Protocol (OCSP) [RFC2560].

   When id-ad-ocsp appears as accessMethod, the accessLocation field is
   the location of the OCSP responder, using the conventions defined in
   [RFC2560].

   Additional access descriptors may be defined in other PKIX
   specifications.

##### 4.2.2.2. Subject Information Access

   The subject information access extension indicates how to access
   information and services for the subject of the certificate in which
   the extension appears.  When the subject is a CA, information and
   services may include certificate validation services and CA policy
   data.  When the subject is an end entity, the information describes
   the type of services offered and how to access them.  In this case,
   the contents of this extension are defined in the protocol 
   specifications for the supported services.  This extension may be
   included in end entity or CA certificates.  Conforming CAs MUST mark
   this extension as non-critical.

   id-pe-subjectInfoAccess OBJECT IDENTIFIER ::= { id-pe 11 }

   SubjectInfoAccessSyntax  ::=
           SEQUENCE SIZE (1..MAX) OF AccessDescription

   AccessDescription  ::=  SEQUENCE {
           accessMethod          OBJECT IDENTIFIER,
           accessLocation        GeneralName  }

   Each entry in the sequence SubjectInfoAccessSyntax describes the
   format and location of additional information provided by the subject
   of the certificate in which this extension appears.  The type and
   format of the information are specified by the accessMethod field;
   the accessLocation field specifies the location of the information.
   The retrieval mechanism may be implied by the accessMethod or
   specified by accessLocation.

   This profile defines one access method to be used when the subject is
   a CA and one access method to be used when the subject is an end
   entity.  Additional access methods may be defined in the future in
   the protocol specifications for other services.

   The id-ad-caRepository OID is used when the subject is a CA that
   publishes certificates it issues in a repository.  The accessLocation
   field is defined as a GeneralName, which can take several forms.

   When the accessLocation is a directoryName, the information is to be
   obtained by the application from whatever directory server is locally
   configured.  When the extension is used to point to CA certificates,
   the entry for the directoryName contains CA certificates in the
   crossCertificatePair and/or cACertificate attributes as specified in
   [RFC4523].  The protocol the application uses to access the directory
   (e.g., DAP or LDAP) is a local matter.

   Where the information is available via LDAP, the accessLocation
   SHOULD be a uniformResourceIdentifier.  The LDAP URI [RFC4516] MUST
   include a <dn> field containing the distinguished name of the entry
   holding the certificates, MUST include an <attributes> field that
   lists appropriate attribute descriptions for the attributes that hold
   the DER encoded certificates or cross-certificate pairs [RFC4523],
   and SHOULD include a <host> (e.g., <ldap://ldap.example.com/cn=CA,
   dc=example,dc=com?cACertificate;binary,crossCertificatePair;binary>). 
   Omitting the <host> (e.g., <ldap:///cn=exampleCA,dc=example,dc=com?
   cACertificate;binary>) has the effect of relying on whatever a priori
   knowledge the client might have to contact an appropriate server.

   Where the information is available via HTTP or FTP, accessLocation
   MUST be a uniformResourceIdentifier and the URI MUST point to either
   a single DER encoded certificate as specified in [RFC2585] or a
   collection of certificates in a BER or DER encoded "certs-only" CMS
   message as specified in [RFC2797].

   Conforming applications that support HTTP or FTP for accessing
   certificates MUST be able to accept individual DER encoded
   certificates and SHOULD be able to accept "certs-only" CMS messages.

   HTTP server implementations accessed via the URI SHOULD specify the
   media type application/pkix-cert [RFC2585] in the content-type header
   field of the response for a single DER encoded certificate and SHOULD
   specify the media type application/pkcs7-mime [RFC2797] in the
   content-type header field of the response for "certs-only" CMS
   messages.  For FTP, the name of a file that contains a single DER
   encoded certificate SHOULD have a suffix of ".cer" [RFC2585] and the
   name of a file that contains a "certs-only" CMS message SHOULD have a
   suffix of ".p7c" [RFC2797].  Consuming clients may use the media type
   or file extension as a hint to the content, but should not depend
   solely on the presence of the correct media type or file extension in
   the server response.

   The semantics of other id-ad-caRepository accessLocation name forms
   are not defined.

   A subjectInfoAccess extension may include multiple instances of the
   id-ad-caRepository accessMethod.  The different instances may specify
   different methods for accessing the same information or may point to
   different information.  When the id-ad-caRepository accessMethod is
   used, at least one instance SHOULD specify an accessLocation that is
   an HTTP [RFC2616] or LDAP [RFC4516] URI.

   The id-ad-timeStamping OID is used when the subject offers
   timestamping services using the Time Stamp Protocol defined in
   [RFC3161].  Where the timestamping services are available via HTTP or
   FTP, accessLocation MUST be a uniformResourceIdentifier.  Where the
   timestamping services are available via electronic mail,
   accessLocation MUST be an rfc822Name.  Where timestamping services
   are available using TCP/IP, the dNSName or iPAddress name forms may
   be used.  The semantics of other name forms of accessLocation (when
   accessMethod is id-ad-timeStamping) are not defined by this
   specification. 
   Additional access descriptors may be defined in other PKIX
   specifications.

   id-ad OBJECT IDENTIFIER ::= { id-pkix 48 }

   id-ad-caRepository OBJECT IDENTIFIER ::= { id-ad 5 }

   id-ad-timeStamping OBJECT IDENTIFIER ::= { id-ad 3 }

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

### C.1. RSA Self-Signed Certificate

   This appendix contains an annotated hex dump of a 578 byte version 3
   certificate.  The certificate contains the following information:

   (a)  the serial number is 17;
   (b)  the certificate is signed with RSA and the SHA-1 hash algorithm;
   (c)  the issuer's distinguished name is
        cn=Example CA,dc=example,dc=com;
   (d)  the subject's distinguished name is
        cn=Example CA,dc=example,dc=com;
   (e)  the certificate was issued on April 30, 2004 and expired on
        April 30, 2005;
   (f)  the certificate contains a 1024-bit RSA public key;
   (g)  the certificate contains a subject key identifier extension
        generated using method (1) of Section 4.2.1.2; and
   (h)  the certificate is a CA certificate (as indicated through the
        basic constraints extension).

   0  574: SEQUENCE {
   4  423:   SEQUENCE {
   8    3:     [0] {
  10    1:       INTEGER 2
         :       }
  13    1:     INTEGER 17
  16   13:     SEQUENCE {
  18    9:       OBJECT IDENTIFIER
         :         sha1withRSAEncryption (1 2 840 113549 1 1 5)
  29    0:       NULL
         :       }
  31   67:     SEQUENCE {
  33   19:       SET {
  35   17:         SEQUENCE {
  37   10:           OBJECT IDENTIFIER
         :             domainComponent (0 9 2342 19200300 100 1 25)
  49    3:           IA5String 'com'
         :           }
         :         }
  54   23:       SET {
  56   21:         SEQUENCE {
  58   10:           OBJECT IDENTIFIER
         :             domainComponent (0 9 2342 19200300 100 1 25)
  70    7:           IA5String 'example'
         :           }
         :         }
  79   19:       SET {
  81   17:         SEQUENCE {
  83    3:           OBJECT IDENTIFIER commonName (2 5 4 3)
  88   10:           PrintableString 'Example CA'
         :           }
         :         }
         :       }
 100   30:     SEQUENCE {
 102   13:       UTCTime 30/04/2004 14:25:34 GMT
 117   13:       UTCTime 30/04/2005 14:25:34 GMT
         :       }
 132   67:     SEQUENCE {
 134   19:       SET {
 136   17:         SEQUENCE {
 138   10:           OBJECT IDENTIFIER
         :             domainComponent (0 9 2342 19200300 100 1 25)
 150    3:           IA5String 'com'
         :           }
         :         }
 155   23:       SET {
 157   21:         SEQUENCE {
 159   10:           OBJECT IDENTIFIER
         :             domainComponent (0 9 2342 19200300 100 1 25)
 171    7:           IA5String 'example'
         :           }
         :         }
 180   19:       SET {
 182   17:         SEQUENCE {
 184    3:           OBJECT IDENTIFIER commonName (2 5 4 3)
 189   10:           PrintableString 'Example CA'
         :           }
         :         }
         :       }
 201  159:     SEQUENCE {
 204   13:       SEQUENCE {
 206    9:         OBJECT IDENTIFIER
         :           rsaEncryption (1 2 840 113549 1 1 1)
 217    0:         NULL
         :         }
 219  141:       BIT STRING, encapsulates {
 223  137:         SEQUENCE {
 226  129:           INTEGER
         :             00 C2 D7 97 6D 28 70 AA 5B CF 23 2E 80 70 39 EE
         :             DB 6F D5 2D D5 6A 4F 7A 34 2D F9 22 72 47 70 1D
         :             EF 80 E9 CA 30 8C 00 C4 9A 6E 5B 45 B4 6E A5 E6
         :             6C 94 0D FA 91 E9 40 FC 25 9D C7 B7 68 19 56 8F
         :             11 70 6A D7 F1 C9 11 4F 3A 7E 3F 99 8D 6E 76 A5
         :             74 5F 5E A4 55 53 E5 C7 68 36 53 C7 1D 3B 12 A6
         :             85 FE BD 6E A1 CA DF 35 50 AC 08 D7 B9 B4 7E 5C
         :             FE E2 A3 2C D1 23 84 AA 98 C0 9B 66 18 9A 68 47
         :             E9
 358    3:           INTEGER 65537
         :           }
         :         }
         :       }
 363   66:     [3] {
 365   64:       SEQUENCE {
 367   29:         SEQUENCE {
 369    3:           OBJECT IDENTIFIER subjectKeyIdentifier (2 5 29 14)
 374   22:           OCTET STRING, encapsulates {
 376   20:             OCTET STRING
         :               08 68 AF 85 33 C8 39 4A 7A F8 82 93 8E 70 6A 4A
         :               20 84 2C 32
         :             }
         :           }
 398   14:         SEQUENCE {
 400    3:           OBJECT IDENTIFIER keyUsage (2 5 29 15)
 405    1:           BOOLEAN TRUE
 408    4:           OCTET STRING, encapsulates {
 410    2:             BIT STRING 1 unused bits
         :               '0000011'B
         :             }
         :           }
 414   15:         SEQUENCE {
 416    3:           OBJECT IDENTIFIER basicConstraints (2 5 29 19)
 421    1:           BOOLEAN TRUE
 424    5:           OCTET STRING, encapsulates {
 426    3:             SEQUENCE {
 428    1:               BOOLEAN TRUE
         :               }
         :             }
         :           }
         :         }
         :       }
         :     }
 431   13:   SEQUENCE {
 433    9:     OBJECT IDENTIFIER
         :         sha1withRSAEncryption (1 2 840 113549 1 1 5)
 444    0:     NULL
         :     }
 446  129:   BIT STRING
         :     6C F8 02 74 A6 61 E2 64 04 A6 54 0C 6C 72 13 AD
         :     3C 47 FB F6 65 13 A9 85 90 33 EA 76 A3 26 D9 FC
         :     D1 0E 15 5F 28 B7 EF 93 BF 3C F3 E2 3E 7C B9 52
         :     FC 16 6E 29 AA E1 F4 7A 6F D5 7F EF B3 95 CA F3
         :     66 88 83 4E A1 35 45 84 CB BC 9B B8 C8 AD C5 5E
         :     46 D9 0B 0E 8D 80 E1 33 2B DC BE 2B 92 7E 4A 43
         :     A9 6A EF 8A 63 61 B3 6E 47 38 BE E8 0D A3 67 5D
         :     F3 FA 91 81 3C 92 BB C5 5F 25 25 EB 7C E7 D8 A1
         :   }

# Referenced Sections from RFC 8032: Edwards-Curve Digital Signature Algorithm (EdDSA)

The following sections were referenced. Remaining sections are not included.

## 5. EdDSA Instances

   This section instantiates the general EdDSA algorithm for the
   edwards25519 and edwards448 curves, each for the PureEdDSA and
   HashEdDSA variants (plus a contextualized extension of the Ed25519
   scheme).  Thus, five different parameter sets are described.

### 5.1. Ed25519ph, Ed25519ctx, and Ed25519

   Ed25519 is EdDSA instantiated with:

   +-----------+-------------------------------------------------------+
   | Parameter | Value                                                 |
   +-----------+-------------------------------------------------------+
   |     p     | p of edwards25519 in [RFC7748] (i.e., 2^255 - 19)     |
   |     b     | 256                                                   |
   |  encoding | 255-bit little-endian encoding of {0, 1, ..., p-1}    |
   |  of GF(p) |                                                       |
   |    H(x)   | SHA-512(dom2(phflag,context)||x) [RFC6234]            |
   |     c     | base 2 logarithm of cofactor of edwards25519 in       |
   |           | [RFC7748] (i.e., 3)                                   |
   |     n     | 254                                                   |
   |     d     | d of edwards25519 in [RFC7748] (i.e., -121665/121666  |
   |           | = 370957059346694393431380835087545651895421138798432 |
   |           | 19016388785533085940283555)                           |
   |     a     | -1                                                    |
   |     B     | (X(P),Y(P)) of edwards25519 in [RFC7748] (i.e., (1511 |
   |           | 22213495354007725011514095885315114540126930418572060 |
   |           | 46113283949847762202, 4631683569492647816942839400347 |
   |           | 5163141307993866256225615783033603165251855960))      |
   |     L     | order of edwards25519 in [RFC7748] (i.e.,             |
   |           | 2^252+27742317777372353535851937790883648493).        |
   |   PH(x)   | x (i.e., the identity function)                       |
   +-----------+-------------------------------------------------------+

                      Table 1: Parameters of Ed25519

   For Ed25519, dom2(f,c) is the empty string.  The phflag value is
   irrelevant.  The context (if present at all) MUST be empty.  This
   causes the scheme to be one and the same with the Ed25519 scheme
   published earlier.

   For Ed25519ctx, phflag=0.  The context input SHOULD NOT be empty. 
   For Ed25519ph, phflag=1 and PH is SHA512 instead.  That is, the input
   is hashed using SHA-512 before signing with Ed25519.

   Value of context is set by the signer and verifier (maximum of 255
   octets; the default is empty string, except for Ed25519, which can't
   have context) and has to match octet by octet for verification to be
   successful.

   The curve used is equivalent to Curve25519 [CURVE25519], under a
   change of coordinates, which means that the difficulty of the
   discrete logarithm problem is the same as for Curve25519.

#### 5.1.1. Modular Arithmetic

   For advice on how to implement arithmetic modulo p = 2^255 - 19
   efficiently and securely, see Curve25519 [CURVE25519].  For inversion
   modulo p, it is recommended to use the identity x^-1 = x^(p-2) (mod
   p).  Inverting zero should never happen, as it would require invalid
   input, which would have been detected before, or would be a
   calculation error.

   For point decoding or "decompression", square roots modulo p are
   needed.  They can be computed using the Tonelli-Shanks algorithm or
   the special case for p = 5 (mod 8).  To find a square root of a,
   first compute the candidate root x = a^((p+3)/8) (mod p).  Then there
   are three cases:

      x^2 = a (mod p).  Then x is a square root.

      x^2 = -a (mod p).  Then 2^((p-1)/4) * x is a square root.

      a is not a square modulo p.

#### 5.1.2. Encoding

   All values are coded as octet strings, and integers are coded using
   little-endian convention, i.e., a 32-octet string h h[0],...h[31]
   represents the integer h[0] + 2^8 * h[1] + ... + 2^248 * h[31].

   A curve point (x,y), with coordinates in the range 0 <= x,y < p, is
   coded as follows.  First, encode the y-coordinate as a little-endian
   string of 32 octets.  The most significant bit of the final octet is
   always zero.  To form the encoding of the point, copy the least
   significant bit of the x-coordinate to the most significant bit of
   the final octet. 





#### 5.1.3. Decoding

   Decoding a point, given as a 32-octet string, is a little more
   complicated.

   1.  First, interpret the string as an integer in little-endian
       representation.  Bit 255 of this number is the least significant
       bit of the x-coordinate and denote this value x_0.  The
       y-coordinate is recovered simply by clearing this bit.  If the
       resulting value is >= p, decoding fails.

   2.  To recover the x-coordinate, the curve equation implies
       x^2 = (y^2 - 1) / (d y^2 + 1) (mod p).  The denominator is always
       non-zero mod p.  Let u = y^2 - 1 and v = d y^2 + 1.  To compute
       the square root of (u/v), the first step is to compute the
       candidate root x = (u/v)^((p+3)/8).  This can be done with the
       following trick, using a single modular powering for both the
       inversion of v and the square root:

                          (p+3)/8      3        (p-5)/8
                 x = (u/v)        = u v  (u v^7)         (mod p)

   3.  Again, there are three cases:

       1.  If v x^2 = u (mod p), x is a square root.

       2.  If v x^2 = -u (mod p), set x <-- x * 2^((p-1)/4), which is a
           square root.

       3.  Otherwise, no square root exists for modulo p, and decoding
           fails.

   4.  Finally, use the x_0 bit to select the right square root.  If
       x = 0, and x_0 = 1, decoding fails.  Otherwise, if x_0 != x mod
       2, set x <-- p - x.  Return the decoded point (x,y).

#### 5.1.4. Point Addition

   For point addition, the following method is recommended.  A point
   (x,y) is represented in extended homogeneous coordinates (X, Y, Z,
   T), with x = X/Z, y = Y/Z, x * y = T/Z.

   The neutral point is (0,1), or equivalently in extended homogeneous
   coordinates (0, Z, Z, 0) for any non-zero Z. 
   The following formulas for adding two points, (x3,y3) =
   (x1,y1)+(x2,y2), on twisted Edwards curves with a=-1, square a, and
   non-square d are described in Section 3.1 of [Edwards-revisited ] and
   in [EFD-TWISTED-ADD ].  They are complete, i.e., they work for any
   pair of valid input points.

                 A = (Y1-X1)*(Y2-X2)
                 B = (Y1+X1)*(Y2+X2)
                 C = T1*2*d*T2
                 D = Z1*2*Z2
                 E = B-A
                 F = D-C
                 G = D+C
                 H = B+A
                 X3 = E*F
                 Y3 = G*H
                 T3 = E*H
                 Z3 = F*G

   For point doubling, (x3,y3) = (x1,y1)+(x1,y1), one could just
   substitute equal points in the above (because of completeness, such
   substitution is valid) and observe that four multiplications turn
   into squares.  However, using the formulas described in Section 3.2
   of [Edwards-revisited ] and in [EFD-TWISTED-DBL ] saves a few smaller
   operations.

                 A = X1^2
                 B = Y1^2
                 C = 2*Z1^2
                 H = A+B
                 E = H-(X1+Y1)^2
                 G = A-B
                 F = C+G
                 X3 = E*F
                 Y3 = G*H
                 T3 = E*H
                 Z3 = F*G 





#### 5.1.5. Key Generation

   The private key is 32 octets (256 bits, corresponding to b) of
   cryptographically secure random data.  See [RFC4086] for a discussion
   about randomness.

   The 32-byte public key is generated by the following steps.

   1.  Hash the 32-byte private key using SHA-512, storing the digest in
       a 64-octet large buffer, denoted h.  Only the lower 32 bytes are
       used for generating the public key.

   2.  Prune the buffer: The lowest three bits of the first octet are
       cleared, the highest bit of the last octet is cleared, and the
       second highest bit of the last octet is set.

   3.  Interpret the buffer as the little-endian integer, forming a
       secret scalar s.  Perform a fixed-base scalar multiplication
       [s]B.

   4.  The public key A is the encoding of the point [s]B.  First,
       encode the y-coordinate (in the range 0 <= y < p) as a little-
       endian string of 32 octets.  The most significant bit of the
       final octet is always zero.  To form the encoding of the point
       [s]B, copy the least significant bit of the x coordinate to the
       most significant bit of the final octet.  The result is the
       public key.

#### 5.1.6. Sign

   The inputs to the signing procedure is the private key, a 32-octet
   string, and a message M of arbitrary size.  For Ed25519ctx and
   Ed25519ph, there is additionally a context C of at most 255 octets
   and a flag F, 0 for Ed25519ctx and 1 for Ed25519ph.

   1.  Hash the private key, 32 octets, using SHA-512.  Let h denote the
       resulting digest.  Construct the secret scalar s from the first
       half of the digest, and the corresponding public key A, as
       described in the previous section.  Let prefix denote the second
       half of the hash digest, h[32],...,h[63].

   2.  Compute SHA-512(dom2(F, C) || prefix || PH(M)), where M is the
       message to be signed.  Interpret the 64-octet digest as a little-
       endian integer r.

   3.  Compute the point [r]B.  For efficiency, do this by first
       reducing r modulo L, the group order of B.  Let the string R be
       the encoding of this point. 
   4.  Compute SHA512(dom2(F, C) || R || A || PH(M)), and interpret the
       64-octet digest as a little-endian integer k.

   5.  Compute S = (r + k * s) mod L.  For efficiency, again reduce k
       modulo L first.

   6.  Form the signature of the concatenation of R (32 octets) and the
       little-endian encoding of S (32 octets; the three most
       significant bits of the final octet are always zero).

#### 5.1.7. Verify

   1.  To verify a signature on a message M using public key A, with F
       being 0 for Ed25519ctx, 1 for Ed25519ph, and if Ed25519ctx or
       Ed25519ph is being used, C being the context, first split the
       signature into two 32-octet halves.  Decode the first half as a
       point R, and the second half as an integer S, in the range
       0 <= s < L.  Decode the public key A as point A'.  If any of the
       decodings fail (including S being out of range), the signature is
       invalid.

   2.  Compute SHA512(dom2(F, C) || R || A || PH(M)), and interpret the
       64-octet digest as a little-endian integer k.

   3.  Check the group equation [8][S]B = [8]R + [8][k]A'.  It's
       sufficient, but not required, to instead check [S]B = R + [k]A'. 





### 5.2. Ed448ph and Ed448

   Ed448 is EdDSA instantiated with:

   +-----------+-------------------------------------------------------+
   | Parameter | Value                                                 |
   +-----------+-------------------------------------------------------+
   |     p     | p of edwards448 in [RFC7748] (i.e., 2^448 - 2^224 -   |
   |           | 1)                                                    |
   |     b     | 456                                                   |
   |  encoding | 455-bit little-endian encoding of {0, 1, ..., p-1}    |
   |  of GF(p) |                                                       |
   |    H(x)   | SHAKE256(dom4(phflag,context)||x, 114)                |
   |   phflag  | 0                                                     |
   |     c     | base 2 logarithm of cofactor of edwards448 in         |
   |           | [RFC7748] (i.e., 2)                                   |
   |     n     | 447                                                   |
   |     d     | d of edwards448 in [RFC7748] (i.e., -39081)           |
   |     a     | 1                                                     |
   |     B     | (X(P),Y(P)) of edwards448 in [RFC7748] (i.e., (224580 |
   |           | 04029592430018760433409989603624678964163256413424612 |
   |           | 54616869504154674060329090291928693579532825780320751 |
   |           | 46446173674602635247710, 2988192100784814926760179304 |
   |           | 43930673437544040154080242095928241372331506189835876 |
   |           | 00353687865541878473398230323350346250053154506283266 |
   |           | 0))                                                   |
   |     L     | order of edwards448 in [RFC7748] (i.e., 2^446 - 13818 |
   |           | 06680989511535200738674851542688033669247488217860989 |
   |           | 4547503885).                                          |
   |   PH(x)   | x (i.e., the identity function)                       |
   +-----------+-------------------------------------------------------+

                       Table 2: Parameters of Ed448

   Ed448ph is the same but with PH being SHAKE256(x, 64) and phflag
   being 1, i.e., the input is hashed before signing with Ed448 with a
   hash constant modified.

   Value of context is set by signer and verifier (maximum of 255
   octets; the default is empty string) and has to match octet by octet
   for verification to be successful.

   The curve is equivalent to Ed448-Goldilocks under change of the
   basepoint, which preserves difficulty of the discrete logarithm. 





#### 5.2.1. Modular Arithmetic

   For advice on how to implement arithmetic modulo p = 2^448 - 2^224 -
   1 efficiently and securely, see [ED448].  For inversion modulo p, it
   is recommended to use the identity x^-1 = x^(p-2) (mod p).  Inverting
   zero should never happen, as it would require invalid input, which
   would have been detected before, or would be a calculation error.

   For point decoding or "decompression", square roots modulo p are
   needed.  They can be computed by first computing candidate root
   x = a ^ (p+1)/4 (mod p) and then checking if x^2 = a.  If it is, then
   x is the square root of a; if it isn't, then a does not have a square
   root.

#### 5.2.2. Encoding

   All values are coded as octet strings, and integers are coded using
   little-endian convention, i.e., a 57-octet string h h[0],...h[56]
   represents the integer h[0] + 2^8 * h[1] + ... + 2^448 * h[56].

   A curve point (x,y), with coordinates in the range 0 <= x,y < p, is
   coded as follows.  First, encode the y-coordinate as a little-endian
   string of 57 octets.  The final octet is always zero.  To form the
   encoding of the point, copy the least significant bit of the
   x-coordinate to the most significant bit of the final octet.

#### 5.2.3. Decoding

   Decoding a point, given as a 57-octet string, is a little more
   complicated.

   1.  First, interpret the string as an integer in little-endian
       representation.  Bit 455 of this number is the least significant
       bit of the x-coordinate, and denote this value x_0.  The
       y-coordinate is recovered simply by clearing this bit.  If the
       resulting value is >= p, decoding fails.

   2.  To recover the x-coordinate, the curve equation implies
       x^2 = (y^2 - 1) / (d y^2 - 1) (mod p).  The denominator is always
       non-zero mod p.  Let u = y^2 - 1 and v = d y^2 - 1.  To compute
       the square root of (u/v), the first step is to compute the
       candidate root x = (u/v)^((p+1)/4).  This can be done using the
       following trick, to use a single modular powering for both the
       inversion of v and the square root:

                          (p+1)/4    3            (p-3)/4
                 x = (u/v)        = u  v (u^5 v^3)         (mod p)
   3.  If v * x^2 = u, the recovered x-coordinate is x.  Otherwise, no
       square root exists, and the decoding fails.

   4.  Finally, use the x_0 bit to select the right square root.  If
       x = 0, and x_0 = 1, decoding fails.  Otherwise, if x_0 != x mod
       2, set x <-- p - x.  Return the decoded point (x,y).

#### 5.2.4. Point Addition

   For point addition, the following method is recommended.  A point
   (x,y) is represented in projective coordinates (X, Y, Z), with
   x = X/Z, y = Y/Z.

   The neutral point is (0,1), or equivalently in projective coordinates
   (0, Z, Z) for any non-zero Z.

   The following formulas for adding two points, (x3,y3) =
   (x1,y1)+(x2,y2) on untwisted Edwards curve (i.e., a=1) with non-
   square d, are described in Section 4 of [Faster-ECC ] and in
   [EFD-ADD ].  They are complete, i.e., they work for any pair of valid
   input points.

                 A = Z1*Z2
                 B = A^2
                 C = X1*X2
                 D = Y1*Y2
                 E = d*C*D
                 F = B-E
                 G = B+E
                 H = (X1+Y1)*(X2+Y2)
                 X3 = A*F*(H-C-D)
                 Y3 = A*G*(D-C)
                 Z3 = F*G 
   Again, similar to the other curve, doubling formulas can be obtained
   by substituting equal points, turning four multiplications into
   squares.  However, this is not even nearly optimal; the following
   formulas described in Section 4 of [Faster-ECC ] and in [EFD-DBL ] save
   multiple multiplications.

                 B = (X1+Y1)^2
                 C = X1^2
                 D = Y1^2
                 E = C+D
                 H = Z1^2
                 J = E-2*H
                 X3 = (B-E)*J
                 Y3 = E*(C-D)
                 Z3 = E*J

#### 5.2.5. Key Generation

   The private key is 57 octets (456 bits, corresponding to b) of
   cryptographically secure random data.  See [RFC4086] for a discussion
   about randomness.

   The 57-byte public key is generated by the following steps:

   1.  Hash the 57-byte private key using SHAKE256(x, 114), storing the
       digest in a 114-octet large buffer, denoted h.  Only the lower 57
       bytes are used for generating the public key.

   2.  Prune the buffer: The two least significant bits of the first
       octet are cleared, all eight bits the last octet are cleared, and
       the highest bit of the second to last octet is set.

   3.  Interpret the buffer as the little-endian integer, forming a
       secret scalar s.  Perform a known-base-point scalar
       multiplication [s]B.

   4.  The public key A is the encoding of the point [s]B.  First encode
       the y-coordinate (in the range 0 <= y < p) as a little-endian
       string of 57 octets.  The most significant bit of the final octet
       is always zero.  To form the encoding of the point [s]B, copy the
       least significant bit of the x coordinate to the most significant
       bit of the final octet.  The result is the public key. 





#### 5.2.6. Sign

   The inputs to the signing procedure is the private key, a 57-octet
   string, a flag F, which is 0 for Ed448, 1 for Ed448ph, context C of
   at most 255 octets, and a message M of arbitrary size.

   1.  Hash the private key, 57 octets, using SHAKE256(x, 114).  Let h
       denote the resulting digest.  Construct the secret scalar s from
       the first half of the digest, and the corresponding public key A,
       as described in the previous section.  Let prefix denote the
       second half of the hash digest, h[57],...,h[113].

   2.  Compute SHAKE256(dom4(F, C) || prefix || PH(M), 114), where M is
       the message to be signed, F is 1 for Ed448ph, 0 for Ed448, and C
       is the context to use.  Interpret the 114-octet digest as a
       little-endian integer r.

   3.  Compute the point [r]B.  For efficiency, do this by first
       reducing r modulo L, the group order of B.  Let the string R be
       the encoding of this point.

   4.  Compute SHAKE256(dom4(F, C) || R || A || PH(M), 114), and
       interpret the 114-octet digest as a little-endian integer k.

   5.  Compute S = (r + k * s) mod L.  For efficiency, again reduce k
       modulo L first.

   6.  Form the signature of the concatenation of R (57 octets) and the
       little-endian encoding of S (57 octets; the ten most significant
       bits of the final octets are always zero).

#### 5.2.7. Verify

   1.  To verify a signature on a message M using context C and public
       key A, with F being 0 for Ed448 and 1 for Ed448ph, first split
       the signature into two 57-octet halves.  Decode the first half as
       a point R, and the second half as an integer S, in the range 0 <=
       s < L.  Decode the public key A as point A'.  If any of the
       decodings fail (including S being out of range), the signature is
       invalid.

   2.  Compute SHAKE256(dom4(F, C) || R || A || PH(M), 114), and
       interpret the 114-octet digest as a little-endian integer k.

   3.  Check the group equation [4][S]B = [4]R + [4][k]A'.  It's
       sufficient, but not required, to instead check [S]B = R + [k]A'. 





# Referenced Sections from RFC 5756: Updates for RSAES-OAEP and RSASSA-PSS Algorithm Parameters

The following sections were referenced. Remaining sections are not included.

## 2. Changes to Section 3 (Second and Third Paragraphs)

   This change clarifies the placement of RSASSA-PSS-params in the
   signature, signatureAlgorithm, and subjectPublicKeyInfo fields for
   certification authority (CA) and end-entity (EE) certificates.  It
   also clarifies the placement of RSASSA-PSS-params in the
   signatureAlgorithm field in certificate revocation lists (CRLs).

   Old:

   CAs that issue certificates with the id-RSASSA-PSS algorithm
   identifier SHOULD require the presence of parameters in the
   publicKeyAlgorithms field if the cA boolean flag is set in the basic
   constraints certificate extension.  CAs MAY require that the
   parameters be present in the publicKeyAlgorithms field for end-entity
   certificates.

   CAs that use the RSASSA-PSS algorithm for signing certificates SHOULD
   include RSASSA-PSS-params in the subjectPublicKeyInfo algorithm
   parameters in their own certificates.  CAs that use the RSASSA-PSS
   algorithm for signing certificates or CRLs MUST include RSASSA-PSS-
   params in the signatureAlgorithm parameters in the TBSCertificate or
   TBSCertList structures.

   New:

   When the id-RSASSA-PSS object identifier appears in the
   TBSCertificate or TBSCertList signature algorithm field, then the
   RSASSA-PSS-params structure MUST be included in the TBSCertificate or
   TBSCertList signature parameters field.

   When the id-RSASSA-PSS object identifier appears in the
   TBSCertificate subjectPublicKeyInfo algorithm field of CA
   certificates, then the parameters field SHOULD include the RSASSA-
   PSS-params structure.  When the id-RSASSA-PSS object identifier
   appears in the TBSCertificate subjectPublicKeyInfo algorithm field of
   EE certificates, then the parameters field MAY include the RSASSA-
   PSS-params structure. 
   All certificates and CRLs signed by a CA that supports the id-RSASSA-
   PSS algorithm MUST include the RSASSA-PSS-params in the
   signatureAlgorithm parameters in Certificate and CertList structures,
   respectively.

# Referenced Sections from RFC 8422: Elliptic Curve Cryptography (ECC) Cipher Suites for Transport Layer Security (TLS) Versions 1.2 and Earlier

The following sections were referenced. Remaining sections are not included.

### 10.2. Informative References

   [FIPS.180-4]
              National Institute of Standards and Technology, "Secure
              Hash Standard (SHS)", FIPS PUB 180-4, DOI
              10.6028/NIST.FIPS.180-4, August 2015,
              <http://nvlpubs.nist.gov/nistpubs/FIPS/
              NIST.FIPS.180-4.pdf >.

   [IEEE.P1363]
              IEEE, "Standard Specifications for Public Key
              Cryptography", IEEE Std P1363,
              <http://ieeexplore.ieee.org/document/891000/>.

   [Menezes ]  Menezes, A. and B. Ustaoglu, "On reusing ephemeral keys in
              Diffie-Hellman key agreement protocols", International
              Journal of Applied Cryptography, Vol. 2, Issue 2,
              DOI 10.1504/IJACT.2010.038308, January 2010.

   [RFC4492]  Blake-Wilson, S., Bolyard, N., Gupta, V., Hawk, C., and B.
              Moeller, "Elliptic Curve Cryptography (ECC) Cipher Suites
              for Transport Layer Security (TLS)", RFC 4492,
              DOI 10.17487/RFC4492, May 2006,
              <https://www.rfc-editor.org/info/rfc4492>.

   [RFC7919]  Gillmor, D., "Negotiated Finite Field Diffie-Hellman
              Ephemeral Parameters for Transport Layer Security (TLS)",RFC 7919, DOI 10.17487/RFC7919, August 2016,
              <https://www.rfc-editor.org/info/rfc7919>.

   [TLS1.3]   Rescorla, E., "The Transport Layer Security (TLS) Protocol
              Version 1.3", Work in Progress, draft-ietf-tls-tls13-28,
              March 2018. 





# Referenced Sections from RFC 7748: Elliptic Curves for Security

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Since the initial standardization of Elliptic Curve Cryptography (ECC
   [RFC6090]) in [SEC1], there has been significant progress related to
   both efficiency and security of curves and implementations.  Notable
   examples are algorithms protected against certain side-channel
   attacks, various "special" prime shapes that allow faster modular
   arithmetic, and a larger set of curve models from which to choose.
   There is also concern in the community regarding the generation and
   potential weaknesses of the curves defined by NIST [NIST ].

   This memo specifies two elliptic curves ("curve25519" and "curve448")
   that lend themselves to constant-time implementation and an
   exception-free scalar multiplication that is resistant to a wide
   range of side-channel attacks, including timing and cache attacks.
   They are Montgomery curves (where v^2 = u^3 + A*u^2 + u) and thus
   have birationally equivalent Edwards versions.  Edwards curves
   support the fastest (currently known) complete formulas for the
   elliptic-curve group operations, specifically the Edwards curve
   x^2 + y^2 = 1 + d*x^2*y^2 for primes p when p = 3 mod 4, and the
   twisted Edwards curve -x^2 + y^2 = 1 + d*x^2*y^2 when p = 1 mod 4.
   The maps to/from the Montgomery curves to their (twisted) Edwards
   equivalents are also given. 
   This memo also specifies how these curves can be used with the
   Diffie-Hellman protocol for key agreement.

# Referenced Sections from RFC 8126: Guidelines for Writing an IANA Considerations Section in RFCs

The following sections were referenced. Remaining sections are not included.

### 1.1. Keep IANA Considerations for IANA

   The purpose of having a dedicated IANA Considerations section is to
   provide a single place to collect clear and concise information and
   instructions for IANA.  Technical documentation should reside in
   other parts of the document; the IANA Considerations should refer to
   these other sections by reference only (as needed).  Using the IANA
   Considerations section as primary technical documentation both hides
   it from the target audience of the document and interferes with
   IANA's review of the actions they need to take. 
   An ideal IANA Considerations section clearly enumerates and specifies
   each requested IANA action; includes all information IANA needs, such
   as the full names of all applicable registries; and includes clear
   references to elsewhere in the document for other information.

   The IANA actions are normally phrased as requests for IANA (such as,
   "IANA is asked to assign the value TBD1 from the Frobozz
   Registry..."); the RFC Editor will change those sentences to reflect
   the actions taken ("IANA has assigned the value 83 from the Frobozz
   Registry...").

# Referenced Sections from RFC 8773: TLS 1.3 Extension for Certificate-Based Authentication with an External Pre-Shared Key

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   The TLS 1.3 [RFC8446] handshake protocol provides two mutually
   exclusive forms of server authentication.  First, the server can be
   authenticated by providing a signature certificate and creating a
   valid digital signature to demonstrate that it possesses the
   corresponding private key.  Second, the server can be authenticated
   by demonstrating that it possesses a pre-shared key (PSK) that was
   established by a previous handshake.  A PSK that is established in
   this fashion is called a resumption PSK.  A PSK that is established
   by any other means is called an external PSK.  This document
   specifies a TLS 1.3 extension permitting certificate-based server
   authentication to be combined with an external PSK as an input to the
   TLS 1.3 key schedule.

   Several implementors wanted to gain more experience with this
   specification before producing a Standards Track RFC.  As a result,
   this specification is being published as an Experimental RFC to
   enable interoperable implementations and gain deployment and
   operational experience.

# Referenced Sections from RFC 6091: Using OpenPGP Keys for Transport Layer Security (TLS) Authentication

The following sections were referenced. Remaining sections are not included.

### 3.3. Server Certificate

   The contents of the certificate message sent from server to client
   and vice versa are determined by the negotiated certificate type and
   the selected cipher suite's key exchange algorithm.

   If the OpenPGP certificate type is negotiated, then it is required to
   present an OpenPGP certificate in the certificate message.  The
   certificate must contain a public key that matches the selected key
   exchange algorithm, as shown below.

      Key Exchange Algorithm    OpenPGP Certificate Type

      RSA                       RSA public key that can be used for
                                encryption.

      DHE_DSS                   DSA public key that can be used for
                                authentication.

      DHE_RSA                   RSA public key that can be used for
                                authentication.

   An OpenPGP certificate appearing in the certificate message is sent
   using the binary OpenPGP format.  The certificate MUST contain all
   the elements required by Section 11.1 of [RFC4880].

   OpenPGP certificates to be transferred are placed in the Certificate
   structure and tagged with the OpenPGPCertDescriptorType
   "subkey_cert".  Since those certificates might contain several
   subkeys, the subkey ID to be used for this session is explicitly 
   specified in the OpenPGPKeyID field.  The key ID must be specified
   even if the certificate has only a primary key.  The peer, upon
   receiving this type, has to either use the specified subkey or
   terminate the session with a fatal alert of
   "unsupported_certificate".

   The option is also available to send an OpenPGP fingerprint, instead
   of sending the entire certificate, by using the
   "subkey_cert_fingerprint" tag.  This tag uses the
   OpenPGPSubKeyFingerprint structure and requires the primary key
   fingerprint to be specified, as well as the subkey ID to be used for
   this session.  The peer shall respond with a
   "certificate_unobtainable" fatal alert if the certificate with the
   given fingerprint cannot be found.  The "certificate_unobtainable"
   fatal alert is defined in Section 5 of [RFC6066].

   Implementations of this protocol MUST ensure that the sizes of key
   IDs and fingerprints in the OpenPGPSubKeyCert and
   OpenPGPSubKeyFingerprint structures comply with [RFC4880].  Moreover,
   it is RECOMMENDED that the keys to be used with this protocol have
   the authentication flag (0x20) set.

   The process of fingerprint generation is described in Section 12.2 of
   [RFC4880].

   The enumerated types "cert_fingerprint" and "cert" of
   OpenPGPCertDescriptorType that were defined in [RFC5081] are not used
   and are marked as obsolete by this document.  The "empty_cert" type
   has replaced "cert" and is a backward-compatible way to specify an
   empty certificate; "cert_fingerprint" MUST NOT be used with this
   updated specification, and hence that old alternative has been
   removed from the Certificate struct description. 
      enum {
           empty_cert(1),
           subkey_cert(2),
           subkey_cert_fingerprint(3),
           (255)
      } OpenPGPCertDescriptorType;

      uint24 OpenPGPEmptyCert = 0;

      struct {
          opaque OpenPGPKeyID<8..255>;
          opaque OpenPGPCert<0..2^24-1>;
      } OpenPGPSubKeyCert;

      struct {
          opaque OpenPGPKeyID<8..255>;
          opaque OpenPGPCertFingerprint<20..255>;
      } OpenPGPSubKeyFingerprint;

      struct {
           OpenPGPCertDescriptorType descriptorType;
           select (descriptorType) {
                case empty_cert: OpenPGPEmptyCert;
                case subkey_cert: OpenPGPSubKeyCert;
                case subkey_cert_fingerprint:
                    OpenPGPSubKeyCertFingerprint;
           }
      } Certificate;

# Referenced Sections from RFC 6960: X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP

The following sections were referenced. Remaining sections are not included.

#### 4.2.2. Notes on OCSP Responses





##### 4.2.2.1. Time

   Responses can contain four times -- thisUpdate, nextUpdate,
   producedAt, and revocationTime.  The semantics of these fields are
   defined in Section 2.4.  The format for GeneralizedTime is as
   specified in Section 4.1.2.5.2 of [RFC5280].

   The thisUpdate and nextUpdate fields define a recommended validity
   interval.  This interval corresponds to the {thisUpdate, nextUpdate}
   interval in CRLs.  Responses whose nextUpdate value is earlier than
   the local system time value SHOULD be considered unreliable.
   Responses whose thisUpdate time is later than the local system time
   SHOULD be considered unreliable.

   If nextUpdate is not set, the responder is indicating that newer
   revocation information is available all the time.

##### 4.2.2.2. Authorized Responders

   The key that signs a certificate's status information need not be the
   same key that signed the certificate.  It is necessary, however, to
   ensure that the entity signing this information is authorized to do
   so.  Therefore, a certificate's issuer MUST do one of the following:

   - sign the OCSP responses itself, or

   - explicitly designate this authority to another entity

   OCSP signing delegation SHALL be designated by the inclusion of
   id-kp-OCSPSigning in an extended key usage certificate extension
   included in the OCSP response signer's certificate.  This certificate
   MUST be issued directly by the CA that is identified in the request.

   The CA SHOULD use the same issuing key to issue a delegation
   certificate as that used to sign the certificate being checked for
   revocation.  Systems relying on OCSP responses MUST recognize a
   delegation certificate as being issued by the CA that issued the
   certificate in question only if the delegation certificate and the
   certificate being checked for revocation were signed by the same key. 
   Note: For backwards compatibility with RFC 2560 [RFC2560], it is not
         prohibited to issue a certificate for an Authorized Responder
         using a different issuing key than the key used to issue the
         certificate being checked for revocation.  However, such a
         practice is strongly discouraged, since clients are not
         required to recognize a responder with such a certificate as an
         Authorized Responder.

   id-kp-OCSPSigning OBJECT IDENTIFIER ::= {id-kp 9}

   Systems or applications that rely on OCSP responses MUST be capable
   of detecting and enforcing the use of the id-kp-OCSPSigning value as
   described above.  They MAY provide a means of locally configuring one
   or more OCSP signing authorities and specifying the set of CAs for
   which each signing authority is trusted.  They MUST reject the
   response if the certificate required to validate the signature on the
   response does not meet at least one of the following criteria:

   1. Matches a local configuration of OCSP signing authority for the
      certificate in question, or

   2. Is the certificate of the CA that issued the certificate in
      question, or

   3. Includes a value of id-kp-OCSPSigning in an extended key usage
      extension and is issued by the CA that issued the certificate in
      question as stated above.

   Additional acceptance or rejection criteria may apply to either the
   response itself or to the certificate used to validate the signature
   on the response.

###### 4.2.2.2.1. Revocation Checking of an Authorized Responder

   Since an authorized OCSP responder provides status information for
   one or more CAs, OCSP clients need to know how to check that an
   Authorized Responder's certificate has not been revoked.  CAs may
   choose to deal with this problem in one of three ways:

   - A CA may specify that an OCSP client can trust a responder for the
     lifetime of the responder's certificate.  The CA does so by
     including the extension id-pkix-ocsp-nocheck.  This SHOULD be a
     non-critical extension.  The value of the extension SHALL be NULL.
     CAs issuing such a certificate should realize that a compromise of
     the responder's key is as serious as the compromise of a CA key 
     used to sign CRLs, at least for the validity period of this
     certificate.  CAs may choose to issue this type of certificate with
     a very short lifetime and renew it frequently.

     id-pkix-ocsp-nocheck OBJECT IDENTIFIER ::= { id-pkix-ocsp 5 }

   - A CA may specify how the responder's certificate is to be checked
     for revocation.  This can be done by using CRL Distribution Points
     if the check should be done using CRLs, or by using Authority
     Information Access if the check should be done in some other way.
     Details for specifying either of these two mechanisms are available
     in [RFC5280].

   - A CA may choose not to specify any method of revocation checking
     for the responder's certificate, in which case it would be up to
     the OCSP client's local security policy to decide whether that
     certificate should be checked for revocation or not.

##### 4.2.2.3. Basic Response

   The basic response type contains:

   o  the version of the response syntax, which MUST be v1 (value is 0)
      for this version of the basic response syntax;

   o  either the name of the responder or a hash of the responder's
      public key as the ResponderID;

   o  the time at which the response was generated;

   o  responses for each of the certificates in a request;

   o  optional extensions;

   o  a signature computed across a hash of the response; and

   o  the signature algorithm OID.

   The purpose of the ResponderID information is to allow clients to
   find the certificate used to sign a signed OCSP response.  Therefore,
   the information MUST correspond to the certificate that was used to
   sign the response.

   The responder MAY include certificates in the certs field of
   BasicOCSPResponse that help the OCSP client verify the responder's
   signature. 
   The response for each of the certificates in a request consists of:

   o  an identifier of the certificate for which revocation status
      information is being provided (i.e., the target certificate);

   o  the revocation status of the certificate (good, revoked, or
      unknown); if revoked, it indicates the time at which the
      certificate was revoked and, optionally, the reason why it was
      revoked;

   o  the validity interval of the response; and

   o  optional extensions.

   The response MUST include a SingleResponse for each certificate in
   the request.  The response SHOULD NOT include any additional
   SingleResponse elements, but, for example, OCSP responders that
   pre-generate status responses might include additional SingleResponse
   elements if necessary to improve response pre-generation performance
   or cache efficiency (according to [RFC5019], Section 2.2.1).

# Referenced Sections from RFC 6961: The Transport Layer Security (TLS) Multiple Certificate Status Request Extension

The following sections were referenced. Remaining sections are not included.

## 2. Multiple Certificate Status Extension





### 2.1. New Extension

   The extension defined by this document is indicated by
   "status_request_v2" in the ExtensionType enum (originally defined by
   [RFC6066]), which uses the following value:

     enum {
       status_request_v2(17), (65535)
     } ExtensionType;

### 2.2. Multiple Certificate Status Request Record

   Clients that support a certificate status protocol like OCSP may send
   the "status_request_v2" extension to the server in order to use the
   TLS handshake to transfer such data instead of downloading it through
   separate connections.  When using this extension, the
   "extension_data" field (defined in Section 7.4.1.4 of [RFC5246]) of
   the extension SHALL contain a CertificateStatusRequestListV2 where:

     struct {
       CertificateStatusType status_type;
       uint16 request_length; /* Length of request field in bytes */
       select (status_type) {
         case ocsp: OCSPStatusRequest;
         case ocsp_multi: OCSPStatusRequest;
       } request;
     } CertificateStatusRequestItemV2;

     enum { ocsp(1), ocsp_multi(2), (255) } CertificateStatusType;
     struct {
       ResponderID responder_id_list<0..2^16-1>;
       Extensions request_extensions;
     } OCSPStatusRequest;

     opaque ResponderID<1..2^16-1>;
     opaque Extensions<0..2^16-1>;

     struct {
       CertificateStatusRequestItemV2
                        certificate_status_req_list<1..2^16-1>;
     } CertificateStatusRequestListV2;

   In the OCSPStatusRequest (originally defined by [RFC6066]), the
   "ResponderID" provides a list of OCSP responders that the client
   trusts.  A zero-length "responder_id_list" sequence has the special
   meaning that the responders are implicitly known to the server, e.g.,
   by prior arrangement, or are identified by the certificates used by
   the server.  "Extensions" is a DER encoding [X.690] of the OCSP
   request extensions, and if the server supports the forwarding of OCSP
   request extensions, this value MUST be forwarded without
   modification.

   Both "ResponderID" and "Extensions" are DER-encoded ASN.1 types as
   defined in [RFC6960].  "Extensions" is imported from [RFC5280].  A
   zero-length "request_extensions" value means that there are no
   extensions (as opposed to a DER-encoded zero-length ASN.1 SEQUENCE,
   which is not valid for the "Extensions" type).

   Servers that support a client's selection of responders using the
   ResponderID field could implement this selection by matching the
   responder ID values from the client's list with the ResponderIDs of
   known OCSP responders, either by using a binary compare of the values
   or a hash calculation and compare method.

   In the case of the "id-pkix-ocsp-nonce" OCSP extension, [RFC2560] is
   unclear about its encoding; for clarification, the nonce MUST be a
   DER-encoded OCTET STRING, which is encapsulated as another OCTET
   STRING (note that implementations based on an existing OCSP client
   will need to be checked for conformance to this requirement).  This
   has been clarified in [RFC6960].

   The items in the list of CertificateStatusRequestItemV2 entries are
   ordered according to the client's preference (favorite choice first).

   A server that receives a client hello containing the
   "status_request_v2" extension MAY return a suitable certificate
   status response message to the client along with the server's 
   certificate message.  If OCSP is requested, it SHOULD use the
   information contained in the extension when selecting an OCSP
   responder and SHOULD include request_extensions in the OCSP request.

   The server returns a certificate status response along with its
   certificate by sending a "CertificateStatus" message (originally
   defined by [RFC6066]) immediately after the "Certificate" message
   (Section 7.4.2 of [RFC5246]) (and before any "ServerKeyExchange" or
   "CertificateRequest" messages).  If a server returns a
   "CertificateStatus" message in response to a "status_request_v2"
   request, then the server MUST have included an extension of type
   "status_request_v2" with empty "extension_data" in the extended
   server hello.

   The "CertificateStatus" message is conveyed using the handshake
   message type "certificate_status" (defined in [RFC6066]) as follows
   (updated from the definition in [RFC6066]):

     struct {
       CertificateStatusType status_type;
       select (status_type) {
         case ocsp: OCSPResponse;
         case ocsp_multi: OCSPResponseList;
       } response;
     } CertificateStatus;

     opaque OCSPResponse<0..2^24-1>;

     struct {
       OCSPResponse ocsp_response_list<1..2^24-1>;
     } OCSPResponseList;

   An "OCSPResponse" element (originally defined by [RFC6066]) contains
   a complete, DER-encoded OCSP response (using the ASN.1 [X.680] type
   OCSPResponse defined in [RFC6960]).  Only one OCSP response, with a
   length of at least one byte, may be sent for status_type "ocsp".

   An "ocsp_response_list" contains a list of "OCSPResponse" elements,
   as specified above, each containing the OCSP response for the
   matching corresponding certificate in the server's Certificate TLS
   handshake message.  That is, the first entry is the OCSP response for
   the first certificate in the Certificate list, the second entry is
   the response for the second certificate, and so on.  The list MAY
   contain fewer OCSP responses than there were certificates in the
   Certificate handshake message, but there MUST NOT be more responses
   than there were certificates in the list.  Individual elements of the
   list MAY have a length of 0 (zero) bytes if the server does not have
   the OCSP response for that particular certificate stored, in which 
   case the client MUST act as if a response was not received for that
   particular certificate.  If the client receives a
   "ocsp_response_list" that does not contain a response for one or more
   of the certificates in the completed certificate chain, the client
   SHOULD attempt to validate the certificate using an alternative
   retrieval method, such as downloading the relevant CRL; OCSP SHOULD
   in this situation only be used for the end-entity certificate, not
   intermediate CA certificates, for reasons stated above.

   Note that a server MAY also choose not to send a "CertificateStatus"
   message, even if it has received a "status_request_v2" extension in
   the client hello message and has sent a "status_request_v2" extension
   in the server hello message.  Additionally, note that a server MUST
   NOT send the "CertificateStatus" message unless it received either a
   "status_request" or "status_request_v2" extension in the client hello
   message and sent a corresponding "status_request" or
   "status_request_v2" extension in the server hello message.

   Clients requesting an OCSP response and receiving one or more OCSP
   responses in a "CertificateStatus" message MUST check the OCSP
   response(s) and abort the handshake if the response is a "revoked"
   status or other unacceptable responses (as determined by client
   policy) with a bad_certificate_status_response(113) alert.  This
   alert is always fatal.

   If the OCSP response received from the server does not result in a
   definite "good" or "revoked" status, it is inconclusive.  A TLS
   client in such a case MAY check the validity of the server
   certificate through other means, e.g., by directly querying the
   certificate issuer.  If such processing still results in an
   inconclusive response, then the application using the TLS connection
   will have to decide whether to close the connection or not.  Note
   that this problem cannot be decided by the generic TLS client code
   without information from the application.  If the application doesn't
   provide any such information, then the client MUST abort the
   connection, since the server certificate has not been sufficiently
   validated.

   An example of where the application might wish to continue is with
   EAP-TLS (Extensible Authentication Protocol - TLS), where the
   application can use another mechanism to check the status of a
   certificate once it obtains network access.  In this case, the
   application could have the client continue with the handshake, but it
   MUST NOT disclose a username and password until it has fully
   validated the server certificate. 





# Referenced Sections from RFC 2104: HMAC: Keyed-Hashing for Message Authentication

The following sections were referenced. Remaining sections are not included.

## 2. Definition of HMAC

   The definition of HMAC requires a cryptographic hash function, which
   we denote by H, and a secret key K. We assume H to be a cryptographic
   hash function where data is hashed by iterating a basic compression
   function on blocks of data.   We denote by B the byte-length of such
   blocks (B=64 for all the above mentioned examples of hash functions),
   and by L the byte-length of hash outputs (L=16 for MD5, L=20 for
   SHA-1).  The authentication key K can be of any length up to B, the
   block length of the hash function.  Applications that use keys longer
   than B bytes will first hash the key using H and then use the
   resultant L byte string as the actual key to HMAC. In any case the
   minimal recommended length for K is L bytes (as the hash output
   length). See section 3 for more information on keys.

   We define two fixed and different strings ipad and opad as follows
   (the 'i' and 'o' are mnemonics for inner and outer):

                   ipad = the byte 0x36 repeated B times
                  opad = the byte 0x5C repeated B times.

   To compute HMAC over the data `text' we perform

                    H(K XOR opad, H(K XOR ipad, text))

   Namely,

    (1) append zeros to the end of K to create a B byte string
        (e.g., if K is of length 20 bytes and B=64, then K will be
         appended with 44 zero bytes 0x00)
    (2) XOR (bitwise exclusive-OR) the B byte string computed in step
        (1) with ipad
    (3) append the stream of data 'text' to the B byte string resulting
        from step (2)
    (4) apply H to the stream generated in step (3)
    (5) XOR (bitwise exclusive-OR) the B byte string computed in
        step (1) with opad
    (6) append the H result from step (4) to the B byte string
        resulting from step (5)
    (7) apply H to the stream generated in step (6) and output
        the result

   For illustration purposes, sample code based on MD5 is provided as an
   appendix. 





# Referenced Sections from RFC 8305: Happy Eyeballs Version 2: Better Connectivity Using Concurrency

The following sections were referenced. Remaining sections are not included.

### 9.1. Path Maximum Transmission Unit Discovery

   Since Happy Eyeballs is only active during the initial handshake and
   TCP does not pass the initial handshake, issues related to MTU can be
   masked and go unnoticed during Happy Eyeballs.  Solving this issue is
   out of scope of this document.  One solution is to use "Packetization
   Layer Path MTU Discovery" [RFC4821].

# Summary of reference from Rivest, R., Shamir, A., and L. Adleman, "A method for obtaining digital signatures and public-key cryptosystems", Association for Computing Machinery (ACM), Communications of the ACM vol. 21, no. 2, pp. 120-126, DOI 10.1145/359340.359342, February 1978, <https://doi.org/10.1145/359340.359342>. (RSA)

The paper "A Method for Obtaining Digital Signatures and Public-Key Cryptosystems" by Rivest, Shamir, and Adleman, published in the Communications of the ACM in February 1978, introduces the RSA algorithm, a foundational public-key cryptosystem. This algorithm enables secure data transmission and digital signatures by utilizing the mathematical properties of large prime numbers and modular arithmetic.

The term "static RSA" is not explicitly mentioned in the original paper. In contemporary cryptographic discussions, "static RSA" often refers to RSA keys that remain constant over time, as opposed to "ephemeral RSA" keys, which are generated for temporary use in specific sessions to enhance security. The original RSA paper primarily focuses on the theoretical underpinnings and practical implementation of the RSA algorithm, without delving into the distinctions between static and ephemeral key usage. 

# Summary of reference from ACM.1978

The document "ACM.1978" refers to the paper titled "A Method for Obtaining Digital Signatures and Public-Key Cryptosystems" by Ronald L. Rivest, Adi Shamir, and Leonard Adleman, published in the Communications of the ACM in February 1978. This seminal paper introduces the RSA algorithm, which is foundational in public-key cryptography.

In the paper, the authors describe the RSA algorithm, which enables secure communication over insecure channels without the need for a shared secret key. The algorithm is based on the mathematical difficulty of factoring large composite numbers, specifically the product of two large prime numbers.

The RSA algorithm involves three main steps:

1. **Key Generation**: Select two large prime numbers, \( p \) and \( q \), and compute their product \( n = pq \). Choose an integer \( e \) such that \( 1 < e < \phi(n) \) and \( \gcd(e, \phi(n)) = 1 \), where \( \phi(n) = (p-1)(q-1) \). Compute \( d \) as the modular multiplicative inverse of \( e \) modulo \( \phi(n) \). The public key is \( (n, e) \), and the private key is \( d \).

2. **Encryption**: To encrypt a message \( M \), represent it as an integer \( m \) such that \( 0 \leq m < n \). Compute the ciphertext \( c \) as \( c = m^e \mod n \).

3. **Decryption**: To decrypt the ciphertext \( c \), compute \( m = c^d \mod n \), which retrieves the original message \( m \).

The paper also discusses the application of the RSA algorithm for digital signatures. In this context, the process is as follows:

1. **Signing**: The sender computes a hash of the message and then encrypts the hash using their private key to create the digital signature.

2. **Verification**: The receiver decrypts the digital signature using the sender's public key to retrieve the hash and compares it to the hash of the received message. If they match, the signature is valid, confirming the message's authenticity and integrity.

The authors highlight the security of the RSA algorithm, which relies on the computational difficulty of factoring large numbers. They also discuss potential attacks and the importance of choosing large prime numbers to ensure security.

This paper laid the groundwork for modern public-key cryptography and digital signatures, significantly impacting secure communications and data integrity in the digital age. 

# Summary of reference from NIST FIPS 180-4

The Secure Hash Standard (SHS), as defined in NIST's FIPS 180-4, specifies a suite of cryptographic hash functions designed to produce a condensed representation, known as a message digest, of electronic data. These hash functions are integral to various cryptographic applications, including digital signatures, message authentication codes, and random number generation.

FIPS 180-4 encompasses the following hash algorithms:

- **SHA-1**: Generates a 160-bit message digest for messages less than 2^64 bits in length.

- **SHA-224**: Produces a 224-bit digest for messages under 2^64 bits.

- **SHA-256**: Yields a 256-bit digest for messages less than 2^64 bits.

- **SHA-384**: Creates a 384-bit digest for messages under 2^128 bits.

- **SHA-512**: Outputs a 512-bit digest for messages less than 2^128 bits.

- **SHA-512/224** and **SHA-512/256**: Derivatives of SHA-512, producing 224-bit and 256-bit digests, respectively, for messages under 2^128 bits.

These algorithms are deemed secure because it is computationally infeasible to find a message corresponding to a given digest or to find two different messages that produce the same digest. Any alteration to a message will, with high probability, result in a different digest, thereby ensuring data integrity.

In March 2023, NIST announced plans to revise FIPS 180-4. The proposed updates include removing the SHA-1 specification, incorporating guidance from NIST Special Publication 800-107, enhancing editorial quality, and updating references. This revision aims to strengthen the standard by phasing out SHA-1 due to its known vulnerabilities and aligning the document with current cryptographic practices. ([nist.gov](https://www.nist.gov/news-events/news/2023/03/decision-revise-fips-180-4-secure-hash-standard-shs?utm_source=openai))

For detailed specifications and guidance on implementing these hash functions, refer to the full FIPS 180-4 document. ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.180-4.pdf?utm_source=openai)) 

# Summary of reference from "Secure hash standard", National Institute of Standards and Technology (U.S.), DOI 10.6028/nist.fips.180-4, 2015, <https://doi.org/10.6028/nist.fips.180-4>. (SHS)

The Secure Hash Standard (SHS), as defined in FIPS PUB 180-4 by the National Institute of Standards and Technology (NIST), specifies a suite of cryptographic hash algorithms designed to produce a condensed representation, or digest, of electronic data. These algorithms are integral to various cryptographic applications, including digital signatures, message authentication codes, and the generation of random numbers.

The standard encompasses the following hash algorithms:

- **SHA-1**: Produces a 160-bit message digest.
- **SHA-224**: Produces a 224-bit message digest.
- **SHA-256**: Produces a 256-bit message digest.
- **SHA-384**: Produces a 384-bit message digest.
- **SHA-512**: Produces a 512-bit message digest.
- **SHA-512/224**: Produces a 224-bit message digest using a truncated version of SHA-512.
- **SHA-512/256**: Produces a 256-bit message digest using a truncated version of SHA-512.

These algorithms are deemed secure because it is computationally infeasible to:

1. Find a message that corresponds to a given message digest.
2. Find two different messages that produce the same message digest.

Any alteration to a message will, with a very high probability, result in a different message digest, thereby ensuring data integrity.

It's important to note that while SHA-1 is included in the standard, its use has been deprecated due to identified vulnerabilities. NIST has initiated a process to revise FIPS 180-4 to remove the SHA-1 specification and incorporate guidance from NIST Special Publication 800-107, aiming to enhance the standard's editorial quality and update its references. ([nist.gov](https://www.nist.gov/news-events/news/2023/03/decision-revise-fips-180-4-secure-hash-standard-shs?utm_source=openai))

For the most current information and guidance on the use of these hash algorithms, refer to the official NIST publications and announcements. 

# Summary of reference from ITU-T, "Information technology - ASN.1 encoding Rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)", ITU-T X.690 , February 2021, <https://www.itu.int/rec/T-REC-X.690-202102-I/en >. (X690)

The ITU-T Recommendation X.690, titled "Information technology – ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)," provides standardized encoding rules for data structures defined using Abstract Syntax Notation One (ASN.1). This document specifies three primary encoding rules:

1. **Basic Encoding Rules (BER):** These are the original rules for encoding ASN.1 data structures into a binary format. BER provides flexibility by allowing multiple encoding options for certain data types, which can lead to variations in the encoded output for the same data.

2. **Canonical Encoding Rules (CER):** CER is a restricted subset of BER that eliminates encoding options to ensure a unique encoding for each ASN.1 data structure. It is particularly useful in scenarios where a consistent encoding is necessary, such as in cryptographic applications.

3. **Distinguished Encoding Rules (DER):** Similar to CER, DER is another restricted subset of BER that ensures a unique encoding. The primary difference between CER and DER lies in their handling of length encoding and string encoding forms. DER is commonly used in digital certificates and cryptographic protocols.

These encoding rules are essential for the consistent and interoperable representation of data across different systems and applications. The X.690 standard is part of a series of ITU-T Recommendations that define ASN.1 and its encoding rules, ensuring reliable data communication in various technological domains. 

# Summary of reference from "Digital Signature Standard (DSS)", National Institute of Standards and Technology (U.S.), DOI 10.6028/nist.fips.186-5, February 2023, <https://doi.org/10.6028/nist.fips.186-5>. (DSS)

The Digital Signature Standard (DSS), as specified in FIPS 186-5, defines a suite of algorithms for generating and verifying digital signatures. These signatures are essential for ensuring data integrity, authenticating the identity of the signatory, and providing non-repudiation, meaning the signatory cannot deny the authenticity of the signature at a later time. ([nist.gov](https://www.nist.gov/publications/digital-signature-standard-dss-3?utm_source=openai))

FIPS 186-5 specifies three approved digital signature algorithms:

1. **Rivest-Shamir-Adleman (RSA) Algorithm**: A widely used public-key cryptosystem that enables secure data transmission.

2. **Elliptic Curve Digital Signature Algorithm (ECDSA)**: An algorithm that utilizes elliptic curve cryptography to generate digital signatures, offering security equivalent to RSA but with smaller key sizes.

3. **Edwards Curve Digital Signature Algorithm (EdDSA)**: An algorithm based on Edwards curves, known for its performance, side-channel resistance, and simpler implementation compared to traditional curves. ([nist.gov](https://www.nist.gov/news-events/news/2023/02/nist-revises-digital-signature-standard-dss-and-publishes-guideline?utm_source=openai))

Notably, the Digital Signature Algorithm (DSA), included in previous versions of the standard, is retained in FIPS 186-5 solely for the purpose of verifying existing signatures and is not approved for generating new digital signatures. ([nist.gov](https://www.nist.gov/news-events/news/2023/02/nist-revises-digital-signature-standard-dss-and-publishes-guideline?utm_source=openai))

The standard also emphasizes the use of approved hash functions, such as those specified in FIPS 180 (Secure Hash Standard) and FIPS 202 (SHA-3 Standard), in conjunction with the digital signature algorithms to ensure the security and integrity of the digital signatures. ([nist.gov](https://www.nist.gov/publications/digital-signature-standard-dss-3?utm_source=openai))

For detailed information and technical specifications, you can refer to the full document available at: ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-5.pdf?utm_source=openai)) 

# Summary of reference from Chen, L., Moody, D., Regenscheid, A., Robinson, A., and K. Randall, "Recommendations for Discrete Logarithm-based Cryptography:: Elliptic Curve Domain Parameters", National Institute of Standards and Technology, DOI 10.6028/nist.sp.800-186, February 2023, <https://doi.org/10.6028/nist.sp.800-186>. (ECDP)

The National Institute of Standards and Technology (NIST) Special Publication 800-186, titled "Recommendations for Discrete Logarithm-based Cryptography: Elliptic Curve Domain Parameters," provides comprehensive guidelines on elliptic curve domain parameters for discrete logarithm-based cryptographic applications. ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai))

**Key Highlights:**

- **Recommended Elliptic Curves:** The publication specifies elliptic curves suitable for U.S. Government use, including traditional Weierstrass curves over prime and binary fields, as well as two newly specified Edwards curves. These Edwards curves offer enhanced performance, improved resistance to side-channel attacks, and simpler implementation compared to traditional curves. ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai))

- **Deprecation of Binary Field Curves:** While the document includes specifications for elliptic curves over binary fields, it deprecates their use due to limited industry adoption. The use of curves over prime fields is strongly recommended. ([csrc.nist.gov](https://csrc.nist.gov/News/2023/nist-releases-fips-186-5-and-sp-800-186?utm_source=openai))

- **Interoperability:** The newly specified Edwards curves are designed to be interoperable with those specified by the Crypto Forum Research Group (CFRG) of the Internet Engineering Task Force (IETF). ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai))

- **Security Considerations:** The recommended elliptic curves are not expected to provide resistance against attacks from large-scale quantum computers. NIST plans to specify quantum-resistant digital signature algorithms in future publications. ([csrc.nist.gov](https://csrc.nist.gov/News/2023/nist-releases-fips-186-5-and-sp-800-186?utm_source=openai))

For detailed information, including the mathematical specifications of the recommended curves and their domain parameters, please refer to the full document available at: ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-186.pdf?utm_source=openai)) 

# Summary of reference from "Secure hash standard", National Institute of Standards and Technology (U.S.), DOI 10.6028/nist.fips.180-4, 2015, <https://doi.org/10.6028/nist.fips.180-4>. (SHS)

The "Secure Hash Standard" (FIPS PUB 180-4), published by the National Institute of Standards and Technology (NIST) in August 2015, specifies several secure hash algorithms (SHAs) used to generate message digests. These digests help detect any changes to messages since their creation. The standard includes the following hash algorithms:

- **SHA-1**: Produces a 160-bit message digest.
- **SHA-224**: Produces a 224-bit message digest.
- **SHA-256**: Produces a 256-bit message digest.
- **SHA-384**: Produces a 384-bit message digest.
- **SHA-512**: Produces a 512-bit message digest.
- **SHA-512/224**: Produces a 224-bit message digest.
- **SHA-512/256**: Produces a 256-bit message digest.

These algorithms are designed to be computationally infeasible to reverse-engineer a message from its digest or to find two different messages that produce the same digest. They are commonly used in conjunction with other cryptographic methods, such as digital signatures and keyed-hash message authentication codes, and in random number generation. ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf?utm_source=openai))

In March 2023, NIST announced plans to revise FIPS 180-4. The proposed revisions include removing the SHA-1 specification, incorporating guidance from NIST Special Publication 800-107, enhancing editorial quality, and updating references. This decision reflects the ongoing evolution of cryptographic standards to address emerging security challenges. ([nist.gov](https://www.nist.gov/news-events/news/2023/03/decision-revise-fips-180-4-secure-hash-standard-shs?utm_source=openai)) 

# Summary of reference from NIST.FIPS.180-4

The Secure Hash Standard (SHS), as defined in NIST FIPS 180-4, specifies a set of cryptographic hash functions designed to produce a condensed representation, known as a message digest, of electronic data. These hash functions are integral to various cryptographic applications, including digital signatures, message authentication codes, and random number generation.

FIPS 180-4 outlines the following hash algorithms:

- **SHA-1**: Generates a 160-bit message digest for messages less than 2^64 bits in length.

- **SHA-224**: Produces a 224-bit digest for messages under 2^64 bits.

- **SHA-256**: Yields a 256-bit digest for messages under 2^64 bits.

- **SHA-384**: Creates a 384-bit digest for messages less than 2^128 bits.

- **SHA-512**: Delivers a 512-bit digest for messages under 2^128 bits.

- **SHA-512/224** and **SHA-512/256**: These variants generate 224-bit and 256-bit digests, respectively, for messages less than 2^128 bits.

Each algorithm processes input data to produce a fixed-size output, ensuring that any alteration in the input results in a significantly different digest. This property is crucial for verifying data integrity and authenticity.

In March 2023, NIST announced plans to revise FIPS 180-4. The proposed updates include:

1. Removing the SHA-1 specification due to its diminished security.

2. Incorporating guidance from NIST Special Publication 800-107, which provides recommendations for using approved hash algorithms.

3. Enhancing the standard's editorial quality.

4. Updating references to reflect current standards and practices.

These revisions aim to strengthen the security and applicability of the Secure Hash Standard in modern cryptographic systems. 

# Summary of reference from ACM.21.2

The document you're referring to is "A Method for Obtaining Digital Signatures and Public-Key Cryptosystems" by Rivest, Shamir, and Adleman, published in the February 1978 issue of Communications of the ACM (Volume 21, Issue 2, pages 120–126). ([cacm.acm.org](https://cacm.acm.org/research/a-method-for-obtaining-digital-signatures-and-public-key-cryptosystems-2/?utm_source=openai))

In this seminal paper, the authors introduce an encryption method with the novel property that publicly revealing an encryption key does not reveal the corresponding decryption key. This innovation eliminates the need for secure key transmission channels, as messages can be encrypted using a publicly disclosed key and decrypted only by the intended recipient who possesses the private key. Additionally, the paper presents a mechanism for digital signatures, where a message can be "signed" using a privately held decryption key. Anyone can verify this signature using the corresponding publicly revealed encryption key, ensuring that signatures cannot be forged and that a signer cannot later deny the validity of their signature. These concepts have profound applications in electronic mail and electronic funds transfer systems. ([cacm.acm.org](https://cacm.acm.org/research/a-method-for-obtaining-digital-signatures-and-public-key-cryptosystems-2/?utm_source=openai))

The encryption process involves representing a message as a number \( M \), raising \( M \) to a publicly specified power \( e \), and then taking the remainder when the result is divided by the publicly specified product \( n \) of two large secret prime numbers \( p \) and \( q \). Decryption is performed using a different, secret power \( d \), where \( e \times d \equiv 1 \mod ((p - 1) \times (q - 1)) \). The security of the system relies in part on the difficulty of factoring the published divisor \( n \). ([cacm.acm.org](https://cacm.acm.org/research/a-method-for-obtaining-digital-signatures-and-public-key-cryptosystems-2/?utm_source=openai))

This work laid the foundation for the RSA algorithm, which remains a cornerstone of modern cryptographic systems. 

# Summary of reference from Bhargavan, K. and G. Leurent, "Transcript Collision Attacks: Breaking Authentication in TLS, IKE, and SSH", Internet Society, Proceedings 2016 Network and Distributed System Security Symposium, DOI 10.14722/ndss.2016.23418, 2016, <https://doi.org/10.14722/ndss.2016.23418>. (SLOTH)

The paper "Transcript Collision Attacks: Breaking Authentication in TLS, IKE, and SSH" by Karthikeyan Bhargavan and Gaëtan Leurent introduces a class of attacks termed **transcript collision attacks**. These attacks exploit the vulnerabilities of weak hash functions, such as MD5 and SHA-1, within cryptographic protocols.

**Key Points:**

- **Transcript Collision Attacks:** These attacks leverage the ability to find hash collisions efficiently, allowing adversaries to manipulate protocol transcripts. This manipulation can lead to credential forwarding, impersonation, and downgrade attacks.

- **Affected Protocols:** The study demonstrates practical attacks on TLS 1.2 client authentication, TLS 1.3 server authentication, and TLS channel bindings. Additionally, it describes near-practical impersonation and downgrade attacks in TLS 1.1, IKEv2, and SSH-2.

- **Implications:** The findings challenge the assumption that weak hash functions are secure in certain protocol contexts. The authors advocate for the immediate discontinuation of MD5 and SHA-1 in all protocol applications.

- **Responsible Disclosure:** The vulnerabilities, collectively referred to as SLOTH, were responsibly disclosed, leading to security updates in several TLS libraries.

The paper underscores the critical need to eliminate weak hash functions from cryptographic protocols to prevent potential security breaches. 

# Summary of reference from ACM.359340.359342

I couldn't locate a document with the identifier "ACM.359340.359342" in the available sources. However, I can provide information on RSASSA-PSS (RSA Probabilistic Signature Scheme) from other reputable sources.

**Overview of RSASSA-PSS:**

RSASSA-PSS is a digital signature algorithm that enhances the security of the traditional RSA signature scheme by incorporating a probabilistic element. This approach offers a tighter security proof compared to deterministic methods like RSASSA-PKCS1-v1_5. ([ietf.org](https://www.ietf.org/rfc/rfc3447.txt.pdf?utm_source=openai))

**Key Features:**

- **Probabilistic Padding:** RSASSA-PSS introduces randomness in the padding process, making each signature unique even if the same message is signed multiple times. This randomness enhances security by preventing certain types of attacks.

- **Mask Generation Function (MGF):** The scheme employs a mask generation function, typically MGF1, which uses a hash function to generate a mask of a desired length. This function is crucial for the padding process and contributes to the scheme's security. ([ietf.org](https://www.ietf.org/rfc/rfc3447.txt.pdf?utm_source=openai))

- **Salt Value:** A random salt is used in the signature generation process, adding an additional layer of randomness and security. The length of the salt can be varied, but it is often recommended to match the length of the hash function's output. ([ietf.org](https://www.ietf.org/rfc/rfc3447.txt.pdf?utm_source=openai))

**Algorithm Identifiers and Parameters:**

In the context of Cryptographic Message Syntax (CMS), the algorithm identifier for RSASSA-PSS signatures is defined as:

```
id-RSASSA-PSS OBJECT IDENTIFIER ::= {pkcs-1 10}
```

When this identifier is used for a signature, the `AlgorithmIdentifier` parameters field must contain `RSASSA-PSS-params`, which specify the hash algorithm, mask generation function, salt length, and trailer field. ([rfc-editor.org](https://www.rfc-editor.org/rfc/rfc4056?utm_source=openai))

**Security Considerations:**

Implementations must protect the RSA private key, as its compromise could allow attackers to forge signatures. Additionally, the generation of RSA private keys relies on random numbers; using inadequate pseudo-random number generators can result in weak keys. It's also recommended not to use the same key for both RSASSA-PSS and other signature schemes to prevent potential vulnerabilities. ([rfc-editor.org](https://www.rfc-editor.org/rfc/rfc4056?utm_source=openai))

For a comprehensive understanding of RSASSA-PSS, including its mathematical foundations and implementation details, refer to RFC 3447, which provides the full specifications of the RSA Cryptography Standard. ([ietf.org](https://www.ietf.org/rfc/rfc3447.txt.pdf?utm_source=openai)) 

# Summary of reference from ITU-T, "Information technology - ASN.1 encoding Rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)", ITU-T X.690 , February 2021, <https://www.itu.int/rec/T-REC-X.690-202102-I/en >. (X690)

The ITU-T Recommendation X.690, titled "Information technology – ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)," defines encoding rules for Abstract Syntax Notation One (ASN.1) data structures. These rules specify how to encode ASN.1-defined data types into a binary format suitable for transmission and storage.

**Basic Encoding Rules (BER):**
BER provides a flexible set of encoding rules that can be applied to values of types defined using ASN.1 notation. It allows multiple encoding options for certain data types, offering flexibility but potentially leading to variations in encoding the same data. ([itu.int](https://www.itu.int/rec/T-REC-X.690-202102-I/en?utm_source=openai))

**Canonical Encoding Rules (CER):**
CER imposes additional constraints on BER to ensure a unique encoding for each ASN.1 value. It uses the indefinite length form for encoding, making it more suitable for encoding large values. ([itu.int](https://www.itu.int/rec/T-REC-X.690-202102-I/en?utm_source=openai))

**Distinguished Encoding Rules (DER):**
DER also imposes constraints on BER to achieve a unique encoding but uses the definite length form. This makes DER more suitable for encoding smaller values and is commonly used in applications like digital certificates and cryptographic protocols. ([itu.int](https://www.itu.int/rec/T-REC-X.690-202102-I/en?utm_source=openai))

The key difference between CER and DER lies in their handling of length encoding: CER uses indefinite length encoding, while DER uses definite length encoding. Both CER and DER ensure that each ASN.1 value has a single, unambiguous encoding, which is essential for applications requiring consistent data representation. ([itu.int](https://www.itu.int/rec/T-REC-X.690-202102-I/en?utm_source=openai))

For a comprehensive understanding of these encoding rules, including detailed specifications and examples, refer to the full text of ITU-T Recommendation X.690. ([itu.int](https://www.itu.int/rec/T-REC-X.690-202102-I/en?utm_source=openai)) 

# Summary of reference from ITU-T, "Information Technology - Open Systems Interconnection - The Directory: Models", ISO/IEC 9594-2:2020 , October 2019. (X501)

ISO/IEC 9594-2:2020, also known as ITU-T Recommendation X.501, is part of the X.500 series of standards that define the Directory, a global, distributed database for managing information about objects such as people, organizations, and resources. This standard provides a conceptual and terminological framework for the Directory, detailing various models that support its structure and operation.

**Key Models Defined in the Standard:**

1. **Functional Model:** Describes the Directory as a set of one or more components, each being a Directory System Agent (DSA). This model outlines how the Directory's services are provided, emphasizing that the service is somewhat independent of the physical distribution of the Directory Information Base (DIB). ([iso.org](https://www.iso.org/standard/80314.html?utm_source=openai))

2. **Administrative Authority Model:** Defines how the Directory can be distributed administratively, specifying the principles according to which DIB entries and entry-copies may be distributed among DSAs. ([iso.org](https://www.iso.org/standard/80314.html?utm_source=openai))

3. **Generic Directory Information Models:** Describe the logical structure of the DIB from the perspectives of Directory and Administrative Users. In these models, the distribution of the Directory is transparent to users, presenting a unified view of the information. ([iso.org](https://www.iso.org/standard/80314.html?utm_source=openai))

4. **DSA Information Model:** Details the structure of user and operational information held within a DSA, supporting the distributed nature of the Directory. ([iso.org](https://www.iso.org/standard/80314.html?utm_source=openai))

5. **Operational Framework:** Provides the means by which specific forms of cooperation between DSAs are structured to achieve particular objectives, such as replication to improve overall Directory performance. ([iso.org](https://www.iso.org/standard/80314.html?utm_source=openai))

6. **Security Model:** Establishes a framework for access control mechanisms, including identifying access control schemes within portions of the Directory Information Tree (DIT) and defining flexible access control schemes suitable for various applications. It also provides a framework for protecting the confidentiality and integrity of directory operations using mechanisms like encryption and digital signatures. ([iso.org](https://www.iso.org/standard/80314.html?utm_source=openai))

These models collectively support the distributed and secure operation of the Directory, ensuring it can effectively manage and provide access to information across diverse systems and administrative domains. 

# Summary of reference from ITU-T, "Information technology - ASN.1 encoding Rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)", ITU-T X.690 , February 2021, <https://www.itu.int/rec/T-REC-X.690-202102-I/en >. (X690)

The Distinguished Encoding Rules (DER) are a subset of the Basic Encoding Rules (BER) defined in ITU-T Recommendation X.690. DER provides a unique encoding for each ASN.1 data value by imposing additional constraints on BER to ensure unambiguous serialization. This is particularly important in applications like digital signatures, where consistent encoding is essential.

Key characteristics of DER include:

- **Definite Length Encoding**: DER mandates the use of definite length encoding, specifying the exact number of bytes for the content. This contrasts with BER, which allows both definite and indefinite length encodings.

- **Primitive Encoding for Strings**: For string types such as `BIT STRING`, `OCTET STRING`, and restricted character strings, DER requires the use of primitive encoding forms, disallowing the constructed form permitted in BER.

- **Ordered Set Components**: In DER, the components of a `SET` are encoded in ascending order based on their tag values, ensuring a consistent and predictable encoding order.

These constraints ensure that each ASN.1 data value has a single, unambiguous DER encoding, which is crucial for cryptographic applications and data integrity verification. 

# Summary of reference from Chen, L., Moody, D., Regenscheid, A., Robinson, A., and K. Randall, "Recommendations for Discrete Logarithm-based Cryptography:: Elliptic Curve Domain Parameters", National Institute of Standards and Technology, DOI 10.6028/nist.sp.800-186, February 2023, <https://doi.org/10.6028/nist.sp.800-186>. (ECDP)

NIST Special Publication 800-186 (SP 800-186), titled "Recommendations for Discrete Logarithm-based Cryptography: Elliptic Curve Domain Parameters," was published in February 2023. ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai)) This document specifies the set of elliptic curves recommended for U.S. Government use. It includes the previously recommended Weierstrass curves defined over prime and binary fields and introduces two new Edwards curves. These Edwards curves offer increased performance, enhanced side-channel resistance, and simpler implementation compared to traditional curves. The publication also provides alternative representations for these new curves to allow for greater implementation flexibility. Notably, the new curves are interoperable with those specified by the Crypto Forum Research Group (CFRG) of the Internet Engineering Task Force (IETF). ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-186.pdf?utm_source=openai))

Additionally, SP 800-186 addresses the deprecation of elliptic curves over binary fields due to their limited use in the industry, strongly recommending the use of curves over prime fields instead. It's important to note that the algorithms specified in this standard are not expected to provide resistance against attacks from large-scale quantum computers. NIST plans to specify digital signature algorithms that offer security against quantum computing threats in future publications. ([nist.gov](https://www.nist.gov/news-events/news/2023/02/nist-revises-digital-signature-standard-dss-and-publishes-guideline?utm_source=openai)) 

# Summary of reference from IEEE.DH76

"New Directions in Cryptography" is a seminal paper authored by Whitfield Diffie and Martin E. Hellman, published in 1976 in the IEEE Transactions on Information Theory (Volume 22, Issue 6). This paper introduced the revolutionary concept of public-key cryptography, fundamentally transforming the field of cryptography.

Prior to this work, cryptographic systems relied on symmetric-key algorithms, where both the sender and receiver shared a secret key. Diffie and Hellman proposed a system where each user has a pair of keys: a public key, which can be shared openly, and a private key, which is kept secret. This innovation allows secure communication over insecure channels without the need for a shared secret key.

The paper also introduced the Diffie-Hellman key exchange protocol, enabling two parties to establish a shared secret key over an insecure channel. This protocol laid the groundwork for many modern cryptographic systems and is widely used in securing internet communications today.

The introduction of public-key cryptography addressed significant challenges in key distribution and digital signatures, leading to the development of various cryptographic protocols and standards that underpin modern secure communication. 

# Summary of reference from Diffie, W. and M. Hellman, "New directions in cryptography", Institute of Electrical and Electronics Engineers (IEEE), IEEE Transactions on Information Theory vol. 22, no. 6, pp. 644-654, DOI 10.1109/tit.1976.1055638, November 1976, <https://doi.org/10.1109/tit.1976.1055638>. (DH76)

In their seminal 1976 paper, "New Directions in Cryptography," Whitfield Diffie and Martin Hellman introduced the concept of public-key cryptography and the Diffie-Hellman key exchange protocol. This protocol allows two parties to securely establish a shared secret over an insecure communication channel.

The Diffie-Hellman key exchange operates as follows:

1. **Parameter Selection**: A large prime number \( p \) and a primitive root modulo \( p \), denoted as \( g \), are chosen. These values are publicly known.

2. **Private Key Selection**: Each party selects a private key.

3. **Public Key Computation**: Each party computes their public key by raising \( g \) to the power of their private key, modulo \( p \).

4. **Exchange and Shared Secret Computation**: The parties exchange their public keys and compute the shared secret by raising the received public key to the power of their private key, modulo \( p \).

The security of the Diffie-Hellman protocol relies on the computational difficulty of the discrete logarithm problem in finite fields. The choice of a large prime \( p \) and a suitable primitive root \( g \) is crucial to ensure the protocol's security.

For a detailed explanation and mathematical formulation, refer to the original paper:

Diffie, W., & Hellman, M. (1976). New directions in cryptography. *IEEE Transactions on Information Theory*, 22(6), 644-654.

Available at:

https://doi.org/10.1109/TIT.1976.1055638 

# Summary of reference from Barker, E., Chen, L., Roginsky, A., Vassilev, A., and R. Davis, "Recommendation for pair-wise key-establishment schemes using discrete logarithm cryptography", National Institute of Standards and Technology, DOI 10.6028/nist.sp.800-56ar3, April 2018, <https://doi.org/10.6028/nist.sp.800-56ar3>. (KEYAGREEMENT)

The "Recommendation for Pair-Wise Key-Establishment Schemes Using Discrete Logarithm Cryptography" (NIST Special Publication 800-56A Revision 3) provides guidelines for establishing shared secret keys between two parties using discrete logarithm cryptography. This includes methods based on the Diffie-Hellman (DH) and Menezes-Qu-Vanstone (MQV) algorithms over finite fields and elliptic curves. ([nist.gov](https://www.nist.gov/publications/recommendation-pair-wise-key-establishment-schemes-using-discrete-logarithm-1?utm_source=openai))

The document specifies approved key-agreement schemes that rely on the intractability of the discrete logarithm problem, ensuring secure key establishment. It also discusses the selection and generation of domain parameters, emphasizing the use of specific safe-prime groups and commonly used elliptic curves to enhance security. ([csrc.nist.gov](https://csrc.nist.gov/pubs/sp/800/56/a/r3/final?utm_source=openai))

Additionally, the publication addresses key confirmation methods to assure both parties that they share the same keying material, and outlines security properties associated with each scheme. It also provides practical recommendations for implementers, such as securely destroying sensitive data after its use to prevent unauthorized access. ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-56Ar3.pdf?utm_source=openai))

For a comprehensive understanding of these recommendations, you can access the full document here: ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-56Ar3.pdf?utm_source=openai)) 

# Summary of reference from Chen, L., Moody, D., Regenscheid, A., Robinson, A., and K. Randall, "Recommendations for Discrete Logarithm-based Cryptography:: Elliptic Curve Domain Parameters", National Institute of Standards and Technology, DOI 10.6028/nist.sp.800-186, February 2023, <https://doi.org/10.6028/nist.sp.800-186>. (ECDP)

The National Institute of Standards and Technology (NIST) Special Publication 800-186, titled "Recommendations for Discrete Logarithm-based Cryptography: Elliptic Curve Domain Parameters," provides comprehensive guidelines on elliptic curve domain parameters for discrete logarithm-based cryptographic applications. ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai))

**Key Highlights:**

- **Recommended Elliptic Curves:** The publication specifies elliptic curves suitable for U.S. Government use, including traditional Weierstrass curves over prime and binary fields, as well as two newly specified Edwards curves. These Edwards curves offer enhanced performance, improved resistance to side-channel attacks, and simpler implementation compared to traditional curves. ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai))

- **Deprecation of Binary Field Curves:** While the document includes specifications for elliptic curves over binary fields, it deprecates their use due to limited industry adoption. The use of curves over prime fields is strongly recommended. ([csrc.nist.gov](https://csrc.nist.gov/News/2023/nist-releases-fips-186-5-and-sp-800-186?utm_source=openai))

- **Interoperability:** The newly specified Edwards curves are designed to be interoperable with those specified by the Crypto Forum Research Group (CFRG) of the Internet Engineering Task Force (IETF). ([nist.gov](https://www.nist.gov/publications/recommendations-discrete-logarithm-based-cryptography-elliptic-curve-domain-parameters?utm_source=openai))

- **Security Considerations:** The recommended elliptic curves are not expected to provide resistance against attacks from large-scale quantum computers. NIST plans to specify quantum-resistant digital signature algorithms in future publications. ([csrc.nist.gov](https://csrc.nist.gov/News/2023/nist-releases-fips-186-5-and-sp-800-186?utm_source=openai))

For detailed information, including the mathematical specifications of the recommended curves and their domain parameters, please refer to the full document available at: ([nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-186.pdf?utm_source=openai)) 

# Summary of reference from ACM.359340.359342

The document identified by DOI 10.1145/359340.359342 is the seminal paper titled "A Method for Obtaining Digital Signatures and Public-Key Cryptosystems," authored by Ronald L. Rivest, Adi Shamir, and Leonard Adleman, published in the February 1978 issue of *Communications of the ACM*. ([colab.ws](https://colab.ws/articles/10.1145%2F359340.359342?utm_source=openai))

In this paper, the authors introduce the RSA algorithm, a groundbreaking public-key cryptosystem that enables secure communication over insecure channels. The RSA algorithm is based on the mathematical challenge of factoring large composite numbers, which underpins its security. The paper details the algorithm's design, including key generation, encryption, and decryption processes, and discusses its potential applications in digital signatures and secure data transmission.

The RSA algorithm has since become a cornerstone of modern cryptography, widely used for securing sensitive data across various platforms and applications. 

# Summary of reference from ACM.359340.359342

The document identified by ACM reference number 359340.359342 is titled "A Method for Obtaining Digital Signatures and Public-Key Cryptosystems," authored by Ronald L. Rivest, Adi Shamir, and Leonard M. Adleman. Published in the February 1978 issue of *Communications of the ACM*, this seminal paper introduces the RSA algorithm, a foundational public-key cryptosystem that facilitates both secure data encryption and digital signatures. ([colab.ws](https://colab.ws/articles/10.1145%2F359340.359342?utm_source=openai))

In the context of digital signatures, the RSA algorithm enables a user to generate a signature by encrypting a message digest (a fixed-size hash of the message) with their private key. This signature can then be verified by others using the corresponding public key, ensuring the authenticity and integrity of the message. The security of RSA signatures relies on the computational difficulty of factoring large composite numbers, a problem that remains intractable with current classical computing methods.

The paper details the mathematical underpinnings of the RSA algorithm, including key generation, encryption, decryption, and signature processes. It also discusses the theoretical foundations that contribute to the algorithm's security, such as number theory and the challenges associated with integer factorization. This work has had a profound impact on the field of cryptography, establishing a framework for secure digital communication that is still widely used today. 

