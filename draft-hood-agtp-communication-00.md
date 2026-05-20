---
title: "AGTP Communication Protocol"
abbrev: "AGTP-COMMUNICATION"
docname: draft-hood-agtp-communication-00
category: std
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
      Internet-Draft: draft-hood-independent-agtp-07
    date: 2026
  AGTP-SESSION:
    title: "AGTP Session Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-session-00
    date: 2026

informative:
  RFC3551:
  RFC7656:
  RFC9000:

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

The Agent Transfer Protocol (AGTP) {{AGTP}} defines a dedicated
protocol substrate for agent-to-agent and agent-to-API
communication. AGTP carries agent identity, authority scope,
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
lifecycle, and basic message exchange semantics on AGTP.
AGTP-COMMUNICATION builds on AGTP-SESSION: real-time
communication sessions are established through AGTP-SESSION's
ESTABLISH method, with media-specific parameters negotiated as
part of session setup.

## Scope of This Document

In scope:

- Bilateral real-time audio communication between agents
- Bilateral real-time video communication between agents
- Multi-modal exchange (audio plus video, structured data alongside
  media)
- Codec negotiation and media format selection
- Real-time media framing on AGTP transport
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

Media streams are framed for transport over AGTP. The framing
preserves the timing and sequencing properties that real-time
media requires while carrying the AGTP wire-level facts
(identity, attribution) on each frame.

## Control Layer

Control messages within a Communication Session manage stream
lifecycle: opening streams, modifying parameters, handling
quality degradation, and closing streams. Control messages use
AGTP methods within the established session context.

# Communication Session Establishment

Communication Sessions are established through AGTP-SESSION's
ESTABLISH method with the `communication` capability declared.

## ESTABLISH Request

A Communication Endpoint initiates a session by issuing ESTABLISH
with a communication intent declaration:

~~~~
ESTABLISH /sessions HTTP/AGTP/1.0
Agent-ID: <canonical Agent-ID>
Authority-Scope: communication:bilateral
Session-Intent: communication
Communication-Modalities: audio, video
Audio-Codecs: opus, g722
Video-Codecs: vp9, av1
Content-Type: application/agtp+json
~~~~

The `Communication-Modalities` header declares which modalities
the initiator wishes to use. The `Audio-Codecs` and `Video-Codecs`
headers declare codecs the initiator supports, in order of
preference.

## ESTABLISH Response

The receiving Communication Endpoint responds with the negotiated
parameters or rejects the session:

~~~~
HTTP/AGTP/1.0 200 OK
Agent-ID: <canonical Agent-ID>
Session-ID: <session identifier>
Communication-Modalities: audio, video
Audio-Codec: opus
Video-Codec: vp9
Stream-Parameters: <negotiated stream parameters>
~~~~

Successful establishment returns 200 with the negotiated parameters.
Rejection returns appropriate AGTP status codes (451 Scope Violation
for authority-scope issues, 463 Proposal Rejected for parameter
mismatch, 503 Service Unavailable for capacity limitations).

## Authority Scope Considerations

Communication Sessions carry significant authority implications.
A session that includes audio capture and transmission grants the
initiating agent the ability to capture and transmit audio for the
session duration. Authority-Scope MUST include appropriate
permissions for each modality:

- `communication:audio:capture` for capturing audio
- `communication:audio:transmit` for transmitting audio
- `communication:video:capture` for capturing video
- `communication:video:transmit` for transmitting video
- `communication:bilateral` as a shorthand combining standard
  bilateral capture and transmission

Receivers MUST validate that the initiator's Authority-Scope
includes appropriate permissions for the requested modalities.

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
{{RFC3550}} where compatible, adapted for transport on AGTP rather
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

Structured data streams carry continuous data flows that are not
audio or video: sensor telemetry, conversational state updates,
real-time analytics, contextual data alongside other media.

Structured data streams have different real-time characteristics
than audio or video. Timing may matter (sensor sampling rates) or
may not (state updates). Loss tolerance varies by use case.
Structured data stream parameters are negotiated during session
establishment.

