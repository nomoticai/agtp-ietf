# Agent Transfer Protocol (AGTP)

## Project Charter

This document describes the goals, scope, and design principles of the
Agent Transfer Protocol (AGTP), and outlines the proposed working group
structure for advancing AGTP through the IETF standards process. It is
intended for newcomers, prospective contributors, and reviewers seeking
to understand the work before engaging with the specifications.

AGTP is currently maintained as a family of individual Internet-Drafts.
This charter describes the trajectory toward IETF working group
formation, with AGTP positioned as the foundation protocol and several
coordinated working groups addressing specific concerns built on it.

## The Problem

Three distinct problems with agent infrastructure have emerged in
parallel. Each is significant on its own. Together they describe a
substrate that is structurally insufficient for the agent web.

### The Identification Problem

Today, an enterprise cannot reliably distinguish agent traffic from
human traffic from generic API traffic. The substrate that carries
all three (HTTP) is actor-agnostic by design. There is no protocol-
level fact that says "this request is an agent" because HTTP was
never designed to carry that fact.

The consequence is shadow AI: agentic workflows running inside
enterprise environments without governance, attribution, or reliable
detection. Employees use AI agents through corporate systems and the
network can't see it. Vendors deploy AI agents into customer
environments and the customer can't see it. Internal teams build
agent automation and the security team can't see it.

The current approach is sniffing, monitoring, and pattern-based
blocking at the application layer. Detect HTTP requests that look
like they're from an LLM. Block traffic that matches known agent
signatures. Inspect payloads for AI-style patterns. This approach is
expensive, error-prone, and structurally cannot give a clean answer
because the protocol layer doesn't carry the information needed to
answer cleanly.

The structural answer is a dedicated protocol where agent traffic is
identified by the protocol itself. Infrastructure knows it's carrying
agent traffic because the protocol says so. Detection becomes a
protocol-level fact, not an inference.

### The Governance Problem

The regulatory environment for AI is expanding rapidly. The EU AI
Act requires automatic recording of events for high-risk AI systems
with cryptographically verifiable provenance. Similar frameworks are
emerging in the US, UK, and other jurisdictions. Industry-specific
regulations (financial services, healthcare, aviation, government)
add their own requirements for agent attribution, audit trails, and
authority verification.

These regulatory demands assume that agent operations are
identifiable, attributable, and auditable at the infrastructure
level. Current infrastructure cannot reliably provide that. When the
substrate is actor-agnostic, every governance framework has to
retrofit identification, attribution, and audit at the application
layer. Implementations vary. Audit trails are inconsistent.
Attribution is best-effort. Regulators are increasingly being asked
to accept "trust us, we logged it" as compliance evidence when the
underlying infrastructure cannot structurally guarantee what was
logged is what actually happened.

The structural answer is a protocol that carries identity, authority,
and attribution as wire-level facts on every request. Compliance
becomes a structural property of the protocol rather than an
application-layer assertion. Auditability becomes verifiable rather
than assumed.

### The API Accuracy Problem

APIs were designed for human-driven applications calling them through
software written by humans. The HTTP verbs that carry those calls —
GET, POST, PUT, DELETE — describe operations on resources at a low
level. Application code translates user intent into the right
combination of verbs and paths.

When an AI agent calls an API, the translation breaks down. Agents
reason about intent, not about resources. An agent does not naturally
think "POST to /bookings with this payload." It thinks "book this
flight." The semantic gap between agent intent and HTTP verbs has to
be bridged somewhere, and today it is bridged inconsistently inside
every agent framework, every tool integration, every prompt template.

Empirical research into this gap shows that semantically rich,
intent-aligned method names — verbs like BOOK, QUERY, SUMMARIZE,
DELEGATE — produce higher endpoint selection accuracy when consumed
by LLM-based agents than conventional CRUD verbs do. The performance
advantage is statistically significant at frontier model scale.
Method names that match agent intent reduce the translation burden
that current infrastructure pushes onto application code and prompt
engineering.

