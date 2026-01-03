# Errata Reports

Total reports: 4

---

## Report 1: draft-ietf-dhc-rfc8415bis-12-20-1

**Label:** Recovery after RDM Counter Loss or Rollback

**Bug Type:** Underspecification

**Explanation:**

The specification does not define any recovery or resynchronization mechanism when a server loses or resets its replay-detection counter, potentially causing clients to permanently reject legitimate Reconfigure messages.

**Justification:**

- The text explains that if a server does not persist its replay detection value (e.g., due to a restart, database loss, or counter rollback), clients will reject subsequent Reconfigure messages with lower counter values.
- It explicitly states that there is no defined fallback when the counter state is lost, leaving implementations with a brittle dependency on persistent state.

**Evidence Snippets:**

- **E1:**

  The text notes that if a server does not persist this value across restarts, clients may permanently refuse Reconfigure messages from that server, but it does not specify *any* recovery or resynchronization behavior for the case where the server’s counter state is lost or rolled back (e.g., database loss, migration to a new instance with the same DUID, or a mis-implemented “NTP timestamp” source that jumps backwards).

- **E2:**

  There is no specified way for either side to recover from this mismatch: the server is only told “MUST ensure that the replay detection value is retained between restarts,” and the client has no described mechanism to “forget” or resynchronize the per-server RDM (other than an implementation-specific choice to wipe all state on reboot).

**Evidence Summary:**

- (E1) Highlights that a server losing its replay counter state leads to clients permanently rejecting subsequent messages.
- (E2) Emphasizes the absence of any recovery mechanism or fallback procedure in the specification.

**Fix Direction:**

Specify explicit recovery or resynchronization procedures for when the server’s replay-detection counter is lost or reset.

**Severity:** Low
  *Basis:* The issue is operationally fragile but occurs under rare conditions; however, the lack of a recovery mechanism can silently disable the reconfigure mechanism.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T1

---

## Report 2: draft-ietf-dhc-rfc8415bis-12-20-2

**Label:** Ambiguous Replay-Counter Wrap-Around and NTP Behavior in RDM=0

**Bug Type:** Underspecification

**Explanation:**

The specification mandates a strictly monotonically increasing 64‑bit replay counter while also describing it with modulo arithmetic and suggesting the use of an NTP timestamp, leaving behavior undefined for counter wrap-around or clock adjustments.

**Justification:**

- The spec requires the replay counter to be strictly increasing but also indicates a modulo 2^64 arithmetic, which are mathematically incompatible in the event of counter wrap-around.
- It further suggests using the 64‑bit NTP timestamp format without addressing how backward clock adjustments or wrap-around should be handled.

**Evidence Snippets:**

- **E1:**

  If the RDM field contains 0x00, the replay detection field MUST be set to the value of a strictly monotonically increasing 64-bit unsigned integer (modulo 2^64). One choice might be to use the 64-bit NTP timestamp format [RFC5905]. (Section 20.3)

- **E2:**

  A client that receives a message with the RDM field set to 0x00 MUST compare its replay detection field with the previous value sent by that same server … and only accept the message if the received value is greater and record this as the new value. (Section 20.3)

- **E3:**

  The specification mixes the requirement for a strictly monotonically increasing counter with modulo arithmetic, leaving behavior undefined at counter wrap-around or when using an NTP timestamp that may move backwards due to clock adjustments.

**Evidence Summary:**

- (E1) States the requirement and example using a 64-bit NTP timestamp with modulo arithmetic.
- (E2) Defines the client’s strict greater-than check for the replay detection value.
- (E3) Points out the incompatibility between strict monotonicity and modulo arithmetic without provided resolution.

**Fix Direction:**

Clarify and define the behavior for counter wrap-around and for handling backward adjustments when using NTP-based counters.

**Severity:** Low
  *Basis:* While the practical impact is minimal due to the large counter size, the ambiguity could affect long-term reliability and interoperability in extreme edge cases.

**Confidence:** High

**Experts mentioning this issue:**

- Temporal Expert: T2
- Quantitative Expert: Issue-1

---

## Report 3: draft-ietf-dhc-rfc8415bis-12-20-3

**Label:** Unclear Negotiation Requirements for RKAP-Protected Reconfigure

**Bug Type:** Underspecification

