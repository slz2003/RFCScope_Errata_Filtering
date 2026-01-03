Let us analyze section 1 of draft draft-ietf-tcpm-accurate-ecn-34. All references made by section 1 have also been included below.

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

## 4. Active Queue Management (AQM)

   Random Early Detection (RED) is one mechanism for Active Queue
   Management (AQM) that has been proposed to detect incipient
   congestion [FJ93], and is currently being deployed in the Internet
   [RFC2309].  AQM is meant to be a general mechanism using one of
   several alternatives for congestion indication, but in the absence of
   ECN, AQM is restricted to using packet drops as a mechanism for
   congestion indication.  AQM drops packets based on the average queue
   length exceeding a threshold, rather than only when the queue
   overflows.  However, because AQM may drop packets before the queue
   actually overflows, AQM is not always forced by memory limitations to
   discard the packet.

   AQM can set a Congestion Experienced (CE) codepoint in the packet
   header instead of dropping the packet, when such a field is provided
   in the IP header and understood by the transport protocol.  The use
   of the CE codepoint with ECN allows the receiver(s) to receive the
   packet, avoiding the potential for excessive delays due to
   retransmissions after packet losses.  We use the term 'CE packet' to
   denote a packet that has the CE codepoint set.

## 5. Explicit Congestion Notification in IP

   This document specifies that the Internet provide a congestion
   indication for incipient congestion (as in RED and earlier work
   [RJ90]) where the notification can sometimes be through marking
   packets rather than dropping them.  This uses an ECN field in the IP
   header with two bits, making four ECN codepoints, '00' to '11'.  The
   ECN-Capable Transport (ECT) codepoints '10' and '01' are set by the
   data sender to indicate that the end-points of the transport protocol
   are ECN-capable; we call them ECT(0) and ECT(1) respectively.  The
   phrase "the ECT codepoint" in this documents refers to either of the
   two ECT codepoints.  Routers treat the ECT(0) and ECT(1) codepoints
   as equivalent.  Senders are free to use either the ECT(0) or the
   ECT(1) codepoint to indicate ECT, on a packet-by-packet basis. 
   The use of both the two codepoints for ECT, ECT(0) and ECT(1), is
   motivated primarily by the desire to allow mechanisms for the data
   sender to verify that network elements are not erasing the CE
   codepoint, and that data receivers are properly reporting to the
   sender the receipt of packets with the CE codepoint set, as required
   by the transport protocol.  Guidelines for the senders and receivers
   to differentiate between the ECT(0) and ECT(1) codepoints will be
   addressed in separate documents, for each transport protocol.  In
   particular, this document does not address mechanisms for TCP end-
   nodes to differentiate between the ECT(0) and ECT(1) codepoints.
   Protocols and senders that only require a single ECT codepoint SHOULD
   use ECT(0).

   The not-ECT codepoint '00' indicates a packet that is not using ECN.
   The CE codepoint '11' is set by a router to indicate congestion to
   the end nodes.  Routers that have a packet arriving at a full queue
   drop the packet, just as they do in the absence of ECN.

      +-----+-----+
      | ECN FIELD |
      +-----+-----+
        ECT   CE         [Obsolete] RFC 2481 names for the ECN bits.
         0     0         Not-ECT
         0     1         ECT(1)
         1     0         ECT(0)
         1     1         CE

      Figure 1: The ECN Field in IP.

   The use of two ECT codepoints essentially gives a one-bit ECN nonce
   in packet headers, and routers necessarily "erase" the nonce when
   they set the CE codepoint [SCWA99].  For example, routers that erased
   the CE codepoint would face additional difficulty in reconstructing
   the original nonce, and thus repeated erasure of the CE codepoint
   would be more likely to be detected by the end-nodes.  The ECN nonce
   also can address the problem of misbehaving transport receivers lying
   to the transport sender about whether or not the CE codepoint was set
   in a packet.  The motivations for the use of two ECT codepoints is
   discussed in more detail in Section 20, along with some discussion of
   alternate possibilities for the fourth ECT codepoint (that is, the
   codepoint '01').  Backwards compatibility with earlier ECN
   implementations that do not understand the ECT(1) codepoint is
   discussed in Section 11.

   In RFC 2481 [RFC2481], the ECN field was divided into the ECN-Capable
   Transport (ECT) bit and the CE bit.  The ECN field with only the
   ECN-Capable Transport (ECT) bit set in RFC 2481 corresponds to the
   ECT(0) codepoint in this document, and the ECN field with both the 
   ECT and CE bit in RFC 2481 corresponds to the CE codepoint in this
   document.  The '01' codepoint was left undefined in RFC 2481, and
   this is the reason for recommending the use of ECT(0) when only a
   single ECT codepoint is needed.

         0     1     2     3     4     5     6     7
      +-----+-----+-----+-----+-----+-----+-----+-----+
      |          DS FIELD, DSCP           | ECN FIELD |
      +-----+-----+-----+-----+-----+-----+-----+-----+

        DSCP: differentiated services codepoint
        ECN:  Explicit Congestion Notification

      Figure 2: The Differentiated Services and ECN Fields in IP.

   Bits 6 and 7 in the IPv4 TOS octet are designated as the ECN field.
   The IPv4 TOS octet corresponds to the Traffic Class octet in IPv6,
   and the ECN field is defined identically in both cases.  The
   definitions for the IPv4 TOS octet [RFC791] and the IPv6 Traffic
   Class octet have been superseded by the six-bit DS (Differentiated
   Services) Field [RFC2474, RFC2780].  Bits 6 and 7 are listed in
   [RFC2474] as Currently Unused, and are specified in RFC 2780 as
   approved for experimental use for ECN.  Section 22 gives a brief
   history of the TOS octet.

   Because of the unstable history of the TOS octet, the use of the ECN
   field as specified in this document cannot be guaranteed to be
   backwards compatible with those past uses of these two bits that
   pre-date ECN.  The potential dangers of this lack of backwards
   compatibility are discussed in Section 22.

   Upon the receipt by an ECN-Capable transport of a single CE packet,
   the congestion control algorithms followed at the end-systems MUST be
   essentially the same as the congestion control response to a *single*
   dropped packet.  For example, for ECN-Capable TCP the source TCP is
   required to halve its congestion window for any window of data
   containing either a packet drop or an ECN indication.

   One reason for requiring that the congestion-control response to the
   CE packet be essentially the same as the response to a dropped packet
   is to accommodate the incremental deployment of ECN in both end-
   systems and in routers.  Some routers may drop ECN-Capable packets
   (e.g., using the same AQM policies for congestion detection) while
   other routers set the CE codepoint, for equivalent levels of
   congestion.  Similarly, a router might drop a non-ECN-Capable packet
   but set the CE codepoint in an ECN-Capable packet, for equivalent 
   levels of congestion.  If there were different congestion control
   responses to a CE codepoint than to a packet drop, this could result
   in unfair treatment for different flows.

   An additional goal is that the end-systems should react to congestion
   at most once per window of data (i.e., at most once per round-trip
   time), to avoid reacting multiple times to multiple indications of
   congestion within a round-trip time.

   For a router, the CE codepoint of an ECN-Capable packet SHOULD only
   be set if the router would otherwise have dropped the packet as an
   indication of congestion to the end nodes. When the router's buffer
   is not yet full and the router is prepared to drop a packet to inform
   end nodes of incipient congestion, the router should first check to
   see if the ECT codepoint is set in that packet's IP header.  If so,
   then instead of dropping the packet, the router MAY instead set the
   CE codepoint in the IP header.

   An environment where all end nodes were ECN-Capable could allow new
   criteria to be developed for setting the CE codepoint, and new
   congestion control mechanisms for end-node reaction to CE packets.
   However, this is a research issue, and as such is not addressed in
   this document.

   When a CE packet (i.e., a packet that has the CE codepoint set) is
   received by a router, the CE codepoint is left unchanged, and the
   packet is transmitted as usual. When severe congestion has occurred
   and the router's queue is full, then the router has no choice but to
   drop some packet when a new packet arrives.  We anticipate that such
   packet losses will become relatively infrequent when a majority of
   end-systems become ECN-Capable and participate in TCP or other
   compatible congestion control mechanisms. In an ECN-Capable
   environment that is adequately-provisioned, packet losses should
   occur primarily during transients or in the presence of non-
   cooperating sources.

   The above discussion of when CE may be set instead of dropping a
   packet applies by default to all Differentiated Services Per-Hop
   Behaviors (PHBs) [RFC 2475].  Specifications for PHBs MAY provide
   more specifics on how a compliant implementation is to choose between
   setting CE and dropping a packet, but this is NOT REQUIRED.  A router
   MUST NOT set CE instead of dropping a packet when the drop that would
   occur is caused by reasons other than congestion or the desire to
   indicate incipient congestion to end nodes (e.g., a diffserv edge
   node may be configured to unconditionally drop certain classes of
   traffic to prevent them from entering its diffserv domain). 
   We expect that routers will set the CE codepoint in response to
   incipient congestion as indicated by the average queue size, using
   the RED algorithms suggested in [FJ93, RFC2309].  To the best of our
   knowledge, this is the only proposal currently under discussion in
   the IETF for routers to drop packets proactively, before the buffer
   overflows.  However, this document does not attempt to specify a
   particular mechanism for active queue management, leaving that
   endeavor, if needed, to other areas of the IETF.  While ECN is
   inextricably tied up with the need to have a reasonable active queue
   management mechanism at the router, the reverse does not hold; active
   queue management mechanisms have been developed and deployed
   independent of ECN, using packet drops as indications of congestion
   in the absence of ECN in the IP architecture.

### 5.1. ECN as an Indication of Persistent Congestion

   We emphasize that a *single* packet with the CE codepoint set in an
   IP packet causes the transport layer to respond, in terms of
   congestion control, as it would to a packet drop.  The instantaneous
   queue size is likely to see considerable variations even when the
   router does not experience persistent congestion.  As such, it is
   important that transient congestion at a router, reflected by the
   instantaneous queue size reaching a threshold much smaller than the
   capacity of the queue, not trigger a reaction at the transport layer.
   Therefore, the CE codepoint should not be set by a router based on
   the instantaneous queue size.

   For example, since the ATM and Frame Relay mechanisms for congestion
   indication have typically been defined without an associated notion
   of average queue size as the basis for determining that an
   intermediate node is congested, we believe that they provide a very
   noisy signal. The TCP-sender reaction specified in this document for
   ECN is NOT the appropriate reaction for such a noisy signal of
   congestion notification.  However, if the routers that interface to
   the ATM network have a way of maintaining the average queue at the
   interface, and use it to come to a reliable determination that the
   ATM subnet is congested, they may use the ECN notification that is
   defined here.

   We continue to encourage experiments in techniques at layer 2 (e.g.,
   in ATM switches or Frame Relay switches) to take advantage of ECN.
   For example, using a scheme such as RED (where packet marking is
   based on the average queue length exceeding a threshold), layer 2
   devices could provide a reasonably reliable indication of congestion.
   When all the layer 2 devices in a path set that layer's own
   Congestion Experienced codepoint (e.g., the EFCI bit for ATM, the
   FECN bit in Frame Relay) in this reliable manner, then the interface
   router to the layer 2 network could copy the state of that layer 2
   Congestion Experienced codepoint into the CE codepoint in the IP
   header.  We recognize that this is not the current practice, nor is
   it in current standards. However, encouraging experimentation in this
   manner may provide the information needed to enable evolution of
   existing layer 2 mechanisms to provide a more reliable means of
   congestion indication, when they use a single bit for indicating
   congestion.

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

### 5.3. Fragmentation

   ECN-capable packets MAY have the DF (Don't Fragment) bit set.
   Reassembly of a fragmented packet MUST NOT lose indications of
   congestion.  In other words, if any fragment of an IP packet to be
   reassembled has the CE codepoint set, then one of two actions MUST be
   taken:
      * Set the CE codepoint on the reassembled packet.  However, this
        MUST NOT occur if any of the other fragments contributing to
        this reassembly carries the Not-ECT codepoint.

      * The packet is dropped, instead of being reassembled, for any
        other reason.

   If both actions are applicable, either MAY be chosen.  Reassembly of
   a fragmented packet MUST NOT change the ECN codepoint when all of the
   fragments carry the same codepoint.

   We would note that because RFC 2481 did not specify reassembly
   behavior, older ECN implementations conformant with that Experimental
   RFC do not necessarily perform reassembly correctly, in terms of
   preserving the CE codepoint in a fragment.  The sender could avoid
   the consequences of this behavior by setting the DF bit in ECN-
   Capable packets.

   Situations may arise in which the above reassembly specification is
   insufficiently precise.  For example, if there is a malicious or
   broken entity in the path at or after the fragmentation point, packet
   fragments could carry a mixture of ECT(0), ECT(1), and/or Not-ECT
   codepoints.  The reassembly specification above does not place
   requirements on reassembly of fragments in this case.  In situations
   where more precise reassembly behavior would be required, protocol
   specifications SHOULD instead specify that DF MUST be set in all
   ECN-capable packets sent by the protocol.

## 6. Support from the Transport Protocol

   ECN requires support from the transport protocol, in addition to the
   functionality given by the ECN field in the IP packet header. The
   transport protocol might require negotiation between the endpoints
   during setup to determine that all of the endpoints are ECN-capable,
   so that the sender can set the ECT codepoint in transmitted packets.
   Second, the transport protocol must be capable of reacting
   appropriately to the receipt of CE packets.  This reaction could be
   in the form of the data receiver informing the data sender of the
   received CE packet (e.g., TCP), of the data receiver unsubscribing to
   a layered multicast group (e.g., RLM [MJV96]), or of some other
   action that ultimately reduces the arrival rate of that flow on that
   congested link.  CE packets indicate persistent rather than transient
   congestion (see Section 5.1), and hence reactions to the receipt of
   CE packets should be those appropriate for persistent congestion.

   This document only addresses the addition of ECN Capability to TCP,
   leaving issues of ECN in other transport protocols to further
   research.  For TCP, ECN requires three new pieces of functionality:
   negotiation between the endpoints during connection setup to
   determine if they are both ECN-capable; an ECN-Echo (ECE) flag in the
   TCP header so that the data receiver can inform the data sender when
   a CE packet has been received; and a Congestion Window Reduced (CWR)
   flag in the TCP header so that the data sender can inform the data
   receiver that the congestion window has been reduced. The support
   required from other transport protocols is likely to be different,
   particularly for unreliable or reliable multicast transport
   protocols, and will have to be determined as other transport
   protocols are brought to the IETF for standardization.

   In a mild abuse of terminology, in this document we refer to `TCP
   packets' instead of `TCP segments'.

### 6.1. TCP

   The following sections describe in detail the proposed use of ECN in
   TCP.  This proposal is described in essentially the same form in
   [Floyd94]. We assume that the source TCP uses the standard congestion
   control algorithms of Slow-start, Fast Retransmit and Fast Recovery
   [RFC2581].

   This proposal specifies two new flags in the Reserved field of the
   TCP header.  The TCP mechanism for negotiating ECN-Capability uses
   the ECN-Echo (ECE) flag in the TCP header.  Bit 9 in the Reserved
   field of the TCP header is designated as the ECN-Echo flag.  The
   location of the 6-bit Reserved field in the TCP header is shown in
   Figure 4 of RFC 793 [RFC793] (and is reproduced below for
   completeness).  This specification of the ECN Field leaves the
   Reserved field as a 4-bit field using bits 4-7.

   To enable the TCP receiver to determine when to stop setting the
   ECN-Echo flag, we introduce a second new flag in the TCP header, the
   CWR flag.  The CWR flag is assigned to Bit 8 in the Reserved field of
   the TCP header.

        0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
      |               |                       | U | A | P | R | S | F |
      | Header Length |        Reserved       | R | C | S | S | Y | I |
      |               |                       | G | K | H | T | N | N |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

      Figure 3: The old definition of bytes 13 and 14 of the TCP
                header. 
        0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
      |               |               | C | E | U | A | P | R | S | F |
      | Header Length |    Reserved   | W | C | R | C | S | S | Y | I |
      |               |               | R | E | G | K | H | T | N | N |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

      Figure 4: The new definition of bytes 13 and 14 of the TCP
                Header.

   Thus, ECN uses the ECT and CE flags in the IP header (as shown in
   Figure 1) for signaling between routers and connection endpoints, and
   uses the ECN-Echo and CWR flags in the TCP header (as shown in Figure
   4) for TCP-endpoint to TCP-endpoint signaling.  For a TCP connection,
   a typical sequence of events in an ECN-based reaction to congestion
   is as follows:

      * An ECT codepoint is set in packets transmitted by the sender to
        indicate that ECN is supported by the transport entities for
        these packets.

      * An ECN-capable router detects impending congestion and detects
        that an ECT codepoint is set in the packet it is about to drop.
        Instead of dropping the packet, the router chooses to set the CE
        codepoint in the IP header and forwards the packet.

      * The receiver receives the packet with the CE codepoint set, and
        sets the ECN-Echo flag in its next TCP ACK sent to the sender.

      * The sender receives the TCP ACK with ECN-Echo set, and reacts to
        the congestion as if a packet had been dropped.

      * The sender sets the CWR flag in the TCP header of the next
        packet sent to the receiver to acknowledge its receipt of and
        reaction to the ECN-Echo flag.

   The negotiation for using ECN by the TCP transport entities and the
   use of the ECN-Echo and CWR flags is described in more detail in the
   sections below.

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

# Referenced Sections from RFC 6582: The NewReno Modification to TCP's Fast Recovery Algorithm

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   For the typical implementation of the TCP fast recovery algorithm
   described in [RFC5681] (first implemented in the 1990 BSD Reno
   release, and referred to as the "Reno algorithm" in [FF96]), the TCP
   data sender only retransmits a packet after a retransmit timeout has
   occurred, or after three duplicate acknowledgments have arrived
   triggering the fast retransmit algorithm.  A single retransmit
   timeout might result in the retransmission of several data packets,
   but each invocation of the fast retransmit algorithm in RFC 5681   leads to the retransmission of only a single data packet.

   Two problems arise with Reno TCP when multiple packet losses occur in
   a single window.  First, Reno will often take a timeout, as has been
   documented in [Hoe95].  Second, even if a retransmission timeout is
   avoided, multiple fast retransmits and window reductions can occur,
   as documented in [F94].  When multiple packet losses occur, if the
   SACK option [RFC2883] is available, the TCP sender has the
   information to make intelligent decisions about which packets to
   retransmit and which packets not to retransmit during fast recovery. 
   This document applies to TCP connections that are unable to use the
   TCP Selective Acknowledgment (SACK) option, either because the option
   is not locally supported or because the TCP peer did not indicate a
   willingness to use SACK.

   In the absence of SACK, there is little information available to the
   TCP sender in making retransmission decisions during fast recovery.
   From the three duplicate acknowledgments, the sender infers a packet
   loss, and retransmits the indicated packet.  After this, the data
   sender could receive additional duplicate acknowledgments, as the
   data receiver acknowledges additional data packets that were already
   in flight when the sender entered fast retransmit.

   In the case of multiple packets dropped from a single window of data,
   the first new information available to the sender comes when the
   sender receives an acknowledgment for the retransmitted packet (that
   is, the packet retransmitted when fast retransmit was first entered).
   If there is a single packet drop and no reordering, then the
   acknowledgment for this packet will acknowledge all of the packets
   transmitted before fast retransmit was entered.  However, if there
   are multiple packet drops, then the acknowledgment for the
   retransmitted packet will acknowledge some but not all of the packets
   transmitted before the fast retransmit.  We call this acknowledgment
   a partial acknowledgment.

   Along with several other suggestions, [Hoe95] suggested that during
   fast recovery the TCP data sender respond to a partial acknowledgment
   by inferring that the next in-sequence packet has been lost and
   retransmitting that packet.  This document describes a modification
   to the fast recovery algorithm in RFC 5681 that incorporates a
   response to partial acknowledgments received during fast recovery.
   We call this modified fast recovery algorithm NewReno, because it is
   a slight but significant variation of the behavior that has been
   historically referred to as Reno.  This document does not discuss the
   other suggestions in [Hoe95] and [Hoe96], such as a change to the
   ssthresh parameter during slow start, or the proposal to send a new
   packet for every two duplicate acknowledgments during fast recovery.
   The version of NewReno in this document also draws on other
   discussions of NewReno in the literature [LM97] [Hen98].

   We do not claim that the NewReno version of fast recovery described
   here is an optimal modification of fast recovery for responding to
   partial acknowledgments, for TCP connections that are unable to use
   SACK.  Based on our experiences with the NewReno modification in the
   network simulator known as ns-2 [NS ] and with numerous
   implementations of NewReno, we believe that this modification
   improves the performance of the fast retransmit and fast recovery 
   algorithms in a wide variety of scenarios.  Previous versions of this
   RFC [RFC2582] [RFC3782] provide simulation-based evidence of the
   possible performance gains.

## 3. The Fast Retransmit and Fast Recovery Algorithms in NewReno





### 3.1. Protocol Overview

   The basic idea of these extensions to the fast retransmit and fast
   recovery algorithms described in Section 3.2 of [RFC5681] is as
   follows.  The TCP sender can infer, from the arrival of duplicate
   acknowledgments, whether multiple losses in the same window of data
   have most likely occurred, and avoid taking a retransmit timeout or
   making multiple congestion window reductions due to such an event.

   The NewReno modification applies to the fast recovery procedure that
   begins when three duplicate ACKs are received and ends when either a
   retransmission timeout occurs or an ACK arrives that acknowledges all
   of the data up to and including the data that was outstanding when
   the fast recovery procedure began. 





### 3.2. Specification

   The procedures specified in Section 3.2 of [RFC5681] are followed,
   with the modifications listed below.  Note that this specification
   avoids the use of the key words defined in RFC 2119 [RFC2119], since
   it mainly provides sender-side implementation guidance for
   performance improvement, and does not affect interoperability.

   1)  Initialization of TCP protocol control block:
       When the TCP protocol control block is initialized, recover is
       set to the initial send sequence number.

   2)  Three duplicate ACKs:
       When the third duplicate ACK is received, the TCP sender first
       checks the value of recover to see if the Cumulative
       Acknowledgment field covers more than recover.  If so, the value
       of recover is incremented to the value of the highest sequence
       number transmitted by the TCP so far.  The TCP then enters fast
       retransmit (step 2 of Section 3.2 of [RFC5681]).  If not, the TCP
       does not enter fast retransmit and does not reset ssthresh.

   3)  Response to newly acknowledged data:
       Step 6 of [RFC5681] specifies the response to the next ACK that
       acknowledges previously unacknowledged data.  When an ACK arrives
       that acknowledges new data, this ACK could be the acknowledgment
       elicited by the initial retransmission from fast retransmit, or
       elicited by a later retransmission.  There are two cases:

       Full acknowledgments:
       If this ACK acknowledges all of the data up to and including
       recover, then the ACK acknowledges all the intermediate segments
       sent between the original transmission of the lost segment and
       the receipt of the third duplicate ACK.  Set cwnd to either (1)
       min (ssthresh, max(FlightSize, SMSS) + SMSS) or (2) ssthresh,
       where ssthresh is the value set when fast retransmit was entered,
       and where FlightSize in (1) is the amount of data presently
       outstanding.  This is termed "deflating" the window.  If the
       second option is selected, the implementation is encouraged to
       take measures to avoid a possible burst of data, in case the
       amount of data outstanding in the network is much less than the
       new congestion window allows.  A simple mechanism is to limit the
       number of data packets that can be sent in response to a single
       acknowledgment.  Exit the fast recovery procedure. 
       Partial acknowledgments:
       If this ACK does *not* acknowledge all of the data up to and
       including recover, then this is a partial ACK.  In this case,
       retransmit the first unacknowledged segment.  Deflate the
       congestion window by the amount of new data acknowledged by the
       Cumulative Acknowledgment field.  If the partial ACK acknowledges
       at least one SMSS of new data, then add back SMSS bytes to the
       congestion window.  This artificially inflates the congestion
       window in order to reflect the additional segment that has left
       the network.  Send a new segment if permitted by the new value of
       cwnd.  This "partial window deflation" attempts to ensure that,
       when fast recovery eventually ends, approximately ssthresh amount
       of data will be outstanding in the network.  Do not exit the fast
       recovery procedure (i.e., if any duplicate ACKs subsequently
       arrive, execute step 4 of Section 3.2 of [RFC5681]).

       For the first partial ACK that arrives during fast recovery, also
       reset the retransmit timer.  Timer management is discussed in
       more detail in Section 4.

   4)  Retransmit timeouts:
       After a retransmit timeout, record the highest sequence number
       transmitted in the variable recover, and exit the fast recovery
       procedure if applicable.

   Step 2 above specifies a check that the Cumulative Acknowledgment
   field covers more than recover.  Because the acknowledgment field
   contains the sequence number that the sender next expects to receive,
   the acknowledgment "ack_number" covers more than recover when

      ack_number - 1 > recover;

   i.e., at least one byte more of data is acknowledged beyond the
   highest byte that was outstanding when fast retransmit was last
   entered.

   Note that in step 3 above, the congestion window is deflated after a
   partial acknowledgment is received.  The congestion window was likely
   to have been inflated considerably when the partial acknowledgment
   was received.  In addition, depending on the original pattern of
   packet losses, the partial acknowledgment might acknowledge nearly a
   window of data.  In this case, if the congestion window was not
   deflated, the data sender might be able to send nearly a window of
   data back-to-back. 
   This document does not specify the sender's response to duplicate
   ACKs when the fast retransmit/fast recovery algorithm is not invoked.
   This is addressed in other documents, such as those describing the
   Limited Transmit procedure [RFC3042].  This document also does not
   address issues of adjusting the duplicate acknowledgment threshold,
   but assumes the threshold specified in the IETF standards; the
   current standard is [RFC5681], which specifies a threshold of three
   duplicate acknowledgments.

   As a final note, we would observe that in the absence of the SACK
   option, the data sender is working from limited information.  When
   the issue of recovery from multiple dropped packets from a single
   window of data is of particular importance, the best alternative
   would be to use the SACK option.

