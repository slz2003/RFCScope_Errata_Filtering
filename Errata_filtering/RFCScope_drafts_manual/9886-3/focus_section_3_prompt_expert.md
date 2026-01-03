Let us analyze section 3 of RFC 9886. All references made by section 3 have also been included below.

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

## 2. Terms and Definitions





### 2.1. Requirements Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in
   this document are to be interpreted as described in BCP 14 [RFC2119]
   [RFC8174] when, and only when, they appear in all capitals, as shown
   here.

   The document includes a set of algorithms and recommends the ones
   that should be supported by implementations.  The following term is
   used for that purpose: RECOMMENDED.

### 2.2. Notation

   |  Signifies concatenation of information, e.g., X | Y is the
      concatenation of X and Y.

### 2.3. Definitions

   This document uses the terms defined in Section 2.2 of [RFC9153] and
   in Section 2 of [DRIP-ARCH ].  The following terms are used in the
   document:

   cSHAKE (The customizable SHAKE function [NIST.SP.800-185]):
      Extends the SHAKE scheme [NIST.FIPS.202] to allow users to
      customize their use of the SHAKE function.

   HDA (HHIT Domain Authority):
      The 14-bit field that identifies the HHIT Domain Authority under a
      Registered Assigning Authority (RAA).  See Figure 1.

   HHIT (Hierarchical Host Identity Tag):
      A HIT with extra hierarchical information not found in a standard
      HIT [RFC7401].

   HI (Host Identity):
      The public key portion of an asymmetric key pair as defined in
      [RFC9063].

   HID (Hierarchy ID):
      The 28-bit field providing the HIT Hierarchy ID.  See Figure 1.

   HIP (Host Identity Protocol):
      The origin of HI, HIT, and HHIT [RFC7401].

   HIT (Host Identity Tag):
      A 128-bit handle on the HI.  HITs are valid IPv6 addresses.

   Keccak (KECCAK Message Authentication Code):
      The family of all sponge functions with a KECCAK-f permutation as
      the underlying function and multi-rate padding as the padding
      rule.  In particular, it refers to all the functions referenced
      from [NIST.FIPS.202] and [NIST.SP.800-185].

   KMAC (KECCAK Message Authentication Code [NIST.SP.800-185]):
      A Pseudo Random Function (PRF) and keyed hash function based on
      KECCAK.

   RAA (Registered Assigning Authority):
      The 14-bit field identifying the business or organization that
      manages a registry of HDAs.  See Figure 1.

   RVS (Rendezvous Server):
      A Rendezvous Server such as the HIP Rendezvous Server for enabling
      mobility, as defined in [RFC8004].

   SHAKE (Secure Hash Algorithm KECCAK [NIST.FIPS.202]):
      A secure hash that allows for an arbitrary output length.

   XOF (eXtendable-Output Function [NIST.FIPS.202]):
      A function on bit strings (also called messages) in which the
      output can be extended to any desired length.

## 3. The Hierarchical Host Identity Tag (HHIT)

   The HHIT is a small but important enhancement over the flat Host
   Identity Tag (HIT) space, constructed as an Overlay Routable
   Cryptographic Hash IDentifier (ORCHID) [RFC7343].  By adding two
   levels of hierarchical administration control, the HHIT provides for
   device registration/ownership, thereby enhancing the trust framework
   for HITs.

   The 128-bit HHITs represent the HI in only a 64-bit hash, rather than
   the 96 bits in HITs. 4 of these 32 freed up bits expand the Suite ID
   to 8 bits, and the other 28 bits are used to create a hierarchical
   administration organization for HIT domains.  HHIT construction is
   defined in Section 3.5.  The input values for the encoding rules are
   described in Section 3.5.1.

   A HHIT is built from the following fields (Figure 1):

   *  p = an IPv6 prefix (max 28 bit)

   *  28-bit HID which provides the structure to organize HITs into
      administrative domains.  HIDs are further divided into two fields:

      -  14-bit Registered Assigning Authority (RAA) (Section 3.3.1)

      -  14-bit HHIT Domain Authority (HDA) (Section 3.3.2)

   *  8-bit HHIT Suite ID (HHSI)

   *  ORCHID hash (92 - prefix length, e.g., 64) See Section 3.5 for
      more details.

                  14 bits| 14 bits              8 bits
                 +-------+-------+         +--------------+
                 |  RAA  | HDA   |         |HHIT Suite ID |
                 +-------+-------+         +--------------+
                  \              |    ____/   ___________/
                   \             \  _/    ___/
                    \             \/     /
      |    p bits    |  28 bits   |8bits|      o=92-p bits       |
      +--------------+------------+-----+------------------------+
      | IPv6 Prefix  |    HID     |HHSI |      ORCHID hash       |
      +--------------+------------+-----+------------------------+

                           Figure 1: HHIT Format

   The Context ID (generated with openssl rand) for the ORCHID hash is:

       Context ID :=  0x00B5 A69C 795D F5D5 F008 7F56 843F 2C40

   Context IDs are allocated out of the namespace introduced for
   Cryptographically Generated Addresses (CGA) Type Tags [RFC3972].

