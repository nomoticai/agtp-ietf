---
title: "Applicability of the Agent Transfer Protocol to agentproto Dialog Management"
abbrev: "AGTP for agentproto"
docname: draft-hood-agentproto-agtp-applicability-00
category: info
submissiontype: IETF
ipr: trust200902
area: "Applications and Real-Time"
keyword:
  - AI agents
  - agentproto
  - dialog management
  - transport bindings

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

informative:
  AGENTPROTO-CHARTER:
    title: "Agent Communication Protocols (agentproto) Charter"
    author:
      org: IETF
    target: https://datatracker.ietf.org/group/agentproto/about/
    date: 2026
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026
  AGTP-SESSION:
    title: "AGTP Session Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-session-01
    date: 2026
  AGTP-BINDINGS:
    title: "AGTP Transport Bindings"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-bindings-01
    date: 2026
  AGTP-IDENTIFIERS:
    title: "AGTP Identifier Stack and Attribution-Record"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identifiers-03
    date: 2026
  AGTP-COMPOSITION:
    title: "AGTP Composition Profiles"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-composition-02
    date: 2026
  AGTP-COMMUNICATION:
    title: "AGTP Communication Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-communication-01
    date: 2026
  AGTP-LOG:
    title: "AGTP Transparency Log Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-03
    date: 2026
  RFC9220:
  RFC8470:

--- abstract

The IETF agentproto working group is chartered to define a baseline
protocol for managing agentic dialogs: correlated interactions among
users, AI agents, and tools over the course of a task, with dialog
identifiers, lifecycle states, context propagation across
intermediaries and trust boundaries, and bindings over existing IETF
transports. The Agent Transfer Protocol (AGTP) is an individual
submission that specifies a session layer, a transport binding set,
and a header set carrying agent identity and dialog context at the
wire level. This document places the two side by side. For each item
the charter names, it states what the AGTP drafts specify and where,
and it lists the charter items the AGTP drafts leave open. It
proposes no new mechanism.

--- middle

# Introduction

The agentproto working group was chartered in October 2026
{{AGENTPROTO-CHARTER}}. Its standards-track deliverable is a dialog
management protocol; its informational deliverables are a reference
architecture and a use cases and requirements document. The charter
states that the protocol "neither replaces application-layer agent
communication protocols nor defines a new transport protocol" and
that it is carried "over one or more existing IETF protocols, such as
HTTP, QUIC, WebTransport, WebRTC or MOQ".

The Agent Transfer Protocol (AGTP) {{AGTP}} is an individual
submission with sixteen companion drafts. Three of them cover the
ground the agentproto charter describes: {{AGTP-SESSION}} specifies
session establishment, identification, resumption, and termination;
{{AGTP-BINDINGS}} specifies how AGTP runs over TCP with TLS, over
QUIC, and over HTTP/3 with WebTransport; and the base document
specifies the headers that carry identity and dialog context on
every request.

This document is an applicability statement. {{charter-mapping}}
takes the charter's work items in order and states, for each, what
the AGTP drafts specify. {{headers}} lists the dialog facts the
charter requires to propagate and the AGTP header that carries each
one. {{open}} lists the charter items that the AGTP drafts cover
partially or leave to other work. The document adds no normative
text of its own; every mechanism it names is specified in the draft
it cites.

## Terminology

This document uses the agentproto charter's term "dialog" for the
correlated set of interactions the charter describes, and the AGTP
drafts' term "session" for the construct {{AGTP-SESSION}} specifies.
{{charter-mapping}} states how the two relate. "Connection" means a
transport-level association (a TLS connection, a QUIC connection, or
a WebTransport session), following the usage in {{AGTP-BINDINGS}}.

# Charter Items and the AGTP Drafts {#charter-mapping}

Each entry quotes or paraphrases a charter item, then states what
the AGTP drafts specify and where.

Baseline protocol for managing agentic dialogs (correlated
interactions among users, AI agents, and tools over a task):
: A session, explicitly established between two agents, under which
  requests are correlated; bounded (fixed lifetime) and persistent
  (long-lived, resumable) models. {{AGTP-SESSION}}, Session Models.

