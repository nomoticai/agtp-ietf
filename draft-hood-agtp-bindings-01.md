---
title: "AGTP Transport Bindings: TCP/TLS, QUIC, and HTTP/3"
abbrev: "AGTP-BINDINGS"
docname: draft-hood-agtp-bindings-01
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - AGTP
  - transport bindings
  - QUIC
  - TLS
  - HTTP/3
  - WebTransport
  - replay safety

stand_alone: yes
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  compact: yes

author:
  - fullname: Chris Hood
    organization: Nomotic, Inc.
    email: chris@nomotic.ai
    uri: https://nomotic.ai

normative:
  RFC2119:
  RFC8174:
  RFC8446:
  RFC9000:
  RFC9001:
  RFC9002:
  RFC6066:
  RFC5280:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026

informative:
  RFC8470:
  RFC9110:
  RFC9114:
  RFC9220:
  AGTP-METHODS:
    title: "AGTP Methods (AGTP Verbs): Method Vocabulary, Path Grammar, and Endpoints"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-methods-00
    date: 2026
  AGTP-SESSION:
    title: "AGTP Session Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-session-01
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

--- abstract

The Agent Transfer Protocol (AGTP) is a transfer protocol whose
semantics are defined independently of any transport. This document
specifies three transport bindings: TCP with TLS 1.3, QUIC, and
HTTP/3 (with WebTransport where a browser or an HTTP intermediary is
in the path). Each binding defines how AGTP requests and responses
are carried, how TLS mutual authentication composes with AGTP
identity, and how the transport's connection lifecycle relates to the
agent-level session defined in the AGTP session companion, which a
connection carries and never is.

A central concern for the QUIC binding is replay safety. QUIC supports
0-RTT early data, which is replayable by design. AGTP requests carry
authority-scoped actions; a replayed action method could re-trigger
authorized work without the requesting agent's intent. This document
defines an AGTP-specific replay-safety profile: early data is permitted
only for the two safe idempotent methods whose replay has no
observable effect (QUERY, DISCOVER), **MUST NOT** be used for action
methods (EXECUTE, DELEGATE, PURCHASE, TRANSACT, the lifecycle methods,
and the session methods), and always carries the per-request
signature that bounds replay of even an eligible method.

This document follows the pattern established by HTTP: protocol
semantics defined independently of transport ({{RFC9110}} for HTTP),
with transport-specific bindings in separate documents. AGTP semantics
are defined in {{AGTP}}; this document defines the transport
mechanics.

--- middle

# Introduction

The Agent Transfer Protocol {{AGTP}} is a transfer protocol for AI
agent traffic: it defines the semantics of an exchange, and the
transport delivers the bytes. The base specification defines AGTP
methods, headers, status codes, identity model, and the wire format
for requests and responses. The base specification is
transport-neutral by design: AGTP semantics are independent of which transport
carries them.

This document specifies three transport bindings: TCP with TLS 1.3,
QUIC, and HTTP/3 with WebTransport. The first two are implemented;
the third is specified so that AGTP can be carried where an HTTP
intermediary or a browser is in the path, and so that the question
raised on the IETF agentproto list, whether HTTP/3 or WebTransport
can carry the AGTP session goal, has a written answer. Each binding
defines:

- How AGTP requests and responses are framed on the transport.
- How the transport's connection lifecycle relates to the agent-level
  session of {{AGTP-SESSION}}.
- How TLS mutual authentication composes with AGTP identity per
  {{AGTP-CERT}}.
- Any transport-specific security and operational considerations.

Bindings to other transports may be defined in separate documents
that follow the same pattern as this one.

## Why Transport Bindings Are Separate

The base AGTP specification separates protocol semantics
from transport mechanics. This separation has three benefits:

**Transport neutrality.** AGTP semantics stay the same when the
transport changes. An agent invoking QUERY against another agent
operates the same way whether the connection runs over TCP/TLS or
QUIC. The base specification can be reviewed and reasoned about
without entangling transport-specific concerns.

**Independent evolution.** Transport technology evolves on its own
timeline. The TCP/TLS binding can be refined as TLS evolves. The
QUIC binding can be refined as QUIC evolves. The base AGTP
specification has no need to track these changes.

**Clear interoperability requirements.** Implementers know that
supporting "AGTP" means supporting the base specification plus at
least one transport binding. Different implementations may support
different bindings (one supports both, another supports only
TCP/TLS); interoperability is determined at the binding boundary.

