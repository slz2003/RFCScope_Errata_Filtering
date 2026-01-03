Let us analyze section C of draft draft-ietf-tls-rfc8446bis-14. All references made by section C have also been included below.

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
# Referenced Sections from RFC 8448: Example Handshake Traces for TLS 1.3

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   TLS 1.3 [TLS13] defines a new key schedule and a number of new
   cryptographic operations.  This document includes sample handshakes
   that show all intermediate values.  This allows an implementation to
   be verified incrementally, examining inputs and outputs of each
   cryptographic computation independently.

   A private key is included with the traces so that implementations can
   be checked by importing these values and verifying that the same
   outputs are produced.

   Note:  Invocations of HMAC-based Extract-and-Expand Key Derivation
      Function (HKDF) [RFC5869] are not labeled, but they can be
      identified through the use of the labels used by HKDF.

## 2. Private Keys

   Ephemeral private keys are shown as they are generated in the traces.

   The server in most examples uses an RSA certificate with a private
   key of:

   modulus (public):  b4 bb 49 8f 82 79 30 3d 98 08 36 39 9b 36 c6 98 8c
      0c 68 de 55 e1 bd b8 26 d3 90 1a 24 61 ea fd 2d e4 9a 91 d0 15 ab
      bc 9a 95 13 7a ce 6c 1a f1 9e aa 6a f9 8c 7c ed 43 12 09 98 e1 87
      a8 0e e0 cc b0 52 4b 1b 01 8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a 5f
      da 43 08 46 74 80 30 53 0e f0 46 1c 8c a9 d9 ef bf ae 8e a6 d1 d0
      3e 2b d1 93 ef f0 ab 9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7 9f 7f 1e
      3f 
   public exponent:  01 00 01

   private exponent:  04 de a7 05 d4 3a 6e a7 20 9d d8 07 21 11 a8 3c 81
      e3 22 a5 92 78 b3 34 80 64 1e af 7c 0a 69 85 b8 e3 1c 44 f6 de 62
      e1 b4 c2 30 9f 61 26 e7 7b 7c 41 e9 23 31 4b bf a3 88 13 05 dc 12
      17 f1 6c 81 9c e5 38 e9 22 f3 69 82 8d 0e 57 19 5d 8c 84 88 46 02
      07 b2 fa a7 26 bc f7 08 bb d7 db 7f 67 9f 89 34 92 fc 2a 62 2e 08
      97 0a ac 44 1c e4 e0 c3 08 8d f2 5a e6 79 23 3d f8 a3 bd a2 ff 99
      41

   prime1:  e4 35 fb 7c c8 37 37 75 6d ac ea 96 ab 7f 59 a2 cc 10 69 db
      7d eb 19 0e 17 e3 3a 53 2b 27 3f 30 a3 27 aa 0a aa bc 58 cd 67 46
      6a f9 84 5f ad c6 75 fe 09 4a f9 2c 4b d1 f2 c1 bc 33 dd 2e 05 15

   prime2:  ca bd 3b c0 e0 43 86 64 c8 d4 cc 9f 99 97 7a 94 d9 bb fe ad
      8e 43 87 0a ba e3 f7 eb 8b 4e 0e ee 8a f1 d9 b4 71 9b a6 19 6c f2
      cb ba ee eb f8 b3 49 0a fe 9e 9f fa 74 a8 8a a5 1f c6 45 62 93 03

   exponent1:  3f 57 34 5c 27 fe 1b 68 7e 6e 76 16 27 b7 8b 1b 82 64 33
      dd 76 0f a0 be a6 a6 ac f3 94 90 aa 1b 47 cd a4 86 9d 68 f5 84 dd
      5b 50 29 bd 32 09 3b 82 58 66 1f e7 15 02 5e 5d 70 a4 5a 08 d3 d3
      19

   exponent2:  18 3d a0 13 63 bd 2f 28 85 ca cb dc 99 64 bf 47 64 f1 51
      76 36 f8 64 01 28 6f 71 89 3c 52 cc fe 40 a6 c2 3d 0d 08 6b 47 c6
      fb 10 d8 fd 10 41 e0 4d ef 7e 9a 40 ce 95 7c 41 77 94 e1 04 12 d1
      39

   coefficient:  83 9c a9 a0 85 e4 28 6b 2c 90 e4 66 99 7a 2c 68 1f 21
      33 9a a3 47 78 14 e4 de c1 18 33 05 0e d5 0d d1 3c c0 38 04 8a 43
      c5 9b 2a cc 41 68 89 c0 37 66 5f e5 af a6 05 96 9f 8c 01 df a5 ca
      96 9d

