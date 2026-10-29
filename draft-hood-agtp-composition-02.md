---
title: "AGTP Composition Profiles: Agent Group Messaging Protocols, External Identity Providers, and HTTP Gateways"
abbrev: "AGTP-COMPOSITION"
docname: draft-hood-agtp-composition-02
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent protocol composition
  - MCP
  - A2A
  - ACP
  - OAuth
  - OIDC
  - HTTP gateway
  - AGTP substrate

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
  RFC9110:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026

informative:
  RFC6749:
  RFC6750:
  A2A:
    title: "Agent-to-Agent Protocol Specification"
    author:
      org: Linux Foundation
    date: 2025
    target: https://a2aprotocol.ai
  ACP:
    title: "Agent Communication Protocol"
    author:
      org: IBM Research
    date: 2025
  MCP:
    title: "Model Context Protocol"
    author:
      org: Anthropic
    date: 2024
    target: https://modelcontextprotocol.io
  ANP:
    title: "Agent Network Protocol"
    date: 2025
  AGTP-METHODS:
    title: "AGTP Methods (AGTP Verbs)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-methods-00
    date: 2026
  AGTP-RCNS:
    title: "AGTP Runtime Contract Negotiation Substrate"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-rcns-00
    date: 2026
  AGTP-IDDOC:
    title: "AGTP Agent Identity Document"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identity-document-00
    date: 2026
  AGTP-SESSION:
    title: "AGTP Session Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-session-01
    date: 2026
  AGTP-CERT:
    title: "AGTP Agent Certificate Extension"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-agent-cert-03
    date: 2026
  AGTP-IDENTIFIERS:
    title: "AGTP Identifier Chain"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identifiers-03
    date: 2026
  AGTP-TRUST:
    title: "AGTP Trust and Verification Specification"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-trust-02
    date: 2026
  AGTP-LOG:
    title: "AGTP Transparency Log"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-03
    date: 2026
  AGTP-DISCOVERY:
    title: "AGTP Discovery and Naming"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-discovery-02
    date: 2026
  AGTP-PRESENCE:
    title: "AGTP Presence: Ambient Discovery and Visibility for Agent Substrates"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-presence-01
    date: 2026
  AGTP-LEI:
    title: "AGTP-LEI: Binding the Agent Transfer Protocol to the Verifiable Legal Entity Identifier"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-lei-01
    date: 2026
  AGTP-COMMERCE:
    title: "AGTP-Commerce: Open Commerce Specification for Agent-to-Agent Transactions"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-commerce-01
    date: 2026

--- abstract

The Agent Transfer Protocol (AGTP) operates as a substrate beneath
several adjacent layers: Agent Group Messaging Protocols (AGMPs) such
as the Model Context Protocol (MCP), the Agent-to-Agent Protocol (A2A),
and the Agent Communication Protocol (ACP); external identity providers
(OAuth, OIDC, enterprise IdPs); and HTTP-based clients that interact
with agents through translation gateways. This document specifies the
composition profiles for each of these adjacent layers, defining
normative mapping rules between the adjacent layer and AGTP.

Three composition families are specified. The AGMP composition profile
specifies how MCP, A2A, and ACP messages are carried over AGTP, with
mapping rules for identity, authority scope, delegation, and session
fields between the AGMP and AGTP layers. The External Identity Provider
composition profile specifies how AGTP agent identity composes with
externally-issued credentials, treating "which agent" and "on whose
behalf" as orthogonal axes. The HTTP Gateway composition profile
specifies the translation surface for accepting HTTP traffic into AGTP-
served agents as a REST adoption ramp.

Across all three families, AGTP headers take precedence over equivalent
adjacent-layer fields for all infrastructure-level routing, enforcement,
and audit operations. Agents gain transport-level governance,
observability, and identity without modification to the adjacent layers.

--- middle

# Introduction

## The Composition Problem

AGTP {{AGTP}} is a transfer protocol for AI agent traffic: the
dedicated substrate on which that traffic runs, above TCP, QUIC, or
HTTP/3 and below the agent frameworks. It is designed to compose
with adjacent layers rather than to replace them. Three categories of adjacent layer benefit from explicit
composition profiles with AGTP:

**Agent Group Messaging Protocols (AGMPs).** Protocols such as MCP
{{MCP}}, A2A {{A2A}}, ACP {{ACP}}, and ANP {{ANP}} define agent-to-
agent communication semantics: what agents say to each other, how
tools are invoked, how tasks are delegated, how capabilities are
advertised. These protocols are well-designed for what they address
but share a common substrate limitation: they typically run over HTTP
and inherit HTTP's constraints for agent traffic.

**External identity providers.** OAuth {{RFC6749}}, OIDC, SPIFFE,
enterprise IdPs, and similar services issue credentials that answer
"on whose behalf is the agent acting." AGTP identity answers "which
agent is making this call." The two axes are orthogonal and require
explicit composition rules so they compose cleanly rather than
conflict.

**HTTP gateways.** Many existing clients communicate over HTTP/REST
rather than AGTP. Operators that want to expose AGTP-served agents
to HTTP clients deploy translation gateways. The gateway is a
deployment-time adoption ramp rather than a protocol of its own; the
translation contract benefits from being specified explicitly so it
is interoperable across implementations.

This document specifies the composition profile for each of these
three categories. The profiles share a common architectural principle:
AGTP headers take precedence over equivalent fields in the adjacent
layer for all infrastructure-level routing, enforcement, and audit
operations. Adjacent layers retain their own semantics and development
paths.

## The Limitation This Document Addresses

At the infrastructure layer, an A2A Task message carried over HTTP is
indistinguishable from a human-initiated POST request. An MCP tool
call carries no protocol-level signal of which agent sent it, under
what authority, or within what resource budget. An OAuth bearer token
in an HTTP request says "this principal is authenticated" but says
nothing about which agent acted on that principal's behalf or with
what authority scope. An HTTP gateway translating REST traffic into
agent operations has no canonical mapping from HTTP verbs to agent
semantics.

Governance, observability, and identity enforcement in such systems
require application-layer instrumentation that is expensive,
inconsistent, and invisible to routing infrastructure.

AGTP addresses this at the protocol layer. When messages from
adjacent layers are carried over AGTP, every request carries the
sending agent's verified identity and proof of key possession, its
authority scope, its delegation lineage, and its resource budget in
protocol headers, and resolves through the Agent-ID to an Identity
Document that names the accountable owner.
Infrastructure components -- load balancers, gateways, SEPs -- can
route, filter, and enforce governance without parsing message
payloads.

## Relationship to AGTP Core

This document is a companion to {{AGTP}} and is normative for
implementations that compose adjacent layers with AGTP. The core AGTP
specification defines the protocol; this document defines how
specific adjacent-layer concepts map to AGTP constructs.
Implementations that use AGTP as the substrate for AGMP traffic,
implementations that compose external identity providers with AGTP,
and implementations that deploy HTTP gateways in front of AGTP
agents **MUST** follow the mapping rules in this document.

## Scope

This document covers three composition profile families:

