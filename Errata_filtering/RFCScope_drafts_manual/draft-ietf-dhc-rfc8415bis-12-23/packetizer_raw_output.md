# Errata Reports

Total reports: 1

---


## Report 3: draft-ietf-dhc-rfc8415bis-12-23-3

**Label:** Understated Replay Attack Impact on Resource Exhaustion with Manual IPsec Keys

**Bug Type:** Inconsistency

**Explanation:**

Section 23.2 claims that replay attacks with manually configured pre-shared IPsec keys only lead to CPU/bandwidth exhaustion, but in reality, such replays can continuously refresh or recreate DHCPv6 bindings and tie up address/prefix pools.

**Justification:**

- An attacker replaying captured, ESP-protected messages can force the server to treat them as new requests, resulting in repeated (re)allocation of addresses or prefixes.
- Repeated replays may prevent bindings from expiring, effectively causing resource exhaustion beyond mere processing delays.

**Evidence Snippets:**

- **E1:**

  If an attacker can replay previously captured, IPsec-protected client/relay traffic after the original bindings have expired and been returned to the pool, the server will treat those as fresh requests and (re)allocate addresses and/or prefixes to those DUID/IAID tuples, consuming real pool entries for “ghost” clients. Repeating this with a recorded population of clients can indeed tie up assignable addresses and delegated prefixes without any current client using them, i.e., a resource-exhaustion attack beyond mere CPU/bandwidth DoS.

- **E2:**

  replayed renewals can keep those resources perpetually “in use” from the server’s point of view, even though no real clients are consuming them. That is a form of resource exhaustion of “assignable addresses and delegated prefixes”, not merely CPU exhaustion.

**Evidence Summary:**

- (E1) Describes how replays of IPsec-protected messages can force reallocation of resources, leading to pool exhaustion.
- (E2) Emphasizes that repeated renewals via replay can keep address/prefix resources continuously allocated.

**Fix Direction:**

Update Section 23.2 to acknowledge that replay attacks under manual-key IPsec can lead to address and prefix pool exhaustion, and recommend deployment of effective replay protection measures.

**Severity:** Medium
  *Basis:* The potential for resource depletion poses a significant operational risk if replay protection is not implemented.

**Confidence:** Medium

**Experts mentioning this issue:**

- CrossRFC Expert: Issue-1
- Boundary Expert: Finding-2

---