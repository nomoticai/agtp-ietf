---
title: "AGTP Communication Protocol"
abbrev: "AGTP-COMMUNICATION"
docname: draft-hood-agtp-communication-01
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
keyword:
  - AI agents
  - real-time communication
  - voice
  - video
  - multi-modal

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
  RFC3550:
  RFC8174:
  RFC8825:
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

informative:
  RFC3551:
  RFC7656:
  RFC9000:
  RFC9221:
  AGTP-BINDINGS:
    title: "AGTP Transport Bindings"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-bindings-01
    date: 2026

--- abstract

This document specifies the AGTP Communication Protocol
(AGTP-COMMUNICATION): the companion specification for real-time
multi-modal communication between agents over the Agent Transfer
Protocol (AGTP). AGTP-COMMUNICATION defines how voice, video, and
other real-time media streams are exchanged between agents on the
agent-native substrate, with native support for the wire-level
identity, authority scope, and attribution that AGTP provides.

This is an early specification covering bilateral (two-agent)
real-time communication. Multi-party conversations and conferencing
patterns are out of scope for this revision and are deferred to
future companion work.

--- middle

# Introduction

The Agent Transfer Protocol (AGTP) {{AGTP}} is a transfer protocol
for agent-to-agent and agent-to-API communication, the dedicated
substrate on which agent traffic runs. AGTP carries agent identity, authority scope,
attribution records, and intent-aligned methods at the wire level,
with traffic structurally identified as agent traffic by the
protocol itself.

Agent communication is increasingly multi-modal. Agents communicate
through voice when speaking to humans or to other voice-capable
agents. Agents communicate through video when participating in
visual interactions, screen sharing, or visual data exchange. Agents
communicate through structured data streams for sensor data,
telemetry, and continuous information flows. These real-time
communication patterns require protocol-level support distinct from
the request/response patterns AGTP's base methods address.

This document specifies how real-time multi-modal communication
runs on AGTP. The design reuses established real-time media
patterns where appropriate (drawing on the architectural
principles of {{RFC3550}} and {{RFC7656}}) and defines only what is
specific to agent-native communication on the AGTP substrate.

## Relationship to AGTP-SESSION

AGTP-SESSION {{AGTP-SESSION}} defines session establishment,
lifecycle, resumption, and termination on AGTP. AGTP-COMMUNICATION
builds on AGTP-SESSION: a Communication Session is an AGTP session
established through ESTABLISH, with media parameters carried as an
extension of the ESTABLISH body defined there. AGTP-SESSION's
distinction between a connection (what a binding of
{{AGTP-BINDINGS}} opens) and a session (what ESTABLISH creates)
matters here more than anywhere else in the family: media streams
are bound to a connection and end with it, while the Communication
Session survives connection loss through RESUME and re-opens its
streams.

## Scope of This Document

In scope:

- Bilateral real-time audio communication between agents
- Bilateral real-time video communication between agents
- Multi-modal exchange (audio plus video, structured data alongside
  media)
- Codec negotiation and media format selection
- Real-time media framing on the AGTP substrate
- Quality of service handling at the AGTP layer
- Integration with AGTP-SESSION for session lifecycle

Out of scope for this revision:

- Multi-party conversations (three or more agents)
- Conferencing patterns (mixers, SFUs, broadcast)
- Recording and replay protocols
- Voice-specific applications (telephony, IVR patterns)
- Domain-specific conversational AI patterns

## Conventions and Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described
in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear
in all capitals, as shown here.

# Terminology

Communication Session:
: An AGTP-SESSION established for real-time multi-modal communication
  between two agents, with media parameters negotiated during
  session establishment.

Media Stream:
: A unidirectional flow of real-time media data within a
  Communication Session. A bilateral Communication Session typically
  carries two media streams (one in each direction) per modality.

Modality:
: A category of real-time media. This specification addresses audio,
  video, and structured data modalities. Future revisions may
  address additional modalities.