More concerning than the accuracy gap is the confidence pattern:
agents using CRUD APIs frequently produce incorrect endpoint
selections with high confidence. The model commits to the wrong
operation, then proceeds as if it were correct. The translation
failure does not surface as uncertainty; it surfaces as confidently
incorrect actions taken against production systems.

Getting new methods accepted into HTTP is a slow process, contested
by the broader HTTP community whose use cases are not agent-shaped.
Even if new methods were standardized into HTTP, they would still
run on a protocol that cannot tell whether its caller is a human, a
system, or an agent. HTTP is actor-agnostic by design, and
necessarily so for its intended audience.

The structural answer is a dedicated protocol that ships intent-
aligned methods natively. Methods that match how agents reason about
what they want to do. Accuracy improves because the translation
layer disappears. Confidence calibrates because the methods carry
the semantic signal the model needs.

### The Pattern

These three problems share an underlying cause. Agent infrastructure
runs on a substrate (HTTP) that was not designed for agents. Every
agent framework, every governance platform, every API has to
retrofit agent semantics onto an actor-agnostic protocol. The
retrofits work in narrow contexts and fail at scale.

The architectural answer is a dedicated protocol for agent traffic.
Identification, governance, and API semantics all become protocol-
level concerns rather than application-layer retrofits. AGTP is that
protocol.

## What AGTP Is

AGTP is the protocol of the agent web.

HTTP is the protocol of the human web. It serves people interacting
with websites, web applications, and APIs designed for human-initiated
requests. It is actor-agnostic and necessarily so, because most of its
traffic is not agent traffic and does not need agent semantics.

AGTP is the parallel protocol for agents. It serves agents calling
APIs designed for agent consumption, and agents communicating with
each other. Where HTTP is actor-agnostic, AGTP is agent-only by
definition. AGTP requires agent credentials at the wire level. Traffic
on AGTP is, structurally, an agent acting under verifiable authority.
There is no ambiguity about what a request on AGTP is.

This separation is the architectural foundation everything else in
AGTP builds on. Because the protocol knows its traffic is agent
traffic, identity, authority, and attribution can be wire-level facts
rather than application-layer assertions. Every service exposed on
AGTP knows every caller is an agent. Every decision engine can act on
protocol-level identity without first having to determine whether the
caller has identity at all.

AGTP runs over TCP with TLS 1.3 or over QUIC on the IANA-assigned port
4480. It is not built on HTTP. APIs designed for agent consumption
live on AGTP rather than on HTTP, exposed with intent-aligned semantic
methods rather than HTTP verbs adapted to agent use.

## AGTP Is a Transport Protocol

AGTP is the substrate that carries agent traffic. It is not a
governance framework, a decision engine, or a policy system. Like
SMTP carries email without deciding what's spam or who can send mail,
AGTP carries agent traffic with rich wire-level facts that decision
layers act on.

The protocol carries:

- Agent identity primitives, including the canonical Agent-ID and
  the Agent Genesis origin record from which it derives
- Authority-Scope, the binding-layer vocabulary by which agents
  declare what actions they are permitted to take
- Attribution records signed structurally on every method invocation
- Trust attestation values with normative freshness semantics
- Intent-aligned methods, with extensibility for domain-specific
  methods
- Status codes for protocol-level enforcement failures
- Delegation chain semantics with strict-subset scope enforcement
- Verification paths for identity (DNS-anchored, log-anchored, hybrid)

The protocol does not decide what scope to grant, what trust threshold
matters for a given operation, what constitutes acceptable agent
behavior, or what policies apply to a given deployment. Those
decisions live in decision engines, governance platforms, compliance
systems, and policy frameworks that compose on top of AGTP. Multiple
decision engines will compete in that layer. The protocol provides
the substrate; the application layer makes the decisions.

## Scope Boundaries

AGTP does not specify policy. It specifies the protocol over which
policy is carried and enforced. Governance platforms decide what scope
to grant and to whom; AGTP carries those decisions on the wire and
enforces them at the protocol level.