### 3.1. HHIT Prefix for RID Purposes

   The IPv6 HHIT prefix MUST be distinct from that used in the flat-
   space HIT as allocated in [RFC7343].  Without this distinct prefix,
   the first 4 bits of the RAA would be interpreted as the HIT Suite ID
   per HIPv2 [RFC7401].

   Initially, the IPv6 prefix listed in Table 1 is assigned for DET use.
   It has been registered in the "IANA IPv6 Special-Purpose Address
   Registry" [RFC6890].

                    +==========+======+==============+
                    | HHIT Use | Bits | Value        |
                    +==========+======+==============+
                    | DET      | 28   | 2001:30::/28 |
                    +----------+------+--------------+

                     Table 1: Initial DET IPv6 Prefix

   Other prefixes may be added in the future either for DET use or other
   applications of HHITs.  For a prefix to be added to the registry in Section 8.2, its usage and HID allocation process have to be publicly
   available.

### 3.2. HHIT Suite IDs

   The HHIT Suite IDs specify the HI and hash algorithms.  These are a
   superset of the 4-bit and 8-bit HIT Suite IDs as defined in Section 5.2.10 of [RFC7401].

   The HHIT values 1 - 15 map to the basic 4-bit HIT Suite IDs.  HHIT
   values 17 - 31 map to the extended 8-bit HIT Suite IDs.  HHIT values
   unique to HHIT will start with value 32.

   As HHIT introduces a new Suite ID, EdDSA/cSHAKE128, and because this
   is of value to HIPv2, it will be allocated out of the 4-bit HIT space
   and result in an update to HIT Suite IDs.  Future HHIT Suite IDs may
   be allocated similarly, or they may come out of the additional space
   made available by going to 8 bits.

   The following HHIT Suite IDs are defined:

                     +=================+=============+
                     | HHIT Suite      | Value       |
                     +=================+=============+
                     | RESERVED        | 0           |
                     +-----------------+-------------+
                     | RSA,DSA/SHA-256 | 1 [RFC7401] |
                     +-----------------+-------------+
                     | ECDSA/SHA-384   | 2 [RFC7401] |
                     +-----------------+-------------+
                     | ECDSA_LOW/SHA-1 | 3 [RFC7401] |
                     +-----------------+-------------+
                     | EdDSA/cSHAKE128 | 5           |
                     +-----------------+-------------+

                      Table 2: Initial HHIT Suite IDs

#### 3.2.1. HDA Custom HIT Suite IDs

   Support for 8-bit HHIT Suite IDs allows for HDA custom HIT Suite IDs
   (see Table 3).

                       +===================+=======+
                       | HHIT Suite        | Value |
                       +===================+=======+
                       | HDA Private Use 1 | 254   |
                       +-------------------+-------+
                       | HDA Private Use 2 | 255   |
                       +-------------------+-------+

                          Table 3: HDA Custom HIT
                                 Suite IDs

   These custom HIT Suite IDs, for example, may be used for large-scale
   experimentation with post-quantum computing hashes or similar domain-
   specific needs.  Note that currently there is no support for domain-
   specific HI algorithms.

   They should not be used to create a "de facto standardization". Section 8.2 states that additional Suite IDs can be made through IETF
   Review.

