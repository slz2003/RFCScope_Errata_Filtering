Let us analyze section 5 of RFC 9886. All references made by section 5 have also been included below.

# RFC 9886: DRIP Entity Tags (DETs) in the Domain Name System

## 1. Introduction

   Registries are fundamental to Unmanned Aircraft System (UAS) Remote
   Identification (RID).  Only very limited operational information can
   be sent via Broadcast RID, but extended information is sometimes
   needed.  The most essential element of information from RID is the
   UAS ID, the unique key for lookup of extended information in relevant
   registries (see Figure 1, which is the same as Figure 4 of
   [RFC9434]).

  ***************                                        ***************
  *    UAS1     *                                        *     UAS2    *
  *             *                                        *             *
  * +--------+  *                 DAA/V2V                *  +--------+ *
  * |   UA   o--*----------------------------------------*--o   UA   | *
  * +--o--o--+  *                                        *  +--o--o--+ *
  *    |  |     *   +------+      Lookups     +------+   *     |  |    *
  *    |  |     *   | GPOD o------.    .------o PSOD |   *     |  |    *
  *    |  |     *   +------+      |    |      +------+   *     |  |    *
  *    |  |     *                 |    |                 *     |  |    *
  * C2 |  |     *     V2I      ************     V2I      *     |  | C2 *
  *    |  '-----*--------------*          *--------------*-----'  |    *
  *    |        *              *          *              *        |    *
  *    |        o====Net-RID===*          *====Net-RID===o        |    *
  * +--o--+     *              * Internet *              *     +--o--+ *
  * | GCS o-----*--------------*          *--------------*-----o GCS | *
  * +-----+     * Registration *          * Registration *     +-----+ *
  *             * (and UTM)    *          * (and UTM)    *             *
  ***************              ************              ***************
                                 |  |  |
                  +----------+   |  |  |   +----------+
                  | Public   o---'  |  '---o Private  |
                  | Registry |      |      | Registry |
                  +----------+      |      +----------+
                                 +--o--+
                                 | DNS |
                                 +-----+

  DAA:  Detect And Avoid
  GPOD: General Public Observer Device
  PSOD: Public Safety Observer Device
  V2I:  Vehicle-to-Infrastructure
  V2V:  Vehicle-to-Vehicle

      Figure 1: Global UAS RID Usage Scenario (Figure 4 of RFC 9434)

   When a DRIP Entity Tag (DET) [RFC9374] is used as the UAS ID in RID,
   extended information can be retrieved from a DRIP Identity Management
   Entity (DIME), which manages registration of and associated lookups
   from DETs.  In this document it is assumed the DIME is a function of
   UAS Service Suppliers (USS) (Appendix A.2 of [RFC9434]), but a DIME
   can be independent or handled by another entity as well.

### 1.1. General Concept

   DRIP Entity Tags (DETs) embed a hierarchy scheme that is mapped onto
   the Domain Name System (DNS) [STD13].  DIMEs enforce registration and
   information access of data associated with a DET while also providing
   the trust inherited from being a member of the hierarchy.  Other
   identifiers and their methods are out of scope for this document.

   Authoritative name servers of the DNS provide the public information
   such as the cryptographic keys, endorsements and certificates of
   DETs, and pointers to private information resources.  Cryptographic
   (public) keys are used to authenticate anything signed by a DET, such
   as in the Authentication Messages defined in [RFC9575] for Broadcast
   RID.  Endorsements and certificates are used to endorse the claim of
   being part of the hierarchy.

   This document does not specify AAA mechanisms used by Private
   Information Registries to store and protect Personally Identifiable
   Information (PII).

### 1.2. Scope

   The scope of this document is the DNS registration of DETs with the
   DNS delegation of the reverse domain of the IPv6 prefix (2001:30::/28
   for DETs) and RRsets used to handle DETs.

## 2. Terminology





### 2.1. Required Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

### 2.2. Additional Definitions

   This document makes use of the terms and abbreviations from previous
   DRIP documents.  Below are subsets, grouped by original document, of
   terms used this document.  Please see those documents for additional
   context, definitions, and any further referenced material.

   From Section 2.2 of [RFC9153], this document uses: AAA, CAA, GCS,
   ICAO, PII, Observer, Operator, UA, UAS, USS, and UTM.

   From Section 2 of [RFC9434], this document uses: Certificate, DIME,
   and Endorsement.

   From Section 2 of [RFC9374], this document uses: HDA, HID, and RAA.

## 3. DET Hierarchy in DNS

   [RFC9374] defines the Hierarchical Host Identity Tags (HHIT) and
   further specifies an instance of them used for UAS RID called DET.
   The DET is a 128-bit value that is an IPv6 address intended primarily
   as an identifier rather than locator.  The format is shown in
   Figure 2 and further information is in [RFC9374].

    +-------------+--------------+---------------+-------------+
    | IPv6 Prefix | Hierarchy ID | HHIT Suite ID | ORCHID Hash |
    | (28 bits)   | (28 bits)    | (8 bits)      | (64 bits)   |
    +-------------+--------------+---------------+-------------+
                 /                \
                /                  \
               /                    \-----------------------------\
              /                                                    \
             /                                                      \
            +--------------------------------+-----------------------+
            | Registered Assigning Authority | HHIT Domain Authority |
            | (14 bits)                      | (14 bits)             |
            +--------------------------------+-----------------------+

                    Figure 2: DRIP Entity Tag Breakdown

   [RFC9374] assigns the IPv6 prefix 2001:30::/28 for DETs.  Its
   corresponding domain name for reverse lookups is
   3.0.0.1.0.0.2.ip6.arpa.. The IAB has administrative control of this
   domain name.

   Due to the nature of the hierarchy split and its relationship to
   nibble reversing of the IPv6 address (Section 2.5 of RFC 3596   [STD88]), the upper level of the hierarchy (i.e., Registered
   Assigning Authority (RAA)) "borrows" the upper two bits of their
   respective HHIT Domain Authority (HDA) space for DNS delegation.  As
   such, the IPv6 prefix of RAAs is 2001:3x:xxx0::/44 and HDAs is
   2001:3x:xxxy:yy00::/56 with respective nibble reverse domains of
   x.x.x.x.3.0.0.1.0.0.2.ip6.arpa. and
   y.y.y.x.x.x.x.3.0.0.1.0.0.2.ip6.arpa..

   This document preallocates a subset of RAAs based on the ISO 3166-1
   Numeric Nation Code [ISO3166-1].  This is to support the initial use
   case of DETs in UAS RID on an international level.  See Section 6.2.1   for the RAA allocations.

   The HDA values of 0, 4096, 8192, and 12288 are reserved for
   operational use of an RAA (a by-product of the above mentioned
   borrowing of bits), in particular to specify when to register with
   the apex and endorse delegations of HDAs in their namespace.

   The administration, management, and policy for the operation of a
   DIME at any level in the hierarchy (Apex, RAA or HDA) is out of scope
   for this document.  For RAAs or DETs allocated on a per-country
   basis, these considerations should be determined by the appropriate
   national authorities, presumably the Civil Aviation Authority (CAA).

### 3.1. Use of Existing DNS Models

   DRIP relies on the DNS and as such roughly follows the registrant-
   registrar-registry model.  In the UAS ecosystem, the registrant would
   be the end user who owns/controls the Unmanned Aircraft.  They are
   ultimately responsible for the DET and any other information that
   gets published in the DNS.  Registrants use agents known as
   registrars to manage their interactions with the registry.
   Registrars typically provide optional additional services such as DNS
   hosting.

   The registry maintains a database of the registered domain names and
   their related metadata such as the contact details for domain name
   holder and the relevant registrar.  The registry provides DNS service
   for the zone apex, which contains delegation information for domain
   names.  Registries generally provide services such as the
   Registration Data Access Protocol (RDAP) [STD95] or equivalent to
   publish metadata about the registered domain names and their
   registrants and registrars.

   Registrants have contracts with registrars who in turn have contracts
   with registries.  Payments follow this model too: the registrant buys
   services from a registrar who pays for services provided by the
   registry.

   By definition, there can only be one registry for a domain name.  A
   registry can have an arbitrary number of registrars who compete with
   each other on price, service, and customer support.

#### 3.1.1. DNS Model Considerations for DIMEs

     Apex
     Registry/Registrar
     (IANA)
                              +=========================+
                              | 3.0.0.1.0.0.2.ip6.arpa. |
                              +============o============+
                                           |
     --------------------------------------|-------------------------
     National                              |
     Registries/Registrars                 |
     (RAA)                                 |
                                           |
             +--------------+--------------o-+---------------+
             |              |                |               |
       +=====o====+    +====o=====+    +=====o====+    +=====o====+
       | 0.0.0.0. |    | 1.0.0.0. |    | 2.0.0.0. |    | 3.0.0.0. |
       +====o=====+    +====o=====+    +====o=====+    +====o=====+
                                            |
     ---------------------------------------|------------------------
     Local                                  |
     Registries/Registrars                  |
     (HDA)                                  |
                                            |
             +--------------+---------------o--------...-----+
             |              |               |                |
       +=====o====+    +====o=====+    +====o=====+    +=====o====+
       |  1.0.0.  |    |  2.0.0.  |    |  3.0.0.  |    |  f.f.f.  |
       +====o=====+    +=====o====+    +====o=====+    +====o=====+
                                            |
     ---------------------------------------|------------------------
     Local                                  |
     Registrants                            |
                      +=====================o================+
                      | x.x.x.x.x.x.x.x.x.x.x.x.x.x.x.x.5.0. |
                      +======================================+

                      Figure 3: Example DRIP DNS Model

   While the registrant-registrar-registry model is mature and well
   understood, it may not be appropriate for DRIP registrations in some
   circumstances.  It could add costs and complexity to develop policies
   and contracts as outlined above.  On the other hand, registries and
   registrars offer customer service and support and can provide the
   supporting infrastructure for reliable DNS and RDAP service.

   Another approach could be to handle DRIP registrations in a
   comparable way to how IP address space gets provisioned.  Here,
   blocks of addresses get delegated to a "trusted" third party,
   typically an ISP, who then issues IP addresses to its customers.  For
   DRIP, blocks of IP addresses could be delegated from the
   3.0.0.1.0.0.2.ip6.arpa. domain (reverse domain of prefix allocated by
   [RFC9374]) to an entity chosen by the appropriate Civil Aviation
   Authority (CAA).  This third party would be responsible for the
   corresponding DNS and RDAP infrastructure for these IP address
   blocks.  They would also provision the HHIT records [RFC9374] for
   these IP addresses.  In principle, a manufacturer or vendor of UAS
   devices could provide that role.  This is shown as an example in
   Figure 3.

   Dynamic DRIP registration is another possible solution, for example
   when the operator of a UAS device registers its corresponding HHIT
   record and other resources before a flight and deletes them
   afterwards.  This may be feasible in controlled environments with
   well-behaved actors.  However, this approach may not scale since each
   device is likely to need credentials for updating the IT
   infrastructure that provisions the DNS.

   Registration policies (pricing, renewals, registrar, and registrant
   agreements, etc.) will need to be developed.  These considerations
   should be determined by the CAA, perhaps in consultation with local
   stakeholders to assess the cost-benefits of these approaches (and
   others).  All of these are out of scope for this document.  The
   specifics for the UAS RID use case are detailed in the rest of
   document.

## 4. Public Information Registry

   Per [RFC9434], all information classified as public is stored in the
   DNS, specifically authoritative name servers, to satisfy REG-1 from
   [RFC9153].

   Authoritative name servers use domain names as identifiers and data
   is stored in Resource Records (RRs) with associated RRTypes.  This
   document defines two new RRTypes, one for HHIT metadata (HHIT,Section 5.1) and another for UAS Broadcast RID information (BRID,Section 5.2).  The former RRType is particularly important as it
   contains a URI (as part of the certificate) that points to Private
   Information resources.

   DETs, being IPv6 addresses, are to be under ip6.arpa. (nibble
   reversed per Section 2.5 of RFC 3596 [STD88]) and MUST resolve to an
   HHIT RRType.  Depending on local circumstances or additional use
   cases, other RRTypes MAY be present (for example the inclusion of the
   DS RRTypes or equivalent when using DNSSEC).  For UAS RID, the BRID
   RRType MUST be present to provide the Broadcast Endorsements (BEs)
   defined in Section 3.1.2.1 of [RFC9575].

   DNSSEC MUST be used for apex entities (those which use a self-signed
   Canonical Registration Certificate) and is RECOMMENDED for other
   entities.  When a DIME decides to use DNSSEC, they SHOULD define a
   framework for cryptographic algorithms and key management [RFC6841].
   This may be influenced by the frequency of updates, size of the zone,
   and policies.

   UAS-specific information, such as physical characteristics, may also
   be stored in DNS but is out of scope for this document.

   A DET IPv6 address gets mapped into domain names using the scheme
   defined in Section 2.5 of RFC 3596 [STD88].  However, DNS lookups of
   these names query for HHIT and/or BRID resource records rather than
   the PTR resource records conventionally used in reverse lookups of IP
   addresses.  For example, the HHIT resource record for the DET
   2001:30::1 would be returned from a DNS lookup for the HHIT QTYPE for
   1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.3.0.0.1.0.0.2.ip6.a
   rpa..

   The HHIT RRType provides the public key for signature verification
   and URIs via the certificate.  The BRID RRType provides static
   Broadcast RID information such as the Broadcast Endorsements sent as
   described in [RFC9575].

