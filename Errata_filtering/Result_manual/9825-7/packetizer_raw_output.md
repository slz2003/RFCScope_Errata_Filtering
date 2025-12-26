# Errata Reports

Total reports: 6

---

## Report 1: 9825-7-1

**Label:** Mis-scoped LSDB 'when' Expression: Wrong Ancestor Level for rt:type

**Bug Type:** Inconsistency

**Explanation:**

The LSDB augments use a derived-from-or-self() XPath that climbs one level too few, causing the rt:type lookup to occur under an ospf:ospf node rather than the intended rt:control-plane-protocol, which forces the condition to always evaluate to false.

**Justification:**

- The candidate issue explains that in the OSPFv2 link-scope case, the XPath uses 15 '../' steps instead of the 16 needed to reach the rt:control-plane-protocol level (see evidence snippets E1 and E2).
- This mis-scoping makes the when condition always false, preventing instantiation of the intended admin-tag nodes.

**Evidence Snippets:**

- **E1:**

  Example (OSPFv2 link-scope case):

augment "/rt:routing/.../ospf:extended-prefix-opaque/ospf:extended-prefix-tlv" {
  when "derived-from-or-self(../../../../../../../../../.."
     + "/../../../../rt:type, 'ospf:ospfv2')" {
    description "This augmentation is only valid for OSPFv2.";
  }
  ...
  uses prefix-admin-tag-sub-tlv;
}

- **E2:**

  Other OSPFv2 and OSPFv3 LSDB augments use the same pattern with slightly shorter '../' chains, e.g.:

when "derived-from-or-self(../../../../../../../../../.."
   + "/../../rt:type, 'ospf:ospfv2')" { ... }

when "derived-from-or-self(../../../../../../../.."
   + "/../../rt:type, 'ospf:ospfv3')" { ... }

**Evidence Summary:**

- (E1) Shows the augment for OSPFv2 Extended Prefix TLV with the long '../' chain leading to a check on rt:type under an ospf:ospf node.
- (E2) Demonstrates the similar pattern for both ospfv2 and ospfv3 augments that suffer from the same off-by-one error.

**Fix Direction:**

Increase the '../' depth in the when expressions so that the path inside derived-from-or-self() correctly reaches the rt:control-plane-protocol ancestor.


**Severity:** High
  *Basis:* Since the mis-scoped expression always evaluates to false, the intended admin-tag sub-TLV nodes will never be instantiated, rendering parts of the YANG module non‐functional.

**Confidence:** High

---

## Report 2: 9825-7-2

**Label:** Absolute 'when' Condition in OSPFv3 Augments Is Overly Broad

**Bug Type:** Underspecification

**Explanation:**

The OSPFv3 augments use an absolute XPath in their when conditions that checks for any ospf:ospfv3 instance in the datastore instead of constraining the condition to the specific augmented instance.

**Justification:**

- The candidate issue notes that the absolute when condition evaluates to true if any OSPFv3 instance exists, rather than verifying that the current protocol instance is OSPFv3 (see evidence snippets E1 and E2).
- This broad check creates a stylistic inconsistency with other relative, instance-scoped checks and may lead to fragile future behavior if similar subtree shapes are reused.

**Evidence Snippets:**

- **E1:**

  For OSPFv3 E‑Intra-Area-Prefix TLV:

augment "/rt:routing/.../ospfv3-e-lsa:e-intra-area-prefix
         /ospfv3-e-lsa:e-intra-prefix-tlvs
         /ospfv3-e-lsa:intra-prefix-tlv" {
  when "/rt:routing/rt:control-plane-protocols"
     + "/rt:control-plane-protocol/rt:type = 'ospf:ospfv3'" {
    description
      "This augmentation is only valid for OSPFv3.";
  }
  ...
  uses prefix-admin-tag-sub-tlv;
}

- **E2:**

  For OSPFv3 E‑NSSA External-Prefix TLV:

augment "/rt:routing/.../ospfv3-e-lsa:e-nssa
         /ospfv3-e-lsa:e-external-tlvs
         /ospfv3-e-lsa:external-prefix-tlv" {
  when "/rt:routing/rt:control-plane-protocols"
     + "/rt:control-plane-protocol/rt:type = 'ospf:ospfv3'" {
    description
      "This augmentation is only valid for OSPFv3.";
  }
  ...
  uses prefix-admin-tag-sub-tlv;
}

**Evidence Summary:**

- (E1) Presents the absolute when condition for the OSPFv3 E‑Intra-Area-Prefix TLV augmentation, which checks for any ospf:ospfv3 instance in the datastore.
- (E2) Demonstrates a similar absolute when condition used in the OSPFv3 E‑NSSA External-Prefix TLV augmentation.

**Fix Direction:**

Replace the absolute XPath check with a relative, instance-scoped derived-from-or-self() expression that specifically validates the current protocol instance's rt:type.


**Severity:** Medium
  *Basis:* The overly broad when condition does not currently prevent the nodes from being instantiated, but it undermines the intended instance-specific semantics and may lead to future mis-attachments if the subtree is reused.

**Confidence:** High

---

## Report 3: 9825-7-3

**Label:** Ambiguity in Coverage: Missing Administrative Tag Configuration for NSSA Ranges and Redistributed Summaries

**Bug Type:** Underspecification

**Explanation:**

The protocol text mentions that NSSA ranges and redistributed route summaries should support administrative tags, but the YANG model only defines tags for area ranges and local prefixes, leaving it unclear whether the omissions were intentional.

**Justification:**

- The residual uncertainties note highlights that while the protocol specifies additional contexts (NSSA ranges and redistributed summaries), the YANG module does not include corresponding augmentations, raising questions about design intent.

**Evidence Snippets:**

- **E1:**

  ResidualUncertainties:
  - The protocol text (Sections 3–4) mentions NSSA ranges and redistributed route summaries as candidates for configurable tags, but the YANG only models tags on area ranges and local prefixes. It is unclear whether this omission is intentional (left to future modules) or an oversight.

**Evidence Summary:**

- (E1) Indicates that the YANG model omits administrative tag configurations for NSSA ranges and redistributed summaries even though the protocol text suggests they should be supported.

**Fix Direction:**

Clarify in the design whether these tag configurations were intentionally omitted or add the missing augmentations in a future revision.



**Severity:** Low
  *Basis:* This omission does not currently affect operational behavior but creates ambiguity regarding the full intended scope of administrative tag support.

**Confidence:** Medium
