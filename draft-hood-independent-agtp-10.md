---
title: "Agent Transfer Protocol (AGTP)"
abbrev: "AGTP"
docname: draft-hood-independent-agtp-10
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agentic systems
  - protocol
  - agent traffic
  - agent transfer

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
  RFC3986:
  RFC6335:
  RFC7515:
  RFC8615:
  RFC9000:
  RFC9110:
  RFC5280:
  RFC8126:
  RFC8179:
  RFC9334:
  RFC8785:
  AGTP-METHODS:
    title: "AGTP Methods (AGTP Verbs): Method Vocabulary, Path Grammar, and Endpoints"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-methods-00
    date: 2026
  AGTP-RCNS:
    title: "AGTP-RCNS: Runtime Contract Negotiation Substrate"
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
  AGTP-IDENTIFIERS:
    title: "AGTP Identifier Chain"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identifiers-02
    date: 2026

informative:
  RFC9114:
  RFC7540:
  RFC6749:
  RFC7519:
  RFC7595:
  RFC9162:
  RFC9943:
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
  AGTP-CERT:
    title: "AGTP Agent Certificate Extension"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-agent-cert-03
    date: 2026
  HOOD2026:
    title: "Semantic Method Naming and LLM Agent Accuracy: A Controlled Benchmark of REST/CRUD versus Agentive API Interface Design"
    author:
      fullname: Chris Hood
    date: 2026
    seriesinfo:
      Working Paper: "Available by request. March 2026."
  AGTP-COMMUNICATION:
    title: "AGTP Communication Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-communication-00
    date: 2026
  AGTP-SESSION:
    title: "AGTP Session Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-session-01
    date: 2026
  AGTP-API:
    title: "AGTP-API: Verbs, Paths, Endpoints, and Synthesis"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-api-01
    date: 2026
  AGTP-DISCOVERY:
    title: "AGTP Discovery and Naming"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-discovery-01
    date: 2026
  AGTP-COMPOSITION:
    title: "AGTP Composition Profiles: Agent Group Messaging Protocols, External Identity Providers, and HTTP Gateways"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-composition-01
    date: 2026
  AGTP-BINDINGS:
    title: "AGTP Transport Bindings: TCP/TLS and QUIC"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-bindings-00
    date: 2026
  AGTP-MERCHANT:
    title: "AGTP Merchant Identity and Agentic Commerce Binding"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-merchant-identity-02
    date: 2026
  AGTP-LOG:
    title: "AGTP Transparency Log Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-02
    date: 2026
  AGTP-TRUST:
    title: "AGTP Trust and Verification Specification"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-trust-02
    date: 2026
  AGTP-PRESENCE:
    title: "AGTP Presence: Ambient Discovery and Visibility for Agent Substrates"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-presence-00
    date: 2026
  AGTP-LEI:
    title: "AGTP-LEI: Binding the Agent Transfer Protocol to the Verifiable Legal Entity Identifier"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-lei-00
    date: 2026
  AGTP-COMMERCE:
    title: "AGTP-Commerce: Open Commerce Specification for Agent-to-Agent Transactions"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-commerce-00
    date: 2026

--- abstract

This document defines the Agent Transfer Protocol (AGTP): a transfer
protocol for AI agent traffic. AGTP carries agent identity, authority,
delegation, and attribution as protocol elements of every exchange,
on an eighteen-method floor of intent-named methods, over TCP/TLS or
QUIC on IANA-assigned port 4480 and the IANA-registered `agtp://` URI
scheme. The method vocabulary beyond the floor, the path grammar, and
the endpoint contract are specified in {{AGTP-METHODS}}; runtime
negotiation of endpoints is specified in {{AGTP-RCNS}}; the agent
identity document in {{AGTP-IDDOC}}; transport bindings in
{{AGTP-BINDINGS}}. Version 10 adds per-request proof of possession, so that a verified
identity document and a live sender are bound by signature; specifies
the Delegation-Chain header with pre-authorization semantics (per-hop
scope attenuation checked at mint and at verify, parent
re-verification at each hop, depth and cycle limits, and defined
failure behavior); maps the delegation model onto the WIMSE chain
convention; resolves the 262 and 455 status codes by obtainability;
adds structured failure fields to the status code model, states the protocol's position in
the transfer-protocol vocabulary throughout, and moves module
material to companion specifications so that this document carries
the protocol and only the protocol. AGTP is designed to compose with
existing agent frameworks rather than replace them.

--- middle

# Introduction

## Conventions and Specification Status {#conventions}

AGTP is a transfer protocol. A transfer protocol defines the
semantics of an exchange: its methods, status codes, header fields,
and, in AGTP, the identity, authority, delegation, and attribution
carried with every exchange. A transport protocol delivers the bytes.
TCP, TLS, and QUIC are transport; AGTP runs over them as specified in
{{AGTP-BINDINGS}}. Throughout this document "transport" refers to
those protocols and never to AGTP, and "substrate" names the role
AGTP plays beneath agent group messaging protocols.

Every mechanism this document names is in one of four states, and the
text says which when the state is anything other than specified:

Specified:
: The mechanism is defined normatively in this document.

Specified and implemented:
: Specified, and present in the reference implementation at
  github.com/nomoticai/agtp.

Planned:
: Named, with its carrier reserved, and specified in a later
  revision. A planned mechanism is never a current guarantee.

Inherited:
: Provided by a companion specification or an external standard. The
  dependency is named and the behavior when it is absent is stated.

A header field or body field is a carrier. A carrier by itself implies
no claim; the claim exists when the verifier, the verification rule,
and the failure behavior are specified.

**Note Regarding Intellectual Property:**  Implementers should be
aware that extensions and certain mechanisms referenced in this
document (the Agent Certificate extension of {{AGTP-CERT}}, the
ACTIVATE method, the Agent Genesis mechanism, and the `.agent` and
`.nomo` file format specifications of {{AGTP-IDDOC}}) may be subject
to pending patent applications by the author. The core AGTP
specification is intended for open implementation without royalty
obligation. The licensor is prepared to grant a royalty-free license
to implementers consistent with {{RFC8179}}. IPR disclosures:
https://datatracker.ietf.org/ipr/; see also the Intellectual
Property Considerations subsection of the Security Considerations.


## Background

The deployment of AI agents and multi-agent systems is accelerating
across enterprise, research, and consumer contexts. These systems execute
complex, multi-step workflows, querying data sources, booking resources,
delegating subtasks to peer agents, and escalating decisions to human
principals, with minimal or no human supervision per transaction.

Unlike human-initiated web traffic, agent-generated traffic is dynamic,
high-frequency, intent-driven, and often stateful across sequences of
related requests. The infrastructure carrying this traffic was never
designed with these properties in mind.

## Limitations of HTTP for Agent Traffic

HTTP has served as the internet's primary application-layer transport for
over three decades. Its evolution through HTTP/2 {{RFC7540}} and HTTP/3
{{RFC9114}} has improved performance, multiplexing, and latency. However,
the fundamental model of HTTP being stateless, resource-oriented,
human-initiated request/response, creates specific failures when applied
to agentic systems at scale:

- Traffic indistinguishability: Agent-generated requests are structurally
  identical to human-initiated requests on the wire. Operators
  cannot identify, route, or govern agent traffic without application-layer
  instrumentation.

- Method vocabulary mismatch: HTTP's method set (GET, POST, PUT, DELETE,
  PATCH) describes resource operations. Agent traffic expresses purposeful
  intent, summarize, book, delegate, escalate. The mismatch forces intent
  into request bodies, invisible to protocol-level handlers.

- Identity and attribution absence: HTTP carries no native mechanism for
  asserting agent identity, declared authority scope, or the principal
  accountable for an agent's actions.

- Session semantics mismatch: HTTP's stateless model is optimized for
  isolated request/response cycles. Agent workflows are inherently stateful
  sequences.

## Why Not Evolve HTTP? {#why-not-http}

A natural question is whether these limitations could be addressed by
extending HTTP rather than defining a new protocol. There are three
specific reasons why HTTP extension is the weaker path.

First, the wire image. HTTP's method registry is extensible under
{{RFC9110}}, and new methods can be registered; the constraint is on
the deployed base rather than the registry. An intent-named method
such as SUMMARIZE, DELEGATE, or ESCALATE registered for HTTP would
reach every HTTP client, proxy, cache, and middleware component as an
unknown method, each handling it by its own default. AGTP instead
presents a distinct wire image, with its own URI scheme and port, so
that an intermediary either understands agent traffic or sees that it
fails to, and nothing in the HTTP deployed base is asked to change
its behavior. The case rests on this distinctness and on the failure
cases documented in {{problem-statement}}, rather than on any limit of HTTP's
registry.

Second, HTTP carries decades of backward-compatibility constraints. Features
such as persistent agent identity headers, authority scope declarations, and
session-level governance semantics would require HTTP extensions that interact
unpredictably with existing caching, proxy, and CDN behavior designed for
human-generated traffic patterns.

Third, the observability goal making agent traffic distinguishable from
human traffic at the infrastructure layer cannot be achieved by adding
fields to HTTP. Infrastructure components route and filter HTTP traffic
based on methods and headers that are identical across agent and human
requests. A protocol-level separation is necessary to give infrastructure
the signal it needs.

AGTP is therefore designed as a dedicated protocol rather than an HTTP
extension. HTTP and AGTP coexist: human traffic continues to flow over
HTTP; agent traffic flows over AGTP. The two protocols serve different
classes of network participant.

Note: The abbreviation AGTP is used in this document to distinguish
the Agent Transfer Protocol from the Authenticated Transfer Protocol
(ATP) working group currently chartered within the IETF. The URI
agtp:// is proposed for IANA registration as a new and distinct scheme.

## Motivation for a Dedicated Protocol

These limitations are architectural rather than implementational. They cannot be
resolved by better middleware or application code layered on HTTP. They
require a protocol designed from first principles for AI agent systems.

AGTP is that protocol. It provides a dedicated transport environment for
agent traffic with: native intent-based methods, mandatory agent identity
headers, protocol-level authority scope declaration, and a status code
vocabulary for the conditions AI systems encounter.

## Scope and Target Audience

This document covers AGTP architecture, design principles, stack position,
request and response header format, agent-native method definitions and
semantics, status code vocabulary, security considerations, and IANA
considerations.

The Agent Certificate extension for cryptographic binding of agent
identity to AGTP header fields is described at a high level in
{{verification-levels}}.
Full specification is provided in a separate companion document:
{{AGTP-CERT}}. That extension may be subject to pending intellectual
property claims; see {{ipr}} and the IPR Notice preceding the Abstract.

Merchant-side identity verification for PURCHASE counterparties is
described at a high level in the Merchant Identity section of this
document and specified
in full in a separate companion: {{AGTP-MERCHANT}}. This document
registers the merchant-related request headers, the 458 Counterparty
Unverified status code, and the `merchant` and `intent` Authority-Scope
domains; the Merchant Manifest Document, Merchant Agent Genesis,
counterparty verification procedure, and Intent Assertion JWT format
are specified in the companion.

Target audience: AI agent developers, protocol designers, cloud and network
infrastructure providers, enterprise security and compliance architects, and
standards community participants.

## AGTP as the Transport Foundation for Agent Group Messaging Protocols

AGTP is the purpose-built transport and governance layer for Agent Group
Messaging Protocols (AGMPs): the category of higher-layer AI agent messaging
standards that includes the Model Context Protocol (MCP) {{MCP}}, the
Agent-to-Agent Protocol (A2A) {{A2A}}, the Agent Communication Protocol
(ACP) {{ACP}}, and emerging others.

AGMPs define what agents say. AGTP defines how those messages move, who
sent them, and under what authority. AGTP provides the narrow-waist
foundation that AGMPs inherit without modification: intent-native methods,
mandatory agent identity and scoping, resource budget enforcement,
observability hooks, and normative composition profiles. A deployment
running any AGMP over AGTP gains transport-level governance without changes
to the messaging layer.

The AGMP category term is introduced in this document to provide a stable
collective reference for the class of protocols that AGTP serves as
substrate. It is a descriptive label rather than a formal IETF term of art, a descriptive
classification. Individual AGMP specifications retain their own names and
development paths. AGTP leaves every AGMP ungoverned, unmodified, and in force.