1. **AGMP composition profiles** for MCP {{MCP}}, A2A {{A2A}}, and
   ACP {{ACP}} ({{agmp-section}}).
2. **External identity provider composition** for OAuth, OIDC, and
   similar credential-issuing services ({{idp-section}}).
3. **HTTP gateway composition** for translation of HTTP traffic into
   AGTP method invocations ({{gateway-section}}).

The document modifies no AGMP specification, no identity
provider specification, and never HTTP. AGMPs retain their own schemas,
semantics, and development paths. External identity providers retain
their own credential formats and validation rules. HTTP retains its
semantics. This document specifies only how adjacent-layer concepts
relate to AGTP headers, methods, and operational contracts.

Additional composition surfaces specific to other AGTP companion
specifications are addressed in those specifications:

- Payment provider composition is specified in {{AGTP-COMMERCE}}.
- Credit bureau and trust provider composition is specified in
  {{AGTP-TRUST}}.
- Certificate authority composition is specified in {{AGTP-CERT}}.
- Institutional identity composition (LEI/vLEI) is specified in
  {{AGTP-LEI}}.

This document covers only the three composition profile families
named above. Composition surfaces in other documents compose with
this document without being duplicated here. The strategic
positioning, the substrate model, and the precedence rule are
summarized in {{AGTP}}; this document is where the profiles and the
wire examples live, including the AGMP wire-example appendix that
earlier base revisions carried.

# Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when,
and only when, they appear in all capitals.

AGMP (Agent Group Messaging Protocol):
: The collective term for higher-layer AI agent messaging standards that
  operate over AGTP as their substrate, including MCP, A2A, ACP,
  and ANP. AGMPs define what agents say. AGTP defines how those messages
  move.

Adjacent Layer:
: A protocol, service, or interface that composes with AGTP from above
  (AGMPs), beside (external identity providers), or in front (HTTP
  gateways). Adjacent layers retain their own semantics; AGTP provides
  the substrate primitives they consume.

Substrate:
: The protocol layer beneath an adjacent layer, which carries its
  messages and supplies the properties it lacks. In this document,
  AGTP is the substrate. The substrate is invisible to the adjacent
  layer; adjacent layers require no modification to compose with
  AGTP.

Composition Profile:
: The normative set of mapping rules specifying how a specific adjacent
  layer's identity, delegation, session, capability, or translation
  concepts correspond to AGTP header fields, method semantics, and
  operational contracts.

External Identity Provider (IdP):
: A service that issues credentials proving the identity of a human
  principal, service account, or workload identity. Examples include
  OAuth authorization servers, OIDC identity providers, SPIFFE
  identity controllers, and enterprise IdPs. External IdPs answer "on
  whose behalf is the agent acting"; AGTP identity answers "which
  agent is making this call."

HTTP Gateway:
: A translation surface that accepts inbound HTTP requests, translates
  them into AGTP method invocations, and dispatches them through the
  same response-finalization path as native AGTP traffic. The gateway
  is a deployment-time adoption ramp outside the AGTP wire protocol.

# The Narrow-Waist Architecture

AGTP functions as the narrow-waist layer of the AI agent protocol stack,
analogous to IP in the internet architecture. IP has no understanding
of TCP or UDP; it carries their packets. AGTP has no understanding of
MCP, A2A, OAuth bearer tokens, or HTTP semantics; it carries the
messages, composes with the credentials, and absorbs the
translations. The narrow-waist property
means any adjacent layer can compose with AGTP, and any AGTP-aware
infrastructure can observe and govern traffic across all adjacent layers,
without coupling between the layers.