**Explanation:**

The protocol does not clearly define the negotiation process for using RKAP-protected Reconfigure messages, leading to ambiguity about whether the Reconfigure Accept option, RKAP key exchange, or both are required.

**Justification:**

- The specification states that RKAP is used only if the client and server have 'negotiated' to use Reconfigure messages, but it does not normatively define what constitutes such negotiation.
- Different sections offer conflicting indications by suggesting both the necessity of the Reconfigure Accept option and the sufficiency of merely sending an RKAP key.

**Evidence Snippets:**

- **E1:**

  RKAP is used (initiated by the server) only if the client and server have negotiated to use Reconfigure messages. (Section 20.4)

- **E2:**

  A client uses the Reconfigure Accept option to announce to the server whether the client is willing to accept Reconfigure messages, and a server uses this option to tell the client whether or not to accept Reconfigure messages. In the absence of this option, the default behavior is that the client is unwilling to accept Reconfigure messages. (Section 21.20)

- **E3:**

  The document never normatively specifies what “negotiated to use Reconfigure messages” means in protocol terms, leading to divergent interpretations in implementations.

**Evidence Summary:**

- (E1) Indicates that RKAP usage is contingent upon prior negotiation without details.
- (E2) Describes the use of the Reconfigure Accept option as a signal for willingness to accept Reconfigure messages.
- (E3) Explicitly states that the negotiation process is not defined, resulting in ambiguity.

**Fix Direction:**

Define an explicit negotiation procedure detailing whether the Reconfigure Accept option, the RKAP key exchange, or a combination thereof is required for valid use of RKAP-protected Reconfigure messages.

**Severity:** Low
  *Basis:* The ambiguity affects policy and deployment interpretations rather than core protocol interoperability, but could still lead to inconsistent security behavior.

**Confidence:** High

**Experts mentioning this issue:**

- Scope Expert: Issue-1
- Deontic Expert: Issue-1

---

## Report 4: draft-ietf-dhc-rfc8415bis-12-20-4

**Label:** Misstated Mitigation for RKAP Key Disclosure via RFC 7610/SAVI

**Bug Type:** Inconsistency

**Explanation:**

The security discussion inaccurately implies that deploying RFC 7610 (DHCPv6‑Shield) and RFC 7513 (SAVI) can prevent RKAP reconfigure key disclosure, overstating their protective guarantees.

**Justification:**

- The text suggests that RKAP key exposure over public networks can be easily prevented by using these mechanisms.
- In reality, RFC 7610 provides filtering of rogue DHCP messages and RFC 7513 focuses on validating source bindings, neither of which encrypts or confidentially protects legitimate DHCPv6 exchanges.

**Evidence Snippets:**

- **E1:**

  The privacy/security discussion states that an attacker on a public (e.g., Wi‑Fi) network could observe the RKAP reconfigure key during the initial exchange and that “this is relatively easily prevented by disallowing direct client‑to‑client communication on these networks or using [RFC7610] and [RFC7513].”

- **E2:**

  RFC 7610, DHCPv6‑Shield is explicitly defined as a filter to block DHCPv6 server‑to‑client messages that arrive on untrusted L2 ports, in order to protect against rogue DHCPv6 servers; it does not provide confidentiality for legitimate DHCPv6 traffic and does not prevent a host from listening on a port that is allowed to receive such traffic.

- **E3:**

  Thus, saying that use of RFC 7610 and RFC 7513 “relatively easily” prevents an attacker from learning the RKAP key by eavesdropping over a shared medium overstates what those RFCs actually guarantee and misrepresents their scope.

**Evidence Summary:**

- (E1) Indicates the claimed mitigation for RKAP key exposure using RFC 7610 and RFC 7513.
- (E2) Explains that these RFCs only filter or validate DHCPv6 messages and do not ensure confidentiality.
- (E3) Concludes that the stated mitigation is an overstatement of the RFCs' capabilities.

**Fix Direction:**

Revise the text to accurately reflect that RFC 7610 and RFC 7513 provide filtering and validation, not confidentiality, thereby not fully preventing passive RKAP key disclosure.

**Severity:** Medium
  *Basis:* The misrepresentation could lead operators to rely on insufficient protection measures, posing a potential security risk.

**Confidence:** High

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1

---