## 4. Handling Duplicate Acknowledgments after a Timeout

   After each retransmit timeout, the highest sequence number
   transmitted so far is recorded in the variable recover.  If, after a
   retransmit timeout, the TCP data sender retransmits three consecutive
   packets that have already been received by the data receiver, then
   the TCP data sender will receive three duplicate acknowledgments that
   do not cover more than recover.  In this case, the duplicate
   acknowledgments are not an indication of a new instance of
   congestion.  They are simply an indication that the sender has
   unnecessarily retransmitted at least three packets.

   However, when a retransmitted packet is itself dropped, the sender
   can also receive three duplicate acknowledgments that do not cover
   more than recover.  In this case, the sender would have been better
   off if it had initiated fast retransmit.  For a TCP sender that
   implements the algorithm specified in Section 3.2 of this document,
   the sender does not infer a packet drop from duplicate
   acknowledgments in this scenario.  As always, the retransmit timer is
   the backup mechanism for inferring packet loss in this case.

   There are several heuristics, based on timestamps or on the amount of
   advancement of the Cumulative Acknowledgment field, that allow the
   sender to distinguish, in some cases, between three duplicate
   acknowledgments following a retransmitted packet that was dropped,
   and three duplicate acknowledgments from the unnecessary
   retransmission of three packets [Gur03] [GF04].  The TCP sender may
   use such a heuristic to decide to invoke a fast retransmit in some
   cases, even when the three duplicate acknowledgments do not cover
   more than recover. 
   For example, when three duplicate acknowledgments are caused by the
   unnecessary retransmission of three packets, this is likely to be
   accompanied by the Cumulative Acknowledgment field advancing by at
   least four segments.  Similarly, a heuristic based on timestamps uses
   the fact that when there is a hole in the sequence space, the
   timestamp echoed in the duplicate acknowledgment is the timestamp of
   the most recent data packet that advanced the Cumulative
   Acknowledgment field [RFC1323].  If timestamps are used, and the
   sender stores the timestamp of the last acknowledged segment, then
   the timestamp echoed by duplicate acknowledgments can be used to
   distinguish between a retransmitted packet that was dropped and three
   duplicate acknowledgments from the unnecessary retransmission of
   three packets.

### 4.1. ACK Heuristic

   If the ACK-based heuristic is used, then following the advancement of
   the Cumulative Acknowledgment field, the sender stores the value of
   the previous cumulative acknowledgment as prev_highest_ack, and
   stores the latest cumulative ACK as highest_ack.  In addition, the
   following check is performed if, in step 2 of Section 3.2, the
   Cumulative Acknowledgment field does not cover more than recover.

   2*)  If the Cumulative Acknowledgment field didn't cover more than
        recover, check to see if the congestion window is greater than
        SMSS bytes and the difference between highest_ack and
        prev_highest_ack is at most 4*SMSS bytes.  If true, duplicate
        ACKs indicate a lost segment (enter fast retransmit).
        Otherwise, duplicate ACKs likely result from unnecessary
        retransmissions (do not enter fast retransmit).

   The congestion window check serves to protect against fast retransmit
   immediately after a retransmit timeout.

   If several ACKs are lost, the sender can see a jump in the cumulative
   ACK of more than three segments, and the heuristic can fail.
   [RFC5681] recommends that a receiver should send duplicate ACKs for
   every out-of-order data packet, such as a data packet received during
   fast recovery.  The ACK heuristic is more likely to fail if the
   receiver does not follow this advice, because then a smaller number
   of ACK losses are needed to produce a sufficient jump in the
   cumulative ACK. 





### 4.2. Timestamp Heuristic

   If this heuristic is used, the sender stores the timestamp of the
   last acknowledged segment.  In addition, the last sentence of step 2
   in Section 3.2 of this document is replaced as follows:

   2**) If the Cumulative Acknowledgment field didn't cover more than
        recover, check to see if the echoed timestamp in the last
        non-duplicate acknowledgment equals the stored timestamp.  If
        true, duplicate ACKs indicate a lost segment (enter fast
        retransmit).  Otherwise, duplicate ACKs likely result from
        unnecessary retransmissions (do not enter fast retransmit).

   The timestamp heuristic works correctly, both when the receiver
   echoes timestamps, as specified by [RFC1323], and by its revision
   attempts.  However, if the receiver arbitrarily echoes timestamps,
   the heuristic can fail.  The heuristic can also fail if a timeout was
   spurious and returning ACKs are not from retransmitted segments.
   This can be prevented by detection algorithms such as the Eifel
   detection algorithm [RFC3522].

# Referenced Sections from RFC 7560: Problem Statement and Requirements for Increased Accuracy in Explicit Congestion Notification (ECN) Feedback

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Explicit Congestion Notification (ECN) [RFC3168] is a mechanism where
   network nodes can mark IP packets instead of dropping them to
   indicate congestion to the endpoints.  An ECN-capable receiver will
   feed this information back to the sender.  ECN is specified for TCP
   in such a way that only one feedback signal can be transmitted per
   Round-Trip Time (RTT).  This is sufficient for preexisting TCP
   congestion control mechanisms that perform only one reduction in
   sending rate per RTT, independent of the number of ECN congestion
   marks.  But recently proposed or deployed mechanisms like Congestion
   Exposure (ConEx) [RFC6789] or Data Center TCP (DCTCP) [DCTCP ] need
   more accurate ECN feedback than 'classic ECN' [RFC3168] to work
   correctly in the case where more than one marking is received in any
   one RTT.

   For an in-depth discussion of the application benefits of using ECN
   (including with sufficiently granular feedback), see [ECN-BENEFITS ].

   ECN is also defined for transport protocols beside TCP.  ECN feedback
   as defined for RTP/UDP [RFC6679] provides a very detailed level of
   information, delivering individual counters for all four ECN
   codepoints as well as lost and duplicate segments, but at the cost of
   high signalling overhead.  ECN feedback for SCTP has been proposed in
   [SCTP-ECN ].  This delivers a counter for the number of ECN-capable
   packets that were marked due to congestion (since the last sender-
   side window reduction), but it comes at the cost of increased
   overhead.

   Today, implementations of DCTCP already exist that alter TCP's ECN
   feedback protocol in proprietary ways (DCTCP was released in
   Microsoft Windows 8, and implementations exist for Linux and
   FreeBSD).  However, the changes DCTCP makes to TCP omit capability
   negotiation, relying instead on uniform configuration across all
   hosts and network devices with ECN capability.  A primary motivation
   for this document is to intervene before each proprietary
   implementation invents its own non-interoperable handshake, which
   could lead to _de facto_ consumption of the few flags or codepoints
   that remain available for standardizing capability negotiation.

   This document lists requirements for a robust and interoperable TCP/
   ECN feedback protocol that is more accurate than classic ECN
   [RFC3168] and that all implementations of new TCP extensions, like
   ConEx and/or DCTCP, can use.  While a new feedback scheme should
   still deliver as much information as classic ECN, this document also
   clarifies what has to be taken into consideration in addition.  Thus,
   the listed requirements should be addressed in the specification of a
   more accurate ECN feedback scheme.  A few solutions have already been 
   proposed.  Section 5 demonstrates how to use the requirements to
   compare them, by briefly sketching their high-level design choices
   and discussing the benefits and drawbacks of each.

   The scope of these requirements is not limited to any specific
   environment and is intended for general deployment over public and
   private IP networks.  Candidate solutions should try to adhere to all
   these requirements, but, where this is not possible, they should
   justify the deviation.  The ordering of the requirements listed in
   this document is not to be taken as an order of importance, because
   each requirement might have different weight in different deployment
   scenarios.

   These requirements are only concerned with the type and quality of
   the ECN feedback signal.  The requirements do not stipulate how a TCP
   sender might react to the improved ECN signal.  The requirements also
   do not imply that any modifications to TCP senders or receivers are
   obligatory.

### 1.1. Terminology

   We use the following terminology from [RFC3168] and [RFC3540]:

   The ECN field in the IP header:

      Not-ECT: the not ECN-Capable Transport codepoint,

      CE:      the Congestion Experienced codepoint,

      ECT(0):  the first ECN-Capable Transport codepoint, and

      ECT(1):  the second ECN-Capable Transport codepoint.

   The ECN flags in the TCP header:

      CWR:     the Congestion Window Reduced flag,

      ECE:     the ECN-Echo flag, and

      NS:      ECN Nonce Sum.

   In this document, the ECN feedback scheme as specified in [RFC3168]
   is called 'classic ECN' and any new proposal is called a 'more
   accurate ECN feedback' scheme.  A 'congestion mark' is defined as an
   IP packet where the CE codepoint is set.  A 'congestion episode'
   refers to one or more congestion marks that belong to the same
   overload situation in the network (usually during one RTT).  A TCP
   segment with the acknowledgement flag set is simply called an ACK. 





## 2. Recap of Classic ECN and ECN Nonce in IP/TCP

   ECN requires two bits in the IP header.  The ECN capability of a
   packet is indicated when either one of the two bits is set.  A
   network node can set both bits simultaneously when it experiences
   congestion.  This leads to the four codepoints (Not-ECT, ECT(0),
   ECT(1), and CE) as listed above.

   In the TCP header, the first two bits in byte 14 are defined as ECN
   feedback for each half-connection.  A TCP receiver signals the
   reception of a congestion mark using the ECN-Echo (ECE) flag in the
   TCP header.  For reliability, the receiver continues to set the ECE
   flag on every ACK.  To enable the TCP receiver to determine when to
   stop setting the ECE flag, the sender sets the CWR flag upon
   reception of an ECE feedback signal.  This always leads to a full RTT
   of ACKs with ECE set.  Thus, the receiver cannot signal back any
   additional CE markings arriving within the same RTT.

   The ECN Nonce [RFC3540] is an experimental addition to ECN that the
   TCP sender can use to protect itself against accidental or malicious
   concealment of CE-marked or dropped packets.  This addition defines
   the last bit of byte 13 in the TCP header as the Nonce Sum (NS) flag.
   The receiver maintains a nonce sum that counts the occurrence of
   ECT(1) packets and signals the least significant bit of this sum on
   the NS flag.  There are no known deployments of a TCP stack that
   makes use of the ECN Nonce extension.

       0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
     |               |           | N | C | E | U | A | P | R | S | F |
     | Header Length | Reserved  | S | W | C | R | C | S | S | Y | I |
     |               |           |   | R | E | G | K | H | T | N | N |
     +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

     Figure 1: The (Post-ECN Nonce) Definition of the TCP Header Flags

   An alternative for a sender to assure feedback integrity has been
   proposed where the sender itself occasionally inserts a CE mark or
   reorders packets, and checks that the receiver feeds these back
   faithfully [TEST-RCV ].  This alternative consumes no header bits or
   codepoints, and it releases the ECT(1) codepoint in the IP header and
   the NS flag in the TCP header for other uses. 





## 3. Use Cases

   The following two examples serve to show where existing mechanisms
   would already benefit from more accurate ECN feedback information.
   However, as it is hard to predict the future, once a more accurate
   ECN feedback mechanism that adheres to the requirements stated in
   this document is widely deployed, it's very likely that additional
   uses will be found.  The examples listed below are in no particular
   order.

   ConEx is an experimental approach that allows a sender to relay
   congestion feedback provided by the receiver into the network along
   the forward data path.  ConEx information can be used for traffic
   management to limit traffic proportionate to the actual congestion
   being caused, rather than limiting traffic based on rate or volume
   [RFC6789].  A ConEx sender uses selective acknowledgements (SACK)
   [RFC2018] for accurate feedback of loss signals, but until now TCP
   has offered no equivalent accurate feedback for ECN.

   DCTCP offers very low and predictable queuing delay.  DCTCP changes
   the reaction to congestion of a TCP sender and additionally requires
   switches/routers to have ECN enabled and configured with a low step
   threshold and no signal smoothing, so it is currently only used in
   private networks, e.g., internal to data centers.  DCTCP was released
   in Microsoft Windows 8, and implementations exist for Linux and
   FreeBSD.  To retrieve sufficient congestion information, the
   different DCTCP implementations use a proprietary ECN feedback
   protocol, but they omit capability negotiation.  Moreover, the
   feedback protocol proposed in [DCTCP ] only works if there are no
   losses at all, and otherwise it gets very confused (see Appendix A ).
   Therefore, if a generic, more accurate ECN feedback scheme were
   available, it would solve two problems for DCTCP: i) the need for a
   consistent variant of DCTCP to be deployed network-wide and ii) the
   inability to cope with ACK loss.

   Classic ECN-TCP would not benefit from more accurate ECN feedback,
   but it would not suffer either.  The same signal that is currently
   conveyed with ECN following the specification given in [RFC3168]
   would be available. 
   The following scenarios should briefly show where accurate ECN
   feedback is needed or adds value:

   A sender with standardized TCP congestion control that supports
   ConEx:
      In this case, the ConEx mechanism uses the extra information per
      RTT to re-echo the precise congestion information, but the
      congestion control algorithm still ignores multiple marks per RTT
      [RFC5681].

   A sender using DCTCP congestion control without ConEx:
      The congestion control algorithm uses the extra info per RTT to
      perform its decrease depending on the number of congestion marks.

   A sender using DCTCP congestion control and supporting ConEx:
      Both the congestion control algorithm and ConEx use the more
      accurate ECN feedback mechanism.

   As-yet-unspecified sender mechanisms:
      The above are two examples of more general interest in sender
      mechanisms that respond to the extent of congestion feedback, not
      just its existence.  It will greatly simplify incremental
      deployment if the sender can unilaterally deploy new behaviours
      and rely on the presence of generic receivers that have already
      implemented more accurate feedback.

   A TCP sender using congestion control as specified in RFC 5681   without ConEx:
      No accurate feedback is necessary here.  The congestion control
      algorithm still reacts to only one signal per RTT.  But, it is
      best to feed back all the information the receiver gets, whether
      or not the sender uses it -- at least as long as overhead is low
      or zero.

   Using CE for checking integrity:
      If a more accurate ECN feedback scheme feeds all occurrences of CE
      marks back, a sender could perform integrity checking by
      occasionally injecting CE marks itself.  Specifically, a sender
      can send packets that it randomly marks with CE (at low
      frequency), then check if feedback is received for these packets.
      The congestion notification feedback for these self-injected
      markings would not require a congestion control reaction
      [TEST-RCV ]. 





