---
title: "AGTP Agent Discovery and Name Service"
abbrev: "AGTP-DISCOVER"
docname: draft-hood-agtp-discovery-02
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent discovery
  - agent name service
  - ANS
  - AGTP discovery

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
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026

informative:
  DAWN-INFRA:
    title: "Infrastructure-Assisted Discovery of AI Agents"
    author:
      fullname: Linda Dunbar
    seriesinfo:
      Internet-Draft: draft-dunbar-dawn-infra-assisted-discovery
    date: 2026
  AGTP-PRESENCE:
    title: "AGTP Presence: Ambient Discovery and Visibility for Agent Substrates"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-presence-01
    date: 2026
  RFC6763:
  AGTP-IDDOC:
    title: "AGTP Agent Identity Document"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identity-document-00
    date: 2026
  AGTP-TRUST:
    title: "AGTP Trust and Verification Specification"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-trust-03
    date: 2026

--- abstract

The Agent Transfer Protocol (AGTP) enables agents to communicate once
they know each other's canonical identifiers. This document specifies
two layered mechanisms by which agents find each other and by which
human-readable names resolve to Canonical Agent-IDs: the DISCOVER
method, which queries for agents matching a capability description,
and the Agent Name Service (ANS), which provides governed
name-to-Agent-ID resolution.

The architectural relationship is layered. AGTP-Presence
{{AGTP-PRESENCE}} provides ambient awareness of agents within
visibility scopes as a substrate property. DISCOVER operates over
the Presence substrate to query the live population. ANS provides
human-readable name resolution, analogous to DNS for the web, with
name authorities federating across organizational boundaries. This
document also introduces Anticipatory Discovery Services (ADS) as a
forward-looking composition pattern where application-layer services
preload agent awareness based on observed operational signals.

DISCOVER and ANS together provide the application-layer interfaces
for discovery and naming that compose with the substrate-level
ambient discovery defined in {{AGTP-PRESENCE}}.

--- middle

# Introduction

## The Discovery Layers

A deployed AGTP ecosystem contains agents registered across many
organizations, governance zones, and trust tiers. An orchestrator
agent that needs to delegate a task to a capable peer faces three
distinct but related problems:

1. **Population awareness**: which agents are currently active and
   visible within my scope of operations?
2. **Capability matching**: among those visible agents, which can
   provide the capability I need?
3. **Name resolution**: given a human-readable name (such as
   `concierge.acme.com`), what is its Canonical Agent-ID?

These three problems are architecturally distinct and benefit from
distinct mechanisms. Conflating them produces designs that are
either too narrow (a single query interface that handles all three
poorly) or too broad (a single registry that tries to be all things
to all callers). This document specifies the application-layer
interfaces for capability matching and name resolution, while
deferring to {{AGTP-PRESENCE}} for population awareness.

## Search, Discover, Resolve {#vocabulary}

Earlier revisions of this document and of {{AGTP-PRESENCE}} used
"discovery" for all three problems. This revision adopts the
distinction {{RFC6763}} draws for DNS-based service discovery and
applies it to agents, because the three operations have different
inputs, different outputs, and different trust properties:

Search:
: The caller has a need and no identity. Input: capability criteria,
  an intent, trust and governance constraints. Output: a ranked list
  of candidate agents the caller had no prior knowledge of. Search is
  what the DISCOVER method does when invoked with `intent` or
  `capability_domains`. Its results are candidates to be evaluated,
  never agents to be trusted on the strength of the result.

Discover:
: The caller has an identity and no current location. Input: a
  Canonical Agent-ID. Output: the agent's current attachment, the
  signed, short-lived record of where it can be reached, and its
  Identity Document ({{AGTP-IDDOC}}). Discover is what the presence
  substrate of {{AGTP-PRESENCE}} provides ambiently and what the
  DISCOVER method does when invoked against a known Agent-ID. Its
  result is bound to the identity by signature.

Resolve:
: The caller has a name and no identity. Input: a human-readable
  name under a naming authority. Output: the Canonical Agent-ID the
  authority has bound to it. Resolve is what the Agent Name Service
  does. Its result is bound to the name by the authority's signature
  and is only as trustworthy as the authority.

The method is named DISCOVER for continuity with the registered
AGTP method vocabulary and because its registry-resolved and
presence-resolved modes are discovery in this sense; its
intent-driven mode is search, and the text says so where it
matters. An agent name and an agent address are aliases that
resolve or discover to an Agent-ID; the Agent-ID is the identity,
and nothing in this document treats a name or an address as one.

## The Layered Architecture

The discovery architecture for AGTP is layered:

- **AGTP-Presence** {{AGTP-PRESENCE}} provides population awareness
  as a substrate-level property. Agents joining the AGTP network
  become ambiently aware of other agents within their visibility
  scope without requiring queries against a central directory.

- **The DISCOVER method** (specified in this document) provides the
  search interface. DISCOVER operates over the Presence substrate,
  filtering the ambient population by capability, trust posture, and
  governance constraints, and returns ranked candidates.

- **Agent Name Service (ANS)** (specified in this document) provides
  governed name-to-Agent-ID resolution, analogous to DNS for the web.
  ANS resolves human-readable names to Canonical Agent-IDs through
  federated name authorities. Resolution is distinct from search: a
  resolved name yields one identity bound by the authority's
  signature, where a search yields candidates.

