---
title: "AGTP Transport Bindings: TCP/TLS and QUIC"
abbrev: "AGTP-BINDINGS"
docname: draft-hood-agtp-bindings-00
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
      Internet-Draft: draft-hood-independent-agtp-09
    date: 2026

informative:
  RFC8470:
  RFC9110:
  AGTP-API:
    title: "AGTP-API: Verbs, Paths, Endpoints, and Synthesis"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-api-01
    date: 2026
  AGTP-CERT:
    title: "AGTP Agent Certificate Extension"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-agent-cert-03
    date: 2026

--- abstract

The Agent Transfer Protocol (AGTP) base specification defines AGTP
semantics independently of any specific transport. This document
specifies the AGTP transport bindings for TCP with TLS 1.3 and for
QUIC, defining how AGTP requests and responses are carried over each
transport, how TLS mutual authentication composes with AGTP identity,
and how connection lifecycle events map to AGTP session semantics.

A central concern for the QUIC binding is replay safety. QUIC supports
0-RTT early data, which is replayable by design. AGTP requests carry
authority-scoped actions; a replayed action method could re-trigger
authorized work without the requesting agent's intent. This document
defines an AGTP-specific replay-safety profile: early data is permitted
only for safe idempotent methods (QUERY, DISCOVER) and **MUST NOT** be
used for action methods (EXECUTE, DELEGATE, PURCHASE, TRANSACT, and
the lifecycle methods).

This document follows the pattern established by HTTP: protocol
semantics defined independently of transport ({{RFC9110}} for HTTP),
with transport-specific bindings in separate documents. AGTP semantics
are defined in {{AGTP}}; this document defines the transport
mechanics.

--- middle

# Introduction

The Agent Transfer Protocol {{AGTP}} is a dedicated application-layer
protocol for AI agent traffic. The base specification defines AGTP
methods, headers, status codes, identity model, and the wire format
for requests and responses. The base specification is intentionally
transport-neutral: AGTP semantics do not depend on which transport
carries them.

This document specifies the two transport bindings AGTP implementations
currently support: TCP with TLS 1.3, and QUIC. Each binding defines:

- How AGTP requests and responses are framed on the transport.
- How the transport's connection lifecycle composes with AGTP session
  semantics.
- How TLS mutual authentication composes with AGTP identity per
  {{AGTP-CERT}}.
- Any transport-specific security and operational considerations.

Future revisions of this document may add additional bindings (for
example, AGTP over WebSocket for browser-mediated deployments).
Bindings to other transports may be defined in separate documents
that follow the same pattern as this one.

## Why Transport Bindings Are Separate

The base AGTP specification deliberately separates protocol semantics
from transport mechanics. This separation has three benefits:

**Transport neutrality.** AGTP semantics do not change when the
transport changes. An agent invoking QUERY against another agent
operates the same way whether the connection runs over TCP/TLS or
QUIC. The base specification can be reviewed and reasoned about
without entangling transport-specific concerns.

**Independent evolution.** Transport technology evolves on its own
timeline. The TCP/TLS binding can be refined as TLS evolves. The
QUIC binding can be refined as QUIC evolves. The base AGTP
specification does not need to track these changes.

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
  for TCP/TLS and QUIC.

Safe Idempotent Method:
: An AGTP method whose invocation has no observable side effects on
  agent state and whose repeated invocation produces equivalent
  results. QUERY and DISCOVER are safe idempotent methods per
  {{AGTP}}. Replaying a safe idempotent method does not create
  unauthorized state changes.

Action Method:
: An AGTP method whose invocation produces observable side effects.
  EXECUTE, DELEGATE, PURCHASE, TRANSACT, ACTIVATE, DEACTIVATE,
  REVOKE, REINSTATE, DEPRECATE, and similar methods are action
  methods per {{AGTP}}. Replaying an action method may produce
  duplicate effects.