This pattern mirrors HTTP, where semantics live in {{RFC9110}} and
transport-specific bindings live in separate RFCs (HTTP/1.1 in
RFC 9112, HTTP/2 in RFC 9113, HTTP/3 in RFC 9114).

## Conventions and Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

This document uses terminology from {{AGTP}} and {{AGTP-CERT}}.
Selected additional terms:

Transport Binding:
: A specification defining how AGTP requests and responses are
  carried over a specific transport. This document defines bindings
  for TCP/TLS, QUIC, and HTTP/3.

Connection:
: What a transport binding provides: a TCP/TLS connection, a QUIC
  connection, or an HTTP/3 connection or WebTransport session. A
  connection has the lifecycle the transport gives it and carries
  zero or more AGTP sessions.

Session:
: The agent-level context of {{AGTP-SESSION}}, identified by
  `Session-ID`, bound to its two participants by a signed session
  token and to the live sender of every request by `Request-Signature`
  ({{AGTP}}). A session may span several connections over its
  lifetime and is never identified with any one of them. In this
  document "session" means this; the transport's own notion
  (a TLS session or a WebTransport session) is called a connection
  throughout.

Safe Idempotent Method:
: An AGTP method whose invocation has no observable side effects on
  agent state and whose repeated invocation produces equivalent
  results. The safe property is defined in {{AGTP-METHODS}}; QUERY,
  DISCOVER, DESCRIBE, INSPECT, SUMMARIZE, and PLAN are safe per
  {{AGTP}}. This document admits a narrower set to early data
  ({{eligible}}).

Action Method:
: An AGTP method whose invocation produces observable side effects.
  EXECUTE, DELEGATE, PURCHASE, TRANSACT, ACTIVATE, DEACTIVATE,
  REVOKE, REINSTATE, DEPRECATE, ESTABLISH, RESUME, TERMINATE, and
  similar methods are action methods per {{AGTP}} and
  {{AGTP-SESSION}}. Replaying an action method may produce
  duplicate effects.

Early Data:
: Application data sent before the TLS handshake completes, supported
  by TLS 1.3 ({{RFC8446}}) and QUIC ({{RFC9001}}). Early data is
  replayable by design; an attacker who captures early data may
  replay it.

# AGTP Identity Across the Bindings

Every binding carries AGTP identity in two complementary ways:

1. **At the TLS layer** through mutual authentication using AGTP
   Agent Certificates per {{AGTP-CERT}}, when the agent certificate
   extension is deployed.

2. **At the AGTP wire layer** through the `Agent-ID` and
   `Authority-Scope` headers and the `Request-Signature` header
   carried on every non-anonymous request per {{AGTP}}, with Owner-ID
   resolved from the Identity Document the Agent-ID references
   ({{AGTP-IDDOC}}).

The two layers compose. The certificate (when present) provides
cryptographic identity at the TLS layer that infrastructure
components (load balancers, SEPs, governance gateways) can verify
without parsing AGTP headers. The signature provides proof, on every
request and on every binding, that the sender holds the agent's
current key, so AGTP identity is the same on every binding, with or
without a certificate.

Both bindings **MUST** verify that AGTP identity headers, when
combined with a presented Agent Certificate, are consistent. A
mismatch between the certificate's `agent-id` extension and the
request's Agent-ID header **MUST** be treated as a protocol error per
{{AGTP-CERT}}.

# TCP/TLS Binding {#tcp-tls}

## Overview

AGTP over TCP/TLS runs on a persistent TCP connection wrapped by TLS
1.3 or later. A connection carries zero or more AGTP sessions; a
session may move to a new connection through RESUME
({{AGTP-SESSION}}). AGTP requests and responses are sequential within
the connection: a server processes requests in the order received and
returns responses in the same order.

This binding is the baseline AGTP transport. Implementations **MUST**
support TCP/TLS as the universal fallback. Implementations **MAY**
additionally support the QUIC binding per {{quic-binding}}.

## TLS Requirements

All AGTP TCP connections **MUST** use TLS 1.3 ({{RFC8446}}) or higher.
Implementations **MUST** reject connections using TLS 1.2 or below.

TLS-level mutual authentication **MAY** be required by the server per
{{AGTP-CERT}}. When mutual authentication is required, both the
client (calling agent) and the server (target agent) present
certificates. The server's certificate **MUST** be valid per
{{RFC5280}}. The client's certificate, when presented, **SHOULD** be
an AGTP Agent Certificate per {{AGTP-CERT}} for high-assurance
deployments.