## 5. Resource Records





### 5.1. HHIT Resource Record

   The HHIT Resource Record (HHIT, RRType 67) is a metadata record for
   various bits of HHIT-specific information that isn't available in the
   pre-existing HIP RRType.  The HHIT RRType does not replace the HIP
   RRType [RFC8005].  The primary advantage of the HHIT RRType over the
   existing RRType is the mandatory inclusion of the Canonical
   Registration Certificate containing an entity's public key signed by
   the registrar, or other trust anchor, to confirm registration.

   The data MUST be encoded in the Concise Binary Object Representation
   (CBOR) [RFC8949] bytes.  The Concise Data Definition Language (CDDL)
   [RFC8610] of the data is provided in Figure 4.

#### 5.1.1. Text Representation

   The data are represented in base64 [RFC4648] and may be divided into
   any number of white-space-separated substrings, down to single base64
   digits, which are concatenated to obtain the full object.  These
   substrings can span lines using the standard parenthesis.  Note that
   the data has internal subfields but these do not appear in the zone
   file representation; only a single logical base64 string will appear.

##### 5.1.1.1. Presentation Representation

   The data MAY, for display purposes only, be represented using the
   Extended Diagnostic Notation as defined in Appendix G of [RFC8610].

#### 5.1.2. Field Descriptions

   hhit-rr = [
       hhit-entity-type: uint,
       hid-abbreviation: tstr .size(15),
       canonical-registration-cert: bstr
   ]

                      Figure 4: HHIT Wire Format CDDL

   All fields of the HHIT RRType MUST be included to be properly formed.

   HHIT Entity Type:  The HHIT Entity Type field is a number with values
      defined in Section 6.2.2.  It is envisioned that there may be many
      types of HHITs in use.  In some cases, it may be helpful to
      understand the role of the HHITs in the ecosystem, like that
      described in [drip-dki ].  This field provides such context.  This
      field MAY provide a signal of additional information and/or
      different handling of the data beyond what is defined in this
      document.

   HID Abbreviation:  The HID Abbreviation field is a string that
      provides an abbreviation to the HID (Hierarchy ID) structure of a
      DET for display devices.  The convention for such abbreviations is
      a matter of local policy.  Absent of such a policy, this field
      MUST be filled with the four character hexadecimal representations
      of the RAA and HDA (in that order) with a separator character,
      such as a space, in between.  For example, a DET with an RAA value
      of 10 and HDA value of 20 would be abbreviated as: 000A 0014.

   Canonical Registration Certificate:  The Canonical Registration
      Certificate field is for a certificate-endorsing registration of
      the DET.  It MUST be encoded as X.509 DER [RFC5280].  This
      certificate MAY be self-signed when the entity is acting as a root
      of trust (i.e., an apex).  Such self-signed behavior is defined by
      policy, such as in [drip-dki ], and is out of scope for this
      document.  This certificate is part of a chain of certificates
      that can be used to validate inclusion in the hierarchy.

### 5.2. UAS Broadcast RID Resource Record

   The UAS Broadcast RID Resource Record (BRID, RRType 68) is a format
   to hold information typically sent over UAS Broadcast RID that is
   static.  It can act as a data source if information is not received
   over Broadcast RID or for cross validation.  The primary function for
   DRIP is to include of one or more Broadcast Endorsements as defined
   in [RFC9575] in the auth field.  These Endorsements are generated by
   the registrar upon successful registration and broadcast by the
   entity.

   The data MUST be encoded in CBOR [RFC8949] bytes.  The CDDL [RFC8610]
   of the data is provided in Figure 5.

#### 5.2.1. Text Representation

   The data are represented in base64 [RFC4648] and may be divided into
   any number of white-space-separated substrings, down to single base64
   digits, which are concatenated to obtain the full object.  These
   substrings can span lines using the standard parenthesis.  Note that
   the data has internal subfields but these do not appear in the zone
   file representation; only a single logical base64 string will appear.

##### 5.2.1.1. Presentation Representation

   The data MAY, for display purposes only, be represented using the
   Extended Diagnostic Notation as defined in Appendix G of [RFC8610].
   All byte strings longer than a length of 20 SHOULD be displayed as
   base64 when possible.

#### 5.2.2. Field Descriptions

   bcast-rr = {
       uas_type => nibble-field,
       uas_ids => [+ uas-id-grp],
       ? auth => [+ auth-grp],
       ? self_id => self-grp,
       ? area => area-grp,
       ? classification => classification-grp,
       ? operator_id => operator-grp
   }
   uas-id-grp = [
       id_type: &uas-id-types,
       uas_id: bstr .size(20)
   ]
   auth-grp = [
       a_type: &auth-types,
       a_data: bstr .size(1..362)
   ]
   area-grp = [
       area_count: 1..255,
       area_radius: float,  # in decameters
       area_floor: float,   # wgs84-hae in meters
       area_ceiling: float  # wgs84-hae in meters
   ]
   classification-grp = [
       class_type: 0..8,
       class: nibble-field,
       category: nibble-field
   ]
   self-grp = [
       desc_type: 0..255,
       description: tstr .size(23)
   ]
   operator-grp = [
       operator_id_type: 0..255,
       operator_id: bstr .size(20)
   ]
   uas-id-types = (none: 0, serial: 1, session_id: 4)
   auth-types = (none: 0, specific_method: 5)
   nibble-field = 0..15
   uas_type = 0
   uas_ids = 1
   auth = 2
   self_id = 3
   area = 4
   classification = 5
   operator_id = 6

                      Figure 5: BRID Wire Format CDDL

   The field names and their general typing are taken from the ASTM data
   dictionary (Tables 1 and 2) [F3411].  See that document for
   additional context and background information on aviation
   application-specific interpretation of the field semantics.  The
   explicitly enumerated values included in the CDDL above are relevant
   to DRIP for its operation.  Other values may be valid but are outside
   the scope of DRIP operation.  Application-specific fields, such as
   UAS Type, are transported and authenticated by DRIP but are regarded
   as opaque user data to DRIP.

## 6. IANA Considerations





### 6.1. DET Prefix Delegation

   The reverse domain for the DET Prefix, i.e., 3.0.0.1.0.0.2.ip6.arpa.,
   is managed by IANA.  IANA will liaise, as needed, with the
   International Civil Aviation Organization (ICAO) to verify the
   authenticity of delegations to CAAs (see Section 6.2.1.4).

### 6.2. IANA DRIP Registry





#### 6.2.1. DRIP RAA Allocations

   IANA has created the registry for RAA Allocations under the "Drone
   Remote ID Protocol" registry group <https://www.iana.org/assignments/
   drip >.

   RAA Allocations:  a 14-bit value used to represent RAAs.  Future
      additions to this registry are to be made based on the following
      range and policy table:

   +===========+======================+================================+
   | RAA Range | Allocation           | Policy                         |
   +===========+======================+================================+
   | 0 - 3     | Reserved             |                                |
   +-----------+----------------------+--------------------------------+
   | 4 - 3999  | ISO 3166-1           | IESG Approval (Section 4.10 of |
   |           | Countries            | [RFC8126]), Section 6.2.1.4    |
   +-----------+----------------------+--------------------------------+
   | 4000 -    | Reserved             |                                |
   | 8191      |                      |                                |
   +-----------+----------------------+--------------------------------+
   | 8192 -    | Unassigned           | First Come First Served        |
   | 15359     |                      | (Section 4.4 of [RFC8126])     |
   +-----------+----------------------+--------------------------------+
   | 15360 -   | Private              | Private Use (Section 4.1 of    |
   | 16383     | Use                  | [RFC8126]), Section 6.2.1.5    |
   +-----------+----------------------+--------------------------------+

                            Table 1: RAA Ranges

##### 6.2.1.1. RAA Allocation Fields

   Value:  The RAA value delegated for this entry.

   Name:  Name of the delegated RAA.  For the ISO 3166-1 Countries
      (Section 6.2.1.4), this should be the name of the country.

   Reference:  A publicly accessible link to the policy requirements for
      prospective HDA operators to register under this RAA.  This field
      is OPTIONAL.

   Contact:  Contact details of the administrator of this RAA that
      prospective HDA operators can make informational queries to.

##### 6.2.1.2. RAA Registration Form

   Value:
   Name:
   Reference:
   Contact:
   NS RRType Content (HDA=0):
   NS RRType Content (HDA=4096):
   NS RRType Content (HDA=8192):
   NS RRType Content (HDA=12288):

                   Figure 6: RAA Delegation Request Form

   The NS RRType Content (HDA=X) fields are used by IANA to perform the
   DNS delegations under 3.0.0.1.0.0.2.ip6.arpa.. See Section 6.2.1.3   for technical details.

##### 6.2.1.3. Handling Nibble Split

   To support DNS delegation in 3.0.0.1.0.0.2.ip6.arpa., a single RAA is
   given 4 delegations by borrowing the upper two bits of HDA space (see
   Figure 7 for an example).  This enables a clean nibble boundary in
   the DNS to delegate from (i.e., the prefix 2001:3x:xxx0::/44).  These
   HDAs (0, 4096, 8192 and 12288) are reserved for the RAA.

 7 6 5 4 3 2 1 0 9 8 7 6 5 4 3 2 1 0 0 0 0 0 0 0 0 0 0 0
 2 2 2 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 9 8 7 6 5 4 3 2 1 0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           0           | x |         RAA=16376         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    0   |   0   |   0   |   0   |   E   |   F   |   F     HDA=0,x=00
    0   |   0   |   0   |   1   |   E   |   F   |   F     HDA=4096,x=01
    0   |   0   |   0   |   2   |   E   |   F   |   F     HDA=8192,x=10
    0   |   0   |   0   |   3   |   E   |   F   |   F     HDA=12288,x=11

             Figure 7: Example Bit Borrowing of RAA=16376

##### 6.2.1.4. ISO 3166-1 Countries Range

   The mapping between ISO 3166-1 Numeric Nation Codes and RAAs is
   specified and documented by IANA.  Each Nation is assigned 4 RAAs
   that are left to the national authority for their purpose.  For RAAs
   under this range, a shorter prefix of 2001:3x:xx00::/40 MAY be
   delegated to each CAA, which covers all 4 RAAs (and reserved HDAs)
   assigned to them.

   The registration policy for this range is set to IESG Approval
   (Section 4.10 of [RFC8126]) and IANA will liaise with the ICAO to
   verify the authenticity of delegation requests (using Figure 6) by
   CAAs.

##### 6.2.1.5. Private Range

   If nibble-reversed lookup in DNS is desired, it can only be provided
   by private DNS servers as zone delegations from the global root will
   not be performed for this address range.  Thus the RAAs (with its
   subordinate HDAs) in this range may be used in like manner and IANA
   will not delegate any value in this range to any party (as per
   Private Use, Section 4.1 of [RFC8126]).

   One anticipated acceptable use of the private range is for
   experimentation and testing prior to request allocation or assignment
   from IANA.

#### 6.2.2. HHIT Entity Types

   This document requests a new registry for HHIT Entity Types under the
   "Drone Remote ID Protocol" registry group
   <https://www.iana.org/assignments/drip >.

   HHIT Entity Type:  Numeric, field of the HHIT RRType to encode the
      HHIT Entity Type.  All entries in this registry are under the
      First Come First Served policy (Section 4.4 of [RFC8126]).

##### 6.2.2.1. HHIT Type Registry Fields

   Value:  HHIT Type value of the entry.

   HHIT Type:  Name of the entry and an optional abbreviation.

   Reference:  Public document allocating the value and any additional
      information such as semantics.  This can be a URL pointing to an
      Internet-Draft, IETF RFC, or web page among others.

##### 6.2.2.2. HHIT Type Registration Form

   Value:
   HHIT Type:
   Reference:

                   Figure 8: HHIT Type Registration Form

##### 6.2.2.3. Initial Values

   The following values are defined by this document:

     +=======+===========================================+===========+
     | Value | HHIT Type                                 | Reference |
     +=======+===========================================+===========+
     | 0     | Not Defined                               | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 1     | DRIP Identity Management Entity (DIME)    | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 5     | Apex                                      | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 9     | Registered Assigning Authority (RAA)      | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 13    | HHIT Domain Authority (HDA)               | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 16    | Unmanned Aircraft (UA)                    | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 17    | Ground Control Station (GCS)              | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 18    | Unmanned Aircraft System (UAS)            | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 19    | Remote Identification (RID) Module        | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 20    | Pilot                                     | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 21    | Operator                                  | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 22    | Discovery & Synchronization Service (DSS) | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 23    | UAS Service Supplier (USS)                | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 24    | Network RID Service Provider (SP)         | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 25    | Network RID Display Provider (DP)         | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 26    | Supplemental Data Service Provider (SDSP) | RFC 9886  |
     +-------+-------------------------------------------+-----------+
     | 27    | Crowd Sourced RID Finder                  | RFC 9886  |
     +-------+-------------------------------------------+-----------+

                  Table 2: HHIT Entity Type Initial Values

## 7. Security Considerations





