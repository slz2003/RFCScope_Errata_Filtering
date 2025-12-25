# Errata Reports

Total reports: 3

---

## Report 1: 9810-1-1

**Label:** Misattribution of Security Considerations Sections in RFC 9810 Section 1.3

**Bug Type:** Inconsistency

**Explanation:**

RFC 9810’s change log incorrectly claims that Sections 8.1, 8.5, 8.8, and 8.11 are newly added, even though Sections 8.1 and 8.5 were already present in RFC 4210, leading to a misleading historical record.

**Justification:**

- Section 1.3 states that the document 'added Sections 8.1, 8.5, 8.8, and 8.11' while evidence shows that Section 8.1 (and 8.5) appeared in RFC 4210 (E1, E2).
- The terminology analysis further clarifies that only Sections 8.8 and 8.11 are new, whereas 8.1 and 8.5 are updated, not newly introduced.

**Evidence Snippets:**

- **E1:**

  Section 1.3 of RFC 9810 states that “this document” added Sections 8.1, 8.5, 8.8, and 8.11 to the security considerations. However, RFC 4210 already contains a Section 8.1 (“On the Necessity of POP”), and RFC 9480’s updates add new sections 8.4–8.7 without removing or renumbering 8.1 in that document. RFC 9810’s Section 8.1 covers the same conceptual ground and clearly derives from the long‑standing POP security discussion in RFC 4210, so it is misleading to present 8.1 as newly “added” by RFC 9810. An implementer tracing the provenance of Section 8.1 may incorrectly conclude that the POP discussion did not exist prior to RFC 9810, which is historically and cross‑document inaccurate and can confuse readers trying to map older deployments or profiles (that cite RFC 4210’s Section 8.1) to the new specification.

- **E2:**

  Section 1.3 bullet:  
          *  Added Sections 8.1, 8.5, 8.8, and 8.11 to the security considerations.
Security Considerations in RFC 9810 include:  
          8.1 On the Necessity of POP  
          8.2 POP with a Decryption Key  
          8.3 POP by Exposing the Private Key  
          8.4 Attack Against DH Key Exchange  
          8.5 Perfect Forward Secrecy  
          8.6 Private Keys for Certificate Signing and CMP Message Protection  
          8.7 Entropy of Random Numbers, Key Pairs, and Shared Secret Information  
          8.8 Recurring Usage of KEM Keys for Message Protection  
          8.9 Trust Anchor Provisioning Using CMP Messages  
          8.10 Authorizing Requests for Certificates with Specific EKUs  
          8.11 Usage of CT Logs

**Evidence Summary:**

- (E1) RFC 9810 Section 1.3 misattributes Section 8.1 as newly added despite its earlier existence in RFC 4210.
- (E2) The change log bullet lists Sections 8.1, 8.5, 8.8, and 8.11 without distinguishing between pre‑existing and newly introduced content.

**Fix Direction:**

In Section 1.3, replace the phrase 'Added Sections 8.1, 8.5, 8.8, and 8.11 to the security considerations.' with 'Updated Sections 8.1 and 8.5 and added new Sections 8.8 and 8.11 to the security considerations.'

**Severity:** Low
  *Basis:* The issue is editorial and creates historical confusion but does not impact the protocol functionality.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1
- Terminology Expert: Issue-1

---

## Report 2: 9810-1-2

**Label:** Ambiguous Introduction of CMP Version 3 (cmp2021) in RFC 9810

**Bug Type:** Inconsistency

**Explanation:**

RFC 9810 contains phrasing that ambiguously suggests it newly introduces CMP version 3, even though that version (pvno cmp2021(3)) was already defined in RFC 9480, which may mislead readers about its provenance.

**Justification:**

- Section 1.2 correctly attributes the introduction of CMP version 3 to RFC 9480, yet Section 1.3 reiterates that 'CMP version 3 is introduced', leading to ambiguity (E1, E2).
- This redundancy in language causes confusion over whether RFC 9810 is presenting a new version or extending the existing one.

**Evidence Snippets:**