Codec:
: An encoding format for media data, negotiated between communicating
  agents during session establishment.

Communication Endpoint:
: An AGTP-aware agent participating in a Communication Session.
  Identified by its canonical Agent-ID and carrying authority scope
  appropriate to the communication being undertaken.

# Architectural Model

AGTP-COMMUNICATION extends AGTP's request/response model with
real-time streaming semantics. The architectural model has three
components:

## Session Layer

Communication Sessions are established using AGTP-SESSION's
ESTABLISH method with communication-specific parameters. The
session carries the agent identity, authority scope, and
attribution chain that apply throughout the communication.

Session establishment for communication is more involved than
session establishment for request/response: media parameters
must be negotiated, codecs agreed, and stream characteristics
established before media can flow.

## Media Layer

Media streams carry real-time data between Communication
Endpoints. Each stream has a defined modality (audio, video, or
structured data), a negotiated codec, and timing characteristics
appropriate to its modality.

Media streams are framed for carriage on AGTP. The framing
preserves the timing and sequencing properties that real-time
media requires while binding each frame to the Communication
Session, and through it to the AGTP wire-level facts (identity,
authority scope, attribution) established at ESTABLISH. Media
frames carry the Session-ID and a stream identifier rather than the
full AGTP request header set; the per-request proof of possession
of {{AGTP}} is made once, at ESTABLISH and at each control message,
and the session token binds the frames that follow to it.

## Control Layer

Control messages within a Communication Session manage stream
lifecycle: opening streams, modifying parameters, handling
quality degradation, and closing streams. Control messages use
AGTP methods within the established session context.

# Communication Session Establishment

Communication Sessions are established through the ESTABLISH
method of {{AGTP-SESSION}}, on the path `/session`, with the
standard AGTP request headers including `Request-Signature`, and
with a `communication` object added to the ESTABLISH body. Version
00 of this document carried the media parameters in request headers
(`Session-Intent`, `Communication-Modalities`, `Audio-Codecs`,
`Video-Codecs`); they move into the body so that one ESTABLISH
grammar serves the whole family and the media offer is covered by
the same signature and the same session token as the session
parameters.

## ESTABLISH Request {#establish-request}

A Communication Endpoint initiates a session by issuing ESTABLISH
with a communication offer:

~~~~
AGTP/1.0 ESTABLISH /session
Agent-ID: <initiator Agent-ID>
Authority-Scope: communication:bilateral
Request-Signature: <JWS per AGTP>
Content-Type: application/json
Content-Length: ...

{
  "model": "bounded",
  "session_ttl": 3600,
  "scopes": ["communication:bilateral"],
  "resume_window": 60,
  "initiator_nonce": "<base64url, at least 96 bits>",
  "communication": {
    "modalities": ["audio", "video"],
    "audio_codecs": ["opus", "g722"],
    "video_codecs": ["vp9", "av1"],
    "data_streams": [
      { "label": "telemetry", "reliable": false, "max_rate_hz": 50 }
    ]
  }
}
~~~~

The fields outside `communication` are those of {{AGTP-SESSION}}
and keep their semantics. Within `communication`:

`modalities`:
: The modalities the initiator wishes to use, from `audio`, `video`,
  `data`. **REQUIRED**, at least one.

`audio_codecs`, `video_codecs`:
: Codecs the initiator supports for the modality, in order of
  preference, by registered codec name. **REQUIRED** for each of
  `audio` and `video` that appears in `modalities`.

`data_streams`:
: Structured data streams the initiator proposes, each with a
  `label`, a `reliable` flag, and the rate and schema parameters the
  application needs. **REQUIRED** when `data` appears in
  `modalities`.

A `resume_window` of a minute or less is **RECOMMENDED** for
Communication Sessions: media state is rarely worth retaining
longer, and a shorter window bounds the exposure discussed in
{{AGTP-SESSION}}.

## ESTABLISH Response {#establish-response}

