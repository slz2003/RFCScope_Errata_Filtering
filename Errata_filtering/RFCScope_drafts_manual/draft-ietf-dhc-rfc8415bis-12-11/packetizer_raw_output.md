# Errata Reports

Total reports: 3

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-11-1

**Label:** Ambiguous DUID Stability vs RFC7844 Anonymity: Temporal Scope of 'Exactly One DUID'

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that each DHCP client and server has exactly one stable DUID while also permitting changes per RFC7844, but it fails to explicitly qualify that the stability requirement applies only outside anonymity profiles.

**Justification:**

- The terminology and Section 11 state that each participant has exactly one DUID and that it SHOULD NOT change, yet immediately allow for changes as per RFC7844 without scoping this exception.
- This ambiguity may lead implementers to misinterpret whether a DUID's stability applies in anonymity mode or only in standard operation.

**Evidence Snippets:**

- **E1:**

  Terminology: “DUID … A DHCP Unique Identifier for a DHCP participant. Each DHCP client and server has exactly one DUID. See Section 11 for details of the ways in which a DUID may be constructed.”

- **E2:**

  Section 11: “Each DHCP client and server has a DUID. … The DUID is designed to be unique across all DHCP clients and servers, and stable for any specific client or server. That is, the DUID used by a client or server SHOULD NOT change over time if at all possible…” followed immediately by “The client may change its DUID as specified in [RFC7844].”

- **E3:**

  The terminology section defines a DUID as “a DHCP Unique Identifier for a DHCP participant. Each DHCP client and server has exactly one DUID” and Section 11 says the DUID “is designed to be unique … and stable for any specific client or server” with an exception noted for anonymity profiles per RFC7844.

**Evidence Summary:**

- (E1) Presents the claim of exactly one DUID per client/server.
- (E2) Shows the juxtaposition of stability requirements with the allowance to change via RFC7844.
- (E3) Summarizes the resultant ambiguity regarding the scope of stability in the presence of anonymity profiles.

**Fix Direction:**

Clarify in the text that the stability and uniqueness requirements apply only in non-anonymity operation, and that 'exactly one DUID' is intended to mean one active DUID at any given time.

**Severity:** Low
  *Basis:* The issue is primarily an editorial ambiguity that may confuse implementers but is unlikely to lead to interoperability problems.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-1
- CrossRFC: Issue-1
- Deontic: Issue-1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-11-2

**Label:** Underspecified DUID-Type Generation/Storage Rules under Anonymity Profiles

**Bug Type:** Underspecification

**Explanation:**

The per-type DUID generation and storage rules are written as unconditional requirements, without specifying that they may not apply when a client is operating under an RFC7844 anonymity profile.

**Justification:**

- Sections such as DUID-LLT (Section 11.2) mandate that the identifier be stored in stable or nonvolatile storage, conflicting with the dynamic DUID generation required in anonymity scenarios per RFC7844.
- The text does not clarify that these MUST requirements should be interpreted as applying only in non-anonymity operations.

**Evidence Snippets:**

- **E4:**

  DUID-LLT (Section 11.2): “Clients and servers using this type of DUID MUST store the DUID-LLT in stable storage and MUST continue to use this DUID-LLT even if the network interface used to generate the DUID-LLT is removed. Clients and servers that do not have any stable storage MUST NOT use this type of DUID.”

- **E5:**

  RFC 7844 (excerpted): requires clients in anonymity profiles to generate new randomized DUID-LLT values per link-change event, and (when using randomized MACs) to use DUID-LL with the *current* (potentially frequently changing) link-layer address as the DUID content.

**Evidence Summary:**

- (E4) States the unconditional storage and usage requirements for DUID-LLT.
- (E5) Contrasts these requirements by requiring dynamic DUID generation under RFC7844 anonymity profiles.

**Fix Direction:**

Introduce conditional language to indicate that the storage and persistence requirements for each DUID type apply only during non-anonymity operation, or when the client is not following RFC7844.

**Severity:** Low
  *Basis:* This is an editorial clarity issue that could create confusion for implementers wishing to support RFC7844-based anonymity without violating the normative storage rules.

**Confidence:** High

**Experts mentioning this issue:**

- Scope: Issue-2

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-11-3

**Label:** Inconsistent DUID-UUID Prose Regarding Total Length versus Generic DUID Structure

**Bug Type:** Inconsistency

**Explanation:**

The DUID-UUID section states that it consists of 16 octets for the UUID, which appears to conflict with the generic DUID structure that specifies a 2-octet type code preceding a variable-length identifier.

**Justification:**

- The generic DUID definition in Section 11.1 defines a DUID as a 2-octet type code followed by an identifier of 1 to 128 octets.
- In contrast, Section 11.5 on DUID-UUID states it 'consists of 16 octets containing a 128-bit UUID', risking misinterpretation about whether the type field is included.

**Evidence Snippets:**

- **E6:**

  A DUID consists of a 2-octet type code represented in network byte order, followed by a variable number of octets that make up the actual identifier. The length of the DUID (not including the type code) is at least 1 octet and at most 128 octets.

- **E7:**

  This type of DUID consists of 16 octets containing a 128-bit UUID. [RFC6355] details when to use this type and how to pick an appropriate source of the UUID.  Figure 8 then shows: | DUID-Type (4) | UUID (128 bits) |

**Evidence Summary:**

- (E6) Defines the generic DUID structure with a 2-octet type code and a variable-length identifier.
- (E7) Describes DUID-UUID in a way that suggests a total of 16 octets, potentially excluding the type code.

**Fix Direction:**

Revise the DUID-UUID section to clarify that the 16 octets refer only to the identifier field, with an additional 2 octets for the DUID type code as defined in the generic DUID structure.

**Severity:** Low
  *Basis:* This inconsistency is editorial in nature and may temporarily confuse readers, but it is unlikely to affect actual DUID encoding in implementations.

**Confidence:** High

**Experts mentioning this issue:**

- Structural: Issue-1

---