- **Anticipatory Discovery Services (ADS)** (introduced as a
  forward-looking composition pattern in this document) provide
  application-layer services that observe agent operational signals
  and preload awareness of agents the observed agent is likely to
  need before explicit queries are issued.

This document leaves population awareness undefined; that is the
responsibility of {{AGTP-PRESENCE}}. This document specifies the
application-layer query and naming interfaces that compose with
that substrate.

## Why Separate Search and Discovery From Naming

The internet has DNS for naming and presence systems (XMPP, Matrix)
for population awareness as separate layers, because they answer
different questions. DNS answers "what is the address for this
name?" Presence systems answer "who is here right now?" These are
genuinely different functions with different governance models,
different update cadences, and different consumer use cases.

AGTP follows the same pattern. ANS is the naming layer, providing
stable name-to-identifier mappings with the governance and federation
properties that naming systems require. Presence is the population
layer, providing ambient awareness with the convergence and visibility
properties that population systems require. Conflating them, as some
agent infrastructure proposals do, produces architectures that fit
neither use case well.

## Design Principles

**Agent-native.** Search, discovery, and naming results return
Canonical Agent-IDs and Agent Identity Documents rather than web
pages or proprietary records. The result of a DISCOVER query is
structurally identical to the result of resolving a known agent URI.

**Governed.** ANS servers and DISCOVER endpoints are AGTP-aware
infrastructure. They enforce trust tier requirements, behavioral
trust score floors, and governance zone constraints before returning
any result. ANS servers act as Scope-Enforcement Points for naming
queries; DISCOVER endpoints act as Scope-Enforcement Points for
capability queries.

**Ranked.** DISCOVER results are ranked by behavioral trust score,
trust tier, and declared capability depth. The highest-quality
agents matching the query appear first.

**Brokered.** DISCOVER endpoints **MAY** negotiate authority scope
on behalf of the requesting agent, informing it of what scope is
required to interact with each discovered agent before commitment.

**Layered.** This document leaves population awareness to
{{AGTP-PRESENCE}}. The DISCOVER method and ANS compose with Presence
rather than replicating its function.

# Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when,
and only when, they appear in all capitals.

DISCOVER:
: An AGTP floor method that searches for agents matching a capability
  description, trust requirement, and governance context, or
  discovers a known Agent-ID's current attachment. Returns a ranked
  list of Agent Identity Documents. See {{vocabulary}}.

ANS (Agent Name Service):
: An AGTP-aware server implementing the DISCOVER method and maintaining
  an index of registered agents and their capabilities. Acts as a SEP
  for discovery queries.

Behavioral Trust Score:
: A numeric value in the range 0.0 to 1.0 assigned to an agent at
  packaging time by a governance platform's pre-packaging verification
  pipeline. The score reflects the degree of observed behavioral
  alignment between the agent's declared capabilities, its archetype,
  and its verified conduct during behavioral testing. A score of 1.0
  indicates full alignment; a score of 0.0 indicates disqualifying
  behavioral discrepancy. The score is embedded in the agent's `.agent`
  or `.nomo` package at issuance, covered by the package integrity hash,
  and surfaced in the Agent Identity Document. It cannot be
  agent-asserted or modified without invalidating the package. ANS
  servers **MUST** use the behavioral trust score from the verified
  Agent Identity Document when ranking DISCOVER results.

Search Query:
: A structured expression of the requesting agent's capability need,
  expressed as a natural language intent, structured capability criteria,
  or both. Earlier revisions called this a discovery query.

Agent Identity Document:
: As defined in {{AGTP-IDDOC}}. The signed document a Canonical
  Agent-ID resolves to, carrying the agent's identity, lifecycle
  state, trust posture, and capabilities; returned as a search or
  discovery result. Earlier revisions used the retired name Agent
  Manifest Document.

Attachment:
: The signed, short-lived record of where an agent can currently be
  reached, which the presence substrate of {{AGTP-PRESENCE}} resolves
  an Agent-ID to. The Agent-ID is permanent identity; the attachment
  is its current location and changes without the identity changing.

Scope Negotiation:
: The ANS's declaration of what Authority-Scope the requesting agent
  will need to interact with each discovered agent.

# The DISCOVER Method

## Method Definition

DISCOVER is a floor method of {{AGTP}} and is registered in the IANA
AGTP Method Registry. It is the protocol-level interface for agent
search and for discovery of a known agent's attachment, operating
over the AGTP-Presence substrate {{AGTP-PRESENCE}} for population
awareness and against ANS servers for governed query brokering.

Purpose: Search for agents matching a capability description and
governance constraints, or discover the current attachment of a
known Agent-ID. Returns a ranked list of Agent Identity Documents
drawn from the live presence layer (per {{AGTP-PRESENCE}}) within
the requesting agent's visibility scope. The requesting agent
**MUST** carry `discovery:query` in its `Authority-Scope` header to
invoke DISCOVER.

Category: ACQUIRE. Idempotent: Yes.

## Operational Modes

DISCOVER operates in three modes, distinguished by the endpoint it
queries:

**Direct Presence query.** The requesting agent invokes DISCOVER
against the local AGTP-Presence overlay coordinator. Results are
drawn from agents currently visible in the requesting agent's
visibility scope per the scoped overlay partitioning of
{{AGTP-PRESENCE}}. This mode requires no ANS server.

