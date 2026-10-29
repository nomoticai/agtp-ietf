---
title: "AGTP Session Protocol"
abbrev: "AGTP-SESSION"
docname: draft-hood-agtp-session-01
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - sessions
  - agent communication
  - long-lived connections

stand_alone: yes
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
  - fullname: Chris Hood
    organization: Nomotic, Inc.
    email: chris@nomotic.ai
    uri: https://nomotic.ai

normative:
  RFC2119:
  RFC8174:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026
  RFC7515:
  RFC8785:

informative:
  RFC9000:
  RFC9001:
  AGTP-IDENTIFIERS:
    title: "AGTP Identifier Chain"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identifiers-03
    date: 2026
  HCTP:
    title: "The Hash-Chain Context Transfer Protocol"
    author:
      fullname: Cristian Xavier Ruvalcaba
    seriesinfo:
      Internet-Draft: draft-ruvalcaba-hctp-00
    date: 2026
  AGTP-IDDOC:
    title: "AGTP Agent Identity Document"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identity-document-00
    date: 2026
  AGTP-CERT:
    title: "AGTP Agent Certificate Extension"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-agent-cert-04
    date: 2026
  AGTP-LOG:
    title: "AGTP Transparency Log Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-03
    date: 2026
  AGTP-BINDINGS:
    title: "AGTP Transport Bindings: TCP/TLS and QUIC"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-bindings-01
    date: 2026
  AGTP-COMMUNICATION:
    title: "AGTP Communication Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-communication-01
    date: 2026

--- abstract

This document specifies the AGTP Session Protocol (AGTP-SESSION), a
companion to {{AGTP}} that defines session semantics for agent-to-agent
and agent-to-API communication. AGTP-SESSION addresses two distinct
session models: bounded sessions for time-limited transactional flows,
and persistent sessions for long-lived agent contexts. Both inherit
identity, authority, and attribution from base AGTP. This revision
specifies the session identifier format, the ESTABLISH, RESUME, and
TERMINATE methods and their wire format, and resumption after
transport loss, in which a session is bound to its participants by a
signed session token and to the live sender by the per-request
signature of base AGTP rather than to any transport connection.

--- middle

# Introduction

Base AGTP defines request-and-response semantics for individual method
invocations between agents. Many real-world agent interactions span
multiple invocations under shared context: a chatbot conversation that
continues across hours, an agentic commerce flow that completes a
transaction across a sequence of methods, a research agent maintaining
working state across a long-running task.

These interactions need session semantics that base AGTP leaves
unspecified. AGTP-SESSION defines those semantics as a companion
protocol that builds on base AGTP without modifying it.

The architectural commitment is simple: a session in AGTP is an
agent-level abstraction, distinct from a TLS connection, a QUIC
connection, or any transport-level session. It is a logical context
shared between two or more agents (or between an agent and an API),
within which multiple method invocations carry shared identity,
authority scope, and attribution chain.

A session may map to a single underlying transport connection, may
span multiple connections over time, or may persist while the
underlying transport is intermittently disconnected and reestablished.
The session state is the agent-level fact; the connection is the
mechanism that carries traffic when active.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in
all capitals, as shown here.

Session:
: A logical context shared between AGTP participants under which
  multiple method invocations are correlated, identity and authority
  remain consistent, and shared state evolves across the lifetime of
  the session. Sessions are agent-level abstractions rather than
  transport-level constructs.

Bounded Session:
: A session with an explicit termination condition: a time limit, an
  action completion, or a transaction outcome. Bounded sessions are
  appropriate for commerce flows, multi-step delegations with finite
  scope, and any agent interaction with a defined end state.

Persistent Session:
: A long-lived session intended to span hours, days, or longer. State
  is maintained across reconnections. Persistent sessions are
  appropriate for chatbot interactions, research agents, monitoring
  agents, and other contexts where the agent retains continuity.

Session Identifier:
: A 256-bit cryptographically random identifier assigned by the
  responder at session establishment, encoded as 64 lowercase
  hexadecimal characters, carried in the `Session-ID` header of every
  method invocation within the session. Unique per responder; never
  derived from either participant's Agent-ID. See
  {{session-identifier}}.

Session Token:
: The signed record of a session's establishment: a compact JWS
  {{RFC7515}} issued by the responder over the session identifier,
  the two participant Agent-IDs, the session model, the expiry, the
  resume window, and the session-scoped Authority-Scope grants. The
  token is what binds a session to its participants. See
  {{session-token}}.