## 3. Simple 1-RTT Handshake

   In this example, the simplest possible handshake is completed.  The
   server is authenticated, but the client remains anonymous.  After
   connecting, a few application data octets are exchanged.  The server
   sends a session ticket that permits the use of 0-RTT data in any
   resumed session.

   {client}  create an ephemeral x25519 key pair:

      private key (32 octets):  49 af 42 ba 7f 79 94 85 2d 71 3e f2 78
         4b cb ca a7 91 1d e2 6a dc 56 42 cb 63 45 40 e7 ea 50 05

      public key (32 octets):  99 38 1d e5 60 e4 bd 43 d2 3d 8e 43 5a 7d
         ba fe b3 c0 6e 51 c1 3c ae 4d 54 13 69 1e 52 9a af 2c 
   {client}  construct a ClientHello handshake message:

      ClientHello (196 octets):  01 00 00 c0 03 03 cb 34 ec b1 e7 81 63
         ba 1c 38 c6 da cb 19 6a 6d ff a2 1a 8d 99 12 ec 18 a2 ef 62 83
         02 4d ec e7 00 00 06 13 01 13 03 13 02 01 00 00 91 00 00 00 0b
         00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 23
         00 00 00 33 00 26 00 24 00 1d 00 20 99 38 1d e5 60 e4 bd 43 d2
         3d 8e 43 5a 7d ba fe b3 c0 6e 51 c1 3c ae 4d 54 13 69 1e 52 9a
         af 2c 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03
         02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06
         02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01

   {client}  send handshake record:

      payload (196 octets):  01 00 00 c0 03 03 cb 34 ec b1 e7 81 63 ba
         1c 38 c6 da cb 19 6a 6d ff a2 1a 8d 99 12 ec 18 a2 ef 62 83 02
         4d ec e7 00 00 06 13 01 13 03 13 02 01 00 00 91 00 00 00 0b 00
         09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00 12
         00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 23 00
         00 00 33 00 26 00 24 00 1d 00 20 99 38 1d e5 60 e4 bd 43 d2 3d
         8e 43 5a 7d ba fe b3 c0 6e 51 c1 3c ae 4d 54 13 69 1e 52 9a af
         2c 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03 02
         03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06 02
         02 02 00 2d 00 02 01 01 00 1c 00 02 40 01

      complete record (201 octets):  16 03 01 00 c4 01 00 00 c0 03 03 cb
         34 ec b1 e7 81 63 ba 1c 38 c6 da cb 19 6a 6d ff a2 1a 8d 99 12
         ec 18 a2 ef 62 83 02 4d ec e7 00 00 06 13 01 13 03 13 02 01 00
         00 91 00 00 00 0b 00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01
         00 00 0a 00 14 00 12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02
         01 03 01 04 00 23 00 00 00 33 00 26 00 24 00 1d 00 20 99 38 1d
         e5 60 e4 bd 43 d2 3d 8e 43 5a 7d ba fe b3 c0 6e 51 c1 3c ae 4d
         54 13 69 1e 52 9a af 2c 00 2b 00 03 02 03 04 00 0d 00 20 00 1e
         04 03 05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02
         01 04 02 05 02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01

   {server}  extract secret "early":

      salt:  0 (all zero octets)

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c
         e2 10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a 
   {server}  create an ephemeral x25519 key pair:

      private key (32 octets):  b1 58 0e ea df 6d d5 89 b8 ef 4f 2d 56
         52 57 8c c8 10 e9 98 01 91 ec 8d 05 83 08 ce a2 16 a2 1e

      public key (32 octets):  c9 82 88 76 11 20 95 fe 66 76 2b db f7 c6
         72 e1 56 d6 cc 25 3b 83 3d f1 dd 69 b1 b0 4e 75 1f 0f

   {server}  construct a ServerHello handshake message:

      ServerHello (90 octets):  02 00 00 56 03 03 a6 af 06 a4 12 18 60
         dc 5e 6e 60 24 9c d3 4c 95 93 0c 8a c5 cb 14 34 da c1 55 77 2e
         d3 e2 69 28 00 13 01 00 00 2e 00 33 00 24 00 1d 00 20 c9 82 88
         76 11 20 95 fe 66 76 2b db f7 c6 72 e1 56 d6 cc 25 3b 83 3d f1
         dd 69 b1 b0 4e 75 1f 0f 00 2b 00 02 03 04

   {server}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {server}  extract secret "handshake":

      salt (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba b6 97
         16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

      IKM (32 octets):  8b d4 05 4f b5 5b 9d 63 fd fb ac f9 f0 4b 9f 0d
         35 e6 d6 3f 53 75 63 ef d4 62 72 90 0f 89 49 2d

      secret (32 octets):  1d c8 26 e9 36 06 aa 6f dc 0a ad c1 2f 74 1b
         01 04 6a a6 b9 9f 69 1e d2 21 a9 f0 ca 04 3f be ac

   {server}  derive secret "tls13 c hs traffic":

      PRK (32 octets):  1d c8 26 e9 36 06 aa 6f dc 0a ad c1 2f 74 1b 01
         04 6a a6 b9 9f 69 1e d2 21 a9 f0 ca 04 3f be ac 
      hash (32 octets):  86 0c 06 ed c0 78 58 ee 8e 78 f0 e7 42 8c 58 ed
         d6 b4 3f 2c a3 e6 e9 5f 02 ed 06 3c f0 e1 ca d8

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 68 73 20 74 72
         61 66 66 69 63 20 86 0c 06 ed c0 78 58 ee 8e 78 f0 e7 42 8c 58
         ed d6 b4 3f 2c a3 e6 e9 5f 02 ed 06 3c f0 e1 ca d8

      expanded (32 octets):  b3 ed db 12 6e 06 7f 35 a7 80 b3 ab f4 5e
         2d 8f 3b 1a 95 07 38 f5 2e 96 00 74 6a 0e 27 a5 5a 21

   {server}  derive secret "tls13 s hs traffic":

      PRK (32 octets):  1d c8 26 e9 36 06 aa 6f dc 0a ad c1 2f 74 1b 01
         04 6a a6 b9 9f 69 1e d2 21 a9 f0 ca 04 3f be ac

      hash (32 octets):  86 0c 06 ed c0 78 58 ee 8e 78 f0 e7 42 8c 58 ed
         d6 b4 3f 2c a3 e6 e9 5f 02 ed 06 3c f0 e1 ca d8

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 68 73 20 74 72
         61 66 66 69 63 20 86 0c 06 ed c0 78 58 ee 8e 78 f0 e7 42 8c 58
         ed d6 b4 3f 2c a3 e6 e9 5f 02 ed 06 3c f0 e1 ca d8

      expanded (32 octets):  b6 7b 7d 69 0c c1 6c 4e 75 e5 42 13 cb 2d
         37 b4 e9 c9 12 bc de d9 10 5d 42 be fd 59 d3 91 ad 38

   {server}  derive secret for master "tls13 derived":

      PRK (32 octets):  1d c8 26 e9 36 06 aa 6f dc 0a ad c1 2f 74 1b 01
         04 6a a6 b9 9f 69 1e d2 21 a9 f0 ca 04 3f be ac

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  43 de 77 e0 c7 77 13 85 9a 94 4d b9 db 25
         90 b5 31 90 a6 5b 3e e2 e4 f1 2d d7 a0 bb 7c e2 54 b4

   {server}  extract secret "master":

      salt (32 octets):  43 de 77 e0 c7 77 13 85 9a 94 4d b9 db 25 90 b5
         31 90 a6 5b 3e e2 e4 f1 2d d7 a0 bb 7c e2 54 b4

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
      secret (32 octets):  18 df 06 84 3d 13 a0 8b f2 a4 49 84 4c 5f 8a
         47 80 01 bc 4d 4c 62 79 84 d5 a4 1d a8 d0 40 29 19

   {server}  send handshake record:

      payload (90 octets):  02 00 00 56 03 03 a6 af 06 a4 12 18 60 dc 5e
         6e 60 24 9c d3 4c 95 93 0c 8a c5 cb 14 34 da c1 55 77 2e d3 e2
         69 28 00 13 01 00 00 2e 00 33 00 24 00 1d 00 20 c9 82 88 76 11
         20 95 fe 66 76 2b db f7 c6 72 e1 56 d6 cc 25 3b 83 3d f1 dd 69
         b1 b0 4e 75 1f 0f 00 2b 00 02 03 04

      complete record (95 octets):  16 03 03 00 5a 02 00 00 56 03 03 a6
         af 06 a4 12 18 60 dc 5e 6e 60 24 9c d3 4c 95 93 0c 8a c5 cb 14
         34 da c1 55 77 2e d3 e2 69 28 00 13 01 00 00 2e 00 33 00 24 00
         1d 00 20 c9 82 88 76 11 20 95 fe 66 76 2b db f7 c6 72 e1 56 d6
         cc 25 3b 83 3d f1 dd 69 b1 b0 4e 75 1f 0f 00 2b 00 02 03 04

   {server}  derive write traffic keys for handshake data:

      PRK (32 octets):  b6 7b 7d 69 0c c1 6c 4e 75 e5 42 13 cb 2d 37 b4
         e9 c9 12 bc de d9 10 5d 42 be fd 59 d3 91 ad 38

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  3f ce 51 60 09 c2 17 27 d0 f2 e4 e8 6e
         e4 03 bc

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  5d 31 3e b2 67 12 76 ee 13 00 0b 30

   {server}  construct an EncryptedExtensions handshake message:

      EncryptedExtensions (40 octets):  08 00 00 24 00 22 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c
         00 02 40 01 00 00 00 00

   {server}  construct a Certificate handshake message:

      Certificate (445 octets):  0b 00 01 b9 00 00 01 b5 00 01 b0 30 82
         01 ac 30 82 01 15 a0 03 02 01 02 02 01 02 30 0d 06 09 2a 86 48
         86 f7 0d 01 01 0b 05 00 30 0e 31 0c 30 0a 06 03 55 04 03 13 03
         72 73 61 30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35 39 5a 17
         0d 32 36 30 37 33 30 30 31 32 33 35 39 5a 30 0e 31 0c 30 0a 06
         03 55 04 03 13 03 72 73 61 30 81 9f 30 0d 06 09 2a 86 48 86 f7
         0d 01 01 01 05 00 03 81 8d 00 30 81 89 02 81 81 00 b4 bb 49 8f
         82 79 30 3d 98 08 36 39 9b 36 c6 98 8c 0c 68 de 55 e1 bd b8 26
         d3 90 1a 24 61 ea fd 2d e4 9a 91 d0 15 ab bc 9a 95 13 7a ce 6c 
         1a f1 9e aa 6a f9 8c 7c ed 43 12 09 98 e1 87 a8 0e e0 cc b0 52
         4b 1b 01 8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a 5f da 43 08 46 74
         80 30 53 0e f0 46 1c 8c a9 d9 ef bf ae 8e a6 d1 d0 3e 2b d1 93
         ef f0 ab 9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7 9f 7f 1e 3f 02 03
         01 00 01 a3 1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06
         03 55 1d 0f 04 04 03 02 05 a0 30 0d 06 09 2a 86 48 86 f7 0d 01
         01 0b 05 00 03 81 81 00 85 aa d2 a0 e5 b9 27 6b 90 8c 65 f7 3a
         72 67 17 06 18 a5 4c 5f 8a 7b 33 7d 2d f7 a5 94 36 54 17 f2 ea
         e8 f8 a5 8c 8f 81 72 f9 31 9c f3 6b 7f d6 c5 5b 80 f2 1a 03 01
         51 56 72 60 96 fd 33 5e 5e 67 f2 db f1 02 70 2e 60 8c ca e6 be
         c1 fc 63 a4 2a 99 be 5c 3e b7 10 7c 3c 54 e9 b9 eb 2b d5 20 3b
         1c 3b 84 e0 a8 b2 f7 59 40 9b a3 ea c9 d9 1d 40 2d cc 0c c8 f8
         96 12 29 ac 91 87 b4 2b 4d e1 00 00

   {server}  construct a CertificateVerify handshake message:

      CertificateVerify (136 octets):  0f 00 00 84 08 04 00 80 5a 74 7c
         5d 88 fa 9b d2 e5 5a b0 85 a6 10 15 b7 21 1f 82 4c d4 84 14 5a
         b3 ff 52 f1 fd a8 47 7b 0b 7a bc 90 db 78 e2 d3 3a 5c 14 1a 07
         86 53 fa 6b ef 78 0c 5e a2 48 ee aa a7 85 c4 f3 94 ca b6 d3 0b
         be 8d 48 59 ee 51 1f 60 29 57 b1 54 11 ac 02 76 71 45 9e 46 44
         5c 9e a5 8c 18 1e 81 8e 95 b8 c3 fb 0b f3 27 84 09 d3 be 15 2a
         3d a5 04 3e 06 3d da 65 cd f5 ae a2 0d 53 df ac d4 2f 74 f3

   {server}  calculate finished "tls13 finished":

      PRK (32 octets):  b6 7b 7d 69 0c c1 6c 4e 75 e5 42 13 cb 2d 37 b4
         e9 c9 12 bc de d9 10 5d 42 be fd 59 d3 91 ad 38

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  00 8d 3b 66 f8 16 ea 55 9f 96 b5 37 e8 85
         c3 1f c0 68 bf 49 2c 65 2f 01 f2 88 a1 d8 cd c1 9f c8

      finished (32 octets):  9b 9b 14 1d 90 63 37 fb d2 cb dc e7 1d f4
         de da 4a b4 2c 30 95 72 cb 7f ff ee 54 54 b7 8f 07 18

   {server}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 9b 9b 14 1d 90 63 37 fb d2 cb
         dc e7 1d f4 de da 4a b4 2c 30 95 72 cb 7f ff ee 54 54 b7 8f 07
         18
   {server}  send handshake record:

      payload (657 octets):  08 00 00 24 00 22 00 0a 00 14 00 12 00 1d
         00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c 00 02 40
         01 00 00 00 00 0b 00 01 b9 00 00 01 b5 00 01 b0 30 82 01 ac 30
         82 01 15 a0 03 02 01 02 02 01 02 30 0d 06 09 2a 86 48 86 f7 0d
         01 01 0b 05 00 30 0e 31 0c 30 0a 06 03 55 04 03 13 03 72 73 61
         30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35 39 5a 17 0d 32 36
         30 37 33 30 30 31 32 33 35 39 5a 30 0e 31 0c 30 0a 06 03 55 04
         03 13 03 72 73 61 30 81 9f 30 0d 06 09 2a 86 48 86 f7 0d 01 01
         01 05 00 03 81 8d 00 30 81 89 02 81 81 00 b4 bb 49 8f 82 79 30
         3d 98 08 36 39 9b 36 c6 98 8c 0c 68 de 55 e1 bd b8 26 d3 90 1a
         24 61 ea fd 2d e4 9a 91 d0 15 ab bc 9a 95 13 7a ce 6c 1a f1 9e
         aa 6a f9 8c 7c ed 43 12 09 98 e1 87 a8 0e e0 cc b0 52 4b 1b 01
         8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a 5f da 43 08 46 74 80 30 53
         0e f0 46 1c 8c a9 d9 ef bf ae 8e a6 d1 d0 3e 2b d1 93 ef f0 ab
         9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7 9f 7f 1e 3f 02 03 01 00 01
         a3 1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06 03 55 1d
         0f 04 04 03 02 05 a0 30 0d 06 09 2a 86 48 86 f7 0d 01 01 0b 05
         00 03 81 81 00 85 aa d2 a0 e5 b9 27 6b 90 8c 65 f7 3a 72 67 17
         06 18 a5 4c 5f 8a 7b 33 7d 2d f7 a5 94 36 54 17 f2 ea e8 f8 a5
         8c 8f 81 72 f9 31 9c f3 6b 7f d6 c5 5b 80 f2 1a 03 01 51 56 72
         60 96 fd 33 5e 5e 67 f2 db f1 02 70 2e 60 8c ca e6 be c1 fc 63
         a4 2a 99 be 5c 3e b7 10 7c 3c 54 e9 b9 eb 2b d5 20 3b 1c 3b 84
         e0 a8 b2 f7 59 40 9b a3 ea c9 d9 1d 40 2d cc 0c c8 f8 96 12 29
         ac 91 87 b4 2b 4d e1 00 00 0f 00 00 84 08 04 00 80 5a 74 7c 5d
         88 fa 9b d2 e5 5a b0 85 a6 10 15 b7 21 1f 82 4c d4 84 14 5a b3
         ff 52 f1 fd a8 47 7b 0b 7a bc 90 db 78 e2 d3 3a 5c 14 1a 07 86
         53 fa 6b ef 78 0c 5e a2 48 ee aa a7 85 c4 f3 94 ca b6 d3 0b be
         8d 48 59 ee 51 1f 60 29 57 b1 54 11 ac 02 76 71 45 9e 46 44 5c
         9e a5 8c 18 1e 81 8e 95 b8 c3 fb 0b f3 27 84 09 d3 be 15 2a 3d
         a5 04 3e 06 3d da 65 cd f5 ae a2 0d 53 df ac d4 2f 74 f3 14 00
         00 20 9b 9b 14 1d 90 63 37 fb d2 cb dc e7 1d f4 de da 4a b4 2c
         30 95 72 cb 7f ff ee 54 54 b7 8f 07 18

      complete record (679 octets):  17 03 03 02 a2 d1 ff 33 4a 56 f5 bf
         f6 59 4a 07 cc 87 b5 80 23 3f 50 0f 45 e4 89 e7 f3 3a f3 5e df
         78 69 fc f4 0a a4 0a a2 b8 ea 73 f8 48 a7 ca 07 61 2e f9 f9 45
         cb 96 0b 40 68 90 51 23 ea 78 b1 11 b4 29 ba 91 91 cd 05 d2 a3
         89 28 0f 52 61 34 aa dc 7f c7 8c 4b 72 9d f8 28 b5 ec f7 b1 3b
         d9 ae fb 0e 57 f2 71 58 5b 8e a9 bb 35 5c 7c 79 02 07 16 cf b9
         b1 18 3e f3 ab 20 e3 7d 57 a6 b9 d7 47 76 09 ae e6 e1 22 a4 cf
         51 42 73 25 25 0c 7d 0e 50 92 89 44 4c 9b 3a 64 8f 1d 71 03 5d
         2e d6 5b 0e 3c dd 0c ba e8 bf 2d 0b 22 78 12 cb b3 60 98 72 55
         cc 74 41 10 c4 53 ba a4 fc d6 10 92 8d 80 98 10 e4 b7 ed 1a 8f
         d9 91 f0 6a a6 24 82 04 79 7e 36 a6 a7 3b 70 a2 55 9c 09 ea d6
         86 94 5b a2 46 ab 66 e5 ed d8 04 4b 4c 6d e3 fc f2 a8 94 41 ac
         66 27 2f d8 fb 33 0e f8 19 05 79 b3 68 45 96 c9 60 bd 59 6e ea 
         52 0a 56 a8 d6 50 f5 63 aa d2 74 09 96 0d ca 63 d3 e6 88 61 1e
         a5 e2 2f 44 15 cf 95 38 d5 1a 20 0c 27 03 42 72 96 8a 26 4e d6
         54 0c 84 83 8d 89 f7 2c 24 46 1a ad 6d 26 f5 9e ca ba 9a cb bb
         31 7b 66 d9 02 f4 f2 92 a3 6a c1 b6 39 c6 37 ce 34 31 17 b6 59
         62 22 45 31 7b 49 ee da 0c 62 58 f1 00 d7 d9 61 ff b1 38 64 7e
         92 ea 33 0f ae ea 6d fa 31 c7 a8 4d c3 bd 7e 1b 7a 6c 71 78 af
         36 87 90 18 e3 f2 52 10 7f 24 3d 24 3d c7 33 9d 56 84 c8 b0 37
         8b f3 02 44 da 8c 87 c8 43 f5 e5 6e b4 c5 e8 28 0a 2b 48 05 2c
         f9 3b 16 49 9a 66 db 7c ca 71 e4 59 94 26 f7 d4 61 e6 6f 99 88
         2b d8 9f c5 08 00 be cc a6 2d 6c 74 11 6d bd 29 72 fd a1 fa 80
         f8 5d f8 81 ed be 5a 37 66 89 36 b3 35 58 3b 59 91 86 dc 5c 69
         18 a3 96 fa 48 a1 81 d6 b6 fa 4f 9d 62 d5 13 af bb 99 2f 2b 99
         2f 67 f8 af e6 7f 76 91 3f a3 88 cb 56 30 c8 ca 01 e0 c6 5d 11
         c6 6a 1e 2a c4 c8 59 77 b7 c7 a6 99 9b bf 10 dc 35 ae 69 f5 51
         56 14 63 6c 0b 9b 68 c1 9e d2 e3 1c 0b 3b 66 76 30 38 eb ba 42
         f3 b3 8e dc 03 99 f3 a9 f2 3f aa 63 97 8c 31 7f c9 fa 66 a7 3f
         60 f0 50 4d e9 3b 5b 84 5e 27 55 92 c1 23 35 ee 34 0b bc 4f dd
         d5 02 78 40 16 e4 b3 be 7e f0 4d da 49 f4 b4 40 a3 0c b5 d2 af
         93 98 28 fd 4a e3 79 4e 44 f9 4d f5 a6 31 ed e4 2c 17 19 bf da
         bf 02 53 fe 51 75 be 89 8e 75 0e dc 53 37 0d 2b

   {server}  derive secret "tls13 c ap traffic":

      PRK (32 octets):  18 df 06 84 3d 13 a0 8b f2 a4 49 84 4c 5f 8a 47
         80 01 bc 4d 4c 62 79 84 d5 a4 1d a8 d0 40 29 19

      hash (32 octets):  96 08 10 2a 0f 1c cc 6d b6 25 0b 7b 7e 41 7b 1a
         00 0e aa da 3d aa e4 77 7a 76 86 c9 ff 83 df 13

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 61 70 20 74 72
         61 66 66 69 63 20 96 08 10 2a 0f 1c cc 6d b6 25 0b 7b 7e 41 7b
         1a 00 0e aa da 3d aa e4 77 7a 76 86 c9 ff 83 df 13

      expanded (32 octets):  9e 40 64 6c e7 9a 7f 9d c0 5a f8 88 9b ce
         65 52 87 5a fa 0b 06 df 00 87 f7 92 eb b7 c1 75 04 a5

   {server}  derive secret "tls13 s ap traffic":

      PRK (32 octets):  18 df 06 84 3d 13 a0 8b f2 a4 49 84 4c 5f 8a 47
         80 01 bc 4d 4c 62 79 84 d5 a4 1d a8 d0 40 29 19

      hash (32 octets):  96 08 10 2a 0f 1c cc 6d b6 25 0b 7b 7e 41 7b 1a
         00 0e aa da 3d aa e4 77 7a 76 86 c9 ff 83 df 13

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 61 70 20 74 72
         61 66 66 69 63 20 96 08 10 2a 0f 1c cc 6d b6 25 0b 7b 7e 41 7b
         1a 00 0e aa da 3d aa e4 77 7a 76 86 c9 ff 83 df 13
      expanded (32 octets):  a1 1a f9 f0 55 31 f8 56 ad 47 11 6b 45 a9
         50 32 82 04 b4 f4 4b fb 6b 3a 4b 4f 1f 3f cb 63 16 43

   {server}  derive secret "tls13 exp master":

      PRK (32 octets):  18 df 06 84 3d 13 a0 8b f2 a4 49 84 4c 5f 8a 47
         80 01 bc 4d 4c 62 79 84 d5 a4 1d a8 d0 40 29 19

      hash (32 octets):  96 08 10 2a 0f 1c cc 6d b6 25 0b 7b 7e 41 7b 1a
         00 0e aa da 3d aa e4 77 7a 76 86 c9 ff 83 df 13

      info (52 octets):  00 20 10 74 6c 73 31 33 20 65 78 70 20 6d 61 73
         74 65 72 20 96 08 10 2a 0f 1c cc 6d b6 25 0b 7b 7e 41 7b 1a 00
         0e aa da 3d aa e4 77 7a 76 86 c9 ff 83 df 13

      expanded (32 octets):  fe 22 f8 81 17 6e da 18 eb 8f 44 52 9e 67
         92 c5 0c 9a 3f 89 45 2f 68 d8 ae 31 1b 43 09 d3 cf 50

   {server}  derive write traffic keys for application data:

      PRK (32 octets):  a1 1a f9 f0 55 31 f8 56 ad 47 11 6b 45 a9 50 32
         82 04 b4 f4 4b fb 6b 3a 4b 4f 1f 3f cb 63 16 43

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  9f 02 28 3b 6c 9c 07 ef c2 6b b9 f2 ac
         92 e3 56

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  cf 78 2b 88 dd 83 54 9a ad f1 e9 84

   {server}  derive read traffic keys for handshake data:

      PRK (32 octets):  b3 ed db 12 6e 06 7f 35 a7 80 b3 ab f4 5e 2d 8f
         3b 1a 95 07 38 f5 2e 96 00 74 6a 0e 27 a5 5a 21

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  db fa a6 93 d1 76 2c 5b 66 6a f5 d9 50
         25 8d 01

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  5b d3 c7 1b 83 6e 0b 76 bb 73 26 5f

   {client}  extract secret "early" (same as server early secret)
   {client}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {client}  extract secret "handshake" (same as server handshake
      secret)

   {client}  derive secret "tls13 c hs traffic" (same as server)

   {client}  derive secret "tls13 s hs traffic" (same as server)

   {client}  derive secret for master "tls13 derived" (same as server)

   {client}  extract secret "master" (same as server master secret)

   {client}  derive read traffic keys for handshake data (same as server
      handshake data write traffic keys)

   {client}  calculate finished "tls13 finished" (same as server)

   {client}  derive secret "tls13 c ap traffic" (same as server)

   {client}  derive secret "tls13 s ap traffic" (same as server)

   {client}  derive secret "tls13 exp master" (same as server)

   {client}  derive write traffic keys for handshake data (same as
      server handshake data read traffic keys)

   {client}  derive read traffic keys for application data (same as
      server application data write traffic keys)

   {client}  calculate finished "tls13 finished":

      PRK (32 octets):  b3 ed db 12 6e 06 7f 35 a7 80 b3 ab f4 5e 2d 8f
         3b 1a 95 07 38 f5 2e 96 00 74 6a 0e 27 a5 5a 21
      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  b8 0a d0 10 15 fb 2f 0b d6 5f f7 d4 da 5d
         6b f8 3f 84 82 1d 1f 87 fd c7 d3 c7 5b 5a 7b 42 d9 c4

      finished (32 octets):  a8 ec 43 6d 67 76 34 ae 52 5a c1 fc eb e1
         1a 03 9e c1 76 94 fa c6 e9 85 27 b6 42 f2 ed d5 ce 61

   {client}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 a8 ec 43 6d 67 76 34 ae 52 5a
         c1 fc eb e1 1a 03 9e c1 76 94 fa c6 e9 85 27 b6 42 f2 ed d5 ce
         61

   {client}  send handshake record:

      payload (36 octets):  14 00 00 20 a8 ec 43 6d 67 76 34 ae 52 5a c1
         fc eb e1 1a 03 9e c1 76 94 fa c6 e9 85 27 b6 42 f2 ed d5 ce 61

      complete record (58 octets):  17 03 03 00 35 75 ec 4d c2 38 cc e6
         0b 29 80 44 a7 1e 21 9c 56 cc 77 b0 51 7f e9 b9 3c 7a 4b fc 44
         d8 7f 38 f8 03 38 ac 98 fc 46 de b3 84 bd 1c ae ac ab 68 67 d7
         26 c4 05 46

   {client}  derive write traffic keys for application data:

      PRK (32 octets):  9e 40 64 6c e7 9a 7f 9d c0 5a f8 88 9b ce 65 52
         87 5a fa 0b 06 df 00 87 f7 92 eb b7 c1 75 04 a5

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  17 42 2d da 59 6e d5 d9 ac d8 90 e3 c6
         3f 50 51

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  5b 78 92 3d ee 08 57 90 33 e5 23 d9

   {client}  derive secret "tls13 res master":

      PRK (32 octets):  18 df 06 84 3d 13 a0 8b f2 a4 49 84 4c 5f 8a 47
         80 01 bc 4d 4c 62 79 84 d5 a4 1d a8 d0 40 29 19

      hash (32 octets):  20 91 45 a9 6e e8 e2 a1 22 ff 81 00 47 cc 95 26
         84 65 8d 60 49 e8 64 29 42 6d b8 7c 54 ad 14 3d 
      info (52 octets):  00 20 10 74 6c 73 31 33 20 72 65 73 20 6d 61 73
         74 65 72 20 20 91 45 a9 6e e8 e2 a1 22 ff 81 00 47 cc 95 26 84
         65 8d 60 49 e8 64 29 42 6d b8 7c 54 ad 14 3d

      expanded (32 octets):  7d f2 35 f2 03 1d 2a 05 12 87 d0 2b 02 41
         b0 bf da f8 6c c8 56 23 1f 2d 5a ba 46 c4 34 ec 19 6c

   {server}  calculate finished "tls13 finished" (same as client)

   {server}  derive read traffic keys for application data (same as
      client application data write traffic keys)

   {server}  derive secret "tls13 res master" (same as client)

   {server}  generate resumption secret "tls13 resumption":

      PRK (32 octets):  7d f2 35 f2 03 1d 2a 05 12 87 d0 2b 02 41 b0 bf
         da f8 6c c8 56 23 1f 2d 5a ba 46 c4 34 ec 19 6c

      hash (2 octets):  00 00

      info (22 octets):  00 20 10 74 6c 73 31 33 20 72 65 73 75 6d 70 74
         69 6f 6e 02 00 00

      expanded (32 octets):  4e cd 0e b6 ec 3b 4d 87 f5 d6 02 8f 92 2c
         a4 c5 85 1a 27 7f d4 13 11 c9 e6 2d 2c 94 92 e1 c4 f3

   {server}  construct a NewSessionTicket handshake message:

      NewSessionTicket (205 octets):  04 00 00 c9 00 00 00 1e fa d6 aa
         c5 02 00 00 00 b2 2c 03 5d 82 93 59 ee 5f f7 af 4e c9 00 00 00
         00 26 2a 64 94 dc 48 6d 2c 8a 34 cb 33 fa 90 bf 1b 00 70 ad 3c
         49 88 83 c9 36 7c 09 a2 be 78 5a bc 55 cd 22 60 97 a3 a9 82 11
         72 83 f8 2a 03 a1 43 ef d3 ff 5d d3 6d 64 e8 61 be 7f d6 1d 28
         27 db 27 9c ce 14 50 77 d4 54 a3 66 4d 4e 6d a4 d2 9e e0 37 25
         a6 a4 da fc d0 fc 67 d2 ae a7 05 29 51 3e 3d a2 67 7f a5 90 6c
         5b 3f 7d 8f 92 f2 28 bd a4 0d da 72 14 70 f9 fb f2 97 b5 ae a6
         17 64 6f ac 5c 03 27 2e 97 07 27 c6 21 a7 91 41 ef 5f 7d e6 50
         5e 5b fb c3 88 e9 33 43 69 40 93 93 4a e4 d3 57 00 08 00 2a 00
         04 00 00 04 00

   {server}  send handshake record:

      payload (205 octets):  04 00 00 c9 00 00 00 1e fa d6 aa c5 02 00
         00 00 b2 2c 03 5d 82 93 59 ee 5f f7 af 4e c9 00 00 00 00 26 2a
         64 94 dc 48 6d 2c 8a 34 cb 33 fa 90 bf 1b 00 70 ad 3c 49 88 83
         c9 36 7c 09 a2 be 78 5a bc 55 cd 22 60 97 a3 a9 82 11 72 83 f8
         2a 03 a1 43 ef d3 ff 5d d3 6d 64 e8 61 be 7f d6 1d 28 27 db 27
         9c ce 14 50 77 d4 54 a3 66 4d 4e 6d a4 d2 9e e0 37 25 a6 a4 da
         fc d0 fc 67 d2 ae a7 05 29 51 3e 3d a2 67 7f a5 90 6c 5b 3f 7d
         8f 92 f2 28 bd a4 0d da 72 14 70 f9 fb f2 97 b5 ae a6 17 64 6f
         ac 5c 03 27 2e 97 07 27 c6 21 a7 91 41 ef 5f 7d e6 50 5e 5b fb
         c3 88 e9 33 43 69 40 93 93 4a e4 d3 57 00 08 00 2a 00 04 00 00
         04 00

      complete record (227 octets):  17 03 03 00 de 3a 6b 8f 90 41 4a 97
         d6 95 9c 34 87 68 0d e5 13 4a 2b 24 0e 6c ff ac 11 6e 95 d4 1d
         6a f8 f6 b5 80 dc f3 d1 1d 63 c7 58 db 28 9a 01 59 40 25 2f 55
         71 3e 06 1d c1 3e 07 88 91 a3 8e fb cf 57 53 ad 8e f1 70 ad 3c
         73 53 d1 6d 9d a7 73 b9 ca 7f 2b 9f a1 b6 c0 d4 a3 d0 3f 75 e0
         9c 30 ba 1e 62 97 2a c4 6f 75 f7 b9 81 be 63 43 9b 29 99 ce 13
         06 46 15 13 98 91 d5 e4 c5 b4 06 f1 6e 3f c1 81 a7 7c a4 75 84
         00 25 db 2f 0a 77 f8 1b 5a b0 5b 94 c0 13 46 75 5f 69 23 2c 86
         51 9d 86 cb ee ac 87 aa c3 47 d1 43 f9 60 5d 64 f6 50 db 4d 02
         3e 70 e9 52 ca 49 fe 51 37 12 1c 74 bc 26 97 68 7e 24 87 46 d6
         df 35 30 05 f3 bc e1 86 96 12 9c 81 53 55 6b 3b 6c 67 79 b3 7b
         f1 59 85 68 4f

   {client}  generate resumption secret "tls13 resumption" (same as
      server)

   {client}  send application_data record:

      payload (50 octets):  00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e
         0f 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23
         24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30 31

      complete record (72 octets):  17 03 03 00 43 a2 3f 70 54 b6 2c 94
         d0 af fa fe 82 28 ba 55 cb ef ac ea 42 f9 14 aa 66 bc ab 3f 2b
         98 19 a8 a5 b4 6b 39 5b d5 4a 9a 20 44 1e 2b 62 97 4e 1f 5a 62
         92 a2 97 70 14 bd 1e 3d ea e6 3a ee bb 21 69 49 15 e4

   {server}  send application_data record:

      payload (50 octets):  00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e
         0f 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23
         24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30 31

      complete record (72 octets):  17 03 03 00 43 2e 93 7e 11 ef 4a c7
         40 e5 38 ad 36 00 5f c4 a4 69 32 fc 32 25 d0 5f 82 aa 1b 36 e3
         0e fa f9 7d 90 e6 df fc 60 2d cb 50 1a 59 a8 fc c4 9c 4b f2 e5
         f0 a2 1c 00 47 c2 ab f3 32 54 0d d0 32 e1 67 c2 95 5d

   {client}  send alert record:

      payload (2 octets):  01 00
      complete record (24 octets):  17 03 03 00 13 c9 87 27 60 65 56 66
         b7 4d 7f f1 15 3e fd 6d b6 d0 b0 e3

   {server}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 b5 8f d6 71 66 eb f5
         99 d2 47 20 cf be 7e fa 7a 88 64 a9

## 4. Resumed 0-RTT Handshake

   This handshake resumes from the handshake in Section 3.  Since the
   server provided a session ticket that permitted 0-RTT, and the client
   is configured for 0-RTT, the client is able to send 0-RTT data.

   Note: The PSK binder uses the same construction as Finished and so is
   labeled as finished here.

   {client}  create an ephemeral x25519 key pair:

      private key (32 octets):  bf f9 11 88 28 38 46 dd 6a 21 34 ef 71
         80 ca 2b 0b 14 fb 10 dc e7 07 b5 09 8c 0d dd c8 13 b2 df

      public key (32 octets):  e4 ff b6 8a c0 5f 8d 96 c9 9d a2 66 98 34
         6c 6b e1 64 82 ba dd da fe 05 1a 66 b4 f1 8d 66 8f 0b

   {client}  extract secret "early":

      salt:  0 (all zero octets)

      IKM (32 octets):  4e cd 0e b6 ec 3b 4d 87 f5 d6 02 8f 92 2c a4 c5
         85 1a 27 7f d4 13 11 c9 e6 2d 2c 94 92 e1 c4 f3

      secret (32 octets):  9b 21 88 e9 b2 fc 6d 64 d7 1d c3 29 90 0e 20
         bb 41 91 50 00 f6 78 aa 83 9c bb 79 7c b7 d8 33 2c

   {client}  construct a ClientHello handshake message:

      ClientHello (477 octets):  01 00 01 fc 03 03 1b c3 ce b6 bb e3 9c
         ff 93 83 55 b5 a5 0a db 6d b2 1b 7a 6a f6 49 d7 b4 bc 41 9d 78
         76 48 7d 95 00 00 06 13 01 13 03 13 02 01 00 01 cd 00 00 00 0b
         00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 33
         00 26 00 24 00 1d 00 20 e4 ff b6 8a c0 5f 8d 96 c9 9d a2 66 98
         34 6c 6b e1 64 82 ba dd da fe 05 1a 66 b4 f1 8d 66 8f 0b 00 2a
         00 00 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03
         02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06
         02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01 00 15 00 57 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 29 00 dd 00 b8 00 b2 2c 03 5d 82 93 59 ee 5f f7 af 4e c9
         00 00 00 00 26 2a 64 94 dc 48 6d 2c 8a 34 cb 33 fa 90 bf 1b 00
         70 ad 3c 49 88 83 c9 36 7c 09 a2 be 78 5a bc 55 cd 22 60 97 a3
         a9 82 11 72 83 f8 2a 03 a1 43 ef d3 ff 5d d3 6d 64 e8 61 be 7f
         d6 1d 28 27 db 27 9c ce 14 50 77 d4 54 a3 66 4d 4e 6d a4 d2 9e
         e0 37 25 a6 a4 da fc d0 fc 67 d2 ae a7 05 29 51 3e 3d a2 67 7f
         a5 90 6c 5b 3f 7d 8f 92 f2 28 bd a4 0d da 72 14 70 f9 fb f2 97
         b5 ae a6 17 64 6f ac 5c 03 27 2e 97 07 27 c6 21 a7 91 41 ef 5f
         7d e6 50 5e 5b fb c3 88 e9 33 43 69 40 93 93 4a e4 d3 57 fa d6
         aa cb

   {client}  calculate PSK binder:

      ClientHello prefix (477 octets):  01 00 01 fc 03 03 1b c3 ce b6 bb
         e3 9c ff 93 83 55 b5 a5 0a db 6d b2 1b 7a 6a f6 49 d7 b4 bc 41
         9d 78 76 48 7d 95 00 00 06 13 01 13 03 13 02 01 00 01 cd 00 00
         00 0b 00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00
         14 00 12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04
         00 33 00 26 00 24 00 1d 00 20 e4 ff b6 8a c0 5f 8d 96 c9 9d a2
         66 98 34 6c 6b e1 64 82 ba dd da fe 05 1a 66 b4 f1 8d 66 8f 0b
         00 2a 00 00 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03
         06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05
         02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01 00 15 00 57
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 29 00 dd 00 b8 00 b2 2c 03 5d 82 93 59 ee 5f f7 af
         4e c9 00 00 00 00 26 2a 64 94 dc 48 6d 2c 8a 34 cb 33 fa 90 bf
         1b 00 70 ad 3c 49 88 83 c9 36 7c 09 a2 be 78 5a bc 55 cd 22 60
         97 a3 a9 82 11 72 83 f8 2a 03 a1 43 ef d3 ff 5d d3 6d 64 e8 61
         be 7f d6 1d 28 27 db 27 9c ce 14 50 77 d4 54 a3 66 4d 4e 6d a4
         d2 9e e0 37 25 a6 a4 da fc d0 fc 67 d2 ae a7 05 29 51 3e 3d a2
         67 7f a5 90 6c 5b 3f 7d 8f 92 f2 28 bd a4 0d da 72 14 70 f9 fb
         f2 97 b5 ae a6 17 64 6f ac 5c 03 27 2e 97 07 27 c6 21 a7 91 41
         ef 5f 7d e6 50 5e 5b fb c3 88 e9 33 43 69 40 93 93 4a e4 d3 57
         fa d6 aa cb

      binder hash (32 octets):  63 22 4b 2e 45 73 f2 d3 45 4c a8 4b 9d
         00 9a 04 f6 be 9e 05 71 1a 83 96 47 3a ef a0 1e 92 4a 14

      PRK (32 octets):  69 fe 13 1a 3b ba d5 d6 3c 64 ee bc c3 0e 39 5b
         9d 81 07 72 6a 13 d0 74 e3 89 db c8 a4 e4 72 56
      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  55 88 67 3e 72 cb 59 c8 7d 22 0c af fe 94
         f2 de a9 a3 b1 60 9f 7d 50 e9 0a 48 22 7d b9 ed 7e aa

      finished (32 octets):  3a dd 4f b2 d8 fd f8 22 a0 ca 3c f7 67 8e
         f5 e8 8d ae 99 01 41 c5 92 4d 57 bb 6f a3 1b 9e 5f 9d

   {client}  send handshake record:

      payload (512 octets):  01 00 01 fc 03 03 1b c3 ce b6 bb e3 9c ff
         93 83 55 b5 a5 0a db 6d b2 1b 7a 6a f6 49 d7 b4 bc 41 9d 78 76
         48 7d 95 00 00 06 13 01 13 03 13 02 01 00 01 cd 00 00 00 0b 00
         09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00 12
         00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 33 00
         26 00 24 00 1d 00 20 e4 ff b6 8a c0 5f 8d 96 c9 9d a2 66 98 34
         6c 6b e1 64 82 ba dd da fe 05 1a 66 b4 f1 8d 66 8f 0b 00 2a 00
         00 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03 02
         03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06 02
         02 02 00 2d 00 02 01 01 00 1c 00 02 40 01 00 15 00 57 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 29 00 dd 00 b8 00 b2 2c 03 5d 82 93 59 ee 5f f7 af 4e c9 00
         00 00 00 26 2a 64 94 dc 48 6d 2c 8a 34 cb 33 fa 90 bf 1b 00 70
         ad 3c 49 88 83 c9 36 7c 09 a2 be 78 5a bc 55 cd 22 60 97 a3 a9
         82 11 72 83 f8 2a 03 a1 43 ef d3 ff 5d d3 6d 64 e8 61 be 7f d6
         1d 28 27 db 27 9c ce 14 50 77 d4 54 a3 66 4d 4e 6d a4 d2 9e e0
         37 25 a6 a4 da fc d0 fc 67 d2 ae a7 05 29 51 3e 3d a2 67 7f a5
         90 6c 5b 3f 7d 8f 92 f2 28 bd a4 0d da 72 14 70 f9 fb f2 97 b5
         ae a6 17 64 6f ac 5c 03 27 2e 97 07 27 c6 21 a7 91 41 ef 5f 7d
         e6 50 5e 5b fb c3 88 e9 33 43 69 40 93 93 4a e4 d3 57 fa d6 aa
         cb 00 21 20 3a dd 4f b2 d8 fd f8 22 a0 ca 3c f7 67 8e f5 e8 8d
         ae 99 01 41 c5 92 4d 57 bb 6f a3 1b 9e 5f 9d

      complete record (517 octets):  16 03 01 02 00 01 00 01 fc 03 03 1b
         c3 ce b6 bb e3 9c ff 93 83 55 b5 a5 0a db 6d b2 1b 7a 6a f6 49
         d7 b4 bc 41 9d 78 76 48 7d 95 00 00 06 13 01 13 03 13 02 01 00
         01 cd 00 00 00 0b 00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01
         00 00 0a 00 14 00 12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02
         01 03 01 04 00 33 00 26 00 24 00 1d 00 20 e4 ff b6 8a c0 5f 8d
         96 c9 9d a2 66 98 34 6c 6b e1 64 82 ba dd da fe 05 1a 66 b4 f1
         8d 66 8f 0b 00 2a 00 00 00 2b 00 03 02 03 04 00 0d 00 20 00 1e
         04 03 05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02
         01 04 02 05 02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01
         00 15 00 57 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 29 00 dd 00 b8 00 b2 2c 03 5d 82 93 59
         ee 5f f7 af 4e c9 00 00 00 00 26 2a 64 94 dc 48 6d 2c 8a 34 cb
         33 fa 90 bf 1b 00 70 ad 3c 49 88 83 c9 36 7c 09 a2 be 78 5a bc
         55 cd 22 60 97 a3 a9 82 11 72 83 f8 2a 03 a1 43 ef d3 ff 5d d3
         6d 64 e8 61 be 7f d6 1d 28 27 db 27 9c ce 14 50 77 d4 54 a3 66
         4d 4e 6d a4 d2 9e e0 37 25 a6 a4 da fc d0 fc 67 d2 ae a7 05 29
         51 3e 3d a2 67 7f a5 90 6c 5b 3f 7d 8f 92 f2 28 bd a4 0d da 72
         14 70 f9 fb f2 97 b5 ae a6 17 64 6f ac 5c 03 27 2e 97 07 27 c6
         21 a7 91 41 ef 5f 7d e6 50 5e 5b fb c3 88 e9 33 43 69 40 93 93
         4a e4 d3 57 fa d6 aa cb 00 21 20 3a dd 4f b2 d8 fd f8 22 a0 ca
         3c f7 67 8e f5 e8 8d ae 99 01 41 c5 92 4d 57 bb 6f a3 1b 9e 5f
         9d

   {client}  derive secret "tls13 c e traffic":

      PRK (32 octets):  9b 21 88 e9 b2 fc 6d 64 d7 1d c3 29 90 0e 20 bb
         41 91 50 00 f6 78 aa 83 9c bb 79 7c b7 d8 33 2c

      hash (32 octets):  08 ad 0f a0 5d 7c 72 33 b1 77 5b a2 ff 9f 4c 5b
         8b 59 27 6b 7f 22 7f 13 a9 76 24 5f 5d 96 09 13

      info (53 octets):  00 20 11 74 6c 73 31 33 20 63 20 65 20 74 72 61
         66 66 69 63 20 08 ad 0f a0 5d 7c 72 33 b1 77 5b a2 ff 9f 4c 5b
         8b 59 27 6b 7f 22 7f 13 a9 76 24 5f 5d 96 09 13

      expanded (32 octets):  3f bb e6 a6 0d eb 66 c3 0a 32 79 5a ba 0e
         ff 7e aa 10 10 55 86 e7 be 5c 09 67 8d 63 b6 ca ab 62

   {client}  derive secret "tls13 e exp master":

      PRK (32 octets):  9b 21 88 e9 b2 fc 6d 64 d7 1d c3 29 90 0e 20 bb
         41 91 50 00 f6 78 aa 83 9c bb 79 7c b7 d8 33 2c

      hash (32 octets):  08 ad 0f a0 5d 7c 72 33 b1 77 5b a2 ff 9f 4c 5b
         8b 59 27 6b 7f 22 7f 13 a9 76 24 5f 5d 96 09 13

      info (54 octets):  00 20 12 74 6c 73 31 33 20 65 20 65 78 70 20 6d
         61 73 74 65 72 20 08 ad 0f a0 5d 7c 72 33 b1 77 5b a2 ff 9f 4c
         5b 8b 59 27 6b 7f 22 7f 13 a9 76 24 5f 5d 96 09 13

      expanded (32 octets):  b2 02 68 66 61 09 37 d7 42 3e 5b e9 08 62
         cc f2 4c 0e 60 91 18 6d 34 f8 12 08 9f f5 be 2e f7 df 
   {client}  derive write traffic keys for early application data:

      PRK (32 octets):  3f bb e6 a6 0d eb 66 c3 0a 32 79 5a ba 0e ff 7e
         aa 10 10 55 86 e7 be 5c 09 67 8d 63 b6 ca ab 62

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  92 02 05 a5 b7 bf 21 15 e6 fc 5c 29 42
         83 4f 54

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  6d 47 5f 09 93 c8 e5 64 61 0d b2 b9

   {client}  send application_data record:

      payload (6 octets):  41 42 43 44 45 46

      complete record (28 octets):  17 03 03 00 17 ab 1d f4 20 e7 5c 45
         7a 7c c5 d2 84 4f 76 d5 ae e4 b4 ed bf 04 9b e0

   {server}  extract secret "early" (same as client early secret)

   {server}  calculate PSK binder (same as client):

   {server}  create an ephemeral x25519 key pair:

      private key (32 octets):  de 5b 44 76 e7 b4 90 b2 65 2d 33 8a cb
         f2 94 80 66 f2 55 f9 44 0e 23 b9 8f c6 98 35 29 8d c1 07

      public key (32 octets):  12 17 61 ee 42 c3 33 e1 b9 e7 7b 60 dd 57
         c2 05 3c d9 45 12 ab 47 f1 15 e8 6e ff 50 94 2c ea 31

   {server}  derive secret "tls13 c e traffic" (same as client)

   {server}  derive secret "tls13 e exp master" (same as client)

   {server}  construct a ServerHello handshake message:

      ServerHello (96 octets):  02 00 00 5c 03 03 3c cf d2 de c8 90 22
         27 63 47 2a e8 13 67 77 c9 d7 35 87 77 bb 66 e9 1e a5 12 24 95
         f5 59 ea 2d 00 13 01 00 00 34 00 29 00 02 00 00 00 33 00 24 00
         1d 00 20 12 17 61 ee 42 c3 33 e1 b9 e7 7b 60 dd 57 c2 05 3c d9
         45 12 ab 47 f1 15 e8 6e ff 50 94 2c ea 31 00 2b 00 02 03 04
   {server}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  9b 21 88 e9 b2 fc 6d 64 d7 1d c3 29 90 0e 20 bb
         41 91 50 00 f6 78 aa 83 9c bb 79 7c b7 d8 33 2c

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  5f 17 90 bb d8 2c 5e 7d 37 6e d2 e1 e5 2f
         8e 60 38 c9 34 6d b6 1b 43 be 9a 52 f7 7e f3 99 8e 80

   {server}  extract secret "handshake":

      salt (32 octets):  5f 17 90 bb d8 2c 5e 7d 37 6e d2 e1 e5 2f 8e 60
         38 c9 34 6d b6 1b 43 be 9a 52 f7 7e f3 99 8e 80

      IKM (32 octets):  f4 41 94 75 6f f9 ec 9d 25 18 06 35 d6 6e a6 82
         4c 6a b3 bf 17 99 77 be 37 f7 23 57 0e 7c cb 2e

      secret (32 octets):  00 5c b1 12 fd 8e b4 cc c6 23 bb 88 a0 7c 64
         b3 ed e1 60 53 63 fc 7d 0d f8 c7 ce 4f f0 fb 4a e6

   {server}  derive secret "tls13 c hs traffic":

      PRK (32 octets):  00 5c b1 12 fd 8e b4 cc c6 23 bb 88 a0 7c 64 b3
         ed e1 60 53 63 fc 7d 0d f8 c7 ce 4f f0 fb 4a e6

      hash (32 octets):  f7 36 cb 34 fe 25 e7 01 55 1b ee 6f d2 4c 1c c7
         10 2a 7d af 94 05 cb 15 d9 7a af e1 6f 75 7d 03

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 68 73 20 74 72
         61 66 66 69 63 20 f7 36 cb 34 fe 25 e7 01 55 1b ee 6f d2 4c 1c
         c7 10 2a 7d af 94 05 cb 15 d9 7a af e1 6f 75 7d 03

      expanded (32 octets):  2f aa c0 8f 85 1d 35 fe a3 60 4f cb 4d e8
         2d c6 2c 9b 16 4a 70 97 4d 04 62 e2 7f 1a b2 78 70 0f

   {server}  derive secret "tls13 s hs traffic":

      PRK (32 octets):  00 5c b1 12 fd 8e b4 cc c6 23 bb 88 a0 7c 64 b3
         ed e1 60 53 63 fc 7d 0d f8 c7 ce 4f f0 fb 4a e6

      hash (32 octets):  f7 36 cb 34 fe 25 e7 01 55 1b ee 6f d2 4c 1c c7
         10 2a 7d af 94 05 cb 15 d9 7a af e1 6f 75 7d 03
      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 68 73 20 74 72
         61 66 66 69 63 20 f7 36 cb 34 fe 25 e7 01 55 1b ee 6f d2 4c 1c
         c7 10 2a 7d af 94 05 cb 15 d9 7a af e1 6f 75 7d 03

      expanded (32 octets):  fe 92 7a e2 71 31 2e 8b f0 27 5b 58 1c 54
         ee f0 20 45 0d c4 ec ff aa 05 a1 a3 5d 27 51 8e 78 03

   {server}  derive secret for master "tls13 derived":

      PRK (32 octets):  00 5c b1 12 fd 8e b4 cc c6 23 bb 88 a0 7c 64 b3
         ed e1 60 53 63 fc 7d 0d f8 c7 ce 4f f0 fb 4a e6

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  e2 f1 60 30 25 1d f0 87 4b a1 9b 9a ba 25
         76 10 bc 6d 53 1c 1d d2 06 df 0c a6 e8 4a e2 a2 67 42

   {server}  extract secret "master":

      salt (32 octets):  e2 f1 60 30 25 1d f0 87 4b a1 9b 9a ba 25 76 10
         bc 6d 53 1c 1d d2 06 df 0c a6 e8 4a e2 a2 67 42

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  e2 d3 2d 4e d6 6d d3 78 97 a0 e8 0c 84 10 75
         03 ce 58 bf 8a ad 4c b5 5a 50 02 d7 7e cb 89 0e ce

   {server}  send handshake record:

      payload (96 octets):  02 00 00 5c 03 03 3c cf d2 de c8 90 22 27 63
         47 2a e8 13 67 77 c9 d7 35 87 77 bb 66 e9 1e a5 12 24 95 f5 59
         ea 2d 00 13 01 00 00 34 00 29 00 02 00 00 00 33 00 24 00 1d 00
         20 12 17 61 ee 42 c3 33 e1 b9 e7 7b 60 dd 57 c2 05 3c d9 45 12
         ab 47 f1 15 e8 6e ff 50 94 2c ea 31 00 2b 00 02 03 04

      complete record (101 octets):  16 03 03 00 60 02 00 00 5c 03 03 3c
         cf d2 de c8 90 22 27 63 47 2a e8 13 67 77 c9 d7 35 87 77 bb 66
         e9 1e a5 12 24 95 f5 59 ea 2d 00 13 01 00 00 34 00 29 00 02 00
         00 00 33 00 24 00 1d 00 20 12 17 61 ee 42 c3 33 e1 b9 e7 7b 60
         dd 57 c2 05 3c d9 45 12 ab 47 f1 15 e8 6e ff 50 94 2c ea 31 00
         2b 00 02 03 04
   {server}  derive write traffic keys for handshake data:

      PRK (32 octets):  fe 92 7a e2 71 31 2e 8b f0 27 5b 58 1c 54 ee f0
         20 45 0d c4 ec ff aa 05 a1 a3 5d 27 51 8e 78 03

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  27 c6 bd c0 a3 dc ea 39 a4 73 26 d7 9b
         c9 e4 ee

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  95 69 ec dd 4d 05 36 70 5e 9e f7 25

   {server}  construct an EncryptedExtensions handshake message:

      EncryptedExtensions (44 octets):  08 00 00 28 00 26 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c
         00 02 40 01 00 00 00 00 00 2a 00 00

   {server}  calculate finished "tls13 finished":

      PRK (32 octets):  fe 92 7a e2 71 31 2e 8b f0 27 5b 58 1c 54 ee f0
         20 45 0d c4 ec ff aa 05 a1 a3 5d 27 51 8e 78 03

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  4b b7 4c ae 7a 5d c8 91 46 04 c0 bf be 2f
         0c 06 23 96 88 39 22 be c8 a1 5e 2a 9b 53 2a 5d 39 2c

      finished (32 octets):  48 d3 e0 e1 b3 d9 07 c6 ac ff 14 5e 16 09
         03 88 c7 7b 05 c0 50 b6 34 ab 1a 88 bb d0 dd 1a 34 b2

   {server}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 48 d3 e0 e1 b3 d9 07 c6 ac ff
         14 5e 16 09 03 88 c7 7b 05 c0 50 b6 34 ab 1a 88 bb d0 dd 1a 34
         b2
   {server}  send handshake record:

      payload (80 octets):  08 00 00 28 00 26 00 0a 00 14 00 12 00 1d 00
         17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c 00 02 40 01
         00 00 00 00 00 2a 00 00 14 00 00 20 48 d3 e0 e1 b3 d9 07 c6 ac
         ff 14 5e 16 09 03 88 c7 7b 05 c0 50 b6 34 ab 1a 88 bb d0 dd 1a
         34 b2

      complete record (102 octets):  17 03 03 00 61 dc 48 23 7b 4b 87 9f
         50 d0 d4 d2 62 ea 8b 47 16 eb 40 dd c1 eb 95 7e 11 12 6e 8a 71
         49 c2 d0 12 d3 7a 71 15 95 7e 64 ce 30 00 8b 9e 03 23 f2 c0 5a
         9c 1c 77 b4 f3 78 49 a6 95 ab 25 50 60 a3 3f ee 77 0c a9 5c b8
         48 6b fd 08 43 b8 70 24 86 5c a3 5c c4 1c 4e 51 5c 64 dc b1 36
         9f 98 63 5b c7 a5

   {server}  derive secret "tls13 c ap traffic":

      PRK (32 octets):  e2 d3 2d 4e d6 6d d3 78 97 a0 e8 0c 84 10 75 03
         ce 58 bf 8a ad 4c b5 5a 50 02 d7 7e cb 89 0e ce

      hash (32 octets):  b0 ae ff c4 6a 2c fe 33 11 4e 6f d7 d5 1f 9f 04
         b1 ca 3c 49 7d ab 08 93 4a 77 4a 9d 9a d7 db f3

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 61 70 20 74 72
         61 66 66 69 63 20 b0 ae ff c4 6a 2c fe 33 11 4e 6f d7 d5 1f 9f
         04 b1 ca 3c 49 7d ab 08 93 4a 77 4a 9d 9a d7 db f3

      expanded (32 octets):  2a bb f2 b8 e3 81 d2 3d be be 1d d2 a7 d1
         6a 8b f4 84 cb 49 50 d2 3f b7 fb 7f a8 54 70 62 d9 a1

   {server}  derive secret "tls13 s ap traffic":

      PRK (32 octets):  e2 d3 2d 4e d6 6d d3 78 97 a0 e8 0c 84 10 75 03
         ce 58 bf 8a ad 4c b5 5a 50 02 d7 7e cb 89 0e ce

      hash (32 octets):  b0 ae ff c4 6a 2c fe 33 11 4e 6f d7 d5 1f 9f 04
         b1 ca 3c 49 7d ab 08 93 4a 77 4a 9d 9a d7 db f3

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 61 70 20 74 72
         61 66 66 69 63 20 b0 ae ff c4 6a 2c fe 33 11 4e 6f d7 d5 1f 9f
         04 b1 ca 3c 49 7d ab 08 93 4a 77 4a 9d 9a d7 db f3

      expanded (32 octets):  cc 21 f1 bf 8f eb 7d d5 fa 50 5b d9 c4 b4
         68 a9 98 4d 55 4a 99 3d c4 9e 6d 28 55 98 fb 67 26 91
   {server}  derive secret "tls13 exp master":

      PRK (32 octets):  e2 d3 2d 4e d6 6d d3 78 97 a0 e8 0c 84 10 75 03
         ce 58 bf 8a ad 4c b5 5a 50 02 d7 7e cb 89 0e ce

      hash (32 octets):  b0 ae ff c4 6a 2c fe 33 11 4e 6f d7 d5 1f 9f 04
         b1 ca 3c 49 7d ab 08 93 4a 77 4a 9d 9a d7 db f3

      info (52 octets):  00 20 10 74 6c 73 31 33 20 65 78 70 20 6d 61 73
         74 65 72 20 b0 ae ff c4 6a 2c fe 33 11 4e 6f d7 d5 1f 9f 04 b1
         ca 3c 49 7d ab 08 93 4a 77 4a 9d 9a d7 db f3

      expanded (32 octets):  3f d9 3d 4f fd dc 98 e6 4b 14 dd 10 7a ed
         f8 ee 4a dd 23 f4 51 0f 58 a4 59 2d 0b 20 1b ee 56 b4

   {server}  derive write traffic keys for application data:

      PRK (32 octets):  cc 21 f1 bf 8f eb 7d d5 fa 50 5b d9 c4 b4 68 a9
         98 4d 55 4a 99 3d c4 9e 6d 28 55 98 fb 67 26 91

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  e8 57 c6 90 a3 4c 5a 91 29 d8 33 61 96
         84 f9 5e

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  06 85 d6 b5 61 aa b9 ef 10 13 fa f9

   {server}  derive read traffic keys for early application data (same
      as client early application data write traffic keys)

   {client}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  9b 21 88 e9 b2 fc 6d 64 d7 1d c3 29 90 0e 20 bb
         41 91 50 00 f6 78 aa 83 9c bb 79 7c b7 d8 33 2c

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  5f 17 90 bb d8 2c 5e 7d 37 6e d2 e1 e5 2f
         8e 60 38 c9 34 6d b6 1b 43 be 9a 52 f7 7e f3 99 8e 80
   {client}  extract secret "handshake" (same as server handshake
      secret)

   {client}  derive secret "tls13 c hs traffic" (same as server)

   {client}  derive secret "tls13 s hs traffic" (same as server)

   {client}  derive secret for master "tls13 derived" (same as server)

   {client}  extract secret "master" (same as server master secret)

   {client}  derive read traffic keys for handshake data (same as server
      handshake data write traffic keys)

   {client}  calculate finished "tls13 finished" (same as server)

   {client}  derive secret "tls13 c ap traffic" (same as server)

   {client}  derive secret "tls13 s ap traffic" (same as server)

   {client}  derive secret "tls13 exp master" (same as server)

   {client}  construct an EndOfEarlyData handshake message:

      EndOfEarlyData (4 octets):  05 00 00 00

   {client}  send handshake record:

      payload (4 octets):  05 00 00 00

      complete record (26 octets):  17 03 03 00 15 ac a6 fc 94 48 41 29
         8d f9 95 93 72 5f 9b f9 75 44 29 b1 2f 09

   {client}  derive write traffic keys for handshake data:

      PRK (32 octets):  2f aa c0 8f 85 1d 35 fe a3 60 4f cb 4d e8 2d c6
         2c 9b 16 4a 70 97 4d 04 62 e2 7f 1a b2 78 70 0f

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  b1 53 08 06 f4 ad fe ac 83 f1 41 30 32
         bb fa 82

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  eb 50 c1 6b e7 65 4a bf 99 dd 06 d9
   {client}  derive read traffic keys for application data (same as
      server application data write traffic keys)

   {client}  calculate finished "tls13 finished":

      PRK (32 octets):  2f aa c0 8f 85 1d 35 fe a3 60 4f cb 4d e8 2d c6
         2c 9b 16 4a 70 97 4d 04 62 e2 7f 1a b2 78 70 0f

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  5a ce 39 4c 26 98 0d 58 12 43 f6 27 d1 15
         0a e2 7e 37 fa 52 36 4e 0a 7f 20 ac 68 6d 09 cd 0e 8e

      finished (32 octets):  72 30 a9 c9 52 c2 5c d6 13 8f c5 e6 62 83
         08 c4 1c 53 35 dd 81 b9 f9 6b ce a5 0f d3 2b da 41 6d

   {client}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 72 30 a9 c9 52 c2 5c d6 13 8f
         c5 e6 62 83 08 c4 1c 53 35 dd 81 b9 f9 6b ce a5 0f d3 2b da 41
         6d

   {client}  send handshake record:

      payload (36 octets):  14 00 00 20 72 30 a9 c9 52 c2 5c d6 13 8f c5
         e6 62 83 08 c4 1c 53 35 dd 81 b9 f9 6b ce a5 0f d3 2b da 41 6d

      complete record (58 octets):  17 03 03 00 35 00 f8 b4 67 d1 4c f2
         2a 4b 3f 0b 6a e0 d8 e6 cc 8d 08 e0 db 35 15 ef 5c 2b df 19 22
         ea fb b7 00 09 96 47 16 d8 34 fb 70 c3 d2 a5 6c 5b 1f 5f 6b db
         a6 c3 33 cf

   {client}  derive write traffic keys for application data:

      PRK (32 octets):  2a bb f2 b8 e3 81 d2 3d be be 1d d2 a7 d1 6a 8b
         f4 84 cb 49 50 d2 3f b7 fb 7f a8 54 70 62 d9 a1

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  3c f1 22 f3 01 c6 35 8c a7 98 95 53 25
         0e fd 72

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  ab 1a ec 26 aa 78 b8 fc 11 76 b9 ac 
   {client}  derive secret "tls13 res master":

      PRK (32 octets):  e2 d3 2d 4e d6 6d d3 78 97 a0 e8 0c 84 10 75 03
         ce 58 bf 8a ad 4c b5 5a 50 02 d7 7e cb 89 0e ce

      hash (32 octets):  c3 c1 22 e0 bd 90 7a 4a 3f f6 11 2d 8f d5 3d bf
         89 c7 73 d9 55 2e 8b 6b 9d 56 d3 61 b3 a9 7b f6

      info (52 octets):  00 20 10 74 6c 73 31 33 20 72 65 73 20 6d 61 73
         74 65 72 20 c3 c1 22 e0 bd 90 7a 4a 3f f6 11 2d 8f d5 3d bf 89
         c7 73 d9 55 2e 8b 6b 9d 56 d3 61 b3 a9 7b f6

      expanded (32 octets):  5e 95 bd f1 f8 90 05 ea 2e 9a a0 ba 85 e7
         28 e3 c1 9c 5f e0 c6 99 e3 f5 be e5 9f ae bd 0b 54 06

   {server}  derive read traffic keys for handshake data (same as client
      handshake data write traffic keys)

   {server}  calculate finished "tls13 finished" (same as client)

   {server}  derive read traffic keys for application data (same as
      client application data write traffic keys)

   {server}  derive secret "tls13 res master" (same as client)

   {client}  send application_data record:

      payload (50 octets):  00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e
         0f 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23
         24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30 31

      complete record (72 octets):  17 03 03 00 43 b1 ce bc e2 42 aa 20
         1b e9 ae 5e 1c b2 a9 aa 4b 33 d4 e8 66 af 1e db 06 89 19 23 77
         41 aa 03 1d 7a 74 d4 91 c9 9b 9d 4e 23 2b 74 20 6b c6 fb aa 04
         fe 78 be 44 a9 b4 f5 43 20 a1 7e b7 69 92 af ac 31 03

   {server}  send application_data record:

      payload (50 octets):  00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e
         0f 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23
         24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30 31

      complete record (72 octets):  17 03 03 00 43 27 5e 9f 20 ac ff 57
         bc 00 06 57 d3 86 7d f0 39 cc cf 79 04 78 84 cf 75 77 17 46 f7
         40 b5 a8 3f 46 2a 09 54 c3 58 13 93 a2 03 a2 5a 7d d1 41 41 ef
         1a 37 90 0c db 62 ff 62 de e1 ba 39 ab 25 90 cb f1 94
   {client}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 0f ac ce 32 46 bd fc
         63 69 83 8d 6a 82 ae 6d e5 d4 22 dc

   {server}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 5b 18 af 44 4e 8e 1e
         ec 71 58 fb 62 d8 f2 57 7d 37 ba 5d

## 5. HelloRetryRequest

   In this example, the client initiates a handshake with an X25519
   [RFC7748] share.  The server, however, prefers P-256
   [FIPS.186-4.2013] and sends a HelloRetryRequest that requires the
   client to generate a key share on the P-256 curve.

   Note:  The HelloRetryRequest uses the same handshake message type as
      a ServerHello and so is labeled as ServerHello here.

   {client}  create an ephemeral x25519 key pair:

      private key (32 octets):  0e d0 2f 8e 81 17 ef c7 5c a7 ac 32 aa
         7e 34 ed a6 4c dc 0d da d1 54 a5 e8 52 89 f9 59 f6 32 04

      public key (32 octets):  e8 e8 e3 f3 b9 3a 25 ed 97 a1 4a 7d ca cb
         8a 27 2c 62 88 e5 85 c6 48 4d 05 26 2f ca d0 62 ad 1f

   {client}  construct a ClientHello handshake message:

      ClientHello (180 octets):  01 00 00 b0 03 03 b0 b1 c5 a5 aa 37 c5
         91 9f 2e d1 d5 c6 ff f7 fc b7 84 97 16 94 5a 2b 8c ee 92 58 a3
         46 67 7b 6f 00 00 06 13 01 13 03 13 02 01 00 00 81 00 00 00 0b
         00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 08 00
         06 00 1d 00 17 00 18 00 33 00 26 00 24 00 1d 00 20 e8 e8 e3 f3
         b9 3a 25 ed 97 a1 4a 7d ca cb 8a 27 2c 62 88 e5 85 c6 48 4d 05
         26 2f ca d0 62 ad 1f 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04
         03 05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01
         04 02 05 02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01
   {client}  send handshake record:

      payload (180 octets):  01 00 00 b0 03 03 b0 b1 c5 a5 aa 37 c5 91
         9f 2e d1 d5 c6 ff f7 fc b7 84 97 16 94 5a 2b 8c ee 92 58 a3 46
         67 7b 6f 00 00 06 13 01 13 03 13 02 01 00 00 81 00 00 00 0b 00
         09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 08 00 06
         00 1d 00 17 00 18 00 33 00 26 00 24 00 1d 00 20 e8 e8 e3 f3 b9
         3a 25 ed 97 a1 4a 7d ca cb 8a 27 2c 62 88 e5 85 c6 48 4d 05 26
         2f ca d0 62 ad 1f 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03
         05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04
         02 05 02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01

      complete record (185 octets):  16 03 01 00 b4 01 00 00 b0 03 03 b0
         b1 c5 a5 aa 37 c5 91 9f 2e d1 d5 c6 ff f7 fc b7 84 97 16 94 5a
         2b 8c ee 92 58 a3 46 67 7b 6f 00 00 06 13 01 13 03 13 02 01 00
         00 81 00 00 00 0b 00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01
         00 00 0a 00 08 00 06 00 1d 00 17 00 18 00 33 00 26 00 24 00 1d
         00 20 e8 e8 e3 f3 b9 3a 25 ed 97 a1 4a 7d ca cb 8a 27 2c 62 88
         e5 85 c6 48 4d 05 26 2f ca d0 62 ad 1f 00 2b 00 03 02 03 04 00
         0d 00 20 00 1e 04 03 05 03 06 03 02 03 08 04 08 05 08 06 04 01
         05 01 06 01 02 01 04 02 05 02 06 02 02 02 00 2d 00 02 01 01 00
         1c 00 02 40 01

   {server}  construct a ServerHello handshake message:

      ServerHello (176 octets):  02 00 00 ac 03 03 cf 21 ad 74 e5 9a 61
         11 be 1d 8c 02 1e 65 b8 91 c2 a2 11 16 7a bb 8c 5e 07 9e 09 e2
         c8 a8 33 9c 00 13 01 00 00 84 00 33 00 02 00 17 00 2c 00 74 00
         72 71 dc d0 4b b8 8b c3 18 91 19 39 8a 00 00 00 00 ee fa fc 76
         c1 46 b8 23 b0 96 f8 aa ca d3 65 dd 00 30 95 3f 4e df 62 56 36
         e5 f2 1b b2 e2 3f cc 65 4b 1b 5b 40 31 8d 10 d1 37 ab cb b8 75
         74 e3 6e 8a 1f 02 5f 7d fa 5d 6e 50 78 1b 5e da 4a a1 5b 0c 8b
         e7 78 25 7d 16 aa 30 30 e9 e7 84 1d d9 e4 c0 34 22 67 e8 ca 0c
         af 57 1f b2 b7 cf f0 f9 34 b0 00 2b 00 02 03 04

   {server}  send handshake record:

      payload (176 octets):  02 00 00 ac 03 03 cf 21 ad 74 e5 9a 61 11
         be 1d 8c 02 1e 65 b8 91 c2 a2 11 16 7a bb 8c 5e 07 9e 09 e2 c8
         a8 33 9c 00 13 01 00 00 84 00 33 00 02 00 17 00 2c 00 74 00 72
         71 dc d0 4b b8 8b c3 18 91 19 39 8a 00 00 00 00 ee fa fc 76 c1
         46 b8 23 b0 96 f8 aa ca d3 65 dd 00 30 95 3f 4e df 62 56 36 e5
         f2 1b b2 e2 3f cc 65 4b 1b 5b 40 31 8d 10 d1 37 ab cb b8 75 74
         e3 6e 8a 1f 02 5f 7d fa 5d 6e 50 78 1b 5e da 4a a1 5b 0c 8b e7
         78 25 7d 16 aa 30 30 e9 e7 84 1d d9 e4 c0 34 22 67 e8 ca 0c af
         57 1f b2 b7 cf f0 f9 34 b0 00 2b 00 02 03 04
      complete record (181 octets):  16 03 03 00 b0 02 00 00 ac 03 03 cf
         21 ad 74 e5 9a 61 11 be 1d 8c 02 1e 65 b8 91 c2 a2 11 16 7a bb
         8c 5e 07 9e 09 e2 c8 a8 33 9c 00 13 01 00 00 84 00 33 00 02 00
         17 00 2c 00 74 00 72 71 dc d0 4b b8 8b c3 18 91 19 39 8a 00 00
         00 00 ee fa fc 76 c1 46 b8 23 b0 96 f8 aa ca d3 65 dd 00 30 95
         3f 4e df 62 56 36 e5 f2 1b b2 e2 3f cc 65 4b 1b 5b 40 31 8d 10
         d1 37 ab cb b8 75 74 e3 6e 8a 1f 02 5f 7d fa 5d 6e 50 78 1b 5e
         da 4a a1 5b 0c 8b e7 78 25 7d 16 aa 30 30 e9 e7 84 1d d9 e4 c0
         34 22 67 e8 ca 0c af 57 1f b2 b7 cf f0 f9 34 b0 00 2b 00 02 03
         04

   {client}  create an ephemeral P-256 key pair:

      private key (32 octets):  ab 54 73 46 7e 19 34 6c eb 0a 04 14 e4
         1d a2 1d 4d 24 45 bc 30 25 af e9 7c 4e 8d c8 d5 13 da 39

      public key (65 octets):  04 a6 da 73 92 ec 59 1e 17 ab fd 53 59 64
         b9 98 94 d1 3b ef b2 21 b3 de f2 eb e3 83 0e ac 8f 01 51 81 26
         77 c4 d6 d2 23 7e 85 cf 01 d6 91 0c fb 83 95 4e 76 ba 73 52 83
         05 34 15 98 97 e8 06 57 80

   {client}  construct a ClientHello handshake message:

      ClientHello (512 octets):  01 00 01 fc 03 03 b0 b1 c5 a5 aa 37 c5
         91 9f 2e d1 d5 c6 ff f7 fc b7 84 97 16 94 5a 2b 8c ee 92 58 a3
         46 67 7b 6f 00 00 06 13 01 13 03 13 02 01 00 01 cd 00 00 00 0b
         00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 08 00
         06 00 1d 00 17 00 18 00 33 00 47 00 45 00 17 00 41 04 a6 da 73
         92 ec 59 1e 17 ab fd 53 59 64 b9 98 94 d1 3b ef b2 21 b3 de f2
         eb e3 83 0e ac 8f 01 51 81 26 77 c4 d6 d2 23 7e 85 cf 01 d6 91
         0c fb 83 95 4e 76 ba 73 52 83 05 34 15 98 97 e8 06 57 80 00 2b
         00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03 02 03 08 04
         08 05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06 02 02 02 00
         2c 00 74 00 72 71 dc d0 4b b8 8b c3 18 91 19 39 8a 00 00 00 00
         ee fa fc 76 c1 46 b8 23 b0 96 f8 aa ca d3 65 dd 00 30 95 3f 4e
         df 62 56 36 e5 f2 1b b2 e2 3f cc 65 4b 1b 5b 40 31 8d 10 d1 37
         ab cb b8 75 74 e3 6e 8a 1f 02 5f 7d fa 5d 6e 50 78 1b 5e da 4a
         a1 5b 0c 8b e7 78 25 7d 16 aa 30 30 e9 e7 84 1d d9 e4 c0 34 22
         67 e8 ca 0c af 57 1f b2 b7 cf f0 f9 34 b0 00 2d 00 02 01 01 00
         1c 00 02 40 01 00 15 00 af 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
   {client}  send handshake record:

      payload (512 octets):  01 00 01 fc 03 03 b0 b1 c5 a5 aa 37 c5 91
         9f 2e d1 d5 c6 ff f7 fc b7 84 97 16 94 5a 2b 8c ee 92 58 a3 46
         67 7b 6f 00 00 06 13 01 13 03 13 02 01 00 01 cd 00 00 00 0b 00
         09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 08 00 06
         00 1d 00 17 00 18 00 33 00 47 00 45 00 17 00 41 04 a6 da 73 92
         ec 59 1e 17 ab fd 53 59 64 b9 98 94 d1 3b ef b2 21 b3 de f2 eb
         e3 83 0e ac 8f 01 51 81 26 77 c4 d6 d2 23 7e 85 cf 01 d6 91 0c
         fb 83 95 4e 76 ba 73 52 83 05 34 15 98 97 e8 06 57 80 00 2b 00
         03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03 02 03 08 04 08
         05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06 02 02 02 00 2c
         00 74 00 72 71 dc d0 4b b8 8b c3 18 91 19 39 8a 00 00 00 00 ee
         fa fc 76 c1 46 b8 23 b0 96 f8 aa ca d3 65 dd 00 30 95 3f 4e df
         62 56 36 e5 f2 1b b2 e2 3f cc 65 4b 1b 5b 40 31 8d 10 d1 37 ab
         cb b8 75 74 e3 6e 8a 1f 02 5f 7d fa 5d 6e 50 78 1b 5e da 4a a1
         5b 0c 8b e7 78 25 7d 16 aa 30 30 e9 e7 84 1d d9 e4 c0 34 22 67
         e8 ca 0c af 57 1f b2 b7 cf f0 f9 34 b0 00 2d 00 02 01 01 00 1c
         00 02 40 01 00 15 00 af 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      complete record (517 octets):  16 03 03 02 00 01 00 01 fc 03 03 b0
         b1 c5 a5 aa 37 c5 91 9f 2e d1 d5 c6 ff f7 fc b7 84 97 16 94 5a
         2b 8c ee 92 58 a3 46 67 7b 6f 00 00 06 13 01 13 03 13 02 01 00
         01 cd 00 00 00 0b 00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01
         00 00 0a 00 08 00 06 00 1d 00 17 00 18 00 33 00 47 00 45 00 17
         00 41 04 a6 da 73 92 ec 59 1e 17 ab fd 53 59 64 b9 98 94 d1 3b
         ef b2 21 b3 de f2 eb e3 83 0e ac 8f 01 51 81 26 77 c4 d6 d2 23
         7e 85 cf 01 d6 91 0c fb 83 95 4e 76 ba 73 52 83 05 34 15 98 97
         e8 06 57 80 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03
         06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05
         02 06 02 02 02 00 2c 00 74 00 72 71 dc d0 4b b8 8b c3 18 91 19
         39 8a 00 00 00 00 ee fa fc 76 c1 46 b8 23 b0 96 f8 aa ca d3 65
         dd 00 30 95 3f 4e df 62 56 36 e5 f2 1b b2 e2 3f cc 65 4b 1b 5b
         40 31 8d 10 d1 37 ab cb b8 75 74 e3 6e 8a 1f 02 5f 7d fa 5d 6e
         50 78 1b 5e da 4a a1 5b 0c 8b e7 78 25 7d 16 aa 30 30 e9 e7 84
         1d d9 e4 c0 34 22 67 e8 ca 0c af 57 1f b2 b7 cf f0 f9 34 b0 00
         2d 00 02 01 01 00 1c 00 02 40 01 00 15 00 af 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00

   {server}  extract secret "early":

      salt:  0 (all zero octets)

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c
         e2 10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

   {server}  create an ephemeral P-256 key pair:

      private key (32 octets):  8c 51 06 01 f9 76 5b fb 8e d6 93 44 9a
         48 98 98 59 b5 cf a8 79 cb 9f 54 43 c4 1c 5f f1 06 34 ed

      public key (65 octets):  04 58 3e 05 4b 7a 66 67 2a e0 20 ad 9d 26
         86 fc c8 5b 5a d4 1a 13 4a 0f 03 ee 72 b8 93 05 2b d8 5b 4c 8d
         e6 77 6f 5b 04 ac 07 d8 35 40 ea b3 e3 d9 c5 47 bc 65 28 c4 31
         7d 29 46 86 09 3a 6c ad 7d

   {server}  construct a ServerHello handshake message:

      ServerHello (123 octets):  02 00 00 77 03 03 bb 34 1d 84 7f d7 89
         c4 7c 38 71 72 dc 0c 9b f1 47 fc ca cb 50 43 d8 6c a4 c5 98 d3
         ff 57 1b 98 00 13 01 00 00 4f 00 33 00 45 00 17 00 41 04 58 3e
         05 4b 7a 66 67 2a e0 20 ad 9d 26 86 fc c8 5b 5a d4 1a 13 4a 0f
         03 ee 72 b8 93 05 2b d8 5b 4c 8d e6 77 6f 5b 04 ac 07 d8 35 40
         ea b3 e3 d9 c5 47 bc 65 28 c4 31 7d 29 46 86 09 3a 6c ad 7d 00
         2b 00 02 03 04

   {server}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55
      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {server}  extract secret "handshake":

      salt (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba b6 97
         16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

      IKM (32 octets):  c1 42 ce 13 ca 11 b5 c2 23 36 52 e6 3a d3 d9 78
         44 f1 62 1f bf b9 de 69 d5 47 dc 8f ed ea be b4

      secret (32 octets):  ce 02 2e 5e 6e 81 e5 07 36 d7 73 f2 d3 ad fc
         e8 22 0d 04 9b f5 10 f0 db fa c9 27 ef 42 43 b1 48

   {server}  derive secret "tls13 c hs traffic":

      PRK (32 octets):  ce 02 2e 5e 6e 81 e5 07 36 d7 73 f2 d3 ad fc e8
         22 0d 04 9b f5 10 f0 db fa c9 27 ef 42 43 b1 48

      hash (32 octets):  8a a8 e8 28 ec 2f 8a 88 4f ec 95 a3 13 9d e0 1c
         15 a3 da a7 ff 5b fc 3f 4b fc c2 1b 43 8d 7b f8

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 68 73 20 74 72
         61 66 66 69 63 20 8a a8 e8 28 ec 2f 8a 88 4f ec 95 a3 13 9d e0
         1c 15 a3 da a7 ff 5b fc 3f 4b fc c2 1b 43 8d 7b f8

      expanded (32 octets):  15 8a a7 ab 88 55 07 35 82 b4 1d 67 4b 40
         55 ca bc c5 34 72 8f 65 93 14 86 1b 4e 08 e2 01 15 66

   {server}  derive secret "tls13 s hs traffic":

      PRK (32 octets):  ce 02 2e 5e 6e 81 e5 07 36 d7 73 f2 d3 ad fc e8
         22 0d 04 9b f5 10 f0 db fa c9 27 ef 42 43 b1 48

      hash (32 octets):  8a a8 e8 28 ec 2f 8a 88 4f ec 95 a3 13 9d e0 1c
         15 a3 da a7 ff 5b fc 3f 4b fc c2 1b 43 8d 7b f8

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 68 73 20 74 72
         61 66 66 69 63 20 8a a8 e8 28 ec 2f 8a 88 4f ec 95 a3 13 9d e0
         1c 15 a3 da a7 ff 5b fc 3f 4b fc c2 1b 43 8d 7b f8

      expanded (32 octets):  34 03 e7 81 e2 af 7b 65 08 da 28 57 4f 6e
         95 a1 ab f1 62 de 83 a9 79 27 c3 76 72 a4 a0 ce f8 a1

   {server}  derive secret for master "tls13 derived":

      PRK (32 octets):  ce 02 2e 5e 6e 81 e5 07 36 d7 73 f2 d3 ad fc e8
         22 0d 04 9b f5 10 f0 db fa c9 27 ef 42 43 b1 48
      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  ad 1c bc d3 a0 dc 70 53 ee b3 ed 3a 47 90
         1d 16 a9 fc 63 a7 3c 64 be b5 67 48 1a 7d fb 3a 2c b3

   {server}  extract secret "master":

      salt (32 octets):  ad 1c bc d3 a0 dc 70 53 ee b3 ed 3a 47 90 1d 16
         a9 fc 63 a7 3c 64 be b5 67 48 1a 7d fb 3a 2c b3

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  11 31 54 5d 0b af 79 dd ce 9b 87 f0 69 45 78
         1a 57 dd 18 ef 37 8d cd 20 60 f8 f9 a5 69 02 7e d8

   {server}  send handshake record:

      payload (123 octets):  02 00 00 77 03 03 bb 34 1d 84 7f d7 89 c4
         7c 38 71 72 dc 0c 9b f1 47 fc ca cb 50 43 d8 6c a4 c5 98 d3 ff
         57 1b 98 00 13 01 00 00 4f 00 33 00 45 00 17 00 41 04 58 3e 05
         4b 7a 66 67 2a e0 20 ad 9d 26 86 fc c8 5b 5a d4 1a 13 4a 0f 03
         ee 72 b8 93 05 2b d8 5b 4c 8d e6 77 6f 5b 04 ac 07 d8 35 40 ea
         b3 e3 d9 c5 47 bc 65 28 c4 31 7d 29 46 86 09 3a 6c ad 7d 00 2b
         00 02 03 04

      complete record (128 octets):  16 03 03 00 7b 02 00 00 77 03 03 bb
         34 1d 84 7f d7 89 c4 7c 38 71 72 dc 0c 9b f1 47 fc ca cb 50 43
         d8 6c a4 c5 98 d3 ff 57 1b 98 00 13 01 00 00 4f 00 33 00 45 00
         17 00 41 04 58 3e 05 4b 7a 66 67 2a e0 20 ad 9d 26 86 fc c8 5b
         5a d4 1a 13 4a 0f 03 ee 72 b8 93 05 2b d8 5b 4c 8d e6 77 6f 5b
         04 ac 07 d8 35 40 ea b3 e3 d9 c5 47 bc 65 28 c4 31 7d 29 46 86
         09 3a 6c ad 7d 00 2b 00 02 03 04

   {server}  derive write traffic keys for handshake data:

      PRK (32 octets):  34 03 e7 81 e2 af 7b 65 08 da 28 57 4f 6e 95 a1
         ab f1 62 de 83 a9 79 27 c3 76 72 a4 a0 ce f8 a1

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  46 46 bf ac 17 12 c4 26 cd 78 d8 a2 4a
         8a 6f 6b 
      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  c7 d3 95 c0 8d 62 f2 97 d1 37 68 ea

   {server}  construct an EncryptedExtensions handshake message:

      EncryptedExtensions (28 octets):  08 00 00 18 00 16 00 0a 00 08 00
         06 00 17 00 18 00 1d 00 1c 00 02 40 01 00 00 00 00

   {server}  construct a Certificate handshake message:

      Certificate (445 octets):  0b 00 01 b9 00 00 01 b5 00 01 b0 30 82
         01 ac 30 82 01 15 a0 03 02 01 02 02 01 02 30 0d 06 09 2a 86 48
         86 f7 0d 01 01 0b 05 00 30 0e 31 0c 30 0a 06 03 55 04 03 13 03
         72 73 61 30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35 39 5a 17
         0d 32 36 30 37 33 30 30 31 32 33 35 39 5a 30 0e 31 0c 30 0a 06
         03 55 04 03 13 03 72 73 61 30 81 9f 30 0d 06 09 2a 86 48 86 f7
         0d 01 01 01 05 00 03 81 8d 00 30 81 89 02 81 81 00 b4 bb 49 8f
         82 79 30 3d 98 08 36 39 9b 36 c6 98 8c 0c 68 de 55 e1 bd b8 26
         d3 90 1a 24 61 ea fd 2d e4 9a 91 d0 15 ab bc 9a 95 13 7a ce 6c
         1a f1 9e aa 6a f9 8c 7c ed 43 12 09 98 e1 87 a8 0e e0 cc b0 52
         4b 1b 01 8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a 5f da 43 08 46 74
         80 30 53 0e f0 46 1c 8c a9 d9 ef bf ae 8e a6 d1 d0 3e 2b d1 93
         ef f0 ab 9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7 9f 7f 1e 3f 02 03
         01 00 01 a3 1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06
         03 55 1d 0f 04 04 03 02 05 a0 30 0d 06 09 2a 86 48 86 f7 0d 01
         01 0b 05 00 03 81 81 00 85 aa d2 a0 e5 b9 27 6b 90 8c 65 f7 3a
         72 67 17 06 18 a5 4c 5f 8a 7b 33 7d 2d f7 a5 94 36 54 17 f2 ea
         e8 f8 a5 8c 8f 81 72 f9 31 9c f3 6b 7f d6 c5 5b 80 f2 1a 03 01
         51 56 72 60 96 fd 33 5e 5e 67 f2 db f1 02 70 2e 60 8c ca e6 be
         c1 fc 63 a4 2a 99 be 5c 3e b7 10 7c 3c 54 e9 b9 eb 2b d5 20 3b
         1c 3b 84 e0 a8 b2 f7 59 40 9b a3 ea c9 d9 1d 40 2d cc 0c c8 f8
         96 12 29 ac 91 87 b4 2b 4d e1 00 00

   {server}  construct a CertificateVerify handshake message:

      CertificateVerify (136 octets):  0f 00 00 84 08 04 00 80 33 ab 13
         d4 46 27 07 23 1b 5d ca e6 c8 19 0b 63 d1 da bc 74 f2 8c 39 53
         70 da 0b 07 e5 b8 30 66 d0 24 6a 31 ac d9 5d f4 75 bf d7 99 a4
         a7 0d 33 ad 93 d3 a3 17 a9 b2 c0 d2 37 a5 68 5b 21 9e 77 41 12
         e3 91 a2 47 60 7d 1a ef f1 bb d0 a3 9f 38 2e e1 a5 fe 88 ae 99
         ec 59 22 8e 64 97 e4 5d 48 ce 27 5a 6d 5e f4 0d 16 9f b6 f9 d3
         3b 05 2e d3 dc dd 6b 5a 48 ba af ff bc b2 90 12 84 15 bd 38

   {server}  calculate finished "tls13 finished":

      PRK (32 octets):  34 03 e7 81 e2 af 7b 65 08 da 28 57 4f 6e 95 a1
         ab f1 62 de 83 a9 79 27 c3 76 72 a4 a0 ce f8 a1
      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  e7 f8 bb 3e a4 b6 c3 0c 47 10 b3 d0 9c 33
         13 65 81 17 e7 0b 09 7e 85 03 68 e2 51 0c a5 63 1f 74

      finished (32 octets):  88 63 e6 bf b0 42 0a 92 7f a2 7f 34 33 6a
         70 ae 42 6e 96 8e 3e b8 84 94 5b 96 85 6d ba 39 76 d1

   {server}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 88 63 e6 bf b0 42 0a 92 7f a2
         7f 34 33 6a 70 ae 42 6e 96 8e 3e b8 84 94 5b 96 85 6d ba 39 76
         d1

   {server}  send handshake record:

      payload (645 octets):  08 00 00 18 00 16 00 0a 00 08 00 06 00 17
         00 18 00 1d 00 1c 00 02 40 01 00 00 00 00 0b 00 01 b9 00 00 01
         b5 00 01 b0 30 82 01 ac 30 82 01 15 a0 03 02 01 02 02 01 02 30
         0d 06 09 2a 86 48 86 f7 0d 01 01 0b 05 00 30 0e 31 0c 30 0a 06
         03 55 04 03 13 03 72 73 61 30 1e 17 0d 31 36 30 37 33 30 30 31
         32 33 35 39 5a 17 0d 32 36 30 37 33 30 30 31 32 33 35 39 5a 30
         0e 31 0c 30 0a 06 03 55 04 03 13 03 72 73 61 30 81 9f 30 0d 06
         09 2a 86 48 86 f7 0d 01 01 01 05 00 03 81 8d 00 30 81 89 02 81
         81 00 b4 bb 49 8f 82 79 30 3d 98 08 36 39 9b 36 c6 98 8c 0c 68
         de 55 e1 bd b8 26 d3 90 1a 24 61 ea fd 2d e4 9a 91 d0 15 ab bc
         9a 95 13 7a ce 6c 1a f1 9e aa 6a f9 8c 7c ed 43 12 09 98 e1 87
         a8 0e e0 cc b0 52 4b 1b 01 8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a
         5f da 43 08 46 74 80 30 53 0e f0 46 1c 8c a9 d9 ef bf ae 8e a6
         d1 d0 3e 2b d1 93 ef f0 ab 9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7
         9f 7f 1e 3f 02 03 01 00 01 a3 1a 30 18 30 09 06 03 55 1d 13 04
         02 30 00 30 0b 06 03 55 1d 0f 04 04 03 02 05 a0 30 0d 06 09 2a
         86 48 86 f7 0d 01 01 0b 05 00 03 81 81 00 85 aa d2 a0 e5 b9 27
         6b 90 8c 65 f7 3a 72 67 17 06 18 a5 4c 5f 8a 7b 33 7d 2d f7 a5
         94 36 54 17 f2 ea e8 f8 a5 8c 8f 81 72 f9 31 9c f3 6b 7f d6 c5
         5b 80 f2 1a 03 01 51 56 72 60 96 fd 33 5e 5e 67 f2 db f1 02 70
         2e 60 8c ca e6 be c1 fc 63 a4 2a 99 be 5c 3e b7 10 7c 3c 54 e9
         b9 eb 2b d5 20 3b 1c 3b 84 e0 a8 b2 f7 59 40 9b a3 ea c9 d9 1d
         40 2d cc 0c c8 f8 96 12 29 ac 91 87 b4 2b 4d e1 00 00 0f 00 00
         84 08 04 00 80 33 ab 13 d4 46 27 07 23 1b 5d ca e6 c8 19 0b 63
         d1 da bc 74 f2 8c 39 53 70 da 0b 07 e5 b8 30 66 d0 24 6a 31 ac
         d9 5d f4 75 bf d7 99 a4 a7 0d 33 ad 93 d3 a3 17 a9 b2 c0 d2 37
         a5 68 5b 21 9e 77 41 12 e3 91 a2 47 60 7d 1a ef f1 bb d0 a3 9f
         38 2e e1 a5 fe 88 ae 99 ec 59 22 8e 64 97 e4 5d 48 ce 27 5a 6d
         5e f4 0d 16 9f b6 f9 d3 3b 05 2e d3 dc dd 6b 5a 48 ba af ff bc 
         b2 90 12 84 15 bd 38 14 00 00 20 88 63 e6 bf b0 42 0a 92 7f a2
         7f 34 33 6a 70 ae 42 6e 96 8e 3e b8 84 94 5b 96 85 6d ba 39 76
         d1

      complete record (667 octets):  17 03 03 02 96 99 be e2 0b af 5b 7f
         c7 27 bf ab 62 23 92 8a 38 1e 6d 0c f9 c4 da 65 3f 9d 2a 7b 23
         f7 de 11 cc e8 42 d5 cf 75 63 17 63 45 0f fb 8b 0c c1 d2 38 e6
         58 af 7a 12 ad c8 62 43 11 4a b1 4a 1d a2 fa e4 26 21 ce 48 3f
         b6 24 2e ab fa ad 52 56 6b 02 b3 1d 2e dd ed ef eb 80 e6 6a 99
         00 d5 f9 73 b4 0c 4f df 74 71 9e cf 1b 68 d7 f9 c3 b6 ce b9 03
         ca 13 dd 1b b8 f8 18 7a e3 34 17 e1 d1 52 52 2c 58 22 a1 a0 3a
         d5 2c 83 8c 55 95 3d 61 02 22 87 4c ce 8e 17 90 b2 29 a2 aa 0b
         53 c8 d3 77 ee 72 01 82 95 1d c6 18 1d c5 d9 0b d1 f0 10 5e d1
         e8 4a a5 f7 59 57 c6 66 18 97 07 9e 5e a5 00 74 49 e3 19 7b dc
         7c 9b ee ed dd ea fd d8 44 af a5 c3 15 ec fe 65 e5 76 af e9 09
         81 28 80 62 0e c7 04 8b 42 d7 f5 c7 8d 76 f2 99 d6 d8 25 34 bd
         d8 f5 12 fe bc 0e d3 81 4a ca 47 0c d8 00 0d 3e 1c b9 96 2b 05
         2f bb 95 0d f6 83 a5 2c 2b a7 7e d3 71 3b 12 29 37 a6 e5 17 09
         64 e2 ab 79 69 dc d9 80 b3 db 9b 45 8d a7 60 31 24 d6 dc 00 5e
         4d 6e 04 b4 d0 c4 ba f3 27 5d b8 27 db ba 0a 6d b0 96 72 17 1f
         c0 57 b3 85 1d 7e 02 68 41 e2 97 8f bd 23 46 bb ef dd 03 76 bb
         11 08 fe 9a cc 92 18 9f 56 50 aa 5e 85 d8 e8 c7 b6 7a c5 10 db
         a0 03 d3 d7 e1 63 50 bb 66 d4 50 13 ef d4 4c 9b 60 7c 0d 31 8c
         4c 7d 1a 1f 5c bc 57 e2 06 11 80 4e 37 87 d7 b4 a4 b5 f0 8e d8
         fd 70 bd ae ad e0 22 60 b1 2a b8 42 ef 69 0b 4a 3e e7 91 1e 84
         1b 37 4e cd 5e bb bc 2a 54 d0 47 b6 00 33 6d d7 d0 c8 8b 4b c1
         0e 58 ee 6c b6 56 de 72 47 fa 20 d8 e9 1d eb 84 62 86 08 cf 80
         61 5b 62 e9 6c 14 91 c7 ac 37 55 eb 69 01 40 5d 34 74 fe 1a c7
         9d 10 6a 0c ee 56 c2 57 7f c8 84 80 f9 6c b6 b8 c6 81 b7 b6 8b
         53 c1 46 09 39 08 f3 50 88 81 75 bd fb 0b 1e 31 ad 61 e3 0b a0
         ad fe 6d 22 3a a0 3c 07 83 b5 00 1a 57 58 7c 32 8a 9a fc fc fb
         97 8d 1c d4 32 8f 7d 9d 60 53 0e 63 0b ef d9 6c 0c 81 6e e2 0b
         01 00 76 8a e2 a6 df 51 fc 68 f1 72 74 0a 79 af 11 39 8e e3 be
         12 52 49 1f a9 c6 93 47 9e 87 7f 94 ab 7c 5f 8c ad 48 02 03 e6
         ab 7b 87 dd 71 e8 a0 72 91 13 df 17 f5 ee e8 6c e1 08 d1 d7 20
         07 ec 1c d1 3c 85 a6 c1 49 62 1e 77 b7 d7 8d 80 5a 30 f0 be 03
         0c 31 5e 54

   {server}  derive secret "tls13 c ap traffic":

      PRK (32 octets):  11 31 54 5d 0b af 79 dd ce 9b 87 f0 69 45 78 1a
         57 dd 18 ef 37 8d cd 20 60 f8 f9 a5 69 02 7e d8

      hash (32 octets):  50 f6 3c bf 36 b0 dd 04 9e 7a 0b a2 7d 64 55 74
         5e a2 aa ac 54 bb 16 7f 99 50 b2 b7 ce 95 09 da 
      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 61 70 20 74 72
         61 66 66 69 63 20 50 f6 3c bf 36 b0 dd 04 9e 7a 0b a2 7d 64 55
         74 5e a2 aa ac 54 bb 16 7f 99 50 b2 b7 ce 95 09 da

      expanded (32 octets):  75 ec f4 b9 72 52 5a a0 dc d0 57 c9 94 4d
         4c d5 d8 26 71 d8 84 31 41 d7 dc 2a 4f f1 5a 21 dc 51

   {server}  derive secret "tls13 s ap traffic":

      PRK (32 octets):  11 31 54 5d 0b af 79 dd ce 9b 87 f0 69 45 78 1a
         57 dd 18 ef 37 8d cd 20 60 f8 f9 a5 69 02 7e d8

      hash (32 octets):  50 f6 3c bf 36 b0 dd 04 9e 7a 0b a2 7d 64 55 74
         5e a2 aa ac 54 bb 16 7f 99 50 b2 b7 ce 95 09 da

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 61 70 20 74 72
         61 66 66 69 63 20 50 f6 3c bf 36 b0 dd 04 9e 7a 0b a2 7d 64 55
         74 5e a2 aa ac 54 bb 16 7f 99 50 b2 b7 ce 95 09 da

      expanded (32 octets):  5c 74 f8 7d f0 42 25 db 0f 82 09 c9 de 64
         29 e4 94 35 fd ef a7 ca d6 18 64 87 4d 12 f3 1c fc 8d

   {server}  derive secret "tls13 exp master":

      PRK (32 octets):  11 31 54 5d 0b af 79 dd ce 9b 87 f0 69 45 78 1a
         57 dd 18 ef 37 8d cd 20 60 f8 f9 a5 69 02 7e d8

      hash (32 octets):  50 f6 3c bf 36 b0 dd 04 9e 7a 0b a2 7d 64 55 74
         5e a2 aa ac 54 bb 16 7f 99 50 b2 b7 ce 95 09 da

      info (52 octets):  00 20 10 74 6c 73 31 33 20 65 78 70 20 6d 61 73
         74 65 72 20 50 f6 3c bf 36 b0 dd 04 9e 7a 0b a2 7d 64 55 74 5e
         a2 aa ac 54 bb 16 7f 99 50 b2 b7 ce 95 09 da

      expanded (32 octets):  7c 06 d3 ae 10 6a 3a 37 4a ce 48 37 b3 98
         5c ac 67 78 0a 6e 2c 5c 04 b5 83 19 d5 84 df 09 d2 23

   {server}  derive write traffic keys for application data:

      PRK (32 octets):  5c 74 f8 7d f0 42 25 db 0f 82 09 c9 de 64 29 e4
         94 35 fd ef a7 ca d6 18 64 87 4d 12 f3 1c fc 8d

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  f2 7a 5d 97 bd 25 55 0c 48 23 b0 f3 e5
         d2 93 88

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00
      iv expanded (12 octets):  0d d6 31 f7 b7 1c bb c7 97 c3 5f e7

   {server}  derive read traffic keys for handshake data:

      PRK (32 octets):  15 8a a7 ab 88 55 07 35 82 b4 1d 67 4b 40 55 ca
         bc c5 34 72 8f 65 93 14 86 1b 4e 08 e2 01 15 66

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  2f 1f 91 86 63 d5 90 e7 42 11 49 a2 9d
         94 b0 b6

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  41 4d 54 85 23 5e 1a 68 87 93 bd 74

   {client}  extract secret "early" (same as server early secret)

   {client}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {client}  extract secret "handshake" (same as server handshake
      secret)

   {client}  derive secret "tls13 c hs traffic" (same as server)

   {client}  derive secret "tls13 s hs traffic" (same as server)

   {client}  derive secret for master "tls13 derived" (same as server)

   {client}  extract secret "master" (same as server master secret)

   {client}  derive read traffic keys for handshake data (same as server
      handshake data write traffic keys)

   {client}  calculate finished "tls13 finished" (same as server)
   {client}  derive secret "tls13 c ap traffic" (same as server)

   {client}  derive secret "tls13 s ap traffic" (same as server)

   {client}  derive secret "tls13 exp master" (same as server)

   {client}  derive write traffic keys for handshake data (same as
      server handshake data read traffic keys)

   {client}  derive read traffic keys for application data (same as
      server application data write traffic keys)

   {client}  calculate finished "tls13 finished":

      PRK (32 octets):  15 8a a7 ab 88 55 07 35 82 b4 1d 67 4b 40 55 ca
         bc c5 34 72 8f 65 93 14 86 1b 4e 08 e2 01 15 66

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  81 be 41 31 fb b9 b6 f4 47 14 50 84 6f 74
         fd 1e 68 c5 22 4b a7 c2 a8 67 7f 5c 53 ad 22 6f dc 13

      finished (32 octets):  23 f5 2f db 07 09 a5 5b d7 f7 9b 99 1f 25
         48 40 87 bc fd 4d 43 80 b1 23 26 a5 2a 28 b2 e3 68 e1

   {client}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 23 f5 2f db 07 09 a5 5b d7 f7
         9b 99 1f 25 48 40 87 bc fd 4d 43 80 b1 23 26 a5 2a 28 b2 e3 68
         e1

   {client}  send handshake record:

      payload (36 octets):  14 00 00 20 23 f5 2f db 07 09 a5 5b d7 f7 9b
         99 1f 25 48 40 87 bc fd 4d 43 80 b1 23 26 a5 2a 28 b2 e3 68 e1

      complete record (58 octets):  17 03 03 00 35 d7 4f 19 23 c6 62 fd
         34 13 7c 6f 50 2f 3d d2 b9 3d 95 1d 1b 3b c9 7e 42 af e2 3c 31
         ab ea 92 fe 91 b4 74 99 9e 85 e3 b7 91 ce 25 2f e8 c3 e9 f9 39
         a4 12 0c b2

   {client}  derive write traffic keys for application data:

      PRK (32 octets):  75 ec f4 b9 72 52 5a a0 dc d0 57 c9 94 4d 4c d5
         d8 26 71 d8 84 31 41 d7 dc 2a 4f f1 5a 21 dc 51
      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  a7 eb 2a 05 25 eb 43 31 d5 8f cb f9 f7
         ca 2e 9c

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  86 e8 be 22 7c 1b d2 b3 e3 9c b4 44

   {client}  derive secret "tls13 res master":

      PRK (32 octets):  11 31 54 5d 0b af 79 dd ce 9b 87 f0 69 45 78 1a
         57 dd 18 ef 37 8d cd 20 60 f8 f9 a5 69 02 7e d8

      hash (32 octets):  0e 8b 34 91 58 b8 55 fd cd 0c 11 db bc 4e 83 e4
         3c aa 6e 48 3c 6c 65 df 53 15 18 88 e5 01 65 f4

      info (52 octets):  00 20 10 74 6c 73 31 33 20 72 65 73 20 6d 61 73
         74 65 72 20 0e 8b 34 91 58 b8 55 fd cd 0c 11 db bc 4e 83 e4 3c
         aa 6e 48 3c 6c 65 df 53 15 18 88 e5 01 65 f4

      expanded (32 octets):  09 17 0c 6d 47 27 21 56 6f 9c f9 9b 08 69
         9d af f5 61 ec 8f b2 2d 5a 32 c3 f9 4c e0 09 b6 99 75

   {server}  calculate finished "tls13 finished" (same as client)

   {server}  derive read traffic keys for application data (same as
      client application data write traffic keys)

   {server}  derive secret "tls13 res master" (same as client)

   {client}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 2e a6 cd f7 49 19 60
         23 e2 b3 a4 94 91 69 55 36 42 60 47

   {server}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 51 9f c5 07 5c b0 88
         43 49 75 9f f9 ef 6f 01 1b b4 c6 f2





## 6. Client Authentication

   In this example, the server requests client authentication.  The
   client uses a certificate with an RSA key, the server uses an
   Elliptic Curve Digital Signature Algorithm (ECDSA) certificate with a
   P-256 key.  Note that private keys for the certificates used in this
   example are not shown.

   {client}  create an ephemeral x25519 key pair:

      private key (32 octets):  c0 40 b2 bb 8f 3a dd d2 0f d4 05 8c 54
         70 03 a3 c6 f9 c1 cd 91 5d 5e 53 5c 87 d8 d1 91 aa f0 71

      public key (32 octets):  08 9c c2 67 1f 73 8d 9a 67 1e 5b 2e 46 49
         81 d0 5b 76 e3 61 aa 22 ae a9 1f 1d 49 ca 10 a7 a3 62

   {client}  construct a ClientHello handshake message:

      ClientHello (192 octets):  01 00 00 bc 03 03 6a 47 22 36 32 8b 83
         af 40 38 6d 3a 3e 1f 1c e6 24 fa 4e d8 9a b8 65 a4 ff 0f 41 44
         ce 3a e2 33 00 00 06 13 01 13 03 13 02 01 00 00 8d 00 00 00 0b
         00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 33
         00 26 00 24 00 1d 00 20 08 9c c2 67 1f 73 8d 9a 67 1e 5b 2e 46
         49 81 d0 5b 76 e3 61 aa 22 ae a9 1f 1d 49 ca 10 a7 a3 62 00 2b
         00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03 02 03 08 04
         08 05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06 02 02 02 00
         2d 00 02 01 01 00 1c 00 02 40 01

   {client}  send handshake record:

      payload (192 octets):  01 00 00 bc 03 03 6a 47 22 36 32 8b 83 af
         40 38 6d 3a 3e 1f 1c e6 24 fa 4e d8 9a b8 65 a4 ff 0f 41 44 ce
         3a e2 33 00 00 06 13 01 13 03 13 02 01 00 00 8d 00 00 00 0b 00
         09 00 00 06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00 12
         00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 33 00
         26 00 24 00 1d 00 20 08 9c c2 67 1f 73 8d 9a 67 1e 5b 2e 46 49
         81 d0 5b 76 e3 61 aa 22 ae a9 1f 1d 49 ca 10 a7 a3 62 00 2b 00
         03 02 03 04 00 0d 00 20 00 1e 04 03 05 03 06 03 02 03 08 04 08
         05 08 06 04 01 05 01 06 01 02 01 04 02 05 02 06 02 02 02 00 2d
         00 02 01 01 00 1c 00 02 40 01

      complete record (197 octets):  16 03 01 00 c0 01 00 00 bc 03 03 6a
         47 22 36 32 8b 83 af 40 38 6d 3a 3e 1f 1c e6 24 fa 4e d8 9a b8
         65 a4 ff 0f 41 44 ce 3a e2 33 00 00 06 13 01 13 03 13 02 01 00
         00 8d 00 00 00 0b 00 09 00 00 06 73 65 72 76 65 72 ff 01 00 01
         00 00 0a 00 14 00 12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02
         01 03 01 04 00 33 00 26 00 24 00 1d 00 20 08 9c c2 67 1f 73 8d 
         9a 67 1e 5b 2e 46 49 81 d0 5b 76 e3 61 aa 22 ae a9 1f 1d 49 ca
         10 a7 a3 62 00 2b 00 03 02 03 04 00 0d 00 20 00 1e 04 03 05 03
         06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02 05
         02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01

   {server}  extract secret "early":

      salt:  0 (all zero octets)

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c
         e2 10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

   {server}  create an ephemeral x25519 key pair:

      private key (32 octets):  73 82 a5 ad 1c dd 20 56 ae 18 cc 70 8b
         d0 07 d9 81 30 db e2 cd 4d 9e ad 9b 96 95 2b ec bb 08 88

      public key (32 octets):  6c 2e 50 e8 65 91 9a 6b 5a 12 df af 91 8f
         92 b4 42 56 7b 0f 89 bc 54 47 8c 69 21 36 66 58 f0 62

   {server}  construct a ServerHello handshake message:

      ServerHello (90 octets):  02 00 00 56 03 03 3b 50 fd f1 c3 d5 72
         e4 0e 68 95 3e 7f ff 4e 27 58 45 9c 59 af a0 58 2c 0e a0 32 87
         42 55 fe 6e 00 13 01 00 00 2e 00 33 00 24 00 1d 00 20 6c 2e 50
         e8 65 91 9a 6b 5a 12 df af 91 8f 92 b4 42 56 7b 0f 89 bc 54 47
         8c 69 21 36 66 58 f0 62 00 2b 00 02 03 04

   {server}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba 
   {server}  extract secret "handshake":

      salt (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba b6 97
         16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

      IKM (32 octets):  7d c1 14 f6 47 5d fa 79 77 be 73 6e f7 cb eb c4
         8c 70 32 9e 8e 9a 74 b4 d7 03 3c 43 f9 59 7d 4f

      secret (32 octets):  d9 95 24 36 74 fb 64 00 d7 d3 7b c0 e9 86 1b
         db d9 ed 09 56 01 dc f2 99 48 74 f2 80 3d e2 2e 39

   {server}  derive secret "tls13 c hs traffic":

      PRK (32 octets):  d9 95 24 36 74 fb 64 00 d7 d3 7b c0 e9 86 1b db
         d9 ed 09 56 01 dc f2 99 48 74 f2 80 3d e2 2e 39

      hash (32 octets):  88 eb c0 42 bd 0d 5a 64 3b 22 fc a7 a4 7d ef d4
         00 7d fe 18 49 49 a6 26 1c 59 6c 4e 00 2a 74 a2

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 68 73 20 74 72
         61 66 66 69 63 20 88 eb c0 42 bd 0d 5a 64 3b 22 fc a7 a4 7d ef
         d4 00 7d fe 18 49 49 a6 26 1c 59 6c 4e 00 2a 74 a2

      expanded (32 octets):  ce c7 a3 0c 68 72 07 0f 22 a7 ee b0 65 76
         8d b6 7c 45 e2 95 33 db 87 99 08 ce 6d c6 6f 59 11 de

   {server}  derive secret "tls13 s hs traffic":

      PRK (32 octets):  d9 95 24 36 74 fb 64 00 d7 d3 7b c0 e9 86 1b db
         d9 ed 09 56 01 dc f2 99 48 74 f2 80 3d e2 2e 39

      hash (32 octets):  88 eb c0 42 bd 0d 5a 64 3b 22 fc a7 a4 7d ef d4
         00 7d fe 18 49 49 a6 26 1c 59 6c 4e 00 2a 74 a2

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 68 73 20 74 72
         61 66 66 69 63 20 88 eb c0 42 bd 0d 5a 64 3b 22 fc a7 a4 7d ef
         d4 00 7d fe 18 49 49 a6 26 1c 59 6c 4e 00 2a 74 a2

      expanded (32 octets):  8b 02 d3 c0 04 42 a2 72 2c 40 98 eb e8 67
         5b 23 e8 01 51 0f 0d 7e d7 78 d8 eb 0b 8f 42 a1 9a 5e

   {server}  derive secret for master "tls13 derived":

      PRK (32 octets):  d9 95 24 36 74 fb 64 00 d7 d3 7b c0 e9 86 1b db
         d9 ed 09 56 01 dc f2 99 48 74 f2 80 3d e2 2e 39

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55
      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  74 57 55 26 b0 7c 81 a9 c1 b1 7e 6b 34 e0
         e6 d0 84 74 7a 61 f3 96 f5 97 eb b9 2c 07 36 ec 60 e8

   {server}  extract secret "master":

      salt (32 octets):  74 57 55 26 b0 7c 81 a9 c1 b1 7e 6b 34 e0 e6 d0
         84 74 7a 61 f3 96 f5 97 eb b9 2c 07 36 ec 60 e8

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  57 c1 5d 7b 9d 44 1b 3d 40 a9 c6 ea 8a 3d 73
         0e 07 b3 a1 ea 7a 33 39 ed 70 70 b9 a7 4a 3f 4f 28

   {server}  send handshake record:

      payload (90 octets):  02 00 00 56 03 03 3b 50 fd f1 c3 d5 72 e4 0e
         68 95 3e 7f ff 4e 27 58 45 9c 59 af a0 58 2c 0e a0 32 87 42 55
         fe 6e 00 13 01 00 00 2e 00 33 00 24 00 1d 00 20 6c 2e 50 e8 65
         91 9a 6b 5a 12 df af 91 8f 92 b4 42 56 7b 0f 89 bc 54 47 8c 69
         21 36 66 58 f0 62 00 2b 00 02 03 04

      complete record (95 octets):  16 03 03 00 5a 02 00 00 56 03 03 3b
         50 fd f1 c3 d5 72 e4 0e 68 95 3e 7f ff 4e 27 58 45 9c 59 af a0
         58 2c 0e a0 32 87 42 55 fe 6e 00 13 01 00 00 2e 00 33 00 24 00
         1d 00 20 6c 2e 50 e8 65 91 9a 6b 5a 12 df af 91 8f 92 b4 42 56
         7b 0f 89 bc 54 47 8c 69 21 36 66 58 f0 62 00 2b 00 02 03 04

   {server}  derive write traffic keys for handshake data:

      PRK (32 octets):  8b 02 d3 c0 04 42 a2 72 2c 40 98 eb e8 67 5b 23
         e8 01 51 0f 0d 7e d7 78 d8 eb 0b 8f 42 a1 9a 5e

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  6c b6 e6 06 19 d8 c7 35 5c 5d 4c 4b c2
         be 90 d5

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  64 f2 39 53 0c 3b 88 8f de 85 e0 be 
   {server}  construct an EncryptedExtensions handshake message:

      EncryptedExtensions (40 octets):  08 00 00 24 00 22 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c
         00 02 40 01 00 00 00 00

   {server}  construct a CertificateRequest handshake message:

      CertificateRequest (43 octets):  0d 00 00 27 00 00 24 00 0d 00 20
         00 1e 04 03 05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06
         01 02 01 04 02 05 02 06 02 02 02

   {server}  construct a Certificate handshake message:

      Certificate (319 octets):  0b 00 01 3b 00 00 01 37 00 01 32 30 82
         01 2e 30 81 d5 a0 03 02 01 02 02 01 07 30 0a 06 08 2a 86 48 ce
         3d 04 03 02 30 13 31 11 30 0f 06 03 55 04 03 13 08 65 63 64 73
         61 32 35 36 30 1e 17 0d 31 36 30 37 33 30 30 31 32 34 30 30 5a
         17 0d 32 36 30 37 33 30 30 31 32 34 30 30 5a 30 13 31 11 30 0f
         06 03 55 04 03 13 08 65 63 64 73 61 32 35 36 30 59 30 13 06 07
         2a 86 48 ce 3d 02 01 06 08 2a 86 48 ce 3d 03 01 07 03 42 00 04
         08 d5 30 16 15 75 f4 cf e7 f1 54 ee 34 48 18 00 86 00 1e 88 43
         1a 79 ee 62 ee 6e 2f 83 ef 38 ba 61 e9 fb 37 f3 4e 00 7a 7d f4
         d2 f5 b5 6d 1f 04 ec e4 5d 62 1f 46 84 06 f5 c3 a1 51 58 94 8d
         d0 a3 1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06 03 55
         1d 0f 04 04 03 02 07 80 30 0a 06 08 2a 86 48 ce 3d 04 03 02 03
         48 00 30 45 02 21 00 df 30 fd 45 07 f5 ed d2 2c 1a 6f f8 6d b4
         79 ca 69 3f ee ca 3b 71 b3 f9 ef 55 6b 29 37 c0 59 4d 02 20 62
         e2 a4 72 50 d3 20 fe a8 3c 7e 2d cb 5b 76 a5 0e 02 00 c0 9a db
         d1 3f ee 94 6e 51 3e 01 1d 11 00 00

   {server}  construct a CertificateVerify handshake message:

      CertificateVerify (79 octets):  0f 00 00 4b 04 03 00 47 30 45 02
         21 00 d7 a4 d3 4b d5 4f 55 fe e1 a8 96 25 67 8c 3d d5 e5 f6 0d
         ac 73 ec 94 0c 5c 7b 93 04 a0 20 84 a9 02 20 28 9f 59 5e d4 88
         b9 ac 68 9a 3d 19 2b 1a 8b b3 8f 34 af 78 74 c0 59 c9 80 6a 1f
         38 26 93 53 e8

   {server}  calculate finished "tls13 finished":

      PRK (32 octets):  8b 02 d3 c0 04 42 a2 72 2c 40 98 eb e8 67 5b 23
         e8 01 51 0f 0d 7e d7 78 d8 eb 0b 8f 42 a1 9a 5e

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00
      expanded (32 octets):  4e 79 5c de 23 9d 5e 19 0e ae 44 1b 9e 71
         6e eb 13 85 49 05 8c db 76 fa 9a ee af 54 8a ef 56 3e

      finished (32 octets):  93 b7 0c df 47 81 98 5b 96 34 5c aa c7 01
         b4 e7 50 d3 04 2d f1 a6 89 d8 fa ca 81 22 51 11 3c 11

   {server}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 93 b7 0c df 47 81 98 5b 96 34
         5c aa c7 01 b4 e7 50 d3 04 2d f1 a6 89 d8 fa ca 81 22 51 11 3c
         11

   {server}  send handshake record:

      payload (517 octets):  08 00 00 24 00 22 00 0a 00 14 00 12 00 1d
         00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c 00 02 40
         01 00 00 00 00 0d 00 00 27 00 00 24 00 0d 00 20 00 1e 04 03 05
         03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02 01 04 02
         05 02 06 02 02 02 0b 00 01 3b 00 00 01 37 00 01 32 30 82 01 2e
         30 81 d5 a0 03 02 01 02 02 01 07 30 0a 06 08 2a 86 48 ce 3d 04
         03 02 30 13 31 11 30 0f 06 03 55 04 03 13 08 65 63 64 73 61 32
         35 36 30 1e 17 0d 31 36 30 37 33 30 30 31 32 34 30 30 5a 17 0d
         32 36 30 37 33 30 30 31 32 34 30 30 5a 30 13 31 11 30 0f 06 03
         55 04 03 13 08 65 63 64 73 61 32 35 36 30 59 30 13 06 07 2a 86
         48 ce 3d 02 01 06 08 2a 86 48 ce 3d 03 01 07 03 42 00 04 08 d5
         30 16 15 75 f4 cf e7 f1 54 ee 34 48 18 00 86 00 1e 88 43 1a 79
         ee 62 ee 6e 2f 83 ef 38 ba 61 e9 fb 37 f3 4e 00 7a 7d f4 d2 f5
         b5 6d 1f 04 ec e4 5d 62 1f 46 84 06 f5 c3 a1 51 58 94 8d d0 a3
         1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06 03 55 1d 0f
         04 04 03 02 07 80 30 0a 06 08 2a 86 48 ce 3d 04 03 02 03 48 00
         30 45 02 21 00 df 30 fd 45 07 f5 ed d2 2c 1a 6f f8 6d b4 79 ca
         69 3f ee ca 3b 71 b3 f9 ef 55 6b 29 37 c0 59 4d 02 20 62 e2 a4
         72 50 d3 20 fe a8 3c 7e 2d cb 5b 76 a5 0e 02 00 c0 9a db d1 3f
         ee 94 6e 51 3e 01 1d 11 00 00 0f 00 00 4b 04 03 00 47 30 45 02
         21 00 d7 a4 d3 4b d5 4f 55 fe e1 a8 96 25 67 8c 3d d5 e5 f6 0d
         ac 73 ec 94 0c 5c 7b 93 04 a0 20 84 a9 02 20 28 9f 59 5e d4 88
         b9 ac 68 9a 3d 19 2b 1a 8b b3 8f 34 af 78 74 c0 59 c9 80 6a 1f
         38 26 93 53 e8 14 00 00 20 93 b7 0c df 47 81 98 5b 96 34 5c aa
         c7 01 b4 e7 50 d3 04 2d f1 a6 89 d8 fa ca 81 22 51 11 3c 11

      complete record (539 octets):  17 03 03 02 16 6d 0a 7a c0 79 b3 2a
         94 aa 68 c4 e2 89 3e 8b d0 d3 c1 85 f5 49 c2 36 fb bc e3 d6 47
         f0 8f 3c 94 a2 bf 42 4d 87 08 88 36 05 ad 89 55 f9 77 18 b0 21
         3d ea d1 3d fb 23 eb b8 38 1d a5 82 75 66 12 bc b5 a5 d4 08 47
         71 9f be 9f 17 9b fa e6 56 f3 ec fd 59 a4 c0 d3 51 32 ce 41 8a
         7e 46 f6 b6 a6 06 22 f8 a6 c0 6b 28 d8 33 60 16 35 63 be 9c 37
         f9 7e b9 02 32 69 24 a7 2b 3e d8 c8 38 12 77 d1 58 1c ab 9c 37
         15 ac 24 01 39 84 67 ad 7e bf ab 3d 0c 34 19 e7 50 10 4f 7d 62
         c5 02 79 01 f2 e4 cd 4c a5 b8 07 1e b0 3d 3c 73 2d 83 21 50 66
         df c4 d2 91 d4 c1 ff 3b 8d 7e 42 98 f6 77 d4 d5 1d ea 11 68 d8
         f1 6c b2 7b a4 02 66 31 3a 1f ed f9 e2 3c c7 7f 76 54 50 f9 e9
         6f 05 d0 8f 3d a2 45 b1 4d 49 46 f0 7e c8 1e ed 6d 56 f2 6b d5
         74 f0 b7 f7 c7 04 70 37 c1 6f ce 3b 23 75 4e 66 2f ad 73 e2 b7
         21 3f 6a f2 96 76 9c 99 a1 d3 8e 62 32 e0 ec 8d c4 f8 4d 6a a6
         f7 de 38 87 be 00 57 86 2f 90 18 e0 ab 39 67 05 aa 40 90 ab 5f
         2d ff 63 25 a5 57 e7 32 0d 4e ff d4 6b b4 f9 97 d1 63 20 7c ce
         66 65 29 4a a4 46 55 41 e3 fe 37 ee 73 50 65 9e a5 50 d6 dc b6
         af 3c 51 88 52 c7 a1 4c 3c c1 5b c3 2b 32 73 bd f1 75 1d a1 84
         20 31 35 b1 17 d3 00 20 4f b1 2d 58 ca 9a c3 4b 68 ec a2 70 30
         83 2f 7a 4b 46 d2 a5 57 57 f6 3f e8 f6 e8 5a c4 74 69 e6 19 8d
         a8 8a 64 58 6b f2 3c 69 59 0d e8 22 26 3b e7 5f d8 36 84 72 40
         c4 8f 8c 14 5c d6 bd 69 89 62 e7 ed c2 34 eb e5 92 31 35 1e ef
         8d 76 52 cf 3b 08 ab 3a f6 e5 ec 74 c5 8a 8d a3 4b 39 f9 b0 d6
         c4 27 9a 9a 1f 82 07 17 29 e7 05 9d d7 f7 b9 5b 94 33 c4 68 4c
         e1 89 1a 6d 33 43 2d 52 ed db 0b 8c ee 91 81 d4 03 ec cc 12 99
         1f 1a d4 aa 62 c3 60 49 71 3a 7b b1 35 fd da 66 61 a0 5a 93 f8
         c1 6f

   {server}  derive secret "tls13 c ap traffic":

      PRK (32 octets):  57 c1 5d 7b 9d 44 1b 3d 40 a9 c6 ea 8a 3d 73 0e
         07 b3 a1 ea 7a 33 39 ed 70 70 b9 a7 4a 3f 4f 28

      hash (32 octets):  51 77 a2 9a f5 a1 7f 9b 49 33 e4 31 85 1d 12 83
         45 36 6c 17 20 d3 8f 8f 04 65 ee ea e6 74 03 72

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 61 70 20 74 72
         61 66 66 69 63 20 51 77 a2 9a f5 a1 7f 9b 49 33 e4 31 85 1d 12
         83 45 36 6c 17 20 d3 8f 8f 04 65 ee ea e6 74 03 72

      expanded (32 octets):  73 c2 e8 90 fa 8d 06 72 58 d6 d5 0f a9 2f
         e4 56 b0 98 cf 00 d9 72 7e ed 91 e8 89 2e f4 e6 f8 60

   {server}  derive secret "tls13 s ap traffic":

      PRK (32 octets):  57 c1 5d 7b 9d 44 1b 3d 40 a9 c6 ea 8a 3d 73 0e
         07 b3 a1 ea 7a 33 39 ed 70 70 b9 a7 4a 3f 4f 28

      hash (32 octets):  51 77 a2 9a f5 a1 7f 9b 49 33 e4 31 85 1d 12 83
         45 36 6c 17 20 d3 8f 8f 04 65 ee ea e6 74 03 72

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 61 70 20 74 72
         61 66 66 69 63 20 51 77 a2 9a f5 a1 7f 9b 49 33 e4 31 85 1d 12
         83 45 36 6c 17 20 d3 8f 8f 04 65 ee ea e6 74 03 72
      expanded (32 octets):  c4 9a 91 fa f5 7f 8c 54 5d 50 48 a0 15 bf
         84 9f f6 39 42 e4 a7 ed cd 31 9f 8b 43 8a 97 c5 2e 21

   {server}  derive secret "tls13 exp master":

      PRK (32 octets):  57 c1 5d 7b 9d 44 1b 3d 40 a9 c6 ea 8a 3d 73 0e
         07 b3 a1 ea 7a 33 39 ed 70 70 b9 a7 4a 3f 4f 28

      hash (32 octets):  51 77 a2 9a f5 a1 7f 9b 49 33 e4 31 85 1d 12 83
         45 36 6c 17 20 d3 8f 8f 04 65 ee ea e6 74 03 72

      info (52 octets):  00 20 10 74 6c 73 31 33 20 65 78 70 20 6d 61 73
         74 65 72 20 51 77 a2 9a f5 a1 7f 9b 49 33 e4 31 85 1d 12 83 45
         36 6c 17 20 d3 8f 8f 04 65 ee ea e6 74 03 72

      expanded (32 octets):  05 2e 39 79 5e 5f 2b e6 e4 e0 97 4c fd d8
         6c 6a 7a fe 3e 57 e5 58 98 10 a3 cc cf 64 29 58 be b2

   {server}  derive write traffic keys for application data:

      PRK (32 octets):  c4 9a 91 fa f5 7f 8c 54 5d 50 48 a0 15 bf 84 9f
         f6 39 42 e4 a7 ed cd 31 9f 8b 43 8a 97 c5 2e 21

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  88 b3 12 3d de ca df 8c 1b a2 98 e2 c1
         81 76 b0

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  4e 09 78 51 3f 9d e8 32 7c 08 e4 f3

   {server}  derive read traffic keys for handshake data:

      PRK (32 octets):  ce c7 a3 0c 68 72 07 0f 22 a7 ee b0 65 76 8d b6
         7c 45 e2 95 33 db 87 99 08 ce 6d c6 6f 59 11 de

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  91 69 48 f7 28 d9 82 3f a4 1a 00 4d 08
         3f 21 7f

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  64 15 3d 79 ba c9 ea 10 ca 5a 0a 88

   {client}  extract secret "early" (same as server early secret)
   {client}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {client}  extract secret "handshake" (same as server handshake
      secret)

   {client}  derive secret "tls13 c hs traffic" (same as server)

   {client}  derive secret "tls13 s hs traffic" (same as server)

   {client}  derive secret for master "tls13 derived" (same as server)

   {client}  extract secret "master" (same as server master secret)

   {client}  derive read traffic keys for handshake data (same as server
      handshake data write traffic keys)

   {client}  calculate finished "tls13 finished" (same as server)

   {client}  derive secret "tls13 c ap traffic" (same as server)

   {client}  derive secret "tls13 s ap traffic" (same as server)

   {client}  derive secret "tls13 exp master" (same as server)

   {client}  derive write traffic keys for handshake data (same as
      server handshake data read traffic keys)

   {client}  derive read traffic keys for application data (same as
      server application data write traffic keys)

   {client}  construct a Certificate handshake message:

      Certificate (451 octets):  0b 00 01 bf 00 00 01 bb 00 01 b6 30 82
         01 b2 30 82 01 1b a0 03 02 01 02 02 01 01 30 0d 06 09 2a 86 48
         86 f7 0d 01 01 0b 05 00 30 11 31 0f 30 0d 06 03 55 04 03 13 06
         63 6c 69 65 6e 74 30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35
         39 5a 17 0d 32 36 30 37 33 30 30 31 32 33 35 39 5a 30 11 31 0f
         30 0d 06 03 55 04 03 13 06 63 6c 69 65 6e 74 30 81 9f 30 0d 06
         09 2a 86 48 86 f7 0d 01 01 01 05 00 03 81 8d 00 30 81 89 02 81
         81 00 c3 81 75 e0 04 a6 8d 09 3f 82 3b 9c 37 9d 20 1f bc 0b b7
         a1 c7 91 90 5e 3f bf 76 84 7e 44 e7 51 eb bc d3 60 bd 94 5c 81
         e5 22 2b cc 88 46 d3 a8 a0 f9 3e 9b f5 be ba bd 92 ed f1 de 1f
         f1 90 21 70 3e 7a b6 c0 90 15 13 f9 7e 39 b1 11 f0 9c 93 48 97
         1c 7b 21 19 84 a7 54 cd 45 fe 09 5a f0 ea 42 36 82 9b cc f7 a7
         fe 9b 28 88 e7 8a b4 77 69 0a 5b 9e 1c cb e9 1c 6a 4a 0f 97 a7
         e0 28 42 01 02 03 01 00 01 a3 1a 30 18 30 09 06 03 55 1d 13 04
         02 30 00 30 0b 06 03 55 1d 0f 04 04 03 02 07 80 30 0d 06 09 2a
         86 48 86 f7 0d 01 01 0b 05 00 03 81 81 00 1a 7a 5a 01 85 32 b0
         22 af 07 67 d4 86 16 0c ff 2d 16 7a 19 15 d2 38 35 b5 45 94 91
         6d c6 80 be 5d 2e 62 60 76 c5 d5 27 22 eb cc 77 5d 7d 99 f9 80
         be 2f c9 4d 34 ac f6 cc 00 ba 90 cb cf b0 60 8a a1 e7 e3 97 1e
         f0 c0 7a 41 d4 7a d8 34 5d 1f 81 fe 41 8a 1c f4 10 54 42 9f d2
         17 bd 77 7d c1 cf 08 f0 5d f9 07 99 c6 59 36 1e 0f 1a 8e e4 ac
         0f 78 97 42 0b db c8 23 da 80 a2 f2 ba 23 08 1c 00 00

   {client}  construct a CertificateVerify handshake message:

      CertificateVerify (136 octets):  0f 00 00 84 08 04 00 80 18 6b 22
         23 b5 03 a7 59 c3 5d ba 0e 97 21 b4 b5 79 13 8d 5f 0f 5e 6e c7
         fe aa f2 7f 3a d7 f3 86 c2 c7 bd 7c b2 be 52 fb f5 ed 83 93 f4
         06 ee 79 36 96 92 ec 7a c6 95 65 1d 85 82 19 e6 72 a8 eb 7b 2a
         67 7b 64 0b 46 ab 63 0e dc 5f 3f 2f 82 72 b9 c0 d9 06 f8 1f 84
         dd c5 b8 c7 bc f9 55 c7 8a 3c f9 9e 50 16 f7 3e 04 eb 7d fc b2
         88 33 f1 3e 8f 75 ec 2f f3 58 1e 2f 09 8a d4 15 7f d6 d6 ad

   {client}  calculate finished "tls13 finished":

      PRK (32 octets):  ce c7 a3 0c 68 72 07 0f 22 a7 ee b0 65 76 8d b6
         7c 45 e2 95 33 db 87 99 08 ce 6d c6 6f 59 11 de

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  4f dd d7 6b bc b8 e3 0c 72 61 b1 db 40 1b
         b1 36 ed 39 bc e6 a4 81 5a 21 24 47 6e 27 e6 cb cb f6

      finished (32 octets):  9a fe 2b a2 f6 3a 09 d2 29 d8 a4 29 e5 b3
         7f fd 9f cc 73 bd b5 91 1b 82 42 59 72 aa 28 92 44 0f 
   {client}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 9a fe 2b a2 f6 3a 09 d2 29 d8
         a4 29 e5 b3 7f fd 9f cc 73 bd b5 91 1b 82 42 59 72 aa 28 92 44
         0f

   {client}  send handshake record:

      payload (623 octets):  0b 00 01 bf 00 00 01 bb 00 01 b6 30 82 01
         b2 30 82 01 1b a0 03 02 01 02 02 01 01 30 0d 06 09 2a 86 48 86
         f7 0d 01 01 0b 05 00 30 11 31 0f 30 0d 06 03 55 04 03 13 06 63
         6c 69 65 6e 74 30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35 39
         5a 17 0d 32 36 30 37 33 30 30 31 32 33 35 39 5a 30 11 31 0f 30
         0d 06 03 55 04 03 13 06 63 6c 69 65 6e 74 30 81 9f 30 0d 06 09
         2a 86 48 86 f7 0d 01 01 01 05 00 03 81 8d 00 30 81 89 02 81 81
         00 c3 81 75 e0 04 a6 8d 09 3f 82 3b 9c 37 9d 20 1f bc 0b b7 a1
         c7 91 90 5e 3f bf 76 84 7e 44 e7 51 eb bc d3 60 bd 94 5c 81 e5
         22 2b cc 88 46 d3 a8 a0 f9 3e 9b f5 be ba bd 92 ed f1 de 1f f1
         90 21 70 3e 7a b6 c0 90 15 13 f9 7e 39 b1 11 f0 9c 93 48 97 1c
         7b 21 19 84 a7 54 cd 45 fe 09 5a f0 ea 42 36 82 9b cc f7 a7 fe
         9b 28 88 e7 8a b4 77 69 0a 5b 9e 1c cb e9 1c 6a 4a 0f 97 a7 e0
         28 42 01 02 03 01 00 01 a3 1a 30 18 30 09 06 03 55 1d 13 04 02
         30 00 30 0b 06 03 55 1d 0f 04 04 03 02 07 80 30 0d 06 09 2a 86
         48 86 f7 0d 01 01 0b 05 00 03 81 81 00 1a 7a 5a 01 85 32 b0 22
         af 07 67 d4 86 16 0c ff 2d 16 7a 19 15 d2 38 35 b5 45 94 91 6d
         c6 80 be 5d 2e 62 60 76 c5 d5 27 22 eb cc 77 5d 7d 99 f9 80 be
         2f c9 4d 34 ac f6 cc 00 ba 90 cb cf b0 60 8a a1 e7 e3 97 1e f0
         c0 7a 41 d4 7a d8 34 5d 1f 81 fe 41 8a 1c f4 10 54 42 9f d2 17
         bd 77 7d c1 cf 08 f0 5d f9 07 99 c6 59 36 1e 0f 1a 8e e4 ac 0f
         78 97 42 0b db c8 23 da 80 a2 f2 ba 23 08 1c 00 00 0f 00 00 84
         08 04 00 80 18 6b 22 23 b5 03 a7 59 c3 5d ba 0e 97 21 b4 b5 79
         13 8d 5f 0f 5e 6e c7 fe aa f2 7f 3a d7 f3 86 c2 c7 bd 7c b2 be
         52 fb f5 ed 83 93 f4 06 ee 79 36 96 92 ec 7a c6 95 65 1d 85 82
         19 e6 72 a8 eb 7b 2a 67 7b 64 0b 46 ab 63 0e dc 5f 3f 2f 82 72
         b9 c0 d9 06 f8 1f 84 dd c5 b8 c7 bc f9 55 c7 8a 3c f9 9e 50 16
         f7 3e 04 eb 7d fc b2 88 33 f1 3e 8f 75 ec 2f f3 58 1e 2f 09 8a
         d4 15 7f d6 d6 ad 14 00 00 20 9a fe 2b a2 f6 3a 09 d2 29 d8 a4
         29 e5 b3 7f fd 9f cc 73 bd b5 91 1b 82 42 59 72 aa 28 92 44 0f

      complete record (645 octets):  17 03 03 02 80 b4 6a 63 93 4e 67 38
         41 ab af 26 74 03 bc 67 7f 6b 6d 2a 1e 2f 12 bb 5f 62 68 3b fe
         36 a8 26 73 f0 6d 62 87 dd d6 09 bc f2 f5 fd 32 25 92 3d 24 af
         3c 76 68 2c 18 0e e5 71 a1 7c a4 bf be 2f 51 0d c9 a0 e1 fc a5
         cf f2 ce e8 7d 11 cb 53 1a 6e f9 0b f5 30 9a 6b 63 bb bc 0b 88
         ea 45 10 3a 43 04 09 15 43 85 9f a1 1e c0 32 ed 87 34 44 cd 51
         85 ea d5 f6 a7 64 20 f0 f0 28 6a ce f8 02 c8 e4 78 8c 23 27 5f
         1b 06 da 60 0f 4a 7d ec d0 bc 59 d7 be f1 0e 64 9a e3 26 90 39
         7f c3 d4 ed 6f 30 f8 01 d8 cd 56 9b 71 ad 4f a0 5e a7 cf 2a c2
         df a1 50 d2 20 50 5d 40 11 b3 4d 09 d5 38 53 eb a6 1a 10 1e 4f
         8d ca 47 d8 17 1a 88 4b 19 25 9a 3d d4 8c 5a c1 41 98 3e dc 77
         81 4d 25 e7 f6 6b bb db 90 96 83 92 66 e0 65 61 82 8e cf b2 7e
         af d4 e9 e8 1a 0b 96 e3 bf a4 2d ae 5a d8 03 59 b9 a6 66 14 02
         c3 a2 10 41 77 03 01 06 db d8 f6 5b b6 a0 15 9d 51 2e b1 3a f2
         2a 25 9f 31 3b d5 8c 2e 21 fe 05 3d 57 f2 a9 62 b0 a4 ea 68 2c
         96 f7 0b 79 b5 60 13 61 92 82 3b 27 be 6a 2f b7 b1 c7 51 cc c0
         e3 30 36 15 54 14 85 b7 b3 07 b4 23 33 2c 11 ef a8 0b 72 f9 b8
         0a 53 e5 3f 7b b3 8a 3a f4 c5 9f 80 08 ba d0 54 4e 56 14 e6 88
         ff 57 bc cd 69 35 f8 1f 44 7f 42 0c 1c 1b f4 05 88 18 e9 0b f5
         dc 71 6c ca e4 25 24 85 6d f8 25 0b cd bd 7a f6 5f 82 dd 53 06
         1d 02 4f 6d 2f f5 c1 1e 37 92 a9 a7 0e 0e e2 a3 c2 0a 1b 96 8a
         c3 91 f8 f9 28 31 13 5d 25 24 2a da 2f e2 41 c2 65 3e c9 96 33
         9d fa 12 df ae 7a 33 73 df 88 b0 7c a2 7a ef 6d c2 66 a2 5f 13
         f7 5c 76 03 9c 1f 46 fd 7a 53 ae 63 99 c9 99 f4 b2 ae e1 8e 48
         0d 6d 12 bf ae 22 6b bd c9 2a 6a d5 0b 4d 3b ac 7a bc 3b 36 51
         eb 5b e5 6f 33 bf 41 12 7b 3c a8 86 dc 71 4a 50 d1 49 03 57 bd
         40 d9 fd 6b e4 22 09 a4 dd b9 eb b2 98 7e 29 f1 20 f0 58 14 61
         4d 2c 79 32 00 15 b4 61 fe 73 24 44 76 70 a1 af 5f 65 ca ed 15
         b4 74 ab 7f aa 49 50 16 ad f8 08 e5 3b 94 ef 54 af bb 0e 0a 3a
         27 32 ab 59 7f 7d 59 23 c7 73 86 aa 51 24 73 1f 8c c7 3e 70 3b
         34 1c 17 5a 45 49 39 a7 7a b6 43 13 c1 5c f3 fe 03 c4 f3 38 42
         56 49 76

   {client}  derive write traffic keys for application data:

      PRK (32 octets):  73 c2 e8 90 fa 8d 06 72 58 d6 d5 0f a9 2f e4 56
         b0 98 cf 00 d9 72 7e ed 91 e8 89 2e f4 e6 f8 60

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  cd c0 9c 80 6a a8 f8 6d fc d5 1e fc 44
         a0 c0 39

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  6e f8 52 e7 8b 46 d9 13 66 8e 53 e7

   {client}  derive secret "tls13 res master":

      PRK (32 octets):  57 c1 5d 7b 9d 44 1b 3d 40 a9 c6 ea 8a 3d 73 0e
         07 b3 a1 ea 7a 33 39 ed 70 70 b9 a7 4a 3f 4f 28

      hash (32 octets):  39 1d 00 4b d8 4c 83 1b 15 82 44 44 14 b4 dc 80
         64 01 0e cc 76 f3 7f 88 bf eb 1e 88 fe 13 5c 25
      info (52 octets):  00 20 10 74 6c 73 31 33 20 72 65 73 20 6d 61 73
         74 65 72 20 39 1d 00 4b d8 4c 83 1b 15 82 44 44 14 b4 dc 80 64
         01 0e cc 76 f3 7f 88 bf eb 1e 88 fe 13 5c 25

      expanded (32 octets):  10 06 dc cb f4 0e b4 eb 97 8b ff 03 92 a9
         e4 52 a4 fb ad 58 aa 14 78 4d 5a 24 1c 6b 49 da cc fb

   {server}  calculate finished "tls13 finished" (same as client)

   {server}  derive read traffic keys for application data (same as
      client application data write traffic keys)

   {server}  derive secret "tls13 res master" (same as client)

   {client}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 e4 ad 7d 44 c2 92 45
         33 9d 35 59 62 c7 79 b8 9e f4 4c 58

   {server}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 1d ec c5 d6 e6 4b ba
         8a 6f 21 b4 fd 07 74 97 da 2a 90 cb

## 7. Compatibility Mode

   This example shows use of the handshake with the client requesting
   that the server use compatibility mode as defined in Appendix D.4 of
   [TLS13].

   {client}  create an ephemeral x25519 key pair:

      private key (32 octets):  de a0 0b 45 69 5d c7 81 f1 9d 34 a6 2c
         1a fd 31 ab 43 69 af 1e 85 5a 3b bb 25 8d 84 42 cd e6 d7

      public key (32 octets):  8e 72 92 cf 30 56 db b0 d2 5f cb e5 5c 10
         7d c9 bb f8 3d d9 70 8f 39 20 3b a3 41 24 9a 7d 9b 63

   {client}  construct a ClientHello handshake message:

      ClientHello (224 octets):  01 00 00 dc 03 03 4e 64 0a 3f 2c 27 38
         f0 9c 94 18 bd 78 ed cc d7 55 9d 05 31 19 92 76 d4 d9 2a 0e 9e
         e9 d7 7d 09 20 a8 0c 16 55 81 a8 e0 d0 6c 00 18 d5 4d 3a 06 dd
         32 cf d4 05 1e b0 26 fa d3 fd 0b a9 92 69 e6 ef 00 06 13 01 13
         03 13 02 01 00 00 8d 00 00 00 0b 00 09 00 00 06 73 65 72 76 65
         72 ff 01 00 01 00 00 0a 00 14 00 12 00 1d 00 17 00 18 00 19 01
         00 01 01 01 02 01 03 01 04 00 33 00 26 00 24 00 1d 00 20 8e 72
         92 cf 30 56 db b0 d2 5f cb e5 5c 10 7d c9 bb f8 3d d9 70 8f 39
         20 3b a3 41 24 9a 7d 9b 63 00 2b 00 03 02 03 04 00 0d 00 20 00
         1e 04 03 05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01
         02 01 04 02 05 02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40
         01

   {client}  send handshake record:

      payload (224 octets):  01 00 00 dc 03 03 4e 64 0a 3f 2c 27 38 f0
         9c 94 18 bd 78 ed cc d7 55 9d 05 31 19 92 76 d4 d9 2a 0e 9e e9
         d7 7d 09 20 a8 0c 16 55 81 a8 e0 d0 6c 00 18 d5 4d 3a 06 dd 32
         cf d4 05 1e b0 26 fa d3 fd 0b a9 92 69 e6 ef 00 06 13 01 13 03
         13 02 01 00 00 8d 00 00 00 0b 00 09 00 00 06 73 65 72 76 65 72
         ff 01 00 01 00 00 0a 00 14 00 12 00 1d 00 17 00 18 00 19 01 00
         01 01 01 02 01 03 01 04 00 33 00 26 00 24 00 1d 00 20 8e 72 92
         cf 30 56 db b0 d2 5f cb e5 5c 10 7d c9 bb f8 3d d9 70 8f 39 20
         3b a3 41 24 9a 7d 9b 63 00 2b 00 03 02 03 04 00 0d 00 20 00 1e
         04 03 05 03 06 03 02 03 08 04 08 05 08 06 04 01 05 01 06 01 02
         01 04 02 05 02 06 02 02 02 00 2d 00 02 01 01 00 1c 00 02 40 01

      complete record (229 octets):  16 03 01 00 e0 01 00 00 dc 03 03 4e
         64 0a 3f 2c 27 38 f0 9c 94 18 bd 78 ed cc d7 55 9d 05 31 19 92
         76 d4 d9 2a 0e 9e e9 d7 7d 09 20 a8 0c 16 55 81 a8 e0 d0 6c 00
         18 d5 4d 3a 06 dd 32 cf d4 05 1e b0 26 fa d3 fd 0b a9 92 69 e6
         ef 00 06 13 01 13 03 13 02 01 00 00 8d 00 00 00 0b 00 09 00 00
         06 73 65 72 76 65 72 ff 01 00 01 00 00 0a 00 14 00 12 00 1d 00
         17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 33 00 26 00 24
         00 1d 00 20 8e 72 92 cf 30 56 db b0 d2 5f cb e5 5c 10 7d c9 bb
         f8 3d d9 70 8f 39 20 3b a3 41 24 9a 7d 9b 63 00 2b 00 03 02 03
         04 00 0d 00 20 00 1e 04 03 05 03 06 03 02 03 08 04 08 05 08 06
         04 01 05 01 06 01 02 01 04 02 05 02 06 02 02 02 00 2d 00 02 01
         01 00 1c 00 02 40 01

   {server}  extract secret "early":

      salt:  0 (all zero octets)

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c
         e2 10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a 
   {server}  create an ephemeral x25519 key pair:

      private key (32 octets):  01 7c 38 a3 64 79 21 ca 2d 9e d6 bd 7a
         e7 13 2b 94 21 1b 13 31 bb 20 8c 8c cd d5 15 56 40 99 95

      public key (32 octets):  3e 30 f0 f4 ba 55 1a fd 62 76 83 41 17 5f
         52 65 e4 da f0 c8 84 16 17 aa 4f af dd 21 42 32 0c 22

   {server}  construct a ServerHello handshake message:

      ServerHello (122 octets):  02 00 00 76 03 03 e5 dd 59 48 c4 35 f7
         a3 8f 0f 01 30 70 8d c3 22 d9 df 09 ab d4 83 81 17 c1 83 a7 bb
         6d 99 4f 2c 20 a8 0c 16 55 81 a8 e0 d0 6c 00 18 d5 4d 3a 06 dd
         32 cf d4 05 1e b0 26 fa d3 fd 0b a9 92 69 e6 ef 13 01 00 00 2e
         00 33 00 24 00 1d 00 20 3e 30 f0 f4 ba 55 1a fd 62 76 83 41 17
         5f 52 65 e4 da f0 c8 84 16 17 aa 4f af dd 21 42 32 0c 22 00 2b
         00 02 03 04

   {server}  send handshake record:

      payload (122 octets):  02 00 00 76 03 03 e5 dd 59 48 c4 35 f7 a3
         8f 0f 01 30 70 8d c3 22 d9 df 09 ab d4 83 81 17 c1 83 a7 bb 6d
         99 4f 2c 20 a8 0c 16 55 81 a8 e0 d0 6c 00 18 d5 4d 3a 06 dd 32
         cf d4 05 1e b0 26 fa d3 fd 0b a9 92 69 e6 ef 13 01 00 00 2e 00
         33 00 24 00 1d 00 20 3e 30 f0 f4 ba 55 1a fd 62 76 83 41 17 5f
         52 65 e4 da f0 c8 84 16 17 aa 4f af dd 21 42 32 0c 22 00 2b 00
         02 03 04

      complete record (127 octets):  16 03 03 00 7a 02 00 00 76 03 03 e5
         dd 59 48 c4 35 f7 a3 8f 0f 01 30 70 8d c3 22 d9 df 09 ab d4 83
         81 17 c1 83 a7 bb 6d 99 4f 2c 20 a8 0c 16 55 81 a8 e0 d0 6c 00
         18 d5 4d 3a 06 dd 32 cf d4 05 1e b0 26 fa d3 fd 0b a9 92 69 e6
         ef 13 01 00 00 2e 00 33 00 24 00 1d 00 20 3e 30 f0 f4 ba 55 1a
         fd 62 76 83 41 17 5f 52 65 e4 da f0 c8 84 16 17 aa 4f af dd 21
         42 32 0c 22 00 2b 00 02 03 04

   {server}  send change_cipher_spec record:

      payload (1 octets):  01

      complete record (6 octets):  14 03 03 00 01 01

   {server}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a 
      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {server}  extract secret "handshake":

      salt (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba b6 97
         16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

      IKM (32 octets):  ee f7 90 55 90 77 db 5b b6 3b 66 84 e4 16 9f 05
         1e 8f b3 4c e5 9b af ce 2f 9c 8e e6 8c c4 eb 79

      secret (32 octets):  f9 17 61 35 4a 67 e9 b0 7c 6d cc 3a 55 70 7e
         fa 69 c4 51 9d 80 40 e5 f2 15 12 1e 0d f6 9a fa 4a

   {server}  derive secret "tls13 c hs traffic":

      PRK (32 octets):  f9 17 61 35 4a 67 e9 b0 7c 6d cc 3a 55 70 7e fa
         69 c4 51 9d 80 40 e5 f2 15 12 1e 0d f6 9a fa 4a

      hash (32 octets):  74 5c 55 ba c3 99 31 0b 7b 5a 7c 81 a2 c1 30 b4
         d5 6d ff 6f 68 c3 ab 47 78 57 60 1e 01 f1 f8 d1

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 68 73 20 74 72
         61 66 66 69 63 20 74 5c 55 ba c3 99 31 0b 7b 5a 7c 81 a2 c1 30
         b4 d5 6d ff 6f 68 c3 ab 47 78 57 60 1e 01 f1 f8 d1

      expanded (32 octets):  2c 3c b2 4a 10 81 ed b5 95 18 ee 68 61 e8
         9a 6b 72 b3 80 1a fe 77 13 e4 cb bc 21 c0 79 5b f8 31

   {server}  derive secret "tls13 s hs traffic":

      PRK (32 octets):  f9 17 61 35 4a 67 e9 b0 7c 6d cc 3a 55 70 7e fa
         69 c4 51 9d 80 40 e5 f2 15 12 1e 0d f6 9a fa 4a

      hash (32 octets):  74 5c 55 ba c3 99 31 0b 7b 5a 7c 81 a2 c1 30 b4
         d5 6d ff 6f 68 c3 ab 47 78 57 60 1e 01 f1 f8 d1

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 68 73 20 74 72
         61 66 66 69 63 20 74 5c 55 ba c3 99 31 0b 7b 5a 7c 81 a2 c1 30
         b4 d5 6d ff 6f 68 c3 ab 47 78 57 60 1e 01 f1 f8 d1
      expanded (32 octets):  ca ce 3d 55 5c c1 c5 77 cf 97 0c ff 28 cf
         97 8d 6a 98 00 08 54 42 e1 8d 69 5b 50 f3 15 1d 18 c8

   {server}  derive secret for master "tls13 derived":

      PRK (32 octets):  f9 17 61 35 4a 67 e9 b0 7c 6d cc 3a 55 70 7e fa
         69 c4 51 9d 80 40 e5 f2 15 12 1e 0d f6 9a fa 4a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  5d a1 2d c4 78 35 ba 73 fd d9 94 b1 4a b7
         e6 3c c6 3f 0d 79 16 2f 67 56 e9 a4 67 56 c8 b2 b6 42

   {server}  extract secret "master":

      salt (32 octets):  5d a1 2d c4 78 35 ba 73 fd d9 94 b1 4a b7 e6 3c
         c6 3f 0d 79 16 2f 67 56 e9 a4 67 56 c8 b2 b6 42

      IKM (32 octets):  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
         00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

      secret (32 octets):  62 81 12 da e2 f7 02 48 80 63 e4 2d e6 c8 50
         a5 c0 82 0b 90 90 3e 00 ab c3 18 75 da 03 d4 bc 5b

   {server}  derive write traffic keys for handshake data:

      PRK (32 octets):  ca ce 3d 55 5c c1 c5 77 cf 97 0c ff 28 cf 97 8d
         6a 98 00 08 54 42 e1 8d 69 5b 50 f3 15 1d 18 c8

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  04 10 91 fd ab 29 f2 c8 ab fb 15 6d c5
         fc 8d 54

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  74 64 d7 91 68 5d e0 59 98 fc ba db

   {server}  construct an EncryptedExtensions handshake message:

      EncryptedExtensions (40 octets):  08 00 00 24 00 22 00 0a 00 14 00
         12 00 1d 00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c
         00 02 40 01 00 00 00 00
   {server}  construct a Certificate handshake message:

      Certificate (445 octets):  0b 00 01 b9 00 00 01 b5 00 01 b0 30 82
         01 ac 30 82 01 15 a0 03 02 01 02 02 01 02 30 0d 06 09 2a 86 48
         86 f7 0d 01 01 0b 05 00 30 0e 31 0c 30 0a 06 03 55 04 03 13 03
         72 73 61 30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35 39 5a 17
         0d 32 36 30 37 33 30 30 31 32 33 35 39 5a 30 0e 31 0c 30 0a 06
         03 55 04 03 13 03 72 73 61 30 81 9f 30 0d 06 09 2a 86 48 86 f7
         0d 01 01 01 05 00 03 81 8d 00 30 81 89 02 81 81 00 b4 bb 49 8f
         82 79 30 3d 98 08 36 39 9b 36 c6 98 8c 0c 68 de 55 e1 bd b8 26
         d3 90 1a 24 61 ea fd 2d e4 9a 91 d0 15 ab bc 9a 95 13 7a ce 6c
         1a f1 9e aa 6a f9 8c 7c ed 43 12 09 98 e1 87 a8 0e e0 cc b0 52
         4b 1b 01 8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a 5f da 43 08 46 74
         80 30 53 0e f0 46 1c 8c a9 d9 ef bf ae 8e a6 d1 d0 3e 2b d1 93
         ef f0 ab 9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7 9f 7f 1e 3f 02 03
         01 00 01 a3 1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06
         03 55 1d 0f 04 04 03 02 05 a0 30 0d 06 09 2a 86 48 86 f7 0d 01
         01 0b 05 00 03 81 81 00 85 aa d2 a0 e5 b9 27 6b 90 8c 65 f7 3a
         72 67 17 06 18 a5 4c 5f 8a 7b 33 7d 2d f7 a5 94 36 54 17 f2 ea
         e8 f8 a5 8c 8f 81 72 f9 31 9c f3 6b 7f d6 c5 5b 80 f2 1a 03 01
         51 56 72 60 96 fd 33 5e 5e 67 f2 db f1 02 70 2e 60 8c ca e6 be
         c1 fc 63 a4 2a 99 be 5c 3e b7 10 7c 3c 54 e9 b9 eb 2b d5 20 3b
         1c 3b 84 e0 a8 b2 f7 59 40 9b a3 ea c9 d9 1d 40 2d cc 0c c8 f8
         96 12 29 ac 91 87 b4 2b 4d e1 00 00

   {server}  construct a CertificateVerify handshake message:

      CertificateVerify (136 octets):  0f 00 00 84 08 04 00 80 a2 30 1a
         68 dd 1c ee e6 93 8f e9 d4 0c 46 b9 20 1b 34 d5 99 52 a3 7e 06
         52 3a 39 cf 8b a6 c9 c8 b6 8a e9 44 92 af 78 05 16 ed 7b 73 c8
         28 12 e9 9d d3 fa be a4 5e 09 d9 c6 84 87 21 c2 80 8c 61 50 1b
         0c 75 e7 fc ab a5 f7 8b ef 68 a2 c2 b6 9b 19 55 8b 3e 40 38 7e
         ea 93 d2 5c 77 81 c1 cc 00 e9 f5 19 f7 e2 e4 ad b7 3e 76 d6 60
         89 00 0a 2d c8 66 c2 ed 30 bb a5 0a 0d 45 7f 19 dc 6e b9 f3

   {server}  calculate finished "tls13 finished":

      PRK (32 octets):  ca ce 3d 55 5c c1 c5 77 cf 97 0c ff 28 cf 97 8d
         6a 98 00 08 54 42 e1 8d 69 5b 50 f3 15 1d 18 c8

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  2c 9f 72 f2 7b 81 e7 df 66 8c ac cd 49 37
         1f 12 86 d4 11 e1 6c 8c cc 1c 0d 9a ed 72 cb bd c0 80
      finished (32 octets):  c8 c3 a8 f1 bf f5 27 40 61 f4 bc 3a 7c af
         fb dc 96 16 09 4c a6 25 ca a6 5f 8e 76 ed 46 db 74 d3

   {server}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 c8 c3 a8 f1 bf f5 27 40 61 f4
         bc 3a 7c af fb dc 96 16 09 4c a6 25 ca a6 5f 8e 76 ed 46 db 74
         d3

   {server}  send handshake record:

      payload (657 octets):  08 00 00 24 00 22 00 0a 00 14 00 12 00 1d
         00 17 00 18 00 19 01 00 01 01 01 02 01 03 01 04 00 1c 00 02 40
         01 00 00 00 00 0b 00 01 b9 00 00 01 b5 00 01 b0 30 82 01 ac 30
         82 01 15 a0 03 02 01 02 02 01 02 30 0d 06 09 2a 86 48 86 f7 0d
         01 01 0b 05 00 30 0e 31 0c 30 0a 06 03 55 04 03 13 03 72 73 61
         30 1e 17 0d 31 36 30 37 33 30 30 31 32 33 35 39 5a 17 0d 32 36
         30 37 33 30 30 31 32 33 35 39 5a 30 0e 31 0c 30 0a 06 03 55 04
         03 13 03 72 73 61 30 81 9f 30 0d 06 09 2a 86 48 86 f7 0d 01 01
         01 05 00 03 81 8d 00 30 81 89 02 81 81 00 b4 bb 49 8f 82 79 30
         3d 98 08 36 39 9b 36 c6 98 8c 0c 68 de 55 e1 bd b8 26 d3 90 1a
         24 61 ea fd 2d e4 9a 91 d0 15 ab bc 9a 95 13 7a ce 6c 1a f1 9e
         aa 6a f9 8c 7c ed 43 12 09 98 e1 87 a8 0e e0 cc b0 52 4b 1b 01
         8c 3e 0b 63 26 4d 44 9a 6d 38 e2 2a 5f da 43 08 46 74 80 30 53
         0e f0 46 1c 8c a9 d9 ef bf ae 8e a6 d1 d0 3e 2b d1 93 ef f0 ab
         9a 80 02 c4 74 28 a6 d3 5a 8d 88 d7 9f 7f 1e 3f 02 03 01 00 01
         a3 1a 30 18 30 09 06 03 55 1d 13 04 02 30 00 30 0b 06 03 55 1d
         0f 04 04 03 02 05 a0 30 0d 06 09 2a 86 48 86 f7 0d 01 01 0b 05
         00 03 81 81 00 85 aa d2 a0 e5 b9 27 6b 90 8c 65 f7 3a 72 67 17
         06 18 a5 4c 5f 8a 7b 33 7d 2d f7 a5 94 36 54 17 f2 ea e8 f8 a5
         8c 8f 81 72 f9 31 9c f3 6b 7f d6 c5 5b 80 f2 1a 03 01 51 56 72
         60 96 fd 33 5e 5e 67 f2 db f1 02 70 2e 60 8c ca e6 be c1 fc 63
         a4 2a 99 be 5c 3e b7 10 7c 3c 54 e9 b9 eb 2b d5 20 3b 1c 3b 84
         e0 a8 b2 f7 59 40 9b a3 ea c9 d9 1d 40 2d cc 0c c8 f8 96 12 29
         ac 91 87 b4 2b 4d e1 00 00 0f 00 00 84 08 04 00 80 a2 30 1a 68
         dd 1c ee e6 93 8f e9 d4 0c 46 b9 20 1b 34 d5 99 52 a3 7e 06 52
         3a 39 cf 8b a6 c9 c8 b6 8a e9 44 92 af 78 05 16 ed 7b 73 c8 28
         12 e9 9d d3 fa be a4 5e 09 d9 c6 84 87 21 c2 80 8c 61 50 1b 0c
         75 e7 fc ab a5 f7 8b ef 68 a2 c2 b6 9b 19 55 8b 3e 40 38 7e ea
         93 d2 5c 77 81 c1 cc 00 e9 f5 19 f7 e2 e4 ad b7 3e 76 d6 60 89
         00 0a 2d c8 66 c2 ed 30 bb a5 0a 0d 45 7f 19 dc 6e b9 f3 14 00
         00 20 c8 c3 a8 f1 bf f5 27 40 61 f4 bc 3a 7c af fb dc 96 16 09
         4c a6 25 ca a6 5f 8e 76 ed 46 db 74 d3

      complete record (679 octets):  17 03 03 02 a2 48 de 89 1d 9c 36 24
         a6 7a 6c 6f 06 01 ab 7a c2 0c 1f 6a 9e 14 d2 e6 00 7e 99 9e 13
         03 67 a8 af 1b cf ea 94 98 fb ce 19 df 45 05 ee ce 3a 25 da 52
         3c be 55 ea 1b 3b da 4e 91 99 5e 45 5d 50 0a 4f aa 62 27 b7 11
         1e 1c 85 47 e2 d7 c1 79 db 21 53 03 d2 58 27 f3 cd 18 f4 8f 64
         91 32 8c f5 c0 f8 14 d3 88 15 0b d9 e9 26 4a ae 49 1d b6 99 50
         69 be a1 76 65 d5 e0 c8 17 28 4d 4a c2 18 80 05 4c 36 57 33 1e
         23 a9 30 4d c8 8a 15 c0 4e c8 0b d3 85 2b f7 f9 d3 c6 61 5b 15
         fa c8 3b bc a0 31 c6 d2 31 0d 9f 5d 7a 4b 02 0a 4f 7c 19 06 2b
         65 c0 5a 1d 32 64 b5 57 ec 9d 8e 0f 7c ee 27 e3 6f 79 30 39 de
         8d d9 6e df ca 90 09 e0 65 10 34 bf f3 1d 7f 34 9e ec e0 1d 99
         fc b5 fc ab 84 0d 77 07 c7 22 99 c3 b5 d0 45 64 e8 80 a3 3c 5e
         84 6c 76 2e 3d 92 2b b5 53 03 d1 d8 7c c0 f0 65 73 f1 7d cb 9b
         8f fd 35 bb d8 83 c1 cb 3a a2 4f cc 32 50 05 f7 68 ce 2f b6 24
         ca 97 b6 c4 d9 8e 17 f3 5b c2 c7 94 0a 06 10 0c 2d 44 8d b7 18
         0b 2d 86 21 64 43 5c 9c 21 0e 98 60 39 4e 05 aa b2 3f f1 b0 20
         3f 66 2c 58 8d a5 bc 44 11 47 7a 30 b4 11 36 c4 88 a0 a6 3f ca
         b5 c1 5a c6 13 22 6d ae 82 7a 1d 1f e9 5e ce 6b 30 bc ee 15 60
         a8 d4 08 d2 64 55 5e 76 0f 9b fc 62 4c 2c 87 fd 04 56 c9 bf b4
         1b cd 1a 7b 21 27 86 d2 b6 7f d5 78 04 fa cf a1 ee f7 cf 29 19
         d8 b9 98 c9 78 9f 76 3b 4d 9c aa 09 3a 9d ed 43 17 5d 46 a7 6b
         4d 54 f0 ce 0c 5d 22 59 b6 07 e3 0a 9d 24 12 63 87 4f a5 9d 6f
         57 0d c4 0d 83 a2 d8 3b f9 e9 85 0d 45 4c 57 80 65 35 a8 99 8a
         e0 35 7d f9 2f 00 b9 66 73 44 c2 41 14 cc c9 ef 53 91 24 b2 04
         e7 e6 e7 48 c3 0a 28 a3 d1 d1 83 99 72 43 ea cc bb d3 3b 0c 11
         15 a0 32 71 06 a1 e6 a7 52 71 d4 98 30 86 f6 32 ff 0e b8 b4 c6
         31 02 cb ce f5 bb 72 da e1 27 9d 5d e8 eb 19 09 6d 8c db 07 fa
         8e a9 89 78 8f ac 23 e6 6e 04 88 c1 93 f3 f3 fe a8 c8 83 88 96
         bf 3a e4 b6 84 8d 42 ce d4 bd f4 1a be 6f c3 31 b4 42 25 e7 a1
         f7 d3 56 41 47 d5 45 8e 71 aa 90 9c b0 2b e9 58 bb c4 2e 3a a5
         a2 7c c6 ea f4 b6 fe 51 ae 44 95 69 4d 8a b6 32 0a ab 92 01 83
         fd 5b 31 a3 59 04 2f bd 67 39 1e c5 e4 d1 89 2a 2e 52 10 14 1a
         49 4e 93 01 b2 4a 11 3c 47 4c 7f 2a 73 45 78 47

   {server}  derive secret "tls13 c ap traffic":

      PRK (32 octets):  62 81 12 da e2 f7 02 48 80 63 e4 2d e6 c8 50 a5
         c0 82 0b 90 90 3e 00 ab c3 18 75 da 03 d4 bc 5b

      hash (32 octets):  07 07 dc ac 7b 2f a4 28 cc 7f 69 16 94 a2 59 0c
         80 6a aa 5c 0c f5 08 7e d5 38 50 12 e7 f9 6c d4

      info (54 octets):  00 20 12 74 6c 73 31 33 20 63 20 61 70 20 74 72
         61 66 66 69 63 20 07 07 dc ac 7b 2f a4 28 cc 7f 69 16 94 a2 59
         0c 80 6a aa 5c 0c f5 08 7e d5 38 50 12 e7 f9 6c d4

      expanded (32 octets):  74 3e 4c 6b 56 cf 39 09 d1 b0 6d 01 95 6c
         cd 2c 4b 37 75 84 49 ae c4 1d 98 da e4 49 24 ea a2 99
   {server}  derive secret "tls13 s ap traffic":

      PRK (32 octets):  62 81 12 da e2 f7 02 48 80 63 e4 2d e6 c8 50 a5
         c0 82 0b 90 90 3e 00 ab c3 18 75 da 03 d4 bc 5b

      hash (32 octets):  07 07 dc ac 7b 2f a4 28 cc 7f 69 16 94 a2 59 0c
         80 6a aa 5c 0c f5 08 7e d5 38 50 12 e7 f9 6c d4

      info (54 octets):  00 20 12 74 6c 73 31 33 20 73 20 61 70 20 74 72
         61 66 66 69 63 20 07 07 dc ac 7b 2f a4 28 cc 7f 69 16 94 a2 59
         0c 80 6a aa 5c 0c f5 08 7e d5 38 50 12 e7 f9 6c d4

      expanded (32 octets):  b6 b8 14 4a a3 35 ed 30 59 c0 c9 c8 f0 ec
         ab f7 af c9 4a f6 64 3b de cd fd 92 10 18 8f ab 74 51

   {server}  derive secret "tls13 exp master":

      PRK (32 octets):  62 81 12 da e2 f7 02 48 80 63 e4 2d e6 c8 50 a5
         c0 82 0b 90 90 3e 00 ab c3 18 75 da 03 d4 bc 5b

      hash (32 octets):  07 07 dc ac 7b 2f a4 28 cc 7f 69 16 94 a2 59 0c
         80 6a aa 5c 0c f5 08 7e d5 38 50 12 e7 f9 6c d4

      info (52 octets):  00 20 10 74 6c 73 31 33 20 65 78 70 20 6d 61 73
         74 65 72 20 07 07 dc ac 7b 2f a4 28 cc 7f 69 16 94 a2 59 0c 80
         6a aa 5c 0c f5 08 7e d5 38 50 12 e7 f9 6c d4

      expanded (32 octets):  fb 69 12 1c ea 33 4d b4 59 e1 22 72 d1 79
         ba ca 23 69 b6 43 d1 1a 6a c7 2b 8b 27 a5 c9 64 fe b1

   {server}  derive write traffic keys for application data:

      PRK (32 octets):  b6 b8 14 4a a3 35 ed 30 59 c0 c9 c8 f0 ec ab f7
         af c9 4a f6 64 3b de cd fd 92 10 18 8f ab 74 51

      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  ed c4 cb d0 04 1c 28 cc 71 67 44 1d 7c
         a5 3e 6a

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  bf 6c 7d 8e 0a 95 45 b4 27 dc f1 39

   {server}  derive read traffic keys for handshake data:

      PRK (32 octets):  2c 3c b2 4a 10 81 ed b5 95 18 ee 68 61 e8 9a 6b
         72 b3 80 1a fe 77 13 e4 cb bc 21 c0 79 5b f8 31
      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  62 d1 3c 13 ff d7 40 2f c1 c0 9e 3d 16
         36 65 cb

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  71 66 f2 00 28 bf 14 6d cf bd 5a 40

   {client}  extract secret "early" (same as server early secret)

   {client}  derive secret for handshake "tls13 derived":

      PRK (32 octets):  33 ad 0a 1c 60 7e c0 3b 09 e6 cd 98 93 68 0c e2
         10 ad f3 00 aa 1f 26 60 e1 b2 2e 10 f1 70 f9 2a

      hash (32 octets):  e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24
         27 ae 41 e4 64 9b 93 4c a4 95 99 1b 78 52 b8 55

      info (49 octets):  00 20 0d 74 6c 73 31 33 20 64 65 72 69 76 65 64
         20 e3 b0 c4 42 98 fc 1c 14 9a fb f4 c8 99 6f b9 24 27 ae 41 e4
         64 9b 93 4c a4 95 99 1b 78 52 b8 55

      expanded (32 octets):  6f 26 15 a1 08 c7 02 c5 67 8f 54 fc 9d ba
         b6 97 16 c0 76 18 9c 48 25 0c eb ea c3 57 6c 36 11 ba

   {client}  extract secret "handshake" (same as server handshake
      secret)

   {client}  derive secret "tls13 c hs traffic" (same as server)

   {client}  derive secret "tls13 s hs traffic" (same as server)

   {client}  derive secret for master "tls13 derived" (same as server)

   {client}  extract secret "master" (same as server master secret)

   {client}  derive read traffic keys for handshake data (same as server
      handshake data write traffic keys)

   {client}  calculate finished "tls13 finished" (same as server)

   {client}  derive secret "tls13 c ap traffic" (same as server)

   {client}  derive secret "tls13 s ap traffic" (same as server)

   {client}  derive secret "tls13 exp master" (same as server)
   {client}  send change_cipher_spec record:

      payload (1 octets):  01

      complete record (6 octets):  14 03 03 00 01 01

   {client}  derive write traffic keys for handshake data (same as
      server handshake data read traffic keys)

   {client}  derive read traffic keys for application data (same as
      server application data write traffic keys)

   {client}  calculate finished "tls13 finished":

      PRK (32 octets):  2c 3c b2 4a 10 81 ed b5 95 18 ee 68 61 e8 9a 6b
         72 b3 80 1a fe 77 13 e4 cb bc 21 c0 79 5b f8 31

      hash (0 octets):  (empty)

      info (18 octets):  00 20 0e 74 6c 73 31 33 20 66 69 6e 69 73 68 65
         64 00

      expanded (32 octets):  77 34 1a bc 8c 0f fa b5 18 07 36 71 3e 41
         d2 f6 65 c4 10 a4 04 c8 c2 1e dc d9 48 a4 44 0f d8 0c

      finished (32 octets):  69 2c ab 15 5c c6 c1 00 ea d6 07 33 d0 61
         7f 6f b0 9b 71 aa 1e 8c 9a cc bb bc 9e 8e d3 36 c1 dd

   {client}  construct a Finished handshake message:

      Finished (36 octets):  14 00 00 20 69 2c ab 15 5c c6 c1 00 ea d6
         07 33 d0 61 7f 6f b0 9b 71 aa 1e 8c 9a cc bb bc 9e 8e d3 36 c1
         dd

   {client}  send handshake record:

      payload (36 octets):  14 00 00 20 69 2c ab 15 5c c6 c1 00 ea d6 07
         33 d0 61 7f 6f b0 9b 71 aa 1e 8c 9a cc bb bc 9e 8e d3 36 c1 dd

      complete record (58 octets):  17 03 03 00 35 32 d0 30 e2 73 77 3a
         86 96 c7 99 98 1a f6 ce d0 7f 87 48 2e 81 56 5e 39 4e 87 c8 67
         f3 3d f3 d6 5b 75 06 f1 a6 26 af 91 d4 82 1d 5f 7a 1f 21 0e f8
         dd 3c 6d 16

   {client}  derive write traffic keys for application data:

      PRK (32 octets):  74 3e 4c 6b 56 cf 39 09 d1 b0 6d 01 95 6c cd 2c
         4b 37 75 84 49 ae c4 1d 98 da e4 49 24 ea a2 99
      key info (13 octets):  00 10 09 74 6c 73 31 33 20 6b 65 79 00

      key expanded (16 octets):  33 d7 f9 70 97 56 c9 66 48 8a d4 43 84
         37 e6 73

      iv info (12 octets):  00 0c 08 74 6c 73 31 33 20 69 76 00

      iv expanded (12 octets):  c5 f3 0d 34 b0 e9 1b 7d 6c 8e ea 65

   {client}  derive secret "tls13 res master":

      PRK (32 octets):  62 81 12 da e2 f7 02 48 80 63 e4 2d e6 c8 50 a5
         c0 82 0b 90 90 3e 00 ab c3 18 75 da 03 d4 bc 5b

      hash (32 octets):  a0 21 d3 a0 5b d4 18 a7 72 81 38 75 ef 79 b0 af
         68 c5 12 32 15 42 7a b7 33 3f 8c 27 72 2a 9f d5

      info (52 octets):  00 20 10 74 6c 73 31 33 20 72 65 73 20 6d 61 73
         74 65 72 20 a0 21 d3 a0 5b d4 18 a7 72 81 38 75 ef 79 b0 af 68
         c5 12 32 15 42 7a b7 33 3f 8c 27 72 2a 9f d5

      expanded (32 octets):  0b 5d 44 07 ce a0 a4 2a 3a 81 dd 47 76 47
         b7 fe 91 80 db 29 7e 51 14 f1 ad 87 96 b4 dc 47 50 04

   {server}  calculate finished "tls13 finished" (same as client)

   {server}  derive read traffic keys for application data (same as
      client application data write traffic keys)

   {server}  derive secret "tls13 res master" (same as client)

   {client}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 0f 62 91 55 38 2d ba
         23 c4 e2 c5 f7 f8 4e 6f 2e d3 08 3d

   {server}  send alert record:

      payload (2 octets):  01 00

      complete record (24 octets):  17 03 03 00 13 b7 25 7b 0f ec af 69
         d4 f0 9e 3f 89 1e 2a 25 d1 e2 88 45





## 8. Security Considerations

   It probably isn't a good idea to use the private key included in this
   document.  In addition to the fact that it is too small to provide
   any meaningful security, it is now very well known.

# Referenced Sections from RFC 4086: Randomness Requirements for Security

The following sections were referenced. Remaining sections are not included.

### 3.2. Existing Hardware Can Be Used For Randomness

   As described below, many computers come with hardware that can, with
   care, be used to generate truly random quantities.

#### 3.2.1. Using Existing Sound/Video Input

   Many computers are built with inputs that digitize some real-world
   analog source, such as sound from a microphone or video input from a
   camera.  The "input" from a sound digitizer with no source plugged in
   or from a camera with the lens cap on is essentially thermal noise.
   If the system has enough gain to detect anything, such input can
   provide reasonably high quality random bits.  This method is
   extremely dependent on the hardware implementation.

   For example, on some UNIX-based systems, one can read from the
   /dev/audio device with nothing plugged into the microphone jack or
   with the microphone receiving only low level background noise.  Such
   data is essentially random noise, although it should not be trusted
   without some checking, in case of hardware failure, and it will have
   to be de-skewed.

   Combining this approach with compression to de-skew (see Section 4),
   one can generate a huge amount of medium-quality random data with the
   UNIX-style command line:

        cat /dev/audio | compress - >random-bits-file

   A detailed examination of this type of randomness source appears in
   [TURBID ].

#### 3.2.2. Using Existing Disk Drives

   Disk drives have small random fluctuations in their rotational speed
   due to chaotic air turbulence [DAVIS , Jakobsson ].  The addition of
   low-level disk seek-time instrumentation produces a series of
   measurements that contain this randomness.  Such data is usually
   highly correlated, so significant processing is needed, as described
   in Section 5.2 below.  Nevertheless, experimentation a decade ago
   showed that, with such processing, even slow disk drives on the
   slower computers of that day could easily produce 100 bits a minute
   or more of excellent random data.

   Every increase in processor speed, which increases the resolution
   with which disk motion can be timed or increases the rate of disk
   seeks, increases the rate of random bit generation possible with this
   technique.  At the time of this paper and with modern hardware, a
   more typical rate of random bit production would be in excess of 
   10,000 bits a second.  This technique is used in random number
   generators included in many operating system libraries.

   Note: the inclusion of cache memories in disk controllers has little
   effect on this technique if very short seek times, which represent
   cache hits, are simply ignored.

# Referenced Sections from RFC 8937: Randomness Improvements for Security Protocols

The following sections were referenced. Remaining sections are not included.

### 10.2. Informative References

   [DebianBug ]
              Yilek, S., Rescorla, E., Shacham, H., Enright, B., and S.
              Savage, "When private keys are public: results from the
              2008 Debian OpenSSL vulnerability", ICM '09,
              DOI 10.1145/1644893.1644896, November 2009,
              <https://pdfs.semanticscholar.org/fcf9/
              fe0946c20e936b507c023bbf89160cc995b9.pdf >.

   [DualEC ]   Bernstein, D., Lange, T., and R. Niederhagen, "Dual EC: A
              Standardized Back Door", DOI 10.1007/978-3-662-49301-4_17,
              March 2016, <https://projectbullrun.org/dual-ec/documents/
              dual-ec-20150731.pdf >.

   [MAFS2017] McGrew, D., Anderson, B., Fluhrer, S., and C. Schenefiel,
              "PRNG Failures and TLS Vulnerabilities in the Wild",
              January 2017,
              <https://rwc.iacr.org/2017/Slides/david.mcgrew.pptx >.

   [NAXOS ]    LaMacchia, B., Lauter, K., and A. Mityagin, "Stronger
              Security of Authenticated Key Exchange",
              DOI 10.1007/978-3-540-75670-5_1, November 2007,
              <https://www.microsoft.com/en-us/research/wp-
              content/uploads/2016/02/strongake-submitted.pdf >.

   [RY2010]   Ristenpart, T. and S. Yilek, "When Good Randomness Goes
              Bad: Virtual Machine Reset Vulnerabilities and Hedging
              Deployed Cryptography", January 2010,
              <https://rist.tech.cornell.edu/papers/sslhedge.pdf >.

   [SecAnalysis ]
              Akhmetzyanova, L., Cremers, C., Garratt, L., Smyshlyaev,
              S., and N. Sullivan, "Limiting the impact of unreliable
              randomness in deployed security protocols",
              DOI 10.1109/CSF49147.2020.00027, IEEE 33rd Computer
              Security Foundations Symposium (CSF), Boston, MA, USA, pp.
              385-393, 2020,
              <https://doi.org/10.1109/CSF49147.2020.00027>.

   [SP80090A ] National Institute of Standards and Technology,
              "Recommendation for Random Number Generation Using
              Deterministic Random Bit Generators, Special Publication
              800-90A Revision 1", DOI 10.6028/NIST.SP.800-90Ar1, June
              2015, <https://doi.org/10.6028/NIST.SP.800-90Ar1>.

   [X962]     American National Standard for Financial Services (ANSI),
              "Public Key Cryptography for the Financial Services
              Industry, The Elliptic Curve Digital Signature Algorithm
              (ECDSA)", ANSI X9.62, November 2005,
              <https://www.techstreet.com/standards/
              x9-x9-62-2005?product_id=1327225>.


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

# Referenced Sections from RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3

The following sections were referenced. Remaining sections are not included.

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

#### 7.4.1. Finite Field Diffie-Hellman

   For finite field groups, a conventional Diffie-Hellman [DH76]
   computation is performed.  The negotiated key (Z) is converted to a
   byte string by encoding in big-endian form and left-padded with zeros
   up to the size of the prime.  This byte string is used as the shared
   secret in the key schedule as specified above.

   Note that this construction differs from previous versions of TLS
   which removed leading zeros. 





# Referenced Sections from RFC 6979: Deterministic Usage of the Digital Signature Algorithm (DSA) and Elliptic Curve Digital Signature Algorithm (ECDSA)

The following sections were referenced. Remaining sections are not included.

## 3. Deterministic DSA and ECDSA

   Deterministic (EC)DSA is the process of generating an (EC)DSA
   signature over an input message m by using the standard (EC)DSA
   signature generation process (discussed in the previous section),
   except that the value k, instead of being randomly generated, is
   obtained through the process described in this section.

   We use the notations described in Section 2.

### 3.1. Building Blocks





#### 3.1.1. HMAC

   HMAC [RFC2104] is a construction of a Message Authentication Code
   using a hash function and a secret key.  Here, we use HMAC with the
   same hash function H as the one used to process the input message
   prior to signature generation or verification.

   We denote the process of applying HMAC with key K over data V by:

      HMAC_K(V)

   which returns a sequence of bits of length hlen (the output length of
   the underlying hash function H).

### 3.2. Generation of k

   Given the input message m, the following process is applied:

   a.  Process m through the hash function H, yielding:

          h1 = H(m)

       (h1 is a sequence of hlen bits).

   b.  Set:V  = 0x01 0x01 0x01 ... 0x01       such that the length of V, in bits, is equal to 8*ceil(hlen/8).
       For instance, on an octet-based system, if H is SHA-256, then V
       is set to a sequence of 32 octets of value 1.  Note that in this
       step and all subsequent steps, we use the same H function as the
       one used in step 'a' to process the input message; this choice
       will be discussed in more detail in Section 3.6. 
   c.  Set:K  = 0x00 0x00 0x00 ... 0x00       such that the length of K, in bits, is equal to 8*ceil(hlen/8).

   d.  Set:

          K = HMAC_K(V || 0x00 || int2octets(x) || bits2octets(h1))

       where '||' denotes concatenation.  In other words, we compute
       HMAC with key K, over the concatenation of the following, in
       order: the current value of V, a sequence of eight bits of value
       0, the encoding of the (EC)DSA private key x, and the hashed
       message (possibly truncated and extended as specified by the
       bits2octets transform).  The HMAC result is the new value of K.
       Note that the private key x is in the [1, q-1] range, hence a
       proper input for int2octets, yielding rlen bits of output, i.e.,
       an integral number of octets (rlen is a multiple of 8).

   e.  Set:

          V = HMAC_K(V)

   f.  Set:

          K = HMAC_K(V || 0x01 || int2octets(x) || bits2octets(h1))

       Note that the "internal octet" is 0x01 this time.

   g.  Set:

          V = HMAC_K(V)

   h.  Apply the following algorithm until a proper value is found for
       k:

       1.  Set T to the empty sequence.  The length of T (in bits) is
           denoted tlen; thus, at that point, tlen = 0.

       2.  While tlen < qlen, do the following:

              V = HMAC_K(V)

              T = T || V 
       3.  Compute:

              k = bits2int(T)

           If that value of k is within the [1,q-1] range, and is
           suitable for DSA or ECDSA (i.e., it results in an r value
           that is not 0; see Section 3.4), then the generation of k is
           finished.  The obtained value of k is used in DSA or ECDSA.
           Otherwise, compute:

              K = HMAC_K(V || 0x00)

              V = HMAC_K(V)

           and loop (try to generate a new T, and so on).

   Please note that when k is generated from T, the result of bits2int
   is compared to q, not reduced modulo q.  If the value is not between
   1 and q-1, the process loops.  Performing a simple modular reduction
   would induce biases that would be detrimental to signature security.

### 3.3. Alternate Description of the Generation of k

   The process described in the previous section is actually derived
   from the "HMAC_DRBG" pseudorandom number generator, described in
   [SP800-90A ] and Annex D of [X9.62].  Using the terminology from
   [SP800-90A ], the generation of k can be described as such:

   a.  Instantiate HMAC_DRBG using HMAC parameterized with the same hash
       function H as the one used for processing the message that is to
       be signed.  Instantiation parameters are:

       requested_instantiation_security_strength
          Set this parameter to any value that the HMAC_DRBG
          implementation will accept, when using H as base hash
          function.

       prediction_resistance_flag
          Set this parameter to "false".

       personalization_string
          Set this parameter to "Null" (the empty bit sequence).

       entropy_input
          Use int2octets(x) as entropy string.

       nonce
          Use bits2octets(H(m)) as nonce. 
       Note that the last two parameters are not parameters to the
       HMAC_DRBG instantiation function per se; instead, those values
       are requested from the internal Get_entropy_input function during
       instantiation.  For deterministic (EC)DSA, we want HMAC_DRBG to
       run with the entropy string and nonce that we specify, without
       accessing an actual entropy source.

   b.  Generate a candidate value for k by requesting qlen bits from
       HMAC_DRBG and converting the resulting bits into an integer with
       the bits2int transform.  Repeat this step until a value is
       obtained, which is non-zero, less than q, and suitable for
       (EC)DSA (see Section 3.4).

   Note that we instantiate a new HMAC_DRBG instance for each signature
   generation process.  There is no "personalization string" and no
   "additional input" when generating bits.  The reseed function of
   HMAC_DRBG is never invoked, neither externally nor as a consequence
   of the internal HMAC_DRBG processing.

   As shown above, we use the encoding of the private key as "entropy
   string" and the hashed message (truncated and expanded by
   bits2octets) as "nonce".  In HMAC_DRBG, the entropy string and nonce
   are simply concatenated into the initial seed; hence, the split
   between "entropy" and "nonce" is quite arbitrary.  Using qlen bits
   for each ought to be compatible with most HMAC_DRBG implementation
   input requirements.

### 3.4. Usage Notes

   With DSA or ECDSA, the value k is used to compute the first half of
   the signature, dubbed r (see Section 2.4).  The DSA and ECDSA
   standards mandate that, if r is zero, then a new k should be
   selected.  In that situation, this document specifies that the value
   k is "unsuitable", and the generation process shall keep on looping.

   This occurrence is utterly improbable.  Actually, it would require
   considerable computational effort (similar to breaking preimage
   resistance of the hash function) to find a private key and a message
   that lead to a zero value for r; hitting such a case by pure chance
   is thus deemed implausible, and an attacker cannot force it with
   carefully crafted messages.  In practice, such a code path will not
   be triggered and thus can be implemented with little optimization.

### 3.5. Rationale

   The process described in the previous sections mimics the "Approved"
   generation process of k described in Annex D of [X9.62], with the
   "HMAC_DRBG" pseudorandom number generator.  The main difference is 
   that we use the concatenation of the private key x and the hashed
   message H(m) as the pseudorandom number generator (PRNG) seed.  If
   using a "security level" of n bits, then HMAC_DRBG should be used
   with seed entropy at least n+64 bits; however, the key x should also
   have been generated with that much entropy, and the length of x is
   qlen, which is at least equal to 2*n and thus larger than n+64 (DSA
   and ECDSA, as specified by the standards, require qlen >= 160).  It
   can then be argued that deterministic ECDSA fulfills the entropy
   requirements of Annex D of [X9.62].

   We use bits2octets(H(m)) instead of H(m) in order to ease
   integration.  Indeed, many existing signature systems offload the
   message hashing; the signature engine (which has access to the
   private key) receives only H(m).  In some applications, where data
   bandwidth is constrained, only the first qlen bits of H(m) are
   transferred to the signature engine, on the basis that the bits2int
   transform will ignore subsequent bits anyway.  Possibly, in some
   systems, the truncated H(m) could be externally reduced modulo q,
   since that is the first thing that (EC)DSA performs on the hashed
   message.  With the definition of bits2octets, deterministic (EC)DSA
   can be applied with the same input.

### 3.6. Variants

   Many parts of the specification of deterministic (EC)DSA are quite
   arbitrary.  It is possible to define variants that are NOT
   "deterministic (EC)DSA" but that may nonetheless be useful in some
   contexts:

   o  It is possible to use H(m) directly, instead of bits2octets(H(m)),
      as part of the HMAC input.  As explained in Section 3.5, we use
      bits2octets(H(m)) in order to ease integration into systems that
      already use an (EC)DSA signature engine by sending it an already-
      truncated hash value.  Using the whole H(m) does not introduce any
      vulnerability.

   o  Additional data may be added to the input of HMAC, concatenated
      after bits2octets(H(m)):

         K = HMAC_K(V || 0x00 || int2octets(x) || bits2octets(h1) || k')

      A use case may be a protocol that requires a non-deterministic
      signature algorithm on a system that does not have access to a
      high-quality random source.  It suffices that the additional data
      k' is non-repeating (e.g., a signature counter or a monotonic
      clock) to ensure "random-looking" signatures are
      indistinguishable, in a cryptographic way, from plain (EC)DSA
      signatures.  In [SP800-90A ] terminology, k' is the "additional 
      input" that can be set as a parameter when generating pseudorandom
      bits.  This variant can be thought of as a "strengthening" of the
      randomness of the source of the additional data k'.

   o  Instead of using x (the private key) as input to HMAC, it is
      possible to use additional secret data, stored along with the
      private key with the same security measures.  The entropy of that
      additional data SHALL be at least n bits, preferably n+64 bits or
      more, where n is the target security level.  Having additional
      secret data may help in formally proving the security of
      derandomization, but it implies an extra storage cost and
      incompatibility with already-generated (EC)DSA private keys.

   o  Similarly, the private key could be a value z, from which both x
      (the "private key" in the plain (EC)DSA sense) and another value
      x', to be used as input to HMAC in the generation of k, would be
      derived through a suitable Pseudorandom Function (PRF) (such as
      HMAC_DRBG).  This would keep private key storage requirements to a
      minimum while providing a more easily proven security, but it
      would impact private key generation and would not be compatible
      with already-generated key pairs.

   o  In this document, we use the same hash function H for processing
      the input message and as a parameter to HMAC.  Two distinct hash
      functions could be used, provided that both are adequately secure.
      The overall security will be limited by the weaker of the two hash
      functions, i.e., the one with the smaller output.  Using a
      specific, constant hash function for HMAC may be useful for
      constrained implementations that accept externally hashed
      messages, regardless of what hash function was used for that, but
      have resources for implementing only one hash function for HMAC.

   The main disadvantage of any variant is that it ceases to be
   verifiable against the test vectors published in this document.

# Referenced Sections from RFC 9112: HTTP/1.1

The following sections were referenced. Remaining sections are not included.

### C.2. Changes from HTTP/1.0





#### C.2.1. Multihomed Web Servers

   The requirements that clients and servers support the Host header
   field (Section 7.2 of [HTTP ]), report an error if it is missing from
   an HTTP/1.1 request, and accept absolute URIs (Section 3.2) are among
   the most important changes defined by HTTP/1.1.

   Older HTTP/1.0 clients assumed a one-to-one relationship of IP
   addresses and servers; there was no established mechanism for
   distinguishing the intended server of a request other than the IP
   address to which that request was directed.  The Host header field
   was introduced during the development of HTTP/1.1 and, though it was
   quickly implemented by most HTTP/1.0 browsers, additional
   requirements were placed on all HTTP/1.1 requests in order to ensure
   complete adoption.  At the time of this writing, most HTTP-based
   services are dependent upon the Host header field for targeting
   requests.

#### C.2.2. Keep-Alive Connections

   In HTTP/1.0, each connection is established by the client prior to
   the request and closed by the server after sending the response.
   However, some implementations implement the explicitly negotiated
   ("Keep-Alive") version of persistent connections described in Section 19.7.1 of [RFC2068].

   Some clients and servers might wish to be compatible with these
   previous approaches to persistent connections, by explicitly
   negotiating for them with a "Connection: keep-alive" request header
   field.  However, some experimental implementations of HTTP/1.0
   persistent connections are faulty; for example, if an HTTP/1.0 proxy
   server doesn't understand Connection, it will erroneously forward
   that header field to the next inbound server, which would result in a
   hung connection.

   One attempted solution was the introduction of a Proxy-Connection
   header field, targeted specifically at proxies.  In practice, this
   was also unworkable, because proxies are often deployed in multiple
   layers, bringing about the same problem discussed above.

   As a result, clients are encouraged not to send the Proxy-Connection
   header field in any requests.

   Clients are also encouraged to consider the use of "Connection: keep-
   alive" in requests carefully; while they can enable persistent
   connections with HTTP/1.0 servers, clients using them will need to
   monitor the connection for "hung" requests (which indicate that the
   client ought to stop sending the header field), and this mechanism
   ought not be used by clients at all when a proxy is being used.

#### C.2.3. Introduction of Transfer-Encoding

   HTTP/1.1 introduces the Transfer-Encoding header field (Section 6.1).
   Transfer codings need to be decoded prior to forwarding an HTTP
   message over a MIME-compliant protocol.

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





# Referenced Sections from RFC 5929: Channel Bindings for TLS

The following sections were referenced. Remaining sections are not included.

## 9. IANA Considerations

   IANA updated three existing channel binding type registrations.  See
   the rest of this document.

# Summary of reference from Checkoway, S., Maskiewicz, J., Garman, C., Fried, J., Cohney, S., Green, M., Heninger, N., Weinmann, R., Rescorla, E., and H. Shacham, "A Systematic Analysis of the Juniper Dual EC Incident", ACM, Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security pp. 468-479, DOI 10.1145/2976749.2978395, October 2016, <https://doi.org/10.1145/2976749.2978395>. (CHECKOWAY)

The paper "A Systematic Analysis of the Juniper Dual EC Incident" by Checkoway et al. examines the cryptographic vulnerabilities in Juniper's ScreenOS, particularly focusing on the use of the Dual EC pseudorandom number generator (PRNG). The authors detail how the internal state of the CSPRNG (Cryptographically Secure Pseudorandom Number Generator) can be exploited to predict future outputs, compromising the security of the system. They analyze the implementation flaws that allowed attackers to infer the internal state, leading to potential unauthorized access and data decryption. The study underscores the critical importance of secure PRNG design and implementation in cryptographic systems. 

# Summary of reference from The Debian Project, "openssl -- predictable random number generator", May 2008, <https://www.debian.org/security/2008/dsa-1571>. (DSA-1571-1)

The Debian Security Advisory DSA-1571-1, issued on May 13, 2008, addresses a critical vulnerability in Debian's OpenSSL package, identified as CVE-2008-0166. This flaw resulted from a Debian-specific modification that rendered the random number generator (RNG) predictable, compromising the security of cryptographic key material generated by affected versions. 

**Key Details:**

- **Affected Versions:** OpenSSL versions starting from 0.9.8c-1, introduced into Debian's unstable distribution on September 17, 2006, and subsequently propagated to testing and stable distributions. The older stable distribution (sarge) remained unaffected.

- **Impacted Key Types:** SSH keys, OpenVPN keys, DNSSEC keys, and key material for X.509 certificates and SSL/TLS session keys generated using the compromised OpenSSL versions. Notably, keys produced with GnuPG or GNUTLS were not affected.

- **Recommendations:**
  - **Key Regeneration:** Users were strongly advised to regenerate all cryptographic key material created with the vulnerable OpenSSL versions.
  - **DSA Key Caution:** All DSA keys used for signing or authentication on affected systems should be considered compromised due to the reliance on secret random values during signature generation.

- **Detection and Mitigation:**
  - **Weak Key Detector:** A tool to identify known weak key material was made available.
  - **Key Rollover Instructions:** Guidelines for implementing key rollover for various packages were provided and continuously updated.

Additionally, the advisory addressed two other vulnerabilities:

- **CVE-2007-4995:** A flaw in OpenSSL's Datagram TLS (DTLS) implementation that did not adhere to the DTLS specification, potentially allowing arbitrary code execution.

- **CVE-2007-3108:** A side-channel attack vulnerability in the integer multiplication routines.

To mitigate these issues, users were urged to upgrade their OpenSSL packages to the fixed versions:

- **Stable Distribution (etch):** Version 0.9.8c-4etch3

- **Unstable (sid) and Testing (lenny) Distributions:** Version 0.9.8g-9

Following the upgrade, it was imperative to regenerate any cryptographic material as outlined in the advisory.  

# Summary of reference from DOI 10.1145/359340.359342

The document with DOI 10.1145/359340.359342 is the seminal paper titled "A Method for Obtaining Digital Signatures and Public-Key Cryptosystems" by Rivest, Shamir, and Adleman, published in 1978. This paper introduces the RSA algorithm, a foundational public-key cryptosystem that enables secure data transmission and digital signatures.

In the RSA algorithm, each user generates a pair of keys: a public key for encryption and a private key for decryption. The security of RSA relies on the computational difficulty of factoring large composite numbers, specifically the product of two large prime numbers. This difficulty ensures that, while the public key is available to everyone, only the holder of the private key can decrypt messages encrypted with the public key.

The paper details the mathematical foundations of the RSA algorithm, including key generation, encryption, and decryption processes. It also discusses the algorithm's applications in digital signatures, where the private key is used to sign a message, and the signature can be verified by anyone using the public key, ensuring the message's authenticity and integrity.

The introduction of RSA marked a significant advancement in cryptography, providing a practical method for secure communication over insecure channels and laying the groundwork for modern secure communication protocols. 

# Summary of reference from Boneh.2003

"Remote Timing Attacks Are Practical" is a 2003 paper by David Brumley and Dan Boneh that demonstrates the feasibility of timing attacks on general software systems, specifically targeting OpenSSL. The authors devised an attack capable of extracting private keys from an OpenSSL-based web server within a local network. Their experiments showed that, contrary to prior beliefs, timing attacks are effective against network servers, emphasizing the need for security systems to implement defenses against such vulnerabilities. ([web.stanford.edu](https://web.stanford.edu/~dabo/papers/ssl-timing.pdf?utm_source=openai)) 

# Summary of reference from Weimer, F., "Factoring RSA Keys With TLS Perfect Forward Secrecy", September 2015. (FW15)

In his September 2015 report, "Factoring RSA Keys With TLS Perfect Forward Secrecy," Florian Weimer examines vulnerabilities in TLS servers that implement forward secrecy using RSA keys. The study focuses on the Chinese Remainder Theorem (CRT) optimization in RSA, which, if improperly implemented, can lead to private key exposure.

**Key Findings:**

- **RSA-CRT Vulnerability:** The CRT optimization accelerates RSA operations by performing calculations separately modulo each prime factor of the modulus. However, if a fault occurs during the computation of a signature in one component (e.g., modulo p), and an attacker obtains this faulty signature along with the original message, they can compute the greatest common divisor (GCD) of the difference between the faulty signature raised to the public exponent and the original message with the modulus. This GCD reveals a prime factor of the modulus, effectively compromising the private key. ([people.redhat.com](https://people.redhat.com/~fweimer/rsa-crt-leaks.pdf?utm_source=openai))

- **Empirical Evidence:** Weimer's team conducted extensive TLS handshakes with various servers, identifying several instances where RSA-CRT key leaks occurred. These leaks were traced back to specific hardware and software configurations, notably devices using Cavium hardware and custom versions of OpenSSL. ([redhat.com](https://www.redhat.com/en/blog/factoring-rsa-keys-tls-perfect-forward-secrecy?utm_source=openai))

- **Potential Causes:** The report outlines several factors that can lead to erroneous signatures, including:
  - Use of outdated or vulnerable libraries with flawed integer operations.
  - Race conditions in multithreaded applications.
  - Defective CPU arithmetic units.
  - Corrupted private keys.
  - Errors in CPU caches or main memory. ([redhat.com](https://www.redhat.com/en/blog/factoring-rsa-keys-tls-perfect-forward-secrecy?utm_source=openai))

**Implications:**

An attacker exploiting this vulnerability can impersonate a server by decrypting communications or conducting man-in-the-middle attacks. The attack is particularly concerning because it can be executed offline with minimal computational resources once the faulty signature is obtained. ([redhat.com](https://www.redhat.com/en/blog/factoring-rsa-keys-tls-perfect-forward-secrecy?utm_source=openai))

**Recommendations:**

To mitigate this risk, it's crucial to implement RSA-CRT hardening measures, such as verifying the correctness of computed signatures. Many modern cryptographic libraries have incorporated these safeguards, but some implementations may still lack them. Regular updates and audits of cryptographic software and hardware are essential to ensure the security of TLS communications. ([redhat.com](https://www.redhat.com/en/blog/factoring-rsa-keys-tls-perfect-forward-secrecy?utm_source=openai)) 

# Summary of reference from WHATWG, "Fetch Standard", September 2025, <https://fetch.spec.whatwg.org/>. (FETCH)

In the Fetch Standard, a **network partition key** is a tuple consisting of a site and an optional, implementation-defined value. This key is used to partition network connections and caches, enhancing security and privacy by isolating resources based on their origin.

**Determining the Network Partition Key:**

1. **For an Environment:**
   - Obtain the environment's top-level origin.
   - If the top-level origin is null, use the environment's top-level creation URL's origin.
   - Derive the top-level site from this origin.
   - Set the second key to null or an implementation-defined value.
   - Return the tuple (top-level site, second key).

2. **For a Request:**
   - If the request has a reserved client, determine the network partition key from that client.
   - If the request has a client, determine the network partition key from that client.
   - If neither applies, return null.

**Usage of Network Partition Keys:**

- **Resolving Domains:** When resolving an origin, the network partition key helps in determining the set of IP addresses associated with that origin.
- **Obtaining Connections:** The network partition key is used to obtain appropriate network connections from the user agent's connection pool, ensuring that connections are correctly partitioned.
- **HTTP Cache Partitions:** The HTTP cache is partitioned based on the network partition key, meaning that cached resources are isolated according to their partition key, preventing cross-site data leakage.
- **CORS-Preflight Cache:** The CORS-preflight cache uses the network partition key to store and retrieve cache entries, ensuring that preflight requests are appropriately partitioned.

By utilizing network partition keys, the Fetch Standard aims to enhance security and privacy by isolating network resources and caches based on their origin, thereby mitigating potential cross-origin attacks and data leaks. 