~~~~
+-----------------------------------------------------------+
|        Agent Application Logic                            |
+-----------------------------------------------------------+
|  AGMP Layer: MCP / A2A / ACP / ANP       [messaging]      |
|  External IdP Layer: OAuth / OIDC        [authorization]  |
|  HTTP Gateway Layer: REST translation    [adoption ramp]  |
+-----------------------------------------------------------+
|  AGTP - Agent Transfer Protocol          [this composes]  |
+-----------------------------------------------------------+
|  TLS 1.3+                                [mandatory]      |
+-----------------------------------------------------------+
|  TCP / QUIC / HTTP3+WebTransport         [bindings]       |
+-----------------------------------------------------------+
~~~~
{: #narrow-waist title="AGTP as Narrow-Waist Composition Layer"}

The composition rules in this document specify how adjacent-layer concepts
map to the AGTP layer below them. The adjacent layers are unaware of AGTP
internals; the AGTP layer carries adjacent-layer payloads, validates
adjacent-layer credentials, and translates adjacent-layer requests without
interpreting their content beyond the mapping rules specified here.

# Precedence Rule {#precedence}

This rule applies universally across all composition profiles defined in
this document.

AGTP headers (Agent-ID, Request-Signature, Authority-Scope,
Delegation-Chain, Session-ID, Task-ID, Budget-Limit, and AGTP-Zone-ID)
and the Identity Document the Agent-ID resolves to (which carries
the owner) take precedence
over equivalent fields in any adjacent-layer payload, credential, or
translated HTTP request for all infrastructure-level operations,
including routing, enforcement, rate limiting, audit, and attribution.

Specifically:

- Infrastructure components including SEPs, governance gateways, load
  balancers, and monitoring systems **MUST** use AGTP header values
  for all protocol-level decisions. They **MUST NOT** parse AGMP
  payload fields, external IdP claims, or HTTP headers to make
  routing or enforcement decisions, even when those fields appear
  to carry equivalent information.

- Adjacent-layer fields **MAY** carry equivalent identity or
  delegation information for application-layer use by the receiving
  agent. These fields carry no authority for infrastructure
  decisions.

- In the event of a conflict between an AGTP header value and an
  adjacent-layer field carrying equivalent information, the AGTP
  header value **MUST** be treated as authoritative.

- An AGTP request carrying an adjacent-layer payload **MUST** be
  rejected by infrastructure if the AGTP headers are absent or
  invalid, regardless of whether the adjacent-layer payload itself
  is well-formed.

This rule enables infrastructure to govern agent traffic uniformly
regardless of which adjacent layer is composed, without requiring
adjacent-layer-specific parsing.

# AGMP Composition Profiles {#agmp-section}

The AGMP composition profiles specify how messages from Agent Group
Messaging Protocols are carried over AGTP. Each profile maps the
AGMP's identity, delegation, session, capability, and message
concepts to AGTP headers and method semantics.

# A2A Composition Profile

## Overview

The Agent-to-Agent Protocol (A2A) {{A2A}} defines semantics for
agent-to-agent task delegation, capability advertisement, artifact
exchange, and provenance tracking. A2A runs over HTTP in its base
specification. When running over AGTP, A2A messages are carried in
AGTP request and response bodies, with A2A task, capability, and
identity concepts mapped to AGTP headers.

## A2A Concept Mapping

| A2A Concept | A2A Field | AGTP Mapping |
|---|---|---|
| Sending agent identity | `agent.id` | `Agent-ID` header, proven by `Request-Signature` |
| Accountable principal | `agent.owner` | `owner_id` of the Agent Identity Document the Agent-ID resolves to ({{AGTP-IDDOC}}); never a request header |
| Agent capabilities | Agent Card | AGTP DESCRIBE response |
| Agent identity document | Agent Card | AGTP Agent Identity Document ({{AGTP-IDDOC}}); the Card is a self-asserted description, the Identity Document an issuer-signed credential |
| Task delegation | Task object | AGTP DELEGATE method body |
| Task identifier | `task.id` | `Task-ID` header |
| Delegation provenance (pre-authorization) | Task lineage | Delegation Sequence in the `Delegation-Chain` header ({{AGTP}}) |
| Delegation provenance (attribution) | Task lineage | `prior_actions` chain of the Attribution-Record ({{AGTP-IDENTIFIERS}}) |
| Session context | Conversation ID | `Session-ID` header |
| Artifact delivery | Artifact object | AGTP NOTIFY body |
| Capability scope | Agent Card capabilities | `Authority-Scope` header |
{: title="A2A-to-AGTP Field Mapping"}

## Mapping Rules

### Agent Identity

The AGTP `Agent-ID` header carries the sending agent's Canonical
Agent-ID, proven per request by `Request-Signature` ({{AGTP}}); the
A2A `agent.id` **MAY** appear in the payload for application-layer
use and **MUST NOT** be used to populate `Agent-ID`. An A2A agent
with no Canonical Agent-ID has no AGTP identity and cannot send over
AGTP until one is issued through Agent Genesis.

The A2A agent's declared owner or registrant is recorded as the
`owner_id` of its Agent Identity Document at registration
({{AGTP-IDDOC}}), where a relying party reads it by resolving the
Agent-ID. Owner identity rides in no request header. Where the A2A
`agent.owner` and the Identity Document's `owner_id` differ, the
Identity Document governs and the receiving agent **SHOULD** log the
difference.

### Task Delegation

An A2A Task sent from one agent to another **MUST** be carried as
an AGTP DELEGATE method invocation. The AGTP DELEGATE parameters
**MUST** include:

- `target_agent_id`: the canonical AGTP Agent-ID of the receiving agent
- `authority_scope`: the subset of the sending agent's Authority-Scope
  applicable to this task; **MUST NOT** exceed the sending agent's
  declared scope
- `delegation_token`: the delegating agent's token for this hop,
  signed with its `agent_public_key` over the delegate, the scope,
  and the digest of its own parent hop, per the Delegation Token
  Format of {{AGTP}}
- `task`: the A2A Task object as the AGTP body payload

The A2A `task.id` **MUST** be used as the AGTP `Task-ID` header
value, or if no A2A task ID is present, a new Task-ID **MUST** be
generated by the AGTP layer.

### Delegation Provenance

A2A's task provenance chain maps to two AGTP artifacts, as
{{AGTP}} states for this profile. On the pre-authorization side, a
request the delegate makes under the delegated authority carries
the Delegation Sequence in the `Delegation-Chain` header: one hop
per DELEGATE, each hop the parent's Agent-ID and its delegation
token, origination to current, verified per {{AGTP}} (strict-subset
attenuation, parent re-verification, depth and cycle limits). The
DELEGATE invocation itself carries `Delegation-Chain` only when the
delegating agent is itself acting under delegation. On the
attribution side, the completed action's Attribution-Record records
the same sequence through `prior_actions` per {{AGTP-IDENTIFIERS}}.
A2A lineage fields in the payload are informational; where they
diverge from the Delegation Sequence, the sequence governs
({{precedence}}).

### Capability Advertisement

When an agent exposes an A2A Agent Card, the same capability
information **SHOULD** be exposed via the AGTP DESCRIBE method.
An AGTP DESCRIBE request to the agent **MUST** return a Capability
Document ({{AGTP}}) that reflects the agent's A2A capabilities
translated into AGTP capability domain format. The Agent Card and
the Agent Identity Document are different kinds of document: the
Card is what the agent says about itself, the Identity Document is
what its issuer signed about it, and only the latter is a credential
({{AGTP-IDDOC}}).

### Artifact Delivery

A2A Artifact objects **MUST** be carried as AGTP NOTIFY method
invocations when delivered asynchronously. The NOTIFY body
**MUST** include the A2A Artifact object and a `content_type`
field with value `a2a-artifact`.

## Wire Example: A2A Task over AGTP

The orchestrator acts under its own authority, so its DELEGATE
carries no `Delegation-Chain`; the analyst's subsequent requests
under the delegated scope carry one hop, `<orchestrator
Agent-ID>;<token>`.

~~~~
AGTP/1.0 DELEGATE /tasks
Agent-ID: agt-orchestrator
Authority-Scope: agents:delegate documents:query
Request-Signature: <JWS per AGTP>
Session-ID: sess-a1b2c3d4
Task-ID: task-0099
Budget-Limit: tokens=50000, ttl=300
Content-Schema: https://a2aprotocol.ai/schema/task/v1
Content-Type: application/agtp+json

{
  "method": "DELEGATE",
  "task_id": "task-0099",
  "parameters": {
    "target_agent_id": "agt-analyst",
    "authority_scope": "documents:query",
    "delegation_token": "<compact JWS, kid = agt-orchestrator>",
    "task": {
      "a2a_task_id": "a2a-task-7f3a",
      "message": {
        "role": "user",
        "parts": [{"type": "text",
          "text": "Summarize Q1 financial reports"}]
      },
      "artifacts": []
    }
  }
}

AGTP/1.0 200 OK
Server-ID: srv-acme-01
Agent-ID: agt-orchestrator
Task-ID: task-0099
Owner-ID: acme.tld
Attribution-Record: <JWS per AGTP-IDENTIFIERS>
Cost-Estimate: tokens=12450
Content-Type: application/agtp+json

{
  "status": 200,
  "task_id": "task-0099",
  "result": {
    "a2a_task_id": "a2a-task-7f3a",
    "status": "completed",
    "artifacts": [
      {
        "name": "Q1 Summary",
        "mimeType": "text/plain",
        "parts": [{"type": "text", "text": "..."}]
      }
    ]
  }
}
~~~~

# MCP Composition Profile

## Overview

The Model Context Protocol (MCP) {{MCP}} defines structured communication
between AI models and tools, resources, and context providers. MCP
distinguishes between clients (AI models) and servers (tool/resource
providers). When running over AGTP, MCP messages are carried in AGTP
request and response bodies, with MCP session, context, and tool-call
semantics mapped to AGTP constructs.

## MCP Concept Mapping

| MCP Concept | MCP Field | AGTP Mapping |
|---|---|---|
| Client identity | `client_id` | `Agent-ID` header, proven by `Request-Signature` |
| Server identity | `server_id` | The responding agent's Identity Document, reached through the `Server-ID` response header and the signer of the Attribution-Record ({{AGTP-IDENTIFIERS}}) |
| Session context | `context` | `Session-ID` header ({{AGTP-SESSION}}) + LEARN method |
| Conversation state | Message history | AGTP persistent session state |
| Tool call | `tools/call` | AGTP EXECUTE with `payload_type: application/vnd.mcp+json`, `action: tools/call` |
| Resource request | `resources/read` | AGTP QUERY with `scope: documents:*` |
| Sampling request | `sampling/createMessage` | AGTP EXECUTE with `action: sampling/createMessage` |
| Prompt | `prompts/get` | AGTP QUERY with `scope: knowledge:query` |
| Capability list | `capabilities` | AGTP DESCRIBE response |
| Tool result | Tool response | AGTP EXECUTE response body |
{: title="MCP-to-AGTP Field Mapping"}

## Mapping Rules

### Client and Server Identity

The MCP client **MUST** be identified in the AGTP `Agent-ID` header
using its canonical AGTP Agent-ID. The MCP `client_id` field **MAY**
appear in the payload for application-layer use.

The MCP server is itself an agent with a Canonical Agent-ID; its
responses carry `Server-ID` (the host) and an Attribution-Record
signed under its key, from which the responding agent's identity is
established per {{AGTP-IDENTIFIERS}}. Earlier revisions named a
`Server-Agent-ID` response header, which {{AGTP}} retired. The MCP
server's capability list **SHOULD** be accessible via AGTP DESCRIBE.

### Session and Context Management

MCP's context window (the accumulated message history and state for a
conversation) **MUST** be associated with an AGTP `Session-ID`. The
AGTP session persists across multiple MCP message exchanges within a
single conversation.