### 7.1. DNS Operational and Registration Considerations

   The Registrar and Registry are commonly used concepts in the DNS.
   These components connect the DIME with the DNS hierarchy and thus
   operation SHOULD follow best common practices, specifically in
   security (such as running DNSSEC) as appropriate except when national
   regulations prevent it.  [BCP237] provides suitable guidance.

   If DNSSEC is used, a DNSSEC Practice Statement SHOULD be developed
   and published.  It SHOULD explain how DNSSEC has been deployed and
   what security measures are in place.  [RFC6841] documents a framework
   for DNSSEC policies and DNSSEC Practice Statements.  A self-signed
   entity (i.e., an entity that self-signed its certificate as part of
   the HHIT RRType) MUST use DNSSEC.

   The interfaces and protocol specifications for registry-registrar
   interactions are intentionally not specified in this document.  These
   will depend on nationally defined policy and prevailing local
   circumstances.  It is expected that registry-registrar activity will
   use the Extensible Provisioning Protocol (EPP) [STD69] or equivalent.
   The registry SHOULD provide a lookup service such as RDAP [STD95] or
   equivalent to publish public information about registered domain
   names.

   Decisions about DNS or registry best practices and other operational
   matters that influence security SHOULD be made by the CAA, ideally in
   consultation with local stakeholders.

   The guidance above is intended to reduce the likelihood of
   interoperability problems and minimize security and stability
   concerns.  For instance, validation and authentication of DNS
   responses depends on DNSSEC.  If this is not used, entities using
   DRIP will be vulnerable to DNS spoofing attacks and could be exposed
   to bogus data.  DRIP DNS responses that have not been validated by
   DNSSEC could contain bogus data that have the potential to create
   serious security problems and operational concerns for DRIP entities.
   These threats include denial-of-service attacks, replay attacks,
   impersonation or cloning of UAVs, hijacking of DET registrations,
   injection of corrupt metadata, and compromising established trust
   architecture/relationships.  Some regulatory and legal considerations
   are expected to be simplified by providing a lookup service for
   access to public information about registered domain names for DETs.

   When DNSSEC is not in use, due to the length of the ORCHID hash
   selected for DETs (Section 3.5 of [RFC9374]), clients MUST "walk" the
   tree of certificates locating each certificate by performing DNS
   lookups of HHIT RRTypes for each DET verifying inclusion into the
   hierarchy.  The collection of these certificates (which provide both
   signature protection from the parent entity and the public key of the
   entity) is the only way without DNSSEC to prove valid registration.

   The contents of the BRID RRType auth key, containing Endorsements as
   described in Section 4.2 of [RFC9575], are a shadow of the X.509
   certificate found in the HHIT RRType.  The validation of these
   Endorsements follow the guidelines written in Section 6.4.2 of
   [RFC9575] for Broadcast RID Observers and when present MUST also be
   validated.

### 7.2. DET and Public Key Exposure

   DETs are built upon asymmetric keys.  As such the public key must be
   revealed to enable clients to perform signature verifications.
   [RFC9374] security considerations cover various attacks on such keys.
   While unlikely, the forging of a corresponding private key is
   possible if given enough time (and computational power).

   When practical, it is RECOMMENDED that no RRTypes under a DET's
   specific domain name be published unless and until it is required for
   use by other parties.  Such action would cause at least the HHIT
   RRType to not be in the DNS, protecting the public key in the
   certificate from being exposed before its needed.  The combination of
   this "just in time" publishing mechanism and DNSSEC is out of scope
   for this document.

   Optimally this requires that the UAS somehow signal to the DIME that
   a flight using a Specific Session ID will soon be underway or
   complete.  It may also be facilitated under UTM if the USS (which may
   or may not be a DIME) signals when a given operation using a Session
   ID goes active.

## A. Example Zone Files and RRType Contents

   An example zone file ip6.arpa., run by IANA, is not shown.  It would
   contain NS RRTypes to delegate to a respective RAA.  To avoid any
   future collisions with production deployments an apex of
   ip6.example.com. is used instead of ip6.arpa.. All hexadecimal
   strings in the examples are broken into the lengths of a word, for
   document formatting purposes.

   An RAA with a HID of RAA=16376, HDA=0 and HDA with a the HID
   RAA=16376, HDA=10 were used in the examples.

### A.1. Example RAA





#### A.1.1. Authentication HHIT

   $ORIGIN 5.0.0.0.0.0.e.f.f.3.0.0.1.0.0.2.ip6.example.com.
   7.b.0.a.1.9.e.1.7.5.1.a.0.6.e.5. IN HHIT (
       gwppM2ZmOCAwMDAwWQFGMIIBQjCB9aAD
       AgECAgE1MAUGAytlcDArMSkwJwYDVQQD
       DCAyMDAxMDAzZmZlMDAwMDA1NWU2MGEx
       NTcxZTkxYTBiNzAeFw0yNTA0MDkyMDU2
       MjZaFw0yNTA0MDkyMTU2MjZaMB0xGzAZ
       BgNVBAMMEkRSSVAtUkFBLUEtMTYzNzYt
       MDAqMAUGAytlcAMhAJmQ1bBLcqGAZtQJ
       K1LH1JlPt8Fr1+jB9ED/qNBP8eE/o0ww
       SjAPBgNVHRMBAf8EBTADAQH/MDcGA1Ud
       EQEB/wQtMCuHECABAD/+AAAFXmChVx6R
       oLeGF2h0dHBzOi8vcmFhLmV4YW1wbGUu
       Y29tMAUGAytlcANBALUPjhIB3rwqXQep
       r9/VDB+hhtwuWZIw1OUkEuDrF6DCkgc7
       5widXnXa5/uDfdKL7dZ83mPHm2Tf32Dv
       b8AzEw8=
   )

                   Figure 9: RAA Auth HHIT RRType Example

   Figure 10 shows the CBOR decoded RDATA in the HHIT RRType found in
   Figure 9.

   [
       10,  # Reserved (RAA Auth from DKI)
       "3ff8 0000",
       h'308201423081F5A00302010202013530
       0506032B6570302B312930270603550403
       0C20323030313030336666653030303030
       3535653630613135373165393161306237
       301E170D3235303430393230353632365A
       170D3235303430393231353632365A301D
       311B301906035504030C12445249502D52
       41412D412D31363337362D30302A300506
       032B65700321009990D5B04B72A18066D4
       092B52C7D4994FB7C16BD7E8C1F440FFA8
       D04FF1E13FA34C304A300F0603551D1301
       01FF040530030101FF30370603551D1101
       01FF042D302B87102001003FFE0000055E
       60A1571E91A0B7861768747470733A2F2F
       7261612E6578616D706C652E636F6D3005
       06032B6570034100B50F8E1201DEBC2A5D
       07A9AFDFD50C1FA186DC2E599230D4E524
       12E0EB17A0C292073BE7089D5E75DAE7FB
       837DD28BEDD67CDE63C79B64DFDF60EF6F
       C033130F'
   ]

     Figure 10: 2001:3f:fe00:5:5e60:a157:1e91:a0b7 Decoded HHIT RRType
                                    CBOR

   Figure 11 shows the decoded DER X.509 found in Figure 10.

   Certificate:
       Data:
           Version: 3 (0x2)
           Serial Number: 53 (0x35)
           Signature Algorithm: ED25519
           Issuer: CN = 2001003ffe0000055e60a1571e91a0b7
           Validity
               Not Before: Apr  9 20:56:26 2025 GMT
               Not After : Apr  9 21:56:26 2025 GMT
           Subject: CN = DRIP-RAA-A-16376-0
           Subject Public Key Info:
               Public Key Algorithm: ED25519
                   ED25519 Public-Key:
                   pub:
                       99:90:d5:b0:4b:72:a1:80:66:d4:09:2b:52:c7:d4:
                       99:4f:b7:c1:6b:d7:e8:c1:f4:40:ff:a8:d0:4f:f1:
                       e1:3f
           X509v3 extensions:
               X509v3 Basic Constraints: critical
                   CA:TRUE
               X509v3 Subject Alternative Name: critical
                   IP Address:2001:3F:FE00:5:5E60:A157:1E91:A0B7,
                   URI:https://raa.example.com
       Signature Algorithm: ED25519
       Signature Value:
           b5:0f:8e:12:01:de:bc:2a:5d:07:a9:af:df:d5:0c:1f:a1:86:
           dc:2e:59:92:30:d4:e5:24:12:e0:eb:17:a0:c2:92:07:3b:e7:
           08:9d:5e:75:da:e7:fb:83:7d:d2:8b:ed:d6:7c:de:63:c7:9b:
           64:df:df:60:ef:6f:c0:33:13:0f

     Figure 11: 2001:3f:fe00:5:5e60:a157:1e91:a0b7 Decoded Certificate

#### A.1.2. Delegation of HDA

   $ORIGIN c.d.f.f.3.0.0.1.0.0.2.ip6.example.com.
   a.0.0. IN NS ns1.hda-10.example.com

                     Figure 12: HDA Delegation Example

### A.2. Example HDA





#### A.2.1. Authentication and Issue HHITs

   $ORIGIN 5.0.a.0.0.0.e.f.f.3.0.0.1.0.0.2.ip6.example.com.
   0.a.9.0.7.2.4.d.5.4.e.e.5.1.6.6.5.0. IN HHIT (
       gw5pM2ZmOCAwMDBhWQFHMIIBQzCB9qAD
       AgECAgFfMAUGAytlcDArMSkwJwYDVQQD
       DCAyMDAxMDAzZmZlMDAwMDA1NWU2MGEx
       NTcxZTkxYTBiNzAeFw0yNTA0MDkyMTAz
       MTlaFw0yNTA0MDkyMjAzMTlaMB4xHDAa
       BgNVBAMME0RSSVAtSERBLUEtMTYzNzYt
       MTAwKjAFBgMrZXADIQDOaB424RQa61YN
       bna8eWt7fLRU5GPMsfEt4wo4AQGAP6NM
       MEowDwYDVR0TAQH/BAUwAwEB/zA3BgNV
       HREBAf8ELTArhxAgAQA//gAKBWYV7kXU
       JwmghhdodHRwczovL3JhYS5leGFtcGxl
       LmNvbTAFBgMrZXADQQAhMpOSOmgMkJY1
       f+B9nTgawUjK4YEERBtczMknHDkOowX0
       ynbaLN60TYe9hqN6+CJ3SN8brJke3hpM
       gorvhDkJ
   )
   8.2.e.6.5.2.b.6.7.3.4.d.e.0.6.2.5.0. IN HHIT (
       gw9pM2ZmOCAwMDBhWQFHMIIBQzCB9qAD
       AgECAgFYMAUGAytlcDArMSkwJwYDVQQD
       DCAyMDAxMDAzZmZlMDAwYTA1NjYxNWVl
       NDVkNDI3MDlhMDAeFw0yNTA0MDkyMTA1
       MTRaFw0yNTA0MDkyMjA1MTRaMB4xHDAa
       BgNVBAMME0RSSVAtSERBLUktMTYzNzYt
       MTAwKjAFBgMrZXADIQCCM/2utQaLwUhZ
       0ROg7fz43AeBTj3Sdl5rW4LgTQcFl6NM
       MEowDwYDVR0TAQH/BAUwAwEB/zA3BgNV
       HREBAf8ELTArhxAgAQA//gAKBSYO1Ddr
       JW4ohhdodHRwczovL2hkYS5leGFtcGxl
       LmNvbTAFBgMrZXADQQBa8lZyftxHJqDF
       Vgv4Rt+cMUmc8aQwet4UZdO3yQOB9uq4
       sLVAScaZCWjC0nmeRkgVRhize1esfyi3
       RRU44IAE
   )

               Figure 13: HDA Auth/Issue HHIT RRType Example

   Figure 14 shows the CBOR decoded RDATA in the HHIT RRType found in
   Figure 13.

   [
       14,  # Reserved (HDA Auth from DKI)
       "3ff8 000a",
       h'308201433081F6A00302010202015F30
       0506032B6570302B312930270603550403
       0C20323030313030336666653030303030
       3535653630613135373165393161306237
       301E170D3235303430393231303331395A
       170D3235303430393232303331395A301E
       311C301A06035504030C13445249502D48
       44412D412D31363337362D3130302A3005
       06032B6570032100CE681E36E1141AEB56
       0D6E76BC796B7B7CB454E463CCB1F12DE3
       0A380101803FA34C304A300F0603551D13
       0101FF040530030101FF30370603551D11
       0101FF042D302B87102001003FFE000A05
       6615EE45D42709A0861768747470733A2F
       2F7261612E6578616D706C652E636F6D30
       0506032B6570034100213293923A680C90
       96357FE07D9D381AC148CAE18104441B5C
       CCC9271C390EA305F4CA76DA2CDEB44D87
       BD86A37AF8227748DF1BAC991EDE1A4C82
       8AEF843909'
   ]

        Figure 14: 2001:3f:fe00:a05:6615:ee45:d427:9a0 Decoded HHIT
                                RRType CBOR

   Figure 15 shows the decoded DER X.509 found in Figure 14.

   Certificate:
       Data:
           Version: 3 (0x2)
           Serial Number: 95 (0x5f)
           Signature Algorithm: ED25519
           Issuer: CN = 2001003ffe0000055e60a1571e91a0b7
           Validity
               Not Before: Apr  9 21:03:19 2025 GMT
               Not After : Apr  9 22:03:19 2025 GMT
           Subject: CN = DRIP-HDA-A-16376-10
           Subject Public Key Info:
               Public Key Algorithm: ED25519
                   ED25519 Public-Key:
                   pub:
                       ce:68:1e:36:e1:14:1a:eb:56:0d:6e:76:bc:79:6b:
                       7b:7c:b4:54:e4:63:cc:b1:f1:2d:e3:0a:38:01:01:
                       80:3f
           X509v3 extensions:
               X509v3 Basic Constraints: critical
                   CA:TRUE
               X509v3 Subject Alternative Name: critical
                   IP Address:2001:3F:FE00:A05:6615:EE45:D427:9A0,
                   URI:https://raa.example.com
       Signature Algorithm: ED25519
       Signature Value:
           21:32:93:92:3a:68:0c:90:96:35:7f:e0:7d:9d:38:1a:c1:48:
           ca:e1:81:04:44:1b:5c:cc:c9:27:1c:39:0e:a3:05:f4:ca:76:
           da:2c:de:b4:4d:87:bd:86:a3:7a:f8:22:77:48:df:1b:ac:99:
           1e:de:1a:4c:82:8a:ef:84:39:09

     Figure 15: 2001:3f:fe00:a05:6615:ee45:d427:9a0 Decoded Certificate

   Figure 16 shows the CBOR decoded RDATA in the HHIT RRType found in
   Figure 13.

   [
       15,  # Reserved (HDA Issue from DKI)
       "3ff8 000a",
       h'308201433081F6A00302010202015830
       0506032B6570302B312930270603550403
       0C20323030313030336666653030306130
       3536363135656534356434323730396130
       301E170D3235303430393231303531345A
       170D3235303430393232303531345A301E
       311C301A06035504030C13445249502D48
       44412D492D31363337362D3130302A3005
       06032B65700321008233FDAEB5068BC148
       59D113A0EDFCF8DC07814E3DD2765E6B5B
       82E04D070597A34C304A300F0603551D13
       0101FF040530030101FF30370603551D11
       0101FF042D302B87102001003FFE000A05
       260ED4376B256E28861768747470733A2F
       2F6864612E6578616D706C652E636F6D30
       0506032B65700341005AF256727EDC4726
       A0C5560BF846DF9C31499CF1A4307ADE14
       65D3B7C90381F6EAB8B0B54049C6990968
       C2D2799E4648154618B37B57AC7F28B745
       1538E08004'
   ]

        Figure 16: 2001:3f:fe00:a05:260e:d437:6b25:6e28 Decoded HHIT
                                RRType CBOR

   Figure 17 shows the decoded DER X.509 found in Figure 16.

   Certificate:
       Data:
           Version: 3 (0x2)
           Serial Number: 88 (0x58)
           Signature Algorithm: ED25519
           Issuer: CN = 2001003ffe000a056615ee45d42709a0
           Validity
               Not Before: Apr  9 21:05:14 2025 GMT
               Not After : Apr  9 22:05:14 2025 GMT
           Subject: CN = DRIP-HDA-I-16376-10
           Subject Public Key Info:
               Public Key Algorithm: ED25519
                   ED25519 Public-Key:
                   pub:
                       82:33:fd:ae:b5:06:8b:c1:48:59:d1:13:a0:ed:fc:
                       f8:dc:07:81:4e:3d:d2:76:5e:6b:5b:82:e0:4d:07:
                       05:97
           X509v3 extensions:
               X509v3 Basic Constraints: critical
                   CA:TRUE
               X509v3 Subject Alternative Name: critical
                   IP Address:2001:3F:FE00:A05:260E:D437:6B25:6E28,
                   URI:https://hda.example.com
       Signature Algorithm: ED25519
       Signature Value:
           5a:f2:56:72:7e:dc:47:26:a0:c5:56:0b:f8:46:df:9c:31:49:
           9c:f1:a4:30:7a:de:14:65:d3:b7:c9:03:81:f6:ea:b8:b0:b5:
           40:49:c6:99:09:68:c2:d2:79:9e:46:48:15:46:18:b3:7b:57:
           ac:7f:28:b7:45:15:38:e0:80:04

    Figure 17: 2001:3f:fe00:a05:260e:d437:6b25:6e28 Decoded Certificate