Session Root:
: A 32-byte rolling SHA-256 commitment over the ordered sequence of
  Audit-IDs of the responses exchanged under a session, folded by
  both participants independently. Equal roots mean the two agree on
  what the session has carried; unequal roots name the point of
  divergence. See {{session-root}}.

Initiator, Responder:
: The agent that sends ESTABLISH and the agent that accepts it. The
  roles persist for the session's lifetime: only the initiator may
  RESUME, and either may TERMINATE.

Session Context:
: The shared state associated with a session, including at minimum
  participant Agent-IDs, Authority-Scope grants applicable
  to the session, accumulated Attribution-Records for invocations
  within the session, and any session-specific governance metadata.

# Session Models

AGTP-SESSION defines two session models. Implementations MAY support
one or both depending on deployment profile.

## Bounded Sessions

Bounded sessions exist for a finite, often short period and complete
when one of the following conditions is met:

- An explicit time limit (`session_ttl`) expires
- A defined transaction or workflow completes
- A participant explicitly terminates the session
- An authority constraint is violated (e.g., scope limit reached)

Bounded sessions match the semantics of agentic transactions where the
session has a clear lifecycle: an agent initiates a purchase flow,
exchanges several methods with a merchant, completes the transaction,
and the session closes. The session boundary is meaningful to
governance and attribution: all method invocations within a bounded
session are correlated under a single transactional context.

## Persistent Sessions

Persistent sessions exist for extended periods and survive transport
disconnection, network instability, and platform restarts through
the resumption procedure of {{resumption}}. The session identifier
remains stable; the underlying transport may reconnect any number of
times during the session lifetime, and each reconnection is
re-bound to the session by the initiator's RESUME.

Persistent sessions match the semantics of long-lived agent contexts:
a chatbot maintaining conversation continuity, a research agent
working on a multi-day task, a monitoring agent maintaining persistent
attention to a system. Authority-Scope grants applicable to the
session may be re-validated periodically; identity verification
follows base AGTP semantics on reconnection.

# Session Establishment

A session is established explicitly via an `ESTABLISH` method invoked
between two AGTP participants. Implicit establishment via an extended-
context flag on a standard method invocation is noted as an
alternative under consideration but is **NOT RECOMMENDED** as the
primary mechanism. Explicit establishment provides:

- Unambiguous negotiation of session model (bounded or persistent),
  session lifetime, and any session-scoped Authority-Scope grants
- A clear root for the session attribution chain, with the ESTABLISH
  invocation serving as the first signed Attribution-Record under
  the session
- Clean error semantics when negotiation fails (455 Scope Violation
  if session-scoped grants exceed the establishing agent's
  Authority-Scope, 4xx for unsupported session models, 5xx for
  governance platform unavailability)
- Predictable enforcement boundaries for governance platforms and
  Scope-Enforcement Points, which can recognize session establishment
  as a distinct event rather than inferring it from method context

## ESTABLISH Wire Format {#establish}

ESTABLISH is invoked on the path `/session` with no `Session-ID`
header and carries the standard AGTP request headers of {{AGTP}},
including `Request-Signature`. The request body is a JSON object:

~~~~
AGTP/1.0 ESTABLISH /session
Agent-ID: <initiator Agent-ID>
Authority-Scope: documents:query, knowledge:query
Request-Signature: <JWS per AGTP>
Content-Type: application/json
Content-Length: ...

{
  "model": "persistent",
  "session_ttl": 86400,
  "scopes": ["documents:query", "knowledge:query"],
  "resume_window": 3600,
  "initiator_nonce": "<base64url, at least 96 bits>"
}
~~~~

`model`:
: `bounded` or `persistent`. **REQUIRED**.

`session_ttl`:
: Proposed lifetime in seconds from establishment. **REQUIRED** for
  `bounded`; **OPTIONAL** for `persistent`, where absence means the
  responder's default.

`scopes`:
: The session-scoped Authority-Scope grants the initiator proposes,
  in the grammar of {{AGTP}}. **MUST** be a subset of the initiator's
  claimed or inherited Authority-Scope. **OPTIONAL**; absence means
  the session carries no scope narrower than the request's.

`resume_window`:
: Proposed seconds after transport loss during which the responder
  retains session state for RESUME. **OPTIONAL**; the responder
  **MAY** shorten it.