Context that should persist beyond the current AGTP session **SHOULD**
be written using the AGTP LEARN method with scope `principal` or
`global` as appropriate.

### Tool Calls

An MCP `tools/call` operation **MUST** be carried as an AGTP EXECUTE
method invocation, the generic carrier {{AGTP}} names for framework
payloads, with `payload_type` set to `application/vnd.mcp+json`,
`action` set to `tools/call`, and the MCP request as `payload`. The
`Authority-Scope` header is enforced against the tool being called.
Version 01 of this document carried tool calls on QUERY; QUERY is a
safe method, eligible for early data under {{AGTP}}'s bindings, and
a tool call may have side effects, so EXECUTE is the correct
carrier. Clients **SHOULD** supply `idempotency_key` for tools whose
effects are unsafe to repeat.

### Resource Access

An MCP `resources/read` operation **MUST** be carried as an AGTP QUERY
method invocation. The AGTP `Authority-Scope` header **MUST** include
the appropriate resource scope (e.g., `documents:query` for document
resources).

### Sampling

An MCP `sampling/createMessage` operation **MUST** be carried as an
AGTP EXECUTE method invocation with `action: sampling/createMessage`
and a `Budget-Limit` in tokens. Sampling consumes budget and
produces a fresh result on every invocation, so it belongs on the
mechanics carrier rather than on QUERY; the `action` value signals to
AGTP infrastructure that the request involves model inference,
enabling differentiated routing and budget enforcement.

## Wire Example: MCP Tool Call over AGTP

~~~~
AGTP/1.0 EXECUTE /tools
Agent-ID: agt-assistant
Authority-Scope: tools:web_search
Request-Signature: <JWS per AGTP>
Session-ID: sess-mcp-b2c3d4
Task-ID: task-0100
Budget-Limit: calls=1, ttl=60
Content-Schema: https://modelcontextprotocol.io/schema/tool-call/v1
Content-Type: application/agtp+json

{
  "method": "EXECUTE",
  "task_id": "task-0100",
  "parameters": {
    "action": "tools/call",
    "payload_type": "application/vnd.mcp+json",
    "idempotency_key": "01J9Z...",
    "payload": {
      "name": "web_search",
      "arguments": {
        "query": "IETF agent protocol drafts 2026"
      }
    }
  }
}

AGTP/1.0 200 OK
Server-ID: srv-acme-tools
Agent-ID: agt-assistant
Task-ID: task-0100
Attribution-Record: <JWS per AGTP-IDENTIFIERS>
Cost-Estimate: calls=1
Content-Type: application/agtp+json

{
  "status": 200,
  "task_id": "task-0100",
  "result": {
    "payload_type": "application/vnd.mcp+json",
    "content": [
      {
        "type": "text",
        "text": "draft-hood-independent-agtp-02 ..."
      }
    ]
  }
}
~~~~

## Wire Example: MCP Resource Fetch over AGTP

A resource read is read-only and maps onto QUERY, since no
application-side action is performed. This is the example the base
specification's appendix carried through v09.

~~~~
AGTP/1.0 QUERY /resources
Agent-ID: agt-assistant
Authority-Scope: documents:query, knowledge:query
Request-Signature: <JWS per AGTP>
Session-ID: sess-mcp-b2c3d4
Task-ID: task-0102
Content-Type: application/agtp+json

{
  "method": "QUERY",
  "task_id": "task-0102",
  "parameters": {
    "intent": "fetch document corpus for Q1 financial analysis",
    "scope": ["documents:financial"],
    "modality": "mcp.resource",
    "mcp_resource_uri": "mcp://corpus/financial/q1-2026"
  }
}
~~~~

## Wire Example: MCP Sampling over AGTP

~~~~
AGTP/1.0 EXECUTE /inference
Agent-ID: agt-assistant
Authority-Scope: inference:sample
Request-Signature: <JWS per AGTP>
Session-ID: sess-mcp-b2c3d4
Task-ID: task-0101
Budget-Limit: tokens=10000, ttl=120
Content-Type: application/agtp+json

{
  "method": "EXECUTE",
  "task_id": "task-0101",
  "parameters": {
    "action": "sampling/createMessage",
    "payload_type": "application/vnd.mcp+json",
    "payload": {
      "messages": [
        {
          "role": "user",
          "content": {
            "type": "text",
            "text": "Summarize these IETF drafts for me."
          }
        }
      ],
      "modelPreferences": {
        "hints": [{"name": "claude"}]
      },
      "maxTokens": 1000
    }
  }
}
~~~~

# ACP Composition Profile

## Overview

The Agent Communication Protocol (ACP) {{ACP}} defines messaging
semantics for agent-to-agent communication, including message routing,
broadcast, and capability discovery. When running over AGTP, ACP
messages are carried in AGTP request and response bodies.

## ACP Concept Mapping

| ACP Concept | ACP Field | AGTP Mapping |
|---|---|---|
| Sender identity | Sender agent ID | `Agent-ID` header |
| Recipient | Recipient agent ID | AGTP NOTIFY `recipient` parameter |
| Broadcast | Multi-recipient message | AGTP NOTIFY with broadcast group |
| Capability discovery | Capability query | AGTP DESCRIBE method |
| Message | Message object | AGTP NOTIFY or COLLABORATE body |
| Multi-agent task | Coordinated task | AGTP COLLABORATE method |
{: title="ACP-to-AGTP Field Mapping"}

## Mapping Rules

### Unicast Messages

ACP unicast messages sent from one agent to a specific recipient
**MUST** be carried as AGTP NOTIFY method invocations. The NOTIFY
`recipient` parameter **MUST** carry the canonical AGTP Agent-ID of
the intended recipient.

### Broadcast Messages

ACP broadcast messages **MUST** be carried as AGTP NOTIFY method
invocations with a broadcast group identifier in the `recipient`
parameter. The broadcast group **MUST** be registered in the
governance platform before use.

