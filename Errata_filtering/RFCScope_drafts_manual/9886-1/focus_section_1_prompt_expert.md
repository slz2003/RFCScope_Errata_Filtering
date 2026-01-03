Let us analyze section 1 of RFC 9886. All references made by section 1 have also been included below.

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
# Referenced Sections from RFC 9434: Drone Remote Identification Protocol (DRIP) Architecture

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   This document describes an architecture for protocols and services to
   support Unmanned Aircraft System Remote Identification and tracking
   (UAS RID), plus UAS-RID-related communications.  The architecture
   takes into account both current (including proposed) regulations and
   non-IETF technical standards.

   The architecture adheres to the requirements listed in the DRIP
   Requirements document [RFC9153] and illustrates how all of them can
   be met, except for GEN-7 QoS, which is left for future work.  The
   requirements document provides an extended introduction to the
   problem space and use cases.  Further, this architecture document
   frames the DRIP Entity Tag (DET) [RFC9374] within the architecture.

### 1.1. Overview of UAS RID and Its Standardization

   UAS RID is an application that enables UAS to be identified by UAS
   Traffic Management (UTM), UAS Service Suppliers (USS) (Appendix A),
   and third-party entities, such as law enforcement.  Many
   considerations (e.g., safety and security) dictate that UAS be
   remotely identifiable.

   Civil Aviation Authorities (CAAs) worldwide are mandating UAS RID.
   CAAs currently promulgate performance-based regulations that do not
   specify techniques but rather cite industry consensus technical
   standards as acceptable means of compliance.

   USA Federal Aviation Administration (FAA)
      The FAA published a Notice of Proposed Rule Making [NPRM ] in 2019
      and thereafter published a "Final Rule" in 2021 [FAA_RID ],
      imposing requirements on UAS manufacturers and operators, both
      commercial and recreational.  The rule states that Automatic
      Dependent Surveillance Broadcast (ADS-B) Out and transponders
      cannot be used to satisfy the UAS RID requirements on UAS to which
      the rule applies (see Appendix B ).

   European Union Aviation Safety Agency (EASA)
      In pursuit of the "U-space" concept of a single European airspace
      safely shared by manned and unmanned aircraft, the EASA published
      a [Delegated ] regulation in 2019, imposing requirements on UAS
      manufacturers and third-country operators, including but not
      limited to UAS RID requirements.  The same year, the EASA also
      published a regulation [Implementing ], laying down detailed rules
      and procedures for UAS operations and operating personnel, which
      then was updated in 2021 [Implementing_update ].  A Notice of
      Proposed Amendment [NPA ] was published in 2021 to provide more
      information about the development of acceptable means of
      compliance and guidance material to support U-space regulations.

   American Society for Testing and Materials (ASTM)
      ASTM International, Technical Committee F38 (UAS), Subcommittee
      F38.02 (Aircraft Operations), Work Item WK65041 developed an ASTM
      standard [F3411-22a ], titled "Standard Specification for Remote ID
      and Tracking".

      ASTM defines one set of UAS RID information and two means, Media
      Access Control (MAC) layer broadcast and IP layer network, of
      communicating it.  If a UAS uses both communication methods, the
      same information must be provided via both means.  [F3411-22a ] is
      the technical standard basis of the Means Of Compliance (MOC)
      specified in [F3586-22].  The FAA has accepted [F3586-22] as a MOC
      to the FAA's UAS RID Final Rule [FAA_RID ], with some caveats, as
      per [MOC-NOA ].  Other CAAs are expected to accept the same or
      other MOCs likewise based on [F3411-22a ].

   3rd Generation Partnership Project (3GPP)
      With Release 16, the 3GPP completed the UAS RID requirement study
      [TR-22.825] and proposed a set of use cases in the mobile network
      and services that can be offered based on UAS RID.  The Release 17
      study [TR-23.755] and specification [TS-23.255] focus on enhanced
      UAS service requirements and provide the protocol and application
      architecture support that will be applicable for both 4G and 5G
      networks.  The study of Further Architecture Enhancement for
      Uncrewed Aerial Vehicles (UAV) and Urban Air Mobility (UAM) in
      Release 18 [FS_AEUA ] further enhances the communication mechanism
      between UAS and USS/UTM.  The DET in Section 3 may be used as the
      3GPP CAA-level UAS ID for RID purposes.

### 1.2. Overview of Types of UAS Remote ID

   This specification introduces two types of UAS Remote IDs as defined
   in ASTM [F3411-22a ].

#### 1.2.1. Broadcast RID

   [F3411-22a ] defines a set of UAS RID messages for direct, one-way
   broadcast transmissions from the Unmanned Aircraft (UA) over
   Bluetooth or Wi-Fi.  These are currently defined as MAC layer
   messages.  Internet (or other Wide Area Network) connectivity is only
   needed for UAS registry information lookup by Observers using the
   directly received UAS ID.  Broadcast RID should be functionally
   usable in situations with no Internet connectivity.

   The minimum Broadcast RID data flow is illustrated in Figure 1.

                   +------------------------+
                   | Unmanned Aircraft (UA) |
                   +-----------o------------+
                               |
                               | app messages directly over
                               | one-way RF data link (no IP)
                               |
                               v
             +------------------o-------------------+
             | Observer's device (e.g., smartphone) |
             +--------------------------------------+

                 Figure 1: Minimum Broadcast RID Data Flow

   Broadcast RID provides information only about UA within direct Radio
   Frequency (RF) Line Of Sight (LOS), typically similar to Visual LOS
   (VLOS), with a range up to approximately 1 km.  This information may
   be 'harvested' from received broadcasts and made available via the
   Internet, enabling surveillance of areas too large for local direct
   visual observation or direct RF link-based identification (see Section 6).