`initiator_nonce`:
: Random value echoed into the session token so the initiator can
  bind the token to this request. **REQUIRED**.

### Body Extensions {#body-extensions}

Companion specifications **MAY** add top-level keys to the ESTABLISH
request and response bodies, registered in the ESTABLISH Body
Extension Registry of {{iana}}. A responder that recognizes an
extension key negotiates it and returns the negotiated value under
the same key in the response; a responder that fails to recognize a
key **MUST** ignore it and **MUST** omit it from the response, so
that the absence of the key in the response tells the initiator the
extension went unnegotiated. An initiator for which the extension
is essential **MUST** then TERMINATE the session rather than use it
without the extension. The first registered extension is
`communication`, defined by {{AGTP-COMMUNICATION}}.

A successful response is `200 OK` with the `Session-ID` response
header carrying the new session identifier and a JSON body:

~~~~
{
  "session_id": "<64 lowercase hex>",
  "model": "persistent",
  "expires_at": "2026-10-28T22:00:00Z",
  "resume_window": 3600,
  "scopes": ["documents:query", "knowledge:query"],
  "session_token": "<compact JWS>"
}
~~~~

The responder **MUST** verify the initiator at Level 2 or Level 3
per {{AGTP}} before issuing a token, and the ESTABLISH invocation's
Attribution-Record is the first record under the session.

Rejection: `455 Scope Violation` with reason `scope-exceeds-parent`
when `scopes` exceeds the initiator's Authority-Scope; `422` when the
requested `model` is unsupported by the responder; `503` when the
responder's governance platform cannot be reached to record the
session. Implementations **MUST NOT** establish a session by side
effect of any other method invocation; the absence of a successful
ESTABLISH response means no session exists.

## Session Identifier {#session-identifier}

The responder generates the session identifier as 256 bits from a
cryptographically secure random source, encoded as 64 lowercase
hexadecimal characters. It **MUST NOT** be derived from, or
correlate with, either participant's Agent-ID or any prior session
identifier. It is carried in the `Session-ID` header on every
subsequent invocation within the session, in both directions.
Base AGTP assigns `Session-ID` no semantics; this document assigns
it the binding below.

## Session Token {#session-token}

The session token is a compact JWS {{RFC7515}}, algorithm `EdDSA`,
`kid` equal to the responder's Agent-ID, signed with the responder's
`agent_public_key` as published in its Identity Document
({{AGTP-IDDOC}}), over the {{RFC8785}} canonical form of:

~~~~
{
  "sid":        "<session identifier>",
  "initiator":  "<initiator Agent-ID>",
  "responder":  "<responder Agent-ID>",
  "model":      "bounded" | "persistent",
  "iat":        <seconds since epoch>,
  "exp":        <seconds since epoch>,
  "resume":     <resume window, seconds>,
  "scopes":     [ ... ],
  "nonce":      "<initiator_nonce>"
}
~~~~

The token is the binding between a session identifier and its two
participants. A responder's governance platform **MAY** countersign
the token (a second JWS over the same payload, carried in a
`governance_token` body field) where deployment policy requires a
platform record of the session; the responder's signature is what the
protocol depends on.

The initiator retains the token and presents it on RESUME. The
responder **MAY** retain only the session identifier and verify the
presented token by signature, which lets a responder that restarted
recover the session's binding from the token itself.

## Binding to the Live Sender

A session identifier and token establish which two agents a session
belongs to. They never establish that the sender of a given request
is one of them. That proof is the `Request-Signature` of {{AGTP}},
present on every invocation and verified under the sender's current
key. Within a session the responder **MUST** additionally check that
the request's `Agent-ID` is one of the two participants in the
token for that `Session-ID`, and **MUST** reject with `401` and
reason `session-participant-mismatch` otherwise. A session is
therefore bound to its participants by signature at establishment
and to the live sender by signature on every request, and to no
transport connection. A responder **MAY** also bind each live
connection to the session with a TLS exporter or QUIC connection
identifier as defense in depth; that binding is per connection,
re-established at every RESUME, and never the session's identity.

## Session Root {#session-root}

Two participants in a long session need a compact way to stay in
agreement about what the session has carried without resending it.
Each response under a session carries an `Audit-ID` ({{AGTP}}), the
identifier of its Attribution-Record. Both participants fold the
sequence of Audit-IDs, in the order the responder emitted them, into
a session root:

~~~~
root_0 = SHA-256( "agtp-session-root" || session_id )
root_n = SHA-256( root_{n-1} || audit_id_n )
~~~~

where `||` is byte concatenation, `session_id` is the 32 raw bytes
of the session identifier, and `audit_id_n` is the 32 raw bytes of
the n-th response's Audit-ID. The root after n responses commits to
every response so far and is independent of how long the session
has run; it is 32 bytes on the wire however many exchanges precede
it.

A participant that holds a root and the Audit-IDs since can verify
the other's root by refolding, and can fetch any response's full
record with `INSPECT target=audit` ({{AGTP}}) to verify against the
raw record later. The root is carried on RESUME, where it lets the
responder detect divergence beyond the single lost response that
`last_response_id` covers, and on TERMINATE, where the final
Attribution-Record commits to the whole session. Carriage elsewhere
(for example on every n-th response as a periodic checkpoint) is
permitted and is operator policy.

The construction is the rolling-commitment pattern of {{HCTP}},
restricted here to Audit-IDs so that it composes with the per-agent
audit chain of {{AGTP-IDENTIFIERS}}: the per-agent chain records
what one agent did; the session root records what two agents agree
the session carried. A deployment that needs the full acknowledged
history and unacknowledged-window machinery of {{HCTP}} **MAY**
carry HCTP payloads inside the session as EXECUTE content; the
session root is the minimum the session protocol itself needs.

## Resumption After Transport Loss {#resumption}

When the transport carrying a session is lost (connection reset,
network change, responder restart), the session persists in the
`Suspended-by-transport` state for `resume_window` seconds from the
last successful exchange. Within that window the initiator opens a
new transport, re-establishing identity at Level 2 or Level 3 as for
any new connection, and sends:

~~~~
AGTP/1.0 RESUME /session
Agent-ID: <initiator Agent-ID>
Session-ID: <session identifier>
Request-Signature: <JWS per AGTP>
Content-Type: application/json
Content-Length: ...

{
  "session_token": "<compact JWS from ESTABLISH>",
  "last_response_id": "<Response-ID of last response seen, or null>",
  "session_root": "<64 lowercase hex, the initiator's root, or null>"
}
~~~~

The responder **MUST** verify, in order: the `Request-Signature`;
that the token's signature verifies under its own key and its `sid`
equals the `Session-ID` header; that the request's `Agent-ID` equals
the token's `initiator`; that `exp` is in the future and the resume
window is still open; and that the session is in
`Suspended-by-transport` rather than terminated. On success the
responder returns `200 OK` with the `Session-ID` header and a body
carrying `last_response_id` and `session_root` as seen on its side,
so the initiator can detect a response lost in transit and
re-request it, and the session returns to `Active`. When the
initiator's `session_root` is present and differs from the
responder's, the responder **MUST** still return `200 OK` with its
own root and `last_response_id`, and **MUST** include
`divergence: true`; the initiator then reconciles by fetching
records with `INSPECT target=audit` from its last agreed point. A
divergent root is a signal to reconcile, never a reason to refuse
resumption, since the usual cause is a response lost in transit. On failure the responder returns `410 Gone` with
reason `session-expired` (window or `exp` passed), `session-terminated`
(a TERMINATE was processed), or `session-unknown`; a token that fails
verification is `401` with reason `session-token-invalid`.

If a RESUME arrives while an earlier connection for the same session
is still open, the responder **MUST** treat the newer connection as
authoritative, release the earlier connection's binding, and log the
event; two live connections never share a session.

RESUME is an action method for the purposes of {{AGTP-BINDINGS}} and
**MUST NOT** be sent as QUIC early data.

## Termination

Either participant ends a session with:

~~~~
AGTP/1.0 TERMINATE /session
Session-ID: <session identifier>
Request-Signature: <JWS per AGTP>

{ "reason": "<token>", "session_root": "<64 lowercase hex>" }
~~~~

`reason` is one of `completed`, `ttl-expired`, `scope-violation`,
`revoked`, `operator`, or `initiator-request`. `session_root` is the
terminating party's root over the session ({{session-root}}). The
response is `200 OK`, carries the responder's own `session_root`, and
its Attribution-Record is the session's final record, signed by the
terminating party, naming the reason, and committing to the root so
that the session's whole history is verifiable from one record. A terminated
session identifier **MUST NOT** be reused and RESUME against it
returns `410 Gone` with reason `session-terminated`. Expiry of a
bounded session's `session_ttl` is enforced against the responder's
clock and produces the same final record with reason `ttl-expired`
without a TERMINATE invocation.