AGTP is not a messaging framework. It does not define queues, topics,
brokers, persistence semantics, or pub/sub patterns. It is a protocol
for agent-initiated requests and the responses to them, carrying the
governance metadata those requests need.

AGTP is not an agent orchestration or workflow framework. Multi-step
reasoning, tool composition, and agent collaboration patterns belong
to application-layer frameworks that compose on top of AGTP.

AGTP is not a capability specification. What an agent can functionally
do is a matter for application-layer protocols. AGTP specifies how
that capability is identified, authorized, and attributed when invoked.

AGTP does not replace TLS, QUIC, or other underlying transports. It
runs on these substrates and contributes nothing to their
specification.

## Design Principles

*Intent-aligned methods.* Methods are named for what the agent is
trying to accomplish, not for which CRUD operation maps closest. The
semantic gap between agent intent and protocol verb closes at the
protocol level rather than in application code.

*Agent-only by protocol.* AGTP traffic is agent traffic. The protocol
does not need to determine the actor; the actor is structural.

*Agent-first identity.* The canonical Agent-ID is the authoritative
identifier in every protocol operation. All other identification forms
resolve to a canonical Agent-ID. Identity is permanent and stable
across organizational change, domain transfers, and resolution-path
changes.

*Transport, not decision.* AGTP carries wire-level facts (identity,
authority, attribution, trust attestation, intent). Decision engines,
governance platforms, and policy systems make policy decisions using
those facts. The protocol enforces what the wire-level facts already
establish; it does not itself decide what those facts should mean
operationally.

*Composability with adjacent work.* AGTP is designed to serve as
substrate for application-layer agent frameworks. MCP, AGNTCY tool
frameworks, and similar application-level work compose on top of AGTP
and gain protocol-level identity, authority, and attribution by
running on it.

*Scope-bound, not capability-bound.* Authority-Scope declares what an
agent is permitted to do, not what it is functionally capable of
doing. Capability discovery is application-layer; permission
enforcement is protocol-layer.

*Attribution by default.* Every method invocation produces a signed
attribution record. There is no protocol mode in which agent actions
are unattributable.

## Empirical Foundation

The architectural commitment to intent-aligned methods is grounded in
benchmarking research conducted using the Agentic API Test Lab. The
research compared LLM-based agent endpoint selection accuracy across
four conditions: pure CRUD/REST endpoints, pure agentic-named
endpoints, mixed-paradigm endpoints, and description-mismatch
ablations.

Findings, confirmed across multiple frontier-class models:

- Agentic naming produces a substantial accuracy advantage in mixed-
  paradigm conditions, statistically significant at conventional
  thresholds.
- The effect is independent of documentation quality. Description-swap
  ablations show CRUD endpoints collapse under documentation
  mismatch while agentic endpoints remain resilient. The method name
  itself carries the intent signal.
- The effect appears to be capability-threshold dependent: it is
  absent at small model scales (around 3 billion parameters) and
  present at frontier scale.

These findings inform the AGTP design choice to ship intent-aligned
methods natively rather than retrofitting them into HTTP or treating
them as application-layer conventions. Methodology, results, and
replication materials are available in this repository.

## Working Group Structure

AGTP work is sized for multiple coordinated working groups rather than
a single broad group. This pattern follows how IETF has historically
organized large protocol families. HTTP-related work is distributed
across HTTPBIS, HTTPAPI, OAUTH, QUIC, MOQ, and others. SMTP-related
work is distributed across SMTP, EAI, IMAP, JMAP, MARC, and others.
No single WG owns the entire ecosystem; multiple WGs collaborate
around a shared substrate.

AGTP follows the same pattern. The Foundation WG provides the
substrate; six additional working groups work in parallel on specific
concerns built on top of it. Each WG is sized for focused work with
appropriate participants, with cross-WG coordination on shared
architectural questions.

The proposed working groups:

1. **AGTP Foundation** — the protocol substrate (base specification,
   transport binding, methods, status codes, IANA registrations)
2. **AGTP Identity** — agent identity primitives, naming, discovery,
   registry, certificates
