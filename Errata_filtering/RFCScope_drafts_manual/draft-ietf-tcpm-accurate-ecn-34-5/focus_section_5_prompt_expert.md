Let us analyze section 5 of draft draft-ietf-tcpm-accurate-ecn-34. All references made by section 5 have also been included below.

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
# Referenced Sections from RFC 4987: TCP SYN Flooding Attacks and Common Mitigations

The following sections were referenced. Remaining sections are not included.

## A. SYN Cookies Description

   This information is taken from Bernstein's web page on SYN cookies
   [cr.yp.to ].  This is a rewriting of the technical information on that
   web page and not a full replacement.  There are other slightly
   different ways of implementing the SYN cookie concept than the exact
   means described here, although the basic idea of encoding data into
   the SYN-ACK sequence number is constant.

   A SYN cookie is an initial sequence number sent in the SYN-ACK, that
   is chosen based on the connection initiator's initial sequence
   number, MSS, a time counter, and the relevant addresses and port
   numbers.  The actual bits comprising the SYN cookie are chosen to be
   the bitwise difference (exclusive-or) between the SYN's sequence
   number and a 32 bit quantity computed so that the top five bits come
   from a 32-bit counter value modulo 32, where the counter increases
   every 64 seconds, the next 3 bits encode a usable MSS near to the one
   in the SYN, and the bottom 24 bits are a server-selected secret
   function of pair of IP addresses, the pair of port numbers, and the
   32-bit counter used for the first 5 bits.  This means of selecting an
   initial sequence number for use in the SYN-ACK complies with the rule
   that TCP sequence numbers increase slowly.

   When a connection in LISTEN receives a SYN segment, it can generate a
   SYN cookie and send it in the sequence number of a SYN-ACK, without
   allocating any other state.  If an ACK comes back, the difference
   between the acknowledged sequence number and the sequence number of
   the ACK segment can be checked against recent values of the counter
   and the secret function's output given those counter values and the
   IP addresses and port numbers in the ACK segment.  If there is a
   match, the connection can be accepted, since it is statistically very
   likely that the other side received the SYN cookie and did not simply
   guess a valid cookie value.  If there is not a match, the connection
   can be rejected under the heuristic that it is probably not in
   response to a recently sent SYN-ACK.

   With SYN cookies enabled, a host will be able to remain responsive
   even when under a SYN flooding attack.  The largest price to be paid
   for using SYN cookies is in the disabling of the window scaling
   option, which disables high performance.

   Bernstein's web page [cr.yp.to ] contains more information about the
   initial conceptualization and implementation of SYN cookies, and
   archives of emails documenting this history.  It also lists some
   false negative claims that have been made about SYN cookies, and
   discusses reducing the vulnerability of SYN cookie implementations to
   blind connection forgery by an attacker guessing valid cookies. 
   The best description of the exact SYN cookie algorithms is in a part
   of an email from Bernstein, that is archived on the web site (notice
   it does not set the top five bits from the counter modulo 32, as the
   previous description did, but instead uses 29 bits from the second
   MD5 operation and 3 bits for the index into the MSS table;
   establishing the secret values is also not discussed).  The remainder
   of this section is excerpted from Bernstein's email [cr.yp.to ]:

      Here's what an implementation would involve:

         Maintain two (constant) secret keys, sec1 and sec2.

         Maintain a (constant) sorted table of 8 common MSS values,
         msstab[8].

         Keep track of a "last overflow time".

         Maintain a counter that increases slowly over time and never
         repeats, such as "number of seconds since 1970, shifted right 6
         bits".

         When a SYN comes in from (saddr,sport) to (daddr,dport) with
         ISN x, find the largest i for which msstab[i] <= the incoming
         MSS.  Compute

            z = MD5(sec1,saddr,sport,daddr,dport,sec1)

               + x

               + (counter << 24)

               + (MD5(sec2,counter,saddr,sport,daddr,dport,sec2) % (1 <<
               24))

         and then

            y = (i << 29) + (z % (1 << 29))

         Create a TCB as usual, with y as our ISN.  Send back a SYNACK.

         Exception: _If_ we're out of memory for TCBs, set the "last
         overflow time" to the current time.  Send the SYNACK anyway,
         with all fancy options turned off.

         When an ACK comes back, follow this procedure to find a TCB:
         (1)  Look for a (saddr,sport,daddr,dport) TCB.  If it's there,
              done.

         (2)  If the "last overflow time" is earlier than a few minutes
              ago, give up.

         (3)  Figure out whether our alleged ISN makes sense.  This
              means recomputing y as above, for each of the counters
              that could have been used in the last few minutes (say,
              the last four counters), and seeing whether any of the y's
              match the ISN in the bottom 29 bits.  If none of them do,
              give up.

         (4)  Create a new TCB.  The top three bits of our ISN give a
              usable MSS.  Turn off all fancy options.


# Referenced Sections from RFC 3168: The Addition of Explicit Congestion Notification (ECN) to IP

The following sections were referenced. Remaining sections are not included.

## 11. Evaluations of ECN





### 11.1. Related Work Evaluating ECN

   This section discusses some of the related work evaluating the use of
   ECN.  The ECN Web Page [ECN ] has pointers to other papers, as well as
   to implementations of ECN.

   [Floyd94] considers the advantages and drawbacks of adding ECN to the
   TCP/IP architecture.  As shown in the simulation-based comparisons,
   one advantage of ECN is to avoid unnecessary packet drops for short
   or delay-sensitive TCP connections.  A second advantage of ECN is in
   avoiding some unnecessary retransmit timeouts in TCP.  This paper
   discusses in detail the integration of ECN into TCP's congestion
   control mechanisms.  The possible disadvantages of ECN discussed in
   the paper are that a non-compliant TCP connection could falsely
   advertise itself as ECN-capable, and that a TCP ACK packet carrying
   an ECN-Echo message could itself be dropped in the network.  The
   first of these two issues is discussed in the appendix of this
   document, and the second is addressed by the addition of the CWR flag
   in the TCP header.

   Experimental evaluations of ECN include [RFC2884,K98].  The
   conclusions of [K98] and [RFC2884] are that ECN TCP gets moderately
   better throughput than non-ECN TCP; that ECN TCP flows are fair
   towards non-ECN TCP flows; and that ECN TCP is robust with two-way
   traffic (with congestion in both directions) and with multiple
   congested gateways.  Experiments with many short web transfers show
   that, while most of the short connections have similar transfer times
   with or without ECN, a small percentage of the short connections have
   very long transfer times for the non-ECN experiments as compared to
   the ECN experiments.

### 11.2. A Discussion of the ECN nonce.

   The use of two ECT codepoints, ECT(0) and ECT(1), can provide a one-
   bit ECN nonce in packet headers [SCWA99].  The primary motivation for
   this is the desire to allow mechanisms for the data sender to verify
   that network elements are not erasing the CE codepoint, and that data
   receivers are properly reporting to the sender the receipt of packets
   with the CE codepoint set, as required by the transport protocol.
   This section discusses issues of backwards compatibility with IP ECN
   implementations in routers conformant with RFC 2481, in which only
   one ECT codepoint was defined.  We do not believe that the 
   incremental deployment of ECN implementations that understand the
   ECT(1) codepoint will cause significant operational problems.  This
   is particularly likely to be the case when the deployment of the
   ECT(1) codepoint begins with routers, before the ECT(1) codepoint
   starts to be used by end-nodes.

#### 11.2.1. The Incremental Deployment of ECT(1) in Routers.

   ECN has been an Experimental standard since January 1999, and there
   are already implementations of ECN in routers that do not understand
   the ECT(1) codepoint.  When the use of the ECT(1) codepoint is
   standardized for TCP or for other transport protocols, this could
   mean that a data sender is using the ECT(1) codepoint, but that this
   codepoint is not understood by a congested router on the path.

   If allowed by the transport protocol, a data sender would be free not
   to make use of ECT(1) at all, and to send all ECN-capable packets
   with the codepoint ECT(0).  However, if an ECN-capable sender is
   using ECT(1), and the congested router on the path did not understand
   the ECT(1) codepoint, then the router would end up marking some of
   the ECT(0) packets, and dropping some of the ECT(1) packets, as
   indications of congestion.  Since TCP is required to react to both
   marked and dropped packets, this behavior of dropping packets that
   could have been marked poses no significant threat to the network,
   and is consistent with the overall approach to ECN that allows
   routers to determine when and whether to mark packets as they see fit
   (see Section 5).

### 20.2. The Motivation for two ECT Codepoints.

   The primary motivation for the two ECT codepoints is to provide a
   one-bit ECN nonce.  The ECN nonce allows the development of
   mechanisms for the sender to probabilistically verify that network
   elements are not erasing the CE codepoint, and that data receivers
   are properly reporting to the sender the receipt of packets with the
   CE codepoint set. 
   Another possibility for senders to detect misbehaving network
   elements or receivers would be for the data sender to occasionally
   send a data packet with the CE codepoint set, to see if the receiver
   reports receiving the CE codepoint.  Of course, if these packets
   encountered congestion in the network, the router might make no
   change in the packets, because the CE codepoint would already be set.
   Thus, for packets sent with the CE codepoint set, the TCP end-nodes
   could not determine if some router intended to set the CE codepoint
   in these packets.  For this reason, sending packets with the CE
   codepoint would have to be done sparingly, and would be a less
   effective check against misbehaving network elements and receivers
   than would be the ECN nonce.

   The assignment of the fourth ECN codepoint to ECT(1) precludes the
   use of this codepoint for some other purposes.  For clarity, we
   briefly list other possible purposes here.

   One possibility might have been for the data sender to use the fourth
   ECN codepoint to indicate an alternate semantics for ECN.  However,
   this seems to us more appropriate to be signaled using a
   differentiated services codepoint in the DS field.

   A second possible use for the fourth ECN codepoint would have been to
   give the router two separate codepoints for the indication of
   congestion, CE(0) and CE(1), for mild and severe congestion
   respectively.  While this could be useful in some cases, this
   certainly does not seem a compelling requirement at this point.  If
   there was judged to be a compelling need for this, the complications
   of incremental deployment would most likely necessitate more that
   just one codepoint for this function.

   A third use that has been informally proposed for the ECN codepoint
   is for use in some forms of multicast congestion control, based on
   randomized procedures for duplicating marked packets at routers.
   Some proposed multicast packet duplication procedures are based on a
   new ECN codepoint that (1) conveys the fact that congestion occurred
   upstream of the duplication point that marked the packet with this
   codepoint and (2) can detect congestion downstream of that
   duplication point.  ECT(1) can serve this purpose because it is both
   distinct from ECT(0) and is replaced by CE when ECN marking occurs in
   response to congestion or incipient congestion.  Explanation of how
   this enhanced version of ECN would be used by multicast congestion
   control is beyond the scope of this document, as are ECN-aware
   multicast packet duplication procedures and the processing of the ECN
   field at multicast receivers in all cases (i.e., irrespective of the
   multicast packet duplication procedure(s) used). 
   The specification of IP tunnel modifications for ECN in this document
   assumes that the only change made to the outer IP header's ECN field
   between tunnel endpoints is to set the CE codepoint to indicate
   congestion.  This is not consistent with some of the proposed uses of
   ECT(1) by the multicast duplication procedures in the previous
   paragraph, and such procedures SHOULD NOT be deployed unless this
   inconsistency between multicast duplication procedures and IP tunnels
   with full ECN functionality is resolved.  Limited ECN functionality
   may be used instead, although in practice many tunnel protocols
   (including IPsec) will not work correctly if multicast traffic
   duplication occurs within the tunnel