Server Name Indication ({{RFC6066}}) **SHOULD** be used by clients to
allow servers to present the correct certificate when multiple agents
share a serving infrastructure. The SNI value carries the agent's
authority hostname per the URI structure in {{AGTP}}.

## Connection Establishment

The TCP/TLS connection establishment sequence:

1. The client establishes a TCP connection to the target server on
   port 4480 (or the configured port).

2. The client initiates a TLS 1.3 handshake. Server name indication
   carries the target agent's authority hostname.

3. If mutual authentication is required, the client presents its
   Agent Certificate during the handshake. The server verifies the
   certificate against its trust anchors per {{RFC5280}}.

4. The server presents its certificate. The client verifies the
   certificate per {{RFC5280}}.

5. Upon successful handshake, the connection is ready to carry AGTP
   requests and responses.

Implementations **MUST NOT** use TLS False Start or any early-data
mechanism over TCP/TLS. AGTP over TCP/TLS carries no 0-RTT
data; all AGTP messages **MUST** be sent only after the TLS handshake
completes.

## Request/Response Framing

AGTP requests and responses over TCP/TLS use the wire format defined
in {{AGTP}}. Each message consists of:

1. A request line (for requests) or response line (for responses)
   terminated by CRLF.

2. Zero or more header lines, each terminated by CRLF.

3. An empty line (CRLF only) signaling the end of headers.

4. A message body whose length is specified by the `Content-Length`
   header.

The `Content-Length` header is the sole signal of message completion
per {{AGTP}}. Receivers **MUST** treat a message as complete when,
and only when, the declared number of body octets has been read after
the header terminator.

AGTP over TCP/TLS **MUST NOT** use HTTP-style chunked transfer
encoding. Streaming responses are framed by repeated `Content-Length`-
delimited messages within a single connection per {{AGTP}}.

## Connection Lifecycle and Sessions {#tcp-lifecycle}

A TCP/TLS connection begins with the completion of the TLS handshake
and ends with either:

- Inactivity timeout (default 60 seconds, configurable).
- TCP connection close, by either side.

Connection close is never session termination. A session established
on the connection ({{AGTP-SESSION}}) persists in the
`Suspended-by-transport` state for its resume window and is re-bound
to a new connection by RESUME; it ends only by TERMINATE, by expiry,
or by the resume window elapsing. A connection that closes with
sessions open **SHOULD** be logged by the server with the affected
`Session-ID` values so that a RESUME that never arrives can be
reconciled against the log.

Implementations **MUST NOT** use socket-level half-close
(`shutdown(SHUT_WR)` or equivalent) to signal end-of-request per the
Wire-Format Framing section of {{AGTP}}. The TLS `close_notify` alert
that results from a half-close terminates the secure session before
the peer can transmit a response, producing a truncation that is
indistinguishable at the application layer from a malicious
downgrade.

When a connection closes normally, the TLS connection terminates per
{{RFC8446}}. The TCP connection then closes.

## Connection Reuse and Pooling

A single TCP/TLS connection **MAY** carry several AGTP sessions,
sequentially or interleaved by `Session-ID`. The binding keeps no
per-session state of its own: which session a request belongs to is
the `Session-ID` header, and the server **MUST** check that the
request's `Agent-ID` is a participant in that session per
{{AGTP-SESSION}}; the connection the request arrived on says
nothing about membership.

Connection pools **MAY** maintain pre-established connections to
frequent counterparties to reduce handshake overhead. Pooled
connections **MUST** be validated as live before reuse (typically by
sending a low-cost AGTP DISCOVER request).

# QUIC Binding {#quic-binding}

## Overview

AGTP over QUIC runs on QUIC connections per {{RFC9000}}, with QUIC's
built-in TLS 1.3 per {{RFC9001}}. QUIC provides multiplexed streams,
reduced connection establishment latency, and connection migration.

The QUIC binding is **OPTIONAL** for AGTP implementations.
Implementations **MAY** support QUIC in addition to TCP/TLS.
Implementations **MUST NOT** support QUIC without also supporting
TCP/TLS.

## QUIC Version

AGTP over QUIC **MUST** use QUIC version 1 ({{RFC9000}}) or higher.
Negotiation of higher QUIC versions follows the QUIC version
negotiation procedure ({{RFC9000}} Section 6).

## ALPN Identifier