Early Data:
: Application data sent before the TLS handshake completes, supported
  by TLS 1.3 ({{RFC8446}}) and QUIC ({{RFC9001}}). Early data is
  replayable by design; an attacker who captures early data may
  replay it.

# AGTP Identity at the Transport Layer

Both transport bindings carry AGTP identity in two complementary ways:

1. **At the TLS layer** through mutual authentication using AGTP
   Agent Certificates per {{AGTP-CERT}}, when the agent certificate
   extension is deployed.

2. **At the AGTP wire layer** through the Agent-ID, Owner-ID, and
   Authority-Scope headers carried on every request per {{AGTP}}.

The two layers compose. The certificate (when present) provides
cryptographic identity at the transport layer that infrastructure
components (load balancers, SEPs, governance gateways) can verify
without parsing AGTP headers. The headers provide the AGTP-native
identity that is always present and that any deployment can use
regardless of certificate deployment.

Both bindings **MUST** verify that AGTP identity headers, when
combined with a presented Agent Certificate, are consistent. A
mismatch between the certificate's `agent-id` extension and the
request's Agent-ID header **MUST** be treated as a protocol error per
{{AGTP-CERT}}.

# TCP/TLS Binding {#tcp-tls}

## Overview

AGTP over TCP/TLS runs on a single persistent TCP connection wrapped
by TLS 1.3 or later. Each AGTP session corresponds to one underlying
TCP connection. AGTP requests and responses are sequential within the
connection: a server processes requests in the order received and
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
mechanism over TCP/TLS. AGTP over TCP/TLS does not support 0-RTT
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
delimited messages within a single AGTP session per {{AGTP}}.

## Session Lifecycle

An AGTP session corresponds to a single TCP/TLS connection. The
session begins with the completion of the TLS handshake and ends with
either:

- An explicit SUSPEND or REVOKE method invocation closing the
  session per {{AGTP}}.
- Inactivity timeout (default 60 seconds, configurable).
- TCP connection close.

Implementations **MUST NOT** use socket-level half-close
(`shutdown(SHUT_WR)` or equivalent) to signal end-of-request per the
Wire-Format Framing section of {{AGTP}}. The TLS `close_notify` alert
that results from a half-close terminates the secure session before
the peer can transmit a response, producing a truncation that is
indistinguishable at the application layer from a malicious
downgrade.

When an AGTP session terminates normally, the TLS connection
terminates per {{RFC8446}}. The TCP connection then closes.

## Connection Reuse and Pooling

A single TCP/TLS connection **MAY** carry multiple AGTP sessions
sequentially over its lifetime, though each session is conceptually
distinct. Implementations supporting connection reuse **MUST**
ensure that session state (Session-ID, Task-ID, accumulated context)
does not leak between sessions on the same connection.

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

This document does not define multi-stream coordination beyond what
{{AGTP}} specifies for session semantics. A single AGTP session
**MAY** carry multiple concurrent requests on separate QUIC streams.
The Session-ID header per {{AGTP}} identifies the session across
streams.

QUIC connection-level concerns (flow control, congestion control,
loss recovery per {{RFC9002}}) are handled by the QUIC implementation
and are transparent to AGTP. AGTP implementations **MUST NOT** rely
on stream-level acknowledgment or loss-recovery semantics.

## Connection Migration

QUIC supports connection migration: a connection can move between
network paths (e.g., from Wi-Fi to cellular) without disruption per
{{RFC9000}} Section 9. AGTP over QUIC inherits this property.

An AGTP session **MAY** survive a QUIC connection migration. The
Session-ID per {{AGTP}} is independent of the underlying QUIC
connection state. AGTP implementations **SHOULD** treat QUIC
connection migration as transparent.

When a QUIC connection migration changes the apparent client network
location, AGTP implementations **MAY** apply additional verification
(e.g., re-checking the Agent Certificate, requesting attestation per
trust posture per {{AGTP-CERT}}) for sensitive sessions, but this is
operator policy rather than protocol requirement.