**ANS-brokered query.** The requesting agent invokes DISCOVER against
an ANS server. The ANS server combines its own indexed naming records
with current Presence overlay state to return ranked results spanning
both. ANS-brokered queries provide capability-aware ranking, scope
negotiation, and cross-zone federation.

**Cross-scope resolution.** The requesting agent invokes DISCOVER
with a capability or industry partition that resolves into a scope
the agent has yet to join. The cross-scope resolution mechanism in
{{AGTP-PRESENCE}} returns agent records from the target scope, which
DISCOVER then ranks and returns to the requester.

The mode is determined by the request target. Implementations
**SHOULD** prefer direct Presence queries for in-scope search and
ANS-brokered queries for cross-zone or capability searches requiring
governed brokering. A DISCOVER whose criteria name a single Agent-ID
is a discovery in the sense of {{vocabulary}} in any mode; one that
carries `intent` or `capability_domains` is a search.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| intent | **SHOULD** | Natural language description of the needed capability |
| capability\_domains | **MAY** | Structured capability criteria matching DESCRIBE domain names |
| trust\_tier\_min | **MAY** | Minimum Trust Tier to include in results (1, 2, or 3) |
| behavioral\_trust\_min | **MAY** | Minimum behavioral trust score (0.0-1.0) |
| governance\_zone | **MAY** | Restrict results to a specific governance zone |
| org\_domain | **MAY** | Restrict results to agents from a specific org domain |
| available\_only | **MAY** | If true, exclude agents whose availability signal ({{availability}}) is unavailable: lifecycle state other than Active, or at session capacity. Default false; unavailable agents are returned and marked |
| max\_cost | **MAY** | Upper bound, as a decimal string in the unit named by `cost_unit`, on the price an agent's pricing manifest declares for the capability sought. Pricing manifests and their indexing by ANS are specified in the AGTP commerce companion; a server that indexes no pricing **MUST** ignore this parameter and say so in the response |
| cost\_unit | **MAY** | Unit for `max_cost`, from the AGTP Budget Unit Registry of {{AGTP}} |
| limit | **MAY** | Maximum number of results to return (default: 10) |
| scope\_negotiate | **MAY** | If true, include required Authority-Scope for each result |
{: title="DISCOVER Parameters"}

## DISCOVER Request Example

~~~~
AGTP/1.0 DISCOVER
Agent-ID: agtp://agtp.acme.tld/agents/orchestrator
Authority-Scope: discovery:query, agents:delegate
Request-Signature: <JWS per AGTP>
Session-ID: sess-discovery-01
Task-ID: task-disc-001
Content-Type: application/vnd.agtp+json

{
  "method": "DISCOVER",
  "task_id": "task-disc-001",
  "parameters": {
    "intent": "Audit smart contracts for security vulnerabilities",
    "capability_domains": ["methods", "tools"],
    "trust_tier_min": 1,
    "behavioral_trust_min": 0.85,
    "governance_zone": "zone:external-partner",
    "limit": 5,
    "scope_negotiate": true
  }
}
~~~~

## DISCOVER Request Example: Domain-Constrained Search

The following example illustrates a targeted DISCOVER query locating
agents capable of auditing Solidity smart contracts within a specific
governance zone and trust tier threshold. The requesting orchestrator
carries `discovery:query` and `agents:delegate` scope, signaling that
it intends to act on the results by delegating work.

~~~~
AGTP/1.0 DISCOVER
Agent-ID: agtp://agtp.finco.tld/agents/orchestrator
Request-Signature: <JWS per AGTP>
Authority-Scope: discovery:query agents:delegate audit:*
Session-ID: sess-compliance-q2
Task-ID: task-disc-solidity-01
Content-Type: application/vnd.agtp+json

{
  "method": "DISCOVER",
  "task_id": "task-disc-solidity-01",
  "parameters": {
    "intent": "Audit Solidity smart contracts for reentrancy, overflow, and access control misconfiguration",
    "capability_domains": ["methods", "tools", "zones"],
    "trust_tier_min": 2,
    "behavioral_trust_min": 0.88,
    "governance_zone": "zone:finance",
    "limit": 3,
    "scope_negotiate": true
  }
}
~~~~

The ANS server returns a ranked result set. The top result carries
Trust Tier 1 with a behavioral trust score of 0.97, exposing the
`required_scope` field indicating what the orchestrator must carry
before a DELEGATE to this agent will be accepted:

~~~~json
{
  "status": 200,
  "task_id": "task-disc-solidity-01",
  "result": {
    "query_id": "qry-sc-audit-3c9f",
    "total_matches": 7,
    "returned": 3,
    "results": [
      {
        "rank": 1,
        "canonical_id": "4f7e1a2b9c3d...",
        "attachment": {
          "node": "9b21...a7d4",
          "binding": "quic",
          "endpoint": "agents.auditor.tld:4480",
          "mode": "full",
          "exp": 1793300000,
          "seq": 17
        },
        "identity_document_uri": "agtp://agtp.auditor.tld/agents/solidity-auditor",
        "available": true,
        "agent_label": "solidity-auditor",
        "org_domain": "auditor.tld",
        "trust_tier": 1,
        "behavioral_trust_score": 0.97,
        "supported_methods": ["QUERY", "ANALYZE", "VALIDATE", "REPORT"],
        "capability_match_score": 0.96,
        "governance_zone": "zone:finance",
        "required_scope": "audit:read code:analyze",
        "job_description": "Static and dynamic analysis of Solidity smart
                            contracts. Detects reentrancy, overflow, and
                            access control vulnerabilities."
      }
    ],
    "ans_signature": {
      "algorithm": "ES256",
      "key_id": "ans-key-finance-01",
      "value": "[base64-encoded-signature]"
    }
  }
}
~~~~

