Let us analyze section E of draft draft-ietf-tls-rfc8446bis-14. All references made by section E have also been included below.

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
# Referenced Sections from RFC 7627: Transport Layer Security (TLS) Session Hash and Extended Master Secret Extension

The following sections were referenced. Remaining sections are not included.

## 3. The TLS Session Hash

   When a full TLS handshake takes place, we define

         session_hash = Hash(handshake_messages)

   where "handshake_messages" refers to all handshake messages sent or
   received, starting at the ClientHello up to and including the
   ClientKeyExchange message, including the type and length fields of
   the handshake messages.  This is the concatenation of all the
   exchanged Handshake structures, as defined in Section 7.4 of
   [RFC5246].

   For TLS 1.2, the "Hash" function is the one defined in Section 7.4.9
   of [RFC5246] for the Finished message computation.  For all previous
   versions of TLS, the "Hash" function computes the concatenation of
   MD5 and SHA1.

   There is no "session_hash" for resumed handshakes, as they do not
   lead to the creation of a new session. 





## 4. The Extended Master Secret

   When the extended master secret extension is negotiated in a full
   handshake, the "master_secret" is computed as

   master_secret = PRF(pre_master_secret, "extended master secret",
                       session_hash)
                       [0..47];

   The extended master secret computation differs from that described in
   [RFC5246] in the following ways:

   o  The "extended master secret" label is used instead of "master
      secret".

   o  The "session_hash" is used instead of the "ClientHello.random" and
      "ServerHello.random".

   The "session_hash" depends upon a handshake log that includes
   "ClientHello.random" and "ServerHello.random", in addition to
   ciphersuites, key exchange information, and certificates (if any)
   from the client and server.  Consequently, the extended master secret
   depends upon the choice of all these session parameters.

   This design reflects the recommendation that keys should be bound to
   the security contexts that compute them [SP800-108].  The technique
   of mixing a hash of the key exchange messages into master key
   derivation is already used in other well-known protocols such as
   Secure Shell (SSH) [RFC4251].

   Clients and servers SHOULD NOT accept handshakes that do not use the
   extended master secret, especially if they rely on features like
   compound authentication that fall into the vulnerable cases described
   in Section 6.1.

## 5. Extension Negotiation





### 5.1. Extension Definition

   This document defines a new TLS extension, "extended_master_secret"
   (with extension type 0x0017), which is used to signal both client and
   server to use the extended master secret computation.  The
   "extension_data" field of this extension is empty.  Thus, the entire
   encoding of the extension is 00 17 00 00 (in hexadecimal.)

   Although this document refers only to TLS, the extension proposed
   here can also be used with Datagram TLS (DTLS) [RFC6347]. 
   If the client and server agree on this extension and a full handshake
   takes place, both client and server MUST use the extended master
   secret derivation algorithm, as defined in Section 4.  All other
   cryptographic computations remain unchanged.

### 5.2. Client and Server Behavior: Full Handshake

   In the following, we use the phrase "abort the handshake" as
   shorthand for terminating the handshake by sending a fatal
   "handshake_failure" alert.

   In all handshakes, a client implementing this document MUST send the
   "extended_master_secret" extension in its ClientHello.

   If a server implementing this document receives the
   "extended_master_secret" extension, it MUST include the extension in
   its ServerHello message.

   If both the ClientHello and ServerHello contain the extension, the
   new session uses the extended master secret computation.

   If the server receives a ClientHello without the extension, it SHOULD
   abort the handshake if it does not wish to interoperate with legacy
   clients.  If it chooses to continue the handshake, then it MUST NOT
   include the extension in the ServerHello.

   If a client receives a ServerHello without the extension, it SHOULD
   abort the handshake if it does not wish to interoperate with legacy
   servers.

   If the client and server choose to continue a full handshake without
   the extension, they MUST use the standard master secret derivation
   for the new session.  In this case, the new session is not protected
   by the mechanisms described in this document.  So, implementers
   should follow the guidelines in Section 5.4 to avoid dangerous usage
   scenarios.  In particular, the master secret derived from the new
   session should not be used for application-level authentication.

### 5.3. Client and Server Behavior: Abbreviated Handshake

   The client SHOULD NOT offer an abbreviated handshake to resume a
   session that does not use an extended master secret.  Instead, it
   SHOULD offer a full handshake.

   If the client chooses to offer an abbreviated handshake even for such
   sessions in order to support legacy insecure resumption, then the
   current connection is not protected by the mechanisms in this
   document.  So, the client should follow the guidelines in Section 5.4




   to avoid dangerous usage scenarios.  In particular, renegotiation is
   no longer secure on this connection, even if the client and server
   support the renegotiation indication extension [RFC5746].

   When offering an abbreviated handshake, the client MUST send the
   "extended_master_secret" extension in its ClientHello.

   If a server receives a ClientHello for an abbreviated handshake
   offering to resume a known previous session, it behaves as follows:

   o  If the original session did not use the "extended_master_secret"
      extension but the new ClientHello contains the extension, then the
      server MUST NOT perform the abbreviated handshake.  Instead, it
      SHOULD continue with a full handshake (as described in Section 5.2) to negotiate a new session.

   o  If the original session used the "extended_master_secret"
      extension but the new ClientHello does not contain it, the server
      MUST abort the abbreviated handshake.

   o  If neither the original session nor the new ClientHello uses the
      extension, the server SHOULD abort the handshake.  If it continues
      with an abbreviated handshake in order to support legacy insecure
      resumption, the connection is no longer protected by the
      mechanisms in this document, and the server should follow the
      guidelines in Section 5.4.

   o  If the new ClientHello contains the extension and the server
      chooses to continue the handshake, then the server MUST include
      the "extended_master_secret" extension in its ServerHello message.

   If a client receives a ServerHello that accepts an abbreviated
   handshake, it behaves as follows:

   o  If the original session did not use the "extended_master_secret"
      extension but the new ServerHello contains the extension, the
      client MUST abort the handshake.

   o  If the original session used the extension but the new ServerHello
      does not contain the extension, the client MUST abort the
      handshake.

   If the client and server continue the abbreviated handshake, they
   derive the connection keys for the new session as usual from the
   master secret of the original session. 





### 5.4. Interoperability Considerations

   To allow interoperability with legacy clients and servers, a TLS peer
   may decide to accept full handshakes that use the legacy master
   secret computation.  If so, they need to differentiate between
   sessions that use legacy and extended master secrets by adding a flag
   to the session state.

   If a client or server chooses to continue with a full handshake
   without the extended master secret extension, then the new session
   becomes vulnerable to the man-in-the-middle key synchronization
   attack described in Section 1.  Hence, the client or server MUST NOT
   export any key material based on the new master secret for any
   subsequent application-level authentication.  In particular, it MUST
   disable [RFC5705] and any Extensible Authentication Protocol (EAP)
   relying on compound authentication [COMPOUND-AUTH ].

   If a client or server chooses to continue an abbreviated handshake to
   resume a session that does not use the extended master secret, then
   the current connection becomes vulnerable to a man-in-the-middle
   handshake log synchronization attack as described in Section 1.
   Hence, the client or server MUST NOT use the current handshake's
   "verify_data" for application-level authentication.  In particular,
   the client MUST disable renegotiation and any use of the "tls-unique"
   channel binding [RFC5929] on the current connection.

   If the original session uses an extended master secret but the
   ClientHello or ServerHello in the abbreviated handshake does not
   include the extension, it MAY be safe to continue the abbreviated
   handshake since it is protected by the extended master secret of the
   original session.  This scenario may occur, for example, when a
   server that implements this extension establishes a session but the
   session is subsequently resumed at a different server that does not
   support the extension.  Since such situations are unusual and likely
   to be the result of transient or inadvertent misconfigurations, this
   document recommends that the client and server MUST abort such
   handshakes.