### 3.3. The Hierarchy ID (HID)

   The HID provides the structure to organize HITs into administrative
   domains.  HIDs are further divided into two fields:

   *  14-bit Registered Assigning Authority (RAA)

   *  14-bit HHIT Domain Authority (HDA)

   The rationale for splitting the HID into two 14-bit domains is
   described in Appendix B .

   The two levels of hierarchy allow for Civil Aviation Authorities
   (CAAs) to have it least one RAA for their National Air Space (NAS).
   Within its RAAs, the CAAs can delegate HDAs as needed.  There may be
   other RAAs allowed to operate within a given NAS; this is a policy
   decision of each CAA.

#### 3.3.1. The Registered Assigning Authority (RAA)

   An RAA is a business or organization that manages a registry of HDAs.
   For example, the Federal Aviation Authority (FAA) or Japan Civil
   Aviation Bureau (JCAB) could be RAAs.

   The RAA is a 14-bit field (16,384 RAAs).  Management of this space is
   further described in [DRIP-REG ].  An RAA MUST provide a set of
   services to allocate HDAs to organizations.  It SHOULD have a public
   policy on what is necessary to obtain an HDA.  The RAA need not
   maintain any HIP-related services.  At minimum, it MUST maintain a
   DNS zone for the HDA zone delegation for discovering HIP RVS servers
   [RFC8004] for the HID.  Zone delegation is covered in [DRIP-REG ].

   As DETs under administrative control may be used in many different
   domains (e.g., commercial, recreation, military), RAAs should be
   allocated in blocks (e.g., 16-19) with consideration of the likely
   size of a particular usage.  Alternatively, different prefixes can be
   used to separate different domains of use of HHITs.

   The RAA DNS zone within the UAS DNS tree may be a PTR for its RAA.
   It may be a zone in a HHIT-specific DNS zone.  Assume that the RAA is
   decimal 100.  The PTR record could be constructed as follows (where
   20010030 is the DET prefix):

   100.20010030.hhit.arpa.   IN PTR      raa.example.com.

   Note that if the zone 20010030.hhit.arpa is ultimately used, a
   registrar will need to manage this for all HHIT applications.  Thus,
   further thought will be needed in the actual DNS zone tree and
   registration process [DRIP-REG ].

#### 3.3.2. The HHIT Domain Authority (HDA)

   An HDA may be an Internet Service Provider (ISP), UAS Service
   Supplier (USS), or any third party that takes on the business to
   provide UAS services management, HIP RVSs or other needed services
   such as those required for HHIT and/or HIP-enabled devices.

   The HDA is a 14-bit field (16,384 HDAs per RAA) assigned by an RAA
   and is further described in [DRIP-REG ].  An HDA must maintain public
   and private UAS registration information and should maintain a set of
   RVS servers for UAS clients that may use HIP.  How this is done and
   scales to the potentially millions of customers are outside the scope
   of this document; they are covered in [DRIP-REG ].  This service
   should be discoverable through the DNS zone maintained by the HDA's
   RAA.

   An RAA may assign a block of values to an individual organization.
   This is completely up to the individual RAA's published policy for
   delegation.  Such a policy is out of scope for this document.

### 3.4. Edwards-Curve Digital Signature Algorithm for HHITs

   The Edwards-Curve Digital Signature Algorithm (EdDSA) [RFC8032] is
   specified here for use as HIs per HIPv2 [RFC7401].

   The intent in this document is to add EdDSA as a HI algorithm for
   DETs, but doing so impacts the HIP parameters used in a HIP exchange.
   Sections 3.4.1 through 3.4.2 describe the required updates to HIP
   parameters.  Other than the HIP DNS RR (Resource Record) [RFC8005],
   these should not be needed in a DRIP implementation that does not use
   HIP.

   See Section 3.2 for use of the HIT Suite in the context of DRIP.

#### 3.4.1. HOST_ID

   The HOST_ID parameter specifies the public key algorithm, and for
   elliptic curves, a name.  The HOST_ID parameter is defined in Section 5.2.9 of [RFC7401].  Table 4 adds a new HI Algorithm.

                 +===================+=======+===========+
                 | Algorithm profile | Value | Reference |
                 +===================+=======+===========+
                 | EdDSA             | 13    | [RFC8032] |
                 +-------------------+-------+-----------+

                         Table 4: New EdDSA Host ID