~~~~
+-----------------------------------------------------+
|            Agent Application Logic                  |
+-----------------------------------------------------+
|  AGMP Layer: MCP / A2A / ACP / ANP  [optional]      |
+-----------------------------------------------------+
|   AGTP - Agent Transfer Protocol      [this spec]    |
+-----------------------------------------------------+
|            TLS 1.3+                  [mandatory]    |
+-----------------------------------------------------+
|         TCP / QUIC / UDP                            |
+-----------------------------------------------------+
~~~~
{: #agmp-stack title="AGTP as Substrate for AGMPs"}

# Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when,
and only when, they appear in all capitals.

Agent:
: An AI software system that executes tasks, makes decisions, and takes
  actions without continuous human supervision per transaction.

Principal:
: The human, organization, or system that authorized an agent to act and
  is accountable for its actions.

Agent-ID:
: A unique identifier for a specific agent instance. Carried in the
  `Agent-ID` request header on non-anonymous AGTP requests, and in
  the `agent_id` field of the Agent Identity Document.

Owner-ID:
: The identifier of the principal (human, organization, or service) on
  whose behalf an agent operates. Carried in the agent identity document
  referenced by `Agent-ID`; also surfaced as the `Owner-ID` request
  header when explicit wire-layer carriage is needed. The locked
  identifier name throughout the AGTP family is `Owner-ID`; earlier
  drafts used `Principal-ID`, now retired. See {{AGTP-IDENTIFIERS}}.

Authority-Scope:
: A declared set of permissions defining what actions an agent is
  authorized to take, in the format `domain:action` or `domain:*`.
  Declared in the agent's identity document. **MAY** be carried on
  individual requests as a claimed-scopes header narrowing the
  agent's full authorized set to those needed for the request;
  claimed scopes **MUST** be a subset of the document's declared
  set.

Intent Method:
: An AGTP method name expressing the agent's purpose, as distinguished from
  HTTP resource-operation verbs.

Delegation Sequence:
: The ordered set of DELEGATE invocations, each a signed hop from a
  delegating agent to a sub-agent, that produced the current request.
  Reconstructable from the hop records and verified per {{delegation}}.

Delegation-Chain:
: The request header that carries the Delegation Sequence as an
  ordered list of canonical Agent-IDs, origination to current, and
  the per-hop delegation tokens. Specified in {{delegation}}.

Escalation:
: An agent's intentional deferral of a decision or action to a human
  principal or higher-authority agent.

Attribution Record:
: A logged record of an agent action sufficient for audit and compliance
  purposes.

Session:
: An AGTP persistent connection context shared across multiple method
  invocations within a single agent workflow.

SEP (Scope-Enforcement Point):
: An AGTP-aware infrastructure component, load balancer, gateway, and proxy,
  that enforces Authority-Scope compliance without application-layer access.
  Requires the Agent Certificate extension ({{AGTP-CERT}}).

Agent Package (.agent):
: A portable, open deployment artifact for an AI agent. An `.agent` file
  contains an embedded Agent Manifest, an integrity hash covering all
  package contents, and a behavioral trust score computed at packaging
  time. The `.agent` format is an open specification. It is analogous to
  a container image: a self-describing, portable unit of deployment. The
  `.agent` suffix is a file format designator and **MUST NOT** appear as
  a hostname component or top-level label in `agtp://` URIs. Note: the
  `.agent` file format specification may be subject to pending patent
  claims by the author; see {{ipr}}.

Governed Agent Package (.nomo):
: A deployment artifact in the `.nomo` format, which extends the `.agent`
  format with a CA-signed certificate chain binding the package to a
  verified governance zone and issuing principal. The `.nomo` format is
  to `.agent` as HTTPS is to HTTP: the same structural foundation with
  an added layer of cryptographic trust. A `.nomo` package is required
  for agents operating at Trust Tier 1 (see {{verification-paths}}). The `.nomo`
  suffix is a file format designator and **MUST NOT** appear as a
  hostname component in `agtp://` URIs.

  The name derives from the Greek *nomos* (νόμος), meaning law, rule,
  or governance, the same root that underlies *autonomy* (self-law),
  *nomocracy* (rule of law), and *onomastics*. A `.nomo` package is
  literally an agent operating under law: its behavior is bounded by a
  cryptographically enforced governance context at the packaging layer.
  Note: the `.nomo` file format specification may be subject to pending
  patent claims by the author; see {{ipr}}.

Agent Transfer Document (.agtp):
: The wire-level manifest document format defined by this specification.
  An `.agtp` document is a signed JSON structure containing the fields
  defined in {{AGTP-IDDOC}} (Agent Identity Document). It is the output
  format returned by all AGTP URI resolution requests. Both `.agent` and
  `.nomo` packages produce `.agtp` documents when queried; the `.agtp`
  format is the protocol's canonical representation of agent identity
  and is independent of the underlying packaging format. The `.agtp`
  suffix **MAY** appear in filenames for stored manifest documents but
  **MUST NOT** appear in `agtp://` URIs. The Content-Type for `.agtp`
  documents is `application/vnd.agtp+json`.

URI (AGTP):
: An `agtp://` scheme URI that identifies an agent or agent namespace.
  AGTP URIs are addresses rather than filenames. File extensions (`.agent`,
  `.nomo`, `.agtp`) **MUST NOT** appear in canonical AGTP URIs. See
  {{uri-forms}} for the canonical URI forms and resolution semantics.

Agent Namespace Document:
: The signed `application/vnd.agtp.identity+json` document returned in
  response to a request targeting an organization's agent registry
  root (for example `agtp://acme.tld/agents`). Lists every Active agent
  registered under the organization's governance zone. Generated and
  re-signed by the governance platform on any registry change; never a
  manually edited file. Specified in {{AGTP-IDDOC}}; see
  {{document-taxonomy}}.

Agent Identity Document:
: The signed `application/vnd.agtp.identity+json` document a canonical
  Agent-ID resolves to, returned in response to a request targeting a
  specific agent (for example `agtp://acme.tld/agents/customer-service`).
  Carries the Agent Genesis fields, lifecycle state, behavioral trust
  score, authority scope set, supported methods, and governance zone.
  Derived from the agent's `.agent` or `.nomo` package, whose integrity
  hash is verified before the document is served, and verified by the
  recipient before any field is trusted. Mutable: re-signed on every
  change. Specified in {{AGTP-IDDOC}}; see {{document-taxonomy}}.

Agent Genesis:
: A cryptographically signed origin document issued to an agent at
  registration time by a governance platform. The Agent Genesis is
  the genesis record of an agent's existence: it establishes the
  agent's identity, ownership, authorized scope, behavioral archetype,
  and governance zone before the agent takes any action. Authority is
  issued through the Agent Genesis; it is never self-assumed.

  The Agent Genesis is the source document from which the Agent
  Identity Document ({{AGTP-IDDOC}}) is derived when an AGTP URI is
  resolved. The canonical Agent-ID is computed as the 256-bit
  SHA-256 hash of the Agent Genesis in its canonical-form
  serialization with the signature field excluded; the canonical
  form is specified in {{agent-genesis-contents}}. In this sense
  the Agent Genesis functions as the agent's permanent origin
  record: issued once at creation, permanently bound to the
  agent, and the authoritative identity record from which all
  other identity representations derive.

  Agent Genesis fields map to AGTP protocol elements: `agent_id`
  maps to the `Agent-ID` header on every request; `owner` is the
  principal identifier recorded in the agent identity document
  referenced by `Agent-ID` (never a separate header); `scope` is the
  Authority-Scope set declared in the agent identity document,
  optionally narrowed per-request via the `Authority-Scope` header.
  See {{agent-genesis-contents}}.

  Anonymous agents are ungovernable. Without an Agent Genesis, there
  is no mechanism to trace decisions to a responsible principal,
  enforce scope boundaries, or maintain a meaningful audit trail.

  The taxonomy is: **Agent Genesis** (the permanent signed governance-
  layer origin document) → **canonical Agent-ID** (the 256-bit hash
  of the Agent Genesis, used in all AGTP protocol operations) →
  **Agent Certificate** (an optional X.509 v3 credential for TLS
  mutual authentication; specified in {{AGTP-CERT}}). Note: the
  Agent Genesis mechanism may be subject to pending patent claims
  by the author; see {{ipr}}.

Governance Token:
: A signed, time-limited JWT artifact issued by a governance runtime
  that encodes a specific governance decision for a specific action.
  Governance tokens are the runtime companion to the static Agent
  Genesis: where the Agent Genesis establishes persistent
  identity, the Governance Token carries a bounded authorization for a
  single action or session. Tokens carry the governance verdict
  (ALLOW, DENY), the agent ID, action details, trust score dimensions,
  issuer identity, and expiry. Default TTL: 30 seconds. Tokens
  **MUST NOT** be reused across actions; each action requires a fresh
  evaluation and a fresh token.

Trust Tier:
: A classification assigned to an agent based on the strength of
  identity verification backing its registration. Tier 1 (Verified):
  org anchor is a real DNS domain with confirmed ownership and a `.nomo`
  governed package. Tier 2 (Org-Asserted): org label is present but
  DNS ownership is unverified; `.agent` package acceptable. Tier 3
  (Experimental): X- prefix required; undiscoverable through the
  public AGTP registry. See {{verification-paths}}.

AGMP (Agent Group Messaging Protocol):
: The collective term for higher-layer AI agent messaging standards
  that operate over AGTP as their transport substrate, including
  MCP {{MCP}}, A2A {{A2A}}, ACP {{ACP}}, and ANP {{ANP}}. AGMPs
  define what agents say to each other. AGTP defines how those
  messages move. The term is introduced in this document as a
  descriptive classification rather than a formal IETF term of art.

DESCRIBE:
: An AGTP cognitive floor method that returns the declared capabilities,
  supported modalities, method vocabulary, and versioned feature set
  of a specific agent endpoint. Distinguished from URI resolution
  (which returns identity) by returning operational capability
  metadata suitable for pre-task negotiation. If the
  `capability_domains` parameter is omitted, the server **SHOULD**
  return all supported domains. Category: ACQUIRE.

SUSPEND (method):
: An AGTP mechanics floor method that places a specific active session
  workflow into a recoverable paused state, issuing a resumption
  nonce for re-entry. Distinguished from the lifecycle SUSPEND event
  ({{registration-lifecycle}}): method-level SUSPEND is session-scoped and
  leaves the agent's registry lifecycle state and Agent Genesis unchanged
  validity. Category: ORCHESTRATE.

Budget-Limit:
: A request header declaring the maximum resource consumption the
  principal authorizes for a method invocation, expressed as
  comma-separated `unit=value` tokens drawn from the IANA AGTP Budget
  Unit Registry per {{RFC9110}} list-valued header conventions.
  Example: `Budget-Limit: tokens=5000, compute-seconds=120,
  financial=10.00USD, ttl=3600`. Exceeding the declared limit
  **MUST** cause the server to return 456 Budget Exceeded rather
  than continue execution. Note: ttl= is RECOMMENDED to bound budget
  lifetime. Reserved for v01+ per {{header-format}}.

AGTP-Zone-ID:
: A request header declaring the network zone or organizational
  boundary within which a request must be processed. Scope-Enforcement
  Points (SEPs) **MUST** enforce zone boundaries and **MUST** return
  457 Zone Violation if a DELEGATE request would route
  outside the declared zone.

Identity-first architecture:
: The architectural commitment that the canonical Agent-ID
  (256-bit content-addressed identifier) is the authoritative
  identity primitive in AGTP. Hosting, DNS anchors, and other
  resolution paths are aliases.

Intent Verb:
: An AGTP method name expressing the agent's purpose, as distinguished
  from HTTP resource-operation verbs (GET, POST, PUT, DELETE). The
  vocabulary is specified in {{AGTP-METHODS}}.

Scope-Enforcement Point (SEP):
: An AGTP-aware infrastructure component (load balancer, gateway,
  proxy) that enforces Authority-Scope compliance on AGTP requests
  without application-layer access. Line-rate enforcement requires
  {{AGTP-CERT}}.

## Document Taxonomy {#document-taxonomy}

AGTP identity rests on three objects with one join key. The
canonical Agent-ID is the SHA-256 of the Agent Genesis, so the
Genesis is the root; the Agent Identity Document is what the Agent-ID
resolves to; the Attribution-Record chain is what the agent did,
anchored back to the Genesis through the Audit-ID of
{{AGTP-IDENTIFIERS}}.

| Object | Answers | Signed by | Changes | Specified in |
|---|---|---|---|---|
| Agent Genesis | who this agent is, permanently | issuing registrar | never; a change is a new agent | this document |
| Agent Identity Document | what this agent is now | the agent's issuer | re-signed on each change | {{AGTP-IDDOC}} |
| Agent Namespace Document | which agents a namespace hosts | namespace operator | re-signed on each registry change | {{AGTP-IDDOC}} |
| Attribution-Record and Audit-ID chain | what this agent did | the acting agent, per action | append-only | this document, {{AGTP-IDENTIFIERS}} |
{: title="AGTP Identity Objects"}

Two terms outside this taxonomy appear in related work. "Agent Birth
Certificate" is the retired name for the Agent Genesis; the object is
unchanged. "Agent Card" names the A2A self-description served at a
well-known HTTPS path and trusted through its domain; it is
self-asserted, unsigned, and carries no lifecycle state or enforced
scope. An Agent Identity Document maps to an Agent Card for
interchange with a loss of those properties ({{AGTP-DISCOVERY}}
specifies the mapping); an Agent Card maps to an Identity Document
only at Level 1, with no issuer. The two are different objects and
this family uses neither name for the other.

# Architectural Principles {#architectural-principles}

AGTP's design follows five architectural principles that distinguish it
from incremental extensions to HTTP and from policy-layer approaches
to agent governance. These principles motivate the specific protocol
decisions in the rest of this document; they also explain why certain
vulnerability classes that affect HTTP-based agent infrastructure
are structurally impossible on AGTP, rather than merely mitigated.

The principles are stated abstractly so they apply beyond any specific
example (such as the BadHost vulnerability discussed in
{{related-work-section}}). A reviewer evaluating whether AGTP succeeds
at its design goals should ask whether the protocol satisfies all five
principles rather than whether it addresses any single concrete vulnerability.

## Substrate vs Application Layer Separation

AGTP provides substrate primitives. Adjacent layers (Agent Group
Messaging Protocols, external identity providers, HTTP gateways,
agent application logic) compose over the substrate. The substrate
**MUST NOT** rely on application-layer cooperation for properties it
provides; it **MUST** enforce them structurally at the transport
boundary.

This principle has direct consequences:

- Identity headers (Agent-ID, Owner-ID, Authority-Scope) are wire-
  layer facts the substrate verifies before dispatch to the
  application. They are never application-layer conventions an
  application is expected to honor.

- Authority enforcement happens at the protocol layer before
  application code runs. A scope violation returns 455 from the
  AGTP layer rather than from the application's authorization middleware.

- Attribution is produced by the substrate on every response. An
  application cannot opt out of attribution, fail to produce it, or
  produce a malformed attribution that infrastructure cannot read.

The substrate's correctness is independent of the application
implementing required behaviors. Vulnerabilities of the form
"middleware was supposed to check X but checked Y instead" cannot
arise in the substrate layer because the substrate enforces X
structurally, before any middleware runs.

## Identity at the Wire

Agent identity is a wire-layer fact in AGTP rather than an application-layer
convention reconstructed from request properties.

Every AGTP request carries Agent-ID and Authority-Scope as structured
headers verified at the AGTP layer, with Owner-ID resolved from the
identity document the Agent-ID references ({{AGTP-IDDOC}}). Infrastructure
components (load balancers, Scope-Enforcement Points, governance
gateways) read these directly from the wire. They never reconstruct
identity from hostnames, paths, query parameters, or application
payloads. They never parse application content to determine which
agent sent a request.

This principle eliminates an entire vulnerability class: bugs where
application-layer code reconstructs identity (or authority, or
routing decisions) from request properties that an attacker can
manipulate. The reconstruction surface is absent in AGTP because
identity is read from headers the protocol parses as identity rather than
inferred from headers the protocol parses for other purposes.

Vulnerabilities affecting HTTP-based agent infrastructure that depend
on host header manipulation, URL reconstruction, path-based
authorization middleware, or similar reconstruction patterns are
structurally impossible on AGTP because AGTP never reconstructs
identity from those properties.

Agent identity and workload identity are different concerns that
compose. A workload identity credential of the kind the IETF WIMSE
working group specifies authenticates the workload an agent runs on;
the AGTP agent identity, bound to a certificate through
{{AGTP-CERT}}, persists across the workloads that host it. A mature
deployment carries both, and neither substitutes for the other.

AGTP addresses both roles the agent literature distinguishes: the
user agent, which acts for a person and owes that person loyalty,
and the service agent, which acts for an organization and offers a
capability to others. Every mechanism in this document holds for
both. What differs is the Owner-ID the Identity Document carries, a
person or an organization, and the governance policy that owner sets.
The loyalty, protection, and honesty obligations of a user agent are
that policy's content; the protocol carries the accountable owner
and leaves the obligations to the owner and the governance platform.

## Heteronomy: Agents Are Created by Humans

Agents lack autonomy in the strong sense. Every agent has a
creator, an owner, and a governance context established outside the
agent's own operation. AGTP's identity primitives reflect this fact
structurally.

- The Agent Genesis is signed by a registrar before any agent traffic
  exists. The Agent Genesis is the permanent governance-layer record
  of the agent's existence; the agent never creates its own
  Genesis.

- The Owner-ID identifies the human or organizational principal
  accountable for the agent. The owner is recorded on the Agent
  Genesis at registration rather than asserted by the agent at runtime.

- The Authority-Scope is granted by a principal at registration rather than
  claimed by the agent at runtime. An agent **MAY** narrow its claimed
  scope on a specific request below its declared scope; an agent
  **MUST NOT** claim scope beyond what its Genesis grants.

- The Agent Certificate is issued by a Certificate Authority that
  validates the agent's Genesis-anchored identity. The certificate
  confirms rather than asserts identity: it cryptographically confirms an identity
  established outside the certificate.

This architectural principle distinguishes AGTP from approaches that
treat agents as self-asserting entities. An agent cannot register
itself; an agent cannot grant itself authority; an agent cannot issue
its own certificate. The trust root is the human or organizational
authority that created the agent. AGTP makes this trust root visible
at the wire layer through the identifier chain (Agent-ID, Owner-ID,
issuing registrar).

The implications for governance, attribution, and accountability are
substantial. An agent operating outside its granted authority is never
a protocol failure (the protocol structurally prevents the
out-of-scope action); it is a governance failure traceable to the
principal who granted the authority, the registrar who recorded it,
or the operator who deployed the agent. Each is identified at the
wire layer.

## Structural Enforcement, Not Policy Enforcement

Security properties in AGTP are structural --- they hold because the
protocol's design eliminates the architectural pattern that creates
the vulnerability rather than because operators enforce a policy correctly
or middleware checks the right field.

The distinction matters in two ways:

**Vulnerability impossibility vs vulnerability mitigation.** A
structural property holds because the protocol cannot express the
vulnerable configuration. A policy property holds because operators
correctly configure and enforce it; misconfiguration restores the
vulnerability. AGTP's design prefers structural properties wherever
the architecture allows.

**Defense in depth vs defense by configuration.** AGTP composes with
application-layer security (input validation, application-specific
authorization, business logic enforcement). AGTP leaves in place
these. AGTP provides structural primitives that compose cleanly with
application-layer security so that the combined system is stronger
than either layer alone.

Concrete example: the BadHost vulnerability class (CVE-2026-48710
and similar) exists because HTTP-based agent infrastructure
reconstructs URLs from the Host header and operates path-based
authorization on the reconstruction. AGTP eliminates this vulnerability
class by lacking a Host header altogether rather than by checking it more carefully, by lacking
the Host header entirely, by parsing path and query as separate wire-
layer tokens rather than reconstructing URLs, and by enforcing
authorization on Authority-Scope rather than path. The vulnerable
configuration cannot be expressed on AGTP because AGTP lacks
the architectural pattern that produces the vulnerability.

This principle applies to other vulnerability classes as well: scope
escalation through header manipulation, attribution forgery through
log gaps, and identity spoofing through certificate confusion are
addressed structurally rather than through policy enforcement.

## Substrate Carries Its Own Operational Properties

Identity, authority, attribution, delegation chain, trust posture,
and session state are properties of the substrate itself rather than
properties layered on top by applications.

This principle enables uniform infrastructure handling regardless of
which adjacent layer is composed. An infrastructure component
(load balancer, Scope-Enforcement Point, governance gateway,
monitoring system) processes AGTP traffic identically whether the
payload is an MCP tool call, an A2A task, an OAuth-credentialed
request, an HTTP gateway translation, or a future protocol yet to be
designed. The substrate's operational properties are visible to
infrastructure at the wire layer.

The principle has economic and operational consequences:

- **One governance layer covers all agent traffic.** Operators never
  need separate enforcement for each AGMP. The AGTP governance layer
  handles MCP, A2A, ACP, ANP, and any future protocols uniformly.

- **Audit and attribution compose across protocols.** An attribution
  record produced by an EXECUTE invocation carrying an MCP tool call
  has the same structure as an attribution record produced by an
  EXECUTE invocation carrying an A2A task. Audit consumers never
  need protocol-specific parsing.

- **Future-proof composition.** Adjacent layers yet to be defined can
  compose with AGTP using the existing substrate properties. New
  AGMPs, new identity providers, new translation patterns benefit
  from substrate properties without negotiating each property anew.

Adjacent layers benefit from substrate properties without needing to
implement them. This is why composition works: the substrate is the
load-bearing layer for identity, authority, attribution, and audit;
adjacent layers focus on what they actually contribute (tool
semantics, conversational structure, authorization framework, REST
translation).

# Problem Statement {#problem-statement}

AGTP is motivated by three distinct, compounding failures in how current
internet infrastructure handles AI agent traffic.

## Problem 1: Undifferentiated Agent Traffic on HTTP

AI agents generate intent-driven, structured traffic that is functionally
invisible to the infrastructure it traverses. This traffic flows through
HTTP alongside human traffic with no protocol-level differentiation.
Observability failure, routing inefficiency, and security blindness result,
operators cannot determine what fraction of traffic is agent-generated
without application-layer instrumentation that is expensive, inconsistent,
and easy to circumvent.

AGTP response: a dedicated protocol environment for agent traffic.
Infrastructure can distinguish, route, monitor, and govern agent traffic
natively.

## Problem 2: Semantic Mismatch Between Agent Intent and Available Methods

AI agents operate on intent. HTTP's method vocabulary was designed to
describe operations on resources rather than purposeful action. When an agent
intends to SUMMARIZE a document, EXECUTE a reservation, and PLAN a sequence,
all three arrive as POST requests. The server receives identical verbs
with meaningfully different intent buried in request bodies, invisible to
any protocol-level handler.

AGTP response: a vocabulary of agent-native methods that express intent at
the protocol level.

## Problem 3: No Protocol-Level Identity, Authority, or Attribution for Agents

When an AI agent takes an action, there is currently no protocol-level
mechanism to verify who authorized this agent, what scope of authority it
holds, which principal is accountable for its actions, or whether it is
the agent it claims to be. Accountability gaps, authority laundering,
auditability failure, and multi-agent trust collapse result.

AGTP response: agent identity and authority scope embedded in protocol
headers on every request, with an optional Agent Certificate extension for
cryptographic verification.

## Problem Summary

| # | Problem | Current Failure | AGTP Response |
|---|---------|----------------|--------------|
| 1 | Undifferentiated traffic | HTTP cannot separate agent traffic | Dedicated protocol environment |
| 2 | Semantic mismatch | HTTP verbs obscure agent intent | Native intent-based method vocabulary |
| 3 | No protocol-level identity | Attribution is untraceable | Agent identity and scope in headers |
{: title="Summary of Problems Addressed by AGTP"}

# Related Work and Existing Approaches {#related-work-section}

## HTTP/REST as the De Facto Standard

HTTP remains the universal transport for all agent traffic currently
deployed. REST conventions layered on HTTP provide a degree of semantic
structure, but REST remains a resource-manipulation paradigm. As described
in {{why-not-http}}, evolving HTTP to address agent-specific needs is
constrained by the frozen method registry, backward-compatibility
requirements, and the impossibility of achieving infrastructure-level
traffic differentiation through HTTP extensions alone.

## Existing Agent Group Messaging Protocols

MCP {{MCP}} (Model Context Protocol, Anthropic):
: Defines structured communication between AI models and tools/resources.
  Runs over HTTP. Addresses tool-calling semantics rather than agent traffic
  transport.

ACP {{ACP}} (Agent Communication Protocol, IBM):
: Defines messaging semantics for agent-to-agent communication. Runs over
  HTTP.

A2A {{A2A}} (Agent-to-Agent Protocol, Linux Foundation):
: Defines inter-agent communication and task delegation semantics. Runs
  over HTTP.

ANP {{ANP}} (Agent Network Protocol):
: Defines discovery and communication for networked agents. Runs over HTTP.

All of these are messaging protocols. They define what agents say to each
other. None of them defines how agent traffic moves across a network. Each
presupposes HTTP as its transport and inherits all of HTTP's limitations
for agentic systems.

## Transfer and Transport Alternatives

gRPC:
: High-performance RPC over HTTP/2. Strong typing and efficient
  serialization. Leaves agent-specific semantics, identity, and
  authority.

WebSockets:
: Persistent bidirectional connections over HTTP. Useful for real-time
  communication but leaves method semantics and identity unaddressed.

QUIC {{RFC9000}}:
: Modern multiplexed transport with reduced connection overhead.
  QUIC is a transport primitive; AGTP is the application-layer
  protocol above it. AGTP's QUIC transport binding is specified in
  {{AGTP-BINDINGS}}.

## The Critical Distinction: Messaging vs. Transfer

The most important positioning principle for AGTP is the distinction
between message formats and the transfer protocol that carries them.
MCP, ACP, A2A, and ANP are messaging protocols: they define what
agents say to each other. AGTP is the transfer protocol: it defines
the exchange that moves those messages, and the identity, authority,
and attribution carried with each one.

An analogy: HTTP transfers HTML. HTML defines what a page says; HTTP
defines the exchange that delivers it, with its own methods, status
codes, and headers. HTML cannot replace HTTP, and saying "HTTP is
unnecessary because HTML exists" is a category error. SMTP and the
Internet Message Format stand in the same relation. MCP and its peers
are the message formats of agent traffic. AGTP is the transfer
protocol, and TCP, TLS, and QUIC beneath it are the transport.

## Discovery and Naming

Agent discovery is specified in {{AGTP-DISCOVERY}} and
{{AGTP-PRESENCE}}, and this document carries only the identity
commitments those documents build on. The Canonical Agent-ID is
permanent identity; the current location of an agent is a
short-lived, signed, updatable record the substrate resolves it to.
Following RFC 6763, "discover" names the resolution of a known
identity to a reachable attachment, and "search" names the
capability-class query that returns candidate identities. An agent
name (a friendly label or a domain-anchored alias) and an agent
address (a URL or network address) are aliases for the Agent-ID
rather than the identity itself. Work on agent discovery in the IETF
DAWN effort is treated as adjacent in {{AGTP-DISCOVERY}}, which
specifies the mapping between the AGTP Agent Identity Document
({{AGTP-IDDOC}}) and the record shape the DAWN work describes agents
with (the A2A AgentCard in its initial revision).

## AGTP Positioning: The Proposed Stack

~~~~
+-----------------------------------------------------+
|            Agent Application Logic                  |
+-----------------------------------------------------+
|  Messaging Layer  (MCP / ACP / A2A)  [optional]     |
+-----------------------------------------------------+
|   AGTP - Agent Transfer Protocol     [this spec]    |
+-----------------------------------------------------+
|            TLS 1.3+                  [mandatory]    |
+-----------------------------------------------------+
|         TCP / QUIC / UDP                            |
+-----------------------------------------------------+
~~~~
{: #protocol-stack title="AGTP in the Protocol Stack"}

AGTP replaces no messaging protocol. Agents using MCP or A2A
route those messages over AGTP and gain transport-level observability and
identity without modifying the messaging layer. AGTP-native agents that
use no separate messaging protocol interact with AGTP methods directly.

# Protocol Overview

## Stack Position

AGTP is an application-layer protocol. It operates above the transport
layer and is wrapped by TLS. It sits below any agent messaging protocol
in deployments that use one.

AGTP is transport-neutral: AGTP semantics are independent of which
transport carries them. Transport bindings for TCP/TLS and QUIC are
specified in {{AGTP-BINDINGS}}, including the AGTP-specific replay-
safety profile that governs early-data use over QUIC. Selection
between supported transports is a deployment-time choice based on
operator infrastructure, counterparty ecosystem, and latency
requirements; this specification establishes no normative
preference between transports.

AGTP uses port **4480** (TCP and UDP), assigned by IANA under the
service names `agtp` (TCP/TLS) and `agtp-quic` (QUIC). The port
assignment is permanent and applies to all AGTP transports. Full IANA
registration metadata is documented in {{iana-ports}}.

## Design Principles

Minimalist core:
: The base spec defines only what is necessary for agent traffic
  differentiation, method semantics, and identity headers. Extensions
  belong in companion specifications.

Extensible by design:
: New methods are registered through an IANA-managed Method Registry.
  New header fields follow a defined extension convention. Additive
  changes require no version increment.

Agent-native:
: Every design decision assumes the initiating party is an AI system,
  never a human.

Secure by default:
: TLS 1.3 or higher is mandatory. Unencrypted AGTP connections **MUST**
  be rejected. Agent identity headers are present on every request.

Observable by design:
: Native metadata in every AGTP header provides the minimum information
  needed for routing, monitoring, and audit without application-layer
  instrumentation.

Composable:
: AGTP works alongside existing agent messaging protocols without requiring
  modification to those protocols.

Runtime contract negotiation:
: AGTP fixes the protocol surface at eighteen methods. Beyond that floor,
  the surface is open. An agent that needs an endpoint the
  server has yet to advertise proposes the endpoint via PROPOSE with an
  AGTP-METHODS endpoint definition; the server evaluates whether it can
  synthesize the endpoint from existing capabilities and either
  instantiates it as a session-scoped endpoint or refuses. The
  negotiation completes in a single round-trip. See {{rcns-section}}.

## AGTP as a Runtime Contract Negotiation Substrate {#rcns-section}

AGTP is a runtime contract negotiation substrate (RCNS). The
substrate fixes a floor of eighteen protocol-level methods (the
cognitive, mechanics, and lifecycle verbs of {{methods-section}});
beyond that floor, the method and endpoint surface a server presents
is declared by the operator in the vocabulary of {{AGTP-METHODS}} or
negotiated at runtime between the agent and the server through the
PROPOSE floor method, in a single round-trip, as specified in
{{AGTP-RCNS}}.

This document carries the kernel: the PROPOSE method
({{methods-section}}), the negotiation status codes 261, 262, 263,
461, 463, and 464 ({{status-codes}}), the reason vocabulary for 461
and 464, the `Allow-RCNS`, `Contract-Synthesized`, and
`Idempotency-Key` headers ({{header-format}}), and the `rcns_*`
lifecycle events ({{contract-lifecycle-events}}). The dispatcher
gate, delivery modes, contract scoping and lifecycle, recipe
versioning, and the inventory surfaces are specified in
{{AGTP-RCNS}}. A server that implements this document and
{{AGTP-METHODS}} without {{AGTP-RCNS}} is conformant and refuses
unregistered method and path pairs with 404 or 405.

The RCNS property is what makes composition with higher-level agent
frameworks ({{composition-section}}) tractable: a server that wants
to expose a framework-specific operation as a first-class AGTP
endpoint negotiates it rather than waiting for a registry.

## Connection Model

AGTP uses a persistent session model by default, reflecting the reality that
agents typically execute multi-step workflows rather than isolated single
requests. An AGTP session is established with a single TLS handshake
including agent identity assertion, persists across multiple method
exchanges, carries a Session-ID header identifying the agent's task
context, and terminates on explicit session close or inactivity timeout
(RECOMMENDED minimum: 60 seconds).

Per-request (stateless) mode is supported for constrained environments.
In stateless mode, agent identity headers **MUST** be present on every
individual request.

### Wire-Format Framing {#wire-framing}

AGTP requests and responses **MUST** be framed by an explicit
`Content-Length` header. `Content-Length` is the sole signal of request
and response completion. Receivers **MUST** treat the message as
complete when, and only when, the declared number of body octets has
been read after the header terminator.

Implementations **MUST NOT** rely on transport-level connection
termination as a request-completion signal in addition to
`Content-Length`. Transport-specific framing requirements, including
the prohibition on TLS socket-level half-close and the handling of
QUIC stream termination, are specified in {{AGTP-BINDINGS}}.
Implementations that require an explicit completion signal in
addition to `Content-Length` **MUST** rely on the AGTP session-close
semantics described in {{methods-suspend}}.

AGTP uses no chunked transfer encoding. Streaming method
responses are framed by repeated `Content-Length`-delimited messages
within a single AGTP session. Real-time media streams between agents
(voice, video, sensor data) are specified in {{AGTP-COMMUNICATION}},
which builds on the session establishment of {{AGTP-SESSION}}.

## Header Format {#header-format}

The AGTP wire format puts the protocol version and method on the
request line (`AGTP/1.0 METHOD PATH\r\n`) and the status code on
the response line (`AGTP/1.0 STATUS STATUS-TEXT\r\n`). Headers
**MUST NOT** carry information already present on those lines:
there is no `AGTP-Version` header, no `AGTP-Method` header, no
`AGTP-Status` header. The version and method are read from the
request line; the status is read from the response line.

### Request Line Grammar {#request-line}

The request line carries three tokens separated by single space
characters, terminated by CRLF:

~~~~
request-line   = agtp-version SP method SP request-target CRLF
agtp-version   = "AGTP/1.0"
method         = token                  ; per {{AGTP-METHODS}} catalog
request-target = path-absolute [ "?" query ]
                                        ; per RFC 3986
path-absolute  = "/" [ segment-nz *( "/" segment ) ]
                                        ; per RFC 3986 Section 3.3
query          = *( pchar / "/" / "?" )
                                        ; per RFC 3986 Section 3.4
~~~~

The `request-target` follows the path-absolute production of
{{RFC3986}}, optionally followed by `?` and a query string. Path
and query are parsed as separate tokens: the path is everything
from the leading `/` up to the first `?` or end-of-line; the query
string is everything after the first `?` up to the CRLF.

Servers **MUST** parse path and query as separate tokens before
dispatch, matching {{RFC3986}} URI generic syntax. Implementations
**MUST NOT** treat a `?`-prefixed query as part of the path; the
path-grammar enforcement in {{AGTP-METHODS}} applies to the path
component only.

URI fragments (`#anchor`) **MUST NOT** appear on the request line.
A request line containing `#` **MUST** be rejected at the wire
layer as malformed.

The path-pattern grammar (template parameters in `{param}` form,
verb-leakage prevention, structural minimums) is normatively
specified in {{AGTP-METHODS}}. v07 servers **MUST** implement the
AGTP-METHODS path grammar.

### Response Line Grammar {#response-line}

The response line carries three tokens separated by single space
characters, terminated by CRLF:

~~~~
response-line  = agtp-version SP status-code SP status-text CRLF
agtp-version   = "AGTP/1.0"
status-code    = 3DIGIT                 ; per {{status-codes}}
status-text    = *( VCHAR / SP / HTAB )
                                        ; freeform reason phrase
~~~~

The `status-text` is informational and **MUST NOT** be used by
servers or clients for protocol decisions; the numeric `status-code`
is authoritative.

### Request Headers

| Field | Required | Description |
|-------|----------|-------------|
| Agent-ID | **MUST** for non-anonymous requests | Canonical 256-bit identifier of the invoking agent. Identifies the request's source rather than its target. Servers reject requests lacking `Agent-ID` against non-anonymous endpoints. |
| Authority-Scope | **MAY** | Scopes the agent claims for this specific request. When present, every claimed scope **MUST** be a subset of the scopes declared in the agent's identity document; servers **MUST** validate and **MUST** return 455 Scope Violation with reason `scope-claim-invalid` on failure, since a claim beyond the document's grant exceeds authority the agent holds. When absent, the request inherits the full scope set of the agent's identity document. The server uses the claimed (or inherited) set when evaluating the endpoint's `required_scopes`. |
| Session-ID | **MAY** | Opaque session identifier grouping multiple requests into an operational session. The protocol assigns no semantics; servers pass the value through to handlers via the endpoint context. |
| Task-ID | **MAY** | Opaque identifier tracing a specific task or operation across multiple requests. Useful for audit correlation. Servers **SHOULD** echo this value in the response `Task-ID` header. |
| Delegation-Chain | **MUST** on requests made under delegated authority | The Delegation Sequence that produced this request: an ordered list of canonical Agent-IDs, origination to current, with the per-hop delegation tokens. Format, minting, verification, and failure behavior are specified in {{delegation}}. Absent on requests the agent makes under its own authority. |
| Request-Signature | **MUST** on non-anonymous requests | Proof that the sender holds the agent's current signing key: a compact JWS over the method, path, Agent-ID, Authority-Scope, Delegation-Chain digest, body digest, timestamp, and nonce, per {{request-signature}}. |
| Merchant-ID | **MUST** on PURCHASE | Canonical identifier of the intended merchant counterparty. See {{AGTP-MERCHANT}}. |
| Merchant-Manifest-Fingerprint | **MUST** on PURCHASE | SHA-256 fingerprint of the Merchant Manifest Document verified by the requesting agent. Receiving server **MUST** reject with 458 if this differs from its current manifest. See {{AGTP-MERCHANT}}. |
| Intent-Assertion | **SHOULD** on PURCHASE | Detached JWT {{RFC7519}} carrying signed principal-authorized purchase intent. Forwardable to payment networks as standalone evidence. See {{AGTP-MERCHANT}}. |
| Authorization | **MAY** | Application-layer authorization credential carried for composition with external identity providers (e.g., OIDC/OAuth bearer tokens, SPIFFE SVIDs). When present, the value follows the HTTP `Authorization` header syntax per {{RFC9110}} (e.g., `Bearer TOKEN`). AGTP identity (Agent-ID, Agent Certificate) and the credential in this header answer different questions and are orthogonal: AGTP identity identifies the requesting agent; the credential in this header identifies the human or service principal on whose behalf the agent acts. Servers **MAY** require an `Authorization` header on specific methods per `[policies.oauth]` configuration; the External Identity Provider composition profile in {{AGTP-COMPOSITION}} defines the normative composition rules. Semantics of any token in the header are application-defined and outside AGTP's wire scope. |
| Cart-Digest | **MAY** | Cryptographic digest of a structured cart returned by a prior QUOTE invocation. Binds a PURCHASE to a previously quoted cart without retransmission of line-item detail. See {{AGTP-MERCHANT}}. |
| Allow-RCNS | **MAY** | Caller opt-in to Runtime Contract Negotiation. Value `true` signals the caller is willing to receive a 461 RCNS Contract Available response or an inline optimistic synthesis when a requested `(method, path)` pair is unregistered. Absent or non-`true` keeps the RCNS gate closed; the request is then refused per the standard unregistered-pair rules. See {{AGTP-RCNS}}. |
| Contract-Synthesized | **MAY** | The `synthesis_id` of a previously-issued RCNS contract. Presents the contract to bypass the four-lock gate and dispatch the request directly under the synthesized contract. The server **MUST** refuse the presentation with `464 contract-not-yours` if the caller's Agent-ID differs from the contract's `originating_agent_id`. See {{AGTP-RCNS}}. |
| Idempotency-Key | **MAY** | Per-agent idempotency key for RCNS-eligible requests and other state-modifying calls. Servers **MUST** scope the idempotency cache by `(Agent-ID, Idempotency-Key)`; replays from the same agent return the cached response, replays from different agents are independent. See {{AGTP-RCNS}}. |
{: title="AGTP Request Header Fields"}

### Request Signature and Proof of Possession {#request-signature}

A verified Agent Identity Document establishes that an Agent-ID
exists, who owns it, and what it may do. It leaves unestablished whether
the sender of a given request is that agent. Without a per-request
proof, `Agent-ID` is a self-asserted header bound to a verified
document, and any party that can read the document can present the
header. The `Request-Signature` header closes that gap.

Every non-anonymous request **MUST** carry `Request-Signature`: a
compact JWS {{RFC7515}} with algorithm `EdDSA`, `kid` equal to the
canonical Agent-ID, signed with the private key whose public half is
the `agent_public_key` of the agent's current Identity Document
({{AGTP-IDDOC}}). The JWS payload is the JSON object:

~~~~
{
  "m":   "<method>",
  "p":   "<path>",
  "aid": "<Agent-ID>",
  "as":  "<Authority-Scope header value, or null>",
  "dc":  "<SHA-256 of the Delegation-Chain header value, or null>",
  "bd":  "<SHA-256 of the request body, or null when empty>",
  "iat": <issued-at, seconds since epoch>,
  "n":   "<nonce, at least 96 bits, base64url>"
}
~~~~

serialized in the canonical form of {{RFC8785}}. The server **MUST**
verify the signature before dispatch and **MUST** reject the request
when: the signature fails under the current `agent_public_key`; any
covered field differs from the request as received; `iat` is outside
the server's acceptance window (default 300 seconds, operator
configurable); or the `(aid, n)` pair has been seen within the window.
Rejection is 401 with reason `possession-unproven`, and the event
**MUST** be logged. Anonymous requests (no `Agent-ID`) carry no
signature and receive only what the server offers anonymously.

Key rotation is a re-issuance of the Identity Document with a new
`agent_public_key` by the agent's issuer; the Agent-ID is unchanged,
because authority attaches to the agent rather than to the key. A
verifier caches the document per the freshness rules of
{{AGTP-IDDOC}} and re-resolves on a signature failure before
rejecting, so a rotated key is picked up without a round trip in the
common case and with one in the worst case. Delegation tokens
({{delegation}}) are signed with the same key.

Identity Documents issued before this revision carry no
`agent_public_key`. A request from such an agent **MAY** be accepted
by operator policy and **MUST** be recorded as possession-unproven in
the Attribution-Record; it is verified at Level 1 ({{verification-levels}})
regardless of document verification. Operators **SHOULD** set a date
after which such documents are refused.

### Response Headers {#response-headers}

| Field | Required | Description |
|-------|----------|-------------|
| Server-ID | **MUST** | Canonical identifier of the server that produced the response. **MUST** be populated from the server's configured `server_id` on every response. Useful for audit, load-balanced deployments, and verifying which server processed a request. Distinct from `Agent-ID`: `Server-ID` identifies the host that processed the request, `Agent-ID` (when echoed) identifies the requesting agent. |
| Agent-ID | **SHOULD** when present in request | Echo of the request's `Agent-ID` header to correlate response with the requesting agent. Absent when the request was anonymous. Echoed verbatim from the request; servers **MUST NOT** substitute or normalize the value on the response path. |
| Task-ID | **SHOULD** when present in request | Echo of the request's `Task-ID` header to correlate response with request. Absent when the request carried no `Task-ID`. |
| Request-ID | **SHOULD** when present in request | Echo of the request's `Request-ID` header per the per-interaction identifier convention. See {{AGTP-IDENTIFIERS}}. |
| Response-ID | **MUST** | Per-interaction identifier of this response. Distinct from `Request-ID`; together they pair a request and its response on the wire. See {{AGTP-IDENTIFIERS}}. |
| Audit-ID | **MUST** when Attribution-Record is present | Identifier of this response's Attribution-Record, computed as `sha256(jws_compact_serialization)` of the Attribution-Record JWS. The same value appears as `previous_audit_id` in the agent's next emitted Attribution-Record, closing the per-agent hash chain. See {{AGTP-IDENTIFIERS}}. |
| Owner-ID | **SHOULD** | Identifier of the human or organizational principal accountable for the responding agent. Recorded in the agent's Agent Genesis and stamped on every response by the daemon. See {{AGTP-IDENTIFIERS}}. |
| Trust-Tier | **SHOULD** | Resolved trust tier of the responding agent (`1`, `2`, or `3`) per the trust-posture loading rule in {{AGTP-TRUST}}. Stamped on every response when the responding server has resolved a trust tier for the addressed agent; omitted when no trust tier is known. Allows relying parties to apply trust-tier-conditional policy on every response without consulting the Agent Identity Document. |
| Verification-Path | **SHOULD** when `Trust-Tier` is present | Resolved verification path of the responding agent (`dns-anchored`, `log-anchored`, `hybrid`, or `org-asserted`) per the trust-posture loading rule in {{AGTP-TRUST}}. Stamped on every response when the responding server has resolved a verification path; omitted when no path is known. |
| Trust-Warning | **MUST** when resolved `Trust-Tier` is `2` and a warning is set | Trust warning token recorded on the responding agent's Agent Identity Document (e.g., `verification-incomplete`, `verification-path-unsupported`). Stamped on every response from a Tier 2 agent so relying parties can surface the warning without consulting the Agent Identity Document; omitted on Tier 1 and Tier 3 responses. See {{AGTP-TRUST}}. |
| Contract-Synthesized | **MUST** when serving under RCNS optimistic mode | The `synthesis_id` of the contract under which this response was served. Notifies the caller that the response was served under a runtime-negotiated contract per {{AGTP-RCNS}}. Callers that wish to repeat the action send subsequent requests with the same header value in the request to bypass the gate. |
| RCNS-Attempt-Id | **MUST** on every 464 RCNS No Contract response from an RCNS-capable server | Opaque identifier of the failed synthesis attempt record. Retrievable via `INSPECT target=rcns-attempt` and carries the diagnostic detail for the failure (which gate lock closed, the requested `(method, path)`, the trust tier resolved at evaluation time). See {{AGTP-RCNS}}. |
| Attribution-Record | **MUST** | JWS-signed attestation of the response's origin, serialized in JWS Compact form per {{RFC7515}} (`base64url(protected_header).base64url(payload).base64url(signature)`). The payload carries the base attribution fields (server ID, response timestamp, request hash, response status) together with the identifier-chain fields and the per-agent chain link (`previous_audit_id`); see {{AGTP-IDENTIFIERS}} for the extended payload schema. Servers with a configured manifest signing key sign the payload with that key; servers without a configured key emit a fallback JWS with `alg: none` and an empty signature octet, preserving the wire format and the Audit-ID chain. Consumers that require cryptographic attestation **MUST** reject `alg: none` records. |
| Continuation-Token | **MAY** | Token for retrieving additional results in streaming contexts. |
| Supported-Methods | **SHOULD** (on session open) | List of AGTP methods supported by this server. |
| Cost-Estimate | **MAY** | Estimated resource consumption in Budget-Limit unit format. Returned by QUOTE; **MAY** appear on any response as an informational signal. |
| Attestation-Evidence | **MAY** | RATS attestation evidence token or reference URI per {{RFC9334}}. Format indicated by `attestation_type` in response body: `rats-eat`, `rats-corim`, or `rats-uri`. |
{: title="AGTP Response Header Fields"}

Implementations **MAY** emit implementation-specific headers, by
convention prefixed `X-`, but such headers have no protocol
semantics; agents **MUST NOT** rely on them and servers **MUST
NOT** require them.

### Headers Reserved for Future Revisions

The following headers were specified in earlier drafts and are
reserved for future revisions. v00 servers **MUST NOT** require
them and **MAY** ignore them when received:

| Field | Status | Anticipated Use |
|-------|--------|-----------------|
| Priority | Reserved (v01+) | Request priority hint: critical, normal, background |
| TTL | Reserved (v01+) | Maximum acceptable response latency in milliseconds. Pairs with status code 408 Timeout. |
| Budget-Limit | Reserved (v01+) | Maximum resource budget per invocation. Pairs with status code 456 Budget Exceeded. |
| AGTP-Zone-ID | Reserved (v01+) | Network zone boundary constraint. Pairs with status code 457 Zone Violation; SEP-enforced. |
| Content-Schema | Reserved (v01+) | URI reference to JSON Schema describing the request body structure. |
| Telemetry-Export | Reserved (v01+) | OTLP endpoint URI for metric export, or `inline` to receive metrics embedded in the response `Attribution-Record`. |
{: title="Headers Reserved for Future AGTP Revisions"}

Status codes 456 Budget Exceeded and 457 Zone Violation remain
allocated in the AGTP Status Code Registry; their normative
trigger headers (`Budget-Limit`, `AGTP-Zone-ID`) are deferred to a
future revision. v00 servers **MAY** enforce equivalent semantics
through implementation-specific mechanisms and return the
corresponding status code, but the wire-level headers are outside
of the v00 contract.

### Retired Headers

The following headers appeared in earlier drafts and have been
removed. Implementations **MUST NOT** emit them and **MAY** reject
requests that carry them:

| Field | Reason for Removal |
|-------|--------------------|
| AGTP-Version | Redundant with the request and response lines (which carry `AGTP/1.0`). |
| AGTP-Method | Redundant with the request line (which carries the method token). |
| AGTP-Status | Redundant with the response line (which carries the status code). |
| Principal-ID | Renamed to `Owner-ID` in line with the locked AGTP taxonomy. The Owner-ID is recorded in the agent identity document referenced by `Agent-ID` and never transmitted as a separate request header. See {{AGTP-IDENTIFIERS}}. |
| Server-Agent-ID | Renamed to `Server-ID`. The earlier name conflated server identity with agent identity. |
{: title="Retired AGTP Headers"}

## Status Codes {#status-codes}

AGTP defines its own status code space. Codes 261 through 263,
455 through 465, and 550 through 555 are AGTP-specific with no HTTP
equivalent and are registered in the IANA AGTP Status Code Registry
(see {{iana-status}}). AGTP-specific code numbers are deliberately
chosen from ranges unassigned in the IANA HTTP Status Code Registry
to avoid semantic collision with HTTP status codes that may appear
in payloads carried by AGTP method invocations.

The AGTP status code model carries four structural rejection codes
(404, 405, 459, 460) that together cover the failure surface for
contract-level invocation. Each code is independently actionable:
404 indicates the path is absent from this server; 405 indicates
the verb and path are each individually valid but the server
leaves this combination unexposed; 459 indicates the verb is absent from the
AGTP-METHODS approved method catalog; 460 indicates the path violates
AGTP-METHODS path grammar. The structural rejection model and the
curated method catalog are specified in {{AGTP-METHODS}}.

| Code | Name | Meaning |
|------|------|---------|
| 200 | OK | Method executed successfully |
| 202 | Accepted | Method accepted; execution is asynchronous |
| 204 | No Content | Method executed; no response body |
| 261 | Negotiation In Progress | PROPOSE evaluation in progress; agent **MAY** poll for terminal status. See {{AGTP-RCNS}}. |
| 262 | Authorization Required | Authorization the agent could obtain is absent: credential establishment, consent, or a scope the endpoint requires that the agent holds in its identity document but left unclaimed. Never returned when the request exceeds the authority the agent holds; that is 455. Covers PROPOSE-time authorization, unclaimed scope at endpoint dispatch, ad-hoc method invocation requiring wildcards consent, and discovery requests blocked by anonymous-discovery policy. See {{AGTP-METHODS}}; the RCNS gate case is in {{AGTP-RCNS}}. |
| 263 | Proposal Approved | PROPOSE accepted; the proposed endpoint has been synthesized and instantiated. Response body carries the synthesized endpoint contract. See {{AGTP-RCNS}}. |
| 400 | Bad Request | Malformed AGTP request |
| 401 | Unauthorized | Authentication failure. Covers an unrecognized or unauthenticated Agent-ID, INSPECT read-access ACL failure for unauthenticated callers, lifecycle-method invocation without a verified client certificate when `genesis_issuer` mode is in effect, and external-credential failures (e.g., missing `Authorization` header on a method that requires one, or an `Authorization` header that fails validation). The response body **SHOULD** carry a structured `reason` from the vocabulary `agent-unauthenticated`, `oauth-required`, `oauth-invalid`, `acl-anonymous-blocked`, or `genesis-issuer-cert-required` to disambiguate the failure mode. See {{AGTP-COMPOSITION}} for the External Identity Provider composition profile and the OAuth-specific reasons. |
| 403 | Forbidden | Agent lacks authority for requested action per Authority-Scope |
| 404 | Not Found | The path is absent from this server. No endpoints are registered under it. |
| 405 | Method Not Allowed | The method is recognized and the path is valid, but the server's policy or registry leaves this combination unexposed. The response body **MUST** list allowed methods for the path and any redirects from the manifest's `policies.methods` sub-block. The agent **MAY** PROPOSE the combination when policy leaves it unexposed. AGTP-specific. See {{AGTP-METHODS}}; PROPOSE per {{AGTP-RCNS}}. |
| 408 | Timeout | TTL exceeded before method could execute. AGTP-specific semantics; see below. |
| 409 | Conflict | Method conflicts with current state |
| 410 | Gone | Agent has been REVOKEd; canonical Agent-ID is permanently retired (status `retired`). A Deprecated agent (status `deprecated`) does **NOT** return 410 and continues to serve traffic; deprecation signals planned end-of-life rather than immediate retirement. AGTP-specific semantics; see below. |
| 422 | Unprocessable | Request well-formed but semantically invalid |
| 429 | Rate Limited | Agent is exceeding permitted request frequency |
| 455 | Scope Violation | The request exceeds authority the agent holds: a claimed Authority-Scope beyond the identity document's grant (`scope-claim-invalid`), an endpoint whose required scope the agent's document omits (`scope-exceeded`), a DELEGATE granting more than the delegator's effective scope (`scope-exceeds-parent`), or a token- or query-defined scope dimension outside the standard set. Budget, rate, and zone have dedicated codes 456, 429, 457. AGTP-specific. |
| 456 | Budget Exceeded | Method execution would exceed the Budget-Limit declared in the request. AGTP-specific. |
| 457 | Zone Violation | Request would route outside the AGTP-Zone-ID boundary. SEP-enforced. AGTP-specific. |
| 458 | Counterparty Unverified | PURCHASE counterparty failed merchant identity verification: Merchant-ID absent, Merchant-Manifest-Fingerprint mismatch, or merchant in non-Active lifecycle state. AGTP-specific. See {{AGTP-MERCHANT}}. |
| 459 | Method Violation | The method name is absent from the AGTP-METHODS approved method catalog. The method itself is the problem. AGTP-specific. See {{AGTP-METHODS}}. |
| 460 | Endpoint Violation | The endpoint path violates AGTP-METHODS path grammar. A path segment matches an approved method name, indicating method-name leakage into the path. AGTP-specific. See {{AGTP-METHODS}}. |
| 461 | RCNS Contract Available | The requested endpoint is unregistered but the server is prepared to synthesize a contract for it. The response body carries a contract preview the caller may accept or decline. Returned in the confirm-first RCNS delivery mode. AGTP-specific. See {{AGTP-RCNS}}. |
| 462 | Insufficient Budget | Commerce request budget envelope is insufficient for the receiving agent's pricing tier or settlement requirements. AGTP-specific. See {{AGTP-COMMERCE}}. |
| 463 | Proposal Rejected | The service cannot or declines to instantiate the proposed endpoint. Returned in response to PROPOSE. Response body **MUST** carry a structured reason (e.g., `out-of-scope`, `policy-refused`, `composition-impossible`, `ambiguous`, `synthesis-disabled`) and **MAY** carry an optional counter-proposal. AGTP-specific. See {{AGTP-RCNS}}. |
| 464 | RCNS No Contract | An RCNS synthesis attempt was made but no contract could be delivered. Response body **MUST** carry a structured reason: `rcns-disabled` (RCNS policy off on this server), `trust-tier-insufficient` (caller's resolved trust tier is below the server's RCNS minimum), `composition-impossible` (no synthesis path exists from registered primitives), `synthesis-error` (synthesis attempt failed at runtime), `contract-not-yours` (presented `synthesis_id` was issued to a different agent), or `contract-revoked` (presented `synthesis_id` was revoked). AGTP-specific. See {{AGTP-RCNS}}. |
| 465 | Reserved | Reserved for AGTP expansion. |
| 500 | Server Error | Internal failure in the responding system |
| 503 | Unavailable | Responding agent or system temporarily unavailable or Suspended |
| 550 | Delegation Failure | A delegated sub-agent failed to complete the requested action. AGTP-specific. |
| 551 | Authority Chain Broken | The Delegation Sequence cannot be verified: a hop fails parent re-verification, a hop's scope exceeds its parent's, the sequence exceeds the depth limit or contains a cycle, a member is in a non-Active lifecycle state, or the sequence has expired. The `reason` token names which ({{delegation}}). AGTP-specific. |
| 552 | Reserved | Reserved for AGTP expansion. |
| 553 | Reserved | Reserved for AGTP expansion. |
| 554 | Reserved | Reserved for AGTP expansion. |
| 555 | Reserved | Reserved for AGTP expansion. |
{: title="AGTP Status Codes"}

The four structural rejection codes (404, 405, 459, 460) form the
contract-level structural failure surface and have distinct recovery
semantics. A 459 tells the agent to choose a different method from
the AGTP-METHODS catalog. A 460 tells the agent to restructure the path
so that no path segment is a method name. A 405 tells the agent the
method and path are each individually valid but the failure is a
server policy decision; a different method-and-path combination
might work, or PROPOSE might negotiate access. A 404 tells the
agent the path is absent from this server altogether. Beyond the
structural surface, 463 (Proposal Rejected) is the runtime
negotiation rejection: the agent's PROPOSE was structurally valid
but the server cannot or declines to synthesize the requested endpoint.

Status code 262 (Authorization Required) consolidates the
authority-related rejection conditions an agent might encounter
when interacting with a server. It is returned when: (a) a PROPOSE
request requires credential establishment before the server will
evaluate it; (b) an endpoint invocation requires Authority-Scope
that the agent has yet to declare; (c) an ad-hoc method invocation
requires wildcards consent (`wildcards: true` on the agent identity
document and `wildcards_accepted: true` in server policy) that is
absent on either side; or (d) a discovery request is blocked by
server policy that requires authenticated identity for manifest
retrieval. The response body **MUST** identify which condition
applies so the agent knows what to remediate.

The rule separating 262 from 455 is obtainability. 262
Authorization Required is returned when the authorization the
request lacks is one the agent could obtain without exceeding what it
holds: a credential it has yet to establish, a consent it has yet to
give or receive, or a scope its identity document grants but the
request left unclaimed. 455 Scope Violation is returned when the
request exceeds the authority the agent holds: a claimed scope beyond
the document's grant, an endpoint requiring a scope the document
omits, or a delegation granting more than the delegator's
effective scope. Earlier revisions used both codes for Authority-Scope
failures in different sections; this revision applies the rule above
throughout. 455 also covers operator-defined scope dimensions outside
the standard set. Status code 456 (Budget Exceeded)
is the dedicated code for resource-consumption scope violations:
the agent's requested action would consume resources beyond what
the principal authorized for this invocation. Status code 457
(Zone Violation) is returned by SEPs when a DELEGATE request would
route to an agent outside the declared zone boundary. Status code
458 (Counterparty Unverified) is returned on PURCHASE invocations
when the receiving server cannot verify that the requesting agent
has performed valid merchant identity verification against the
server's current Merchant Manifest Document, or when the merchant
is in a non-Active lifecycle state; see {{AGTP-MERCHANT}}.

Status code 551 (Authority Chain Broken) is returned when a server
processing a delegated request cannot verify the Delegation Sequence
carried in the `Delegation-Chain` header per {{delegation}}. The
response body **MUST** carry the `reason` token and **SHOULD**
identify the failing hop. Status code 408 (Timeout) is
reused from HTTP with AGTP-specific semantics: it signals that the
method's declared TTL expired before execution completed, distinct
from HTTP's request-timeout semantics. Status code 410 (Gone) is
reused from HTTP with AGTP-specific semantics: it indicates that an
Agent-ID has been permanently retired through REVOKE of its Agent
Genesis, distinct from HTTP's resource-removed semantics. A
Deprecated agent does **NOT** return 410; deprecation signals
planned end-of-life while the agent continues to serve traffic
(see DEPRECATE in {{methods-deprecate}}). The canonical Agent-ID
of a retired agent **MUST NOT** be retried. All AGTP-specific
status codes are operational signals rather than protocol errors, and
**MUST** be logged for audit purposes.

### Structured Failure Fields {#failure-fields}

A status code names the failure class. Agents retrying, resuming, or
reporting a failure need three more facts: whether the request can
be retried as sent, whether any of it completed, and whether it left
side effects. Every 4xx and 5xx response body **SHOULD** carry the
following fields, and servers that implement {{AGTP-RCNS}} or any
action method **MUST** carry them:

`retryable`:
: `true` when the identical request **MAY** be re-sent without
  further change; `false` when the request must change first
  (different scope, a corrected body, a new proposal); `unknown`
  when the server cannot determine this.

`partial`:
: `true` when part of a multi-step invocation completed before the
  failure. When `true`, `completed_steps` carries the ordered list of
  step identifiers that completed, so a caller can resume rather than
  repeat.

`side_effects`:
: `none` when the failed request changed no state; `committed` when
  it changed state that persists despite the failure; `unknown` when
  the server cannot determine this. Action methods ({{AGTP-METHODS}}
  defines the property) **MUST NOT** report `none` unless the server
  verified it.

`evaluation`:
: `refused` when the server evaluated the request and decided against
  it; `incomplete` when a check the decision depends on was unable to run.
  When `incomplete`, `missing_input` names the input that was
  unavailable (an unreachable registry, an expired credential, a log
  that failed to answer). An intermediary that forwards a response
  **MUST NOT** convert `incomplete` into `refused`; an intermediary
  that narrows a request is a decision point and records its own
  evaluation. A refusal and an incomplete check are different
  outcomes and are never collapsed.

`reason`:
: A token from the vocabulary registered for the status code, as
  already required for 262, 455, 463, 464, and 551.

The fields are additive and wire-compatible: a v09 client ignores
them; a v09 server omits them. They answer the typed-error question
raised in review without changing any status code.

## Wire Format and Content-Type {#wire-format}

AGTP request and response bodies are encoded as JSON or YAML. The
following media types are defined by AGTP. Their IANA registration
status varies; see the IANA Considerations section for the
authoritative status table.

| Media Type | Use | IANA Status |
|---|---|---|
| `application/vnd.agtp+json` | AGTP method request/response bodies (JSON) | Planned (this document) |
| `application/vnd.agtp+yaml` | AGTP method request/response bodies (YAML) | Planned (this document) |
| `application/vnd.agtp.identity+json` | Agent Identity Document (JSON) | Registered (vendor tree) |
| `application/vnd.agtp.identity+yaml` | Agent Identity Document (YAML) | Registered (vendor tree) |
{: title="AGTP Media Types Defined in This Document"}

Two additional media types are defined in the companion AGTP-METHODS
specification {{AGTP-METHODS}} and registered concurrently:
`application/vnd.agtp.manifest+json` (server manifest) and
`application/vnd.agtp.endpoint+json` (endpoint definition). Both
are listed in the master Media Type Registry section of this
document for cross-document discoverability.

Implementations **MUST** include the appropriate Content-Type on all
AGTP requests and responses that carry a message body. Responses with
no body (e.g., 204 No Content) **MUST NOT** include a Content-Type
header. Binary or streaming extensions **MAY** define additional
Content-Type values as part of their companion specifications.

EXECUTE method invocations carry application-layer payloads whose
Content-Type is set by the carried application protocol (for example,
`application/vnd.mcp.tools+json` for MCP tool invocations carried over
AGTP). The AGTP server dispatches based on the carried Content-Type; see
{{methods-execute}} and {{composition-section}}.

The common structure for AGTP method-level request bodies:

~~~~json
{
  "method": "QUERY",
  "task_id": "task-0042",
  "session_id": "sess-a1b2c3d4",
  "parameters": { },
  "context": { }
}
~~~~

And for AGTP method-level response bodies:

~~~~json
{
  "status": 200,
  "task_id": "task-0042",
  "result": { },
  "attribution": { }
}
~~~~

## Early Implementations

AGTP is a proposed specification. No production implementations exist at
the time of this writing. The author encourages early prototype
implementations to validate the protocol design, identify gaps, and
generate feedback prior to IETF working group submission.

> If you are building an AGTP prototype or reference implementation,
> please share your findings via the feedback channel listed on the cover
> of this document. A reference implementation in Python and/or Go is
> planned as open-source software concurrent with or shortly after IETF
> I-D submission. Implementation reports are welcome and will be
> incorporated into subsequent draft revisions.

Implementers wishing to experiment before final IANA port assignment
**SHOULD** use a locally-chosen port from the Dynamic Ports range
(49152-65535) on developer-controlled endpoints. Implementations
**MUST NOT** publish or document any specific port number as an
AGTP-associated value until IANA assignment is complete.

ACTIVATE is one of the three Lifecycle methods on the
eighteen-method floor; see {{methods-activate}}. Earlier
revisions of this document described ACTIVATE as an optional
method extension carried in a separate companion. With the
promotion of the Lifecycle group to the embedded floor,
ACTIVATE, DEACTIVATE, and REVOKE are all core methods; any
conformant AGTP implementation **MUST** support them.
Package-binding semantics specific to `.nomo` governed
packages remain governance-platform-specific and **MAY** be
layered on top of the protocol-level ACTIVATE transaction
without affecting the wire contract.

# Agent Identity, URI Structure, and Registration

## URI Structure and Resolution Mechanics

### Foundational Principle

AGTP identity is agent-first. Every agent is identified by a canonical
Agent-ID: a 256-bit cryptographic identifier derived from the agent's
Agent Genesis hash at ACTIVATE time. The canonical Agent-ID is the
authoritative identifier in every AGTP protocol operation. It appears
in the `Agent-ID` header of every request, is the key in the registry,
and is the cross-layer reference linking the AGTP Agent Certificate
extension to the governance-layer Agent Genesis.

All other agent identification forms recognized by AGTP, including
domain-anchored URIs and Web3 resolution targets, are aliases that
resolve to a canonical Agent-ID. In the event of any conflict
between an alias and a canonical Agent-ID, the canonical Agent-ID
**MUST** be treated as authoritative.

AGTP URIs are addresses rather than filenames. File format suffixes
(`.agtp`) **MUST NOT** appear in canonical `agtp://` URIs. A URI
resolves to an Agent Identity Document or Agent Namespace Document
derived from the underlying package; it never exposes or serves the
package itself.

Implementations **MUST** treat any URI containing a file extension
in the path as non-canonical and **SHOULD** issue a 301 Moved
Permanently redirect to the canonical form prior to resolution.

The `.agent` suffix and the `.nomo` suffix are file format
designators for AGTP agent packages; they are never URI hostname
labels. Earlier drafts admitted hostname forms ending in `.agent`
or `.nomo` as agent-native hierarchical TLDs; this revision removes
that admission. Hostnames in AGTP URIs are standard DNS hostnames
per RFC 3986.

### Canonical URI Forms {#uri-forms}

AGTP is identity-first. Agent-IDs are canonical and content-addressed;
hosting is incidental. Form 1 (`agtp://[Agent-ID]`) is the canonical
identity form. Form 1a (`agtp://[Agent-ID]@[host]`) provides direct
addressing for cases where the resolver has yet to learn how to
reach the canonical ID, and bridges the gap between identity-first
addressing and the existing DNS-based reachability infrastructure.

Forms 2 and 2a address servers and organizations rather than
specific agents, providing entry points for discovery operations
that target no named agent. Forms 3 and 4 address agents by
local name within a domain's namespace; they differ only in
deployment convention.

~~~~
Form 1.  Canonical identity (cryptographic, authoritative):
  agtp://[Agent-ID]

Form 1a. Canonical identity with explicit host (direct addressing):
  agtp://[Agent-ID]@[host]

Form 2.  Server-level discovery (specific server endpoint):
  agtp://[host]

Form 2a. Organization-level discovery (DNS-registered domain root):
  agtp://[domain]

Form 3.  Domain-anchored agent (domain owns the namespace):
  agtp://[domain]/agents/[agent-name]

Form 4.  Subdomain-anchored agent (dedicated AGTP subdomain):
  agtp://agtp.[domain]/agents/[agent-name]
~~~~

The port portion of any AGTP URI is **OPTIONAL**. When omitted,
AGTP clients **MUST** use the IANA-assigned default port 4480. The
port is included in URIs only for non-default deployments and
appears in the host component (Form 1a host or Form 2 host); ports
**MUST NOT** appear in Form 2a domain components or in Forms 3/4
domain components.

#### Form 1: Canonical Identity

Form 1 carries only the 64-character lowercase hex representation
of the Agent Genesis SHA-256 hash:

~~~
agtp://7f3a9c2d...e1f8b0a4
~~~

Form 1 resolves to a signed Agent Identity Document through any
verification path declared in the agent's registry record (Section
5.2). The canonical ID is self-describing: any AGTP-aware
governance platform, transparency log, or resolution service can
return the Identity Document given the canonical ID alone, without
prior knowledge of which organization, domain, or blockchain the
agent is registered under.