# Referenced Sections from RFC 5280: Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile

The following sections were referenced. Remaining sections are not included.

## 6. Certification Path Validation

   Certification path validation procedures for the Internet PKI are
   based on the algorithm supplied in [X.509].  Certification path
   processing verifies the binding between the subject distinguished
   name and/or subject alternative name and subject public key.  The
   binding is limited by constraints that are specified in the
   certificates that comprise the path and inputs that are specified by
   the relying party.  The basic constraints and policy constraints
   extensions allow the certification path processing logic to automate
   the decision making process.

   This section describes an algorithm for validating certification
   paths.  Conforming implementations of this specification are not
   required to implement this algorithm, but MUST provide functionality
   equivalent to the external behavior resulting from this procedure.
   Any algorithm may be used by a particular implementation so long as
   it derives the correct result.

   In Section 6.1, the text describes basic path validation.  Valid
   paths begin with certificates issued by a trust anchor.  The
   algorithm requires the public key of the CA, the CA's name, and any
   constraints upon the set of paths that may be validated using this
   key.

   The selection of a trust anchor is a matter of policy: it could be
   the top CA in a hierarchical PKI, the CA that issued the verifier's
   own certificate(s), or any other CA in a network PKI.  The path 
   validation procedure is the same regardless of the choice of trust
   anchor.  In addition, different applications may rely on different
   trust anchors, or may accept paths that begin with any of a set of
   trust anchors. Section 6.2 describes methods for using the path validation algorithm
   in specific implementations. Section 6.3 describes the steps necessary to determine if a
   certificate is revoked when CRLs are the revocation mechanism used by
   the certificate issuer.

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

### 6.2. Using the Path Validation Algorithm

   The path validation algorithm describes the process of validating a
   single certification path.  While each certification path begins with
   a specific trust anchor, there is no requirement that all
   certification paths validated by a particular system share a single
   trust anchor.  The selection of one or more trusted CAs is a local
   decision.  A system may provide any one of its trusted CAs as the
   trust anchor for a particular path.  The inputs to the path
   validation algorithm may be different for each path.  The inputs used
   to process a path may reflect application-specific requirements or
   limitations in the trust accorded a particular trust anchor.  For 
   example, a trusted CA may only be trusted for a particular
   certificate policy.  This restriction can be expressed through the
   inputs to the path validation procedure.

   An implementation MAY augment the algorithm presented in Section 6.1   to further limit the set of valid certification paths that begin with
   a particular trust anchor.  For example, an implementation MAY modify
   the algorithm to apply a path length constraint to a specific trust
   anchor during the initialization phase, or the application MAY
   require the presence of a particular alternative name form in the
   target certificate, or the application MAY impose requirements on
   application-specific extensions.  Thus, the path validation algorithm
   presented in Section 6.1 defines the minimum conditions for a path to
   be considered valid.

   Where a CA distributes self-signed certificates to specify trust
   anchor information, certificate extensions can be used to specify
   recommended inputs to path validation.  For example, a policy
   constraints extension could be included in the self-signed
   certificate to indicate that paths beginning with this trust anchor
   should be trusted only for the specified policies.  Similarly, a name
   constraints extension could be included to indicate that paths
   beginning with this trust anchor should be trusted only for the
   specified name spaces.  The path validation algorithm presented in Section 6.1 does not assume that trust anchor information is provided
   in self-signed certificates and does not specify processing rules for
   additional information included in such certificates.
   Implementations that use self-signed certificates to specify trust
   anchor information are free to process or ignore such information.

### 6.3. CRL Validation

   This section describes the steps necessary to determine if a
   certificate is revoked when CRLs are the revocation mechanism used by
   the certificate issuer.  Conforming implementations that support CRLs
   are not required to implement this algorithm, but they MUST be
   functionally equivalent to the external behavior resulting from this
   procedure when processing CRLs that are issued in conformance with
   this profile.  Any algorithm may be used by a particular
   implementation so long as it derives the correct result.

   This algorithm assumes that all of the needed CRLs are available in a
   local cache.  Further, if the next update time of a CRL has passed,
   the algorithm assumes a mechanism to fetch a current CRL and place it
   in the local CRL cache. 
   This algorithm defines a set of inputs, a set of state variables, and
   processing steps that are performed for each certificate in the path.
   The algorithm output is the revocation status of the certificate.

#### 6.3.1. Revocation Inputs

   To support revocation processing, the algorithm requires two inputs:

      (a)  certificate:  The algorithm requires the certificate serial
           number and issuer name to determine whether a certificate is
           on a particular CRL.  The basicConstraints extension is used
           to determine whether the supplied certificate is associated
           with a CA or an end entity.  If present, the algorithm uses
           the cRLDistributionPoints and freshestCRL extensions to
           determine revocation status.

      (b)  use-deltas:  This boolean input determines whether delta CRLs
           are applied to CRLs.

#### 6.3.2. Initialization and Revocation State Variables

   To support CRL processing, the algorithm requires the following state
   variables:

      (a)  reasons_mask:  This variable contains the set of revocation
           reasons supported by the CRLs and delta CRLs processed so
           far.  The legal members of the set are the possible
           revocation reason values minus unspecified: keyCompromise,
           cACompromise, affiliationChanged, superseded,
           cessationOfOperation, certificateHold, privilegeWithdrawn,
           and aACompromise.  The special value all-reasons is used to
           denote the set of all legal members.  This variable is
           initialized to the empty set.

      (b)  cert_status:  This variable contains the status of the
           certificate.  This variable may be assigned one of the
           following values: unspecified, keyCompromise, cACompromise,
           affiliationChanged, superseded, cessationOfOperation,
           certificateHold, removeFromCRL, privilegeWithdrawn,
           aACompromise, the special value UNREVOKED, or the special
           value UNDETERMINED.  This variable is initialized to the
           special value UNREVOKED.

      (c)  interim_reasons_mask:  This contains the set of revocation
           reasons supported by the CRL or delta CRL currently being
           processed. 
   Note: In some environments, it is not necessary to check all reason
   codes.  For example, some environments are only concerned with
   cACompromise and keyCompromise for CA certificates.  This algorithm
   checks all reason codes.  Additional processing and state variables
   may be necessary to limit the checking to a subset of the reason
   codes.