AGTP over QUIC **MUST** negotiate the application-layer protocol
using ALPN per {{RFC9001}}. The ALPN identifier for AGTP over QUIC
is `agtp`. Implementations **MUST** include `agtp` in the
ClientHello's ALPN extension and **MUST** select `agtp` in the
ServerHello when offered.

## Stream Usage

AGTP requests and responses over QUIC are carried on bidirectional
client-initiated streams. Each AGTP request initiates a new stream;
the response is carried on the same stream.

This document defines no multi-stream coordination beyond what
{{AGTP-SESSION}} specifies. A single AGTP session **MAY** carry
multiple concurrent requests on separate QUIC streams, and a single
QUIC connection **MAY** carry several sessions. The `Session-ID`
header identifies the session across streams and connections.

QUIC connection-level concerns (flow control, congestion control,
loss recovery per {{RFC9002}}) are handled by the QUIC implementation
and are transparent to AGTP. AGTP implementations **MUST NOT** rely
on stream-level acknowledgment or loss-recovery semantics.

## Connection Migration

QUIC supports connection migration: a connection can move between
network paths (e.g., from Wi-Fi to cellular) without disruption per
{{RFC9000}} Section 9. AGTP over QUIC inherits this property.

A migrated QUIC connection is the same connection, so the sessions it
carries continue without a RESUME; migration is the case in which
the transport itself preserves the session's connection. When a QUIC
connection is lost outright (idle timeout, stateless reset, path
failure the transport cannot recover), the resumption procedure of
{{AGTP-SESSION}} applies exactly as over TCP/TLS. Implementations
**SHOULD** treat migration as transparent and **MUST NOT** treat it
as a resumption event.

When a QUIC connection migration changes the apparent client network
location, AGTP implementations **MAY** apply additional verification
(e.g., re-checking the Agent Certificate, requesting attestation per
trust posture per {{AGTP-CERT}}) for sensitive sessions, but this is
operator policy and no protocol requirement.

## QUIC-Specific Replay Safety Profile {#replay-safety}

QUIC supports 0-RTT early data per {{RFC9001}}. Early data is sent
before the QUIC handshake completes. Early data is replayable: an
attacker who captures early data may replay it, and the server
cannot distinguish a replay from the original.

AGTP requests carry authority-scoped actions. A replayed EXECUTE,
DELEGATE, PURCHASE, or other action method could re-trigger
authorized work, transfer funds, modify state, or produce duplicate
side effects.

This document defines an AGTP-specific replay-safety profile
governing early data use.

### Methods Eligible for Early Data {#eligible}

Safety is a property of a method's definition ({{AGTP-METHODS}}).
Admission to early data is a property of a deployment's replay
exposure, and this binding admits a narrower set than the safe
methods of {{AGTP}}. The methods eligible for early data are:

- **QUERY**: information retrieval. Replays return equivalent results
  and touch no agent state.
- **DISCOVER**: capability and presence discovery. Replays return
  equivalent results and touch no agent state.

DESCRIBE, INSPECT, SUMMARIZE, and PLAN are safe per {{AGTP}} and are
excluded here for cause: INSPECT reads audit records whose access
is scoped and logged, SUMMARIZE and PLAN invoke model inference whose
cost a replay multiplies, and DESCRIBE is cheap but gains nothing from
0-RTT a cached result cannot give. Any of them **MAY** become eligible
through an update to this document once implementation evidence shows
replay to be harmless in deployment.

### Request Signature in Early Data {#early-signature}

An early-data request carries `Request-Signature` like any other
request ({{AGTP}}). The server **MUST** verify the signature, the
`iat` window, and the `(Agent-ID, nonce)` replay cache before
honoring the request, even for an eligible method; an early-data
request whose signature fails is rejected with 401 and reason
`possession-unproven` and is never retried as early data. The
signature's nonce bounds replay of an eligible request to
one acceptance per nonce per window, so the eligibility list limits
what a replay can do and the signature limits how often it can do
it. The two defenses compose and neither substitutes for the other.

### Methods Prohibited from Early Data

AGTP action methods **MUST NOT** be sent as early data. The currently
defined action methods prohibited from early data are:

- **EXECUTE**: carries arbitrary agent operations including
  framework payloads (MCP tool calls, A2A tasks, etc.). May have
  arbitrary side effects.
- **DELEGATE**: delegates authority. Creates state.
- **PURCHASE**: commerce action. Moves value.
- **TRANSACT**: commits to commerce transaction per AGTP-COMMERCE.
- **ACTIVATE**, **DEACTIVATE**, **REVOKE**, **REINSTATE**,
  **DEPRECATE**: lifecycle methods. Modify agent governance state.