The receiving Communication Endpoint responds with the negotiated
parameters or rejects the session:

~~~~
AGTP/1.0 200 OK
Server-ID: <responder server>
Agent-ID: <initiator Agent-ID, echoed>
Session-ID: <64 lowercase hex>
Content-Type: application/json

{
  "session_id": "<64 lowercase hex>",
  "model": "bounded",
  "expires_at": "2026-10-29T18:00:00Z",
  "resume_window": 60,
  "scopes": ["communication:bilateral"],
  "session_token": "<compact JWS>",
  "communication": {
    "modalities": ["audio", "video"],
    "audio_codec": "opus",
    "video_codec": "vp9",
    "data_streams": [
      { "label": "telemetry", "reliable": false, "max_rate_hz": 50 }
    ],
    "stream_parameters": { "...": "..." }
  }
}
~~~~

Successful establishment returns 200 with the `Session-ID` header
and the negotiated parameters, one codec per modality. The
ESTABLISH invocation is the first Attribution-Record under the
session and the first entry in its session root per
{{AGTP-SESSION}}.

Rejection uses the status codes of {{AGTP}}: `455 Scope Violation`
when the initiator's Authority-Scope lacks a permission the offered
modalities require, with the reason codes of {{AGTP}}; `422` when
no offered codec is acceptable for a required modality or a
requested modality is unsupported, with a body naming the modality
and the codecs the responder would accept so the initiator can
re-offer; `503` when the responder is at capacity or its governance
platform cannot be reached to record the session. A responder
**MUST NOT** accept a subset of the offered modalities silently; it
either negotiates each modality in `modalities` or rejects with
422 naming the one it cannot serve. A response that omits the
`communication` key entirely means the responder predates this
extension ({{AGTP-SESSION}}); the initiator **MUST** TERMINATE the
session rather than send media into it.

## Authority Scope Considerations

Communication Sessions carry significant authority implications.
A session that includes audio capture and transmission grants the
initiating agent the ability to capture and transmit audio for the
session duration. Authority-Scope **MUST** include appropriate
permissions for each modality:

- `communication:audio:capture` for capturing audio
- `communication:audio:transmit` for transmitting audio
- `communication:video:capture` for capturing video
- `communication:video:transmit` for transmitting video
- `communication:bilateral` as a shorthand combining standard
  bilateral capture and transmission

Receivers **MUST** validate that the initiator's Authority-Scope
includes appropriate permissions for the requested modalities, and
that the session-scoped `scopes` the initiator proposes are a
subset of it, per {{AGTP-SESSION}}. A delegated initiator carries
`Delegation-Chain` on ESTABLISH and the receiver verifies it per
{{AGTP}} before the first media frame flows.

# Media Stream Semantics

Media streams within a Communication Session carry real-time data
with timing, sequencing, and quality requirements appropriate to
their modality.

## Audio Streams

Audio streams carry audio media between Communication Endpoints.
Audio framing follows established real-time audio practice with
adaptation for AGTP transport:

- Frames carry timestamp information for synchronization
- Sequence numbers detect loss and reordering
- Frame size is negotiated during session establishment
- Codec-specific parameters (sample rate, channels) are negotiated

AGTP-COMMUNICATION reuses RTP timestamp and sequence semantics
{{RFC3550}} where compatible, adapted for carriage on AGTP rather
than UDP. This preserves established real-time audio handling while
gaining AGTP's wire-level identity and attribution properties.

## Video Streams

Video streams carry video media between Communication Endpoints.
Video framing addresses the additional complexity of variable
frame sizes, key frame management, and bandwidth adaptation:

- Frames carry timestamp and sequence information
- Frame type (key/delta) is indicated
- Codec-specific parameters (resolution, frame rate) are
  negotiated
- Bandwidth adaptation signals are exchanged through control
  messages

## Structured Data Streams