Dialog identifiers:
: `Session-ID`, 256 bits of responder-generated randomness as 64
  lowercase hex characters, on every request in the session,
  uncorrelated with any Agent-ID. {{AGTP-SESSION}}, Session
  Identifier. `Task-ID` groups the requests of one task within or
  across sessions. {{AGTP}}, Request Headers.

Lifecycle from establishment through revocation:
: ESTABLISH creates the session and returns a responder-signed
  token; TERMINATE ends it with a signed final record; a terminated
  or expired token is refused with `410` and a reason code;
  revocation of either participant's Genesis or certificate
  mid-session terminates the session. {{AGTP-SESSION}}, Session
  Establishment; Graceful Termination and Revocation.

Propagating dialog context across intermediaries and trust
boundaries:
: `Session-ID`, `Task-ID`, `Agent-ID`, `Authority-Scope`, and
  `Delegation-Chain` are request headers an intermediary reads
  without payload access, and the precedence rule makes them
  authoritative over any equivalent field in a carried payload.
  {{AGTP}}, Request Headers; Precedence Rule. See {{headers}}.

Recovering from path or intermediary failures:
: RESUME re-attaches a session to a new connection within a
  `resume_window` negotiated at ESTABLISH; both parties fold a
  session root, a rolling hash over the session's responses, so a
  divergent history is detected at RESUME. {{AGTP-SESSION}},
  Resumption After Transport Loss; Session Root.

Transport bindings over existing IETF protocols:
: Bindings for TCP with TLS 1.3, for QUIC, and for HTTP/3 with
  WebTransport using extended CONNECT per {{RFC9220}}; the session
  is independent of the connection carrying it. {{AGTP-BINDINGS}}.

Privacy risks from persistent dialog identifiers:
: The identifier is random and responder-assigned, so two sessions
  of one agent share no identifier material; the threat and its
  mitigations are stated. {{AGTP-SESSION}}, Cross-Session
  Correlation.

Agent identity distinct from user identity; agents authenticated
independently of the users they represent:
: `Agent-ID` names the agent and `Request-Signature` proves per
  request that the sender holds the agent's key; the user or
  principal is a validated external credential in `Authorization`,
  recorded as `principal_id`. The two are specified as independent
  axes. {{AGTP}}, Request Signature and Proof of Possession;
  {{AGTP-COMPOSITION}}, External Identity Provider Composition.

Long-lived and low-latency dialogs:
: Persistent sessions with resumption; on the QUIC and HTTP/3
  bindings, 0-RTT is permitted for QUERY and DISCOVER and refused
  for every other method per {{RFC8470}}. {{AGTP-BINDINGS}}, Replay
  Safety Profile.

Security and privacy over untrusted networks; reuse of existing
identity, authentication, authorization, and encryption building
blocks:
: TLS 1.3 on every binding; JWS (RFC 7515) for signatures and
  tokens; OAuth and OIDC credentials in `Authorization`; a mapping
  to the WIMSE delegation convention; SCITT-aligned receipts for log
  inclusion. {{AGTP}}; {{AGTP-COMPOSITION}}; {{AGTP-LOG}}.

Real-time media (WebRTC and MoQ named among transports):
: A companion specifies audio, video, and data streams established
  through the same session layer, carried on the QUIC and
  WebTransport bindings. {{AGTP-COMMUNICATION}}.

Two charter statements bear on how AGTP relates to the group's
deliverable. The charter protocol is carried over existing IETF
transports and defines no new one; AGTP is a transfer protocol that
runs over TCP, QUIC, and HTTP/3, and {{AGTP-BINDINGS}} specifies
those three as its bindings. The charter lists audit data structures
as out of scope; the AGTP drafts specify an Attribution-Record and a
transparency log in {{AGTP-IDENTIFIERS}} and {{AGTP-LOG}}, which this
document therefore cites only where the charter's security and
privacy items touch them.

# Dialog Context on the Wire {#headers}

The charter requires dialog context to propagate across
intermediaries and trust boundaries. When the dialog protocol is
carried over HTTP, each item of context an intermediary must read
without payload access requires an HTTP header field, existing or
newly registered. The AGTP drafts carry each item as a header of the
transfer protocol itself.