- **E1:**

  Section 1.2 of RFC 9810 correctly describes RFC 9480 as introducing CMP version 3 (pvno value cmp2021(3)) “in case a transaction is supposed to use EnvelopedData,” which aligns with RFC 9480’s update to PKIHeader and version negotiation: cmp2021 is used when EnvelopedData or hashAlg is needed. Later, Section 1.3, describing changes made “by this document,” states that “CMP version 3 is introduced for changes to the ASN.1 syntax, which support EnvelopedData, certConf with hashAlg, POPOPrivKey with agreeMAC, and RootCaKeyUpdateContent in ckuann messages.” Taken literally, this reads as though RFC 9810 is newly introducing version 3, even though Section 1.2 (and RFC 9480 §2.3 and §2.20) already attribute that introduction to RFC 9480. The intent seems to be that RFC 9810 *extends the use* of cmp2021 to additional syntax (agreeMAC and RootCaKeyUpdateContent) while keeping the same version number, but the “is introduced” wording contradicts the earlier description and may mislead readers about where CMP v3 first appeared. This is mostly historical/documentation confusion rather than a wire‑level interoperability problem.

- **E2:**

  Section 1.2 (describing RFC 9480):  
          “To properly differentiate the support of EnvelopedData instead of EncryptedValue, CMP version 3 is introduced in case a transaction is supposed to use EnvelopedData.”
Section 1.3 (describing RFC 9810 itself):  
          “CMP version 3 is introduced for changes to the ASN.1 syntax, which support EnvelopedData, certConf with hashAlg, POPOPrivKey with agreeMAC, and RootCaKeyUpdateContent in ckuann messages.”
Protocol version definition in PKIHeader:  
          “pvno INTEGER { cmp1999(1), cmp2000(2), cmp2021(3) }”  
Version negotiation rules:  
          “Version cmp2021 SHOULD only be used if cmp2021 syntax is needed for the request being sent or for the expected response.”

**Evidence Summary:**

- (E1) Section 1.3’s wording implies RFC 9810 is introducing CMP version 3, contradicting Section 1.2 and earlier RFC 9480 attributions.
- (E2) Multiple excerpts reveal conflicting statements about the introduction of CMP version 3, creating ambiguity.

**Fix Direction:**

In Section 1.3, adjust the wording to acknowledge that CMP version 3 (pvno cmp2021(3)) was originally introduced in RFC 9480 and is merely extended in RFC 9810 for additional syntax support.

**Severity:** Low
  *Basis:* This is a documentation ambiguity that may confuse historical understanding but does not affect protocol interoperability.

**Confidence:** Medium

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-3
- Terminology Expert: Issue-2

---

## Report 3: 9810-1-3

**Label:** Incorrect Historical Appendix Reference for Algorithm Profile in RFC 9810 Section 1.2

**Bug Type:** Inconsistency

**Explanation:**

RFC 9810 erroneously refers to Appendix C.2 as the location of the deleted mandatory algorithm profile from RFC 4210, whereas the profile is actually found in Appendix D.2, potentially misleading readers.

**Justification:**

- RFC 9810 Section 1.2 states that RFC 9480 'Deleted the mandatory algorithm profile in Appendix C.2', but RFC 4210’s algorithm profile is located in Appendix D.2 (E1).
- RFC 9480’s explicit reference in §2.29 confirms that Appendix D.2 of RFC 4210 provided the algorithm list, making the reference in RFC 9810 historically inaccurate.

**Evidence Snippets:**

- **E1:**

  In Section 1.2, RFC 9810 says that RFC 9480 “Deleted the mandatory algorithm profile in Appendix C.2 and instead referred to Section 7 of [RFC9481].” However, RFC 4210’s mandatory algorithm profile is actually in Appendix D.2, and RFC 9480 explicitly refers to and replaces Appendix D.2, not C.2, when it redirects implementations to the CMP Algorithms specification. RFC 9480 §2.29 states that “Appendix D.2 of [RFC4210] provides a list of algorithms…” and that this is replaced by a pointer to Section 7.1 of RFC 9481. RFC 9810 later reorganizes appendices so that its own algorithm profile is in Appendix C.2, but Section 1.2 is specifically describing what RFC 9480 did to RFC 4210, so the correct historical reference should be “Appendix D.2”, not “Appendix C.2”. This mismatch can cause confusion for readers who go back to RFC 4210 looking for Appendix C.2 and do not find the algorithm profile there.

**Evidence Summary:**

- (E1) RFC 9810 incorrectly references Appendix C.2 instead of Appendix D.2 for RFC 4210’s mandatory algorithm profile, conflicting with the explicit guidance in RFC 9480 §2.29.

**Fix Direction:**

In Section 1.2, update the reference from 'Appendix C.2' to 'Appendix D.2' to correctly reflect the historical location of the algorithm profile in RFC 4210.

**Severity:** Low
  *Basis:* This error is an editorial inconsistency that might mislead readers checking historical documents but does not impact protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-2

---