## 4. Requirements

   The requirements of the accurate ECN feedback protocol are to have
   fairly accurate (not necessarily perfect), timely, and protected
   signalling.  This leads to the following requirements, which should
   be discussed for any proposed more accurate ECN feedback scheme:

   Resilience
      The ECN feedback signal is carried within the ACK.  Pure TCP ACKs
      can get lost without recovery (not just due to congestion but also
      due to deliberate ACK thinning).  Moreover, delayed ACKs are
      commonly used with TCP.  Typically, an ACK is triggered after two
      data segments (or more, e.g., due to receive segment coalescing,
      ACK compression, ACK congestion control [RFC5690], or other
      phenomena; see [RFC3449]).  In a high-congestion situation where
      most of the packets are marked with CE, an accurate feedback
      mechanism should still be able to signal sufficient congestion
      information.  Thus, the accurate ECN feedback extension has to
      take delayed ACKs and ACK loss into account.  Also, a more
      accurate feedback protocol should still provide more accurate
      feedback than classic ECN when delayed ACKs cover more than two
      segments, or when a thin stream disables Nagle's algorithm
      [RFC896].  Finally, the feedback mechanism should not be impacted
      by reordering of ACKs, even when the ACKed sequence number does
      not increase.


   Timeliness
      A CE mark can be induced by the sending host, or more commonly a
      network node on the transmission path, and is then echoed by the
      receiver in the TCP ACK.  Thus, when this information arrives at
      the sender, it is naturally already about one RTT old.  With a
      sufficient ACK rate, a further delay of a small number of packets
      can be tolerated.  However, this information will become stale
      with large delays, given the dynamic nature of networks.  TCP
      congestion control (which itself partly introduces these dynamics)
      operates on a time scale of one RTT.  Thus, to be timely,
      congestion feedback information should be delivered within about
      one RTT.

   Integrity
      The integrity of the feedback in a more accurate ECN feedback
      scheme should be assured, at least as well as the ECN Nonce.
      Alternatively, it should at least be possible to give strong
      incentives for the receiver and network nodes to cooperate
      honestly. 
      Given there are known problems with ECN Nonce deployment, this
      document only requires that the integrity of the more accurate ECN
      feedback can be assured; it does not require that the ECN Nonce
      mechanism is employed to achieve this.  Indeed, if integrity could
      be provided in another manner, a more accurate ECN feedback
      protocol might repurpose the nonce sum (NS) flag in the TCP
      header.

      If the more accurate ECN feedback scheme provides sufficient
      information, the integrity check could be performed by, e.g.,
      deterministically setting the CE in the sender and monitoring the
      respective feedback (similar to ECT(1) and the ECN Nonce sum).
      Whether a sender should enforce when it detects wrong feedback
      information, and what kind of enforcement it should apply, are
      policy issues that need not be specified as part of the more
      accurate ECN feedback signal scheme itself, but rather when
      specifying an update to core TCP mechanisms like congestion
      control that make use of the more accurate ECN signal.

   Accuracy
      Classic ECN feeds back one congestion notification per RTT; this
      is sufficient for classic TCP congestion control, which reduces
      the sending rate at most once per RTT.  Thus, the more accurate
      ECN feedback scheme should ensure that, if a congestion episode
      occurs, at least one congestion notification is echoed and
      received per RTT as classic ECN would do.  Of course, the goal of
      a more accurate ECN extension is to reconstruct the number of CE
      markings more accurately.  In the best case, the new scheme should
      even allow reconstruction of the exact number of payload bytes
      that a CE-marked packet was carrying.  However, it is accepted
      that it may be too complex for a sender to get the exact number of
      congestion markings or marked bytes in all situations.  Ideally,
      the feedback scheme should preserve the order in which any (of the
      four) ECN signals were received.  And, ideally, it would even be
      possible for the sender to determine which of the packets covered
      by one delayed ACK were congestion marked, e.g., if the flow
      consists of packets of different sizes, or to allow for future
      protocols where the order of the markings may be important.

      In the best case, a sender that sees more accurate ECN feedback
      information would be able to reconstruct the occurrence of any of
      the four codepoints (Not-ECT, CE, ECT(0), ECT(1)).  However,
      assuming the sender marks all data packets as ECN-capable and uses
      a default setting of ECT(0) (as with [RFC3168]), solely feeding
      back the occurrence of CE and ECT(1) might be sufficient.  Because
      the sender can keep account of the transmitted segments with any
      of the three ECN codepoints, conveying any two of these back to
      the sender is sufficient for it to reconstruct the third as 
      observed by the receiver.  Thus, a more accurate ECN feedback
      scheme should at least provide information on two of these
      signals, e.g., CE and ECT(1).

      If a more accurate ECN scheme can reliably deliver feedback in
      most but not all circumstances, ideally the scheme should at least
      not introduce bias.  In other words, undetected loss of some ACKs
      should be as likely to increase as decrease the sender's estimate
      of the probability of ECN marking.

   Complexity
      Implementation should be as simple as possible, and only a minimum
      of additional state information should be needed.  This will
      enable more accurate ECN feedback to be used as the default
      feedback mechanism, even if only one ECN feedback signal per RTT
      is needed.

   Overhead
      A more accurate ECN feedback signal should limit the additional
      network load, because ECN feedback is ultimately not critical
      information (in the worst case, loss will still be available as a
      congestion signal of last resort).  As feedback information has to
      be provided frequently and in a timely fashion, potentially all or
      a large fraction of TCP acknowledgements might carry this
      information.  Ideally, no additional segments should be exchanged
      compared to a TCP session as specified in RFC 3168, and the
      overhead in each segment should be minimized.

   Backward and forward compatibility
      Given more accurate ECN feedback will involve a change to the TCP
      protocol, it should be negotiated between the two TCP endpoints.
      If either end does not support the more accurate feedback, they
      should both be able to fall back to classic ECN feedback.

      A more accurate ECN feedback extension should aim to traverse most
      middleboxes, including firewalls and Network Address Translators
      (NATs).  Further, a feedback mechanism should provide a method to
      fall back to classic ECN signalling if the new signal is
      suppressed by certain middleboxes.

      In order to avoid a fork in the TCP protocol specifications, if
      experiments with the new ECN feedback protocol are successful, the
      intention is to eventually update RFC 3168 for any TCP/ECN sender,
      not just for ConEx or DCTCP senders.  Then, future senders will be
      able to unilaterally deploy new behaviours that exploit the
      existence of more accurate ECN feedback in receivers (forward 
      compatibility).  Conversely, even if another sender only needs one
      ECN feedback signal per RTT, it should be able to use more
      accurate ECN feedback and simply ignore the excess information.

   Furthermore, the receiver should not make assumptions about the
   mechanism that was used to set the markings nor about any
   interpretation or reaction to the congestion signal.  The receiver
   only needs to faithfully reflect congestion information back to the
   sender.

### 5.1. Redefinition of ECN/NS Header Bits

   Schemes in this category can additionally use the NS bit for
   capability negotiation during the TCP handshake exchange.  Thus a
   more accurate ECN could be negotiated without changing the classic
   ECN negotiation and thus being backwards compatible.

   Schemes in this category can simply redefine the ECN header flags,
   ECE and CWR, to encode the occurrence of a CE marking at the
   receiver.  This approach provides very limited resilience against
   loss of ACK, particularly pure ACKs (no payload and therefore
   delivered unreliably). 
   A couple of schemes have been proposed so far:

   o  A naive 1-bit scheme that sends one ECE for each CE received could
      use CWR to increase robustness against ACK loss by introducing
      redundant information on the next ACK, but this is still
      vulnerable to ACK loss.

   o  The scheme defined for DCTCP [DCTCP ], which toggles the ECE
      feedback on an immediate ACK whenever the CE marking changes, and
      otherwise feeds back delayed ACKs with the ECE value unchanged. Appendix A  demonstrates that this scheme is still ambiguous to the
      sender if the ACKs are pure ACKs, and if some may have been lost.

   Alternatively, the receiver uses the three ECN/NS header flags, ECE,
   CWR, and NS, to represent a counter that signals the accumulated
   number of CE markings it has received.  Resilience against loss is
   better than the flag-based schemes but may not suffice in the
   presence of extended ACK loss that otherwise would not affect the TCP
   sender's performance.

   A number of coding schemes have been proposed so far in this
   category:

   o  A 3-bit counter scheme continuously feeds back the three least
      significant bits of a CE counter;

   o  A scheme that defines a standardized lookup table to map the eight
      codepoints onto either a CE counter or an ECT(1) counter.

   These proposed schemes provide accumulated information on CE marking
   feedback, similar to the number of acknowledged bytes in the TCP
   header.  Due to the limited number of bits, the ECN feedback
   information will wrap much more often than the acknowledgement field.
   Thus, feedback information could be lost due to a relatively small
   sequence of pure-ACK losses.  Resilience could be increased by
   introducing redundancy, e.g., send each counter increase two or more
   times.  Of course, any of these additional mechanisms will increase
   the complexity.  If the congestion rate is greater than the ACK rate
   (multiplied by the number of congestion marks that can be signaled
   per ACK), the congestion information cannot correctly be fed back.
   Covering the worst case (where every packet is CE marked) can
   potentially be realized by dynamically adapting the ACK rate and
   redundancy.  This again increases complexity and perhaps the
   signalling overhead as well.  Schemes that do not repurpose the ECN
   NS bit could still support the ECN Nonce. 





### 7.2. Informative References

   [DCTCP ]    Bensley, S., Eggert, L., and D. Thaler, "Microsoft's
              Datacenter TCP (DCTCP): TCP Congestion Control for
              Datacenters", Work in Progress,draft-bensley-tcpm-dctcp-05, July 2015.

   [ECN-BENEFITS ]
              Fairhurst, G. and M. Welzl, "The Benefits of using
              Explicit Congestion Notification (ECN)", Work in Progress draft-ietf-aqm-ecn-benefits-06, July 2015. 
   [RFC896]   Nagle, J., "Congestion Control in IP/TCP Internetworks",RFC 896, DOI 10.17487/RFC0896, January 1984,
              <http://www.rfc-editor.org/info/rfc896>.

   [RFC2018]  Mathis, M., Mahdavi, J., Floyd, S., and A. Romanow, "TCP
              Selective Acknowledgment Options", RFC 2018,
              DOI 10.17487/RFC2018, October 1996,
              <http://www.rfc-editor.org/info/rfc2018>.

   [RFC3449]  Balakrishnan, H., Padmanabhan, V., Fairhurst, G., and M.
              Sooriyabandara, "TCP Performance Implications of Network
              Path Asymmetry", BCP 69, RFC 3449, DOI 10.17487/RFC3449,
              December 2002, <http://www.rfc-editor.org/info/rfc3449>.

   [RFC5681]  Allman, M., Paxson, V., and E. Blanton, "TCP Congestion
              Control", RFC 5681, DOI 10.17487/RFC5681, September 2009,
              <http://www.rfc-editor.org/info/rfc5681>.

   [RFC5690]  Floyd, S., Arcia, A., Ros, D., and J. Iyengar, "Adding
              Acknowledgement Congestion Control to TCP", RFC 5690,
              DOI 10.17487/RFC5690, February 2010,
              <http://www.rfc-editor.org/info/rfc5690>.

   [RFC6093]  Gont, F. and A. Yourtchenko, "On the Implementation of the
              TCP Urgent Mechanism", RFC 6093, DOI 10.17487/RFC6093,
              January 2011, <http://www.rfc-editor.org/info/rfc6093>.

   [RFC6679]  Westerlund, M., Johansson, I., Perkins, C., O'Hanlon, P.,
              and K. Carlberg, "Explicit Congestion Notification (ECN)
              for RTP over UDP", RFC 6679, DOI 10.17487/RFC6679, August
              2012, <http://www.rfc-editor.org/info/rfc6679>.

   [RFC6789]  Briscoe, B., Ed., Woundy, R., Ed., and A. Cooper, Ed.,
              "Congestion Exposure (ConEx) Concepts and Use Cases",RFC 6789, DOI 10.17487/RFC6789, December 2012,
              <http://www.rfc-editor.org/info/rfc6789>.

   [SCTP-ECN ] Stewart, R., Tuexen, M., and X. Dong, "ECN for Stream
              Control Transmission Protocol (SCTP)", Work in Progress,draft-stewart-tsvwg-sctpecn-05, January 2014.

   [TEST-RCV ] Moncaster, T., Briscoe, B., and A. Jacquet, "A TCP Test to
              Allow Senders to Identify Receiver Non-Compliance", Work
              in Progress, draft-moncaster-tcpm-rcv-cheat-03, July 2014. 





# Referenced Sections from RFC 7713: Congestion Exposure (ConEx) Concepts, Abstract Mechanism, and Requirements

The following sections were referenced. Remaining sections are not included.

## 2. Overview

   As typical end-to-end transport protocols continually seek out more
   network capacity, network elements signal whenever congestion
   results, and the transports are responsible for controlling this
   network congestion [RFC5681].  The more a transport tries to use
   capacity that others want to use, the more congestion signals will be
   attributable to that transport.  Likewise, the more transport
   sessions sustained by a user and the longer the user sustains them,
   the more congestion signals will be attributable to that user.  The
   goal of ConEx is to ensure that the resulting congestion signals are
   sufficiently visible and robust, because they are an ideal metric for
   networks to use as the basis of traffic management or other related
   functions.

   Networks indicate congestion by three possible signals: packet loss,
   ECN marking, or queueing delay.  ECN marking and some packet loss may
   be the outcome of Active Queue Management (AQM), which the network
   uses to warn senders to reduce their rates.  Packet loss is also the
   natural consequence of complete exhaustion of a buffer or other
   network resource.  Some experimental transport protocols and TCP
   variants infer impending congestion from increasing queuing delay.
   However, delay is too amorphous to use as a congestion metric.  In
   this and other ConEx documents, the term 'congestion signals' is
   generally used solely for ECN markings and packet losses because they
   are unambiguous signals of congestion. 
   In both cases, the congestion signals follow the route indicated in
   Figure 1.  A congested network device sends a signal in the data
   stream on the forward path to the transport receiver, the receiver
   passes it back to the sender through transport-level feedback, and
   the sender makes some congestion control adjustment.

   This document extends the capabilities of the Internet protocol suite
   with the addition of a new Congestion Exposure signal.  To a first
   approximation, this signal (also shown in Figure 1) relays the
   congestion information from the transport sender back through the
   internetwork layer where it is visible to any interested
   internetwork-layer devices along the forward path.  This document
   frames the engineering problem of designing the ConEx Signal.  The
   requirements are described in Section 3 and some example encodings
   are presented in Section 4.  Section 5 describes all of the protocol
   components.

   This new signal is expressly designed to support a variety of new
   policy mechanisms that might be used to instrument, monitor, or
   manage traffic.  The policy devices are not shown in Figure 1 but
   might be placed anywhere along the forward data path (see Section 5.4).

   ,---------.                                               ,---------.
   |Transport|                                               |Transport|
   | Sender  |   .                                           |Receiver |
   |         |  /|___________________________________________|         |
   |     ,-<---------------Congestion-Feedback-Signals--<--------.     |
   |     |   |/                                              |   |     |
   |     |   |\           Transport Layer Feedback Flow      |   |     |
   |     |   | \  ___________________________________________|   |     |
   |     |   |  \|                                           |   |     |
   |     |   |   '         ,-----------.               .     |   |     |
   |     |   |_____________|           |_______________|\    |   |     |
   |     |   |    IP Layer |           |  Data Flow      \   |   |     |
   |     |   |             |(Congested)|                  \  |   |     |
   |     |   |             |  Network  |--Congestion-Signals--->-'     |
   |     |   |             |  Device   |                    \|         |
   |     |   |             |           |                    /|         |
   |     `----------->--(new)-IP-Layer-ConEx-Signals-------->|         |
   |         |             |           |                  /  |         |
   |         |_____________|           |_______________  /   |         |
   |         |             |           |               |/    |         |
   `---------'             `-----------'               '     `---------'

            Figure 1: The Flow of Congestion and ConEx Signals 
   Since the policy devices can affect how traffic is treated, it is
   assumed that there is an intrinsic motivation for users,
   applications, or operating systems to understate the congestion that
   they are causing.  Therefore, it is important to be able to audit
   ConEx Signals and to be able to apply sufficient sanction to
   discourage cheating of congestion policies.  The general approach to
   auditing is to count signals on the forward path to confirm that
   there are never fewer ConEx Signals than congestion signals.  Many
   ConEx design constraints come from the need to assure that the audit
   function is sufficiently robust.  The audit function is described in Section 5.5; however, significant portions of this document (and
   prior research [Refb-dis ]) are motivated by issues relating to the
   audit function and making it robust.

   The congestion and ConEx Signals shown in Figure 1 represent a series
   of discrete events: ECN marks or lost packets, carried by the forward
   data stream and fed back into the internetwork layer.  The policy and
   audit functions are most likely to act on the accumulated values of
   these signals, for which we use the term "volume".  For example,
   "traffic volume" is the total number of bytes delivered optionally
   over a specified time interval and over some aggregate of traffic
   (e.g., all traffic from a site), while "loss volume" is the total
   amount of bytes discarded from some aggregate over an interval.  The
   term "congestion-volume" is defined precisely in [RFC6789].  Note
   that volume per unit time is average rate.

   A design goal of the ConEx protocol is that the important policy
   mechanisms can be implemented per logical link without per-flow state
   (see Section 5.4).  However, the trade-off is that per-flow state
   could be needed to audit ConEx Signals (Section 5.5).  This is
   justified in that i) auditing at the edges, with a limited number of
   flows, enables policy elsewhere, including in the core, without any
   per-flow state; ii) auditing can use soft flow state, which does not
   require route pinning.

   There is a long standing argument over units of congestion: bytes vs
   packets (see [RFC7141] and its references).  Section 4.6 explains why
   this problem must be addressed carefully.  However, this document
   does not take a strong position on this issue.  Nonetheless, it does
   require that the units of congestion must be an explicitly stated
   property of any proposed encoding, and the consequences of that
   design decision must be evaluated along with other aspects of the
   design.

   To be successful, the ConEx protocol needs to have the property that
   the relevant stakeholders each have the incentive to unilaterally
   start on each stage of partial deployment, which in turn creates 
   incentives for further deployment.  Furthermore, legacy systems that
   will never be upgraded do not become a barrier to deploying ConEx.
   Issues relating to partial deployment are described in Section 6.

   Note that ConEx Signals are not intended to be used for fine-grained
   congestion control.  They are anticipated to be most useful at longer
   time scales and/or at coarser granularity than single microflows.
   For example, the total congestion caused by a user might serve as an
   input to higher-level policy or accountability functions designed to
   create incentives for improving user behavior, such as choosing to
   send large quantities of data at off-peak times, at lower data rates,
   or with less aggressive protocols such as Low Extra Delay Background
   Transport (LEDBAT) [RFC6817]; see [RFC6789].

   Ultimately, ConEx Signals have the potential to provide a mechanism
   to regulate global Internet congestion.  From the earliest days of
   research on congestion control, there has been a concern that there
   is no mechanism to prevent transport designers from incrementally
   making protocols more aggressive without bound and spiraling to a
   "tragedy of the commons" Internet congestion collapse.  The "TCP
   friendly" paradigm was created in part to forestall this failure.
   However, it no longer commands any authority because it has little to
   say about the Internet of today, which has moved beyond the scaling
   range of standard TCP.  As a consequence, many transports and
   applications are opening arbitrarily large numbers of connections or
   using arbitrary levels of aggressiveness.  ConEx represents a
   recognition that the IETF cannot regulate this space directly because
   it concerns the behaviour of users and applications, not individual
   transport protocols.  Instead, the IETF can give network operators
   the protocol tools to arbitrate the space themselves with better bulk
   traffic management.  This, in turn, should create incentives for
   users and designers of applications and of transport protocols to be
   more mindful about contributing to congestion.