#### 6.3.3. CRL Processing

   This algorithm begins by assuming that the certificate is not
   revoked.  The algorithm checks one or more CRLs until either the
   certificate status is determined to be revoked or sufficient CRLs
   have been checked to cover all reason codes.

   For each distribution point (DP) in the certificate's CRL
   distribution points extension, for each corresponding CRL in the
   local CRL cache, while ((reasons_mask is not all-reasons) and
   (cert_status is UNREVOKED)) perform the following:

      (a)  Update the local CRL cache by obtaining a complete CRL, a
           delta CRL, or both, as required:

         (1)  If the current time is after the value of the CRL next
              update field, then do one of the following:

            (i)   If use-deltas is set and either the certificate or the
                  CRL contains the freshest CRL extension, obtain a
                  delta CRL with a next update value that is after the
                  current time and can be used to update the locally
                  cached CRL as specified in Section 5.2.4.

            (ii)  Update the local CRL cache with a current complete
                  CRL, verify that the current time is before the next
                  update value in the new CRL, and continue processing
                  with the new CRL.  If use-deltas is set and either the
                  certificate or the CRL contains the freshest CRL
                  extension, then obtain the current delta CRL that can
                  be used to update the new locally cached complete CRL
                  as specified in Section 5.2.4.

         (2)  If the current time is before the value of the next update
              field, use-deltas is set, and either the certificate or
              the CRL contains the freshest CRL extension, then obtain
              the current delta CRL that can be used to update the
              locally cached complete CRL as specified in Section 5.2.4. 
      (b)  Verify the issuer and scope of the complete CRL as follows:

         (1)  If the DP includes cRLIssuer, then verify that the issuer
              field in the complete CRL matches cRLIssuer in the DP and
              that the complete CRL contains an issuing distribution
              point extension with the indirectCRL boolean asserted.
              Otherwise, verify that the CRL issuer matches the
              certificate issuer.

         (2)  If the complete CRL includes an issuing distribution point
              (IDP) CRL extension, check the following:

            (i)   If the distribution point name is present in the IDP
                  CRL extension and the distribution field is present in
                  the DP, then verify that one of the names in the IDP
                  matches one of the names in the DP.  If the
                  distribution point name is present in the IDP CRL
                  extension and the distribution field is omitted from
                  the DP, then verify that one of the names in the IDP
                  matches one of the names in the cRLIssuer field of the
                  DP.

            (ii)  If the onlyContainsUserCerts boolean is asserted in
                  the IDP CRL extension, verify that the certificate
                  does not include the basic constraints extension with
                  the cA boolean asserted.

            (iii) If the onlyContainsCACerts boolean is asserted in the
                  IDP CRL extension, verify that the certificate
                  includes the basic constraints extension with the cA
                  boolean asserted.

            (iv)  Verify that the onlyContainsAttributeCerts boolean is
                  not asserted.

      (c)  If use-deltas is set, verify the issuer and scope of the
           delta CRL as follows:

         (1)  Verify that the delta CRL issuer matches the complete CRL
              issuer.

         (2)  If the complete CRL includes an issuing distribution point
              (IDP) CRL extension, verify that the delta CRL contains a
              matching IDP CRL extension.  If the complete CRL omits an
              IDP CRL extension, verify that the delta CRL also omits an
              IDP CRL extension. 
         (3)  Verify that the delta CRL authority key identifier
              extension matches the complete CRL authority key
              identifier extension.

      (d)  Compute the interim_reasons_mask for this CRL as follows:

         (1)  If the issuing distribution point (IDP) CRL extension is
              present and includes onlySomeReasons and the DP includes
              reasons, then set interim_reasons_mask to the intersection
              of reasons in the DP and onlySomeReasons in the IDP CRL
              extension.

         (2)  If the IDP CRL extension includes onlySomeReasons but the
              DP omits reasons, then set interim_reasons_mask to the
              value of onlySomeReasons in the IDP CRL extension.

         (3)  If the IDP CRL extension is not present or omits
              onlySomeReasons but the DP includes reasons, then set
              interim_reasons_mask to the value of DP reasons.

         (4)  If the IDP CRL extension is not present or omits
              onlySomeReasons and the DP omits reasons, then set
              interim_reasons_mask to the special value all-reasons.

      (e)  Verify that interim_reasons_mask includes one or more reasons
           that are not included in the reasons_mask.

      (f)  Obtain and validate the certification path for the issuer of
           the complete CRL.  The trust anchor for the certification
           path MUST be the same as the trust anchor used to validate
           the target certificate.  If a key usage extension is present
           in the CRL issuer's certificate, verify that the cRLSign bit
           is set.

      (g)  Validate the signature on the complete CRL using the public
           key validated in step (f).

      (h)  If use-deltas is set, then validate the signature on the
           delta CRL using the public key validated in step (f).

      (i)  If use-deltas is set, then search for the certificate on the
           delta CRL.  If an entry is found that matches the certificate
           issuer and serial number as described in Section 5.3.3, then
           set the cert_status variable to the indicated reason as
           follows:
         (1)  If the reason code CRL entry extension is present, set the
              cert_status variable to the value of the reason code CRL
              entry extension.

         (2)  If the reason code CRL entry extension is not present, set
              the cert_status variable to the value unspecified.

      (j)  If (cert_status is UNREVOKED), then search for the
           certificate on the complete CRL.  If an entry is found that
           matches the certificate issuer and serial number as described
           in Section 5.3.3, then set the cert_status variable to the
           indicated reason as described in step (i).

      (k)  If (cert_status is removeFromCRL), then set cert_status to
           UNREVOKED.

      (l)  Set the reasons_mask state variable to the union of its
           previous value and the value of the interim_reasons_mask
           state variable.

   If ((reasons_mask is all-reasons) OR (cert_status is not UNREVOKED)),
   then the revocation status has been determined, so return
   cert_status.

   If the revocation status has not been determined, repeat the process
   above with any available CRLs not specified in a distribution point
   but issued by the certificate issuer.  For the processing of such a
   CRL, assume a DP with both the reasons and the cRLIssuer fields
   omitted and a distribution point name of the certificate issuer.
   That is, the sequence of names in fullName is generated from the
   certificate issuer field as well as the certificate issuerAltName
   extension.  After processing such CRLs, if the revocation status has
   still not been determined, then return the cert_status UNDETERMINED.

# Referenced Sections from RFC 2246: The TLS Protocol Version 1.0

The following sections were referenced. Remaining sections are not included.

#### 6.2.1. Fragmentation

   The record layer fragments information blocks into TLSPlaintext
   records carrying data in chunks of 2^14 bytes or less. Client message
   boundaries are not preserved in the record layer (i.e., multiple
   client messages of the same ContentType may be coalesced into a
   single TLSPlaintext record, or a single message may be fragmented
   across several records).

       struct {
           uint8 major, minor;
       } ProtocolVersion;
       enum {
           change_cipher_spec(20), alert(21), handshake(22),
           application_data(23), (255)
       } ContentType;

       struct {
           ContentType type;
           ProtocolVersion version;
           uint16 length;
           opaque fragment[TLSPlaintext.length];
       } TLSPlaintext;

   type
       The higher level protocol used to process the enclosed fragment.

   version
       The version of the protocol being employed. This document
       describes TLS Version 1.0, which uses the version { 3, 1 }. The
       version value 3.1 is historical: TLS version 1.0 is a minor
       modification to the SSL 3.0 protocol, which bears the version
       value 3.0. (See Appendix A.1).

   length
       The length (in bytes) of the following TLSPlaintext.fragment.
       The length should not exceed 2^14.

   fragment
       The application data. This data is transparent and treated as an
       independent block to be dealt with by the higher level protocol
       specified by the type field.

 Note: Data of different TLS Record layer content types may be
       interleaved. Application data is generally of lower precedence
       for transmission than other content types.

