Let us analyze section 4 of draft draft-ietf-tcpm-accurate-ecn-34. All references made by section 4 have also been included below.

# Draft draft-ietf-tcpm-accurate-ecn-34: Updates: 3168 (if approved)                                 M. Kühlewind

## 1. Introduction

   Explicit Congestion Notification (ECN) [RFC3168] is a mechanism where
   network nodes can mark IP packets instead of dropping them to
   indicate incipient congestion to the endpoints.  Receivers with an
   ECN-capable transport protocol feed back this information to the
   sender.  In RFC 3168, ECN was specified for TCP in such a way that
   only one feedback signal could be transmitted per Round-Trip Time
   (RTT).  This is sufficient for congestion control scheme like Reno
   [RFC6582] and Cubic [RFC9438], as those schemes reduce their
   congestion window by a fixed factor if congestion occurs within an
   RTT independent of the number of received congestion markings.
   Recently, proposed mechanisms like Congestion Exposure (ConEx
   [RFC7713]), DCTCP [RFC8257] or L4S [RFC9330] need to know when more
   than one marking is received in one RTT, which is information that
   cannot be provided by the feedback scheme as specified in [RFC3168].
   This document specifies an update to the ECN feedback scheme of RFC 
   3168 that provides more accurate information and could be used by
   these and potentially other future TCP extensions, while still also
   supporting the pre-existing TCP congestion controllers that use just
   one feedback signal per round.  Congestion control is the term the
   IETF uses to describe data rate management.  It is the algorithm that
   a sender uses to optimize its sending rate so that it transmits data
   as fast as the network can carry it, but no faster.  A fuller
   treatment of the motivation for this specification is given in the
   associated requirements document [RFC7560].

   This document specifies a standards track scheme for ECN feedback in
   the TCP header to provide more than one feedback signal per RTT.  It
   will be called the more Accurate ECN feedback scheme, or AccECN for
   short.  This document updates RFC 3168 with respect to negotiation
   and use of the feedback scheme for TCP.  All aspects of RFC 3168   other than the TCP feedback scheme and its negotiation remain
   unchanged by this specification.  In particular the definition of ECN
   at the IP layer is unaffected.  Section 4 gives a more detailed
   specification of exactly which aspects of RFC 3168 this document
   updates. 
   This document uses the term Classic ECN feedback when it needs to
   distinguish the TCP/ECN feedback scheme defined in [RFC3168] from the
   AccECN TCP feedback scheme.  AccECN is intended to offer a complete
   replacement for Classic TCP/ECN feedback, not a fork in the design of
   TCP.  AccECN feedback complements TCP's loss feedback and it can
   coexist alongside hosts using Classic TCP/ECN feedback.  So its
   applicability is intended to include the public Internet as well as
   private IP network such as data centres (and even any non-IP networks
   over which TCP is used), whether or not any nodes on the path support
   ECN, of whatever flavour.

   AccECN feedback overloads the two existing ECN flags in the TCP
   header and allocates the currently reserved flag (previously called
   NS) in the TCP header, to be used as one three-bit counter field for
   feeding back the number of packets marked as congestion experienced
   (CE).  Given the new definitions of these three bits, both ends have
   to support the new wire protocol before it can be used.  Therefore,
   during the TCP handshake, the two ends use these three bits in the
   TCP header to negotiate the most advanced feedback protocol that they
   can both support, in a way that is backward compatible with
   [RFC3168].

   AccECN is solely a change to the TCP wire protocol; it covers the
   negotiation and signaling of more Accurate ECN feedback from a TCP
   Data Receiver to a Data Sender.  It is completely independent of how
   TCP might respond to congestion feedback, which is out of scope, but
   ultimately the motivation for Accurate ECN feedback.  Like Classic
   ECN feedback, AccECN can be used by standard Reno or CUBIC congestion
   control [RFC5681] [RFC9438] to respond to the existence of at least
   one congestion notification within a round trip.  Or, unlike Reno or
   CUBIC, AccECN can be used to respond to the extent of congestion
   notification over a round trip, as for example DCTCP does in
   controlled environments [RFC8257].  For congestion response, this
   specification refers to the original ECN specificiation adopted in
   2001 [RFC3168], as updated by the more relaxed rules introduced in
   2018 to allow ECN experiments [RFC8311], namely: a TCP-based Low
   Latency Low Loss Scalable (L4S) congestion control [RFC9330]; or
   Alternative Backoff with ECN (ABE) [RFC8511]. Section 5.2 explains how AccECN is compatible with current commonly
   used TCP options, and a number of current experimental modifications
   to TCP, as well as SYN cookies.

### 1.1. Document Roadmap

   The following introductory section outlines the goals of AccECN
   (Section 1.2).  Then, terminology is defined (Section 1.3) and a
   recap of existing prerequisite technology is given (Section 1.4). 



   Section 2 gives an informative overview of the AccECN protocol.  Then Section 3 gives the normative protocol specification, and Section 3.3   collects together requirements for proxies, offload engines and other
   middleboxes.  Section 4 clarifies which aspects of RFC 3168 are
   updated by AccECN.  Section 5 assesses the interaction of AccECN with
   commonly used variants of TCP, whether standardized or not.  Then Section 6 summarizes the features and properties of AccECN. Section 7 summarizes the protocol fields and numbers that IANA will
   need to assign and Section 8 points to the aspects of the protocol
   that will be of interest to the security community. Appendix A  gives pseudocode examples for the various algorithms that
   AccECN uses and Appendix B  explains why AccECN uses flags in the main
   TCP header and quantifies the space left for future use.

### 1.2. Goals

   [RFC7560] enumerates requirements that a candidate feedback scheme
   will need to satisfy, under the headings: resilience, timeliness,
   integrity, accuracy (including ordering and lack of bias),
   complexity, overhead and compatibility (both backward and forward).
   It recognizes that a perfect scheme that fully satisfies all the
   requirements is unlikely and trade-offs between requirements are
   likely.  Section 6 presents the properties of AccECN against these
   requirements and discusses the trade-offs made.

   The requirements document recognizes that a protocol as ubiquitous as
   TCP needs to be able to serve as-yet-unspecified requirements.
   Therefore an AccECN receiver acts as a generic (mechanistic)
   reflector of congestion information with the aim that in future new
   sender behaviours can be deployed unilaterally (see Section 2.5).

### 1.3. Terminology

   AccECN:  The more Accurate ECN feedback scheme will be called AccECN
      for short.

   Classic ECN:  The ECN protocol specified in [RFC3168].

   Classic ECN feedback:  The feedback aspect of the ECN protocol
      specified in [RFC3168], including generation, encoding,
      transmission and decoding of feedback, but not the Data Sender's
      subsequent response to that feedback.

   ACK:  A TCP acknowledgement, with or without a data payload (ACK=1).

   Pure ACK:  A TCP acknowledgement without a data payload. 
   Acceptable packet / segment:  A packet or segment that passes the
      acceptability tests in [RFC9293] and [RFC5961], or that has passed
      other tests with equivalent protection.

   TCP Client:  The TCP stack that originates a connection (the
      initiator).

   TCP Server:  The TCP stack that responds to a connection request (the
      listener).

   Three-way handshake:  The procedure used to establish a TCP
      connection as described in the TCP protocol specification
      [RFC9293].

   Data Receiver:  The endpoint of a TCP half-connection that receives
      data and sends AccECN feedback.

   Data Sender:  The endpoint of a TCP half-connection that sends data
      and receives AccECN feedback.

   In a mild abuse of terminology, this document sometimes refers to
   'TCP packets' instead of 'TCP segments'.

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 
   14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

### 1.4. Recap of Existing ECN feedback in IP/TCP

   Explicit Congestion Notification (ECN) [RFC3168] can be split into
   two parts conceptionally.  In the forward direction, alongside the
   data stream, it uses a two-bit field in the IP header.  This is
   referred to as IP-ECN later on.  This signal carried in the IP (Layer
   3) header is exposed to network devices and may be modified when such
   a device starts to experience congestion (see Table 1).  The second
   part is the feedback mechanism, by which the original data sender is
   notified of the current congestion state of the intermediate path.
   That returned signal is carried in a protocol specific manner, and is
   not to be modified by intermediate network devices.  While ECN is in
   active use for protocols such as QUIC [RFC9000], SCTP [RFC9260], RTP
   [RFC6679] and Remote Direct Memory Access over Converged Ethernet
   [RoCEv2], this document only concerns itself with the specific
   implementation for the TCP protocol. 
   Once ECN has been negotiated for a transport layer connection, the
   Data Sender for either half-connection can set two possible
   codepoints (ECT(0) or ECT(1)) in the IP header of a data packet to
   indicate an ECN-capable transport (ECT).  If the ECN codepoint is
   0b00, the packet is considered to have been sent by a Not ECN-capable
   Transport (Not-ECT).  When a network node experiences congestion, it
   will occasionally either drop or mark a packet, with the choice
   depending on the packet's ECN codepoint.  If the codepoint is Not-
   ECT, only drop is appropriate.  If the codepoint is ECT(0) or ECT(1),
   the node can mark the packet by setting the ECN codepoint to 0b11,
   which is termed 'Congestion Experienced' (CE), or loosely a
   'congestion mark'.  Table 1 summarises these codepoints.

     +==================+================+===========================+
     | IP-ECN codepoint | Codepoint name | Description               |
     +==================+================+===========================+
     | 0b00             | Not-ECT        | Not ECN-Capable Transport |
     +------------------+----------------+---------------------------+
     | 0b01             | ECT(1)         | ECN-Capable Transport (1) |
     +------------------+----------------+---------------------------+
     | 0b10             | ECT(0)         | ECN-Capable Transport (0) |
     +------------------+----------------+---------------------------+
     | 0b11             | CE             | Congestion Experienced    |
     +------------------+----------------+---------------------------+

                  Table 1: The ECN Field in the IP Header

   In the TCP header the first two bits in byte 14 (the TCP header flags
   at bit offsets 8 and 9 labelled Congestion Window Reduced (CWR) and
   Explicit Congestion notification Echo (ECE) in Figure 1) are defined
   as flags for the use of Classic ECN [RFC3168].  A TCP Client
   indicates that it supports Classic ECN feedback by setting (CWR,ECE)
   = (1,1) in the SYN, and an ECN-enabled TCP Server confirms Classic
   ECN support by setting (CWR,ECE) = (0,1) in the SYN/ACK.  On
   reception of a CE-marked packet at the IP layer, the Data Receiver
   for that half-connection starts to set the Echo Congestion
   Experienced (ECE) flag continuously in the TCP header of ACKs, which
   gives the signal resilience to loss or reordering of ACKs.  The Data
   Sender for the same half-connection confirms that it has received at
   least one ECE signal by responding with the congestion window reduced
   (CWR) flag, which allows the Data Receiver to stop repeating the ECN-
   Echo flag.  This always leads to a full RTT of ACKs with ECE set.
   Thus Classic ECN cannot feed back any additional CE markings arriving
   within this RTT.

   The last bit in byte 13 of the TCP header (the TCP header flag at bit
   offset 7 in Figure 1) was defined as the Nonce Sum (NS) for the ECN
   Nonce [RFC3540].  In the absence of widespread deployment RFC 3540




   has been reclassified as historic [RFC8311] and the respective flag
   has been marked as "reserved", making this TCP flag available for use
   by AccECN instead.


       0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     |               |           | N | C | E | U | A | P | R | S | F |
     | Header Length | Reserved  | S | W | C | R | C | S | S | Y | I |
     |               |           |   | R | E | G | K | H | T | N | N |
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

      Figure 1: TCP header flags as defined before the Nonce Sum flag
                            reverted to Reserved

## 2. AccECN Protocol Overview and Rationale

   This section provides an informative overview of the AccECN protocol
   that will be normatively specified in Section 3   Like the general TCP approach, the Data Receiver of each TCP half-
   connection sends AccECN feedback to the Data Sender on TCP
   acknowledgements, reusing data packets of the other half-connection
   whenever possible.

   The AccECN protocol has had to be designed in two parts:

   *  an essential feedback part that re-uses the TCP-ECN header bits
      for the Data Receiver to feed back the number of packets arriving
      with CE in the IP-ECN field.  This provides more accuracy than
      Classic ECN feedback, but limited resilience against ACK loss;

   *  a supplementary feedback part using one of two new alternative
      AccECN TCP options that provide additional feedback on the number
      of payload bytes that arrive marked with each of the three ECN
      codepoints in the IP-ECN field (not just CE marks).  See the BCP
      on Byte and Packet Congestion Notification [RFC7141] for the
      rationale determining that conveying congested payload bytes
      should be preferred over just providing feedback about congested
      packets.  This also provides greater resilience against ACK loss
      than the essential feedback, but it is currently more likely to
      suffer from middlebox interference.

   The two part design was necessary, given limitations on the space
   available for TCP options and given the possibility that certain
   incorrectly designed middleboxes might prevent TCP using any new
   options. 
   The essential feedback part overloads the previous definition of the
   three flags in the TCP header that had been assigned for use by
   Classic ECN.  This design choice deliberately allows AccECN peers to
   replace the Classic ECN feedback protocol, rather than leaving
   Classic ECN feedback intact and adding more accurate feedback
   separately because:

   *  this efficiently reuses scarce TCP header space, given TCP option
      space is approaching saturation;

   *  a single upgrade path for the TCP protocol is preferable to a fork
      in the design which modifies the TCP header to convey all ECN
      feedback;

   *  otherwise Classic and Accurate ECN feedback could give conflicting
      feedback about the same segment, which could open up new security
      concerns and make implementations unnecessarily complex;

   *  middleboxes are more likely to faithfully forward the TCP ECN
      flags than newly defined areas of the TCP header.

   AccECN is designed to work even if the supplementary feedback part is
   removed or zeroed out, as long as the essential feedback part gets
   through.

### 2.1. Capability Negotiation

   AccECN is a change to the wire protocol of the main TCP header,
   therefore it can only be used if both endpoints have been upgraded to
   understand it.  The TCP Client signals support for AccECN on the
   initial SYN of a connection and the TCP Server signals whether it
   supports AccECN on the SYN/ACK.  The TCP flags on the SYN that the
   TCP Client uses to signal AccECN support have been carefully chosen
   so that a TCP Server will interpret them as a request to support the
   most recent variant of ECN feedback that it supports.  Then the TCP
   Client falls back to the same variant of ECN feedback.

   An AccECN TCP Client does not send an AccECN Option on the SYN as SYN
   option space is limited.  The TCP Server sends an AccECN Option on
   the SYN/ACK and the TCP Client sends one on the first ACK to test
   whether the network path forwards these options correctly. 





### 2.2. Feedback Mechanism

   A Data Receiver maintains four counters initialized at the start of
   the half-connection.  Three count the number of arriving payload
   bytes marked CE, ECT(1) and ECT(0) in the IP-ECN field.  These byte
   counters reflect only the TCP payload length, excluding the TCP
   header and TCP options.  The fourth counter counts the number of
   packets arriving marked with a CE codepoint (including control
   packets without payload if they are CE-marked).

   The Data Sender maintains four equivalent counters for the half
   connection, and the AccECN protocol is designed to ensure they will
   match the values in the Data Receiver's counters, albeit after a
   little delay.

   Each ACK carries the three least significant bits (LSBs) of the
   packet-based CE counter using the ECN bits in the TCP header, now
   renamed the Accurate ECN (ACE) field (see Figure 3 later).  The 24
   LSBs of some or all of the byte counters can be optionally carried in
   an AccECN Option.  For efficient use of limited option space, two
   alternative forms of AccECN Option are specified with the fields in
   the opposite order to each other.

### 2.3. Delayed ACKs and Resilience Against ACK Loss

   With both the ACE and the AccECN Option mechanisms, the Data Receiver
   continually repeats the current LSBs of each of its respective
   counters.  There is no need to acknowledge these continually repeated
   counters, so the congestion window reduced (CWR) mechanism of
   [RFC3168] is no longer used.  Even if some ACKs are lost, the Data
   Sender ought to be able to infer how much to increment its own
   counters, even if the protocol field has wrapped.

   The 3-bit ACE field can wrap fairly frequently.  Therefore, even if
   it appears to have incremented by one (say), the field might have
   actually cycled completely then incremented by one.  The Data
   Receiver is not allowed to delay sending an ACK to such an extent
   that the ACE field would cycle.  However ACKs received at the Data
   Sender could still cycle because a whole sequence of ACKs carrying
   intervening values of the field might all be lost or delayed in
   transit.

   The fields in an AccECN Option are larger, but they will increment in
   larger steps because they count bytes not packets.  Nonetheless,
   their size has been chosen such that a whole cycle of the field would
   never occur between ACKs unless there had been an infeasibly long
   sequence of ACK losses.  Therefore, provided that an AccECN Option is
   available, it can be treated as a dependable feedback channel. 
   If an AccECN Option is not available, e.g., it is being stripped by a
   middlebox, the AccECN protocol will only feed back information on CE
   markings (using the ACE field).  Although not ideal, this will be
   sufficient, because it is envisaged that neither ECT(0) nor ECT(1)
   will ever indicate more severe congestion than CE, even though future
   uses for ECT(0) or ECT(1) are still unclear [RFC8311].  Because the
   3-bit ACE field is so small, when it is the only field available, the
   Data Sender has to interpret it assuming the most likely wrap, but
   with a degree of conservatism.

   Certain specified events trigger the Data Receiver to include an
   AccECN Option on an ACK.  The rules are designed to ensure that the
   order in which different markings arrive at the receiver is
   communicated to the sender (as long as options are reaching the
   sender and as long as there is no ACK loss).  Implementations are
   encouraged to send an AccECN Option more frequently, but this is left
   up to the implementer.

### 2.4. Feedback Metrics

   The CE packet counter in the ACE field and the CE byte counter in
   AccECN Options both provide feedback on received CE-marks.  The CE
   packet counter includes control packets that do not have payload
   data, while the CE byte counter solely includes marked payload bytes.
   If both are present, the byte counter in an AccECN Option will
   provide the more accurate information needed for modern congestion
   control and policing schemes, such as L4S, DCTCP or ConEx.  If AccECN
   Options are stripped, a simple algorithm to estimate the number of
   marked bytes from the ACE field is given in Appendix A.3.

   The AccECN design has been generalized so that it ought to be able to
   support possible future uses of the experimental ECT(1) codepoint
   other than the L4S experiment [RFC9330], such as a lower severity or
   a more instant congestion signal than CE.

   Feedback in bytes is provided to protect against the receiver or a
   middlebox using attacks similar to 'ACK-Division' to artificially
   inflate the congestion window, which is why [RFC5681] now recommends
   that TCP counts acknowledged bytes not packets.

### 2.5. Generic (Mechanistic) Reflector

   The ACE field provides feedback about CE markings in the IP-ECN field
   of both data and control packets.  According to [RFC3168] the Data
   Sender is meant to set the IP-ECN field of control packets to Not-
   ECT.  However, mechanisms in certain private networks (e.g., data
   centres) set control packets to be ECN capable because they are
   precisely the packets that performance depends on most. 
   For this reason, AccECN is designed to be a generic reflector of
   whatever ECN markings it sees, whether or not they are compliant with
   a current standard.  Then as standards evolve, Data Senders can
   upgrade unilaterally without any need for receivers to upgrade too.

   It is also useful to be able to rely on generic reflection behaviour
   when senders need to test for unexpected interference with markings
   (for instance Section 3.2.2.3, Section 3.2.2.4 and Section 3.2.3.2 of
   the present document and paragraph 2 of Section 20.2 of [RFC3168]).

   The initial SYN and SYN/ACK are the most critical control packets, so
   AccECN feeds back their IP-ECN fields.  Although RFC 3168 prohibits
   ECN-capable SYNs and SYN/ACKs, providing feedback of ECN marking on
   the SYN and SYN/ACK supports future scenarios in which SYNs might be
   ECN-enabled (without prejudging whether they ought to be).  For
   instance, [RFC8311] updates this aspect of RFC 3168 to allow
   experimentation with ECN-capable TCP control packets.

   Even if the TCP Client (or Server) has set the SYN (or SYN/ACK) to
   not-ECT in compliance with RFC 3168, feedback on the state of the IP-
   ECN field when it arrives at the receiver could still be useful,
   because middleboxes have been known to overwrite the IP-ECN field as
   if it is still part of the old Type of Service (ToS) field
   [Mandalari18].  For example, if a TCP Client has set the SYN to Not-
   ECT, but receives feedback that the IP-ECN field on the SYN arrived
   with a different codepoint, it can detect such middlebox
   interference.  Previously, neither end knew what IP-ECN field the
   other had sent.  So, if a TCP Server received ECT or CE on a SYN, it
   could not know whether it was invalid because only the TCP Client
   knew whether it originally marked the SYN as Not-ECT (or ECT).
   Therefore, prior to AccECN, the Server's only safe course of action
   in this example was to disable ECN for the connection.  Instead, the
   AccECN protocol allows the Server and Client to feed back the ECN
   field received on the SYN and SYN/ACK to their peer, which then has
   all the information to decide whether the connection has to fall-back
   from supporting ECN (or not).

## 3. AccECN Protocol Specification





### 3.1. Negotiating to use AccECN






#### 3.1.1. Negotiation during the TCP three-way handshake

   Given the ECN Nonce [RFC3540] has been reclassified as historic
   [RFC8311], the TCP flag that was previously called NS (Nonce Sum) is
   renamed as the AE (Accurate ECN) flag (the TCP header flag at bit
   offset 7 in Figure 2).  See the IANA Considerations in Section 7. 
       0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     |               |           | A | C | E | U | A | P | R | S | F |
     | Header Length | Reserved  | E | W | C | R | C | S | S | Y | I |
     |               |           |   | R | E | G | K | H | T | N | N |
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

      Figure 2: The new definition of the TCP header flags during the
                          TCP three-way handshake

   During the TCP three-way handshake at the start of a connection, to
   request more Accurate ECN feedback the TCP Client (host A) MUST set
   the TCP flags (AE,CWR,ECE) = (1,1,1) in the initial SYN segment.

   If a TCP Server (host B) that is AccECN-enabled receives a SYN with
   the above three flags set, it MUST set both its half connections into
   AccECN mode.  Then it MUST set the AE, CWR and ECE TCP flags on the
   SYN/ACK to the combination in the top block of Table 2 that feeds
   back the IP-ECN field that arrived on the SYN.  This applies whether
   or not the Server itself supports setting the IP-ECN field on a SYN
   or SYN/ACK (see Section 2.5 for rationale).

   When the TCP Server returns any of the 4 combinations in the top
   block of Table 2, it confirms that it supports AccECN.  The TCP
   Server MUST NOT set one of these 4 combination of flags on the SYN/
   ACK unless the preceding SYN requested support for AccECN as above.

   Once a TCP Client (A) has sent the above SYN to declare that it
   supports AccECN, and once it has received the above SYN/ACK segment
   that confirms that the TCP Server supports AccECN, the TCP Client
   MUST set both its half connections into AccECN mode.  The TCP Client
   MUST NOT enter AccECN mode (or any feedback mode) before it has
   received the first SYN/ACK.

   Once in AccECN mode, a TCP Client or Server has the rights and
   obligations to participate in the ECN protocol defined in Section 3.1.5.

   The procedures to follow for retransmission of SYNs or SYN/ACKs are
   given in Section 3.1.4.

   It is RECOMMENDED that the AccECN protocol is implemented alongside
   Selective Acknowledgement (SACK) [RFC2018].  If SACK is implemented
   with AccECN, Duplicate Selective Acknowledgement (D-SACK) [RFC2883]
   MUST also be implemented. 





#### 3.1.2. Backward Compatibility

   The three flags set to 1 to indicate AccECN support on the SYN have
   been carefully chosen to enable natural fall-back to prior stages in
   the evolution of ECN.  Table 2 tabulates all the negotiation
   possibilities for ECN-related capabilities that involve at least one
   AccECN-capable host.  The entries in the first two columns have been
   abbreviated, as follows:

   AccECN:  Supports more Accurate ECN Feedback (the present
       specification)

   Nonce:  Supports ECN Nonce feedback [RFC3540]

   ECN:  Supports 'Classic' ECN feedback [RFC3168]

   No ECN:  Not ECN-capable.  Implicit congestion notification using
       packet drop.

   +========+========+============+============+======================+
   | Host A | Host B |    SYN     |  SYN/ACK   | Feedback Mode        |
   |        |        |    A->B    |    B->A    | of Host A            |
   |        |        | AE CWR ECE | AE CWR ECE |                      |
   +========+========+============+============+======================+
   | AccECN | AccECN | 1   1   1  | 0   1   0  | AccECN (Not-ECT SYN) |
   | AccECN | AccECN | 1   1   1  | 0   1   1  | AccECN (ECT1 on SYN) |
   | AccECN | AccECN | 1   1   1  | 1   0   0  | AccECN (ECT0 on SYN) |
   | AccECN | AccECN | 1   1   1  | 1   1   0  | AccECN (CE on SYN)   |
   +--------+--------+------------+------------+----------------------+
   +--------+--------+------------+------------+----------------------+
   | AccECN | Nonce  | 1   1   1  | 1   0   1  | (Reserved)           |
   | AccECN | ECN    | 1   1   1  | 0   0   1  | Classic ECN          |
   | AccECN | No ECN | 1   1   1  | 0   0   0  | Not ECN              |
   +--------+--------+------------+------------+----------------------+
   +--------+--------+------------+------------+----------------------+
   | Nonce  | AccECN | 0   1   1  | 0   0   1  | Classic ECN          |
   | ECN    | AccECN | 0   1   1  | 0   0   1  | Classic ECN          |
   | No ECN | AccECN | 0   0   0  | 0   0   0  | Not ECN              |
   +--------+--------+------------+------------+----------------------+
   +--------+--------+------------+------------+----------------------+
   | AccECN | Broken | 1   1   1  | 1   1   1  | Not ECN              |
   +--------+--------+------------+------------+----------------------+

        Table 2: ECN capability negotiation between Client (A) and
                                Server (B)

   Table 2 is divided into blocks each separated by an empty row. 
   1.  The top block shows the case already described in Section 3.1       where both endpoints support AccECN and how the TCP Server (B)
       indicates congestion feedback.

   2.  The second block shows the cases where the TCP Client (A)
       supports AccECN but the TCP Server (B) supports some earlier
       variant of TCP feedback, indicated in its SYN/ACK.  Therefore, as
       soon as an AccECN-capable TCP Client (A) receives the SYN/ACK
       shown it MUST set both its half connections into the feedback
       mode shown in the rightmost column.  If the TCP Client has set
       itself into Classic ECN feedback mode it MUST then comply with
       [RFC3168].

       An AccECN implementation has no need to recognize or support the
       Server response labelled 'Nonce' or ECN Nonce feedback more
       generally [RFC3540], which has been reclassified as historic
       [RFC8311].  AccECN is compatible with alternative ECN feedback
       integrity approaches to the nonce (see Section 5.3).  The SYN/ACK
       labelled 'Nonce' with (AE,CWR,ECE) = (1,0,1) is reserved for
       future use.  A TCP Client (A) that receives such a SYN/ACK
       follows the procedure for forward compatibility given in Section 3.1.3.

   3.  The third block shows the cases where the TCP Server (B) supports
       AccECN but the TCP Client (A) supports some earlier variant of
       TCP feedback, indicated in its SYN.

       When an AccECN-enabled TCP Server (B) receives a SYN with
       (AE,CWR,ECE) = (0,1,1) it MUST do one of the following:

       *  set both its half connections into the Classic ECN feedback
          mode and return a SYN/ACK with (AE,CWR,ECE) = (0,0,1) as
          shown.  Then it MUST comply with [RFC3168].

       *  set both its half-connections into Not ECN mode and return a
          SYN/ACK with (AE,CWR,ECE) = (0,0,0), then continue with ECN
          disabled.  This latter case is unlikely to be desirable, but
          it is allowed as a possibility, e.g., for minimal TCP
          implementations.

       When an AccECN-enabled TCP Server (B) receives a SYN with
       (AE,CWR,ECE) = (0,0,0) it MUST set both its half connections into
       the Not ECN feedback mode, return a SYN/ACK with (AE,CWR,ECE) =
       (0,0,0) as shown and continue with ECN disabled.

   4.  The fourth block displays a combination labelled `Broken'.  Some
       older TCP Server implementations incorrectly set the TCP-ECN
       flags in the SYN/ACK by reflecting those in the SYN.  Such broken 
       TCP Servers (B) cannot support ECN, so as soon as an AccECN-
       capable TCP Client (A) receives such a broken SYN/ACK it MUST
       fall back to Not ECN mode for both its half connections and
       continue with ECN disabled.

   The following additional rules do not fit the structure of the table,
   but they complement it:

   Simultaneous Open:  An originating AccECN Host (A), having sent a SYN
      with (AE,CWR,ECE) = (1,1,1), might receive another SYN from host
      B.  Host A MUST then enter the same feedback mode as it would have
      entered had it been a responding host and received the same SYN.
      Then host A MUST send the same SYN/ACK as it would have sent had
      it been a responding host.

   In-window SYN during TIME-WAIT:  Many TCP implementations create a
      new TCP connection if they receive an in-window SYN packet during
      TIME-WAIT state.  When a TCP host enters TIME-WAIT or CLOSED
      state, it ought to ignore any previous state about the negotiation
      of AccECN for that connection and renegotiate the feedback mode
      according to Table 2.

#### 3.1.3. Forward Compatibility

   If a TCP Server that implements AccECN receives a SYN with the three
   TCP header flags (AE,CWR,ECE) set to any combination other than
   (0,0,0), (0,1,1) or (1,1,1) and it does not have logic specific to
   such a combination, the Server MUST negotiate the use of AccECN as if
   the three flags had been set to (1,1,1).  However, an AccECN Client
   implementation MUST NOT send a SYN with any combination other than
   the three listed.

   If a TCP Client has sent a SYN requesting AccECN feedback with
   (AE,CWR,ECE) = (1,1,1) then receives a SYN/ACK with the currently
   reserved combination (AE,CWR,ECE) = (1,0,1) but it does not have
   logic specific to such a combination, the Client MUST enable AccECN
   mode as if the SYN/ACK confirmed that the Server supported AccECN and
   as if it fed back that the IP-ECN field on the SYN had arrived
   unchanged.  However, an AccECN Server implementation MUST NOT send a
   SYN/ACK with this combination (AE,CWR,ECE) = (1,0,1).

      |  For the avoidance of doubt, the behaviour described in the
      |  present specification applies whether or not the three
      |  remaining reserved TCP header flags are zero. 
   All these requirements ensure that future uses of all the Reserved
   combinations on a SYN or SYN/ACK can rely on consistent behaviour
   from the installed base of AccECN implementations.  See Appendix B.3   for related discussion.

#### 3.1.4. Multiple SYNs or SYN/ACKs






##### 3.1.4.1. Retransmitted SYNs

   If the sender of an AccECN SYN (the TCP Client) times out before
   receiving the SYN/ACK, it SHOULD attempt to negotiate the use of
   AccECN at least one more time by continuing to set all three TCP ECN
   flags (AE,CWR,ECE) = (1,1,1) on the first retransmitted SYN (using
   the usual retransmission time-outs).  If this first retransmission
   also fails to be acknowledged, in deployment scenarios where AccECN
   path traversal might be problematic, the TCP Client SHOULD send
   subsequent retransmissions of the SYN with the three TCP-ECN flags
   cleared (AE,CWR,ECE) = (0,0,0).  Such a retransmitted SYN MUST use
   the same initial sequence number (ISN) as the original SYN.

   Retrying once before fall-back adds delay in the case where a
   middlebox drops an AccECN (or ECN) SYN deliberately.  However, recent
   measurements [Mandalari18] imply that a drop is less likely to be due
   to middlebox interference than other intermittent causes of loss,
   e.g., congestion, wireless transmission loss, etc.

   Implementers MAY use other fall-back strategies if they are found to
   be more effective (e.g., attempting to negotiate AccECN on the SYN
   only once or more than twice (most appropriate during high levels of
   congestion).

   Further it might make sense to also remove any other new or
   experimental fields or options on the SYN in case a middlebox might
   be blocking them, although the required behaviour will depend on the
   specification of the other option(s) and any attempt to co-ordinate
   fall-back between different modules of the stack.  For instance, even
   if taking part in an [RFC8311] experiment that allows ECT on a SYN,
   it would be advisable to try it without.

   Whichever fall-back strategy is used, the TCP initiator SHOULD cache
   failed connection attempts.  If it does, it SHOULD NOT give up
   attempting to negotiate AccECN on the SYN of subsequent connection
   attempts until it is clear that the blockage is persistently and
   specifically due to AccECN.  The cache needs to be arranged to expire
   so that the initiator will infrequently attempt to check whether the
   problem has been resolved. 
   All fall-back strategies will need to follow all the normative rules
   in Section 3.1.5, which concern behaviour when SYNs or SYN/ACKs
   negotiating different types of feedback have been sent within the
   same connection, including the possibility that they arrive out of
   order.  As examples, the following non-normative bullets call out
   those rules from Section 3.1.5 that apply to the above fall-back
   strategies:

   *  Once the TCP Client has sent SYNs with (AE,CWR,ECE) = (1,1,1) and
      with (AE,CWR,ECE) = (0,0,0), it might eventually receive a SYN/ACK
      from the Server in response to one, the other, or both and
      possibly reordered;

   *  Such a TCP Client enters the feedback mode appropriate to the
      first SYN/ACK it receives according to Table 2, and it does not
      switch to a different mode, whatever other SYN/ACKs it might
      receive or send;

   *  If a TCP Client has entered AccECN mode but then subsequently
      sends a SYN or receives a SYN/ACK with (AE,CWR,ECE) = (0,0,0), it
      is still allowed to set ECT on packets for the rest of the
      connection.  Note that this rule is different to that of a Server
      in an equivalent position (Section 3.1.5 explains).

   *  Having entered AccECN mode, in general a TCP Client commits to
      respond to any incoming congestion feedback, whether or not it
      sets ECT on outgoing packets (for rationale and some exceptions
      see Section 3.2.2.3, Section 3.2.2.4);

   *  Having entered AccECN mode, a TCP Client commits to using AccECN
      to feed back the IP-ECN field in incoming packets for the rest of
      the connection, as specified in Section 3.2, even if it is not
      itself setting ECT on outgoing packets.

##### 3.1.4.2. Retransmitted SYN/ACKs

   A TCP Server might send multiple SYN/ACKs indicating different
   feedback modes.  For instance, when falling back to sending a SYN/ACK
   with (AE,CWR,ECE) = (0,0,0) after previous AccECN SYN/ACKs have timed
   out (Section 3.2.3.2.2); or to acknowledge different retransmissions
   of the SYN (Section 3.1.4.1).

   All fall-back strategies will need to follow all the normative rules
   in Section 3.1.5, which concern behaviour when SYNs or SYN/ACKs
   negotiating different types of feedback are sent within the same
   connection, including the possibility that they arrive out of order.
   As examples, the following non-normative bullets call out those rules
   from Section 3.1.5 that apply to the above fall-back strategies:
   *  An AccECN-capable TCP Server enters the feedback mode appropriate
      to the first SYN it receives using Table 2, and it does not switch
      to a different mode, whatever other SYNs it might receive and
      whatever SYN/ACKs it might send;

   *  if a TCP Server in AccECN mode receives a SYN with (AE,CWR,ECE) =
      (0,0,0), it preferably acknowledges it first using an AccECN SYN/
      ACK, but it can retry using a SYN/ACK with (AE,CWR,ECE) = (0,0,0);

   *  If a TCP Server in AccECN mode sends multiple AccECN SYN/ACKs, it
      uses the TCP-ECN flags in each SYN/ACK to feed back the IP-ECN
      field on the latest SYN to have arrived;

   *  If a TCP Server enters AccECN mode then subsequently sends a SYN/
      ACK or receives a SYN with (AE,CWR,ECE) = (0,0,0), it is
      prohibited from setting ECT on any packet for the rest of the
      connection;

   *  Having entered AccECN mode, in general a TCP Server commits to
      respond to any incoming congestion feedback, whether or not it
      sets ECT on outgoing packets (for rationale and some exceptions
      see Section 3.2.2.3, Section 3.2.2.4);

   *  Having entered AccECN mode, a TCP Server commits to using AccECN
      to feed back the IP-ECN field in incoming packets for the rest of
      the connection, as specified in Section 3.2, even if it is not
      itself setting ECT on outgoing packets.

#### 3.1.5. Implications of AccECN Mode



   Section 3.1.1 describes the only ways that a host can enter AccECN
   mode, whether as a Client or as a Server.

   An implementation that supports AccECN has the rights and obligations
   concerning the use of ECN defined below, which update those in Section 6.1.1 of [RFC3168].  This section uses the following
   definitions:

   'During the handshake':  The connection states prior to
      synchronization;

   'Valid SYN':  A SYN that has the same port numbers and the same ISN
      as the SYN that first caused the Server to open the connection.
      An 'Acceptable' packet is defined in Section 1.3.

   Handling SYNs or SYN/ACKs of multiple types (e.g., fall-back):

   *  Any implementation that supports AccECN:
      -  MUST NOT switch into a different feedback mode to the one it
         first entered according to Table 2, no matter whether it
         subsequently receives valid SYNs or Acceptable SYN/ACKs of
         different types.

      -  SHOULD ignore the TCP-ECN flags in SYNs or SYN/ACKs that are
         received after the implementation reaches the Established
         state, in line with the general TCP approach [RFC9293];

         Reason: Reaching established state implies that at least one
         SYN and one SYN/ACK have successfully been delivered.  And all
         the rules for handshake fall-back are designed to work based on
         those packets that successfully traverse the path, whatever
         other handshake packets are lost or delayed.

      -  MUST NOT send a 'Classic' ECN-setup SYN [RFC3168] with
         (AE,CWR,ECE) = (0,1,1) and a SYN with (AE,CWR,ECE) = (1,1,1)
         requesting AccECN feedback within the same connection;

      -  MUST NOT send a 'Classic' ECN-setup SYN/ACK [RFC3168] with
         (AE,CWR,ECE) = (0,0,1) and a SYN/ACK agreeing to use AccECN
         feedback within the same connection;

      -  MUST reset the connection with a RST packet, if it receives a
         'Classic' ECN-setup SYN with (AE,CWR,ECE) = (0,1,1) and a SYN
         requesting AccECN feedback during the same handshake;

      -  MUST reset the connection with a RST packet, if it receives
         'Classic' ECN-setup SYN/ACK with (AE,CWR,ECE) = (0,0,1) and a
         SYN/ACK agreeing to use AccECN feedback during the same
         handshake;

      The last four rules are necessary because, if one peer were to
      negotiate the feedback mode in two different types of handshake,
      it would not be possible for the other peer to know for certain
      which handshake packet(s) the other end had eventually received or
      in which order it received them.  So, in the absence of these
      rules, the two peers could end up using different ECN feedback
      modes without knowing it.

   *  A host in AccECN mode that is feeding back the IP-ECN field on a
      SYN or SYN/ACK:

      -  MUST feed back the IP-ECN field on the latest valid SYN or
         acceptable SYN/ACK to arrive.

   *  A TCP Server already in AccECN mode:
      -  SHOULD acknowledge a valid SYN arriving with (AE,CWR,ECE) =
         (0,0,0) by emitting an AccECN SYN/ACK (with the appropriate
         combination of TCP-ECN flags to feed back the IP-ECN field of
         this latest SYN);

      -  MAY acknowledge a valid SYN arriving with (AE,CWR,ECE) =
         (0,0,0) by sending a SYN/ACK with (AE,CWR,ECE) = (0,0,0);

      Rationale: When a SYN arrives with (AE,CWR,ECE) = (0,0,0) at a TCP
      Server that is already in AccECN mode, it implies that the TCP
      Client had probably not received the previous AccECN SYN/ACK
      emitted by the TCP Server.  Therefore, the first bullet recommends
      attempting at least one more AccECN SYN/ACK.  Nonetheless, the
      second bullet recognizes that the Server might eventually need to
      fall back to a non-ECN SYN/ACK.  In either case, the TCP Server
      remains in AccECN feedback mode (according to the earlier
      requirement not to switch modes).

   *  An AccECN-capable TCP Server already in Not ECN mode:

      -  SHOULD respond to any subsequent valid SYN using a SYN/ACK with
         (AE,CWR,ECE) = (0,0,0), even if the SYN is offering to
         negotiate Classic ECN or AccECN feedback mode;

         Rationale: There would be no point in the Server offering any
         type of ECN feedback, because the Client will not be using ECN.
         However, there is no interoperability reason to make this rule
         mandatory.

   If for any reason a host is not willing to provide ECN feedback on a
   particular TCP connection, it SHOULD clear the AE, CWR and ECE flags
   in all SYN and/or SYN/ACK packets that it sends.

   Sending ECT:

   *  Any implementation that supports AccECN:

      -  MUST NOT set ECT if it is in Not ECN feedback mode.

      A Data Sender in AccECN mode:

      -  SHOULD set an ECT codepoint in the IP header of packets to
         indicate to the network that the transport is capable and
         willing to participate in ECN for this packet;

      -  MAY not set ECT on any packet (for instance if it has reason to
         believe such a packet would be blocked);
      A TCP Server in AccECN mode:

      -  MUST NOT set ECT on any packet for the rest of the connection,
         if it has received or sent at least one valid SYN or Acceptable
         SYN/ACK with (AE,CWR,ECE) = (0,0,0) during the handshake.

         This rule solely applies to a Server because, when a Server
         enters AccECN mode it doesn't know for sure whether the Client
         will end up in AccECN mode.  But when a Client enters AccECN
         mode, it can be certain that the Server is already in AccECN
         feedback mode.

   Congestion response:

   *  A host in AccECN mode:

      -  is obliged to respond appropriately to AccECN feedback that
         indicates there were ECN marks on packets it had previously
         sent, where 'appropriately' is defined in Section 6.1 of
         [RFC3168] and updated by Sections 2.1 and 4.1 of [RFC8311];

      -  is still obliged to respond appropriately to congestion
         feedback, even when it is solely sending non-ECN-capable
         packets (for rationale, some examples and some exceptions see Section 3.2.2.3, Section 3.2.2.4).

      -  is still obliged to respond appropriately to congestion
         feedback, even if it has sent or received a SYN or SYN/ACK
         packet with (AE,CWR,ECE) = (0,0,0) during the handshake;

      -  MUST NOT set CWR to indicate that it has received and responded
         to indications of congestion.

         For the avoidance of doubt, this is unlike an RFC 3168 data
         sender and this does not preclude the Data Sender from setting
         the bits of the ACE counter field, which includes an overloaded
         use of the same bit.

   Receiving ECT:

   *  A host in AccECN mode:

      -  MUST feed back the information in the IP-ECN field of incoming
         packets using Accurate ECN feedback, as specified in Section 3.2. 
         For the avoidance of doubt, this requirement stands even if the
         AccECN host has also sent or received a SYN or SYN/ACK with
         (AE,CWR,ECE) = (0,0,0).  Reason: Such a SYN or SYN/ACK implies
         some form of packet mangling might be present.  Even if the
         remote peer is not setting ECT, it could still be set
         erroneously by packet mangling at the IP layer (see Section 3.2.2.3).  In such cases, the Data Sender is best
         placed to decide whether ECN markings are valid, but it can
         only do that if the Data Receiver mechanistically feeds back
         any ECN markings.  This approach will not lead to TCP Options
         being generated unnecessarily if the recommended simple scheme
         in Section 3.2.3.3 is used, because no byte counters will
         change if no packets are set to ECT.

      -  MUST NOT use reception of packets with ECT set in the IP-ECN
         field as an implicit signal that the peer is ECN-capable.

         Reason: ECT at the IP layer does not explicitly confirm the
         peer has the correct ECN feedback logic, because the packets
         could have been mangled at the IP layer.

### 3.2. AccECN Feedback

   Each Data Receiver of each half connection maintains four counters,
   r.cep, r.ceb, r.e0b and r.e1b:

   *  The Data Receiver MUST increment the CE packet counter (r.cep),
      for every Acceptable packet that it receives with the CE code
      point in the IP ECN field, including CE marked control packets and
      retransmissions but excluding CE on SYN packets (SYN=1; ACK=0).

   *  A Data Receiver that supports sending of AccECN TCP Options MUST
      increment the r.ceb, r.e0b or r.e1b byte counters by the number of
      TCP payload octets in Acceptable packets marked with the CE,
      ECT(0) and ECT(1) codepoint in their IP-ECN field, including any
      payload octets on control packets and retransmissions, but not
      including any payload octets on SYN packets (SYN=1; ACK=0).

   Each Data Sender of each half connection maintains four counters,
   s.cep, s.ceb, s.e0b and s.e1b intended to track the equivalent
   counters at the Data Receiver.

   A Data Receiver feeds back the CE packet counter using the Accurate
   ECN (ACE) field, as explained in Section 3.2.2.  And it optionally
   feeds back all the byte counters using the AccECN TCP Option, as
   specified in Section 3.2.3. 
   Whenever a Data Receiver feeds back the value of any counter, it MUST
   report the most recent value, no matter whether it is in a pure ACK,
   or an ACK piggybacked on a packet used by the other half-connection,
   whether new payload data or a retransmission.  Therefore the feedback
   piggybacked on a retransmitted packet is unlikely to be the same as
   the feedback on the original packet.

#### 3.2.1. Initialization of Feedback Counters

   When a host first enters AccECN mode, in its role as a Data Receiver
   it initializes its counters to r.cep = 5, r.e0b = r.e1b = 1 and r.ceb
   = 0,

   Non-zero initial values are used to support a stateless handshake
   (see Section 5.1) and to be distinct from cases where the fields are
   incorrectly zeroed (e.g., by middleboxes - see Section 3.2.3.2.4).

   When a host enters AccECN mode, in its role as a Data Sender it
   initializes its counters to s.cep = 5, s.e0b = s.e1b = 1 and s.ceb =
   0.

#### 3.2.2. The ACE Field

   After AccECN has been negotiated on the SYN and SYN/ACK, both hosts
   overload the three TCP flags (AE, CWR and ECE) in the main TCP header
   as one 3-bit field.  Then the field is given a new name, ACE, as
   shown in Figure 3.


       0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     |               |           |           | U | A | P | R | S | F |
     | Header Length | Reserved  |    ACE    | R | C | S | S | Y | I |
     |               |           |           | G | K | H | T | N | N |
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

      Figure 3: Definition of the ACE field within bytes 13 and 14 of
        the TCP Header (when AccECN has been negotiated and SYN=0).

   The original definition of these three flags in the TCP header,
   including the addition of support for the ECN Nonce, is shown for
   comparison in Figure 1.  This specification does not rename these
   three TCP flags to ACE unconditionally; it merely overloads them with
   another name and definition once an AccECN connection has been
   established. 
   With one exception (Section 3.2.2.1), a host with both of its half-
   connections in AccECN mode MUST interpret the AE, CWR and ECE flags
   as the 3-bit ACE counter on a segment with the SYN flag cleared
   (SYN=0).  On such a packet, a Data Receiver MUST encode the three
   least significant bits of its r.cep counter into the ACE field that
   it feeds back to the Data Sender.  The least significant bit is at
   bit offset 9 in Figure 3.  A host MUST NOT interpret the 3 flags as a
   3-bit ACE field on any segment with SYN=1 (whether ACK is 0 or 1), or
   if AccECN negotiation is incomplete or has not succeeded.

   Both parts of each of these conditions are equally important.  For
   instance, even if AccECN negotiation has been successful, the ACE
   field is not defined on any segments with SYN=1 (e.g., a
   retransmission of an unacknowledged SYN/ACK, or when both ends send
   SYN/ACKs after AccECN support has been successfully negotiated during
   a simultaneous open).

##### 3.2.2.1. ACE Field on the ACK of the SYN/ACK

   A TCP Client (A) in AccECN mode MUST feed back which of the 4
   possible values of the IP-ECN field was on the SYN/ACK by writing it
   into the ACE field of a pure ACK with no SACK blocks using the binary
   encoding in Table 3 (which is the same as that used on the SYN/ACK in
   Table 2).  This shall be called the handshake encoding of the ACE
   field, and it is the only exception to the rule that the ACE field
   carries the 3 least significant bits of the r.cep counter on packets
   with SYN=0.

   Normally, a TCP Client acknowledges a SYN/ACK with an ACK that
   satisfies the above conditions anyway (SYN=0, no data, no SACK
   blocks).  If an AccECN TCP Client intends to acknowledge the SYN/ACK
   with a packet that does not satisfy these conditions (e.g., it has
   data to include on the ACK), it SHOULD first send a pure ACK that
   does satisfy these conditions (see Section 5.2), so that it can feed
   back which of the four values of the IP-ECN field arrived on the SYN/
   ACK.  A valid exception to this "SHOULD" would be where the
   implementation will only be used in an environment where mangling of
   the ECN field is unlikely.

   The TCP Client MUST also use the handshake encoding for the pure ACK
   of any retransmitted SYN/ACK that confirms that the TCP Server
   supports AccECN.  The procedure for the TCP Server to follow if the
   final ACK of the handshake does not arrive before its retransmission
   timer expires is given in Section 3.1.4.2. 
        +==================+================+=====================+
        | IP-ECN codepoint | ACE on pure    | r.cep of TCP Client |
        | on SYN/ACK       | ACK of SYN/ACK | in AccECN mode      |
        +==================+================+=====================+
        | Not-ECT          | 0b010          | 5                   |
        +------------------+----------------+---------------------+
        | ECT(1)           | 0b011          | 5                   |
        +------------------+----------------+---------------------+
        | ECT(0)           | 0b100          | 5                   |
        +------------------+----------------+---------------------+
        | CE               | 0b110          | 6                   |
        +------------------+----------------+---------------------+

            Table 3: The encoding of the ACE field in the ACK of
             the SYN-ACK to reflect the SYN-ACK's IP-ECN field

   When an AccECN Server in SYN-RCVD state receives a pure ACK with
   SYN=0 and no SACK blocks, instead of treating the ACE field as a
   counter, it MUST infer the meaning of each possible value of the ACE
   field from Table 4, which also shows the value that an AccECN Server
   MUST set s.cep to as a result.

   Given this encoding of the ACE field on the ACK of a SYN/ACK is
   exceptional, an AccECN Server using large receive offload (LRO) might
   prefer to disable LRO until such an ACK has transitioned it out of
   SYN-RCVD state. 
      +============+==========================+=====================+
      | ACE on ACK | IP-ECN codepoint on SYN/ | s.cep of TCP Server |
      | of SYN/ACK | ACK inferred by Server   | in AccECN mode      |
      +============+==========================+=====================+
      | 0b000      | {Notes 1, 3}             | Disable s.cep       |
      +------------+--------------------------+---------------------+
      | 0b001      | {Notes 2, 3}             | 5                   |
      +------------+--------------------------+---------------------+
      | 0b010      | Not-ECT                  | 5                   |
      +------------+--------------------------+---------------------+
      | 0b011      | ECT(1)                   | 5                   |
      +------------+--------------------------+---------------------+
      | 0b100      | ECT(0)                   | 5                   |
      +------------+--------------------------+---------------------+
      | 0b101      | Currently Unused {Note   | 5                   |
      |            | 2}                       |                     |
      +------------+--------------------------+---------------------+
      | 0b110      | CE                       | 6                   |
      +------------+--------------------------+---------------------+
      | 0b111      | Currently Unused {Note   | 5                   |
      |            | 2}                       |                     |
      +------------+--------------------------+---------------------+

        Table 4: Meaning of the ACE field on the ACK of the SYN/ACK

   {Note 1}: If the Server is in AccECN mode and in SYN-RCVD state, and
   if it receives a value of zero on a pure ACK with SYN=0 and no SACK
   blocks, for the rest of the connection the Server MUST NOT set ECT on
   outgoing packets and MUST NOT respond to AccECN feedback.
   Nonetheless, as a Data Receiver it MUST NOT disable AccECN feedback.

   Any of the circumstances below could cause a value of zero but,
   whatever the cause, the actions above would be the appropriate
   response:

   *  The TCP Client has somehow entered No ECN feedback mode (most
      likely if the Server received a SYN or sent a SYN/ACK with
      (AE,CWR,ECE) = (0,0,0) after entering AccECN mode, but possible
      even if it didn't);

   *  The TCP Client genuinely might be in AccECN mode, but its count of
      received CE marks might have caused the ACE field to wrap to zero.
      This is highly unlikely, but not impossible because the Server
      might have already sent multiple packets while still in SYN-RCVD
      state, e.g., using TFO (see Section 5.2) and some might have been
      CE-marked.  Then ACE on the first ACK seen by the Server might be
      zero, due to previous ACKs experiencing an unfortunate pattern of
      loss or delay. 
   *  Some form of non-compliance at the TCP Client or on the path (see Section 3.2.2.4).

   {Note 2}: If the Server is in AccECN mode, these values are Currently
   Unused but the AccECN Server's behaviour is still defined for forward
   compatibility.  Then the designer of a future protocol can know for
   certain what AccECN Servers will do with these codepoints.

   {Note 3}: In the case where a Server that implements AccECN is also
   using a stateless handshake (termed a SYN cookie) it will not
   remember whether it entered AccECN mode.  The values 0b000 or 0b001
   will remind it that it did not enter AccECN mode, because AccECN does
   not use them (see Section 5.1 for details).  If a Server that uses a
   stateless handshake and implements AccECN receives either of these
   two values in the ACK, its action is implementation-dependent and
   outside the scope of this document.  It will certainly not take the
   action in the third column because, after it receives either of these
   values, it is not in AccECN mode.  In example, it will not disable
   ECN (at least not just because ACE is 0b000) and it will not set
   s.cep.

##### 3.2.2.2. Encoding and Decoding Feedback in the ACE Field

   Whenever the Data Receiver sends an ACK with SYN=0 (with or without
   data), unless the handshake encoding in Section 3.2.2.1 applies, the
   Data Receiver MUST encode the least significant 3 bits of its r.cep
   counter into the ACE field (see Appendix A.2).

   Whenever the Data Sender receives an ACK with SYN=0 (with or without
   data), it first checks whether it has already been superseded
   (defined in Appendix A.1) by another ACK in which case it ignores the
   ECN feedback.  If the ACK has not been superseded, and if the special
   handshake encoding in Section 3.2.2.1 does not apply, the Data Sender
   decodes the ACE field as follows (see Appendix A.2 for examples).

   *  It takes the least significant 3 bits of its local s.cep counter
      and subtracts them from the incoming ACE counter to work out the
      minimum positive increment it could apply to s.cep (assuming the
      ACE field only wrapped at most once).

   *  It then follows the safety procedures in Section 3.2.2.5.2 to
      calculate or estimate how many packets the ACK could have
      acknowledged under the prevailing conditions to determine whether
      the ACE field might have wrapped more than once.

   The encode/decode procedures during the three-way handshake are
   exceptions to the general rules given so far, so they are spelled out
   step by step below for clarity:
   *  If a TCP Server in AccECN mode receives a CE mark in the IP-ECN
      field of a SYN (SYN=1, ACK=0), it MUST NOT increment r.cep (it
      remains at its initial value of 5).

      Reason: It would be redundant for the Server to include CE-marked
      SYNs in its r.cep counter, because it already reliably delivers
      feedback of any CE marking using the encoding in the top block of
      Table 2 in the SYN/ACK.  This also ensures that, when the Server
      starts using the ACE field, it has not unnecessarily consumed more
      than one initial value, given they can be used to negotiate
      variants of the AccECN protocol (see Appendix B.3).

   *  If a TCP Client in AccECN mode receives CE feedback in the TCP
      flags of a SYN/ACK, it MUST NOT increment s.cep (it remains at its
      initial value of 5), so that it stays in step with r.cep on the
      Server.  Nonetheless, the TCP Client still triggers the congestion
      control actions necessary to respond to the CE feedback.

   *  If a TCP Client in AccECN mode receives a CE mark in the IP-ECN
      field of a SYN/ACK, it MUST increment r.cep, but no more than once
      no matter how many CE-marked SYN/ACKs it receives
      (i.e., incremented from 5 to 6, but no further).

      Reason: Incrementing r.cep ensures the Client will eventually
      deliver any CE marking to the Server reliably when it starts using
      the ACE field.  Even though the Client also feeds back any CE
      marking on the ACK of the SYN/ACK using the encoding in Table 3,
      this ACK is not delivered reliably, so it can be considered as a
      timely notification that is redundant but unreliable.  The Client
      does not increment r.cep more than once, because the Server can
      only increment s.cep once (see next bullet).  Also, this limits
      the unnecessarily consumed initial values of the ACE field to two.

   *  If a TCP Server in AccECN mode and in SYN-RCVD state receives CE
      feedback in the TCP flags of a pure ACK with no SACK blocks, it
      MUST increment s.cep (from 5 to 6).  The TCP Server then triggers
      the congestion control actions necessary to respond to the CE
      feedback.

      Reasoning: The TCP Server can only increment s.cep once, because
      the first ACK it receives will cause it to transition out of SYN-
      RCVD state.  The Server's congestion response would be no
      different even if it could receive feedback of more than one CE-
      marked SYN/ACK.

      Once the TCP Server transitions to ESTABLISHED state, it might
      later receive other pure ACK(s) with the handshake encoding in the
      ACE field.  A Server MAY implement a test for such a case, but it 
      is not required.  Therefore, once in the ESTABLISHED state, it
      will be sufficient for the Server to consider the ACE field to be
      encoded as the normal ACE counter on all packets with SYN=0.

      Reasoning: Such ACKs will be quite unusual, e.g., a SYN/ACK (or
      ACK of the SYN/ACK) that is delayed for longer than the Server's
      retransmission timeout; or packet duplication by the network.  And
      the impact of any error in the feedback on such ACKs will only be
      temporary.

##### 3.2.2.3. Testing for Mangling of the IP/ECN Field

   *  TCP Client side:

      The value of the TCP-ECN flags on the SYN/ACK indicates the value
      of the IP-ECN field when the SYN arrived at the Server.  The TCP
      Client can compare this with how it originally set the IP-ECN
      field on the SYN.  If this comparison implies an invalid
      transition (defined below) of the IP-ECN field, for the remainder
      of the half-connection the Client is advised to send non-ECN-
      capable packets, but it still ought to respond to any feedback of
      CE markings (explained below).  However, the TCP Client MUST
      remain in the AccECN feedback mode and it MUST continue to feed
      back any ECN markings on arriving packets (in its role as Data
      Receiver).

   *  TCP Server side:

      The value of the ACE field on the last ACK of the three-way
      handshake indicates the value of the IP-ECN field when the SYN/ACK
      arrived at the TCP Client.  The Server can compare this with how
      it originally set the IP-ECN field on the SYN/ACK.  If this
      comparison implies an invalid transition of the IP-ECN field, for
      the remainder of the half-connection the Server is advised to send
      non-ECN-capable packets, but it still ought to respond to any
      feedback of CE markings (explained below).  However, the Server
      MUST remain in the AccECN feedback mode and it MUST continue to
      feed back any ECN markings on arriving packets (in its role as
      Data Receiver).

   If a Data Sender in AccECN mode starts sending non-ECN-capable
   packets because it has detected mangling, it is still advised to
   respond to CE feedback.  Reason: any CE-marking arriving at the Data
   Receiver could be due to something early in the path mangling the
   non-ECN-capable IP-ECN field into an ECN-capable codepoint and then,
   later in the path, a network bottleneck might be applying CE-markings
   to indicate genuine congestion.  This argument applies whether the
   handshake packet originally sent by the TCP Client or Server was non-
   ECN-capable or ECN-capable because, in either case, an unsafe
   transition could imply that non-ECN-capable packets later in the
   connection might get mangled.

   Once a Data Sender has entered AccECN mode it is advised to check
   whether it is receiving continuous feedback of CE.  Specifying
   exactly how to do this is beyond the scope of the present
   specification, but the sender might check whether the feedback for
   every packet it sends for the first three or four rounds indicates
   CE-marking.  If continuous CE-marking is detected, for the remainder
   of the half-connection, the Data Sender ought to send non-ECN-capable
   packets and it is advised not to respond to any feedback of CE
   markings.  The Data Sender might occasionally test whether it can
   resume sending ECN-capable packets.

   The above advice on switching to sending non-ECN-capable packets but
   still responding to CE-markings unless they become continuous is not
   stated normatively (in capitals), because the best strategy might
   depend on experience of the most likely types of mangling, which can
   only be known at the time of deployment.  The same is true for other
   forms of mangling (or resumption of expected marking) during later
   stages of a connection.

   As always, once a host has entered AccECN mode, it follows the
   general mandatory requirements (Section 3.1.5) to remain in the same
   feedback mode and to continue feeding back any ECN markings on
   arriving packets using AccECN feedback.  This follows the general
   approach where an AccECN Data Receiver mechanistically reflects
   whatever it receives (Section 2.5).

   The ACK of the SYN/ACK is not reliably delivered (nonetheless, the
   count of CE marks is still eventually delivered reliably).  If this
   ACK does not arrive, the Server is advised to continue to send ECN-
   capable packets without having tested for mangling of the IP-ECN
   field on the SYN/ACK.

   All the fall-back behaviours in this section are necessary in case
   mangling of the IP-ECN field is asymmetric, which is currently common
   over some mobile networks [Mandalari18].  Then one end might see no
   unsafe transition and continue sending ECN-capable packets, while the
   other end sees an unsafe transition and stops sending ECN-capable
   packets.

   Invalid transitions of the IP-ECN field are defined in section 18 of
   the Classic ECN specification [RFC3168] and repeated here for
   convenience:

   *  the not-ECT codepoint changes;
   *  either ECT codepoint transitions to not-ECT;

   *  the CE codepoint changes. RFC 3168 says that a router that changes ECT to not-ECT is invalid
   but safe.  However, from a host's viewpoint, this transition is
   unsafe because it could be the result of two transitions at different
   routers on the path: ECT to CE (safe) then CE to not-ECT (unsafe).
   This scenario could well happen where an ECN-enabled home router
   congests its upstream mobile broadband bottleneck link, then the
   ingress to the mobile network clears the ECN field [Mandalari18].

##### 3.2.2.4. Testing for Zeroing of the ACE Field



   Section 3.2.2 required the Data Receiver to initialize the r.cep
   counter to a non-zero value.  Therefore, in either direction the
   initial value of the ACE counter ought to be non-zero.

   This section does not concern the case where the ACE field is zero
   when the handshake encoding has been used on the ACK of the SYN/ACK
   under the carefully worded conditions in Section 3.2.2.1.

   If AccECN has been successfully negotiated, the Data Sender MAY check
   the value of the ACE counter in the first feedback packet (with or
   without data) that arrives after the three-way handshake.  If the
   value of this ACE field is found to be zero (0b000), for the
   remainder of the half-connection the Data Sender ought to send non-
   ECN-capable packets and it is advised not to respond to any feedback
   of CE markings.

   Reason: the symptoms imply any or all of the following:

   *  the remote peer has somehow entered Not ECN feedback mode;

   *  a broken remote TCP implementation;

   *  potential mangling of the ECN fields in the TCP headers (although
      unlikely given they clearly survived during the handshake).

   This advice is not stated normatively (in capitals), because the best
   strategy might depend on experience of the most likely scenarios,
   which can only be known at the time of deployment.

   Note that a host in AccECN mode MUST continue to provide Accurate ECN
   feedback to its peer, even if it is no longer sending ECT itself over
   the other half connection. 
   If reordering occurs, the first feedback packet that arrives will not
   necessarily be the same as the first packet in sequence order.  The
   test has been specified loosely like this to simplify implementation,
   and because it would not have been any more precise to have specified
   the first packet in sequence order, which would not necessarily be
   the first ACE counter that the Data Receiver fed back anyway, given
   it might have been a retransmission.

   The possibility of re-ordering means that there is a small chance
   that the ACE field on the first packet to arrive is genuinely zero
   (without middlebox interference).  This would cause a host to
   unnecessarily disable ECN for a half connection.  Therefore, in
   environments where there is no evidence of the ACE field being
   zeroed, implementations MAY skip this test.

   Note that the Data Sender MUST NOT test whether the arriving counter
   in the initial ACE field has been initialized to a specific valid
   value - the above check solely tests whether the ACE fields have been
   incorrectly zeroed.  This allows hosts to use different initial
   values as an additional signalling channel in future.

##### 3.2.2.5. Safety against Ambiguity of the ACE Field

   If too many CE-marked segments are acknowledged at once, or if a long
   run of ACKs is lost or thinned out, the 3-bit counter in the ACE
   field might have cycled between two ACKs arriving at the Data Sender.
   The following safety procedures minimize this ambiguity.

###### 3.2.2.5.1. Packet Receiver Safety Procedures

   The following rules define when the receiver of a packet in AccECN
   mode emits an ACK:

   Change-Triggered ACKs:  An AccECN Data Receiver SHOULD emit an ACK
      whenever a data packet marked CE arrives after the previous packet
      was not CE.

      Even though this rule is stated as a "SHOULD", it is important for
      a transition to trigger an ACK if at all possible, The only valid
      exception to this rule is given below these bullets.

      For the avoidance of doubt, this rule is deliberately worded to
      apply solely when _data_ packets arrive, but the comparison with
      the previous packet includes any packet, not just data packets.

   Increment-Triggered ACKs:  An AccECN receiver of a packet MUST emit 
      an ACK if 'n' CE marks have arrived since the previous ACK.  If
      there is unacknowledged data at the receiver, 'n' SHOULD be 2.  If
      there is no unacknowledged data at the receiver, 'n' SHOULD be 3
      and MUST be no less than 3.  In either case, 'n' MUST be no
      greater than 7.

   The above rules for when to send an ACK are designed to be
   complemented by those in Section 3.2.3.3, which concern whether an
   AccECN TCP Option ought to be included on ACKs.

   If the arrivals of a number of data packets are all processed as one
   event, e.g., using large receive offload (LRO) or generic receive
   offload (GRO), both the above rules SHOULD be interpreted as
   requiring multiple ACKs to be emitted back-to-back (for each
   transition and for each sequence of 'n' CE marks).  If this is
   problematic for high performance, either rule can be interpreted as
   requiring just a single ACK at the end of the whole receive event.

   Even if a number of data packets do not arrive as one event, the
   'Change-Triggered ACKs' rule could sometimes cause the ACK rate to be
   problematic for high performance (although high performance protocols
   such as DCTCP already successfully use change-triggered ACKs).  The
   rationale for change-triggered ACKs is so that the Data Sender can
   rely on them to detect queue growth as soon as possible, particularly
   at the start of a flow.  The approach can lead to some additional
   ACKs but it feeds back the timing and the order in which ECN marks
   are received with minimal additional complexity.  If CE marks are
   infrequent, as is the case for most Active Queue Managment (AQM)
   packet schedulers at the time of writing, or there are multiple marks
   in a row, the additional load will be low.  However, marking patterns
   with numerous non-contiguous CE marks could increase the load
   significantly.  One possible compromise would be for the receiver to
   heuristically detect whether the sender is in slow-start, then to
   implement change-triggered ACKs while the sender is in slow-start,
   and offload otherwise.

   In a scenario where both endpoints support AccECN, if host B has
   chosen to use ECN-capable pure ACKs (as allowed in [RFC8311]
   experiments) and enough of these ACKs become CE-marked, then the
   'Increment-Triggered ACKs' rule ensures that its peer (host A) gives
   B sufficient feedback about this congestion on the ACKs from B to A.
   Normally, for instance in a unidirectional data scenario from host A
   to B, the Data Sender (A) can piggyback that feedback on its data.
   But if A stops sending data, the second part of the 'Increment-
   Triggered ACKs' rule requires A to emit a pure ACK for at least every
   third CE-marked incoming ACK over the subsequent round trip. 
   Although TCP normally only ACKs data segments, in this case the
   increment-triggered ACK rule makes it mandatory for A to emit ACKs of
   ACKs.  This is justifiable because the ACKs in this case are ECN-
   capable and so, even though the ACKs of these ACKs do not acknowledge
   new data, they feed back new congestion state (useful in case B
   starts sending).  The minimum of 3 for 'n' in this case ensures that,
   even if A also uses ECN-capable pure ACKs, and even if there is
   pathological congestion in both directions, any resulting ping-pong
   of ACKs will be rapidly damped.

   In the above bidirectional scenario, incoming ACKs of ACKs could be
   mistaken for duplicate ACKs.  But ACKs of ACKs can be distinguished
   from duplicate ACKs because they do not contain any SACK blocks even
   when SACK has been negotiated.  It is outside the scope of this
   AccECN specification to normatively specify this additional test for
   DupACKs, because ACKs of ACKs can only arise if the original ACKs are
   ECN-capable.  Instead any specification that allows ECN-capable pure
   ACKs MUST make sending ACKs of ACKs conditional on measures to
   distinguish ACKs of ACKs from DupACKs (see for example
   [I-D.ietf-tcpm-generalized-ecn ]).  All that is necessary here is to
   require that these ACKs of ACKs MUST NOT contain any SACK blocks
   (which would normally not happen anyway).

###### 3.2.2.5.2. Data Sender Safety Procedures

   If the Data Sender has not received AccECN TCP Options to give it
   more dependable information, and it detects that the ACE field could
   have cycled, it SHOULD deem whether it cycled by taking the safest
   likely case under the prevailing conditions.  It can detect if the
   counter could have cycled by using the jump in the acknowledgement
   number since the last ACK to calculate or estimate how many segments
   could have been acknowledged.  An example algorithm to implement this
   policy is given in Appendix A.2.  An implementation MAY use an
   alternative algorithm as long as it satisfies the requirements in
   this subsection.

   If missing acknowledgement numbers arrive later (reordering) and
   prove that the counter did not cycle, the Data Sender MAY attempt to
   neutralize the effect of any action it took based on a conservative
   assumption that it later found to be incorrect.

   The Data Sender can estimate how many packets (of any marking) an ACK
   acknowledges.  If the ACE counter on an ACK seems to imply that the
   minimum number of newly CE-marked packets is greater than the number
   of newly acknowledged packets, the Data Sender SHOULD consider the
   ACE counter to be correct (and its count of control packets to be
   incomplete), unless it can be sure that it is counting all control
   packets correctly. 





#### 3.2.3. The AccECN Option

   Two alternative AccECN Options are defined as shown in Figure 4.  The
   initial 'E' of each field name stands for 'Echo'.

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Kind = 172   |  Length = 11  |          EE0B field           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | EE0B (cont'd) |           ECEB field                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  EE1B field                   |             Order 0
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Kind = 174   |  Length = 11  |          EE1B field           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | EE1B (cont'd) |           ECEB field                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  EE0B field                   |             Order 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

              Figure 4: The Two Alternative AccECN TCP Options

   Figure 4 shows two option field orders; order 0 and order 1.  They
   both consists of three 24-bit fields.  Order 0 provides the 24 least
   significant bits of the r.e0b, r.ceb and r.e1b counters,
   respectively.  Order 1 provides the same fields, but in the opposite
   order.  On each packet, the Data Receiver can use whichever order is
   more efficient.  In either case, the bytes within the fields are in
   network byte order (big-endian).

   The choice to use three bytes (24 bits) fields in the options was
   made to strike a balance between TCP option space usage, and the
   required fidelity of the counters to accomodate typical scenarios
   such as hardware TCP segmentation offloading (TSO), and periods where
   no option may be transmitted (e.g., SACK loss recovery).  Providing
   only 2 bytes (16 bits) for these counters could easily roll over
   within a single TSO transmission or large/generic receive offload
   (LRO/GRO) event.  Having two distinct orderings further allows the
   transmission of the most pertinent changes in an abbreviated option
   (see below). 
   When a Data Receiver sends an AccECN Option, it MUST set the Kind
   field to 172 if using Order 0, or to 174 if using Order 1.  These two
   new TCP Option Kinds are registered in Section 7 and called
   respectively AccECN0 and AccECN1.

   Note that there is no field to feed back Not-ECT bytes.  Nonetheless
   an algorithm for the Data Sender to calculate the number of payload
   bytes received as Not-ECT is given in Appendix A.4.

   Whenever a Data Receiver sends an AccECN Option, the rules in Section 3.2.3.3 allow it to omit unchanged fields from the tail of
   the option, to help cope with option space limitations, as long as it
   preserves the order of the remaining fields and includes any field
   that has changed.  The length field MUST indicate which fields are
   present as follows:

             +========+==================+==================+
             | Length | Order 0          | Order 1          |
             +========+==================+==================+
             | 11     | EE0B, ECEB, EE1B | EE1B, ECEB, EE0B |
             +--------+------------------+------------------+
             | 8      | EE0B, ECEB       | EE1B, ECEB       |
             +--------+------------------+------------------+
             | 5      | EE0B             | EE1B             |
             +--------+------------------+------------------+
             | 2      | (empty)          | (empty)          |
             +--------+------------------+------------------+

                  Table 5: Fields included in AccECN TCP
                     Options of each length and order

   The empty option of Length=2 is provided to allow for a case where an
   AccECN Option has to be sent (e.g., on the SYN/ACK to test the path),
   but there is very limited space for the option.

   All implementations of a Data Sender that read any AccECN Option MUST
   be able to read AccECN Options of any of the above lengths.  For
   forward compatibility, if the AccECN Option is of any other length,
   implementations MUST use those whole 3-octet fields that fit within
   the length and ignore the remainder of the option, treating it as
   padding.

   AccECN Options have to be optional to implement, because both sender
   and receiver have to be able to cope without options anyway - in
   cases where they do not traverse a network path.  It is RECOMMENDED
   to implement both sending and receiving of AccECN Options.  Support
   for AccECN Options is particularly valuable over paths that introduce
   a high degree of ACK filtering, where the 3-bit ACE counter alone 
   might sometimes be insufficient, when it is ambiguous whether it has
   wrapped.  If sending of AccECN Options is implemented, the fall-backs
   described in this document will need to be implemented as well
   (unless solely for a controlled environment where path traversal is
   not considered a problem).  Even if a developer does not implement
   logic to understand received AccECN Options, it is RECOMMENDED that
   they implement logic to send AccECN Options.  Otherwise, those remote
   peers that implement the receiving logic will still be excluded from
   congestion feedback that is robust against the increasingly
   aggressive ACK filtering in the Internet.  The logic to send AccECN
   Options is the simpler to implement of the two sides.

   If a Data Receiver intends to send an AccECN Option at any time
   during the rest of the connection it is RECOMMENDED to also test path
   traversal of the AccECN Option as specified in Section 3.2.3.2.

##### 3.2.3.1. Encoding and Decoding Feedback in the AccECN Option Fields

   Whenever the Data Receiver includes any of the counter fields (ECEB,
   EE0B, EE1B) in an AccECN Option, it MUST encode the 24 least
   significant bits of the current value of the associated counter into
   the field (respectively r.ceb, r.e0b, r.e1b).

   Whenever the Data Sender receives an ACK carrying an AccECN Option,
   it first checks whether the ACK has already been superseded by
   another ACK in which case it ignores the ECN feedback.  If the ACK
   has not been superseded, the Data Sender normally decodes the fields
   in the AccECN Option as follows.  For each field, it takes the least
   significant 24 bits of its associated local counter (s.ceb, s.e0b or
   s.e1b) and subtracts them from the counter in the associated field of
   the incoming AccECN Option (respectively ECEB, EE0B, EE1B), to work
   out the minimum positive increment it could apply to s.ceb, s.e0b or
   s.e1b (assuming the field in the option only wrapped at most once). Appendix A.1 gives an example algorithm for the Data Receiver to
   encode its byte counters into an AccECN Option, and for the Data
   Sender to decode the AccECN Option fields into its byte counters.

   Note that, as specified in Section 3.2, any data on the SYN (SYN=1,
   ACK=0) is not included in any of the byte counters held locally for
   each ECN marking nor in an AccECN Option on the wire.

##### 3.2.3.2. Path Traversal of the AccECN Option
















###### 3.2.3.2.1. Testing the AccECN Option during the Handshake

   The TCP Client MUST NOT include an AccECN TCP Option on the SYN.  If
   there is somehow an AccECN Option on a SYN, it MUST be ignored when
   forwarded or received.

   A TCP Server that confirms its support for AccECN (in response to an
   AccECN SYN from the Client as described in Section 3.1) SHOULD
   include an AccECN TCP Option on the SYN/ACK.

   A TCP Client that has successfully negotiated AccECN SHOULD include
   an AccECN Option in the first ACK at the end of the three-way
   handshake.  However, this first ACK is not delivered reliably, so the
   TCP Client SHOULD also include an AccECN Option on the first data
   segment it sends (if it ever sends one).

   A host MAY omit an AccECN Option in any of the above three cases due
   to insufficient option space or if it has cached knowledge that the
   packet would be likely to be blocked on the path to the other host if
   it included an AccECN Option.

###### 3.2.3.2.2. Testing for Loss of Packets Carrying the AccECN Option

   If the TCP Server has not received an ACK to acknowledge its SYN/ACK
   after the normal TCP timeout or it receives a second SYN with a
   request for AccECN support, then either the SYN/ACK might just have
   been lost, e.g., due to congestion, or a middlebox might be blocking
   AccECN Options.  To expedite connection setup in deployment scenarios
   where AccECN path traversal might be problematic, the TCP Server
   SHOULD retransmit the SYN/ACK, but with no AccECN Option.  If this
   retransmission times out, to expedite connection setup, the TCP
   Server SHOULD retransmit the SYN/ACK with (AE,CWR,ECE) = (0,0,0) and
   no AccECN Option, but it remains in AccECN feedback mode (per Section 3.1.5).

      |  Note that a retransmitted AccECN SYN/ACK will not necessarily
      |  have the same TCP-ECN flags as the original SYN/ACK, because it
      |  feeds back the IP-ECN field of the latest SYN to have arrived
      |  (by the rule in Section 3.1.5).

   The above fall-back approach limits any interference by middleboxes
   that might drop packets with unknown options, even though it is more
   likely that SYN/ACK loss is due to congestion.  The TCP Server MAY
   try to send another packet with an AccECN Option at a later point
   during the connection but it ought to monitor if that packet got lost
   as well, in which case it SHOULD disable the sending of AccECN
   Options for this half-connection. 
   Implementers MAY use other fall-back strategies if they are found to
   be more effective (e.g., retrying an AccECN Option for a second time
   before fall-back - most appropriate during high levels of
   congestion).  However, other fall-back strategies will need to follow
   all the rules in Section 3.1.5, which concern behaviour when SYNs or
   SYN/ACKs negotiating different types of feedback have been sent
   within the same connection.

   Further it might make sense to also remove any other new or
   experimental fields or options on the SYN/ACK, although the required
   behaviour will depend on the specification of the other option(s) and
   on any attempt to co-ordinate fall-back between different modules of
   the stack.

   If the TCP Client detects that the first data segment it sent with an
   AccECN Option was lost, in deployment scenarios where AccECN path
   traversal might be problematic, it SHOULD fall back to no AccECN
   Option on the retransmission.  Again, implementers MAY use other
   fall-back strategies such as attempting to retransmit a second
   segment with an AccECN Option before fall-back, and/or caching
   whether AccECN Options are blocked for subsequent connections.
   [RFC9040] further discusses caching of TCP parameters and status
   information.

   If a middlebox is dropping packets with options it does not
   recognize, a host that is sending little or no data but mostly pure
   ACKs will not inherently detect such losses.  Such a host MAY detect
   loss of ACKs carrying the AccECN Option by detecting whether the
   acknowledged data always reappears as a retransmission.  In such
   cases, the host SHOULD disable the sending of the AccECN Option for
   this half-connection.

   If a host falls back to not sending AccECN Options, it will continue
   to process any incoming AccECN Options as normal.

   Either host MAY include AccECN Options in a subsequent segment or
   segments to retest whether AccECN Options can traverse the path.

   Similarly, an AccECN endpoint MAY separately memorize which data
   packets carried an AccECN Option and disable the sending of AccECN
   Options if the loss probability of those packets is significantly
   higher than that of all other data packets in the same connection. 





###### 3.2.3.2.3. Testing for Absence of the AccECN Option

   If the TCP Client has successfully negotiated AccECN but does not
   receive an AccECN Option on the SYN/ACK (e.g., because is has been
   stripped by a middlebox or not sent by the Server), the Client
   switches into a mode that assumes that the AccECN Option is not
   available for this half connection.

   Similarly, if the TCP Server has successfully negotiated AccECN but
   does not receive an AccECN Option on the first segment that
   acknowledges sequence space at least covering the ISN, it switches
   into a mode that assumes that the AccECN Option is not available for
   this half connection.

   While a host is in this mode that assumes incoming AccECN Options are
   not available, it MUST adopt the conservative interpretation of the
   ACE field discussed in Section 3.2.2.5.  However, it cannot make any
   assumption about support of outgoing AccECN Options on the other half
   connection, so it SHOULD continue to send AccECN Options itself
   (unless it has established that sending AccECN Options is causing
   packets to be blocked as in Section 3.2.3.2.2).

   If a host is in the mode that assumes incoming AccECN Options are not
   available, but it receives an AccECN Option at any later point during
   the connection, this clearly indicates that AccECN Options are no
   longer blocked on the respective path, and the AccECN endpoint MAY
   switch out of the mode that assumes AccECN Options are not available
   for this half connection.

###### 3.2.3.2.4. Test for Zeroing of the AccECN Option

   For a related test for invalid initialization of the ACE field, see Section 3.2.2.4

   Section 3.2.1 required the Data Receiver to initialize the r.e0b and
   r.e1b counters to a non-zero value.  Therefore, in either direction
   the initial value of the EE0B field or EE1B field in an AccECN Option
   (if one exists) ought to be non-zero.  If AccECN has been negotiated:

   *  the TCP Server MAY check that the initial value of the EE0B field
      or the EE1B field is non-zero in the first segment that
      acknowledges sequence space that at least covers the ISN plus 1.
      If it runs a test and either initial value is zero, the Server
      will switch into a mode that ignores AccECN Options for this half
      connection. 
   *  the TCP Client MAY check the initial value of the EE0B field or
      the EE1B field is non-zero on the SYN/ACK.  If it runs a test and
      either initial value is zero, the Client will switch into a mode
      that ignores AccECN Options for this half connection.

   While a host is in the mode that ignores AccECN Options it MUST adopt
   the conservative interpretation of the ACE field discussed in Section 3.2.2.5.

   Note that the Data Sender MUST NOT test whether the arriving byte
   counters in an initial AccECN Option have been initialized to
   specific valid values - the above checks solely test whether these
   fields have been incorrectly zeroed.  This allows hosts to use
   different initial values as an additional signalling channel in
   future.  Also note that the initial value of either field might be
   greater than its expected initial value, because the counters might
   already have been incremented.  Nonetheless, the initial values of
   the counters have been chosen so that they cannot wrap to zero on
   these initial segments.

###### 3.2.3.2.5. Consistency between AccECN Feedback Fields

   When AccECN Options are available they ought to provide more
   unambiguous feedback.  However, they supplement but do not replace
   the ACE field.  An endpoint using AccECN feedback MUST always
   reconcile the information provided in the ACE field with that in any
   AccECN Option, so that the state of the ACE-related packet counter
   can be relied on if future feedback does not carry an AccECN Option.

   If an AccECN Option is present, the s.cep counter might increase more
   than expected from the increase of the s.ceb counter (e.g., due to a
   CE-marked control packet).  The sender's response to such a situation
   is out of scope, and needs to be dealt with in a specification that
   uses ECN-capable control packets.  Theoretically, this situation
   could also occur if a middlebox mangled an AccECN Option but not the
   ACE field.  However, the Data Sender has to assume that the integrity
   of AccECN Options is sound, based on the above test of the well-known
   initial values and optionally other integrity tests (Section 5.3).

   If either endpoint detects that the s.ceb counter has increased but
   the s.cep has not (and by testing ACK coverage it is certain how much
   the ACE field has wrapped), and if there is no explanation other than
   an invalid protocol transition due to some form of feedback mangling,
   the Data Sender MUST disable sending ECN-capable packets for the
   remainder of the half-connection by setting the IP-ECN field in all
   subsequent packets to Not-ECT. 





##### 3.2.3.3. Usage of the AccECN TCP Option

   If a Data Receiver in AccECN mode intends to use AccECN TCP Options
   to provide feedback, the rules below determine when it includes an
   AccECN TCP Option, and which fields to include, given other options
   might be competing for limited option space:

   Importance of Congestion Control:  AccECN is for congestion control,
      which implementations SHOULD generally prioritize over other TCP
      options when there is insufficient space for all the options in
      use.

      If SACK has been negotiated [RFC2018], and the smallest
      recommended AccECN Option would leave insufficient space for two
      SACK blocks on a particular ACK, the Data Receiver MUST give
      precedence to the SACK option (total 18 octets), because loss
      feedback is more critical.

   Recommended Simple Scheme:  The Data Receiver SHOULD include an
      AccECN TCP Option on every scheduled ACK if any byte counter has
      incremented since the last ACK.  Whenever possible, it SHOULD
      include a field for every byte counter that has changed at some
      time during the connection (see examples later).

      A scheduled ACK means an ACK that the Data Receiver would send by
      its regular delayed ACK rules.  Recall that Section 1.3 defines an
      'ACK' as either with data payload or without.  But the above rule
      is worded so that, in the common case when most of the data is
      from a Server to a Client, the Server only includes an AccECN TCP
      Option while it is acknowledging data from the Client.

   When available TCP option space is limited on particular packets, the
   recommended scheme will need to include compromises.  To guide the
   implementer the rules below are ranked in order of importance, but
   the final decision has to be implementation-dependent, because
   tradeoffs will alter as new TCP options are defined and new use-cases
   arise.

   Necessary Option Length:  When TCP option space is limited, an AccECN
      TCP option MAY be truncated to omit one or two fields from the end
      of the option, as indicated by the permitted variants listed in
      Table 5, provided that the counter(s) that have changed since the
      previous AccECN TCP option are not omitted.

      If there is insufficient space to include an AccECN TCP option
      containing the counter(s) that have changed since the previous
      AccECN TCP option, then the entire AccECN TCP option MUST be
      omitted. (see Section 3.2.3);
   Change-Triggered AccECN TCP Options:  If an arriving packet
      increments a different byte counter to that incremented by the
      previous packet, the Data Receiver SHOULD feed it back in an
      AccECN Option on the next scheduled ACK.

      For the avoidance of doubt, this rule does not concern the arrival
      of control packets with no payload, because they cannot alter any
      byte counters.

   Continual Repetition:  Otherwise, if arriving packets continue to
      increment the same byte counter:

      *  the Data Receiver SHOULD include a counter that has continued
         to increment on the next scheduled ACK following a change-
         triggered AccECN TCP Option;

      *  while the same counter continues to increment, it SHOULD
         include the counter every n ACKs as consistently as possible,
         where n can be chosen by the implementer;

      *  It SHOULD always include an AccECN Option if the r.ceb counter
         is incrementing and it MAY include an AccECN Option if r.ec0b
         or r.ec1b is incrementing

      *  It SHOULD include each counter at least once for every 2^22
         bytes incremented to prevent overflow during continual
         repetition.

   The above rules complement those in Section 3.2.2.5, which determine
   when to generate an ACK irrespective of whether an AccECN TCP Option
   is to be included.

   The recommended scheme is intended as a simple way to ensure that all
   the relevant byte counters will be carried on any ACK that reaches
   the Data Sender, no matter how many pure ACKs are filtered or
   coalesced along the network path, and without consuming the space
   available for payload data with counter field(s) that have never
   changed.

   As an example of the recommended scheme, if ECT(0) is the only
   codepoint that has ever arrived in the IP-ECN field, the Data
   Receiver will feed back an AccECN0 TCP Option with only the EE0B
   field on every packet that acknowledges new data.  However, as soon
   as even one CE-marked packet arrives, on every packet that
   acknowledges new data it will start to include an option with two
   fields, EE0B and ECEB.  As a second example, if the first packet to
   arrive happens to be CE-marked, the Data Receiver will have to
   arbitrarily choose whether to precede the ECEB field with an EE0B 
   field or an EE1B field.  If it chooses, say, EEB0 but it turns out
   never to receive ECT(0), it can start sending EE1B and ECEB instead -
   it does not have to include the EE0B field if the r.e0b counter has
   never changed during the connection.

   With the recommended scheme, if the data sending direction switches
   during a connection, there can be cases where the AccECN TCP Option
   that is meant to feed back the counter values at the end of a volley
   in one direction never reaches the other peer, due to packet loss.
   ACE feedback ought to be sufficient to fill this gap, given accurate
   feedback becomes moot after data transmission has paused. Appendix A.3 gives an example algorithm to estimate the number of
   marked bytes from the ACE field alone, if AccECN Options are not
   available.

   If a host has determined that segments with AccECN Options always
   seem to be discarded somewhere along the path, it is no longer
   obliged to follow any of the rules in this section.

### 3.3. AccECN Compliance Requirements for TCP Proxies, Offload Engines


      and other Middleboxes    Given AccECN alters the TCP protocol on the wire, this section
   specifies new requirements on certain networking equipment that
   forwards TCP and inspects TCP header information.

#### 3.3.1. Requirements for TCP Proxies

   A large class of middleboxes split TCP connections.  Such a middlebox
   would be compliant with the AccECN protocol if the TCP implementation
   on each side complied with the present AccECN specification and each
   side negotiated AccECN independently of the other side.

#### 3.3.2. Requirements for Transparent Middleboxes and TCP Normalizers

   Another large class of middleboxes intervenes to some degree at the
   transport layer, but attempts to be transparent (invisible) to the
   end-to-end connection.  A subset of this class of middleboxes
   attempts to `normalize' the TCP wire protocol by checking that all
   values in header fields comply with a rather narrow interpretation of
   the TCP specifications that is also not always up to date.

   A middlebox that is not normalizing the TCP protocol and does not
   itself act as a back-to-back pair of TCP endpoints (i.e., a middlebox
   that intends to be transparent or invisible at the transport layer)
   ought to forward AccECN TCP Options unaltered, whether or not the
   length value matches one of those specified in Section 3.2.3, and 
   whether or not the initial values of the byte-counter fields match
   those in Section 3.2.1.  This is because blocking apparently invalid
   values prevents the standardized set of values being extended in
   future (such outdated normalizers would block updated hosts from
   using the extended AccECN standard).

   A TCP normalizer is likely to block or alter an AccECN TCP Option if
   the length value or the initial values of its byte-counter fields do
   not match one of those specified in Section 3.2.3 or Section 3.2.1.
   However, to comply with the present AccECN specification, a middlebox
   MUST NOT change the ACE field; or those fields of an AccECN Option
   that are currently specified in Section 3.2.3; or any AccECN field
   covered by integrity protection (e.g., [RFC5925]).

#### 3.3.3. Requirements for TCP ACK Filtering



   Section 5.2.1 of BCP 69 [RFC3449] gives best current practice on
   filtering (aka. thinning or coalescing) of pure TCP ACKs.  It advises
   that filtering ACKs carrying ECN feedback ought to preserve the
   correct operation of ECN feedback.  As the present specification
   updates the operation of ECN feedback, this section discusses how an
   ACK filter might preserve correct operation of AccECN feedback as
   well.

   The problem divides into two parts: determining if an ACK is part of
   a connection that is using AccECN and then preserving the correct
   operation of AccECN feedback:

   *  To determine whether a pure TCP ACK is part of an AccECN
      connection without resorting to connection tracking and per-flow
      state, a useful heuristic would be to check for a non-zero ECN
      field at the IP layer (because the ECN++ experiment only allows
      TCP pure ACKs to be ECN-capable if AccECN has been negotiated
      [I-D.ietf-tcpm-generalized-ecn ]).  This heuristic is simple and
      stateless.  However, it might omit some AccECN ACKs, because
      AccECN can be used without ECN++ and even if it is, ECN++ does not
      have to make pure ACKs ECN-capable - only deployment experience
      will tell.  Also, TCP ACKs might be ECN-capable owing to some
      scheme other than AccECN, e.g., [RFC5690] or some future standards
      action.  Again, only deployment experience will tell.

   *  The main concern with preserving correct AccECN operation involves
      leaving enough ACKs for the Data Sender to work out whether the
      3-bit ACE field has wrapped.  In the worst case, in feedback about
      a run of received packets that were all ECN-marked, the ACE field
      will wrap every 8 acknowledged packets.  ACE field wrap might be
      of less concern if packets also carry AccECN TCP Options.
      However, note that logic to read an AccECN TCP Option is optional 
      to implement (albeit recommended — see Section 3.2.3).  So one end
      writing an AccECN TCP Option into a packet does not necessarily
      imply that the other end will read it.

   Note that the present specification of AccECN in TCP does not presume
   to rely on any of the above ACK filtering behaviour in the network,
   because it has to be robust against pre-existing network nodes that
   do not distinguish AccECN ACKs, and robust against ACK loss during
   overload more generally.

#### 3.3.4. Requirements for TCP Segmentation Offload and Large Receive


        Offload    Hardware to offload certain TCP processing represents another large
   class of middleboxes (even though it is often a function of a host's
   network interface and rarely in its own 'box').

   Offloading can happen in the transmit path, usually referred to as
   TCP Segmentation Offload (TSO), and the receive path where it is
   called Large Receive Offload (LRO).

   In the transmit direction, with AccECN, all segments created from the
   same super-segment should retain the same ACE field, which should
   make TSO straighforward.

   However, with TSO hardware that supports [RFC3168], the CWR bit is
   usually masked out on the middle and last segment.  If applied to an
   AccECN segment, this would change the ACE field, and would be
   interpreted as having received numerous CE marks in the receive
   direction.  Therefore, currently available TSO hardware with
   [RFC3168] support may need some minor driver changes, to adjust the
   bitmask for the first, middle and last segment processed with TSO.

   Initially, when Classic ECN [RFC3168] and Accurate ECN flows coexist
   on the same offloading engine, the host software may need to work
   around incompatibilities (e.g., when only global configurable TSO TCP
   Flag bitmasks are available), otherwise this would cause some issues.

   One way around this could be to only negotiate for Accurate ECN, but
   not offer a fall back to [RFC3168] ECN.  Another way could be to
   allow TSO only as long as the CWR flag in the TCP header is not set -
   at the cost of more processing overhead while the ACE field has this
   bit set.

   For LRO in the receive direction, a different issue may get exposed
   with [RFC3168] ECN supporting hardware. 
   The ACE field changes with every received CE marking, so today's
   receive offloading could lead to many interrupts in high congestion
   situations.  Although that would be useful (because congestion
   information is received sooner), it could also significantly increase
   processor load, particularly in scenarios such as DCTCP or L4S where
   the marking rate is generally higher.

   Current offload hardware ejects a segment from the coalescing process
   whenever the TCP ECN flags change.  In data centres it has been
   fortunate for this offload hardware that DCTCP-style feedback changes
   less often when there are long sequences of CE marks, which is more
   common with a step marking threshold (but less likely the more short
   flows are in the mix).  The ACE counter approach has been designed so
   that coalescing can continue over arbitrary patterns of marking and
   only needs to stop when the counter wraps.  Nonetheless, until the
   particular offload hardware in use implements this more efficient
   approach, it is likely to be more efficient for AccECN connections to
   implement this counter-style logic using software segmentation
   offload.

   ECN encodes a varying signal in the ACK stream, so it is inevitable
   that offload hardware will ultimately need to handle any form of ECN
   feedback exceptionally.  The ACE field has been designed as a counter
   so that it is straightforward for offload hardware to pass on the
   highest counter, and to push a segment from its cache before the
   counter wraps.  The purpose of working towards standardized TCP ECN
   feedback is to reduce the risk for hardware developers, who would
   otherwise have to guess which scheme is likely to become dominant.

   The above process has been designed to enable a continuing
   incremental deployment path - to more highly dynamic congestion
   control.  Once offload hardware supports AccECN, it will be able to
   coalesce efficiently for any sequence of marks, instead of relying
   for efficiency on the long marking sequences from step marking.  In
   the next stage, marking can evolve from a step to a ramp function.
   That in turn will allow host congestion control algorithms to respond
   faster to dynamics, while being backwards compatible with existing
   host algorithms.

## 4. Updates to RFC 3168

   This section clarifies which parts of RFC3168 are updated and maps
   them to the sections of the present AccECN specification that update
   them:

   *  The whole of "6.1.1 TCP Initialization" of [RFC3168] is updated by Section 3.1 of the present specification. 
   *  In "6.1.2.  The TCP Sender" of [RFC3168], all mentions of a
      congestion response to an ECN-Echo (ECE) ACK packet are updated by Section 3.2 of the present specification to mean an increment to
      the sender's count of CE-marked packets, s.cep.  And the
      requirements to set the CWR flag no longer apply, as specified in Section 3.1.5 of the present specification.  Otherwise, the
      remaining requirements in "6.1.2.  The TCP Sender" still stand.

      It will be noted that RFC 8311 already updates, or potentially
      updates, a number of the requirements in "6.1.2.  The TCP Sender". Section 6.1.2 of RFC 3168 extended standard TCP congestion control
      [RFC5681] to cover ECN marking as well as packet drop.  Whereas,RFC 8311 enables experimentation with alternative responses to ECN
      marking, if specified for instance by an experimental RFC on the
      IETF document stream.  RFC 8311 also strengthened the statement
      that "ECT(0) SHOULD be used" to a "MUST" (see [RFC8311] for the
      details).

   *  The whole of "6.1.3.  The TCP Receiver" of [RFC3168] is updated by Section 3.2 of the present specification, with the exception of
      the last paragraph (about congestion response to drop and ECN in
      the same round trip), which still stands.  Incidentally, this last
      paragraph is in the wrong section, because it relates to "TCP
      Sender" behaviour.

   *  The following text within "6.1.5.  Retransmitted TCP packets":

         "the TCP data receiver SHOULD ignore the ECN field on arriving
         data packets that are outside of the receiver's current
         window."

      is updated by more stringent acceptability tests for any packet
      (not just data packets) in the present specification.
      Specifically, in the normative specification of AccECN (Section 3)
      only 'Acceptable' packets contribute to the ECN counters at the
      AccECN receiver and Section 1.3 defines an Acceptable packet as
      one that passes acceptability tests equivalent in strength to
      those in both [RFC9293] and [RFC5961].

   *  Sections 5.2, 6.1.1, 6.1.4, 6.1.5 and 6.1.6 of [RFC3168] prohibit
      use of ECN on TCP control packets and retransmissions.  The
      present specification does not update that aspect of RFC 3168, but
      it does say what feedback an AccECN Data Receiver ought to provide
      if it receives an ECN-capable control packet or retransmission.
      This ensures AccECN is forward compatible with any future scheme
      that allows ECN on these packets, as provided for in section 4.3
      of [RFC8311] and as proposed in [I-D.ietf-tcpm-generalized-ecn ]. 





## 5. Interaction with TCP Variants

   This section is informative, not normative.

### 5.1. Compatibility with SYN Cookies

   A TCP Server can use SYN Cookies (see Appendix A of [RFC4987]) to
   protect itself from SYN flooding attacks.  It places minimal commonly
   used connection state in the SYN/ACK, and deliberately does not hold
   any state while waiting for the subsequent ACK (e.g., it closes the
   thread).  Therefore it cannot record the fact that it entered AccECN
   mode for both half-connections.  Indeed, it cannot even remember
   whether it negotiated the use of Classic ECN [RFC3168].

   Nonetheless, such a Server can determine that it negotiated AccECN as
   follows.  If a TCP Server using SYN Cookies supports AccECN and if it
   receives a pure ACK that acknowledges an ISN that is a valid SYN
   cookie, and if the ACK contains an ACE field with the value 0b010 to
   0b111 (decimal 2 to 7), the Server can infer the first two stages of
   the handshake:

   *  the TCP Client has to have requested AccECN support on the SYN;

   *  then, even though the Server kept no state, it has to have
      confirmed that it supported AccECN.

   Therefore the Server can switch itself into AccECN mode, and continue
   as if it had never forgotten that it switched itself into AccECN mode
   earlier.

   If the pure ACK that acknowledges a SYN cookie contains an ACE field
   with the value 0b000 or 0b001, these values indicate that the TCP
   Client did not request support for AccECN and therefore the Server
   does not enter AccECN mode for this connection.  Further, 0b001 on
   the ACK implies that the Server sent an ECN-capable SYN/ACK, which
   was marked CE in the network, and the non-AccECN TCP Client fed this
   back by setting ECE on the ACK of the SYN/ACK.

### 5.2. Compatibility with TCP Experiments and Common TCP Options

   AccECN is compatible (at least on paper) with the most commonly used
   TCP options: MSS, time-stamp, window scaling, SACK and TCP-AO.  It is
   also compatible with Multipath TCP (MPTCP [RFC8684]) and the
   experimental TCP option TCP Fast Open (TFO [RFC7413]).  AccECN is
   friendly to all these protocols, because space for TCP options is
   particularly scarce on the SYN, where AccECN consumes zero additional
   header space. 
   When option space is under pressure from other options,Section 3.2.3.3 provides guidance on how important it is to send an
   AccECN Option relative to other options, and which fields are more
   important to include.

   Implementers of TFO need to take careful note of the recommendation
   in Section 3.2.2.1.  That section recommends that, if the TCP Client
   has successfully negotiated AccECN, when acknowledging the SYN/ACK,
   even if it has data to send, it sends a pure ACK immediately before
   the data.  Then it can reflect the IP-ECN field of the SYN/ACK on
   this pure ACK, which allows the Server to detect ECN mangling.  Note
   that, as specified in Section 3.2, any data on the SYN (SYN=1, ACK=0)
   is not included in any of the byte counters held locally for each ECN
   marking, nor in the AccECN Option on the wire.

   AccECN feedback is compatible with the ECN++
   [I-D.ietf-tcpm-generalized-ecn ] experiment, which allows TCP control
   packets and retransmissions to be ECN-capable ([RFC3168] was updated
   by [RFC8311] to permit such experiments).  AccECN is likely to
   inherently support any experiment with ECN-capable packets, because
   it feeds back the contents of the ECN field mechanistically, without
   judging whether a packet ought to use the ECN capability or not
   (Section 2.5).  This specification does not discuss implementing
   AccECN alongside [RFC5562], which was an earlier experimental
   protocol with narrower scope than ECN++ and a 5-way handshake.

### 5.3. Compatibility with Feedback Integrity Mechanisms

   Three alternative mechanisms are available to assure the integrity of
   ECN and/or loss signals.  AccECN is compatible with any of these
   approaches:

   *  The Data Sender can test the integrity of the receiver's ECN (or
      loss) feedback by occasionally setting the IP-ECN field to a value
      normally only set by the network (and/or deliberately leaving a
      sequence number gap).  Then it can test whether the Data
      Receiver's feedback faithfully reports what it expects (similar to
      paragraph 2 of Section 20.2 of [RFC3168]).  Unlike the ECN Nonce
      [RFC3540], this approach does not waste the ECT(1) codepoint in
      the IP header, it does not require standardization and it does not
      rely on misbehaving receivers volunteering to reveal feedback
      information that allows them to be detected.  However, setting the
      CE mark by the sender might conceal actual congestion feedback
      from the network and therefore ought to only be done sparingly.

   *  Networks generate congestion signals when they are becoming
      congested, so networks are more likely than Data Senders to be
      concerned about the integrity of the receiver's feedback of these 
      signals.  A network can enforce a congestion response to its ECN
      markings (or packet losses) using congestion exposure (ConEx)
      audit [RFC7713].  Whether the receiver or a downstream network is
      suppressing congestion feedback or the sender is unresponsive to
      the feedback, or both, ConEx audit can neutralize any advantage
      that any of these three parties would otherwise gain.

      ConEx is an experimental change to the Data Sender that would be
      most useful when combined with AccECN.  Without AccECN, the ConEx
      behaviour of a Data Sender would have to be more conservative than
      would be necessary if it had the accurate feedback of AccECN.

   *  The standards track TCP authentication option (TCP-AO [RFC5925])
      can be used to detect any tampering with AccECN feedback between
      the Data Receiver and the Data Sender (whether malicious or
      accidental).  The AccECN fields are immutable end-to-end, so they
      are amenable to TCP-AO protection, which covers TCP options by
      default.  However, TCP-AO is often too brittle to use on many end-
      to-end paths, where middleboxes can make verification fail in
      their attempts to improve performance or security, e.g., Network
      Address (and Port) Translation (NAT/NAPT), resegmentation or
      shifting the sequence space.

## 6. Summary: Protocol Properties

   This section is informative not normative.  It describes how well the
   protocol satisfies the agreed requirements for a more Accurate ECN
   feedback protocol [RFC7560].

   Accuracy:  From each ACK, the Data Sender can infer the number of new
      CE marked segments since the previous ACK.  This provides better
      accuracy on CE feedback than Classic ECN.  In addition if an
      AccECN Option is present (not blocked by the network path) the
      number of bytes marked with CE, ECT(1) and ECT(0) are provided.

   Overhead:  The AccECN scheme is divided into two parts.  The
      essential feedback part reuses the 3 flags already assigned to ECN
      in the TCP header.  The supplementary feedback part adds an
      additional TCP option consuming up to 11 bytes.  However, no TCP
      option space is consumed in the SYN.

   Ordering:  The order in which marks arrive at the Data Receiver is
      preserved in AccECN feedback, because the Data Receiver is
      expected to send an ACK immediately whenever a different mark
      arrives.

   Timeliness:  While the same ECN markings are arriving continually at 
      the Data Receiver, it can defer ACKs as TCP does normally, but it
      will immediately send an ACK as soon as a different ECN marking
      arrives.

   Timeliness vs Overhead:  Change-Triggered ACKs are intended to enable
      latency-sensitive uses of ECN feedback by capturing the timing of
      transitions but not wasting resources while the state of the
      signalling system is stable.  Within the constraints of the
      change-triggered ACK rules, the receiver can control how
      frequently it sends AccECN TCP Options and therefore to some
      extent it can control the overhead induced by AccECN.

   Resilience:  All information is provided based on counters.
      Therefore if ACKs are lost, the counters on the first ACK
      following the losses allows the Data Sender to immediately recover
      the number of the ECN markings that it missed.  And if data or
      ACKs are reordered, stale congestion information can be identified
      and ignored.

   Resilience against Bias:  Because feedback is based on repetition of
      counters, random losses do not remove any information, they only
      delay it.  Therefore, even though some ACKs are change-triggered,
      random losses will not alter the proportions of the different ECN
      markings in the feedback.

   Resilience vs Overhead:  If space is limited in some segments
      (e.g., because more options are needed on some segments, such as
      the SACK option after loss), the Data Receiver can send AccECN
      Options less frequently or truncate fields that have not changed,
      usually down to as little as 5 bytes.

   Resilience vs Timeliness and Ordering:  Ordering information and the
      timing of transitions cannot be communicated in three cases: i)
      during ACK loss; ii) if something on the path strips AccECN
      Options; or iii) if the Data Receiver is unable to support Change-
      Triggered ACKs.  Following ACK reordering, the Data Sender can
      reconstruct the order in which feedback was sent, but not until
      all the missing feedback has arrived.

   Complexity:  An AccECN implementation solely involves simple counter
      increments, some modulo arithmetic to communicate the least
      significant bits and allow for wrap, and some heuristics for
      safety against fields cycling due to prolonged periods of ACK
      loss.  Each host needs to maintain eight additional counters.  The
      hosts have to apply some additional tests to detect tampering by
      middleboxes, but in general the protocol is simple to understand,
      simple to implement and requires few cycles per packet to execute. 
   Integrity:  AccECN is compatible with at least three approaches that
      can assure the integrity of ECN feedback.  If AccECN Options are
      stripped the resolution of the feedback is degraded, but the
      integrity of this degraded feedback can still be assured.

   Backward Compatibility:  If only one endpoint supports the AccECN
      scheme, it will fall-back to the most advanced ECN feedback scheme
      supported by the other end.

      If AccECN Options are stripped by a middlebox, AccECN still
      provides basic congestion feedback in the ACE field.  Further,
      AccECN can be used to detect mangling of the IP ECN field;
      mangling of the TCP ECN flags; blocking of ECT-marked segments;
      and blocking of segments carrying an AccECN Option.  It can detect
      these conditions during TCP's three-way handshake so that it can
      fall back to operation without ECN and/or operation without AccECN
      Options.

   Forward Compatibility:  The behaviour of endpoints and middleboxes is
      carefully defined for all reserved or currently unused codepoints
      in the scheme.  Then, the designers of security devices can
      understand which currently unused values might appear in future.
      So, even if they choose to treat such values as anomalous while
      they are not widely used, any blocking will at least be under
      policy control not hard-coded.  Then, if previously unused values
      start to appear on the Internet (or in standards), such policies
      could be quickly reversed.

## 7. IANA Considerations

   This document reassigns the TCP header flag at bit offset 7 to the
   AccECN protocol.  This bit was previously called the Nonce Sum (NS)
   flag [RFC3540], but RFC 3540 has been reclassified as historic
   [RFC8311].  The flag will now be defined as the following in the "TCP
   Header Flags" registry in the "Transmission Control Protocol (TCP)
   Parameters" registry group:

     +=====+==============+===========+==============================+
     | Bit | Name         | Reference | Assignment Notes             |
     +=====+==============+===========+==============================+
     | 7   | AE (Accurate | RFC XXXX  | Previously used as NS (Nonce |
     |     | ECN)         |           | Sum) by [RFC3540], which is  |
     |     |              |           | now historic [RFC8311]       |
     +-----+--------------+-----------+------------------------------+

                   Table 6: TCP header flag reassignment 
   [TO BE REMOVED: IANA is requested to update the existing entry in the
   TCP Header Flags registry (https://www.iana.org/assignments/tcp-
   parameters/tcp-parameters.xhtml#tcp-header-flags ) for Bit 7 to "AE
   (Accurate ECN)" and to change the reference to this RFC-to-be instead
   of RFC8311.  Also IANA is requested to change the assignment note to
   "Previously used as NS (Nonce Sum) by [RFC3540], which is now
   historic [RFC8311]."]

   This document also defines two new TCP options for AccECN, assigned
   values of 172 and 174 (decimal) from the TCP option space.  These
   values are defined as the following in the "TCP Option Kind Numbers"
   registry in the "Transmission Control Protocol (TCP) Parameters"
   registry group:

      +======+========+================================+===========+
      | Kind | Length | Meaning                        | Reference |
      +======+========+================================+===========+
      | 172  | N      | Accurate ECN Order 0 (AccECN0) | RFC XXXX  |
      +------+--------+--------------------------------+-----------+
      | 174  | N      | Accurate ECN Order 1 (AccECN1) | RFC XXXX  |
      +------+--------+--------------------------------+-----------+

                   Table 7: New TCP Option assignments

   [TO BE REMOVED: These registrations have taken place using the early
   registration procedure, which may be temporary if this draft does not
   proceed, at the following location: http://www.iana.org/assignments/
   tcp-parameters/tcp-parameters.xhtml#tcp-parameters-1 ]

   Early experimental implementations of the two AccECN Options used
   experimental option 254 per [RFC6994] with the 16-bit magic numbers
   0xACC0 and 0xACC1 respectively for Order 0 and 1, as allocated in the
   IANA "TCP Experimental Option Experiment Identifiers (TCP ExIDs)"
   registry.  Even earlier experimental implementations used the single
   magic number 0xACCE (16 bits).  Uses of these experimental options
   SHOULD migrate to use the new option kinds (172 & 174).

   [TO BE REMOVED: IANA is requested to replace the references for all
   three of the above experimental options (0xACC0, 0xACC1 and 0xACCE)
   with a reference to the present RFC XXXX.]

   [TO BE REMOVED: If the early registrations, which may be temporary,
   do not proceed, the three references to them in the TCP ExIDs
   registry at the following location will also need to be edited out:https://www.iana.org/assignments/tcp-parameters/tcp-
   parameters.xhtml#tcp-exids  ]





## 8. Security and Privacy Considerations

   If ever the supplementary feedback part of AccECN based on one of the
   new AccECN TCP Options is unusable (due for example to middlebox
   interference) the essential feedback part of AccECN's congestion
   feedback offers only limited resilience to long runs of ACK loss (see Section 3.2.2.5).  These problems are unlikely to be due to malicious
   intervention (because if an attacker could strip a TCP option or
   discard a long run of ACKs it could wreak other arbitrary havoc).
   However, it would be of concern if AccECN's resilience could be
   indirectly compromised during a flooding attack.  AccECN is still
   considered safe though, because if AccECN Options are not present,
   the AccECN Data Sender is then required to switch to more
   conservative assumptions about wrap of congestion indication counters
   (see Section 3.2.2.5 and Appendix A.2). Section 5.1 describes how a TCP Server can negotiate AccECN and use
   the SYN cookie method for mitigating SYN flooding attacks.

   There is concern that ECN feedback could be altered or suppressed,
   particularly because a misbehaving Data Receiver could increase its
   own throughput at the expense of others.  AccECN is compatible with
   the three schemes known to assure the integrity of ECN feedback (see Section 5.3 for details).  If AccECN Options are stripped by an
   incorrectly implemented middlebox, the resolution of the feedback
   will be degraded, but the integrity of this degraded information can
   still be assured.  Assuring that Data Senders respond appropriately
   to ECN feedback is possible, but the scope of the present document is
   confined to the feedback protocol, and excludes the response to this
   feedback.

   In Section 3.2.3 a Data Sender is allowed to ignore an unrecognized
   TCP AccECN Option length and read as many whole 3-octet fields from
   it as possible up to a maximum of 3, treating the remainder as
   padding.  This opens up a potential covert channel of up to 29B (40 -
   (2+3*3)) B.  However, it is really an overt channel (not hidden) and
   it is no different to the use of unknown TCP options with unknown
   option lengths in general.  Therefore, where this is of concern, it
   can already be adequately mitigated by regular TCP normalizer
   technology (see Section 3.3.2).

   The AccECN protocol is not believed to introduce any new privacy
   concerns, because it merely counts and feeds back signals at the
   transport layer that had already been visible at the IP layer.  A
   covert channel can be used to compromise privacy.  However, as
   explained above, undefined TCP options in general open up such
   channels and common techniques are available to close them off. 
   There is a potential concern that a Data Receiver could deliberately
   omit AccECN Options pretending that they had been stripped by a
   middlebox.  No known way can yet be contrived for a receiver to take
   advantage of this behaviour, which seems to always degrade its own
   performance.  However, the concern is mentioned here for
   completeness.

   A generic privacy concern of any new protocol is that for a while it
   will be used by a small population of hosts, and thus show up more
   easily.  However, it is expected that this option will become
   available in operating systems over time, and eventually turned on by
   default in them.  Thus a individual identification of a particular
   user is less of a concern than the fingerprinting of specific
   versions of operation systems.  However, the latter can be done using
   different means independent of Accurate ECN.

   As Accurate ECN exposes more bits in the TCP header which could be
   tampered with without interfering with the transport excessively, it
   may allow an additional way to identify specific data streams across
   a virtual private network (VPN) to an attacker which has access to
   the datastream before and after the VPN tunnel endpoints.  This may
   be achieved by injecting or modifying the ACE field in specific
   patters that can be recognized.

   Overall, Accurate ECN does not change the risk profile on privacy to
   a user dramatically beyond what is already possible using classic
   ECN.  However, in order to prevent such attacks and means of easier
   identification of flows, it is adviseable for privacy conscious users
   behind VPNs to not enable the Accurate ECN, or Classic ECN for that
   matter.

## A. Example Algorithms

   This appendix is informative, not normative.  It gives example
   algorithms that would satisfy the normative requirements of the
   AccECN protocol.  However, implementers are free to choose other ways
   to implement the requirements.

### A.1. Example Algorithm to Encode/Decode the AccECN Option

   The example algorithms below show how a Data Receiver in AccECN mode
   could encode its CE byte counter r.ceb into the ECEB field within an
   AccECN TCP Option, and how a Data Sender in AccECN mode could decode
   the ECEB field into its byte counter s.ceb.  The other counters for
   bytes marked ECT(0) and ECT(1) in an AccECN Option would be similarly
   encoded and decoded.

   It is assumed that each local byte counter is an unsigned integer
   greater than 24b (probably 32b), and that the following constant has
   been assigned:

      DIVOPT = 2^24

   Every time a CE marked data segment arrives, the Data Receiver
   increments its local value of r.ceb by the size of the TCP Data.
   Whenever it sends an ACK with an AccECN Option, the value it writes
   into the ECEB field is

      ECEB = r.ceb % DIVOPT 
   where '%' is the remainder operator.

   On the arrival of an AccECN Option, the Data Sender first makes sure
   the ACK has not been superseded in order to avoid winding the s.ceb
   counter backwards.  It uses the TCP acknowledgement number and any
   SACK options [RFC2018] to calculate newlyAckedB, the amount of new
   data that the ACK acknowledges in bytes (newlyAckedB can be zero but
   not negative).  If newlyAckedB is zero, either the ACK has been
   superseded or CE-marked packet(s) without data could have arrived.
   To break the tie for the latter case, the Data Sender could use time-
   stamps [RFC7323] (if present) to work out newlyAckedT, the amount of
   new time that the ACK acknowledges.  If the Data Sender determines
   that the ACK has been superseded it ignores the AccECN Option.
   Otherwise, the Data Sender calculates the minimum non-negative
   difference d.ceb between the ECEB field and its local s.ceb counter,
   using modulo arithmetic as follows:

      if ((newlyAckedB > 0) || (newlyAckedT > 0)) {
          d.ceb = (ECEB + DIVOPT - (s.ceb % DIVOPT)) % DIVOPT
          s.ceb += d.ceb
      }

   For example, if s.ceb is 33,554,433 and ECEB is 1461 (both decimal),
   then

      s.ceb % DIVOPT = 1
      d.ceb = (1461 + 2^24 - 1) % 2^24
            = 1460
      s.ceb = 33,554,433 + 1460
            = 33,555,893

   In practice an implementation might use heuristics to guess the
   feedback in missing ACKs, then when it subsequently receives feedback
   it might find that it needs to correct its earlier heuristics as part
   of the decoding process.  The above decoding process does not include
   any such heuristics.

### A.2. Example Algorithm for Safety Against Long Sequences of ACK Loss

   The example algorithms below show how a Data Receiver in AccECN mode
   could encode its CE packet counter r.cep into the ACE field, and how
   the Data Sender in AccECN mode could decode the ACE field into its
   s.cep counter.  The Data Sender's algorithm includes code to
   heuristically detect a long enough unbroken string of ACK losses that
   could have concealed a cycle of the congestion counter in the ACE
   field of the next ACK to arrive. 
   Two variants of the algorithm are given: i) a more conservative
   variant for a Data Sender to use if it detects that AccECN Options
   are not available (see Section 3.2.2.5 and Section 3.2.3.2); and ii)
   a less conservative variant that is feasible when complementary
   information is available from AccECN Options.

#### A.2.1. Safety Algorithm without the AccECN Option

   It is assumed that each local packet counter is a sufficiently sized
   unsigned integer (probably 32b) and that the following constant has
   been assigned:

      DIVACE = 2^3

   Every time an Acceptable CE marked packet arrives (Section 3.2.2.2),
   the Data Receiver increments its local value of r.cep by 1.  It
   repeats the same value of ACE in every subsequent ACK until the next
   CE marking arrives, where

      ACE = r.cep % DIVACE.

   If the Data Sender received an earlier value of the counter that had
   been delayed due to ACK reordering, it might incorrectly calculate
   that the ACE field had wrapped.  Therefore, on the arrival of every
   ACK, the Data Sender ensures the ACK has not been superseded using
   the TCP acknowledgement number, any SACK options and timestamps (if
   available) to calculate newlyAckedB, as in Appendix A.1.  If the ACK
   has not been superseded, the Data Sender calculates the minimum
   difference d.cep between the ACE field and its local s.cep counter,
   using modulo arithmetic as follows:

      if ((newlyAckedB > 0) || (newlyAckedT > 0))
          d.cep = (ACE + DIVACE - (s.cep % DIVACE)) % DIVACE Section 3.2.2.5 expects the Data Sender to assume that the ACE field
   cycled if it is the safest likely case under prevailing conditions.
   The 3-bit ACE field in an arriving ACK could have cycled and become
   ambiguous to the Data Sender if a sequence of ACKs goes missing that
   covers a stream of data long enough to contain 8 or more CE marks.
   We use the word `missing' rather than `lost', because some or all the
   missing ACKs might arrive eventually, but out of order.  Even if some
   of the missing ACKs were piggy-backed on data (i.e., not pure ACKs)
   retransmissions will not repair the lost AccECN information, because
   AccECN requires retransmissions to carry the latest AccECN counters,
   not the original ones. 
   The phrase `under prevailing conditions' allows for implementation-
   dependent interpretation.  A Data Sender might take account of the
   prevailing size of data segments and the prevailing CE marking rate
   just before the sequence of missing ACKs.  However, we shall start
   with the simplest algorithm, which assumes segments are all full-
   sized and ultra-conservatively it assumes that ECN marking was 100%
   on the forward path when ACKs on the reverse path started to all be
   dropped.  Specifically, if newlyAckedB is the amount of data that an
   ACK acknowledges since the previous ACK, then the Data Sender could
   assume that this acknowledges newlyAckedPkt full-sized segments,
   where newlyAckedPkt = newlyAckedB/MSS.  Then it could assume that the
   ACE field incremented by

       dSafer.cep = newlyAckedPkt - ((newlyAckedPkt - d.cep) % DIVACE),

   For example, imagine an ACK acknowledges newlyAckedPkt=9 more full-
   size segments than any previous ACK, and that ACE increments by a
   minimum of 2 CE marks (d.cep=2).  The above formula works out that it
   would still be safe to assume 2 CE marks (because 9 - ((9-2) % 8) =
   2).  However, if ACE increases by a minimum of 2 but acknowledges 10
   full-sized segments, then it would be necessary to assume that there
   could have been 10 CE marks (because 10 - ((10-2) % 8) = 10).

   Note that checks would need to be added to the above pseudocode for
   (d.cep > newlyAckedPkt), which could occur if newlyAckedPkt had been
   wrongly estimated using an inappropriate packet size.

   ACKs that acknowledge a large stretch of packets might be common in
   data centres to achieve a high packet rate or might be due to ACK
   thinning by a middlebox.  In these cases, cycling of the ACE field
   would often appear to have been possible, so the above algorithm
   would be over-conservative, leading to a false high marking rate and
   poor performance.  Therefore it would be reasonable to only use
   dSafer.cep rather than d.cep if the moving average of newlyAckedPkt
   was well below 8.

   Implementers could build in more heuristics to estimate prevailing
   average segment size and prevailing ECN marking.  For instance,
   newlyAckedPkt in the above formula could be replaced with
   newlyAckedPktHeur = newlyAckedPkt*p*MSS/s, where s is the prevailing
   segment size and p is the prevailing ECN marking probability.
   However, ultimately, if TCP's ECN feedback becomes inaccurate it
   still has loss detection to fall back on.  Therefore, it would seem
   safe to implement a simple algorithm, rather than a perfect one. 
   The simple algorithm for dSafer.cep above requires no monitoring of
   prevailing conditions and it would still be safe if, for example,
   segments were on average at least 5% of full-sized as long as ECN
   marking was 5% or less.  Assuming it was used, the Data Sender would
   increment its packet counter as follows:

      s.cep += dSafer.cep

   If missing acknowledgement numbers arrive later (due to reordering),Section 3.2.2.5 says "the Data Sender MAY attempt to neutralize the
   effect of any action it took based on a conservative assumption that
   it later found to be incorrect".  To do this, the Data Sender would
   have to store the values of all the relevant variables whenever it
   made assumptions, so that it could re-evaluate them later.  Given
   this could become complex and it is not required, we do not attempt
   to provide an example of how to do this.

#### A.2.2. Safety Algorithm with the AccECN Option

   When AccECN Options are available on the ACKs before and after the
   possible sequence of ACK losses, if the Data Sender only needs CE-
   marked bytes, it will have sufficient information in AccECN Options
   without needing to process the ACE field.  If for some reason it
   needs CE-marked packets, if dSafer.cep is different from d.cep, it
   can determine whether d.cep is likely to be a safe enough estimate by
   checking whether the average marked segment size (s = d.ceb/d.cep) is
   less than the MSS (where d.ceb is the amount of newly CE-marked bytes
   - see Appendix A.1).  Specifically, it could use the following
   algorithm:

      SAFETY_FACTOR = 2
      if (dSafer.cep > d.cep) {
          if (d.ceb <= MSS * d.cep) {  % Same as (s <= MSS), but no DBZ
             sSafer = d.ceb/dSafer.cep
             if (sSafer < MSS/SAFETY_FACTOR)
                 dSafer.cep = d.cep    % d.cep is a safe enough estimate
          } % else
              % No need for else; dSafer.cep is already correct,
              % because d.cep must have been too small
      }

   The chart below shows when the above algorithm will consider d.cep
   can replace dSafer.cep as a safe enough estimate of the number of CE-
   marked packets:
                    ^
              sSafer|
                    |
                 MSS+
                    |
                    |         dSafer.cep
                    |                  is
   MSS/SAFETY_FACTOR+--------------+    safest
                    |              |
                    | d.cep is safe|
                    |    enough    |
                    +-------------------->
                                  MSS     s

   The following examples give the reasoning behind the algorithm,
   assuming MSS=1460 :

   *  if d.cep=0, dSafer.cep=8 and d.ceb=1460, then s=infinity and
      sSafer=182.5.

      Therefore even though the average size of 8 data segments is
      unlikely to have been as small as MSS/8, d.cep cannot have been
      correct, because it would imply an average segment size greater
      than the MSS.

   *  if d.cep=2, dSafer.cep=10 and d.ceb=1460, then s=730 and
      sSafer=146.

      Therefore d.cep is safe enough, because the average size of 10
      data segments is unlikely to have been as small as MSS/10.

   *  if d.cep=7, dSafer.cep=15 and d.ceb=10200, then s=1457 and
      sSafer=680.

      Therefore d.cep is safe enough, because the average data segment
      size is more likely to have been just less than one MSS, rather
      than below MSS/2.

   If pure ACKs were allowed to be ECN-capable, missing ACKs would be
   far less likely.  However, because [RFC3168] currently precludes
   this, the above algorithm assumes that pure ACKs are not ECN-capable. 





### A.3. Example Algorithm to Estimate Marked Bytes from Marked Packets

   If AccECN Options are not available, the Data Sender can only decode
   CE-marking from the ACE field in packets.  Every time an ACK arrives,
   to convert this into an estimate of CE-marked bytes, it needs an
   average of the segment size, s_ave.  Then it can add or subtract
   s_ave from the value of d.ceb as the value of d.cep increments or
   decrements.  Some possible ways to calculate s_ave are outlined
   below.  The precise details will depend on why an estimate of marked
   bytes is needed.

   The implementation could keep a record of the byte numbers of all the
   boundaries between packets in flight (including control packets), and
   recalculate s_ave on every ACK.  However it would be simpler to
   merely maintain a counter packets_in_flight for the number of packets
   in flight (including control packets), which is reset once per RTT.
   Either way, it would estimate s_ave as:

      s_ave ~= flightsize / packets_in_flight,

   where flightsize is the variable that TCP already maintains for the
   number of bytes in flight and '~=' means 'approximately equal to'.
   To avoid floating point arithmetic, it could right-bit-shift by
   lg(packets_in_flight), where lg() means log base 2.

   An alternative would be to maintain an exponentially weighted moving
   average (EWMA) of the segment size:

      s_ave = a * s + (1-a) * s_ave,

   where a is the decay constant for the EWMA.  However, then it is
   necessary to choose a good value for this constant, which ought to
   depend on the number of packets in flight.  Also the decay constant
   needs to be power of two to avoid floating point arithmetic.

### A.4. Example Algorithm to Count Not-ECT Bytes

   A Data Sender in AccECN mode can infer the amount of TCP payload data
   arriving at the receiver marked Not-ECT from the difference between
   the amount of newly ACKed data and the sum of the bytes with the
   other three markings, d.ceb, d.e0b and d.e1b.

   For this approach to be precise, it has to be assumed that spurious
   (unnecessary) retransmissions do not lead to double counting.  This
   assumption is currently correct, given that RFC 3168 requires that
   the Data Sender marks retransmitted segments as Not-ECT.  However,
   the converse is not true; necessary retransmissions will result in
   under-counting. 
   However, such precision is unlikely to be necessary.  The only known
   use of a count of Not-ECT marked bytes is to test whether equipment
   on the path is clearing the ECN field (perhaps due to an out-dated
   attempt to clear, or bleach, what used to be the IPv4 ToS byte or the
   IPv6 Traffic Class field).  To detect bleaching it will be sufficient
   to detect whether nearly all bytes arrive marked as Not-ECT.
   Therefore there ought to be no need to keep track of the details of
   retransmissions.

## B. Rationale for Usage of TCP Header Flags





### B.1. Three TCP Header Flags in the SYN-SYN/ACK Handshake

   AccECN uses a rather unorthodox approach to negotiate the highest
   version TCP ECN feedback scheme that both ends support, as justified
   below.  It follows from the original TCP ECN capability negotiation
   [RFC3168], in which the Client set the 2 least significant of the
   original reserved flags in the TCP header, and fell back to no ECN
   support if the Server responded with the 2 flags cleared, which had
   previously been the default.

   Classic ECN used header flags rather than a TCP option because it was
   considered more efficient to use a header flag for 1 bit of feedback
   per ACK, and this bit could be overloaded to indicate support for
   Classic ECN during the handshake.  During the development of ECN, 1
   bit crept up to 2, in order to deliver the feedback reliably and to
   work round some broken hosts that reflected the reserved flags during
   the handshake.

   In order to be backward compatible with RFC 3168, AccECN continues
   this approach, using the 3rd least significant TCP header flag that
   had previously been allocated for the ECN nonce (now historic).
   Then, whatever form of Server an AccECN Client encounters, the
   connection can fall back to the highest version of feedback protocol
   that both ends support, as explained in Section 3.1.

   If AccECN capability negotiation had used the more orthodox approach
   of a TCP option, it would still have had to set the two ECN flags in
   the main TCP header, in order to be able to fall back to Classic RFC 
   3168 ECN, or to disable ECN support, without another round of
   negotiation.  Then AccECN would also have had to handle all the
   different ways that Servers currently respond to settings of the ECN
   flags in the main TCP header, including all the conflicting cases
   where a Server might have said it supported one approach in the flags
   and another approach in a new TCP option.  And AccECN would have had
   to deal with all the additional possibilities where a middlebox might
   have mangled the ECN flags, or removed TCP options.  Thus, usage of
   the 3rd reserved TCP header flag simplified the protocol. 
   The third flag was used in a way that could be distinguished from the
   ECN nonce, in case any nonce deployment was encountered.  Previous
   usage of this flag for the ECN nonce was integrated into the original
   ECN negotiation.  This further justified the 3rd flag's use for
   AccECN, because a non-ECN usage of this flag would have had to use it
   as a separate single bit, rather than in combination with the other 2
   ECN flags.

   Indeed, having overloaded the original uses of these three flags for
   its handshake, AccECN overloads all three bits again as a 3-bit
   counter.

### B.2. Four Codepoints in the SYN/ACK

   Of the 8 possible codepoints that the 3 TCP header flags can indicate
   on the SYN/ACK, 4 already indicated earlier (or broken) versions of
   ECN support, 1 now being historic.  In the early design of AccECN, an
   AccECN Server could use only 2 of the 4 remaining codepoints.  They
   both indicated AccECN support, but one fed back that the SYN had
   arrived marked as CE.  Even though ECN support on a SYN is not yet on
   the standards track, the idea is for either end to act as a
   mechanistic reflector, so that future capabilities can be
   unilaterally deployed without requiring 2-ended deployment (justified
   in Section 2.5).

   During traversal testing it was discovered that the IP-ECN field in
   the SYN was mangled on a non-negligible proportion of paths.
   Therefore it was necessary to allow the SYN/ACK to feed all four IP-
   ECN codepoints that the SYN could arrive with back to the Client.
   Without this, the Client could not know whether to disable ECN for
   the connection due to mangling of the IP-ECN field (also explained in Section 2.5).  This development consumed the remaining 2 codepoints
   on the SYN/ACK that had been reserved for future use by AccECN in
   earlier versions.

### B.3. Space for Future Evolution

   Despite availability of usable TCP header space being extremely
   scarce, the AccECN protocol has taken all possible steps to ensure
   that there is space to negotiate possible future variants of the
   protocol, either if a variant of AccECN is required, or if a
   completely different ECN feedback approach is needed:

   Future AccECN variants:  When the AccECN capability is negotiated
      during TCP's three-way handshake, the rows in Table 2 tagged as
      'Nonce' and 'Broken' in the column for the capability of node B
      are unused by any current protocol in the RFC series.  These could
      be used by TCP Servers in future to indicate a variant of the 
      AccECN protocol.  In recent measurement studies in which the
      response of large numbers of Servers to an AccECN SYN has been
      tested, e.g., [Mandalari18], a very small number of SYN/ACKs
      arrive with the pattern tagged as 'Nonce', and a small but more
      significant number arrive with the pattern tagged as 'Broken'.
      The 'Nonce' pattern could be a sign that a few Servers have
      implemented the ECN Nonce [RFC3540], which has now been
      reclassified as historic [RFC8311], or it could be the random
      result of some unknown middlebox behaviour.  The greater
      prevalence of the 'Broken' pattern suggests that some instances
      still exist of the broken code that reflects the reserved flags on
      the SYN.

      The requirement not to reject unexpected initial values of the ACE
      counter (in the main TCP header) in the last paragraph of Section 3.2.2.4 ensures that 3 unused codepoints on the ACK of the
      SYN/ACK, 6 unused values on the first SYN=0 data packet from the
      Client and 7 unused values on the first SYN=0 data packet from the
      Server could be used to declare future variants of the AccECN
      protocol.  The word 'declare' is used rather than 'negotiate'
      because, at this late stage in the three-way handshake, it would
      be too late for a negotiation between the endpoints to be
      completed.  A similar requirement not to reject unexpected initial
      values in AccECN TCP Options (Section 3.2.3.2.4) is for the same
      purpose.  If traversal of AccECN TCP Options were reliable, this
      would have enabled a far wider range of future variation of the
      whole AccECN protocol.  Nonetheless, it could be used to reliably
      negotiate a wide range of variation in the semantics of the AccECN
      Option.

   Future non-AccECN variants:  Five codepoints out of the 8 possible in
      the 3 TCP header flags used by AccECN are unused on the initial
      SYN (in the order (AE,CWR,ECE)): (0,0,1), (0,1,0), (1,0,0),
      (1,0,1), (1,1,0).  Section 3.1.3 ensures that the installed base
      of AccECN Servers will all assume these are equivalent to AccECN
      negotiation with (1,1,1) on the SYN.  These codepoints would not
      allow fall-back to Classic ECN support for a Server that did not
      understand them, but this approach ensures they are available in
      future, perhaps for uses other than ECN alongside the AccECN
      scheme.  All possible combinations of SYN/ACK could be used in
      response except either (0,0,0) or reflection of the same values
      sent on the SYN.

      In order to extend AccECN or ECN in future, other ways could be
      resorted to, although their traversal properties are likely to be
      inferior.  They include a new TCP option; using the remaining
      reserved flags in the main TCP header (preferably extending the
      3-bit combinations used by AccECN to 4-bit combinations, rather 
      than burning one bit for just one state); a non-zero urgent
      pointer in combination with the URG flag cleared; or some other
      unexpected combination of fields yet to be invented.



---
# Referenced Sections from RFC 3168: The Addition of Explicit Congestion Notification (ECN) to IP

The following sections were referenced. Remaining sections are not included.

### 5.2. Dropped or Corrupted Packets

   For the proposed use for ECN in this document (that is, for a
   transport protocol such as TCP for which a dropped data packet is an
   indication of congestion), end nodes detect dropped data packets, and
   the congestion response of the end nodes to a dropped data packet is
   at least as strong as the congestion response to a received CE
   packet.  To ensure the reliable delivery of the congestion indication
   of the CE codepoint, an ECT codepoint MUST NOT be set in a packet
   unless the loss of that packet in the network would be detected by
   the end nodes and interpreted as an indication of congestion.

   Transport protocols such as TCP do not necessarily detect all packet
   drops, such as the drop of a "pure" ACK packet; for example, TCP does
   not reduce the arrival rate of subsequent ACK packets in response to
   an earlier dropped ACK packet.  Any proposal for extending ECN-
   Capability to such packets would have to address issues such as the
   case of an ACK packet that was marked with the CE codepoint but was
   later dropped in the network. We believe that this aspect is still
   the subject of research, so this document specifies that at this
   time, "pure" ACK packets MUST NOT indicate ECN-Capability.

   Similarly, if a CE packet is dropped later in the network due to
   corruption (bit errors), the end nodes should still invoke congestion
   control, just as TCP would today in response to a dropped data
   packet. This issue of corrupted CE packets would have to be
   considered in any proposal for the network to distinguish between
   packets dropped due to corruption, and packets dropped due to
   congestion or buffer overflow.  In particular, the ubiquitous
   deployment of ECN would not, in and of itself, be a sufficient
   development to allow end-nodes to interpret packet drops as
   indications of corruption rather than congestion.

#### 6.1.1. TCP Initialization

   In the TCP connection setup phase, the source and destination TCPs
   exchange information about their willingness to use ECN.  Subsequent
   to the completion of this negotiation, the TCP sender sets an ECT
   codepoint in the IP header of data packets to indicate to the network
   that the transport is capable and willing to participate in ECN for
   this packet. This indicates to the routers that they may mark this 
   packet with the CE codepoint, if they would like to use that as a
   method of congestion notification. If the TCP connection does not
   wish to use ECN notification for a particular packet, the sending TCP
   sets the ECN codepoint to not-ECT, and the TCP receiver ignores the
   CE codepoint in the received packet.

   For this discussion, we designate the initiating host as Host A and
   the responding host as Host B.  We call a SYN packet with the ECE and
   CWR flags set an "ECN-setup SYN packet", and we call a SYN packet
   with at least one of the ECE and CWR flags not set a "non-ECN-setup
   SYN packet".  Similarly, we call a SYN-ACK packet with only the ECE
   flag set but the CWR flag not set an "ECN-setup SYN-ACK packet", and
   we call a SYN-ACK packet with any other configuration of the ECE and
   CWR flags a "non-ECN-setup SYN-ACK packet".

   Before a TCP connection can use ECN, Host A sends an ECN-setup SYN
   packet, and Host B sends an ECN-setup SYN-ACK packet.  For a SYN
   packet, the setting of both ECE and CWR in the ECN-setup SYN packet
   is defined as an indication that the sending TCP is ECN-Capable,
   rather than as an indication of congestion or of response to
   congestion. More precisely, an ECN-setup SYN packet indicates that
   the TCP implementation transmitting the SYN packet will participate
   in ECN as both a sender and receiver.  Specifically, as a receiver,
   it will respond to incoming data packets that have the CE codepoint
   set in the IP header by setting ECE in outgoing TCP Acknowledgement
   (ACK) packets.  As a sender, it will respond to incoming packets that
   have ECE set by reducing the congestion window and setting CWR when
   appropriate.  An ECN-setup SYN packet does not commit the TCP sender
   to setting the ECT codepoint in any or all of the packets it may
   transmit.  However, the commitment to respond appropriately to
   incoming packets with the CE codepoint set remains even if the TCP
   sender in a later transmission, within this TCP connection, sends a
   SYN packet without ECE and CWR set.

   When Host B sends an ECN-setup SYN-ACK packet, it sets the ECE flag
   but not the CWR flag.  An ECN-setup SYN-ACK packet is defined as an
   indication that the TCP transmitting the SYN-ACK packet is ECN-
   Capable.  As with the SYN packet, an ECN-setup SYN-ACK packet does
   not commit the TCP host to setting the ECT codepoint in transmitted
   packets.

   The following rules apply to the sending of ECN-setup packets within
   a TCP connection, where a TCP connection is defined by the standard
   rules for TCP connection establishment and termination.

      * If a host has received an ECN-setup SYN packet, then it MAY send
        an ECN-setup SYN-ACK packet.  Otherwise, it MUST NOT send an
        ECN-setup SYN-ACK packet. 
      * A host MUST NOT set ECT on data packets unless it has sent at
        least one ECN-setup SYN or ECN-setup SYN-ACK packet, and has
        received at least one ECN-setup SYN or ECN-setup SYN-ACK packet,
        and has sent no non-ECN-setup SYN or non-ECN-setup SYN-ACK
        packet.  If a host has received at least one non-ECN-setup SYN
        or non-ECN-setup SYN-ACK packet, then it SHOULD NOT set ECT on
        data packets.

      * If a host ever sets the ECT codepoint on a data packet, then
        that host MUST correctly set/clear the CWR TCP bit on all
        subsequent packets in the connection.

      * If a host has sent at least one ECN-setup SYN or ECN-setup SYN-
        ACK packet, and has received no non-ECN-setup SYN or non-ECN-
        setup SYN-ACK packet, then if that host receives TCP data
        packets with ECT and CE codepoints set in the IP header, then
        that host MUST process these packets as specified for an ECN-
        capable connection.

      * A host that is not willing to use ECN on a TCP connection SHOULD
        clear both the ECE and CWR flags in all non-ECN-setup SYN and/or
        SYN-ACK packets that it sends to indicate this unwillingness.
        Receivers MUST correctly handle all forms of the non-ECN-setup
        SYN and SYN-ACK packets.

      * A host MUST NOT set ECT on SYN or SYN-ACK packets.

   A TCP client enters TIME-WAIT state after receiving a FIN-ACK, and
   transitions to CLOSED state after a timeout.  Many TCP
   implementations create a new TCP connection if they receive an in-
   window SYN packet during TIME-WAIT state.  When a TCP host enters
   TIME-WAIT or CLOSED state, it should ignore any previous state about
   the negotiation of ECN for that connection.

##### 6.1.1.1. Middlebox Issues

   ECN introduces the use of the ECN-Echo and CWR flags in the TCP
   header (as shown in Figure 3) for initialization.  There exist some
   faulty firewalls, load balancers, and intrusion detection systems in
   the Internet that either drop an ECN-setup SYN packet or respond with
   a RST, in the belief that such a packet (with these bits set) is a
   signature for a port-scanning tool that could be used in a denial-
   of-service attack.  Some of the offending equipment has been
   identified, and a web page [FIXES ] contains a list of non-compliant
   products and the fixes posted by the vendors, where these are
   available.  The TBIT web page [TBIT ] lists some of the web servers
   affected by this faulty equipment.  We mention this in this document
   as a warning to the community of this problem. 
   To provide robust connectivity even in the presence of such faulty
   equipment, a host that receives a RST in response to the transmission
   of an ECN-setup SYN packet MAY resend a SYN with CWR and ECE cleared.
   This could result in a TCP connection being established without using
   ECN.

   A host that receives no reply to an ECN-setup SYN within the normal
   SYN retransmission timeout interval MAY resend the SYN and any
   subsequent SYN retransmissions with CWR and ECE cleared.  To overcome
   normal packet loss that results in the original SYN being lost, the
   originating host may retransmit one or more ECN-setup SYN packets
   before giving up and retransmitting the SYN with the CWR and ECE bits
   cleared.

   We note that in this case, the following example scenario is
   possible:

   (1) Host A: Sends an ECN-setup SYN.
   (2) Host B: Sends an ECN-setup SYN/ACK, packet is dropped or delayed.
   (3) Host A: Sends a non-ECN-setup SYN.
   (4) Host B: Sends a non-ECN-setup SYN/ACK.

   We note that in this case, following the procedures above, neither
   Host A nor Host B may set the ECT bit on data packets.  Further, an
   important consequence of the rules for ECN setup and usage in Section 
   6.1.1 is that a host is forbidden from using the reception of ECT
   data packets as an implicit signal that the other host is ECN-
   capable.

##### 6.1.1.2. Robust TCP Initialization with an Echoed Reserved Field

   There is the question of why we chose to have the TCP sending the SYN
   set two ECN-related flags in the Reserved field of the TCP header for
   the SYN packet, while the responding TCP sending the SYN-ACK sets
   only one ECN-related flag in the SYN-ACK packet.  This asymmetry is
   necessary for the robust negotiation of ECN-capability with some
   deployed TCP implementations.  There exists at least one faulty TCP
   implementation in which TCP receivers set the Reserved field of the
   TCP header in ACK packets (and hence the SYN-ACK) simply to reflect
   the Reserved field of the TCP header in the received data packet.
   Because the TCP SYN packet sets the ECN-Echo and CWR flags to
   indicate ECN-capability, while the SYN-ACK packet sets only the ECN-
   Echo flag, the sending TCP correctly interprets a receiver's
   reflection of its own flags in the Reserved field as an indication
   that the receiver is not ECN-capable.  The sending TCP is not mislead
   by a faulty TCP implementation sending a SYN-ACK packet that simply
   reflects the Reserved field of the incoming SYN packet. 





#### 6.1.2. The TCP Sender

   For a TCP connection using ECN, new data packets are transmitted with
   an ECT codepoint set in the IP header.  When only one ECT codepoint
   is needed by a sender for all packets sent on a TCP connection,
   ECT(0) SHOULD be used.  If the sender receives an ECN-Echo (ECE) ACK
   packet (that is, an ACK packet with the ECN-Echo flag set in the TCP
   header), then the sender knows that congestion was encountered in the
   network on the path from the sender to the receiver.  The indication
   of congestion should be treated just as a congestion loss in non-
   ECN-Capable TCP. That is, the TCP source halves the congestion window
   "cwnd" and reduces the slow start threshold "ssthresh".  The sending
   TCP SHOULD NOT increase the congestion window in response to the
   receipt of an ECN-Echo ACK packet.

   TCP should not react to congestion indications more than once every
   window of data (or more loosely, more than once every round-trip
   time). That is, the TCP sender's congestion window should be reduced
   only once in response to a series of dropped and/or CE packets from a
   single window of data.  In addition, the TCP source should not
   decrease the slow-start threshold, ssthresh, if it has been decreased
   within the last round trip time.  However, if any retransmitted
   packets are dropped, then this is interpreted by the source TCP as a
   new instance of congestion.

   After the source TCP reduces its congestion window in response to a
   CE packet, incoming acknowledgments that continue to arrive can
   "clock out" outgoing packets as allowed by the reduced congestion
   window.  If the congestion window consists of only one MSS (maximum
   segment size), and the sending TCP receives an ECN-Echo ACK packet,
   then the sending TCP should in principle still reduce its congestion
   window in half. However, the value of the congestion window is
   bounded below by a value of one MSS.  If the sending TCP were to
   continue to send, using a congestion window of 1 MSS, this results in
   the transmission of one packet per round-trip time.  It is necessary
   to still reduce the sending rate of the TCP sender even further, on
   receipt of an ECN-Echo packet when the congestion window is one.  We
   use the retransmit timer as a means of reducing the rate further in
   this circumstance.  Therefore, the sending TCP MUST reset the
   retransmit timer on receiving the ECN-Echo packet when the congestion
   window is one.  The sending TCP will then be able to send a new
   packet only when the retransmit timer expires.

   When an ECN-Capable TCP sender reduces its congestion window for any
   reason (because of a retransmit timeout, a Fast Retransmit, or in
   response to an ECN Notification), the TCP sender sets the CWR flag in
   the TCP header of the first new data packet sent after the window
   reduction.  If that data packet is dropped in the network, then the 
   sending TCP will have to reduce the congestion window again and
   retransmit the dropped packet.

   We ensure that the "Congestion Window Reduced" information is
   reliably delivered to the TCP receiver.  This comes about from the
   fact that if the new data packet carrying the CWR flag is dropped,
   then the TCP sender will have to again reduce its congestion window,
   and send another new data packet with the CWR flag set.  Thus, the
   CWR bit in the TCP header SHOULD NOT be set on retransmitted packets.

   When the TCP data sender is ready to set the CWR bit after reducing
   the congestion window, it SHOULD set the CWR bit only on the first
   new data packet that it transmits.

   [Floyd94] discusses TCP's response to ECN in more detail.  [Floyd98]
   discusses the validation test in the ns simulator, which illustrates
   a wide range of ECN scenarios. These scenarios include the following:
   an ECN followed by another ECN, a Fast Retransmit, or a Retransmit
   Timeout; a Retransmit Timeout or a Fast Retransmit followed by an
   ECN; and a congestion window of one packet followed by an ECN.

   TCP follows existing algorithms for sending data packets in response
   to incoming ACKs, multiple duplicate acknowledgments, or retransmit
   timeouts [RFC2581].  TCP also follows the normal procedures for
   increasing the congestion window when it receives ACK packets without
   the ECN-Echo bit set [RFC2581].

#### 6.1.3. The TCP Receiver

   When TCP receives a CE data packet at the destination end-system, the
   TCP data receiver sets the ECN-Echo flag in the TCP header of the
   subsequent ACK packet.  If there is any ACK withholding implemented,
   as in current "delayed-ACK" TCP implementations where the TCP
   receiver can send an ACK for two arriving data packets, then the
   ECN-Echo flag in the ACK packet will be set to '1' if the CE
   codepoint is set in any of the data packets being acknowledged.  That
   is, if any of the received data packets are CE packets, then the
   returning ACK has the ECN-Echo flag set.

   To provide robustness against the possibility of a dropped ACK packet
   carrying an ECN-Echo flag, the TCP receiver sets the ECN-Echo flag in
   a series of ACK packets sent subsequently.  The TCP receiver uses the
   CWR flag received from the TCP sender to determine when to stop
   setting the ECN-Echo flag.

   After a TCP receiver sends an ACK packet with the ECN-Echo bit set,
   that TCP receiver continues to set the ECN-Echo flag in all the ACK
   packets it sends (whether they acknowledge CE data packets or non-CE 
   data packets) until it receives a CWR packet (a packet with the CWR
   flag set).  After the receipt of the CWR packet, acknowledgments for
   subsequent non-CE data packets do not have the ECN-Echo flag set. If
   another CE packet is received by the data receiver, the receiver
   would once again send ACK packets with the ECN-Echo flag set.  While
   the receipt of a CWR packet does not guarantee that the data sender
   received the ECN-Echo message, this does suggest that the data sender
   reduced its congestion window at some point *after* it sent the data
   packet for which the CE codepoint was set.

   We have already specified that a TCP sender is not required to reduce
   its congestion window more than once per window of data.  Some care
   is required if the TCP sender is to avoid unnecessary reductions of
   the congestion window when a window of data includes both dropped
   packets and (marked) CE packets.  This is illustrated in [Floyd98].

#### 6.1.4. Congestion on the ACK-path

   For the current generation of TCP congestion control algorithms, pure
   acknowledgement packets (e.g., packets that do not contain any
   accompanying data) MUST be sent with the not-ECT codepoint.  Current
   TCP receivers have no mechanisms for reducing traffic on the ACK-path
   in response to congestion notification.  Mechanisms for responding to
   congestion on the ACK-path are areas for current and future research.
   (One simple possibility would be for the sender to reduce its
   congestion window when it receives a pure ACK packet with the CE
   codepoint set). For current TCP implementations, a single dropped ACK
   generally has only a very small effect on the TCP's sending rate.

#### 6.1.5. Retransmitted TCP packets

   This document specifies ECN-capable TCP implementations MUST NOT set
   either ECT codepoint (ECT(0) or ECT(1)) in the IP header for
   retransmitted data packets, and that the TCP data receiver SHOULD
   ignore the ECN field on arriving data packets that are outside of the
   receiver's current window.  This is for greater security against
   denial-of-service attacks, as well as for robustness of the ECN
   congestion indication with packets that are dropped later in the
   network.

   First, we note that if the TCP sender were to set an ECT codepoint on
   a retransmitted packet, then if an unnecessarily-retransmitted packet
   was later dropped in the network, the end nodes would never receive
   the indication of congestion from the router setting the CE
   codepoint.  Thus, setting an ECT codepoint on retransmitted data
   packets is not consistent with the robust delivery of the congestion
   indication even for packets that are later dropped in the network. 
   In addition, an attacker capable of spoofing the IP source address of
   the TCP sender could send data packets with arbitrary sequence
   numbers, with the CE codepoint set in the IP header.  On receiving
   this spoofed data packet, the TCP data receiver would determine that
   the data does not lie in the current receive window, and return a
   duplicate acknowledgement.  We define an out-of-window packet at the
   TCP data receiver as a data packet that lies outside the receiver's
   current window.  On receiving an out-of-window packet, the TCP data
   receiver has to decide whether or not to treat the CE codepoint in
   the packet header as a valid indication of congestion, and therefore
   whether to return ECN-Echo indications to the TCP data sender.  If
   the TCP data receiver ignored the CE codepoint in an out-of-window
   packet, then the TCP data sender would not receive this possibly-
   legitimate indication of congestion from the network, resulting in a
   violation of end-to-end congestion control.  On the other hand, if
   the TCP data receiver honors the CE indication in the out-of-window
   packet, and reports the indication of congestion to the TCP data
   sender, then the malicious node that created the spoofed, out-of-
   window packet has successfully "attacked" the TCP connection by
   forcing the data sender to unnecessarily reduce (halve) its
   congestion window.  To prevent such a denial-of-service attack, we
   specify that a legitimate TCP data sender MUST NOT set an ECT
   codepoint on retransmitted data packets, and that the TCP data
   receiver SHOULD ignore the CE codepoint on out-of-window packets.

   One drawback of not setting ECT(0) or ECT(1) on retransmitted packets
   is that it denies ECN protection for retransmitted packets.  However,
   for an ECN-capable TCP connection in a fully-ECN-capable environment
   with mild congestion, packets should rarely be dropped due to
   congestion in the first place, and so instances of retransmitted
   packets should rarely arise.  If packets are being retransmitted,
   then there are already packet losses (from corruption or from
   congestion) that ECN has been unable to prevent.

   We note that if the router sets the CE codepoint for an ECN-capable
   data packet within a TCP connection, then the TCP connection is
   guaranteed to receive that indication of congestion, or to receive
   some other indication of congestion within the same window of data,
   even if this packet is dropped or reordered in the network.  We
   consider two cases, when the packet is later retransmitted, and when
   the packet is not later retransmitted.

   In the first case, if the packet is either dropped or delayed, and at
   some point retransmitted by the data sender, then the retransmission
   is a result of a Fast Retransmit or a Retransmit Timeout for either
   that packet or for some prior packet in the same window of data.  In
   this case, because the data sender already has retransmitted this
   packet, we know that the data sender has already responded to an 
   indication of congestion for some packet within the same window of
   data as the original packet.  Thus, even if the first transmission of
   the packet is dropped in the network, or is delayed, if it had the CE
   codepoint set, and is later ignored by the data receiver as an out-
   of-window packet, this is not a problem, because the sender has
   already responded to an indication of congestion for that window of
   data.

   In the second case, if the packet is never retransmitted by the data
   sender, then this data packet is the only copy of this data received
   by the data receiver, and therefore arrives at the data receiver as
   an in-window packet, regardless of how much the packet might be
   delayed or reordered.  In this case, if the CE codepoint is set on
   the packet within the network, this will be treated by the data
   receiver as a valid indication of congestion.

#### 6.1.6. TCP Window Probes.

   When the TCP data receiver advertises a zero window, the TCP data
   sender sends window probes to determine if the receiver's window has
   increased.  Window probe packets do not contain any user data except
   for the sequence number, which is a byte.  If a window probe packet
   is dropped in the network, this loss is not detected by the receiver.
   Therefore, the TCP data sender MUST NOT set either an ECT codepoint
   or the CWR bit on window probe packets.

   However, because window probes use exact sequence numbers, they
   cannot be easily spoofed in denial-of-service attacks.  Therefore, if
   a window probe arrives with the CE codepoint set, then the receiver
   SHOULD respond to the ECN indications.

### 18.1. Possible Changes to the IP Header





#### 18.1.1. Erasing the Congestion Indication

   First, we consider the changes that a router could make that would
   result in effectively erasing the congestion indication after it had
   been set by a router upstream.  The convention followed is:  ECN
   codepoint of received packet -> ECN codepoint of packet transmitted.

   Replacing the CE codepoint with the ECT(0) or ECT(1) codepoint
   effectively erases the congestion indication.  However, with the use
   of two ECT codepoints, a router erasing the CE codepoint has no way
   to know whether the original ECT codepoint was ECT(0) or ECT(1).
   Thus, it is possible for the transport protocol to deploy mechanisms
   to detect such erasures of the CE codepoint.

   The consequence of the erasure of the CE codepoint for the upstream
   router is that there is a potential for congestion to build for a
   time, because the congestion indication does not reach the source.
   However, the packet would be received and acknowledged.

   The potential effect of erasing the congestion indication is complex,
   and is discussed in depth in Section 19 below.  Note that the effect
   of erasing the congestion indication is different from dropping a
   packet in the network.  When a data packet is dropped, the drop is
   detected by the TCP sender, and interpreted as an indication of
   congestion.  Similarly, if a sufficient number of consecutive
   acknowledgement packets are dropped, causing the cumulative
   acknowledgement field not to be advanced at the sender, the sender is
   limited by the congestion window from sending additional packets, and
   ultimately the retransmit timer expires.

   In contrast, a systematic erasure of the CE bit by a downstream
   router can have the effect of causing a queue buildup at an upstream
   router, including the possible loss of packets due to buffer
   overflow.  There is a potential of unfairness in that another flow
   that goes through the congested router could react to the CE bit set
   while the flow that has the CE bit erased could see better
   performance.  The limitations on this potential unfairness are
   discussed in more detail in Section 19 below.

   The last of the three changes is to replace the CE codepoint with the
   not-ECT codepoint, thus erasing the congestion indication and
   disabling ECN-Capability at the same time.

   The `erasure' of the congestion indication is only effective if the
   packet does not end up being marked or dropped again by a downstream
   router.  If the CE codepoint is replaced by an ECT codepoint, the 
   packet remains ECN-Capable, and could be either marked or dropped by
   a downstream router as an indication of congestion.  If the CE
   codepoint is replaced by the not-ECT codepoint, the packet is no
   longer ECN-capable, and can therefore be dropped but not marked by a
   downstream router as an indication of congestion.

#### 18.1.2. Falsely Reporting Congestion

   This change is to set the CE codepoint when an ECT codepoint was
   already set, even though there was no congestion.  This change does
   not affect the treatment of that packet along the rest of the path.
   In particular, a router does not examine the CE codepoint in deciding
   whether to drop or mark an arriving packet.

   However, this could result in the application unnecessarily invoking
   end-to-end congestion control, and reducing its arrival rate.  By
   itself, this is no worse (for the application or for the network)
   than if the tampering router had actually dropped the packet.

#### 18.1.3. Disabling ECN-Capability

   This change is to turn off the ECT codepoint of a packet.  This means
   that if the packet later encounters congestion (e.g., by arriving to
   a RED queue with a moderate average queue size), it will be dropped
   instead of being marked.  By itself, this is no worse (for the
   application) than if the tampering router had actually dropped the
   packet.  The saving grace in this particular case is that there is no
   congested router upstream expecting a reaction from setting the CE
   bit.

#### 18.1.4. Falsely Indicating ECN-Capability

   This change would incorrectly label a packet as ECN-Capable. The
   packet may have been sent either by an ECN-Capable transport or a
   transport that is not ECN-Capable.

   If the packet later encounters moderate congestion at an ECN-Capable
   router, the router could set the CE codepoint instead of dropping the
   packet.  If the transport protocol in fact is not ECN-Capable, then
   the transport will never receive this indication of congestion, and
   will not reduce its sending rate in response.  The potential
   consequences of falsely indicating ECN-capability are discussed
   further in Section 19 below.

   If the packet never later encounters congestion at an ECN-Capable
   router, then the first of these two changes would have no effect,
   other than possibly interfering with the use of the ECN nonce by the
   transport protocol.  The last change, however, would have the effect 
   of giving false reports of congestion to a monitoring device along
   the path.  If the transport protocol is ECN-Capable, then this change
   could also have an effect at the transport level, by combining
   falsely indicating ECN-Capability with falsely reporting congestion.
   For an ECN-capable transport, this would cause the transport to
   unnecessarily react to congestion.  In this particular case, the
   router that is incorrectly changing the ECN field could have dropped
   the packet. Thus for this case of an ECN-capable transport, the
   consequence of this change to the ECN field is no worse than dropping
   the packet.

### 23.1. IPv4 TOS Byte and IPv6 Traffic Class Octet

   The codepoints for the ECN Field of the IP header are specified by
   the Standards Action of this RFC, as is required by RFC 2780.

   When this document is published as an RFC, IANA should create a new
   registry, "IPv4 TOS Byte and IPv6 Traffic Class Octet", with the
   namespace as follows:

   IPv4 TOS Byte and IPv6 Traffic Class Octet

   Description:  The registrations are identical for IPv4 and IPv6.

   Bits 0-5:  see Differentiated Services Field Codepoints Registry
           (http://www.iana.org/assignments/dscp-registry )
   Bits 6-7, ECN Field:

   Binary  Keyword                                  References
   ------  -------                                  ----------
     00     Not-ECT (Not ECN-Capable Transport)     [RFC 3168]
     01     ECT(1) (ECN-Capable Transport(1))       [RFC 3168]
     10     ECT(0) (ECN-Capable Transport(0))       [RFC 3168]
     11     CE (Congestion Experienced)             [RFC 3168]

### 23.2. TCP Header Flags

   The codepoints for the CWR and ECE flags in the TCP header are
   specified by the Standards Action of this RFC, as is required by RFC 
   2780.

   When this document is published as an RFC, IANA should create a new
   registry, "TCP Header Flags", with the namespace as follows:

   TCP Header Flags

   The Transmission Control Protocol (TCP) included a 6-bit Reserved
   field defined in RFC 793, reserved for future use, in bytes 13 and 14
   of the TCP header, as illustrated below.  The other six Control bits
   are defined separately by RFC 793.

     0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   |               |                       | U | A | P | R | S | F |
   | Header Length |        Reserved       | R | C | S | S | Y | I |
   |               |                       | G | K | H | T | N | N |
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+RFC 3168 defines two of the six bits from the Reserved field to be
   used for ECN, as follows:

     0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   |               |               | C | E | U | A | P | R | S | F |
   | Header Length |    Reserved   | W | C | R | C | S | S | Y | I |
   |               |               | R | E | G | K | H | T | N | N |
   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
   TCP Header Flags

   Bit      Name                                    Reference
   ---      ----                                    ---------
    8        CWR (Congestion Window Reduced)        [RFC 3168]
    9        ECE (ECN-Echo)                         [RFC 3168]

# Referenced Sections from RFC 5681: TCP Congestion Control

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   This document specifies four TCP [RFC793] congestion control
   algorithms: slow start, congestion avoidance, fast retransmit and
   fast recovery.  These algorithms were devised in [Jac88] and [Jac90].
   Their use with TCP is standardized in [RFC1122].  Additional early
   work in additive-increase, multiplicative-decrease congestion control
   is given in [CJ89].

   Note that [Ste94] provides examples of these algorithms in action and
   [WS95] provides an explanation of the source code for the BSD
   implementation of these algorithms.

   In addition to specifying these congestion control algorithms, this
   document specifies what TCP connections should do after a relatively
   long idle period, as well as specifying and clarifying some of the
   issues pertaining to TCP ACK generation.

   This document obsoletes [RFC2581], which in turn obsoleted [RFC2001].

   This document is organized as follows.  Section 2 provides various
   definitions that will be used throughout the document.  Section 3   provides a specification of the congestion control algorithms. Section 4 outlines concerns related to the congestion control
   algorithms and finally, section 5 outlines security considerations. 
   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

### 3.1. Slow Start and Congestion Avoidance

   The slow start and congestion avoidance algorithms MUST be used by a
   TCP sender to control the amount of outstanding data being injected
   into the network.  To implement these algorithms, two variables are
   added to the TCP per-connection state.  The congestion window (cwnd)
   is a sender-side limit on the amount of data the sender can transmit
   into the network before receiving an acknowledgment (ACK), while the
   receiver's advertised window (rwnd) is a receiver-side limit on the
   amount of outstanding data.  The minimum of cwnd and rwnd governs
   data transmission.

   Another state variable, the slow start threshold (ssthresh), is used
   to determine whether the slow start or congestion avoidance algorithm
   is used to control data transmission, as discussed below. 
   Beginning transmission into a network with unknown conditions
   requires TCP to slowly probe the network to determine the available
   capacity, in order to avoid congesting the network with an
   inappropriately large burst of data.  The slow start algorithm is
   used for this purpose at the beginning of a transfer, or after
   repairing loss detected by the retransmission timer.  Slow start
   additionally serves to start the "ACK clock" used by the TCP sender
   to release data into the network in the slow start, congestion
   avoidance, and loss recovery algorithms.

   IW, the initial value of cwnd, MUST be set using the following
   guidelines as an upper bound.

   If SMSS > 2190 bytes:
       IW = 2 * SMSS bytes and MUST NOT be more than 2 segments
   If (SMSS > 1095 bytes) and (SMSS <= 2190 bytes):
       IW = 3 * SMSS bytes and MUST NOT be more than 3 segments
   if SMSS <= 1095 bytes:
       IW = 4 * SMSS bytes and MUST NOT be more than 4 segments

   As specified in [RFC3390], the SYN/ACK and the acknowledgment of the
   SYN/ACK MUST NOT increase the size of the congestion window.
   Further, if the SYN or SYN/ACK is lost, the initial window used by a
   sender after a correctly transmitted SYN MUST be one segment
   consisting of at most SMSS bytes.

   A detailed rationale and discussion of the IW setting is provided in
   [RFC3390].

   When initial congestion windows of more than one segment are
   implemented along with Path MTU Discovery [RFC1191], and the MSS
   being used is found to be too large, the congestion window cwnd
   SHOULD be reduced to prevent large bursts of smaller segments.
   Specifically, cwnd SHOULD be reduced by the ratio of the old segment
   size to the new segment size.

   The initial value of ssthresh SHOULD be set arbitrarily high (e.g.,
   to the size of the largest possible advertised window), but ssthresh
   MUST be reduced in response to congestion.  Setting ssthresh as high
   as possible allows the network conditions, rather than some arbitrary
   host limit, to dictate the sending rate.  In cases where the end
   systems have a solid understanding of the network path, more
   carefully setting the initial ssthresh value may have merit (e.g.,
   such that the end host does not create congestion along the path). 
   The slow start algorithm is used when cwnd < ssthresh, while the
   congestion avoidance algorithm is used when cwnd > ssthresh.  When
   cwnd and ssthresh are equal, the sender may use either slow start or
   congestion avoidance.

   During slow start, a TCP increments cwnd by at most SMSS bytes for
   each ACK received that cumulatively acknowledges new data.  Slow
   start ends when cwnd exceeds ssthresh (or, optionally, when it
   reaches it, as noted above) or when congestion is observed.  While
   traditionally TCP implementations have increased cwnd by precisely
   SMSS bytes upon receipt of an ACK covering new data, we RECOMMEND
   that TCP implementations increase cwnd, per:

      cwnd += min (N, SMSS)                      (2)

   where N is the number of previously unacknowledged bytes acknowledged
   in the incoming ACK.  This adjustment is part of Appropriate Byte
   Counting [RFC3465] and provides robustness against misbehaving
   receivers that may attempt to induce a sender to artificially inflate
   cwnd using a mechanism known as "ACK Division" [SCWA99].  ACK
   Division consists of a receiver sending multiple ACKs for a single
   TCP data segment, each acknowledging only a portion of its data.  A
   TCP that increments cwnd by SMSS for each such ACK will
   inappropriately inflate the amount of data injected into the network.

   During congestion avoidance, cwnd is incremented by roughly 1 full-
   sized segment per round-trip time (RTT).  Congestion avoidance
   continues until congestion is detected.  The basic guidelines for
   incrementing cwnd during congestion avoidance are:

      * MAY increment cwnd by SMSS bytes

      * SHOULD increment cwnd per equation (2) once per RTT

      * MUST NOT increment cwnd by more than SMSS bytes

   We note that [RFC3465] allows for cwnd increases of more than SMSS
   bytes for incoming acknowledgments during slow start on an
   experimental basis; however, such behavior is not allowed as part of
   the standard.

   The RECOMMENDED way to increase cwnd during congestion avoidance is
   to count the number of bytes that have been acknowledged by ACKs for
   new data.  (A drawback of this implementation is that it requires
   maintaining an additional state variable.)  When the number of bytes
   acknowledged reaches cwnd, then cwnd can be incremented by up to SMSS
   bytes.  Note that during congestion avoidance, cwnd MUST NOT be 
   increased by more than SMSS bytes per RTT.  This method both allows
   TCPs to increase cwnd by one segment per RTT in the face of delayed
   ACKs and provides robustness against ACK Division attacks.

   Another common formula that a TCP MAY use to update cwnd during
   congestion avoidance is given in equation (3):

      cwnd += SMSS*SMSS/cwnd                     (3)

   This adjustment is executed on every incoming ACK that acknowledges
   new data.  Equation (3) provides an acceptable approximation to the
   underlying principle of increasing cwnd by 1 full-sized segment per
   RTT.  (Note that for a connection in which the receiver is
   acknowledging every-other packet, (3) is less aggressive than allowed
   -- roughly increasing cwnd every second RTT.)

   Implementation Note: Since integer arithmetic is usually used in TCP
   implementations, the formula given in equation (3) can fail to
   increase cwnd when the congestion window is larger than SMSS*SMSS.
   If the above formula yields 0, the result SHOULD be rounded up to 1
   byte.

   Implementation Note: Older implementations have an additional
   additive constant on the right-hand side of equation (3).  This is
   incorrect and can actually lead to diminished performance [RFC2525].

   Implementation Note: Some implementations maintain cwnd in units of
   bytes, while others in units of full-sized segments.  The latter will
   find equation (3) difficult to use, and may prefer to use the
   counting approach discussed in the previous paragraph.

   When a TCP sender detects segment loss using the retransmission timer
   and the given segment has not yet been resent by way of the
   retransmission timer, the value of ssthresh MUST be set to no more
   than the value given in equation (4):

      ssthresh = max (FlightSize / 2, 2*SMSS)            (4)

   where, as discussed above, FlightSize is the amount of outstanding
   data in the network.

   On the other hand, when a TCP sender detects segment loss using the
   retransmission timer and the given segment has already been
   retransmitted by way of the retransmission timer at least once, the
   value of ssthresh is held constant. 
   Implementation Note: An easy mistake to make is to simply use cwnd,
   rather than FlightSize, which in some implementations may
   incidentally increase well beyond rwnd.

   Furthermore, upon a timeout (as specified in [RFC2988]) cwnd MUST be
   set to no more than the loss window, LW, which equals 1 full-sized
   segment (regardless of the value of IW).  Therefore, after
   retransmitting the dropped segment the TCP sender uses the slow start
   algorithm to increase the window from 1 full-sized segment to the new
   value of ssthresh, at which point congestion avoidance again takes
   over.

   As shown in [FF96] and [RFC3782], slow-start-based loss recovery
   after a timeout can cause spurious retransmissions that trigger
   duplicate acknowledgments.  The reaction to the arrival of these
   duplicate ACKs in TCP implementations varies widely.  This document
   does not specify how to treat such acknowledgments, but does note
   this as an area that may benefit from additional attention,
   experimentation and specification.

# Referenced Sections from RFC 8311: Relaxing Restrictions on Explicit Congestion Notification (ECN) Experimentation

The following sections were referenced. Remaining sections are not included.

## 2. ECN Experimentation: Overview

   Three areas of ECN experimentation are covered by this memo; the
   cited documents should be consulted for the detailed goals and
   rationale of each proposed experiment:

   Congestion Response Differences:  An ECN congestion indication
      communicates a higher likelihood than a dropped packet that a
      short queue exists at the network bottleneck node [TCP-ABE ].  This
      difference suggests that for congestion indicated by ECN, a
      different sender congestion response (e.g., sender backs off by a
      smaller amount) may be appropriate by comparison to the sender
      response to congestion indicated by loss.  Two examples of
      proposed sender congestion response changes are described in
      [TCP-ABE ] and [ECN-L4S ] -- the proposal in the latter document
      couples the sender congestion response change to Congestion
      Marking Differences functionality (see next paragraph).  These
      changes are at variance with the requirement in RFC 3168 that a
      sender's congestion control response to ECN congestion indications
      be the same as to drops.  IETF approval, e.g., via an Experimental
      RFC in the IETF document stream, is required for any sender
      congestion response used in this area of experimentation.  See Section 4.1 for further discussion.

   Congestion Marking Differences:  Congestion marking at network nodes
      can be configured to maintain very shallow queues in conjunction
      with a different sender response to congestion indications (CE
      marks), e.g., as proposed in [ECN-L4S ].  The traffic involved
      needs to be identified by the senders to the network nodes in
      order to avoid damage to other network traffic whose senders do
      not expect the more frequent congestion marking used to maintain
      very shallow queues.  Use of different ECN codepoints,
      specifically ECT(0) and ECT(1), is a promising means of traffic
      identification for this purpose, but that technique is at variance
      with the requirement in RFC 3168 that traffic marked as ECT(0) not
      receive different treatment in the network by comparison to
      traffic marked as ECT(1).  IETF approval, e.g., via an
      Experimental RFC in the IETF document stream, is required for any
      differences in congestion marking or sender congestion response
      used in this area of experimentation.  See Section 4.2 for further
      discussion. 
   TCP Control Packets and Retransmissions:  RFC 3168 limits the use of
      ECN with TCP to data packets, excluding retransmissions.  With the
      successful deployment of ECN in large portions of the Internet,
      there is interest in extending the benefits of ECN to TCP control
      packets (e.g., SYNs) and retransmitted packets, e.g., as proposed
      in [ECN-TCP ].  This is at variance with RFC 3168's prohibition of
      ECN for TCP control packets and retransmitted packets.  See Section 4.3 for further discussion.

   The scope of this memo is limited to these three areas of
   experimentation.  This memo expresses no view on the likely outcomes
   of the proposed experiments and does not specify the experiments in
   detail.  Additional experiments in these areas are possible, e.g., on
   use of ECN to support deployment of a protocol similar to Data Center
   TCP (DCTCP) [RFC8257] beyond DCTCP's current applicability that is
   limited to data center environments.  The purpose of this memo is to
   remove constraints in Standards Track RFCs that stand in the way of
   these areas of experimentation.

### 2.1. Effective Congestion Control is Required

   Congestion control remains an important aspect of the Internet
   architecture [RFC2914].  Any Experimental RFC in the IETF document
   stream that takes advantage of this memo's updates to any RFC is
   required to discuss the congestion control implications of the
   experiment(s) in order to provide assurance that deployment of the
   experiment(s) does not pose a congestion-based threat to the
   operation of the Internet.

### 2.2. Network Considerations for ECN Experimentation

   ECN functionality [RFC3168] is becoming widely deployed in the
   Internet and is being designed into additional protocols such as
   Transparent Interconnection of Lots of Links (TRILL) [ECN-TRILL ].
   ECN experiments are expected to coexist with deployed ECN
   functionality, with the responsibility for that coexistence falling
   primarily upon designers of experimental changes to ECN.  In
   addition, protocol designers and implementers, as well as network
   operators, may desire to anticipate and/or support ECN experiments.
   The following guidelines will help avoid conflicts with the areas of
   ECN experimentation enabled by this memo:

   1.  Forwarding behavior as described in RFC 3168 remains the
       preferred approach for routers that are not involved in ECN
       experiments, in particular continuing to treat the ECT(0) and
       ECT(1) codepoints as equivalent, as specified in Section 4.2       below. 
   2.  Network nodes that forward packets SHOULD NOT assume that the ECN
       CE codepoint indicates that the packet would have been dropped if
       ECN were not in use.  This is because Congestion Response
       Differences experiments employ different congestion responses to
       dropped packets by comparison to receipt of CE-marked packets
       (see Section 4.1 below), so CE-marked packets SHOULD NOT be
       arbitrarily dropped.  A corresponding difference in congestion
       responses already occurs when the ECN field is used for
       Pre-Congestion Notification (PCN) [RFC6660].

   3.  A network node MUST NOT originate traffic marked with ECT(1)
       unless the network node is participating in a Congestion Marking
       Differences experiment that uses ECT(1), as specified in Section 4.2 below.

   Some ECN experiments use ECN with packets where ECN has not been used
   previously, specifically TCP control packets and retransmissions; see Section 4.3 below.  The new middlebox behavior requirements in that
   section are of particular importance.  In general, any system or
   protocol that inspects or monitors network traffic SHOULD be prepared
   to encounter ECN usage on packets and traffic that currently do not
   use ECN.

   ECN field handling requirements for tunnel encapsulation and
   decapsulation are specified in [RFC6040], which is in the process of
   being updated by [ECN-SHIM ].  Related guidance for encapsulations
   whose outer headers are not IP headers can be found in [ECN-ENCAP ].
   These requirements and guidance apply to all traffic, including
   traffic that is part of any ECN experiment.

### 2.3. Operational and Management Considerations

   Changes in network traffic behavior that result from ECN
   experimentation are likely to impact network operations and
   management.  Designers of ECN experiments are expected to anticipate
   possible impacts and consider how they may be dealt with.  Specific
   topics to consider include possible network management changes or
   extensions, monitoring of the experimental deployment, collection of
   data for evaluation of the experiment, and possible interactions with
   other protocols, particularly protocols that encapsulate network
   traffic.

   For further discussion, see [RFC5706]; the questions in Appendix A of
   RFC 5706 provide a concise survey of some important aspects to
   consider. 





### 4.1. Congestion Response Differences



   RFC 3168 specifies that senders respond identically to packet drops
   and ECN congestion indications.  ECN congestion indications are
   predominately originated by Active Queue Management (AQM) mechanisms
   in intermediate buffers.  AQM mechanisms are usually configured to
   maintain shorter queue lengths than non-AQM-based mechanisms,
   particularly non-AQM drop-based mechanisms such as tail-drop, as AQM
   mechanisms indicate congestion before the queue overflows.  While the
   occurrence of loss does not easily enable the receiver to determine
   if AQM is used, the receipt of an ECN CE mark conveys a strong
   likelihood that AQM was used to manage the bottleneck queue.  Hence,
   an ECN congestion indication communicates a higher likelihood than a
   dropped packet that a short queue exists at the network bottleneck
   node [TCP-ABE ].  This difference suggests that for congestion
   indicated by ECN, a different sender congestion response (e.g.,
   sender backs off by a smaller amount) may be appropriate by
   comparison to the sender response to congestion indicated by loss.
   However, Section 5 of RFC 3168 specifies that:

      Upon the receipt by an ECN-Capable transport of a single CE
      packet, the congestion control algorithms followed at the end-
      systems MUST be essentially the same as the congestion control
      response to a *single* dropped packet.

   This memo updates this text from RFC 3168 to allow the congestion
   control response (including the TCP Sender's congestion control
   response) to a CE-marked packet to differ from the response to a
   dropped packet, provided that the changes from RFC 3168 are 
   documented in an Experimental RFC in the IETF document stream.  The
   specific change to RFC 3168 is to insert the words "unless otherwise
   specified by an Experimental RFC in the IETF document stream" at the
   end of the sentence quoted above. RFC 4774 [RFC4774] quotes the above text from RFC 3168 as background,
   but it does not impose requirements based on that text.  Therefore,
   no update to RFC 4774 is required to enable this area of
   experimentation. Section 6.1.2 of RFC 3168 specifies that:

      If the sender receives an ECN-Echo (ECE) ACK packet (that is, an
      ACK packet with the ECN-Echo flag set in the TCP header), then the
      sender knows that congestion was encountered in the network on the
      path from the sender to the receiver.  The indication of
      congestion should be treated just as a congestion loss in
      non-ECN-Capable TCP.  That is, the TCP source halves the
      congestion window "cwnd" and reduces the slow start threshold
      "ssthresh".

   This memo also updates this text from RFC 3168 to allow the
   congestion control response (including the TCP Sender's congestion
   control response) to a CE-marked packet to differ from the response
   to a dropped packet, provided that the changes from RFC 3168 are
   documented in an Experimental RFC in the IETF document stream.  The
   specific change to RFC 3168 is to insert the words "Unless otherwise
   specified by an Experimental RFC in the IETF document stream" at the
   beginning of the second sentence quoted above.

### 4.2. Congestion Marking Differences

   Taken to its limit, an AQM algorithm that uses ECN congestion
   indications can be configured to maintain very shallow queues,
   thereby reducing network latency by comparison to maintaining a
   larger queue.  Significantly more aggressive sender responses to ECN
   are needed to make effective use of such very shallow queues;
   "Datacenter TCP (DCTCP)" [RFC8257] provides an example.  In this
   case, separate network node treatments are essential, both to prevent
   the aggressive low-latency traffic from starving conventional traffic
   (if present) and to prevent any conventional traffic disruption to
   any lower-latency service that uses the very shallow queues.  Use of
   different ECN codepoints is a promising means of identifying these
   two classes of traffic to network nodes; hence, this area of
   experimentation is based on the use of the ECT(1) codepoint to
   request ECN congestion marking behavior in the network that differs
   from ECT(0).  It is essential that any such change in ECN congestion
   marking behavior be counterbalanced by use of a different IETF-
   approved congestion response to CE marks at the sender, e.g., as
   proposed in [ECN-L4S ]. Section 5 of RFC 3168 specifies that "Routers treat the ECT(0) and
   ECT(1) codepoints as equivalent."

   This memo updates RFC 3168 to allow routers to treat the ECT(0) and
   ECT(1) codepoints differently, provided that the changes from RFC 
   3168 are documented in an Experimental RFC in the IETF document
   stream.  The specific change to RFC 3168 is to insert the words
   "unless otherwise specified by an Experimental RFC in the IETF
   document stream" at the end of the above sentence.

   When an AQM is configured to use ECN congestion indications to
   maintain a very shallow queue, congestion indications are marked on
   packets that would not have been dropped if ECN was not in use. Section 5 of RFC 3168 specifies that:

      For a router, the CE codepoint of an ECN-Capable packet SHOULD
      only be set if the router would otherwise have dropped the packet
      as an indication of congestion to the end nodes.  When the
      router's buffer is not yet full and the router is prepared to drop
      a packet to inform end nodes of incipient congestion, the router
      should first check to see if the ECT codepoint is set in that
      packet's IP header.  If so, then instead of dropping the packet,
      the router MAY instead set the CE codepoint in the IP header.

   This memo updates RFC 3168 to allow congestion indications that are
   not equivalent to drops, provided that the changes from RFC 3168 are
   documented in an Experimental RFC in the IETF document stream.  The
   specific change is to change "For a router" to "Unless otherwise
   specified by an Experimental RFC in the IETF document stream" at the
   beginning of the first sentence of the above paragraph.

   A larger update to RFC 3168 is necessary to enable sender usage of
   ECT(1) to request network congestion marking behavior that maintains
   very shallow queues at network nodes.  When using loss as a
   congestion signal, the number of signals provided should be reduced
   to a minimum; hence, only the presence or absence of congestion is
   communicated.  In contrast, ECN can provide a richer signal, e.g., to
   indicate the current level of congestion, without the disadvantage of
   a larger number of packet losses.  A proposed experiment in this
   area, Low Latency Low Loss Scalable throughput (L4S) [ECN-L4S ],
   significantly increases the CE marking probability for traffic marked
   as ECT(1) in a fashion that would interact badly with existing sender
   congestion response functionality because that functionality assumes
   that the network marks ECT packets as frequently as it would drop
   Not-ECT packets.  If network traffic that uses such a conventional 
   sender congestion response were to encounter L4S's increased marking
   probability (and hence rate) at a network bottleneck queue, the
   resulting traffic throughput is likely to be much less than intended
   for the level of congestion at the bottleneck queue.

   This memo updates RFC 3168 to remove that interaction for ECT(1).
   The specific update to Section 5 of RFC 3168 is to replace the
   following two paragraphs:

      Senders are free to use either the ECT(0) or the ECT(1) codepoint
      to indicate ECT, on a packet-by-packet basis.

      The use of both the two codepoints for ECT, ECT(0) and ECT(1), is
      motivated primarily by the desire to allow mechanisms for the data
      sender to verify that network elements are not erasing the CE
      codepoint, and that data receivers are properly reporting to the
      sender the receipt of packets with the CE codepoint set, as
      required by the transport protocol.  Guidelines for the senders
      and receivers to differentiate between the ECT(0) and ECT(1)
      codepoints will be addressed in separate documents, for each
      transport protocol.  In particular, this document does not address
      mechanisms for TCP end-nodes to differentiate between the ECT(0)
      and ECT(1) codepoints.  Protocols and senders that only require a
      single ECT codepoint SHOULD use ECT(0).

   with this paragraph:

      Protocols and senders MUST use the ECT(0) codepoint to indicate
      ECT unless otherwise specified by an Experimental RFC in the IETF
      document stream.  Protocols and senders MUST NOT use the ECT(1)
      codepoint to indicate ECT unless otherwise specified by an
      Experimental RFC in the IETF document stream.  Guidelines for
      senders and receivers to differentiate between the ECT(0) and
      ECT(1) codepoints will be addressed in separate documents, for
      each transport protocol.  In particular, this document does not
      address mechanisms for TCP end-nodes to differentiate between the
      ECT(0) and ECT(1) codepoints.

   Congestion Marking Differences experiments SHOULD modify the network
   behavior for traffic marked as ECT(1) rather than ECT(0) if network
   behavior for only one ECT codepoint is modified.  Congestion Marking
   Differences experiments MUST NOT modify the network behavior for
   traffic marked as ECT(0) in a fashion that requires changes to the
   sender congestion response to obtain desired network behavior.  If a
   Congestion Marking Differences experiment modifies the network
   behavior for traffic marked as ECT(1), e.g., CE-marking behavior, in 
   a fashion that requires changes to the sender congestion response to
   obtain desired network behavior, then the Experimental RFC in the
   IETF document stream for that experiment MUST specify:

   o  The sender congestion response to CE marking in the network, and

   o  Router behavior changes, or the absence thereof, in forwarding CE-
      marked packets that are part of the experiment.

   In addition, this memo updates RFC 3168 to remove discussion of the
   ECN nonce, as noted in Section 3 above.

### 4.3. TCP Control Packets and Retransmissions

   With the successful use of ECN for traffic in large portions of the
   Internet, there is interest in extending the benefits of ECN to TCP
   control packets (e.g., SYNs) and retransmitted packets, e.g., as
   proposed by ECN++ [ECN-TCP ]. RFC 3168 prohibits use of ECN for TCP control packets and
   retransmitted packets in a number of places:

   o  Section 5.2: "To ensure the reliable delivery of the congestion
      indication of the CE codepoint, an ECT codepoint MUST NOT be set
      in a packet unless the loss of that packet in the network would be
      detected by the end nodes and interpreted as an indication of
      congestion."

   o  Section 6.1.1: "A host MUST NOT set ECT on SYN or SYN-ACK packets"

   o  Section 6.1.4: "...pure acknowledgement packets (e.g., packets
      that do not contain any accompanying data) MUST be sent with the
      not-ECT codepoint."

   o  Section 6.1.5: "This document specifies ECN-capable TCP
      implementations MUST NOT set either ECT codepoint (ECT(0) or
      ECT(1)) in the IP header for retransmitted data packets, and that
      the TCP data receiver SHOULD ignore the ECN field on arriving data
      packets that are outside of the receiver's current window."

   o  Section 6.1.6: "...the TCP data sender MUST NOT set either an ECT
      codepoint or the CWR bit on window probe packets.

   This memo updates RFC 3168 to allow the use of ECT codepoints on SYN
   and SYN-ACK packets, pure acknowledgement packets, window probe
   packets, and retransmissions of packets that were originally sent
   with an ECT codepoint, provided that the changes from RFC 3168 are
   documented in an Experimental RFC in the IETF document stream.  The 
   specific change to RFC 3168 is to insert the words "unless otherwise
   specified by an Experimental RFC in the IETF document stream" at the
   end of each sentence quoted above.

   In addition, beyond requiring TCP senders not to set ECT on TCP
   control packets and retransmitted packets, RFC 3168 is silent on
   whether it is appropriate for a network element, e.g., a firewall, to
   discard such a packet as invalid.  For this area of ECN
   experimentation to be useful, middleboxes ought not to do that;
   therefore, RFC 3168 is updated by adding the following text to the
   end of Section 6.1.1.1 on Middlebox Issues:

      Unless otherwise specified by an Experimental RFC in the IETF
      document stream, middleboxes SHOULD NOT discard TCP control
      packets and retransmitted TCP packets solely because the ECN field
      in the IP header does not contain Not-ECT.  An exception to this
      requirement occurs in responding to an attack that uses ECN
      codepoints other than Not-ECT.  For example, as part of the
      response, it may be appropriate to drop ECT-marked TCP SYN packets
      with higher probability than TCP SYN packets marked with Not-ECT.
      Any such exceptional discarding of TCP control packets and
      retransmitted TCP packets in response to an attack MUST NOT be
      done routinely in the absence of an attack and SHOULD only be done
      if it is determined that the use of ECN is contributing to the
      attack.

## 5. ECN for RTP Updates to RFC 6679



   RFC 6679 [RFC6679] specifies use of ECN for RTP traffic; it allows
   use of both the ECT(0) and ECT(1) codepoints and provides the
   following guidance on use of these codepoints in Section 7.3.1:

      The sender SHOULD mark packets as ECT(0) unless the receiver
      expresses a preference for ECT(1) or for a random ECT value using
      the "ect" parameter in the "a=ecn-capable-rtp:" attribute.

   The Congestion Marking Differences area of experimentation increases
   the potential consequences of using ECT(1) instead of ECT(0); hence,
   the above guidance is updated by adding the following two sentences:

      Random ECT values MUST NOT be used, as that may expose RTP to
      differences in network treatment of traffic marked with ECT(1) and
      ECT(0) and differences in associated endpoint congestion
      responses.  In addition, ECT(0) MUST be used unless otherwise
      specified in an Experimental RFC in the IETF document stream. 



   Section 7.3.3 of RFC 6679 specifies RTP's response to receipt of
   CE-marked packets as being identical to the response to dropped
   packets:

      The reception of RTP packets with ECN-CE marks in the IP header is
      a notification that congestion is being experienced.  The default
      reaction on the reception of these ECN-CE-marked packets MUST be
      to provide the congestion control algorithm with a congestion
      notification that triggers the algorithm to react as if packet
      loss had occurred.  There should be no difference in congestion
      response if ECN-CE marks or packet drops are detected.

   In support of Congestion Response Differences experimentation, this
   memo updates this text in a fashion similar to RFC 3168 to allow the
   RTP congestion control response to a CE-marked packet to differ from
   the response to a dropped packet, provided that the changes from RFC 
   6679 are documented in an Experimental RFC in the IETF document
   stream.  The specific change to RFC 6679 is to insert the words
   "Unless otherwise specified by an Experimental RFC in the IETF
   document stream" and reformat the last two sentences to be subject to
   that condition; that is:

      The reception of RTP packets with ECN-CE marks in the IP header is
      a notification that congestion is being experienced.  Unless
      otherwise specified by an Experimental RFC in the IETF document
      stream:

      *  The default reaction on the reception of these ECN-CE-marked
         packets MUST be to provide the congestion control algorithm
         with a congestion notification that triggers the algorithm to
         react as if packet loss had occurred.

      *  There should be no difference in congestion response if ECN-CE
         marks or packet drops are detected.

   The second sentence of the immediately following paragraph in Section 7.3.3 of RFC 6679 requires a related update:

      Other reactions to ECN-CE may be specified in the future,
      following IETF Review.  Detailed designs of such alternative
      reactions MUST be specified in a Standards Track RFC and be
      reviewed to ensure they are safe for deployment under any
      restrictions specified.

   The update is to change "Standards Track RFC" to "Standards Track RFC
   or Experimental RFC in the IETF document stream" for consistency with
   the first update. 





## 9. References





### 9.1. Normative References

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997,
              <https://www.rfc-editor.org/info/rfc2119>.

   [RFC2914]  Floyd, S., "Congestion Control Principles", BCP 41,RFC 2914, DOI 10.17487/RFC2914, September 2000,
              <https://www.rfc-editor.org/info/rfc2914>.

   [RFC3168]  Ramakrishnan, K., Floyd, S., and D. Black, "The Addition
              of Explicit Congestion Notification (ECN) to IP",RFC 3168, DOI 10.17487/RFC3168, September 2001,
              <https://www.rfc-editor.org/info/rfc3168>.

   [RFC3540]  Spring, N., Wetherall, D., and D. Ely, "Robust Explicit
              Congestion Notification (ECN) Signaling with Nonces",RFC 3540, DOI 10.17487/RFC3540, June 2003,
              <https://www.rfc-editor.org/info/rfc3540>.

   [RFC4341]  Floyd, S. and E. Kohler, "Profile for Datagram Congestion
              Control Protocol (DCCP) Congestion Control ID 2: TCP-like
              Congestion Control", RFC 4341, DOI 10.17487/RFC4341, March
              2006, <https://www.rfc-editor.org/info/rfc4341>.

   [RFC4342]  Floyd, S., Kohler, E., and J. Padhye, "Profile for
              Datagram Congestion Control Protocol (DCCP) Congestion
              Control ID 3: TCP-Friendly Rate Control (TFRC)", RFC 4342,
              DOI 10.17487/RFC4342, March 2006,
              <https://www.rfc-editor.org/info/rfc4342>.

   [RFC5622]  Floyd, S. and E. Kohler, "Profile for Datagram Congestion
              Control Protocol (DCCP) Congestion ID 4: TCP-Friendly Rate
              Control for Small Packets (TFRC-SP)", RFC 5622,
              DOI 10.17487/RFC5622, August 2009,
              <https://www.rfc-editor.org/info/rfc5622>.

   [RFC6679]  Westerlund, M., Johansson, I., Perkins, C., O'Hanlon, P.,
              and K. Carlberg, "Explicit Congestion Notification (ECN)
              for RTP over UDP", RFC 6679, DOI 10.17487/RFC6679, August
              2012, <https://www.rfc-editor.org/info/rfc6679>. 
   [RFC8174]  Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 
              2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174,
              May 2017, <https://www.rfc-editor.org/info/rfc8174>.

### 9.2. Informative References

   [ECN-ENCAP ]
              Briscoe, B., Kaippallimalil, J., and P. Thaler,
              "Guidelines for Adding Congestion Notification to
              Protocols that Encapsulate IP", Work in Progress,draft-ietf-tsvwg-ecn-encap-guidelines-09, July 2017.

   [ECN-EXPERIMENT ]
              Khademi, N., Welzl, M., Armitage, G., and G. Fairhurst,
              "Updating the Explicit Congestion Notification (ECN)
              Specification to Allow IETF Experimentation", Work in
              Progress, draft-khademi-tsvwg-ecn-response-01, July 2016.

   [ECN-L4S ]  Schepper, K. and B. Briscoe, "Identifying Modified
              Explicit Congestion Notification (ECN) Semantics for
              Ultra-Low Queuing Delay", Work in Progress,draft-ietf-tsvwg-ecn-l4s-id-01, October 2017.

   [ECN-SHIM ] Briscoe, B., "Propagating Explicit Congestion Notification
              Across IP Tunnel Headers Separated by a Shim", Work in
              Progress, draft-ietf-tsvwg-rfc6040update-shim-05, November
              2017.

   [ECN-TCP ]  Bagnulo, M. and B. Briscoe, "ECN++: Adding Explicit
              Congestion Notification (ECN) to TCP Control Packets",
              Work in Progress, draft-ietf-tcpm-generalized-ecn-02,
              October 2017.

   [ECN-TRILL ]
              Eastlake, D. and B. Briscoe, "TRILL: ECN (Explicit
              Congestion Notification) Support", Work in Progress,draft-ietf-trill-ecn-support-04, November 2017.

   [RFC4774]  Floyd, S., "Specifying Alternate Semantics for the
              Explicit Congestion Notification (ECN) Field", BCP 124,RFC 4774, DOI 10.17487/RFC4774, November 2006,
              <https://www.rfc-editor.org/info/rfc4774>.

   [RFC4844]  Daigle, L., Ed. and Internet Architecture Board, "The RFC
              Series and RFC Editor", RFC 4844, DOI 10.17487/RFC4844,
              July 2007, <https://www.rfc-editor.org/info/rfc4844>. 
   [RFC5706]  Harrington, D., "Guidelines for Considering Operations and
              Management of New Protocols and Protocol Extensions",RFC 5706, DOI 10.17487/RFC5706, November 2009,
              <https://www.rfc-editor.org/info/rfc5706>.

   [RFC6040]  Briscoe, B., "Tunnelling of Explicit Congestion
              Notification", RFC 6040, DOI 10.17487/RFC6040, November
              2010, <https://www.rfc-editor.org/info/rfc6040>.

   [RFC6660]  Briscoe, B., Moncaster, T., and M. Menth, "Encoding Three
              Pre-Congestion Notification (PCN) States in the IP Header
              Using a Single Diffserv Codepoint (DSCP)", RFC 6660,
              DOI 10.17487/RFC6660, July 2012,
              <https://www.rfc-editor.org/info/rfc6660>.

   [RFC8257]  Bensley, S., Thaler, D., Balasubramanian, P., Eggert, L.,
              and G. Judd, "Data Center TCP (DCTCP): TCP Congestion
              Control for Data Centers", RFC 8257, DOI 10.17487/RFC8257,
              October 2017, <https://www.rfc-editor.org/info/rfc8257>.

   [TCP-ABE ]  Khademi, N., Welzl, M., Armitage, G., and G. Fairhurst,
              "TCP Alternative Backoff with ECN (ABE)", Work in
              Progress, draft-ietf-tcpm-alternativebackoff-ecn-05,
              December 2017.

   [Trammell15]
              Trammell, B., Kuehlewind, M., Boppart, D., Learmonth, I.,
              Fairhurst, G., and R. Scheffenegger, "Enabling Internet-
              Wide Deployment of Explicit Congestion Notification", In
              Conference Proceedings of Passive and Active Measurement
              (PAM), pp. 193-205, March 2015,
              <https://doi.org/10.1007/978-3-319-15509-8_15>. 

# Referenced Sections from RFC 5961: Improving TCP's Robustness to Blind In-Window Attacks

The following sections were referenced. Remaining sections are not included.

### 3.2. Mitigation

   [RFC0793] currently requires handling of a segment with the RST bit
   when in a synchronized state to be processed as follows:

   1) If the RST bit is set and the sequence number is outside the
      current receive window (SEG.SEQ <= RCV.NXT || SEG.SEQ > RCV.NXT+
      RCV.WND), silently drop the segment.

   2) If the RST bit is set and the sequence number is acceptable, i.e.,
      (RCV.NXT <= SEG.SEQ < RCV.NXT+RCV.WND), then reset the connection.

   Instead, implementations SHOULD implement the following steps in
   place of those specified in [RFC0793] (as listed above).

   1) If the RST bit is set and the sequence number is outside the
      current receive window, silently drop the segment.

   2) If the RST bit is set and the sequence number exactly matches the
      next expected sequence number (RCV.NXT), then TCP MUST reset the
      connection. 
   3) If the RST bit is set and the sequence number does not exactly
      match the next expected sequence value, yet is within the current
      receive window (RCV.NXT < SEG.SEQ < RCV.NXT+RCV.WND), TCP MUST
      send an acknowledgment (challenge ACK):

      <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>

      After sending the challenge ACK, TCP MUST drop the unacceptable
      segment and stop processing the incoming packet further.  Further
      segments destined to this connection will be processed as normal.

   The modified RST segment processing would thus become:

   In all states except SYN-SENT, all reset (RST) segments are validated
   by checking their SEQ-fields [sequence numbers].  A reset is valid if
   its sequence number exactly matches the next expected sequence
   number.  If the RST arrives and its sequence number field does NOT
   match the next expected sequence number but is within the window,
   then the receiver should generate an ACK.  In all other cases, where
   the SEQ-field does not match and is outside the window, the receiver
   MUST silently discard the segment.

   In the SYN-SENT state (a RST received in response to an initial SYN),
   the RST is acceptable if the ACK field acknowledges the SYN.  In all
   other cases the receiver MUST silently discard the segment.

   With the above slight change to the TCP state machine, it becomes
   much harder for an attacker to generate an acceptable reset segment.

   In cases where the remote peer did generate a RST, but it fails to
   meet the above criteria (the RST sequence number was within the
   window but NOT the exact expected sequence number), when the
   challenge ACK is sent back, it will no longer have the transmission
   control block (TCB) related to this connection and hence as per
   [RFC0793], the remote peer will send a second RST back.  The sequence
   number of the second RST is derived from the acknowledgment number of
   the incoming ACK.  This second RST, if it reaches the sender, will
   cause the connection to be aborted since the sequence number would
   now be an exact match.

   A valid RST received out of order would still generate a challenge
   ACK in response.  If this RST happens to be a genuine one, the other
   end would send an RST with an exact sequence number match that would
   cause the connection to be dropped.

   Note that the above mitigation may cause a non-amplification ACK
   exchange.  This concern is discussed in Section 10. 





### 5.2. Mitigation

   All TCP stacks MAY implement the following mitigation.  TCP stacks
   that implement this mitigation MUST add an additional input check to
   any incoming segment.  The ACK value is considered acceptable only if
   it is in the range of ((SND.UNA - MAX.SND.WND) <= SEG.ACK <=
   SND.NXT).  All incoming segments whose ACK value doesn't satisfy the
   above condition MUST be discarded and an ACK sent back.  It needs to
   be noted that RFC 793 on page 72 (fifth check) says: "If the ACK is a
   duplicate (SEG.ACK < SND.UNA), it can be ignored.  If the ACK
   acknowledges something not yet sent (SEG.ACK > SND.NXT) then send an
   ACK, drop the segment, and return".  The "ignored" above implies that
   the processing of the incoming data segment continues, which means
   the ACK value is treated as acceptable.  This mitigation makes the
   ACK check more stringent since any ACK < SND.UNA wouldn't be
   accepted, instead only ACKs that are in the range ((SND.UNA -
   MAX.SND.WND) <= SEG.ACK <= SND.NXT) get through.

   A new state variable MAX.SND.WND is defined as the largest window
   that the local sender has ever received from its peer.  This window
   may be scaled to a value larger than 65,535 bytes ([RFC1323]).  This
   small check will reduce the vulnerability to an attacker guessing a 
   valid sequence number, since, not only one must guess the in-window
   sequence number, but also guess a proper ACK value within a scoped
   range.  This mitigation reduces, but does not eliminate, the ability
   to generate false segments.  It does however reduce the probability
   that invalid data will be injected.

   Implementations can also chose to hard code the MAX.SND.WND value to
   the maximum permissible window size, i.e., 65535 in the absence of
   window scaling.  In the presence of the window scaling option, the
   value becomes (MAX.SND.WND << Snd.Wind.Scale).

   This mitigation also helps in improving robustness on accepting
   spoofed FIN segments (FIN attacks).  Among other things, this
   mitigation requires that the attacker also needs to get the
   acknowledgment number to fall in the range mentioned above in order
   to successfully spoof a FIN segment leading to the closure of the
   connection.  Thus, this mitigation greatly improves the robustness to
   spoofed FIN segments.

   Note that the above mitigation may cause a non-amplification ACK
   exchange.  This concern is discussed in Section 10.

## 8. Backward Compatibility and Other Considerations

   All of the new required mitigation techniques in this document are
   totally compatible with existing ([RFC0793]) compliant TCP
   implementations as this document introduces no new assumptions or
   conditions.

   There is a corner scenario in the above mitigations that will require
   more than one round-trip time to successfully abort the connection as
   per the figure below.  This scenario is similar to the one in which
   the original RST was lost in the network. 
          TCP A                                                 TCP B
   1.a. ESTAB        <-- <SEQ=300><ACK=101><CTL=ACK><DATA> <--  ESTAB
     b. (delayed)    ... <SEQ=400><ACK=101><CTL=ACK><DATA> <--  ESTAB
     c. (in flight)  ... <SEQ=500><ACK=101><CTL=RST>       <--  CLOSED
   2.   ESTAB        --> <SEQ=101><ACK=400><CTL=ACK>       -->  CLOSED
       (ACK for 1.a)
                     ... <SEQ=400><ACK=0><CTL=RST>         <--  CLOSED
   3.   CHALLENGE    --> <SEQ=101><ACK=400><CTL=ACK>       -->  CLOSED
        (for 1.c)
                     ... <SEQ=400><ACK=0><CTL=RST>         <--  RESPONSE
   4.a. ESTAB        <-- <SEQ=400><ACK=101><CTL=ACK><DATA> 1.b reaches A
     b. ESTAB        --> <SEQ=101><ACK=500><CTL=ACK>
     c. (in flight)  ... <SEQ=500><ACK=0><CTL=RST>         <--  CLOSED
   5.   RESPONSE arrives at A, but dropped since its outside of window.
   6.   ESTAB        <-- <SEQ=500><ACK=0><CTL=RST>         4.c reaches A
   7.   CLOSED                                                   CLOSED

   For the mitigation to be maximally effective against the
   vulnerabilities discussed in this document, both ends of the TCP
   connection need to have the fix.  Although, having the mitigations at
   one end might prevent that end from being exposed to the attack, the
   connection is still vulnerable at the other end.

# Referenced Sections from RFC 9293: Transmission Control Protocol (TCP)

The following sections were referenced. Remaining sections are not included.

## 3. Functional Specification





### 3.1. Header Format

   TCP segments are sent as internet datagrams.  The Internet Protocol
   (IP) header carries several information fields, including the source
   and destination host addresses [1] [13].  A TCP header follows the IP
   headers, supplying information specific to TCP.  This division allows
   for the existence of host-level protocols other than TCP.  In the
   early development of the Internet suite of protocols, the IP header
   fields had been a part of TCP.

   This document describes TCP, which uses TCP headers.

   A TCP header, followed by any user data in the segment, is formatted
   as follows, using the style from [66]:

       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |          Source Port          |       Destination Port        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                        Sequence Number                        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                    Acknowledgment Number                      |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |  Data |       |C|E|U|A|P|R|S|F|                               |
      | Offset| Rsrvd |W|C|R|C|S|S|Y|I|            Window             |
      |       |       |R|E|G|K|H|T|N|N|                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |           Checksum            |         Urgent Pointer        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                           [Options]                           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               :
      :                             Data                              :
      :                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

             Note that one tick mark represents one bit position.

                        Figure 1: TCP Header Format

   where:

   Source Port:  16 bits

     The source port number.

   Destination Port:  16 bits

     The destination port number.

   Sequence Number:  32 bits

     The sequence number of the first data octet in this segment (except
     when the SYN flag is set).  If SYN is set, the sequence number is
     the initial sequence number (ISN) and the first data octet is
     ISN+1.

   Acknowledgment Number:  32 bits

     If the ACK control bit is set, this field contains the value of the
     next sequence number the sender of the segment is expecting to
     receive.  Once a connection is established, this is always sent.

   Data Offset (DOffset):  4 bits

     The number of 32-bit words in the TCP header.  This indicates where
     the data begins.  The TCP header (even one including options) is an
     integer multiple of 32 bits long.

   Reserved (Rsrvd):  4 bits

     A set of control bits reserved for future use.  Must be zero in
     generated segments and must be ignored in received segments if the
     corresponding future features are not implemented by the sending or
     receiving host.

   Control bits:  The control bits are also known as "flags".
     Assignment is managed by IANA from the "TCP Header Flags" registry
     [62].  The currently assigned control bits are CWR, ECE, URG, ACK,
     PSH, RST, SYN, and FIN.

     CWR:  1 bit

         Congestion Window Reduced (see [6]).

     ECE:  1 bit

         ECN-Echo (see [6]).

     URG:  1 bit

         Urgent pointer field is significant.

     ACK:  1 bit

         Acknowledgment field is significant.

     PSH:  1 bit

         Push function (see the Send Call description in Section 3.9.1).

     RST:  1 bit

         Reset the connection.

     SYN:  1 bit

         Synchronize sequence numbers.

     FIN:  1 bit

         No more data from sender.

   Window:  16 bits

     The number of data octets beginning with the one indicated in the
     acknowledgment field that the sender of this segment is willing to
     accept.  The value is shifted when the window scaling extension is
     used [47].

     The window size MUST be treated as an unsigned number, or else
     large window sizes will appear like negative windows and TCP will
     not work (MUST-1).  It is RECOMMENDED that implementations will
     reserve 32-bit fields for the send and receive window sizes in the
     connection record and do all window computations with 32 bits (REC-
     1).

   Checksum:  16 bits

     The checksum field is the 16-bit ones' complement of the ones'
     complement sum of all 16-bit words in the header and text.  The
     checksum computation needs to ensure the 16-bit alignment of the
     data being summed.  If a segment contains an odd number of header
     and text octets, alignment can be achieved by padding the last
     octet with zeros on its right to form a 16-bit word for checksum
     purposes.  The pad is not transmitted as part of the segment.
     While computing the checksum, the checksum field itself is replaced
     with zeros.

     The checksum also covers a pseudo-header (Figure 2) conceptually
     prefixed to the TCP header.  The pseudo-header is 96 bits for IPv4
     and 320 bits for IPv6.  Including the pseudo-header in the checksum
     gives the TCP connection protection against misrouted segments.
     This information is carried in IP headers and is transferred across
     the TCP/network interface in the arguments or results of calls by
     the TCP implementation on the IP layer.

                     +--------+--------+--------+--------+
                     |           Source Address          |
                     +--------+--------+--------+--------+
                     |         Destination Address       |
                     +--------+--------+--------+--------+
                     |  zero  |  PTCL  |    TCP Length   |
                     +--------+--------+--------+--------+

                         Figure 2: IPv4 Pseudo-header

     Pseudo-header components for IPv4:
       Source Address:  the IPv4 source address in network byte order

       Destination Address:  the IPv4 destination address in network
          byte order

       zero:  bits set to zero

       PTCL:  the protocol number from the IP header

       TCP Length:  the TCP header length plus the data length in octets
          (this is not an explicitly transmitted quantity but is
          computed), and it does not count the 12 octets of the pseudo-
          header.

     For IPv6, the pseudo-header is defined in Section 8.1 of RFC 8200     [13] and contains the IPv6 Source Address and Destination Address,
     an Upper-Layer Packet Length (a 32-bit value otherwise equivalent
     to TCP Length in the IPv4 pseudo-header), three bytes of zero
     padding, and a Next Header value, which differs from the IPv6
     header value if there are extension headers present between IPv6
     and TCP.

     The TCP checksum is never optional.  The sender MUST generate it
     (MUST-2) and the receiver MUST check it (MUST-3).

   Urgent Pointer:  16 bits

     This field communicates the current value of the urgent pointer as
     a positive offset from the sequence number in this segment.  The
     urgent pointer points to the sequence number of the octet following
     the urgent data.  This field is only to be interpreted in segments
     with the URG control bit set.

   Options:  [TCP Option]; size(Options) == (DOffset-5)*32; present only
     when DOffset > 5.  Note that this size expression also includes any
     padding trailing the actual options present.

     Options may occupy space at the end of the TCP header and are a
     multiple of 8 bits in length.  All options are included in the
     checksum.  An option may begin on any octet boundary.  There are
     two cases for the format of an option:

     Case 1:  A single octet of option-kind.

     Case 2:  An octet of option-kind (Kind), an octet of option-length,
        and the actual option-data octets.

     The option-length counts the two octets of option-kind and option-
     length as well as the option-data octets.

     Note that the list of options may be shorter than the Data Offset
     field might imply.  The content of the header beyond the End of
     Option List Option MUST be header padding of zeros (MUST-69).

     The list of all currently defined options is managed by IANA [62],
     and each option is defined in other RFCs, as indicated there.  That
     set includes experimental options that can be extended to support
     multiple concurrent usages [45].

     A given TCP implementation can support any currently defined
     options, but the following options MUST be supported (MUST-4 --
     note Maximum Segment Size Option support is also part of MUST-14 in Section 3.7.1):

               +======+========+============================+
               | Kind | Length | Meaning                    |
               +======+========+============================+
               | 0    | -      | End of Option List Option. |
               +------+--------+----------------------------+
               | 1    | -      | No-Operation.              |
               +------+--------+----------------------------+
               | 2    | 4      | Maximum Segment Size.      |
               +------+--------+----------------------------+

                       Table 1: Mandatory Option Set

     These options are specified in detail in Section 3.2.

     A TCP implementation MUST be able to receive a TCP Option in any
     segment (MUST-5).

     A TCP implementation MUST (MUST-6) ignore without error any TCP
     Option it does not implement, assuming that the option has a length
     field.  All TCP Options except End of Option List Option (EOL) and
     No-Operation (NOP) MUST have length fields, including all future
     options (MUST-68).  TCP implementations MUST be prepared to handle
     an illegal option length (e.g., zero); a suggested procedure is to
     reset the connection and log the error cause (MUST-7).

     Note: There is ongoing work to extend the space available for TCP
     Options, such as [65].

   Data:  variable length

     User data carried by the TCP segment.

### 3.2. Specific Option Definitions

   A TCP Option, in the mandatory option set, is one of an End of Option
   List Option, a No-Operation Option, or a Maximum Segment Size Option.

   An End of Option List Option is formatted as follows:

       0
       0 1 2 3 4 5 6 7
      +-+-+-+-+-+-+-+-+
      |       0       |
      +-+-+-+-+-+-+-+-+

   where:

   Kind:  1 byte; Kind == 0.

     This option code indicates the end of the option list.  This might
     not coincide with the end of the TCP header according to the Data
     Offset field.  This is used at the end of all options, not the end
     of each option, and need only be used if the end of the options
     would not otherwise coincide with the end of the TCP header.

   A No-Operation Option is formatted as follows:

       0
       0 1 2 3 4 5 6 7
      +-+-+-+-+-+-+-+-+
      |       1       |
      +-+-+-+-+-+-+-+-+

   where:

   Kind:  1 byte; Kind == 1.

     This option code can be used between options, for example, to align
     the beginning of a subsequent option on a word boundary.  There is
     no guarantee that senders will use this option, so receivers MUST
     be prepared to process options even if they do not begin on a word
     boundary (MUST-64).

   A Maximum Segment Size Option is formatted as follows:

       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |       2       |     Length    |   Maximum Segment Size (MSS)  |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   where:

   Kind:  1 byte; Kind == 2.

     If this option is present, then it communicates the maximum receive
     segment size at the TCP endpoint that sends this segment.  This
     value is limited by the IP reassembly limit.  This field may be
     sent in the initial connection request (i.e., in segments with the
     SYN control bit set) and MUST NOT be sent in other segments (MUST-
     65).  If this option is not used, any segment size is allowed.  A
     more complete description of this option is provided in Section 3.7.1.

   Length:  1 byte; Length == 4.

     Length of the option in bytes.

   Maximum Segment Size (MSS):  2 bytes.

     The maximum receive segment size at the TCP endpoint that sends
     this segment.

#### 3.2.1. Other Common Options

   Additional RFCs define some other commonly used options that are
   recommended to implement for high performance but are not necessary
   for basic TCP interoperability.  These are the TCP Selective
   Acknowledgment (SACK) Option [22] [26], TCP Timestamp (TS) Option
   [47], and TCP Window Scale (WS) Option [47].

#### 3.2.2. Experimental TCP Options

   Experimental TCP Option values are defined in [30], and [45]
   describes the current recommended usage for these experimental
   values.

### 3.3. TCP Terminology Overview

   This section includes an overview of key terms needed to understand
   the detailed protocol operation in the rest of the document.  There
   is a glossary of terms in Section 4.

#### 3.3.1. Key Connection State Variables

   Before we can discuss the operation of the TCP implementation in
   detail, we need to introduce some detailed terminology.  The
   maintenance of a TCP connection requires maintaining state for
   several variables.  We conceive of these variables being stored in a
   connection record called a Transmission Control Block or TCB.  Among
   the variables stored in the TCB are the local and remote IP addresses
   and port numbers, the IP security level, and compartment of the
   connection (see Appendix A.1), pointers to the user's send and
   receive buffers, pointers to the retransmit queue and to the current
   segment.  In addition, several variables relating to the send and
   receive sequence numbers are stored in the TCB.

    +==========+=====================================================+
    | Variable | Description                                         |
    +==========+=====================================================+
    | SND.UNA  | send unacknowledged                                 |
    +----------+-----------------------------------------------------+
    | SND.NXT  | send next                                           |
    +----------+-----------------------------------------------------+
    | SND.WND  | send window                                         |
    +----------+-----------------------------------------------------+
    | SND.UP   | send urgent pointer                                 |
    +----------+-----------------------------------------------------+
    | SND.WL1  | segment sequence number used for last window update |
    +----------+-----------------------------------------------------+
    | SND.WL2  | segment acknowledgment number used for last window  |
    |          | update                                              |
    +----------+-----------------------------------------------------+
    | ISS      | initial send sequence number                        |
    +----------+-----------------------------------------------------+

                     Table 2: Send Sequence Variables

              +==========+=================================+
              | Variable | Description                     |
              +==========+=================================+
              | RCV.NXT  | receive next                    |
              +----------+---------------------------------+
              | RCV.WND  | receive window                  |
              +----------+---------------------------------+
              | RCV.UP   | receive urgent pointer          |
              +----------+---------------------------------+
              | IRS      | initial receive sequence number |
              +----------+---------------------------------+

                   Table 3: Receive Sequence Variables

   The following diagrams may help to relate some of these variables to
   the sequence space.

                      1         2          3          4
                 ----------|----------|----------|----------
                        SND.UNA    SND.NXT    SND.UNA
                                             +SND.WND

           1 - old sequence numbers that have been acknowledged
           2 - sequence numbers of unacknowledged data
           3 - sequence numbers allowed for new data transmission
           4 - future sequence numbers that are not yet allowed

                       Figure 3: Send Sequence Space

   The send window is the portion of the sequence space labeled 3 in
   Figure 3.

                          1          2          3
                      ----------|----------|----------
                             RCV.NXT    RCV.NXT
                                       +RCV.WND

           1 - old sequence numbers that have been acknowledged
           2 - sequence numbers allowed for new reception
           3 - future sequence numbers that are not yet allowed

                      Figure 4: Receive Sequence Space

   The receive window is the portion of the sequence space labeled 2 in
   Figure 4.

   There are also some variables used frequently in the discussion that
   take their values from the fields of the current segment.

               +==========+===============================+
               | Variable | Description                   |
               +==========+===============================+
               | SEG.SEQ  | segment sequence number       |
               +----------+-------------------------------+
               | SEG.ACK  | segment acknowledgment number |
               +----------+-------------------------------+
               | SEG.LEN  | segment length                |
               +----------+-------------------------------+
               | SEG.WND  | segment window                |
               +----------+-------------------------------+
               | SEG.UP   | segment urgent pointer        |
               +----------+-------------------------------+

                    Table 4: Current Segment Variables

#### 3.3.2. State Machine Overview

   A connection progresses through a series of states during its
   lifetime.  The states are: LISTEN, SYN-SENT, SYN-RECEIVED,
   ESTABLISHED, FIN-WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK,
   TIME-WAIT, and the fictional state CLOSED.  CLOSED is fictional
   because it represents the state when there is no TCB, and therefore,
   no connection.  Briefly the meanings of the states are:

   LISTEN -  represents waiting for a connection request from any remote
      TCP peer and port.

   SYN-SENT -  represents waiting for a matching connection request
      after having sent a connection request.

   SYN-RECEIVED -  represents waiting for a confirming connection
      request acknowledgment after having both received and sent a
      connection request.

   ESTABLISHED -  represents an open connection, data received can be
      delivered to the user.  The normal state for the data transfer
      phase of the connection.

   FIN-WAIT-1 -  represents waiting for a connection termination request
      from the remote TCP peer, or an acknowledgment of the connection
      termination request previously sent.

   FIN-WAIT-2 -  represents waiting for a connection termination request
      from the remote TCP peer.

   CLOSE-WAIT -  represents waiting for a connection termination request
      from the local user.

   CLOSING -  represents waiting for a connection termination request
      acknowledgment from the remote TCP peer.

   LAST-ACK -  represents waiting for an acknowledgment of the
      connection termination request previously sent to the remote TCP
      peer (this termination request sent to the remote TCP peer already
      included an acknowledgment of the termination request sent from
      the remote TCP peer).

   TIME-WAIT -  represents waiting for enough time to pass to be sure
      the remote TCP peer received the acknowledgment of its connection
      termination request and to avoid new connections being impacted by
      delayed segments from previous connections.

   CLOSED -  represents no connection state at all.

   A TCP connection progresses from one state to another in response to
   events.  The events are the user calls, OPEN, SEND, RECEIVE, CLOSE,
   ABORT, and STATUS; the incoming segments, particularly those
   containing the SYN, ACK, RST, and FIN flags; and timeouts.

   The OPEN call specifies whether connection establishment is to be
   actively pursued, or to be passively waited for.

   A passive OPEN request means that the process wants to accept
   incoming connection requests, in contrast to an active OPEN
   attempting to initiate a connection.

   The state diagram in Figure 5 illustrates only state changes,
   together with the causing events and resulting actions, but addresses
   neither error conditions nor actions that are not connected with
   state changes.  In a later section, more detail is offered with
   respect to the reaction of the TCP implementation to events.  Some
   state names are abbreviated or hyphenated differently in the diagram
   from how they appear elsewhere in the document.

   NOTA BENE:  This diagram is only a summary and must not be taken as
      the total specification.  Many details are not included.

                               +---------+ ---------\      active OPEN
                               |  CLOSED |            \    -----------
                               +---------+<---------\   \   create TCB
                                 |     ^              \   \  snd SYN
                    passive OPEN |     |   CLOSE        \   \
                    ------------ |     | ----------       \   \
                     create TCB  |     | delete TCB         \   \
                                 V     |                      \   \
             rcv RST (note 1)  +---------+            CLOSE    |    \
          -------------------->|  LISTEN |          ---------- |     |
         /                     +---------+          delete TCB |     |
        /           rcv SYN      |     |     SEND              |     |
       /           -----------   |     |    -------            |     V
   +--------+      snd SYN,ACK  /       \   snd SYN          +--------+
   |        |<-----------------           ------------------>|        |
   |  SYN   |                    rcv SYN                     |  SYN   |
   |  RCVD  |<-----------------------------------------------|  SENT  |
   |        |                  snd SYN,ACK                   |        |
   |        |------------------           -------------------|        |
   +--------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +--------+
      |         --------------   |     |   -----------
      |                x         |     |     snd ACK
      |                          V     V
      |  CLOSE                 +---------+
      | -------                |  ESTAB  |
      | snd FIN                +---------+
      |                 CLOSE    |     |    rcv FIN
      V                -------   |     |    -------
   +---------+         snd FIN  /       \   snd ACK         +---------+
   |  FIN    |<----------------          ------------------>|  CLOSE  |
   | WAIT-1  |------------------                            |   WAIT  |
   +---------+          rcv FIN  \                          +---------+
     | rcv ACK of FIN   -------   |                          CLOSE  |
     | --------------   snd ACK   |                         ------- |
     V        x                   V                         snd FIN V
   +---------+               +---------+                    +---------+
   |FINWAIT-2|               | CLOSING |                    | LAST-ACK|
   +---------+               +---------+                    +---------+
     |              rcv ACK of FIN |                 rcv ACK of FIN |
     |  rcv FIN     -------------- |    Timeout=2MSL -------------- |
     |  -------            x       V    ------------        x       V
      \ snd ACK              +---------+delete TCB          +---------+
        -------------------->|TIME-WAIT|------------------->| CLOSED  |
                             +---------+                    +---------+

                   Figure 5: TCP Connection State Diagram

   The following notes apply to Figure 5:

   Note 1:  The transition from SYN-RECEIVED to LISTEN on receiving a
      RST is conditional on having reached SYN-RECEIVED after a passive
      OPEN.

   Note 2:  The figure omits a transition from FIN-WAIT-1 to TIME-WAIT
      if a FIN is received and the local FIN is also acknowledged.

   Note 3:  A RST can be sent from any state with a corresponding
      transition to TIME-WAIT (see [70] for rationale).  These
      transitions are not explicitly shown; otherwise, the diagram would
      become very difficult to read.  Similarly, receipt of a RST from
      any state results in a transition to LISTEN or CLOSED, though this
      is also omitted from the diagram for legibility.

### 3.4. Sequence Numbers

   A fundamental notion in the design is that every octet of data sent
   over a TCP connection has a sequence number.  Since every octet is
   sequenced, each of them can be acknowledged.  The acknowledgment
   mechanism employed is cumulative so that an acknowledgment of
   sequence number X indicates that all octets up to but not including X
   have been received.  This mechanism allows for straightforward
   duplicate detection in the presence of retransmission.  The numbering
   scheme of octets within a segment is as follows: the first data octet
   immediately following the header is the lowest numbered, and the
   following octets are numbered consecutively.

   It is essential to remember that the actual sequence number space is
   finite, though large.  This space ranges from 0 to 2^32 - 1.  Since
   the space is finite, all arithmetic dealing with sequence numbers
   must be performed modulo 2^32.  This unsigned arithmetic preserves
   the relationship of sequence numbers as they cycle from 2^32 - 1 to 0
   again.  There are some subtleties to computer modulo arithmetic, so
   great care should be taken in programming the comparison of such
   values.  The symbol "=<" means "less than or equal" (modulo 2^32).

   The typical kinds of sequence number comparisons that the TCP
   implementation must perform include:

   (a)  Determining that an acknowledgment refers to some sequence
        number sent but not yet acknowledged.

   (b)  Determining that all sequence numbers occupied by a segment have
        been acknowledged (e.g., to remove the segment from a
        retransmission queue).

   (c)  Determining that an incoming segment contains sequence numbers
        that are expected (i.e., that the segment "overlaps" the receive
        window).

   In response to sending data, the TCP endpoint will receive
   acknowledgments.  The following comparisons are needed to process the
   acknowledgments:

      SND.UNA = oldest unacknowledged sequence number

      SND.NXT = next sequence number to be sent

      SEG.ACK = acknowledgment from the receiving TCP peer (next
      sequence number expected by the receiving TCP peer)

      SEG.SEQ = first sequence number of a segment

      SEG.LEN = the number of octets occupied by the data in the segment
      (counting SYN and FIN)

      SEG.SEQ+SEG.LEN-1 = last sequence number of a segment

   A new acknowledgment (called an "acceptable ack") is one for which
   the inequality below holds:

      SND.UNA < SEG.ACK =< SND.NXT

   A segment on the retransmission queue is fully acknowledged if the
   sum of its sequence number and length is less than or equal to the
   acknowledgment value in the incoming segment.

   When data is received, the following comparisons are needed:

      RCV.NXT = next sequence number expected on an incoming segment,
      and is the left or lower edge of the receive window

      RCV.NXT+RCV.WND-1 = last sequence number expected on an incoming
      segment, and is the right or upper edge of the receive window

      SEG.SEQ = first sequence number occupied by the incoming segment

      SEG.SEQ+SEG.LEN-1 = last sequence number occupied by the incoming
      segment

   A segment is judged to occupy a portion of valid receive sequence
   space if

      RCV.NXT =< SEG.SEQ < RCV.NXT+RCV.WND

   or

      RCV.NXT =< SEG.SEQ+SEG.LEN-1 < RCV.NXT+RCV.WND

   The first part of this test checks to see if the beginning of the
   segment falls in the window, the second part of the test checks to
   see if the end of the segment falls in the window; if the segment
   passes either part of the test, it contains data in the window.

   Actually, it is a little more complicated than this.  Due to zero
   windows and zero-length segments, we have four cases for the
   acceptability of an incoming segment:

       +=========+=========+======================================+
       | Segment | Receive | Test                                 |
       | Length  | Window  |                                      |
       +=========+=========+======================================+
       | 0       | 0       | SEG.SEQ = RCV.NXT                    |
       +---------+---------+--------------------------------------+
       | 0       | >0      | RCV.NXT =< SEG.SEQ < RCV.NXT+RCV.WND |
       +---------+---------+--------------------------------------+
       | >0      | 0       | not acceptable                       |
       +---------+---------+--------------------------------------+
       | >0      | >0      | RCV.NXT =< SEG.SEQ < RCV.NXT+RCV.WND |
       |         |         |                                      |
       |         |         | or                                   |
       |         |         |                                      |
       |         |         | RCV.NXT =< SEG.SEQ+SEG.LEN-1 <       |
       |         |         | RCV.NXT+RCV.WND                      |
       +---------+---------+--------------------------------------+

                   Table 5: Segment Acceptability Tests

   Note that when the receive window is zero no segments should be
   acceptable except ACK segments.  Thus, it is possible for a TCP
   implementation to maintain a zero receive window while transmitting
   data and receiving ACKs.  A TCP receiver MUST process the RST and URG
   fields of all incoming segments, even when the receive window is zero
   (MUST-66).

   We have taken advantage of the numbering scheme to protect certain
   control information as well.  This is achieved by implicitly
   including some control flags in the sequence space so they can be
   retransmitted and acknowledged without confusion (i.e., one and only
   one copy of the control will be acted upon).  Control information is
   not physically carried in the segment data space.  Consequently, we
   must adopt rules for implicitly assigning sequence numbers to
   control.  The SYN and FIN are the only controls requiring this
   protection, and these controls are used only at connection opening
   and closing.  For sequence number purposes, the SYN is considered to
   occur before the first actual data octet of the segment in which it
   occurs, while the FIN is considered to occur after the last actual
   data octet in a segment in which it occurs.  The segment length
   (SEG.LEN) includes both data and sequence space-occupying controls.
   When a SYN is present, then SEG.SEQ is the sequence number of the
   SYN.

#### 3.4.1. Initial Sequence Number Selection

   A connection is defined by a pair of sockets.  Connections can be
   reused.  New instances of a connection will be referred to as
   incarnations of the connection.  The problem that arises from this is
   -- "how does the TCP implementation identify duplicate segments from
   previous incarnations of the connection?"  This problem becomes
   apparent if the connection is being opened and closed in quick
   succession, or if the connection breaks with loss of memory and is
   then reestablished.  To support this, the TIME-WAIT state limits the
   rate of connection reuse, while the initial sequence number selection
   described below further protects against ambiguity about which
   incarnation of a connection an incoming packet corresponds to.

   To avoid confusion, we must prevent segments from one incarnation of
   a connection from being used while the same sequence numbers may
   still be present in the network from an earlier incarnation.  We want
   to assure this even if a TCP endpoint loses all knowledge of the
   sequence numbers it has been using.  When new connections are
   created, an initial sequence number (ISN) generator is employed that
   selects a new 32-bit ISN.  There are security issues that result if
   an off-path attacker is able to predict or guess ISN values [42].

   TCP initial sequence numbers are generated from a number sequence
   that monotonically increases until it wraps, known loosely as a
   "clock".  This clock is a 32-bit counter that typically increments at
   least once every roughly 4 microseconds, although it is neither
   assumed to be realtime nor precise, and need not persist across
   reboots.  The clock component is intended to ensure that with a
   Maximum Segment Lifetime (MSL), generated ISNs will be unique since
   it cycles approximately every 4.55 hours, which is much longer than
   the MSL.  Please note that for modern networks that support high data
   rates where the connection might start and quickly advance sequence
   numbers to overlap within the MSL, it is recommended to implement the
   Timestamp Option as mentioned later in Section 3.4.3.

   A TCP implementation MUST use the above type of "clock" for clock-
   driven selection of initial sequence numbers (MUST-8), and SHOULD
   generate its initial sequence numbers with the expression:

   ISN = M + F(localip, localport, remoteip, remoteport, secretkey)

   where M is the 4 microsecond timer, and F() is a pseudorandom
   function (PRF) of the connection's identifying parameters ("localip,
   localport, remoteip, remoteport") and a secret key ("secretkey")
   (SHLD-1).  F() MUST NOT be computable from the outside (MUST-9), or
   an attacker could still guess at sequence numbers from the ISN used
   for some other connection.  The PRF could be implemented as a
   cryptographic hash of the concatenation of the TCP connection
   parameters and some secret data.  For discussion of the selection of
   a specific hash algorithm and management of the secret key data,
   please see Section 3 of [42].

   For each connection there is a send sequence number and a receive
   sequence number.  The initial send sequence number (ISS) is chosen by
   the data sending TCP peer, and the initial receive sequence number
   (IRS) is learned during the connection-establishing procedure.

   For a connection to be established or initialized, the two TCP peers
   must synchronize on each other's initial sequence numbers.  This is
   done in an exchange of connection-establishing segments carrying a
   control bit called "SYN" (for synchronize) and the initial sequence
   numbers.  As a shorthand, segments carrying the SYN bit are also
   called "SYNs".  Hence, the solution requires a suitable mechanism for
   picking an initial sequence number and a slightly involved handshake
   to exchange the ISNs.

   The synchronization requires each side to send its own initial
   sequence number and to receive a confirmation of it in acknowledgment
   from the remote TCP peer.  Each side must also receive the remote
   peer's initial sequence number and send a confirming acknowledgment.

       1) A --> B  SYN my sequence number is X
       2) A <-- B  ACK your sequence number is X
       3) A <-- B  SYN my sequence number is Y
       4) A --> B  ACK your sequence number is Y

   Because steps 2 and 3 can be combined in a single message this is
   called the three-way (or three message) handshake (3WHS).

   A 3WHS is necessary because sequence numbers are not tied to a global
   clock in the network, and TCP implementations may have different
   mechanisms for picking the ISNs.  The receiver of the first SYN has
   no way of knowing whether the segment was an old one or not, unless
   it remembers the last sequence number used on the connection (which
   is not always possible), and so it must ask the sender to verify this
   SYN.  The three-way handshake and the advantages of a clock-driven
   scheme for ISN selection are discussed in [69].

#### 3.4.2. Knowing When to Keep Quiet

   A theoretical problem exists where data could be corrupted due to
   confusion between old segments in the network and new ones after a
   host reboots if the same port numbers and sequence space are reused.
   The "quiet time" concept discussed below addresses this, and the
   discussion of it is included for situations where it might be
   relevant, although it is not felt to be necessary in most current
   implementations.  The problem was more relevant earlier in the
   history of TCP.  In practical use on the Internet today, the error-
   prone conditions are sufficiently unlikely that it is safe to ignore.
   Reasons why it is now negligible include: (a) ISS and ephemeral port
   randomization have reduced likelihood of reuse of port numbers and
   sequence numbers after reboots, (b) the effective MSL of the Internet
   has declined as links have become faster, and (c) reboots often
   taking longer than an MSL anyways.

   To be sure that a TCP implementation does not create a segment
   carrying a sequence number that may be duplicated by an old segment
   remaining in the network, the TCP endpoint must keep quiet for an MSL
   before assigning any sequence numbers upon starting up or recovering
   from a situation where memory of sequence numbers in use was lost.
   For this specification the MSL is taken to be 2 minutes.  This is an
   engineering choice, and may be changed if experience indicates it is
   desirable to do so.  Note that if a TCP endpoint is reinitialized in
   some sense, yet retains its memory of sequence numbers in use, then
   it need not wait at all; it must only be sure to use sequence numbers
   larger than those recently used.

#### 3.4.3. The TCP Quiet Time Concept

   Hosts that for any reason lose knowledge of the last sequence numbers
   transmitted on each active (i.e., not closed) connection shall delay
   emitting any TCP segments for at least the agreed MSL in the internet
   system that the host is a part of.  In the paragraphs below, an
   explanation for this specification is given.  TCP implementers may
   violate the "quiet time" restriction, but only at the risk of causing
   some old data to be accepted as new or new data rejected as old
   duplicated data by some receivers in the internet system.

   TCP endpoints consume sequence number space each time a segment is
   formed and entered into the network output queue at a source host.
   The duplicate detection and sequencing algorithm in TCP relies on the
   unique binding of segment data to sequence space to the extent that
   sequence numbers will not cycle through all 2^32 values before the
   segment data bound to those sequence numbers has been delivered and
   acknowledged by the receiver and all duplicate copies of the segments
   have "drained" from the internet.  Without such an assumption, two
   distinct TCP segments could conceivably be assigned the same or
   overlapping sequence numbers, causing confusion at the receiver as to
   which data is new and which is old.  Remember that each segment is
   bound to as many consecutive sequence numbers as there are octets of
   data and SYN or FIN flags in the segment.

   Under normal conditions, TCP implementations keep track of the next
   sequence number to emit and the oldest awaiting acknowledgment so as
   to avoid mistakenly reusing a sequence number before its first use
   has been acknowledged.  This alone does not guarantee that old
   duplicate data is drained from the net, so the sequence space has
   been made large to reduce the probability that a wandering duplicate
   will cause trouble upon arrival.  At 2 megabits/sec., it takes 4.5
   hours to use up 2^32 octets of sequence space.  Since the maximum
   segment lifetime in the net is not likely to exceed a few tens of
   seconds, this is deemed ample protection for foreseeable nets, even
   if data rates escalate to 10s of megabits/sec.  At 100 megabits/sec.,
   the cycle time is 5.4 minutes, which may be a little short but still
   within reason.  Much higher data rates are possible today, with
   implications described in the final paragraph of this subsection.

   The basic duplicate detection and sequencing algorithm in TCP can be
   defeated, however, if a source TCP endpoint does not have any memory
   of the sequence numbers it last used on a given connection.  For
   example, if the TCP implementation were to start all connections with
   sequence number 0, then upon the host rebooting, a TCP peer might re-
   form an earlier connection (possibly after half-open connection
   resolution) and emit packets with sequence numbers identical to or
   overlapping with packets still in the network, which were emitted on
   an earlier incarnation of the same connection.  In the absence of
   knowledge about the sequence numbers used on a particular connection,
   the TCP specification recommends that the source delay for MSL
   seconds before emitting segments on the connection, to allow time for
   segments from the earlier connection incarnation to drain from the
   system.

   Even hosts that can remember the time of day and use it to select
   initial sequence number values are not immune from this problem
   (i.e., even if time of day is used to select an initial sequence
   number for each new connection incarnation).

   Suppose, for example, that a connection is opened starting with
   sequence number S.  Suppose that this connection is not used much and
   that eventually the initial sequence number function (ISN(t)) takes
   on a value equal to the sequence number, say S1, of the last segment
   sent by this TCP endpoint on a particular connection.  Now suppose,
   at this instant, the host reboots and establishes a new incarnation
   of the connection.  The initial sequence number chosen is S1 = ISN(t)
   -- last used sequence number on old incarnation of connection!  If
   the recovery occurs quickly enough, any old duplicates in the net
   bearing sequence numbers in the neighborhood of S1 may arrive and be
   treated as new packets by the receiver of the new incarnation of the
   connection.

   The problem is that the recovering host may not know for how long it
   was down between rebooting nor does it know whether there are still
   old duplicates in the system from earlier connection incarnations.

   One way to deal with this problem is to deliberately delay emitting
   segments for one MSL after recovery from a reboot -- this is the
   "quiet time" specification.  Hosts that prefer to avoid waiting and
   are willing to risk possible confusion of old and new packets at a
   given destination may choose not to wait for the "quiet time".
   Implementers may provide TCP users with the ability to select on a
   connection-by-connection basis whether to wait after a reboot, or may
   informally implement the "quiet time" for all connections.
   Obviously, even where a user selects to "wait", this is not necessary
   after the host has been "up" for at least MSL seconds.

   To summarize: every segment emitted occupies one or more sequence
   numbers in the sequence space, and the numbers occupied by a segment
   are "busy" or "in use" until MSL seconds have passed.  Upon
   rebooting, a block of space-time is occupied by the octets and SYN or
   FIN flags of any potentially still in-flight segments.  If a new
   connection is started too soon and uses any of the sequence numbers
   in the space-time footprint of those potentially still in-flight
   segments of the previous connection incarnation, there is a potential
   sequence number overlap area that could cause confusion at the
   receiver.

   High-performance cases will have shorter cycle times than those in
   the megabits per second that the base TCP design described above
   considers.  At 1 Gbps, the cycle time is 34 seconds, only 3 seconds
   at 10 Gbps, and around a third of a second at 100 Gbps.  In these
   higher-performance cases, TCP Timestamp Options and Protection
   Against Wrapped Sequences (PAWS) [47] provide the needed capability
   to detect and discard old duplicates.

### 3.5. Establishing a Connection

   The "three-way handshake" is the procedure used to establish a
   connection.  This procedure normally is initiated by one TCP peer and
   responded to by another TCP peer.  The procedure also works if two
   TCP peers simultaneously initiate the procedure.  When simultaneous
   open occurs, each TCP peer receives a SYN segment that carries no
   acknowledgment after it has sent a SYN.  Of course, the arrival of an
   old duplicate SYN segment can potentially make it appear, to the
   recipient, that a simultaneous connection initiation is in progress.
   Proper use of "reset" segments can disambiguate these cases.

   Several examples of connection initiation follow.  Although these
   examples do not show connection synchronization using data-carrying
   segments, this is perfectly legitimate, so long as the receiving TCP
   endpoint doesn't deliver the data to the user until it is clear the
   data is valid (e.g., the data is buffered at the receiver until the
   connection reaches the ESTABLISHED state, given that the three-way
   handshake reduces the possibility of false connections).  It is a
   trade-off between memory and messages to provide information for this
   checking.

   The simplest 3WHS is shown in Figure 6.  The figures should be
   interpreted in the following way.  Each line is numbered for
   reference purposes.  Right arrows (-->) indicate departure of a TCP
   segment from TCP Peer A to TCP Peer B or arrival of a segment at B
   from A.  Left arrows (<--) indicate the reverse.  Ellipses (...)
   indicate a segment that is still in the network (delayed).  Comments
   appear in parentheses.  TCP connection states represent the state
   AFTER the departure or arrival of the segment (whose contents are
   shown in the center of each line).  Segment contents are shown in
   abbreviated form, with sequence number, control flags, and ACK field.
   Other fields such as window, addresses, lengths, and text have been
   left out in the interest of clarity.

       TCP Peer A                                           TCP Peer B

   1.  CLOSED                                               LISTEN

   2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

   3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

   4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED

   5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED

     Figure 6: Basic Three-Way Handshake for Connection Synchronization

   In line 2 of Figure 6, TCP Peer A begins by sending a SYN segment
   indicating that it will use sequence numbers starting with sequence
   number 100.  In line 3, TCP Peer B sends a SYN and acknowledges the
   SYN it received from TCP Peer A.  Note that the acknowledgment field
   indicates TCP Peer B is now expecting to hear sequence 101,
   acknowledging the SYN that occupied sequence 100.

   At line 4, TCP Peer A responds with an empty segment containing an
   ACK for TCP Peer B's SYN; and in line 5, TCP Peer A sends some data.
   Note that the sequence number of the segment in line 5 is the same as
   in line 4 because the ACK does not occupy sequence number space (if
   it did, we would wind up ACKing ACKs!).

   Simultaneous initiation is only slightly more complex, as is shown in
   Figure 7.  Each TCP peer's connection state cycles from CLOSED to
   SYN-SENT to SYN-RECEIVED to ESTABLISHED.

       TCP Peer A                                       TCP Peer B

   1.  CLOSED                                           CLOSED

   2.  SYN-SENT     --> <SEQ=100><CTL=SYN>              ...

   3.  SYN-RECEIVED <-- <SEQ=300><CTL=SYN>              <-- SYN-SENT

   4.               ... <SEQ=100><CTL=SYN>              --> SYN-RECEIVED

   5.  SYN-RECEIVED --> <SEQ=100><ACK=301><CTL=SYN,ACK> ...

   6.  ESTABLISHED  <-- <SEQ=300><ACK=101><CTL=SYN,ACK> <-- SYN-RECEIVED

   7.               ... <SEQ=100><ACK=301><CTL=SYN,ACK> --> ESTABLISHED

             Figure 7: Simultaneous Connection Synchronization

   A TCP implementation MUST support simultaneous open attempts (MUST-
   10).

   Note that a TCP implementation MUST keep track of whether a
   connection has reached SYN-RECEIVED state as the result of a passive
   OPEN or an active OPEN (MUST-11).

   The principal reason for the three-way handshake is to prevent old
   duplicate connection initiations from causing confusion.  To deal
   with this, a special control message, reset, is specified.  If the
   receiving TCP peer is in a non-synchronized state (i.e., SYN-SENT,
   SYN-RECEIVED), it returns to LISTEN on receiving an acceptable reset.
   If the TCP peer is in one of the synchronized states (ESTABLISHED,
   FIN-WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT), it
   aborts the connection and informs its user.  We discuss this latter
   case under "half-open" connections below.

       TCP Peer A                                           TCP Peer B

   1.  CLOSED                                               LISTEN

   2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               ...

   3.  (duplicate) ... <SEQ=90><CTL=SYN>               --> SYN-RECEIVED

   4.  SYN-SENT    <-- <SEQ=300><ACK=91><CTL=SYN,ACK>  <-- SYN-RECEIVED

   5.  SYN-SENT    --> <SEQ=91><CTL=RST>               --> LISTEN

   6.              ... <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

   7.  ESTABLISHED <-- <SEQ=400><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

   8.  ESTABLISHED --> <SEQ=101><ACK=401><CTL=ACK>      --> ESTABLISHED

                 Figure 8: Recovery from Old Duplicate SYN

   As a simple example of recovery from old duplicates, consider
   Figure 8.  At line 3, an old duplicate SYN arrives at TCP Peer B.
   TCP Peer B cannot tell that this is an old duplicate, so it responds
   normally (line 4).  TCP Peer A detects that the ACK field is
   incorrect and returns a RST (reset) with its SEQ field selected to
   make the segment believable.  TCP Peer B, on receiving the RST,
   returns to the LISTEN state.  When the original SYN finally arrives
   at line 6, the synchronization proceeds normally.  If the SYN at line
   6 had arrived before the RST, a more complex exchange might have
   occurred with RSTs sent in both directions.

#### 3.5.1. Half-Open Connections and Other Anomalies

   An established connection is said to be "half-open" if one of the TCP
   peers has closed or aborted the connection at its end without the
   knowledge of the other, or if the two ends of the connection have
   become desynchronized owing to a failure or reboot that resulted in
   loss of memory.  Such connections will automatically become reset if
   an attempt is made to send data in either direction.  However, half-
   open connections are expected to be unusual.

   If at site A the connection no longer exists, then an attempt by the
   user at site B to send any data on it will result in the site B TCP
   endpoint receiving a reset control message.  Such a message indicates
   to the site B TCP endpoint that something is wrong, and it is
   expected to abort the connection.

   Assume that two user processes A and B are communicating with one
   another when a failure or reboot occurs causing loss of memory to A's
   TCP implementation.  Depending on the operating system supporting A's
   TCP implementation, it is likely that some error recovery mechanism
   exists.  When the TCP endpoint is up again, A is likely to start
   again from the beginning or from a recovery point.  As a result, A
   will probably try to OPEN the connection again or try to SEND on the
   connection it believes open.  In the latter case, it receives the
   error message "connection not open" from the local (A's) TCP
   implementation.  In an attempt to establish the connection, A's TCP
   implementation will send a segment containing SYN.  This scenario
   leads to the example shown in Figure 9.  After TCP Peer A reboots,
   the user attempts to reopen the connection.  TCP Peer B, in the
   meantime, thinks the connection is open.

         TCP Peer A                                      TCP Peer B

     1.  (REBOOT)                              (send 300,receive 100)

     2.  CLOSED                                           ESTABLISHED

     3.  SYN-SENT --> <SEQ=400><CTL=SYN>              --> (??)

     4.  (!!)     <-- <SEQ=300><ACK=100><CTL=ACK>     <-- ESTABLISHED

     5.  SYN-SENT --> <SEQ=100><CTL=RST>              --> (Abort!!)

     6.  SYN-SENT                                         CLOSED

     7.  SYN-SENT --> <SEQ=400><CTL=SYN>              -->

                  Figure 9: Half-Open Connection Discovery

   When the SYN arrives at line 3, TCP Peer B, being in a synchronized
   state, and the incoming segment outside the window, responds with an
   acknowledgment indicating what sequence it next expects to hear (ACK
   100).  TCP Peer A sees that this segment does not acknowledge
   anything it sent and, being unsynchronized, sends a reset (RST)
   because it has detected a half-open connection.  TCP Peer B aborts at
   line 5.  TCP Peer A will continue to try to establish the connection;
   the problem is now reduced to the basic three-way handshake of
   Figure 6.

   An interesting alternative case occurs when TCP Peer A reboots and
   TCP Peer B tries to send data on what it thinks is a synchronized
   connection.  This is illustrated in Figure 10.  In this case, the
   data arriving at TCP Peer A from TCP Peer B (line 2) is unacceptable
   because no such connection exists, so TCP Peer A sends a RST.  The
   RST is acceptable so TCP Peer B processes it and aborts the
   connection.

         TCP Peer A                                         TCP Peer B

   1.  (REBOOT)                                  (send 300,receive 100)

   2.  (??)    <-- <SEQ=300><ACK=100><DATA=10><CTL=ACK> <-- ESTABLISHED

   3.          --> <SEQ=100><CTL=RST>                   --> (ABORT!!)

        Figure 10: Active Side Causes Half-Open Connection Discovery

   In Figure 11, two TCP Peers A and B with passive connections waiting
   for SYN are depicted.  An old duplicate arriving at TCP Peer B (line
   2) stirs B into action.  A SYN-ACK is returned (line 3) and causes
   TCP A to generate a RST (the ACK in line 3 is not acceptable).  TCP
   Peer B accepts the reset and returns to its passive LISTEN state.

       TCP Peer A                                    TCP Peer B

   1.  LISTEN                                        LISTEN

   2.       ... <SEQ=Z><CTL=SYN>                -->  SYN-RECEIVED

   3.  (??) <-- <SEQ=X><ACK=Z+1><CTL=SYN,ACK>   <--  SYN-RECEIVED

   4.       --> <SEQ=Z+1><CTL=RST>              -->  (return to LISTEN!)

   5.  LISTEN                                        LISTEN

   Figure 11: Old Duplicate SYN Initiates a Reset on Two Passive Sockets

   A variety of other cases are possible, all of which are accounted for
   by the following rules for RST generation and processing.

#### 3.5.2. Reset Generation

   A TCP user or application can issue a reset on a connection at any
   time, though reset events are also generated by the protocol itself
   when various error conditions occur, as described below.  The side of
   a connection issuing a reset should enter the TIME-WAIT state, as
   this generally helps to reduce the load on busy servers for reasons
   described in [70].

   As a general rule, reset (RST) is sent whenever a segment arrives
   that apparently is not intended for the current connection.  A reset
   must not be sent if it is not clear that this is the case.

   There are three groups of states:

   1.  If the connection does not exist (CLOSED), then a reset is sent
       in response to any incoming segment except another reset.  A SYN
       segment that does not match an existing connection is rejected by
       this means.

       If the incoming segment has the ACK bit set, the reset takes its
       sequence number from the ACK field of the segment; otherwise, the
       reset has sequence number zero and the ACK field is set to the
       sum of the sequence number and segment length of the incoming
       segment.  The connection remains in the CLOSED state.

   2.  If the connection is in any non-synchronized state (LISTEN, SYN-
       SENT, SYN-RECEIVED), and the incoming segment acknowledges
       something not yet sent (the segment carries an unacceptable ACK),
       or if an incoming segment has a security level or compartment
       (Appendix A.1) that does not exactly match the level and
       compartment requested for the connection, a reset is sent.

       If the incoming segment has an ACK field, the reset takes its
       sequence number from the ACK field of the segment; otherwise, the
       reset has sequence number zero and the ACK field is set to the
       sum of the sequence number and segment length of the incoming
       segment.  The connection remains in the same state.

   3.  If the connection is in a synchronized state (ESTABLISHED, FIN-
       WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT),
       any unacceptable segment (out-of-window sequence number or
       unacceptable acknowledgment number) must be responded to with an
       empty acknowledgment segment (without any user data) containing
       the current send sequence number and an acknowledgment indicating
       the next sequence number expected to be received, and the
       connection remains in the same state.

       If an incoming segment has a security level or compartment that
       does not exactly match the level and compartment requested for
       the connection, a reset is sent and the connection goes to the
       CLOSED state.  The reset takes its sequence number from the ACK
       field of the incoming segment.

#### 3.5.3. Reset Processing

   In all states except SYN-SENT, all reset (RST) segments are validated
   by checking their SEQ fields.  A reset is valid if its sequence
   number is in the window.  In the SYN-SENT state (a RST received in
   response to an initial SYN), the RST is acceptable if the ACK field
   acknowledges the SYN.

   The receiver of a RST first validates it, then changes state.  If the
   receiver was in the LISTEN state, it ignores it.  If the receiver was
   in SYN-RECEIVED state and had previously been in the LISTEN state,
   then the receiver returns to the LISTEN state; otherwise, the
   receiver aborts the connection and goes to the CLOSED state.  If the
   receiver was in any other state, it aborts the connection and advises
   the user and goes to the CLOSED state.

   TCP implementations SHOULD allow a received RST segment to include
   data (SHLD-2).  It has been suggested that a RST segment could
   contain diagnostic data that explains the cause of the RST.  No
   standard has yet been established for such data.

### 3.6. Closing a Connection

   CLOSE is an operation meaning "I have no more data to send."  The
   notion of closing a full-duplex connection is subject to ambiguous
   interpretation, of course, since it may not be obvious how to treat
   the receiving side of the connection.  We have chosen to treat CLOSE
   in a simplex fashion.  The user who CLOSEs may continue to RECEIVE
   until the TCP receiver is told that the remote peer has CLOSED also.
   Thus, a program could initiate several SENDs followed by a CLOSE, and
   then continue to RECEIVE until signaled that a RECEIVE failed because
   the remote peer has CLOSED.  The TCP implementation will signal a
   user, even if no RECEIVEs are outstanding, that the remote peer has
   closed, so the user can terminate their side gracefully.  A TCP
   implementation will reliably deliver all buffers SENT before the
   connection was CLOSED so a user who expects no data in return need
   only wait to hear the connection was CLOSED successfully to know that
   all their data was received at the destination TCP endpoint.  Users
   must keep reading connections they close for sending until the TCP
   implementation indicates there is no more data.

   There are essentially three cases:

   1)  The user initiates by telling the TCP implementation to CLOSE the
       connection (TCP Peer A in Figure 12).

   2)  The remote TCP endpoint initiates by sending a FIN control signal
       (TCP Peer B in Figure 12).

   3)  Both users CLOSE simultaneously (Figure 13).

   Case 1:  Local user initiates the close

      In this case, a FIN segment can be constructed and placed on the
      outgoing segment queue.  No further SENDs from the user will be
      accepted by the TCP implementation, and it enters the FIN-WAIT-1
      state.  RECEIVEs are allowed in this state.  All segments
      preceding and including FIN will be retransmitted until
      acknowledged.  When the other TCP peer has both acknowledged the
      FIN and sent a FIN of its own, the first TCP peer can ACK this
      FIN.  Note that a TCP endpoint receiving a FIN will ACK but not
      send its own FIN until its user has CLOSED the connection also.

   Case 2:  TCP endpoint receives a FIN from the network

      If an unsolicited FIN arrives from the network, the receiving TCP
      endpoint can ACK it and tell the user that the connection is
      closing.  The user will respond with a CLOSE, upon which the TCP
      endpoint can send a FIN to the other TCP peer after sending any
      remaining data.  The TCP endpoint then waits until its own FIN is
      acknowledged whereupon it deletes the connection.  If an ACK is
      not forthcoming, after the user timeout the connection is aborted
      and the user is told.

   Case 3:  Both users close simultaneously

      A simultaneous CLOSE by users at both ends of a connection causes
      FIN segments to be exchanged (Figure 13).  When all segments
      preceding the FINs have been processed and acknowledged, each TCP
      peer can ACK the FIN it has received.  Both will, upon receiving
      these ACKs, delete the connection.

       TCP Peer A                                           TCP Peer B

   1.  ESTABLISHED                                          ESTABLISHED

   2.  (Close)
       FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  --> CLOSE-WAIT

   3.  FIN-WAIT-2  <-- <SEQ=300><ACK=101><CTL=ACK>      <-- CLOSE-WAIT

   4.                                                       (Close)
       TIME-WAIT   <-- <SEQ=300><ACK=101><CTL=FIN,ACK>  <-- LAST-ACK

   5.  TIME-WAIT   --> <SEQ=101><ACK=301><CTL=ACK>      --> CLOSED

   6.  (2 MSL)
       CLOSED

                      Figure 12: Normal Close Sequence

       TCP Peer A                                           TCP Peer B

   1.  ESTABLISHED                                          ESTABLISHED

   2.  (Close)                                              (Close)
       FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  ... FIN-WAIT-1
                   <-- <SEQ=300><ACK=100><CTL=FIN,ACK>  <--
                   ... <SEQ=100><ACK=300><CTL=FIN,ACK>  -->

   3.  CLOSING     --> <SEQ=101><ACK=301><CTL=ACK>      ... CLOSING
                   <-- <SEQ=301><ACK=101><CTL=ACK>      <--
                   ... <SEQ=101><ACK=301><CTL=ACK>      -->

   4.  TIME-WAIT                                            TIME-WAIT
       (2 MSL)                                              (2 MSL)
       CLOSED                                               CLOSED

                   Figure 13: Simultaneous Close Sequence

   A TCP connection may terminate in two ways: (1) the normal TCP close
   sequence using a FIN handshake (Figure 12), and (2) an "abort" in
   which one or more RST segments are sent and the connection state is
   immediately discarded.  If the local TCP connection is closed by the
   remote side due to a FIN or RST received from the remote side, then
   the local application MUST be informed whether it closed normally or
   was aborted (MUST-12).

#### 3.6.1. Half-Closed Connections

   The normal TCP close sequence delivers buffered data reliably in both
   directions.  Since the two directions of a TCP connection are closed
   independently, it is possible for a connection to be "half closed",
   i.e., closed in only one direction, and a host is permitted to
   continue sending data in the open direction on a half-closed
   connection.

   A host MAY implement a "half-duplex" TCP close sequence, so that an
   application that has called CLOSE cannot continue to read data from
   the connection (MAY-1).  If such a host issues a CLOSE call while
   received data is still pending in the TCP connection, or if new data
   is received after CLOSE is called, its TCP implementation SHOULD send
   a RST to show that data was lost (SHLD-3).  See [23], Section 2.17
   for discussion.

   When a connection is closed actively, it MUST linger in the TIME-WAIT
   state for a time 2xMSL (Maximum Segment Lifetime) (MUST-13).
   However, it MAY accept a new SYN from the remote TCP endpoint to
   reopen the connection directly from TIME-WAIT state (MAY-2), if it:

   (1)  assigns its initial sequence number for the new connection to be
        larger than the largest sequence number it used on the previous
        connection incarnation, and

   (2)  returns to TIME-WAIT state if the SYN turns out to be an old
        duplicate.

   When the TCP Timestamp Options are available, an improved algorithm
   is described in [40] in order to support higher connection
   establishment rates.  This algorithm for reducing TIME-WAIT is a Best
   Current Practice that SHOULD be implemented since Timestamp Options
   are commonly used, and using them to reduce TIME-WAIT provides
   benefits for busy Internet servers (SHLD-4).

### 3.7. Segmentation

   The term "segmentation" refers to the activity TCP performs when
   ingesting a stream of bytes from a sending application and
   packetizing that stream of bytes into TCP segments.  Individual TCP
   segments often do not correspond one-for-one to individual send (or
   socket write) calls from the application.  Applications may perform
   writes at the granularity of messages in the upper-layer protocol,
   but TCP guarantees no correlation between the boundaries of TCP
   segments sent and received and the boundaries of the read or write
   buffers of user application data.  In some specific protocols, such
   as Remote Direct Memory Access (RDMA) using Direct Data Placement
   (DDP) and Marker PDU Aligned Framing (MPA) [34], there are
   performance optimizations possible when the relation between TCP
   segments and application data units can be controlled, and MPA
   includes a specific mechanism for detecting and verifying this
   relationship between TCP segments and application message data
   structures, but this is specific to applications like RDMA.  In
   general, multiple goals influence the sizing of TCP segments created
   by a TCP implementation.

   Goals driving the sending of larger segments include:

   *  Reducing the number of packets in flight within the network.

   *  Increasing processing efficiency and potential performance by
      enabling a smaller number of interrupts and inter-layer
      interactions.

   *  Limiting the overhead of TCP headers.

   Note that the performance benefits of sending larger segments may
   decrease as the size increases, and there may be boundaries where
   advantages are reversed.  For instance, on some implementation
   architectures, 1025 bytes within a segment could lead to worse
   performance than 1024 bytes, due purely to data alignment on copy
   operations.

   Goals driving the sending of smaller segments include:

   *  Avoiding sending a TCP segment that would result in an IP datagram
      larger than the smallest MTU along an IP network path because this
      results in either packet loss or packet fragmentation.  Making
      matters worse, some firewalls or middleboxes may drop fragmented
      packets or ICMP messages related to fragmentation.

   *  Preventing delays to the application data stream, especially when
      TCP is waiting on the application to generate more data, or when
      the application is waiting on an event or input from its peer in
      order to generate more data.

   *  Enabling "fate sharing" between TCP segments and lower-layer data
      units (e.g., below IP, for links with cell or frame sizes smaller
      than the IP MTU).

   Towards meeting these competing sets of goals, TCP includes several
   mechanisms, including the Maximum Segment Size Option, Path MTU
   Discovery, the Nagle algorithm, and support for IPv6 Jumbograms, as
   discussed in the following subsections.

#### 3.7.1. Maximum Segment Size Option

   TCP endpoints MUST implement both sending and receiving the MSS
   Option (MUST-14).

   TCP implementations SHOULD send an MSS Option in every SYN segment
   when its receive MSS differs from the default 536 for IPv4 or 1220
   for IPv6 (SHLD-5), and MAY send it always (MAY-3).

   If an MSS Option is not received at connection setup, TCP
   implementations MUST assume a default send MSS of 536 (576 - 40) for
   IPv4 or 1220 (1280 - 60) for IPv6 (MUST-15).

   The maximum size of a segment that a TCP endpoint really sends, the
   "effective send MSS", MUST be the smaller (MUST-16) of the send MSS
   (that reflects the available reassembly buffer size at the remote
   host, the EMTU_R [19]) and the largest transmission size permitted by
   the IP layer (EMTU_S [19]):

   Eff.snd.MSS = min(SendMSS+20, MMS_S) - TCPhdrsize - IPoptionsize

   where:

   *  SendMSS is the MSS value received from the remote host, or the
      default 536 for IPv4 or 1220 for IPv6, if no MSS Option is
      received.

   *  MMS_S is the maximum size for a transport-layer message that TCP
      may send.

   *  TCPhdrsize is the size of the fixed TCP header and any options.
      This is 20 in the (rare) case that no options are present but may
      be larger if TCP Options are to be sent.  Note that some options
      might not be included on all segments, but that for each segment
      sent, the sender should adjust the data length accordingly, within
      the Eff.snd.MSS.

   *  IPoptionsize is the size of any IPv4 options or IPv6 extension
      headers associated with a TCP connection.  Note that some options
      or extension headers might not be included on all packets, but
      that for each segment sent, the sender should adjust the data
      length accordingly, within the Eff.snd.MSS.

   The MSS value to be sent in an MSS Option should be equal to the
   effective MTU minus the fixed IP and TCP headers.  By ignoring both
   IP and TCP Options when calculating the value for the MSS Option, if
   there are any IP or TCP Options to be sent in a packet, then the
   sender must decrease the size of the TCP data accordingly.  RFC 6691   [43] discusses this in greater detail.

   The MSS value to be sent in an MSS Option must be less than or equal
   to:

      MMS_R - 20

   where MMS_R is the maximum size for a transport-layer message that
   can be received (and reassembled at the IP layer) (MUST-67).  TCP
   obtains MMS_R and MMS_S from the IP layer; see the generic call
   GET_MAXSIZES in Section 3.4 of RFC 1122.  These are defined in terms
   of their IP MTU equivalents, EMTU_R and EMTU_S [19].

   When TCP is used in a situation where either the IP or TCP headers
   are not fixed, the sender must reduce the amount of TCP data in any
   given packet by the number of octets used by the IP and TCP options.
   This has been a point of confusion historically, as explained in RFC 
   6691, Section 3.1.

#### 3.7.2. Path MTU Discovery

   A TCP implementation may be aware of the MTU on directly connected
   links, but will rarely have insight about MTUs across an entire
   network path.  For IPv4, RFC 1122 recommends an IP-layer default
   effective MTU of less than or equal to 576 for destinations not
   directly connected, and for IPv6 this would be 1280.  Using these
   fixed values limits TCP connection performance and efficiency.
   Instead, implementation of Path MTU Discovery (PMTUD) and
   Packetization Layer Path MTU Discovery (PLPMTUD) is strongly
   recommended in order for TCP to improve segmentation decisions.  Both
   PMTUD and PLPMTUD help TCP choose segment sizes that avoid both on-
   path (for IPv4) and source fragmentation (IPv4 and IPv6).

   PMTUD for IPv4 [2] or IPv6 [14] is implemented in conjunction between
   TCP, IP, and ICMP.  It relies both on avoiding source fragmentation
   and setting the IPv4 DF (don't fragment) flag, the latter to inhibit
   on-path fragmentation.  It relies on ICMP errors from routers along
   the path whenever a segment is too large to traverse a link.  Several
   adjustments to a TCP implementation with PMTUD are described in RFC 
   2923 in order to deal with problems experienced in practice [27].
   PLPMTUD [31] is a Standards Track improvement to PMTUD that relaxes
   the requirement for ICMP support across a path, and improves
   performance in cases where ICMP is not consistently conveyed, but
   still tries to avoid source fragmentation.  The mechanisms in all
   four of these RFCs are recommended to be included in TCP
   implementations.

   The TCP MSS Option specifies an upper bound for the size of packets
   that can be received (see [43]).  Hence, setting the value in the MSS
   Option too small can impact the ability for PMTUD or PLPMTUD to find
   a larger path MTU.  RFC 1191 discusses this implication of many older
   TCP implementations setting the TCP MSS to 536 (corresponding to the
   IPv4 576 byte default MTU) for non-local destinations, rather than
   deriving it from the MTUs of connected interfaces as recommended.

#### 3.7.3. Interfaces with Variable MTU Values

   The effective MTU can sometimes vary, as when used with variable
   compression, e.g., RObust Header Compression (ROHC) [37].  It is
   tempting for a TCP implementation to advertise the largest possible
   MSS, to support the most efficient use of compressed payloads.
   Unfortunately, some compression schemes occasionally need to transmit
   full headers (and thus smaller payloads) to resynchronize state at
   their endpoint compressors/decompressors.  If the largest MTU is used
   to calculate the value to advertise in the MSS Option, TCP
   retransmission may interfere with compressor resynchronization.

   As a result, when the effective MTU of an interface varies packet-to-
   packet, TCP implementations SHOULD use the smallest effective MTU of
   the interface to calculate the value to advertise in the MSS Option
   (SHLD-6).

#### 3.7.4. Nagle Algorithm

   The "Nagle algorithm" was described in RFC 896 [17] and was
   recommended in RFC 1122 [19] for mitigation of an early problem of
   too many small packets being generated.  It has been implemented in
   most current TCP code bases, sometimes with minor variations (see Appendix A.3).

   If there is unacknowledged data (i.e., SND.NXT > SND.UNA), then the
   sending TCP endpoint buffers all user data (regardless of the PSH
   bit) until the outstanding data has been acknowledged or until the
   TCP endpoint can send a full-sized segment (Eff.snd.MSS bytes).

   A TCP implementation SHOULD implement the Nagle algorithm to coalesce
   short segments (SHLD-7).  However, there MUST be a way for an
   application to disable the Nagle algorithm on an individual
   connection (MUST-17).  In all cases, sending data is also subject to
   the limitation imposed by the slow start algorithm [8].

   Since there can be problematic interactions between the Nagle
   algorithm and delayed acknowledgments, some implementations use minor
   variations of the Nagle algorithm, such as the one described in Appendix A.3.

#### 3.7.5. IPv6 Jumbograms

   In order to support TCP over IPv6 Jumbograms, implementations need to
   be able to send TCP segments larger than the 64-KB limit that the MSS
   Option can convey.  RFC 2675 [24] defines that an MSS value of 65,535
   bytes is to be treated as infinity, and Path MTU Discovery [14] is
   used to determine the actual MSS.

   The Jumbo Payload Option need not be implemented or understood by
   IPv6 nodes that do not support attachment to links with an MTU
   greater than 65,575 [24], and the present IPv6 Node Requirements does
   not include support for Jumbograms [55].

### 3.8. Data Communication

   Once the connection is established, data is communicated by the
   exchange of segments.  Because segments may be lost due to errors
   (checksum test failure) or network congestion, TCP uses
   retransmission to ensure delivery of every segment.  Duplicate
   segments may arrive due to network or TCP retransmission.  As
   discussed in the section on sequence numbers (Section 3.4), the TCP
   implementation performs certain tests on the sequence and
   acknowledgment numbers in the segments to verify their acceptability.

   The sender of data keeps track of the next sequence number to use in
   the variable SND.NXT.  The receiver of data keeps track of the next
   sequence number to expect in the variable RCV.NXT.  The sender of
   data keeps track of the oldest unacknowledged sequence number in the
   variable SND.UNA.  If the data flow is momentarily idle and all data
   sent has been acknowledged, then the three variables will be equal.

   When the sender creates a segment and transmits it, the sender
   advances SND.NXT.  When the receiver accepts a segment, it advances
   RCV.NXT and sends an acknowledgment.  When the data sender receives
   an acknowledgment, it advances SND.UNA.  The extent to which the
   values of these variables differ is a measure of the delay in the
   communication.  The amount by which the variables are advanced is the
   length of the data and SYN or FIN flags in the segment.  Note that,
   once in the ESTABLISHED state, all segments must carry current
   acknowledgment information.

   The CLOSE user call implies a push function (see Section 3.9.1), as
   does the FIN control flag in an incoming segment.

#### 3.8.1. Retransmission Timeout

   Because of the variability of the networks that compose an
   internetwork system and the wide range of uses of TCP connections,
   the retransmission timeout (RTO) must be dynamically determined.

   The RTO MUST be computed according to the algorithm in [10],
   including Karn's algorithm for taking RTT samples (MUST-18). RFC 793 contains an early example procedure for computing the RTO,
   based on work mentioned in IEN 177 [71].  This was then replaced by
   the algorithm described in RFC 1122, which was subsequently updated
   in RFC 2988 and then again in RFC 6298. RFC 1122 allows that if a retransmitted packet is identical to the
   original packet (which implies not only that the data boundaries have
   not changed, but also that none of the headers have changed), then
   the same IPv4 Identification field MAY be used (see Section 3.2.1.5
   of RFC 1122) (MAY-4).  The same IP Identification field may be reused
   anyways since it is only meaningful when a datagram is fragmented
   [44].  TCP implementations should not rely on or typically interact
   with this IPv4 header field in any way.  It is not a reasonable way
   to indicate duplicate sent segments nor to identify duplicate
   received segments.

#### 3.8.2. TCP Congestion Control



   RFC 2914 [5] explains the importance of congestion control for the
   Internet. RFC 1122 required implementation of Van Jacobson's congestion control
   algorithms slow start and congestion avoidance together with
   exponential backoff for successive RTO values for the same segment. RFC 2581 provided IETF Standards Track description of slow start and
   congestion avoidance, along with fast retransmit and fast recovery. RFC 5681 is the current description of these algorithms and is the
   current Standards Track specification providing guidelines for TCP
   congestion control.  RFC 6298 describes exponential backoff of RTO
   values, including keeping the backed-off value until a subsequent
   segment with new data has been sent and acknowledged without
   retransmission.

   A TCP endpoint MUST implement the basic congestion control algorithms
   slow start, congestion avoidance, and exponential backoff of RTO to
   avoid creating congestion collapse conditions (MUST-19).  RFC 5681   and RFC 6298 describe the basic algorithms on the IETF Standards
   Track that are broadly applicable.  Multiple other suitable
   algorithms exist and have been widely used.  Many TCP implementations
   support a set of alternative algorithms that can be configured for
   use on the endpoint.  An endpoint MAY implement such alternative
   algorithms provided that the algorithms are conformant with the TCP
   specifications from the IETF Standards Track as described in RFC 
   2914, RFC 5033 [7], and RFC 8961 [15] (MAY-18).

   Explicit Congestion Notification (ECN) was defined in RFC 3168 and is
   an IETF Standards Track enhancement that has many benefits [51].

   A TCP endpoint SHOULD implement ECN as described in RFC 3168 (SHLD-
   8).

#### 3.8.3. TCP Connection Failures

   Excessive retransmission of the same segment by a TCP endpoint
   indicates some failure of the remote host or the internetwork path.
   This failure may be of short or long duration.  The following
   procedure MUST be used to handle excessive retransmissions of data
   segments (MUST-20):

   (a)  There are two thresholds R1 and R2 measuring the amount of
        retransmission that has occurred for the same segment.  R1 and
        R2 might be measured in time units or as a count of
        retransmissions (with the current RTO and corresponding backoffs
        as a conversion factor, if needed).

   (b)  When the number of transmissions of the same segment reaches or
        exceeds threshold R1, pass negative advice (see Section 3.3.1.4
        of [19]) to the IP layer, to trigger dead-gateway diagnosis.

   (c)  When the number of transmissions of the same segment reaches a
        threshold R2 greater than R1, close the connection.

   (d)  An application MUST (MUST-21) be able to set the value for R2
        for a particular connection.  For example, an interactive
        application might set R2 to "infinity", giving the user control
        over when to disconnect.

   (e)  TCP implementations SHOULD inform the application of the
        delivery problem (unless such information has been disabled by
        the application; see the "Asynchronous Reports" section
        (Section 3.9.1.8)), when R1 is reached and before R2 (SHLD-9).
        This will allow a remote login application program to inform the
        user, for example.

   The value of R1 SHOULD correspond to at least 3 retransmissions, at
   the current RTO (SHLD-10).  The value of R2 SHOULD correspond to at
   least 100 seconds (SHLD-11).

   An attempt to open a TCP connection could fail with excessive
   retransmissions of the SYN segment or by receipt of a RST segment or
   an ICMP Port Unreachable.  SYN retransmissions MUST be handled in the
   general way just described for data retransmissions, including
   notification of the application layer.

   However, the values of R1 and R2 may be different for SYN and data
   segments.  In particular, R2 for a SYN segment MUST be set large
   enough to provide retransmission of the segment for at least 3
   minutes (MUST-23).  The application can close the connection (i.e.,
   give up on the open attempt) sooner, of course.

#### 3.8.4. TCP Keep-Alives

   A TCP connection is said to be "idle" if for some long amount of time
   there have been no incoming segments received and there is no new or
   unacknowledged data to be sent.

   Implementers MAY include "keep-alives" in their TCP implementations
   (MAY-5), although this practice is not universally accepted.  Some
   TCP implementations, however, have included a keep-alive mechanism.
   To confirm that an idle connection is still active, these
   implementations send a probe segment designed to elicit a response
   from the TCP peer.  Such a segment generally contains SEG.SEQ =
   SND.NXT-1 and may or may not contain one garbage octet of data.  If
   keep-alives are included, the application MUST be able to turn them
   on or off for each TCP connection (MUST-24), and they MUST default to
   off (MUST-25).

   Keep-alive packets MUST only be sent when no sent data is
   outstanding, and no data or acknowledgment packets have been received
   for the connection within an interval (MUST-26).  This interval MUST
   be configurable (MUST-27) and MUST default to no less than two hours
   (MUST-28).

   It is extremely important to remember that ACK segments that contain
   no data are not reliably transmitted by TCP.  Consequently, if a
   keep-alive mechanism is implemented it MUST NOT interpret failure to
   respond to any specific probe as a dead connection (MUST-29).

   An implementation SHOULD send a keep-alive segment with no data
   (SHLD-12); however, it MAY be configurable to send a keep-alive
   segment containing one garbage octet (MAY-6), for compatibility with
   erroneous TCP implementations.

#### 3.8.5. The Communication of Urgent Information

   As a result of implementation differences and middlebox interactions,
   new applications SHOULD NOT employ the TCP urgent mechanism (SHLD-
   13).  However, TCP implementations MUST still include support for the
   urgent mechanism (MUST-30).  Information on how some TCP
   implementations interpret the urgent pointer can be found in RFC 6093   [39].

   The objective of the TCP urgent mechanism is to allow the sending
   user to stimulate the receiving user to accept some urgent data and
   to permit the receiving TCP endpoint to indicate to the receiving
   user when all the currently known urgent data has been received by
   the user.

   This mechanism permits a point in the data stream to be designated as
   the end of urgent information.  Whenever this point is in advance of
   the receive sequence number (RCV.NXT) at the receiving TCP endpoint,
   then the TCP implementation must tell the user to go into "urgent
   mode"; when the receive sequence number catches up to the urgent
   pointer, the TCP implementation must tell user to go into "normal
   mode".  If the urgent pointer is updated while the user is in "urgent
   mode", the update will be invisible to the user.

   The method employs an urgent field that is carried in all segments
   transmitted.  The URG control flag indicates that the urgent field is
   meaningful and must be added to the segment sequence number to yield
   the urgent pointer.  The absence of this flag indicates that there is
   no urgent data outstanding.

   To send an urgent indication, the user must also send at least one
   data octet.  If the sending user also indicates a push, timely
   delivery of the urgent information to the destination process is
   enhanced.  Note that because changes in the urgent pointer correspond
   to data being written by a sending application, the urgent pointer
   cannot "recede" in the sequence space, but a TCP receiver should be
   robust to invalid urgent pointer values.

   A TCP implementation MUST support a sequence of urgent data of any
   length (MUST-31) [19].

   The urgent pointer MUST point to the sequence number of the octet
   following the urgent data (MUST-62).

   A TCP implementation MUST (MUST-32) inform the application layer
   asynchronously whenever it receives an urgent pointer and there was
   previously no pending urgent data, or whenever the urgent pointer
   advances in the data stream.  The TCP implementation MUST (MUST-33)
   provide a way for the application to learn how much urgent data
   remains to be read from the connection, or at least to determine
   whether more urgent data remains to be read [19].

#### 3.8.6. Managing the Window

   The window sent in each segment indicates the range of sequence
   numbers the sender of the window (the data receiver) is currently
   prepared to accept.  There is an assumption that this is related to
   the data buffer space currently available for this connection.

   The sending TCP endpoint packages the data to be transmitted into
   segments that fit the current window, and may repackage segments on
   the retransmission queue.  Such repackaging is not required but may
   be helpful.

   In a connection with a one-way data flow, the window information will
   be carried in acknowledgment segments that all have the same sequence
   number, so there will be no way to reorder them if they arrive out of
   order.  This is not a serious problem, but it will allow the window
   information to be on occasion temporarily based on old reports from
   the data receiver.  A refinement to avoid this problem is to act on
   the window information from segments that carry the highest
   acknowledgment number (that is, segments with an acknowledgment
   number equal to or greater than the highest previously received).

   Indicating a large window encourages transmissions.  If more data
   arrives than can be accepted, it will be discarded.  This will result
   in excessive retransmissions, adding unnecessarily to the load on the
   network and the TCP endpoints.  Indicating a small window may
   restrict the transmission of data to the point of introducing a
   round-trip delay between each new segment transmitted.

   The mechanisms provided allow a TCP endpoint to advertise a large
   window and to subsequently advertise a much smaller window without
   having accepted that much data.  This so-called "shrinking the
   window" is strongly discouraged.  The robustness principle [19]
   dictates that TCP peers will not shrink the window themselves, but
   will be prepared for such behavior on the part of other TCP peers.

   A TCP receiver SHOULD NOT shrink the window, i.e., move the right
   window edge to the left (SHLD-14).  However, a sending TCP peer MUST
   be robust against window shrinking, which may cause the "usable
   window" (see Section 3.8.6.2.1) to become negative (MUST-34).

   If this happens, the sender SHOULD NOT send new data (SHLD-15), but
   SHOULD retransmit normally the old unacknowledged data between
   SND.UNA and SND.UNA+SND.WND (SHLD-16).  The sender MAY also
   retransmit old data beyond SND.UNA+SND.WND (MAY-7), but SHOULD NOT
   time out the connection if data beyond the right window edge is not
   acknowledged (SHLD-17).  If the window shrinks to zero, the TCP
   implementation MUST probe it in the standard way (described below)
   (MUST-35).

##### 3.8.6.1. Zero-Window Probing

   The sending TCP peer must regularly transmit at least one octet of
   new data (if available), or retransmit to the receiving TCP peer even
   if the send window is zero, in order to "probe" the window.  This
   retransmission is essential to guarantee that when either TCP peer
   has a zero window the reopening of the window will be reliably
   reported to the other.  This is referred to as Zero-Window Probing
   (ZWP) in other documents.

   Probing of zero (offered) windows MUST be supported (MUST-36).

   A TCP implementation MAY keep its offered receive window closed
   indefinitely (MAY-8).  As long as the receiving TCP peer continues to
   send acknowledgments in response to the probe segments, the sending
   TCP peer MUST allow the connection to stay open (MUST-37).  This
   enables TCP to function in scenarios such as the "printer ran out of
   paper" situation described in Section 4.2.2.17 of [19].  The behavior
   is subject to the implementation's resource management concerns, as
   noted in [41].

   When the receiving TCP peer has a zero window and a segment arrives,
   it must still send an acknowledgment showing its next expected
   sequence number and current window (zero).

   The transmitting host SHOULD send the first zero-window probe when a
   zero window has existed for the retransmission timeout period (SHLD-
   29) (Section 3.8.1), and SHOULD increase exponentially the interval
   between successive probes (SHLD-30).

##### 3.8.6.2. Silly Window Syndrome Avoidance

   The "Silly Window Syndrome" (SWS) is a stable pattern of small
   incremental window movements resulting in extremely poor TCP
   performance.  Algorithms to avoid SWS are described below for both
   the sending side and the receiving side.  RFC 1122 contains more
   detailed discussion of the SWS problem.  Note that the Nagle
   algorithm and the sender SWS avoidance algorithm play complementary
   roles in improving performance.  The Nagle algorithm discourages
   sending tiny segments when the data to be sent increases in small
   increments, while the SWS avoidance algorithm discourages small
   segments resulting from the right window edge advancing in small
   increments.

###### 3.8.6.2.1. Sender's Algorithm -- When to Send Data

   A TCP implementation MUST include a SWS avoidance algorithm in the
   sender (MUST-38).

   The Nagle algorithm from Section 3.7.4 additionally describes how to
   coalesce short segments.

   The sender's SWS avoidance algorithm is more difficult than the
   receiver's because the sender does not know (directly) the receiver's
   total buffer space (RCV.BUFF).  An approach that has been found to
   work well is for the sender to calculate Max(SND.WND), which is the
   maximum send window it has seen so far on the connection, and to use
   this value as an estimate of RCV.BUFF.  Unfortunately, this can only
   be an estimate; the receiver may at any time reduce the size of
   RCV.BUFF.  To avoid a resulting deadlock, it is necessary to have a
   timeout to force transmission of data, overriding the SWS avoidance
   algorithm.  In practice, this timeout should seldom occur.

   The "usable window" is:

      U = SND.UNA + SND.WND - SND.NXT

   i.e., the offered window less the amount of data sent but not
   acknowledged.  If D is the amount of data queued in the sending TCP
   endpoint but not yet sent, then the following set of rules is
   recommended.

   Send data:

   (1)  if a maximum-sized segment can be sent, i.e., if:

           min(D,U) >= Eff.snd.MSS;

   (2)  or if the data is pushed and all queued data can be sent now,
        i.e., if:

           [SND.NXT = SND.UNA and] PUSHed and D <= U

        (the bracketed condition is imposed by the Nagle algorithm);

   (3)  or if at least a fraction Fs of the maximum window can be sent,
        i.e., if:

           [SND.NXT = SND.UNA and]

              min(D,U) >= Fs * Max(SND.WND);

   (4)  or if the override timeout occurs.

   Here Fs is a fraction whose recommended value is 1/2.  The override
   timeout should be in the range 0.1 - 1.0 seconds.  It may be
   convenient to combine this timer with the timer used to probe zero
   windows (Section 3.8.6.1).

###### 3.8.6.2.2. Receiver's Algorithm -- When to Send a Window Update

   A TCP implementation MUST include a SWS avoidance algorithm in the
   receiver (MUST-39).

   The receiver's SWS avoidance algorithm determines when the right
   window edge may be advanced; this is customarily known as "updating
   the window".  This algorithm combines with the delayed ACK algorithm
   (Section 3.8.6.3) to determine when an ACK segment containing the
   current window will really be sent to the receiver.

   The solution to receiver SWS is to avoid advancing the right window
   edge RCV.NXT+RCV.WND in small increments, even if data is received
   from the network in small segments.

   Suppose the total receive buffer space is RCV.BUFF.  At any given
   moment, RCV.USER octets of this total may be tied up with data that
   has been received and acknowledged but that the user process has not
   yet consumed.  When the connection is quiescent, RCV.WND = RCV.BUFF
   and RCV.USER = 0.

   Keeping the right window edge fixed as data arrives and is
   acknowledged requires that the receiver offer less than its full
   buffer space, i.e., the receiver must specify a RCV.WND that keeps
   RCV.NXT+RCV.WND constant as RCV.NXT increases.  Thus, the total
   buffer space RCV.BUFF is generally divided into three parts:

                  |<------- RCV.BUFF ---------------->|
                       1             2            3
              ----|---------|------------------|------|----
                         RCV.NXT               ^
                                            (Fixed)

              1 - RCV.USER =  data received but not yet consumed;
              2 - RCV.WND =   space advertised to sender;
              3 - Reduction = space available but not yet
                              advertised.

   The suggested SWS avoidance algorithm for the receiver is to keep
   RCV.NXT+RCV.WND fixed until the reduction satisfies:

                RCV.BUFF - RCV.USER - RCV.WND  >=

                       min( Fr * RCV.BUFF, Eff.snd.MSS )

   where Fr is a fraction whose recommended value is 1/2, and
   Eff.snd.MSS is the effective send MSS for the connection (see Section 3.7.1).  When the inequality is satisfied, RCV.WND is set to
   RCV.BUFF-RCV.USER.

   Note that the general effect of this algorithm is to advance RCV.WND
   in increments of Eff.snd.MSS (for realistic receive buffers:
   Eff.snd.MSS < RCV.BUFF/2).  Note also that the receiver must use its
   own Eff.snd.MSS, making the assumption that it is the same as the
   sender's.

##### 3.8.6.3. Delayed Acknowledgments -- When to Send an ACK Segment

   A host that is receiving a stream of TCP data segments can increase
   efficiency in both the network and the hosts by sending fewer than
   one ACK (acknowledgment) segment per data segment received; this is
   known as a "delayed ACK".

   A TCP endpoint SHOULD implement a delayed ACK (SHLD-18), but an ACK
   should not be excessively delayed; in particular, the delay MUST be
   less than 0.5 seconds (MUST-40).  An ACK SHOULD be generated for at
   least every second full-sized segment or 2*RMSS bytes of new data
   (where RMSS is the MSS specified by the TCP endpoint receiving the
   segments to be acknowledged, or the default value if not specified)
   (SHLD-19).  Excessive delays on ACKs can disturb the round-trip
   timing and packet "clocking" algorithms.  More complete discussion of
   delayed ACK behavior is in Section 4.2 of RFC 5681 [8], including
   recommendations to immediately acknowledge out-of-order segments,
   segments above a gap in sequence space, or segments that fill all or
   part of a gap, in order to accelerate loss recovery.

   Note that there are several current practices that further lead to a
   reduced number of ACKs, including generic receive offload (GRO) [72],
   ACK compression, and ACK decimation [28].

### 3.9. Interfaces

   There are of course two interfaces of concern: the user/TCP interface
   and the TCP/lower-level interface.  We have a fairly elaborate model
   of the user/TCP interface, but the interface to the lower-level
   protocol module is left unspecified here since it will be specified
   in detail by the specification of the lower-level protocol.  For the
   case that the lower level is IP, we note some of the parameter values
   that TCP implementations might use.

#### 3.9.1. User/TCP Interface

   The following functional description of user commands to the TCP
   implementation is, at best, fictional, since every operating system
   will have different facilities.  Consequently, we must warn readers
   that different TCP implementations may have different user
   interfaces.  However, all TCP implementations must provide a certain
   minimum set of services to guarantee that all TCP implementations can
   support the same protocol hierarchy.  This section specifies the
   functional interfaces required of all TCP implementations.

   Section 3.1 of [53] also identifies primitives provided by TCP and
   could be used as an additional reference for implementers.

   The following sections functionally characterize a user/TCP
   interface.  The notation used is similar to most procedure or
   function calls in high-level languages, but this usage is not meant
   to rule out trap-type service calls.

   The user commands described below specify the basic functions the TCP
   implementation must perform to support interprocess communication.
   Individual implementations must define their own exact format and may
   provide combinations or subsets of the basic functions in single
   calls.  In particular, some implementations may wish to automatically
   OPEN a connection on the first SEND or RECEIVE issued by the user for
   a given connection.

   In providing interprocess communication facilities, the TCP
   implementation must not only accept commands, but must also return
   information to the processes it serves.  The latter consists of:

   (a)  general information about a connection (e.g., interrupts, remote
        close, binding of unspecified remote socket).

   (b)  replies to specific user commands indicating success or various
        types of failure.

##### 3.9.1.1. Open

   Format: OPEN (local port, remote socket, active/passive [, timeout]
   [, Diffserv field] [, security/compartment] [, local IP address] [,
   options]) -> local connection name

   If the active/passive flag is set to passive, then this is a call to
   LISTEN for an incoming connection.  A passive OPEN may have either a
   fully specified remote socket to wait for a particular connection or
   an unspecified remote socket to wait for any call.  A fully specified
   passive call can be made active by the subsequent execution of a
   SEND.

   A transmission control block (TCB) is created and partially filled in
   with data from the OPEN command parameters.

   Every passive OPEN call either creates a new connection record in
   LISTEN state, or it returns an error; it MUST NOT affect any
   previously created connection record (MUST-41).

   A TCP implementation that supports multiple concurrent connections
   MUST provide an OPEN call that will functionally allow an application
   to LISTEN on a port while a connection block with the same local port
   is in SYN-SENT or SYN-RECEIVED state (MUST-42).

   On an active OPEN command, the TCP endpoint will begin the procedure
   to synchronize (i.e., establish) the connection at once.

   The timeout, if present, permits the caller to set up a timeout for
   all data submitted to TCP.  If data is not successfully delivered to
   the destination within the timeout period, the TCP endpoint will
   abort the connection.  The present global default is five minutes.

   The TCP implementation or some component of the operating system will
   verify the user's authority to open a connection with the specified
   Diffserv field value or security/compartment.  The absence of a
   Diffserv field value or security/compartment specification in the
   OPEN call indicates the default values must be used.

   TCP will accept incoming requests as matching only if the security/
   compartment information is exactly the same as that requested in the
   OPEN call.

   The Diffserv field value indicated by the user only impacts outgoing
   packets, may be altered en route through the network, and has no
   direct bearing or relation to received packets.

   A local connection name will be returned to the user by the TCP
   implementation.  The local connection name can then be used as a
   shorthand term for the connection defined by the <local socket,
   remote socket> pair.

   The optional "local IP address" parameter MUST be supported to allow
   the specification of the local IP address (MUST-43).  This enables
   applications that need to select the local IP address used when
   multihoming is present.

   A passive OPEN call with a specified "local IP address" parameter
   will await an incoming connection request to that address.  If the
   parameter is unspecified, a passive OPEN will await an incoming
   connection request to any local IP address and then bind the local IP
   address of the connection to the particular address that is used.

   For an active OPEN call, a specified "local IP address" parameter
   will be used for opening the connection.  If the parameter is
   unspecified, the host will choose an appropriate local IP address
   (see RFC 1122, Section 3.3.4.2).

   If an application on a multihomed host does not specify the local IP
   address when actively opening a TCP connection, then the TCP
   implementation MUST ask the IP layer to select a local IP address
   before sending the (first) SYN (MUST-44).  See the function
   GET_SRCADDR() in Section 3.4 of RFC 1122.

   At all other times, a previous segment has either been sent or
   received on this connection, and TCP implementations MUST use the
   same local address that was used in those previous segments (MUST-
   45).

   A TCP implementation MUST reject as an error a local OPEN call for an
   invalid remote IP address (e.g., a broadcast or multicast address)
   (MUST-46).

##### 3.9.1.2. Send

   Format: SEND (local connection name, buffer address, byte count,
   URGENT flag [, PUSH flag] [, timeout])

   This call causes the data contained in the indicated user buffer to
   be sent on the indicated connection.  If the connection has not been
   opened, the SEND is considered an error.  Some implementations may
   allow users to SEND first; in which case, an automatic OPEN would be
   done.  For example, this might be one way for application data to be
   included in SYN segments.  If the calling process is not authorized
   to use this connection, an error is returned.

   A TCP endpoint MAY implement PUSH flags on SEND calls (MAY-15).  If
   PUSH flags are not implemented, then the sending TCP peer: (1) MUST
   NOT buffer data indefinitely (MUST-60), and (2) MUST set the PSH bit
   in the last buffered segment (i.e., when there is no more queued data
   to be sent) (MUST-61).  The remaining description below assumes the
   PUSH flag is supported on SEND calls.

   If the PUSH flag is set, the application intends the data to be
   transmitted promptly to the receiver, and the PSH bit will be set in
   the last TCP segment created from the buffer.

   The PSH bit is not a record marker and is independent of segment
   boundaries.  The transmitter SHOULD collapse successive bits when it
   packetizes data, to send the largest possible segment (SHLD-27).

   If the PUSH flag is not set, the data may be combined with data from
   subsequent SENDs for transmission efficiency.  When an application
   issues a series of SEND calls without setting the PUSH flag, the TCP
   implementation MAY aggregate the data internally without sending it
   (MAY-16).  Note that when the Nagle algorithm is in use, TCP
   implementations may buffer the data before sending, without regard to
   the PUSH flag (see Section 3.7.4).

   An application program is logically required to set the PUSH flag in
   a SEND call whenever it needs to force delivery of the data to avoid
   a communication deadlock.  However, a TCP implementation SHOULD send
   a maximum-sized segment whenever possible (SHLD-28) to improve
   performance (see Section 3.8.6.2.1).

   New applications SHOULD NOT set the URGENT flag [39] due to
   implementation differences and middlebox issues (SHLD-13).

   If the URGENT flag is set, segments sent to the destination TCP peer
   will have the urgent pointer set.  The receiving TCP peer will signal
   the urgent condition to the receiving process if the urgent pointer
   indicates that data preceding the urgent pointer has not been
   consumed by the receiving process.  The purpose of the URGENT flag is
   to stimulate the receiver to process the urgent data and to indicate
   to the receiver when all the currently known urgent data has been
   received.  The number of times the sending user's TCP implementation
   signals urgent will not necessarily be equal to the number of times
   the receiving user will be notified of the presence of urgent data.

   If no remote socket was specified in the OPEN, but the connection is
   established (e.g., because a LISTENing connection has become specific
   due to a remote segment arriving for the local socket), then the
   designated buffer is sent to the implied remote socket.  Users who
   make use of OPEN with an unspecified remote socket can make use of
   SEND without ever explicitly knowing the remote socket address.

   However, if a SEND is attempted before the remote socket becomes
   specified, an error will be returned.  Users can use the STATUS call
   to determine the status of the connection.  Some TCP implementations
   may notify the user when an unspecified socket is bound.

   If a timeout is specified, the current user timeout for this
   connection is changed to the new one.

   In the simplest implementation, SEND would not return control to the
   sending process until either the transmission was complete or the
   timeout had been exceeded.  However, this simple method is both
   subject to deadlocks (for example, both sides of the connection might
   try to do SENDs before doing any RECEIVEs) and offers poor
   performance, so it is not recommended.  A more sophisticated
   implementation would return immediately to allow the process to run
   concurrently with network I/O, and, furthermore, to allow multiple
   SENDs to be in progress.  Multiple SENDs are served in first come,
   first served order, so the TCP endpoint will queue those it cannot
   service immediately.

   We have implicitly assumed an asynchronous user interface in which a
   SEND later elicits some kind of SIGNAL or pseudo-interrupt from the
   serving TCP endpoint.  An alternative is to return a response
   immediately.  For instance, SENDs might return immediate local
   acknowledgment, even if the segment sent had not been acknowledged by
   the distant TCP endpoint.  We could optimistically assume eventual
   success.  If we are wrong, the connection will close anyway due to
   the timeout.  In implementations of this kind (synchronous), there
   will still be some asynchronous signals, but these will deal with the
   connection itself, and not with specific segments or buffers.

   In order for the process to distinguish among error or success
   indications for different SENDs, it might be appropriate for the
   buffer address to be returned along with the coded response to the
   SEND request.  TCP-to-user signals are discussed below, indicating
   the information that should be returned to the calling process.

##### 3.9.1.3. Receive

   Format: RECEIVE (local connection name, buffer address, byte count)
   -> byte count, URGENT flag [, PUSH flag]

   This command allocates a receiving buffer associated with the
   specified connection.  If no OPEN precedes this command or the
   calling process is not authorized to use this connection, an error is
   returned.

   In the simplest implementation, control would not return to the
   calling program until either the buffer was filled or some error
   occurred, but this scheme is highly subject to deadlocks.  A more
   sophisticated implementation would permit several RECEIVEs to be
   outstanding at once.  These would be filled as segments arrive.  This
   strategy permits increased throughput at the cost of a more elaborate
   scheme (possibly asynchronous) to notify the calling program that a
   PUSH has been seen or a buffer filled.

   A TCP receiver MAY pass a received PSH bit to the application layer
   via the PUSH flag in the interface (MAY-17), but it is not required
   (this was clarified in RFC 1122, Section 4.2.2.2).  The remainder of
   text describing the RECEIVE call below assumes that passing the PUSH
   indication is supported.

   If enough data arrive to fill the buffer before a PUSH is seen, the
   PUSH flag will not be set in the response to the RECEIVE.  The buffer
   will be filled with as much data as it can hold.  If a PUSH is seen
   before the buffer is filled, the buffer will be returned partially
   filled and PUSH indicated.

   If there is urgent data, the user will have been informed as soon as
   it arrived via a TCP-to-user signal.  The receiving user should thus
   be in "urgent mode".  If the URGENT flag is on, additional urgent
   data remains.  If the URGENT flag is off, this call to RECEIVE has
   returned all the urgent data, and the user may now leave "urgent
   mode".  Note that data following the urgent pointer (non-urgent data)
   cannot be delivered to the user in the same buffer with preceding
   urgent data unless the boundary is clearly marked for the user.

   To distinguish among several outstanding RECEIVEs and to take care of
   the case that a buffer is not completely filled, the return code is
   accompanied by both a buffer pointer and a byte count indicating the
   actual length of the data received.

   Alternative implementations of RECEIVE might have the TCP endpoint
   allocate buffer storage, or the TCP endpoint might share a ring
   buffer with the user.

##### 3.9.1.4. Close

   Format: CLOSE (local connection name)

   This command causes the connection specified to be closed.  If the
   connection is not open or the calling process is not authorized to
   use this connection, an error is returned.  Closing connections is
   intended to be a graceful operation in the sense that outstanding
   SENDs will be transmitted (and retransmitted), as flow control
   permits, until all have been serviced.  Thus, it should be acceptable
   to make several SEND calls, followed by a CLOSE, and expect all the
   data to be sent to the destination.  It should also be clear that
   users should continue to RECEIVE on CLOSING connections since the
   remote peer may be trying to transmit the last of its data.  Thus,
   CLOSE means "I have no more to send" but does not mean "I will not
   receive any more."  It may happen (if the user-level protocol is not
   well thought out) that the closing side is unable to get rid of all
   its data before timing out.  In this event, CLOSE turns into ABORT,
   and the closing TCP peer gives up.

   The user may CLOSE the connection at any time on their own
   initiative, or in response to various prompts from the TCP
   implementation (e.g., remote close executed, transmission timeout
   exceeded, destination inaccessible).

   Because closing a connection requires communication with the remote
   TCP peer, connections may remain in the closing state for a short
   time.  Attempts to reopen the connection before the TCP peer replies
   to the CLOSE command will result in error responses.

   Close also implies push function.

##### 3.9.1.5. Status

   Format: STATUS (local connection name) -> status data

   This is an implementation-dependent user command and could be
   excluded without adverse effect.  Information returned would
   typically come from the TCB associated with the connection.

   This command returns a data block containing the following
   information:

      local socket,

      remote socket,

      local connection name,

      receive window,

      send window,

      connection state,

      number of buffers awaiting acknowledgment,

      number of buffers pending receipt,

      urgent state,

      Diffserv field value,

      security/compartment, and

      transmission timeout.

   Depending on the state of the connection, or on the implementation
   itself, some of this information may not be available or meaningful.
   If the calling process is not authorized to use this connection, an
   error is returned.  This prevents unauthorized processes from gaining
   information about a connection.

##### 3.9.1.6. Abort

   Format: ABORT (local connection name)

   This command causes all pending SENDs and RECEIVES to be aborted, the
   TCB to be removed, and a special RST message to be sent to the remote
   TCP peer of the connection.  Depending on the implementation, users
   may receive abort indications for each outstanding SEND or RECEIVE,
   or may simply receive an ABORT-acknowledgment.

##### 3.9.1.7. Flush

   Some TCP implementations have included a FLUSH call, which will empty
   the TCP send queue of any data that the user has issued SEND calls
   for but is still to the right of the current send window.  That is,
   it flushes as much queued send data as possible without losing
   sequence number synchronization.  The FLUSH call MAY be implemented
   (MAY-14).

##### 3.9.1.8. Asynchronous Reports

   There MUST be a mechanism for reporting soft TCP error conditions to
   the application (MUST-47).  Generically, we assume this takes the
   form of an application-supplied ERROR_REPORT routine that may be
   upcalled asynchronously from the transport layer:

      ERROR_REPORT(local connection name, reason, subreason)

   The precise encoding of the reason and subreason parameters is not
   specified here.  However, the conditions that are reported
   asynchronously to the application MUST include:

   *  ICMP error message arrived (see Section 3.9.2.2 for description of
      handling each ICMP message type since some message types need to
      be suppressed from generating reports to the application)

   *  Excessive retransmissions (see Section 3.8.3)

   *  Urgent pointer advance (see Section 3.8.5)

   However, an application program that does not want to receive such
   ERROR_REPORT calls SHOULD be able to effectively disable these calls
   (SHLD-20).

##### 3.9.1.9. Set Differentiated Services Field (IPv4 TOS or IPv6 Traffic


          Class)   The application layer MUST be able to specify the Differentiated
   Services field for segments that are sent on a connection (MUST-48).
   The Differentiated Services field includes the 6-bit Differentiated
   Services Codepoint (DSCP) value.  It is not required, but the
   application SHOULD be able to change the Differentiated Services
   field during the connection lifetime (SHLD-21).  TCP implementations
   SHOULD pass the current Differentiated Services field value without
   change to the IP layer, when it sends segments on the connection
   (SHLD-22).

   The Differentiated Services field will be specified independently in
   each direction on the connection, so that the receiver application
   will specify the Differentiated Services field used for ACK segments.

   TCP implementations MAY pass the most recently received
   Differentiated Services field up to the application (MAY-9).

#### 3.9.2. TCP/Lower-Level Interface

   The TCP endpoint calls on a lower-level protocol module to actually
   send and receive information over a network.  The two current
   standard Internet Protocol (IP) versions layered below TCP are IPv4
   [1] and IPv6 [13].

   If the lower-level protocol is IPv4, it provides arguments for a type
   of service (used within the Differentiated Services field) and for a
   time to live.  TCP uses the following settings for these parameters:

   Diffserv field:  The IP header value for the Diffserv field is given
      by the user.  This includes the bits of the Diffserv Codepoint
      (DSCP).

   Time to Live (TTL):  The TTL value used to send TCP segments MUST be
      configurable (MUST-49).

      *  Note that RFC 793 specified one minute (60 seconds) as a
         constant for the TTL because the assumed maximum segment
         lifetime was two minutes.  This was intended to explicitly ask
         that a segment be destroyed if it could not be delivered by the
         internet system within one minute.  RFC 1122 updated RFC 793 to
         require that the TTL be configurable.

      *  Note that the Diffserv field is permitted to change during a
         connection (Section 4.2.4.2 of RFC 1122).  However, the
         application interface might not support this ability, and the
         application does not have knowledge about individual TCP
         segments, so this can only be done on a coarse granularity, at
         best.  This limitation is further discussed in RFC 7657         (Sections 5.1, 5.3, and 6) [50].  Generally, an application
         SHOULD NOT change the Diffserv field value during the course of
         a connection (SHLD-23).

   Any lower-level protocol will have to provide the source address,
   destination address, and protocol fields, and some way to determine
   the "TCP length", both to provide the functional equivalent service
   of IP and to be used in the TCP checksum.

   When received options are passed up to TCP from the IP layer, a TCP
   implementation MUST ignore options that it does not understand (MUST-
   50).

   A TCP implementation MAY support the Timestamp (MAY-10) and Record
   Route (MAY-11) Options.

##### 3.9.2.1. Source Routing

   If the lower level is IP (or other protocol that provides this
   feature) and source routing is used, the interface must allow the
   route information to be communicated.  This is especially important
   so that the source and destination addresses used in the TCP checksum
   be the originating source and ultimate destination.  It is also
   important to preserve the return route to answer connection requests.

   An application MUST be able to specify a source route when it
   actively opens a TCP connection (MUST-51), and this MUST take
   precedence over a source route received in a datagram (MUST-52).

   When a TCP connection is OPENed passively and a packet arrives with a
   completed IP Source Route Option (containing a return route), TCP
   implementations MUST save the return route and use it for all
   segments sent on this connection (MUST-53).  If a different source
   route arrives in a later segment, the later definition SHOULD
   override the earlier one (SHLD-24).

##### 3.9.2.2. ICMP Messages

   TCP implementations MUST act on an ICMP error message passed up from
   the IP layer, directing it to the connection that created the error
   (MUST-54).  The necessary demultiplexing information can be found in
   the IP header contained within the ICMP message.

   This applies to ICMPv6 in addition to IPv4 ICMP.

   [35] contains discussion of specific ICMP and ICMPv6 messages
   classified as either "soft" or "hard" errors that may bear different
   responses.  Treatment for classes of ICMP messages is described
   below:

   Source Quench
     TCP implementations MUST silently discard any received ICMP Source
     Quench messages (MUST-55).  See [11] for discussion.

   Soft Errors
     For IPv4 ICMP, these include: Destination Unreachable -- codes 0,
     1, 5; Time Exceeded -- codes 0, 1; and Parameter Problem.

     For ICMPv6, these include: Destination Unreachable -- codes 0, 3;
     Time Exceeded -- codes 0, 1; and Parameter Problem -- codes 0, 1,
     2.

     Since these Unreachable messages indicate soft error conditions, a
     TCP implementation MUST NOT abort the connection (MUST-56), and it
     SHOULD make the information available to the application (SHLD-25).

   Hard Errors
     For ICMP these include Destination Unreachable -- codes 2-4.

     These are hard error conditions, so TCP implementations SHOULD
     abort the connection (SHLD-26).  [35] notes that some
     implementations do not abort connections when an ICMP hard error is
     received for a connection that is in any of the synchronized
     states.

   Note that [35], Section 4 describes widespread implementation
   behavior that treats soft errors as hard errors during connection
   establishment.

##### 3.9.2.3. Source Address Validation



   RFC 1122 requires addresses to be validated in incoming SYN packets:

   |  An incoming SYN with an invalid source address MUST be ignored
   |  either by TCP or by the IP layer [(MUST-63)] (see
   |  Section 3.2.1.3).
   |
   |  A TCP implementation MUST silently discard an incoming SYN segment
   |  that is addressed to a broadcast or multicast address [(MUST-57)].

   This prevents connection state and replies from being erroneously
   generated, and implementers should note that this guidance is
   applicable to all incoming segments, not just SYNs, as specifically
   indicated in RFC 1122.

### 3.10. Event Processing

   The processing depicted in this section is an example of one possible
   implementation.  Other implementations may have slightly different
   processing sequences, but they should differ from those in this
   section only in detail, not in substance.

   The activity of the TCP endpoint can be characterized as responding
   to events.  The events that occur can be cast into three categories:
   user calls, arriving segments, and timeouts.  This section describes
   the processing the TCP endpoint does in response to each of the
   events.  In many cases, the processing required depends on the state
   of the connection.

   Events that occur:

      User Calls

         OPEN

         SEND

         RECEIVE

         CLOSE

         ABORT

         STATUS

      Arriving Segments

         SEGMENT ARRIVES

      Timeouts

         USER TIMEOUT

         RETRANSMISSION TIMEOUT

         TIME-WAIT TIMEOUT

   The model of the TCP/user interface is that user commands receive an
   immediate return and possibly a delayed response via an event or
   pseudo-interrupt.  In the following descriptions, the term "signal"
   means cause a delayed response.

   Error responses in this document are identified by character strings.
   For example, user commands referencing connections that do not exist
   receive "error: connection not open".

   Please note in the following that all arithmetic on sequence numbers,
   acknowledgment numbers, windows, et cetera, is modulo 2^32 (the size
   of the sequence number space).  Also note that "=<" means less than
   or equal to (modulo 2^32).

   A natural way to think about processing incoming segments is to
   imagine that they are first tested for proper sequence number (i.e.,
   that their contents lie in the range of the expected "receive window"
   in the sequence number space) and then that they are generally queued
   and processed in sequence number order.

   When a segment overlaps other already received segments, we
   reconstruct the segment to contain just the new data and adjust the
   header fields to be consistent.

   Note that if no state change is mentioned, the TCP connection stays
   in the same state.

#### 3.10.1. OPEN Call

   CLOSED STATE (i.e., TCB does not exist)

   *  Create a new transmission control block (TCB) to hold connection
      state information.  Fill in local socket identifier, remote
      socket, Diffserv field, security/compartment, and user timeout
      information.  Note that some parts of the remote socket may be
      unspecified in a passive OPEN and are to be filled in by the
      parameters of the incoming SYN segment.  Verify the security and
      Diffserv value requested are allowed for this user, if not, return
      "error: Diffserv value not allowed" or "error: security/
      compartment not allowed".  If passive, enter the LISTEN state and
      return.  If active and the remote socket is unspecified, return
      "error: remote socket unspecified"; if active and the remote
      socket is specified, issue a SYN segment.  An initial send
      sequence number (ISS) is selected.  A SYN segment of the form
      <SEQ=ISS><CTL=SYN> is sent.  Set SND.UNA to ISS, SND.NXT to ISS+1,
      enter SYN-SENT state, and return.

   *  If the caller does not have access to the local socket specified,
      return "error: connection illegal for this process".  If there is
      no room to create a new connection, return "error: insufficient
      resources".

   LISTEN STATE

   *  If the OPEN call is active and the remote socket is specified,
      then change the connection from passive to active, select an ISS.
      Send a SYN segment, set SND.UNA to ISS, SND.NXT to ISS+1.  Enter
      SYN-SENT state.  Data associated with SEND may be sent with SYN
      segment or queued for transmission after entering ESTABLISHED
      state.  The urgent bit if requested in the command must be sent
      with the data segments sent as a result of this command.  If there
      is no room to queue the request, respond with "error: insufficient
      resources".  If the remote socket was not specified, then return
      "error: remote socket unspecified".

   SYN-SENT STATE

   SYN-RECEIVED STATE

   ESTABLISHED STATE

   FIN-WAIT-1 STATE

   FIN-WAIT-2 STATE

   CLOSE-WAIT STATE

   CLOSING STATE

   LAST-ACK STATE

   TIME-WAIT STATE

   *  Return "error: connection already exists".

#### 3.10.2. SEND Call

   CLOSED STATE (i.e., TCB does not exist)

   *  If the user does not have access to such a connection, then return
      "error: connection illegal for this process".

   *  Otherwise, return "error: connection does not exist".

   LISTEN STATE

   *  If the remote socket is specified, then change the connection from
      passive to active, select an ISS.  Send a SYN segment, set SND.UNA
      to ISS, SND.NXT to ISS+1.  Enter SYN-SENT state.  Data associated
      with SEND may be sent with SYN segment or queued for transmission
      after entering ESTABLISHED state.  The urgent bit if requested in
      the command must be sent with the data segments sent as a result
      of this command.  If there is no room to queue the request,
      respond with "error: insufficient resources".  If the remote
      socket was not specified, then return "error: remote socket
      unspecified".

   SYN-SENT STATE

   SYN-RECEIVED STATE

   *  Queue the data for transmission after entering ESTABLISHED state.
      If no space to queue, respond with "error: insufficient
      resources".

   ESTABLISHED STATE

   CLOSE-WAIT STATE

   *  Segmentize the buffer and send it with a piggybacked
      acknowledgment (acknowledgment value = RCV.NXT).  If there is
      insufficient space to remember this buffer, simply return "error:
      insufficient resources".

   *  If the URGENT flag is set, then SND.UP <- SND.NXT and set the
      urgent pointer in the outgoing segments.

   FIN-WAIT-1 STATE

   FIN-WAIT-2 STATE

   CLOSING STATE

   LAST-ACK STATE

   TIME-WAIT STATE

   *  Return "error: connection closing" and do not service request.

#### 3.10.3. RECEIVE Call

   CLOSED STATE (i.e., TCB does not exist)

   *  If the user does not have access to such a connection, return
      "error: connection illegal for this process".

   *  Otherwise, return "error: connection does not exist".

   LISTEN STATE

   SYN-SENT STATE

   SYN-RECEIVED STATE

   *  Queue for processing after entering ESTABLISHED state.  If there
      is no room to queue this request, respond with "error:
      insufficient resources".

   ESTABLISHED STATE

   FIN-WAIT-1 STATE

   FIN-WAIT-2 STATE

   *  If insufficient incoming segments are queued to satisfy the
      request, queue the request.  If there is no queue space to
      remember the RECEIVE, respond with "error: insufficient
      resources".

   *  Reassemble queued incoming segments into receive buffer and return
      to user.  Mark "push seen" (PUSH) if this is the case.

   *  If RCV.UP is in advance of the data currently being passed to the
      user, notify the user of the presence of urgent data.

   *  When the TCP endpoint takes responsibility for delivering data to
      the user, that fact must be communicated to the sender via an
      acknowledgment.  The formation of such an acknowledgment is
      described below in the discussion of processing an incoming
      segment.

   CLOSE-WAIT STATE

   *  Since the remote side has already sent FIN, RECEIVEs must be
      satisfied by data already on hand, but not yet delivered to the
      user.  If no text is awaiting delivery, the RECEIVE will get an
      "error: connection closing" response.  Otherwise, any remaining
      data can be used to satisfy the RECEIVE.

   CLOSING STATE

   LAST-ACK STATE

   TIME-WAIT STATE

   *  Return "error: connection closing".

#### 3.10.4. CLOSE Call

   CLOSED STATE (i.e., TCB does not exist)

   *  If the user does not have access to such a connection, return
      "error: connection illegal for this process".

   *  Otherwise, return "error: connection does not exist".

   LISTEN STATE

   *  Any outstanding RECEIVEs are returned with "error: closing"
      responses.  Delete TCB, enter CLOSED state, and return.

   SYN-SENT STATE

   *  Delete the TCB and return "error: closing" responses to any queued
      SENDs, or RECEIVEs.

   SYN-RECEIVED STATE

   *  If no SENDs have been issued and there is no pending data to send,
      then form a FIN segment and send it, and enter FIN-WAIT-1 state;
      otherwise, queue for processing after entering ESTABLISHED state.

   ESTABLISHED STATE

   *  Queue this until all preceding SENDs have been segmentized, then
      form a FIN segment and send it.  In any case, enter FIN-WAIT-1
      state.

   FIN-WAIT-1 STATE

   FIN-WAIT-2 STATE

   *  Strictly speaking, this is an error and should receive an "error:
      connection closing" response.  An "ok" response would be
      acceptable, too, as long as a second FIN is not emitted (the first
      FIN may be retransmitted, though).

   CLOSE-WAIT STATE

   *  Queue this request until all preceding SENDs have been
      segmentized; then send a FIN segment, enter LAST-ACK state.

   CLOSING STATE

   LAST-ACK STATE

   TIME-WAIT STATE

   *  Respond with "error: connection closing".

#### 3.10.5. ABORT Call

   CLOSED STATE (i.e., TCB does not exist)

   *  If the user should not have access to such a connection, return
      "error: connection illegal for this process".

   *  Otherwise, return "error: connection does not exist".

   LISTEN STATE

   *  Any outstanding RECEIVEs should be returned with "error:
      connection reset" responses.  Delete TCB, enter CLOSED state, and
      return.

   SYN-SENT STATE

   *  All queued SENDs and RECEIVEs should be given "connection reset"
      notification.  Delete the TCB, enter CLOSED state, and return.

   SYN-RECEIVED STATE

   ESTABLISHED STATE

   FIN-WAIT-1 STATE

   FIN-WAIT-2 STATE

   CLOSE-WAIT STATE

   *  Send a reset segment:

      <SEQ=SND.NXT><CTL=RST>

   *  All queued SENDs and RECEIVEs should be given "connection reset"
      notification; all segments queued for transmission (except for the
      RST formed above) or retransmission should be flushed.  Delete the
      TCB, enter CLOSED state, and return.

   CLOSING STATE

   LAST-ACK STATE

   TIME-WAIT STATE

   *  Respond with "ok" and delete the TCB, enter CLOSED state, and
      return.

#### 3.10.6. STATUS Call

   CLOSED STATE (i.e., TCB does not exist)

   *  If the user should not have access to such a connection, return
      "error: connection illegal for this process".

   *  Otherwise, return "error: connection does not exist".

   LISTEN STATE

   *  Return "state = LISTEN" and the TCB pointer.

   SYN-SENT STATE

   *  Return "state = SYN-SENT" and the TCB pointer.

   SYN-RECEIVED STATE

   *  Return "state = SYN-RECEIVED" and the TCB pointer.

   ESTABLISHED STATE

   *  Return "state = ESTABLISHED" and the TCB pointer.

   FIN-WAIT-1 STATE

   *  Return "state = FIN-WAIT-1" and the TCB pointer.

   FIN-WAIT-2 STATE

   *  Return "state = FIN-WAIT-2" and the TCB pointer.

   CLOSE-WAIT STATE

   *  Return "state = CLOSE-WAIT" and the TCB pointer.

   CLOSING STATE

   *  Return "state = CLOSING" and the TCB pointer.

   LAST-ACK STATE

   *  Return "state = LAST-ACK" and the TCB pointer.

   TIME-WAIT STATE

   *  Return "state = TIME-WAIT" and the TCB pointer.

#### 3.10.7. SEGMENT ARRIVES





##### 3.10.7.1. CLOSED STATE

   If the state is CLOSED (i.e., TCB does not exist), then

      all data in the incoming segment is discarded.  An incoming
      segment containing a RST is discarded.  An incoming segment not
      containing a RST causes a RST to be sent in response.  The
      acknowledgment and sequence field values are selected to make the
      reset sequence acceptable to the TCP endpoint that sent the
      offending segment.

      If the ACK bit is off, sequence number zero is used,

         <SEQ=0><ACK=SEG.SEQ+SEG.LEN><CTL=RST,ACK>

      If the ACK bit is on,

         <SEQ=SEG.ACK><CTL=RST>

      Return.

##### 3.10.7.2. LISTEN STATE

   If the state is LISTEN, then

      First, check for a RST:

      -  An incoming RST segment could not be valid since it could not
         have been sent in response to anything sent by this incarnation
         of the connection.  An incoming RST should be ignored.  Return.

      Second, check for an ACK:

      -  Any acknowledgment is bad if it arrives on a connection still
         in the LISTEN state.  An acceptable reset segment should be
         formed for any arriving ACK-bearing segment.  The RST should be
         formatted as follows:

            <SEQ=SEG.ACK><CTL=RST>

      -  Return.

      Third, check for a SYN:

      -  If the SYN bit is set, check the security.  If the security/
         compartment on the incoming segment does not exactly match the
         security/compartment in the TCB, then send a reset and return.

            <SEQ=0><ACK=SEG.SEQ+SEG.LEN><CTL=RST,ACK>

      -  Set RCV.NXT to SEG.SEQ+1, IRS is set to SEG.SEQ, and any other
         control or text should be queued for processing later.  ISS
         should be selected and a SYN segment sent of the form:

            <SEQ=ISS><ACK=RCV.NXT><CTL=SYN,ACK>

      -  SND.NXT is set to ISS+1 and SND.UNA to ISS.  The connection
         state should be changed to SYN-RECEIVED.  Note that any other
         incoming control or data (combined with SYN) will be processed
         in the SYN-RECEIVED state, but processing of SYN and ACK should
         not be repeated.  If the listen was not fully specified (i.e.,
         the remote socket was not fully specified), then the
         unspecified fields should be filled in now.

      Fourth, other data or control:

      -  This should not be reached.  Drop the segment and return.  Any
         other control or data-bearing segment (not containing SYN) must
         have an ACK and thus would have been discarded by the ACK
         processing in the second step, unless it was first discarded by
         RST checking in the first step.

##### 3.10.7.3. SYN-SENT STATE

   If the state is SYN-SENT, then

      First, check the ACK bit:

      -  If the ACK bit is set,

         o  If SEG.ACK =< ISS or SEG.ACK > SND.NXT, send a reset (unless
            the RST bit is set, if so drop the segment and return)

               <SEQ=SEG.ACK><CTL=RST>

         o  and discard the segment.  Return.

         o  If SND.UNA < SEG.ACK =< SND.NXT, then the ACK is acceptable.
            Some deployed TCP code has used the check SEG.ACK == SND.NXT
            (using "==" rather than "=<"), but this is not appropriate
            when the stack is capable of sending data on the SYN because
            the TCP peer may not accept and acknowledge all of the data
            on the SYN.

      Second, check the RST bit:

      -  If the RST bit is set,

         o  A potential blind reset attack is described in RFC 5961 [9].
            The mitigation described in that document has specific
            applicability explained therein, and is not a substitute for
            cryptographic protection (e.g., IPsec or TCP-AO).  A TCP
            implementation that supports the mitigation described in RFC 
            5961 SHOULD first check that the sequence number exactly
            matches RCV.NXT prior to executing the action in the next
            paragraph.

         o  If the ACK was acceptable, then signal to the user "error:
            connection reset", drop the segment, enter CLOSED state,
            delete TCB, and return.  Otherwise (no ACK), drop the
            segment and return.

      Third, check the security:

      -  If the security/compartment in the segment does not exactly
         match the security/compartment in the TCB, send a reset:

         o  If there is an ACK,

               <SEQ=SEG.ACK><CTL=RST>

         o  Otherwise,

               <SEQ=0><ACK=SEG.SEQ+SEG.LEN><CTL=RST,ACK>

      -  If a reset was sent, discard the segment and return.

      Fourth, check the SYN bit:

      -  This step should be reached only if the ACK is ok, or there is
         no ACK, and the segment did not contain a RST.

      -  If the SYN bit is on and the security/compartment is
         acceptable, then RCV.NXT is set to SEG.SEQ+1, IRS is set to
         SEG.SEQ.  SND.UNA should be advanced to equal SEG.ACK (if there
         is an ACK), and any segments on the retransmission queue that
         are thereby acknowledged should be removed.

      -  If SND.UNA > ISS (our SYN has been ACKed), change the
         connection state to ESTABLISHED, form an ACK segment

            <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>

      -  and send it.  Data or controls that were queued for
         transmission MAY be included.  Some TCP implementations
         suppress sending this segment when the received segment
         contains data that will anyways generate an acknowledgment in
         the later processing steps, saving this extra acknowledgment of
         the SYN from being sent.  If there are other controls or text
         in the segment, then continue processing at the sixth step
         under Section 3.10.7.4 where the URG bit is checked; otherwise,
         return.

      -  Otherwise, enter SYN-RECEIVED, form a SYN,ACK segment

            <SEQ=ISS><ACK=RCV.NXT><CTL=SYN,ACK>

      -  and send it.  Set the variables:

            SND.WND <- SEG.WND

            SND.WL1 <- SEG.SEQ

            SND.WL2 <- SEG.ACK

         If there are other controls or text in the segment, queue them
         for processing after the ESTABLISHED state has been reached,
         return.

      -  Note that it is legal to send and receive application data on
         SYN segments (this is the "text in the segment" mentioned
         above).  There has been significant misinformation and
         misunderstanding of this topic historically.  Some firewalls
         and security devices consider this suspicious.  However, the
         capability was used in T/TCP [21] and is used in TCP Fast Open
         (TFO) [48], so is important for implementations and network
         devices to permit.

      Fifth, if neither of the SYN or RST bits is set, then drop the
      segment and return.

##### 3.10.7.4. Other States

   Otherwise,

      First, check sequence number:

      -  SYN-RECEIVED STATE

      -  ESTABLISHED STATE

      -  FIN-WAIT-1 STATE

      -  FIN-WAIT-2 STATE

      -  CLOSE-WAIT STATE

      -  CLOSING STATE

      -  LAST-ACK STATE

      -  TIME-WAIT STATE

         o  Segments are processed in sequence.  Initial tests on
            arrival are used to discard old duplicates, but further
            processing is done in SEG.SEQ order.  If a segment's
            contents straddle the boundary between old and new, only the
            new parts are processed.

         o  In general, the processing of received segments MUST be
            implemented to aggregate ACK segments whenever possible
            (MUST-58).  For example, if the TCP endpoint is processing a
            series of queued segments, it MUST process them all before
            sending any ACK segments (MUST-59).

         o  There are four cases for the acceptability test for an
            incoming segment:

            +=========+=========+======================================+
            | Segment | Receive | Test                                 |
            | Length  | Window  |                                      |
            +=========+=========+======================================+
            | 0       | 0       | SEG.SEQ = RCV.NXT                    |
            +---------+---------+--------------------------------------+
            | 0       | >0      | RCV.NXT =< SEG.SEQ <                 |
            |         |         | RCV.NXT+RCV.WND                      |
            +---------+---------+--------------------------------------+
            | >0      | 0       | not acceptable                       |
            +---------+---------+--------------------------------------+
            | >0      | >0      | RCV.NXT =< SEG.SEQ <                 |
            |         |         | RCV.NXT+RCV.WND                      |
            |         |         |                                      |
            |         |         | or                                   |
            |         |         |                                      |
            |         |         | RCV.NXT =< SEG.SEQ+SEG.LEN-1         |
            |         |         | < RCV.NXT+RCV.WND                    |
            +---------+---------+--------------------------------------+

                        Table 6: Segment Acceptability Tests

         o  In implementing sequence number validation as described
            here, please note Appendix A.2.

         o  If the RCV.WND is zero, no segments will be acceptable, but
            special allowance should be made to accept valid ACKs, URGs,
            and RSTs.

         o  If an incoming segment is not acceptable, an acknowledgment
            should be sent in reply (unless the RST bit is set, if so
            drop the segment and return):

            <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>

         o  After sending the acknowledgment, drop the unacceptable
            segment and return.

         o  Note that for the TIME-WAIT state, there is an improved
            algorithm described in [40] for handling incoming SYN
            segments that utilizes timestamps rather than relying on the
            sequence number check described here.  When the improved
            algorithm is implemented, the logic above is not applicable
            for incoming SYN segments with Timestamp Options, received
            on a connection in the TIME-WAIT state.

         o  In the following it is assumed that the segment is the
            idealized segment that begins at RCV.NXT and does not exceed
            the window.  One could tailor actual segments to fit this
            assumption by trimming off any portions that lie outside the
            window (including SYN and FIN) and only processing further
            if the segment then begins at RCV.NXT.  Segments with higher
            beginning sequence numbers SHOULD be held for later
            processing (SHLD-31).

      Second, check the RST bit:

      -  RFC 5961 [9], Section 3 describes a potential blind reset
         attack and optional mitigation approach.  This does not provide
         a cryptographic protection (e.g., as in IPsec or TCP-AO) but
         can be applicable in situations described in RFC 5961.  For
         stacks implementing the protection described in RFC 5961, the
         three checks below apply; otherwise, processing for these
         states is indicated further below.

         1)  If the RST bit is set and the sequence number is outside
             the current receive window, silently drop the segment.

         2)  If the RST bit is set and the sequence number exactly
             matches the next expected sequence number (RCV.NXT), then
             TCP endpoints MUST reset the connection in the manner
             prescribed below according to the connection state.

         3)  If the RST bit is set and the sequence number does not
             exactly match the next expected sequence value, yet is
             within the current receive window, TCP endpoints MUST send
             an acknowledgment (challenge ACK):

             <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>

             After sending the challenge ACK, TCP endpoints MUST drop
             the unacceptable segment and stop processing the incoming
             packet further.  Note that RFC 5961 and Errata ID 4772 [99]
             contain additional considerations for ACK throttling in an
             implementation.

      -  SYN-RECEIVED STATE

         o  If the RST bit is set,

            +  If this connection was initiated with a passive OPEN
               (i.e., came from the LISTEN state), then return this
               connection to LISTEN state and return.  The user need not
               be informed.  If this connection was initiated with an
               active OPEN (i.e., came from SYN-SENT state), then the
               connection was refused; signal the user "connection
               refused".  In either case, the retransmission queue
               should be flushed.  And in the active OPEN case, enter
               the CLOSED state and delete the TCB, and return.

      -  ESTABLISHED STATE

      -  FIN-WAIT-1 STATE

      -  FIN-WAIT-2 STATE

      -  CLOSE-WAIT STATE

         o  If the RST bit is set, then any outstanding RECEIVEs and
            SEND should receive "reset" responses.  All segment queues
            should be flushed.  Users should also receive an unsolicited
            general "connection reset" signal.  Enter the CLOSED state,
            delete the TCB, and return.

      -  CLOSING STATE

      -  LAST-ACK STATE

      -  TIME-WAIT STATE

         o  If the RST bit is set, then enter the CLOSED state, delete
            the TCB, and return.

      Third, check security:

      -  SYN-RECEIVED STATE

         o  If the security/compartment in the segment does not exactly
            match the security/compartment in the TCB, then send a reset
            and return.

      -  ESTABLISHED STATE

      -  FIN-WAIT-1 STATE

      -  FIN-WAIT-2 STATE

      -  CLOSE-WAIT STATE

      -  CLOSING STATE

      -  LAST-ACK STATE

      -  TIME-WAIT STATE

         o  If the security/compartment in the segment does not exactly
            match the security/compartment in the TCB, then send a
            reset; any outstanding RECEIVEs and SEND should receive
            "reset" responses.  All segment queues should be flushed.
            Users should also receive an unsolicited general "connection
            reset" signal.  Enter the CLOSED state, delete the TCB, and
            return.

      -  Note this check is placed following the sequence check to
         prevent a segment from an old connection between these port
         numbers with a different security from causing an abort of the
         current connection.

      Fourth, check the SYN bit:

      -  SYN-RECEIVED STATE

         o  If the connection was initiated with a passive OPEN, then
            return this connection to the LISTEN state and return.
            Otherwise, handle per the directions for synchronized states
            below.

      -  ESTABLISHED STATE

      -  FIN-WAIT-1 STATE

      -  FIN-WAIT-2 STATE

      -  CLOSE-WAIT STATE

      -  CLOSING STATE

      -  LAST-ACK STATE

      -  TIME-WAIT STATE

         o  If the SYN bit is set in these synchronized states, it may
            be either a legitimate new connection attempt (e.g., in the
            case of TIME-WAIT), an error where the connection should be
            reset, or the result of an attack attempt, as described in RFC 5961 [9].  For the TIME-WAIT state, new connections can
            be accepted if the Timestamp Option is used and meets
            expectations (per [40]).  For all other cases, RFC 5961            provides a mitigation with applicability to some situations,
            though there are also alternatives that offer cryptographic
            protection (see Section 7).  RFC 5961 recommends that in
            these synchronized states, if the SYN bit is set,
            irrespective of the sequence number, TCP endpoints MUST send
            a "challenge ACK" to the remote peer:

            <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>

         o  After sending the acknowledgment, TCP implementations MUST
            drop the unacceptable segment and stop processing further.
            Note that RFC 5961 and Errata ID 4772 [99] contain
            additional ACK throttling notes for an implementation.

         o  For implementations that do not follow RFC 5961, the
            original behavior described in RFC 793 follows in this
            paragraph.  If the SYN is in the window it is an error: send
            a reset, any outstanding RECEIVEs and SEND should receive
            "reset" responses, all segment queues should be flushed, the
            user should also receive an unsolicited general "connection
            reset" signal, enter the CLOSED state, delete the TCB, and
            return.

         o  If the SYN is not in the window, this step would not be
            reached and an ACK would have been sent in the first step
            (sequence number check).

      Fifth, check the ACK field:

      -  if the ACK bit is off, drop the segment and return

      -  if the ACK bit is on,

         o  RFC 5961 [9], Section 5 describes a potential blind data
            injection attack, and mitigation that implementations MAY
            choose to include (MAY-12).  TCP stacks that implement RFC 
            5961 MUST add an input check that the ACK value is
            acceptable only if it is in the range of ((SND.UNA -
            MAX.SND.WND) =< SEG.ACK =< SND.NXT).  All incoming segments
            whose ACK value doesn't satisfy the above condition MUST be
            discarded and an ACK sent back.  The new state variable
            MAX.SND.WND is defined as the largest window that the local
            sender has ever received from its peer (subject to window
            scaling) or may be hard-coded to a maximum permissible
            window value.  When the ACK value is acceptable, the per-
            state processing below applies:

         o  SYN-RECEIVED STATE

            +  If SND.UNA < SEG.ACK =< SND.NXT, then enter ESTABLISHED
               state and continue processing with the variables below
               set to:

                  SND.WND <- SEG.WND

                  SND.WL1 <- SEG.SEQ

                  SND.WL2 <- SEG.ACK

            +  If the segment acknowledgment is not acceptable, form a
               reset segment

                  <SEQ=SEG.ACK><CTL=RST>

            +  and send it.

         o  ESTABLISHED STATE

            +  If SND.UNA < SEG.ACK =< SND.NXT, then set SND.UNA <-
               SEG.ACK.  Any segments on the retransmission queue that
               are thereby entirely acknowledged are removed.  Users
               should receive positive acknowledgments for buffers that
               have been SENT and fully acknowledged (i.e., SEND buffer
               should be returned with "ok" response).  If the ACK is a
               duplicate (SEG.ACK =< SND.UNA), it can be ignored.  If
               the ACK acks something not yet sent (SEG.ACK > SND.NXT),
               then send an ACK, drop the segment, and return.

            +  If SND.UNA =< SEG.ACK =< SND.NXT, the send window should
               be updated.  If (SND.WL1 < SEG.SEQ or (SND.WL1 = SEG.SEQ
               and SND.WL2 =< SEG.ACK)), set SND.WND <- SEG.WND, set
               SND.WL1 <- SEG.SEQ, and set SND.WL2 <- SEG.ACK.

            +  Note that SND.WND is an offset from SND.UNA, that SND.WL1
               records the sequence number of the last segment used to
               update SND.WND, and that SND.WL2 records the
               acknowledgment number of the last segment used to update
               SND.WND.  The check here prevents using old segments to
               update the window.

         o  FIN-WAIT-1 STATE

            +  In addition to the processing for the ESTABLISHED state,
               if the FIN segment is now acknowledged, then enter FIN-
               WAIT-2 and continue processing in that state.

         o  FIN-WAIT-2 STATE

            +  In addition to the processing for the ESTABLISHED state,
               if the retransmission queue is empty, the user's CLOSE
               can be acknowledged ("ok") but do not delete the TCB.

         o  CLOSE-WAIT STATE

            +  Do the same processing as for the ESTABLISHED state.

         o  CLOSING STATE

            +  In addition to the processing for the ESTABLISHED state,
               if the ACK acknowledges our FIN, then enter the TIME-WAIT
               state; otherwise, ignore the segment.

         o  LAST-ACK STATE

            +  The only thing that can arrive in this state is an
               acknowledgment of our FIN.  If our FIN is now
               acknowledged, delete the TCB, enter the CLOSED state, and
               return.

         o  TIME-WAIT STATE

            +  The only thing that can arrive in this state is a
               retransmission of the remote FIN.  Acknowledge it, and
               restart the 2 MSL timeout.

      Sixth, check the URG bit:

      -  ESTABLISHED STATE

      -  FIN-WAIT-1 STATE

      -  FIN-WAIT-2 STATE

         o  If the URG bit is set, RCV.UP <- max(RCV.UP,SEG.UP), and
            signal the user that the remote side has urgent data if the
            urgent pointer (RCV.UP) is in advance of the data consumed.
            If the user has already been signaled (or is still in the
            "urgent mode") for this continuous sequence of urgent data,
            do not signal the user again.

      -  CLOSE-WAIT STATE

      -  CLOSING STATE

      -  LAST-ACK STATE

      -  TIME-WAIT STATE

         o  This should not occur since a FIN has been received from the
            remote side.  Ignore the URG.

      Seventh, process the segment text:

      -  ESTABLISHED STATE

      -  FIN-WAIT-1 STATE

      -  FIN-WAIT-2 STATE

         o  Once in the ESTABLISHED state, it is possible to deliver
            segment data to user RECEIVE buffers.  Data from segments
            can be moved into buffers until either the buffer is full or
            the segment is empty.  If the segment empties and carries a
            PUSH flag, then the user is informed, when the buffer is
            returned, that a PUSH has been received.

         o  When the TCP endpoint takes responsibility for delivering
            the data to the user, it must also acknowledge the receipt
            of the data.

         o  Once the TCP endpoint takes responsibility for the data, it
            advances RCV.NXT over the data accepted, and adjusts RCV.WND
            as appropriate to the current buffer availability.  The
            total of RCV.NXT and RCV.WND should not be reduced.

         o  A TCP implementation MAY send an ACK segment acknowledging
            RCV.NXT when a valid segment arrives that is in the window
            but not at the left window edge (MAY-13).

         o  Please note the window management suggestions in Section 3.8.

         o  Send an acknowledgment of the form:

            <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>

         o  This acknowledgment should be piggybacked on a segment being
            transmitted if possible without incurring undue delay.

      -  CLOSE-WAIT STATE

      -  CLOSING STATE

      -  LAST-ACK STATE

      -  TIME-WAIT STATE

         o  This should not occur since a FIN has been received from the
            remote side.  Ignore the segment text.

      Eighth, check the FIN bit:

      -  Do not process the FIN if the state is CLOSED, LISTEN, or SYN-
         SENT since the SEG.SEQ cannot be validated; drop the segment
         and return.

      -  If the FIN bit is set, signal the user "connection closing" and
         return any pending RECEIVEs with same message, advance RCV.NXT
         over the FIN, and send an acknowledgment for the FIN.  Note
         that FIN implies PUSH for any segment text not yet delivered to
         the user.

         o  SYN-RECEIVED STATE

         o  ESTABLISHED STATE

            +  Enter the CLOSE-WAIT state.

         o  FIN-WAIT-1 STATE

            +  If our FIN has been ACKed (perhaps in this segment), then
               enter TIME-WAIT, start the time-wait timer, turn off the
               other timers; otherwise, enter the CLOSING state.

         o  FIN-WAIT-2 STATE

            +  Enter the TIME-WAIT state.  Start the time-wait timer,
               turn off the other timers.

         o  CLOSE-WAIT STATE

            +  Remain in the CLOSE-WAIT state.

         o  CLOSING STATE

            +  Remain in the CLOSING state.

         o  LAST-ACK STATE

            +  Remain in the LAST-ACK state.

         o  TIME-WAIT STATE

            +  Remain in the TIME-WAIT state.  Restart the 2 MSL time-
               wait timeout.

      and return.

#### 3.10.8. Timeouts

   USER TIMEOUT

   *  For any state if the user timeout expires, flush all queues,
      signal the user "error: connection aborted due to user timeout" in
      general and for any outstanding calls, delete the TCB, enter the
      CLOSED state, and return.

   RETRANSMISSION TIMEOUT

   *  For any state if the retransmission timeout expires on a segment
      in the retransmission queue, send the segment at the front of the
      retransmission queue again, reinitialize the retransmission timer,
      and return.

   TIME-WAIT TIMEOUT

   *  If the time-wait timeout expires on a connection, delete the TCB,
      enter the CLOSED state, and return.