#### A.2.2. Registrant HHIT and BRID

   $ORIGIN 5.0.a.0.0.0.e.f.f.3.0.0.1.0.0.2.ip6.example.com.
   2.b.6.c.b.4.a.9.9.6.4.2.8.0.3.1. IN HHIT (
       gxJpM2ZmOCAwMDBhWQEYMIIBFDCBx6AD
       AgECAgFUMAUGAytlcDArMSkwJwYDVQQD
       DCAyMDAxMDAzZmZlMDAwYTA1MjYwZWQ0
       Mzc2YjI1NmUyODAeFw0yNTA0MDkyMTEz
       MDBaFw0yNTA0MDkyMjEzMDBaMAAwKjAF
       BgMrZXADIQDJLi+dl+iWD5tfFlT4sJA5
       +drcW88GHqxPDOp56Oh3+qM7MDkwNwYD
       VR0RAQH/BC0wK4cQIAEAP/4ACgUTCCRp
       mkvGsoYXaHR0cHM6Ly9oZGEuZXhhbXBs
       ZS5jb20wBQYDK2VwA0EA0DbcdngC7/BB
       /aLjZmLieo0ZFCDbd/KIxAy+3X2KtT4J
       todVxRMPAkN6o008gacbNfTG8p9npEcD
       eYhesl2jBQ==
   )
   2.b.6.c.b.4.a.9.9.6.4.2.8.0.3.1. IN BRID (
       owAAAYIEUQEgAQA//gAKBRMIJGmaS8ay
       AogFWIkB+t72Zwrt9mcgAQA//gAABV5g
       oVcekaC3mZDVsEtyoYBm1AkrUsfUmU+3
       wWvX6MH0QP+o0E/x4T8gAQA//gAABV5g
       oVcekaC3vC9m1JguvXt7W2o4wxPumaT1
       IP3TQN3fQP28hpInSIlsSwq8UCNjm2ad
       7pdTvm2EqfOJQNPKClvRZm4qTO5FDAVY
       iQGX4PZnp+72ZyABAD/+AAoFZhXuRdQn
       CaDOaB424RQa61YNbna8eWt7fLRU5GPM
       sfEt4wo4AQGAPyABAD/+AAAFXmChVx6R
       oLfv3q+mLRB3ya5TmjY8+3CzdoDZT9RZ
       +XpN5hDiA6JyyxBJvUewxLzPNhTXQp8v
       ED71XAE82tMmt3fB4zbzWNQLBViJAQrh
       9mca7/ZnIAEAP/4ACgUmDtQ3ayVuKIIz
       /a61BovBSFnRE6Dt/PjcB4FOPdJ2Xmtb
       guBNBwWXIAEAP/4ACgVmFe5F1CcJoIjy
       CriJCxAyAWTOHPmlHL02MKSpsHviiTze
       qwBH9K/Rrz41CYix9HazAIOAZO8FcfU5
       M+WLLJZoaQWBHnMbTQwFWIkB3OL2Z+zw
       9mcgAQA//gAKBRMIJGmaS8ayyS4vnZfo
       lg+bXxZU+LCQOfna3FvPBh6sTwzqeejo
       d/ogAQA//gAKBSYO1DdrJW4ogOfc8jTi
       mYLmTOOyFZoUx2jOOwtB1jnqUJr6bYaw
       MoPrR3MlKGBGWsVz1yXNqUURoCqYdwsY
       e61vd5i6YJqnAQ==
   )

               Figure 18: Registrant HHIT/BRID RRType Example

   Figure 19 shows the CBOR decoded RDATA in the HHIT RRType found in
   Figure 18.

   [
       18,  # Uncrewed Aircraft System (UAS)
       "3ff8 000a",
       h'308201143081C7A00302010202015430
       0506032B6570302B312930270603550403
       0C20323030313030336666653030306130
       3532363065643433373662323536653238
       301E170D3235303430393231313330305A
       170D3235303430393232313330305A3000
       302A300506032B6570032100C92E2F9D97
       E8960F9B5F1654F8B09039F9DADC5BCF06
       1EAC4F0CEA79E8E877FAA33B3039303706
       03551D110101FF042D302B87102001003F
       FE000A05130824699A4BC6B28617687474
       70733A2F2F6864612E6578616D706C652E
       636F6D300506032B6570034100D036DC76
       7802EFF041FDA2E36662E27A8D191420DB
       77F288C40CBEDD7D8AB53E09B68755C513
       0F02437AA34D3C81A71B35F4C6F29F67A4
       470379885EB25DA305'
   ]

        Figure 19: 2001:3f:fe00:a05:1308:2469:9a4b:c6b2 Decoded HHIT
                                RRType CBOR

   Figure 20 shows the decoded DER X.509 found in Figure 19.

   Certificate:
       Data:
           Version: 3 (0x2)
           Serial Number: 84 (0x54)
           Signature Algorithm: ED25519
           Issuer: CN = 2001003ffe000a05260ed4376b256e28
           Validity
               Not Before: Apr  9 21:13:00 2025 GMT
               Not After : Apr  9 22:13:00 2025 GMT
           Subject:
           Subject Public Key Info:
               Public Key Algorithm: ED25519
                   ED25519 Public-Key:
                   pub:
                       c9:2e:2f:9d:97:e8:96:0f:9b:5f:16:54:f8:b0:90:
                       39:f9:da:dc:5b:cf:06:1e:ac:4f:0c:ea:79:e8:e8:
                       77:fa
           X509v3 extensions:
               X509v3 Subject Alternative Name: critical
                   IP Address:2001:3F:FE00:A05:1308:2469:9A4B:C6B2,
                   URI:https://hda.example.com
       Signature Algorithm: ED25519
       Signature Value:
           d0:36:dc:76:78:02:ef:f0:41:fd:a2:e3:66:62:e2:7a:8d:19:
           14:20:db:77:f2:88:c4:0c:be:dd:7d:8a:b5:3e:09:b6:87:55:
           c5:13:0f:02:43:7a:a3:4d:3c:81:a7:1b:35:f4:c6:f2:9f:67:
           a4:47:03:79:88:5e:b2:5d:a3:05

    Figure 20: 2001:3f:fe00:a05:1308:2469:9a4b:c6b2 Decoded Certificate

   Figure 21 shows the CBOR decoded RDATA of the BRID RRType in
   Figure 18.

   {
       0: 0,
       1: [4, h'012001003FFE000A05130824699A4BC6B2'],
       2: [
           5,
           h'01FADEF6670AEDF6672001003FFE0000
           055E60A1571E91A0B79990D5B04B72A180
           66D4092B52C7D4994FB7C16BD7E8C1F440
           FFA8D04FF1E13F2001003FFE0000055E60
           A1571E91A0B7BC2F66D4982EBD7B7B5B6A
           38C313EE99A4F520FDD340DDDF40FDBC86
           922748896C4B0ABC5023639B669DEE9753
           BE6D84A9F38940D3CA0A5BD1666E2A4CEE
           450C',
           5,
           h'0197E0F667A7EEF6672001003FFE000A
           056615EE45D42709A0CE681E36E1141AEB
           560D6E76BC796B7B7CB454E463CCB1F12D
           E30A380101803F2001003FFE0000055E60
           A1571E91A0B7EFDEAFA62D1077C9AE539A
           363CFB70B37680D94FD459F97A4DE610E2
           03A272CB1049BD47B0C4BCCF3614D7429F
           2F103EF55C013CDAD326B777C1E336F358
           D40B',
           5,
           h'010AE1F6671AEFF6672001003FFE000A
           05260ED4376B256E288233FDAEB5068BC1
           4859D113A0EDFCF8DC07814E3DD2765E6B
           5B82E04D0705972001003FFE000A056615
           EE45D42709A088F20AB8890B10320164CE
           1CF9A51CBD3630A4A9B07BE2893CDEAB00
           47F4AFD1AF3E350988B1F476B300838064
           EF0571F53933E58B2C96686905811E731B
           4D0C',
           5,
           h'01DCE2F667ECF0F6672001003FFE000A
           05130824699A4BC6B2C92E2F9D97E8960F
           9B5F1654F8B09039F9DADC5BCF061EAC4F
           0CEA79E8E877FA2001003FFE000A05260E
           D4376B256E2880E7DCF234E29982E64CE3
           B2159A14C768CE3B0B41D639EA509AFA6D
           86B03283EB4773252860465AC573D725CD
           A94511A02A98770B187BAD6F7798BA609A
           A701'
       ]
   }

        Figure 21: 2001:3f:fe00:a05:1308:2469:9a4b:c6b2 Decoded BRID
                                RRType CBOR



---
# Referenced Sections from RFC 8005: Host Identity Protocol (HIP) Domain Name System (DNS) Extension

The following sections were referenced. Remaining sections are not included.