Structured data streams carry continuous data flows other than
audio or video: sensor telemetry, conversational state updates,
real-time analytics, contextual data alongside other media.

Structured data streams have different real-time characteristics
than audio or video. Timing may matter (sensor sampling rates) or
matter little (state updates). Loss tolerance varies by use case.
Structured data stream parameters are negotiated during session
establishment through `data_streams`.

## Carriage on the Bindings {#carriage}

Which transport features a media stream can use depends on the
binding of {{AGTP-BINDINGS}} beneath the connection. On the QUIC
binding, each media stream **SHOULD** be carried on its own QUIC
unidirectional stream when the stream is marked reliable, and on
QUIC DATAGRAM frames {{RFC9221}} when it is marked unreliable, so
that loss in one stream never blocks another. On the TCP binding,
every stream shares one ordered byte stream; head-of-line blocking
applies and the latency figures of {{latency}} are unlikely to be
met, so the TCP binding is suitable for structured data and for
audio where latency tolerance is known to be wide. On the HTTP/3
and WebTransport binding, WebTransport unidirectional streams and
datagrams serve the same roles as their QUIC counterparts.

Media streams belong to the connection. When the connection is lost
and the Communication Session is resumed through RESUME per
{{AGTP-SESSION}}, every media stream is closed and the endpoints
re-open the streams they still need through control messages; the
negotiated codecs and parameters carry over with the session and
need no renegotiation.

# Quality of Service

Real-time communication has quality requirements that the binding
beneath AGTP must meet. AGTP-COMMUNICATION specifies quality of
service handling appropriate to each modality.

## Latency Requirements {#latency}

Audio communication typically requires latency under 150ms for
natural conversational flow. Video communication tolerates higher
latency but synchronization between audio and video is critical.
Structured data streams have application-specific latency
requirements.

When AGTP runs over QUIC {{RFC9000}}, the underlying transport supports
multiple streams with independent flow control, which enables
appropriate handling of different modality requirements within a
single Communication Session.

## Bandwidth Adaptation

Communication Endpoints **MUST** be capable of adapting media
parameters in response to bandwidth constraints. Control messages
within a Communication Session signal:

- Bandwidth estimates from the receiving endpoint
- Requested adaptations from the sending endpoint
- Confirmation of parameter changes

Bandwidth adaptation is negotiated; both endpoints participate in
the decision to adapt.

## Priority Within AGTP

AGTP traffic on port 4480 **SHOULD** be treated by the network with
priority appropriate to its modality. Real-time
audio and video streams require lower latency than request/response
traffic; structured data streams may have varying requirements.

Network operators carrying AGTP traffic **SHOULD** consider that
AGTP-COMMUNICATION sessions are likely to include latency-sensitive
real-time media and apply appropriate QoS handling.

# Attribution and Recording

AGTP's attribution model applies to Communication Sessions: the
ESTABLISH invocation, every control message (stream open, parameter
change, stream close), RESUME, and TERMINATE each produce an
Attribution-Record, and the session root of {{AGTP-SESSION}}
summarizes them. Media frames produce no Attribution-Records of
their own; the stream-open record names the stream, its modality
and codec, and the Agent-IDs on each end, and the stream-close
record carries the frame and byte counts, so the existence, extent,
and parties of every media exchange are attributable without
logging the media.

Media content within streams is unrecorded by the protocol by
default. Recording is an application-layer decision made by
governance frameworks or specific deployments. AGTP-COMMUNICATION
provides the session-level attribution that recording systems can
build on; it performs no recording itself.

When recording is performed at the application layer, the
attribution records produced by AGTP-COMMUNICATION provide
verifiable evidence of session participants, authority scope, and
session lifecycle that supports compliance with recording-relevant
regulations.

# Security Considerations

Real-time communication on AGTP inherits AGTP's security
properties: encryption from the binding (TLS 1.3 or QUIC), agent
identity verification including per-request proof of possession,
and authority scope enforcement at the protocol layer.