# Referenced Sections from RFC 7413: TCP Fast Open

The following sections were referenced. Remaining sections are not included.

#### 4.2.2. TCP Fast Open

   Once the client obtains the cookie from the target server, it can
   perform subsequent TFO connections until the cookie is expired by the
   server.

   Client: Sending SYN

   To open a TFO connection, the client MUST have obtained a cookie from
   the server:

   1. Send a SYN packet.

      a. If the SYN packet does not have enough option space for the
         Fast Open option, abort TFO and fall back to the regular 3WHS.

      b. Otherwise, include the Fast Open option with the cookie of the
         server.  Include any data up to the cached server MSS or
         default 536 bytes.

   2. Advance to SYN-SENT state and update SND.NXT to include the data
      accordingly.

   To deal with network or servers dropping SYN packets with payload or
   unknown options, when the SYN timer fires, the client SHOULD
   retransmit a SYN packet without data and Fast Open options.

   Server: Receiving SYN and responding with SYN-ACK

   Upon receiving the SYN packet with Fast Open option:

   1. Initialize and reset a local FastOpened flag.  If FastOpenEnabled
      is false, go to step 5.

   2. If PendingFastOpenRequests is over the system limit, go to step 5. 
   3. If IsCookieValid() (in Section 4.1.2) returns false, go to step 5.

   4. Buffer the data and notify the application.  Set the FastOpened
      flag and increment PendingFastOpenRequests.

   5. Send the SYN-ACK packet.  The packet MAY include a Fast Open
      option.  If the FastOpened flag is set, the packet acknowledges
      the SYN and data sequence.  Otherwise, it acknowledges only the
      SYN sequence.  The server MAY include data in the SYN-ACK packet
      if the response data is readily available.  Some applications may
      favor delaying the SYN-ACK, allowing the application to process
      the request in order to produce a response, but this is left up to
      the implementation.

   6. Advance to the SYN-RCVD state.  If the FastOpened flag is set, the
      server MUST follow [RFC5681] (based on [RFC3390]) to set the
      initial congestion window for sending more data packets.

   If the SYN-ACK timer fires, the server SHOULD retransmit a SYN-ACK
   segment with neither data nor Fast Open options for compatibility
   reasons.

   A special case is simultaneous open where the SYN receiver is a
   client in SYN-SENT state.  The protocol remains the same because
   [RFC793] already supports both data in the SYN and simultaneous open.
   But the client's socket may have data available to read before it's
   connected.  This document does not cover the corresponding API
   change.

   Client: Receiving SYN-ACK

   The client SHOULD perform the following steps upon receiving the SYN-
   ACK:

   1. If the SYN-ACK has a Fast Open option, an MSS option, or both,
      update the corresponding cookie and MSS information in the cookie
      cache.

   2. Send an ACK packet.  Set acknowledgment number to RCV.NXT and
      include the data after SND.UNA if data is available.

   3. Advance to the ESTABLISHED state.

   Note there is no latency penalty if the server does not acknowledge
   the data in the original SYN packet.  The client SHOULD retransmit
   any unacknowledged data in the first ACK packet in step 2.  The data
   exchange will start after the handshake like a regular TCP
   connection. 
   If the client has timed out and retransmitted only regular SYN
   packets, it can heuristically detect paths that intentionally drop
   SYNs with the Fast Open option or data.  If the SYN-ACK acknowledges
   only the initial sequence and does not carry a Fast Open cookie
   option, presumably it is triggered by a retransmitted (regular) SYN
   and the original SYN or the corresponding SYN-ACK was lost.

   Server: Receiving ACK

   Upon receiving an ACK acknowledging the SYN sequence, the server
   decrements PendingFastOpenRequests and advances to the ESTABLISHED
   state.  No special handling is required further.

### 7.2. Impact on Congestion Control

   Although TFO does not directly change TCP's congestion control, there
   are subtle cases where it could do so.  When a SYN-ACK times out,
   regular TCP reduces the initial congestion window before sending any
   data [RFC5681].  However, in TFO, the server may have already sent up
   to an initial window of data.

   If the server serves mostly short connections, then the losses of
   SYN-ACKs are not as effective as regular TCP on reducing the
   congestion window.  This could result in an unstable network
   condition.  The connections that experience losses may attempt again
   and add more load under congestion.  A potential solution is to
   temporarily disable Fast Open if the server observes many SYN-ACK or
   data losses during the handshake across connections.  Further
   experimentation regarding the congestion control impact will be
   useful.

# Referenced Sections from RFC 8684: TCP Extensions for Multipath Operation with Multiple Addresses

The following sections were referenced. Remaining sections are not included.

## 2. Operation Overview

   This section presents a single description of common MPTCP operation,
   with reference to the protocol operation.  This is a high-level
   overview of the key functions; the full specification follows in Section 3.  Extensibility and negotiated features are not discussed
   here.  Considerable reference is made to symbolic names of MPTCP
   options throughout this section -- these are subtypes of the
   IANA-assigned MPTCP option (see Section 7), and their formats are
   defined in the detailed protocol specification provided in Section 3.

   A Multipath TCP connection provides a bidirectional bytestream
   between two hosts communicating like normal TCP and thus does not
   require any change to the applications.  However, Multipath TCP
   enables the hosts to use different paths with different IP addresses
   to exchange packets belonging to the MPTCP connection.  A Multipath
   TCP connection appears like a normal TCP connection to an
   application.  However, to the network layer, each MPTCP subflow looks
   like a regular TCP flow whose segments carry a new TCP option type.
   Multipath TCP manages the creation, removal, and utilization of these
   subflows to send data.  The number of subflows that are managed
   within a Multipath TCP connection is not fixed, and it can fluctuate
   during the lifetime of the Multipath TCP connection.

   All MPTCP operations are signaled with a TCP option -- a single
   numerical type for MPTCP, with "subtypes" for each MPTCP message.
   What follows is a summary of the purpose and rationale of these
   messages.