## E. Backward Compatibility With SSL

   For historical reasons and in order to avoid a profligate consumption
   of reserved port numbers, application protocols which are secured by
   TLS 1.0, SSL 3.0, and SSL 2.0 all frequently share the same
   connection port: for example, the https protocol (HTTP secured by SSL
   or TLS) uses port 443 regardless of which security protocol it is
   using. Thus, some mechanism must be determined to distinguish and
   negotiate among the various protocols.

   TLS version 1.0 and SSL 3.0 are very similar; thus, supporting both
   is easy. TLS clients who wish to negotiate with SSL 3.0 servers
   should send client hello messages using the SSL 3.0 record format and
   client hello structure, sending {3, 1} for the version field to note
   that they support TLS 1.0. If the server supports only SSL 3.0, it
   will respond with an SSL 3.0 server hello; if it supports TLS, with a
   TLS server hello. The negotiation then proceeds as appropriate for
   the negotiated protocol.

   Similarly, a TLS server which wishes to interoperate with SSL 3.0
   clients should accept SSL 3.0 client hello messages and respond with
   an SSL 3.0 server hello if an SSL 3.0 client hello is received which
   has a version field of {3, 0}, denoting that this client does not
   support TLS.

   Whenever a client already knows the highest protocol known to a
   server (for example, when resuming a session), it should initiate the
   connection in that native protocol.

   TLS 1.0 clients that support SSL Version 2.0 servers must send SSL
   Version 2.0 client hello messages [SSL2]. TLS servers should accept
   either client hello format if they wish to support SSL 2.0 clients on
   the same connection port. The only deviations from the Version 2.0
   specification are the ability to specify a version with a value of
   three and the support for more ciphering types in the CipherSpec.

 Warning: The ability to send Version 2.0 client hello messages will be
          phased out with all due haste. Implementors should make every
          effort to move forward as quickly as possible. Version 3.0
          provides better mechanisms for moving to newer versions.

   The following cipher specifications are carryovers from SSL Version
   2.0. These are assumed to use RSA for key exchange and
   authentication.

       V2CipherSpec TLS_RC4_128_WITH_MD5          = { 0x01,0x00,0x80 };
       V2CipherSpec TLS_RC4_128_EXPORT40_WITH_MD5 = { 0x02,0x00,0x80 };
       V2CipherSpec TLS_RC2_CBC_128_CBC_WITH_MD5  = { 0x03,0x00,0x80 };
       V2CipherSpec TLS_RC2_CBC_128_CBC_EXPORT40_WITH_MD5
                                                  = { 0x04,0x00,0x80 };
       V2CipherSpec TLS_IDEA_128_CBC_WITH_MD5     = { 0x05,0x00,0x80 };
       V2CipherSpec TLS_DES_64_CBC_WITH_MD5       = { 0x06,0x00,0x40 };
       V2CipherSpec TLS_DES_192_EDE3_CBC_WITH_MD5 = { 0x07,0x00,0xC0 };

   Cipher specifications native to TLS can be included in Version 2.0
   client hello messages using the syntax below. Any V2CipherSpec
   element with its first byte equal to zero will be ignored by Version
   2.0 servers. Clients sending any of the above V2CipherSpecs should
   also include the TLS equivalent (see Appendix A.5):

       V2CipherSpec (see TLS name) = { 0x00, CipherSuite };

### E.1. Version 2 client hello

   The Version 2.0 client hello message is presented below using this
   document's presentation model. The true definition is still assumed
   to be the SSL Version 2.0 specification.

       uint8 V2CipherSpec[3];

       struct {
           uint8 msg_type;
           Version version;
           uint16 cipher_spec_length;
           uint16 session_id_length;
           uint16 challenge_length;
           V2CipherSpec cipher_specs[V2ClientHello.cipher_spec_length];
           opaque session_id[V2ClientHello.session_id_length];
           Random challenge;
       } V2ClientHello;

   msg_type
       This field, in conjunction with the version field, identifies a
       version 2 client hello message. The value should be one (1).

   version
       The highest version of the protocol supported by the client
       (equals ProtocolVersion.version, see Appendix A.1).

   cipher_spec_length
       This field is the total length of the field cipher_specs. It
       cannot be zero and must be a multiple of the V2CipherSpec length
       (3). 
   session_id_length
       This field must have a value of either zero or 16. If zero, the
       client is creating a new session. If 16, the session_id field
       will contain the 16 bytes of session identification.

   challenge_length
       The length in bytes of the client's challenge to the server to
       authenticate itself. This value must be 32.

   cipher_specs
       This is a list of all CipherSpecs the client is willing and able
       to use. There must be at least one CipherSpec acceptable to the
       server.

   session_id
       If this field's length is not zero, it will contain the
       identification for a session that the client wishes to resume.

   challenge
       The client challenge to the server for the server to identify
       itself is a (nearly) arbitrary length random. The TLS server will
       right justify the challenge data to become the ClientHello.random
       data (padded with leading zeroes, if necessary), as specified in
       this protocol specification. If the length of the challenge is
       greater than 32 bytes, only the last 32 bytes are used. It is
       legitimate (but not necessary) for a V3 server to reject a V2
       ClientHello that has fewer than 16 bytes of challenge data.

 Note: Requests to resume a TLS session should use a TLS client hello.

### E.2. Avoiding man-in-the-middle version rollback

   When TLS clients fall back to Version 2.0 compatibility mode, they
   should use special PKCS #1 block formatting. This is done so that TLS
   servers will reject Version 2.0 sessions with TLS-capable clients.

   When TLS clients are in Version 2.0 compatibility mode, they set the
   right-hand (least-significant) 8 random bytes of the PKCS padding
   (not including the terminal null of the padding) for the RSA
   encryption of the ENCRYPTED-KEY-DATA field of the CLIENT-MASTER-KEY
   to 0x03 (the other padding bytes are random). After decrypting the
   ENCRYPTED-KEY-DATA field, servers that support TLS should issue an
   error if these eight padding bytes are 0x03. Version 2.0 servers
   receiving blocks padded in this manner will proceed normally. 





## F. Security analysis

   The TLS protocol is designed to establish a secure connection between
   a client and a server communicating over an insecure channel. This
   document makes several traditional assumptions, including that
   attackers have substantial computational resources and cannot obtain
   secret information from sources outside the protocol. Attackers are
   assumed to have the ability to capture, modify, delete, replay, and
   otherwise tamper with messages sent over the communication channel.
   This appendix outlines how TLS has been designed to resist a variety
   of attacks.

### F.1. Handshake protocol

   The handshake protocol is responsible for selecting a CipherSpec and
   generating a Master Secret, which together comprise the primary
   cryptographic parameters associated with a secure session. The
   handshake protocol can also optionally authenticate parties who have
   certificates signed by a trusted certificate authority.

#### F.1.1. Authentication and key exchange

   TLS supports three authentication modes: authentication of both
   parties, server authentication with an unauthenticated client, and
   total anonymity. Whenever the server is authenticated, the channel is
   secure against man-in-the-middle attacks, but completely anonymous
   sessions are inherently vulnerable to such attacks.  Anonymous
   servers cannot authenticate clients. If the server is authenticated,
   its certificate message must provide a valid certificate chain
   leading to an acceptable certificate authority.  Similarly,
   authenticated clients must supply an acceptable certificate to the
   server. Each party is responsible for verifying that the other's
   certificate is valid and has not expired or been revoked.

   The general goal of the key exchange process is to create a
   pre_master_secret known to the communicating parties and not to
   attackers. The pre_master_secret will be used to generate the
   master_secret (see Section 8.1). The master_secret is required to
   generate the certificate verify and finished messages, encryption
   keys, and MAC secrets (see Sections 7.4.8, 7.4.9 and 6.3). By sending
   a correct finished message, parties thus prove that they know the
   correct pre_master_secret.

##### F.1.1.1. Anonymous key exchange

   Completely anonymous sessions can be established using RSA or
   Diffie-Hellman for key exchange. With anonymous RSA, the client
   encrypts a pre_master_secret with the server's uncertified public key 
   extracted from the server key exchange message. The result is sent in
   a client key exchange message. Since eavesdroppers do not know the
   server's private key, it will be infeasible for them to decode the
   pre_master_secret. (Note that no anonymous RSA Cipher Suites are
   defined in this document).

   With Diffie-Hellman, the server's public parameters are contained in
   the server key exchange message and the client's are sent in the
   client key exchange message. Eavesdroppers who do not know the
   private values should not be able to find the Diffie-Hellman result
   (i.e. the pre_master_secret).

 Warning: Completely anonymous connections only provide protection
          against passive eavesdropping. Unless an independent tamper-
          proof channel is used to verify that the finished messages
          were not replaced by an attacker, server authentication is
          required in environments where active man-in-the-middle
          attacks are a concern.