##### 3.4.1.1. HIP Parameter support for EdDSA

   The addition of EdDSA as a HI algorithm requires a subfield in the
   HIP HOST_ID parameter (Section 5.2.9 of [RFC7401]) as was done for
   ECDSA when used in a HIP exchange.

   For HIP hosts that implement EdDSA as the algorithm, the following
   EdDSA curves are represented by the fields in Figure 2.

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |         EdDSA Curve           |             NULL              |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                         Public Key                            |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                       Figure 2: EdDSA Curves Fields

   EdDSA Curve:  Curve label

   Public Key:  Represented in Octet-string format [RFC8032]

   For hosts that implement EdDSA as a HIP algorithm, the following
   EdDSA curves are defined.  Recommended curves are tagged accordingly:

         +===========+==============+===========================+
         | Algorithm | Curve        | Values                    |
         +===========+==============+===========================+
         | EdDSA     | RESERVED     | 0                         |
         +-----------+--------------+---------------------------+
         | EdDSA     | EdDSA25519   | 1 [RFC8032] (RECOMMENDED) |
         +-----------+--------------+---------------------------+
         | EdDSA     | EdDSA25519ph | 2 [RFC8032]               |
         +-----------+--------------+---------------------------+
         | EdDSA     | EdDSA448     | 3 [RFC8032] (RECOMMENDED) |
         +-----------+--------------+---------------------------+
         | EdDSA     | EdDSA448ph   | 4 [RFC8032]               |
         +-----------+--------------+---------------------------+

                          Table 5: EdDSA Curves

##### 3.4.1.2. HIP DNS RR support for EdDSA

   The HIP DNS RR is defined in [RFC8005].  It uses the values defined
   for the 'Algorithm Type' of the IPSECKEY RR [RFC4025] for its PK
   Algorithm field.

   The 'Algorithm Type' value and EdDSA HI encoding are assigned per
   [RFC9373].

#### 3.4.2. HIT_SUITE_LIST

   The HIT_SUITE_LIST parameter contains a list of the HIT suite IDs
   that the HIP Responder supports.  The HIT_SUITE_LIST allows the HIP
   Initiator to determine which source HIT Suite IDs are supported by
   the Responder.  The HIT_SUITE_LIST parameter is defined in Section 5.2.10 of [RFC7401].

   The following HIT Suite ID is defined:

                        +=================+=======+
                        | HIT Suite       | Value |
                        +=================+=======+
                        | EdDSA/cSHAKE128 | 5     |
                        +-----------------+-------+

                           Table 6: HIT Suite ID

   Table 7 provides more detail on the above HIT Suite combination.

   The output of cSHAKE128 is variable per the needs of a specific
   ORCHID construction.  It is at most 96 bits long and is directly used
   in the ORCHID (without truncation).

     +=======+===========+=========+===========+====================+
     | Index | Hash      | HMAC    | Signature | Description        |
     |       | function  |         | algorithm |                    |
     |       |           |         | family    |                    |
     +=======+===========+=========+===========+====================+
     |     5 | cSHAKE128 | KMAC128 | EdDSA     | EdDSA HI hashed    |
     |       |           |         |           | with cSHAKE128,    |
     |       |           |         |           | output is variable |
     +-------+-----------+---------+-----------+--------------------+

                           Table 7: HIT Suites

### 3.5. ORCHIDs for HHITs

   This section improves on ORCHIDv2 [RFC7343] with three enhancements:

   *  the inclusion of an optional "Info" field between the Prefix and
      ORCHID Generation Algorithm (OGA) ID.

   *  an increase in flexibility on the length of each component in the
      ORCHID construction, provided the resulting ORCHID is 128 bits.

   *  the use of cSHAKE [NIST.SP.800-185] for the hashing function.

   The cSHAKE XOF hash function based on Keccak [Keccak ] is a variable
   output length hash function.  As such, it does not use the truncation
   operation that other hashes need.  The invocation of cSHAKE specifies
   the desired number of bits in the hash output.  Further, cSHAKE has a
   parameter 'S' as a customization bit string.  This parameter will be
   used for including the ORCHID Context Identifier in a standard
   fashion.

   This ORCHID construction includes the fields in the ORCHID in the
   hash to protect them against substitution attacks.  It also provides
   for inclusion of additional information (in particular, the
   hierarchical bits of the HHIT) in the ORCHID generation.  This should
   be viewed as an update to ORCHIDv2 [RFC7343], as it can produce
   ORCHIDv2 output.

   The following subsections define the new general ORCHID construct
   with the specific application for HHITs.  Thus items like the hash
   size are only discussed in terms of how they impact the HHIT's 64-bit
   hash.  Other hash sizes should be discussed for other specific uses
   of this new ORCHID construct.