The requesting agent verifies the `ans_signature` before trusting any
result. It then issues a DESCRIBE to `agtp://agtp.auditor.tld/agents/
solidity-auditor` to confirm method support before committing to
DELEGATE.

## DISCOVER Response

The DISCOVER response body contains a ranked list of result entries,
each carrying the agent's Canonical Agent-ID, its current attachment
per {{AGTP-PRESENCE}} so that the result is reachable without a
further resolution step, and optional scope negotiation fields:

~~~~json
{
  "status": 200,
  "task_id": "task-disc-001",
  "result": {
    "query_id": "qry-7f3a9c",
    "total_matches": 23,
    "returned": 5,
    "results": [
      {
        "rank": 1,
        "canonical_id": "3a9f2c1d8b7e4a6f...",
        "attachment": {
          "node": "7f3a...c2e1",
          "binding": "quic",
          "endpoint": "agents.auditor.tld:4480",
          "mode": "full",
          "exp": 1793300000,
          "seq": 42
        },
        "identity_document_uri": "agtp://agtp.auditor.tld/agents/contract-auditor",
        "available": true,
        "agent_label": "contract-auditor",
        "org_domain": "auditor.tld",
        "trust_tier": 1,
        "behavioral_trust_score": 0.97,
        "supported_methods": ["QUERY", "VALIDATE", "REPORT"],
        "capability_match_score": 0.94,
        "required_scope": "audit:read code:analyze",
        "job_description": "Static and dynamic analysis of Solidity smart contracts."
      }
    ],
    "ans_signature": {
      "algorithm": "ES256",
      "key_id": "ans-key-public-01",
      "value": "[base64-encoded-signature]"
    }
  }
}
~~~~

The `ans_signature` covers the full result set including all manifests
and the `query_id`. The requesting agent **MUST** verify the signature
before trusting any discovery result. An unsigned or invalid DISCOVER
response **MUST** be rejected.

## Ranking Algorithm

ANS servers **MUST** rank results by a composite score computed as:

~~~~
rank_score = (trust_tier_weight * normalized_trust_tier)
           + (behavioral_trust_weight * behavioral_trust_score)
           + (capability_match_weight * capability_match_score)
~~~~

Default weights: trust_tier_weight = 0.3, behavioral_trust_weight = 0.4,
capability_match_weight = 0.3. ANS servers **MAY** publish alternative
weight configurations for domain-specific deployments.

The `capability_match_score` is computed by the ANS server as a
normalized relevance score between the query's `intent` and
`capability_domains` and the candidate agent's declared capabilities.
The computation method is implementation-defined but **MUST** be
deterministic for identical inputs.

# Agent Name Service {#ans}

## ANS Architecture

The Agent Name Service provides governed resolution of human-readable
names to Canonical Agent-IDs. ANS is the naming layer for AGTP,
architecturally analogous to DNS for the web. ANS leaves to Presence
the population awareness functionality provided by {{AGTP-PRESENCE}};
it provides the stable name-to-identifier mapping that complements
ambient discovery in the sense of {{vocabulary}}.

An ANS server is an AGTP endpoint that:

1. Maintains a registry of name-to-Agent-ID bindings for agents in
   its naming authority. The registry is structured for fast lookup
   on the name key, comparable to DNS zone file lookup.

2. Resolves DISCOVER queries that include name-based criteria by
   returning the bound Agent-ID and associated Agent Identity
   Document. ANS may consult the Presence overlay for current
   operational state of the resolved agent.

3. Enforces Trust Tier and behavioral trust score thresholds on all
   queries before returning results.

4. Signs all DISCOVER responses with the ANS's governance key.

5. Acts as a SEP for naming traffic: agents without `discovery:query`
   scope **MUST** be rejected with 455 Scope Violation.

ANS servers are themselves AGTP agents: they have canonical Agent-IDs,
Agent Genesis records, and Agent Identity Documents. Their capabilities
include `discovery:query` and `discovery:register` as Authority-Scope
domains.

Multiple ANS servers **MAY** be deployed for different naming
authorities, organizations, or trust tier populations. Cross-ANS
federation is described in {{cross-ans-federation}}.

## Relationship to Presence

ANS and AGTP-Presence are complementary layers serving different
discovery use cases:

| Question | Layer | Mechanism |
|----------|-------|-----------|
| Who is here right now? | Presence | Ambient overlay membership |
| What is the Agent-ID for this name? | ANS | Name-to-identifier resolution |
| Which agents can do X? | DISCOVER over Presence | Capability filter over visible population |
| Which agents named X.acme.com exist? | DISCOVER via ANS | Name-anchored capability resolution |

An agent's full discovery toolkit consists of all three: ambient
awareness through Presence, governed name resolution through ANS, and
capability-driven query through DISCOVER. These compose without
duplication.