- **NOTIFY**: delivers messages with at-least-once semantics.
  Replays could produce duplicate delivery.
- **PROPOSE**: proposes runtime contracts. Creates negotiation
  state.
- **ESTABLISH**, **RESUME**, **TERMINATE**: session methods per
  {{AGTP-SESSION}}. Create, re-bind, or end session state; a replayed
  RESUME would re-bind a session to a connection the initiator no
  longer holds.
- **CREATE**, **REPLACE**, **MODIFY**, **REMOVE**: when used as
  resource-affecting methods per the legacy mapping in
  {{AGTP-METHODS}}.
- **CONFIRM**, **ESCALATE**: mechanics methods with external effect.
- **DESCRIBE**, **INSPECT**, **SUMMARIZE**, **PLAN**: safe per
  {{AGTP}} but excluded from early data per {{eligible}}.

Servers receiving an early-data request carrying any method absent from
the eligible list **MUST** reject the request with status code 425
(Too Early) per {{RFC8470}} and **MUST** require the client to
retransmit the request after the QUIC handshake completes.

### Server Enforcement

QUIC implementations supporting AGTP **MUST** make the AGTP method
visible to the server's request dispatcher before the dispatcher
applies any side-effectful logic. The server's AGTP layer **MUST**
verify that any request received as early data carries a method on
the eligibility list, and **MUST** verify its `Request-Signature`
per {{early-signature}}, before processing.

A server **MAY** reject all early data unconditionally, regardless
of method. This is a conforming and conservative implementation
choice. The replay-safety profile defines the maximum permitted
early data; servers **MAY** be more restrictive.

### Client Behavior

Clients sending AGTP requests over QUIC **MUST NOT** send action
methods as early data. Clients that wish to use 0-RTT for latency
reduction **MUST** restrict early-data usage to the eligible methods.

If a client receives a 425 Too Early response, the client **MUST**
retry the request after the handshake completes. The client **MUST
NOT** retry the same request as early data on the same or a future
connection unless the AGTP method becomes eligible (a specification
change this document would document).

### Anti-Replay Defense in Depth

The replay-safety profile is the primary defense against early-data
replay attacks for AGTP. Implementations **SHOULD** also use
QUIC-level anti-replay mechanisms per {{RFC9001}}, including
per-server anti-replay caches that detect duplicate early-data
records. These mechanisms compose with the AGTP replay-safety
profile: even on eligible methods, defense in depth limits
exploitability of any future protocol-level oversight.

## Connection Establishment Sequence

The QUIC connection establishment sequence:

1. The client constructs a QUIC ClientHello including ALPN extension
   advertising `agtp`.

2. The QUIC handshake proceeds per {{RFC9001}}. If the client has a
   valid resumption ticket and the deployment policy allows, the
   client **MAY** send 0-RTT early data subject to the replay-safety
   profile in {{replay-safety}}.

3. If mutual authentication is required, the client presents its
   Agent Certificate during the handshake. The server verifies the
   certificate against its trust anchors per {{RFC5280}} and
   {{AGTP-CERT}}.

4. The server completes the handshake, selecting `agtp` in the ALPN
   extension.

5. Upon successful handshake, the connection is ready to carry
   normal AGTP requests and responses on QUIC streams.

## Connection Lifecycle and Sessions {#quic-lifecycle}

A QUIC connection is ready to carry AGTP requests when the handshake
completes; eligible early data **MAY** precede that point per
{{replay-safety}}. A session **MUST NOT** be established in early
data ({{AGTP-SESSION}}). The connection ends with either:

- Inactivity timeout (default 60 seconds, configurable).
- QUIC connection close or stateless reset.

As over TCP/TLS ({{tcp-lifecycle}}), connection close is never
session termination; the sessions the connection carried enter
`Suspended-by-transport` and are re-bound by RESUME. Connection
migration ends nothing.

# HTTP/3 and WebTransport Binding {#h3-binding}

## Purpose and Status