### Multi-Agent Coordination

ACP multi-agent coordination workflows **SHOULD** be carried as AGTP
COLLABORATE method invocations where two or more agents work in
parallel or defined roles toward a shared objective. AGTP COLLABORATE
is peer-to-peer, consistent with ACP's coordination model.

### Capability Discovery

ACP capability queries **MUST** be carried as AGTP DESCRIBE method
invocations. The AGTP DESCRIBE response **MUST** include all
capability domains relevant to the querying agent's task.

## Wire Example: ACP Unicast Message over AGTP

~~~~
AGTP/1.0 NOTIFY /messages
Agent-ID: agt-coordinator
Authority-Scope: agents:notify
Request-Signature: <JWS per AGTP>
Session-ID: sess-acp-c3d4e5
Task-ID: task-0201
Content-Type: application/agtp+json

{
  "method": "NOTIFY",
  "task_id": "task-0201",
  "parameters": {
    "recipient": "agt-worker-01",
    "content": {
      "acp_message_type": "task_assignment",
      "payload": {
        "task": "Process batch-2026-Q1",
        "deadline": "2026-04-01T00:00:00Z"
      }
    },
    "urgency": "normal",
    "delivery_guarantee": "at_least_once"
  }
}
~~~~

# External Identity Provider Composition {#idp-section}

The External Identity Provider composition profile specifies how AGTP
agent identity composes with externally-issued credentials such as
OAuth bearer tokens, OIDC `id_token`s, SPIFFE SVIDs, and enterprise
IdP session tokens.

## The Two-Axis Model

AGTP identity and external IdP credentials answer different questions
and **MUST** be treated as orthogonal axes:

- **AGTP identity answers "which agent is making this call."**
  Hash-anchored on the Agent Genesis ({{AGTP}}), presented via the
  Agent-ID header, proven per request by `Request-Signature`,
  optionally bound to a client certificate per {{AGTP-CERT}}, scoped
  by Authority-Scope. The trust root is the
  Genesis-issuer registrar per {{AGTP-TRUST}}.

- **External IdP credentials answer "on whose behalf is the agent
  acting."** A human principal, a service account, or a workload
  identity in an enterprise identity stack. The trust root is the
  IdP.

The two axes compose. AGTP places no constraints on how external
credentials are validated; servers **MAY** require external
credentials on specific methods, **MAY** lift a named claim from the
validated credential into the Attribution-Record, and **MUST** treat
the failure modes of the two axes as independent.

## Three Composition Patterns

**Pattern 1: AGTP identity only.** Agent-ID plus optional Agent
Certificate plus Authority-Scope. No external IdP. Trust anchored in
the Genesis-issuer registrar. Appropriate for closed agent ecosystems
where the registrar is the trust root and no human-on-whose-behalf
claim is required.

**Pattern 2: AGTP identity plus external IdP credential.** The
request carries Agent-ID (wire-layer identity) and an `Authorization`
header carrying an IdP-issued credential (application-layer
principal). Both ride on the same request. AGTP Authority-Scope and
external authorization scope (e.g., OAuth scopes per {{RFC6749}}) are
independent: Authority-Scope is the agent's *capacity* to act, the
IdP scope is the *delegation* from a principal authorizing the agent
to do so on its behalf. A server's `[policies.oauth]` block (defined
operationally outside this document) declares which methods require
an external credential and which validator processes the credential.
On validation success, the configured claim (typically `sub`) is
lifted into the request context as the acting-principal identifier
and stamped on the Attribution-Record as `principal_id` per
{{AGTP-IDENTIFIERS}}.

**Pattern 3: OIDC-federated Genesis-issuer trust.** The
Genesis-issuer key itself is attested by an IdP. Trust anchors that
would otherwise be a static trusted-registrars list are replaced by
OIDC discovery: the verifier configures one or more OIDC anchors
(discovery URL plus trusted issuer), fetches the JWKS at runtime,
and confirms the `issuer_public_key` on the Agent Genesis matches
one of the IdP's published keys. The Genesis schema is unchanged;
the change is the resolution path the verifier uses to decide
whether a recorded issuer key is trusted. See {{AGTP-TRUST}} for the
trust-anchor schema.

## Authorization Header

AGTP servers process the `Authorization` request header per HTTP
semantics ({{RFC9110}}). The header value is opaque to AGTP itself;
the configured validator interprets it. Common forms include:

- `Bearer TOKEN`: OAuth 2.0 / OIDC bearer tokens per {{RFC6750}}.
- Other schemes (`Basic`, `Digest`, custom schemes): passed
  through to the configured validator unchanged.

Validation failures **MUST** return `401 Unauthorized` with a
response body carrying a structured `reason` from the vocabulary:

| Reason | Meaning |
|---|---|
| `oauth-required` | The invoked method is in the server's `required_on_methods` list and no `Authorization` header was presented |
| `oauth-invalid` | An `Authorization` header was presented and the configured validator rejected it (bad signature, expired token, untrusted issuer, etc.) |
{: title="External IdP Composition 401 Reason Vocabulary"}

The 401 codes for external-credential failures are the same 401
code AGTP uses for other authentication failures; the `reason`
field disambiguates.

## Token Opacity and Attribution

AGTP **MUST NOT** stamp the raw `Authorization` header value or the
raw token onto the Attribution-Record. The Attribution-Record is
signed and may be replayed by chain walkers and audit consumers;
embedding bearer tokens would create a credential-disclosure
surface. Only the validated, lifted claim (the `principal_id`)
appears on the Attribution-Record per {{AGTP-IDENTIFIERS}}.

## Backward Compatibility

The External IdP composition surface is opt-in. Servers with no
`[policies.oauth]` configuration process requests identically to
servers built without IdP composition awareness. Requests without an
`Authorization` header on servers that require none are
dispatched normally. This preserves Pattern 1 deployments
byte-for-byte.

## Wire Example: AGTP Request with OIDC Bearer Token

~~~~
AGTP/1.0 EXECUTE /loans
Agent-ID: agt-loan-evaluator
Authority-Scope: loan:evaluate documents:read
Request-Signature: <JWS per AGTP>
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
Session-ID: sess-loan-eval-001
Task-ID: task-2026-0601
Content-Type: application/agtp+json

{
  "method": "EXECUTE",
  "task_id": "task-2026-0601",
  "parameters": {
    "payload_type": "application/vnd.bank.loan-eval+json",
    "action": "evaluate-application",
    "payload": {
      "application_id": "app-12345",
      "applicant_consent_ref": "consent-2026-001"
    }
  }
}
~~~~

The bearer token in the `Authorization` header authorizes the
loan-evaluator agent to act on behalf of a specific banking customer.
The server's `[policies.oauth]` block requires Bearer tokens on the
`loan:evaluate` scope, validates the token against the configured
IdP, lifts the `sub` claim as `principal_id`, and stamps it on the
Attribution-Record. The Identity Document that `agt-loan-evaluator`
resolves to records the bank as owner (`owner_id` carrying an
LEI-anchored institutional identity per {{AGTP-LEI}}); the bearer
token identifies the customer. The two are independent axes.