#### 3.5.1. Adding Additional Information to the ORCHID

   ORCHIDv2 [RFC7343] is defined as consisting of three components:

   ORCHID     :=  Prefix | OGA ID | Encode_96( Hash )

   where:

   Prefix
      A constant 28-bit-long bitstring value (IPv6 prefix)

   OGA ID
      A 4-bit-long identifier for the Hash_function in use within the
      specific usage context.  When used for HIT generation, this is the
      HIT Suite ID.

   Encode_96( )
      An extraction function in which output is obtained by extracting
      the middle 96-bit-long bitstring from the argument bitstring.

   The new ORCHID function is as follows:

   ORCHID     :=  Prefix (p) | Info (n) | OGA ID (o) | Hash (m)

   where:

   Prefix (p)
      An IPv6 prefix of length p (max 28 bits long).

   Info (n)
      n bits of information that define a use of the ORCHID.  'n' can be
      zero, which means no additional information.

   OGA ID (o)
      A 4- or 8-bit long identifier for the Hash_function in use within
      the specific usage context.  When used for HIT generation, this is
      the HIT Suite ID [IANA-HIP ].  When used for HHIT generation, this
      is the HHIT Suite ID [HHSI ].

   Hash (m)
      An extraction function in which output is 'm' bits.

   Sizeof(p + n + o + m) = 128 bits

   The ORCHID length MUST be 128 bits.  For HHITs with a 28-bit IPv6
   prefix, there are 100 bits remaining to be divided in any manner
   between the additional information ("Info"), OGA ID, and the hash
   output.  Consideration must be given to the size of the hash portion,
   taking into account risks like pre-image attacks. 64 bits, as used
   here for HHITs, may be as small as is acceptable.  The size of 'n',
   for the HID, is then determined as what is left; in the case of the
   8-bit OGA used for HHIT, this is 28 bits.

#### 3.5.2. ORCHID Encoding

   This update adds a different encoding process to that currently used
   in ORCHIDv2.  The input to the hash function explicitly includes all
   the header content plus the Context ID.  The header content consists
   of the Prefix, the Additional Information ("Info"), and the OGA ID
   (HIT Suite ID).  Secondly, the length of the resulting hash is set by
   the sum of the length of the ORCHID header fields.  For example, a
   28-bit prefix with 28 bits for the HID and 8 bits for the OGA ID
   leaves 64 bits for the hash length.

   To achieve the variable length output in a consistent manner, the
   cSHAKE hash is used.  For this purpose, cSHAKE128 is appropriate.
   The cSHAKE function call is:

       cSHAKE128(Input, L, "", Context ID)

       Input      :=  Prefix | Additional Information | OGA ID | HOST_ID
       L          :=  Length in bits of the hash portion of ORCHID

   For full Suite ID support (those that use fixed length hashes like
   SHA256), the following hashing can be used (Note: this does not
   produce output identical to ORCHIDv2 for a /28 prefix and Additional
   Information of zero length):

       Hash[L](Context ID | Input)

       Input      :=  Prefix | Additional Information | OGA ID | HOST_ID
       L          :=  Length in bits of the hash portion of ORCHID

       Hash[L]    :=  An extraction function in which output is obtained
                      by extracting the middle L-bit-long bitstring
                      from the argument bitstring.

   The middle L-bits are those bits from the source number where either
   there is an equal number of bits before and after these bits, or
   there is one more bit prior (when the difference between hash size
   and L is odd).

   HHITs use the Context ID defined in Section 3.