## QUIC-Specific Replay Safety Profile {#replay-safety}

QUIC supports 0-RTT early data per {{RFC9001}}. Early data is sent
before the QUIC handshake completes. Early data is replayable: an
attacker who captures early data may replay it, and the server
cannot distinguish a replay from the original.

AGTP requests carry authority-scoped actions. A replayed EXECUTE,
DELEGATE, PURCHASE, or other action method could re-trigger
authorized work, transfer funds, modify state, or produce duplicate
side effects. This is a real security concern, not a theoretical
one.

This document defines an AGTP-specific replay-safety profile
governing early data use.

### Methods Eligible for Early Data

AGTP methods are eligible for early data **only if** the method is a
safe idempotent method per {{AGTP}}: an invocation that has no
observable side effects on agent state and whose repeated invocation
produces equivalent results.

The currently defined safe idempotent methods eligible for early
data are:

- **QUERY** --- information retrieval. Has no side effects on agent
  state. Replays return equivalent results.
- **DISCOVER** --- capability and presence discovery. Has no side
  effects. Replays return equivalent results.

Future revisions of AGTP may add additional safe idempotent methods.
Any such method **MAY** become eligible for early data through an
update to this binding document.

### Methods Prohibited from Early Data

AGTP action methods **MUST NOT** be sent as early data. The currently
defined action methods prohibited from early data are:

- **EXECUTE** --- carries arbitrary agent operations including
  framework payloads (MCP tool calls, A2A tasks, etc.). May have
  arbitrary side effects.
- **DELEGATE** --- delegates authority. Creates state.
- **PURCHASE** --- commerce action. Moves value.
- **TRANSACT** --- commits to commerce transaction per AGTP-COMMERCE.
- **ACTIVATE**, **DEACTIVATE**, **REVOKE**, **REINSTATE**,
  **DEPRECATE** --- lifecycle methods. Modify agent governance state.
- **NOTIFY** --- delivers messages with at-least-once semantics.
  Replays could produce duplicate delivery.
- **PROPOSE** --- proposes runtime contracts. Creates negotiation
  state.
- **CREATE**, **REPLACE**, **MODIFY**, **REMOVE** --- when used as
  resource-affecting methods per the alias mapping in {{AGTP-API}}.
- **PLAN**, **CONFIRM**, **ESCALATE**, **SUMMARIZE** --- cognitive
  methods that may have stateful effects depending on agent
  implementation.

Servers receiving an early-data request carrying any method not on
the eligible list **MUST** reject the request with status code 425
(Too Early) per {{RFC8470}} and **MUST** require the client to
retransmit the request after the QUIC handshake completes.

### Server Enforcement

QUIC implementations supporting AGTP **MUST** make the AGTP method
visible to the server's request dispatcher before the dispatcher
applies any side-effectful logic. The server's AGTP layer **MUST**
verify that any request received as early data carries a method on
the eligibility list before processing.

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

## Session Lifecycle

An AGTP session over QUIC begins when the QUIC handshake completes
(for connections without 0-RTT) or when the first non-early-data
request is processed (for connections with 0-RTT).

The session ends with either:

- An explicit SUSPEND or REVOKE method invocation closing the
  session per {{AGTP}}.
- Inactivity timeout (default 60 seconds, configurable).
- QUIC connection close.

QUIC connection migration does not end the AGTP session per the
Connection Migration section of this document.

# Connection Selection Between Bindings

When both bindings are available on the server, the client selects
the binding based on its own operational requirements. AGTP defines
no normative preference between TCP/TLS and QUIC.

Clients **MAY** use any of the following selection strategies:

- **TCP/TLS only.** Conservative choice. Universal availability.
- **QUIC with TCP/TLS fallback.** Attempt QUIC first; fall back to
  TCP/TLS if the QUIC handshake fails or 0-RTT replay-safety
  considerations apply. Common in deployments where QUIC offers
  latency or multiplexing benefits.