## Method Registration

ESTABLISH, RESUME, and TERMINATE are registered in the AGTP Method
Registry by this document ({{iana}}). They are session-management
methods: none is safe, none is idempotent, and all three are action
methods under the properties defined in {{AGTP}} and its methods
companion.

# Inheritance from Base AGTP

AGTP-SESSION inherits all base AGTP properties:

- Agent-only at the wire level. Sessions exist between agents rather
  than between general-purpose actors.
- Agent-ID present on every method invocation, including invocations
  within a session, with Owner-ID resolved from the agent identity
  document the Agent-ID references.
- Authority-Scope enforced on every method invocation. Scopes stay
  exactly as granted inside sessions.
- Attribution Records signed for every invocation, including those
  within sessions, with the session identifier included in the
  attribution metadata.
- Status codes, including 455 Scope Violation and 551 Authority Chain
  Broken, apply within sessions exactly as outside them.
- Three-level verification model. Sessions leave verification
  unchanged.

AGTP-SESSION adds session-specific semantics on top of these
properties and leaves every one of them as base AGTP defines it.

# Transport Considerations

Base AGTP runs over TCP with TLS 1.3 or over QUIC, as specified in
{{AGTP-BINDINGS}}. AGTP-SESSION inherits these transport options.

QUIC is a particularly natural fit for persistent sessions because of
its connection migration capability {{RFC9000}}, which allows a
logical connection to survive IP address changes and network path
changes without renegotiation. AGTP-SESSION over QUIC can leverage
QUIC's connection migration to maintain a persistent session across
network transitions transparently. Connection migration is an
optimization: when it succeeds no RESUME is needed, and when the QUIC
connection is lost outright the resumption procedure of
{{resumption}} applies as over TCP.

# Relationship to MOQT and Real-Time Media

The charter discussion that produced the IETF agentproto working
group anticipated a session protocol on WebTransport or MOQ as a
foundational building block for agent communication. That work and
AGTP-SESSION reflect inverse architectural commitments.

The agentproto approach treats session and transport semantics as
the foundation, with agent-specific concerns (identity, authority,
attribution) layered on top by the protocols that compose on the
session. Under this model, the session protocol carries agent
traffic without knowing it; agent semantics are an application-layer
concern.

AGTP-SESSION reflects the inverse: agent-specific concerns are the
foundation (in base AGTP), and session semantics layer on top of an
agent-aware substrate. Under this model, a session is, by definition,
an agent session. The protocol carries identity, authority, and
attribution at the wire level for every method invocation within the
session, because the substrate beneath the session already commits to
carrying agent traffic structurally.

The two architectures can coexist in deployment. MOQT-style
relay-based pub/sub patterns and real-time media streaming address
use cases (particularly voice with sub-100ms barge-time interruption)
that sit outside AGTP-SESSION. {{AGTP-COMMUNICATION}} is the companion
that builds on ESTABLISH for real-time multi-modal communication
between agents; composition with MOQT-style transport for real-time
agent media, or MOQT-style sessions carrying AGTP semantic envelopes,
remains future work.

The architectural choice between these two foundations is a question
for community discussion. Both deliver session semantics; they differ
on which layer commits to knowing the traffic is agent traffic.

# Application Layering: Voice and Multi-Modal Agents

AGTP-SESSION focuses on session semantics for agent communication.
Protocols for real-time voice, video, or multi-modal data exchange
are specified in {{AGTP-COMMUNICATION}}.

Voice agents, video agents, and multi-modal agents are application-
layer constructs that can compose on top of AGTP and AGTP-SESSION.
At the protocol layer they are the same kind of agent; they are
agents that produce or consume specific media types.

This raises a related architectural question worth surfacing for
community discussion: AGTP may benefit from an agent type system,
analogous to HTTP Content-Type, that allows the protocol to identify
the kind of agent or kind of agent payload involved in a given
exchange. This sits outside AGTP-SESSION as currently scoped and is
flagged here as adjacent work that may warrant its own companion
specification.

# Out of Scope for v01

The following are deliberately out of scope for this version:

- Real-time voice and barge-time interruption semantics
  ({{AGTP-COMMUNICATION}})