#### 1.2.2. Network RID

   [F3411-22a ], using the same data dictionary that is the basis of
   Broadcast RID messages, defines a Network Remote Identification (Net-
   RID) data flow as follows.

   *  The information to be reported via UAS RID is generated by the
      UAS.  Typically, some of this data is generated by the UA and some
      by the Ground Control Station (GCS), e.g., their respective
      locations derived from the Global Navigation Satellite System
      (GNSS).

   *  The information is sent by the UAS (UA or GCS) via unspecified
      means to the cognizant Network Remote Identification Service
      Provider (Net-RID SP), typically the USS under which the UAS is
      operating if it is participating in UTM.

   *  The Net-RID SP publishes, via the Discovery and Synchronization
      Service (DSS) over the Internet, that it has operations in various
      4-D airspace volumes (Section 2.2 of [RFC9153]), describing the
      volumes but not the operations.

   *  An Observer's device, which is expected but not specified to be
      based on the Web, queries a Network Remote Identification Display
      Provider (Net-RID DP), typically also a USS, about any operations
      in a specific 4-D airspace volume.

   *  Using fully specified Web-based methods over the Internet, the
      Net-RID DP queries all Net-RID SPs that have operations in volumes
      intersecting that of the Observer's query for details on all such
      operations.

   *  The Net-RID DP aggregates information received from all such Net-
      RID SPs and responds to the Observer's query.

   The minimum Net-RID data flow is illustrated in Figure 2:

    +-------------+     ******************
    |     UA      |     *    Internet    *
    +--o-------o--+     *                *
       |       |        *                *     +------------+
       |       '--------*--(+)-----------*-----o            |
       |                *   |            *     |            |
       |       .--------*--(+)-----------*-----o Net-RID SP |
       |       |        *                *     |            |
       |       |        *         .------*-----o            |
       |       |        *         |      *     +------------+
       |       |        *         |      *
       |       |        *         |      *     +------------+
       |       |        *         '------*-----o            |
       |       |        *                *     | Net-RID DP |
       |       |        *         .------*-----o            |
       |       |        *         |      *     +------------+
       |       |        *         |      *
       |       |        *         |      *     +------------+
    +--o-------o--+     *         '------*-----o Observer's |
    |     GCS     |     *                *     | Device     |
    +-------------+     ******************     +------------+

                    Figure 2: Minimum Net-RID Data Flow

   Command and Control (C2) must flow from the GCS to the UA via some
   path.  Currently (in the year 2023), this is typically a direct RF
   link; however, with increasing Beyond Visual Line Of Sight (BVLOS)
   operations, it is expected to often be a wireless link at either end
   with the Internet between.

   Telemetry (at least the UA's position and heading) flows from the UA
   to the GCS via some path, typically the reverse of the C2 path.
   Thus, UAS RID information pertaining to both the GCS and the UA can
   be sent by whichever has Internet connectivity to the Net-RID SP,
   typically the USS managing the UAS operation.

   The Net-RID SP forwards UAS RID information via the Internet to
   subscribed Net-RID DPs, typically the USS.  Subscribed Net-RID DPs
   then forward RID information via the Internet to subscribed Observer
   devices.  Regulations require and [F3411-22a ] describes UAS RID data
   elements that must be transported end to end from the UAS to the
   subscribed Observer devices.

   [F3411-22a ] prescribes the protocols between the Net-RID SP, Net-RID
   DP, and DSS.  It also prescribes data elements (in JSON) between the
   Observer and the Net-RID DP.  DRIP could address standardization of
   secure protocols between the UA and the GCS (over direct wireless and
   Internet connection), between the UAS and the Net-RID SP, and/or
   between the Net-RID DP and Observer devices.

   _Neither link-layer protocols nor the use of links (e.g., the link
   often existing between the GCS and the UA) for any purpose other than
   carriage of UAS RID information are in the scope of Network RID
   [F3411-22a ]._

### 1.3. Overview of USS Interoperability

   With Net-RID, there is direct communication between each UAS and its
   USS.  Multiple USS exchange information with the assistance of a DSS
   so all USS collectively have knowledge about all activities in a 4-D
   airspace.  The interactions among an Observer, multiple UAS, and
   their USS are shown in Figure 3.

                   +------+    +----------+    +------+
                   | UAS1 |    | Observer |    | UAS2 |
                   +---o--+    +-----o----+    +--o---+
                       |             |            |
                 ******|*************|************|******
                 *     |             |            |     *
                 *     |         +---o--+         |     *
                 *     |  .------o USS3 o------.  |     *
                 *     |  |      +--o---+      |  |     *
                 *     |  |         |          |  |     *
                 *   +-o--o-+    +--o--+     +-o--o-+   *
                 *   |      o----o DSS o-----o      |   *
                 *   | USS1 |    +-----+     | USS2 |   *
                 *   |      o----------------o      |   *
                 *   +------+                +------+   *
                 *                                      *
                 *               Internet               *
                 ****************************************

           Figure 3: Interactions Between Observers, UAS, and USS

### 1.4. Overview of DRIP Architecture

   Figure 4 illustrates a global UAS RID usage scenario.  Broadcast RID
   links are not shown, as they reach from any UA to any listening
   receiver in range and thus would obscure the intent of the figure.
   Figure 4 shows, as context, some entities and interfaces beyond the
   scope of DRIP (as currently (2023) chartered).  Multiple UAS are
   shown, each with its own UA controlled by its own GCS, potentially
   using the same or different USS, with the UA potentially
   communicating directly with each other (V2V), especially for low-
   latency, safety-related purposes (DAA).

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

                 Figure 4: Global UAS RID Usage Scenario

      |  Informative note: See [RFC9153] for detailed definitions.

   DRIP is meant to leverage existing Internet resources (standard
   protocols, services, infrastructures, and business models) to meet
   UAS RID and closely related needs.  DRIP will specify how to apply
   IETF standards, complementing [F3411-22a ] and other external
   standards, to satisfy UAS RID requirements.

   This document outlines the DRIP architecture in the context of the
   UAS RID architecture.  This includes closing the gaps between the
   CAAs' concepts of operations and [F3411-22a ] as it relates to the use
   of Internet technologies and UA-direct RF communications.  Issues
   include, but are not limited to:

   *  the design of trustworthy remote identifiers required by GEN-1
      (Section 3), especially but not exclusively for use as single-use
      session IDs,

   *  mechanisms to leverage the Domain Name System (DNS) [RFC1034] for
      registering and publishing public and private information (see
      Sections 4.1 and 4.2), as required by REG-1 and REG-2,

   *  specific authentication methods and message payload formats to
      enable verification that Broadcast RID messages were sent by the
      claimed sender (Section 5) and that the sender is in the claimed
      DRIP Identity Management Entity (DIME) (see Sections 4 and 5), as
      required by GEN-2,

   *  harvesting Broadcast RID messages for UTM inclusion, with the
      optional DRIP extension of Crowdsourced Remote ID (CS-RID)
      (Section 6), using the DRIP support for gateways required by GEN-5
      [RFC9153],

   *  methods for instantly establishing secure communications between
      an Observer and the pilot of an observed UAS (Section 7), using
      the DRIP support for dynamic contact required by GEN-4 [RFC9153],
      and

   *  privacy in UAS RID messages (personal data protection)
      (Section 10).

   This document should serve as a main point of entry into the set of
   IETF documents addressing the basic DRIP requirements.

## 3. HHIT as the DRIP Entity Identifier

   This section describes the DRIP architectural approach to meeting the
   basic requirements of a DRIP entity identifier within the external
   technical standard ASTM [F3411-22a ] and regulatory constraints.  It
   justifies and explains the use of Hierarchical Host Identity Tags
   (HHITs) [RFC9374] as self-asserting IPv6 addresses suitable as a UAS
   ID type and, more generally, as trustworthy multipurpose remote
   identifiers.

   Self-asserting in this usage means that, given the Host Identity
   (HI), the HHIT Overlay Routable Cryptographic Hash IDentifier
   (ORCHID) construction (see Section 3.5 of [RFC9374]), and a signature
   of the DIME on the HHIT and HI, the HHIT can be verified by the
   receiver as a trusted UAS ID.  The explicit registration hierarchy
   within the HHIT provides registration discovery (managed by a DIME)
   to either yield the HI for third-party (seeking UAS ID endorsement)
   validation or prove that the HHIT and HI have been registered
   uniquely.

### 3.1. UAS Remote Identifiers Problem Space

   A DRIP entity identifier needs to be "Trustworthy" (see DRIP
   requirements GEN-1, ID-4, and ID-5 in [RFC9153]).  This means that
   given a sufficient collection of UAS RID messages, an Observer can
   establish that the identifier claimed therein uniquely belongs to the
   claimant.  To satisfy DRIP requirements and maintain important
   security properties, the DRIP identifier should be self-generated by
   the entity it names (e.g., a UAS) and registered (e.g., with a USS;
   see DRIP requirements GEN-3 and ID-2).

   However, Broadcast RID, especially its support for Bluetooth 4,
   imposes severe constraints.  A previous revision of the ASTM UAS RID,
   [F3411-19], allowed a UAS ID of types (1, 2, and 3), each of 20
   bytes.  [F3411-22a ] adds type 4, Specific Session ID, for other
   Standards Development Organizations (SDOs) to extend ASTM UAS RID.
   Type 4 uses one byte to index the Specific Session ID subtype,
   leaving 19 bytes (see ID-1 of DRIP Requirements [RFC9153]).  As
   described in Section 3 of [RFC9153], ASTM has allocated Specific
   Session ID subtype 1 to IETF DRIP.

   The maximum ASTM UAS RID Authentication Message payload is 201 bytes
   each for Authentication Types 1, 2, 3, and 4.  [F3411-22a ] adds
   Authentication Type 5 for other SDOs (including the IETF) to extend
   ASTM UAS RID with Specific Authentication Methods (SAMs).  With Type
   5, one of the 201 bytes is consumed to index the SAM Type, leaving
   only 200 bytes for DRIP authentication payloads, including one or
   more DRIP entity identifiers and associated authentication data.

### 3.2. HHIT as a Cryptographic Identifier

   The only (known to the authors at the time of writing) existing types
   of IP-address-compatible identifiers cryptographically derived from
   the public keys of the identified entities are Cryptographically
   Generated Addresses (CGAs) [RFC3972] and Host Identity Tags (HITs)
   [RFC7401].  CGAs and HITs lack registration/retrieval capability.  To
   provide this, each HHIT embeds plaintext information designating the
   hierarchy within which it is registered, a cryptographic hash of that
   information concatenated with the entity's public key, etc.  Although
   hash collisions may occur, the DIME can detect them and reject
   registration requests rather than issue credentials, e.g., by
   enforcing a First Come First Served policy [RFC8126].  Preimage hash
   attacks are also mitigated through this registration process, locking
   the HHIT to a specific HI.

### 3.3. HHIT as a Trustworthy DRIP Entity Identifier

   A Remote UAS ID that can be trustworthy for use in Broadcast RID can
   be built from an asymmetric key pair.  In this method, the UAS ID is
   cryptographically derived directly from the public key.  The proof of
   UAS ID ownership (verifiable endorsement versus mere claim) is
   guaranteed by signing this cryptographic UAS ID with the associated
   private key.  The association between the UAS ID and the private key
   is ensured by cryptographically binding the public key with the UAS
   ID; more specifically, the UAS ID results from the hash of the public
   key.  The public key is designated as the HI, while the UAS ID is
   designated as the HIT.

   By construction, the HIT is statistically unique through the
   mandatory use of cryptographic hash functions with second-preimage
   resistance.  The cryptographically bound addition of the hierarchy
   and a HHIT registration process provide complete, global HHIT
   uniqueness.  This registration forces the attacker to generate the
   same public key rather than a public key that generates the same
   HHIT.  This is in contrast to general IDs (e.g., a Universally Unique
   Identifier (UUID) or device serial number) as the subject in an X.509
   certificate.

   A UA equipped for Broadcast RID MUST be provisioned not only with its
   HHIT but also with the HI public key from which the HHIT was derived
   and the corresponding private key to enable message signature.

   A UAS equipped for DRIP-enhanced Network RID MUST be provisioned
   likewise; the private key resides only in the ultimate source of
   Network RID messages.  If the GCS is the source of the Network RID
   messages, the GCS MUST hold the private key.  If the UA is the source
   of the Network RID messages and they are being relayed by the GCS,
   the UA MUST hold the private key, just as a UA that directly connects
   to the network rather than through its GCS.

   Each Observer device functioning with Internet connectivity MAY be
   provisioned either with public keys of the DRIP identifier root
   registries or certificates for subordinate registries; each Observer
   device that needs to operate without Internet connectivity at any
   time MUST be so provisioned.

   HHITs can also be used throughout the USS/UTM system.  Operators and
   Private Information Registries, as well as other UTM entities, can
   use HHITs for their IDs.  Such HHITs can facilitate DRIP security
   functions, such as those used with HIP, to strongly mutually
   authenticate and encrypt communications.

   A self-endorsement of a HHIT used as a UAS ID can be done in as
   little as 88 bytes when Ed25519 [RFC8032] is used by only including
   the 16-byte HHIT, two 4-byte timestamps, and the 64-byte Ed25519
   signature.

   Ed25519 [RFC8032] is used as the HHIT mandatory-to-implement signing
   algorithm, as GEN-1 and ID-5 [RFC9153] can best be met by restricting
   the HI to 32 bytes.  A larger public key would rule out the offline
   endorsement feature that fits within the 200-byte Authentication
   Message maximum length.  Other algorithms that meet this 32-byte
   constraint can be added as deemed needed.

   A DRIP identifier can be assigned to a UAS as a static HHIT by its
   manufacturer, such as a single HI and derived HHIT encoded as a
   hardware serial number, per [CTA2063A ].  Such a static HHIT SHOULD
   only be used to bind one-time-use DRIP identifiers to the unique UA.
   Depending upon implementation, this may leave a HI private key in the
   possession of the manufacturer (see also Section 9).

   In general, Internet access may be needed to validate Endorsements or
   Certificates.  This may be obviated in the most common cases (e.g.,
   endorsement of the UAS ID), even in disconnected environments, by
   prepopulating small caches on Observer devices with DIME public keys
   and a chain of Endorsements or Certificates (tracing a path through
   the DIME tree).  This is assuming all parties on the trust path also
   use HHITs for their identities.

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

## 5. DRIP Identifier Trust

   While the DRIP entity identifier is self-asserting, it alone does not
   provide the trustworthiness (i.e., non-repudiation, protection vs.
   spoofing, message integrity protection, scalability, etc.) essential
   to UAS RID, as justified in [RFC9153].  For that, it MUST be
   registered (under DRIP registries) and actively used by the party (in
   most cases the UA).  A sender's identity cannot be proved merely by
   its possessing of a DRIP Entity Tag (DET) and broadcasting it as a
   claim that it belongs to that sender.  Sending data signed using that
   HI's private key proves little, as it is subject to trivial replay
   attacks using previously broadcast messages.  Only sending the DET
   and a signature on novel (i.e., frequently changing and
   unpredictable) data that can be externally validated by the Observer
   (such as a signed Location/Vector message that matches actually
   seeing the UA at the location and time reported in the signed
   message) proves that the observed UA possesses the private key and
   thus the claimed UAS ID.

   The severe constraints of Broadcast RID make it challenging to
   satisfy UAS RID requirements.  From received Broadcast RID messages
   and information that can be looked up using the received UAS ID in
   online registries or local caches, it is possible to establish levels
   of trust in the asserted information and the operator.

   A combination of different DRIP Authentication Messages enables an
   Observer, without Internet connection (offline) or with (online), to
   validate a UAS DRIP ID in real time.  Some messages must contain the
   relevant registration of the UA's DRIP ID in the claimed DIME.  Some
   messages must contain sender signatures over both static (e.g.,
   registration) and dynamically changing (e.g., current UA location)
   data.  Combining these two sets of information, an Observer can piece
   together a chain of trust, including real-time evidence to make a
   determination on the UA's claims.

   This process (combining the DRIP entity identifier, registries, and
   authentication formats for Broadcast RID) can satisfy the following
   DRIP requirements defined in [RFC9153]: GEN-1, GEN-2, GEN-3, ID-2,
   ID-3, ID-4, and ID-5.

### A.2. UAS Service Supplier (USS)

   A USS plays an important role to fulfill the key performance
   indicators (KPIs) that UTM has to offer.  Such an entity acts as a
   proxy between UAS operators and UTM service providers.  It provides
   services like real-time UAS traffic monitoring and planning,
   aeronautical data archiving, airspace and violation control,
   interacting with other third-party control entities, etc.  A USS can
   coexist with other USS to build a large service coverage map that can
   load-balance, relay, and share UAS traffic information.

   The FAA works with UAS industry shareholders and promotes the Low
   Altitude Authorization and Notification Capability [LAANC ] program,
   which is the first system to realize some of the envisioned
   functionality of UTM.  The LAANC program can automate UAS operational
   intent (flight plan) submissions and applications for airspace
   authorization in real time by checking against multiple aeronautical
   databases, such as airspace classification and operating rules
   associated with it, the FAA UAS facility map, special use airspace,
   Notice to Airmen (NOTAM), and Temporary Flight Restriction (TFR).

# Referenced Sections from RFC 9374: DRIP Entity Tag (DET) for Unmanned Aircraft System Remote ID (UAS RID)

The following sections were referenced. Remaining sections are not included.

## 1. Introduction

   Drone Remote ID Protocol (DRIP) Requirements [RFC9153] describe an
   Unmanned Aircraft System Remote ID (UAS ID) as unique (ID-4), non-
   spoofable (ID-5), and identify a registry where the ID is listed
   (ID-2); all within a 19-character identifier (ID-1).

   This RFC is a foundational document of DRIP, as it describes the use
   of Hierarchical Host Identity Tags (HHITs) (Section 3) as self-
   asserting IPv6 addresses and thereby a trustable identifier for use
   as the UAS Remote ID (see Section 3 of [DRIP-ARCH ]).  All other DRIP-
   related technologies will enable or use HHITs as multipurpose remote
   identifiers.  HHITs add explicit hierarchy to the 128-bit HITs,
   enabling DNS HHIT queries (Host ID for authentication, e.g.,
   [DRIP-AUTH ]) and use with a Differentiated Access Control (e.g.,
   Registration Data Access Protocol (RDAP) [RFC9224]) for 3rd-party
   identification endorsement (e.g., [DRIP-AUTH ]).

   The addition of hierarchy to HITs is an extension to [RFC7401] and
   requires an update to [RFC7343].  As this document also adds EdDSA
   (Section 3.4) for Host Identities (HIs), a number of Host Identity
   Protocol (HIP) parameters in [RFC7401] are updated, but these should
   not be needed in a DRIP implementation that does not use HIP.

   HHITs as used within the context of UAS are labeled as DRIP Entity
   Tags (DETs).  Throughout this document, HHIT and DET will be used
   appropriately.  HHIT will be used when covering the technology, and
   DET will be used in the context of UAS RID.

   HHITs provide self-claims of the HHIT registry.  A HHIT can only be
   in a single registry within a registry system (e.g., DNS).

   HHITs are valid, though non-routable, IPv6 addresses [RFC8200].  As
   such, they fit in many ways within various IETF technologies.

### 1.1. HHIT Statistical Uniqueness Different from UUID or X.509 Subject

   HHITs are statistically unique through the cryptographic hash feature
   of second-preimage resistance.  The cryptographically bound addition
   of the hierarchy and a HHIT registration process [DRIP-REG ] provide
   complete, global HHIT uniqueness.  If the HHITs cannot be looked up
   with services provided by the DRIP Identity Management Entity (DIME)
   identified via the embedded hierarchical information or its
   registration validated by registration endorsement messages
   [DRIP-AUTH ], then the HHIT is either fraudulent or revoked/expired.
   In-depth discussion of these processes are out of scope for this
   document.

   This contrasts with using general identifiers (e.g., Universally
   Unique IDentifiers (UUIDs) [RFC4122] or device serial numbers) as the
   subject in an X.509 [RFC5280] certificate.  In either case, there can
   be no unique proof of ownership/registration.

   For example, in a multi-Certificate Authority (multi-CA) PKI
   alternative to HHITs, a Remote ID as the Subject (Section 4.1.2.6 of
   [RFC5280]) can occur in multiple CAs, possibly fraudulently.  CAs
   within the PKI would need to implement an approach to enforce
   assurance of the uniqueness achieved with HHITs.

### 4.3. Remote ID DET as one Class of HHITs

   UAS Remote ID DET may be one of a number of uses of HHITs.  However,
   it is out of the scope of the document to elaborate on other uses of
   HHITs.  As such these follow-on uses need to be considered in
   allocating the RAAs (Section 3.3.1) or HHIT prefix assignments
   (Section 8).

### 4.4. Hierarchy in ORCHID Generation

   ORCHIDS, as defined in [RFC7343], do not cryptographically bind an
   IPv6 prefix or the OGA ID (the HIT Suite ID) to the hash of the HI.
   At the time ORCHID was being developed, the rationale was attacks
   against these fields are Denial-of-Service (DoS) attacks against
   protocols using ORCHIDs and thus it was up to those protocols to
   address the issue.

   HHITs, as defined in Section 3.5, cryptographically bind all content
   in the ORCHID through the hashing function.  A recipient of a DET
   that has the underlying HI can directly trust and act on all content
   in the HHIT.  This provides a strong, self-claim for using the
   hierarchy to find the DET Registry based on the HID (Section 4.5).

### 4.5. DRIP Entity Tag (DET) Registry

   DETs are registered to HDAs.  The registration process defined in
   [DRIP-REG ] ensures DET global uniqueness (ID-4 in Section 4.2.1 of
   [RFC9153]).  It also allows the mechanism to create UAS public/
   private data that are associated with the DET (REG-1 and REG-2 in Section 4.4.1 of [RFC9153]).

### 4.6. Remote ID Authentication Using DETs

   The EdDSA25519 HI (Section 3.4) underlying the DET can be used in an
   88-byte self-proof evidence (timestamps, HHIT, and signature of
   these) to provide proof to Observers of Remote ID ownership (GEN-1 in Section 4.1.1 of [RFC9153]).  In practice, the Wrapper and Manifest
   authentication formats (Sections 6.3.3 and 6.3.4 of [DRIP-AUTH ])
   implicitly provide this self-proof evidence.  A lookup service like
   DNS can provide the HI and registration proof (GEN-3 in [RFC9153]).

   Similarly, for Observers without Internet access, a 200-byte offline
   self-endorsement (Section 3.1.2 of [DRIP-AUTH ]) could provide the
   same Remote ID ownership proof.  This endorsement would contain the
   HDA's signing of the UA's HHIT, itself signed by the UA's HI.  Only a
   small cache (also Section 3.1.2 of [DRIP-AUTH ]) that contains the
   HDA's HI/HHIT and HDA meta-data is needed by the Observer.  However,
   such an object would just fit in the ASTM Authentication Message
   (Section 2.2 of [RFC9153]) with no room for growth.  In practice,
   [DRIP-AUTH ] provides this offline self-endorsement in two
   authentication messages: the HDA's endorsement of the UA's HHIT
   registration in a Link authentication message whose hash is sent in a
   Manifest authentication message.

   Hashes of any previously sent ASTM messages can be placed in a
   Manifest authentication message (GEN-2 in [RFC9153]).  When a
   Location/Vector Message (i.e., a message that provides UA location,
   altitude, heading, speed, and status) hash along with the hash of the
   HDA's UA HHIT endorsement are sent in a Manifest authentication
   message and the Observer can visually see a UA at the claimed
   location, the Observer has very strong proof of the UA's Remote ID.

   This behavior and how to mix these authentication messages into the
   flow of UA operation messages are detailed in [DRIP-AUTH ].

# Referenced Sections from RFC 1034: Domain names - concepts and facilities

The following sections were referenced. Remaining sections are not included.

## 2. INTRODUCTION

This RFC introduces domain style names, their use for Internet mail and
host address support, and the protocols and servers used to implement
domain name facilities.

### 2.1. The history of domain names

The impetus for the development of the domain system was growth in the
Internet:

   - Host name to address mappings were maintained by the Network
     Information Center (NIC) in a single file (HOSTS.TXT) which
     was FTPed by all hosts [RFC-952, RFC-953].  The total network 
     bandwidth consumed in distributing a new version by this
     scheme is proportional to the square of the number of hosts in
     the network, and even when multiple levels of FTP are used,
     the outgoing FTP load on the NIC host is considerable.
     Explosive growth in the number of hosts didn't bode well for
     the future.

   - The network population was also changing in character.  The
     timeshared hosts that made up the original ARPANET were being
     replaced with local networks of workstations.  Local
     organizations were administering their own names and
     addresses, but had to wait for the NIC to change HOSTS.TXT to
     make changes visible to the Internet at large.  Organizations
     also wanted some local structure on the name space.

   - The applications on the Internet were getting more
     sophisticated and creating a need for general purpose name
     service.


The result was several ideas about name spaces and their management
[IEN-116, RFC-799, RFC-819, RFC-830].  The proposals varied, but a
common thread was the idea of a hierarchical name space, with the
hierarchy roughly corresponding to organizational structure, and names
using "."  as the character to mark the boundary between hierarchy
levels.  A design using a distributed database and generalized resources
was described in [RFC-882, RFC-883].  Based on experience with several
implementations, the system evolved into the scheme described in this
memo.

The terms "domain" or "domain name" are used in many contexts beyond the
DNS described here.  Very often, the term domain name is used to refer
to a name with structure indicated by dots, but no relation to the DNS.
This is particularly true in mail addressing [Quarterman 86].

### 2.2. DNS design goals

The design goals of the DNS influence its structure.  They are:

   - The primary goal is a consistent name space which will be used
     for referring to resources.  In order to avoid the problems
     caused by ad hoc encodings, names should not be required to
     contain network identifiers, addresses, routes, or similar
     information as part of the name.

   - The sheer size of the database and frequency of updates
     suggest that it must be maintained in a distributed manner,
     with local caching to improve performance.  Approaches that 
     attempt to collect a consistent copy of the entire database
     will become more and more expensive and difficult, and hence
     should be avoided.  The same principle holds for the structure
     of the name space, and in particular mechanisms for creating
     and deleting names; these should also be distributed.

   - Where there tradeoffs between the cost of acquiring data, the
     speed of updates, and the accuracy of caches, the source of
     the data should control the tradeoff.

   - The costs of implementing such a facility dictate that it be
     generally useful, and not restricted to a single application.
     We should be able to use names to retrieve host addresses,
     mailbox data, and other as yet undetermined information.  All
     data associated with a name is tagged with a type, and queries
     can be limited to a single type.

   - Because we want the name space to be useful in dissimilar
     networks and applications, we provide the ability to use the
     same name space with different protocol families or
     management.  For example, host address formats differ between
     protocols, though all protocols have the notion of address.
     The DNS tags all data with a class as well as the type, so
     that we can allow parallel use of different formats for data
     of type address.

   - We want name server transactions to be independent of the
     communications system that carries them.  Some systems may
     wish to use datagrams for queries and responses, and only
     establish virtual circuits for transactions that need the
     reliability (e.g., database updates, long transactions); other
     systems will use virtual circuits exclusively.

   - The system should be useful across a wide spectrum of host
     capabilities.  Both personal computers and large timeshared
     hosts should be able to use the system, though perhaps in
     different ways.

### 2.3. Assumptions about usage

The organization of the domain system derives from some assumptions
about the needs and usage patterns of its user community and is designed
to avoid many of the the complicated problems found in general purpose
database systems.

The assumptions are:

   - The size of the total database will initially be proportional 
     to the number of hosts using the system, but will eventually
     grow to be proportional to the number of users on those hosts
     as mailboxes and other information are added to the domain
     system.

   - Most of the data in the system will change very slowly (e.g.,
     mailbox bindings, host addresses), but that the system should
     be able to deal with subsets that change more rapidly (on the
     order of seconds or minutes).

   - The administrative boundaries used to distribute
     responsibility for the database will usually correspond to
     organizations that have one or more hosts.  Each organization
     that has responsibility for a particular set of domains will
     provide redundant name servers, either on the organization's
     own hosts or other hosts that the organization arranges to
     use.

   - Clients of the domain system should be able to identify
     trusted name servers they prefer to use before accepting
     referrals to name servers outside of this "trusted" set.

   - Access to information is more critical than instantaneous
     updates or guarantees of consistency.  Hence the update
     process allows updates to percolate out through the users of
     the domain system rather than guaranteeing that all copies are
     simultaneously updated.  When updates are unavailable due to
     network or host failure, the usual course is to believe old
     information while continuing efforts to update it.  The
     general model is that copies are distributed with timeouts for
     refreshing.  The distributor sets the timeout value and the
     recipient of the distribution is responsible for performing
     the refresh.  In special situations, very short intervals can
     be specified, or the owner can prohibit copies.

   - In any system that has a distributed database, a particular
     name server may be presented with a query that can only be
     answered by some other server.  The two general approaches to
     dealing with this problem are "recursive", in which the first
     server pursues the query for the client at another server, and
     "iterative", in which the server refers the client to another
     server and lets the client pursue the query.  Both approaches
     have advantages and disadvantages, but the iterative approach
     is preferred for the datagram style of access.  The domain
     system requires implementation of the iterative approach, but
     allows the recursive approach as an option. 
The domain system assumes that all data originates in master files
scattered through the hosts that use the domain system.  These master
files are updated by local system administrators.  Master files are text
files that are read by a local name server, and hence become available
through the name servers to users of the domain system.  The user
programs access name servers through standard programs called resolvers.

The standard format of master files allows them to be exchanged between
hosts (via FTP, mail, or some other mechanism); this facility is useful
when an organization wants a domain, but doesn't want to support a name
server.  The organization can maintain the master files locally using a
text editor, transfer them to a foreign host which runs a name server,
and then arrange with the system administrator of the name server to get
the files loaded.

Each host's name servers and resolvers are configured by a local system
administrator [RFC-1033].  For a name server, this configuration data
includes the identity of local master files and instructions on which
non-local master files are to be loaded from foreign servers.  The name
server uses the master files or copies to load its zones.  For
resolvers, the configuration data identifies the name servers which
should be the primary sources of information.

The domain system defines procedures for accessing the data and for
referrals to other name servers.  The domain system also defines
procedures for caching retrieved data and for periodic refreshing of
data defined by the system administrator.

The system administrators provide:

   - The definition of zone boundaries.

   - Master files of data.

   - Updates to master files.

   - Statements of the refresh policies desired.

The domain system provides:

   - Standard formats for resource data.

   - Standard methods for querying the database.

   - Standard methods for name servers to refresh local data from
     foreign name servers. 





### 2.4. Elements of the DNS

The DNS has three major components:

   - The DOMAIN NAME SPACE and RESOURCE RECORDS, which are
     specifications for a tree structured name space and data
     associated with the names.  Conceptually, each node and leaf
     of the domain name space tree names a set of information, and
     query operations are attempts to extract specific types of
     information from a particular set.  A query names the domain
     name of interest and describes the type of resource
     information that is desired.  For example, the Internet
     uses some of its domain names to identify hosts; queries for
     address resources return Internet host addresses.

   - NAME SERVERS are server programs which hold information about
     the domain tree's structure and set information.  A name
     server may cache structure or set information about any part
     of the domain tree, but in general a particular name server
     has complete information about a subset of the domain space,
     and pointers to other name servers that can be used to lead to
     information from any part of the domain tree.  Name servers
     know the parts of the domain tree for which they have complete
     information; a name server is said to be an AUTHORITY for
     these parts of the name space.  Authoritative information is
     organized into units called ZONEs, and these zones can be
     automatically distributed to the name servers which provide
     redundant service for the data in a zone.

   - RESOLVERS are programs that extract information from name
     servers in response to client requests.  Resolvers must be
     able to access at least one name server and use that name
     server's information to answer a query directly, or pursue the
     query using referrals to other name servers.  A resolver will
     typically be a system routine that is directly accessible to
     user programs; hence no protocol is necessary between the
     resolver and the user program.

These three components roughly correspond to the three layers or views
of the domain system:

   - From the user's point of view, the domain system is accessed
     through a simple procedure or OS call to a local resolver.
     The domain space consists of a single tree and the user can
     request information from any section of the tree.

   - From the resolver's point of view, the domain system is
     composed of an unknown number of name servers.  Each name 
     server has one or more pieces of the whole domain tree's data,
     but the resolver views each of these databases as essentially
     static.

   - From a name server's point of view, the domain system consists
     of separate sets of local information called zones.  The name
     server has local copies of some of the zones.  The name server
     must periodically refresh its zones from master copies in
     local files or foreign name servers.  The name server must
     concurrently process queries that arrive from resolvers.

In the interests of performance, implementations may couple these
functions.  For example, a resolver on the same machine as a name server
might share a database consisting of the the zones managed by the name
server and the cache managed by the resolver.

# Referenced Sections from RFC 1035: Domain names - implementation and specification

The following sections were referenced. Remaining sections are not included.

## 9. REFERENCES and BIBLIOGRAPHY

[Dyer 87]       S. Dyer, F. Hsu, "Hesiod", Project Athena
                Technical Plan - Name Service, April 1987, version 1.9.

                Describes the fundamentals of the Hesiod name service.

[IEN-116]       J. Postel, "Internet Name Server", IEN-116,
                USC/Information Sciences Institute, August 1979.

                A name service obsoleted by the Domain Name System, but
                still in use.

[Quarterman 86] J. Quarterman, and J. Hoskins, "Notable Computer Networks",
                Communications of the ACM, October 1986, volume 29, number
                10.

[RFC-742]       K. Harrenstien, "NAME/FINGER", RFC-742, Network
                Information Center, SRI International, December 1977.

[RFC-768]       J. Postel, "User Datagram Protocol", RFC-768,
                USC/Information Sciences Institute, August 1980.

[RFC-793]       J. Postel, "Transmission Control Protocol", RFC-793,
                USC/Information Sciences Institute, September 1981.

[RFC-799]       D. Mills, "Internet Name Domains", RFC-799, COMSAT,
                September 1981.

                Suggests introduction of a hierarchy in place of a flat
                name space for the Internet.

[RFC-805]       J. Postel, "Computer Mail Meeting Notes", RFC-805,
                USC/Information Sciences Institute, February 1982.

[RFC-810]       E. Feinler, K. Harrenstien, Z. Su, and V. White, "DOD
                Internet Host Table Specification", RFC-810, Network
                Information Center, SRI International, March 1982.

                Obsolete.  See RFC-952.

[RFC-811]       K. Harrenstien, V. White, and E. Feinler, "Hostnames
                Server", RFC-811, Network Information Center, SRI
                International, March 1982. 
                Obsolete.  See RFC-953.

[RFC-812]       K. Harrenstien, and V. White, "NICNAME/WHOIS", RFC-812,
                Network Information Center, SRI International, March
                1982.

[RFC-819]       Z. Su, and J. Postel, "The Domain Naming Convention for
                Internet User Applications", RFC-819, Network
                Information Center, SRI International, August 1982.

                Early thoughts on the design of the domain system.
                Current implementation is completely different.

[RFC-821]       J. Postel, "Simple Mail Transfer Protocol", RFC-821,
                USC/Information Sciences Institute, August 1980.

[RFC-830]       Z. Su, "A Distributed System for Internet Name Service",RFC-830, Network Information Center, SRI International,
                October 1982.

                Early thoughts on the design of the domain system.
                Current implementation is completely different.

[RFC-882]       P. Mockapetris, "Domain names - Concepts and
                Facilities," RFC-882, USC/Information Sciences
                Institute, November 1983.

                Superceeded by this memo.

[RFC-883]       P. Mockapetris, "Domain names - Implementation and
                Specification," RFC-883, USC/Information Sciences
                Institute, November 1983.

                Superceeded by this memo.

[RFC-920]       J. Postel and J. Reynolds, "Domain Requirements",RFC-920, USC/Information Sciences Institute,
                October 1984.

                Explains the naming scheme for top level domains.

[RFC-952]       K. Harrenstien, M. Stahl, E. Feinler, "DoD Internet Host
                Table Specification", RFC-952, SRI, October 1985.

                Specifies the format of HOSTS.TXT, the host/address
                table replaced by the DNS. 
[RFC-953]       K. Harrenstien, M. Stahl, E. Feinler, "HOSTNAME Server",RFC-953, SRI, October 1985.

                This RFC contains the official specification of the
                hostname server protocol, which is obsoleted by the DNS.
                This TCP based protocol accesses information stored in
                the RFC-952 format, and is used to obtain copies of the
                host table.

[RFC-973]       P. Mockapetris, "Domain System Changes and
                Observations", RFC-973, USC/Information Sciences
                Institute, January 1986.

                Describes changes to RFC-882 and RFC-883 and reasons for
                them.

[RFC-974]       C. Partridge, "Mail routing and the domain system",RFC-974, CSNET CIC BBN Labs, January 1986.

                Describes the transition from HOSTS.TXT based mail
                addressing to the more powerful MX system used with the
                domain system.

[RFC-1001]      NetBIOS Working Group, "Protocol standard for a NetBIOS
                service on a TCP/UDP transport: Concepts and Methods",RFC-1001, March 1987.

                This RFC and RFC-1002 are a preliminary design for
                NETBIOS on top of TCP/IP which proposes to base NetBIOS
                name service on top of the DNS.

[RFC-1002]      NetBIOS Working Group, "Protocol standard for a NetBIOS
                service on a TCP/UDP transport: Detailed
                Specifications", RFC-1002, March 1987.

[RFC-1010]      J. Reynolds, and J. Postel, "Assigned Numbers", RFC-1010,
                USC/Information Sciences Institute, May 1987.

                Contains socket numbers and mnemonics for host names,
                operating systems, etc.

[RFC-1031]      W. Lazear, "MILNET Name Domain Transition", RFC-1031,
                November 1987.

                Describes a plan for converting the MILNET to the DNS.

[RFC-1032]      M. Stahl, "Establishing a Domain - Guidelines for
                Administrators", RFC-1032, November 1987. 
                Describes the registration policies used by the NIC to
                administer the top level domains and delegate subzones.

[RFC-1033]      M. Lottor, "Domain Administrators Operations Guide",RFC-1033, November 1987.

                A cookbook for domain administrators.

[Solomon 82]    M. Solomon, L. Landweber, and D. Neuhengen, "The CSNET
                Name Server", Computer Networks, vol 6, nr 3, July 1982.

                Describes a name service for CSNET which is independent
                from the DNS and DNS use in the CSNET. 

# Referenced Sections from RFC 9575: DRIP Entity Tag (DET) Authentication Formats and Protocols for Broadcast Remote Identification (RID)

The following sections were referenced. Remaining sections are not included.

### 3.2. ASTM Authentication Message Framing

   The Authentication Message (Message Type 0x2) is unique in the ASTM
   [F3411] Broadcast standard, as it is the only message that can be
   larger than the Legacy Transport size.  To address this limitation
   around transport size, it is defined as a set of "pages", each of
   which fits into a single Legacy Transport frame.  For Extended
   Transports, pages are still used but they are all in a single frame.

      |  Informational Note: Message Pack (Message Type 0xF) is also
      |  larger than the Legacy Transport size but is limited for use
      |  only on Extended Transports where it can be supported.

   The following subsections are a brief overview of the Authentication
   Message format defined in [F3411] for better context on how DRIP
   Authentication fills and uses various fields already defined by ASTM
   [F3411].

#### 3.2.1. Authentication Page

   This document leverages Authentication Type 0x5 (Specific
   Authentication Method (SAM)) as the principal authentication
   container, defining a set of SAM Types in Section 4.  Authentication
   Type is encoded in every Authentication Page in the _Page Header_.
   The SAM Type is defined as a field in the _Authentication Payload_
   (see Section 3.2.3).

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +---------------+---------------+---------------+---------------+
     |  Page Header  |                                               |
     +---------------+                                               |
     |                                                               |
     |                                                               |
     |                     Authentication Payload                    |
     |                                                               |
     |                                                               |
     +---------------+---------------+---------------+---------------+

            Figure 1: Standard ASTM Authentication Message Page

   _Page Header_:  (1 octet)

      Authentication Type (4 bits) and Page Number (4 bits)

   _Authentication Payload_:  (23 octets per page)

      Authentication Payload, including headers.  Null padded.  See Section 3.2.2.

   The Authentication Message is structured as a set of pages per
   Figure 1.  There is a technical maximum of 16 pages (indexed 0 to 15)
   that can be sent for a single Authentication Message, with each page
   carrying a maximum 23-octet _Authentication Payload_. See Section 3.2.4 for more details.  Over Legacy Transports, these
   messages are "fragmented", with each page sent in a separate Legacy
   Transport frame.

   Either as a single Authentication Message or a set of fragmented
   Authentication Message Pages, the structure is further wrapped by
   outer ASTM framing and the specific link framing.

#### 3.2.2. Authentication Payload Field

   Figure 2 is the source data view of the data fields found in the
   Authentication Message as defined by [F3411].  This data is placed
   into the _Authentication Payload_ shown in Figure 1, which spans
   multiple _Authentication Pages_.

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +---------------+---------------+---------------+---------------+
     |                     Authentication Headers                    |
     |                               +---------------+---------------+
     |                               |                               |
     +---------------+---------------+                               |
     .                                                               .
     .                Authentication Data / Signature                .
     .                                                               .
     |                                                               |
     +---------------+---------------+---------------+---------------+
     |      ADL      |                                               |
     +---------------+                                               |
     .                                                               .
     .                       Additional Data                         .
     .                                                               .
     |                                                               |
     +---------------+---------------+---------------+---------------+

                Figure 2: ASTM Authentication Message Fields

   _Authentication Headers_:  (6 octets)

      As defined in [F3411].

   _Authentication Data / Signature_:  (0 to 255 octets)

      Opaque authentication data.  The length of this payload is known
      through a field in the _Authentication Headers_ (defined in
      [F3411]).

   _Additional Data Length (ADL)_:  (1 octet - unsigned)

      Length in octets of _Additional Data_. The value of _ADL_ is
      calculated as the minimum of 361 - Authentication Data / Signature
      Length and 255.  Only present with _Additional Data_.

   _Additional Data:_  (_ADL_ octets)

      Data that follows the _Authentication Data / Signature_ but is not
      considered part of the _Authentication Data_, and thus is not
      covered by a signature.  For DRIP, this field is used to carry
      Forward Error Correction (FEC) generated by transmitters and
      parsed by receivers as defined in Section 5.

#### 3.2.3. SAM Data Format

   Figure 3 is the general format to hold authentication data when using
   SAM and is placed inside the _Authentication Data / Signature_ field
   in Figure 2.

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +---------------+---------------+---------------+---------------+
     |   SAM Type    |                                               |
     +---------------+                                               |
     .                                                               .
     .                     SAM Authentication Data                   .
     .                                                               .
     |                                                               |
     +---------------+---------------+---------------+---------------+

                         Figure 3: SAM Data Format

   _SAM Type_:  (1 octet)

      The following SAM Types are allocated to DRIP:

                  +==========+=============================+
                  | SAM Type | Description                 |
                  +==========+=============================+
                  | 0x01     | DRIP Link (Section 4.2)     |
                  +----------+-----------------------------+
                  | 0x02     | DRIP Wrapper (Section 4.3)  |
                  +----------+-----------------------------+
                  | 0x03     | DRIP Manifest (Section 4.4) |
                  +----------+-----------------------------+
                  | 0x04     | DRIP Frame (Section 4.5)    |
                  +----------+-----------------------------+

                           Table 1: DRIP SAM Types

      |  Note: ASTM International is the owner of these code points as
      |  they are defined in [F3411].  In accordance with Annex 5 of
      |  [F3411], the International Civil Aviation Organization (ICAO)
      |  has been selected by ASTM as the registrar to manage
      |  allocations of these code points.  The list is available at
      |  [ASTM-Remote-ID ].

   _SAM Authentication Data_:  (0 to 200 octets)

      Contains opaque authentication data formatted as defined by the
      preceding SAM Type.

#### 3.2.4. ASTM Broadcast RID Constraints





##### 3.2.4.1. Wireless Frame Constraints

   A UA has the option to broadcast using Bluetooth (4.x and 5.x), Wi-Fi
   NAN, or IEEE 802.11 Beacon; see Section 6.  With Bluetooth, FAA and
   other Civil Aviation Authorities (CAA) mandate transmitting
   simultaneously over both 4.x and 5.x.  The same application-layer
   information defined in [F3411] MUST be transmitted over all the
   physical-layer interfaces performing RID, because Observer transports
   may be limited.  If an Observer can support multiple transports, it
   should use (display, report, etc.) the latest data regardless of the
   transport over which that data was received.

   Bluetooth 4.x presents a payload-size challenge in that it can only
   transmit 25 octets of payload per frame, while other transports can
   support larger payloads per frame.  As [F3411] message formats are
   the same for all media, and their framing was designed to fit within
   these legacy constraints, Extended Transports cannot send larger
   messages; instead, the Message Pack format encapsulates multiple
   messages (each of which fits within these legacy constraints).

   By definition Extended Transports provide FEC, but Legacy Transports
   lack FEC.  Thus over Legacy Transports, paged Authentication Messages
   may suffer the loss of one or more pages.  This would result in
   delivery to the Observer application of incomplete (typically
   unusable) messages, so DRIP FEC (Section 5) is specified to enable
   recovery of a single lost page and thereby reduce the likelihood of
   receiving incompletely reconstructable Authentication Messages.
   Authentication Messages sent using Extended Transports do not suffer
   this issue, as the full message (all pages) is sent using a single
   Message Pack.  Furthermore, the use of one-way RF broadcasts
   prohibits the use of any congestion-control or loss-recovery schemes
   that require ACKs or NACKs.

##### 3.2.4.2. Paged Authentication Message Constraints

   To keep consistent formatting across the different transports (Legacy
   and Extended) and their independent restrictions, the authentication
   data being sent is REQUIRED to fit within the page limit that the
   most constrained existing transport can support.  Under Broadcast
   RID, the Extended Transport that can hold the least amount of
   authentication data is Bluetooth 5.x at 9 pages.

   As such, DRIP transmitters are REQUIRED to adhere to the following
   when using the Authentication Message:

   1.  _Authentication Data / Signature_ data MUST fit in the first 9
       pages (Page Numbers 0 through 8).

   2.  The _Length_ field in the _Authentication Headers_ (which encodes
       the length in octets of _Authentication Data / Signature_ only)
       MUST NOT exceed the value of 201.  This includes the SAM Type but
       excludes _Additional Data_.

##### 3.2.4.3. Timestamps

   In ASTM [F3411], timestamps are a Unix-style timestamp with an epoch
   of 2019-01-01 00:00:00 UTC.  For DRIP, this format is adopted for
   Authentication to keep a common time format in Broadcast payloads.

   Under DRIP, there are two timestamps defined: Valid Not Before (VNB)
   and Valid Not After (VNA).

   Valid Not Before (VNB) Timestamp:  (4 octets)

      Timestamp denoting the recommended time at which to start trusting
      data.  MUST follow the format defined in [F3411] as described
      above.  MUST be set no earlier than the time the signature (across
      a given structure) is generated.

   Valid Not After (VNA) Timestamp:  (4 octets)

      Timestamp denoting the recommended time at which to stop trusting
      data.  MUST follow the format defined in [F3411] as described
      above.  Has an offset (relative to VNB) to avoid replay attacks.
      The exact offset is not defined in this document.  Best practice
      for identifying an acceptable offset should be used and should
      take into consideration the UA environment, propagation
      characteristics of the messages being sent, and clock differences
      between the UA and Observers.  For UA signatures in scenarios
      typical as of 2024, a reasonable offset would be to set VNA
      approximately 2 minutes after VNB; see Appendix B  for examples
      that may aid in tuning this value.