3. **AGTP Security** — authorization, validation, trust attestation,
   delegation integrity
4. **AGTP Communication** — session protocols, media types, real-time
   exchange, conversational patterns
5. **AGTP API** — method catalogs, endpoint contracts, runtime
   contract negotiation, agentic API design
6. **AGTP Logging and Audit** — transparency logging, audit records,
   pre-commitment intent declaration
7. **AGTP Commerce** — merchant identity, purchase flows, budget
   authorization

Each working group is described below.

### 1. AGTP Foundation WG

The Foundation WG defines the protocol substrate for agent-to-agent
and agent-to-API communication. The substrate is what makes the rest
of the work possible: traffic on AGTP is, structurally, an agent
acting under verifiable authority.

Deliverables include the base protocol specification (wire format,
methods, status codes, identity headers), transport binding (TCP/TLS
1.3 and QUIC), and the IANA registrations that anchor AGTP in the
internet identification layer.

The Foundation WG also defines how application-layer agent frameworks
(MCP, AGNTCY tool frameworks, and others) compose on top of AGTP.
These frameworks operate at a different layer; AGTP provides the
protocol substrate they can run on to gain wire-level identity and
governance support.

Input documents: draft-hood-independent-agtp.

### 2. AGTP Identity WG

The Identity WG addresses agent identity primitives, naming and
discovery, registry infrastructure, and certificate-based identity
binding. Agent identity is distinct from user identity (handled in
OAuth and WIMSE) and service identity (handled in WIMSE), and
requires its own work.

Deliverables include identity primitives and identifier formats,
discovery and naming protocols, registry resolution mechanisms, X.509
extensions for agent certificates, and coordination with WIMSE on
identity boundary questions.

Input documents: draft-hood-agtp-agent-cert, draft-hood-agtp-discovery.

### 3. AGTP Security WG

The Security WG addresses authorization, validation, and trust
attestation for agent operations. Distinct from OAuth (which handles
user-to-service authorization), agent security covers protocol-level
authority scoping, attribution integrity, delegation chain
validation, and runtime trust attestation.

Deliverables include Authority-Scope vocabulary and enforcement,
attribution record formats and validation, delegation chain integrity
protocols, trust attestation semantics and freshness requirements,
and coordination with OAuth on user-to-agent boundary questions.

Input documents: draft-hood-agtp-trust.

### 4. AGTP Communication WG

The Communication WG addresses session semantics, media type handling,
real-time exchange (voice, video), conversational state, and multi-
modal communication patterns between agents and between agents and
tools.

Deliverables include session establishment and lifecycle protocols,
bounded versus persistent session semantics, real-time media handling
for agent-to-agent voice and video, multi-modal exchange patterns,
and coordination with MOQ, WebTransport, and QUIC working groups.

Input documents: draft-hood-agtp-session.

### 5. AGTP API WG

The API WG addresses API design for agent-consumable services,
including method vocabularies, endpoint contract specification, schema
validation, and runtime contract negotiation.

Deliverables include the method catalog and intent-aligned verb
vocabulary, the endpoint primitive with semantic contract
specification, API description language (grammar-constrained,
machine-validatable), runtime contract negotiation protocols (PROPOSE
and synthesis semantics), and status codes for API-level enforcement.

Input documents: draft-hood-agtp-api, draft-hood-independent-agis,
empirical research from the Agentic API Test Lab.

### 6. AGTP Logging and Audit WG

The Logging and Audit WG addresses transparency logging, audit
records, and attribution infrastructure for agent operations. Two
distinct subscopes operate within this WG's scope, with different
threat models and architectural requirements.

*Post-hoc audit logging.* Recording what happened after an action
executes: identity events, lifecycle transitions, delegation chains,
attribution records. Built on transparency log mechanisms aligned
with RFC 9162 and RFC 9943 (SCITT).

*Pre-commitment intent declaration.* Recording what an agent declares
it is about to do, committed to a tamper-evident log before the
action evaluates and executes. The ordering guarantee (declaration
committed < policy evaluation < execution) prevents ex-post
fabrication of intent and addresses regulatory requirements (EU AI
Act Article 12 and similar) that auditable records faithfully reflect
agent behavior.