# HTTP Gateway Composition {#gateway-section}

The HTTP Gateway composition profile specifies the translation
surface for accepting HTTP traffic into AGTP-served agents. The
gateway is a deployment-time adoption ramp outside the AGTP wire
protocol; this profile specifies the translation contract so it is
interoperable across implementations.

## Architectural Position

Operators that need to accept inbound HTTP/REST traffic on the same
agent surface that serves AGTP **MAY** deploy an HTTP gateway sidecar
alongside the AGTP daemon. The gateway is a parallel listener that
accepts HTTP requests, translates them into AGTP method invocations,
and dispatches them through the same response-finalization path as
native AGTP traffic.

The gateway is no substitute for an AGTP binding. Two AGTP servers
exchanging traffic with one another **MUST NOT** use the gateway as
their hop; AGTP runs on its own bindings, and gateway-mediated
AGTP-to-AGTP flows would lose the wire-level identity, attribution,
and trust-posture guarantees AGTP defines. The gateway exists to translate non-AGTP
clients into AGTP method invocations.

~~~~
+--------------------+              +--------------------+
|  HTTP/REST Client  |  HTTP/1.1    |  HTTP Gateway      |
|  (non-AGTP-aware)  | -----------> |  (Sidecar)         |
+--------------------+              +--------------------+
                                              |
                                              | AGTP method
                                              | invocation
                                              v
                                    +--------------------+
                                    |  AGTP Daemon       |
                                    |  (port 4480)       |
                                    +--------------------+
~~~~
{: #gateway-arch title="HTTP Gateway Translation Pattern"}

## Translation Rules

The gateway applies the following translation rules:

1. **HTTP method to AGTP method.** The HTTP method is run through
   the server's `policies.methods.aliases` map per {{AGTP-METHODS}}; the
   resolved canonical AGTP method is used for dispatch. The default
   alias seed (GET to FETCH, POST to CREATE, PUT to REPLACE, DELETE
   to REMOVE, PATCH to MODIFY) handles the five common verbs. The
   original HTTP method is preserved on the Attribution-Record as
   `requested_method` per {{AGTP-IDENTIFIERS}}.

2. **HTTP path to AGTP path.** The path is forwarded verbatim,
   subject to the path grammar in {{AGTP-METHODS}}.

3. **HTTP headers to AGTP headers.** The gateway **MAY** forward
   selected headers (e.g., `Authorization` mapped to AGTP
   authentication context per the External IdP composition profile
   in {{idp-section}}); the specific header forwarding policy is
   operator-defined.

4. **Response.** The AGTP response (status, headers, body) is
   translated into an HTTP response. The gateway **MUST** invoke the
   standard response-finalization path so the AGTP response carries
   a valid Attribution-Record; the resulting Audit-ID and Server-ID
   participate in the same per-agent chain as native AGTP responses.

## Constraints

- **The gateway **MUST** strip the `Allow-RCNS` header from inbound
  HTTP requests before dispatch.** HTTP callers cannot opt into
  Runtime Contract Negotiation Substrate ({{AGTP-RCNS}}) through the
  gateway by design; RCNS is a substrate feature for AGTP-native callers whose
  identity, trust tier, and scope are bound at the AGTP layer.
  Operators that want to expose RCNS to HTTP callers **MUST** layer
  AGTP-native authentication on top of the gateway, in which case
  the gateway is no longer the right abstraction.

- **The gateway is a translation surface rather than a binding.**
  Two AGTP servers exchanging traffic with one another **MUST NOT**
  use the gateway as their hop; AGTP runs on its own bindings, and
  gateway-mediated AGTP-to-AGTP flows would lose the wire-level
  identity, attribution, and trust-posture guarantees this
  specification defines.

- **The gateway **MUST NOT** synthesize AGTP identity headers from
  HTTP request properties.** The `Agent-ID` forwarded to the AGTP
  daemon **MUST** be the gateway's own Canonical Agent-ID, signed
  with the gateway's own key in `Request-Signature` (the agent on
  whose behalf the gateway is calling into the AGTP layer), never an
  identity inferred from the HTTP client; the owner recorded on the
  gateway's Identity Document is therefore the gateway operator. HTTP
  clients that need authenticated agent identity at the AGTP layer
  **MUST** present credentials the gateway can validate and lift
  into a Pattern 2 External IdP composition.

## Reference Deployment Pattern

The reference implementation pattern is an operational module
distributed as `mod_http_gateway` or equivalent, listening on a
separate port from the AGTP listener (default `127.0.0.1:8080` for
the gateway, port 4480 for AGTP), so the two protocols never share
a socket.

## Wire Example: HTTP GET Translated to AGTP FETCH

~~~~
GET /agents/library/books/2026-spring HTTP/1.1
Host: library.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
Accept: application/json
~~~~

The gateway translates this to:

~~~~
AGTP/1.0 FETCH /agents/library/books/2026-spring
Agent-ID: agt-http-bridge
Authority-Scope: catalog:read
Request-Signature: <JWS per AGTP, gateway key>
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
Session-ID: sess-http-bridge-9f3a
Task-ID: task-http-001
Content-Type: application/agtp+json

{
  "method": "FETCH",
  "task_id": "task-http-001",
  "parameters": {
    "path": "/agents/library/books/2026-spring",
    "format": "structured"
  }
}
~~~~

The Agent-ID is the gateway's own agent identity rather than one
synthesized from the HTTP client. The bearer token from the HTTP request is forwarded
and validated by the AGTP server's External IdP composition per
Pattern 2. The original HTTP method (`GET`) is preserved on the
Attribution-Record as `requested_method` for audit reconstruction.

# The Content-Schema Header {#content-schema}

{{AGTP}} lists `Content-Schema` among the headers reserved for a
companion to specify; this document is that companion, since the
header exists for carried payloads.