### 2.1. Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in RFC 2119 [RFC2119].

   ConEx Signals in IP packet headers from the sender to the network:

   Not-ConEx:  The transport (or at least this packet) is not using
      ConEx.

   ConEx-Capable:  The transport is using ConEx.  This is the opposite
      of Not-ConEx. 
   ConEx Signal:  A signal in a packet sent by a ConEx-capable
      transport.  It carries at least one of the following signals:

      Re-Echo-Loss:  The transport has experienced a loss.

      Re-Echo-ECN:  The transport has detected an ECN Congestion
         Experienced (CE) mark.

      Credit:  The transport is building up credit to signal advance
         notice of the risk of packets contributing to congestion, in
         contrast to signalling only after inherently delayed feedback
         of actual congestion.

      ConEx-Not-Marked:  The transport is ConEx-capable but is not
         signaling Re-Echo-Loss, Re-Echo-ECN, or Credit.

   ConEx-Marked:  At least one of Re-Echo-Loss, Re-Echo-ECN, or Credit.

   ConEx-Re-Echo:  At least one of Re-Echo-Loss or Re-Echo-ECN.

### 5.2. Modified Senders

   The sending transport needs to be modified to send Congestion
   Exposure signals in response to congestion feedback signals (e.g.,
   for the case of a TCP transport, see [TCP-MODIFICATION ]).  We want to
   permit ConEx without ECN (e.g., if the receiver does not support
   ECN).  However, we want to encourage a ConEx sender to at least
   attempt to negotiate ECN (a ConEx transport protocol specification
   may require this) because it is believed that ConEx without ECN is
   harder to audit and thus potentially exposed to cheating.  Since
   honest users have the potential to benefit from stronger mechanisms 
   to manage traffic, they have an incentive to deploy ConEx and ECN
   together.  This incentive is not sufficient to prevent a dishonest
   user from constructing (or configuring) a sender that enables ConEx
   after choosing not to negotiate ECN, but it should be sufficient to
   prevent this from being the sustained default case for any
   significant pool of users.

   Permitting ConEx without ECN is necessary to facilitate bootstrapping
   other parts of ConEx deployment.

# Referenced Sections from RFC 8257: Data Center TCP (DCTCP): TCP Congestion Control for Data Centers

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Large data centers necessarily need many network switches to
   interconnect their many servers.  Therefore, a data center can
   greatly reduce its capital expenditure by leveraging low-cost
   switches.  However, such low-cost switches tend to have limited queue
   capacities; thus, they are more susceptible to packet loss due to
   congestion.

   Network traffic in a data center is often a mix of short and long
   flows, where the short flows require low latencies and the long flows
   require high throughputs.  Data centers also experience incast
   bursts, where many servers send traffic to a single server at the
   same time.  For example, this traffic pattern is a natural
   consequence of the MapReduce [MAPREDUCE ] workload: the worker nodes
   complete at approximately the same time, and all reply to the master
   node concurrently.

   These factors place some conflicting demands on the queue occupancy
   of a switch:

   o  The queue must be short enough that it does not impose excessive
      latency on short flows.

   o  The queue must be long enough to buffer sufficient data for the
      long flows to saturate the path capacity.

   o  The queue must be long enough to absorb incast bursts without
      excessive packet loss.

   Standard TCP congestion control [RFC5681] relies on packet loss to
   detect congestion.  This does not meet the demands described above.
   First, short flows will start to experience unacceptable latencies
   before packet loss occurs.  Second, by the time TCP congestion
   control kicks in on the senders, most of the incast burst has already
   been dropped.

   [RFC3168] describes a mechanism for using Explicit Congestion
   Notification (ECN) from the switches for detection of congestion.
   However, this method only detects the presence of congestion, not its
   extent.  In the presence of mild congestion, the TCP congestion
   window is reduced too aggressively, and this unnecessarily reduces
   the throughput of long flows.

   Data Center TCP (DCTCP) changes traditional ECN processing by
   estimating the fraction of bytes that encounter congestion rather
   than simply detecting that some congestion has occurred.  DCTCP then
   scales the TCP congestion window based on this estimate.  This method 
   achieves high-burst tolerance, low latency, and high throughput with
   shallow-buffered switches.  DCTCP is a modification to the processing
   of ECN by a conventional TCP and requires that standard TCP
   congestion control be used for handling packet loss.

   DCTCP should only be deployed in an intra-data-center environment
   where both endpoints and the switching fabric are under a single
   administrative domain.  DCTCP MUST NOT be deployed over the public
   Internet without additional measures, as detailed in Section 5.

   The objective of this Informational RFC is to document DCTCP as a new
   approach (which is known to be widely implemented and deployed) to
   address TCP congestion control in data centers.  The IETF TCPM
   Working Group reached consensus regarding the fact that a DCTCP
   standard would require further work.  A precise documentation of
   running code enables follow-up Experimental or Standards Track RFCs
   through the IETF stream.

   This document describes DCTCP as implemented in Microsoft Windows
   Server 2012 [WINDOWS ].  The Linux [LINUX ] and FreeBSD [FREEBSD ]
   operating systems have also implemented support for DCTCP in a way
   that is believed to follow this document.  Deployment experiences
   with DCTCP have been documented in [MORGANSTANLEY ].

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

## 5. Deployment Issues

   DCTCP and conventional TCP congestion control do not coexist well in
   the same network.  In typical DCTCP deployments, the marking
   threshold in the switching fabric is set to a very low value to
   reduce queueing delay, and a relatively small amount of congestion
   will exceed the marking threshold.  During such periods of
   congestion, conventional TCP will suffer packet loss and quickly and
   drastically reduce cwnd.  DCTCP, on the other hand, will use the
   fraction of marked packets to reduce cwnd more gradually.  Thus, the
   rate reduction in DCTCP will be much slower than that of conventional
   TCP, and DCTCP traffic will gain a larger share of the capacity
   compared to conventional TCP traffic traversing the same path.  If
   the traffic in the data center is a mix of conventional TCP and
   DCTCP, it is RECOMMENDED that DCTCP traffic be segregated from
   conventional TCP traffic.  [MORGANSTANLEY ] describes a deployment
   that uses the IP Differentiated Services Codepoint (DSCP) bits to
   segregate the network such that Active Queue Management (AQM)
   [RFC7567] is applied to DCTCP traffic, whereas TCP traffic is managed
   via drop-tail queueing.

   Deployments should take into account segregation of non-TCP traffic
   as well.  Today's commodity switches allow configuration of different
   marking/drop profiles for non-TCP and non-IP packets.  Non-TCP and
   non-IP packets should be able to pass through such switches, unless
   they really run out of buffer space.

   Since DCTCP relies on congestion marking by the switches, DCTCP's
   potential can only be realized in data centers where the entire
   network infrastructure supports ECN.  The switches may also support
   configuration of the congestion threshold used for marking.  The
   proposed parameterization can be configured with switches that
   implement Random Early Detection (RED) [RFC2309].  [DCTCP10] provides
   a theoretical basis for selecting the congestion threshold, but, as
   with the estimation gain, it may be more practical to rely on
   experimentation or simply to use the default configuration of the
   device.  DCTCP will revert to loss-based congestion control when
   packet loss is experienced (e.g., when transiting a congested drop-
   tail link, or a link with an AQM drop behavior).

   DCTCP requires changes on both the sender and the receiver, so both
   endpoints must support DCTCP.  Furthermore, DCTCP provides no
   mechanism for negotiating its use, so both endpoints must be
   configured through some out-of-band mechanism to use DCTCP.  A
   variant of DCTCP that can be deployed unilaterally and that only
   requires standard ECN behavior has been described in [ODCTCP ] and
   [BSDCAN ], but it requires additional experimental evaluation. 





# Referenced Sections from RFC 9330: Low Latency, Low Loss, and Scalable Throughput (L4S) Internet Service: Architecture

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   At any one time, it is increasingly common for all of the traffic in
   a bottleneck link (e.g., a household's Internet access or Wi-Fi) to
   come from applications that prefer low delay: interactive web, web
   services, voice, conversational video, interactive video, interactive
   remote presence, instant messaging, online and cloud-rendered gaming,
   remote desktop, cloud-based applications, cloud-rendered virtual
   reality or augmented reality, and video-assisted remote control of
   machinery and industrial processes.  In the last decade or so, much
   has been done to reduce propagation delay by placing caches or
   servers closer to users.  However, queuing remains a major, albeit
   intermittent, component of latency.  For instance, spikes of hundreds
   of milliseconds are not uncommon, even with state-of-the-art Active
   Queue Management (AQM) [COBALT ] [DOCSIS3AQM ].  A Classic AQM in an
   access network bottleneck is typically configured to buffer the
   sawteeth of lone flows, which can cause peak overall network delay to
   roughly double during a long-running flow, relative to expected base
   (unloaded) path delay [BufferSize ].  Low loss is also important
   because, for interactive applications, losses translate into even
   longer retransmission delays.

   It has been demonstrated that, once access network bit rates reach
   levels now common in the developed world, increasing link capacity
   offers diminishing returns if latency (delay) is not addressed
   [Dukkipati06] [Rajiullah15].  Therefore, the goal is an Internet
   service with very low queuing latency, very low loss, and scalable
   throughput.  Very low queuing latency means less than 1 millisecond
   (ms) on average and less than about 2 ms at the 99th percentile.
   End-to-end delay above 50 ms [Raaen14], or even above 20 ms [NASA04],
   starts to feel unnatural for more demanding interactive applications.
   Therefore, removing unnecessary delay variability increases the reach
   of these applications (the distance over which they are comfortable
   to use) and/or provides additional latency budget that can be used
   for enhanced processing.  This document describes the L4S
   architecture for achieving these goals.

   Differentiated services (Diffserv) offers Expedited Forwarding (EF)
   [RFC3246] for some packets at the expense of others, but this makes
   no difference when all (or most) of the traffic at a bottleneck at
   any one time requires low latency.  In contrast, L4S still works well
   when all traffic is L4S -- a service that gives without taking needs
   none of the configuration or management baggage (traffic policing or
   traffic contracts) associated with favouring some traffic flows over
   others.

   Queuing delay degrades performance intermittently [Hohlfeld14].  It
   occurs i) when a large enough capacity-seeking (e.g., TCP) flow is
   running alongside the user's traffic in the bottleneck link, which is
   typically in the access network, or ii) when the low latency
   application is itself a large capacity-seeking or adaptive rate flow
   (e.g., interactive video).  At these times, the performance
   improvement from L4S must be sufficient for network operators to be
   motivated to deploy it.

   Active Queue Management (AQM) is part of the solution to queuing
   under load.  AQM improves performance for all traffic, but there is a
   limit to how much queuing delay can be reduced by solely changing the
   network without addressing the root of the problem.

   The root of the problem is the presence of standard congestion
   control (Reno [RFC5681]) or compatible variants (e.g., CUBIC
   [RFC8312]) that are used in TCP and in other transports, such as QUIC
   [RFC9000].  We shall use the term 'Classic' for these Reno-friendly
   congestion controls.  Classic congestion controls induce relatively
   large sawtooth-shaped excursions of queue occupancy.  So if a network
   operator naively attempts to reduce queuing delay by configuring an
   AQM to operate at a shallower queue, a Classic congestion control
   will significantly underutilize the link at the bottom of every
   sawtooth.  These sawteeth have also been growing in duration as flow
   rate scales (see Section 5.1 and [RFC3649]).

   It has been demonstrated that, if the sending host replaces a Classic
   congestion control with a 'Scalable' alternative, the performance
   under load of all the above interactive applications can be
   significantly improved once a suitable AQM is deployed in the
   network.  Taking the example solution cited below that uses Data
   Center TCP (DCTCP) [RFC8257] and a Dual-Queue Coupled AQM [RFC9332]
   on a DSL or Ethernet link, queuing delay under heavy load is roughly
   1-2 ms at the 99th percentile without losing link utilization
   [L4Seval22] [DualPI2Linux ] (for other link types, see Section 6.3).
   This compares with 5-20 ms on _average_ with a Classic congestion
   control and current state-of-the-art AQMs, such as Flow Queue CoDel
   [RFC8290], Proportional Integral controller Enhanced (PIE) [RFC8033],
   or DOCSIS PIE [RFC8034] and about 20-30 ms at the 99th percentile
   [DualPI2Linux ].

   L4S is designed for incremental deployment.  It is possible to deploy
   the L4S service at a bottleneck link alongside the existing best
   efforts service [DualPI2Linux ] so that unmodified applications can
   start using it as soon as the sender's stack is updated.  Access
   networks are typically designed with one link as the bottleneck for
   each site (which might be a home, small enterprise, or mobile
   device), so deployment at either or both ends of this link should
   give nearly all the benefit in the respective direction.  With some
   transport protocols, namely TCP [ACCECN ], the sender has to check
   that the receiver has been suitably updated to give more accurate
   feedback, whereas with more recent transport protocols, such as QUIC
   [RFC9000] and Datagram Congestion Control Protocol (DCCP) [RFC4340],
   all receivers have always been suitable.

   This document presents the L4S architecture.  It consists of three
   components: network support to isolate L4S traffic from Classic
   traffic; protocol features that allow network elements to identify
   L4S traffic; and host support for L4S congestion controls.  The
   protocol is defined separately in [RFC9331] as an experimental change
   to Explicit Congestion Notification (ECN).  This document describes
   and justifies the component parts and how they interact to provide
   the low latency, low loss, and scalable Internet service.  It also
   details the approach to incremental deployment, as briefly summarized
   above.

### 1.1. Document Roadmap

   This document describes the L4S architecture in three passes.  First,
   the brief overview in Section 2 gives the very high-level idea and
   states the main components with minimal rationale.  This is only
   intended to give some context for the terminology definitions that
   follow in Section 3 and to explain the structure of the rest of the
   document.  Then, Section 4 goes into more detail on each component
   with some rationale but still mostly stating what the architecture
   is, rather than why.  Finally, Section 5 justifies why each element
   of the solution was chosen (Section 5.1) and why these choices were
   different from other solutions (Section 5.2).

   After the architecture has been described, Section 6 clarifies its
   applicability by describing the applications and use cases that
   motivated the design, the challenges applying the architecture to
   various link technologies, and various incremental deployment models
   (including the two main deployment topologies, different sequences
   for incremental deployment, and various interactions with preexisting
   approaches).  The document ends with the usual tailpieces, including
   extensive discussion of traffic policing and other security
   considerations in Section 8.

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

### 5.1. Why These Primary Components?

   Explicit congestion signalling (protocol):  Explicit congestion
      signalling is a key part of the L4S approach.  In contrast, use of
      drop as a congestion signal creates tension because drop is both
      an impairment (less would be better) and a useful signal (more
      would be better):

      *  Explicit congestion signals can be used many times per round
         trip to keep tight control without any impairment.  Under heavy
         load, even more explicit signals can be applied so that the
         queue can be kept short whatever the load.  In contrast,
         Classic AQMs have to introduce very high packet drop at high
         load to keep the queue short.  By using ECN, an L4S congestion
         control's sawtooth reduction can be smaller and therefore
         return to the operating point more often, without worrying that
         more sawteeth will cause more signals.  The consequent smaller
         amplitude sawteeth fit between an empty queue and a very
         shallow marking threshold (~1 ms in the public Internet), so
         queue delay variation can be very low, without risk of
         underutilization.

      *  Explicit congestion signals can be emitted immediately to track
         fluctuations of the queue.  L4S shifts smoothing from the
         network to the host.  The network doesn't know the round-trip
         times (RTTs) of any of the flows.  So if the network is
         responsible for smoothing (as in the Classic approach), it has
         to assume a worst case RTT, otherwise long RTT flows would
         become unstable.  This delays Classic congestion signals by
         100-200 ms.  In contrast, each host knows its own RTT.  So, in
         the L4S approach, the host can smooth each flow over its own
         RTT, introducing no more smoothing delay than strictly
         necessary (usually only a few milliseconds).  A host can also
         choose not to introduce any smoothing delay if appropriate,
         e.g., during flow start-up.

      Neither of the above are feasible if explicit congestion
      signalling has to be considered 'equivalent to drop' (as was
      required with Classic ECN [RFC3168]), because drop is an
      impairment as well as a signal.  So drop cannot be excessively
      frequent, and drop cannot be immediate; otherwise, too many drops
      would turn out to have been due to only a transient fluctuation in
      the queue that would not have warranted dropping a packet in
      hindsight.  Therefore, in an L4S AQM, the L4S queue uses a new L4S
      variant of ECN that is not equivalent to drop (see Section 5.2 of
      the L4S ECN spec [RFC9331]), while the Classic queue uses either
      Classic ECN [RFC3168] or drop, which are still equivalent to each
      other.

      Before Classic ECN was standardized, there were various proposals
      to give an ECN mark a different meaning from drop.  However, there
      was no particular reason to agree on any one of the alternative
      meanings, so 'equivalent to drop' was the only compromise that
      could be reached.  [RFC3168] contains a statement that:

         An environment where all end nodes were ECN-Capable could
          allow new criteria to be developed for setting the CE
          codepoint, and new congestion control mechanisms for end-node
          reaction to CE packets.  However, this is a research issue,
          and as such is not addressed in this document.

   Latency isolation (network):  L4S congestion controls keep queue
      delay low, whereas Classic congestion controls need a queue of the
      order of the RTT to avoid underutilization.  One queue cannot have
      two lengths; therefore, L4S traffic needs to be isolated in a
      separate queue (e.g., DualQ) or queues (e.g., FQ).

   Coupled congestion notification:  Coupling the congestion
      notification between two queues as in the DualQ Coupled AQM is not
      necessarily essential, but it is a simple way to allow senders to
      determine their rate packet by packet, rather than be overridden
      by a network scheduler.  An alternative is for a network scheduler
      to control the rate of each application flow (see the discussion
      in Section 5.2).

   L4S packet identifier (protocol):  Once there are at least two
      treatments in the network, hosts need an identifier at the IP
      layer to distinguish which treatment they intend to use.

   Scalable congestion notification:  A Scalable congestion control in
      the host keeps the signalling frequency from the network high,
      whatever the flow rate, so that queue delay variations can be
      small when conditions are stable, and rate can track variations in
      available capacity as rapidly as possible otherwise.

   Low loss:  Latency is not the only concern of L4S.  The 'Low Loss'
      part of the name denotes that L4S generally achieves zero
      congestion loss due to its use of ECN.  Otherwise, loss would
      itself cause delay, particularly for short flows, due to
      retransmission delay [RFC2884].

   Scalable throughput:  The 'Scalable throughput' part of the name
      denotes that the per-flow throughput of Scalable congestion
      controls should scale indefinitely, avoiding the imminent scaling
      problems with Reno-friendly congestion control algorithms
      [RFC3649].  It was known when TCP congestion avoidance was first
      developed in 1988 that it would not scale to high bandwidth-delay
      products (see footnote 6 in [TCP-CA ]).  Today, regular broadband
      flow rates over WAN distances are already beyond the scaling range
      of Classic Reno congestion control.  So 'less unscalable' CUBIC
      [RFC8312] and Compound [CTCP ] variants of TCP have been
      successfully deployed.  However, these are now approaching their
      scaling limits.

      For instance, we will consider a scenario with a maximum RTT of 30
      ms at the peak of each sawtooth.  As Reno packet rate scales 8
      times from 1,250 to 10,000 packet/s (from 15 to 120 Mb/s with 1500
      B packets), the time to recover from a congestion event rises
      proportionately by 8 times as well, from 422 ms to 3.38 s.  It is
      clearly problematic for a congestion control to take multiple
      seconds to recover from each congestion event.  CUBIC [RFC8312]
      was developed to be less unscalable, but it is approaching its
      scaling limit; with the same max RTT of 30 ms, at 120 Mb/s, CUBIC
      is still fully in its Reno-friendly mode, so it takes about 4.3 s
      to recover.  However, once flow rate scales by 8 times again to
      960 Mb/s it enters true CUBIC mode, with a recovery time of 12.2
      s.  From then on, each further scaling by 8 times doubles CUBIC's
      recovery time (because the cube root of 8 is 2), e.g., at 7.68 Gb/
      s, the recovery time is 24.3 s.  In contrast, a Scalable
      congestion control like DCTCP or Prague induces 2 congestion
      signals per round trip on average, which remains invariant for any
      flow rate, keeping dynamic control very tight.

      For a feel of where the global average lone-flow download sits on
      this scale at the time of writing (2021), according to [BDPdata ],
      the global average fixed access capacity was 103 Mb/s in 2020 and
      the average base RTT to a CDN was 25 to 34 ms in 2019.  Averaging
      of per-country data was weighted by Internet user population (data
      collected globally is necessarily of variable quality, but the
      paper does double-check that the outcome compares well against a
      second source).  So a lone CUBIC flow would at best take about 200
      round trips (5 s) to recover from each of its sawtooth reductions,
      if the flow even lasted that long.  This is described as 'at best'
      because it assumes everyone uses an AQM, whereas in reality, most
      users still have a (probably bloated) tail-drop buffer.  In the
      tail-drop case, the likely average recovery time would be at least
      4 times 5 s, if not more, because RTT under load would be at least
      double that of an AQM, and the recovery time of Reno-friendly
      flows depends on the square of RTT.

      Although work on scaling congestion controls tends to start with
      TCP as the transport, the above is not intended to exclude other
      transports (e.g., SCTP and QUIC) or less elastic algorithms (e.g.,
      RMCAT), which all tend to adopt the same or similar developments.