#### Form 1a: Canonical Identity with Explicit Host

Form 1a embeds an explicit host alongside the canonical Agent-ID:

~~~
agtp://7f3a9c2d...e1f8b0a4@agents.acme.com
agtp://7f3a9c2d...e1f8b0a4@192.0.2.42
agtp://7f3a9c2d...e1f8b0a4@agents.acme.com:9999
~~~

The Agent-ID identifies; the host tells the resolver where to reach
a server that can return the Identity Document or accept method
invocations. Form 1a is the preferred form when:

- The agent has been issued a canonical Agent-ID but is yet to be
  registered with an AGTP discovery service.
- The resolver has yet to acquire a path from canonical Agent-ID to
  endpoint (no DNS anchor, no transparency log entry, no
  governance-platform resolution).
- A client wishes to address an agent by ID directly without round-
  tripping through a registry.

Form 1a leaves the identity-first model intact. The canonical
Agent-ID remains authoritative; the host portion is treated as a
resolution hint and **MUST NOT** be used to derive identity. If the
host returns an Identity Document whose `agent_id` field fails to
match the canonical Agent-ID in the URI, the response **MUST** be
rejected.

#### Form 2: Server-Level Discovery

Form 2 addresses a specific server endpoint without naming an agent:

~~~
agtp://agents.acme.com
agtp://agtp-edge-7.us-east.acme.com
agtp://192.0.2.42
agtp://[2001:db8::42]
agtp://agents.acme.com:9999
~~~

The host component **MAY** be any RFC 3986 host: a registered
hostname, an FQDN, an IPv4 address, an IPv6 address, optionally
followed by a port. Form 2 is used for diagnostic operations
addressing a specific server instance (a particular edge node, a
specific deployment in a load-balanced fleet, a server identified
by IP) and for method invocations that target server-level state
rather than agent-level state.

#### Form 2a: Organization-Level Discovery

Form 2a addresses an organization's AGTP presence via its registered
DNS domain:

~~~
agtp://acme.com
agtp://example.org
~~~

The domain component **MUST** be a DNS-registered domain name with
at least one label. IP addresses, ports, and userinfo components
**MUST NOT** appear in Form 2a; URIs carrying any of these are
Form 2 rather than Form 2a.

Form 2a resolution proceeds via standard DNS lookup of the domain,
then AGTP connection establishment on port 4480 against the
resolved address. The response is the organization's canonical
server manifest. Whether the domain resolves to a single AGTP
server, a load-balanced fleet, or a CDN-fronted edge is a
deployment concern; Form 2a commits the caller to no
specific topology.

The syntactic distinction between Form 2 (any RFC 3986 host,
possibly with port) and Form 2a (bare DNS domain) corresponds to a
semantic distinction. Form 2 addresses a specific reachable
server; Form 2a addresses whichever server the organization's DNS
currently points at. Both are valid; they differ in caller intent.

#### Forms 3 and 4: Domain-Anchored Agents

Forms 3 and 4 address an agent by local name within a domain's
agent namespace:

~~~
Form 3: agtp://acme.com/agents/bookbot
Form 4: agtp://agtp.acme.com/agents/bookbot
~~~

Form 3 places the agent under the organization's primary domain.
Form 4 places the agent under a dedicated `agtp.` subdomain.
Resolution semantics are identical: the AGTP server at the domain
(or subdomain) consults its `hosted_agents` manifest entries and
returns the canonical Agent-ID for the named local agent. The local
agent name is unique within the domain's namespace; the same name
under a different domain is a different agent.

Forms 3 and 4 differ only in deployment convention. Operators
choose based on their infrastructure preferences: organizations
that prefer to keep agent-protocol traffic on a dedicated subdomain
use Form 4; organizations that operate AGTP as the canonical face
of their primary domain use Form 3. The protocol treats them as
equivalent.

In both forms the domain component **MUST** be a DNS-registered
domain name; IPs and ports **MUST NOT** appear. Direct addressing
with explicit host is available via Form 1a if the resolver knows
the canonical Agent-ID.

### URI Grammar

The URI grammar is defined in ABNF as:

~~~~
AGTP-URI     = "agtp://" agtp-locator
agtp-locator = agent-id [ "@" host ]                 ; Form 1, 1a
             / host                                    ; Form 2
             / domain                                  ; Form 2a
             / domain "/agents/" agent-name            ; Form 3
             / "agtp." domain "/agents/" agent-name    ; Form 4
agent-id     = 64HEXDIG                               ; lowercase hex
host         = IP-literal / IPv4address / reg-name [ ":" port ]
                                                      ; per RFC 3986
domain       = label *("." label)                     ; DNS-registered, no port
label        = ALPHA *( ALPHA / DIGIT / "-" )
agent-name   = 1*( ALPHA / DIGIT / "-" / "_" )
port         = 1*DIGIT
                                                      ; OPTIONAL; defaults to 4480
~~~~

AGTP URIs are addresses rather than filenames. File extensions in the
path **MUST NOT** appear in canonical `agtp://` URIs. A URI
resolves to an Agent Identity Document, a server manifest, or an
endpoint response derived from server state; it never exposes or
serve a package file.

Implementations **MUST** treat any URI containing a file extension
in the path as non-canonical and **SHOULD** issue a 301 Moved
Permanently redirect to the canonical form prior to resolution.

### Method-on-URI Invocation Pattern

AGTP method invocations target a URI plus an optional endpoint
path. The conceptual pattern is:

~~~
METHOD agtp://[locator]/[endpoint-path]
~~~

The locator addresses an agent (Forms 1, 1a, 3, 4), a server
(Form 2), or an organization (Form 2a). The endpoint path is the
path portion of the request line as specified in {{wire-format}}.
For server-level and organization-level discovery operations the
endpoint path is omitted (target-less DISCOVER); for agent-level
operations the endpoint path identifies the endpoint to invoke.

The wire-level encoding of this pattern is the AGTP request line
(method and path) plus the connection target (host derived from
the URI). The URI is the agent-facing addressing notation; the
wire format is what travels over TLS.

### Web3 Verification Anchors

AGTP supports Web3-anchored verification paths for canonical
Agent-IDs whose underlying Agent Genesis is registered through a
blockchain-based verification service. The verification path is
declared in the agent's registry record per {{verification-paths}}
and leaves URI syntax unchanged: a Web3-anchored agent is addressed
by its canonical Agent-ID via Form 1 or Form 1a like any other
agent.

Integration with specific Web3 naming and resolution systems is
outside this document; the earlier Web3 bridge draft is withdrawn
and any future binding will be a companion specification.

### Non-Canonical Forms and Redirect Behavior

The following non-canonical forms **SHOULD** be redirected to their
canonical equivalents. Implementations **MUST NOT** serve package
contents in response to any URI form.

| Received URI | Canonical Redirect Target |
|---|---|
| `agtp://acme.tld/agents/customer-service.agent` | `agtp://acme.tld/agents/customer-service` |
| `agtp://acme.tld/agents/customer-service.nomo` | `agtp://acme.tld/agents/customer-service` |
| `agtp://acme.tld/agents/customer-service.agtp` | `agtp://acme.tld/agents/customer-service` |
{: title="Non-Canonical URI Forms and Redirect Targets"}

### Query Parameters for Format Selection

All AGTP URI resolution requests accept an optional `format` query
parameter controlling the serialization of the returned document.

| Query Parameter | Returned Representation |
|---|---|
| (none) | Agent Identity Document, human-readable `application/vnd.agtp+json` |
| `?format=manifest` | Agent Identity Document, human-readable `application/vnd.agtp+json` |
| `?format=json` | Agent Identity Document, compact `application/vnd.agtp+json` |
| `?format=certificate` | Agent Genesis fields only, `application/vnd.agtp+json` |
| `?format=status` | Lifecycle state and operational status only, `application/vnd.agtp+json` |
{: title="AGTP URI Format Query Parameters"}

All format variants return signed `application/vnd.agtp+json` content.
The `?format=json` parameter is intended for programmatic consumers.
The default returns the full human-readable manifest suitable for
browser rendering by an AGTP-aware client.

### Resolution Mechanics

AGTP URI resolution proceeds according to the URI form presented.
Form 1 and Form 1a (canonical agent identity) resolve through a
governance-platform registry or transparency-log lookup. Forms 2
and 2a (server and organization discovery) resolve through
direct AGTP connection establishment to the host or domain. Forms
3 and 4 (domain-anchored agents) resolve through DNS to an AGTP
server whose `hosted_agents` manifest maps the URI's local agent
name to a canonical Agent-ID.

#### Form 1 Resolution (Canonical ID)

When an AGTP resolver receives a URI of the form
`agtp://[256-bit-hex-id]`, it **MUST** perform the following steps:

1. Parse and validate the canonical Agent-ID. If the identifier is
   malformed (length, character set), return `400 Bad Request` with
   error code `invalid-canonical-id`.

2. Query the agent's governance platform registry for the record
   associated with the canonical Agent-ID. If the resolver fails to
   know which governance platform holds the record, it **MAY** query
   a transparency log per {{verification-paths}} to locate the record.

3. Verify the registry record lifecycle state. If `suspended`,
   return `503 Service Unavailable` with lifecycle state in the
   response body. If `retired`, return `410 Gone` with lifecycle
   state and revocation timestamp. If `deprecated`, the resolver
   **SHOULD** continue with resolution; the registry record
   **MUST** carry the deprecation signal so the relying party can
   surface it to clients and act on any `successor_agent_id`
   declared in the deprecation event.

4. Retrieve the agent's package (`.agent` or `.nomo`) from the
   package store referenced by the registry record.

5. **Verify the package integrity hash before proceeding.** If
   integrity verification fails, return `500 Internal Error` with
   error code `package-integrity-failure`. **MUST** be logged.

6. Extract the embedded manifest from the verified package.

7. Sign the Identity Document using the governance platform's signing
   key. Return the signed `application/vnd.agtp+json` document in the
   format specified by the query parameter.

Form 1 resolution requires no prior knowledge of an organization
domain, a DNS record, or a Web3 naming anchor. The canonical Agent-ID
is sufficient input.

#### Forms 2 and 2a Resolution (Server and Organization Discovery)

When an AGTP resolver receives a URI of Form 2 (`agtp://[host]`) or
Form 2a (`agtp://[domain]`), it **MUST** perform the following
steps:

1. Parse and validate the URI. Form 2 admits any RFC 3986 host with
   optional port. Form 2a admits a DNS-registered domain name with
   at least one label and **MUST NOT** include port or userinfo
   components.

2. Open an AGTP/TLS connection to the host or domain. For Form 2a,
   resolve the domain via DNS first. For both forms, AGTP traffic
   targets port 4480 unless an explicit non-default port appears in
   the URI (Form 2 only).

3. Issue the agent's intended method (typically `DISCOVER` for
   discovery operations) with no agent-identifying target.

4. The server returns its server manifest (for Form 2 or Form 2a
   addressed against a server's primary endpoint) or whatever
   server-level response the invoked method produces.

Forms 2 and 2a resolve to no canonical Agent-ID; they target
server or organization state directly. They take no part in
the canonical-Agent-ID identity model.

#### Forms 3 and 4 Resolution (Domain-Anchored Agents)

When an AGTP resolver receives a URI of Form 3 or Form 4, it
**MUST** perform the following steps:

1. Parse and validate the URI. If the URI is malformed, return
   `400 Bad Request` with error code `invalid-uri-form`.

2. Resolve the domain (Form 3) or `agtp.[domain]` subdomain
   (Form 4) via DNS, then open an AGTP/TLS connection to port 4480.

3. Look up the `agent-name` from the URI path against the server's
   `hosted_agents` manifest entries to obtain the canonical
   Agent-ID. If no matching agent is found, return `404 Not Found`.

4. Continue with Form 1 resolution steps 3 through 7 using the
   resolved canonical Agent-ID.

The package's executable content, code, logic, and any fields absent from
included in the Identity Document schema **MUST NOT** be returned
at any step of any resolution path. URI resolution exposes identity
and status exclusively.

### Trust Tiers and Verification Paths {#verification-paths}

The Agent Genesis carries a `trust_tier` field (one of `1`
Verified, `2` Org-Asserted, `3` Experimental) and a
`verification_path` field (one of `dns-anchored`, `log-anchored`,
`hybrid`, or `org-asserted`) recording how the agent's identity
was anchored at ACTIVATE time. These fields are surfaced in the
Agent Identity Document and consulted by Scope-Enforcement Points,
governance gateways, and peer agents during runtime authority
decisions.

Tier 1 agents are eligible for the full Authority-Scope vocabulary,
delegation chains, financial transactions, and multi-organization
collaboration. Tier 1 verification requires exactly one of three
verification paths (`dns-anchored`, `log-anchored`, `hybrid`) to
succeed at ACTIVATE time, each backed by distinct evidence (DNS
TXT record, transparency log inclusion proof, or DNS + blockchain
signature combination). All Tier 1 paths produce identity
attestations of equivalent strength.

Tier 2 agents declare organizational affiliation without
cryptographic verification (`verification_path: org-asserted`) and
carry the `trust_warning: "verification-incomplete"` signal. Tier
3 agents are experimental and confined to development environments.

The normative verification procedures, evidence requirements, tier
assignment rules, and Authority-Scope eligibility constraints are
specified in {{AGTP-TRUST}}.

### Subdomain Deployment Pattern

Organizations **SHOULD** deploy AGTP endpoints at a dedicated
subdomain following the pattern `agtp.[organization-domain.tld]`
(e.g., `agtp.acme.tld`). This is the recommended enterprise
deployment pattern: it provides clean separation between web and
agent infrastructure, allows independent certificate management for
the AGTP endpoint, and is consistent with service-specific subdomain
conventions. An organization with an AGTP subdomain **SHOULD** also
configure their primary domain to redirect AGTP requests:

~~~~
agtp://acme.tld/agents/customer-service
  → 301 → agtp://agtp.acme.tld/agents/customer-service
~~~~

### The /agents/ Reserved Path Prefix

The path prefix `/agents/` is reserved in all `agtp://` URIs for
agent namespace operations. Implementations **MUST** support this
prefix. The registry root at `/agents` (no trailing label) resolves
to the Agent Namespace Document ({{AGTP-IDDOC}}).

### Collision Prevention

The canonical Agent-ID is the collision-prevention primitive. Two
canonical Agent-IDs are distinct if and only if the 256-bit
identifiers differ, and the governance platform enforces uniqueness
at issuance time by deriving the ID from the Agent Genesis hash.

For alias forms, collision prevention operates at the namespace
level. `agtp://acme.com/agents/customer-service` and
`agtp://chrishood.com/agents/customer-service` resolve to distinct
canonical Agent-IDs because they are registered under different
DNS domains. Within a single governance zone, the governance
platform enforces uniqueness of agent local names at registration
time.

Infrastructure **MUST** use the canonical Agent-ID for all routing,
logging, and attribution operations. Alias URIs are a display and
discovery layer only. An alias that resolves to a canonical Agent-ID
different from the one carried in the `Agent-ID` header on a request
**MUST** cause the request to be rejected with `401 Unauthorized`
and **MUST** be logged.

### IANA Considerations for the agtp:// URI Scheme {#uri-scheme-template}

The `agtp://` URI scheme is registered with IANA per {{RFC7595}}.
Registration template:

URI scheme name:
: agtp

Status:
: Permanent

URI scheme syntax:
: `agtp://[Agent-ID]` (canonical identity, Form 1)
  `agtp://[Agent-ID]@[host]` (canonical identity with explicit host, Form 1a)
  `agtp://[host]` (server-level discovery, Form 2)
  `agtp://[domain]` (organization-level discovery, Form 2a)
  `agtp://[domain]/agents/[agent-name]` (domain-anchored agent, Form 3)
  `agtp://agtp.[domain]/agents/[agent-name]` (subdomain-anchored agent, Form 4)

URI scheme semantics:
: Identifies an AI agent, an AGTP server, or an organizational AGTP
  presence operating over the Agent Transfer Protocol. The
  authoritative agent identification form (Form 1) uses a 256-bit
  hex-encoded cryptographic identifier derived from the agent's
  Agent Genesis. Form 1a augments the canonical ID with an
  explicit host to enable direct addressing in deployments where
  the canonical ID has yet to be resolved through a registry.
  Forms 2 and 2a address servers and organizations respectively
  for discovery operations that target no specific agent.
  Forms 3 and 4 address agents by local name within a domain's
  namespace and resolve to canonical Agent-IDs through the
  server's `hosted_agents` manifest. Forms 3 and 4 differ only in
  deployment convention.

Applications/protocols that use this URI scheme:
: Agent Transfer Protocol (this document)

Interoperability considerations:
: The canonical Agent-ID form is the authoritative agent identity
  representation. Form 1a embeds a resolution hint alongside the
  canonical Agent-ID and **MUST NOT** be used to derive identity.
  Forms 3 and 4 resolve through DNS to an AGTP server endpoint
  whose `hosted_agents` manifest maps the URI path component to a
  canonical Agent-ID. Forms 2 and 2a resolve to no canonical
  Agent-IDs; they target server or organization state directly.
  Implementations **MUST** accept canonical Agent-IDs (Form 1) and
  **SHOULD** support at least one of the domain-anchored agent
  forms (Form 3 or Form 4). The port portion of any AGTP URI is
  OPTIONAL and defaults to the IANA-assigned port 4480 when
  omitted; ports **MAY** appear in Form 1a and Form 2 only. File
  format suffixes (`.agtp`, `.agent`, `.nomo`) **MUST NOT** appear
  in `agtp://` URIs.

Contact:
: Chris Hood, chris@nomotic.ai

References:
: This document

The `agtp://` URI scheme registration is open and unencumbered. No
intellectual property claims apply to the URI scheme itself.

### .well-known Bootstrap

Organizations operating an AGTP namespace under a DNS domain
**SHOULD** publish a bootstrap document at the well-known URI
`https://{domain}/.well-known/agtp` per {{RFC8615}}. The document is
returned over HTTPS and declares the organization's AGTP presence,
the endpoint at which AGTP traffic should be directed, and any
verification anchors the organization wishes to surface for
discovery.

The bootstrap document is JSON with the following fields:

~~~~json
{
  "agtp_version": "0.7",
  "endpoint": "agtp://agtp.acme.tld/agents",
  "namespace_root": "agtp://acme.tld/agents",
  "verification": {
    "dns_anchor": "_agtp.acme.tld",
    "log_anchor": null
  },
  "issuer": "https://ca.acme.tld",
  "discovery": {
    "namespace_document": "https://agtp.acme.tld/agents.json",
    "well_known_updated_at": "2026-04-30T00:00:00Z"
  }
}
~~~~

Resolvers encountering an `agtp://` URI for a domain they have yet to
previously interacted with **SHOULD** retrieve the bootstrap document
to learn the canonical AGTP endpoint and verification anchors before
attempting AGTP traffic. The bootstrap is advisory; absence of a
`.well-known/agtp` document is no indication that the domain has no
AGTP presence, only that the domain has chosen against advertising one
through this mechanism.

A `.well-known/agtp` document returning HTTP 404 is a positive
declaration that the domain operates no AGTP namespace. Resolvers
**MAY** cache this negative result for the period indicated by the
HTTP response's `Cache-Control` directive, with a default of 24
hours when no directive is present.

## Agent Identity Document (Summary) {#identity-document}

Every Canonical Agent-ID resolves to an Agent Identity Document: a
signed `application/vnd.agtp.identity+json` document that carries the
agent's Owner-ID, its declared Authority-Scope set, its trust posture
fields, its supported methods and content types, and the issuer
binding that makes the document tamper-evident. The Agent Namespace
Document, the `.agtp` package format, the human-readable identity
view served to browsers, and the status sub-resource are specified in
{{AGTP-IDDOC}}. This document carries only what the protocol depends
on:

- The document is the authority for the Owner-ID (which **MAY** be a
  Legal Entity Identifier bound per {{AGTP-LEI}}) and the full
  Authority-Scope set; a request's claimed Authority-Scope is
  validated as a subset of it ({{header-format}}).
- The document is signed, and the signature is verified before any
  field is trusted ({{AGTP-IDDOC}} specifies the integrity and
  freshness rules). When the optional issuer fields are absent the
  document is verifiable at Level 1 only ({{AGTP-CERT}} supplies
  Level 3).
- Resolution of an Agent-ID to its document follows {{AGTP-DISCOVERY}};
  the media types are registered in {{iana-media-types}}.

## Agent Registration Process

### Overview

An agent cannot participate in AGTP until it has been issued an Agent
Agent Genesis by a governance platform and assigned a canonical
Agent-ID derived from that certificate. Canonical Agent-IDs are issued
through the ACTIVATE transaction; they are never self-declared.

The Agent Genesis is the genesis record of an agent's legal
existence within the AGTP ecosystem. Its relationship to the canonical
Agent-ID is analogous to the relationship between a government-issued
birth registration and a social security number: the birth event
produces a permanent, authoritative identity record, and a durable
identifier is derived from it. The identifier follows the agent for
its entire lifecycle, including after revocation. It is never reissued
to another agent.

Any AGTP infrastructure component **MUST** reject requests carrying
an Agent-ID that fails to resolve to an Agent Genesis record in
an Active lifecycle state in a reachable registry.

### Agent Genesis Contents {#agent-genesis-contents}

The Agent Genesis is issued by the governance platform at
ACTIVATE time and contains the following fields:

| Field | Required | Description |
|---|---|---|
| `agent_id` | **MUST** | Canonical Agent-ID for this agent. Equals the 256-bit SHA-256 hash of the Agent Genesis in its canonical-form serialization with the `signature` field excluded. The field is included for relying-party convenience; verifiers **MUST** independently recompute the hash and confirm it matches before trusting the value. |
| `owner` | **MUST** | Human or team responsible for this agent |
| `archetype` | **MUST** | Behavioral category (see archetypes below) |
| `governance_zone` | **MUST** | Environment context (development, staging, production) |
| `scope` | **MUST** | Authorized action types |
| `issued_at` | **MUST** | Timestamp of issuance |
| `issuer_public_key` | **MUST** | Ed25519 public key of the governance platform (or self-issuer in development) that signed this Agent Genesis. 32 raw bytes encoded as base64url. Verifiers re-verify `signature` against this key. The SHA-256 fingerprint of these raw bytes is the Genesis-issuer fingerprint used for lifecycle-method authorization ({{lifecycle-auth}}). |
| `signature` | **MUST** | Detached Ed25519 signature over the canonical-form serialization (with the `signature` field excluded), produced by `issuer_public_key`'s corresponding private key. Self-signed Agent Genesis records (where `issuer_public_key` is the agent's own key) are permitted for development; production deployments **MUST** use a recognized issuer key. |
| `package_ref` | **SHOULD** | Reference to the `.agent` or `.nomo` package |
| `trust_tier` | **MUST** | Registration tier (1, 2, or 3) |
| `verification_path` | **MUST** (Tier 1) | Path used to verify identity: `dns-anchored`, `log-anchored`, `hybrid`, or `org-asserted` |
| `org_domain` | **SHOULD** | DNS-verified or asserted org domain (required for `dns-anchored` and `hybrid`) |
| `org_label` | **SHOULD** | Agent-native org label (required for Form 2 hierarchical resolution) |
| `log_inclusion_proof` | **MUST** (log-anchored) | Transparency log inclusion proof (RFC 9162 / RFC 9943) |
{: title="Agent Genesis Fields"}

The canonical-form serialization used for both the Agent-ID hash
and the signature **MUST** be deterministic: keys sorted
lexicographically, no insignificant whitespace, UTF-8 encoding,
and the `signature` field omitted from the serialized form.
Implementations **MUST** produce identical bytes for the same
logical Agent Genesis across implementations.

### Agent Archetypes

The `archetype` field classifies the agent's behavioral category.
Archetypes inform scope enforcement and observability tooling; an
`executor` archetype agent exhibiting read-only query patterns, or a
`monitor` archetype agent attempting booking operations, are anomaly
signals. The archetype field leaves scope enforcement unrestricted, 
Authority-Scope headers govern actual permissions at the protocol
level. Archetypes are a classification and observability signal,
never a security boundary.

| Archetype | Description | Typical Scope |
|---|---|---|
| `assistant` | Conversational agent, read-heavy | `documents:query`, `knowledge:query` |
| `analyst` | Data analysis, read and aggregate | `data:read`, `data:aggregate` |
| `executor` | Takes real-world actions, write-heavy | `booking:*`, `payments:confirm` |
| `orchestrator` | Manages other agents | `delegation:*`, `agents:*` |
| `monitor` | Observational only | `telemetry:read`, `logs:read` |
{: title="Agent Archetypes"}

### Agent Genesis to AGTP Protocol Mapping

Agent Genesis fields map to AGTP protocol elements that surface
during request processing. This mapping is the mechanism by which
static identity (the Agent Genesis) becomes runtime identity (the
protocol session):

| Agent Genesis Field | AGTP Protocol Element |
|---|---|
| `agent_id` | `Agent-ID` header on every request |
| `owner` | Principal identifier in the agent identity document; never transmitted as a separate header |
| `scope` | Full Authority-Scope set in the agent identity document; optionally narrowed per-request via the `Authority-Scope` header |
| Canonical Agent Genesis (full document) | Hashed to produce the canonical Agent-ID; the document itself is the identity anchor |
{: title="Agent Genesis to AGTP Protocol Mapping"}

The canonical Agent-ID is computed as
`sha256(canonical_form(Agent_Genesis_without_signature))`. The
chain (Agent Genesis document to canonical hash to Agent-ID
header) ensures that the identifier carried in the `Agent-ID`
header on every AGTP request is traceable back to the original
Agent Genesis and the human principal who authorized the
agent's creation. Any modification to the Agent Genesis
produces a different canonical Agent-ID; tampering is
therefore detectable by any verifier that recomputes the hash.

### Registration Tiers

Registration produces an Agent Genesis at one of three trust tiers
(Tier 1 Verified, Tier 2 Org-Asserted, Tier 3 Experimental). Tier 1
registration requires one of three verification paths
(`dns-anchored`, `log-anchored`, `hybrid`) to succeed at ACTIVATE
time, each backed by distinct cryptographic evidence. Tier 2
registration declares organizational affiliation without
cryptographic proof; the resulting Agent Genesis carries
`trust_warning: "verification-incomplete"`. Tier 3 registration is
confined to development environments and produces a locally-scoped
Agent Genesis.

The complete tier-specific packaging, evidence, and validation
requirements are specified in {{AGTP-TRUST}}.

### Registration Lifecycle {#registration-lifecycle}

~~~~
1. PACKAGE
   Author creates .agent or .nomo package containing:
   - Embedded manifest (agent_label, job_description,
     authority_scope_categories, supported_methods,
     trust_score)
   - Integrity hash of all package contents
   - For .nomo: CA-signed certificate chain

2. SUBMIT (ACTIVATE transaction)
   Registrant submits ACTIVATE request to governance endpoint:
   - Package file (.agent or .nomo)
   - Proposed agent label and optional org_domain or org_label
   - Owner identity (maps to Agent Genesis owner field)
   - Archetype declaration
   - Declared trust_tier and verification_path with corresponding
     tier-specific evidence (see {{AGTP-TRUST}} for the evidence
     required by each verification path)

3. VALIDATE (governance platform)
   Governance platform:
   - Verifies package integrity hash
   - For .nomo: validates certificate chain
   - Applies verification-path-specific validation per
     {{AGTP-TRUST}} (DNS challenge, transparency log submission,
     or hybrid evidence verification, depending on declared path)
   - Checks proposed label for uniqueness within the relevant
     namespace (org_domain, org_label, or log-scoped)

4. ISSUE (Agent Genesis and canonical Agent-ID assigned)
   Governance platform:
   - Issues Agent Genesis with all fields populated,
     including trust_tier and verification_path
   - Computes canonical Agent-ID as
     `sha256(canonical_form(Agent_Genesis_without_signature))`
   - For log-anchored Tier 1: submits Agent Genesis to
     transparency log and embeds inclusion proof in the registry
     record (see {{AGTP-LOG}})
   - Creates registry record with Active lifecycle state
   - Records genesis audit entry in immutable audit log
     (genesis record includes full Agent Genesis and
     verification evidence)
   - Publishes agent to Namespace Document
     (triggers Namespace Document re-signing)

   The Agent Genesis is delivered to the registrant.
   It is the permanent record of the agent's genesis.
   Loss of the Agent Genesis leaves the agent valid;
   the canonical Agent-ID derived from the original Agent
   Genesis remains the authoritative identity anchor.

5. ACTIVE
   Agent enters Active lifecycle state.
   Canonical Agent-ID is valid for AGTP protocol sessions.
   All applicable alias URIs resolve to the Agent Identity
   Document derived from the Agent Genesis.

6. LIFECYCLE EVENTS (post-activation)
   The five Lifecycle methods on the protocol floor
   ({{methods-section}}) drive post-activation state
   transitions. Each transition produces a signed lifecycle
   event in the agent's per-agent lifecycle stream and is
   retrievable via `INSPECT target=lifecycle`.

   DEACTIVATE: Agent temporarily inactive. Identity Document
               returns 503. Agent Genesis and canonical
               Agent-ID remain valid. Initiated by trust
               violation or operator decision. See
               {{methods-deactivate}}.
   REINSTATE:  Operator-authorized return to Active from
               Suspended. Agent Genesis unchanged. Recorded
               in the lifecycle stream. See
               {{methods-reinstate}}.
   REVOKE:     Agent permanently retired. Identity Document
               returns 410. Agent Genesis archived. Canonical
               Agent-ID retired permanently and never
               reissued. See {{methods-revoke}}.
   DEPRECATE:  Controlled end-of-life. Agent continues to
               serve traffic; deprecation event signals
               planned retirement and **MAY** carry a
               `successor_agent_id` and a `migration_deadline`.
               Agent Genesis retained per the retention policy
               in this section. See {{methods-deprecate}}.