### 2.1. Initiating an MPTCP Connection

   This is the same signaling as for initiating a normal TCP connection,
   but the SYN, SYN/ACK, and initial ACK (and data) packets also carry
   the MP_CAPABLE option.  This option has a variable length and serves
   multiple purposes.  Firstly, it verifies whether the remote host
   supports Multipath TCP; secondly, this option allows the hosts to
   exchange some information to authenticate the establishment of
   additional subflows.  Further details are given in Section 3.1.

      Host A                                  Host B
      ------                                  ------
      MP_CAPABLE                ->
      [flags]
                                <-            MP_CAPABLE
                                              [B's key, flags]
      ACK + MP_CAPABLE (+ data) ->
      [A's key, B's key, flags, (data-level details)]

   Retransmission of the ACK + MP_CAPABLE can occur if it is not known
   if it has been received.  The following diagrams show all possible
   exchanges for the initial subflow setup to ensure this reliability.

      Host A (with data to send immediately)  Host B
      ------                                  ------
      MP_CAPABLE                ->
      [flags]
                                <-            MP_CAPABLE
                                              [B's key, flags]
      ACK + MP_CAPABLE + data   ->
      [A's key, B's key, flags, data-level details]


      Host A (with data to send later)        Host B
      ------                                  ------
      MP_CAPABLE                ->
      [flags]
                                <-            MP_CAPABLE
                                              [B's key, flags]
      ACK + MP_CAPABLE          ->
      [A's key, B's key, flags]

      ACK + MP_CAPABLE + data   ->
      [A's key, B's key, flags, data-level details]


      Host A                                  Host B (sending first)
      ------                                  ------
      MP_CAPABLE                ->
      [flags]
                                <-            MP_CAPABLE
                                              [B's key, flags]
      ACK + MP_CAPABLE          ->
      [A's key, B's key, flags]

                                <-            ACK + DSS + data
                                              [data-level details]

### 2.2. Associating a New Subflow with an Existing MPTCP Connection

   The exchange of keys in the MP_CAPABLE handshake provides material
   that can be used to authenticate the endpoints when new subflows will
   be set up.  Additional subflows begin in the same way as initiating a
   normal TCP connection, but the SYN, SYN/ACK, and ACK packets also
   carry the MP_JOIN option.

   Host A initiates a new subflow between one of its addresses and one
   of Host B's addresses.  The token -- generated from the key -- is
   used to identify which MPTCP connection it is joining, and the
   Hash-based Message Authentication Code (HMAC) is used for
   authentication.  The HMAC uses the keys exchanged in the MP_CAPABLE
   handshake and the random numbers (nonces) exchanged in these MP_JOIN
   options.  MP_JOIN also contains flags and an Address ID that can be
   used to refer to the source address without the sender needing to
   know if it has been changed by a NAT.  Further details are given in Section 3.2.

      Host A                                  Host B
      ------                                  ------
      MP_JOIN               ->
      [B's token, A's nonce,
       A's Address ID, flags]
                            <-                MP_JOIN
                                              [B's HMAC, B's nonce,
                                               B's Address ID, flags]
      ACK + MP_JOIN         ->
      [A's HMAC]

                            <-                ACK

### 2.3. Informing the Other Host about Another Potential Address

   The set of IP addresses associated to a multihomed host may change
   during the lifetime of an MPTCP connection.  MPTCP supports the
   addition and removal of addresses on a host both implicitly and
   explicitly.  If Host A has established a subflow starting at
   address/port pair IP#-A1 and wants to open a second subflow starting
   at address/port pair IP#-A2, it simply initiates the establishment of
   the subflow as explained above.  The remote host will then be
   implicitly informed about the new address.

   In some circumstances, a host may want to advertise to the remote
   host the availability of an address without establishing a new
   subflow -- for example, when a NAT prevents setup in one direction.
   In the example below, Host A informs Host B about its alternative
   IP address/port pair (IP#-A2).  Host B may later send an MP_JOIN to
   this new address.  The ADD_ADDR option contains an HMAC to
   authenticate the address as having been sent from the originator of
   the connection.  The receiver of this option echoes it back to the
   client to indicate successful receipt.  Further details are given in Section 3.4.1.

      Host A                                 Host B
      ------                                 ------
      ADD_ADDR                  ->
      [Echo-flag=0,
       IP#-A2,
       IP#-A2's Address ID,
       HMAC of IP#-A2]

                                <-          ADD_ADDR
                                            [Echo-flag=1,
                                             IP#-A2,
                                             IP#-A2's Address ID,
                                             HMAC of IP#-A2]

   There is a corresponding signal for address removal, making use of
   the Address ID that is signaled in the ADD_ADDR handshake.  Further
   details are given in Section 3.4.2.

      Host A                                 Host B
      ------                                 ------
      REMOVE_ADDR               ->
      [IP#-A2's Address ID]

### 2.4. Data Transfer Using MPTCP

   To ensure reliable, in-order delivery of data over subflows that may
   appear and disappear at any time, MPTCP uses a 64-bit Data Sequence
   Number (DSN) to number all data sent over the MPTCP connection.  Each
   subflow has its own 32-bit sequence number space, utilizing the
   regular TCP sequence number header, and an MPTCP option maps the
   subflow sequence space to the data sequence space.  In this way, data
   can be retransmitted on different subflows (mapped to the same DSN)
   in the event of failure.

   The Data Sequence Signal (DSS) carries the Data Sequence Mapping.
   The Data Sequence Mapping consists of the subflow sequence number,
   data sequence number, and length for which this mapping is valid.
   This option can also carry a connection-level acknowledgment (the
   "Data ACK") for the received DSN.

   With MPTCP, all subflows share the same receive buffer and advertise
   the same receive window.  There are two levels of acknowledgment in
   MPTCP.  Regular TCP acknowledgments are used on each subflow to
   acknowledge the reception of the segments sent over the subflow
   independently of their DSN.  In addition, there are connection-level
   acknowledgments for the data sequence space.  These acknowledgments
   track the advancement of the bytestream and slide the receive window.

   Further details are given in Section 3.3.

      Host A                                 Host B
      ------                                 ------
      DSS                       ->
      [Data Sequence Mapping]
      [Data ACK]
      [Checksum]

### 2.5. Requesting a Change in a Path's Priority

   Hosts can indicate at initial subflow setup whether they wish the
   subflow to be used as a regular or backup path -- a backup path only
   being used if there are no regular paths available.  During a
   connection, Host A can request a change in the priority of a subflow
   through the MP_PRIO signal to Host B.  Further details are given in Section 3.3.8.

      Host A                                 Host B
      ------                                 ------
      MP_PRIO                   ->

### 2.6. Closing an MPTCP Connection

   When a host wants to close an existing subflow but not the whole
   connection, it can initiate a regular TCP FIN/ACK exchange.

   When Host A wants to inform Host B that it has no more data to send,
   it signals this "Data FIN" as part of the DSS (see above).  It has
   the same semantics and behavior as a regular TCP FIN, but at the
   connection level.  Once all the data on the MPTCP connection has been
   successfully received, this message is acknowledged at the connection
   level with a Data ACK.  Further details are given in Section 3.3.3.

      Host A                                 Host B
      ------                                 ------
      DSS                       ->
      [Data FIN]
                                <-           DSS
                                             [Data ACK]

   There is an additional method of connection closure, referred to as
   "Fast Close", which is analogous to closing a single-path TCP
   connection with a RST signal.  The MP_FASTCLOSE signal is used to
   indicate to the peer that the connection will be abruptly closed and
   no data will be accepted anymore.  This can be used on an ACK (which
   ensures reliability of the signal) or a RST (which does not).  Both
   examples are shown in the following diagrams.  Further details are
   given in Section 3.5.

      Host A                                 Host B
      ------                                 ------
      ACK + MP_FASTCLOSE          ->
      [B's key]

      [RST on all other subflows] ->

                                  <-         [RST on all subflows]


      Host A                                 Host B
      ------                                 ------
      RST + MP_FASTCLOSE          ->
      [B's key] [on all subflows]

                                  <-         [RST on all subflows]

### 2.7. Notable Features

   It is worth highlighting that MPTCP's signaling has been designed
   with several key requirements in mind:

   *  To cope with NATs on the path, addresses are referred to by
      Address IDs, in case the IP packet's source address gets changed
      by a NAT.  Setting up a new TCP flow is not possible if the
      receiver of the SYN is behind a NAT; to allow subflows to be
      created when either end is behind a NAT, MPTCP uses the ADD_ADDR
      message.

   *  MPTCP falls back to ordinary TCP if MPTCP operation is not
      possible -- for example, if one host is not MPTCP capable or if a
      middlebox alters the payload.  This is discussed in Section 3.7.

   *  To address the threats identified in [RFC6181], the following
      steps are taken: keys are sent in the clear in the MP_CAPABLE
      messages; MP_JOIN messages are secured with HMAC-SHA256 ([RFC2104]
      using the algorithm in [RFC6234]) using those keys; and standard
      TCP validity checks are made on the other messages (ensuring that
      sequence numbers are in-window [RFC5961]).  Residual threats to
      MPTCP v0 were identified in [RFC7430], and those affecting the
      protocol (i.e., modifications to ADD_ADDR) have been incorporated
      in this document.  Further discussion of security can be found in Section 5.

# Referenced Sections from RFC 5562: Adding Explicit Congestion Notification (ECN) Capability to TCP's SYN/ACK Packets

The following sections were referenced. Remaining sections are not included.

## 3. Specification

   This section specifies the modification to RFC 3168 [RFC3168] to
   allow TCP SYN/ACK packets to be ECN-Capable. Section 6.1.1 of RFC 3168 [RFC3168] states that "A host MUST NOT set
   ECT on SYN or SYN-ACK packets".  In this section, we specify that a
   TCP node may respond to an initial ECN-setup SYN packet by setting
   ECT in the responding ECN-setup SYN/ACK packet, indicating to routers
   that the SYN/ACK packet is ECN-Capable.  This allows a congested
   router along the path to mark the packet instead of dropping the
   packet as an indication of congestion.

   Assume that TCP node A transmits to TCP node B an ECN-setup SYN
   packet, indicating willingness to use ECN for this connection.  As
   specified by RFC 3168 [RFC3168], if TCP node B is willing to use ECN,
   node B responds with an ECN-setup SYN-ACK packet. 





### 3.1. SYN/ACK Packets Dropped in the Network

   Figure 1 shows an interchange with the SYN/ACK packet dropped by a
   congested router.  Node B waits for a retransmission timeout, and
   then retransmits the SYN/ACK packet.

      ---------------------------------------------------------------
         TCP Node A             Router                  TCP Node B
         (initiator)                                   (responder)
         ----------             ------                  ----------

         ECN-setup SYN packet --->
                                          ECN-setup SYN packet --->

                               <--- ECN-setup SYN/ACK, possibly ECT
                                                 3-second timer set
                             SYN/ACK dropped               .
                                                           .
                                                           .
                                             3-second timer expires
                                    <--- ECN-setup SYN/ACK, not ECT
         <--- ECN-setup SYN/ACK
         Data/ACK --->
                                                      Data/ACK --->
                                   <--- Data (one to four segments)
      ---------------------------------------------------------------

            Figure 1: SYN exchange with the SYN/ACK packet dropped

   If the SYN/ACK packet is dropped in the network, the responder (node
   B) responds by waiting three seconds for the retransmission timer to
   expire [RFC2988].  If a SYN/ACK packet with the ECT codepoint is
   dropped, the responder should resend the SYN/ACK packet without the
   ECN-Capable codepoint.  (Although we are not aware of any middleboxes
   that drop SYN/ACK packets that contain an ECN-Capable codepoint in
   the IP header, we have learned to design our protocols defensively in
   this regard [RFC3360].)

   We note that if syn-cookies were used by the responder (node B) in
   the exchange in Figure 1, the responder wouldn't set a timer upon
   transmission of the SYN/ACK packet [SYN-COOK ] [RFC4987].  In this
   case, if the SYN/ACK packet was lost, the initiator (node A) would
   have to timeout and retransmit the SYN packet in order to trigger
   another SYN-ACK. 





### 3.2. SYN/ACK Packets ECN-Marked in the Network

   Figure 2 shows an interchange with the SYN/ACK packet sent as ECN-
   Capable, and ECN-marked instead of dropped at the congested router.
   This document specifies ECN+/TryOnce, which differs from the original
   proposal for ECN+ in [ECN+]; with ECN+/TryOnce, if the TCP responder
   is informed that the SYN/ACK was ECN-marked, the TCP responder
   immediately sends a SYN/ACK packet that is not ECN-Capable.  The TCP
   responder is only allowed to send data packets after the TCP
   initiator reports the receipt of a SYN/ACK packet that is not ECN-
   marked.

      ---------------------------------------------------------------
         TCP Node A             Router                  TCP Node B
         (initiator)                                   (responder)
         ----------             ------                  ----------

         ECN-setup SYN packet --->
                                         ECN-setup SYN packet --->

                                       <--- ECN-setup SYN/ACK, ECT
                                                3-second timer set
                            <--- Sets CE on SYN/ACK
         <--- ECN-setup SYN/ACK, CE

         ACK, ECN-Echo --->
                                                ACK, ECN-Echo --->
                                    Window reduced to one segment.
                                   <--- ECN-setup SYN/ACK, not ECT
         <--- ECN-setup SYN/ACK

         Data/ACK, ECT --->
                                                Data/ACK, ECT --->
                                 <--- Data, ECT (one segment only)
      ---------------------------------------------------------------

           Figure 2: SYN exchange with the SYN/ACK packet marked -
                                 ECN+/TryOnce

   If the initiator (node A) receives a SYN/ACK packet that has been
   ECN-marked by the congested router, with the CE codepoint set, the
   initiator restarts the retransmission timer.  The initiator responds
   to the ECN-marked SYN/ACK packet by setting the ECN-Echo flag in the
   TCP header of the responding ACK packet.  The initiator uses the
   standard rules in setting the cumulative acknowledgement field in the
   responding ACK packet. 
   The initiator does not advance from the "SYN-Sent" to the
   "Established" state until it receives a SYN/ACK packet that is not
   ECN-marked.

   When the responder (node B) receives the ECN-Echo packet reporting
   the Congestion Experienced indication in the SYN/ACK packet, the
   responder sets the initial congestion window to one segment, instead
   of two segments as allowed by [RFC2581], or three or four segments
   allowed by [RFC3390].  As illustrated in Figure 2, if the responder
   (node B) receives an ECN-Echo packet informing it of a Congestion
   Experienced indication on its SYN/ACK packet, the responder sends a
   SYN/ACK packet that is not ECN-Capable, in addition to setting the
   initial window to one segment.  The responder does not advance the
   send sequence number.  The responder also sets the retransmission
   timer.  The responder follows RFC 2988 [RFC2988] in setting the RTO
   (retransmission timeout).

   The TCP hosts follow the standard specification for the response to
   duplicate SYN/ACK packets (e.g., Section 3.4 of RFC 793 [RFC793]).

   We note that the mechanism in this document differs from RFC 3168   [RFC3168], which specifies that "the sending TCP MUST restart the
   retransmission timer on receiving the ECN-Echo packet when the
   congestion window is one".  RFC 3168 [RFC3168] does not allow SYN/ACK
   packets to be ECN-Capable.  RFC 3168 [RFC3168] specifies that in
   response to an ECN-Echo packet, the TCP responder also sets the CWR
   flag in the TCP header of the next data packet sent, to acknowledge
   its receipt of and reaction to the ECN-Echo flag.  In contrast, in
   response to an ECN-Echo packet acknowledging the receipt of an ECN-
   Capable SYN/ACK packet, the TCP responder doesn't set the CWR flag,
   but simply sends a SYN/ACK packet that is not ECN-Capable.  On
   receiving the non-ECN-Capable SYN/ACK packet, the TCP initiator
   clears the ECN-Echo flag on replying packets. 
      ---------------------------------------------------------------
         TCP Node A             Router                  TCP Node B
         (initiator)                                   (responder)
         ----------             ------                  ----------

         ECN-setup SYN packet --->
                                         ECN-setup SYN packet --->

                                       <--- ECN-setup SYN/ACK, ECT
                            <--- Sets CE on SYN/ACK
         <--- ECN-setup SYN/ACK, CE

         ACK, ECN-Echo --->
                                                ACK, ECN-Echo --->
                                    Window reduced to one segment.

                                    <--- ECN-setup SYN/ACK, not ECT
                                                 3-second timer set
                             SYN/ACK dropped               .
                                                           .
                                                           .
                                             3-second timer expires
                                    <--- ECN-setup SYN/ACK, not ECT
         <--- ECN-setup SYN/ACK, not ECT
         Data/ACK, ECT --->
                                                 Data/ACK, ECT --->
                                  <--- Data, ECT (one segment only)
      ---------------------------------------------------------------

         Figure 3: SYN exchange with the first SYN/ACK packet marked
             and the second SYN/ACK packet dropped - ECN+/TryOnce

   In contrast to Figure 2, Figure 3 shows an interchange where the
   first SYN/ACK packet is ECN-marked and the second SYN/ACK packet is
   dropped in the network.  As in Figure 2, the TCP responder sets a
   timer when the second SYN/ACK packet is sent.  Figure 3 shows that if
   the timer expires before the TCP responder receives an
   acknowledgement for the other end, the TCP responder resends the
   SYN/ACK packet, following the TCP standards.

### 3.3. Management Interface

   The TCP implementation using ECN-Capable SYN/ACK packets should
   include a management interface to allow the use of ECN to be turned
   off for SYN/ACK packets.  This is to deal with possible backwards
   compatibility problems such as those discussed in Appendix B . 





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





# Referenced Sections from RFC 3540: Robust Explicit Congestion Notification (ECN) Signaling with Nonces

The following sections were referenced. Remaining sections are not included.

### 6.2. Sender Behavior - Incorrect Nonce Received

   The sender's response to an incorrect nonce is a matter of policy.
   It is separate from the checking mechanism and does not need to be
   handled uniformly by senders.  Further, checking received nonce sums
   at all is optional, and may be disabled.

   If the receiver has never sent a non-zero nonce sum, the sender can
   infer that the receiver does not understand the nonce, and rate limit
   the connection, place it in a lower-priority queue, or cease setting
   ECT in outgoing segments.

   If the received nonce sum has been set in a previous acknowledgement,
   the sender might infer that a network device has interfered with
   correct ECN signaling between ECN-nonce supporting endpoints.  The
   minimum response to an incorrect nonce is the same as the response to
   a received ECE.  However, to compensate for hidden congestion
   signals, the sender might reduce the congestion window to one segment
   and cease setting ECT in outgoing segments.  An incorrect nonce sum
   is a sign of misbehavior or error between ECN-nonce supporting
   endpoints.

#### 6.2.1. Using the ECN-nonce to Protect Against Other Misbehaviors

   The ECN-nonce can provide robustness beyond checking that marked
   packets are signaled to the sender.  It also ensures that dropped
   packets cannot be concealed from the sender (because their nonces
   have been lost).  Drops could potentially be concealed by a faulty
   TCP implementation, certain attacks, or even a hypothetical TCP
   accelerator.  Such an accelerator could gamble that it can either
   successfully "fast start" to a preset bandwidth quickly, retry with
   another connection, or provide reliability at the application level.
   If robustness against these faults is also desired, then the ECN-
   nonce should not be disabled.  Instead, reducing the congestion
   window to one, or using a low-priority queue, would penalize faulty
   operation while providing continued checking.

   The ECN-nonce can also detect misbehavior in Eifel [Eifel ], a
   recently proposed mechanism for removing the retransmission ambiguity
   to improve TCP performance.  A misbehaving receiver might claim to
   have received only original transmissions to convince the sender to
   undo congestion actions.  Since retransmissions are sent without ECT,
   and thus no nonce, returning the correct nonce sum confirms that only
   original transmissions were received. 





# Referenced Sections from RFC 7713: Congestion Exposure (ConEx) Concepts, Abstract Mechanism, and Requirements

The following sections were referenced. Remaining sections are not included.

## 6. Support for Incremental Deployment

   The ConEx abstract protocol described so far is intended to support
   incremental deployment in every possible respect.  For convenience,
   the following list collects together all the features that support
   incremental deployment in the concrete ConEx specifications and
   points to further information on each:

   Packets:  The wire protocol encoding allows each packet to indicate
      whether it is using ConEx or not (see Section 4 on
      Encoding Congestion Exposure).

   Senders:  ConEx requires a modification to the source in order to
      send ConEx packet markings (see Section 5.2).  Although ConEx
      support can be indicated on a packet-by-packet basis, it is likely
      that all the packets in a flow will either consistently support
      ConEx or consistently not.  It is also likely that, if the
      implementation of a transport protocol supports ConEx, all the
      packets sent from that host using that protocol will be ConEx-
      Capable.

      The implementations of some of the transport protocols on a host
      might not support ConEx (e.g., the implementation of DNS over UDP
      might not support ConEx, while perhaps RTP over UDP and TCP will).
      Any non-upgraded transports and non-upgraded hosts will simply
      continue to send regular Not-ConEx packets as always.

      A network operator can create incentives for senders to
      voluntarily reveal ConEx information (see the item on incremental
      deployment by 'Networks' below).

   Receivers:  A ConEx source should be able to work with the regular
      receiver for the transport in question without requiring any
      ConEx-specific modifications.  This is true for modern transport
      protocols (RTCP, SCTP, etc.) and it is even true for TCP, as long
      as the receiver supports SACK, which is widely deployed anyway.
      However, it is not true for ECN feedback in TCP.  The need for
      more precise ECN feedback in TCP is not exclusive to ConEx; for
      instance, Data Centre TCP [DCTCP ] uses precise feedback to good
      effect.  Therefore, if a receiver offers precise feedback,
      [RFC7560] it will be best if ConEx uses it (see Section 5.3). 
      Alternatively, without sufficiently precise congestion feedback
      from the receiver, the source may have to conservatively send
      extra ConEx markings in order to avoid understating congestion.

   Proxies:  Although it was stated above that ConEx requires a
      modification to the source, ConEx Signals could theoretically be
      introduced by a proxy for the source as long as it can intercept
      feedback from the receiver.  Similarly, more precise feedback
      could theoretically be provided by a proxy for the receiver rather
      than modifying the receiver itself.

   Forwarding:  No modification to forwarding or queuing is needed for
      ConEx.

      However, once some ConEx is deployed, it is possible that a queue
      implementation could optionally take advantage of the ConEx
      information in packets.  For instance, it has been suggested
      [CONEX-DESTOPT ] that a queue would be more robust against flooding
      if it preferentially discarded Not-ConEx packets then Not-Marked
      ConEx packets.

      A ConEx sender re-echoes congestion whether the queues signaling
      congestion are ECN enabled or not.  Nonetheless, an operator
      relying on ConEx Signals is recommended to enable ECN in queues
      wherever possible.  This is because auditing works best if most
      congestion is indicated by ECN rather than loss (see Section 3).
      Also, monitoring rest-of-path congestion is not accurate if there
      are congested non-ECN queues upstream of the monitoring point
      (Section 5.4.2).

   Networks:  If a subset of traffic sources (or proxies) use ConEx
      Signals to reveal congestion in the internetwork layer, a network
      operator can choose (or not) to use this information for traffic
      management.  As long as the end-to-end ConEx Signals are present,
      each network can unilaterally choose to use them -- independently
      of whether other networks do.

      ConEx marked packets may safely traverse a network that ignores
      them.  ConEx Signals are defined to remain unchanged once set by
      the sender, but some encodings may allow changes in transit (e.g.,
      by proxies).  In no circumstances will a network node change
      ConEx-Capable packets to Not-ConEx (network-layer encoding
      requirement I in Section 3.3).  If necessary, endpoints should be
      able to detect if a network is removing ConEx Signals (network-
      layer encoding requirement H in Section 3.3). 
      An operator can deploy policy devices (Section 5.4) wherever
      traffic enters its network in order to monitor the downstream
      congestion that incoming traffic contributes to and control it if
      necessary.  A network operator can create incentives for the
      developers of sending applications and transports to voluntarily
      reveal ConEx information.  Without ConEx information, a network
      operator tends to have to limit the bit-rate or volume from a site
      more than is necessary, just in case it might congest others.
      With ConEx information, the operator can solely limit congestion-
      causing traffic and otherwise allow complete freedom.  This
      greater freedom acts as an inducement for the source to volunteer
      ConEx information.  An operator may also monitor whether a source
      transport has sent ConEx packets and treat the same transport with
      greater suspicion (e.g., a more stringent rate limit) whenever it
      selectively sends packets without ConEx support.  See [RFC6789]
      for further discussion of deployment incentives for networks and
      references to scenarios where some networks use ConEx-based policy
      devices and others don't.

      An operator can deploy audit devices (Section 5.5) unilaterally
      within its own network to verify that traffic sources are not
      understating ConEx information.  From the viewpoint of one network
      operator (say N_a), it only cares that the level of ConEx
      signaling is sufficient to cover congestion in its own network.
      If traffic continues into a congested downstream network (say
      N_b), it is of no concern to the first network (N_a) if the end-
      to-end ConEx signaling is insufficient to cover the congestion in
      N_b as well.  This is N_b's concern, and N_b can both detect such
      anomalous traffic and deal with it using ConEx-based audit devices
      itself.

# Referenced Sections from RFC 5925: The TCP Authentication Option

The following sections were referenced. Remaining sections are not included.

## 7. TCP-AO Interaction with TCP

   The following is a description of how TCP-AO affects various TCP
   states, segments, events, and interfaces.  This description is
   intended to augment the description of TCP as provided in RFC 793,
   and its presentation mirrors that of RFC 793 as a result [RFC793]. 





### 7.1. TCP User Interface

   The TCP user interface supports active and passive OPEN, SEND,
   RECEIVE, CLOSE, STATUS, and ABORT commands.  TCP-AO does not alter
   this interface as it applies to TCP, but some commands or command
   sequences of the interface need to be modified to support TCP-AO.
   TCP-AO does not specify the details of how this is achieved.

   TCP-AO requires that the TCP user interface be extended to allow the
   MKTs to be configured, as well as to allow an ongoing connection to
   manage which MKTs are active.  The MKTs need to be configured prior
   to connection establishment, and the set of MKTs may change during a
   connection:

   >> TCP OPEN, or the sequence of commands that configure a connection
   to be in the active or passive OPEN state, MUST be augmented so that
   an MKT can be configured.

   >> A TCP-AO implementation MUST allow the set of MKTs for ongoing TCP
   connections (i.e., not in the CLOSED state) to be modified.

   The MKTs associated with a connection need to be available for
   confirmation; this includes the ability to read the MKTs:

   >> TCP STATUS SHOULD be augmented to allow the MKTs of a current or
   pending connection to be read (for confirmation).

   Senders may need to be able to determine when the outgoing MKT
   changes (KeyID) or when a new preferred MKT (RNextKeyID) is
   indicated; these changes immediately affect all subsequent outgoing
   segments:

   >> TCP SEND, or a sequence of commands resulting in a SEND, MUST be
   augmented so that the preferred outgoing MKT (current_key) and/or the
   preferred incoming MKT (rnext_key) of a connection can be indicated.

   It may be useful to change the outgoing active MKT (current_key) even
   when no data is being sent, which can be achieved by sending a zero-
   length buffer or by using a non-send interface (e.g., socket options
   in Unix), depending on the implementation.

   It is also useful to indicate recent segment KeyID and RNextKeyID
   values received; although there could be a number of such values,
   they are not expected to change quickly, so any recent sample should
   be sufficient:
   >> TCP RECEIVE, or the sequence of commands resulting in a RECEIVE,
   MUST be augmented so that the KeyID and RNextKeyID of a recently
   received segment is available to the user out of band (e.g., as an
   additional parameter to RECEIVE or via a STATUS call).

### 7.2. TCP States and Transitions

   TCP includes the states LISTEN, SYN-SENT, SYN-RECEIVED, ESTABLISHED,
   FIN-WAIT-1, FIN-WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT, and
   CLOSED.

   >> An MKT MAY be associated with any TCP state.

### 7.3. TCP Segments

   TCP includes control (at least one of SYN, FIN, RST flags set) and
   data (none of SYN, FIN, or RST flags set) segments.  Note that some
   control segments can include data (e.g., SYN).

   >> All TCP segments MUST be checked against the set of MKTs for
   matching TCP connection identifiers.

   >> TCP segments whose TCP-AO does not validate MUST be silently
   discarded.

   >> A TCP-AO implementation MUST allow for configuration of the
   behavior of segments with TCP-AO but that do not match an MKT.  The
   initial default of this configuration SHOULD be to silently accept
   such connections.  If this is not the desired case, an MKT can be
   included to match such connections, or the connection can indicate
   that TCP-AO is required.  Alternately, the configuration can be
   changed to discard segments with the AO option not matching an MKT.

   >> Silent discard events SHOULD be signaled to the user as a warning,
   and silent accept events MAY be signaled to the user as a warning.
   Both warnings, if available, MUST be accessible via the STATUS
   interface.  Either signal MAY be asynchronous, but if so, they MUST
   be rate-limited.  Either signal MAY be logged; logging SHOULD allow
   rate-limiting as well.

   All TCP-AO processing occurs between the interface of TCP and IP; for
   incoming segments, this occurs after validation of the TCP checksum.
   For outgoing segments, this occurs before computation of the TCP
   checksum. 
   Note that use of TCP-AO on a connection is not negotiated within TCP.
   It is the responsibility of the receiver to determine when TCP-AO is
   required via other means (e.g., out of band, manually or with a key
   management protocol) and to enforce that requirement.

### 7.4. Sending TCP Segments

   The following procedure describes the modifications to TCP to support
   inserting TCP-AO when a segment departs.

   >> Note that TCP-AO MUST be the last TCP option processed on outgoing
   segments, because its MAC calculation may include the values of other
   TCP options.

   1. Find the per-connection parameters for the segment:

       a. If the segment is a SYN, then this is the first segment of a
          new connection.  Find the matching MKT for this segment based
          on the segment's socket pair.

          i. If there is no matching MKT, omit TCP-AO.  Proceed with
             transmitting the segment.

         ii. If there is a matching MKT, then set the per-connection
             parameters as needed (see Section 4).  Proceed with the
             step 2.

       b. If the segment is not a SYN, then determine whether TCP-AO is
          being used for the connection and use the MKT as indicated by
          the current_key value from the per-connection parameters (see Section 4) and proceed with the step 2.

   2. Using the per-connection parameters:

       a. Augment the TCP header with TCP-AO, inserting the appropriate
          Length and KeyID based on the MKT indicated by current_key
          (using the current_key MKT's SendID as the TCP-AO KeyID).
          Update the TCP header length accordingly.

       b. Determine SND.SNE as described in Section 6.2.

       c. Determine the appropriate traffic key, i.e., as pointed to by
          the current_key (as noted in Section 6.1, and as probably
          cached in the TCB).  That is, use the send_SYN_traffic_key for
          SYN segments and the send_other_traffic_key for other
          segments. 
       d. Determine the RNextKeyID as indicated by the rnext_key
          pointer, and insert it in the TCP-AO RNextKeyID field (using
          the rnext_key MKT's RecvID as the TCP-AO KeyID) (as noted in Section 6.1).

       e. Compute the MAC using the MKT (and cached traffic key) and
          data from the segment as specified in Section 5.1.

       f. Insert the MAC in the TCP-AO MAC field.

       g. Proceed with transmitting the segment.

### 7.5. Receiving TCP Segments

   The following procedure describes the modifications to TCP to support
   TCP-AO when a segment arrives.

   >> Note that TCP-AO MUST be the first TCP option processed on
   incoming segments, because its MAC calculation may include the values
   of other TCP options that could change during TCP option processing.
   This also protects the behavior of all other TCP options from the
   impact of spoofed segments or modified header information.

   >> Note that TCP-AO checks MUST be performed for all incoming SYNs to
   avoid accepting SYNs lacking TCP-AO where required.  Other segments
   can cache whether TCP-AO is needed in the TCB.

   1. Find the per-connection parameters for the segment:

       a. If the segment is a SYN, then this is the first segment of a
          new connection.  Find the matching MKT for this segment, using
          the segment's socket pair and its TCP-AO KeyID, matched
          against the MKT's TCP connection identifier and the MKT's
          RecvID.

          i. If there is no matching MKT, remove TCP-AO from the
             segment.  Proceed with further TCP handling of the segment.

             NOTE: this presumes that connections that do not match any
             MKT should be silently accepted, as noted in Section 7.3.

         ii. If there is a matching MKT, then set the per-connection
             parameters as needed (see Section 4).  Proceed with step 2. 
   2. Using the per-connection parameters:

       a. Check that the segment's TCP-AO Length matches the length
          indicated by the MKT.

          i. If the lengths differ, silently discard the segment.  Log
             and/or signal the event as indicated in Section 7.3.

       b. Determine the segment's RCV.SNE as described in Section 6.2.

       c. Determine the segment's traffic key from the MKT as described
          in Section 5.1 (and as likely cached in the TCB).  That is,
          use the receive_SYN_traffic_key for SYN segments and the
          receive_other_traffic_key for other segments.

       d. Compute the segment's MAC using the MKT (and its derived
          traffic key) and portions of the segment as indicated in Section 5.1.

          i. If the computed MAC differs from the TCP-AO MAC field
             value, silently discard the segment.  Log and/or signal the
             event as indicated in Section 7.3.

       e. Compare the received RNextKeyID value to the currently active
          outgoing KeyID value (current_key MKT's SendID).

          i. If they match, no further action is required.

         ii. If they differ, determine whether the RNextKeyID MKT is
             ready.

             1. If the MKT corresponding to the segment's socket pair
                and RNextKeyID is not available, no action is required
                (RNextKeyID of a received segment needs to match the
                MKT's SendID).

             2. If the matching MKT corresponding to the segment's
                socket pair and RNextKeyID is available:

                a. Set current_key to the RNextKeyID MKT.

       f. Proceed with TCP processing of the segment.

   It is suggested that TCP-AO implementations validate a segment's
   Length field before computing a MAC to reduce the overhead incurred
   by spoofed segments with invalid TCP-AO fields. 
   Additional reductions in MAC validation overhead can be supported in
   the MAC algorithms, e.g., by using a computation algorithm that
   prepends a fixed value to the computed portion and a corresponding
   validation algorithm that verifies the fixed value before investing
   in the computed portion.  Such optimizations would be contained in
   the MAC algorithm specification, and thus are not specified in TCP-AO
   explicitly.  Note that the KeyID cannot be used for connection
   validation per se, because it is not assumed random.

### 7.6. Impact on TCP Header Size

   TCP-AO, using the initially required 96-bit MACs, uses a total of 16
   bytes of TCP header space [RFC5926].  TCP-AO is thus 2 bytes smaller
   than the TCP MD5 option (18 bytes).

   Note that the TCP option space is most critical in SYN segments,
   because flags in those segments could potentially increase the option
   space area in other segments.  Because TCP ignores unknown segments,
   however, it is not possible to extend the option space of SYNs
   without breaking backward compatibility.

   TCP's 4-bit data offset requires that the options end 60 bytes (15
   32-bit words) after the header begins, including the 20-byte header.
   This leaves 40 bytes for options, of which 15 are expected in current
   implementations (listed below), leaving at most 25 for other uses.
   TCP-AO consumes 16 bytes, leaving 9 bytes for additional SYN options
   (depending on implementation dependant alignment padding, which could
   consume another 2 bytes at most).

   o  SACK permitted (2 bytes) [RFC2018][RFC3517]

   o  Timestamps (10 bytes) [RFC1323]

   o  Window scale (3 bytes) [RFC1323]

   After a SYN, the following options are expected in current
   implementations of TCP:

   o  SACK (10bytes) [RFC2018][RFC3517] (18 bytes if D-SACK [RFC2883])

   o  Timestamps (10 bytes) [RFC1323]

   TCP-AO continues to consume 16 bytes in non-SYN segments, leaving a
   total of 24 bytes for other options, of which the timestamp consumes
   10.  This leaves 14 bytes, of which 10 are used for a single SACK
   block.  When two SACK blocks are used, such as to handle D-SACK, a
   smaller TCP-AO MAC would be required to make room for the additional
   SACK block (i.e., to leave 18 bytes for the D-SACK variant of the 
   SACK option) [RFC2883].  Note that D-SACK is not supportable in TCP
   MD5 in the presence of timestamps, because TCP MD5's MAC length is
   fixed and too large to leave sufficient option space.

   Although TCP option space is limited, we believe TCP-AO is consistent
   with the desire to authenticate TCP at the connection level for
   similar uses as were intended by TCP MD5.

### 7.7. Connectionless Resets

   TCP-AO allows TCP resets (RSTs) to be exchanged provided both sides
   have established valid connection state.  After such state is
   established, if one side reboots, TCP-AO prevents TCP's RST mechanism
   from clearing out old state on the side that did not reboot.  This
   happens because the rebooting side has lost its connection state, and
   thus its traffic keys.

   It is important that implementations are capable of detecting
   excesses of TCP connections in such a configuration and can clear
   them out if needed to protect its memory usage [Ba10].  To protect
   against such state from accumulating and not being cleared out, a
   number of recommendations are made:

   >> Connections using TCP-AO SHOULD also use TCP keepalives [RFC1122].

   The use of TCP keepalives ensures that connections whose keys are
   lost are terminated after a finite time; a similar effect can be
   achieved at the application layer, e.g., with BGP keepalives
   [RFC4271].  Either kind of keepalive helps ensure the TCP state is
   cleared out in such a case; the alternative, of allowing
   unauthenticated RSTs to be received, would allow one of the primary
   vulnerabilities that TCP-AO is intended to prevent.

   Keepalives ensure that connections are dropped across reboots, but
   this can have a detrimental effect on some protocols.  Specifically,
   BGP reacts poorly to such connection drops, even if caused by the use
   of BGP keepalives; "graceful restart" was introduced to address this
   effect [RFC4724], and extended to support BGP with MPLS [RFC4781].
   As a result:

   >> BGP connections SHOULD require support for graceful restart when
   using TCP-AO. 
   We recognize that support for graceful restart is not always
   feasible.  As a result:

   >> When BGP without graceful restart is used with TCP-AO, both sides
   of the connection SHOULD save traffic keys in storage that persists
   across reboots and restore them after a reboot, and SHOULD limit any
   performance impacts that result from this storage/restoration.

### 7.8. ICMP Handling

   TCP can be attacked both in band, using TCP segments, or out of band
   using ICMP.  ICMP packets cannot be protected using TCP-AO
   mechanisms; however, in this way, both TCP-AO and IPsec do not
   directly solve the need for protected ICMP signaling.  TCP-AO does
   make specific recommendations on how to handle certain ICMPs, beyond
   what IPsec requires, and these are made possible because TCP-AO
   operates inside the context of a TCP connection.

   IPsec makes recommendations regarding dropping ICMPs in certain
   contexts or requiring that they are endpoint authenticated in others
   [RFC4301].  There are other mechanisms proposed to reduce the impact
   of ICMP attacks by further validating ICMP contents and changing the
   effect of some messages based on TCP state, but these do not provide
   the level of authentication for ICMP that TCP-AO provides for TCP
   [Go10].  As a result, we recommend a conservative approach to
   accepting ICMP messages as summarized in [Go10]:

   >> A TCP-AO implementation MUST default to ignore incoming ICMPv4
   messages of Type 3 (destination unreachable), Codes 2-4 (protocol
   unreachable, port unreachable, and fragmentation needed -- 'hard
   errors'), and ICMPv6 Type 1 (destination unreachable), Code 1
   (administratively prohibited) and Code 4 (port unreachable) intended
   for connections in synchronized states (ESTABLISHED, FIN-WAIT-1, FIN-
   WAIT-2, CLOSE-WAIT, CLOSING, LAST-ACK, TIME-WAIT) that match MKTs.

   >> A TCP-AO implementation SHOULD allow whether such ICMPs are
   ignored to be configured on a per-connection basis.

   >> A TCP-AO implementation SHOULD implement measures to protect ICMP
   "packet too big" messages, some examples of which are discussed in
   [Go10].

   >> An implementation SHOULD allow ignored ICMPs to be logged.

   This control affects only ICMPs that currently require 'hard errors',
   which would abort the TCP connection [RFC1122].  This recommendation
   is intended to be similar to how IPsec would handle those messages,
   with an additional default assumed [RFC4301]. 





# Referenced Sections from RFC 6928: Increasing TCP's Initial Window

The following sections were referenced. Remaining sections are not included.

## 2. TCP Modification

   This document proposes an increase in the permitted upper bound for
   TCP's initial window (IW) to 10 segments, depending on the maximum
   segment size (MSS).  This increase is optional: a TCP MAY start with
   an initial window that is smaller than 10 segments.

   More precisely, the upper bound for the initial window will be

         min (10*MSS, max (2*MSS, 14600))                            (1)

   This upper bound for the initial window size represents a change from RFC 3390 [RFC3390], which specified that the congestion window be
   initialized between 2 and 4 segments, depending on the MSS.

   This change applies to the initial window of the connection in the
   first round-trip time (RTT) of data transmission during or following
   the TCP three-way handshake.  Neither the SYN/ACK nor its ACK in the
   three-way handshake should increase the initial window size.

   Note that all the test results described in this document were based
   on the regular Ethernet MTU of 1500 bytes.  Future study of the
   effect of a different MTU may be needed to fully validate (1) above.

   Furthermore, RFC 3390 states (and RFC 5681 [RFC5681] has similar
   text):

      If the SYN or SYN/ACK is lost, the initial window used by a sender
      after a correctly transmitted SYN MUST be one segment consisting
      of MSS bytes.

   The proposed change to reduce the default retransmission timeout
   (RTO) to 1 second [RFC6298] increases the chance for spurious SYN or
   SYN/ACK retransmission, thus unnecessarily penalizing connections
   with RTT > 1 second if their initial window is reduced to 1 segment.
   For this reason, it is RECOMMENDED that implementations refrain from
   resetting the initial window to 1 segment, unless there have been
   more than one SYN or SYN/ACK retransmissions or true loss detection
   has been made. 
   TCP implementations use slow start in as many as three different
   ways: (1) to start a new connection (the initial window); (2) to
   restart transmission after a long idle period (the restart window);
   and (3) to restart transmission after a retransmit timeout (the loss
   window).  The change specified in this document affects the value of
   the initial window.  Optionally, a TCP MAY set the restart window to
   the minimum of the value used for the initial window and the current
   value of cwnd (in other words, using a larger value for the restart
   window should never increase the size of cwnd).  These changes do NOT
   change the loss window, which must remain 1 segment of MSS bytes (to
   permit the lowest possible window size in the case of severe
   congestion).

   Furthermore, to limit any negative effect that a larger initial
   window may have on links with limited bandwidth or buffer space,
   implementations SHOULD fall back to RFC 3390 for the restart window
   (RW) if any packet loss is detected during either the initial window
   or a restart window, and more than 4 KB of data is sent.
   Implementations must also follow RFC 6298 [RFC6298] in order to avoid
   spurious RTO as described in Section 9.

# Referenced Sections from RFC 8257: Data Center TCP (DCTCP): TCP Congestion Control for Data Centers

The following sections were referenced. Remaining sections are not included.

## 3. DCTCP Algorithm

   There are three components involved in the DCTCP algorithm:

   o  The switches (or other intermediate devices in the network) detect
      congestion and set the Congestion Encountered (CE) codepoint in
      the IP header.

   o  The receiver echoes the congestion information back to the sender,
      using the ECN-Echo (ECE) flag in the TCP header.

   o  The sender computes a congestion estimate and reacts by reducing
      the TCP congestion window (cwnd) accordingly.

### 3.1. Marking Congestion on the L3 Switches and Routers

   The Layer 3 (L3) switches and routers in a data-center fabric
   indicate congestion to the end nodes by setting the CE codepoint in
   the IP header as specified in Section 5 of [RFC3168].  For example,
   the switches may be configured with a congestion threshold.  When a
   packet arrives at a switch and its queue length is greater than the
   congestion threshold, the switch sets the CE codepoint in the packet.
   For example, Section 3.4 of [DCTCP10] suggests threshold marking with
   a threshold of K > (RTT * C)/7, where C is the link rate in packets
   per second.  In typical deployments, the marking threshold is set to
   be a small value to maintain a short average queueing delay.
   However, the actual algorithm for marking congestion is an
   implementation detail of the switch and will generally not be known
   to the sender and receiver.  Therefore, the sender and receiver
   should not assume that a particular marking algorithm is implemented
   by the switching fabric.

### 3.2. Echoing Congestion Information on the Receiver

   According to Section 6.1.3 of [RFC3168], the receiver sets the ECE
   flag if any of the packets being acknowledged had the CE codepoint
   set.  The receiver then continues to set the ECE flag until it
   receives a packet with the Congestion Window Reduced (CWR) flag set.
   However, the DCTCP algorithm requires more-detailed congestion
   information.  In particular, the sender must be able to determine the
   number of bytes sent that encountered congestion.  Thus, the scheme
   described in [RFC3168] does not suffice.

   One possible solution is to ACK every packet and set the ECE flag in
   the ACK if and only if the CE codepoint was set in the packet being
   acknowledged.  However, this prevents the use of delayed ACKs, which
   are an important performance optimization in data centers.  If the
   delayed ACK frequency is n, then an ACK is generated every n packets. 
   The typical value of n is 2, but it could be affected by ACK
   throttling or packet-coalescing techniques designed to improve
   performance.

   Instead, DCTCP introduces a new Boolean TCP state variable, DCTCP
   Congestion Encountered (DCTCP.CE), which is initialized to false and
   stored in the Transmission Control Block (TCB).  When sending an ACK,
   the ECE flag MUST be set if and only if DCTCP.CE is true.  When
   receiving packets, the CE codepoint MUST be processed as follows:

   1.  If the CE codepoint is set and DCTCP.CE is false, set DCTCP.CE to
       true and send an immediate ACK.

   2.  If the CE codepoint is not set and DCTCP.CE is true, set DCTCP.CE
       to false and send an immediate ACK.

   3.  Otherwise, ignore the CE codepoint.

   Since the immediate ACK reflects the new DCTCP.CE state, it may
   acknowledge any previously unacknowledged packets in the old state.
   This can lead to an incorrect rate computation at the sender per Section 3.3.  To avoid this, an implementation MAY choose to send two
   ACKs: one for previously unacknowledged packets and another
   acknowledging the most recently received packet.

   Receiver handling of the CWR bit is also per [RFC3168] (including
   [Err3639]).  That is, on receipt of a segment with both the CE and
   CWR bits set, CWR is processed first and then CE is processed.

                             Send immediate
                             ACK with ECE=0
                 .-----.     .--------------.     .-----.
    Send 1 ACK  /      v     v              |     |      \
     for every |     .------------.    .------------.     | Send 1 ACK
     n packets |     | DCTCP.CE=0 |    | DCTCP.CE=1 |     | for every
    with ECE=0 |     '------------'    '------------'     | n packets
                \      |     |              ^     ^      /  with ECE=1
                 '-----'     '--------------'     '-----'
                              Send immediate
                              ACK with ECE=1


                  Figure 1: ACK Generation State Machine 





### 3.3. Processing Echoed Congestion Indications on the Sender

   The sender estimates the fraction of bytes sent that encountered
   congestion.  The current estimate is stored in a new TCP state
   variable, DCTCP.Alpha, which is initialized to 1 and SHOULD be
   updated as follows:

      DCTCP.Alpha = DCTCP.Alpha * (1 - g) + g * M

   where:

   o  g is the estimation gain, a real number between 0 and 1.  The
      selection of g is left to the implementation.  See Section 4 for
      further considerations.

   o  M is the fraction of bytes sent that encountered congestion during
      the previous observation window, where the observation window is
      chosen to be approximately the Round-Trip Time (RTT).  In
      particular, an observation window ends when all bytes in flight at
      the beginning of the window have been acknowledged.

   In order to update DCTCP.Alpha, the TCP state variables defined in
   [RFC0793] are used, and three additional TCP state variables are
   introduced:

   o  DCTCP.WindowEnd: the TCP sequence number threshold when one
      observation window ends and another is to begin; initialized to
      SND.UNA.

   o  DCTCP.BytesAcked: the number of sent bytes acknowledged during the
      current observation window; initialized to 0.

   o  DCTCP.BytesMarked: the number of bytes sent during the current
      observation window that encountered congestion; initialized to 0.

   The congestion estimator on the sender MUST process acceptable ACKs
   as follows:

   1.  Compute the bytes acknowledged (TCP Selective Acknowledgment
       (SACK) options [RFC2018] are ignored for this computation):

          BytesAcked = SEG.ACK - SND.UNA

   2.  Update the bytes sent:

          DCTCP.BytesAcked += BytesAcked 
   3.  If the ECE flag is set, update the bytes marked:

          DCTCP.BytesMarked += BytesAcked

   4.  If the acknowledgment number is less than or equal to
       DCTCP.WindowEnd, stop processing.  Otherwise, the end of the
       observation window has been reached, so proceed to update the
       congestion estimate as follows:

   5.  Compute the congestion level for the current observation window:

          M = DCTCP.BytesMarked / DCTCP.BytesAcked

   6.  Update the congestion estimate:

          DCTCP.Alpha = DCTCP.Alpha * (1 - g) + g * M

   7.  Determine the end of the next observation window:

          DCTCP.WindowEnd = SND.NXT

   8.  Reset the byte counters:

          DCTCP.BytesAcked = DCTCP.BytesMarked = 0

   9.  Rather than always halving the congestion window as described in
       [RFC3168], the sender SHOULD update cwnd as follows:

          cwnd = cwnd * (1 - DCTCP.Alpha / 2)

   Just as specified in [RFC3168], DCTCP does not react to congestion
   indications more than once for every window of data.  The setting of
   the CWR bit is also as per [RFC3168].  This is required for
   interoperation with classic ECN receivers due to potential
   misconfigurations.

### 3.4. Handling of Congestion Window Growth

   A DCTCP sender grows its congestion window in the same way as
   conventional TCP.  Slow start and congestion avoidance algorithms are
   handled as specified in [RFC5681].

### 3.5. Handling of Packet Loss

   A DCTCP sender MUST react to loss episodes in the same way as
   conventional TCP, including fast retransmit and fast recovery
   algorithms, as specified in [RFC5681].  For cases where the packet
   loss is inferred and not explicitly signaled by ECN, the cwnd and 
   other state variables like ssthresh MUST be changed in the same way
   that a conventional TCP would have changed them.  As with ECN, a
   DCTCP sender will only reduce the cwnd once per window of data across
   all loss signals.  Just as specified in [RFC5681], upon a timeout,
   the cwnd MUST be set to no more than the loss window (1 full-sized
   segment), regardless of previous cwnd reductions in a given window of
   data.

### 3.6. Handling of SYN, SYN-ACK, and RST Packets

   If SYN, SYN-ACK, and RST packets for DCTCP connections have the ECN-
   Capable Transport (ECT) codepoint set in the IP header, they will
   receive the same treatment as other DCTCP packets when forwarded by a
   switching fabric under load.  Lack of ECT in these packets can result
   in a higher drop rate, depending on the switching fabric
   configuration.  Hence, for DCTCP connections, the sender SHOULD set
   ECT for SYN, SYN-ACK, and RST packets.  A DCTCP receiver ignores CE
   codepoints set on any SYN, SYN-ACK, or RST packets.

# Referenced Sections from RFC 9331: The Explicit Congestion Notification (ECN) Protocol for Low Latency, Low Loss, and Scalable Throughput (L4S)

The following sections were referenced. Remaining sections are not included.

# Referenced Sections from RFC 9330: Low Latency, Low Loss, and Scalable Throughput (L4S) Internet Service: Architecture

The following sections were referenced. Remaining sections are not included.

## 4. L4S Architecture Components

   The L4S architecture is composed of the elements in the following
   three subsections.

### 4.1. Protocol Mechanisms

   The L4S architecture involves: a) unassignment of the previous use of
   the identifier; b) reassignment of the same identifier; and c)
   optional further identifiers:

   a.  An essential aspect of a Scalable congestion control is the use
       of explicit congestion signals.  Classic ECN [RFC3168] requires
       an ECN signal to be treated as equivalent to drop, both when it
       is generated in the network and when it is responded to by hosts.
       L4S needs networks and hosts to support a more fine-grained
       meaning for each ECN signal that is less severe than a drop, so
       that the L4S signals:

       *  can be much more frequent and

       *  can be signalled immediately, without the significant delay
          required to smooth out fluctuations in the queue.

       To enable L4S, the Standards Track Classic ECN spec [RFC3168] has
       had to be updated to allow L4S packets to depart from the
       'equivalent-to-drop' constraint.  [RFC8311] is a Standards Track
       update to relax specific requirements in [RFC3168] (and certain
       other Standards Track RFCs), which clears the way for the
       experimental changes proposed for L4S.  Also, the ECT(1)
       codepoint was previously assigned as the experimental ECN nonce
       [RFC3540], which [RFC8311] recategorizes as historic to make the
       codepoint available again.

   b.  [RFC9331] specifies that ECT(1) is used as the identifier to
       classify L4S packets into a separate treatment from Classic
       packets.  This satisfies the requirement for identifying an
       alternative ECN treatment in [RFC4774].

       The CE codepoint is used to indicate Congestion Experienced by
       both L4S and Classic treatments.  This raises the concern that a
       Classic AQM earlier on the path might have marked some ECT(0)
       packets as CE.  Then, these packets will be erroneously
       classified into the L4S queue.  Appendix B of [RFC9331] explains
       why five unlikely eventualities all have to coincide for this to
       have any detrimental effect, which even then would only involve a
       vanishingly small likelihood of a spurious retransmission.

   c.  A network operator might wish to include certain unresponsive,
       non-L4S traffic in the L4S queue if it is deemed to be paced
       smoothly enough and at a low enough rate not to build a queue,
       for instance, VoIP, low rate datagrams to sync online games,
       relatively low rate application-limited traffic, DNS, Lightweight
       Directory Access Protocol (LDAP), etc.  This traffic would need
       to be tagged with specific identifiers, e.g., a low-latency
       Diffserv codepoint such as Expedited Forwarding (EF) [RFC3246],
       Non-Queue-Building (NQB) [NQB-PHB ], or operator-specific
       identifiers.

### 4.2. Network Components

   The L4S architecture aims to provide low latency without the _need_
   for per-flow operations in network components.  Nonetheless, the
   architecture does not preclude per-flow solutions.  The following
   bullets describe the known arrangements: a) the DualQ Coupled AQM
   with an L4S AQM in one queue coupled from a Classic AQM in the other;
   b) per-flow queues with an instance of a Classic and an L4S AQM in
   each queue; and c) Dual queues with per-flow AQMs but no per-flow
   queues:

   a.  The Dual-Queue Coupled AQM (illustrated in Figure 1) achieves the
       'semi-permeable' membrane property mentioned earlier as follows:

       *  Latency isolation: Two separate queues are used to isolate L4S
          queuing delay from the larger queue that Classic traffic needs
          to maintain full utilization.

       *  Bandwidth pooling: The two queues act as if they are a single
          pool of bandwidth in which flows of either type get roughly
          equal throughput without the scheduler needing to identify any
          flows.  This is achieved by having an AQM in each queue, but
          the Classic AQM provides a congestion signal to both queues in
          a manner that ensures a consistent response from the two
          classes of congestion control.  Specifically, the Classic AQM
          generates a drop/mark probability based on congestion in its
          own queue, which it uses both to drop/mark packets in its own
          queue and to affect the marking probability in the L4S queue.
          The strength of the coupling of the congestion signalling
          between the two queues is enough to make the L4S flows slow
          down to leave the right amount of capacity for the Classic
          flows (as they would if they were the same type of traffic
          sharing the same queue).

       Then, the scheduler can serve the L4S queue with priority
       (denoted by the '1' on the higher priority input), because the
       L4S traffic isn't offering up enough traffic to use all the
       priority that it is given.  Therefore:

       *  for latency isolation on short timescales (sub-round-trip),
          the prioritization of the L4S queue protects its low latency
          by allowing bursts to dissipate quickly;

       *  but for bandwidth pooling on longer timescales (round-trip and
          longer), the Classic queue creates an equal and opposite
          pressure against the L4S traffic to ensure that neither has
          priority when it comes to bandwidth -- the tension between
          prioritizing L4S and coupling the marking from the Classic AQM
          results in approximate per-flow fairness.

       To protect against the prioritization of persistent L4S traffic
       deadlocking the Classic queue for a while in some
       implementations, it is advisable for the priority to be
       conditional, not strict (see Appendix A  of the DualQ spec
       [RFC9332]).

       When there is no Classic traffic, the L4S queue's own AQM comes
       into play.  It starts congestion marking with a very shallow
       queue, so L4S traffic maintains very low queuing delay.

       If either queue becomes persistently overloaded, drop of some
       ECN-capable packets is introduced, as recommended in Section 7 of
       the ECN spec [RFC3168] and Section 4.2.1 of the AQM
       recommendations [RFC7567].  The trade-offs with different
       approaches are discussed in Section 4.2.3 of the DualQ spec
       [RFC9332] (not shown in the figure here).

       The Dual-Queue Coupled AQM has been specified as generically as
       possible [RFC9332] without specifying the particular AQMs to use
       in the two queues so that designers are free to implement diverse
       ideas.  Informational appendices in that document give pseudocode
       examples of two different specific AQM approaches: one called
       DualPI2 (pronounced Dual PI Squared) [DualPI2Linux ] that uses the
       PI2 variant of PIE and a zero-config variant of Random Early
       Detection (RED) called Curvy RED.  A DualQ Coupled AQM based on
       PIE has also been specified and implemented for Low Latency
       DOCSIS [DOCSIS3.1].

                     (3)                  (2)
                     .-------^------..------------^------------------.
        ,-(1)-----.                               _____
       ; ________  :            L4S  -------.    |     |
       :|Scalable| :               _\      ||__\_|mark |
       :| sender | :  __________  / /      ||  / |_____|\   _________
       :|________|\; |          |/   -------'       ^    \1|condit'nl|
        `---------'\_|  IP-ECN  |          Coupling :     \|priority |_\
         ________  / |Classifier|                   :     /|scheduler| /
        |Classic |/  |__________|\   -------.     __:__  / |_________|
        | sender |                \_\ || | ||__\_|mark/|/
        |________|                  / || | ||  / |drop |
                             Classic -------'    |_____|


       (1) Scalable sending host
       (2) Isolation in separate network queues
       (3) Packet identification protocol

           Figure 1: Components of an L4S DualQ Coupled AQM Solution

   b.  Per-Flow Queues and AQMs: A scheduler with per-flow queues, such
       as FQ-CoDel or FQ-PIE, can be used for L4S.  For instance, within
       each queue of an FQ-CoDel system, as well as a CoDel AQM, there
       is typically also the option of ECN marking at an immediate
       (unsmoothed) shallow threshold to support use in data centres
       (see Section 5.2.7 of the FQ-CoDel spec [RFC8290]).  In Linux,
       this has been modified so that the shallow threshold can be
       solely applied to ECT(1) packets [FQ_CoDel_Thresh ].  Then, if
       there is a flow of Not-ECT or ECT(0) packets in the per-flow
       queue, the Classic AQM (e.g., CoDel) is applied; whereas, if
       there is a flow of ECT(1) packets in the queue, the shallower
       (typically sub-millisecond) threshold is applied.  In addition,
       ECT(0) and Not-ECT packets could potentially be classified into a
       separate flow queue from ECT(1) and CE packets to avoid them
       mixing if they share a common flow identifier (e.g., in a VPN).

   c.  Dual queues but per-flow AQMs: It should also be possible to use
       dual queues for isolation but with per-flow marking to control
       flow rates (instead of the coupled per-queue marking of the Dual-
       Queue Coupled AQM).  One of the two queues would be for isolating
       L4S packets, which would be classified by the ECN codepoint.
       Flow rates could be controlled by flow-specific marking.  The
       policy goal of the marking could be to differentiate flow rates
       (e.g., [Nadas20], which requires additional signalling of a per-
       flow 'value') or to equalize flow rates (perhaps in a similar way
       to Approx Fair CoDel [AFCD ] [CODEL-APPROX-FAIR ] but with two
       queues not one).

       Note that, whenever the term 'DualQ' is used loosely without
       saying whether marking is per queue or per flow, it means a dual-
       queue AQM with per-queue marking.

### 4.3. Host Mechanisms

   The L4S architecture includes two main mechanisms in the end host
   that we enumerate next:

   a.  Scalable congestion control at the sender: Section 2 defines a
       Scalable congestion control as one where the average time from
       one congestion signal to the next (the recovery time) remains
       invariant as flow rate scales, all other factors being equal.
       DCTCP is the most widely used example.  It has been documented as
       an informational record of the protocol currently in use in
       controlled environments [RFC8257].  A list of safety and
       performance improvements for a Scalable congestion control to be
       usable on the public Internet has been drawn up (see the so-
       called 'Prague L4S requirements' in Appendix A of [RFC9331]).
       The subset that involve risk of harm to others have been captured
       as normative requirements in Section 4 of [RFC9331].  TCP Prague
       [PRAGUE-CC ] has been implemented in Linux as a reference
       implementation to address these requirements [PragueLinux ].

       Transport protocols other than TCP use various congestion
       controls that are designed to be friendly with Reno.  Before they
       can use the L4S service, they will need to be updated to
       implement a Scalable congestion response, which they will have to
       indicate by using the ECT(1) codepoint.  Scalable variants are
       under consideration for more recent transport protocols (e.g.,
       QUIC), and the L4S ECN part of BBRv2 [BBRv2] [BBR-CC ] is a
       Scalable congestion control intended for the TCP and QUIC
       transports, amongst others.  Also, an L4S variant of the RMCAT
       SCReAM controller [RFC8298] has been implemented [SCReAM-L4S ] for
       media transported over RTP. Section 4.3 of the L4S ECN spec [RFC9331] defines Scalable
       congestion control in more detail and specifies the requirements
       that an L4S Scalable congestion control has to comply with.

   b.  The ECN feedback in some transport protocols is already
       sufficiently fine-grained for L4S (specifically DCCP [RFC4340]
       and QUIC [RFC9000]).  But others either require updates or are in
       the process of being updated:

       *  For the case of TCP, the feedback protocol for ECN embeds the
          assumption from Classic ECN [RFC3168] that an ECN mark is
          equivalent to a drop, making it unusable for a Scalable TCP.
          Therefore, the implementation of TCP receivers will have to be
          upgraded [RFC7560].  Work to standardize and implement more
          accurate ECN feedback for TCP (AccECN) is in progress [ACCECN ]
          [PragueLinux ].

       *  ECN feedback was only roughly sketched in the appendix of the
          now obsoleted second specification of SCTP [RFC4960], while a
          fuller specification was proposed in a long-expired document
          [ECN-SCTP ].  A new design would need to be implemented and
          deployed before SCTP could support L4S.

       *  For RTP, sufficient ECN feedback was defined in [RFC6679], but
          [RFC8888] defines the latest Standards Track improvements.