Additional security considerations specific to communication:

## Media Capture Authorization

Agents that capture audio or video **MUST** have appropriate
Authority-Scope. This is enforced at session establishment.
Capture without scope is a 455 Scope Violation.

## Replay and Tampering

Audio and video streams **MUST NOT** be replayable across sessions
without the cryptographic markers that identify them as
recordings. Session identifiers, timestamps, and attribution
records carried with streams enable verification that media was
captured in the context the recipient believes.

## Privacy Considerations

Communication Sessions may involve sensitive content (private
conversations, confidential video, sensor data with privacy
implications). AGTP's wire-level identity verification and
attribution provide the structural facts that privacy frameworks
require. Application-layer privacy controls build on these
foundations.

## Denial of Service

Real-time communication can be used to consume substantial
bandwidth and processing resources. Communication Endpoints
**SHOULD** implement appropriate rate limits and resource controls.
Authority-Scope can include resource limitations that the
protocol enforces at session establishment.

# IANA Considerations

This document defines no new headers. It requests, in the
registries established by {{AGTP}} and {{AGTP-SESSION}} once they
exist:

- The `communication` key in the ESTABLISH body extension registry
  of {{AGTP-SESSION}}, with this document as reference
- Authority-Scope tokens for communication
  (`communication:audio:capture`, `communication:audio:transmit`,
  `communication:video:capture`, `communication:video:transmit`,
  `communication:bilateral`)

Codec names in `audio_codecs` and `video_codecs` are drawn from the
RTP payload format registries {{RFC3551}}; this document creates no
codec registry.

# Open Questions

Several design decisions remain open for this revision:

- Whether to define an AGTP-specific real-time media framing or
  to reuse RTP framing carried over AGTP transport
- The relationship to WebRTC {{RFC8825}} for browser-based agents
  communicating over AGTP
- Whether to define agent-specific codecs (e.g., for low-bandwidth
  agent-to-agent voice with no need to sound human) or to
  rely entirely on existing codec registries
- How AGTP-COMMUNICATION sessions interact with AGTP's intent
  methods for non-real-time exchanges within the same agent pair
- Multi-party conversation patterns and whether they belong as a
  v01 extension or as a separate companion specification

These will be addressed in future revisions of this draft based on
community feedback and implementation experience.

# Changes from v00

Version 01 is an alignment revision; the architectural model,
modalities, scope tokens, and open questions are unchanged.

- **Informational.** Version 00 was marked Standards Track; the
  document is now Informational, matching every other draft in the
  AGTP family.
- **ESTABLISH aligned to AGTP-SESSION** ({{establish-request}},
  {{establish-response}}). The media offer moves from request
  headers into a `communication` object in the ESTABLISH body
  defined by draft-hood-agtp-session-01; the request line, path,
  `Request-Signature`, `Session-ID`, and session token are those of
  that document. The four v00 headers are withdrawn.
- **Status codes.** 451 becomes 455 Scope Violation, matching the
  renumbering in draft-hood-independent-agtp-10; parameter mismatch
  is 422 with a re-offer body rather than 463, which base AGTP
  reserves for PROPOSE.
- **Carriage on the bindings** ({{carriage}}): QUIC streams and
  datagrams per modality, the TCP binding's limits, and the rule
  that media streams belong to the connection and are re-opened
  after RESUME.
- **Attribution** restated around the session root: control
  messages are attributed, media frames are summarized by the
  stream-close record.
- References pinned to draft-hood-independent-agtp-10 and
  draft-hood-agtp-session-01; AGTP-BINDINGS and RFC 9221 added.
  Transfer-protocol wording applied. House style pass.

--- back

# Acknowledgments

This document builds on the broader AGTP family and incorporates
architectural principles from established real-time media work
including RTP/RTCP {{RFC3550}} and WebRTC {{RFC8825}}.

# Contributors

Contributors will be acknowledged in future revisions as community
participation develops.