ANS servers **MAY** consult Presence overlay state when answering
DISCOVER queries to ensure that resolved agents are currently
reachable. An ANS-resolved name pointing to an Agent-ID that is no
longer in any visible Presence overlay **SHOULD** be returned with
an availability indicator, allowing the requesting agent to decide
whether to attempt connection.

## ANS Registration

An agent registers with an ANS server through a governed process
tied to AGTP activation:

1. On completion of the AGTP ACTIVATE transaction, the governance
   platform automatically submits the agent's Agent Identity Document
   and DESCRIBE capability data to the designated ANS server(s) for
   the agent's governance zone.

2. The ANS server verifies the agent's lifecycle state is Active.

3. The ANS server indexes the agent's capabilities, trust tier, and
   behavioral trust score.

4. The ANS server updates its result index within 60 seconds.

Manual registration is unsupported. ANS registration is a consequence
of AGTP activation rather than an independent step. This ensures that only
Active, registered agents appear in discovery results.

## ANS Deregistration

When an agent's lifecycle state transitions to Suspended, Revoked, or
Deprecated, the governance platform **MUST** notify the relevant ANS
servers within 60 seconds. The ANS server **MUST** remove the agent
from its result index before the next DISCOVER request is processed.

ANS servers **MUST** treat deregistration as urgent: a Revoked agent
that continues to appear in discovery results is a governance failure.
ANS servers **MUST** log deregistration events.

## Index Freshness

ANS servers **MUST** refresh capability data for indexed agents at
a configurable interval (recommended: 24 hours). Refresh is triggered
by issuing DESCRIBE requests to each indexed agent's canonical URI.
If an agent fails to respond to a DESCRIBE request after three
consecutive attempts, the ANS server **MUST** remove it from the
index and notify the governance platform.

# Cross-ANS Federation {#cross-ans-federation}

## The Federation Problem

An agent in one naming authority may need to resolve names registered
in a different authority. Organization-scoped ANS servers lack
visibility into other organizations' naming records.

Naming federation in ANS is architecturally analogous to DNS root and
TLD hierarchy. Cross-organizational name resolution proceeds through
established federation patterns at the naming authority layer.
Population federation across organizations is handled separately by
the cross-scope resolution mechanism in {{AGTP-PRESENCE}}.

## Federation Protocol

AGTP supports a federated discovery model in which ANS servers can
query peer ANS servers on behalf of requesting agents. A federated
DISCOVER query:

1. The requesting agent sends a DISCOVER request to its local ANS server.

2. If the local ANS server's index holds insufficient results
   (fewer than `limit` results above the requested thresholds), it
   MAY forward the query to federated peer ANS servers it trusts.

3. Federated queries **MUST** carry the original requesting agent's
   Agent-ID and scope requirements. The forwarding ANS server **MUST
   NOT** expand the scope or lower the trust requirements in the
   forwarded query.

4. Peer ANS servers apply their own governance policies to the
   forwarded query and return signed result sets.

5. The local ANS server merges, re-ranks, and re-signs the combined
   result set before returning it to the requesting agent.

6. The merged result includes a `federation_path` field listing the
   ANS servers that contributed results.

## Trust in Federation

Federated ANS results **MUST** be treated with the same trust level
as the federated ANS server's own Trust Tier claim. An ANS server
**MUST** declare the trust tier of its indexed agents accurately;
misrepresenting agent trust tiers in federated results is a
governance violation.

# Dynamic Routing and Failover

## Capability-Based Routing

ANS servers **MAY** support dynamic routing: when a requesting agent's
preferred target agent is unavailable (lifecycle state Suspended, absent from
responding, or rate-limited), the ANS server **MAY** return an
equivalent agent from its index as a routing alternative.

Dynamic routing responses **MUST** indicate:
- `routing_reason`: the reason the primary agent was withheld
- `equivalent_confidence`: a score (0.0-1.0) indicating how closely
  the alternative matches the primary agent's capabilities

The requesting agent retains the right to reject the alternative.

## Agent Availability Signals {#availability}

ANS servers **SHOULD** monitor indexed agents' availability by
periodically checking the `/status` sub-resource defined in
{{AGTP-IDDOC}}. Agents with `lifecycle_state: Suspended` or
`active_session_count` at capacity **SHOULD** be marked as unavailable
in routing responses.

# Anticipatory Discovery Services {#anticipatory-discovery}

This section introduces Anticipatory Discovery Services (ADS) as a
forward-looking composition pattern. ADS is an architectural concept
specified here in outline form; a full specification is deferred to
a future document.

## The Anticipatory Pattern

Baseline discovery in AGTP combines ambient awareness within
Presence overlays, governed name resolution through ANS, and
capability-driven query through DISCOVER. An agent's effective
awareness is bounded by the scopes it has joined and the queries it
explicitly issues.

Anticipatory Discovery Services expand this awareness proactively.
An ADS observes an agent's operational signals (methods invoked,
capabilities accessed, recent peers, principal context, time-of-day
patterns) and predicts which categories of agents the observed agent
is likely to need next. ADS then preloads awareness of relevant
agents through proactive Presence overlay joining or rendezvous
lookups, before the observed agent has issued an explicit DISCOVER
query.