`Content-Schema` is an **OPTIONAL** request header whose value is an
absolute URI identifying the schema (JSON Schema or the adjacent
layer's own schema language) that the request body, or the
`payload` parameter of an EXECUTE, conforms to:

~~~~
Content-Schema = absolute-URI
~~~~

Semantics:

- The sender asserts that the body conforms to the named schema.
  `Request-Signature` covers the body digest and leaves this header
  outside its signed set, so the header is an unsigned hint about a
  signed body: a receiver that validates learns whether the body
  conforms, and an altered header is caught by that validation
  rather than by the signature.
- A receiver **MAY** validate the body against the schema. A
  receiver that validates and finds the body non-conformant **MUST**
  reject with `422` and reason `schema-mismatch`; one that lacks the
  schema **MAY** proceed and **SHOULD** record `schema-unverified`
  in the Attribution-Record's evaluation fields.
- Infrastructure **MUST NOT** route or enforce on `Content-Schema`;
  it is a payload property, below the precedence line of
  {{precedence}}.
- Schema URIs are owned by the adjacent-layer specification bodies;
  this document registers none.

# Cross-Composition Interoperability

This section addresses interoperability concerns that arise when
multiple composition families are active simultaneously: an agent
serving MCP and A2A workflows on the same session, a Pattern 2 IdP
composition combined with an HTTP gateway, or a cross-AGMP delegation
chain that traverses an external IdP boundary.

## Agent Discovery Across AGMPs

An agent registered with an AGTP governance platform is discoverable
regardless of which AGMP it uses: it is found by search and
discovered by Agent-ID per {{AGTP-DISCOVERY}} and {{AGTP-PRESENCE}}.
The AGTP Agent Identity Document ({{AGTP-IDDOC}}) exposes the
agent's supported methods and modalities; the AGTP DESCRIBE method
exposes its capabilities. AGMP-specific capability formats (A2A
Agent Cards, MCP capability lists) **MAY** be included as extensions
to the AGTP Capability Document and are never required.

## Session Continuity Across AGMPs

A single AGTP session **MAY** carry messages from multiple AGMPs within
the same workflow. For example, an orchestrator agent might use A2A to
delegate a task, then use MCP to invoke a tool within that task, all
within a single AGTP session identified by the same `Session-ID`. AGTP
session semantics ({{AGTP-SESSION}}) persist across AGMP message
types within the session.

## Identity Consistency Across Composition Families

An agent operating across multiple composition families **MUST** use the
same canonical AGTP Agent-ID in all AGTP headers, regardless of which
adjacent layer is composed. AGMP-layer identity fields (A2A `agent.id`,
MCP `client_id`), external IdP claims, and HTTP-client identifiers
**MAY** differ from the canonical AGTP Agent-ID for backward
compatibility with non-AGTP deployments, but the AGTP Agent-ID is
authoritative for all governance and audit purposes.

## IdP and Gateway Combined

An HTTP request with a Bearer token arriving at an HTTP gateway
composes both the External IdP profile and the HTTP Gateway profile:

1. The gateway translates the HTTP request to an AGTP method
   invocation per {{gateway-section}}, forwarding the
   `Authorization` header.

2. The AGTP server validates the bearer token per the External IdP
   composition Pattern 2 in {{idp-section}}, lifting the configured
   claim as `principal_id`.

3. The Attribution-Record carries both the gateway's Agent-ID
   (which agent invoked the method) and the lifted `principal_id`
   (on whose behalf the call was made).

The gateway is the agent at the AGTP layer; the bearer token's
subject is the principal. Both compose cleanly into the existing
two-axis model.

# Security Considerations

## AGMP Payload Injection

Threat: A malicious actor embeds forged identity or scope fields in an
AGMP message payload, attempting to override or supplement the AGTP
header values with elevated permissions.

Mitigation: Infrastructure components **MUST NOT** parse AGMP payloads
for identity or scope information. The precedence rule in {{precedence}}
is absolute: AGTP headers govern all infrastructure decisions. Application
code **SHOULD** validate that AGMP payload identity fields are consistent
with the AGTP headers and **MUST** reject requests where they conflict
in ways that suggest an escalation attempt.

## Cross-AGMP Delegation Elevation

Threat: An agent uses AGMP composition to cross protocol boundaries in
a way that circumvents AGTP scope enforcement. For example, receiving
a task via A2A with a delegated scope, then re-delegating it via MCP
with an elevated scope.

Mitigation: the delegation rules of {{AGTP}} apply unchanged across
AGMP boundaries. Each re-delegation is a new AGTP DELEGATE
invocation, minting a new hop whose scope is a strict subset of the
delegating agent's effective scope, and every request under the
delegated authority carries the full Delegation Sequence, which the
receiving server verifies hop by hop. An elevation attempted through
an AGMP payload fails at the first conformant verifier, because the
payload carries no authority ({{precedence}}).

## Payload Schema Validation

Implementers **SHOULD** use the `Content-Schema` header
({{content-schema}}) to declare the schema of the AGMP payload.
Recipients **SHOULD** validate the payload against the declared
schema before processing. This mitigates injection attacks that rely
on schema ambiguity at the AGMP layer.

## Bearer Token Leakage via Attribution-Record

Threat: An implementation incorrectly stamps the raw `Authorization`
header value or the raw bearer token onto the Attribution-Record. Since
Attribution-Records are signed and may be replayed by chain walkers and
audit consumers, this would create a credential-disclosure surface.

Mitigation: AGTP **MUST NOT** stamp the raw `Authorization` header
value or the raw token onto the Attribution-Record. Only the
validated, lifted claim (the `principal_id`) appears on the
Attribution-Record per {{AGTP-IDENTIFIERS}}. Implementations **MUST**
verify this constraint in their attribution-record builder before
release.

## External IdP Validator Confusion

Threat: A server with multiple configured IdP validators dispatches a
credential to the wrong validator, causing a credential issued by one
IdP to be interpreted as if issued by another. An attacker could
exploit this to present a credential to a validator that accepts it
when the intended validator would have rejected it.

Mitigation: Servers **MUST** route credentials to the correct
validator based on policy-defined matching rules (token issuer,
audience claim, header marker). Servers **MUST NOT** dispatch a
credential to multiple validators in parallel and accept any
success; this creates the confusion surface described above.

## HTTP Gateway Identity Synthesis

Threat: An HTTP gateway implementation synthesizes AGTP identity
headers from HTTP request properties (client IP, user-agent string,
hostname), creating a forgeable identity surface.

Mitigation: The gateway **MUST NOT** synthesize `Agent-ID` from HTTP
request properties per the constraints in {{gateway-section}}, and
cannot forge `Request-Signature` for an identity whose key it lacks.
The Agent-ID forwarded to the AGTP daemon **MUST** identify the
gateway's own agent identity. HTTP clients that need authenticated
agent identity at the AGTP layer **MUST** present credentials the
gateway can validate and lift into a Pattern 2 External IdP
composition.

## HTTP Gateway RCNS Bypass

Threat: An HTTP client attempts to opt into Runtime Contract
Negotiation Substrate (RCNS) through the gateway by sending an
`Allow-RCNS` header, gaining access to a substrate feature that is
designed for AGTP-native callers with bound identity, trust tier, and
scope.

Mitigation: The gateway **MUST** strip the `Allow-RCNS` header from
inbound HTTP requests before dispatch per {{gateway-section}}.
Operators that want to expose RCNS to HTTP-originating callers
**MUST** layer AGTP-native authentication on top of the gateway, in
which case the gateway is no longer the right abstraction.

# IANA Considerations

This document requests registration of the `Content-Schema` request
header ({{content-schema}}) in the AGTP Header Field registry of
{{AGTP}}, status Permanent, with this document as reference, and of
the reason tokens `schema-mismatch` (status 422) and
`schema-unverified` (evaluation) in the AGTP reason vocabulary. All
other AGTP method, header, and status code registrations are defined
in {{AGTP}}.
External identity providers retain their own registrations
({{RFC6749}}, {{RFC6750}}). AGMP specifications retain their own
registrations. The `Content-Schema` URIs used in this document
reference external schema registries maintained by the respective
adjacent-layer specification bodies.

# Changes from v01

Version 02 aligns the profiles with draft-hood-independent-agtp-10
and the companions it split out. The three profile families and
their structure are unchanged.

## Substantive Changes

1. **Owner identity is a document field.** `Owner-ID` is removed
   from the precedence header list, the A2A mapping, the mapping
   rules, and all wire examples; the owner is read from the Agent
   Identity Document the Agent-ID resolves to ({{AGTP-IDDOC}}). The
   response-side `Owner-ID` header of {{AGTP}} is unaffected.

2. **Delegation restated against base v10.** The A2A provenance
   rule now maps to the Delegation Sequence (pre-authorization) and
   `prior_actions` (attribution), with the hop format, token
   signer, and verification rules by reference to {{AGTP}} rather
   than restated as independent MUSTs. The DELEGATE example no
   longer carries `Delegation-Chain` on an undelegated invocation,
   and `delegation_token` is the parent's signature rather than a
   governance-platform token.

3. **MCP tool calls and sampling move from QUERY to EXECUTE.** QUERY
   is safe and early-data eligible; a tool call may have side
   effects and sampling consumes budget. EXECUTE with
   `payload_type: application/vnd.mcp+json` is the carrier {{AGTP}}
   names. Resource reads and prompts stay on QUERY. The MCP resource
   fetch example from the base appendix is added.

4. **`Request-Signature`** shown on every example; `Agent-ID`
   values shown as canonical identifiers; request lines carry a
   path; `Server-Agent-ID` replaced by `Server-ID` and the
   Attribution-Record signer.

5. **`Content-Schema` specified** ({{content-schema}}) as the
   reserved header's home, with the 422 reason `schema-mismatch`.

6. **Agent Card distinguished from Agent Identity Document**: a
   self-asserted description versus an issuer-signed credential.

7. **AGMP wire-example appendix** from base v09 absorbed; the base
   document now summarizes and points here.

8. References: base to -10; AGTP-API split into AGTP-METHODS and
   AGTP-RCNS; AGTP-IDENTIFIERS -03, AGTP-LOG -03, AGTP-DISCOVERY -02,
   AGTP-PRESENCE -01, AGTP-LEI -01, AGTP-COMMERCE -01; AGTP-IDDOC and
   AGTP-SESSION added. Transfer-protocol wording in place of
   transport. House style pass.

## Wire Format Compatibility

Removing `Owner-ID` from requests and moving MCP tool calls to
EXECUTE are breaking for v01 implementations of those two profiles;
both track changes already made in base AGTP v10, so a v01
implementation updated to base v10 needs the same changes. The A2A
header shape is otherwise unchanged. `Content-Schema` is additive.

# Changes from v00

Version 01 broadens the document's scope from AGMP-only composition
to a three-family composition profile structure. The AGMP profiles
are updated to track the v08 AGTP method floor and renamed
identifiers. Two new composition profile families are introduced:
External Identity Provider composition and HTTP Gateway composition.
These were previously specified in the v08 base spec's "Composition
with Higher-Level Frameworks" section; v01 of this document is the
canonical location for the normative specification.

## Substantive Changes

1. **Title expanded** to "AGTP Composition Profiles: Agent Group
   Messaging Protocols, External Identity Providers, and HTTP
   Gateways" reflecting the broader scope.

2. **External Identity Provider composition profile added** as
   {{idp-section}}. Three composition patterns (AGTP-only,
   AGTP+IdP, OIDC-federated Genesis trust). Authorization header
   semantics. Token opacity rule. 401 reason vocabulary
   (`oauth-required`, `oauth-invalid`). Backward compatibility
   guidance. Wire example showing OIDC bearer token composition.

3. **HTTP Gateway composition profile added** as {{gateway-section}}.
   Architectural position. Translation rules (HTTP method to AGTP
   method via alias map; HTTP path verbatim; header forwarding
   policy; response finalization). Constraints (Allow-RCNS header
   stripping; transport-vs-translation distinction; no identity
   synthesis from HTTP properties). Reference deployment pattern.
   Wire example showing HTTP GET translated to AGTP FETCH.

4. **Cross-AGMP Interoperability renamed and broadened** to
   Cross-Composition Interoperability. New subsection covering the
   combined IdP-plus-Gateway case.

5. **Identifier rename applied throughout.** The `Principal-ID`
   wire header is renamed to `Owner-ID` in all AGMP mapping tables,
   mapping rules, and wire examples, tracking the locked AGTP
   taxonomy from {{AGTP-IDENTIFIERS}}.

6. **Security Considerations expanded** to cover the new profile
   families. Four new subsections: Bearer Token Leakage via
   Attribution-Record; External IdP Validator Confusion; HTTP
   Gateway Identity Synthesis; HTTP Gateway RCNS Bypass.

7. **Normative reference to {{AGTP}} updated to v08** (was v02
   in v00).

8. **Informative references added** to AGTP-API (since split into
   {{AGTP-METHODS}} and {{AGTP-RCNS}}),
   {{AGTP-IDENTIFIERS}}, {{AGTP-TRUST}}, {{AGTP-LOG}},
   {{AGTP-PRESENCE}}, {{AGTP-LEI}}, {{AGTP-COMMERCE}}, and
   {{AGTP-DISCOVERY}}, reflecting the broader AGTP draft family
   that has emerged since v00 of this document.

9. **Normative reference to {{RFC9110}}** added to support the
   Authorization header semantics in the External IdP composition
   profile. Informative references to {{RFC6749}} (OAuth 2.0) and
   {{RFC6750}} (Bearer tokens) added.

10. **Method floor references updated.** Where v00 referenced the
    12-method or 16-method floor, v01 references the current
    18-method floor per {{AGTP}}.

## Composition Profile Boundary Clarifications

This document covers three composition profile families. Other
composition surfaces specific to AGTP companion specifications are
addressed in those specifications rather than duplicated here:

- Payment provider composition is specified in {{AGTP-COMMERCE}}.
- Credit bureau and trust provider composition is specified in
  {{AGTP-TRUST}}.
- Certificate authority composition is specified in {{AGTP-CERT}}.
- Institutional identity composition (LEI/vLEI) is specified in
  {{AGTP-LEI}}.

## Wire Format Compatibility

The v01 changes are additive (new profile families, expanded
security considerations, broader scope) and identifier-alignment
(Principal-ID to Owner-ID rename). No AGMP composition wire format
changes are introduced beyond the identifier rename that tracks the
broader AGTP taxonomy update. Implementations conforming to v00
require the Owner-ID rename to interoperate with v01 implementations
and AGTP v08 servers.

--- back

# Acknowledgments

The AGMP composition profiles in this document build on the foundational
work of the MCP, A2A, and ACP specification teams. AGTP is designed to
be additive to and compatible with their work rather than competitive
with it.
Thank you to Anthropic, Linux Foundation A2A team, and IBM ACP.

The External Identity Provider composition profile builds on the
foundational work of the OAuth and OIDC communities. The composition
treats AGTP identity and external IdP credentials as orthogonal axes
that compose cleanly without modification to either.

The HTTP Gateway composition profile addresses the practical reality
that many existing clients communicate over HTTP. The translation
contract specified here makes the adoption ramp interoperable across
implementations without requiring HTTP to be modified.