Deliverables include the agent transparency log protocol, pre-
commitment intent declaration protocol with ordering guarantees,
integration with RFC 9162 and RFC 9943, lifecycle event formats, and
audit record requirements for regulated industries.

Input documents: draft-hood-agtp-log.

### 7. AGTP Commerce WG

The Commerce WG addresses agentic commerce infrastructure, including
merchant identity verification, purchase flows, budget controls, and
payment authorization for agent-initiated transactions.

Deliverables include merchant identity verification for agent
transactions, purchase flow protocols (PURCHASE method semantics,
budget binding), spend authorization and budget enforcement, and
coordination with payments industry standards bodies (ISO/TC 68).

Input documents: draft-hood-agtp-merchant-identity.

## Phasing

Chartering seven working groups simultaneously is neither realistic
nor desirable. Phased adoption matches both the IETF process and the
maturity of the underlying work.

**Phase 1 (Foundation).** AGTP Foundation WG and AGTP Identity WG
charter first. These are tightly coupled: identity primitives depend
on the substrate, and the substrate's defining property is agent
identity at the wire level. Existing draft work in both areas is most
mature.

**Phase 2 (Adjacent Work).** AGTP Communication WG and AGTP Security
WG charter once the foundation work is underway.

**Phase 3 (Specialized Areas).** AGTP API, AGTP Logging and Audit, and
AGTP Commerce WGs charter as community interest and input documents
mature.

This phasing is suggested sequencing, not prescriptive order. Actual
charter formation follows community interest and the standard IETF
process.

## Cross-WG Coordination

Several architectural questions cross WG boundaries and require
explicit coordination:

*Intent declaration.* Pre-commitment intent declaration (Logging and
Audit WG) interacts with authorization (Security WG) and identity
verification (Identity WG). The Foundation WG surfaces this as a
cross-cutting coordination requirement.

*Trust attestation.* Trust scores carried at the wire level
(Foundation) are computed by frameworks (Security) and may be logged
(Logging and Audit). Coordination on trust semantics and freshness
spans multiple WGs.

*Method extensibility.* The method catalog (API WG) extends the
protocol-floor methods defined in Foundation. Coordination on
extension mechanisms and registry governance crosses both WGs.

*Session lifecycle.* Session semantics (Communication WG) interact
with delegation chains (Security WG) and identity verification
(Identity WG). Coordination ensures consistency in how session-bound
authority is established and revoked.

Cross-WG coordination happens through standard IETF mechanisms:
cross-WG liaisons appointed by chairs, joint meetings to address
layering and dependencies, and shared use of common substrate
primitives defined by the Foundation WG.

## Global Out of Scope

The following items are explicitly out of scope for all AGTP working
groups:

- AI model internals (training, architecture, weights)
- Agent reasoning algorithms, planning semantics, or decision-making
  logic
- Hallucination mitigation at the model layer
- Non-protocol agent behavior (heuristics, runtime tuning, prompt
  engineering)
- AI safety policy or governance frameworks beyond protocol-level
  primitives that support them

AGTP addresses protocol-level concerns for agent infrastructure.
What models do internally, and how agent operators configure or
train their agents, are matters for other venues.

## Relationship to Adjacent Work

AGTP coexists with several related efforts. Some compose with AGTP;
others represent alternative architectural approaches at the same
layer.

*Application-layer agent frameworks (MCP, AGNTCY tool frameworks):*
These define how agents work with tools and data. They compose
cleanly on top of AGTP. Running on AGTP, they gain protocol-level
identity, authority scope, and attribution natively, without
retrofitting these onto HTTP.

*HTTP-based agent communication protocols (A2A, ACP, ANP):* These
represent an alternative architectural approach to agent
communication, layering agent semantics on HTTP rather than running
on a dedicated agent substrate. Both AGTP and HTTP-based approaches
will exist; the choice is an architectural decision each service
operator makes.