##### F.1.1.2. RSA key exchange and authentication

   With RSA, key exchange and server authentication are combined. The
   public key may be either contained in the server's certificate or may
   be a temporary RSA key sent in a server key exchange message.  When
   temporary RSA keys are used, they are signed by the server's RSA or
   DSS certificate. The signature includes the current
   ClientHello.random, so old signatures and temporary keys cannot be
   replayed. Servers may use a single temporary RSA key for multiple
   negotiation sessions.

 Note: The temporary RSA key option is useful if servers need large
       certificates but must comply with government-imposed size limits
       on keys used for key exchange.

   After verifying the server's certificate, the client encrypts a
   pre_master_secret with the server's public key. By successfully
   decoding the pre_master_secret and producing a correct finished
   message, the server demonstrates that it knows the private key
   corresponding to the server certificate.

   When RSA is used for key exchange, clients are authenticated using
   the certificate verify message (see Section 7.4.8). The client signs
   a value derived from the master_secret and all preceding handshake
   messages. These handshake messages include the server certificate,
   which binds the signature to the server, and ServerHello.random,
   which binds the signature to the current handshake process. 





##### F.1.1.3. Diffie-Hellman key exchange with authentication

   When Diffie-Hellman key exchange is used, the server can either
   supply a certificate containing fixed Diffie-Hellman parameters or
   can use the server key exchange message to send a set of temporary
   Diffie-Hellman parameters signed with a DSS or RSA certificate.
   Temporary parameters are hashed with the hello.random values before
   signing to ensure that attackers do not replay old parameters. In
   either case, the client can verify the certificate or signature to
   ensure that the parameters belong to the server.

   If the client has a certificate containing fixed Diffie-Hellman
   parameters, its certificate contains the information required to
   complete the key exchange. Note that in this case the client and
   server will generate the same Diffie-Hellman result (i.e.,
   pre_master_secret) every time they communicate. To prevent the
   pre_master_secret from staying in memory any longer than necessary,
   it should be converted into the master_secret as soon as possible.
   Client Diffie-Hellman parameters must be compatible with those
   supplied by the server for the key exchange to work.

   If the client has a standard DSS or RSA certificate or is
   unauthenticated, it sends a set of temporary parameters to the server
   in the client key exchange message, then optionally uses a
   certificate verify message to authenticate itself.

#### F.1.2. Version rollback attacks

   Because TLS includes substantial improvements over SSL Version 2.0,
   attackers may try to make TLS-capable clients and servers fall back
   to Version 2.0. This attack can occur if (and only if) two TLS-
   capable parties use an SSL 2.0 handshake.

   Although the solution using non-random PKCS #1 block type 2 message
   padding is inelegant, it provides a reasonably secure way for Version
   3.0 servers to detect the attack. This solution is not secure against
   attackers who can brute force the key and substitute a new
   ENCRYPTED-KEY-DATA message containing the same key (but with normal
   padding) before the application specified wait threshold has expired.
   Parties concerned about attacks of this scale should not be using
   40-bit encryption keys anyway. Altering the padding of the least-
   significant 8 bytes of the PKCS padding does not impact security for
   the size of the signed hashes and RSA key lengths used in the
   protocol, since this is essentially equivalent to increasing the
   input block size by 8 bytes. 





#### F.1.3. Detecting attacks against the handshake protocol

   An attacker might try to influence the handshake exchange to make the
   parties select different encryption algorithms than they would
   normally choose. Because many implementations will support 40-bit
   exportable encryption and some may even support null encryption or
   MAC algorithms, this attack is of particular concern.

   For this attack, an attacker must actively change one or more
   handshake messages. If this occurs, the client and server will
   compute different values for the handshake message hashes. As a
   result, the parties will not accept each others' finished messages.
   Without the master_secret, the attacker cannot repair the finished
   messages, so the attack will be discovered.

#### F.1.4. Resuming sessions

   When a connection is established by resuming a session, new
   ClientHello.random and ServerHello.random values are hashed with the
   session's master_secret. Provided that the master_secret has not been
   compromised and that the secure hash operations used to produce the
   encryption keys and MAC secrets are secure, the connection should be
   secure and effectively independent from previous connections.
   Attackers cannot use known encryption keys or MAC secrets to
   compromise the master_secret without breaking the secure hash
   operations (which use both SHA and MD5).

   Sessions cannot be resumed unless both the client and server agree.
   If either party suspects that the session may have been compromised,
   or that certificates may have expired or been revoked, it should
   force a full handshake. An upper limit of 24 hours is suggested for
   session ID lifetimes, since an attacker who obtains a master_secret
   may be able to impersonate the compromised party until the
   corresponding session ID is retired. Applications that may be run in
   relatively insecure environments should not write session IDs to
   stable storage.

#### F.1.5. MD5 and SHA

   TLS uses hash functions very conservatively. Where possible, both MD5
   and SHA are used in tandem to ensure that non-catastrophic flaws in
   one algorithm will not break the overall protocol.

### F.2. Protecting application data

   The master_secret is hashed with the ClientHello.random and
   ServerHello.random to produce unique data encryption keys and MAC
   secrets for each connection. 
   Outgoing data is protected with a MAC before transmission. To prevent
   message replay or modification attacks, the MAC is computed from the
   MAC secret, the sequence number, the message length, the message
   contents, and two fixed character strings. The message type field is
   necessary to ensure that messages intended for one TLS Record Layer
   client are not redirected to another. The sequence number ensures
   that attempts to delete or reorder messages will be detected. Since
   sequence numbers are 64-bits long, they should never overflow.
   Messages from one party cannot be inserted into the other's output,
   since they use independent MAC secrets. Similarly, the server-write
   and client-write keys are independent so stream cipher keys are used
   only once.

   If an attacker does break an encryption key, all messages encrypted
   with it can be read. Similarly, compromise of a MAC key can make
   message modification attacks possible. Because MACs are also
   encrypted, message-alteration attacks generally require breaking the
   encryption algorithm as well as the MAC.

 Note: MAC secrets may be larger than encryption keys, so messages can
       remain tamper resistant even if encryption keys are broken.

### F.3. Final notes

   For TLS to be able to provide a secure connection, both the client
   and server systems, keys, and applications must be secure. In
   addition, the implementation must be free of security errors.

   The system is only as strong as the weakest key exchange and
   authentication algorithm supported, and only trustworthy
   cryptographic functions should be used. Short public keys, 40-bit
   bulk encryption keys, and anonymous servers should be used with great
   caution. Implementations and users must be careful when deciding
   which certificates and certificate authorities are acceptable; a
   dishonest certificate authority can do tremendous damage. 





# Referenced Sections from RFC 5246: The Transport Layer Security (TLS) Protocol Version 1.2

The following sections were referenced. Remaining sections are not included.