### 4.1. Storing HI, HIT, and RVS in the DNS

   For any HIP node, its HI, the associated HIT, and the FQDN of its
   possible RVSs can be stored in a DNS HIP RR.  Any conforming
   implementation may store an HI and its associated HIT in a DNS HIP
   RDATA format.  HI and HIT are defined in Section 3 of the HIP
   specification [RFC7401]. 
   Upon return of a HIP RR, a host MUST always calculate the
   HI-derivative HIT to be used in the HIP exchange, as specified in Section 3 of the HIP specification [RFC7401], while the HIT included
   in the HIP RR SHOULD only be used as an optimization (e.g., table
   lookup).

   The HIP RR may also contain one or more domain names of one or more
   RVSs towards which HIP I1 packets might be sent to trigger the
   establishment of an association with the entity named by this RR
   [RFC8004].

   The Rendezvous Server field of the HIP RR stored at a given owner
   name MAY include the owner name itself.  A semantically equivalent
   situation occurs if no RVS is present in the HIP RR stored at that
   owner name.  Such situations occur in two cases:

   o  The host is mobile, and the A and/or AAAA RR(s) stored at its host
      name contain the IP address(es) of its RVS rather than its own
      one.

   o  The host is stationary and can be reached directly at the IP
      address(es) contained in the A and/or AAAA RR(s) stored at its
      host name.  This is a degenerate case of rendezvous service where
      the host somewhat acts as an RVS for itself.

   An RVS receiving such an I1 would then relay it to the appropriate
   Responder (the owner of the I1 receiver HIT).  The Responder will
   then complete the exchange with the Initiator, typically without
   ongoing help from the RVS.

## 5. HIP RR Storage Format

   The RDATA for a HIP RR consists of a PK Algorithm Type, the HIT
   length, a HIT, a PK (i.e., an HI), and optionally one or more RVSs.

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  HIT length   | PK algorithm  |          PK length            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   ~                           HIT                                 ~
   |                                                               |
   +                     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                     |                                         |
   +-+-+-+-+-+-+-+-+-+-+-+                                         +
   |                           Public Key                          |
   ~                                                               ~
   |                                                               |
   +                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                               |                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
   |                                                               |
   ~                       Rendezvous Servers                      ~
   |                                                               |
   +             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |             |
   +-+-+-+-+-+-+-+

   The HIT length, PK algorithm, PK length, HIT, and Public Key fields
   are REQUIRED.  The Rendezvous Server field is OPTIONAL.

### 5.1. HIT Length Format

   The HIT length indicates the length in bytes of the HIT field.  This
   is an 8-bit unsigned integer.

### 5.2. PK Algorithm Format

   The PK algorithm field indicates the PK cryptographic algorithm and
   the implied Public Key field format.  This is an 8-bit unsigned
   integer.  This document reuses the values defined for the 'Algorithm
   Type' of the IPSECKEY RR [RFC4025]. 
   Presently defined values are listed in Section 9 for reference.

### 5.3. PK Length Format

   The PK length indicates the length in bytes of the Public Key field.
   This is a 16-bit unsigned integer in network byte order.

### 5.4. HIT Format

   The HIT is stored as a binary value in network byte order.

### 5.5. Public Key Format

   Two of the PK types defined in this document (RSA and Digital
   Signature Algorithm (DSA)) reuse the PK formats defined for the
   IPSECKEY RR [RFC4025].

   The DSA key format is defined in RFC 2536 [RFC2536].

   The RSA key format is defined in RFC 3110 [RFC3110], and the RSA key
   size limit (4096 bits) is relaxed in the IPSECKEY RR [RFC4025]
   specification.

   In addition, this document similarly defines the PK format of type
   Elliptic Curve Digital Signature Algorithm (ECDSA) as the algorithm-
   specific portion of the DNSKEY RR RDATA for ECDSA [RFC6605], i.e, all
   of the DNSKEY RR DATA after the first four octets, corresponding to
   the same portion of the DNSKEY RR that must be specified by documents
   that define a DNSSEC algorithm.

### 5.6. Rendezvous Servers Format

   The Rendezvous Server field indicates one or more variable length
   wire-encoded domain names of one or more RVSs, concatenated and
   encoded as described in Section 3.3 of RFC 1035 [RFC1035]:
   "<domain-name> is a domain name represented as a series of labels,
   and terminated by a label with zero length".  Since the wire-encoded
   format is self-describing, the length of each domain name is
   implicit: The zero length label termination serves as a separator
   between multiple RVS domain names concatenated in the Rendezvous
   Server field of a same HIP RR.  Since the length of the other portion
   of the RR's RRDATA is known, and the overall length of the RR's RDATA
   is also known (RDLENGTH), all the length information necessary to
   parse the HIP RR is available.

   The domain names MUST NOT be compressed.  The RVS or servers are
   listed in order of preference (i.e., the first RVS or servers are
   preferred), defining an implicit order amongst RVSs of a single RR. 
   When multiple HIP RRs are present at the same owner name, this
   implicit order of RVSs within an RR MUST NOT be used to infer a
   preference order between RVSs stored in different RRs.

# Referenced Sections from RFC 8949: Concise Binary Object Representation (CBOR)

The following sections were referenced. Remaining sections are not included.

### 9.1. CBOR Simple Values Registry

   IANA has created the "Concise Binary Object Representation (CBOR)
   Simple Values" registry at [IANA.cbor-simple-values ].  The initial
   values are shown in Table 4.

   New entries in the range 0 to 19 are assigned by Standards Action
   [RFC8126].  It is suggested that IANA allocate values starting with
   the number 16 in order to reserve the lower numbers for contiguous
   blocks (if any).

   New entries in the range 32 to 255 are assigned by Specification
   Required.

## A. Examples of Encoded CBOR Data Items

   The following table provides some CBOR-encoded values in hexadecimal
   (right column), together with diagnostic notation for these values
   (left column).  Note that the string "\u00fc" is one form of
   diagnostic notation for a UTF-8 string containing the single Unicode
   character U+00FC (LATIN SMALL LETTER U WITH DIAERESIS, "ü").
   Similarly, "\u6c34" is a UTF-8 string in diagnostic notation with a
   single character U+6C34 (CJK UNIFIED IDEOGRAPH-6C34, "水"), often
   representing "water", and "\ud800\udd51" is a UTF-8 string in
   diagnostic notation with a single character U+10151 (GREEK ACROPHONIC
   ATTIC FIFTY STATERS, "𐅑").  (Note that all these single-character
   strings could also be represented in native UTF-8 in diagnostic
   notation, just not if an ASCII-only specification is required.)  In
   the diagnostic notation provided for bignums, their intended numeric
   value is shown as a decimal number (such as 18446744073709551616)
   instead of a tagged byte string (such as 2(h'010000000000000000')).

   +==============================+====================================+
   |Diagnostic                    | Encoded                            |
   +==============================+====================================+
   |0                             | 0x00                               |
   +------------------------------+------------------------------------+
   |1                             | 0x01                               |
   +------------------------------+------------------------------------+
   |10                            | 0x0a                               |
   +------------------------------+------------------------------------+
   |23                            | 0x17                               |
   +------------------------------+------------------------------------+
   |24                            | 0x1818                             |
   +------------------------------+------------------------------------+
   |25                            | 0x1819                             |
   +------------------------------+------------------------------------+
   |100                           | 0x1864                             |
   +------------------------------+------------------------------------+
   |1000                          | 0x1903e8                           |
   +------------------------------+------------------------------------+
   |1000000                       | 0x1a000f4240                       |
   +------------------------------+------------------------------------+
   |1000000000000                 | 0x1b000000e8d4a51000               |
   +------------------------------+------------------------------------+
   |18446744073709551615          | 0x1bffffffffffffffff               |
   +------------------------------+------------------------------------+
   |18446744073709551616          | 0xc249010000000000000000           |
   +------------------------------+------------------------------------+
   |-18446744073709551616         | 0x3bffffffffffffffff               |
   +------------------------------+------------------------------------+
   |-18446744073709551617         | 0xc349010000000000000000           |
   +------------------------------+------------------------------------+
   |-1                            | 0x20                               |
   +------------------------------+------------------------------------+
   |-10                           | 0x29                               |
   +------------------------------+------------------------------------+
   |-100                          | 0x3863                             |
   +------------------------------+------------------------------------+
   |-1000                         | 0x3903e7                           |
   +------------------------------+------------------------------------+
   |0.0                           | 0xf90000                           |
   +------------------------------+------------------------------------+
   |-0.0                          | 0xf98000                           |
   +------------------------------+------------------------------------+
   |1.0                           | 0xf93c00                           |
   +------------------------------+------------------------------------+
   |1.1                           | 0xfb3ff199999999999a               |
   +------------------------------+------------------------------------+
   |1.5                           | 0xf93e00                           |
   +------------------------------+------------------------------------+
   |65504.0                       | 0xf97bff                           |
   +------------------------------+------------------------------------+
   |100000.0                      | 0xfa47c35000                       |
   +------------------------------+------------------------------------+
   |3.4028234663852886e+38        | 0xfa7f7fffff                       |
   +------------------------------+------------------------------------+
   |1.0e+300                      | 0xfb7e37e43c8800759c               |
   +------------------------------+------------------------------------+
   |5.960464477539063e-8          | 0xf90001                           |
   +------------------------------+------------------------------------+
   |0.00006103515625              | 0xf90400                           |
   +------------------------------+------------------------------------+
   |-4.0                          | 0xf9c400                           |
   +------------------------------+------------------------------------+
   |-4.1                          | 0xfbc010666666666666               |
   +------------------------------+------------------------------------+
   |Infinity                      | 0xf97c00                           |
   +------------------------------+------------------------------------+
   |NaN                           | 0xf97e00                           |
   +------------------------------+------------------------------------+
   |-Infinity                     | 0xf9fc00                           |
   +------------------------------+------------------------------------+
   |Infinity                      | 0xfa7f800000                       |
   +------------------------------+------------------------------------+
   |NaN                           | 0xfa7fc00000                       |
   +------------------------------+------------------------------------+
   |-Infinity                     | 0xfaff800000                       |
   +------------------------------+------------------------------------+
   |Infinity                      | 0xfb7ff0000000000000               |
   +------------------------------+------------------------------------+
   |NaN                           | 0xfb7ff8000000000000               |
   +------------------------------+------------------------------------+
   |-Infinity                     | 0xfbfff0000000000000               |
   +------------------------------+------------------------------------+
   |false                         | 0xf4                               |
   +------------------------------+------------------------------------+
   |true                          | 0xf5                               |
   +------------------------------+------------------------------------+
   |null                          | 0xf6                               |
   +------------------------------+------------------------------------+
   |undefined                     | 0xf7                               |
   +------------------------------+------------------------------------+
   |simple(16)                    | 0xf0                               |
   +------------------------------+------------------------------------+
   |simple(255)                   | 0xf8ff                             |
   +------------------------------+------------------------------------+
   |0("2013-03-21T20:04:00Z")     | 0xc074323031332d30332d32315432303a |
   |                              | 30343a30305a                       |
   +------------------------------+------------------------------------+
   |1(1363896240)                 | 0xc11a514b67b0                     |
   +------------------------------+------------------------------------+
   |1(1363896240.5)               | 0xc1fb41d452d9ec200000             |
   +------------------------------+------------------------------------+
   |23(h'01020304')               | 0xd74401020304                     |
   +------------------------------+------------------------------------+
   |24(h'6449455446')             | 0xd818456449455446                 |
   +------------------------------+------------------------------------+
   |32("http://www.example.com")  | 0xd82076687474703a2f2f7777772e6578 |
   |                              | 616d706c652e636f6d                 |
   +------------------------------+------------------------------------+
   |h''                           | 0x40                               |
   +------------------------------+------------------------------------+
   |h'01020304'                   | 0x4401020304                       |
   +------------------------------+------------------------------------+
   |""                            | 0x60                               |
   +------------------------------+------------------------------------+
   |"a"                           | 0x6161                             |
   +------------------------------+------------------------------------+
   |"IETF"                        | 0x6449455446                       |
   +------------------------------+------------------------------------+
   |"\"\\"                        | 0x62225c                           |
   +------------------------------+------------------------------------+
   |"\u00fc"                      | 0x62c3bc                           |
   +------------------------------+------------------------------------+
   |"\u6c34"                      | 0x63e6b0b4                         |
   +------------------------------+------------------------------------+
   |"\ud800\udd51"                | 0x64f0908591                       |
   +------------------------------+------------------------------------+
   |[]                            | 0x80                               |
   +------------------------------+------------------------------------+
   |[1, 2, 3]                     | 0x83010203                         |
   +------------------------------+------------------------------------+
   |[1, [2, 3], [4, 5]]           | 0x8301820203820405                 |
   +------------------------------+------------------------------------+
   |[1, 2, 3, 4, 5, 6, 7, 8, 9,   | 0x98190102030405060708090a0b0c0d0e |
   |10, 11, 12, 13, 14, 15, 16,   | 0f101112131415161718181819         |
   |17, 18, 19, 20, 21, 22, 23,   |                                    |
   |24, 25]                       |                                    |
   +------------------------------+------------------------------------+
   |{}                            | 0xa0                               |
   +------------------------------+------------------------------------+
   |{1: 2, 3: 4}                  | 0xa201020304                       |
   +------------------------------+------------------------------------+
   |{"a": 1, "b": [2, 3]}         | 0xa26161016162820203               |
   +------------------------------+------------------------------------+
   |["a", {"b": "c"}]             | 0x826161a161626163                 |
   +------------------------------+------------------------------------+
   |{"a": "A", "b": "B", "c": "C",| 0xa5616161416162614261636143616461 |
   |"d": "D", "e": "E"}           | 4461656145                         |
   +------------------------------+------------------------------------+
   |(_ h'0102', h'030405')        | 0x5f42010243030405ff               |
   +------------------------------+------------------------------------+
   |(_ "strea", "ming")           | 0x7f657374726561646d696e67ff       |
   +------------------------------+------------------------------------+
   |[_ ]                          | 0x9fff                             |
   +------------------------------+------------------------------------+
   |[_ 1, [2, 3], [_ 4, 5]]       | 0x9f018202039f0405ffff             |
   +------------------------------+------------------------------------+
   |[_ 1, [2, 3], [4, 5]]         | 0x9f01820203820405ff               |
   +------------------------------+------------------------------------+
   |[1, [2, 3], [_ 4, 5]]         | 0x83018202039f0405ff               |
   +------------------------------+------------------------------------+
   |[1, [_ 2, 3], [4, 5]]         | 0x83019f0203ff820405               |
   +------------------------------+------------------------------------+
   |[_ 1, 2, 3, 4, 5, 6, 7, 8, 9, | 0x9f0102030405060708090a0b0c0d0e0f |
   |10, 11, 12, 13, 14, 15, 16,   | 101112131415161718181819ff         |
   |17, 18, 19, 20, 21, 22, 23,   |                                    |
   |24, 25]                       |                                    |
   +------------------------------+------------------------------------+
   |{_ "a": 1, "b": [_ 2, 3]}     | 0xbf61610161629f0203ffff           |
   +------------------------------+------------------------------------+
   |["a", {_ "b": "c"}]           | 0x826161bf61626163ff               |
   +------------------------------+------------------------------------+
   |{_ "Fun": true, "Amt": -2}    | 0xbf6346756ef563416d7421ff         |
   +------------------------------+------------------------------------+

                Table 6: Examples of Encoded CBOR Data Items