### 5.2. What L4S Adds to Existing Approaches

   All the following approaches address some part of the same problem
   space as L4S.  In each case, it is shown that L4S complements them or
   improves on them, rather than being a mutually exclusive alternative:

   Diffserv:  Diffserv addresses the problem of bandwidth apportionment
      for important traffic as well as queuing latency for delay-
      sensitive traffic.  Of these, L4S solely addresses the problem of
      queuing latency.  Diffserv will still be necessary where important
      traffic requires priority (e.g., for commercial reasons or for
      protection of critical infrastructure traffic) -- see
      [L4S-DIFFSERV ].  Nonetheless, the L4S approach can provide low
      latency for all traffic within each Diffserv class (including the
      case where there is only the one default Diffserv class).

      Also, Diffserv can only provide a latency benefit if a small
      subset of the traffic on a bottleneck link requests low latency.
      As already explained, it has no effect when all the applications
      in use at one time at a single site (e.g., a home, small business,
      or mobile device) require low latency.  In contrast, because L4S
      works for all traffic, it needs none of the management baggage
      (traffic policing or traffic contracts) associated with favouring
      some packets over others.  This lack of management baggage ought
      to give L4S a better chance of end-to-end deployment.

      In particular, if networks do not trust end systems to identify
      which packets should be favoured, they assign packets to Diffserv
      classes themselves.  However, the techniques available to such
      networks, like inspection of flow identifiers or deeper inspection
      of application signatures, do not always sit well with encryption
      of the layers above IP [RFC8404].  In these cases, users can have
      either privacy or Quality of Service (QoS), but not both.

      As with Diffserv, the L4S identifier is in the IP header.  But, in
      contrast to Diffserv, the L4S identifier does not convey a want or
      a need for a certain level of quality.  Rather, it promises a
      certain behaviour (Scalable congestion response), which networks
      can objectively verify if they need to.  This is because low delay
      depends on collective host behaviour, whereas bandwidth priority
      depends on network behaviour.

   State-of-the-art AQMs:  AQMs for Classic traffic, such as PIE and FQ-
      CoDel, give a significant reduction in queuing delay relative to
      no AQM at all.  L4S is intended to complement these AQMs and
      should not distract from the need to deploy them as widely as
      possible.  Nonetheless, AQMs alone cannot reduce queuing delay too
      far without significantly reducing link utilization, because the
      root cause of the problem is on the host -- where Classic
      congestion controls use large sawtoothing rate variations.  The
      L4S approach resolves this tension between delay and utilization
      by enabling hosts to minimize the amplitude of their sawteeth.  A
      single-queue Classic AQM is not sufficient to allow hosts to use
      small sawteeth for two reasons: i) smaller sawteeth would not get
      lower delay in an AQM designed for larger amplitude Classic
      sawteeth, because a queue can only have one length at a time and
      ii) much smaller sawteeth implies much more frequent sawteeth, so
      L4S flows would drive a Classic AQM into a high level of ECN-
      marking, which would appear as heavy congestion to Classic flows,
      which in turn would greatly reduce their rate as a result (see Section 6.4.4).

   Per-flow queuing or marking:  Similarly, per-flow approaches, such as
      FQ-CoDel or Approx Fair CoDel [AFCD ], are not incompatible with
      the L4S approach.  However, per-flow queuing alone is not enough
      -- it only isolates the queuing of one flow from others, not from
      itself.  Per-flow implementations need to have support for
      Scalable congestion control added, which has already been done for
      FQ-CoDel in Linux (see Section 5.2.7 of [RFC8290] and
      [FQ_CoDel_Thresh ]).  Without this simple modification, per-flow
      AQMs, like FQ-CoDel, would still not be able to support
      applications that need both very low delay and high bandwidth,
      e.g., video-based control of remote procedures or interactive
      cloud-based video (see Note 1 below).

      Although per-flow techniques are not incompatible with L4S, it is
      important to have the DualQ alternative.  This is because handling
      end-to-end (layer 4) flows in the network (layer 3 or 2) precludes
      some important end-to-end functions.  For instance:

      a.  Per-flow forms of L4S, like FQ-CoDel, are incompatible with
          full end-to-end encryption of transport layer identifiers for
          privacy and confidentiality (e.g., IPsec or encrypted VPN
          tunnels, as opposed to DTLS over UDP), because they require
          packet inspection to access the end-to-end transport flow
          identifiers.

          In contrast, the DualQ form of L4S requires no deeper
          inspection than the IP layer.  So as long as operators take
          the DualQ approach, their users can have both very low queuing
          delay and full end-to-end encryption [RFC8404].

      b.  With per-flow forms of L4S, the network takes over control of
          the relative rates of each application flow.  Some see it as
          an advantage that the network will prevent some flows running
          faster than others.  Others consider it an inherent part of
          the Internet's appeal that applications can control their rate
          while taking account of the needs of others via congestion
          signals.  They maintain that this has allowed applications
          with interesting rate behaviours to evolve, for instance: i) a
          variable bit-rate video that varies around an equal share,
          rather than being forced to remain equal at every instant or
          ii) end-to-end scavenger behaviours [RFC6817] that use less
          than an equal share of capacity [LEDBAT_AQM ].

          The L4S architecture does not require the IETF to commit to
          one approach over the other, because it supports both so that
          the 'market' can decide.  Nonetheless, in the spirit of 'Do
          one thing and do it well' [McIlroy78], the DualQ option
          provides low delay without prejudging the issue of flow-rate
          control.  Then, flow rate policing can be added separately if
          desired.  In contrast to scheduling, a policer would allow
          application control up to a point, but the network would still
          be able to set the point at which it intervened to prevent one
          flow completely starving another.

      Note:

      1.  It might seem that self-inflicted queuing delay within a per-
          flow queue should not be counted, because if the delay wasn't
          in the network, it would just shift to the sender.  However,
          modern adaptive applications, e.g., HTTP/2 [RFC9113] or some
          interactive media applications (see Section 6.1), can keep low
          latency objects at the front of their local send queue by
          shuffling priorities of other objects dependent on the
          progress of other transfers (for example, see [lowat ]).  They
          cannot shuffle objects once they have released them into the
          network.

   Alternative Back-off ECN (ABE):  Here again, L4S is not an
      alternative to ABE but a complement that introduces much lower
      queuing delay.  ABE [RFC8511] alters the host behaviour in
      response to ECN marking to utilize a link better and give ECN
      flows faster throughput.  It uses ECT(0) and assumes the network
      still treats ECN and drop the same.  Therefore, ABE exploits any
      lower queuing delay that AQMs can provide.  But, as explained
      above, AQMs still cannot reduce queuing delay too much without
      losing link utilization (to allow for other, non-ABE, flows).

   BBR:  Bottleneck Bandwidth and Round-trip propagation time (BBR)
      [BBR-CC ] controls queuing delay end-to-end without needing any
      special logic in the network, such as an AQM.  So it works pretty
      much on any path.  BBR keeps queuing delay reasonably low, but
      perhaps not quite as low as with state-of-the-art AQMs, such as
      PIE or FQ-CoDel, and certainly nowhere near as low as with L4S.
      Queuing delay is also not consistently low, due to BBR's regular
      bandwidth probing spikes and its aggressive flow start-up phase.

      L4S complements BBR.  Indeed, BBRv2 can use L4S ECN where
      available and a Scalable L4S congestion control behaviour in
      response to any ECN signalling from the path [BBRv2].  The L4S ECN
      signal complements the delay-based congestion control aspects of
      BBR with an explicit indication that hosts can use, both to
      converge on a fair rate and to keep below a shallow queue target
      set by the network.  Without L4S ECN, both these aspects need to
      be assumed or estimated.

## 6. Applicability





### 6.1. Applications

   A transport layer that solves the current latency issues will provide
   new service, product, and application opportunities.

   With the L4S approach, the following existing applications also
   experience significantly better quality of experience under load:

   *  gaming, including cloud-based gaming;

   *  VoIP;

   *  video conferencing;

   *  web browsing;

   *  (adaptive) video streaming; and

   *  instant messaging.

   The significantly lower queuing latency also enables some interactive
   application functions to be offloaded to the cloud that would hardly
   even be usable today, including:

   *  cloud-based interactive video and

   *  cloud-based virtual and augmented reality.

   The above two applications have been successfully demonstrated with
   L4S, both running together over a 40 Mb/s broadband access link
   loaded up with the numerous other latency-sensitive applications in
   the previous list, as well as numerous downloads, with all sharing
   the same bottleneck queue simultaneously [L4Sdemo16]
   [L4Sdemo16-Video ].  For the former, a panoramic video of a football
   stadium could be swiped and pinched so that, on the fly, a proxy in
   the cloud could generate a sub-window of the match video under the
   finger-gesture control of each user.  For the latter, a virtual
   reality headset displayed a viewport taken from a 360-degree camera
   in a racing car.  The user's head movements controlled the viewport
   extracted by a cloud-based proxy.  In both cases, with a 7 ms end-to-
   end base delay, the additional queuing delay of roughly 1 ms was so
   low that it seemed the video was generated locally.

   Using a swiping finger gesture or head movement to pan a video are
   extremely latency-demanding actions -- far more demanding than VoIP
   -- because human vision can detect extremely low delays of the order
   of single milliseconds when delay is translated into a visual lag
   between a video and a reference point (the finger or the orientation
   of the head sensed by the balance system in the inner ear, i.e., the
   vestibular system).  With an alternative AQM, the video noticeably
   lagged behind the finger gestures and head movements.

   Without the low queuing delay of L4S, cloud-based applications like
   these would not be credible without significantly more access-network
   bandwidth (to deliver all possible areas of the video that might be
   viewed) and more local processing, which would increase the weight
   and power consumption of head-mounted displays.  When all interactive
   processing can be done in the cloud, only the data to be rendered for
   the end user needs to be sent.

   Other low latency high bandwidth applications, such as:

   *  interactive remote presence and

   *  video-assisted remote control of machinery or industrial processes

   are not credible at all without very low queuing delay.  No amount of
   extra access bandwidth or local processing can make up for lost time.

### 6.2. Use Cases

   The following use cases for L4S are being considered by various
   interested parties:

   *  where the bottleneck is one of various types of access network,
      e.g., DSL, Passive Optical Networks (PONs), DOCSIS cable, mobile,
      satellite; or where it's a Wi-Fi link (see Section 6.3 for some
      technology-specific details)

   *  private networks of heterogeneous data centres, where there is no
      single administrator that can arrange for all the simultaneous
      changes to senders, receivers, and networks needed to deploy
      DCTCP:

      -  a set of private data centres interconnected over a wide area
         with separate administrations but within the same company

      -  a set of data centres operated by separate companies
         interconnected by a community of interest network (e.g., for
         the finance sector)

      -  multi-tenant (cloud) data centres where tenants choose their
         operating system stack (Infrastructure as a Service (IaaS))

   *  different types of transport (or application) congestion control:

      -  elastic (TCP/SCTP);

      -  real-time (RTP, RMCAT); and

      -  query-response (DNS/LDAP).

   *  where low delay QoS is required but without inspecting or
      intervening above the IP layer [RFC8404]:

      -  Mobile and other networks have tended to inspect higher layers
         in order to guess application QoS requirements.  However, with
         growing demand for support of privacy and encryption, L4S
         offers an alternative.  There is no need to select which
         traffic to favour for queuing when L4S can give favourable
         queuing to all traffic.

   *  If queuing delay is minimized, applications with a fixed delay
      budget can communicate over longer distances or via more
      circuitous paths, e.g., longer chains of service functions
      [RFC7665] or of onion routers.

   *  If delay jitter is minimized, it is possible to reduce the
      dejitter buffers on the receiving end of video streaming, which
      should improve the interactive experience.

### 6.3. Applicability with Specific Link Technologies

   Certain link technologies aggregate data from multiple packets into
   bursts and buffer incoming packets while building each burst.  Wi-Fi,
   PON, and cable all involve such packet aggregation, whereas fixed
   Ethernet and DSL do not.  No sender, whether L4S or not, can do
   anything to reduce the buffering needed for packet aggregation.  So
   an AQM should not count this buffering as part of the queue that it
   controls, given no amount of congestion signals will reduce it.

   Certain link technologies also add buffering for other reasons,
   specifically:

   *  Radio links (cellular, Wi-Fi, or satellite) that are distant from
      the source are particularly challenging.  The radio link capacity
      can vary rapidly by orders of magnitude, so it is considered
      desirable to hold a standing queue that can utilize sudden
      increases of capacity.

   *  Cellular networks are further complicated by a perceived need to
      buffer in order to make hand-overs imperceptible.

   L4S cannot remove the need for all these different forms of
   buffering.  However, by removing 'the longest pole in the tent'
   (buffering for the large sawteeth of Classic congestion controls),
   L4S exposes all these 'shorter poles' to greater scrutiny.

   Until now, the buffering needed for these additional reasons tended
   to be over-specified -- with the excuse that none were 'the longest
   pole in the tent'.  But having removed the 'longest pole', it becomes
   worthwhile to minimize them, for instance, reducing packet
   aggregation burst sizes and MAC scheduling intervals.

   Also, certain link types, particularly radio-based links, are far
   more prone to transmission losses.  Section 6.4.3 explains how an L4S
   response to loss has to be as drastic as a Classic response.
   Nonetheless, research referred to in the same section has
   demonstrated potential for considerably more effective loss repair at
   the link layer, due to the relaxed ordering constraints of L4S
   packets.

### 6.4. Deployment Considerations

   L4S AQMs, whether DualQ [RFC9332] or FQ [RFC8290], are in themselves
   an incremental deployment mechanism for L4S -- so that L4S traffic
   can coexist with existing Classic (Reno-friendly) traffic. Section 6.4.1 explains why only deploying an L4S AQM in one node at
   each end of the access link will realize nearly all the benefit of
   L4S.

   L4S involves both the network and end systems, so Section 6.4.2   suggests some typical sequences to deploy each part and why there
   will be an immediate and significant benefit after deploying just one
   part.

   Sections 6.4.3 and 6.4.4 describe the converse incremental deployment
   case where there is no L4S AQM at the network bottleneck, so any L4S
   flow traversing this bottleneck has to take care in case it is
   competing with Classic traffic.

