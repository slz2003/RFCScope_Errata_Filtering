# Errata Reports

Total reports: 5

---

## Report 1: 9904-1-1

**Label:** Ambiguous Lifecycle Ordering from MAY to RECOMMENDED to MUST for DNSSEC Algorithms

**Bug Type:** Underspecification

**Explanation:**

The document describes the evolution of algorithm requirement levels without clearly specifying when a new algorithm should transition from an initial MAY registration to being introduced as RECOMMENDED and later declared MUST.

**Justification:**

- Sections 1.2 and 2.2 present conflicting descriptions—one implying an immediate RECOMMENDED status and the other mandating an initial MAY phase—leading to ambiguity about the ordered lifecycle steps for algorithm status escalation (see evidence E1–E3).

**Evidence Snippets:**

- **E1:**

  Similarly, an algorithm that has not been mentioned as mandatory to implement is expected to be first introduced as RECOMMENDED instead of a MUST. (Section 1.2)

- **E2:**

  Adding a new entry to the ‘DNS Security Algorithm Numbers’ registry with a recommended value of ‘MAY’… New entries added through the Specification Required process will have the value of ‘MAY’ for all columns. (Section 2.2)

- **E3:**

  Adding a new entry to, or changing an existing value in, the ‘DNS Security Algorithm Numbers’ registry that has any value other than ‘MAY’… requires Standards Action. (Section 2.2)

**Evidence Summary:**

- (E1) Indicates that an algorithm is expected to be introduced as RECOMMENDED under certain conditions.
- (E2) Defines the initial registration as MAY in the registry.
- (E3) Establishes that any non-MAY value requires Standards Action.

**Fix Direction:**

Clarify and explicitly outline the ordered steps for status escalation, detailing the transition from MAY to RECOMMENDED and then to MUST.

**Severity:** Low
  *Basis:* The ambiguity could lead to inconsistent interpretations among future RFC authors and IANA reviewers, though it does not impact wire-level interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T1

---

## Report 2: 9904-1-2

**Label:** Under-specified Timing for DNSSEC Algorithm Deprecation Sequence

**Bug Type:** Underspecification

**Explanation:**

The deprecation sequence for DNSSEC algorithms is described qualitatively without specifying objective timing or observable deployment thresholds for transitioning between statuses.

**Justification:**

- Section 1.2 outlines a gradual deprecation process but does not provide criteria or timelines for moving from MUST/RECOMMENDED to NOT RECOMMENDED/MAY and finally to MUST NOT, leaving the timing ambiguous (see evidence E1–E5).

**Evidence Snippets:**

- **E1:**

  It is expected that the deprecation of an algorithm will be performed gradually. This provides time for implementations to update their implemented algorithms while remaining interoperable. (Section 1.2)

- **E2:**

  Unless there are strong security reasons, an algorithm is expected to be downgraded from MUST to NOT RECOMMENDED or MAY, instead of directly from MUST to MUST NOT. (Section 1.2)

- **E3:**

  Once an algorithm has reached a sufficiently low level of deployment, it can be marked as MUST NOT, so that recursive resolvers can remove support for validating it. (Section 1.2)

- **E4:**

  Validating recursive resolvers are encouraged to retain support for all algorithms not marked as MUST NOT. (Section 1.2)

- **E5:**

  We note that the values for ‘Implement for’ and ‘Use for’ may diverge in the future as implementations generally precede deployments. (Section 2.2)

**Evidence Summary:**

- (E1) Introduces gradual deprecation without specific timing.
- (E2) Describes a downgrade from MUST to NOT RECOMMENDED/MAY without criteria.
- (E3) Sets the transition to MUST NOT based on an unspecified low deployment threshold.
- (E4) Encourages validators to retain support, adding to the ambiguity.
- (E5) Notes potential divergence between implementation and usage statuses.

**Fix Direction:**

Provide objective criteria or recommended timelines for each phase of deprecation to remove ambiguity.

**Severity:** Low
  *Basis:* The lack of concrete timing guidance may lead to varied deprecation schedules across RFCs, causing operational uncertainty without breaking interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- TemporalExpert: T2

---

## Report 3: 9904-1-3

**Label:** Ambiguity in Actor Specification for DS Algorithm Rollover Requirements

**Bug Type:** Underspecification

**Explanation:**

The guidance for DS algorithm rollover uses the vague term 'users' instead of clearly specifying the intended actor, such as zone operators, leading to potential confusion over who is responsible for the upgrade.

**Justification:**

- The requirement states that 'users MUST upgrade the DS algorithm first' but does not clarify that this obligation is intended for zone administrators, creating ambiguity in actor responsibility (see evidence E1).