- Pub/sub and relay-based session patterns (MOQT-style multicast)
- Session migration across organizations or governance platforms
- Session state synchronization across multiple agents in a chain
- Multi-party sessions (more than two participants)
- Session encryption beyond what TLS 1.3 or QUIC already provides

These are flagged as future work or as open questions to be resolved
through community input.

# Security Considerations

Base AGTP security considerations apply to every method invocation
within a session, including invocations occurring under bounded or
persistent session models. This section identifies session-specific
risks and mitigations that supplement the base AGTP threat model.

## Session Identifier Theft and Replay

Threat: An attacker who obtains a session identifier may attempt to
inject method invocations into an active session by replaying or
forging requests carrying the identifier.

Mitigation: Session identifiers are bound at establishment to the
participating Agent-IDs by the session token ({{session-token}}),
and every invocation within a session is bound to its live sender by
`Request-Signature` ({{AGTP}}). A session identifier in isolation
**MUST NOT** be sufficient to authorize a method invocation: an
attacker holding the identifier and the token still lacks the
initiator's signing key, and an attacker holding a captured request
is stopped by the signature's timestamp and nonce. The receiving
party **MUST** reject invocations whose `Agent-ID` is outside the
token's two participants or whose signature fails. Transport channel
binding is permitted as an additional per-connection check and is
never what a session's continuity depends on, since the transport is
what resumption survives.

## Persistent Session Compromise

Threat: A persistent session that survives transport disconnection
and reconnection has a longer attack window than a bounded session.
An attacker who compromises a participant's signing key during the
session lifetime can impersonate the participant for the remainder
of the session.

Mitigation: Persistent sessions **SHOULD** require periodic
Authority-Scope re-validation against the establishing governance
platform. Recommended re-validation interval is implementation-
defined but **SHOULD NOT** exceed the lifetime of the underlying
Authority-Scope grants. Persistent sessions **MUST** support
revocation propagation: if a participant's Agent Genesis or Agent
Certificate is revoked during the session, the session **MUST**
terminate with appropriate status code on the next invocation.
Implementations **SHOULD** subscribe to revocation notifications
via mechanisms defined in {{AGTP-CERT}} or {{AGTP-LOG}}.

## Cross-Session Correlation

Threat: An adversary observing multiple sessions from the same agent
may be able to correlate sessions to construct a profile of agent
behavior across contexts, even if individual session contents are
encrypted.

Mitigation: Session identifiers **MUST NOT** be derived from
participant Agent-IDs in a way that enables cross-session linkage.
Session identifiers **SHOULD** be cryptographically random and
unique per session. Implementations supporting privacy-sensitive
deployments **SHOULD** consider session identifier rotation policies
that limit linkability, with the trade-off that rotation complicates
revocation propagation.

## Authority Constraint Violation Mid-Session

Threat: An agent participating in an established session may attempt
to invoke methods that exceed session-scoped Authority-Scope grants,
either through scope drift across multiple invocations or through
deliberate scope expansion attempts.

Mitigation: Authority-Scope is enforced on every invocation
regardless of session context. A scope violation **MUST** trigger
455 Scope Violation per base AGTP. Receiving parties **MAY** elect
to terminate the session on first scope violation rather than
continuing to accept further invocations. Termination on scope
violation is **RECOMMENDED** for bounded sessions where the violation
suggests the session boundary has been compromised.

## QUIC 0-RTT Resumption Considerations

Threat: When AGTP-SESSION runs over QUIC, 0-RTT connection resumption
allows initial method invocations on a resumed connection to be
processed before full handshake completion. Replay of 0-RTT data is
a known QUIC consideration.

Mitigation: Method invocations transmitted in 0-RTT data **MUST NOT**
be permitted to ESTABLISH, RESUME, or TERMINATE a session; all three
are action methods. Session establishment and resumption **MUST**
occur after the QUIC handshake has completed. Invocations
within an existing session **MAY** use 0-RTT only for methods on the
early-data eligibility list in the replay-safety profile of
{{AGTP-BINDINGS}}; action methods are excluded there and that
exclusion applies unchanged inside sessions. Implementations
**SHOULD** consult QUIC 0-RTT replay guidance ({{RFC9001}} Section
9.2) when configuring session resumption behavior.

## Graceful Termination and Revocation

Threat: A session may need to be terminated mid-flow due to
revocation, scope violation, time expiry, or governance policy
changes. Improper termination handling can leave participants in
inconsistent state.