The pattern is analogous to anticipatory service in hospitality:
recognizing that a guest walking toward the pool will need towels
and providing them before the request is articulated. The signal
precedes the explicit ask; the response is preloaded based on the
observed pattern.

## Architectural Position

ADS is an application-layer service rather than a substrate-level mechanism.
It composes with the layers specified elsewhere in the AGTP draft
family:

- **AGTP base** provides the wire-level identity and methods
- **{{AGTP-TRUST}}** provides trust posture for filtering predictions
- **{{AGTP-PRESENCE}}** provides the population substrate that ADS
  expands awareness within
- **This document** provides the DISCOVER and ANS mechanisms that
  ADS proactively invokes on behalf of observed agents

ADS operates as a service that agents subscribe to or organizations
host. It uses AGTP primitives (DISCOVER queries, ANS resolutions,
Presence overlay joining, attribution records) but is outside
the substrate itself.

## Composable ADS Providers

This document explicitly accommodates multiple deployment models for
ADS:

**Self-hosted ADS.** Organizations operate their own ADS for their
agents. The model is trained on the organization's own data.
Operational signals never leave the org. Anticipatory benefit is
bounded by the org's own scale and pattern diversity.

**Trust-tier-scoped ADS.** ADS services operate per trust tier,
observing patterns within a tier and serving anticipatory predictions
to participating agents. Cross-organizational learning improves
predictions; privacy is managed through aggregated or differentially
private signal exchange.

**Specialized provider ADS.** Just as trust providers can specialize
(as described in {{AGTP-TRUST}}), ADS providers may specialize by
capability domain, industry vertical, or trust tier. A finance ADS
for trading agents, a consumer-services ADS for retail agents, a
healthcare ADS for clinical agents. Agents subscribe to one or more
ADS providers appropriate to their domain.

**Capability-derived ADS.** ADS scoped per capability dimension
(per the partition model in {{AGTP-PRESENCE}}). Capability-specific
prediction models operate within each capability scope, with
cross-scope coordination through standard federation patterns.

## Trust and Privacy Considerations

ADS observes operational patterns that may include sensitive
information about agent activity, principals served, capabilities
accessed, and interaction patterns. Several mitigations are
acknowledged in this outline; full specification is deferred:

- ADS subscription **MUST** be opt-in. Agents that never subscribe
  to any ADS provider operate with baseline Presence and on-demand
  ANS/DISCOVER without anticipatory preloading.
- ADS providers **MUST** scope their predictions within the
  visibility constraints of the observed agent. ADS expands
  discovery within existing access; it grants no access the
  agent would otherwise lack.
- Local-only ADS deployments preserve privacy by keeping
  operational signals within the organizational boundary.
- ADS providers that aggregate signals across organizations
  **SHOULD** apply differential privacy or aggregate-only
  observation patterns.

## Forward-Looking Composition

ADS is positioned in this document as an architectural concept worth
specifying. The agent population is growing; the operational
sophistication of agent discovery is going to grow with it. ADS
represents a natural composition pattern where application-layer
services consume AGTP primitives to provide value beyond what the
substrate or the basic query interfaces deliver on their own.

Future revisions or companion documents will specify:

- The signal vocabulary that ADS providers ingest
- The prediction interface between ADS and observed agents
- The preloading protocol for proactive Presence overlay joining
- Privacy semantics for cross-organizational signal aggregation
- Feedback mechanisms for prediction quality assessment

This document establishes the architectural slot for ADS within the
AGTP discovery layer, naming the pattern and positioning it as a
composable application-layer service alongside the credit-rating
trust providers described in {{AGTP-TRUST}}.

# Security Considerations

## Discovery Result Injection

Threat: A malicious ANS server returns forged Agent Identity Documents
pointing to attacker-controlled agents.

Mitigation: All DISCOVER responses **MUST** be signed by the ANS
server's governance key. The requesting agent **MUST** verify the
`ans_signature` field before trusting any result. The ANS server's
governance key **MUST** be resolvable via the ANS server's own Agent
Identity Document, creating a verifiable trust chain.

## Discovery Enumeration

Threat: A malicious agent uses repeated DISCOVER queries to enumerate
all agents in a governance zone, gaining knowledge of the organization's
agent architecture.

Mitigation: ANS servers **MUST** enforce rate limiting per requesting
Agent-ID. ANS servers **MAY** require `discovery:query` scope to include
agents from restricted governance zones. ANS servers **SHOULD** log
high-frequency DISCOVER queries as anomaly signals.

## Behavioral Trust Score Manipulation

Threat: An agent artificially inflates its behavioral trust score to
appear higher in discovery results.

Mitigation: Behavioral trust scores are computed by the governance
platform's pre-packaging verification pipeline and embedded in the
agent's `.agent` or `.nomo` package at issuance time. The score is
part of the signed manifest and cannot be modified without invalidating
the package integrity hash. ANS servers **MUST** use the trust score
from the verified Agent Identity Document rather than from any agent-asserted
field.

## Scope Negotiation Abuse

Threat: An ANS server inflates the `required_scope` field in discovery
results, causing requesting agents to request broader scope than
necessary.

Mitigation: Requesting agents **MUST NOT** blindly request the scope
declared in discovery results. The `required_scope` field is informational;
agents **MUST** evaluate the scope against their own authorization and
request only what is needed for the specific task. Governance platforms
**SHOULD** flag scope requests that significantly exceed the scope of
prior tasks in the same session.

