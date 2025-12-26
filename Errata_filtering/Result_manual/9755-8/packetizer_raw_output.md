# Errata Reports

Total reports: 2

---

## Report 1: 9755-8-1

**Label:** Ambiguous UID/UIDVALIDITY Semantics for Surrogate and Original Representations

**Bug Type:** Underspecification / Cross-RFC Conflict

**Explanation:**

The specification permits a server to deliver different on‐the‐wire representations (an original internationalized message versus a downgraded surrogate) under the same (mailbox, UIDVALIDITY, UID) triple, which conflicts with RFC 3501’s mandate that message contents remain immutable.

**Justification:**

- Multiple experts note that while RFC 3501 requires a single immutable message per UID triple, RFC 9755 (with RFC 6857/6858) contemplates delivering different representations depending on client capability.
- The document relies on a client-side cache flush as a workaround while leaving server behavior ambiguous, risking interoperability issues.

**Evidence Snippets:**

- **E1:**

  servers that may be accessed by the same user with different clients or methods … will need to exert extreme care to be sure that UIDVALIDITY [RFC3501] behaves as the user would expect. Those issues may be especially sensitive if the server caches the surrogate message or computes and stores it when the message arrives with the intent of making either form available depending on client capabilities.

- **E2:**

  Additionally, in order to cope with the case when a server compliant with this extension returns the same UIDVALIDITY to both legacy and ‘UTF8=ACCEPT’-aware clients, a client upgraded from being non-‘UTF8=ACCEPT’-aware MUST discard its cache of messages downloaded from the server.

- **E3:**

  RFC 3501: “The combination of mailbox name, UIDVALIDITY, and UID must refer to a single immutable message on that server forever. In particular, the internal date, [RFC-2822] size, envelope, body structure, and message texts … must never change.”

- **E4:**

  RFC 6857/6858 description that servers may create “surrogate” messages by downgrading the original internationalized message

**Evidence Summary:**

- (E1) Describes the need for extreme care when servers cache or compute surrogate messages under the same UIDVALIDITY.
- (E2) Specifies the client-side requirement to discard caches when the same UIDVALIDITY is returned for different representations.
- (E3) Quotes RFC 3501’s mandate that a UID triple must always refer to an immutable message.
- (E4) Indicates that the surrogate mechanism allows creation of alternate message representations.

**Fix Direction:**

Clarify the allowed server behavior: either mandate that different message representations use distinct UIDs/UIDVALIDITY (or mailboxes) to preserve immutability per RFC 3501, or explicitly allow per‐client presentation differences with a clear exception.


**Severity:** High
  *Basis:* Multiple expert analyses highlight that the potential for different on‐the‐wire representations under a stable UID/UIDVALIDITY conflicts with RFC 3501’s immutability guarantee, risking severe interoperability and synchronization issues.

**Confidence:** High

---

## Report 2: 9755-8-2

**Label:** Ambiguous Scope of Client Cache Invalidation Requirement

**Bug Type:** Underspecification

**Explanation:**

The specification mandates that a client upgraded to UTF8=ACCEPT discard its cache when the server returns the same UIDVALIDITY, yet it does not precisely define which caches or data domains this applies to, leading to potential inconsistent client behavior.

**Justification:**

- The normative text uses broad language without clarifying whether the cache in question is per-mailbox, global, or even applies to protocols other than IMAP.
- Different interpretations could result in either over-aggressive cache invalidation or insufficient updating, undermining the goal of ensuring consistency.

**Evidence Snippets:**

- **E1:**

  Additionally, in order to cope with the case when a server compliant with this extension returns the same UIDVALIDITY to both legacy and ‘UTF8=ACCEPT’-aware clients, a client upgraded from being non-‘UTF8=ACCEPT’-aware MUST discard its cache of messages downloaded from the server.

- **E2:**

  What exact set of data constitutes “its cache of messages downloaded from the server” (per-mailbox vs all mailboxes; IMAP-only vs POP as well; headers-only caches vs full-body offline copies).

**Evidence Summary:**

- (E1) Specifies the client must discard its cache when faced with the same UIDVALIDITY serving different representations, but does not define the scope of the cache.
- (E2) Raises questions about exactly which cached data the term 'cache' is intended to cover.

**Fix Direction:**

Clearly define the cache invalidation boundary by specifying which caches (e.g., per-mailbox IMAP caches, POP caches, or other local archives) must be discarded upon client upgrade.


**Severity:** Medium
  *Basis:* Ambiguity in the cache invalidation requirement can lead to inconsistent client behavior and potential message synchronization issues across different access methods.

**Confidence:** High

---