~~~~
{: #agent-reg-lifecycle title="AGTP Agent Registration Lifecycle"}

### Contract-Related Lifecycle Events {#contract-lifecycle-events}

In addition to the five agent-state lifecycle events
above, AGTP servers that operate the Runtime Contract
Negotiation Substrate ({{AGTP-RCNS}}) emit three
contract-related events on the per-agent lifecycle stream
of the contract's originating agent. The events ride the
same signed-envelope lifecycle stream described in
{{methods-inspect}} and are retrievable via
`INSPECT target=lifecycle`; they are stored outside any
separate audit surface.

| Event Type | Triggered By | Subject |
|---|---|---|
| `rcns_propose_accepted` | A synthesized contract is instantiated, whether through the RCNS gate or an explicit PROPOSE | Originating Agent-ID; payload carries `synthesis_id`, `(method, path)`, recipe lineage, and `negotiation_origin` |
| `rcns_revoke` | `REVOKE target=contract` invocation per {{methods-revoke-contract}} | Originating Agent-ID; payload carries the revoked `synthesis_id` and operator-supplied reason |
| `rcns_release` | `SUSPEND synthesis_id=...` invocation per {{methods-suspend-contract}}, or `REVOKE target=stale-contracts` invocation per {{methods-revoke-stale-contracts}} when a drifted contract is evicted | Originating Agent-ID; payload carries the released `synthesis_id`. For agent-initiated release the reason is agent-supplied; for operator-fired stale-contract eviction the reason **MUST** be `policy-change-invalidation` and the operator's identifier **MUST** be recorded so audit consumers can distinguish operator-fired invalidations from agent self-releases |
{: title="RCNS Contract-Related Lifecycle Events"}

The three RCNS event types are reserved on the Agent
Identity Document lifecycle stream alongside the five
agent-state events. They are absent from the AGTP-LOG
transparency log statement vocabulary defined in
{{AGTP-LOG}}; transparency-log submission of contract
events, if any, is left to a future revision.

### Governance Tokens and Runtime Authorization

Following successful registration, the agent's Agent Genesis is
the static identity anchor. Runtime authorization for specific actions
is carried by Governance Tokens: signed, time-limited JWT artifacts
issued by the governance platform encoding a specific governance
verdict (ALLOW, DENY) for a specific action.

Governance Tokens **MUST NOT** be reused. Each action requires a
fresh evaluation and a fresh token. Default TTL is 30 seconds. The
token's `agent_id` field **MUST** match the canonical Agent-ID from
the Agent Genesis. Tokens that fail this validation **MUST** be
rejected and the failure **MUST** be logged.

A Governance Token authorizes a specific action, and the verifier
**MUST** be able to check that the action executed is the action
authorized. Every Governance Token **MUST** carry `action_digest`:
the SHA-256 of the {{RFC8785}} canonical serialization of the object
`{method, path, action, parameters, payload_type, payload_sha256}`
drawn from the request the token authorizes, with absent members
omitted. At dispatch the server recomputes the digest from the
request as received and **MUST** reject with 262 Authorization
Required and reason `token-action-mismatch` when the two differ,
logging both digests. A token therefore binds to one request shape;
a replay against a different path, parameter set, or payload fails
the comparison before any handler runs.

The relationship between Agent Genesis and Governance Token
parallels the relationship between a passport and a visa: the passport
establishes persistent identity; the visa encodes a specific
time-bounded permission. Holding a passport implies no particular
any particular visa.

### Friendly Name Availability and Re-Registration

An agent label becomes available for re-registration 90 days after
its associated agent enters Revoked or Deprecated lifecycle state.
The canonical Agent-ID and Agent Genesis are permanently archived.
The canonical Agent-ID **MUST NOT** be reissued under any
circumstances, including re-registration of the same label by the same
organization. This policy prevents ID reuse attacks in which a newly
registered agent inherits the trust history of a revoked predecessor.

# Method Definitions

## Design Philosophy {#methods-section}

AGTP methods are intent verbs rather than resource operations. Each method
expresses what an agent is trying to accomplish. Method names are
uppercase ASCII strings. Methods that modify state are NOT idempotent
by default unless explicitly marked. All methods accept a context
parameter carrying agent session state. Requirement language follows
{{RFC2119}}.

### The Sixteen-Method Floor

AGTP defines eighteen protocol-level methods that constitute the
embedded floor of the protocol. Any conformant AGTP implementation
**MUST** support all eighteen. The eighteen are organized as three
groups.

**Cognitive methods** express requests for information or reasoning.
A cognitive method invocation produces understanding without changing
state external to the agent's own session:

- QUERY: semantic data retrieval
- DISCOVER: locate agents, resources, or services
- DESCRIBE: retrieve operational capabilities of a known endpoint
- INSPECT: retrieve audit records, chain heads, and lifecycle entries from the responding server
- SUMMARIZE: synthesize provided content
- PLAN: produce an unexecuted sequence of actions toward a goal
- PROPOSE: submit a dynamic endpoint or method proposal

**Mechanics methods** perform actions, exercise authority, or change
state. A mechanics method invocation has external effect:

- EXECUTE: invoke a specific action or carried protocol payload
- DELEGATE: transfer execution to a sub-agent with bounded authority
- ESCALATE: route a decision to a higher authority or human principal
- CONFIRM: attest to a prior action, state, or item
- SUSPEND: pause an active session workflow with a resumption nonce
- NOTIFY: push information without expectation of synchronous response

**Lifecycle methods** transition an agent between registry states.
A lifecycle method invocation produces a signed lifecycle event in
the per-agent lifecycle stream and updates the agent's registry
status; it changes how the agent is treated by AGTP infrastructure
but invokes no application-layer behavior:

- ACTIVATE: transition an agent into the Active state, emit an activation event
- DEACTIVATE: transition an agent out of Active (typically to Suspended), emit a deactivation event
- REINSTATE: transition a Suspended agent back to Active, emit a reinstatement event
- REVOKE: permanently retire an agent's canonical Agent-ID, emit a revocation event
- DEPRECATE: transition an agent to Deprecated (controlled end-of-life; agent continues to accept traffic), emit a deprecation event

The cognitive/mechanics/lifecycle distinction is informational,
informational; servers behave identically regardless of the
category of a method. The classification provides a stable mental
model for spec readers and implementers and constrains future
additions to the floor.

EXECUTE is the generic carrier for application-layer payloads.
When a higher-level agent framework such as MCP, A2A, or ACP is
composed over AGTP (see {{composition-section}}), EXECUTE is the
method that dispatches the carried payload to the application based
on its Content-Type. EXECUTE absorbs the role that earlier drafts of
this specification considered for an INVOKE method.

## Core Methods {#core-methods}

The eighteen core methods are presented below in the cognitive group
followed by the mechanics group, then the lifecycle group. The four
methods present in v06 core that are absent from the v07 floor
(BOOK, SCHEDULE, LEARN, COLLABORATE) are demoted to Tier 2 standard
extended methods; their specifications continue to apply where
implementations choose to support them, and they are catalogued in
the AGTP-METHODS method catalog ({{AGTP-METHODS}}).

### QUERY

Purpose: Semantic data retrieval. The agent specifies what it needs to
know rather than where to find it. Distinguished from HTTP GET by expressing an
information need rather than retrieving a known resource at a known
location. Cognitive.

| Parameter | Required | Description |
|-----------|----------|-------------|
| intent | **MUST** | Natural language or structured expression of the information need |
| scope | **SHOULD** | Data domains or sources to include or exclude |
| format | **MAY** | Desired response format: structured, natural, raw |
| confidence\_threshold | **MAY** | Minimum confidence score for included results (0.0-1.0) |
| context | **MAY** | Session context for disambiguation |
{: title="QUERY Parameters"}

Response: Result set with confidence scores per item. Server **SHOULD**
indicate provenance of each result. Idempotent: Yes.

### DISCOVER

Purpose: Search for agents matching a capability need, or discover
the current attachment of a known Agent-ID. Returns a ranked list of
candidate Agent Identity Documents, each carrying the attachment at
which the agent is currently reachable. Distinguished from QUERY:
DISCOVER returns agents to talk to, QUERY returns data to consume.
Cognitive.

The two uses are distinguished by the vocabulary of {{AGTP-DISCOVERY}}
following RFC 6763: a DISCOVER whose criteria describe a need
(intent, capability domains, trust and governance constraints) is a
search and returns candidates to be evaluated; a DISCOVER whose
criteria name a single Agent-ID is a discovery and returns that
agent's signed attachment. The parameter set, the response shape,
the ranking rule, and the operational modes (direct Presence,
ANS-brokered, cross-scope) are specified normatively in
{{AGTP-DISCOVERY}}; the population a search runs over is the presence
substrate of {{AGTP-PRESENCE}}. This document fixes only the floor:
the method name, its safe and idempotent properties, the
`discovery:query` scope requirement, and the governance-zone rule
below.

Servers **MUST** apply governance-zone filtering: candidates outside
the requesting agent's permitted zones **MUST NOT** be returned.
Every result **MUST** carry the agent's Canonical Agent-ID; a result
with no Agent-ID is a search hit from an external catalog and
**MUST** be marked as such. Idempotent: Yes. Primary error codes:
422, 455.

### DESCRIBE

Purpose: Return the operational capabilities of a known agent endpoint.
The requesting agent specifies what capability dimensions it needs to
evaluate; the server returns a structured Capability Document. Used for
pre-task negotiation before committing to DELEGATE or EXECUTE. If
`capability_domains` is omitted, the server **SHOULD** return all supported
domains. Cognitive.

| Parameter | Required | Description |
|-----------|----------|-------------|
| capability\_domains | **SHOULD** | Comma-separated domains to return: `methods`, `modalities`, `tools`, `version`, `budget`, `zones`. If omitted, server **SHOULD** return all. |
| version\_min | **MAY** | Minimum acceptable version for capability negotiation. |
| context | **MAY** | Session context for capability filtering. |
{: title="DESCRIBE Parameters"}

Response: Capability Document with the following structure:

~~~~json
{
  "methods": ["QUERY", "DISCOVER", "DESCRIBE", "SUMMARIZE", "PLAN",
              "PROPOSE", "EXECUTE", "DELEGATE", "ESCALATE", "CONFIRM",
              "SUSPEND", "NOTIFY"],
  "modalities": ["text", "image", "streaming"],
  "tools": ["web_search", "code_execute"],
  "version": "0.7",
  "version_min_satisfied": true,
  "trust_score": 0.94,
  "budget_units_accepted": ["tokens", "compute-seconds"],
  "zones_accepted": ["zone:internal", "zone:partner"]
}
~~~~

Idempotent: Yes. Primary error codes: 404, 422.

### INSPECT {#methods-inspect}

Purpose: Retrieve audit records, per-agent chain heads, or lifecycle
log entries from the responding server's audit store. INSPECT is the
read counterpart to the per-action Attribution-Record and per-agent
audit chain established by the response path: a relying party that
holds an Audit-ID can fetch the underlying JWS, walk the chain by
following `previous_audit_id`, and verify the chain's integrity
end-to-end. Cognitive.

| Parameter | Required | Description |
|-----------|----------|-------------|
| target | **MUST** | Target family. One of `audit`, `chain_head`, `lifecycle`, `contract`, or `rcns-attempt`; the last two are the RCNS extensions whose semantics are in {{AGTP-RCNS}}. |
| audit_id | **CONDITIONAL** | Required when `target` is `audit`. The 64-character lowercase hex Audit-ID of the record to retrieve. |
| agent_id | **CONDITIONAL** | Required when `target` is `chain_head` or `lifecycle`. The canonical Agent-ID whose chain head or lifecycle stream is being retrieved. |
| limit | **MAY** | When `target` is `lifecycle`, the maximum number of entries to return. Servers **MAY** cap the value. |
{: title="INSPECT Parameters"}

Response shape depends on `target`:

- `target=audit`: returns the JWS Compact serialization of the
  Attribution-Record identified by `audit_id`, along with the
  parsed payload for convenience. The relying party **MUST**
  recompute `sha256(jws)` and confirm it equals the requested
  `audit_id` before trusting the parsed payload.
- `target=chain_head`: returns the most recently emitted
  Audit-ID for the named `agent_id`. Useful for chain walkers
  that need a starting point without an explicit Audit-ID.
- `target=lifecycle`: returns recent lifecycle log entries for
  the named `agent_id`, ordered newest-first. Each entry
  carries an explicit `format` field with value `"jws"` or
  `"cose"` reflecting the envelope used to sign the
  lifecycle event (see {{AGTP-LOG}}). The signed bytes are
  preserved on the wire so relying parties can re-verify
  signatures end-to-end. Servers **MAY** emit mixed-format
  streams when the operator has flipped the configured
  signing envelope between events; the per-entry `format`
  field disambiguates each line. The configured envelope is
  selected by operator policy (typically a `mode` setting in
  the audit configuration), with `jws` as the default and
  `cose` selected for SCITT-aligned deployments.
- `target=contract`: returns the full record for a
  synthesized contract identified by `synthesis_id`. The
  response includes the resolved `(method, path)` pair, the
  recipe lineage (recipe name and captured recipe version),
  the originating Agent-ID, the contract hash, the
  negotiation origin, the issuance and expiration timestamps,
  and the contract's current lifecycle state. See
  {{AGTP-RCNS}} for the contract concept and the RCNS
  substrate that produces and manages contracts.
- `target=rcns-attempt`: returns diagnostic information for
  a failed RCNS synthesis attempt identified by an attempt
  identifier. Every 464 RCNS No Contract response from an
  RCNS-capable server **MUST** carry the `RCNS-Attempt-Id`
  response header naming an attempt record retrievable
  through this target. The attempt record contains the
  requested `(method, path)`, the requesting Agent-ID, the
  resolved trust tier, the four-lock-gate evaluation
  outcomes, and any structured reason produced by the
  synthesis machinery. Attempt records are diagnostic
  surfaces only; servers **MAY** retain them in a ring
  buffer with a finite size and **MAY** evict older
  records when the buffer fills.

Servers **MUST** return `404 Not Found` when the requested record
or chain head is absent from the store. Servers **MAY** apply
read-access control over INSPECT responses. Three access modes
are defined:

- `public`: any caller may read any stored audit record, chain
  head, or lifecycle entry. This is the default and matches the
  posture expected for chain walkers and external verifiers.
- `agent_only`: a caller authenticated as Agent-ID `X` (via
  mTLS per {{AGTP-CERT}}, with the dispatcher's existing
  Agent-ID header cross-check against the verified certificate)
  may read records, chain heads, and lifecycle entries whose
  subject Agent-ID is `X`. Cross-agent reads are refused.
- `operator_only`: only callers presenting an mTLS certificate
  whose public-key fingerprint matches an operator-controlled
  allowlist may read INSPECT responses.

When access control is configured and the request fails the
configured check, servers **MUST** respond with `401 Unauthorized`
if the request presented no authenticated identity, and
`403 Forbidden` if the request presented an identity other than one
permitted under the configured mode. The response body **SHOULD**
identify which mode is in effect; it **MUST NOT** disclose
operator-allowlist contents or other principals' Agent-IDs.

INSPECT is intended for chain walkers, auditors, and external
verifiers; it is no substitute for the Attribution-Record
emitted on every response. The Attribution-Record carries the
authoritative attestation of the current response; INSPECT
retrieves a stored attestation by identifier after the fact.

Idempotent: Yes. Primary error codes: 400, 401, 403, 404, 422.

### SUMMARIZE

Purpose: Request a concise synthesis of provided content or a referenced
resource. The agent is requesting a cognitive operation on data rather than
retrieving data. Cognitive.

| Parameter | Required | Description |
|-----------|----------|-------------|
| source | **MUST** | Content inline (up to implementation limit) or URI reference |
| length | **SHOULD** | Target summary length: brief, standard, detailed |
| focus | **MAY** | Aspect to emphasize in the summary |
| format | **MAY** | Output format: bullets, prose, structured |
| audience | **MAY** | Intended reader context, for calibrating complexity |
{: title="SUMMARIZE Parameters"}

Response: Summary content with a source_hash and a confidence score.
Idempotent: Yes.

### PLAN

Purpose: Produce a sequence of actions or method invocations that
would accomplish a stated goal, without executing them. The response
is a structured plan that the requesting agent (or its principal)
reviews before committing to execution. Distinguished from the
deprecated v06 SCHEDULE method (which committed the plan): PLAN is
purely cognitive and produces no external state change. Cognitive.

| Parameter | Required | Description |
|-----------|----------|-------------|
| goal | **MUST** | Natural language or structured description of the desired outcome |
| constraints | **SHOULD** | Time, cost, scope, or policy constraints the plan must respect |
| available\_methods | **MAY** | Methods the planning agent may assume the executor will support |
| context | **MAY** | Session context informing plan selection |
{: title="PLAN Parameters"}

Response: Plan document containing `ordered_steps` (each an AGTP
method invocation specification), `estimated_cost`, `confidence`, and
`assumptions`. Idempotent: Yes. Primary error codes: 422, 503.

### PROPOSE

Purpose: Submit a dynamic endpoint or method proposal to a server
that has yet to declare support for the proposed method. PROPOSE
initiates the runtime contract negotiation flow specified in
{{AGTP-RCNS}} and summarized in {{dynamic-negotiation}}. The proposing agent submits a method name,
parameter signature, and intended outcome; the server responds with
acceptance, counter-proposal, or rejection. Cognitive.

| Parameter | Required | Description |
|-----------|----------|-------------|
| proposed\_method | **MUST** | Uppercase ASCII method name being proposed |
| signature | **MUST** | Parameter set, response shape, and outcome semantics |
| intent | **MUST** | Natural language statement of what the method would accomplish |
| contract\_version | **SHOULD** | Reference to the AGTP-METHODS contract version (default `AGTP-API/1.0`) |
| context | **MAY** | Session context informing the proposal |
{: title="PROPOSE Parameters"}

Response: Synthesis result. On accept, 263 Proposal Approved with
the AGTP-METHODS endpoint definition, `synthesis_id`, and `expires_at`.
On refuse, 463 Proposal Rejected with a structured `reason`. On
async evaluation, 261 Negotiation In Progress with a `proposal_id`
the agent can poll via `QUERY /proposals/{proposal_id}` (see
{{AGTP-RCNS}}). Idempotent: No. Primary error codes: 261, 262, 263,
400, 459, 460, 463.

### EXECUTE {#methods-execute}

Purpose: Invoke a specific action with parameters, or dispatch a
carried application-layer payload. EXECUTE is the protocol's generic
mechanics carrier. When a higher-level agent framework is composed
over AGTP, EXECUTE is the method that carries the framework's payload
and dispatches it to the application based on the carried Content-Type.
The Authority-Scope header is enforced against the action being
performed; scope violations return 455. Mechanics.

| Parameter | Required | Description |
|-----------|----------|-------------|
| action | **MUST** | Identifier of the action to invoke (application-defined when carrying a higher-level protocol) |
| parameters | **SHOULD** | Action-specific parameters (structured) |
| payload | **MAY** | Application-layer payload, when EXECUTE is carrying a higher-level protocol invocation |
| payload\_type | **MAY** | Content-Type of the carried payload, for dispatch |
| idempotency\_key | **MAY** | Client-provided key enabling safe retry |
| context | **MAY** | Session context |
{: title="EXECUTE Parameters"}

Response: Execution result, with structure determined by the
action or carried payload. Servers **MUST** return the AGTP envelope
(status, task_id, attribution) wrapping the action's response.
Idempotent: depends on the action; clients **SHOULD** supply
`idempotency_key` for retry safety. Primary error codes: 403, 422,
455, 456, 503.

### DELEGATE

Purpose: Transfer execution of a task or method to a sub-agent or
downstream system. Initiates a new AGTP session on behalf of the delegating
agent, carrying forward authority lineage. Mechanics.

| Parameter | Required | Description |
|-----------|----------|-------------|
| target\_agent\_id | **MUST** | Identifier of the agent to delegate to |
| task | **MUST** | AGTP method call (or sequence) to execute |
| authority\_scope | **MUST** | Scope granted to sub-agent **MUST** be a strict subset of delegating agent's scope |
| delegation\_token | **MUST** | Signed token proving delegation authority |
| callback | **SHOULD** | AGTP endpoint for result delivery |
| deadline | **MAY** | Maximum time for task completion |
{: title="DELEGATE Parameters"}

Security note: the authority_scope in a DELEGATE request **MUST NOT**
exceed the delegating agent's own Authority-Scope. Servers **MUST** enforce
this and **MUST** return 455 Scope Violation with reason
`scope-exceeds-parent` if violated. This is the protocol-level defense against
authority laundering. Idempotent: No.

### ESCALATE

Purpose: Route a task, decision, or exception to a human principal or
higher-authority agent when the current agent cannot or ought to stop.
ESCALATE is the protocol-level expression of meaningful friction in AI
systems as a first-class method. Mechanics.

| Parameter | Required | Description |
|-----------|----------|-------------|
| task\_id | **MUST** | The task or method invocation triggering escalation |
| reason | **MUST** | Structured reason: confidence\_threshold, scope\_limit, ethical\_flag, ambiguous\_instruction, resource\_unavailable |
| context | **MUST** | Full context needed for the escalation recipient to act |
| priority | **SHOULD** | urgent, normal, or low |
| recipient | **MAY** | Specific human or agent to escalate to; if absent, routes to default handler |
| deadline | **MAY** | Time by which a response is needed |
{: title="ESCALATE Parameters"}

Response: Escalation receipt with escalation_id and routing confirmation.
The escalated task is paused until resolved via CONFIRM. Idempotent: Yes.
An agent that escalates appropriately is functioning correctly. Governance
frameworks built on AGTP can use escalation frequency and reason codes as
observability signals for systemic issues.

### CONFIRM

Purpose: Explicit acknowledgment of a prior action, state, or data item.
Creates a signed attestation record. Mechanics.

| Parameter | Required | Description |
|-----------|----------|-------------|
| target\_id | **MUST** | ID of the action, booking, schedule, or item being confirmed |
| status | **MUST** | accepted, rejected, or deferred |
| reason | **SHOULD** (if rejected/deferred) | Explanation of the decision |
| attestation | **MAY** | Agent-signed confirmation payload for audit |
{: title="CONFIRM Parameters"}

Response: Confirmation receipt with timestamp and attestation_id.
Idempotent: Yes.

### SUSPEND {#methods-suspend}

Purpose: Pause a specific active session workflow in a recoverable state.
Issues a resumption nonce the requesting agent uses to resume the session.
Method-level SUSPEND is session-scoped and leaves registry
lifecycle state or Agent Genesis validity. The distinction between
method-level SUSPEND and lifecycle SUSPEND ({{registration-lifecycle}}) is
architectural: method-level SUSPEND is a workflow primitive; lifecycle
SUSPEND is an administrative action on the agent's registry record.
Mechanics.

| Parameter | Required | Description |
|-----------|----------|-------------|
| session\_id | **MUST** | The session to suspend. |
| reason | **SHOULD** | Structured reason: `awaiting_input`, `resource_limit`, `scheduled_pause`, `external_dependency`. |
| resume\_by | **MAY** | ISO 8601 deadline for resumption. If exceeded without RESUME, session transitions to expired. |
| checkpoint | **MAY** | Agent-provided state snapshot for resumption context. Stored by server for duration of suspension. |
{: title="SUSPEND Parameters"}

Response: Suspension receipt with the following structure:

~~~~json
{
  "suspension_id": "susp-0042",
  "session_id": "sess-a1b2c3d4",
  "resumption_nonce": "[128-bit random value, base64url]",
  "resume_by": "2026-04-15T09:00:00Z",
  "status": "suspended"
}
~~~~

The `resumption_nonce` **MUST** be a cryptographically random 128-bit
value encoded as base64url. It is single-use: once presented to resume
a session, the nonce is invalidated and **MUST NOT** be accepted again.
Idempotent: No. Primary error codes: 404, 408.

Servers MUST generate nonces with at least 128 bits of entropy using
a CSPRNG.

#### SUSPEND `synthesis_id=` {#methods-suspend-contract}

SUSPEND accepts an alternative parameter set for releasing a
synthesized contract the requesting agent originated. When
invoked with `synthesis_id` instead of `session_id`, the call
releases the named contract from the synthesis runtime, evicts
it from active resolution, and emits an `rcns_release` event
in the originating agent's lifecycle stream.

| Parameter | Required | Description |
|-----------|----------|-------------|
| synthesis_id | **MUST** | Opaque identifier of the contract being released, as issued by RCNS. |
| reason | **SHOULD** | Human-readable rationale recorded in the lifecycle event payload. |
{: title="SUSPEND synthesis_id Parameters"}

The release path is reserved for the originating agent
(self-release): a caller presenting a `synthesis_id`
**MUST** be the contract's `originating_agent_id` or the
server **MUST** return `403 Forbidden`. Operator-initiated
contract eviction uses `REVOKE target=contract`
({{methods-revoke-contract}}) and produces an `rcns_revoke`
event instead. The release leaves the
originating agent's lifecycle state. See {{AGTP-RCNS}} for
the RCNS contract concept.

### NOTIFY

Purpose: Asynchronous push of information from an agent to a recipient.
Expects no response. Fire-and-forget. Delivery confirmation (if
required) returned via a subsequent CONFIRM from the recipient. Mechanics.

| Parameter | Required | Description |
|-----------|----------|-------------|
| recipient | **MUST** | Target Agent-ID, human endpoint, or broadcast group |
| content | **MUST** | Notification payload |
| urgency | **SHOULD** | critical, informational, or background |
| delivery\_guarantee | **MAY** | at\_most\_once, at\_least\_once, or exactly\_once |
| expiry | **MAY** | Timestamp after which delivery of the notification **SHOULD** cease |
{: title="NOTIFY Parameters"}

Response: Delivery receipt with notification_id. Idempotent: No.

### ACTIVATE {#methods-activate}

Purpose: Transition an agent into the Active registry lifecycle
state. The responding server records the transition, signs a
lifecycle event into the agent's per-agent lifecycle stream, and
makes the agent eligible to be addressed by AGTP requests. The
canonical Agent-ID **MUST** already exist (its Agent Genesis
**MUST** have been issued and loaded); ACTIVATE never mints
Agent-IDs. Lifecycle.

A successful ACTIVATE is the publish event for the agent: it is the
trigger for the governance platform's registration of the agent
with the Agent Name Service of its zone per {{AGTP-DISCOVERY}}, and
for the agent's first ANNOUNCE into the presence substrate per
{{AGTP-PRESENCE}}. Neither is a separate registration step an
operator performs; both follow from the lifecycle transition, so
that an agent is findable exactly when it is Active and never
otherwise. DEACTIVATE, REVOKE, and DEPRECATE are the corresponding
withdrawal events.

| Parameter | Required | Description |
|-----------|----------|-------------|
| agent_id | **MUST** | Canonical Agent-ID of the agent being activated. |
| reason | **MAY** | Human-readable rationale recorded in the lifecycle event payload. |
| actor | **SHOULD** | Identifier of the operator or registrar invoking the transition. Recorded in the lifecycle event. |
{: title="ACTIVATE Parameters"}

Response: A status document containing the new `status` (`active`),
the `previous_status`, the `event_type` (`agent-genesis-issued` or
`agent-lifecycle-reinstated` depending on whether the agent has
prior lifecycle history), and the `audit_id` of the signed
lifecycle event. When the agent is already in the Active state,
the server **MUST** return `200 OK` with a `noop: true` flag and
**MUST NOT** emit a duplicate lifecycle event.

Lifecycle events are signed with the responding server's
manifest signing key. The signed envelope is appended to the
per-agent lifecycle stream and is retrievable via
`INSPECT target=lifecycle` per {{methods-inspect}}. Two
envelope formats are defined: JWS Compact per {{RFC7515}} and
COSE_Sign1 per {{RFC9943}}. Servers select one envelope by
operator policy; both are normatively conformant and the
choice leaves the wire contract of the Lifecycle
method itself, only the on-disk and INSPECT-returned event
representation. Servers that have yet to provision a manifest
signing key **MUST** emit lifecycle events with `alg: none`
per the fallback in {{AGTP-IDENTIFIERS}}; relying parties
**MUST** treat such events as unverified. See {{AGTP-LOG}}
for the envelope format details and the Audit-ID derivation
rule for each envelope.

ACTIVATE is the protocol-level replacement for the
package-binding ACTIVATE method extension referenced in earlier
revisions of this document. The two transactions converge: a
governance platform that operates a `.nomo` package binding flow
issues an ACTIVATE on completion of the binding to record the
state transition on the wire.

Idempotent: No. Authorization: see {{lifecycle-auth}}. Primary error codes: 400, 401, 403, 404, 422.

### DEACTIVATE {#methods-deactivate}

Purpose: Transition an agent out of the Active registry lifecycle
state, typically into Suspended. The responding server records
the transition, signs a lifecycle event, and ceases to treat the
agent as eligible to receive AGTP requests until a subsequent
REINSTATE returns it to Active. DEACTIVATE is reversible via
REINSTATE; permanent retirement uses REVOKE; controlled
end-of-life uses DEPRECATE. Lifecycle.

| Parameter | Required | Description |
|-----------|----------|-------------|
| agent_id | **MUST** | Canonical Agent-ID of the agent being deactivated. |
| reason | **SHOULD** | Human-readable rationale recorded in the lifecycle event payload (e.g., `compliance-hold`, `fraud-review`, `operator-pause`). |
| actor | **SHOULD** | Identifier of the operator or registrar invoking the transition. |
{: title="DEACTIVATE Parameters"}

Response: A status document containing the new `status`
(`suspended`), the `previous_status`, the `event_type`
(`agent-lifecycle-suspended`), and the `audit_id` of the signed
lifecycle event. When the agent is already in a non-Active state,
the server **MUST** return `200 OK` with `noop: true` and
**MUST NOT** emit a duplicate lifecycle event.

Idempotent: No. Authorization: see {{lifecycle-auth}}. Primary error codes: 400, 401, 403, 404, 422.

### REVOKE {#methods-revoke}

Purpose: Permanently retire an agent's canonical Agent-ID. The
responding server records the transition, signs a lifecycle
event, and refuses all subsequent requests addressed to the
agent. The Agent-ID **MUST NOT** be reissued to another agent.
REVOKE is non-reversible; an agent that needs to return to
service after REVOKE requires a new Agent Genesis and a new
canonical Agent-ID. Lifecycle.

| Parameter | Required | Description |
|-----------|----------|-------------|
| agent_id | **MUST** | Canonical Agent-ID of the agent being revoked. |
| reason | **MUST** | Human-readable rationale recorded in the lifecycle event payload (e.g., `compromise-detected`, `principal-request`, `policy-violation`). |
| actor | **SHOULD** | Identifier of the operator or registrar invoking the transition. |
{: title="REVOKE Parameters"}

Response: A status document containing the new `status`
(`retired`), the `previous_status`, the `event_type`
(`agent-genesis-revoked`), and the `audit_id` of the signed
lifecycle event. When the agent is already in the Retired state,
the server **MUST** return `200 OK` with `noop: true` and
**MUST NOT** emit a duplicate lifecycle event.

Servers **SHOULD** propagate REVOKE events to subscribing
registries and Agent Name Service operators within 60 seconds
per {{AGTP-TRUST}}. A revoked agent's certificate remains valid
for TLS-layer authentication only insofar as the
revocation event has yet to propagate; relying parties that
require strict revocation enforcement **MUST** consult the
lifecycle stream for the Agent-ID before treating an AGTP
exchange with that agent as authoritative.

Idempotent: No. Authorization: see {{lifecycle-auth}}. Primary error codes: 400, 401, 403, 404, 422.

#### REVOKE target=contract {#methods-revoke-contract}

REVOKE accepts an alternative parameter set for revoking a
synthesized contract identified by `synthesis_id` rather than
an entire agent. When invoked in this form, REVOKE evicts the
contract from the synthesis runtime, refuses subsequent
presentations of the `synthesis_id` with `464 contract-revoked`,
and emits an `rcns_revoke` event in the originating agent's
lifecycle stream.

| Parameter | Required | Description |
|-----------|----------|-------------|
| target | **MUST** | The literal string `contract`. |
| synthesis_id | **MUST** | Opaque identifier of the synthesized contract being revoked, as issued by RCNS. |
| reason | **SHOULD** | Human-readable rationale recorded in the lifecycle event payload. |
| actor | **SHOULD** | Identifier of the operator or originating agent invoking the revocation. |
{: title="REVOKE target=contract Parameters"}

Authorization for `REVOKE target=contract` is independent of
the lifecycle-method authorization in {{lifecycle-auth}}; servers
**MUST** accept the invocation from either the originating agent
(self-revocation) or from operators carrying the `inspect:all`
scope. Cross-agent revocation by any other principal **MUST**
return `403 Forbidden`. The contract revocation leaves
transition the originating agent's lifecycle state. See
{{AGTP-RCNS}} for the RCNS contract concept and the
`negotiation_origin` field that drives revocation semantics.

#### REVOKE target=stale-contracts {#methods-revoke-stale-contracts}

REVOKE accepts a third parameter set for sweeping
synthesized contracts whose captured `recipe_version`
diverges from the current loaded recipe version. The sweep
is an operator surface for handling drift after a recipe
edit; the full normative semantics (sweep walk, per-contract
disposition, `grandfather` vs `invalidate` modes, the
emitted `rcns_release` events for evictions) are specified
in the Stale Contract Sweep section in {{AGTP-RCNS}}.

| Parameter | Required | Description |
|-----------|----------|-------------|
| target | **MUST** | The literal string `stale-contracts`. |
| mode | **MAY** | One of `grandfather` or `invalidate`. Default is the server's `on_policy_change` configuration value. Invalid values return `400 Bad Request`. |
| actor | **SHOULD** | Identifier of the operator invoking the sweep, recorded on emitted `rcns_release` events. |
{: title="REVOKE target=stale-contracts Parameters"}

Authorization requires the operator `inspect:all` scope
token. The sweep leaves every agent's lifecycle
state; evicted contracts produce `rcns_release` events on
their originating agents' lifecycle streams per
the Stale Contract Sweep section in {{AGTP-RCNS}}.

### REINSTATE {#methods-reinstate}

Purpose: Transition a Suspended agent back to the Active registry
lifecycle state. The responding server records the transition,
signs a lifecycle event, and resumes treating the agent as
eligible to receive AGTP requests. REINSTATE is the recovery
counterpart to DEACTIVATE; an agent that was DEACTIVATEd for a
recoverable reason (compliance hold lifted, fraud review cleared,
operator pause ended) returns to service via REINSTATE.
REINSTATE preserves the canonical Agent-ID and the underlying
Agent Genesis; the reinstated agent is the same agent it was
before suspension. Lifecycle.

| Parameter | Required | Description |
|-----------|----------|-------------|
| agent_id | **MUST** | Canonical Agent-ID of the agent being reinstated. |
| reason | **SHOULD** | Human-readable rationale recorded in the lifecycle event payload (e.g., `compliance-hold-lifted`, `fraud-review-cleared`, `operator-resume`). |
| actor | **SHOULD** | Identifier of the operator or registrar invoking the transition. |
{: title="REINSTATE Parameters"}

Response: A status document containing the new `status`
(`active`), the `previous_status`, the `event_type`
(`agent-lifecycle-reinstated`), and the `audit_id` of the signed
lifecycle event. When the agent is already in the Active state,
the server **MUST** return `200 OK` with `noop: true` and
**MUST NOT** emit a duplicate lifecycle event. When the agent is
in the Retired state, the server **MUST** return `422
Unprocessable Entity`; REVOKE is non-reversible and REINSTATE
cannot bring a Retired agent back to service.

Idempotent: No. Authorization: see {{lifecycle-auth}}. Primary error codes: 400, 401, 403, 404, 422.

### DEPRECATE {#methods-deprecate}

Purpose: Transition an agent to the Deprecated registry lifecycle
state. A Deprecated agent continues to accept AGTP traffic
(distinguishing DEPRECATE from REVOKE and DEACTIVATE), but the
deprecation event signals planned end-of-life. Clients addressing
the agent **SHOULD** migrate to a successor agent or capability
within the migration window declared in the event payload.
DEPRECATE is the controlled-retirement path: it gives consumers
time to migrate before traffic stops. Lifecycle.

| Parameter | Required | Description |
|-----------|----------|-------------|
| agent_id | **MUST** | Canonical Agent-ID of the agent being deprecated. |
| reason | **SHOULD** | Human-readable rationale recorded in the lifecycle event payload. |
| successor_agent_id | **MAY** | Canonical Agent-ID of a successor agent that clients **SHOULD** migrate to. |
| migration_deadline | **MAY** | RFC 3339 timestamp after which the agent **MAY** be transitioned to Retired without further notice. |
| actor | **SHOULD** | Identifier of the operator or registrar invoking the transition. |
{: title="DEPRECATE Parameters"}

Response: A status document containing the new `status`
(`deprecated`), the `previous_status`, the `event_type`
(`agent-lifecycle-deprecated`), and the `audit_id` of the signed
lifecycle event. When the agent is already in the Deprecated
state, the server **MUST** return `200 OK` with `noop: true` and
**MUST NOT** emit a duplicate lifecycle event.

A Deprecated agent **MUST** continue to serve requests under the
same Authority-Scope and trust posture it held before
deprecation. Servers **SHOULD** surface the deprecation signal to
clients on every response from the deprecated agent; the
specific surfacing mechanism (response header, response body
field, DISCOVER listing flag) is implementation-defined and
**MAY** be specified normatively in a future revision.

A subsequent REVOKE against a Deprecated agent transitions it to
the Retired state and stops traffic. A subsequent ACTIVATE or
REINSTATE against a Deprecated agent transitions it back to
Active (cancelling the deprecation); the cancellation **MUST**
be recorded as a separate lifecycle event.

Idempotent: No. Authorization: see {{lifecycle-auth}}. Primary error codes: 400, 401, 403, 404, 422.

### Lifecycle Method Authorization {#lifecycle-auth}

Lifecycle methods (ACTIVATE, DEACTIVATE, REINSTATE, REVOKE,
DEPRECATE) transition the registry state of an agent's
canonical Agent-ID. Because the registry state determines
whether AGTP infrastructure treats the agent as operational,
unrestricted invocation of these methods would allow any
caller to suspend, retire, or deprecate any agent. Servers
**MUST** implement an authorization model for lifecycle
methods. Two modes are defined; future revisions **MAY**
define additional modes.

`open`:
: Any caller may invoke any lifecycle method against any
  agent the server hosts. This mode is appropriate only for
  development and single-tenant deployments where the
  operator implicitly trusts all callers. It is the simplest
  default and **MAY** be the implementation default; it
  **MUST NOT** be used in multi-tenant or production
  deployments.

`genesis_issuer`:
: A lifecycle method invocation **MUST** be presented over
  mTLS per {{AGTP-CERT}}, and the verified client
  certificate's public-key fingerprint **MUST** equal the
  Genesis-issuer fingerprint recorded for the target
  Agent-ID. The Genesis-issuer fingerprint is computed as
  `sha256(raw_ed25519_public_key_bytes(Agent_Genesis.issuer_public_key))`,
  giving a 256-bit value rendered as 64 lowercase
  hexadecimal characters. Only the registrar that issued an
  agent's Agent Genesis may transition that agent's
  lifecycle. Agents whose Agent Genesis is unloaded by the
  server (transport-only deployments) have no Genesis-issuer
  fingerprint to compare against; lifecycle methods against
  such agents under this mode **MUST** be refused.

When authorization fails, servers **MUST** respond with
`401 Unauthorized` if the request presented no verified
client certificate, and `403 Forbidden` if the request
presented a certificate whose fingerprint differed from the
Genesis-issuer requirement. The response body **SHOULD**
identify which mode is in effect.

Lifecycle method authorization is independent of INSPECT
read-access control. An operator **MAY** configure
`lifecycle_auth = genesis_issuer` (write-side governance)
alongside `read_acl = public` (read-side transparency) to
support a regulator-friendly posture in which the agent's
lifecycle history is publicly auditable but only the issuing
registrar may transition its state. Implementations **MUST**
log lifecycle authorization failures.

## Method Summary Table

| Method | Group | Intent | State-Modifying | Idempotent | Primary Error Codes |
|--------|-------|--------|----------------|------------|---------------------|
| QUERY | Cognitive | Retrieve information | No | Yes | 404, 422 |
| DISCOVER | Cognitive | Locate agents or resources | No | Yes | 422 |
| DESCRIBE | Cognitive | Retrieve endpoint capabilities | No | Yes | 404, 422 |
| INSPECT | Cognitive | Retrieve stored audit, chain head, or lifecycle records | No | Yes | 400, 401, 403, 404, 422 |
| SUMMARIZE | Cognitive | Synthesize content | No | Yes | 400, 422 |
| PLAN | Cognitive | Produce an unexecuted plan | No | Yes | 422, 503 |
| PROPOSE | Cognitive | Submit a dynamic endpoint proposal | No | No | 261, 262, 263, 400, 459, 460, 463 |
| EXECUTE | Mechanics | Invoke an action or carried payload | Yes | Per action | 262, 403, 422, 455, 456, 503 |
| DELEGATE | Mechanics | Transfer task to sub-agent | Yes | No | 262, 403, 551 |
| ESCALATE | Mechanics | Defer to human or authority | Yes | Yes | 404 |
| CONFIRM | Mechanics | Attest to a prior action | Yes | Yes | 404, 400 |
| SUSPEND | Mechanics | Pause session workflow | Yes | No | 404, 408 |
| NOTIFY | Mechanics | Push information | No | No | 400, 404 |
| ACTIVATE | Lifecycle | Transition agent to Active | Yes | No | 400, 401, 403, 404, 422 |
| DEACTIVATE | Lifecycle | Transition agent out of Active | Yes | No | 400, 401, 403, 404, 422 |
| REINSTATE | Lifecycle | Return Suspended agent to Active | Yes | No | 400, 401, 403, 404, 422 |
| REVOKE | Lifecycle | Permanently retire Agent-ID | Yes | No | 400, 401, 403, 404, 422 |
| DEPRECATE | Lifecycle | Signal planned end-of-life; agent continues serving | Yes | No | 400, 401, 403, 404, 422 |
{: title="AGTP Core Method Summary"}

## Method Registry and Extensibility {#method-registry}

AGTP defines a formal Method Registry maintained by IANA (see {{iana-methods}}).
Any party may submit a new method for registration. The registration
procedure is Expert Review, and registration **MUST** be accompanied by a
published specification, at minimum an IETF Internet-Draft or equivalent
publicly available document. Registered methods **MUST**:

1. Have a unique uppercase ASCII name
2. Define required and optional parameters
3. Define expected response structure
4. Specify idempotency behavior
5. Specify applicable error codes
6. Include a security considerations section
7. Be accompanied by a published reference specification (Internet-Draft or RFC)
8. Appear in the AGTP-METHODS approved method catalog {{AGTP-METHODS}}. The verb
   list is the authoritative source for what method names are
   recognized by AGTP servers; registered methods are a subset of the
   method catalog with formal IETF registration. Submissions whose method
   name is absent from the AGTP-METHODS method catalog **MUST** be rejected by the
   Designated Expert.

Experimental methods **MAY** be used prior to registration using the X-
prefix convention (e.g., X-NEGOTIATE). Experimental methods **MUST NOT**
be used in production deployments without registration. Experimental
methods are subject to verb-list validation; non-conformant
experimental methods **MUST** be rejected with 459 Method Violation
by AGTP-aware infrastructure components.

### Verb-List Validation

AGTP servers **MUST** validate inbound method names against the
AGTP-METHODS approved method catalog. A method name absent from the method catalog
**MUST** result in a 459 Method Violation response. A method name
in the method catalog whose path violates AGTP-METHODS path grammar **MUST**
result in a 460 Endpoint Violation response. A method name and
path that are individually valid but left unexposed by the server's
policy **MUST** result in a 405 Method Not Allowed response, with
the response body listing allowed methods for the path. The full
contract validation model is specified in {{AGTP-RCNS}}.

Capability negotiation occurs during session establishment. The server
returns a Supported-Methods header listing the methods it implements.
Clients **SHOULD** check this list before invoking non-core methods.

The `Negotiation-ID` header is used to correlate turns within a dynamic
endpoint negotiation sequence (see {{dynamic-negotiation}}). It **MUST** be a
UUID generated by the service upon receiving a PROPOSE request and
**MUST** be echoed in all subsequent turns of the same negotiation.
Maximum three turns before the agent **MUST** ESCALATE.

~~~
Negotiation-ID: 550e8400-e29b-41d4-a716-446655440000
~~~

QUOTE is defined as a Tier 2 Standard Extended Method in {{AGTP-METHODS}}.
QUOTE provides pre-flight cost estimation for a proposed method invocation:
the requesting agent submits a proposed method call; the server returns a
`Cost-Estimate` response without executing the method. Servers supporting
budget negotiation via the `Budget-Limit` header **SHOULD** implement QUOTE
to enable agents to validate cost before committing to execution. Servers
that implement QUOTE **MUST** list it in the `Supported-Methods` response
header at session establishment.

## Dynamic Endpoint Negotiation (Summary) {#dynamic-negotiation}

An agent that needs a method or endpoint the server has yet to
declare submits a PROPOSE request carrying an endpoint definition in
the form specified by {{AGTP-METHODS}}. The server evaluates the
proposal against the method catalog, path grammar, its own policy,
and the four-lock dispatcher gate, and answers with 263 Proposal
Approved (the endpoint is synthesized and the response names it),
261 Negotiation In Progress, 262 Authorization Required, or 463
Proposal Rejected with a structured reason. A server may also treat an
unregistered method and path pair as a synthesis candidate without a
PROPOSE, delivering a contract with 461 RCNS Contract Available or
inline under a `Contract-Synthesized` header. The full negotiation
protocol, including contract scoping, lifecycle, and revocation, is
specified in {{AGTP-RCNS}}.

## Extended Method Vocabulary (Summary)

Methods beyond the eighteen-method floor are registered in the
catalog specified by {{AGTP-METHODS}}: at catalog version 1.0.0, 425
methods in nine categories, with the floor as the embedded set and
the five HTTP verbs mapped to their preferred AGTP forms as legacy
entries. Industry profiles, custom methods declared in a server's
method policy, and the method properties (safe, idempotent, action,
authority-bearing) are specified there, with the empirical basis for
intent-named methods documented in {{HOOD2026}}. A method name in neither the
floor nor the catalog nor the server's custom set is refused with 459
Method Violation; a path that leaks a method name is refused with
460 Endpoint Violation.

# Delegated Authority {#delegation}

AGTP specifies delegation on two sides of execution. The
pre-authorization side, specified here, is the conveyance of narrowed
authority from a delegating agent to a sub-agent before the
sub-agent's request is accepted. The post-execution side, specified
in {{AGTP-IDENTIFIERS}}, is the attribution of a completed action to
the sequence of agents that caused it, through the `prior_actions`
chain in the Attribution-Record. The two compose: the Delegation
Sequence a request carries is the same sequence its Attribution-Record
later records.

## The Delegation Sequence

A Delegation Sequence is the ordered set of DELEGATE invocations that
produced a request. Each hop is minted by one DELEGATE invocation
({{methods-section}}) and consists of:

- the delegating agent's canonical Agent-ID (the parent),
- the sub-agent's canonical Agent-ID (the delegate),
- the `authority_scope` granted to the delegate, which **MUST** be a
  strict subset of the parent's effective Authority-Scope at the time
  of minting, and
- the `delegation_token`: the parent's signature over the three fields
  above, the parent's own hop (or the token `origin` for the first
  hop), the minting time, and an expiry.