# Quality of Service

Real-time communication has quality requirements that AGTP must
support at the transport layer. AGTP-COMMUNICATION specifies
quality of service handling appropriate to each modality.

## Latency Requirements

Audio communication typically requires latency under 150ms for
natural conversational flow. Video communication tolerates higher
latency but synchronization between audio and video is critical.
Structured data streams have application-specific latency
requirements.

When AGTP runs over QUIC, the underlying transport supports
multiple streams with independent flow control, which enables
appropriate handling of different modality requirements within a
single Communication Session.

## Bandwidth Adaptation

Communication Endpoints MUST be capable of adapting media
parameters in response to bandwidth constraints. Control messages
within a Communication Session signal:

- Bandwidth estimates from the receiving endpoint
- Requested adaptations from the sending endpoint
- Confirmation of parameter changes

Bandwidth adaptation is negotiated; both endpoints participate in
the decision to adapt.

## Priority Within AGTP

AGTP traffic on port 4480 SHOULD be treated with priority
appropriate to its modality at the transport layer. Real-time
audio and video streams require lower latency than request/response
traffic; structured data streams may have varying requirements.

Network operators carrying AGTP traffic SHOULD consider that
AGTP-COMMUNICATION sessions are likely to include latency-sensitive
real-time media and apply appropriate QoS handling.

# Attribution and Recording

AGTP's attribution model applies to Communication Sessions: every
session establishes attribution chains, and attribution records
are produced for session lifecycle events.

Media content within streams is not, by default, recorded by the
protocol. Recording is an application-layer decision made by
governance frameworks or specific deployments. AGTP-COMMUNICATION
provides the session-level attribution that recording systems can
build on; it does not itself perform recording.

When recording is performed at the application layer, the
attribution records produced by AGTP-COMMUNICATION provide
verifiable evidence of session participants, authority scope, and
session lifecycle that supports compliance with recording-relevant
regulations.

# Security Considerations

Real-time communication on AGTP inherits AGTP's security
properties: transport encryption (TLS 1.3 or QUIC), agent identity
verification, and authority scope enforcement at the protocol
layer.

Additional security considerations specific to communication:

## Media Capture Authorization

Agents that capture audio or video MUST have appropriate
Authority-Scope. This is enforced at session establishment.
Capture without scope is a 451 Scope Violation.

## Replay and Tampering

Audio and video streams MUST NOT be replayable across sessions
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
SHOULD implement appropriate rate limits and resource controls.
Authority-Scope can include resource limitations that the
protocol enforces at session establishment.

# IANA Considerations

This document defines several new headers and parameters that
require IANA registration:

- Session-Intent header (registered under AGTP header registry)
- Communication-Modalities header
- Audio-Codecs, Video-Codecs headers (codec negotiation)
- Audio-Codec, Video-Codec response headers
- Authority-Scope tokens for communication
  (`communication:audio:*`, `communication:video:*`,
  `communication:bilateral`)

Specific registry assignments will be detailed in a future revision
once the AGTP header and scope token registries are established.

# Open Questions

Several design decisions remain open for this revision:

- Whether to define an AGTP-specific real-time media framing or
  to reuse RTP framing carried over AGTP transport
- The relationship to WebRTC {{RFC8825}} for browser-based agents
  communicating over AGTP
- Whether to define agent-specific codecs (e.g., for low-bandwidth
  agent-to-agent voice that doesn't need to sound human) or to
  rely entirely on existing codec registries
- How AGTP-COMMUNICATION sessions interact with AGTP's intent
  methods for non-real-time exchanges within the same agent pair
- Multi-party conversation patterns and whether they belong as a
  v01 extension or as a separate companion specification

These will be addressed in future revisions of this draft based on
community feedback and implementation experience.

--- back

# Acknowledgments

This document builds on the broader AGTP family and incorporates
architectural principles from established real-time media work
including RTP/RTCP {{RFC3550}} and WebRTC {{RFC8825}}.

# Contributors

Contributors will be acknowledged in future revisions as community
participation develops.