The TCP/TLS and QUIC bindings present AGTP as a distinct wire image
on its own port. Two deployment situations need AGTP carried inside
an HTTP/3 connection instead: a browser, which can open HTTP/3 and
WebTransport but cannot open a raw QUIC connection to port 4480; and
an HTTP intermediary (a CDN edge, an API gateway, a corporate
egress proxy) that the operator wants AGTP traffic to traverse
without a protocol exception. This binding carries unmodified AGTP
messages over an HTTP/3 connection ({{RFC9114}}), using WebTransport
({{RFC9220}}) where the client is a browser. AGTP semantics are
unchanged: the request line, headers, `Request-Signature`, status
codes, and session model are exactly those of {{AGTP}} and
{{AGTP-SESSION}}; only the framing beneath them differs.

Status: the framing in this section is specified. Interoperability
details marked planned are to be settled against implementation
experience, and the reference implementation has yet to ship this
binding. A server **MAY** omit it and remain conformant; TCP/TLS
remains the universal fallback.

## Framing over an HTTP/3 Connection

An AGTP client that has an HTTP/3 connection to a server
advertising AGTP support opens one bidirectional HTTP/3 stream per
AGTP request and sends the AGTP message on that stream as the body
of a single HTTP/3 request:

~~~~
:method = CONNECT
:protocol = agtp
:scheme = https
:authority = <server authority>
:path = /
~~~~

The extended CONNECT form of {{RFC9220}} is used so that the stream
is handed to AGTP in place of HTTP request interpretation. After
the server answers with `:status = 200`, the stream carries AGTP
messages framed exactly as over TCP/TLS: the AGTP request line,
headers, and `Content-Length`-delimited body in one direction and
the AGTP response in the other. The HTTP/3 layer sees opaque bytes.
Servers **MUST** advertise support with the `SETTINGS_ENABLE_CONNECT_PROTOCOL`
setting per {{RFC9220}}.

A client **MAY** reuse one such stream for several sequential AGTP
exchanges, as over a TCP/TLS connection, or open one stream per
exchange for concurrency. The stream is the AGTP connection for the
purposes of {{tcp-lifecycle}}: its close is never session
termination.

## WebTransport

Where the client is a browser, the client opens a WebTransport
session to the server per {{RFC9220}} and carries each AGTP exchange
on a WebTransport bidirectional stream, framed as above. The
WebTransport session is a connection in this document's vocabulary;
the AGTP session rides on it and survives it through RESUME. A
browser client authenticates as an agent through `Request-Signature`
with a key the page holds; Agent Certificates at the TLS layer are
unavailable to a browser client, so such a deployment is at Level 2
of {{AGTP}} and never Level 3.

## Early Data

HTTP/3 0-RTT applies to the HTTP/3 request that opens the stream,
and the AGTP message inside it is early data if the stream is opened
in 0-RTT. The replay-safety profile of {{replay-safety}} applies
unchanged: only QUERY and DISCOVER **MAY** be sent on a stream opened
in early data, every such request carries `Request-Signature`, and a
server receiving any other method on such a stream rejects it with
425 per {{RFC8470}}.

## Identity and Intermediaries

An HTTP intermediary in the path sees an extended CONNECT to the
server and opaque stream data; it sees no AGTP headers and
cannot act on them. Infrastructure that needs to enforce
Authority-Scope at line rate (an SEP) therefore sits at the AGTP
server, beyond the intermediary, when this binding is in use,
or terminates the CONNECT itself and becomes the AGTP server. The
`Request-Signature` is verified end to end by whichever party is the
AGTP server, so an intermediary cannot forge or alter a request
without detection; it can only refuse to carry it.

Planned: a well-known path for advertising AGTP support on an HTTPS
origin, so that a browser can discover that an origin accepts this
binding before opening a WebTransport session; the ALPN and
`:protocol` token registrations ({{iana}}); and test vectors for the
stream framing.

# Connection Selection Between Bindings

When several bindings are available on the server, the client selects
the binding based on its own operational requirements. AGTP defines
no normative preference among them.

Clients **MAY** use any of the following selection strategies:

- **TCP/TLS only.** Conservative choice. Universal availability.
- **QUIC with TCP/TLS fallback.** Attempt QUIC first; fall back to
  TCP/TLS if the QUIC handshake fails or 0-RTT replay-safety
  considerations apply. Common in deployments where QUIC offers
  latency or multiplexing benefits.
- **QUIC only.** Appropriate where the deployment ecosystem fully
  supports QUIC. Limits interoperability with TCP/TLS-only
  counterparties.
- **HTTP/3 or WebTransport.** Required for browser clients and for
  paths through HTTP intermediaries; otherwise a choice the operator
  makes to reuse HTTP/3 infrastructure ({{h3-binding}}).