The strict-subset rule is enforced at mint: a server processing a
DELEGATE whose `authority_scope` falls outside a strict subset of the
delegating agent's effective scope **MUST** refuse with 455 Scope
Violation and reason `scope-exceeds-parent` ({{status-codes}}). Same-scope relay, where a delegate exercises
exactly its parent's scope, is therefore outside delegation; it is a
request the parent makes itself, with the relay as transport. The
rule is also enforced at verify ({{delegation-verification}}), so a
sequence minted by a non-conformant server fails at the first
conformant verifier.

## The Delegation-Chain Header {#delegation-chain-header}

A request made under delegated authority **MUST** carry the
`Delegation-Chain` header. A request the agent makes under its own
authority **MUST NOT** carry it. The header carries the Delegation
Sequence as an ordered list of hops, origination to current:

~~~~
Delegation-Chain = hop *( OWS "," OWS hop )
hop              = agent-id ";" delegation-token
agent-id         = 64HEXDIG          ; canonical Agent-ID
delegation-token = 1*( ALPHA / DIGIT / "-" / "_" )   ; base64url JWS
OWS              = *( SP / HTAB )
~~~~

The first hop names the originating agent and carries its token
signed over `origin`. Each subsequent hop names the delegate of the
previous hop. The last hop names the agent making the request, whose
Agent-ID **MUST** equal the request's `Agent-ID` header.

### Delegation Token Format {#delegation-token}

A delegation token is the compact JWS serialization of {{RFC7515}},
algorithm `EdDSA`, `kid` equal to the parent's Agent-ID, over the
{{RFC8785}} canonical form of:

~~~~
{
  "parent":    "<parent Agent-ID, or the literal origin>",
  "delegate":  "<delegate Agent-ID>",
  "scope":     [ "<scope-token>", ... ],
  "parent_tok":"<SHA-256 of the parent hop's token, or null>",
  "iat":       <minting time, seconds since epoch>,
  "exp":       <expiry, seconds since epoch>,
  "depth":     <0 for the origin hop, parent depth plus 1 after>,
  "authority": "<digest reference to an authority assertion, or null>"
}
~~~~

The signing key is the parent's `agent_public_key` as published in
its Identity Document ({{AGTP-IDDOC}}) or bound by {{AGTP-CERT}}. For
the origin hop the originating agent signs over `parent: "origin"`
and `parent_tok: null`; its `scope` **MUST** be a subset of its own
Identity Document grant, which is the check {{delegation-verification}}
performs at step 3 for that hop. `parent_tok` binds each hop to the
exact parent token, so a parent swap changes the digest and fails
step 2 even when the swapped-in parent is a valid agent. `authority`
is the optional digest reference described in
{{authority-evidence}}.

The header is a carrier. The claim it carries, that the requesting
agent holds authority narrowed from the origin through every listed
hop, exists only after verification.

## Verification {#delegation-verification}

A server receiving a request with a `Delegation-Chain` header
**MUST** verify the sequence before dispatch, in order from the first
hop to the last. Verification is a function of a fixed input set: the
header as received, the request's `Agent-ID` and `Authority-Scope`,
the verification time, the server's configured maximum depth, and
the Identity Documents and lifecycle states resolved for the listed
Agent-IDs. A verifier **MUST NOT** consult any other input and
**MUST NOT** apply local policy inside verification; the relying
party applies its policy to the result afterwards, which is what
lets two conforming verifiers given the same inputs produce the same
result. For every hop the verifier **MUST** check:

1. **Membership.** The hop's Agent-ID resolves ({{AGTP-DISCOVERY}})
   to an agent in the Active lifecycle state. Failure reason:
   `member-inactive`.
2. **Parent re-verification.** The hop's delegation token verifies
   under the parent's published key, and the parent named in the
   token equals the previous hop's Agent-ID (or `origin` for the
   first hop). A token that verifies under a key other than the
   listed parent's, or that names a different parent, is a parent
   swap. Failure reason: `link-unverifiable`.