##### 3.5.2.1. Encoding ORCHIDs for HIPv2

   This section discusses how to provide backwards compatibility for
   ORCHIDv2 [RFC7343] as used in HIPv2 [RFC7401].

   For HIPv2, the Prefix is 2001:20::/28 (Section 6 of [RFC7343]).
   'Info' is zero-length (i.e., not included), and OGA ID is 4-bit.
   Thus, the HI Hash is 96 bits in length.  Further, the Prefix and OGA
   ID are not included in the hash calculation.  Thus, the following
   ORCHID calculations for fixed output length hashes are used:

       Hash[L](Context ID | Input)

       Input      :=  HOST_ID
       L          :=  96
       Context ID :=  0xF0EF F02F BFF4 3D0F E793 0C3C 6E61 74EA

       Hash[L]    :=  An extraction function in which output is obtained
                      by extracting the middle L-bit-long bitstring
                      from the argument bitstring.

   For variable output length hashes use:

       Hash[L](Context ID | Input)

       Input      :=  HOST_ID
       L          :=  96
       Context ID :=  0xF0EF F02F BFF4 3D0F E793 0C3C 6E61 74EA

       Hash[L]    :=  The L-bit output from the hash function

   Then, the ORCHID is constructed as follows:

       Prefix | OGA ID | Hash Output

#### 3.5.3. ORCHID Decoding

   With this update, the decoding of an ORCHID is determined by the
   Prefix and OGA ID.  ORCHIDv2 [RFC7343] decoding is selected when the
   Prefix is: 2001:20::/28.

   For HHITs, the decoding is determined by the presence of the HHIT
   Prefix as specified in Section 8.2.

#### 3.5.4. Decoding ORCHIDs for HIPv2

   This section is included to provide backwards compatibility for
   ORCHIDv2 [RFC7343] as used for HIPv2 [RFC7401].

   HITs are identified by a Prefix of 2001:20::/28.  The next 4 bits are
   the OGA ID.  The remaining 96 bits are the HI Hash.

## 4. HHITs as DRIP Entity Tags

   HHITs for UAS ID (called, DETs) use the new EdDSA/SHAKE128 HIT suite
   defined in Section 3.4 (GEN-2 in [RFC9153]).  This hierarchy,
   cryptographically bound within the HHIT, provides the information for
   finding the UA's HHIT registry (ID-3 in [RFC9153]).

   The ASTM Standard Specification for Remote ID and Tracking
   [F3411-22a ] adds support for DETs.  This is only available via the
   new UAS ID type 4, "Specific Session ID (SSI)".

   This new SSI uses the first byte of the 20-byte UAS ID for the SSI
   Type, thus restricting the UAS ID of this type to a maximum of 19
   bytes.  The SSI Types initially assigned are:

   SSI 1:  IETF - DRIP Drone Remote ID Protocol (DRIP) entity ID.

   SSI 2:  3GPP - IEEE 1609.2-2016 HashedID8

### 4.1. Nontransferablity of DETs

   A HI and its DET SHOULD NOT be transferable between UAs or even
   between replacement electronics (e.g., replacement of damaged
   controller CPU) for a UA.  The private key for the HI SHOULD be held
   in a cryptographically secure component.