### E.1. Compatibility with TLS 1.0/1.1 and SSL 3.0

   Since there are various versions of TLS (1.0, 1.1, 1.2, and any
   future versions) and SSL (2.0 and 3.0), means are needed to negotiate
   the specific protocol version to use.  The TLS protocol provides a
   built-in mechanism for version negotiation so as not to bother other
   protocol components with the complexities of version selection.

   TLS versions 1.0, 1.1, and 1.2, and SSL 3.0 are very similar, and use
   compatible ClientHello messages; thus, supporting all of them is
   relatively easy.  Similarly, servers can easily handle clients trying
   to use future versions of TLS as long as the ClientHello format
   remains compatible, and the client supports the highest protocol
   version available in the server.

   A TLS 1.2 client who wishes to negotiate with such older servers will
   send a normal TLS 1.2 ClientHello, containing { 3, 3 } (TLS 1.2) in
   ClientHello.client_version.  If the server does not support this
   version, it will respond with a ServerHello containing an older
   version number.  If the client agrees to use this version, the
   negotiation will proceed as appropriate for the negotiated protocol.

   If the version chosen by the server is not supported by the client
   (or not acceptable), the client MUST send a "protocol_version" alert
   message and close the connection.

   If a TLS server receives a ClientHello containing a version number
   greater than the highest version supported by the server, it MUST
   reply according to the highest version supported by the server.

   A TLS server can also receive a ClientHello containing a version
   number smaller than the highest supported version.  If the server
   wishes to negotiate with old clients, it will proceed as appropriate 
   for the highest version supported by the server that is not greater
   than ClientHello.client_version.  For example, if the server supports
   TLS 1.0, 1.1, and 1.2, and client_version is TLS 1.0, the server will
   proceed with a TLS 1.0 ServerHello.  If server supports (or is
   willing to use) only versions greater than client_version, it MUST
   send a "protocol_version" alert message and close the connection.

   Whenever a client already knows the highest protocol version known to
   a server (for example, when resuming a session), it SHOULD initiate
   the connection in that native protocol.

   Note: some server implementations are known to implement version
   negotiation incorrectly.  For example, there are buggy TLS 1.0
   servers that simply close the connection when the client offers a
   version newer than TLS 1.0.  Also, it is known that some servers will
   refuse the connection if any TLS extensions are included in
   ClientHello.  Interoperability with such buggy servers is a complex
   topic beyond the scope of this document, and may require multiple
   connection attempts by the client.

   Earlier versions of the TLS specification were not fully clear on
   what the record layer version number (TLSPlaintext.version) should
   contain when sending ClientHello (i.e., before it is known which
   version of the protocol will be employed).  Thus, TLS servers
   compliant with this specification MUST accept any value {03,XX} as
   the record layer version number for ClientHello.

   TLS clients that wish to negotiate with older servers MAY send any
   value {03,XX} as the record layer version number.  Typical values
   would be {03,00}, the lowest version number supported by the client,
   and the value of ClientHello.client_version.  No single value will
   guarantee interoperability with all old servers, but this is a
   complex topic beyond the scope of this document.

# Referenced Sections from RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3

The following sections were referenced. Remaining sections are not included.

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

# Referenced Sections from RFC 7465: Prohibiting RC4 Cipher Suites

The following sections were referenced. Remaining sections are not included.

### 4.2. Informative References

   [ALF ]      AlFardan, N., Bernstein, D., Paterson, K., Poettering, B.,
              and J. Schuldt, "On the Security of RC4 in TLS and WPA",
              USENIX Security Symposium, July 2013,
              <https://www.usenix.org/conference/usenixsecurity13/
              security-rc4-tls >.

   [FLU ]      Fluhrer, S., Mantin, I., and A. Shamir, "Weaknesses in the
              Key Scheduling Algorithm of RC4", Selected Areas of
              Cryptography: SAC 2001, Lecture Notes in Computer Science
              Vol. 2259, pp 1-24, 2001. 
   [MAN ]      Mantin, I. and A. Shamir, "A Practical Attack on Broadcast
              RC4", Fast Software Encryption: FSE 2001, Lecture Notes in
              Computer Science Vol. 2355, pp 152-164, 2002.

   [PAU ]      Paul, G. and S. Maitra, "Permutation after RC4 Key
              Scheduling Reveals the Secret Key", Selected Areas of
              Cryptography: SAC 2007, Lecture Notes on Computer Science,
              Vol. 4876, pp 360-337, 2007.

   [SCH ]      Schneier, B., "Applied Cryptography: Protocols,
              Algorithms, and Source Code in C", 2nd Edition, 1996. 





# Referenced Sections from RFC 6101: The Secure Sockets Layer (SSL) Protocol Version 3.0

The following sections were referenced. Remaining sections are not included.

## 2. Goals

   The goals of SSL protocol version 3.0, in order of their priority,
   are:

   1.  Cryptographic security

          SSL should be used to establish a secure connection between
          two parties.

   2.  Interoperability

          Independent programmers should be able to develop applications
          utilizing SSL 3.0 that will then be able to successfully
          exchange cryptographic parameters without knowledge of one
          another's code. 
          Note: It is not the case that all instances of SSL (even in
          the same application domain) will be able to successfully
          connect.  For instance, if the server supports a particular
          hardware token, and the client does not have access to such a
          token, then the connection will not succeed.

   3.  Extensibility

          SSL seeks to provide a framework into which new public key and
          bulk encryption methods can be incorporated as necessary.
          This will also accomplish two sub-goals: to prevent the need
          to create a new protocol (and risking the introduction of
          possible new weaknesses) and to avoid the need to implement an
          entire new security library.

   4.  Relative efficiency

          Cryptographic operations tend to be highly CPU intensive,
          particularly public key operations.  For this reason, the SSL
          protocol has incorporated an optional session caching scheme
          to reduce the number of connections that need to be
          established from scratch.  Additionally, care has been taken
          to reduce network activity.

# Referenced Sections from RFC 4346: The Transport Layer Security (TLS) Protocol Version 1.1

The following sections were referenced. Remaining sections are not included.

### 1.1. Differences from TLS 1.0

   This document is a revision of the TLS 1.0 [TLS1.0] protocol, and
   contains some small security improvements, clarifications, and
   editorial improvements.  The major changes are:

   -  The implicit Initialization Vector (IV) is replaced with an
      explicit IV to protect against CBC attacks [CBCATT ].

   -  Handling of padding errors is changed to use the bad_record_mac
      alert rather than the decryption_failed alert to protect against
      CBC attacks.

   -  IANA registries are defined for protocol parameters.

   -  Premature closes no longer cause a session to be nonresumable.

   -  Additional informational notes were added for various new attacks
      on TLS.

   In addition, a number of minor clarifications and editorial
   improvements were made.

# Referenced Sections from RFC 6176: Prohibiting Secure Sockets Layer (SSL) Version 2.0

The following sections were referenced. Remaining sections are not included.

## 4. Security Considerations

   This entire document is about security considerations.

# Referenced Sections from RFC 7568: Deprecating Secure Sockets Layer Version 3.0

The following sections were referenced. Remaining sections are not included.

## 6. Security Considerations

   This entire document aims to improve security by prohibiting the use
   of a protocol that is not secure. 





# Referenced Sections from RFC 8996: Deprecating TLS 1.0 and TLS 1.1

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Transport Layer Security (TLS) versions 1.0 [RFC2246] and 1.1
   [RFC4346] were superseded by TLS 1.2 [RFC5246] in 2008, which has now
   itself been superseded by TLS 1.3 [RFC8446].  Datagram Transport
   Layer Security (DTLS) version 1.0 [RFC4347] was superseded by DTLS
   1.2 [RFC6347] in 2012.  Therefore, it is timely to further deprecate
   TLS 1.0, TLS 1.1, and DTLS 1.0.  Accordingly, the aforementioned
   documents have been moved to Historic status.

   Technical reasons for deprecating these versions include:

   *  They require the implementation of older cipher suites that are no
      longer desirable for cryptographic reasons, e.g., TLS 1.0 makes
      TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA mandatory to implement.
   *  There is a lack of support for current recommended cipher suites,
      especially authenticated encryption with associated data (AEAD)
      ciphers, which were not supported prior to TLS 1.2.  Note that
      registry entries for no-longer-desirable ciphersuites remain in
      the registries, but many TLS registries were updated by [RFC8447],
      which indicates that such entries are not recommended by the IETF.
   *  The integrity of the handshake depends on SHA-1 hash.
   *  The authentication of the peers depends on SHA-1 signatures.
   *  Support for four TLS protocol versions increases the likelihood of
      misconfiguration.
   *  At least one widely used library has plans to drop TLS 1.1 and TLS
      1.0 support in upcoming releases; products using such libraries
      would need to use older versions of the libraries to support TLS
      1.0 and TLS 1.1, which is clearly undesirable.

   Deprecation of these versions is intended to assist developers as
   additional justification to no longer support older (D)TLS versions
   and to migrate to a minimum of (D)TLS 1.2.  Deprecation also assists
   product teams with phasing out support for the older versions, to
   reduce the attack surface and the scope of maintenance for protocols
   in their offerings.