# Referenced Sections from RFC 8610: Concise Data Definition Language (CDDL): A Notational Convention to Express Concise Binary Object Representation (CBOR) and JSON Data Structures

The following sections were referenced. Remaining sections are not included.

## 4. Making Use of CDDL

   In this section, we discuss several potential ways to employ CDDL.

### 4.1. As a Guide for a Human User

   CDDL can be used to efficiently define the layout of CBOR data, such
   that a human implementer can easily see how data is supposed to be
   encoded.

   Since CDDL maps parts of the CBOR data to human-readable names, tools
   could be built that use CDDL to provide a human-friendly
   representation of the CBOR data and allow them to edit such data
   while remaining compliant with its CDDL definition.

### 4.2. For Automated Checking of CBOR Data Structures

   CDDL has been specified such that a machine can handle the CDDL
   definition and related CBOR data (and, thus, also JSON data).  For
   example, a machine could use CDDL to check whether or not CBOR data
   is compliant with its definition. 
   The need for thoroughness of such compliance checking depends on the
   application.  For example, an application may decide not to check the
   data structure at all and use the CDDL definition solely as a means
   to indicate the structure of the data to the programmer.

   On the other hand, the application may also implement a checking
   mechanism that goes as far as checking that all mandatory map members
   are available.

   The matter of how far the data description must be enforced by an
   application is left to the designers and implementers of that
   application, keeping in mind related security considerations.

   In no case is it intended that a CDDL tool would be "writing code"
   for an implementation.

### 4.3. For Data Analysis Tools

   In the long run, it can be expected that more and more data will be
   stored using the CBOR data format.

   Where there is data, there is data analysis and the need to process
   such data automatically.  CDDL can be used for such automated data
   processing, allowing tools to verify data, clean it, and extract
   particular parts of interest from it.

   Since CBOR is designed with constrained devices in mind, a likely use
   of it would be small sensors.  An interesting use would thus be
   automated analysis of sensor data.

## 5. Security Considerations

   This document presents a content rules language for expressing CBOR
   data structures.  As such, it does not bring any security issues on
   itself, although specifications of protocols that use CBOR naturally
   need security analyses when defined.  General guidelines for writing
   security considerations are defined in [RFC3552] (BCP 72).
   Specifications using CDDL to define CBOR structures in protocols need
   to follow those guidelines.  Additional topics that could be
   considered in a security considerations section for a specification
   that uses CDDL to define CBOR structures include the following:

   o  Where could the language maybe cause confusion in a way that will
      enable security issues?
   o  Where a CDDL matcher is part of the implementation of a system,
      the security of the system ought not depend on the correctness of
      the CDDL specification or CDDL implementation without any further
      defenses in place.

   o  Where the CDDL specification includes extension points, the impact
      of extensions on the security of the system needs to be carefully
      considered.

   Writers of CDDL specifications are strongly encouraged to value
   clarity and transparency of the specification over its elegance.
   Keep it as simple as possible while still expressing the needed data
   model.

   A related observation about formal description techniques in general
   that is strongly recommended to be kept in mind by writers of CDDL
   specifications: just because CDDL makes it easier to handle
   complexity in a specification, that does not make that complexity
   somehow less bad (except maybe on the level of the humans having to
   grasp the complex structure while reading the spec).

## A. Parsing Expression Grammars (PEGs)

   This appendix is normative.

   Since the 1950s, many grammar notations are based on Backus-Naur Form
   (BNF), a notation for context-free grammars (CFGs) within Chomsky's
   generative system of grammars.  The Augmented Backus-Naur Form (ABNF)
   [RFC5234], widely used in IETF specifications and also inspiring the
   syntax of CDDL, is an example of this.

   Generative grammars can express ambiguity well, but this very
   property may make them hard to use in recognition systems, spawning a
   number of subdialects that pose constraints on generative grammars to
   be used with parser generators; this scenario may be hard for the
   specification writer to manage.

   PEGs [PEG ] provide an alternative formal foundation for describing
   grammars that emphasizes recognition over generation and resolves
   what would have been ambiguity in generative systems by introducing
   the concept of "prioritized choice".

   The notation for PEGs is quite close to BNF, with the usual "Extended
   BNF" features, such as repetition, added.  However, where BNF uses
   the unordered (symmetrical) choice operator "|" (incidentally notated
   as "/" in ABNF), PEG provides a prioritized choice operator "/".  The
   two alternatives listed are to be tested in left-to-right order,
   locking in the first successful match and disregarding any further
   potential matches within the choice (but not disabling alternatives
   in choices containing this choice, as a cut (Section 3.5.4) would).

   For example, the ABNF expressions

      A = "a" "b" / "a"    (1)

   and

      A = "a" / "a" "b"    (2)

   are equivalent in ABNF's original generative framework but are very
   different in PEG: in (2), the second alternative will never match, as
   any input string starting with an "a" will already succeed in the
   first alternative, locking in the match.

   Similarly, the occurrence indicators ("?", "*", "+") are "greedy" in
   PEG, i.e., they consume as much input as they match (and, as a
   consequence, "a* a" in PEG notation or "*a a" in CDDL syntax never
   can match anything, as all input matching "a" is already consumed by
   the initial "a*", leaving nothing to match the second "a"). 
   Incidentally, the grammar of CDDL itself, as written in ABNF in Appendix B , can be interpreted both (1) in the generative framework
   on which RFC 5234 is based and (2) as a PEG.  This was made possible
   by ordering the choices in the grammar such that a successful match
   made on the left-hand side of a "/" operator is always the intended
   match, instead of relying on the power of symmetrical choices (for
   example, note the sequence of alternatives in the rule for "uint",
   where the lone zero is behind the longer match alternatives that
   start with a zero).

   The syntax used for expressing the PEG component of CDDL is based on
   ABNF, interpreted in the obvious way with PEG semantics.  The ABNF
   convention of notating occurrence indicators before the controlled
   primary, and of allowing numeric values for minimum and maximum
   occurrence around a "*" sign, is copied.  While PEG is only about
   characters, CDDL has a richer set of elements, such as types and
   groups.  Specifically, the following constructs map:

       +-------+-------+-------------------------------------------+
       | CDDL  | PEG   | Remark                                    |
       +-------+-------+-------------------------------------------+
       | "="   | "<-"  | /= and //= are abbreviations              |
       | "//"  | "/"   | prioritized choice                        |
       | "/"   | "/"   | prioritized choice, limited to types only |
       | "?" P | P "?" | zero or one                               |
       | "*" P | P "*" | zero or more                              |
       | "+" P | P "+" | one or more                               |
       | A B   | A B   | sequence                                  |
       | A, B  | A B   | sequence, comma is decoration only        |
       +-------+-------+-------------------------------------------+

   The literal notation and the use of square brackets, curly braces,
   tildes, ampersands, and hash marks are specific to CDDL and unrelated
   to the conventional PEG notation.  The DOT (".") from PEG is replaced
   by the unadorned "#" or its alias "any".  Also, CDDL does not provide
   the syntactic predicate operators NOT ("!") or AND ("&") from PEG,
   reducing expressiveness as well as complexity.

   For more details about PEG's theoretical foundation and interesting
   properties of the operators such as associativity and distributivity,
   the reader is referred to [PEG ]. 





# Referenced Sections from RFC 4648: The Base16, Base32, and Base64 Data Encodings

The following sections were referenced. Remaining sections are not included.

## 11. ISO C99 Implementation of Base64

   An ISO C99 implementation of Base64 encoding and decoding that is
   believed to follow all recommendations in this RFC is available from:http://josefsson.org/base-encoding/   This code is not normative.

   The code could not be included in this RFC for procedural reasons
   (RFC 3978 section 5.4).

# Referenced Sections from RFC draft-ietf-drip-dki-09: Intended status: Informational                                   S. Card

The following sections were referenced. Remaining sections are not included.

### 1.3. The C509 encoding of X.509 Certificates

   A price in object size is paid in the ASN.1 encoding of X.509
   certificates.  This is often a barrier for use over constrained links
   and even storage demands on constrained processing platforms.  The
   [C509-Certificates ] provides an alternative encoding in two different
   manners:

      1.  An invertible CBOR re-encoding of DER encoded X.509
          certificates [RFC5280], which can be reversed to obtain the
          original DER encoded X.509 certificate.

      2.  Natively signed C509 certificates, where the signature is
          calculated over the CBOR encoding instead of over the DER
          encoding as in 1.  This removes the need for ASN.1 and DER
          parsing and the associated complexity but they are not
          backwards compatible with implementations requiring DER
          encoded X.509.

   The invertible CBOR encoding is recommended for use here.  This can
   be readily implemented through libraries that do the translation, as
   needed, between X.509 and c509.

## 3. The DET public Key Infrastructure (DKI)





### 3.1. The DKI Levels





#### 3.1.1. The Apex

   The Apex Authorization DET is used to endorse RAA Authorization DETs
   and its own Apex Issuing DETs; it has no other use.  This is the case
   for all Authorization DETs.  Apex Issuing DETs are used to endorse
   DETs, with HID= 0|0, used by Apex services.

   The DET Apex may be only theoretical if no Entity steps forward to
   provide this role.