| Dialog fact to propagate | AGTP request header |
|---|---|
| Which dialog this request belongs to | `Session-ID` ({{AGTP-SESSION}}) |
| Which task within the dialog | `Task-ID` ({{AGTP}}) |
| Which agent is sending | `Agent-ID`, the SHA-256 of the agent's Genesis document ({{AGTP}}) |
| Proof the sender is that agent | `Request-Signature`, a JWS over method, path, Agent-ID, scope, delegation digest, body digest, timestamp, and nonce ({{AGTP}}) |
| What the sender may do | `Authority-Scope` ({{AGTP}}) |
| On whose delegated authority, through which agents | `Delegation-Chain`, ordered hops of parent Agent-ID and parent-signed token ({{AGTP}}) |
| On whose behalf, as user or principal | `Authorization`, an external credential whose validated claim is recorded as `principal_id` ({{AGTP-COMPOSITION}}) |
| What the request may consume | `Budget-Limit` ({{AGTP}}; specified in the commerce companion) |
| Per-request correlation | `Request-ID`; `Response-ID` and `Audit-ID` on the response ({{AGTP-IDENTIFIERS}}) |
{: title="Dialog context and the AGTP header that carries it"}

Each header is parsed by the protocol before the application sees
the request, so an intermediary on the path reads it without parsing
the payload.

# Relationship Between a Dialog and a Session

The charter's dialog spans users, agents, and tools over a task. The
AGTP session is bilateral, between two agents. A task that spans
several agents is correlated by `Task-ID` across the sessions it
touches, and a task delegated from one agent to another carries the
delegating agent's authority in `Delegation-Chain`. A dialog in the
charter's sense maps onto one or more AGTP sessions sharing a
`Task-ID`, with the delegation sequence recording how the task passed
between agents.

Tools appear in the AGTP drafts either as agents, when the tool
holds an Agent-ID and is reached over AGTP, or as application
payloads carried by the EXECUTE method under a composition profile
({{AGTP-COMPOSITION}} specifies the MCP profile), where the tool call
inherits the request's session, identity, and scope headers. Users
appear as principals: the owner recorded on the agent's Genesis
document and the per-request principal lifted from an external
credential. The parties to a session are agents; a user acts through
one.

# Charter Items Open in the AGTP Drafts {#open}

The following charter items are covered partially or left to other
work in the AGTP drafts as of the revisions cited.

- **WebRTC and MoQ bindings.** {{AGTP-BINDINGS}} specifies TCP with
  TLS, QUIC, and HTTP/3 with WebTransport. {{AGTP-SESSION}} names
  composition with MoQ-style transport as future work, and
  {{AGTP-COMMUNICATION}} lists the relationship to WebRTC among its
  open questions. No binding to either is specified.
- **Multi-party dialogs.** The AGTP session is bilateral.
  Multi-party coordination is reached through COLLABORATE and
  through `Task-ID` correlation across sessions; a single session
  with more than two participants is unspecified.
- **Dialog context schema.** The charter's deliverable includes how
  dialog context propagates. AGTP carries the identifiers and
  authority facts listed in {{headers}}; it specifies no schema for
  application-level dialog state (conversation history, turn
  structure), which it treats as payload.
- **Intermediary failure semantics beyond resumption.** RESUME
  covers loss of the connection. Failure of an intermediary that
  holds session state on behalf of a party, and recovery of a
  dialog through a different intermediary, are unspecified.
- **Use cases document.** The AGTP drafts carry worked examples in
  each companion and no single use cases and requirements document
  of the kind the charter calls for.
- **Terminology.** The charter's terminology work and the AGTP
  drafts' terminology (session, Agent-ID, Authority-Scope,
  attachment) were developed separately. Where the group's
  reference architecture defines terms for the same constructs,
  the AGTP drafts can adopt them.

# Security Considerations

This document specifies no protocol and introduces no security
considerations of its own. The security considerations of
{{AGTP}}, {{AGTP-SESSION}}, and {{AGTP-BINDINGS}} apply to the
mechanisms named here, and the charter's privacy item on persistent
dialog identifiers is addressed in the Cross-Session Correlation
section of {{AGTP-SESSION}}.

# IANA Considerations

This document has no IANA actions. The headers named in {{headers}}
are registered by the documents cited in that table.

--- back

# Acknowledgments

The agentproto charter text quoted here is the work of the group's
chairs, Leslie Daigle and Orie Steele, the responsible Area
Director, Charles Eckel, and the participants in the BoF and
chartering discussions.