*Transport substrates (HTTP, QUIC, MOQT, WebTransport):* AGTP runs on
TCP/TLS or QUIC. The architectural commitment of AGTP is the
dedicated agent protocol layer above the transport.

*Identity and authentication frameworks (WIMSE, OAuth, SPIFFE):*
These provide proven mechanisms for workload and service identity.
AGTP's identity model is agent-native rather than workload-native,
and the two serve different scopes.

*Transparency standards (RFC 9162 Certificate Transparency 2.0, RFC
9943 SCITT):* AGTP's log-anchored verification path interoperates
with deployed SCITT infrastructure rather than inventing a parallel
format.

*Service discovery (APIX and similar):* AGTP-aware services can be
advertised through general-purpose discovery layers that declare
transport as a service property. AGTP's own discovery work (AGTP
Identity WG) addresses agent-native discovery for AGTP traffic
specifically.

*Higher-layer execution governance (TEE-based attestation, hardware-
rooted execution provenance):* These compose above AGTP, consuming
the wire-level facts AGTP carries to make execution-time decisions
with cryptographic provenance.

## Status

AGTP is a family of individual Internet-Drafts. The base specification
is draft-hood-independent-agtp, currently at version 07. Companion
drafts cover session protocol, API contract layer, trust model,
discovery, transparency logging, merchant identity, certificate
extension, and the Agentic Grammar and Interface Specification
(AGIS).

IANA registrations are completed for port 4480 (TCP and UDP) and the
agtp:// URI scheme. Media type registrations are in expert review.

Source documents, reference implementation, and specification
discussion live in this repository. Drafts are submitted to the IETF
datatracker. Implementation work includes a working reference
implementation with PHP, Symfony, Go, and Drupal bindings, plus an
MCP-on-AGTP connector demonstrating composability with application-
layer frameworks.

The project is currently maintained by Chris Hood (chris@nomotic.ai).
Contributors are welcome and credited; see Contributing below.

## How to Contribute

Three forms of contribution are valuable:

*Issues.* Open a GitHub issue for specification questions, perceived
inconsistencies, threat model gaps, missing edge cases, or areas
where the drafts are unclear. Issues are the lightest-weight
contribution and the most useful for sharpening the specifications.

*Pull requests.* For text edits, schema corrections, or example
contributions, open a pull request against the relevant draft.
Significant architectural changes should be discussed in an issue
first.

*Implementation experience.* If you implement AGTP, partially or
fully, report what you found. Implementation feedback is the most
valuable input a protocol specification receives.

To express interest in participating in a proposed working group,
open a pull request or issue against this repository indicating which
WG(s) you'd like to contribute to and what aspect of the scope you're
most interested in. IETF working groups are formally open to anyone
who joins the relevant mailing list once chartered; this repository
exists to coordinate community interest before chartering happens.

For substantive contributions or proposals to add companion drafts,
contact chris@nomotic.ai directly.

## Governance and IPR

AGTP is published under standard IETF Internet-Draft terms. Patent
claims arising from the specifications are disclosed per BCP 79.
Implementers are advised that royalty-free licensing terms are
intended for any patents covering AGTP mechanisms; specific
commitments are documented in the IPR notices of individual drafts.

Specification-level discussion happens on this GitHub repository.
IETF list discussion happens on the lists where drafts are
introduced.

## Trajectory

The near-term trajectory is to mature the specification family
through implementation feedback and community review. A BoF request
for AGTP work is targeted for IETF 127 (San Francisco, November
2026), conditional on sufficient implementation evidence and
community participation.

The architectural commitment of AGTP is independent of its
standardization venue. The protocol is designed to be useful
regardless of whether it becomes a chartered IETF work item.

## Contact

Chris Hood, Nomotic, Inc.
chris@nomotic.ai

Drafts: https://datatracker.ietf.org/doc/draft-hood-independent-agtp/
Document Repository: https://github.com/nomoticai/agtp-ietf
Code Repository: https://github.com/nomoticai/agtp-ietf