#### 3.1.2. The RAAs

   Each RAA use its Authorization DET (HID = RAA#|0) to endorse its RAA
   Issuing DET(s) (also HID = RAA#|0) and for signing its HDA
   Authorization DETs (HID = RAA#|HDA#).

   An RAA may have multiple Issuing DETs (HID = RAA#|0), each for a
   different use (e.g. CRL signing, RAA server signing).  It is expected
   that, over time, an RAA will rollover its Issuing DETs, thus at times
   there will be more than ONE Issuing DET per role in use.

   These Issuing DETs, like those at the Apex level, constitute an
   implicit HDA.  There is no Authorization DET for this implicit HDA,
   but other than only signing for entities like servers needed by the
   RAA, it should be considered as an HDA in terms of policies.

   An RAA may directly issue DETs for Operators/Pilots, but it is
   recommended that if the RAA has this responsiblity, it runs an HDA
   specifically for this function.  This allows separation of the RAA
   role from the HDA.  It is recommended that the RAA only offer
   Endorsing/Signing services for the functions of running the RAA.

   The initial RAA range assignments are defined in Section 6.2.1 of
   [drip-registries ], Table 1.  It is anticipated that DRIP usage will
   expand to use into General/Civil Aviation.  Thus at some point a
   block of RAAs will be set aside much like for the CTA-2063A
   [CTA2063A ] range. 





#### 3.1.3. The HDAs

   Each HDA use its Authorization DET to endorse its HDA Issuing DETs
   (e.g. RAA=267, HDA=567).

   An HDA Issuing DET is used to endorse Operational DETs; those used by
   the HDA for its services (e.g. USS) and for Devices (e.g.  UA, GCS,
   ground infrastructure) partaking in the HDA's services.

   If the Operational DET is a Manufacturer DET, the "valid not after"
   date (vna) MUST be 99991231235959Z.

   An HDA may directly issue DETs for Operators/Pilots.  It is
   recommended that different Issuing HDAs be used for devices and
   people.  They may be under the same Authentication HDA.  Of
   importance is that there are different LOAs for CAs for people and
   devices per Section 6.1.

### 3.2. The Offline Requirement for Authentication DETs

   The Authentication DETs private keys MUST NEVER be on a system with
   any network connectivity.  Also efforts MUST be taken to limit any
   external digital media connections to these offline systems.
   Compromise of an Authentication DET compromises its and all lower
   hierarchy levels.  Such a compromise could result in a major re-
   signing effort with a new Authentication DET.  Also, during the time
   of compromise, fraudulent additions to the DKI could have occurred.

   This means that the process whereby the Authentication DET is used to
   sign the Endorsement/X.509 certificate of its level's Issuing DET(s)
   and lower level Authentication DETs MUST be conducted in an offline
   manner (e.g. Section 6).

   This offline process need not be onerous.  For example, QR codes
   could be used to pass CSR objects to the offline Authentication DET
   system, and this system could produce QR codes containing the
   Endorsements and X.509 certificates it signed (Section 6.4).

   A video conference between the parties could have one side show its
   QR code and the other copy and print it to move between the video
   conferencing system and the offline system.  This is a simplification
   of a larger signing operation, but shows how such a signing need not
   require travel and expensive hand-off methodologies.

   It should be noted that the endorsement of Issuing DETs follow the
   same restriction, as it is done with the Authentication DET.  It MUST
   be conducted in an offline manner. 





### 3.3. DNS view of DKI

   The primary view of the DKI is within DNS.  This is in the
   3.0.0.1.0.0.2.ip6.arpa zone (Apex level of the DRIP IPv6 DET format).

   In the DET DNS structure, only the Apex and RAA levels MUST be DNSSEC
   signed.  The HDA level may be too dynamic for DNSSEC signing (e.g.
   hundreds of new EE Operational DETs per hour); trust in the EE
   Operational DETs within the HDA level comes through inclusion of the
   HDA Endorsement of EE object.  A slow-churn HDA MAY use DNSSEC.  The
   RAA and HDA levels MUST contain their Endorsement by higher object;
   this provides the needed trust in the Endorsement of EE objects.  The
   Apex level Endorsement is self-signed, thus trust in it is only
   possible via DNSSEC.

   Endorsements are stored in the DNS BRID RR (Section 5.2 of
   [drip-registries ]).  X.509 certificates in the DNS HHIT RR
   (Section 5.1 of [drip-registries ]).

   Authors note: These RR will only be available once [drip-registries ]
   is published.  Until then, [RFC3597] will be used to encode these RR
   Types.

   Other RR within these levels will vary.  There also may be HIP, TLSA,
   and/or URI RR.

   Each level continues on down the 3.0.0.1.0.0.2.ip6.arpa zone for its
   Authorization DET and Issuing DET(s).  RR with FQDNs for services
   offered may also be present in various forms (e.g. a URI for the
   commercial FQDN for the DKI Entity).  TLSA RR of DET SPKI may be
   directly included here.  Same with HIP RR.  The Authorization
   Endorsement SHOULD be present, as SHOULD be Issuing Endorsements.

### 3.4. Managing DET Revocation

   For Operational DETs, there is no direct concept of DET revocation.
   Operational DETs are either discoverable via DNS or not valid despite
   being in a non-expired Endorsement signed an Issuing DET.  Thus if an
   Issuing Entity needs to "revoke" an Operational DET it removes all
   entries for it from DNS, so a short TTL on those records is
   recommended. 
   Authorization and Issuing DETs are not so easily "revoked"; something
   akin to an X.509 CRL mechanism is needed.  This could best be dealt
   with by Endorsements managed in the new DET RR that includes
   revocation status.  Thus Section 5.2 of [drip-registries ] defines the
   specific RR for Endorsements that will be used here.  Minimally, at
   least the revocation status and revocation date(s) need to be in this
   RR.  Until this RR is available, there is no mechanism, other than
   removal for Authorization and Issuing DET revocations.

### 3.5. The Offline cache of HDA Issuing Endorsements

   The Offline cache of HDA Issuing Endorsements, used to verify various
   EE signed objects without needing DNS access, SHOULD consist of the
   HDA Authentication DET Endorsements of the HDA Issuing DETs.  Thus
   the receiver has a trusted source of the HDA Issuing DET Public Key
   (HI) in a DRIP standard object (136 bytes).  If the DKI DNS tree
   includes GEO location data and coverage, a receiver could query some
   service for a trusted cache within some radius of its location.  Such
   as, please tell me of all HDAs within 100KM of...

   This cache MAY contain the full chain up to the Apex.  This could be
   helpful in limited connectivity environments when encountering an HDA
   Issuing DET under a unknown Authenticated HDA or RAA.  The needed
   trust chain could be shorter.

#### 3.5.1. HDA Offline Trust cache

   There are situations where a list of specific HDAs for an entity to
   trust for some application is needed.  This can best be met by
   maintaining a cache as above but only of the trusted HDA Issuing
   Endorsements.  How a list of this limited trust is maintain and
   distributed is out of scope of this document and is left to those
   needing this specific feature.

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

# Referenced Sections from RFC 9575: DRIP Entity Tag (DET) Authentication Formats and Protocols for Broadcast Remote Identification (RID)

The following sections were referenced. Remaining sections are not included.

### B.2. Full Authentication Example

   This example (Figure 13) is focused on showing that 100% of ASTM
   Messages can be authenticated over Legacy Transports with up to 125%
   overhead in Authentication Pages.  Extended Transports are not shown
   in this example, because, for those, Authentication with DRIP is
   achieved using Extended Wrapper (Section 4.3.2).  Two ASTM Message
   Packs are sent in a given cycle: one containing up to four ASTM
   Messages and an Extended Wrapper (authenticating the pack), and one
   containing a Link message with a Broadcast Endorsement and up to two
   other ASTM Messages.

   This example transmit scheme covers and meets every known regulatory
   case enabling manufacturers to use the same firmware worldwide.

         +------------------------------------------------------+
         |                      Frame Slots                     |
         | 00 - 04           | 05 - 07       | 08 - 16 | 17     |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[0] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[1] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[2] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[3] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[4] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[5] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[6] |
         +-------------------+---------------+---------+--------+
         | {A|B|C|D},V,S,I,O | {A|B|C|D},V,S | M[0,8]  | L/W[7] |
         +-------------------+---------------+---------+--------+

         A = Basic ID Message (0x0) ID Type 1
         B = Basic ID Message (0x0) ID Type 2
         C = Basic ID Message (0x0) ID Type 3
         D = Basic ID Message (0x0) ID Type 4
         V = Location/Vector Message (0x1)
         I = Self ID Message (0x3)
         S = System Message (0x4)
         O = Operator ID Message (0x5)

         L[y,z] = DRIP Link Authentication Message (0x2)
         W[y,z] = DRIP Wrapper Authentication Message (0x2)
         M[y,z] = DRIP Manifest Authentication Message (0x2)
           y = Start Page
           z = End Page

         # = Empty Frame Slot
         * = Message in DRIP Manifest Authentication Message

        Figure 13: Example of a Fully Authenticated Legacy Transport
                             Transmit Schedule

   Every common required message (Basic ID, Location/Vector, and System)
   is sent twice along with Operator ID and Self ID in a single second.
   The Manifest is over all messages (8) in slots 00 - 04 and 05 - 07.

   In two seconds, either a Link or Wrapper is sent.  The content and
   order of Links and Wrappers runs as follows:

   Link: HDA on UA
   Link: RAA on HDA
   Link: HDA on UA
   Link: Apex on RAA
   Link: HDA on UA
   Link: RAA on HDA
   Link: HDA on UA
   Wrapper: Location/Vector (0x1), System (0x4)
   Link: HDA on UA
   Link: RAA on HDA
   Link: HDA on UA
   Link: Apex on RAA
   Link: HDA on UA
   Link: RAA on HDA
   Link: HDA on UA
   Wrapper: Location/Vector (0x1), System (0x4)
   Link: IANA on UAS RID Apex

   After perfect receipt of all messages for a period of 8 seconds, all
   messages sent during that period have been authenticated using the
   Manifest (except for the Authentication Messages themselves).  Within
   136 seconds, the entire Broadcast Endorsement chain is received and
   can be validated.  Interspersed in this schedule are 4 messages sent
   not only in their basic [F3411] form, but also in DRIP Wrapper
   messages, together with their attached signatures, to defend against
   the possibility of attack against the detached signatures provided by
   the Manifest messages.

#### B.2.1. Raw Example

   Assuming the following DET and HI:

   2001:3f:fe00:105:a29b:3ff4:2226:c04e
   b5fef530d450dedb59ebafa18b00d7f5ed0ac08a81975034297bea2b00041813

   The following ASTM Messages are to be sent in a single second:

   0240012001003ffe000105a29b3ff42226c04e000000000000
   12000000000000000000000000000000000000000060220000
   32004578616d706c652053656c662049440000000000000000
   420000000000000000000100000000000000000010ea510900
   52004578616d706c65204f70657261746f7220494400000000
   0240012001003ffe000105a29b3ff42226c04e000000000000
   12000000000000000000000000000000000000000060220000
   420000000000000000000100000000000000000010ea510900

   This is a Link with FEC that would be spread out over 8 seconds:

   2250078910ea510904314b8564b17e66662001003ffe000105
   2251a29b3ff42226c04eb5fef530d450dedb59ebafa18b00d7
   2252f5ed0ac08a81975034297bea2b000418132001003ffe00
   22530105b82bf1c99d87273103fc83f6ecd9b91842f205c222
   2254dd71d8e165ad18ca91daf9299a73eec850c756a7e9be46
   2255f51dddfa0f09db7bfdde14eec07c7a6dd1061c1d5ace94
   2256d9ad97940d280000000000000000000000000000000000
   2257a03b0f7a6feb0d198167045058cfc49f73129917024d22

   This is a Wrapper with FEC that would be spread out over 8 seconds:

   2250078b10ea510902e0dd7c6560115e671200000000000000
   22510000000000000000000000000060220000420000000000
   2252000000000100000000000000000010ea5109002001003f
   2253fe000105a29b3ff42226c04ef0ecad581a030ca790152a
   22542f08df5762a463e24a742d1c530ec977bbe0d113697e2b
   2255b909d6c7557bdaf1227ce86154b030daadda4a6b8474de
   22569a62f6c375020826000000000000000000000000000000
   2257f5e8eebcb04f8c2197526053e66c010d5d7297ff7c1fe0

   This is the Manifest with FEC sent in the same second as the original
   messages:

   225008b110ea510903e0dd7c6560115e670000000000000000
   2251d57594875f8608b4d61dc9224ecf8b842bd4862734ed01
   22522ca2e5f2b8a3e61547b81704766ba3eeb651be7eafc928
   22538884e3e28a24fd5529bc2bd4862734ed012ca2e5f2b8a3
   2254e61547b81704766ba3eeb62001003ffe000105a29b3ff4
   22552226c04efb729846e7d110903797066fd96f49a77c5a48
   2256c4c3b330be05bc4a958e9641718aaa31aeabad368386a2
   22579ed2dce2769120da83edbcdc0858dd1e357755e7860317
   2258e7c06a5918ea62a937391cbfe0983539de1b2e688b7c83


# Referenced Sections from RFC 9434: Drone Remote Identification Protocol (DRIP) Architecture

The following sections were referenced. Remaining sections are not included.

### 3.4. HHIT for DRIP Identifier Registration and Lookup

   UAS RID needs a deterministic lookup mechanism that rapidly provides
   actionable information about the identified UA.  Given the size
   constraints imposed by the Bluetooth 4 broadcast media, the UAS ID
   itself needs to be a non-spoofable inquiry input into the lookup.

   A DRIP registration process based on the explicit hierarchy within a
   HHIT provides manageable uniqueness of the HI for the HHIT.  The
   hierarchy is defined in [RFC9374] and consists of 2 levels: an RAA
   and then an HDA.  The registration within this hierarchy is the
   defense against a cryptographic hash second-preimage attack on the
   HHIT (e.g., multiple HIs yielding the same HHIT; see Requirement ID-3
   in [RFC9153]).  The First Come First Served registration policy is
   adequate.

   A lookup of the HHIT into the DIME provides the registered HI for
   HHIT proof of ownership and deterministic access to any other needed
   actionable information based on inquiry access authority (more
   details in Section 4.2).

## 4. DRIP Identifier Registration and Registries

   DRIP registries hold both public and private UAS information (see
   PRIV-1 in [RFC9153]) resulting from the DRIP identifier registration
   process.  Given these different uses, and to improve scalability,
   security, and simplicity of administration, the public and private
   information can be stored in different registries.  This section
   introduces the public and private information registries for DRIP
   identifiers.  In this section, for ease of comprehension, the
   registry functions are described (using familiar terminology) without
   detailing their assignment to specific implementing entities (or
   using unfamiliar jargon).  Elsewhere in this document, and in
   forthcoming documents detailing the DRIP registration processes and
   entities, the more specific term "DRIP Identity Management Entity"
   (DIME) will be used.  This DRIP identifier registration process
   satisfies the following DRIP requirements defined in [RFC9153]: GEN-
   3, GEN-4, ID-2, ID-4, ID-6, PRIV-3, PRIV-4, REG-1, REG-2, REG-3, and
   REG-4.

### 4.1. Public Information Registry





#### 4.1.1. Background

   The public information registry provides trustable information, such
   as endorsements of UAS RID ownership and registration with the HDA.
   Optionally, pointers to the registries for the HDA and RAA implicit
   in the UAS RID can be included (e.g., for HDA and RAA HHIT|HI used in
   endorsement signing operations).  This public information will be
   principally used by Observers of Broadcast RID messages.  Data on UAS
   that only use Network RID is available via an Observer's Net-RID DP
   that would directly provide all public registry information.  The
   Net-RID DP is the only source of information for a query on an
   airspace volume.

      |  Note: In the above paragraph, | signifies concatenation of
      |  information, e.g., X | Y is the concatenation of X and Y.

#### 4.1.2. Public DRIP Identifier Registry

   A DRIP identifier MUST be registered as an Internet domain name (at
   an arbitrary level in the hierarchy, e.g., in .ip6.arpa).  Thus, the
   DNS can provide all the needed public DRIP information.  A
   standardized HHIT Fully Qualified Domain Name (FQDN) can deliver the
   HI via a HIP Resource Record (RR) [RFC8005] and other public
   information (e.g., RAA and HDA PTRs and HIP Rendezvous Servers (RVSs)
   [RFC8004]).  These public information registries can use DNSSEC to
   deliver public information that is not inherently trustable (e.g.,
   everything other than endorsements).

   This DNS entry for the HHIT can also provide a revocation service.
   For example, instead of returning the HI RR, it may return some
   record showing that the HI (and thus HHIT) has been revoked.

### 4.2. Private Information Registry





#### 4.2.1. Background

   The private information required for DRIP identifiers is similar to
   that required for Internet domain name registration.  A DRIP
   identifier solution can leverage existing Internet resources, i.e.,
   registration protocols, infrastructure, and business models, by
   fitting into a UAS ID structure compatible with DNS names.  The HHIT
   hierarchy can provide the needed scalability and management
   structure.  It is expected that the private information registry
   function will be provided by the same organizations that run a USS
   and likely integrated with a USS.  The lookup function may be
   implemented by the Net-RID DPs.

#### 4.2.2. Information Elements

   When a DET is used as a UA's Session ID, the corresponding
   manufacturer-assigned serial number MUST be stored in a private
   information registry that can be identified uniquely from the DET.
   When a DET is used as either a UA's Session ID or a UA's
   manufacturer-assigned serial number, and the operation is being flown
   under UTM, the corresponding UTM-system-assigned Operational Intent
   Identifier SHOULD be so stored.  Other information MAY be stored as
   such, and often must, to satisfy CAA regulations or USS operator
   policies.

#### 4.2.3. Private DRIP Identifier Registry Methods

   A DRIP private information registry supports essential registry
   operations (e.g., add, delete, update, and query) using interoperable
   open standard protocols.  It can accomplish this by leveraging
   aspects of the Extensible Provisioning Protocol (EPP) [RFC5730] and
   the Registry Data Access Protocol (RDAP) [RFC7480] [RFC9082]
   [RFC9083].  The DRIP private information registry in which a given
   UAS is registered needs to be findable, starting from the UAS ID,
   using the methods specified in [RFC9224].

#### 4.2.4. Alternative Private DRIP Registry Methods

   A DRIP private information registry might be an access-controlled DNS
   (e.g., via DNS over TLS).  Additionally, WebFinger [RFC7033] can be
   supported.  These alternative methods may be used by a Net-RID DP
   with specific customers.

# Referenced Sections from RFC 9153: Drone Remote Identification Protocol (DRIP) Requirements and Terminology

The following sections were referenced. Remaining sections are not included.

#### 4.2.1. Normative Requirements

   ID-1     Length: The DRIP Entity Identifier MUST NOT be longer than
            19 bytes, to fit in the Specific Session ID subfield of the
            UAS ID field of the Basic ID Message of the proposed
            revision of [F3411-19] (at the time of writing).

   ID-2     Registry ID: The DRIP identifier MUST be sufficient to
            identify a registry in which the entity identified therewith
            is listed.

   ID-3     Entity ID: The DRIP identifier MUST be sufficient to enable
            lookups of other data associated with the entity identified
            therewith in that registry.

   ID-4     Uniqueness: The DRIP identifier MUST be unique within the
            applicable global identifier space from when it is first
            registered therein until it is explicitly deregistered
            therefrom (due to, e.g., expiration after a specified
            lifetime, revocation by the registry, or surrender by the
            operator).

   ID-5     Non-spoofability: The DRIP identifier MUST NOT be spoofable
            within the context of a minimal Remote ID broadcast message
            set (to be specified within DRIP to be sufficient
            collectively to prove sender ownership of the claimed
            identifier).

   ID-6     Unlinkability: The DRIP identifier MUST NOT facilitate
            adversarial correlation over multiple operations.  If this
            is accomplished by limiting each identifier to a single use
            or brief period of usage, the DRIP identifier MUST support
            well-defined, scalable, timely registration methods.

#### 4.2.2. Rationale

   The DRIP identifier can refer to various entities.  In the primary
   initial use case, the entity to be identified is the UA.  Entities to
   be identified in other likely use cases include, but are not limited
   to, the operator, USS, and Observer.  In all cases, the entity
   identified must own the identifier (i.e., have the exclusive
   capability to use the identifier, such that receivers can verify the
   entity's ownership of it).

   The DRIP identifier can be used at various layers.  In Broadcast RID,
   it would be used by the application running directly over the data
   link.  In Network RID, it would be used by the application running
   over HTTPS (not required by DRIP but generally used by Network RID
   implementations) and possibly other protocols.  In RID-initiated V2X
   applications such as DAA and C2, it could be used between the network
   and transport layers (e.g., with the Host Identity Protocol (HIP)
   [RFC9063] [RFC7401]) or between the transport and application layers
   (e.g., with DTLS [RFC6347]).

   Registry ID (which registry the entity is in) and Entity ID (which
   entity it is, within that registry) are requirements on a single DRIP
   Entity Identifier, not separate (types of) ID.  In the most common
   use case, the entity will be the UA, and the DRIP identifier will be
   the UAS ID; however, other entities may also benefit from having DRIP
   identifiers, so the entity type is not prescribed here.

   Whether a UAS ID is generated by the operator, GCS, UA, USS,
   registry, or some collaboration among them is unspecified; however,
   there must be agreement on the UAS ID among these entities.
   Management of DRIP identifiers is the primary function of their
   registration hierarchies, from the root (presumably IANA), through
   sector-specific and regional authorities (presumably ICAO and CAAs),
   to the identified entities themselves.

   While Uniqueness might be considered an implicit requirement for any
   identifier, here the point of the explicit requirement is not just
   that it should be unique, but also where and when it should be
   unique: global scope within a specified space, from registration to
   deregistration.

   While Non-spoofability imposes requirements for and on a DRIP
   authentication protocol, it also imposes requirements on the
   properties of the identifier itself.  An example of how the nature of
   the identifier can support non-spoofability is embedding a hash of
   both the Registry ID and a public key of the entity in the entity
   identifier, thus making it self-authenticating any time the entity's
   corresponding private key is used to sign a message.

   While Unlinkability is a privacy desideratum (see Section 4.3), it
   imposes requirements on the DRIP identifier itself, as distinct from
   other currently permitted choices for the UAS ID (including primarily
   the static serial number of the UA or RID module).

#### 4.4.2. Rationale

   Registries are fundamental to RID.  Only very limited information can
   be transmitted via Broadcast RID, but extended information is
   sometimes needed.  The most essential element of information sent is
   the UAS ID itself, the unique key for lookup of extended information
   in registries.  The regulatory requirements for the registry
   information models for UAS and their operators for RID and, more
   broadly, for U-space/UTM needs are in flux.  Thus, beyond designating
   the UAS ID as that unique key, the registry information model is not
   specified in this document.  While it is expected that registry
   functions will be integrated with USS, who will provide them is
   expected to vary between jurisdictions and has not yet been
   determined in most jurisdictions.  However this evolves, the
   essential registry functions, starting with management of
   identifiers, are expected to remain the same, so those are specified
   herein.

   While most data to be sent via Broadcast or Network RID is public,
   much of the extended information in registries will be private.
   Thus, AAA for registries is essential, not just to ensure that access
   is granted only to strongly authenticated, duly authorized parties,
   but also to support subsequent attribution of any leaks, audit of who
   accessed information when and for what purpose, etc.  Specific AAA
   requirements will vary by jurisdictional regulation, provider
   philosophy, customer demand, etc., so they are left to specification
   in policies.  Such policies should be human readable to facilitate
   analysis and discussion, be machine readable to enable automated
   enforcement, and use a language amenable to both, e.g., eXtensible
   Access Control Markup Language (XACML).

   The intent of the negative and positive access control requirements
   on registries is to ensure that no member of the public would be
   hindered from accessing public information, while only duly
   authorized parties would be enabled to access private information.
   Mitigation of denial-of-service attacks and refusal to allow database
   mass scraping would be based on those behaviors, not on identity or
   role of the party submitting the query per se; however, information
   on the identity of the party submitting the query might be gathered
   on such misbehavior by security systems protecting DRIP
   implementations.

   "Internet direct contact information" means a locator (e.g., IP
   address), or identifier (e.g., FQDN) that can be resolved to a
   locator, which enables initiation of an end-to-end communication
   session using a well-known protocol (e.g., SIP).

# Referenced Sections from RFC 9374: DRIP Entity Tag (DET) for Unmanned Aircraft System Remote ID (UAS RID)

The following sections were referenced. Remaining sections are not included.

## 5. DRIP Entity Tags (DETs) in DNS

   There are two approaches for storing and retrieving DETs using DNS.
   The following are examples of how this may be done.  This serves as
   guidance to the actual deployment of DETs in DNS.  However, this
   document does not provide a recommendation about which approach to
   use.  Further DNS-related considerations are covered in [DRIP-REG ].

   *  As FQDNs, for example, "20010030.hhit.arpa.".

   *  Reverse DNS lookups as IPv6 addresses per [RFC8005].

   A DET can be used to construct an FQDN that points to the USS that
   has the public/private information for the UA (REG-1 and REG-2 in Section 4.4.1 of [RFC9153]).  For example, the USS for the HHIT could
   be found via the following: assume the RAA is decimal 100 and the HDA
   is decimal 50.  The PTR record is constructed as follows:

       100.50.20010030.hhit.arpa.   IN PTR      foo.uss.example.org.

   The HDA SHOULD provide DNS service for its zone and provide the HHIT
   detail response.

   The DET reverse lookup can be a standard IPv6 reverse look up, or it
   can leverage off the HHIT structure.  Using the allocated prefix for
   HHITs 2001:30::/28 (see Section 3.1), the RAA is decimal 10 and the
   HDA is decimal 20, the DET is:

       2001:30:280:1405:a3ad:1952:ad0:a69e

   See Appendix B.1 for how the upper 64 bits, above, are constructed.
   A DET reverse lookup could be:

       a69e.0ad0.1952.a3ad.1405.0280.20.10.20010030.hhit.arpa.

   or:

       a3ad19520ad0a69e.5.20.10.20010030.hhit.arpa.

   A 'standard' ip6.arpa RR has the advantage of only one Registry
   service supported.

       $ORIGIN  5.0.4.1.0.8.2.0.0.3.0.0.1.0.0.2.ip6.arpa.
       e.9.6.a.0.d.a.0.2.5.9.1.d.a.3.a    IN   PTR
       a3ad1952ad0a69e.20.10.20010030.hhit.arpa.

   This DNS entry for the DET can also provide a revocation service.
   For example, instead of returning the HI RR it may return some record
   showing that the HI (and thus DET) has been revoked.  Guidance on
   revocation service will be provided in [DRIP-REG ].

# Summary of reference from ASTM International, "Standard Specification for Remote ID and Tracking", ASTM F3411-22A, DOI 10.1520/F3411-22A, July 2022, <https://www.astm.org/f3411-22a.html >. (F3411)

ASTM F3411-22a, titled "Standard Specification for Remote ID and Tracking," establishes performance requirements for the remote identification (Remote ID) of unmanned aircraft systems (UAS). This standard aims to enhance safety, security, and compliance by enabling governmental and civil identification of UAS, thereby increasing pilot accountability while preserving operational privacy. Remote ID is crucial for advanced operations such as beyond visual line of sight (BVLOS) flights and operations over people. ([store.astm.org](https://store.astm.org/f3411-22a.html?utm_source=openai))

The standard defines two primary methods for Remote ID:

1. **Broadcast Remote ID**: Involves the UAS transmitting radio signals directly to receivers in its vicinity.

2. **Network Remote ID**: Involves communication via the internet through a network Remote ID service provider (Net-RID SP) that interfaces with the UAS or other sources.

These methods are designed to function in various environments, including rural, urban, networked, network-degraded, and network-denied settings, regardless of airspace class. ([store.astm.org](https://store.astm.org/f3411-22a.html?utm_source=openai))

For aviation applications, understanding the semantics of specific fields within the Remote ID messages is essential:

- **Unique Identifier**: A distinct code assigned to each UAS, facilitating its identification.

- **Location Data**: Includes latitude, longitude, and altitude, providing real-time positioning of the UAS.

- **Time Stamp**: Indicates the exact time of the transmitted data, ensuring temporal accuracy.

- **Emergency Status**: Signals any abnormal conditions or emergencies, aiding in prompt response.

- **Operator Information**: Details about the UAS operator, which may be included depending on privacy considerations and regulatory requirements.

These fields are integral for air traffic management, enabling authorities to monitor UAS operations effectively and ensure compliance with aviation regulations. By standardizing these data elements, ASTM F3411-22a facilitates interoperability and enhances the safety and security of UAS integration into the national airspace. ([store.astm.org](https://store.astm.org/f3411-22a.html?utm_source=openai)) 