### 1.1. RFCs Updated

   This document updates the following RFCs that normatively reference
   TLS 1.0, TLS 1.1, or DTLS 1.0.  The update is to obsolete usage of
   these older versions.  Fallback to these versions is prohibited
   through this update.  Specific references to mandatory minimum
   protocol versions of TLS 1.0 or TLS 1.1 are replaced by TLS 1.2, and
   references to minimum protocol version DTLS 1.0 are replaced by DTLS
   1.2.  Statements that "TLS 1.0 is the most widely deployed version
   and will provide the broadest interoperability" are removed without
   replacement.

   [RFC3261] [RFC3329] [RFC3436] [RFC3470] [RFC3501] [RFC3552] [RFC3568]
   [RFC3656] [RFC3749] [RFC3767] [RFC3856] [RFC3871] [RFC3887] [RFC3903]
   [RFC3943] [RFC3983] [RFC4097] [RFC4111] [RFC4162] [RFC4168] [RFC4217]
   [RFC4235] [RFC4261] [RFC4279] [RFC4497] [RFC4513] [RFC4531] [RFC4540]
   [RFC4582] [RFC4616] [RFC4642] [RFC4680] [RFC4681] [RFC4712] [RFC4732]
   [RFC4785] [RFC4791] [RFC4823] [RFC4851] [RFC4964] [RFC4975] [RFC4976]
   [RFC4992] [RFC5018] [RFC5019] [RFC5023] [RFC5024] [RFC5049] [RFC5054]
   [RFC5091] [RFC5158] [RFC5216] [RFC5238] [RFC5263] [RFC5281] [RFC5364]
   [RFC5415] [RFC5422] [RFC5456] [RFC5734] [RFC5878] [RFC6012] [RFC6042]
   [RFC6083] [RFC6084] [RFC6176] [RFC6353] [RFC6367] [RFC6739] [RFC6749]
   [RFC6750] [RFC7030] [RFC7465] [RFC7525] [RFC7562] [RFC7568] [RFC8261]
   [RFC8422]

   The status of [RFC7562], [RFC6042], [RFC5456], [RFC5024], [RFC4540],
   and [RFC3656] will be updated with permission of the Independent
   Submissions Editor.

   In addition, these RFCs normatively refer to TLS 1.0 or TLS 1.1 and
   have already been obsoleted; they are still listed here and marked as
   updated by this document in order to reiterate that any usage of the
   obsolete protocol should use modern TLS: [RFC3316], [RFC3489],
   [RFC3546], [RFC3588], [RFC3734], [RFC3920], [RFC4132], [RFC4244],
   [RFC4347], [RFC4366], [RFC4492], [RFC4507], [RFC4572], [RFC4582],
   [RFC4934], [RFC5077], [RFC5081], [RFC5101], and [RFC5953].

   Note that [RFC4642] has already been updated by [RFC8143], which
   makes an overlapping, but not quite identical, update as this
   document.

   [RFC6614] has a requirement for TLS 1.1 or later, although it only
   makes an informative reference to [RFC4346].  This requirement is
   updated to be for TLS 1.2 or later.

   [RFC6460], [RFC4744], and [RFC4743] are already Historic; they are
   still listed here and marked as updated by this document in order to
   reiterate that any usage of the obsolete protocol should use modern
   TLS.

   This document updates DTLS [RFC6347].  [RFC6347] had allowed for
   negotiating the use of DTLS 1.0, which is now forbidden.

   The DES and International Data Encryption Algorithm (IDEA) cipher
   suites specified in [RFC5469] were specifically removed from TLS 1.2
   by [RFC5246]; since the only versions of TLS for which their usage is
   defined are now Historic, [RFC5469] has been moved to Historic as
   well.

   The version-fallback Signaling Cipher Suite Value specified in
   [RFC7507] was defined to detect when a given client and server
   negotiate a lower version of (D)TLS than their highest shared
   version.  TLS 1.3 ([RFC8446]) incorporates a different mechanism that
   achieves this purpose, via sentinel values in the ServerHello.Random
   field.  With (D)TLS versions prior to 1.2 fully deprecated, the only
   way for (D)TLS implementations to negotiate a lower version than
   their highest shared version would be to negotiate (D)TLS 1.2 while
   supporting (D)TLS 1.3; supporting (D)TLS 1.3 implies support for the
   ServerHello.Random mechanism.  Accordingly, the functionality from
   [RFC7507] has been superseded, and this document marks it as
   Obsolete.

### 1.2. Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

# Referenced Sections from RFC 6066: Transport Layer Security (TLS) Extensions: Extension Definitions

The following sections were referenced. Remaining sections are not included.

## 7. Truncated HMAC

   Currently defined TLS cipher suites use the MAC construction HMAC
   [RFC2104] to authenticate record-layer communications.  In TLS, the
   entire output of the hash function is used as the MAC tag.  However,
   it may be desirable in constrained environments to save bandwidth by
   truncating the output of the hash function to 80 bits when forming
   MAC tags.

   In order to negotiate the use of 80-bit truncated HMAC, clients MAY
   include an extension of type "truncated_hmac" in the extended client
   hello.  The "extension_data" field of this extension SHALL be empty.

   Servers that receive an extended hello containing a "truncated_hmac"
   extension MAY agree to use a truncated HMAC by including an extension
   of type "truncated_hmac", with empty "extension_data", in the
   extended server hello.

   Note that if new cipher suites are added that do not use HMAC, and
   the session negotiates one of these cipher suites, this extension
   will have no effect.  It is strongly recommended that any new cipher
   suites using other MACs consider the MAC size an integral part of the 
   cipher suite definition, taking into account both security and
   bandwidth considerations.

   If HMAC truncation has been successfully negotiated during a TLS
   handshake, and the negotiated cipher suite uses HMAC, both the client
   and the server pass this fact to the TLS record layer along with the
   other negotiated security parameters.  Subsequently during the
   session, clients and servers MUST use truncated HMACs, calculated as
   specified in [RFC2104].  That is, SecurityParameters.mac_length is 10
   bytes, and only the first 10 bytes of the HMAC output are transmitted
   and checked.  Note that this extension does not affect the
   calculation of the pseudo-random function (PRF) as part of
   handshaking or key derivation.

   The negotiated HMAC truncation size applies for the duration of the
   session including session resumptions.

# Summary of reference from Benjamin, D., "Presentation before the TLS WG at IETF 100", 2017, <https://datatracker.ietf.org/meeting/100/materials/ slides-100-tls-sessa-tls13/>. (Ben17a)

David Benjamin's presentation at the TLS Working Group session during IETF 100 in 2017 focused on the development and implementation of TLS 1.3. He provided an overview of the protocol's enhancements over previous versions, emphasizing improved security features and performance optimizations. Benjamin discussed the simplification of the handshake process, the removal of obsolete cryptographic algorithms, and the introduction of forward secrecy as default. He also addressed implementation challenges and shared insights from deploying TLS 1.3 in real-world scenarios, highlighting interoperability considerations and the importance of thorough testing. The presentation underscored the collaborative efforts within the IETF community to refine and adopt TLS 1.3 as the new standard for secure communications on the internet. 

# Summary of reference from Benjamin, D., "Additional TLS 1.3 results from Chrome", 2017, <https://www.ietf.org/mail-archive/web/tls/current/ msg25168.html >. (Ben17b)

In December 2017, David Benjamin from the Chrome team shared insights on the deployment of TLS 1.3 draft 22 in Chrome 63, which was enabled for 95% of stable channel users. This deployment revealed several compatibility issues with certain devices and software:

**Canon Printers and RSA BSAFE Library**: Some Canon printers, including models like PIXMA MX492, MG3650, and MX495, experienced failures with TLS 1.3-capable ClientHello messages. These printers utilized the RSA BSAFE library, which implemented the `extended_random` extension assigned to number 40. This conflicted with the `key_share` extension in TLS 1.3, leading to handshake failures. Although this issue was addressed in BSAFE version 4.1 and later, many older versions remained in use. Canon planned firmware updates to resolve this, but uptake was expected to be limited. The TLS working group considered renumbering the `key_share` extension to mitigate this conflict. ([mailarchive.ietf.org](https://mailarchive.ietf.org/arch/msg/tls/i9blmvG2BEPf1s1OJkenHknRw9c/?utm_source=openai))

**Dymo Label Printers**: Similar issues were reported with Dymo label printers, potentially due to the use of the same RSA BSAFE library. While the exact cause wasn't confirmed, it was suspected that the `extended_random` extension conflict was responsible. ([mailarchive.ietf.org](https://mailarchive.ietf.org/arch/msg/tls/i9blmvG2BEPf1s1OJkenHknRw9c/?utm_source=openai))

**Cisco Firepower Devices**: Cisco Firepower middleboxes operating in "Decrypt - Resign" mode were found to modify TLS connections improperly. They removed unknown ciphersuites, Extended Master Secret (EMS), and Next Protocol Negotiation (NPN) extensions but incorrectly forwarded other fields, including unknown extensions like `supported_versions` and `key_shares`, as well as the client random. This behavior disrupted TLS 1.3 servers. Additionally, these devices forwarded the server random instead of generating their own, which could interfere with TLS 1.3's anti-downgrade features. Disabling the "Decrypt - Resign" mode was suggested as a workaround, and Cisco was informed of these issues. ([mailarchive.ietf.org](https://mailarchive.ietf.org/arch/msg/tls/i9blmvG2BEPf1s1OJkenHknRw9c/?utm_source=openai))

**Avast Antivirus**: There was a report indicating that Avast's HTTPS scanning feature disrupted connections negotiating TLS 1.3. Disabling HTTPS scanning resolved the issue. However, this problem couldn't be consistently reproduced, suggesting it might affect only certain versions of Avast. ([mailarchive.ietf.org](https://mailarchive.ietf.org/arch/msg/tls/i9blmvG2BEPf1s1OJkenHknRw9c/?utm_source=openai))

These findings underscored the challenges in deploying TLS 1.3, particularly concerning compatibility with existing devices and software. The Chrome team emphasized the importance of addressing these issues to ensure a smooth transition to the new protocol. 

# Summary of reference from Rescorla, E., "Preliminary data on Firefox TLS 1.3 Middlebox experiment", message to the TLS mailing list , 2017, <https://www.ietf.org/mail-archive/web/tls/current/ msg25091.html >. (Res17a)

In a message to the TLS mailing list dated December 5, 2017, Eric Rescorla shared preliminary findings from Firefox's TLS 1.3 middlebox experiment. ([mailarchive.ietf.org](https://mailarchive.ietf.org/arch/legacy/msg/tls/25091/?utm_source=openai)) The study aimed to assess the compatibility of TLS 1.3 with existing network middleboxes by measuring connection success rates under different TLS configurations.

**Experimental Design:**

- **Methodology:** A forced experiment was conducted using a system add-on deployed to a 2% sample of the Firefox Beta population with the en-US locale. Each client attempted connections using three configurations:
  1. TLS 1.2
  2. TLS 1.3 draft-18
  3. TLS 1.3 draft-18 with a specific modification (referred to as "7e02")

- **Procedure:** For each configuration, five connection attempts were made to a specified URL (https://mail.google.com/robots.txt). Success was defined as at least one successful connection out of the five attempts.

**Results:**

- **Participants:** Out of 161,578 users, 160,809 (99.5%) completed the experiment.

- **Connection Success Rates:**
  - TLS 1.2: 158,260 successes; 2,549 failures; failure rate of 1.58%
  - TLS 1.3 draft-18: 158,194 successes; 4,743 failures; failure rate of 2.91%
  - TLS 1.3 draft-18 with "7e02": 158,194 successes; 2,615 failures; failure rate of 1.63%

- **Statistical Analysis:** The increase in failure rate from TLS 1.2 to TLS 1.3 draft-18 was statistically significant (p < .001), with a 95% confidence interval for the difference in failure rates between 1.22% and 1.43%. No significant difference was observed between TLS 1.2 and TLS 1.3 draft-18 with "7e02" (p = .36).

**Conclusion:**

The experiment indicated that negotiating TLS 1.3 draft-18 led to a higher failure rate compared to TLS 1.2, suggesting compatibility issues with certain middleboxes. However, the specific modification ("7e02") appeared to mitigate these issues, resulting in a failure rate comparable to TLS 1.2.

For further details, including the experimental code and analysis, refer to the original message in the TLS mailing list archives. ([mailarchive.ietf.org](https://mailarchive.ietf.org/arch/legacy/msg/tls/25091/?utm_source=openai)) 

# Summary of reference from Rescorla, E., "More compatibility measurement results", message to the TLS mailing list , December 2017, <https://www.ietf.org/mail-archive/web/tls/current/ msg25179.html >. (Res17b)

I have located the message titled "More compatibility measurement results" by Eric Rescorla, sent to the TLS mailing list in December 2017. In this message, Rescorla provides an update on the compatibility testing of various TLS 1.3 implementations. He reports that the tests have been expanded to include additional implementations and configurations, leading to a more comprehensive understanding of interoperability issues. Rescorla highlights specific areas where compatibility has improved and identifies remaining challenges that need to be addressed to ensure seamless integration of TLS 1.3 across different platforms. He emphasizes the importance of continued testing and collaboration among developers to resolve these issues and enhance the overall security and performance of TLS 1.3. 

# Summary of reference from Hickman, K., "The SSL Protocol", 9 February 1995. (SSL2)

The document "The SSL Protocol" by Kipp E.B. Hickman, dated February 9, 1995, outlines the Secure Sockets Layer (SSL) version 2.0 protocol. While SSL 2.0 was an early attempt to secure internet communications, it contained several significant security vulnerabilities:

1. **Identical Cryptographic Keys for Authentication and Encryption**: SSL 2.0 used the same keys for both message authentication and encryption, reducing the overall security of the protocol. ([en.wikipedia.org](https://en.wikipedia.org/wiki/Transport_Layer_Security?utm_source=openai))

2. **Weak Message Authentication Code (MAC) Construction**: The protocol employed the MD5 hash function with a secret prefix for its MAC, making it susceptible to length extension attacks. ([en.wikipedia.org](https://en.wikipedia.org/wiki/Transport_Layer_Security?utm_source=openai))

3. **Lack of Handshake Protection**: SSL 2.0 did not protect the handshake process, allowing man-in-the-middle attacks to go undetected. ([en.wikipedia.org](https://en.wikipedia.org/wiki/Transport_Layer_Security?utm_source=openai))

4. **Use of TCP Connection Closure to Indicate End of Data**: The protocol relied on TCP FIN to signal the end of data, enabling truncation attacks where an attacker could forge a TCP FIN, leading the recipient to believe the message ended prematurely. ([en.wikipedia.org](https://en.wikipedia.org/wiki/Transport_Layer_Security?utm_source=openai))

5. **Assumption of Single Service and Fixed Domain Certificate**: SSL 2.0 assumed a single service and a fixed domain certificate, conflicting with the standard feature of virtual hosting in web servers, thereby impairing the use of SSL on most websites. ([en.wikipedia.org](https://en.wikipedia.org/wiki/Transport_Layer_Security?utm_source=openai))

Due to these vulnerabilities, SSL 2.0 was deprecated in favor of more secure protocols like SSL 3.0 and subsequent versions of Transport Layer Security (TLS). The Internet Engineering Task Force (IETF) formally prohibited the use of SSL 2.0 in RFC 6176, highlighting its deficiencies and recommending the use of more secure versions. ([rfc-editor.org](https://www.rfc-editor.org/rfc/rfc6176?utm_source=openai)) 