# IANA Considerations

## DISCOVER Method Registration

This document requests registration of the DISCOVER method in the
IANA AGTP Method Registry established by {{AGTP}}:

| Method | Status | Reference |
|---|---|---|
| DISCOVER | Permanent | {{AGTP}}; search and discovery semantics in this document |
{: title="DISCOVER Method Registry Entry"}

## New Authority-Scope Domains

This document uses the following Authority-Scope domains defined in
{{AGTP}} Appendix A:

| Domain | Used For |
|---|---|
| discovery:query | Sending DISCOVER requests to ANS servers |
| discovery:register | ANS registration operations (governance platform only) |

No new domains are requested; these are from the existing reserved set.

--- back

# ANS Deployment Considerations

Small organizations may run no dedicated ANS servers. The AGTP
governance platform **MAY** provide built-in ANS functionality as part
of the agent registry. In this case, the governance platform's registry
endpoint serves both registration (ACTIVATE) and discovery (DISCOVER)
queries.

Large organizations with hundreds or thousands of agents **SHOULD**
deploy dedicated ANS infrastructure with appropriate indexing and caching.
The ANS index is a read-heavy workload; standard caching and replication
patterns apply.

Cross-organization federation requires bilateral trust establishment
between ANS operators. The protocol for establishing ANS federation
trust is out of scope for this document but follows the same patterns
as AGTP Trust Tier 1 verification: DNS ownership challenge and mutual
certificate exchange.

# Mapping to DAWN Discovery Records {#dawn-mapping}

The IETF DAWN working group scopes discovery of agents within a local
network, an organization, or a set of cooperating parties, over DNS
and mDNS, and leaves identity, capability catalogs, Internet-wide
search, and trust evaluation out of scope. The DISCOVER method's
search mode and the ANS federation of this document are outside that
scope; the discovery of a known agent's attachment within an
organization is inside it. This appendix states the mapping for that
case, so that an AGTP agent can be found by a DAWN-conformant
resolver and a DAWN-published agent can be reached over AGTP.

The DAWN work describes an agent with a record borrowed from the A2A
Agent Card in its initial revision: a self-asserted JSON document
served at a well-known HTTPS path and trusted through its domain.
The mapping from an AGTP Agent Identity Document to that record is
lossy in one direction and incomplete in the other:

| Agent Identity Document | DAWN record | Carried |
|---|---|---|
| `agent_id` | name or URL of the record | as an opaque identifier in the record's name; the Agent-ID is the identity, the record's URL is an alias |
| `name`, `description`, `capabilities` | the record's descriptive fields | yes |
| `methods`, `scopes_accepted` | the record's capability or skill list | as strings; the AGTP semantics are lost |
| `status` (lifecycle state) | none | lost; a DAWN record cannot say an agent is suspended |
| issuer signature, `agent_public_key` | none | lost; a DAWN record is self-asserted |
| attachment (current location) | the record's endpoint URL | yes, as a URL rather than a signed attachment |
{: title="Identity Document to DAWN Record"}

An AGTP agent that publishes a DAWN record **MUST** place its
Canonical Agent-ID in the record so that a DAWN resolver's result can
be discovered (in the sense of {{vocabulary}}) through AGTP to the
signed Identity Document. A DAWN record with no Agent-ID maps to an
Identity Document at Level 1 of {{AGTP}} only, with no issuer, and
AGTP treats it as a search result rather than a discovered identity.
The reverse mapping, from a DAWN record to an Identity Document, is
therefore never a substitute for registration.

## Infrastructure-Assisted Discovery {#infra-assisted}

{{DAWN-INFRA}} describes managed enterprise, edge, and network
deployments in which discovery is fed by two infrastructure inputs:
a Resource Publication Hook, through which an orchestration or
hosting system supplies the discovery mechanism with the agent
instances currently deployed and their endpoints, and an Attachment
Context Hook, through which the network supplies an abstract
representation of the requester's current attachment (site, VPN,
tenant, slice, edge region). The motivating failure is a discovery
system that keeps returning an instance that has been withdrawn or
is unreachable from where the requester now is. AGTP meets the same
need with one publish event and one verified identity rather than
two feeds, and the mapping is worth stating for readers coming from
that work.

The Resource Publication Hook is ACTIVATE. A successful ACTIVATE is
the publish event of {{AGTP}}: it triggers registration with the ANS
of the agent's zone ({{ans}}) and the first ANNOUNCE into the
presence substrate ({{AGTP-PRESENCE}}), and DEACTIVATE, REVOKE, and
DEPRECATE are the withdrawal events. The dynamic half of what the
hook would carry, the current endpoint, is the attachment record of
{{AGTP-PRESENCE}}: signed by the agent, sequenced, and short-lived,
so that a withdrawn or moved instance expires out of discovery on its
own and a stale attachment yields nothing an attacker can use. An
orchestrator that instantiates, moves, scales, or withdraws instances
does so through the lifecycle methods, and discovery follows without
a side channel.