Servers **MAY** advertise their supported bindings through DNS SRV
records (separate records for `_agtp._tcp` and `_agtp._udp`) or
through AGTP Identity Document fields. Discovery of binding support
is outside wire-layer protocol concern.

The earlier "SHOULD prefer QUIC" guidance of base v08 is withdrawn.
Transport preference is a deployment-time choice; the protocol
expresses none. Operators
make the choice based on their infrastructure, their counterparty
ecosystem, and their latency requirements.

# Implementation Considerations

## TLS Library Choice

Both bindings depend on TLS 1.3 implementations. Common production
TLS libraries (OpenSSL 3.x, BoringSSL, rustls, Go's crypto/tls)
provide TLS 1.3 with appropriate cipher suites. Implementations
**SHOULD** use TLS libraries with active maintenance and security
review.

For QUIC, the TLS implementation **MUST** integrate with the QUIC
implementation per {{RFC9001}}. Some QUIC libraries (quiche, msquic,
quinn) bundle TLS; others integrate with external TLS libraries.

## Certificate Provisioning

AGTP Agent Certificates per {{AGTP-CERT}} are provisioned through
Agent Certificate Authority infrastructure. The same certificate
**MAY** be presented across every binding that carries TLS; the certificate is never
binding-specific.

Implementations supporting several bindings **MAY** maintain separate
TLS configurations for each binding, but the certificate and trust
anchor material **SHOULD** be shared to simplify operational
management.

## Idle Timeout Tuning

Default idle timeouts (60 seconds for AGTP sessions) **MAY** be
adjusted by deployment. For interactive agent workloads where
sessions pause between requests, longer idle timeouts
reduce reconnection overhead. For high-throughput batch workloads,
shorter idle timeouts free server resources more quickly.

QUIC's longer per-connection idle tolerance ({{RFC9000}}) makes
QUIC particularly suitable for sessions with bursty traffic
patterns.

## Connection Multiplexing

QUIC's stream multiplexing supports concurrent AGTP
requests within a session without head-of-line blocking. TCP/TLS
deployments needing concurrency **MAY** open multiple TCP/TLS
connections; head-of-line blocking on a single TCP/TLS connection
will block subsequent requests until the in-flight request
completes.

# Security Considerations

This document inherits the security considerations of {{AGTP}},
{{AGTP-CERT}}, {{RFC8446}} (TLS 1.3), and {{RFC9001}} (QUIC-TLS).
This section addresses considerations specific to the binding choice.

## 0-RTT Replay Attack Surface

The 0-RTT replay-safety profile in {{replay-safety}} is the primary
defense against 0-RTT replay attacks. Implementations that allow
early data **MUST** enforce the eligibility list and **MUST** verify
`Request-Signature` on early data ({{early-signature}}).
Implementations that support neither QUIC nor HTTP/3 are outside
the 0-RTT replay surface because TCP/TLS carries no early data per
this document.

A specification oversight that mistakenly classifies an action method
as eligible for early data would be a protocol-level vulnerability.
Implementations **SHOULD** subject any update to the eligibility list
to rigorous review.

## TLS Downgrade Attacks

AGTP implementations **MUST** reject TLS 1.2 and below per the TLS
Requirements section. Implementations **MUST NOT** accept downgrade
negotiation that would establish a session using a deprecated TLS
version.

Implementations **SHOULD** enable TLS 1.3 downgrade protection
({{RFC8446}} Section 4.1.3) for additional defense.

## QUIC Connection Migration Identity Verification

QUIC connection migration changes the network path the connection
uses; the connection's TLS keys and the agent identity are the same
before and after. An attacker on a new network path cannot impersonate the
client because the attacker lacks the keys.

However, deployment policies for sensitive operations **MAY**
require additional verification after migration. Operators
**SHOULD** consider whether migration tolerance is appropriate for
their threat model.

## Certificate Validation Across Bindings

The same Agent Certificate may be presented across the TCP/TLS, QUIC,
and HTTP/3 bindings. Implementations **MUST** apply the same
validation rules in every binding. A vulnerability allowing a
certificate accepted on one binding but rejected on another could be
exploited; all bindings **MUST** use identical certificate validation
logic.

## Idle Connection and Suspended Session Resource Exhaustion

Long idle timeouts accumulate connection state, and the resume
window of {{AGTP-SESSION}} accumulates suspended-session state after
the connection is gone. An attacker establishing many connections
and sessions and abandoning them could exhaust both. Implementations
**SHOULD** apply per-agent connection limits, **MAY** reduce idle
timeouts under resource pressure, and **SHOULD** cap the number of
suspended sessions retained per Agent-ID, releasing the oldest first
and logging each release with its `Session-ID`.

## Intermediaries on the HTTP/3 Binding

An intermediary carrying the HTTP/3 binding cannot read or alter
AGTP messages without detection, because `Request-Signature` is
verified end to end, but it can observe which agents talk to which
servers and when. Deployments for which that metadata is sensitive
**SHOULD** prefer the TCP/TLS or QUIC binding direct to the server.

# IANA Considerations {#iana}

## ALPN Identifier

This document requests registration of the ALPN identifier `agtp`
for AGTP over QUIC in the IANA TLS Application-Layer Protocol
Negotiation (ALPN) Protocol IDs registry per {{RFC8446}}:

- Protocol: AGTP
- Identification Sequence: `0x61 0x67 0x74 0x70` ("agtp")
- Reference: This document

The TCP/TLS binding requires no separate ALPN identifier
because TCP/TLS deployments traditionally rely on port-based
protocol identification (port 4480 carries AGTP traffic). However,
implementations **MAY** also use the `agtp` ALPN identifier on
TCP/TLS connections for additional clarity in mixed-protocol
deployments.

## HTTP Upgrade Token

This document requests registration of `agtp` in the HTTP Upgrade
Token Registry for use as the `:protocol` value of the extended
CONNECT form in {{h3-binding}}, with this document as the reference.

## Port Assignment

This document references the existing IANA port assignment for AGTP
(port 4480 for both TCP and UDP) per {{AGTP}}. The port assignment
is unmodified by this document. The service names `agtp` (TCP/TLS)
and `agtp-quic` (QUIC) are retained per the existing registration.

# Engagement Invitation

This document is published as an Internet-Draft to invite engagement
from transport experts, TLS practitioners, QUIC implementers, and
operators deploying AGTP infrastructure. The replay-safety profile
in particular benefits from review by participants familiar with
0-RTT security analysis.

Feedback from the IETF QUIC working group on the AGTP-specific
replay-safety profile is welcome, and feedback from the agentproto
working group on the HTTP/3 and WebTransport binding, which is the
binding-target form its charter contemplates, is sought. The profile follows the established
pattern (per {{RFC9001}} and HTTP/3 deployment) of restricting early
data to safe methods, applied to AGTP's specific method catalog.

--- back

# Acknowledgments

This document responds to feedback from Akira Okutomi on
draft-hood-independent-agtp-08, who identified the inappropriate
inclusion of transport-specific guidance and 0-RTT-as-benefit
language in the base specification. The transport-neutrality of the
base specification and the explicit replay-safety profile defined
here both originate from that review.

The replay-safety profile draws on the precedent established by HTTP
over QUIC (HTTP/3) regarding 0-RTT data and safe methods. The AGTP
profile follows the same architectural principle, applied to AGTP's
specific method taxonomy.

# Changes from v00

- Connection and session separated. v00 identified an AGTP session
  with a TCP/TLS or QUIC connection and ended it on connection close
  or on SUSPEND or REVOKE. A session is now the agent-level context of
  {{AGTP-SESSION}}, carried by a connection and surviving it through
  RESUME; both lifecycle sections are rewritten as connection
  lifecycle ({{tcp-lifecycle}}, {{quic-lifecycle}}), connection
  migration is stated as the case needing no RESUME, and connection
  reuse no longer speaks of session state leaking between sessions.
- `Request-Signature` on every binding and on early data
  ({{early-signature}}), with the identity section restated around
  it.
- Eligibility list kept at QUERY and DISCOVER with the reason stated;
  the four other safe methods of {{AGTP}} are excluded, with the
  reason for each ({{eligible}}). ESTABLISH, RESUME, and TERMINATE added to the
  prohibited list.
- HTTP/3 and WebTransport binding added ({{h3-binding}}): extended
  CONNECT framing, WebTransport for browsers, early data under the
  same profile, intermediary considerations, and the planned items.
- References: base to draft-hood-independent-agtp-10; AGTP-API
  replaced by AGTP-METHODS; AGTP-SESSION and AGTP-IDDOC added;
  RFC 9114 and RFC 9220 added.
- Transfer-protocol wording; house style pass.

Wire format compatibility: the TCP/TLS and QUIC framing is unchanged.
A v00 client that treats connection close as session end will
never RESUME, which is permitted. The HTTP/3 binding is new and
optional.