#### 6.4.1. Deployment Topology

   L4S AQMs will not have to be deployed throughout the Internet before
   L4S can benefit anyone.  Operators of public Internet access networks
   typically design their networks so that the bottleneck will nearly
   always occur at one known (logical) link.  This confines the cost of
   queue management technology to one place.

   The case of mesh networks is different and will be discussed later in
   this section.  However, the known-bottleneck case is generally true
   for Internet access to all sorts of different 'sites', where the word
   'site' includes home networks, small- to medium-sized campus or
   enterprise networks and even cellular devices (Figure 2).  Also, this
   known-bottleneck case tends to be applicable whatever the access link
   technology, whether xDSL, cable, PON, cellular, line of sight
   wireless, or satellite.

   Therefore, the full benefit of the L4S service should be available in
   the downstream direction when an L4S AQM is deployed at the ingress
   to this bottleneck link.  And similarly, the full upstream service
   will typically be available once an L4S AQM is deployed at the
   ingress into the upstream link.  (Of course, multihomed sites would
   only see the full benefit once all their access links were covered.)

                                            ______
                                           (      )
                         __          __  (          )
                        |DQ\________/DQ|( enterprise )
                    ___ |__/        \__| ( /campus  )
                   (   )                   (______)
                 (      )                           ___||_
   +----+      (          )  __                 __ /      \
   | DC |-----(    Core    )|DQ\_______________/DQ|| home |
   +----+      (          ) |__/               \__||______|
                  (_____) __
                         |DQ\__/\        __ ,===.
                         |__/    \  ____/DQ||| ||mobile
                                  \/    \__|||_||device
                                            | o |
                                            `---'

       Figure 2: Likely Location of DualQ (DQ) Deployments in Common
                             Access Topologies

   Deployment in mesh topologies depends on how overbooked the core is.
   If the core is non-blocking, or at least generously provisioned so
   that the edges are nearly always the bottlenecks, it would only be
   necessary to deploy an L4S AQM at the edge bottlenecks.  For example,
   some data-centre networks are designed with the bottleneck in the
   hypervisor or host Network Interface Controllers (NICs), while others
   bottleneck at the top-of-rack switch (both the output ports facing
   hosts and those facing the core).

   An L4S AQM would often next be needed where the Wi-Fi links in a home
   sometimes become the bottleneck.  Also an L4S AQM would eventually
   need to be deployed at any other persistent bottlenecks, such as
   network interconnections, e.g., some public Internet exchange points
   and the ingress and egress to WAN links interconnecting data centres.

#### 6.4.2. Deployment Sequences

   For any one L4S flow to provide benefit, it requires three (or
   sometimes two) parts to have been deployed: i) the congestion control
   at the sender; ii) the AQM at the bottleneck; and iii) older
   transports (namely TCP) need upgraded receiver feedback too.  This
   was the same deployment problem that ECN faced [RFC8170], so we have
   learned from that experience.

   Firstly, L4S deployment exploits the fact that DCTCP already exists
   on many Internet hosts (e.g., Windows, FreeBSD, and Linux), both
   servers and clients.  Therefore, an L4S AQM can be deployed at a
   network bottleneck to immediately give a working deployment of all
   the L4S parts for testing, as long as the ECT(0) codepoint is
   switched to ECT(1).  DCTCP needs some safety concerns to be fixed for
   general use over the public Internet (see Section 4.3 of the L4S ECN
   spec [RFC9331]), but DCTCP is not on by default, so these issues can
   be managed within controlled deployments or controlled trials.

   Secondly, the performance improvement with L4S is so significant that
   it enables new interactive services and products that were not
   previously possible.  It is much easier for companies to initiate new
   work on deployment if there is budget for a new product trial.  In
   contrast, if there were only an incremental performance improvement
   (as with Classic ECN), spending on deployment tends to be much harder
   to justify.

   Thirdly, the L4S identifier is defined so that network operators can
   initially enable L4S exclusively for certain customers or certain
   applications.  However, this is carefully defined so that it does not
   compromise future evolution towards L4S as an Internet-wide service.
   This is because the L4S identifier is defined not only as the end-to-
   end ECN field, but it can also optionally be combined with any other
   packet header or some status of a customer or their access link (see Section 5.4 of [RFC9331]).  Operators could do this anyway, even if
   it were not blessed by the IETF.  However, it is best for the IETF to
   specify that, if they use their own local identifier, it must be in
   combination with the IETF's identifier, ECT(1).  Then, if an operator
   has opted for an exclusive local-use approach, they only have to
   remove this extra rule later to make the service work across the
   Internet -- it will already traverse middleboxes, peerings, etc.

   +-+--------------------+----------------------+---------------------+
   | | Servers or proxies |      Access link     |             Clients |
   +-+--------------------+----------------------+---------------------+
   |0| DCTCP (existing)   |                      |    DCTCP (existing) |
   +-+--------------------+----------------------+---------------------+
   |1|                    |Add L4S AQM downstream|                     |
   | |       WORKS DOWNSTREAM FOR CONTROLLED DEPLOYMENTS/TRIALS        |
   +-+--------------------+----------------------+---------------------+
   |2| Upgrade DCTCP to   |                      |Replace DCTCP feedb'k|
   | | TCP Prague         |                      |         with AccECN |
   | |                 FULLY     WORKS     DOWNSTREAM                  |
   +-+--------------------+----------------------+---------------------+
   | |                    |                      |    Upgrade DCTCP to |
   |3|                    | Add L4S AQM upstream |          TCP Prague |
   | |                    |                      |                     |
   | |              FULLY WORKS UPSTREAM AND DOWNSTREAM                |
   +-+--------------------+----------------------+---------------------+

                 Figure 3: Example L4S Deployment Sequence

   Figure 3 illustrates some example sequences in which the parts of L4S
   might be deployed.  It consists of the following stages, preceded by
   a presumption that DCTCP is already installed at both ends:

   1.  DCTCP is not applicable for use over the public Internet, so it
       is emphasized here that any DCTCP flow has to be completely
       contained within a controlled trial environment.

       Within this trial environment, once an L4S AQM has been deployed,
       the trial DCTCP flow will experience immediate benefit, without
       any other deployment being needed.  In this example, downstream
       deployment is first, but in other scenarios, the upstream might
       be deployed first.  If no AQM at all was previously deployed for
       the downstream access, an L4S AQM greatly improves the Classic
       service (as well as adding the L4S service).  If an AQM was
       already deployed, the Classic service will be unchanged (and L4S
       will add an improvement on top).

   2.  In this stage, the name 'TCP Prague' [PRAGUE-CC ] is used to
       represent a variant of DCTCP that is designed to be used in a
       production Internet environment (that is, it has to comply with
       all the requirements in Section 4 of the L4S ECN spec [RFC9331],
       which then means it can be used over the public Internet).  If
       the application is primarily unidirectional, 'TCP Prague' at the
       sending end will provide all the benefit needed, as long as the
       receiving end supports Accurate ECN (AccECN) feedback [ACCECN ].

       For TCP transports, AccECN feedback is needed at the other end,
       but it is a generic ECN feedback facility that is already planned
       to be deployed for other purposes, e.g., DCTCP and BBR.  The two
       ends can be deployed in either order because, in TCP, an L4S
       congestion control only enables itself if it has negotiated the
       use of AccECN feedback with the other end during the connection
       handshake.  Thus, deployment of TCP Prague on a server enables
       L4S trials to move to a production service in one direction,
       wherever AccECN is deployed at the other end.  This stage might
       be further motivated by the performance improvements of TCP
       Prague relative to DCTCP (see Appendix A.2 of the L4S ECN spec
       [RFC9331]).

       Unlike TCP, from the outset, QUIC ECN feedback [RFC9000] has
       supported L4S.  Therefore, if the transport is QUIC, one-ended
       deployment of a Prague congestion control at this stage is simple
       and sufficient.

       For QUIC, if a proxy sits in the path between multiple origin
       servers and the access bottlenecks to multiple clients, then
       upgrading the proxy with a Scalable congestion control would
       provide the benefits of L4S over all the clients' downstream
       bottlenecks in one go -- whether or not all the origin servers
       were upgraded.  Conversely, where a proxy has not been upgraded,
       the clients served by it will not benefit from L4S at all in the
       downstream, even when any origin server behind the proxy has been
       upgraded to support L4S.

       For TCP, a proxy upgraded to support 'TCP Prague' would provide
       the benefits of L4S downstream to all clients that support AccECN
       (whether or not they support L4S as well).  And in the upstream,
       the proxy would also support AccECN as a receiver, so that any
       client deploying its own L4S support would benefit in the
       upstream direction, irrespective of whether any origin server
       beyond the proxy supported AccECN.

   3.  This is a two-move stage to enable L4S upstream.  An L4S AQM or
       TCP Prague can be deployed in either order as already explained.
       To motivate the first of two independent moves, the deferred
       benefit of enabling new services after the second move has to be
       worth it to cover the first mover's investment risk.  As
       explained already, the potential for new interactive services
       provides this motivation.  An L4S AQM also improves the upstream
       Classic service significantly if no other AQM has already been
       deployed.

   Note that other deployment sequences might occur.  For instance, the
   upstream might be deployed first; a non-TCP protocol might be used
   end to end, e.g., QUIC and RTP; a body, such as the 3GPP, might
   require L4S to be implemented in 5G user equipment; or other random
   acts of kindness might arise.

#### 6.4.3. L4S Flow but Non-ECN Bottleneck

   If L4S is enabled between two hosts, the L4S sender is required to
   coexist safely with Reno in response to any drop (see Section 4.3 of
   the L4S ECN spec [RFC9331]).

   Unfortunately, as well as protecting Classic traffic, this rule
   degrades the L4S service whenever there is any loss, even if the
   cause is not persistent congestion at a bottleneck, for example:

   *  congestion loss at other transient bottlenecks, e.g., due to
      bursts in shallower queues;

   *  transmission errors, e.g., due to electrical interference; and

   *  rate policing.

   Three complementary approaches are in progress to address this issue,
   but they are all currently research:

   *  In Prague congestion control, ignore certain losses deemed
      unlikely to be due to congestion (using some ideas from BBR
      [BBR-CC ] regarding isolated losses).  This could mask any of the
      above types of loss while still coexisting with drop-based
      congestion controls.

   *  A combination of Recent Acknowledgement (RACK) [RFC8985], L4S, and
      link retransmission without resequencing could repair transmission
      errors without the head of line blocking delay usually associated
      with link-layer retransmission [UnorderedLTE ] [RFC9331].

   *  Hybrid ECN/drop rate policers (see Section 8.3).

   L4S deployment scenarios that minimize these issues (e.g., over
   wireline networks) can proceed in parallel to this research, in the
   expectation that research success could continually widen L4S
   applicability.

#### 6.4.4. L4S Flow but Classic ECN Bottleneck

   Classic ECN support is starting to materialize on the Internet as an
   increased level of CE marking.  It is hard to detect whether this is
   all due to the addition of support for ECN in implementations of FQ-
   CoDel and/or FQ-COBALT, which is not generally problematic, because
   flow queue (FQ) scheduling inherently prevents a flow from exceeding
   the 'fair' rate irrespective of its aggressiveness.  However, some of
   this Classic ECN marking might be due to single-queue ECN deployment.
   This case is discussed in Section 4.3 of the L4S ECN spec [RFC9331].

#### 6.4.5. L4S AQM Deployment within Tunnels

   An L4S AQM uses the ECN field to signal congestion.  So in common
   with Classic ECN, if the AQM is within a tunnel or at a lower layer,
   correct functioning of ECN signalling requires standards-compliant
   propagation of the ECN field up the layers [RFC6040] [ECN-SHIM ]
   [ECN-ENCAP ].

# Referenced Sections from RFC 9438: CUBIC for Fast and Long-Distance Networks

The following sections were referenced. Remaining sections are not included.

## 4. CUBIC Congestion Control

   This section discusses how the congestion window is updated during
   the different stages of the CUBIC congestion controller.

### 4.1. Definitions

   The unit of all window sizes in this document is segments of the
   SMSS, and the unit of all times is seconds.  Implementations can use
   bytes to express window sizes, which would require factoring in the
   SMSS wherever necessary and replacing _segments_acked_ (Figure 4)
   with the number of acknowledged bytes.

#### 4.1.1. Constants of Interest

   *  β__cubic_: CUBIC multiplicative decrease factor as described in Section 4.6.

   *  α__cubic_: CUBIC additive increase factor used in the Reno-
      friendly region as described in Section 4.3.

   *  _C_: Constant that determines the aggressiveness of CUBIC in
      competing with other congestion control algorithms in high-BDP
      networks.  Please see Section 5 for more explanation on how it is
      set.  The unit for _C_ is

                                  segment
                                  ───────
                                        3
                                  second

#### 4.1.2. Variables of Interest

   This section defines the variables required to implement CUBIC:

   *  _RTT_: Smoothed round-trip time in seconds, calculated as
      described in [RFC6298].

   *  _cwnd_: Current congestion window in segments.

   *  _ssthresh_: Current slow start threshold in segments.

   *  _cwnd_prior_: Size of _cwnd_ in segments at the time of setting
      _ssthresh_ most recently, either upon exiting the first slow start
      or just before _cwnd_ was reduced in the last congestion event.

   *  _W_max_: Size of _cwnd_ in segments just before _cwnd_ was reduced
      in the last congestion event when fast convergence is disabled
      (same as _cwnd_prior_ on a congestion event).  However, if fast
      convergence is enabled, _W_max_ may be further reduced based on
      the current saturation point.

   *  _K_: The time period in seconds it takes to increase the
      congestion window size at the beginning of the current congestion
      avoidance stage to _W_max_.

   *  _t_current_: Current time of the system in seconds.

   *  _t_epoch_: The time in seconds at which the current congestion
      avoidance stage started.

   *  _cwnd_epoch_: The _cwnd_ at the beginning of the current
      congestion avoidance stage, i.e., at time _t_epoch_.

   *  W_cubic(_t_): The congestion window in segments at time _t_ in
      seconds based on the cubic increase function, as described in Section 4.2.

   *  _target_: Target value of the congestion window in segments after
      the next RTT -- that is, W_cubic(_t_ + _RTT_), as described in Section 4.2.

   *  _W_est_: An estimate for the congestion window in segments in the
      Reno-friendly region -- that is, an estimate for the congestion
      window of Reno.

   *  _segments_acked_: Number of SMSS-sized segments acked when a "new
      ACK" is received, i.e., an ACK that cumulatively acknowledges the
      delivery of previously unacknowledged data.  This number will be a
      decimal value when a new ACK acknowledges an amount of data that
      is not SMSS-sized.  Specifically, it can be less than 1 when a new
      ACK acknowledges a segment smaller than the SMSS.

### 4.2. Window Increase Function

   CUBIC maintains the ACK clocking of Reno by increasing the congestion
   window only at the reception of a new ACK.  It does not make any
   changes to the TCP Fast Recovery and Fast Retransmit algorithms
   [RFC6582] [RFC6675].

   During congestion avoidance, after a congestion event is detected as
   described in Section 3.1, CUBIC uses a window increase function
   different from Reno.

   CUBIC uses the following window increase function:

                                             3
                      W     (t) = C * (t - K)  + W
                       cubic                      max

                                  Figure 1

   where _t_ is the elapsed time in seconds from the beginning of the
   current congestion avoidance stage -- that is,

                           t = t        - t
                                current    epoch

   and where _t_epoch_ is the time at which the current congestion
   avoidance stage starts.  _K_ is the time period that the above
   function takes to increase the congestion window size at the
   beginning of the current congestion avoidance stage to _W_max_ if
   there are no further congestion events.  _K_ is calculated using the
   following equation:

                                ┌────────────────┐
                             3  │W    - cwnd
                             ╲  │ max       epoch
                         K =  ╲ │────────────────
                               ╲│       C

                                  Figure 2

   where _cwnd_epoch_ is the congestion window at the beginning of the
   current congestion avoidance stage.

   Upon receiving a new ACK during congestion avoidance, CUBIC computes
   the _target_ congestion window size after the next _RTT_ using
   Figure 1 as follows, where _RTT_ is the smoothed round-trip time.
   The lower and upper bounds below ensure that CUBIC's congestion
   window increase rate is non-decreasing and is less than the increase
   rate of slow start [SXEZ19].

                 ⎧
                 ⎪cwnd            if  W     (t + RTT) < cwnd
                 ⎪                     cubic
                 ⎨1.5 * cwnd      if  W     (t + RTT) > 1.5 * cwnd
        target = ⎪                     cubic
                 ⎪W     (t + RTT) otherwise
                 ⎩ cubic

   The elapsed time _t_ in Figure 1 MUST NOT include periods during
   which _cwnd_ has not been updated due to application-limited behavior
   (see Section 5.8).

   Depending on the value of the current congestion window size _cwnd_,
   CUBIC runs in three different regions:

   1.  The Reno-friendly region, which ensures that CUBIC achieves at
       least the same throughput as Reno.

   2.  The concave region, if CUBIC is not in the Reno-friendly region
       and _cwnd_ is less than _W_max_.

   3.  The convex region, if CUBIC is not in the Reno-friendly region
       and _cwnd_ is greater than _W_max_.

   To summarize, CUBIC computes both W_cubic(_t_) and _W_est_ (see Section 4.3) on receiving a new ACK in congestion avoidance and
   chooses the larger of the two values.

   The next sections describe the exact actions taken by CUBIC in each
   region.

### 4.3. Reno-Friendly Region

   Reno performs well in certain types of networks -- for example, under
   short RTTs and small bandwidths (or small BDPs).  In these networks,
   CUBIC remains in the Reno-friendly region to achieve at least the
   same throughput as Reno.

   The Reno-friendly region is designed according to the analysis
   discussed in [FHP00], which studies the performance of an AIMD
   algorithm with an additive factor of α (segments per _RTT_) and a
   multiplicative factor of β, denoted by AIMD(α, β).  _p_ is the packet
   loss rate.  Specifically, the average congestion window size of
   AIMD(α, β) can be calculated using Figure 3.

                                      ┌───────────────┐
                                      │  α * (1 + β)
                   AVG_AIMD(α, β) = ╲ │───────────────
                                     ╲│2 * (1 - β) * p

                                  Figure 3

   By the same analysis, to achieve an average window size similar to
   Reno that uses AIMD(1, 0.5), α must be equal to

                                     1 - β
                                 3 * ─────
                                     1 + β

   Thus, CUBIC uses Figure 4 to estimate the window size _W_est_ in the
   Reno-friendly region with

                                       1 - β
                                            cubic
                          α      = 3 * ──────────
                           cubic       1 + β
                                            cubic

   which achieves approximately the same average window size as Reno in
   many cases.  The model used to calculate α__cubic_ is not absolutely
   precise, but analysis and simulation as discussed in
   [AIMD-friendliness ], as well as over a decade of experience with
   CUBIC in the public Internet, show that this approach produces
   acceptable levels of rate fairness between CUBIC and Reno flows.
   Also, no significant drawbacks of the model have been reported.
   However, continued detailed analysis of this approach would be
   beneficial.  When receiving a new ACK in congestion avoidance (where
   _cwnd_ could be greater than or less than _W_max_), CUBIC checks
   whether W_cubic(_t_) is less than _W_est_.  If so, CUBIC is in the
   Reno-friendly region and _cwnd_ SHOULD be set to _W_est_ at each
   reception of a new ACK.

   _W_est_ is set equal to _cwnd_epoch_ at the start of the congestion
   avoidance stage.  After that, on every new ACK, _W_est_ is updated
   using Figure 4.  Note that this equation uses _segments_acked_ and
   _cwnd_ is measured in segments.  An implementation that measures
   _cwnd_ in bytes should adjust the equation accordingly using the
   number of acknowledged bytes and the SMSS.  Also note that this
   equation works for connections with enabled or disabled delayed ACKs
   [RFC5681], as _segments_acked_ will be different based on the
   segments actually acknowledged by a new ACK.

                                          segments_acked
                   W    = W    + α      * ──────────────
                    est    est    cubic        cwnd

                                  Figure 4

   Once _W_est_ has grown to reach the _cwnd_ at the time of most
   recently setting _ssthresh_ -- that is, _W_est_ >= _cwnd_prior_ --
   the sender SHOULD set α__cubic_ to 1 to ensure that it can achieve
   the same congestion window increment rate as Reno, which uses AIMD(1,
   0.5).

   The next two sections assume that CUBIC is not in the Reno-friendly
   region and uses the window increase function described in Section 4.2.  Although _cwnd_ is incremented in the same way for both
   concave and convex regions, they are discussed separately to analyze
   and understand the difference between the two regions.

### 4.4. Concave Region

   When receiving a new ACK in congestion avoidance, if CUBIC is not in
   the Reno-friendly region and _cwnd_ is less than _W_max_, then CUBIC
   is in the concave region.  In this region, _cwnd_ MUST be incremented
   by

                               target - cwnd
                               ─────────────
                                    cwnd

   for each received new ACK, where _target_ is calculated as described
   in Section 4.2.

### 4.5. Convex Region

   When receiving a new ACK in congestion avoidance, if CUBIC is not in
   the Reno-friendly region and _cwnd_ is larger than or equal to
   _W_max_, then CUBIC is in the convex region.

   The convex region indicates that the network conditions might have
   changed since the last congestion event, possibly implying more
   available bandwidth after some flow departures.  Since the Internet
   is highly asynchronous, some amount of perturbation is always
   possible without causing a major change in available bandwidth.

   Unless the cwnd is overridden by the AIMD window increase, CUBIC will
   behave cautiously when operating in this region.  The convex profile
   aims to increase the window very slowly at the beginning when _cwnd_
   is around _W_max_ and then gradually increases its rate of increase.
   This region is also called the "maximum probing phase", since CUBIC
   is searching for a new _W_max_.  In this region, _cwnd_ MUST be
   incremented by

                               target - cwnd
                               ─────────────
                                    cwnd

   for each received new ACK, where _target_ is calculated as described
   in Section 4.2.

### 4.6. Multiplicative Decrease

   When a congestion event is detected by the mechanisms described in Section 3.1, CUBIC updates _W_max_ and reduces _cwnd_ and _ssthresh_
   immediately, as described below.  In the case of packet loss, the
   sender MUST reduce _cwnd_ and _ssthresh_ immediately upon entering
   loss recovery, similar to [RFC5681] (and [RFC6675]).  Note that other
   mechanisms, such as Proportional Rate Reduction [RFC6937], can be
   used to reduce the sending rate during loss recovery more gradually.
   The parameter β__cubic_ SHOULD be set to 0.7, which is different from
   the multiplicative decrease factor used in [RFC5681] (and [RFC6675])
   during fast recovery.

   In Figure 5, _flight_size_ is the amount of outstanding
   (unacknowledged) data in the network, as defined in [RFC5681].  Note
   that a rate-limited application with idle periods or periods when
   unable to send at the full rate permitted by _cwnd_ could easily
   encounter notable variations in the volume of data sent from one RTT
   to another, resulting in _flight_size_ that is significantly less
   than _cwnd_ when there is a congestion event.  The congestion
   response would therefore decrease _cwnd_ to a much lower value than
   necessary.  To avoid such suboptimal performance, the mechanisms
   described in [RFC7661] can be used.  [RFC7661] describes how to
   manage and use _cwnd_ and _ssthresh_ during a rate-limited interval,
   and how to update _cwnd_ and _ssthresh_ after congestion has been
   detected.  The mechanisms defined in [RFC7661] are safe to use even
   when _cwnd_ is greater than the receive window, because they validate
   _cwnd_ based on the amount of data acknowledged by the network in an
   RTT, which implicitly accounts for the allowed receive window.

   Some implementations of CUBIC currently use _cwnd_ instead of
   _flight_size_ when calculating a new _ssthresh_.  Implementations
   that use _cwnd_ MUST use other measures to prevent _cwnd_ from
   growing when the volume of bytes in flight is smaller than
   _cwnd_.  This also effectively prevents _cwnd_ from growing beyond
   the receive window.  Such measures are important for preventing a
   CUBIC sender from using an arbitrarily high cwnd _value_ when
   calculating new values for _ssthresh_ and _cwnd_ when congestion is
   detected.  This might not be as robust as the mechanisms described in
   [RFC7661].

   A QUIC sender that uses a _cwnd_ _value_ to calculate new values for
   _cwnd_ and _ssthresh_ after detecting a congestion event is REQUIRED
   to apply similar mechanisms [RFC9002].

    ssthresh =  flight_size * β      new  ssthresh
                               cubic
    cwnd      = cwnd                 save  cwnd
        prior
                ⎧max(ssthresh, 2)    reduction on loss, cwnd >= 2 SMSS
    cwnd =      ⎨max(ssthresh, 1)    reduction on ECE, cwnd >= 1 SMSS
                ⎩
    ssthresh =  max(ssthresh, 2)     ssthresh >= 2 SMSS

                                  Figure 5

   A side effect of setting β__cubic_ to a value bigger than 0.5 is that
   packet loss can happen for more than one RTT in certain cases, but it
   can work efficiently in other cases -- for example, when HyStart++
   [RFC9406] is used along with CUBIC or when the sending rate is
   limited by the application.  While a more adaptive setting of
   β__cubic_ could help limit packet loss to a single round, it would
   require detailed analyses and large-scale evaluations to validate
   such algorithms.

   Note that CUBIC MUST continue to reduce _cwnd_ in response to
   congestion events detected by ECN-Echo ACKs until it reaches a value
   of 1 SMSS.  If congestion events indicated by ECN-Echo ACKs persist,
   a sender with a _cwnd_ of 1 SMSS MUST reduce its sending rate even
   further.  This can be achieved by using a retransmission timer with
   exponential backoff, as described in [RFC3168].

### 4.7. Fast Convergence

   To improve convergence speed, CUBIC uses a heuristic.  When a new
   flow joins the network, existing flows need to give up some of their
   bandwidth to allow the new flow some room for growth if the existing
   flows have been using all the network bandwidth.  To speed up this
   bandwidth release by existing flows, the following fast convergence
   mechanism SHOULD be implemented.

   With fast convergence, when a congestion event occurs, _W_max_ is
   updated as follows, before the window reduction described in Section 4.6.

       ⎧       1 + β
       ⎪            cubic
       ⎪cwnd * ────────── if  cwnd < W     and fast convergence enabled,
W    = ⎨           2                  max
 max   ⎪                  further reduce  W
       ⎪                                   max
       ⎩cwnd             otherwise, remember cwnd before reduction

   During a congestion event, if the current _cwnd_ is less than
   _W_max_, this indicates that the saturation point experienced by this
   flow is getting reduced because of a change in available bandwidth.
   This flow can then release more bandwidth by reducing _W_max_
   further.  This action effectively lengthens the time for this flow to
   increase its congestion window, because the reduced _W_max_ forces
   the flow to plateau earlier.  This allows more time for the new flow
   to catch up to its congestion window size.

   Fast convergence is designed for network environments with multiple
   CUBIC flows.  In network environments with only a single CUBIC flow
   and without any other traffic, fast convergence SHOULD be disabled.

### 4.8. Timeout

   In the case of a timeout, CUBIC follows Reno to reduce _cwnd_
   [RFC5681] but sets _ssthresh_ using β__cubic_ (same as in Section 4.6) in a way that is different from Reno TCP [RFC5681].

   During the first congestion avoidance stage after a timeout, CUBIC
   increases its congestion window size using Figure 1, where _t_ is the
   elapsed time since the beginning of the current congestion avoidance
   stage, _K_ is set to 0, and _W_max_ is set to the congestion window
   size at the beginning of the current congestion avoidance stage.  In
   addition, for the Reno-friendly region, _W_est_ SHOULD be set to the
   congestion window size at the beginning of the current congestion
   avoidance stage.

### 4.9. Spurious Congestion Events

   In cases where CUBIC reduces its congestion window in response to
   having detected packet loss via duplicate ACKs or timeouts, it is
   possible that the missing ACK could arrive after the congestion
   window reduction and a corresponding packet retransmission.  For
   example, packet reordering could trigger this behavior.  A high
   degree of packet reordering could cause multiple congestion window
   reduction events, where spurious losses are incorrectly interpreted
   as congestion signals, thus degrading CUBIC's performance
   significantly.

   For TCP, there are two types of spurious events: spurious timeouts
   and spurious fast retransmits.  In the case of QUIC, there are no
   spurious timeouts, as the loss is only detected after receiving an
   ACK.

#### 4.9.1. Spurious Timeouts

   An implementation MAY detect spurious timeouts based on the
   mechanisms described in Forward RTO-Recovery [RFC5682].  Experimental
   alternatives include the Eifel detection algorithm [RFC3522].  When a
   spurious timeout is detected, a TCP implementation MAY follow the
   response algorithm described in [RFC4015] to restore the congestion
   control state and adapt the retransmission timer to avoid further
   spurious timeouts.

#### 4.9.2. Spurious Fast Retransmits

   Upon receiving an ACK, a TCP implementation MAY detect spurious fast
   retransmits either using TCP Timestamps or via D-SACK [RFC2883].  As
   noted above, experimental alternatives include the Eifel detection
   algorithm [RFC3522], which uses TCP Timestamps; and DSACK-based
   detection [RFC3708], which uses DSACK information.  A QUIC
   implementation can easily determine a spurious fast retransmit if a
   QUIC packet is acknowledged after it has been marked as lost and the
   original data has been retransmitted with a new QUIC packet.

   This section specifies a simple response algorithm when a spurious
   fast retransmit is detected by acknowledgments.  Implementations
   would need to carefully evaluate the impact of using this algorithm
   in different environments that may experience a sudden change in
   available capacity (e.g., due to variable radio capacity, a routing
   change, or a mobility event).

   When packet loss is detected via acknowledgments, a CUBIC
   implementation MAY save the current value of the following variables
   before the congestion window is reduced.

                        undo_cwnd =      cwnd
                        undo_cwnd      = cwnd
                                 prior       prior
                        undo_ssthresh =  ssthresh
                        undo_W    =      W
                              max         max
                        undo_K =         K
                        undo_t      =    t
                              epoch       epoch
                        undo_W    =      W
                              est         est

   Once the previously declared packet loss is confirmed to be spurious,
   CUBIC MAY restore the original values of the above-mentioned
   variables as follows if the current _cwnd_ is lower than
   _cwnd_prior_.  Restoring the original values ensures that CUBIC's
   performance is similar to what it would be without spurious losses.

              cwnd =      undo_cwnd      ⎫
              cwnd      = undo_cwnd      ⎮
                  prior            prior ⎮
              ssthresh =  undo_ssthresh  ⎮
              W    =      undo_W         ⎮
               max              max      ⎬if cwnd < cwnd
              K =         undo_K         ⎮              prior
              t      =    undo_t         ⎮
               epoch            epoch    ⎮
              W    =      undo_W         ⎮
               est              est      ⎭

   In rare cases, when the detection happens long after a spurious fast
   retransmit event and the current _cwnd_ is already higher than
   _cwnd_prior_, CUBIC SHOULD continue to use the current and the most
   recent values of these variables.

### 4.10. Slow Start

   When _cwnd_ is no more than _ssthresh_, CUBIC MUST employ a slow
   start algorithm.  In general, CUBIC SHOULD use the HyStart++ slow
   start algorithm [RFC9406] or MAY use the Reno TCP slow start
   algorithm [RFC5681] in the rare cases when HyStart++ is not suitable.
   Experimental alternatives include hybrid slow start [HR11], a
   predecessor to HyStart++ that some CUBIC implementations have used as
   the default for the last decade, and limited slow start [RFC3742].
   Whichever startup algorithm is used, work might be needed to ensure
   that the end of slow start and the first multiplicative decrease of
   congestion avoidance work well together.

   When CUBIC uses HyStart++ [RFC9406], it may exit the first slow start
   without incurring any packet loss and thus _W_max_ is undefined.  In
   this special case, CUBIC sets _cwnd_prior = cwnd_ and switches to
   congestion avoidance.  It then increases its congestion window size
   using Figure 1, where _t_ is the elapsed time since the beginning of
   the current congestion avoidance stage, _K_ is set to 0, and _W_max_
   is set to the congestion window size at the beginning of the current
   congestion avoidance stage.

### 5.3. Difficult Environments

   CUBIC is designed to remedy the poor performance of Reno in fast and
   long-distance networks.

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





## 3. ECN Nonce and RFC 3540

   As specified in RFC 3168, ECN uses two ECN-Capable Transport (ECT)
   codepoints, ECT(0) and ECT(1), to indicate that a packet supports
   ECN.  RFC 3168 assigned the second codepoint, ECT(1), to support ECN
   nonce functionality that discourages receivers from exploiting ECN to
   improve their throughput at the expense of other network users.  That
   ECN nonce functionality is fully specified in RFC 3540 [RFC3540].
   This section explains why RFC 3540 has been reclassified from
   Experimental to Historic and makes associated updates to RFC 3168.

   While the ECN nonce works as specified, and has been deployed in
   limited environments, widespread usage in the Internet has not
   materialized.  A study of the ECN behavior of the top one million web
   servers using 2014 data [Trammell15] found that after ECN was
   negotiated, none of the 581,711 IPv4 servers tested were using both
   ECT codepoints, which would have been a possible sign of ECN nonce
   usage.  Of the 17,028 IPv6 servers tested, four set both ECT(0) and
   ECT(1) on data packets.  This might have been evidence of use of the
   ECN nonce by these four servers, but it might equally have been due
   to erroneous re-marking of the ECN field by a middlebox or router.

   With the emergence of new experimental functionality that depends on
   use of the ECT(1) codepoint for other purposes, continuing to reserve
   that codepoint for the ECN nonce experiment is no longer justified.
   In addition, other approaches to discouraging receivers from
   exploiting ECN have emerged; see Appendix B.1 of [ECN-L4S ].
   Therefore, in support of ECN experimentation with the ECT(1)
   codepoint, this memo:

   o  Declares that the ECN nonce experiment [RFC3540] has concluded and
      notes the absence of widespread deployment.

   o  Updates RFC 3168 [RFC3168] to remove discussion of the ECN nonce
      and use of ECT(1) for that nonce.

   The four primary updates to RFC 3168 that remove discussion of the
   ECN nonce and use of ECT(1) for that nonce are as follows:

   1.  The removal of the paragraph in Section 5 that immediately
       follows Figure 1; this paragraph discusses the ECN nonce as the
       motivation for two ECT codepoints.

   2.  The removal of Section 11.2, "A Discussion of the ECN nonce", in
       its entirety. 
   3.  The removal of the last paragraph of Section 12, which states
       that ECT(1) may be used as part of the implementation of the ECN
       nonce.

   4.  The removal of the first two paragraphs of Section 20.2, which
       discuss the ECN nonce and alternatives.  No changes are made to
       the rest of Section 20.2, which discusses alternative uses for
       the fourth ECN codepoint.

   In addition, other less-substantive changes to RFC 3168 are required
   to remove all other mentions of the ECN nonce and to remove
   implications that ECT(1) is intended for use by the ECN nonce; these
   specific text updates are omitted for brevity.

## 4. Updates to RFC 3168

   The following subsections specify updates to RFC 3168 to enable the
   three areas of experimentation summarized in Section 2.

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

## 3. Congestion Control Algorithms

   This section defines the four congestion control algorithms: slow
   start, congestion avoidance, fast retransmit, and fast recovery,
   developed in [Jac88] and [Jac90].  In some situations, it may be
   beneficial for a TCP sender to be more conservative than the
   algorithms allow; however, a TCP MUST NOT be more aggressive than the
   following algorithms allow (that is, MUST NOT send data when the
   value of cwnd computed by the following algorithms would not allow
   the data to be sent).

   Also, note that the algorithms specified in this document work in
   terms of using loss as the signal of congestion.  Explicit Congestion
   Notification (ECN) could also be used as specified in [RFC3168].

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

### 3.2. Fast Retransmit/Fast Recovery

   A TCP receiver SHOULD send an immediate duplicate ACK when an out-
   of-order segment arrives.  The purpose of this ACK is to inform the
   sender that a segment was received out-of-order and which sequence
   number is expected.  From the sender's perspective, duplicate ACKs
   can be caused by a number of network problems.  First, they can be
   caused by dropped segments.  In this case, all segments after the
   dropped segment will trigger duplicate ACKs until the loss is
   repaired.  Second, duplicate ACKs can be caused by the re-ordering of
   data segments by the network (not a rare event along some network
   paths [Pax97]).  Finally, duplicate ACKs can be caused by replication
   of ACK or data segments by the network.  In addition, a TCP receiver
   SHOULD send an immediate ACK when the incoming segment fills in all
   or part of a gap in the sequence space.  This will generate more
   timely information for a sender recovering from a loss through a
   retransmission timeout, a fast retransmit, or an advanced loss
   recovery algorithm, as outlined in section 4.3.

   The TCP sender SHOULD use the "fast retransmit" algorithm to detect
   and repair loss, based on incoming duplicate ACKs.  The fast
   retransmit algorithm uses the arrival of 3 duplicate ACKs (as defined
   in section 2, without any intervening ACKs which move SND.UNA) as an
   indication that a segment has been lost.  After receiving 3 duplicate
   ACKs, TCP performs a retransmission of what appears to be the missing
   segment, without waiting for the retransmission timer to expire. 
   After the fast retransmit algorithm sends what appears to be the
   missing segment, the "fast recovery" algorithm governs the
   transmission of new data until a non-duplicate ACK arrives.  The
   reason for not performing slow start is that the receipt of the
   duplicate ACKs not only indicates that a segment has been lost, but
   also that segments are most likely leaving the network (although a
   massive segment duplication by the network can invalidate this
   conclusion).  In other words, since the receiver can only generate a
   duplicate ACK when a segment has arrived, that segment has left the
   network and is in the receiver's buffer, so we know it is no longer
   consuming network resources.  Furthermore, since the ACK "clock"
   [Jac88] is preserved, the TCP sender can continue to transmit new
   segments (although transmission must continue using a reduced cwnd,
   since loss is an indication of congestion).

   The fast retransmit and fast recovery algorithms are implemented
   together as follows.

   1.  On the first and second duplicate ACKs received at a sender, a
       TCP SHOULD send a segment of previously unsent data per [RFC3042]
       provided that the receiver's advertised window allows, the total
       FlightSize would remain less than or equal to cwnd plus 2*SMSS,
       and that new data is available for transmission.  Further, the
       TCP sender MUST NOT change cwnd to reflect these two segments
       [RFC3042].  Note that a sender using SACK [RFC2018] MUST NOT send
       new data unless the incoming duplicate acknowledgment contains
       new SACK information.

   2.  When the third duplicate ACK is received, a TCP MUST set ssthresh
       to no more than the value given in equation (4).  When [RFC3042]
       is in use, additional data sent in limited transmit MUST NOT be
       included in this calculation.

   3.  The lost segment starting at SND.UNA MUST be retransmitted and
       cwnd set to ssthresh plus 3*SMSS.  This artificially "inflates"
       the congestion window by the number of segments (three) that have
       left the network and which the receiver has buffered.

   4.  For each additional duplicate ACK received (after the third),
       cwnd MUST be incremented by SMSS.  This artificially inflates the
       congestion window in order to reflect the additional segment that
       has left the network.

       Note: [SCWA99] discusses a receiver-based attack whereby many
       bogus duplicate ACKs are sent to the data sender in order to
       artificially inflate cwnd and cause a higher than appropriate 
       sending rate to be used.  A TCP MAY therefore limit the number of
       times cwnd is artificially inflated during loss recovery to the
       number of outstanding segments (or, an approximation thereof).

       Note: When an advanced loss recovery mechanism (such as outlined
       in section 4.3) is not in use, this increase in FlightSize can
       cause equation (4) to slightly inflate cwnd and ssthresh, as some
       of the segments between SND.UNA and SND.NXT are assumed to have
       left the network but are still reflected in FlightSize.

   5.  When previously unsent data is available and the new value of
       cwnd and the receiver's advertised window allow, a TCP SHOULD
       send 1*SMSS bytes of previously unsent data.

   6.  When the next ACK arrives that acknowledges previously
       unacknowledged data, a TCP MUST set cwnd to ssthresh (the value
       set in step 2).  This is termed "deflating" the window.

       This ACK should be the acknowledgment elicited by the
       retransmission from step 3, one RTT after the retransmission
       (though it may arrive sooner in the presence of significant out-
       of-order delivery of data segments at the receiver).
       Additionally, this ACK should acknowledge all the intermediate
       segments sent between the lost segment and the receipt of the
       third duplicate ACK, if none of these were lost.

   Note: This algorithm is known to generally not recover efficiently
   from multiple losses in a single flight of packets [FF96].  Section 
   4.3 below addresses such cases.

# Referenced Sections from RFC 8511: TCP Alternative Backoff with ECN (ABE)

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Explicit Congestion Notification (ECN) [RFC3168] makes it possible
   for an Active Queue Management (AQM) mechanism to signal the presence
   of incipient congestion without necessarily incurring packet loss.
   This lets the network deliver some packets to an application that
   would have been dropped if the application or transport did not
   support ECN.  This packet loss reduction is the most obvious benefit
   of ECN, but it is often relatively modest.  Other benefits of
   deploying ECN have been documented in [RFC8087].

   The rules for ECN were originally written to be very conservative,
   and they required the congestion control algorithms of ECN-Capable
   Transport (ECT) protocols to treat indications of congestion
   signalled by ECN exactly the same as they would treat an inferred
   packet loss [RFC3168].  Research has demonstrated the benefits of
   reducing network delays that are caused by interaction of loss-based
   TCP congestion control and excessive buffering [BUFFERBLOAT ].  This
   has led to the creation of AQM mechanisms like Proportional Integral
   Controller Enhanced (PIE) [RFC8033] and Controlling Queue Delay
   (CoDel) [RFC8289], which prevent bloated queues that are common with
   unmanaged and excessively large buffers deployed across the Internet
   [BUFFERBLOAT ].

   The AQM mechanisms mentioned above aim to keep a sustained queue
   short while tolerating transient (short-term) packet bursts.
   However, currently used loss-based congestion control mechanisms are
   not always able to effectively utilise a bottleneck link where there
   are short queues.  For example, a TCP sender using the Reno
   congestion control needs to be able to store at least an end-to-end
   bandwidth-delay product (BDP) worth of data at the bottleneck buffer
   if it is to maintain full path utilisation in the face of loss-
   induced reduction of the congestion window (cwnd) [RFC5681].  This
   amount of buffering effectively doubles the amount of data that can
   be in flight and the maximum round-trip time (RTT) experienced by the
   TCP sender.

   Modern AQM mechanisms can use ECN to signal the early signs of
   impending queue buildup long before a tail-drop queue would be forced
   to resort to dropping packets.  It is therefore appropriate for the
   transport protocol congestion control algorithm to have a more
   measured response when it receives an indication with an early
   warning of congestion after the remote endpoint receives an ECN
   CE-marked packet.  Recognizing these changes in modern AQM practices,
   the strict requirement that ECN CE signals be treated identically to
   inferred packet loss has been relaxed [RFC8311].  This document
   therefore defines a new sender-side-only congestion control response 
   called "ABE" (Alternative Backoff with ECN).  ABE improves TCP's
   average throughput when routers use AQM-controlled buffers that allow
   only for short queues.

## 3. Specification

   This specification changes the congestion control algorithm of an
   ECN-Capable TCP transport protocol by changing the TCP-sender
   response to feedback from the TCP receiver that indicates the
   reception of a CE-marked packet, i.e., receipt of a packet with the
   ECN-Echo flag (defined in [RFC3168]) set, following the process
   defined in [RFC8311].

   The TCP-sender response is currently specified in Section 6.1.2 of
   the ECN specification [RFC3168] and has been slightly updated by Section 4.1 of [RFC8311] to read as:

      The indication of congestion should be treated just as a
      congestion loss in non-ECN-Capable TCP.  That is, the TCP source
      halves the congestion window "cwnd" and reduces the slow start
      threshold "ssthresh", unless otherwise specified by an
      Experimental RFC in the IETF document stream.

   As permitted by RFC 8311, this document specifies a sender-side
   change to TCP where receipt of a packet with the ECN-Echo flag SHOULD
   trigger the TCP source to set the slow start threshold (ssthresh) to
   0.8 times the FlightSize, with a lower bound of 2 * SMSS applied to
   the result (where SMSS stands for Sender Maximum Segment Size)).  As
   in [RFC5681], the TCP sender also reduces the cwnd value to no more
   than the new ssthresh value.  Section 6.1.2 of RFC 3168 provides
   guidance on setting a cwnd less than 2 * SMSS.

### 3.1. Choice of ABE Multiplier

   ABE decouples the reaction of a TCP sender to inferred packet loss
   from the indication of ECN-signalled congestion in the congestion
   avoidance phase.  To achieve this, ABE uses a different scaling
   factor for Equation 4 in Section 3.1 of [RFC5681].  The description
   respectively uses beta_{loss} and beta_{ecn} to refer to the
   multiplicative decrease factors applied in response to inferred 
   packet loss, and in response to a receiver indicating ECN-signalled
   congestion.  For non-ECN-enabled TCP connections, only beta_{loss}
   applies.

   In other words, in response to inferred packet loss:

      ssthresh = max (FlightSize * beta_{loss}, 2 * SMSS)

   and in response to an indication of an ECN-signalled congestion:

      ssthresh = max (FlightSize * beta_{ecn}, 2 * SMSS)

      and

      cwnd = ssthresh

      (If ssthresh == 2 * SMSS, Section 6.1.2 of RFC 3168 provides
      guidance on setting a cwnd lower than 2 * SMSS.)

   where FlightSize is the amount of outstanding data in the network,
   upper-bounded by the smaller of the sender's cwnd and the receiver's
   advertised window (rwnd) [RFC5681].  The higher the values of
   beta_{loss} and beta_{ecn}, the less aggressive the response of any
   individual backoff event.

   The appropriate choice for beta_{loss} and beta_{ecn} values is a
   balancing act between path utilisation and draining the bottleneck
   queue.  More aggressive backoff (smaller beta_*) risks the
   underutilisation of the path, while less-aggressive backoff (larger
   beta_*) can result in slower draining of the bottleneck queue.

   The Internet has already been running with at least two different
   beta_{loss} values for several years: the standard value is 0.5
   [RFC5681], and the Linux implementation of CUBIC [RFC8312] has used a
   multiplier of 0.7 since kernel version 2.6.25 released in 2008.  ABE
   does not change the value of beta_{loss} used by current TCP
   implementations.

   The recommendation in this document specifies a value of
   beta_{ecn}=0.8.  This recommended beta_{ecn} value is only applicable
   for the standard TCP congestion control [RFC5681].  The selection of
   beta_{ecn} enables tuning the response of a TCP connection to shallow
   AQM-marking thresholds.  beta_{loss} characterizes the response of a
   congestion control algorithm to packet loss, i.e., exhaustion of
   buffers (of unknown depth).  Different values for beta_{loss} have
   been suggested for TCP congestion control algorithms.  Consequently,
   beta_{ecn} is likely to be an algorithm-specific parameter rather
   than a constant multiple of the algorithm's existing beta_{loss}. 
   A range of tests (Section IV of [ABE2017]) with NewReno and CUBIC
   over CoDel and PIE in lightly multiplexed scenarios have explored
   this choice of parameter.  The results of these tests indicate that
   CUBIC connections benefit from beta_{ecn} of 0.85 (cf.  beta_{loss} =
   0.7), and NewReno connections see improvements with beta_{ecn} in the
   range 0.7 to 0.85 (cf. beta_{loss} = 0.5).

## 5. ABE Deployment Requirements

   This update is a sender-side-only change.  Like other changes to
   congestion control algorithms, it does not require any change to the
   TCP receiver or to network devices.  It does not require any ABE-
   specific changes in routers or the use of Accurate ECN feedback
   [ACC-ECN-FEEDBACK ] by a receiver.

   If the method is only deployed by some senders, and not by others,
   the senders using it can gain some advantage, possibly at the expense
   of other flows that do not use this updated method.  Because this
   advantage applies only to ECN-marked packets and not to packet-loss
   indications, an ECN-Capable bottleneck will still fall back to
   dropping packets if a TCP sender using ABE is too aggressive.  The
   result is no different than if the TCP sender were using traditional
   loss-based congestion control. 
   When used with bottlenecks that do not support ECN marking, the
   specification does not modify the transport protocol.

# Referenced Sections from RFC 8323: CoAP (Constrained Application Protocol) over TCP, TLS, and WebSockets

The following sections were referenced. Remaining sections are not included.

### 11.2. CoAP Signaling Option Numbers Registry

   IANA has created a subregistry for Option Numbers used in CoAP
   Signaling Options within the "Constrained RESTful Environments (CoRE)
   Parameters" registry.  The name of this subregistry is "CoAP
   Signaling Option Numbers".

   Each entry in the subregistry must include one or more of the codes
   in the "CoAP Signaling Codes" subregistry (Section 11.1), the number
   for the Option, the name of the Option, and a reference to the
   Option's documentation. 
   Initial entries in this subregistry are as follows:

         +------------+--------+---------------------+-----------+
         | Applies to | Number | Name                | Reference |
         +------------+--------+---------------------+-----------+
         | 7.01       |      2 | Max-Message-Size    |  RFC 8323 |
         |            |        |                     |           |
         | 7.01       |      4 | Block-Wise-Transfer |  RFC 8323 |
         |            |        |                     |           |
         | 7.02, 7.03 |      2 | Custody             |  RFC 8323 |
         |            |        |                     |           |
         | 7.04       |      2 | Alternative-Address |  RFC 8323 |
         |            |        |                     |           |
         | 7.04       |      4 | Hold-Off            |  RFC 8323 |
         |            |        |                     |           |
         | 7.05       |      2 | Bad-CSM-Option      |  RFC 8323 |
         +------------+--------+---------------------+-----------+

                   Table 2: CoAP Signaling Option Codes

   The IANA policy for future additions to this subregistry is based on
   number ranges for the option numbers, analogous to the policy defined
   in Section 12.2 of [RFC7252].  (The policy is analogous rather than
   identical because the structure of this subregistry includes an
   additional column ("Applies to"); however, the value of this column
   has no influence on the policy.)

   The documentation for a Signaling Option Number should specify the
   semantics of an option with that number, including the following
   properties:

   o  Whether the option is critical or elective, as determined by the
      Option Number.

   o  Whether the option is repeatable.

   o  The format and length of the option's value.

   o  The base value for the option, if any. 





### 11.3. Service Name and Port Number Registration

   IANA has assigned the port number 5683 and the service name "coap",
   in accordance with [RFC6335].

   Service Name:
      coap

   Transport Protocol:
      tcp

   Assignee:
      IESG <iesg@ietf.org>

   Contact:
      IETF Chair <chair@ietf.org>

   Description:
      Constrained Application Protocol (CoAP)

   Reference:RFC 8323   Port Number:
      5683

### 11.4. Secure Service Name and Port Number Registration

   IANA has assigned the port number 5684 and the service name "coaps",
   in accordance with [RFC6335].  The port number is to address the
   exceptional case of TLS implementations that do not support the ALPN
   extension [RFC7301].

   Service Name:
      coaps

   Transport Protocol:
      tcp

   Assignee:
      IESG <iesg@ietf.org>

   Contact:
      IETF Chair <chair@ietf.org>

   Description:
      Constrained Application Protocol (CoAP)
   Reference:
      [RFC7301], RFC 8323   Port Number:
      5684

# Referenced Sections from RFC 9293: Transmission Control Protocol (TCP)

The following sections were referenced. Remaining sections are not included.

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

## 5. Changes from RFC 793

   This document obsoletes RFC 793 as well as RFCs 6093 and 6528, which
   updated 793.  In all cases, only the normative protocol specification
   and requirements have been incorporated into this document, and some
   informational text with background and rationale may not have been
   carried in.  The informational content of those documents is still
   valuable in learning about and understanding TCP, and they are valid
   Informational references, even though their normative content has
   been incorporated into this document.

   The main body of this document was adapted from RFC 793's Section 3,
   titled "FUNCTIONAL SPECIFICATION", with an attempt to keep formatting
   and layout as close as possible.

   The collection of applicable RFC errata that have been reported and
   either accepted or held for an update to RFC 793 were incorporated
   (Errata IDs: 573 [73], 574 [74], 700 [75], 701 [76], 1283 [77], 1561
   [78], 1562 [79], 1564 [80], 1571 [81], 1572 [82], 2297 [83], 2298
   [84], 2748 [85], 2749 [86], 2934 [87], 3213 [88], 3300 [89], 3301
   [90], 6222 [91]).  Some errata were not applicable due to other
   changes (Errata IDs: 572 [92], 575 [93], 1565 [94], 1569 [95], 2296
   [96], 3305 [97], 3602 [98]).

   Changes to the specification of the urgent pointer described in RFCs
   1011, 1122, and 6093 were incorporated.  See RFC 6093 for detailed
   discussion of why these changes were necessary.

   The discussion of the RTO from RFC 793 was updated to refer to RFC 
   6298.  The text on the RTO in RFC 1122 originally replaced the text
   in RFC 793; however, RFC 2988 should have updated RFC 1122 and has
   subsequently been obsoleted by RFC 6298. RFC 1011 [18] contains a number of comments about RFC 793, including
   some needed changes to the TCP specification.  These are expanded in RFC 1122, which contains a collection of other changes and
   clarifications to RFC 793.  The normative items impacting the
   protocol have been incorporated here, though some historically useful
   implementation advice and informative discussion from RFC 1122 is not
   included here.  The present document, which is now the TCP
   specification rather than RFC 793, updates RFC 1011, and the comments
   noted in RFC 1011 have been incorporated. RFC 1122 contains more than just TCP requirements, so this document
   can't obsolete RFC 1122 entirely.  It is only marked as "updating"RFC 1122; however, it should be understood to effectively obsolete
   all of the material on TCP found in RFC 1122.

   The more secure initial sequence number generation algorithm from RFC 
   6528 was incorporated.  See RFC 6528 for discussion of the attacks
   that this mitigates, as well as advice on selecting PRF algorithms
   and managing secret key data.

   A note based on RFC 6429 was added to explicitly clarify that system
   resource management concerns allow connection resources to be
   reclaimed.  RFC 6429 is obsoleted in the sense that the clarification
   it describes has been reflected within this base TCP specification.

   The description of congestion control implementation was added based
   on the set of documents that are IETF BCP or Standards Track on the
   topic and the current state of common implementations.

# Referenced Sections from RFC 5961: Improving TCP's Robustness to Blind In-Window Attacks

The following sections were referenced. Remaining sections are not included.

### 1.2. Basic Attack Methodology

   Focusing upon the RST attack, we examine this attack in more detail
   to get an overview as to how it works and how this document addresses
   the issue.  For this attack, the goal is for the attacker to cause
   one of the two endpoints of the connection to incorrectly tear down
   the connection state, effectively aborting the connection.  One of
   the important things to note is that for the attack to succeed the
   RST needs to be in the valid receive window.  It also needs to be
   emphasized that the receive window is independent of the current
   congestion window of the TCP connection.  The attacker would try to
   forge many RST segments to try to cover the space of possible windows
   by putting out a packet in each potential window.  To do this, the
   attacker needs to have or guess several pieces of information namely:

   1) The 4-tuple value containing the IP address and TCP port number of
      both ends of the connection.  For one side (usually the server),
      guessing the port number is a trivial exercise.  The client side
      may or may not be easy for an attacker to guess depending on a
      number of factors, most notably the operating system and
      application involved.

   2) A sequence number that will be used in the RST.  This sequence
      number will be a starting point for a series of guesses to attempt
      to present a RST segment to a connection endpoint that would be
      acceptable to it.  Any random value may be used to guess the
      starting sequence number.

   3) The window size that the two endpoints are using.  This value does
      NOT have to be the exact window size since a smaller value used in
      lieu of the correct one will just cause the attacker to generate
      more segments before succeeding in his mischief.  Most modern
      operating systems have a default window size that usually is
      applied to most connections.  Some applications however may change
      the window size to better suit the needs of the application.  So
      often times the attacker, with a fair degree of certainty (knowing
      the application that is under attack), can come up with a very
      close approximation as to the actual window size in use on the
      connection.

   After assembling the above set of information, the attacker begins
   sending spoofed TCP segments with the RST bit set and a guessed TCP
   sequence number.  Each time a new RST segment is sent, the sequence
   number guess is incremented by the window size.  The feasibility of
   this methodology (without mitigations) was first shown in [SITW ].
   This is because [RFC0793] specifies that any RST within the current
   window is acceptable.  Also, [RFC4953] talks about the probability of
   a successful attack with varying window sizes and bandwidth. 
   A slight enhancement to TCP's segment processing rules can be made,
   which makes such an attack much more difficult to accomplish.  If the
   receiver examines the incoming RST segment and validates that the
   sequence number exactly matches the sequence number that is next
   expected, then such an attack becomes much more difficult than
   outlined in [SITW ] (i.e., the attacker would have to generate 1/2 the
   entire sequence space, on average).  This document will discuss the
   exact details of what needs to be changed within TCP's segment
   processing rules to mitigate all three types of attacks (RST, SYN,
   and DATA).

# Referenced Sections from RFC 2119: Key words for use in RFCs to Indicate Requirement Levels

The following sections were referenced. Remaining sections are not included.

## 8. Acknowledgments

   The definitions of these terms are an amalgam of definitions taken
   from a number of RFCs.  In addition, suggestions have been
   incorporated from a number of people including Robert Ullmann, Thomas
   Narten, Neal McBurnett, and Robert Elz. 





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





# Referenced Sections from RFC 9000: QUIC: A UDP-Based Multiplexed and Secure Transport

The following sections were referenced. Remaining sections are not included.

### 1.2. Terms and Definitions

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 
   14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

   Commonly used terms in this document are described below.

   QUIC:  The transport protocol described by this document.  QUIC is a
      name, not an acronym.

   Endpoint:  An entity that can participate in a QUIC connection by
      generating, receiving, and processing QUIC packets.  There are
      only two types of endpoints in QUIC: client and server.

   Client:  The endpoint that initiates a QUIC connection.

   Server:  The endpoint that accepts a QUIC connection.

   QUIC packet:  A complete processable unit of QUIC that can be
      encapsulated in a UDP datagram.  One or more QUIC packets can be
      encapsulated in a single UDP datagram.

   Ack-eliciting packet:  A QUIC packet that contains frames other than
      ACK, PADDING, and CONNECTION_CLOSE.  These cause a recipient to
      send an acknowledgment; see Section 13.2.1.

   Frame:  A unit of structured protocol information.  There are
      multiple frame types, each of which carries different information.
      Frames are contained in QUIC packets.

   Address:  When used without qualification, the tuple of IP version,
      IP address, and UDP port number that represents one end of a
      network path.

   Connection ID:  An identifier that is used to identify a QUIC
      connection at an endpoint.  Each endpoint selects one or more
      connection IDs for its peer to include in packets sent towards the
      endpoint.  This value is opaque to the peer.

   Stream:  A unidirectional or bidirectional channel of ordered bytes
      within a QUIC connection.  A QUIC connection can carry multiple
      simultaneous streams.

   Application:  An entity that uses QUIC to send and receive data.

   This document uses the terms "QUIC packets", "UDP datagrams", and "IP
   packets" to refer to the units of the respective protocols.  That is,
   one or more QUIC packets can be encapsulated in a UDP datagram, which
   is in turn encapsulated in an IP packet.

# Referenced Sections from RFC 9260: Stream Control Transmission Protocol

The following sections were referenced. Remaining sections are not included.

### 1.2. Architectural View of SCTP

   SCTP is viewed as a layer between the SCTP user application ("SCTP
   user" for short) and a connectionless packet network service, such as
   IP.  The remainder of this document assumes SCTP runs on top of IP.
   The basic service offered by SCTP is the reliable transfer of user
   messages between peer SCTP users.  It performs this service within
   the context of an association between two SCTP endpoints.  Section 11   of this document sketches the API that exists at the boundary between
   SCTP and the SCTP upper layers.

   SCTP is connection oriented in nature, but the SCTP association is a
   broader concept than the TCP connection.  SCTP provides the means for
   each SCTP endpoint (Section 1.3) to provide the other endpoint
   (during association startup) with a list of transport addresses
   (i.e., multiple IP addresses in combination with an SCTP port)
   through which that endpoint can be reached and from which it will
   originate SCTP packets.  The association spans transfers over all of
   the possible source/destination combinations that can be generated
   from each endpoint's lists.

     _____________                                      _____________
    |  SCTP User  |                                    |  SCTP User  |
    | Application |                                    | Application |
    |-------------|                                    |-------------|
    |    SCTP     |                                    |    SCTP     |
    |  Transport  |                                    |  Transport  |
    |   Service   |                                    |   Service   |
    |-------------|                                    |-------------|
    |             |One or more    ----      One or more|             |
    | IP Network  |IP address      \/        IP address| IP Network  |
    |   Service   |appearances     /\       appearances|   Service   |
    |_____________|               ----                 |_____________|

      SCTP Node A |<-------- Network transport ------->| SCTP Node B

                       Figure 1: An SCTP Association

   In addition to encapsulating SCTP packets in IPv4 or IPv6, it is also
   possible to encapsulate SCTP packets in UDP as specified in [RFC6951]
   or encapsulate them in DTLS as specified in [RFC8261].

# Referenced Sections from RFC 6679: Explicit Congestion Notification (ECN) for RTP over UDP

The following sections were referenced. Remaining sections are not included.

### 8.3. Generating RTCP ECN Feedback in Media Transcoders

   An RTP translator that acts as a media transcoder cannot directly
   forward RTCP packets corresponding to the transcoded stream, since
   those packets will relate to the non-transcoded stream and will not
   be useful in relation to the transcoded RTP flow.  Such a transcoder
   will need to interpose itself into the RTCP flow, acting as a proxy
   for the receiver to generate RTCP feedback in the direction of the
   sender relating to the pre-transcoded stream and acting in place of
   the sender to generate RTCP relating to the transcoded stream to be
   sent towards the receiver.  This section describes how this proxying
   is to be done for RTCP ECN feedback packets.  Section 7.2 of
   [RFC3550] describes general procedures for other RTCP packet types.

   An RTP translator acting as a media transcoder in this manner does
   not have its own SSRC and hence is not visible to other entities at
   the RTP layer.  RTCP ECN feedback packets and RTCP XR report blocks 
   for ECN summary information that are received from downstream relate
   to the translated stream and so must be processed by the translator
   as if they were the original media source.  These reports drive the
   congestion control loop and media adaptation between the translator
   and the downstream receiver.  If there are multiple downstream
   receivers, a logically separate transcoder instance must be used for
   each receiver and must process RTCP ECN Feedback and Summary Reports
   independently of the other transcoder instances.  An RTP translator
   acting as a media transcoder in this manner MUST NOT forward RTCP ECN
   feedback packets or RTCP XR ECN Summary Reports from downstream
   receivers in the upstream direction.

   An RTP translator acting as a media transcoder will generate RTCP
   reports upstream towards the original media sender, based on the
   reception quality of the original media stream at the translator.
   The translator will run a separate congestion control loop and media
   adaptation between itself and the media sender for each of its
   downstream receivers and must generate RTCP ECN feedback packets and
   RTCP XR ECN Summary Reports for that congestion control loop using
   the SSRC of that downstream receiver.

# Referenced Sections from RFC 3540: Robust Explicit Congestion Notification (ECN) Signaling with Nonces

The following sections were referenced. Remaining sections are not included.

## 13. Authors' Addresses

   Neil Spring
   EMail: nspring@cs.washington.edu


   David Wetherall
   Department of Computer Science and Engineering, Box 352350
   University of Washington
   Seattle WA 98195-2350
   EMail: djw@cs.washington.edu


   David Ely
   Computer Science and Engineering, 352350
   University of Washington
   Seattle, WA 98195-2350
   EMail: ely@cs.washington.edu 





# Summary of reference from InfiniBand Trade Association, "InfiniBand Architecture Specification Volume 1, Release 1.4", 2020, <https://www.infinibandta.org/ibta-specification/>. (RoCEv2)

The InfiniBand Architecture Specification Volume 1, Release 1.4, published in April 2020, integrates the previously separate RoCE (RDMA over Converged Ethernet) and Virtualization Annexes into the main specification. ([infinibandta.org](https://www.infinibandta.org/ibta-enhances-data-center-performance-and-management-with-new-infiniband-architecture-specification-releases/?utm_source=openai)) This integration enhances the manageability and scalability of InfiniBand by providing a unified framework for RDMA operations over Ethernet networks.

RoCE enables Remote Direct Memory Access (RDMA) over Ethernet, allowing direct memory access between computers without involving their operating systems, thereby reducing latency and CPU overhead. The original RoCE specification (RoCEv1) operated over Layer 2 Ethernet networks, limiting its scalability. To address this, RoCEv2 was introduced, adding support for routing over Layer 3 networks by incorporating IP and UDP headers, thus enabling RDMA over more extensive and complex network topologies. ([infinibandta.org](https://www.infinibandta.org/infiniband-trade-association-releases-updated-specification-for-remote-direct-memory-access-over-converged-ethernet-roce/?utm_source=openai))

By incorporating RoCE and its enhancements directly into the InfiniBand Architecture Specification, Release 1.4 provides a comprehensive and standardized approach to implementing RDMA over Ethernet, facilitating improved performance and interoperability in data center environments. 