The Attachment Context Hook divides into two inputs that AGTP already
binds to the requester. Administrative context (tenant, VPN,
organization, governance zone) is a property of the requester's
verified identity: the `governance_zone` and `org_domain` of the
Identity Document the requester's `Agent-ID` resolves to, under the
issuer's signature. That is why the DISCOVER governance-zone rule of
{{AGTP}} needs no network hook: the server reads the requester's
zone from the document it has already verified. Location context
(which site or edge region the requester is attached to) is the
requester's own presence: the overlays it has joined and the
attachment record it has itself announced per {{AGTP-PRESENCE}},
whose `region` partition dimension carries exactly the site and edge
granularity the use case needs. In both cases the context arrives
signed and bound to the requester rather than inferred by a
middlebox, which is the integrity, origin, and freshness protection
{{DAWN-INFRA}} asks for in its security considerations, and the
underlying network detail (UPF, DNN, slice, serving gateway) stays
with the network, since the abstract context is all the requester's
identity and attachment expose.

The use case's own boundary, that discovery returns the applicable
entities and leaves selection to the requester, is the search line
of {{vocabulary}}: DISCOVER returns ranked candidates, each with its
current attachment, and the requester chooses.

On the word itself: {{DAWN-INFRA}} uses "attachment" for the
requester's network attachment, this family uses it for the
discovered agent's. They are the same network term, where a node
attaches, applied to the two parties of a discovery. Both parties
have an attachment; AGTP publishes the target's as the attachment
record and uses the requester's as search context.

# Changes from v01

Version 02 is a vocabulary and alignment revision. The DISCOVER
method, the ANS protocol, federation, and the ADS pattern are
unchanged on the wire.

- **Search, discover, resolve** ({{vocabulary}}). The three
  operations the document already separated architecturally are now
  named distinctly, following {{RFC6763}}: search (criteria to
  candidates), discover (identity to attachment), resolve (name to
  identity). Applied through the introduction, design principles,
  terminology, and method definition. The DISCOVER method keeps its
  name; its intent-driven mode is called search where it matters.
- **Agent Identity Document** replaces the retired name Agent
  Manifest Document throughout; **Agent Genesis** replaces Birth
  Certificate. Attachment is defined as the signed current location
  an Agent-ID resolves to.
- **Owner-ID** is no longer shown as a request header in the
  examples, matching {{AGTP}}; `Request-Signature` is shown instead.
- **Results carry the attachment.** Each DISCOVER result now carries
  the agent's current attachment record per {{AGTP-PRESENCE}} in
  place of the host-anchored `manifest_uri`, so a search result is
  reachable without a further resolution step; the Identity Document
  location is `identity_document_uri`.
- **`available_only`, `max_cost`, `cost_unit`** added to the DISCOVER
  parameters, binding the availability signals and the commerce
  pricing manifests to search.
- **DAWN mapping** ({{dawn-mapping}}): the Identity Document to DAWN
  record mapping for the in-scope organizational case, stated as
  lossy, with the Agent-ID as the join; and the mapping of the
  infrastructure-assisted discovery hooks of
  draft-dunbar-dawn-infra-assisted-discovery onto ACTIVATE as the
  publish event and the requester's verified identity and own
  attachment ({{infra-assisted}}).
- References: base to draft-hood-independent-agtp-10, AGTP-CERT to
  -03, AGTP-PRESENCE to -01, AGTP-IDDOC and RFC 6763 added.
  Hardcoded section numbers replaced. Five unused references
  dropped. House style pass.

# Changes from v00

Version 01 restructures this document around the layered discovery
architecture that emerged from the publication of {{AGTP-PRESENCE}}.
The DISCOVER method definition and the substantive ANS specification
content are preserved; the architectural framing and section
organization are revised to reflect the layered model.

## Substantive Changes

The following substantive changes were made in v01:

1. **Layered architecture framing.** The introduction now establishes
   the three distinct discovery problems (population awareness,
   capability matching, name resolution) and assigns each to its
   appropriate layer ({{AGTP-PRESENCE}}, the DISCOVER method, and
   ANS respectively). The document no longer presents ANS as the
   primary discovery mechanism; ANS is now positioned as the naming
   layer that composes with Presence.

2. **DISCOVER over Presence substrate.** The DISCOVER method is now
   specified to operate over the AGTP-Presence substrate as its
   primary mode, with ANS-brokered queries and cross-scope resolution
   as additional operational modes. The method's wire format is
   unchanged.

3. **ANS reframed as name resolution.** The Agent Name Service section
   now positions ANS as a governed name-to-Agent-ID resolution layer
   analogous to DNS, complementing rather than duplicating the
   population awareness provided by AGTP-Presence. A new "Relationship
   to Presence" subsection makes the layered composition explicit.

4. **Anticipatory Discovery Services introduced.** A new section
   ({{anticipatory-discovery}}) introduces ADS as a forward-looking
   composition pattern where application-layer services preload agent
   awareness based on observed operational signals. The architectural
   concept is named and positioned; full specification is deferred to
   future work.

5. **Informative references added.** AGTP-PRESENCE and AGTP-TRUST are
   added to the informative references. Cross-references throughout
   the document point to the appropriate layer for population
   awareness, trust posture consumption, and credit-rating-analogous
   trust provider composition.

## Wire Format Compatibility

The DISCOVER method wire format is unchanged. Implementations
conforming to v00 of this document continue to interoperate with v01.
The architectural reframing affects how the document describes the
relationships between layers; it leaves unchanged the protocol behavior
that v00 implementations have shipped.