3. **Attenuation.** The hop's granted scope is a strict subset of the
   parent's scope as carried in the parent's hop (for the first hop,
   of the originating agent's identity-document scope). Failure
   reason: `scope-exceeds-parent`.
4. **Freshness.** The hop's token is unexpired, and the minting
   time is no earlier than the parent hop's minting time. Failure
   reason: `chain-expired`.
5. **Depth and cycles.** The sequence length is within the
   server's configured maximum depth (default 8), and no Agent-ID
   appears twice. Failure reasons: `depth-exceeded`,
   `cycle-detected`.

Verification has three outcomes. It is **valid** when every check
passes; **invalid** when any check fails on an input the verifier
holds; and **indeterminate** when a check cannot run because an
input is unavailable (an Agent-ID that cannot be resolved, a
lifecycle state the registry failed to answer for, an Identity
Document that failed to fetch). An indeterminate outcome is
never reported as valid and never reported as invalid; it is carried
as 551 with `evaluation: incomplete` and `missing_input` naming the
input ({{failure-fields}}), and the relying party **MUST** deny by
default. A verifier that is offline for the registry therefore
produces indeterminate for every delegated request rather than
either admitting on stale state or refusing as if the chain were
broken.

The accepted result of a valid verification is the effective scope:
the scope of the last hop, which by construction is the intersection
of every hop's scope with the origin's. The request's claimed
`Authority-Scope`, when present, **MUST** be a subset of the effective
scope; the endpoint's `required_scopes` are evaluated against the
claimed or effective scope exactly as for an undelegated request
({{header-format}}). Verification **MUST** be repeated on every
request; a verified sequence **MUST NOT** be cached as authority
across requests, since membership and freshness change.

## Failure Behavior

A verification failure **MUST** be refused with 551 Authority Chain
Broken, with `evaluation: refused` for an invalid outcome or
`evaluation: incomplete` for an indeterminate one, the `reason` token
from the list above (or `input-unavailable` with `missing_input` when
indeterminate), and, in `failing_hop`, the zero-based index of the
hop that failed. The
server **MUST NOT** dispatch any part of the request, **MUST** log the
event, and **MUST** produce an Attribution-Record for the refusal as
for any failure ({{failure-fields}}). A request whose `Agent-ID`
differs from the last hop's Agent-ID is refused with 551 and reason
`link-unverifiable`. A `Delegation-Chain` header on a request whose
method is a lifecycle method ({{methods-section}}) is refused with
551 and reason `scope-exceeds-parent`, since lifecycle authority is
never delegable.

## Evidence of the Delegator's Authority {#authority-evidence}

A verified Delegation Sequence proves that no hop exceeds its parent
and that the origin hop is within the originating agent's Identity
Document grant. It proves that the registrar that issued the agent
recorded that grant; it leaves unproven that the person or
organization behind the registrar could lawfully confer it. For most
traffic the registrar's record is sufficient. Where a relying party
in another administrative domain needs the stronger statement (a
bank receiving a payment instruction from a broker's agent needs to
know that the broker's signatory could delegate payments at all),
AGTP carries the evidence rather than producing it.

The origin hop's `authority` field **MAY** carry a digest reference
to an authority assertion: a signed statement from an authority
source, separate from the sequence, of the powers the delegator
could delegate. AGTP defines no authority assertion format of its
own. Two are recognized: the vLEI Role Credential chain (OOR, ECR,
AUTH) carried per {{AGTP-LEI}}, whose AUTH credential names the
human signatory and the role under which the agent was authorized;
and the Authority Assertion of the WIMSE agent mandate work, when the
relying party's trust framework accepts it. A verifier that is asked
by policy to require authority evidence **MUST** check that the
assertion's delegator matches the origin hop's Owner-ID and that the
origin hop's `scope` is within the assertion's stated powers; an
origin hop with no `authority` reference under such a policy is
indeterminate with `missing_input: authority`.

Typed constraints on delegated authority (bounded amounts, time
windows, accumulated budgets, co-signature thresholds) are the
natural next step for payment-grade delegation. AGTP's Authority-Scope
is a token set and `Budget-Limit` is its only numeric constraint; a
constraint vocabulary over scope tokens is planned.

### What Each Artifact Asserts {#delegation-attribution}

Each artifact in a delegated request asserts one thing, and a
verifier **MUST NOT** treat a claim as established by an artifact
whose role fails to cover it.

| Artifact | Asserts | Leaves unasserted |
|---|---|---|
| Agent Genesis | that a registrar issued this Agent-ID to this owner with this grant | that the owner could lawfully confer the grant |
| Agent Identity Document | the agent's current grant, key, and lifecycle state, as its issuer records them | that the sender of any request is this agent |
| `Request-Signature` | that the sender of this request holds the agent's current key | anything about delegated authority |
| origin hop token | that the originating agent narrowed its own grant to this scope | that the grant itself was within the owner's authority |
| subsequent hop token | that the named parent conferred this scope on this delegate, and that it was within the parent's hop | that the delegate was unmanipulated in requesting |
| authority assertion (when carried) | the powers the delegator could delegate, per the authority source | that any delegation was in fact made |
| Attribution-Record | what the agent did, after the fact | that the action was authorized before it ran |
{: title="Delegation Artifacts and Their Claims"}

A valid sequence shows that the request is within delegated
authority. It leaves unshown whether the requesting agent was
manipulated into making it; that is outside what any artifact in the
sequence can assert, and a relying party that needs it looks to the
governance platform's runtime verdict ({{AGTP-TRUST}}) and to the
Governance Token's action digest rather than to the chain.

## Mapping to the WIMSE Chain Convention {#wimse-mapping}

The multi-hop convention converging across the WIMSE agent identity
work carries three claims: `sub` is the principal at every hop,
`client_id` is the current actor, and `act` is the history of prior
actors. AGTP carries the same three facts in its own elements, and a
Delegation Sequence maps onto the convention without loss:

| WIMSE claim | AGTP element |
|---|---|
| `sub` (principal at every hop) | Owner-ID of the originating agent, resolved from its Identity Document and unchanged along the sequence |
| `client_id` (current actor) | `Agent-ID` of the request, equal to the last hop |
| `act` (prior actors) | the hops of `Delegation-Chain` before the last, in order |
| scope at this hop | effective scope of {{delegation-verification}} |
| root authority evidence | the origin hop checked against the originating agent's Identity Document grant, plus the optional authority assertion reference of {{authority-evidence}} for the delegator's own powers |
{: title="Delegation Sequence Mapped to the WIMSE Chain Convention"}

Narrowing follows the subset rule stated in that work: what a hop
passes on is a strict subset of what it received, token by token
over the Authority-Scope grammar, the narrowing party signs the hop,
and a party that forwards without narrowing makes no authorization
claim and appears in no hop.

AGTP states its staleness bound as S plus L. S is the maximum age of
the authorization state a verifier relied on when it admitted a
request: the Identity Document freshness window of {{AGTP-IDDOC}} and
the `Request-Signature` acceptance window ({{request-signature}}).
L is the maximum lifetime of a hop's delegation token without
re-validation, which is the token's expiry checked at step 4 of
{{delegation-verification}}; L **MUST NOT** exceed 24 hours and
**SHOULD** be shorter for action methods. The bound limits new
admissions and never the completion of an admitted action. Absence
of enforcement is never a small bound: a verifier that skips a check
has an unbounded S for that check and **MUST** record the request as
possession-unproven or sequence-unverified. Implementations **SHOULD**
publish one test vector exactly on each limit.

## Status of This Section

Specified in this revision. The reference implementation targets
verification steps 1 through 5 and the request signature in its v10
build; revocation of a member after minting is caught at step 1 on the next
request, and the continuity of that check across certificate expiry
and session boundaries is planned ({{revocation-continuity}}).
Delegation across organizations composes with draft-rampalli-pedigree,
draft-rampalli-cross-org-delegation-mapping, and
draft-hulea-wimse-agent-mandate: AGTP's per-hop attenuation and
parent re-verification satisfy the chain requirements those
documents state, the origin check and the authority assertion
reference of {{authority-evidence}} correspond to the mandate work's
Authority Assertion, the three-outcome verification and the
validator-versus-policy split follow its Validation Result model,
and an AGTP Delegation Sequence can be mapped row for row into the
cross-organization template. AGTP carries whichever chain format the
WIMSE group settles on and competes with none of them.

# Composition with Higher-Level Frameworks {#composition-section}

AGTP is intended to compose with, never replace, MCP, A2A, ACP, ANP, and other agent
application frameworks. AGTP is the substrate those frameworks need
to operate at scale. The frameworks define what agents say to one
another; AGTP defines how those messages move, who is allowed to send
them, and how their effects are attributed.

This section establishes composition with higher-level frameworks as
a first-class use case of AGTP, specifies the precedence rules
between AGTP transport semantics and framework-level messaging
semantics, and provides the canonical mapping table from common
framework concepts onto AGTP primitives. Detailed composition
profiles for each framework are specified in {{AGTP-COMPOSITION}}.

## Substrate Model

In a composed deployment, AGTP supplies four properties at the wire
level that no framework currently supplies natively:

1. **Identity at the protocol level.** Every AGTP request carries a
   canonical Agent-ID in the `Agent-ID` header. The framework's
   payload can omit identity; identity is established by the
   transfer protocol.

2. **Authority bounded at the protocol level.** The
   `Authority-Scope` header declares what the requesting agent is
   permitted to do. Servers enforce scope before dispatching to the
   framework's payload handler. A scope violation returns 455 at the
   AGTP layer, before the framework sees the payload.

3. **Attribution at the protocol level.** Every method invocation
   produces an Attribution-Record. The framework's payload can omit
   re-implement audit logging; the attribution is produced by the
   transfer protocol.

4. **Delegation at the protocol level.** Multi-hop agent workflows
   carry their Delegation Sequence in the `Delegation-Chain` header,
   verified per {{delegation}}, independent of framework-level
   concepts of "session" or "task."

A framework composed over AGTP can shed the parts of its design that
exist only because HTTP lacked these properties. The result
is a framework that can focus on its actual contribution (tool
semantics, conversational structure, multi-agent choreography) while
the substrate handles the cross-cutting concerns.

## EXECUTE as the Generic Carrier

When a framework is composed over AGTP, EXECUTE is the canonical
method that carries the framework's payload. The carried payload's
Content-Type identifies the framework; the AGTP server dispatches the
payload to the framework's handler. See {{methods-execute}}.

The EXECUTE invocation supplies:

- The framework identifier (via `payload_type`).
- The framework-specific operation (via the `action` parameter).
- The framework-specific payload (via the `payload` parameter).
- An optional `idempotency_key` enabling safe retry.

The AGTP envelope (status, task_id, attribution) wraps the
framework's response. A framework that wishes to expose a particular
operation as a first-class AGTP method (rather than carrying it
through EXECUTE) **MAY** register a catalog method per {{AGTP-METHODS}}; this is reserved for frameworks whose operations
are widely used enough to justify a dedicated verb.

## Precedence Rule

AGTP headers (Agent-ID, Authority-Scope, Delegation-Chain,
Session-ID, Task-ID) **MUST** take precedence over equivalent fields
in a carried framework payload for routing, enforcement, audit, and
identity purposes. Infrastructure components including SEPs and
governance gateways **MUST** use AGTP header values for all
protocol-level decisions.

Framework-level identity, session, or scope fields **MAY** be present
in the body for application-layer use but **MUST NOT** override AGTP
header values. If an inconsistency is detected between an AGTP header
and a corresponding framework-level field, the AGTP header is
authoritative; the inconsistency **SHOULD** be logged.

## Composition Profiles

The canonical mapping of A2A, MCP, and ACP concepts to AGTP
constructs, the external identity provider patterns, the HTTP gateway
profile, and the wire examples are specified in {{AGTP-COMPOSITION}}.
Two positions this document holds for those profiles:

- The A2A provenance chain maps to the Delegation Sequence
  ({{delegation}}) on the pre-authorization side and to the
  `prior_actions` chain of {{AGTP-IDENTIFIERS}} on the attribution
  side.
- Tool and resource identity is carried through composition profiles
  rather than as a first-class AGTP header. This is a deliberate
  choice under review; a first-class carrier is planned if working
  group review prefers it.

# Merchant Identity and Agentic Commerce Binding

## Overview

AGTP specifies agent-side identity through the Agent Genesis,
canonical Agent-ID, Agent Identity Document, and Trust Tier model. PURCHASE invocations carrying
`payments:purchase` in the Authority-Scope header and a Budget-Limit
constraint are fully governed on the sending side. The receiving side
of a PURCHASE, the merchant counterparty, lacks an
equivalent protocol-level identity in the base specification.

The merchant-side identity model is specified in {{AGTP-MERCHANT}}.
The integration is hook-based: this document registers the required
headers ({{header-format}}), the 458 status code ({{status-codes}}),
and the `merchant` and `intent` Authority-Scope domains; the detailed
semantics, Merchant Manifest Document schema, Merchant Genesis
structure, and counterparty verification procedure are specified in
the companion.

## Merchant Identity Headers (Summary)

PURCHASE invocations in a fully conformant v04 deployment carry the
following additional headers:

- `Merchant-ID`: canonical identifier of the intended merchant
  counterparty.
- `Merchant-Manifest-Fingerprint`: SHA-256 fingerprint of the
  Merchant Manifest Document the requesting agent verified during
  pre-flight counterparty verification.
- `Intent-Assertion`: detached JWT carrying principal-authorized
  purchase intent, forwardable to payment networks as standalone
  evidence.
- `Cart-Digest`: digest of a structured cart returned by a prior
  QUOTE invocation, binding this PURCHASE to that cart.

Full field definitions, wire examples, and security requirements are
in {{AGTP-MERCHANT}}.

# Security Considerations

This section satisfies the mandatory IETF Security Considerations
requirement. All AGTP implementations **MUST** address the considerations
described here.

## Mandatory TLS

All AGTP connections **MUST** use TLS 1.3 or higher. Implementations
**MUST** reject connections using TLS 1.2 or below. Certificate validation
follows standard PKI practices per {{RFC5280}}. Servers **MUST** present
a valid certificate.

## Agent Identity Verification: Three Levels {#verification-levels}

AGTP defines three distinct levels at which agent identity and
Authority-Scope can be verified. Each level serves a different deployment
profile and operational tradeoff. Understanding the distinction is
essential for implementers: the AGTP Agent Certificate extension
({{AGTP-CERT}}) is OPTIONAL, and base AGTP provides cryptographic
verification at the application layer without it.

**Level 1 - Self-asserted headers (raw request fields).**
Every AGTP request **MUST** include the `Agent-ID` header, which
references the agent identity document carrying the principal
identifier and Authority-Scope. As a raw header value on an
individual request, `Agent-ID` is self-asserted: a client writes
the value into the request and the server records what was written.
Level 1 verification is limited to mandatory logging and anomaly
detection against the recorded stream. This is the minimum baseline
every AGTP implementation provides.

**Level 2 - Application-layer cryptographic verification (signed
Agent Identity Document plus proof of possession).** A canonical
Agent-ID resolves to a signed Agent Identity Document ({{AGTP-IDDOC}})
that carries the Agent Genesis's Authority-Scope grant and the agent's
current `agent_public_key`, signed by the issuer. A verifier
(including a stranger with no prior relationship to the agent's
organization) verifies identity, scope, and possession at the
application layer by performing the following steps:

1. Resolve the canonical Agent-ID to retrieve the signed Agent
   Identity Document.

2. Verify the issuer's signature on the document against the
   issuer's published key.

3. Confirm that the canonical Agent-ID in the document matches the
   hash of the Agent Genesis.

4. Read the Authority-Scope grant and the `agent_public_key` from the
   verified document.

5. Verify the request's `Request-Signature` under that key per
   {{request-signature}}.

Steps 1 through 4 establish that the identity exists and what it may
do. Step 5 establishes that the sender is that identity. A verifier
that performs steps 1 through 4 and skips step 5 has verified a
document rather than a sender, and the request is at Level 1 for every
purpose that depends on who sent it. Level 2 is available in base AGTP
without the Agent Certificate extension and is the identity mechanism
the protocol depends on.

**Level 3 - TLS-layer cryptographic verification (AGTP-CERT
extension).** The AGTP Agent Certificate extension {{AGTP-CERT}}
binds the canonical `Agent-ID` and Authority-Scope to an X.509 v3
certificate presented during TLS 1.3 mutual authentication. The
principal identifier remains in the agent identity document
referenced by `Agent-ID`. Level 3 accelerates the Level 2 check to
the TLS handshake and enables Scope-Enforcement Points (SEPs) to
verify Authority-Scope at O(1) per-request cost without
application-layer access. Level 3 is an acceleration and
enforcement path for Level 2 rather than a replacement of it. Deployments
that require line-rate scope enforcement at infrastructure layers
(load balancers, governance gateways) **SHOULD** implement
{{AGTP-CERT}}.

> Note: The Agent Certificate extension and the Agent Genesis
> mechanism may be subject to pending intellectual property claims.
> See {{ipr}} and the IPR Notice preceding the Abstract for details.
> The licensor is prepared to grant a royalty-free license to implementers.

Every AGTP server **MUST** log the `Agent-ID` value from every
request and the principal identifier resolved from the
corresponding agent identity document, creating an attributable
audit trail at Level 1 even in deployments that omit
Level 2 retrieval or Level 3 transport binding.

## Authority Scope Enforcement

The Authority-Scope header declares what actions the agent is authorized
to take. Compliant AGTP servers **MUST** parse the Authority-Scope on every
request, return 455 Scope Violation for any method that exceeds declared
scope, and log all scope violations for audit purposes. At Level 1, scope
declarations are self-asserted in the request header, analogous to scope
assertions in OAuth 2.0 {{RFC6749}}. At Level 2, scope is cryptographically
verifiable through the signed Agent Identity Document; servers **SHOULD**
retrieve and verify the manifest for any Agent-ID whose declared scope
exceeds read-only operations. Level 3 cryptographically signed and
infrastructure-enforced scopes are defined in {{AGTP-CERT}}.

## Threat Model

### Agent Spoofing

Threat: A malicious actor forges the `Agent-ID` header to
impersonate a trusted agent. Mitigation: Level 2 application-layer
verification binds a declared Agent-ID to the signed Agent Identity
Document retrieved via canonical ID resolution. A forged Agent-ID
either fails to resolve or resolves to a manifest whose signature
cannot be verified against the claimed governance platform's published
key. Level 3 raises the mitigation to the TLS handshake via
{{AGTP-CERT}}. Implementations **SHOULD** retrieve and verify the
manifest for any Agent-ID carrying scope beyond read-only query
operations. Mandatory Level 1 logging provides an anomaly-detection
baseline for deployments that perform no active verification on
every request.

### Authority Laundering

Threat: An agent claims an Authority-Scope broader than what it was
granted. Mitigation: server-side scope enforcement; 455 Scope
Violation (reason `scope-claim-invalid`) returned and logged. In DELEGATE
chains, each hop's scope **MUST** be a strict subset of the delegating
agent's scope.

### Delegation Sequence Poisoning

Threat: A malicious agent inserts itself into a Delegation Sequence,
substitutes a parent (parent swap), widens its granted scope, or
extends a sequence past the point where its members remain
authorized.

Mitigation: the per-hop verification of {{delegation}}: parent
re-verification catches the swap, attenuation checked at verify
catches the widening, membership catches the revoked or inactive
member, and depth and cycle limits bound the sequence. Each hop
**MUST** also be logged with the delegating Agent-ID, the delegate
Agent-ID, and the granted scope, so the sequence can be reconstructed
for audit from the hop records and compared with the `prior_actions`
chain of {{AGTP-IDENTIFIERS}}. Single-server reconstruction depends
on the server's own records; reconstruction a distrusted holder
cannot tamper with depends on inclusion in a transparency log per
{{AGTP-LOG}}. Token signatures verify under keys published in the
identity document; line-rate verification at an SEP requires
{{AGTP-CERT}}.

### Revocation Continuity (Planned) {#revocation-continuity}

Revocation reaches a verifier through three boundaries with three
clocks: certificate validity ({{AGTP-CERT}}), session lifetime
({{AGTP-SESSION}}), and agent lifecycle state (the lifecycle methods
of {{methods-section}}). This revision specifies one piece of the
continuity: a member of a Delegation Sequence whose lifecycle state
is no longer Active fails verification on the next request
({{delegation-verification}}). A single specification of revocation
reaching every verifier by effect time, across all three boundaries,
is planned, in this document or as a companion.

### Attribution-Record Tamper Evidence

An Attribution-Record is produced and held by the agent whose action
it records. Against a verifier that trusts the holder, the signature
and the per-agent hash chain of {{AGTP-IDENTIFIERS}} suffice. Against
a verifier that distrusts the holder, the record is tamper-evident only when it
is included in a transparency log per {{AGTP-LOG}} and the inclusion
proof is checked. This document states the dependency; the mechanism
is inherited from those two companions.

### Denial of Service via High-Frequency Agent Traffic

Threat: Agents that are compromised, misconfigured, or adversarial generate
extremely high request volumes. Mitigation: 429 Rate Limited status code.
Rate limiting **SHOULD** be applied per Agent-ID and per resolved
principal identifier (obtained from the agent identity document
referenced by Agent-ID). When {{AGTP-CERT}} is deployed,
per-Agent-ID quotas can be cryptographically tied to verified
identity, preventing quota evasion through Agent-ID spoofing.

### Session Hijacking

Threat: An attacker intercepts or forges a Session-ID. Mitigation:
mandatory TLS protects sessions in transit. Session-IDs **MUST** be
cryptographically random with minimum 128 bits of entropy. Servers **MUST**
validate that Session-ID, Agent-ID, and TLS client identity are consistent.

### Escalation Suppression

Threat: A compromised agent or intermediary suppresses ESCALATE requests,
preventing human oversight. Mitigation: compliant implementations **MUST**
route ESCALATE requests directly to the declared escalation handler without
modification. Intermediaries **MUST NOT** drop, delay, or modify ESCALATE
requests. Escalation handlers **SHOULD** implement independent receipt
confirmation.

### Agent Genesis Spoofing

Threat: A malicious actor fabricates an Agent Genesis to claim a
legitimate agent's identity or construct a false identity with elevated
trust. Mitigation: Agent Genesis documents are issued only by governance
platforms that have completed one of the three Tier 1 verification
paths ({{verification-paths}}). For `dns-anchored` registrations, the governance
platform **MUST** verify DNS ownership of the claimed `org_domain`
before issuance. For `log-anchored` registrations, the governance
platform **MUST** submit the Agent Genesis to a transparency log
per {{RFC9162}} / {{RFC9943}} and record the inclusion proof in the
registry; tampering with a log-anchored Agent Genesis is
detectable by any party with log access. For `hybrid` registrations,
both DNS and blockchain address ownership are verified. In the base
spec, mandatory logging provides auditability. Full mitigation
requires {{AGTP-CERT}} for cryptographically bound Agent Genesis
verification at the TLS layer. Governance platforms **MUST**
treat any ACTIVATE request whose computed canonical Agent-ID
matches an existing registry record as a collision attack and
**MUST** reject it.

### Domain Transfer Identity Hijacking

Threat: An attacker acquires an expired domain to inherit the agent
registry and trust history of prior registrants. Mitigation applies
to `dns-anchored` and `hybrid` Tier 1 agents: agents under an expired
domain are automatically Suspended within 24 hours of domain expiry
detection. A new owner of the domain **MUST NOT** inherit prior agent
registrations. See {{retention}} for the full domain expiry policy.
`log-anchored` Tier 1 agents are unaffected by this threat because
their verification evidence is the transparency log inclusion proof
rather than DNS ownership.

### Attribution Forgery

Threat: A malicious agent submits a fabricated or replayed
Attribution-Record to claim credit for an action another party performed, or
to conceal the true execution context of an action it did perform.

Mitigation: Attribution-Records used for accountability or admission
to an audit trail **MUST** be signed with the agent's governance key.
The signature **MUST** cover the full record including the
Task-ID, Agent-ID, method, timestamp, and result hash. When {{AGTP-CERT}}
is deployed, the signature is verified at the TLS layer against the
agent's X.509 certificate. For high-stakes domains, RATS attestation
evidence in the Attribution-Record per {{RFC9334}} provides hardware-rooted
proof of execution context that cannot be forged without compromising the
attesting environment itself. Attribution-Record signatures **MUST** be
verified before the record is admitted to an audit trail. Unverified records
**MUST** be logged with a `signature_unverified` flag and **MUST NOT** be
treated as authoritative for compliance purposes.

The `alg: none` fallback permitted on the Attribution-Record response
header ({{response-headers}}) exists to preserve wire format and
Audit-ID chain construction in development and transition deployments
that have yet to provision a manifest signing key. An `alg: none`
record carries no anti-forgery claim. Consumers **MUST** treat
`alg: none` records as equivalent to unverified records for the
purposes of this mitigation: they **MUST NOT** be admitted to an audit
trail as authoritative, and they **MUST NOT** satisfy any
accountability requirement that depends on cryptographic signing.
Production deployments **MUST** configure a manifest signing key.

## Privacy Considerations

Agent identity carried on requests, and the agent identity
document referenced by `Agent-ID`, carry information about agent
behavior that may be sensitive:

- Agent-ID together with the resolved principal identifier may
  reveal organizational structure
- Session-ID and Task-ID reveal workflow patterns
- Delegation-Chain reveals multi-agent architecture

AGTP logs containing these fields **MUST** be treated as sensitive
operational data. Operators **MUST** implement appropriate access controls,
retention limits, and data minimization practices consistent with
applicable privacy regulations.

Where privacy-preserving attribution is required, implementations **MAY**
use pseudonymous Agent-IDs with a separate trusted resolution service.
The architecture for pseudonymous agent identity resolution is reserved
for a future companion document.

## Denial-of-Service Considerations

AGTP's agent identity provides a mechanism for more precise
denial-of-service mitigation than is possible with HTTP. Rate
limiting **SHOULD** be applied per Agent-ID and per resolved
principal identifier (obtained from the agent identity document)
in addition to per-IP-address controls.

When {{AGTP-CERT}} is deployed, per-Agent-ID rate limiting can be
cryptographically tied to verified agent identity, preventing quota evasion
through Agent-ID rotation. Implementations planning high-volume governed
agent deployments **SHOULD** plan for {{AGTP-CERT}} as part of their
denial-of-service mitigation strategy.

Additional recommended mitigations: traffic-shaping by request
class once the `Priority` header is normatively specified (reserved
for v01+ per {{header-format}}); and circuit breaker patterns for
ESCALATE request floods.

## Intellectual Property Considerations {#ipr}

The core AGTP specification, including all base methods, header fields,
status codes, connection model, and IANA registrations defined in this
document, is intended for open implementation without royalty obligation.

Certain elements referenced in this document may be subject to pending
patent applications by the author, specifically:

- The Agent Certificate extension {{AGTP-CERT}}, which provides
  cryptographic binding of agent identity and authority scope to AGTP
  header fields.
- The ACTIVATE method, which provides AGTP-native transmission and
  activation of governed agent packages.
- The Agent Genesis mechanism ({{agent-genesis-contents}}), which provides
  the genesis identity record and canonical Agent-ID derivation
  process for AGTP-registered agents.
- The `.agent` file format specification, an open packaging format
  for AI agents.
- The `.nomo` file format specification, a governed packaging format
  for AI agents with cryptographic governance binding.

Implementers of the core AGTP specification are unaffected by any
intellectual property claims on these extensions and associated formats.

The licensor is prepared to grant a royalty-free license to implementers
for any patent claims that cover contributions in this document and its
referenced extensions, consistent with the IETF's IPR framework under
{{RFC8179}}.

IPR disclosures have been filed with the IETF Secretariat and are available
at: https://datatracker.ietf.org/ipr/

# IANA Considerations

This document records the following IANA registrations.

## Port Assignment {#iana-ports}

The following service names and port number are registered in the
IANA Service Name and Transport Protocol Port Number Registry per
{{RFC6335}}:

| Service Name | Port | Transport | Description |
|---|---|---|---|
| agtp | 4480 | TCP | Agent Transfer Protocol over TCP/TLS |
| agtp-quic | 4480 | UDP | Agent Transfer Protocol over QUIC |
{: title="AGTP Port Assignments"}

The unified port assignment (4480 for both TCP and UDP under a
single conceptual `agtp` service) follows the precedent set by HTTPS
(443/TCP and 443/UDP for HTTP/3). The transport is distinguished at
the protocol level rather than at the port number.

Contact:
: Chris Hood, chris@nomotic.ai

Reference:
: This document

## AGTP Method Registry {#iana-methods}

Establishment of an IANA registry: Agent Transfer Protocol Methods.

Registry name:
: Agent Transfer Protocol Methods

Registration procedure:
: Expert Review per {{RFC8126}}, with the additional requirement that each
  registration be accompanied by a published specification, at minimum a
  publicly available Internet-Draft or equivalent document. The Designated
  Expert **SHOULD** verify that the proposed method name is unique, the
  reference specification is publicly accessible, the method definition
  includes the required fields (parameters, response structure, idempotency,
  error codes, security considerations), and the method conforms to the
  AGTP-METHODS specification {{AGTP-METHODS}}.

Reference:
: This document

Initial registrations (the eighteen-method protocol floor):

| Method | Group | Status | Reference |
|--------|-------|--------|-----------|
| QUERY | Cognitive | Permanent | {{core-methods}} |
| DISCOVER | Cognitive | Permanent | {{core-methods}} |
| DESCRIBE | Cognitive | Permanent | {{core-methods}} |
| INSPECT | Cognitive | Permanent | {{methods-inspect}} |
| SUMMARIZE | Cognitive | Permanent | {{core-methods}} |
| PLAN | Cognitive | Permanent | {{core-methods}} |
| PROPOSE | Cognitive | Permanent | {{core-methods}} |
| EXECUTE | Mechanics | Permanent | {{core-methods}} |
| DELEGATE | Mechanics | Permanent | {{core-methods}} |
| ESCALATE | Mechanics | Permanent | {{core-methods}} |
| CONFIRM | Mechanics | Permanent | {{core-methods}} |
| SUSPEND | Mechanics | Permanent | {{core-methods}} |
| NOTIFY | Mechanics | Permanent | {{core-methods}} |
| ACTIVATE | Lifecycle | Permanent | {{methods-activate}} |
| DEACTIVATE | Lifecycle | Permanent | {{methods-deactivate}} |
| REINSTATE | Lifecycle | Permanent | {{methods-reinstate}} |
| REVOKE | Lifecycle | Permanent | {{methods-revoke}} |
| DEPRECATE | Lifecycle | Permanent | {{methods-deprecate}} |
{: title="Initial AGTP Method Registry Entries (Eighteen-Method Floor)"}

The methods BOOK, SCHEDULE, LEARN, and COLLABORATE, present in the
v06 core set, are demoted in v07 to Tier 2 standard extended methods
and are registered through the AGTP-METHODS method catalog ({{AGTP-METHODS}})
rather than through this document.

## AGTP Status Code Registry {#iana-status}

Establishment of an IANA registry: Agent Transfer Protocol Status Codes.

Registry name:
: Agent Transfer Protocol Status Codes

Registration procedure:
: Expert Review + published specification required.

AGTP-specific status code numbers are deliberately chosen from
ranges unassigned in the IANA HTTP Status Code Registry to avoid
semantic collision with HTTP status codes that may appear in
payloads carried by AGTP method invocations.

Two status codes used by AGTP retain their HTTP code numbers (408
and 410) but carry AGTP-specific semantics. They are registered here
with text describing the AGTP semantic.

The following AGTP status codes are registered with full definitions:

| Code | Name | Definition | Reference |
|------|------|------------|-----------|
| 261 | Negotiation In Progress | The service has received a PROPOSE request and is evaluating it asynchronously. The response body **MUST** include a `proposal_id` and an estimated evaluation duration. The agent retrieves the terminal status by invoking `QUERY /proposals/{proposal_id}` until a 263 (Proposal Approved) or 463 (Proposal Rejected) response is returned. Server policy controls whether async evaluation is offered. | {{AGTP-RCNS}} |
| 262 | Authorization Required | Authorization the agent could obtain is absent: credential establishment, consent, or an unclaimed scope its identity document grants. Never used when the request exceeds held authority (455). The response body **MUST** specify which authorization condition applies: `scope-required` (the endpoint requires Authority-Scope the agent has yet to declare), `wildcards-required` (the request is an ad-hoc method invocation and the wildcards consent on either the agent identity document or the server policy is absent), `credentials-missing` (the server requires credentials such as AGTP-CERT or OAuth-scoped token before evaluating the request), or `anonymous-discovery-disabled` (the server requires authenticated identity for manifest retrieval and the request is unauthenticated). Returned for PROPOSE-time authorization, endpoint-dispatch scope checks, ad-hoc invocation wildcards refusal, and discovery requests blocked by policy. | {{AGTP-METHODS}} |
| 263 | Proposal Approved | The service has accepted the PROPOSE request and instantiated the proposed endpoint. The response body **MUST** contain a complete AGTP-METHODS endpoint definition for the instantiated endpoint, a `synthesis_id` identifying the synthesized endpoint for follow-on invocation, and `expires_at` indicating when the synthesis expires. **MAY** include a `persistent` boolean and `granted_duration` indicating the actual duration the server granted (which may be less than the agent requested, bounded by server policy). | {{AGTP-RCNS}} |
| 405 | Method Not Allowed | The method is recognized and the path is valid, but the server's policy or registry leaves this combination unexposed. The response body **MUST** list allowed methods for the path and any redirects from the manifest's `policies.methods` sub-block. The agent **MAY** PROPOSE the combination when policy leaves it unexposed. | {{AGTP-METHODS}} |
| 408 | Timeout | The method's declared TTL expired before execution completed. AGTP-specific semantics distinct from HTTP's request-timeout: applies to AGTP method TTL rather than transport request timeout. **MUST** be logged. | {{status-codes}} |
| 410 | Gone | The Agent-ID is permanently retired through REVOKE of the underlying Agent Genesis. AGTP-specific semantics distinct from HTTP's resource-removed: applies to permanent identity retirement. A Deprecated agent does **NOT** return 410 and continues to serve traffic; deprecation signals planned end-of-life rather than retirement. The canonical Agent-ID of a retired agent **MUST NOT** be retried. | {{status-codes}} |
| 455 | Scope Violation | The request exceeds the authority the agent holds (Authority-Scope claim beyond grant, endpoint scope ungranted, delegation beyond parent) or an operator-defined scope dimension other than rate-limit, budget, or zone (dedicated codes 429, 456, 457). Typical applications include token-based scope violations and query-based scope violations where the operator defines a scope dimension outside the standard set. The server **MUST** log this event. The agent **MUST NOT** retry the same request without modifying its scope declaration. Governance signal rather than a protocol error. | {{status-codes}} |
| 456 | Budget Exceeded | The requested method execution would exceed the resource limits declared in the Budget-Limit request header. The agent **MUST NOT** retry without modifying the Budget-Limit or reducing request scope. Governance signal; **MUST** be logged. | {{status-codes}} |
| 457 | Zone Violation | The request would route outside the network boundary declared in the AGTP-Zone-ID header. SEP-enforced. The agent **MUST NOT** retry without modifying the AGTP-Zone-ID or obtaining explicit cross-zone authorization. **MUST** be logged. | {{status-codes}} |
| 458 | Counterparty Unverified | The merchant counterparty in a PURCHASE invocation failed identity verification. Returned when the `Merchant-ID` or `Merchant-Manifest-Fingerprint` request headers are absent, when the fingerprint differs from the receiving server's current Merchant Manifest Document, when the Merchant-ID differs from the server's canonical ID, or when the merchant is in a non-Active lifecycle state. Governance signal; **MUST** be logged. Full definition in {{AGTP-MERCHANT}}. | {{AGTP-MERCHANT}} |
| 459 | Method Violation | The method name is absent from the AGTP-METHODS approved method catalog. The method itself is the problem. The response body **MUST** identify the unrecognized method and **SHOULD** reference the AGTP-METHODS method catalog version in effect. The agent **MUST** select a different method before retrying, or **MAY** PROPOSE the method if no suitable replacement exists. | {{AGTP-RCNS}} |
| 460 | Endpoint Violation | The endpoint path violates AGTP-METHODS path grammar. A path segment matches an approved method name (case-insensitive), indicating method-name leakage into the path. The response body **MUST** identify the offending path segment. The agent **MUST** restructure the path before retrying. | {{AGTP-METHODS}} |
| 461 | RCNS Contract Available | The requested endpoint is unregistered but the server is prepared to synthesize a contract for it. The response body **MUST** carry a contract preview including the resolved method, path, and `synthesis_id`; the caller **MAY** accept the contract by re-issuing the request with the `Contract-Synthesized` header or decline by ignoring the response. Returned in the confirm-first RCNS delivery mode per {{AGTP-RCNS}}. | {{AGTP-RCNS}} |
| 462 | Insufficient Budget | Commerce request budget envelope is insufficient for the receiving agent's pricing tier or settlement requirements. Returned when a caller's Budget-Limit header is below the receiving agent's published price for the requested method/scope, or when the declared settlementMethod matches none of the receiver's accepted settlement endpoints. Full definition in {{AGTP-COMMERCE}}. | {{AGTP-COMMERCE}} |
| 463 | Proposal Rejected | The service cannot or declines to instantiate the proposed endpoint. Returned in response to PROPOSE. The response body **MUST** include a structured `reason` field with one of the values `out-of-scope`, `policy-refused`, `composition-impossible`, `ambiguous`, or `synthesis-disabled`, an `explanation` string, and **MAY** include an optional `counter_proposal` suggesting a related endpoint the server would accept. | {{AGTP-RCNS}} |
| 464 | RCNS No Contract | An RCNS synthesis attempt was made but no contract could be delivered. The response body **MUST** include a structured `reason` field with one of the values `rcns-disabled`, `trust-tier-insufficient`, `composition-impossible`, `synthesis-error`, `contract-not-yours`, or `contract-revoked`, and an `explanation` string. See {{AGTP-RCNS}}. | {{AGTP-RCNS}} |
| 465 | Reserved | Reserved for AGTP expansion. | {{status-codes}} |
| 550 | Delegation Failure | A sub-agent to which a task was delegated via the DELEGATE method failed to complete the task within the declared deadline or returned an error. The response body **SHOULD** contain the sub-agent's error details. | {{status-codes}} |
| 551 | Authority Chain Broken | The Delegation Sequence carried in the `Delegation-Chain` header fails verification: membership, parent re-verification, attenuation, freshness, depth, or cycle check. The response body **MUST** carry the `reason` token and the index of the failing hop. The server **MUST** log this event. | {{delegation}} |
| 552 | Reserved | Reserved for AGTP expansion. | {{status-codes}} |
| 553 | Reserved | Reserved for AGTP expansion. | {{status-codes}} |
| 554 | Reserved | Reserved for AGTP expansion. | {{status-codes}} |
| 555 | Reserved | Reserved for AGTP expansion. | {{status-codes}} |
{: title="AGTP-Specific Status Code Definitions"}

## Media Type Registry {#iana-media-types}

This section is the master registry of all AGTP-family media
types. Types defined by companion specifications are listed here
for cross-document discoverability; their normative definitions
remain in the defining document.

| Media Type | Use | Defining Reference | IANA Status |
|---|---|---|---|
| `application/vnd.agtp+json` | AGTP method request/response bodies (JSON) | {{wire-format}} | Planned (this document) |
| `application/vnd.agtp+yaml` | AGTP method request/response bodies (YAML) | {{wire-format}} | Planned (this document) |
| `application/vnd.agtp.identity+json` | Agent Identity Document (JSON) | {{AGTP-IDDOC}} | Registered (vendor tree) |
| `application/vnd.agtp.identity+yaml` | Agent Identity Document (YAML) | {{AGTP-IDDOC}} | Registered (vendor tree) |
| `application/vnd.agtp.manifest+json` | AGTP server manifest | {{AGTP-METHODS}} | Planned (AGTP-METHODS) |
| `application/vnd.agtp.endpoint+json` | AGTP-METHODS endpoint definition | {{AGTP-METHODS}} | Planned (AGTP-METHODS) |
| `application/agtp-pricing+json` | AGTP commerce pricing manifest | {{AGTP-COMMERCE}} | Planned (AGTP-COMMERCE) |
{: title="AGTP Media Type Registrations"}

"Registered (vendor tree)" indicates that IANA registration is
complete in the `application/vnd.*` vendor tree as of the date of
this revision. "Planned (this document)" and "Planned (companion
document)" indicate types for which registration applications will
be filed concurrent with publication of the defining document.
Standards-tree promotion of the full set is anticipated at RFC
publication of the AGTP family.

## Header Field Registry

AGTP header fields are distinct from HTTP header fields and are registered
in a new IANA registry: Agent Transfer Protocol Header Fields.

Registry name:
: Agent Transfer Protocol Header Fields

Registration procedure:
: Expert Review + published specification required.

AGTP keeps its own registry rather than reusing the HTTP Field Name Registry, as AGTP header fields have
different semantics, applicability, and versioning constraints from HTTP
fields. HTTP header fields carry no automatic validity in AGTP, and AGTP
header fields are invalid as HTTP fields.

Initial registrations (all Permanent): Agent-ID, Owner-ID, Authority-Scope,
Session-ID, Task-ID, Delegation-Chain, Server-ID, Attribution-Record,
Continuation-Token, Supported-Methods, Cost-Estimate,
Attestation-Evidence, Authorization, Merchant-ID, Merchant-Manifest-Fingerprint,
Intent-Assertion, Cart-Digest, AGTP-Methods-Version,
AGTP-Catalog-Warning, AGTP-Endpoint-Warning, Allow-RCNS.

The four merchant-related headers are defined in {{AGTP-MERCHANT}} and
registered concurrently with this document. The three catalog-related
headers (`AGTP-Methods-Version`, formerly `AGTP-API-Version`,
`AGTP-Catalog-Warning`, and `AGTP-Endpoint-Warning`) are defined in
{{AGTP-METHODS}} and registered
concurrently. The `Authorization` header semantics for composition
with external identity providers are specified in {{AGTP-COMPOSITION}}.
The `Allow-RCNS` header semantics for caller opt-in to Runtime
Contract Negotiation are specified in {{AGTP-RCNS}}.

Headers reserved for future revisions (Priority, TTL, AGTP-Zone-ID,
Content-Schema, Telemetry-Export) are unregistered in this
revision. They will be registered alongside the future revision that
specifies their normative semantics.

The `Budget-Limit` header is defined in {{AGTP-COMMERCE}} and
registered concurrently with that document.

## URI Scheme Registration

The `agtp://` URI scheme is registered per {{RFC7595}}. Full
registration template is documented in {{uri-scheme-template}}.

## AGTP Budget Unit Registry

Establishment of a new IANA sub-registry: Agent Transfer Protocol Budget
Units.

Registry name:
: Agent Transfer Protocol Budget Units

Registration procedure:
: Expert Review per {{RFC8126}}. New unit registrations **MUST** define:
  unit name (lowercase ASCII, no spaces or special characters), semantic
  description, value format (integer or decimal), whether fractional values
  are permitted, and a reference specification. Units representing financial
  denominations **MUST** specify the currency and **MUST** define precision
  (decimal places). The Designated Expert **SHOULD** verify that the proposed
  unit duplicates no existing registration and that the value format
  is unambiguous.

Reference:
: This document

Initial registrations:

| Unit | Description | Value Format | Fractional |
|------|-------------|--------------|------------|
| tokens | Language model token consumption | Integer | No |
| compute-seconds | CPU/GPU compute time in seconds | Decimal | Yes |
| USD | US Dollar financial limit | Decimal, 2 places | Yes |
| EUR | Euro financial limit | Decimal, 2 places | Yes |
| GBP | Pound Sterling financial limit | Decimal, 2 places | Yes |
| calls | Downstream API call count | Integer | No |
{: title="Initial AGTP Budget Unit Registry Entries"}

## Agent Registry Retention Policy {#retention}

The AGTP registry **MUST** retain records for all registered agents
regardless of lifecycle state. The following minimum retention periods apply:

| Lifecycle State | Minimum Retention Period |
|---|---|
| Active | Duration of Active state + 7 years |
| Suspended | Duration of Suspended state + 7 years |
| Revoked | 10 years from revocation date |
| Deprecated | 7 years from deprecation date |
{: title="AGTP Registry Minimum Retention Periods"}

The 7-year minimum reflects common enterprise compliance requirements
(SOX, GDPR audit trails, HIPAA). Governance platform operators in regulated
industries **SHOULD** extend these minimums to match applicable regulatory
requirements.

The retained record for a Revoked or Deprecated agent **MUST** include:

- Canonical Agent-ID (permanently retired, never reissued)
- Agent label and org domain at time of registration
- Trust tier at time of registration
- Activation date and activating principal
- Revocation or deprecation date, initiating principal, and reason code
- Genesis audit record hash (pointer to immutable audit log)
- Full Agent Genesis (archived, inaccessible to the public)
- All lifecycle state transitions with timestamps

The retained record **MUST NOT** contain package executable contents,
active session data, or Authority-Scope grant tokens.

### Domain Name Expiry Interaction

If an organization's `org_domain` expires or transfers to a new owner:

1. All Active agents registered under the expired domain **MUST** be
   automatically Suspended within 24 hours of domain expiry detection.

2. The governance platform **MUST** notify the registered principal
   contact before suspension takes effect, with a minimum notice period
   of 30 days if domain expiry was predictable.

3. Suspended agents under an expired domain transition to Deprecated
   state after 90 days if the domain remains unrenewed.

4. A new owner of the domain **MUST NOT** inherit prior agent
   registrations. New ACTIVATE transactions are required.

This policy prevents domain-transfer-based identity hijacking in which
an attacker acquires an expired domain to claim the trust history of
agents that operated under it.

--- back

# Changes from v09

Version 10 is a structural and language revision with one new
normative mechanism.

## Substantive Changes

1. **Delegation-Chain specified.** The header reserved since v00 is
   specified ({{delegation}}): format (ordered hops of canonical
   Agent-ID and delegation token), minting at DELEGATE under the
   existing strict-subset rule, per-hop verification (membership,
   parent re-verification, attenuation, freshness, depth and cycle),
   the effective scope as the accepted result, and 551 failure
   behavior with a reason vocabulary. Terminology distinguishes the
   Delegation Sequence (the hops) from the Delegation-Chain header
   (its carrier). The composition table, the worked DELEGATE example,
   and the security considerations are aligned to the specification;
   earlier revisions described the header as if specified in those
   places while reserving it in the header table.

2. **Delegation token format and authority evidence.** The hop
   token payload is specified ({{delegation-token}}) with a digest
   of the parent token, so a parent swap fails on the digest as well
   as the key. Verification is defined over a fixed input set with
   no local policy and returns valid, invalid, or indeterminate
   ({{delegation-verification}}); an origin hop may reference an
   authority assertion for the delegator's own powers
   ({{authority-evidence}}); and a table states what each artifact in
   a delegated request asserts ({{delegation-attribution}}).

3. **Structured failure fields.** `retryable`, `partial` with
   `completed_steps`, `side_effects`, and `reason` on 4xx and 5xx
   bodies ({{failure-fields}}). Additive.

4. **Specification status vocabulary.** Specified, specified and
   implemented, planned, inherited ({{conventions}}), with the
   carrier-versus-claim rule stated. Applied through the document.

5. **Transfer protocol.** AGTP is described as a transfer protocol
   throughout; "transport" now refers only to TCP, TLS, and QUIC.
   The positioning section is rewritten around the HTTP and HTML
   analogy in place of SMTP and TCP.

6. **Revocation continuity and tamper evidence** stated as planned
   and inherited respectively ({{revocation-continuity}} and the
   following subsection), with the member-inactive check of
   {{delegation-verification}} as the first specified piece.

7. **Tool and resource identity** stated as a deliberate composition
   choice under review ({{composition-section}}).

8. **Workload and agent identity** composition sentence added to
   Identity at the Wire, and a Discovery and Naming subsection added
   to Related Work carrying the identity and location split and the
   discover versus search vocabulary.

9. **Per-request proof of possession.** The `Request-Signature`
   header ({{request-signature}}) binds every non-anonymous request
   to the agent's current signing key, published as
   `agent_public_key` in the Identity Document. Level 2 verification
   gains a fifth step and now establishes the sender rather than the
   document alone; a verifier that skips the step is at Level 1 for
   every purpose that depends on who sent the request. Answers the
   self-asserted-identity finding raised independently by four
   reviewers and conceded on list in August.

10. **262 versus 455 resolved.** Obtainability is the rule: 262 when
   the agent could obtain the missing authorization, 455 when the
   request exceeds authority the agent holds. A claimed scope beyond
   the document's grant, an endpoint scope the document fails to
   grant, and a delegation beyond the parent are all 455. Earlier
   revisions used both codes for Authority-Scope failures in different
   sections.

11. **HTTP registry argument withdrawn.** The claim that IETF Review
    freezes the HTTP method registry is removed; the case rests on the
    distinct wire image and the documented failure cases.

12. **WIMSE chain convention mapping** and a stated staleness bound
    ({{wimse-mapping}}).

13. **Governance Token action digest** binding a token to one request
    shape, with `token-action-mismatch` on divergence.

14. **`evaluation` failure field** distinguishing a refusal from an
    incomplete check, with `missing_input`.

15. **User agent and service agent** stated as both in scope, with the
    Owner-ID as what differs.

## Structural Changes

16. **AGTP-API replaced by AGTP-METHODS and AGTP-RCNS.** Every
   reference to {{AGTP-API}} is redirected: the method
   vocabulary, path grammar, endpoint primitive, manifest, and policy
   to {{AGTP-METHODS}}; negotiation, contracts, and synthesis to
   {{AGTP-RCNS}}. The `AGTP-API-Version` header registration is
   renamed `AGTP-Methods-Version`.

17. **Agent Identity Document extracted.** The Agent Namespace
   Document, the Agent Identity Document and `.agtp` format, and the
   browser interaction model move to {{AGTP-IDDOC}}; this document
   keeps a summary ({{identity-document}}) stating what the protocol
   depends on. The Agent Genesis, registration lifecycle, and
   lifecycle events remain here because the Agent-ID derivation and
   the lifecycle methods depend on them.

18. **Module summaries.** Dynamic Endpoint Negotiation, the Extended
    Method Vocabulary, Composition Profiles, and the Merchant
    Identity sections are reduced to summaries that point at the
    companion carrying the normative text. The AGMP wire example
    appendix moves to {{AGTP-COMPOSITION}}.

19. **Change history collapsed.** The v06, v07, and v08 change logs
    (about 680 lines) are replaced by {{change-history}}; the full
    text remains in the archived revisions.

20. **Glossary merged** into Terminology; four entries that appeared
    only in the glossary are carried over.

21. **DISCOVER and ACTIVATE aligned with the companions.** DISCOVER
    now distinguishes search (criteria to candidates) from discovery
    (Agent-ID to attachment) per {{AGTP-DISCOVERY}} and defers its
    parameter set and response shape there; the v09 parameter table
    (`criteria`, `filter`, `max_results`, `context`) disagreed with
    the companion and is withdrawn. ACTIVATE is named as the publish
    event: ANS registration ({{AGTP-DISCOVERY}}) and the first
    ANNOUNCE ({{AGTP-PRESENCE}}) follow from the lifecycle
    transition, so an agent is findable exactly when Active.

## Wire Format Compatibility

The `Delegation-Chain` header changes from reserved to specified. A
v09 server receiving it **MAY** have rejected it with 501; a v10
server verifies it. Requests without the header are unchanged.
`Request-Signature` is new and required on non-anonymous requests; a
v10 server **MUST** accept unsigned requests from agents whose
Identity Document predates `agent_public_key` under the
possession-unproven rule of {{request-signature}}, so v09 agents keep
working at Level 1 until re-issued. The 262 to 455 change for
Authority-Scope failures is wire-visible for clients that branched on
262; clients **SHOULD** treat 455 with `scope-claim-invalid` or
`scope-exceeds-parent` as they treated 262 `scope-required`. The
structured failure fields and `action_digest` are additive. The `AGTP-Methods-Version`
rename is advisory; servers **MAY** emit both names during
transition. No other wire-visible behavior changes.

# Change History {#change-history}

Version 09 (June 2026) added the lifecycle method group to the
embedded floor, completing the eighteen-method floor; adopted
Owner-ID in place of Principal-ID, recorded in the identity document
rather than carried as a header; added the RCNS headers and status
codes 461 and 464 with their reason vocabulary; and aligned the
companion references to the v09 family. Version 08 (May 2026)
expanded the floor from twelve to eighteen methods organized as
cognitive, mechanics, and lifecycle groups; added trust posture
headers, INSPECT read access control, lifecycle method authorization,
cross-agent chain walking, signed manifests, DISCOVER path-keyed
dispatch, and SCITT-aligned lifecycle receipts; and introduced the
Runtime Contract Negotiation Substrate framing. Version 07 (May 2026)
trimmed the header model to the request-line plus Agent-ID,
Authority-Scope, Session-ID, Task-ID, and the reserved
Delegation-Chain, with Server-ID and Attribution-Record on responses;
adopted the comma-separated Authority-Scope grammar; removed the
Birth Certificate terminology in favor of Agent Genesis; extracted
trust tier detail to AGTP-TRUST; and folded AGIS and AGTP-Methods into
AGTP-METHODS. Version 06 (April 2026) recorded the IANA port 4480
assignment and the `agtp://` URI scheme registration. The complete
change logs for these revisions are in the archived drafts at
github.com/nomoticai/agtp-ietf.

# Authority-Scope Format {#authority-scope}

Authority-Scope values are expressed as a comma-separated list of
scope tokens, each following the pattern `domain:action` or
`domain:*` for full-domain access. Multi-segment scope tokens of
the form `domain:subdomain:action` are admissible for namespaced
operations (e.g., `mcp:tools:execute`). The encoding follows the
HTTP-standard list-valued header convention {{RFC9110}}: tokens
are separated by a comma, with optional surrounding whitespace
permitted for readability. Tokens **MUST** be lowercase ASCII
segments separated by colons, with no internal whitespace.

The wire-level grammar in ABNF:

~~~~
Authority-Scope = scope-token *( OWS "," OWS scope-token )
scope-token     = scope-segment 1*( ":" scope-segment )
scope-segment   = 1*( ALPHA / DIGIT / "-" / "_" ) / "*"
OWS             = *( SP / HTAB )   ; optional whitespace, per RFC 9110
~~~~

A scope token **MUST** contain at least two segments separated by a
colon. The final segment is the action; preceding segments form the
namespace path. The two-segment form `domain:action` is the
canonical case; the three-or-more-segment form is reserved for
deployments that need namespaced authority hierarchies (e.g.,
external-protocol bridging, hierarchical resource grants).

Parsers **MUST** accept any amount of optional whitespace adjacent
to the comma separator (including none) and **MUST** treat
`"calendar:book,calendar:query"` and
`"calendar:book, calendar:query"` as equivalent. Implementations
**SHOULD** emit a single space after the comma for human readability.

Examples:

~~~~
Authority-Scope: calendar:book, calendar:query
Authority-Scope: documents:summarize, documents:query, knowledge:learn
Authority-Scope: *:query
Authority-Scope: booking:*, payments:confirm
Authority-Scope: mcp:tools:execute, knowledge:query
~~~~

In the Agent Identity Document, Authority-Scope is represented as a
JSON array of scope-token strings (one token per array element) rather than
as a single comma-separated string. The wire-header encoding and the
JSON-document encoding carry the same logical set of tokens.

Reserved domains (initial set):

| Domain | Description |
|--------|-------------|
| calendar | Scheduling and time-based resource management |
| documents | Document access, summarization, and annotation |
| knowledge | Agent context and memory operations |
| booking | Reservation and resource allocation |
| payments | Financial transactions and confirmations |
| agents | Delegation and collaboration with other agents |
| escalation | Escalation routing and handler management |
| activation | Governed agent package activation (ACTIVATE method extension) |
| discovery | Agent discovery and capability query operations (DISCOVER, DESCRIBE) |
| budget | Resource budget declaration and QUOTE pre-flight operations |
| telemetry | Telemetry export and observability operations |
| zone | Network zone boundary declaration and enforcement |
| suspend | Session suspension and resumption operations |
| merchant | Merchant identity resolution and counterparty verification (see {{AGTP-MERCHANT}}) |
| intent | Intent Assertion issuance and validation (see {{AGTP-MERCHANT}}) |
| \* | All domains require explicit grant; use with caution |
{: title="Reserved Authority-Scope Domains"}

## Reserved Operator Tokens {#reserved-scope-tokens}

Two scope tokens are reserved for operators and **MUST NOT** appear in
an agent's identity-document scope set unless the agent is the
server's operator:

`inspect:all`:
: Grants INSPECT read access across every agent the server hosts, and
  authorizes the operator-fired `REVOKE target=stale-contracts` sweep
  and the cross-agent `DISCOVER /contracts` view specified in
  {{AGTP-RCNS}}.

`rcns:negotiate`:
: Opens the caller-scope lock of the RCNS dispatcher gate
  ({{AGTP-RCNS}}). Unlike `inspect:all`, this token **MAY** be granted
  to any agent the operator chooses to admit to runtime negotiation.

# Example AGTP Wire Formats {#wire-examples}

The following examples use a human-readable pseudo-wire format with
HTTP-style headers followed by a JSON body. The Content-Type for all AGTP
message bodies is application/vnd.agtp+json.

## QUERY Request and Response

~~~~
AGTP/1.0 QUERY
Agent-ID: agt-7f3a9c2d
Authority-Scope: documents:query, knowledge:query
Session-ID: sess-a1b2c3d4
Task-ID: task-0042
Content-Type: application/vnd.agtp+json

{
  "task_id": "task-0042",
  "parameters": {
    "intent": "Key arguments against MCP re: HTTP overhead",
    "scope": ["documents:research", "knowledge:session"],
    "format": "structured",
    "confidence_threshold": 0.75
  }
}

AGTP/1.0 200 OK
Task-ID: task-0042
Server-ID: srv-knowledge-01
Attribution-Record: [signed attribution token]
Content-Type: application/vnd.agtp+json

{
  "status": 200,
  "task_id": "task-0042",
  "result": {
    "results": [{"content": "...", "source": "doc-agtp-research",
                 "confidence": 0.91}],
    "result_count": 1
  }
}
~~~~

## EXECUTE Request and Response (Carried Application Payload)

The following example shows EXECUTE carrying an application-layer
booking action. The Authority-Scope on the request governs the
action; a scope mismatch would return 455.

~~~~
AGTP/1.0 EXECUTE
Agent-ID: agt-travel-planner
Authority-Scope: booking:*, calendar:book
Session-ID: sess-trip-2026-04
Task-ID: task-0107
Content-Type: application/vnd.agtp+json

{
  "method": "EXECUTE",
  "task_id": "task-0107",
  "parameters": {
    "action": "reserve-flight",
    "parameters": {
      "resource_id": "flight-AA2847",
      "principal_id": "usr-chris-hood",
      "time_slot": "2026-04-15T08:00:00Z",
      "options": {"seat_preference": "aisle", "class": "economy"}
    },
    "idempotency_key": "trip-2026-04-15-AA2847"
  }
}

AGTP/1.0 200 OK
Task-ID: task-0107
Attribution-Record: [signed attribution token]
Content-Type: application/vnd.agtp+json

{
  "status": 200,
  "task_id": "task-0107",
  "result": {
    "booking_id": "BK-2026-0107",
    "status": "confirmed",
    "resource_id": "flight-AA2847",
    "confirmation_code": "XQRT7Y"
  }
}
~~~~

## EXECUTE Carrying an MCP Tool Invocation

The following example shows EXECUTE carrying an MCP tool invocation.
The carried payload's Content-Type identifies it as MCP; the AGTP
server dispatches to its MCP handler. AGTP supplies identity,
authority, and attribution at the wire level; MCP supplies tool
semantics in the payload.

~~~~
AGTP/1.0 EXECUTE
Agent-ID: agt-7f3a9c2d
Authority-Scope: mcp:tools:execute, knowledge:query
Session-ID: sess-mcp-bridge-01
Task-ID: task-0210
Content-Type: application/vnd.mcp.tools+json

{
  "method": "EXECUTE",
  "task_id": "task-0210",
  "parameters": {
    "action": "mcp.tools.call",
    "payload_type": "application/vnd.mcp.tools+json",
    "payload": {
      "tool": "search_documents",
      "arguments": {
        "query": "AGTP composition with higher-level frameworks",
        "limit": 5
      }
    }
  }
}

AGTP/1.0 200 OK
Task-ID: task-0210
Server-ID: srv-mcp-bridge
Attribution-Record: [signed attribution token]
Content-Type: application/vnd.mcp.tools+json

{
  "status": 200,
  "task_id": "task-0210",
  "result": {
    "tool_response": {
      "matches": [
        {"doc_id": "doc-0042", "score": 0.91, "excerpt": "..."}
      ]
    }
  }
}
~~~~

## ESCALATE Request and Response

~~~~
AGTP/1.0 ESCALATE
Agent-ID: agt-procurement-03
Authority-Scope: booking:*, payments:confirm
Session-ID: sess-procurement-q2
Task-ID: task-0881
Content-Type: application/vnd.agtp+json

{
  "method": "ESCALATE",
  "task_id": "task-0881",
  "parameters": {
    "task_id": "task-0880",
    "reason": "scope_limit",
    "context": {
      "attempted_action": "EXECUTE",
      "resource": "vendor-contract-750k",
      "block_reason": "Exceeds agent authorization threshold"
    },
    "recipient": "usr-cfo",
    "deadline": "2026-03-19T09:00:00Z"
  }
}

AGTP/1.0 202 Accepted
Task-ID: task-0881
Server-ID: srv-escalation-handler
Content-Type: application/vnd.agtp+json

{
  "status": 202,
  "task_id": "task-0881",
  "result": {
    "escalation_id": "ESC-0881",
    "routed_to": "usr-cfo",
    "status": "pending_review",
    "task_paused": true,
    "estimated_review_by": "2026-03-19T09:00:00Z"
  }
}
~~~~

# Comparison Table {#comparison-section}

| Criterion | AGTP | HTTP/REST | gRPC | AGMP (MCP, A2A, ...) |
|---|---|---|---|---|
| Intent-native methods | Yes (12 Tier 1) | No | No | Partial |
| Intent semantics at protocol level | Native | None | None | Messaging layer only |
| Built-in agent identity | Yes | No | No | No |
| Authority scope enforcement | Protocol-level | None | None | Application-layer |
| Built-in attribution/audit | Yes | No | No | Varies by impl. |
| Transport flexibility | TCP/UDP/QUIC | TCP/TLS | HTTP/2 | HTTP |
| Escalation as first-class primitive | Yes | No | No | No |
| Ecosystem maturity | Proposed | Mature | Mature | Emerging |
| Governance/observability | Native | Manual/bolt-on | Manual | Limited |
| Method vocabulary | Intent-named, catalog of 425 | Resource operations, registry extensible | N/A | N/A |
| Open core / royalty-free | Yes | Yes | Yes | Yes |
| Agent Identity Document | Native (.agtp format) | None | None | None |
| Tamper-proof identity surface | Yes (hash + signature) | No | No | No |
| Browser-accessible agent identity | Yes (read-only) | No | No | No |
| URI collision prevention | Domain-anchored | N/A | N/A | N/A |
| Agent Genesis | Yes (genesis record) | No | No | No |
| Domain-expiry lifecycle handling | Specified | N/A | N/A | N/A |
| Capability discovery | Native (DESCRIBE) | None | Reflection (partial) | None |
| Resource budget enforcement | Reserved for v01+ (Budget-Limit, 456) | None | None | None |
| Execution attestation (RATS) | Optional (RFC 9334) | None | None | None |
| Observability hooks | Reserved for v01+ (Telemetry-Export) | None | None | None |
| Network zone enforcement | Reserved for v01+ (AGTP-Zone-ID, 457) | None | None | None |
| Session suspension/recovery | Native (SUSPEND method) | None | None | None |
| AGMP composition profiles | Normative body section | N/A | N/A | N/A |
{: title="AGTP Compared to Existing Approaches"}

HTTP's method registry is extensible per {{RFC9110}}; what constrains
a new HTTP method is the deployed base that must receive it. AGTP's
distinct wire image lets its method vocabulary evolve through the
catalog of {{AGTP-METHODS}} as the agent ecosystem develops, without
the backward-compatibility constraints of the HTTP method space.
