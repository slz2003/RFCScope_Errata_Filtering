# Errata Reports

Total reports: 2

---

## Report 1: 9859-6-1

**Label:** Ambiguous DSYNC Scheme Namespace: Global vs. Per-RRtype Interpretation

**Bug Type:** Both

**Explanation:**

The document presents conflicting descriptions regarding DSYNC scheme values by stating they are independent of the RRtype while also requiring an RRtype field in the registry, leading to ambiguity over whether scheme numbers are globally unique or defined per (RRtype, Scheme) pair.

**Justification:**

- Sections 2.1 and 2.3 describe the Scheme as a global 8-bit unsigned integer that is independent of RRtype, implying a single namespace.
- Section 6.2, however, defines a registry template that requires an RRtype field and includes separate entries (e.g., for CDS and CSYNC with scheme 1), which suggests a per-(RRtype, Scheme) model.
- This inconsistency can lead to registry conflicts and divergent implementations.

**Evidence Snippets:**

- **E1:**

  Scheme:  The mode used for contacting the desired notification address.  This is an 8‑bit unsigned integer.  Records with value 0 (null scheme) are ignored by consumers.  Value 1 is described in this document, and values 128–255 are Reserved for Private Use.  Other values are currently unassigned.  Future assignments are maintained in the registry created in Section 6.2.

- **E2:**

  Schemes are independent of the RRtype.  They merely specify a method of contacting the target (whereas the RRtype is part of the notification payload).

- **E3:**

  IANA has created the following new registry in the ‘Domain Name System (DNS) Parameters’ registry group:

          Name:  DSYNC: Location of Synchronization Endpoints  
          Registration Procedure:  Expert Review  
          Reference:  RFC 9859

          The initial contents for the registry are as follows:

          +========+===================+==========================+===========+  
          | RRtype | Scheme (Mnemonic) | Purpose                  | Reference |  
          +========+===================+==========================+===========+  
          |        | 0                 | Null scheme (no-op)      | RFC 9859  |  
          +--------+-------------------+--------------------------+-----------+  
          | CDS    | 1 (NOTIFY)        | Delegation management    | RFC 9859  |  
          +--------+-------------------+--------------------------+-----------+  
          | CSYNC  | 1 (NOTIFY)        | Delegation management    | RFC 9859  |  
          +--------+-------------------+--------------------------+-----------+  
          |        | 2-127             | Unassigned               |           |  
          +--------+-------------------+--------------------------+-----------+  
          |        | 128-255           | Reserved for Private     | RFC 9859  |  
          |        |                   | Use                      |           |  
          +--------+-------------------+--------------------------+-----------+

- **E4:**

  Requests to register additional entries MUST include the following fields:

          RRtype:  An RRtype that is defined for use  
          Scheme:  The mode used for contacting the desired notification address  
          Mnemonic:  The scheme's shorthand string used in presentation format  
          Purpose:  Use case description  
          Reference:  Location of specification or registration source

**Evidence Summary:**

- (E1) DSYNC RDATA definition describing an 8-bit scheme value with global maintenance via Section 6.2.
- (E2) Statement that schemes are independent of the RRtype, implying a global namespace.
- (E3) Registry table structure in Section 6.2 that includes an RRtype column and separate entries for the same scheme number.
- (E4) Registration template that mandates including the RRtype field alongside the scheme value.

**Fix Direction:**

Clarify the namespace model in Sections 2.3 and 6.2; if schemes are intended to be global, remove the need for an RRtype in the registry entries, or alternatively, adjust the protocol text to state that scheme semantics are defined per (RRtype, Scheme) pair.

**Severity:** High
  *Basis:* Ambiguity between the protocol text and registry structure can lead to inconsistent implementations and conflicting registry entries, significantly affecting interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1
- Structural Expert: Issue-1
- CrossRFC Expert: Issue-1

---

## Report 2: 9859-6-2

**Label:** Missing change controller field in DSYNC registry registration template

**Bug Type:** None

**Explanation:**

The DSYNC registry registration template omits an explicit change controller field, which is recommended by RFC 8126 §4.5 to guide administrative updates, leading to potential process-level ambiguity.

**Justification:**

- Section 6.2 defines required fields for DSYNC registry entries (RRtype, Scheme, Mnemonic, Purpose, Reference) but does not include a change controller field.
- RFC 8126 §4.5 recommends that Expert Review registries should include a field for change controller to clarify who is authorized to approve changes.

**Evidence Snippets:**

- **E1:**

  Name:  DSYNC: Location of Synchronization Endpoints  
        Registration Procedure:  Expert Review

- **E2:**

  Requests to register additional entries MUST include the following fields: … RRtype … Scheme … Mnemonic … Purpose … Reference

- **E3:**

  RFC 8126 §4.5: When creating a new registry with Expert Review as the registration policy, in addition to the contact person field or reference, the registry should contain a field for change controller.

**Evidence Summary:**

- (E1) DSYNC registry defined with Expert Review procedure without a change controller field.
- (E2) Registration template listing required fields but omitting the change controller field.
- (E3) RFC 8126 §4.5 recommendation to include a change controller field in Expert Review registries.

**Fix Direction:**

Optionally add a change controller field to the DSYNC registry registration template or clearly state that the existing Reference field serves that purpose.

**Severity:** Low
  *Basis:* While the omission does not affect protocol interoperability, it may lead to process ambiguities in the long-term administration of the registry.

**Confidence:** High

**Experts mentioning this issue:**

- Deontic Expert: Issue-2
- Structural Expert: Issue-2
- CrossRFC Expert: Issue-2

---