Mitigation: TERMINATE ({{establish}} and following) produces a final
Attribution-Record signed by the terminating party identifying the
termination reason. Mid-session revocation of
either participant's Agent Genesis or Agent Certificate **MUST**
result in session termination. Time-based expiry of bounded sessions
**MUST** be enforced against the establishing governance platform's
authoritative time rather than participant-local clocks.

# IANA Considerations {#iana}

This document requests registration of three methods in the AGTP
Method Registry maintained per {{AGTP}}, with this document as the
reference:

| Method | Group | Safe | Idempotent | Section |
|---|---|---|---|---|
| ESTABLISH | Mechanics | No | No | {{establish}} |
| RESUME | Mechanics | No | No | {{resumption}} |
| TERMINATE | Mechanics | No | No | {{establish}} |
{: title="Session Methods"}

It also registers the reason tokens `session-participant-mismatch`,
`session-token-invalid`, `session-expired`, `session-terminated`, and
`session-unknown` in the AGTP reason vocabulary for status codes 401
and 410.

This document establishes the AGTP ESTABLISH Body Extension
Registry ({{body-extensions}}), registration policy Specification
Required, with one initial entry:

| Key | Reference |
|---|---|
| communication | {{AGTP-COMMUNICATION}} |
{: title="ESTABLISH Body Extensions"}

--- back

# Open Questions

- Should the session root be carried as a periodic checkpoint on
  responses within the session, and at what interval, rather than
  only on RESUME and TERMINATE?
- How are persistent sessions migrated across governance platform
  changes (e.g., merchant changes governance providers mid-session)?
- Should AGTP define an agent type system (Agent Content-Type)
  separately, or as part of session establishment?
- What is the right composition story with MOQT for real-time agent
  media use cases?
- Should multi-party sessions (more than two participants) be in
  scope for a later revision, or deferred to a separate companion?
- Should a responder be permitted to initiate RESUME toward an
  initiator that has a reachable attachment, for responder-side
  transport loss?

# Changes from v00

This revision closes the three gaps conceded on the agentproto list
in August and refreshes references and terminology. Wire format
compatibility: session-00 specified no wire format, so nothing
shipped changes.

- Session identifier specified: 256-bit random, 64 lowercase hex,
  responder-assigned, uncorrelated with Agent-IDs
- ESTABLISH body extensions ({{body-extensions}}) and their
  registry, so companions such as {{AGTP-COMMUNICATION}} negotiate
  through the one ESTABLISH grammar rather than through headers
  ({{session-identifier}}).
- ESTABLISH wire format specified, with RESUME and TERMINATE added
  and all three registered ({{establish}}, {{resumption}}, {{iana}}).
- Re-binding after transport loss specified: the session is bound to
  its participants by a responder-signed session token and to the
  live sender by the base per-request signature; transport channel
  binding is demoted to an optional per-connection check, since the
  transport is what resumption survives ({{session-token}},
  {{resumption}}).
- Session root specified ({{session-root}}): a 32-byte rolling
  commitment over the session's Audit-IDs, carried on RESUME for
  divergence detection and on TERMINATE so the final record commits
  to the whole session. This answers the session-level attribution
  question left open in v00 and follows the rolling-commitment
  pattern of {{HCTP}}.
- Base reference moved to draft-hood-independent-agtp-10, which
  supplies `Request-Signature` and `agent_public_key`.

- References updated: base AGTP to draft-hood-independent-agtp-09,
  AGTP-CERT to -03, AGTP-LOG to -02. Informative references added
  for {{AGTP-BINDINGS}} and {{AGTP-COMMUNICATION}}.
- Status code for scope violation at ESTABLISH corrected from 451 to
  455, matching the base renumbering.
- `Principal-ID` references replaced. Base AGTP records Owner-ID in
  the agent identity document referenced by Agent-ID rather than as
  a request header; the ESTABLISH field list and the inheritance
  section now say so.
- QUIC 0-RTT guidance defers to the replay-safety profile and
  early-data eligibility list in {{AGTP-BINDINGS}} rather than
  restating the rule.
- Real-time voice, video, and multi-modal exchange now points to
  {{AGTP-COMMUNICATION}}, which builds on ESTABLISH.
- Working group reference updated from the acp charter proposal to
  the agentproto working group.
- Editorial pass for house style. No normative change beyond the
  status code correction.

# Contributors

This draft is informed by the IETF agent2agent charter formation
discussion and by feedback on the architectural layering question
raised in that thread.