### 4.2. Encoding HHITs in CTA 2063-A Serial Numbers

   In some cases, it is advantageous to encode HHITs as a CTA 2063-A
   Serial Number [CTA2063A ].  For example, the FAA Remote ID Rules
   [FAA_RID ] state that a Remote ID Module (i.e., not integrated with UA
   controller) must only use "the serial number of the unmanned
   aircraft"; CTA 2063-A meets this requirement.

   Encoding a HHIT within the CTA 2063-A format is not simple.  The CTA
   2063-A format is defined as follows:

   Serial Number   :=  MFR Code | Length Code | MFR SN

   where:

   MFR Code
      4 character code assigned by ICAO (International Civil Aviation
      Organization, a UN Agency).

   Length Code
      1 character Hex encoding of MFR SN length (1-F).

   MFR SN
      US-ASCII alphanumeric code (0-9, A-Z except O and I).  Maximum
      length of 15 characters.

   There is no place for the HID; there will need to be a mapping
   service from Manufacturer Code to HID.  The HHIT Suite ID and ORCHID
   hash will take the full 15 characters (as described below) of the MFR
   SN field.

   A character in a CTA 2063-A Serial Number "shall include any
   combination of digits and uppercase letters, except the letters O and
   I, but may include all digits".  This would allow for a Base34
   encoding of the binary HHIT Suite ID and ORCHID hash in 15
   characters.  Although, programmatically, such a conversion is not
   hard, other technologies (e.g., credit card payment systems) that
   have used such odd base encoding have had performance challenges.
   Thus, here a Base32 encoding will be used by also excluding the
   letters Z and S (because they are too similar to the digits 2 and 5,
   respectively).  See Appendix C  for the encoding scheme.

   The low-order 72 bits (HHIT Suite ID | ORCHID hash) of the HHIT SHALL
   be left-padded with 3 bits of zeros.  This 75-bit number will be
   encoded into the 15-character MFR SN field using the digit/letters as
   described above.  The manufacturer MUST use a Length Code of F (15).

   Note: The manufacturer MAY use the same Manufacturer Code with a
   Length Code of 1 - E (1 - 14) for other types of serial numbers.

   Using the sample DET from Section 5 that is for HDA=20 under RAA=10
   and having the ICAO CTA MFR Code of 8653, the 20-character CTA 2063-A
   Serial Number would be:

       8653F02T7B8RA85D19LX

   A mapping service (e.g., DNS) MUST provide a trusted (e.g., via
   DNSSEC [RFC4034]) conversion of the 4-character Manufacturer Code to
   high-order 58 bits (Prefix | HID) of the HHIT.  That is, given a
   Manufacturer Code, a returned Prefix|HID value is reliable.
   Definition of this mapping service is out of scope of this document.

   It should be noted that this encoding would only be used in the Basic
   ID Message (Section 2.2 of [RFC9153]).  The DET is used in the
   Authentication Messages (i.e., the messages that provide framing for
   authentication data only).

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

### 8.1. New Well-Known IPv6 Prefix for DETs

   Since the DET format is not compatible with [RFC7343], IANA has
   allocated the following prefix per this template for the "IANA IPv6
   Special-Purpose Address Registry" [IPv6-SPECIAL ].

   Address Block:
      2001:30::/28

   Name:
      Drone Remote ID Protocol Entity Tags (DETs) Prefix

   Reference
      This document

   Allocation Date:
      2022-12

   Termination Date:
      N/A

   Source:
      True

   Destination:
      True

   Forwardable:
      True

   Globally Reachable:
      True

   Reserved-by-Protocol:
      False

#### 8.2.1. HHIT Prefixes

   Initially, for DET use, one 28-bit prefix has been assigned out of
   the IANA IPv6 Special Purpose Address Block, namely 2001::/23, as per
   [RFC6890].  Future additions to this subregistry are to be made
   through Expert Review (Section 4.5 of [RFC8126]).  Entries with
   network-specific prefixes may be present in the registry.

              +==========+======+==============+===========+
              | HHIT Use | Bits | Value        | Reference |
              +==========+======+==============+===========+
              | DET      | 28   | 2001:30::/28 | RFC 9374  |
              +----------+------+--------------+-----------+

                   Table 8: Registered DET IPv6 Prefix

   Criteria that should be applied by the designated experts includes
   determining whether the proposed registration duplicates existing
   functionality and whether the registration description is clear and
   fits the purpose of this registry.

   Registration requests MUST be sent to drip-reg-review@ietf.org and be
   evaluated within a three-week review period on the advice of one or
   more designated experts.  Within that review period, the designated
   experts will either approve or deny the registration request, and
   communicate their decision to the review list and IANA.  Denials
   should include an explanation and, if applicable, suggestions to
   successfully register the prefix.

   Registration requests that are undetermined for a period longer than
   28 days can be brought to the IESG's attention for resolution.

# Referenced Sections from RFC 3596: DNS Extensions to Support IP Version 6

The following sections were referenced. Remaining sections are not included.

### 2.5. IP6.ARPA Domain

   A special domain is defined to look up a record given an IPv6
   address.  The intent of this domain is to provide a way of mapping an
   IPv6 address to a host name, although it may be used for other
   purposes as well.  The domain is rooted at IP6.ARPA.

   An IPv6 address is represented as a name in the IP6.ARPA domain by a
   sequence of nibbles separated by dots with the suffix ".IP6.ARPA".
   The sequence of nibbles is encoded in reverse order, i.e., the
   low-order nibble is encoded first, followed by the next low-order
   nibble and so on.  Each nibble is represented by a hexadecimal digit.
   For example, the reverse lookup domain name corresponding to the
   address

       4321:0:1:2:3:4:567:89ab

   would be

   b.a.9.8.7.6.5.0.4.0.0.0.3.0.0.0.2.0.0.0.1.0.0.0.0.0.0.0.1.2.3.4.IP6.
                                                                  ARPA. 