- **QUIC only.** Appropriate where the deployment ecosystem fully
  supports QUIC. Limits interoperability with TCP/TLS-only
  counterparties.

Servers **MAY** advertise their supported bindings through DNS SRV
records (separate records for `_agtp._tcp` and `_agtp._udp`) or
through AGTP Identity Document fields. Discovery of binding support
is not a wire-layer protocol concern.

This document removes the previous "SHOULD prefer QUIC" guidance from
v08 of the base AGTP specification. Transport preference is a
deployment-time choice, not a protocol-level requirement. Operators
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
**MAY** be presented across both bindings; the certificate is not
binding-specific.

Implementations supporting both bindings **MAY** maintain separate
TLS configurations for each binding, but the certificate and trust
anchor material **SHOULD** be shared to simplify operational
management.

## Idle Timeout Tuning

Default idle timeouts (60 seconds for AGTP sessions) **MAY** be
adjusted by deployment. For interactive agent workloads where
sessions naturally pause between requests, longer idle timeouts
reduce reconnection overhead. For high-throughput batch workloads,
shorter idle timeouts free server resources more quickly.

QUIC's longer per-connection idle tolerance ({{RFC9000}}) makes
QUIC particularly suitable for sessions with bursty traffic
patterns.

## Connection Multiplexing

QUIC's stream multiplexing naturally supports concurrent AGTP
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
early data **MUST** enforce the eligibility list. Implementations
that do not support QUIC are not subject to 0-RTT replay attacks
because TCP/TLS does not support early data per this document.

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
uses but does not change the connection's TLS keys or the agent
identity. An attacker on a new network path cannot impersonate the
client because the attacker lacks the keys.

However, deployment policies for sensitive operations **MAY**
require additional verification after migration. Operators
**SHOULD** consider whether migration tolerance is appropriate for
their threat model.

## Certificate Validation Across Bindings

The same Agent Certificate may be presented across TCP/TLS and QUIC
bindings. Implementations **MUST** apply the same validation rules
in both bindings. A vulnerability allowing a certificate accepted on
one binding but rejected on the other could be exploited; both
bindings **MUST** use identical certificate validation logic.

## Idle Session Resource Exhaustion

Long idle session timeouts can accumulate connection state on
servers. An attacker establishing many sessions and abandoning them
could exhaust server resources. Implementations **SHOULD** apply
per-client connection limits and **MAY** reduce idle timeouts under
resource pressure.

# IANA Considerations

## ALPN Identifier

This document requests registration of the ALPN identifier `agtp`
for AGTP over QUIC in the IANA TLS Application-Layer Protocol
Negotiation (ALPN) Protocol IDs registry per {{RFC8446}}:

- Protocol: AGTP
- Identification Sequence: `0x61 0x67 0x74 0x70` ("agtp")
- Reference: This document

The TCP/TLS binding does not require a separate ALPN identifier
because TCP/TLS deployments traditionally rely on port-based
protocol identification (port 4480 carries AGTP traffic). However,
implementations **MAY** also use the `agtp` ALPN identifier on
TCP/TLS connections for additional clarity in mixed-protocol
deployments.

## Port Assignment

This document references the existing IANA port assignment for AGTP
(port 4480 for both TCP and UDP) per {{AGTP}}. The port assignment
is not modified by this document. The service names `agtp` (TCP/TLS)
and `agtp-quic` (QUIC) are retained per the existing registration.

# Engagement Invitation

This document is published as an Internet-Draft to invite engagement
from transport experts, TLS practitioners, QUIC implementers, and
operators deploying AGTP infrastructure. The replay-safety profile
in particular benefits from review by participants familiar with
0-RTT security analysis.

Feedback from the IETF QUIC working group on the AGTP-specific
replay-safety profile is welcome. The profile follows the established
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

Initial version.