**Evidence Snippets:**

- **E1:**

  DS algorithm rollover in a live zone is also a complex process. Upgrading an algorithm at the same time as rolling to the new Key Signing Key (KSK) key will lead to DNSSEC validation failures, and users MUST upgrade the DS algorithm first before rolling to a new KSK.

**Evidence Summary:**

- (E1) The mandate for DS algorithm rollover is directed to 'users' without specifying the proper actor.

**Fix Direction:**

Explicitly specify the intended actor (e.g. 'zone operators' or 'DNS zone administrators') in the DS algorithm rollover guidance.

**Severity:** Low
  *Basis:* This ambiguity may lead to misinterpretation of operational responsibilities, although it does not directly affect protocol behavior.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionalityExpert: NewIssue-1

---

## Report 4: 9904-1-4

**Label:** Inconsistent Terminology for DNSSEC Validators and Validating Resolvers

**Bug Type:** Underspecification

**Explanation:**

The document inconsistently uses multiple terms—such as 'DNSSEC validators', 'validating resolvers', and 'validating recursive resolvers'—which can confuse readers about the specific entities subject to the recommendations.

**Justification:**

- Column descriptions and Section 1.2 interchangeably refer to different validation entities, obscuring the intended scope of the recommendations (see evidence E1 and E2).

**Evidence Snippets:**

- **E1:**

  Column descriptions: Use for DNSSEC Validation: Indicates the recommendation for using the algorithm in DNSSEC validators. / Implement for DNSSEC Validation: Indicates the recommendation for implementing the algorithm within DNSSEC validators. and, for digest algorithms, Implement for DNSSEC Validation: Indicates the recommendation for implementing the algorithm within validating resolvers.

- **E2:**

  Section 1.2: Once an algorithm has reached a sufficiently low level of deployment, it can be marked as MUST NOT, so that recursive resolvers can remove support for validating it. Validating recursive resolvers are encouraged to retain support for all algorithms not marked as MUST NOT.

**Evidence Summary:**

- (E1) Highlights the use of different labels for entities involved in DNSSEC validation.
- (E2) Shows that Section 1.2 reinforces the use of multiple, potentially confusing terms.

**Fix Direction:**

Harmonize the terminology throughout the document to consistently refer to a single entity for DNSSEC validation.

**Severity:** Low
  *Basis:* The inconsistencies may lead to misinterpretation about which validation components are affected, though there is no direct impact on interoperability.

**Confidence:** High

**Experts mentioning this issue:**

- ActorDirectionalityExpert: NewIssue-2

---

## Report 5: 9904-1-5

**Label:** Oversimplified Impact of Unknown DNSKEY Algorithm on Zone Security

**Bug Type:** Underspecification

**Explanation:**

The document states that using an unknown DNSKEY algorithm renders the zone insecure, oversimplifying DNSSEC validation which actually permits secure validation as long as at least one supported algorithm is present.

**Justification:**

- Section 1.2 asserts that an unknown DNSKEY algorithm leads to an insecure zone without specifying that this applies only when no supported algorithm is present (see evidence E1).
- Deontic and CrossRFC analyses highlight that the wording overgeneralizes the effect and ignores the per-validator, multi-algorithm resilience defined in core DNSSEC specifications (see evidence E2).

**Evidence Snippets:**

- **E1:**

  Since the effect of using an unknown DNSKEY algorithm is that the zone is treated as insecure, it is recommended that algorithms that have been downgraded to NOT RECOMMENDED or lower not be used by authoritative nameservers and DNSSEC signers to create new DNSKEYs. This ensures that the use of deprecated algorithms decreases over time.

- **E2:**

  Since the effect of using an unknown DNSKEY algorithm is that the zone is treated as insecure, it is recommended that algorithms that have been downgraded to NOT RECOMMENDED or lower not be used by authoritative nameservers and DNSSEC signers to create new DNSKEYs.

**Evidence Summary:**

- (E1) Claims that an unknown DNSKEY algorithm makes the zone insecure without qualifiers.
- (E2) Reiterates the oversimplification, as noted by multiple expert analyses, neglecting the conditional nature of validator support.

**Fix Direction:**

Revise the statement to clarify that a zone is treated as insecure by a validator only if all DNSKEY algorithms in use are unsupported.

**Severity:** Low
  *Basis:* The oversimplified wording might lead operators to adopt overly conservative practices but does not alter the underlying DNSSEC validation process.

**Confidence:** High

**Experts mentioning this issue:**

- ScopeExpert: Issue-1
- DeonticExpert: Issue-1
- CrossRFCExpert: Issue-1

---