# Referenced Sections from RFC 88: NETRJS: A third level protocol for Remote Job Entry

The following sections were referenced. Remaining sections are not included.

## G. Field Definitions

   Name*           Meaning                 Length (bits)
   _____           _______                 _____________

   BIT             1-bit field                  1

   BIT2            2-bit field                  2

   BIT4            4-bit field                  4

   BLKSEQ          Block sequence number        5

   BYTE            8-bit field aligned on 8-bit 8
                   boundary

   CHECK           Block check number          32

   DEVNO           Device number of a given     3
                   type

   DEVTYPE         Device type                  4

   DUPCOUNT        Number of replications of    5
                   duplicated character in
                   compressed text.

   ERRORCONTROL    Block transmission error     2
                   control.

   LENGTH          Length in bytes of the       6
                   following string of text.

   TEXTBYTE        An 8-bit byte of text        8

   *Note:  All non-terminal fields whose names end in
           "...BYTE" represent bytes in both length and
           alignment. 
   H.  NOTES AND REFERENCES

   1. Martin, V.A. and Springer, T.W., "Implementation of A Remote Job
      Service", Technical Report TR2, Campus Computing Network, UCLA,
      Los Angeles, (undated).

   2. The RJS operator commands and messages are described in detail in
      Reference 1.

   3. We use the phrase "starting a session" rather than "logging on"
      because RJS has its own log on procedure, which is, we suppose, a
      fourth-level protocol.

   4. Note that NETRJS uses closing of connections as end-of-file
      signals.



           REMOTE SITE             CENTRAL SITE (CCN)
      +---------------------+    +--------------------+
      |                 a   |    |                    |
      | Console Input  o----------->o f               |
      |                 b   |    |                    |
      | Console Output o<-----------o g               |
      |                 c   |    |                    |
      | Card Reader    o------------o h               |
      |                 d   |    |                    |
      | Printer        o<-----------o i               |
      |                 e   |    |                    |
      | Card Punch     o<-----------o j               |
      |                     |    |                    |
      +---------------------+    +--------------------+

                           FIGURE 1
                ARPA Network Connections (Channels)
              For a Standard Remote Site Under NETRJS

      R.T. Braden/rb.
      S.M. Wolfe


           [This RFC was put into machine readable form for entry]
            [into the online RFC archives by Lorrie Shiota, 10/01]









# Summary of reference from ISO, "Codes for the representation of names of countries and their subdivisions - Part 1: Country code", ISO 3166-1:2020, August 2020, <https://www.iso.org/standard/72482.html >. (ISO3166-1)

ISO 3166-1:2020, titled "Codes for the representation of names of countries and their subdivisions – Part 1: Country code," establishes standardized codes for country names. These codes are designed for use in applications requiring the expression of current country names in a coded format. ([iso.org](https://www.iso.org/standard/72482.html?utm_source=openai))

The standard defines three types of country codes:

- **Alpha-2 code**: A two-letter code recommended for general use.
- **Alpha-3 code**: A three-letter code that is more closely related to the country's name.
- **Numeric-3 code**: A three-digit numeric code useful for applications requiring language independence.

For example, the United States is represented as "US" (Alpha-2), "USA" (Alpha-3), and "840" (Numeric-3). ([blog.ansi.org](https://blog.ansi.org/2020/10/iso-country-codes-iso-3166-standards/?utm_source=openai))

The codes are intended for use in any application requiring the expression of current country names in coded form. ([iso.org](https://www.iso.org/standard/72482.html?utm_source=openai))

ISO 3166-1:2020 is maintained by the ISO 3166 Maintenance Agency (ISO 3166/MA), which regularly updates the standard to reflect changes in country names and subdivisions. ([iso.org](https://www.iso.org/iso-3166-country-codes.html?utm_source=openai))

The standard is available for purchase through the ISO website. ([iso.org](https://www.iso.org/standard/72482.html?utm_source=openai)) 

