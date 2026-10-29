---
title: "AGTP Presence: Ambient Discovery and Visibility for Agent Substrates"
abbrev: "AGTP-PRESENCE"
docname: draft-hood-agtp-presence-01
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent discovery
  - ambient discovery
  - agent presence
  - attachment
  - presence
  - distributed hash table
  - gossip protocol
  - AGTP

stand_alone: yes
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
 -
    fullname: Chris Hood
    organization: Nomotic, Inc.
    email: chris@nomotic.ai

normative:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
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
      Internet-Draft: draft-hood-agtp-trust-03
    date: 2026
  AGTP-CERT:
    title: "AGTP Agent Certificate Extension"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-agent-cert-04
    date: 2026
  AGTP-DISCOVERY:
    title: "AGTP Agent Discovery and Name Service"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-discovery-02
    date: 2026
  AGTP-IDDOC:
    title: "AGTP Agent Identity Document"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identity-document-00
    date: 2026
  RFC6763:
  RFC2119:
  RFC8174:

informative:
  KADEMLIA:
    title: "Kademlia: A Peer-to-peer Information System Based on the XOR Metric"
    author:
      - name: Petar Maymounkov
      - name: David Mazieres
    date: 2002
    target: https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf
  SKADEMLIA:
    title: "S/Kademlia: A Practicable Approach Towards Secure Key-Based Routing"
    author:
      - name: Ingmar Baumgart
      - name: Sebastian Mies
    date: 2007
  IPFS:
    title: "IPFS - Content Addressed, Versioned, P2P File System"
    target: https://ipfs.tech
  LIBP2P:
    title: "libp2p - Modular peer-to-peer networking stack"
    target: https://libp2p.io
  SERF:
    title: "Serf - Decentralized cluster membership"
    target: https://www.serf.io

--- abstract

This document specifies AGTP Presence, an ambient discovery and visibility
layer for the Agent Transfer Protocol (AGTP). Existing agent discovery
proposals are pull-based: an agent must know where to look (a catalog URL,
a registry endpoint, a DNS name) before a query can begin. AGTP Presence
inverts this model. When an agent joins the AGTP substrate, it becomes
structurally addressable and visible to the relevant scope of other agents
immediately, without requiring registration with a central directory.
In the vocabulary this family adopts from RFC 6763, Presence is the
discovery layer: it resolves a known Canonical Agent-ID to the agent's
current attachment. Search, which turns a capability need into
candidate identities, is the DISCOVER method of the discovery
companion; Presence is what search runs over.

The architecture combines three proven patterns: a Distributed Hash Table
(DHT) keyed by Canonical Agent-ID for content-addressable routing, a
gossip-based protocol for presence announcement and convergence, and
trust-tier-scoped overlay partitioning for visibility control. Per-agent
visibility is cryptographically declared in AGTP-CERT certificate
extensions and enforced structurally, with three orthogonal axes of
control: presence visibility, disclosure visibility, and audience scoping.

This document defines three new lifecycle methods (ANNOUNCE, WITHDRAW,
PROBE), specifies the DHT and gossip protocols, defines the visibility
model and its certificate extensions, addresses scaling through hybrid
participation modes and federation-of-scoped-overlays, and provides a
threat model with concrete mitigations.

--- middle

# Introduction

Every existing agent discovery proposal at the IETF, in the broader
industry, and across adjacent standards bodies treats discovery as a
pull operation against known endpoints. Agentic Resource Discovery
(ARD) publishes catalogs at well-known URIs that an agent must already
know how to reach. DNS for AI Discovery (DNS-AID) publishes SVCB records
under DNS names that an agent must already possess. The AGTP Agent Name
Service (ANS) defined in {{AGTP-DISCOVERY}} provides a richer query
interface but still operates as a query against a directory the agent
has previously located. Even decentralized systems such as BitTorrent
require a magnet link, info hash, or tracker URL as the starting point
for discovery.

For the agentic web, this is the wrong model. Agents come online
expecting to know what population they can interact with, what
capabilities are available within their governance domain, and what
peers exist within their trust tier. The infrastructure that serves
human web traffic was designed around the assumption that someone
already knows the address. Agents have no such starting point.

The deeper mismatch is that the Internet's naming and routing stack
presumes host-nature: a participant is a thing at a network location,
found by resolving a name to that location. An agent is a
coordination participant defined by identity, capability, delegation,
and trust rather than by where it happens to run. The same agent may
run on a laptop this hour and a cloud instance the next, or on no
single host at all when relay-mediated; its Canonical Agent-ID is
unchanged throughout. Two analogies anchor the design. A music
catalog makes a recording findable by every attribute it carries
without a separate publishing step for each; the recording's identity
is intrinsic to it. DHCP makes joining a network the same act as
being registered on it; there is no second step. Presence treats an
agent's intrinsic identity the same way and makes joining the
registration.

This document specifies AGTP Presence, a substrate-level layer that
delivers ambient discovery as a structural property of joining the
AGTP network. The design is informed by patterns that have been
operationally validated at scale in other domains:

- The BitTorrent Mainline DHT operates at over one million concurrent
  nodes daily {{KADEMLIA}}.
- IPFS operates a Kademlia-style DHT across hundreds of thousands of
  nodes {{IPFS}}.
- Gossip protocols deployed in production systems such as Serf and
  Consul handle ten-thousand-node clusters at low bandwidth {{SERF}}.
- Hybrid peer-to-peer architectures with supernodes and light clients
  scaled to tens of millions of concurrent participants in earlier
  systems including Skype.

AGTP Presence applies these patterns to agent infrastructure, with
cryptographically declared visibility as a substrate property rather
than an application-layer convention. The result is a layer that is
more than "AGTP plus better discovery." It is a layer where the
discovery problem dissolves because the substrate carries the
population structurally.

## The Addressing Abstraction {#addressing}

The question put to this work on the IETF lists is the right one to
answer in the document itself: over what protocol, and to what
address. Over AGTP, to a Canonical Agent-ID. The substrate resolves
the Agent-ID to the agent's current attachment ({{attachment}}) and
routes between conventionally addressed substrate nodes; the
application layer never sees an IP address, in the way a user of a
filesystem never sees a disk sector although the sector exists and
the data is on it. The address becomes a substrate concern rather
than an agent concern.

This is also why the model scales with identity rather than with
addresses. Light clients hold no routing state; relay-mediated
agents hold no routable endpoint of their own. Routable addresses
scale with the number of full nodes and relays; agents scale with
the number of identities. Ten full nodes can carry a scope of a
million ephemeral agents, because what the ten nodes hold is a
million attachments keyed by Agent-ID, which is to say a million
32-byte keys and the short signed records they point to. Show up,
get absorbed, get routed.

## Conventions and Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

This document uses terminology from {{AGTP}}, {{AGTP-IDENTIFIERS}},
{{AGTP-TRUST}}, {{AGTP-CERT}}, and {{AGTP-DISCOVERY}}. Selected
additional terms specific to this document:

Presence:
: The state of an agent being known to other agents within an
  applicable visibility scope as a result of joining the AGTP
  substrate, without requiring explicit registration with a directory.

Attachment:
: The signed, short-lived, updatable record of where an agent can
  currently be reached: the substrate node or relay that holds the
  agent's connection, the binding it is reachable over, and a validity
  window. The Canonical Agent-ID is permanent identity; the attachment
  is its current location and changes without the identity changing.
  An ANNOUNCE publishes an attachment; a PROBE returns one. See
  {{attachment}}.

Discover, Search, Resolve:
: As defined in {{AGTP-DISCOVERY}} following {{RFC6763}}. Presence
  provides discovery: a known Agent-ID to its attachment. Search
  (criteria to candidates) is the DISCOVER method of
  {{AGTP-DISCOVERY}} and runs over Presence. Resolve (name to
  Agent-ID) is the Agent Name Service. Where earlier revisions of this
  document said "discovery query" for what a light client sends to a
  relay, this revision says search query or attachment lookup as the
  case requires.

Visibility Scope:
: The set of other agents that can observe a given agent's presence.
  Determined by the agent's certificate-declared visibility posture
  and the requesting agent's trust tier, governance domain, or explicit
  group membership.

Bootstrap Peer:
: An AGTP agent or service that accepts initial connections from new
  agents joining the network and provides initial routing table state
  for DHT participation.

Light Client:
: A participation mode in which an agent maintains no DHT routing
  state but communicates with full-node peers for presence
  announcement, attachment lookup, and search.

Full Node:
: A participation mode in which an agent maintains DHT routing tables,
  participates in gossip, and serves attachment lookups and search
  queries for other agents.

Relay:
: A full-node agent that accepts presence announcements, attachment
  lookups, and search queries on behalf of light-client agents.

# Architecture Overview

AGTP Presence is built on three composable mechanisms operating over
the AGTP substrate.

## DHT Over Canonical Agent-IDs

Canonical Agent-IDs as defined in {{AGTP-IDENTIFIERS}} are 256-bit
SHA-256 hashes of Agent Genesis documents. This makes them
content-addressable identifiers structurally compatible with
distributed hash table addressing.

AGTP Presence specifies a Kademlia-style DHT {{KADEMLIA}} keyed by
Canonical Agent-ID. Full-node agents maintain k-bucket routing tables
of size k=20. Lookups proceed via XOR distance metric. Parallel
lookups (alpha=3) reduce latency and provide resistance against
eclipse attacks per the S/Kademlia hardening approach {{SKADEMLIA}}.

The DHT supports three operations:

- **FIND_NODE**: Given a target Agent-ID, returns the k closest agents
  the responding node is aware of. Used during DHT routing.
- **PROBE**: Given an Agent-ID, queries whether the agent is currently
  reachable and what its declared visibility posture is.
- **ANNOUNCE**: Publishes presence information into the DHT, replicated
  across the k closest nodes to the announcing agent's Agent-ID.

DHT operations are carried as AGTP requests on the AGTP substrate,
authenticated via the AGTP wire-level identity (`Agent-ID` and
`Request-Signature`, with Owner-ID resolved from the Identity
Document per {{AGTP-IDDOC}}, and the certificate where deployed) and
verified per {{AGTP-TRUST}}.

### The Attachment Record {#attachment}

What the DHT stores under an Agent-ID, and what ANNOUNCE publishes
and PROBE returns, is the agent's attachment: a JSON object signed
by the agent with its `agent_public_key` ({{AGTP-IDDOC}}):

~~~~
{
  "agent_id":   "<64 lowercase hex>",
  "node":       "<Agent-ID of the full node or relay holding the connection>",
  "binding":    "tcp-tls" | "quic" | "h3",
  "endpoint":   "<host:port the node accepts AGTP on, or null for relay-mediated>",
  "mode":       "full" | "light" | "relayed",
  "iat":        <seconds since epoch>,
  "exp":        <seconds since epoch; MUST be within the presence TTL>,
  "seq":        <monotonic sequence number per Agent-ID>
}
~~~~

A node **MUST** reject an attachment whose signature fails under the
`agent_public_key` of the Agent-ID's current Identity Document, whose
`seq` is no greater than the stored one, or whose `exp` has passed.
Equal Agent-IDs with higher `seq` replace; this is what lets an
agent that moves hosts republish without any coordination with the
old host. The attachment is the mutable half of the identity split:
the Agent-ID and its Genesis never change, the attachment changes
whenever the agent moves, and a verifier that holds a stale
attachment holds nothing an attacker can use, since reaching the
stale endpoint reaches a node that no longer answers for that
Agent-ID and a `Request-Signature` from the real agent still
verifies wherever the agent is.

The attachment record is published by the agent about itself, on
the ACTIVATE transition and on every move, so an orchestrator that
instantiates, moves, scales, or withdraws instances needs no
separate feed into discovery: the lifecycle methods and the
agent's own ANNOUNCE and WITHDRAW are that feed, and `exp` retires
what the orchestrator forgets to withdraw. The same record, read
for the requesting agent rather than the target, is the requester's
location context for a search ({{AGTP-DISCOVERY}}): the `region`
and other partition dimensions of the overlays a requester has
joined say where it is attached, signed by the requester, with the
underlying network detail left where the network keeps it.

## Gossip-Based Presence Convergence

In parallel with DHT-based addressing, AGTP Presence uses a
gossip-based protocol for presence announcement and state convergence
within a visibility scope. The gossip mechanism is informed by
production systems such as Serf {{SERF}} and uses the following
properties:

- Each full-node agent maintains a list of known peers within its
  applicable visibility scopes.
- At configurable intervals (default 5 seconds, operator-tunable), the
  agent selects a small random subset (default fanout 3) of known peers
  and exchanges presence state.
- Presence announcements include the announcing agent's Agent-ID, its
  declared visibility posture (from its AGTP-CERT extension), a
  timestamp, and a JWS signature over the announcement.
- Receiving agents verify the signature against the announcing agent's
  certificate before incorporating the announcement into their local
  presence state.
- Conflicting announcements for the same Agent-ID (different presence
  states, different timestamps) are resolved by selecting the most
  recent validly signed announcement.

Gossip propagation converges across a scoped population in seconds to
low tens of seconds at scopes of thousands of agents. Implementations
**MAY** tune gossip intervals and fanout for their operational
requirements. The protocol behavior is normatively specified;
numerical parameters are operator-defined.

## Scoped Overlay Partitioning

Presence in AGTP is never a global property. Visibility is partitioned
across overlapping scopes determined by the trust tiers and verification
paths defined in {{AGTP-TRUST}}, the visibility posture declared by
each agent's certificate, and the operational dimensions agents
actually use for search: capability, industry, region.

This document specifies a federation-of-scoped-overlays model. A
flat global DHT containing all agents would impose unnecessary
participation cost on every node. The scoped partitioning model
means most agents participate in views containing thousands to tens
of thousands of relevant peers within bounded scopes, with
deterministic resolution into other scopes when cross-scope reach is
needed.

This mirrors how internet routing scales: BGP operates as a
federation of independently operated systems with
controlled inter-AS routing. AGTP Presence adopts the same
architectural pattern for agent populations, with the partition
axes being meaningful to agent search rather than network
topology.

### Multi-Dimensional Partition Keys

A scope is a composite key derived from one or more partition
dimensions. The dimensions specified by this document are:

| Dimension | Source | Example Values |
|-----------|--------|----------------|
| `tier` | AGTP-TRUST trust tier | `tier:1`, `tier:2`, `tier:3` |
| `owner-domain` | Owner-ID domain | `owner-domain:acme.com` |
| `capability` | AGTP method library bindings | `capability:booking`, `capability:summarization` |
| `industry` | Industry classification | `industry:finance`, `industry:healthcare` |
| `region` | Geographic region | `region:us`, `region:eu`, `region:us-ca` |

A scope is expressed as a tuple of dimension values, for example
`{capability: marketing, region: us}` or `{tier: 1, industry: finance,
capability: settlement}`. The tuple is hashed to derive the overlay
identifier. Every agent that announces with that posture joins the
corresponding overlay, and within the overlay gossip converges to
full membership.

Within a bounded scope, total awareness is the default. Every agent
in the `{capability: marketing, region: us}` overlay knows every
other agent in that overlay by Agent-ID, with no search hop required.

An agent **MAY** join multiple overlays simultaneously. A US marketing
agent might join `{capability: marketing, region: global}` for broad
visibility, `{capability: marketing, region: us}` for the regional
view, and `{capability: marketing, industry: fintech, region: us}`
for a niche scope. Multi-membership is the mechanism by which an
agent declares which subsets of the broader population are
operationally meaningful to it. Agents stay unexposed to overlays
they have yet to join.

### Capability Derivation from the Method Library

The `capability` partition dimension is derived from the AGTP method
library rather than from agent self-declaration. An agent's capability
scope membership is determined by the methods it has bound through
its Agent Genesis and certificate rather than by free-form text claims.

This design eliminates the keyspace fragmentation that would result
from agents inventing arbitrary capability strings. The method
library is the controlled vocabulary that defines the capability
axis. An agent claiming `capability:booking` membership must have
bound the methods that constitute a booking capability in the
method library; agents that have yet to bind those methods cannot
join the overlay.

The architectural property worth noting: the filtering vocabulary
agents use to find peers and the partition vocabulary the substrate
uses to organize overlays are the same vocabulary. The same mechanism
that prevents capability misrepresentation also defines the partition
keyspace.

Industry and region dimensions use thinner registries (industry
classifications such as NAICS or ISIC, geographic hierarchies such
as ISO 3166) maintained outside this document. Implementations
**SHOULD** support extensible dimension registries to allow
operational deployments to define additional partition axes
appropriate to their use cases.

### Adaptive Partition Refinement

Scopes split when they exceed working-set thresholds and merge when
they become sparse. This adaptive refinement is what keeps individual
overlays within the operational envelope of gossip convergence as
agent populations grow.

When the agent population within a scope exceeds an
implementation-defined threshold (typically in the range of ten
thousand to one hundred thousand agents per overlay), the scope
**SHOULD** be split by adding a partition dimension. For example,
`{capability: marketing, region: us}` may split into
`{capability: marketing, region: us-east}` and
`{capability: marketing, region: us-west}` when geographic
sub-partitioning reduces population to manageable sizes. Alternatively
the same scope might split on capability into
`{capability: marketing-b2b, region: us}` and
`{capability: marketing-b2c, region: us}`.

When the population within a scope falls below a sparse-population
threshold, the scope **MAY** be merged with sibling scopes by removing
a partition dimension, reducing operational overhead.

Split and merge decisions are made by overlay coordinators (typically
operated by infrastructure providers, certificate authorities, or
governance bodies for the relevant trust tier) and propagated through
the overlay via the gossip protocol. Agents discovering that their
scope has split rejoin the appropriate sub-scope based on their
current posture. Agents discovering that their scope has merged
transition to the broader scope without manual intervention.

Adaptive partition refinement is structurally similar to DNS zone
delegation (when a zone grows too large, it delegates subdomains)
and to Kademlia k-bucket splitting (when a bucket overflows, it
splits along the next bit of the keyspace). The pattern is well
understood in distributed systems practice.

### Cross-Scope Resolution

When an agent needs to interact with an agent in a scope it has yet to
joined, cross-scope resolution provides the mechanism. The agent
has no need to acquire awareness of the entire other scope; it
needs to resolve to the specific agent.

Cross-scope resolution uses a rendezvous lookup model. Each scope's
overlay coordinator maintains a rendezvous index of agents in the
scope, keyed by Agent-ID and indexed by capability for capability-driven
queries. Cross-scope queries proceed as follows:

1. The querying agent identifies the target scope from the partition
   dimensions implied by its query (e.g., a marketing agent looking
   for science-research capability resolves into `{capability:
   science-research}` scope).
2. The querying agent's local overlay coordinator forwards the query
   to the target scope's rendezvous index.
3. The rendezvous index returns the requested agent record, including
   the agent's Agent-ID, declared visibility posture, and reachability
   information.
4. The querying agent connects to the target agent directly per its
   visibility rules.

The total lookup latency is O(log N) hops across the federation of
scoped overlays, comparable to DNS resolution latency for cross-zone
queries.

The architectural distinction is: agents have total awareness within
the scopes they have joined, and deterministic resolution into the
scopes they have yet to. This composite is what holds at billion-agent
populations. No single agent maintains awareness of the global
population; every agent has efficient access to every other agent
through the scope structure.

# Participation Modes {#participation-modes}

Not every agent participates in AGTP Presence the same way. The
specification defines three participation modes, allowing
resource-constrained, ephemeral, and lightweight agents to remain
discoverable, in the sense of {{attachment}}, without bearing full DHT participation cost.

## Full Node

A full-node agent maintains DHT routing tables, participates in
gossip exchanges with peers, accepts incoming DHT queries, and
optionally serves as a relay for light-client agents.

Full-node agents **MUST** be reachable at a stable AGTP endpoint
during their participation lifetime. Full nodes **SHOULD** have
predictable availability and **SHOULD NOT** be subject to frequent
churn.

Typical full-node deployments include enterprise gateways, long-lived
infrastructure agents, organizational agent directories, and dedicated
relay services.

## Light Client

A light-client agent maintains no DHT routing tables or
participate in gossip directly. Instead, the light-client agent
communicates with one or more full-node peers (typically operated by
the same Owner-ID or trusted relay services) for presence announcement
and attachment lookup and search operations.

Light-client agents announce their presence to their chosen full-node
peers, which then propagate the announcement into the DHT and gossip
layer on their behalf. Search queries from light-client agents are
forwarded by full-node peers, with results returned to the light
client.

Light-client mode is appropriate for resource-constrained agents,
ephemeral task agents with short lifetimes, mobile or intermittent
agents, and consumer agents where DHT participation cost is
disproportionate to their search needs.

## Relay-Mediated

A relay-mediated agent operates entirely through a relay full-node.
The relay receives all incoming and outgoing presence and search
traffic for the relayed agent. From the network's perspective, the
relay appears to be the addressable participant; the relayed agent's
Agent-ID is associated with the relay endpoint.

Relay-mediated mode is appropriate for agents behind NAT or firewall
boundaries, for agents with intermittent connectivity, and for
deployments where direct DHT participation is operationally
infeasible.

Relays **MUST** preserve the cryptographic integrity of the relayed
agent's identity and **MUST NOT** modify presence announcements or
search responses on behalf of the relayed agent without explicit
agent consent.

# Lifecycle Methods {#methods}

This document defines three new AGTP methods extending the lifecycle
verb category defined in {{AGTP}}. These methods are AGTP-native and
operate over the AGTP wire format.

## ANNOUNCE

The ANNOUNCE method publishes an agent's presence into the AGTP
Presence layer. An agent invokes ANNOUNCE upon joining the network,
when its visibility posture changes within the bounds permitted by its
certificate, or as a periodic refresh of its presence state. The
first ANNOUNCE is a consequence of the agent's ACTIVATE transition
({{AGTP}}): the agent **MUST** announce within the presence TTL of
becoming Active, and a DEACTIVATE, REVOKE, or DEPRECATE transition
**MUST** be followed by WITHDRAW. Activation is therefore the single
publish act; ANS registration ({{AGTP-DISCOVERY}}) and presence
announcement both follow from it.

ANNOUNCE requests are sent to one or more peer agents within the
sender's visibility scope. The receiving agents validate the
announcement signature, incorporate the announcement into their local
presence state, and propagate the announcement via gossip subject to
the visibility rules declared by the announcing agent.

Request semantics:

- **Idempotent**: Yes (same announcement state may be sent repeatedly)
- **Authority-Scope**: `presence:announce` **REQUIRED**
- **Response**: 200 OK with current peer presence state, or 404 if the
  receiving agent is outside the announcing agent's visibility scope

## WITHDRAW

The WITHDRAW method removes an agent's presence from the AGTP Presence
layer prior to disconnection. WITHDRAW is the explicit opposite of
ANNOUNCE and signals graceful exit from the network.

WITHDRAW requests propagate via gossip and remove the agent's presence
record from peer state. Agents that omit WITHDRAW before
disconnection have their presence eventually expired via TTL-based
record aging.

Request semantics:

- **Idempotent**: Yes
- **Authority-Scope**: `presence:withdraw` **REQUIRED**
- **Response**: 200 OK on successful withdrawal acknowledgment

## PROBE

The PROBE method queries the current state of a specific agent. PROBE
is used during DHT lookups for liveness verification, by search
clients evaluating returned peer lists, and by monitoring systems
checking agent availability.

PROBE responses are filtered by the responding agent's visibility
posture. An agent whose visibility posture excludes the requesting
agent **MUST** return a 404 response, indistinguishable from a response
indicating the agent is unknown. This is the structural enforcement
mechanism for the `invisible` presence mode.

Request semantics:

- **Idempotent**: Yes
- **Authority-Scope**: `presence:probe` **REQUIRED**
- **Response**: 200 OK with current presence state, 404 if the agent is
  outside the requesting agent's visibility scope or unknown

# Visibility Model {#visibility}

AGTP Presence specifies three orthogonal axes of visibility control.
The visibility model is structurally analogous to operating system
file permissions, gaming platform privacy controls, and similar
well-understood mechanisms. Visibility is declared via AGTP-CERT
certificate extensions and is cryptographically bound to the agent's
identity.

## Presence Visibility

Presence visibility determines whether the agent appears in ambient
search at all. The following modes are defined:

| Mode | Semantics |
|------|-----------|
| `public` | Visible to any AGTP-speaking agent that queries |
| `tier-scoped` | Visible only to agents within the same trust tier per {{AGTP-TRUST}} |
| `owner-domain` | Visible only to agents sharing the same Owner-ID domain |
| `explicit-only` | Visible only to agents in an explicit allow list |
| `invisible` | Absent from every search result; reachable only by direct Agent-ID, which is discovery in the sense of {{attachment}} |

Invisible-mode agents remain on the AGTP substrate and are reachable
by other agents that already possess the Invisible agent's Agent-ID.
The invisible mode applies to search results; it leaves unaffected
prevent direct addressing.

## Disclosure Visibility

Disclosure visibility determines what attributes are advertised about
an agent that is itself visible. The following modes are defined:

| Mode | Disclosed |
|------|-----------|
| `full` | Agent-ID, Owner-ID, capabilities, methods, Authority-Scope |
| `capabilities` | Agent-ID, Owner-ID, capabilities (omitting scope and method detail) |
| `identity-only` | Agent-ID and Owner-ID only |
| `existence-only` | Agent-ID only (presence acknowledged without metadata) |

Agents in `existence-only` disclosure mode are visible in search
results but advertise no operational metadata. Searching agents
that wish to learn more must invoke explicit AGTP methods (DESCRIBE,
QUERY) against the discovered agent, which the agent may then
authorize or deny.

## Audience Scoping

Audience scoping further constrains which agents may observe a
visible agent. Audience scoping composes with presence visibility:
an agent in `public` presence mode with restricted audience scoping
is visible only to the specified audience even though its presence
mode would otherwise allow broad visibility.

The following audience scope expressions are defined:

| Expression | Semantics |
|------------|-----------|
| `tier:N` | Agents at trust tier N or higher |
| `owner-domain:<domain>` | Agents with Owner-ID matching the specified domain |
| `governance-group:<name>` | Agents in a named governance group |
| `agent-id:<list>` | Agents in an explicit Agent-ID allow list |
| `capability:<value>` | Agents bound to the specified capability per AGTP method library |
| `industry:<value>` | Agents asserting membership in the specified industry classification |
| `geographic:<region>` | Agents asserting presence within a geographic region |

The `capability` expression composes with the capability partition
dimension: an agent restricting its audience to `capability:settlement`
is visible only to other agents that have bound the settlement
capability through the method library, regardless of which scope they
joined to make the search.

Audience expressions **MAY** be combined with logical operators (AND,
OR, NOT) in the certificate extension to express compound audience
constraints.

## Certificate Extension Schema

Visibility posture is declared in an AGTP-CERT extension. The
extension OID and schema are defined in {{iana-considerations}}. The
extension carries three required fields corresponding to the three
visibility axes:

```
PresenceVisibilityExtension ::= SEQUENCE {
    presenceMode    PresenceMode,
    disclosureMode  DisclosureMode,
    audienceScope   AudienceExpression
}
```

The visibility posture declared in the certificate is the maximum
envelope within which the agent may operate. Runtime visibility may
be reduced within this envelope via the Presence-Mode header on
ANNOUNCE and outbound AGTP requests, but **MUST NOT** exceed the
certificate-declared envelope.

## Runtime Mode Signaling

Agents **MAY** adjust their effective visibility at runtime within
the envelope declared by their certificate. The Presence-Mode header
on outbound AGTP requests signals the agent's current operational
visibility mode. Receiving agents **MUST** honor the declared runtime
mode when generating presence announcements or PROBE responses.

For example, an agent whose certificate declares
`presenceMode: public` may temporarily operate as `presenceMode:
invisible` by signaling Presence-Mode: invisible on its requests. The
agent cannot exceed its certificate envelope (cannot upgrade from
`owner-domain` to `public`) but may reduce visibility within it at any
time.

# Bootstrap

New agents joining the AGTP Presence network require entry points to
the substrate. The bootstrap problem is non-trivial because the
operators of bootstrap services have structural influence over network
topology. This document specifies a decentralized bootstrap model
that avoids single points of capture.

## Multi-Seed Bootstrap

Agents joining the network **SHOULD** maintain a list of at least
three bootstrap peers operated by independent parties. Bootstrap
peer lists **MAY** be obtained from:

- AGTP-CERT certificate authorities, which include bootstrap peer
  hints in issued certificates
- DNS-anchored seed records published under operator domains
- Configuration provided by deployment tooling
- Previously discovered AGTP peers from prior sessions

Agents **SHOULD** rotate their bootstrap peer list periodically and
**SHOULD NOT** depend on a single bootstrap operator indefinitely.

## Trust-Tier-Specific Bootstrap

Tier 1 Verified, Tier 2 Org-Asserted, and Tier 3 Experimental agents
**MAY** use distinct bootstrap peer lists appropriate to their tier.
Tier 1 bootstrap peers, in particular, are typically operated by
institutional participants (financial institutions, regulated
entities, standards bodies, certificate authorities) with operational
accountability for their bootstrap services.

## Governance-Driven Bootstrap Rotation

Bootstrap peer lists **MAY** be rotated through governance processes
defined in {{AGTP-TRUST}}, including periodic refresh of authorized
bootstrap operators and revocation of bootstrap peers that fail to
meet operational requirements.

# Security Considerations {#security}

The introduction of substrate-level presence creates an attack surface
that this document addresses through multiple layered mitigations.
Implementers and operators **MUST** consider the following.

## Sybil Resistance

Attackers may attempt to flood the network with falsified agent
identities to gain disproportionate influence over routing, gossip
propagation, or search results.

Mitigations:

- All Agent-IDs are 256-bit SHA-256 hashes of Agent Genesis documents
  per {{AGTP-IDENTIFIERS}}. Generating a target Agent-ID for a
  specific DHT region requires brute-forcing the hash preimage, which
  is computationally infeasible.
- AGTP-CERT requirements per {{AGTP-CERT}} bind Agent-IDs to issuing
  authority chains. Sybil attacks require obtaining valid certificates
  from cooperating authorities.
- Tier-scoped presence partitioning means that an attacker generating
  many Tier 3 Experimental agents cannot affect Tier 1 or Tier 2
  visibility scopes.
- Per-Owner-ID announcement rate limits **SHOULD** be enforced by
  participating nodes to prevent presence flooding from a single
  controlling entity.

## Eclipse Attacks

An attacker controlling a sufficient number of nodes near a target
Agent-ID in DHT keyspace may be able to isolate the target from the
broader network, intercepting or dropping its DHT operations.

Mitigations:

- Parallel disjoint lookups (alpha=3) per S/Kademlia {{SKADEMLIA}}
  reduce the probability that all lookup paths are controlled by
  attacker nodes.
- k-bucket diversification requirements ensure routing table entries
  span multiple network locations and Owner-ID domains.
- Deterministic content-hash Agent-IDs prevent attackers from
  cheaply generating IDs that cluster around a target.

## Presence Flooding and Denial of Service

Attackers may attempt to flood the network with announcements,
search queries, or probe requests to exhaust resources of
participating nodes.

Mitigations:

- Per-Owner-ID and per-Agent-ID rate limits on ANNOUNCE, PROBE, and
  search and lookup operations.
- Quota enforcement tied to AGTP-CERT issuance, allowing the issuing
  authority to revoke certificates of abusive participants.
- Gossip protocol bounded fanout (default 3) prevents amplification.
- Light-client and relay-mediated participation modes allow legitimate
  resource-constrained agents to participate without enabling
  amplification attacks.

## Metadata Leakage Through Traffic Analysis

Even with `invisible` presence mode, an agent generates network
traffic that may reveal its existence and activity patterns through
traffic analysis.

Mitigations:

- This document provides incomplete protection against
  sophisticated traffic analysis. The visibility model is best-effort
  with respect to passive network observation.
- Agents requiring strong unobservability **SHOULD** use additional
  measures including TLS connection padding, traffic shaping, or
  participation through privacy-enhancing relays.
- Operators **SHOULD** acknowledge that invisible-mode agents are
  visible to the network operators they transit, even when invisible
  to other AGTP agents.

## Visibility Declaration vs. Enforcement

Cryptographic declaration of visibility posture via AGTP-CERT
extensions ensures that an agent's visibility intent is verifiable.
Enforcement of visibility, however, depends on the cooperation of
participating nodes.

This document specifies that conforming AGTP Presence implementations
**MUST** honor declared visibility postures in their ANNOUNCE,
PROBE, and gossip behaviors. Non-conforming implementations may
ignore declared posture. Visibility is enforced by the protocol where
implementations conform; the cryptographic declaration prevents
visibility claims from being falsified but leaves unprevented
non-conforming actors from observing what they choose to observe.

This is structurally similar to TLS certificate validation: a server's
certificate declares its identity verifiably, but a non-conforming
client may choose to ignore validation failures.

## Bootstrap Peer Capture

An attacker controlling bootstrap services may influence the initial
routing state of joining agents, potentially enabling eclipse-class
attacks on newly joined participants.

Mitigations:

- Multi-seed bootstrap with peers operated by independent parties.
- Periodic rotation of bootstrap peer lists.
- Trust-tier-specific bootstrap with institutional accountability for
  Tier 1 bootstrap services.
- Governance-driven revocation of bootstrap peers that fail operational
  requirements.

## Privacy of Search Queries

Search queries themselves may reveal information about the
querying agent's interests, intentions, or operational state.

Mitigations:

- Search queries carry AGTP wire-level identity, so they cannot be
  fully anonymous within the AGTP substrate.
- Operators concerned about query privacy **MAY** route search
  queries through privacy-enhancing relays or use cover traffic.
- Future revisions of this document may specify query blinding or
  oblivious lookup mechanisms for high-privacy use cases.

## Partition Vocabulary Governance

The capability partition dimension derives from the AGTP method
library and inherits its governance properties. The industry and
region partition dimensions depend on external registries (such as
NAICS, ISIC, ISO 3166) and inherit those registries' governance
properties.

The architectural concern: whoever governs the partition vocabulary
has structural influence over how agents are partitioned and
therefore over which agents become visible to which others. Free-form
partition strings would shatter the keyspace and collapse cross-scope
resolution. Tightly governed vocabularies create central authorities
in a system whose value proposition includes decentralization.

Mitigations:

- The capability axis is governed by the method library, which is
  itself a structurally constrained vocabulary tied to AGTP method
  definitions. This avoids introducing a new governance body for the
  capability axis specifically.
- Industry and region axes use established external registries with
  long-standing governance. AGTP Presence maintains none of these
  registries; it consumes them.
- Implementations **SHOULD** support extensible dimension registries
  to allow operational deployments to define additional partition
  axes for specific use cases without requiring changes to this
  document.
- Overlay coordinators that manage split and merge decisions for
  scopes **SHOULD** operate under governance frameworks appropriate
  to the trust tier they serve.

# Composition with Existing AGTP Specifications

AGTP Presence is built on existing AGTP primitives and composes with
the AGTP draft family without requiring changes to the base
specification.

## Composition with AGTP Base

The new methods (ANNOUNCE, WITHDRAW, PROBE) extend the lifecycle verb
category defined in the base specification. They are carried as
standard AGTP requests on the AGTP wire format, authenticated via
existing identity headers (Agent-ID, Owner-ID, Authority-Scope), and
verified per the existing trust framework.

## Composition with AGTP-IDENTIFIERS

Canonical Agent-IDs as defined in {{AGTP-IDENTIFIERS}} provide the
content-addressable keys used by the DHT. No changes to the
identifier specification are required.

## Composition with AGTP-TRUST

Trust tiers and verification paths defined in {{AGTP-TRUST}} define
the scoping model for visibility partitioning. The Tier 1, Tier 2,
and Tier 3 classifications determine default overlay partitioning.

## Composition with AGTP-CERT

The visibility extension defined in this document is registered in
the AGTP-CERT certificate extension namespace. AGTP-CERT certificate
issuance processes incorporate visibility posture declaration as part
of certificate generation.

## Composition with AGTP-DISCOVERY

{{AGTP-DISCOVERY}} specifies the DISCOVER method and the Agent Name
Service, which operate over the presence substrate. The two documents
divide the three operations of {{RFC6763}} between them: Presence is
discovery (Agent-ID to attachment); DISCOVER's intent-driven mode is
search (criteria to candidates), evaluated against the live presence
layer rather than a separate registry; ANS is resolution (name to
Agent-ID). Agents a search returns are agents currently visible in
the requesting agent's presence scope, and each result carries the
attachment Presence holds for it. The dichotomy between the two
directions is stated once, in {{AGTP-DISCOVERY}}: ambient (push,
continuous, ANNOUNCE and gossip) and reactive (pull, point in time,
DISCOVER); this document uses those two words for the two
directions.

# Scaling Considerations

AGTP Presence is designed to scale through partitioning and hybrid
participation rather than through any single-overlay solution. This
section addresses scaling characteristics at different population
sizes.

## At One Million Agents

Single-tier or single-governance-domain populations of approximately
one million agents are within the proven operational envelope of
existing Kademlia DHT deployments {{KADEMLIA}} {{IPFS}}. Lookup
latency is O(log N), approximately 20 hops for one million nodes,
typically faster in practice due to caching and parallel queries.
Gossip convergence within scope completes in seconds. Routing table
state per node remains bounded at approximately 20 to 30 entries.

## At One Billion Agents

Global populations approaching one billion agents are addressed
through the federation-of-scoped-overlays model. No single overlay
contains the full population. Most agents participate in scoped views
of thousands to tens of thousands of relevant peers. Cross-tier and
cross-domain search proceeds through controlled federation points.

This is structurally analogous to how the internet scales through BGP:
the internet is a federation of
independently operated systems with controlled inter-AS routing
rather than a single flat network. AGTP Presence
adopts the same architectural pattern for agent populations.

## Light-Client Predominance

In production deployments at scale, the participation distribution is
expected to skew heavily toward light-client and relay-mediated
agents. Full nodes are operated by infrastructure providers,
enterprises, and institutional participants. Ephemeral consumer and
task agents participate as light clients. This distribution is
consistent with how other large-scale P2P systems have operated.

## Observability and Operations

Implementers and operators of AGTP Presence networks **SHOULD** track
the following operational metrics:

- DHT routing table convergence time at agent join
- Gossip convergence latency within visibility scopes
- Per-node bandwidth consumption
- Presence record TTL effectiveness and stale-record rates
- Rate of failed validation (signature failures, visibility
  violations, bootstrap failures)

# IANA Considerations {#iana-considerations}

This document requests IANA registration of the following.

## AGTP Method Names

Registration of three new AGTP method names in the AGTP method
registry established by {{AGTP}}:

| Method | Category | Description | Reference |
|--------|----------|-------------|-----------|
| ANNOUNCE | Lifecycle | Publish agent presence to the substrate | This document |
| WITHDRAW | Lifecycle | Remove agent presence prior to disconnection | This document |
| PROBE | Lifecycle | Query agent presence state | This document |

## AGTP-CERT Extension OID

Registration of an OID for the Presence Visibility Extension in the
AGTP-CERT extension namespace established by {{AGTP-CERT}}:

- Extension name: PresenceVisibilityExtension
- OID: TBD (to be assigned)
- Reference: This document

## Authority-Scope Values

Registration of three new Authority-Scope values:

| Scope | Permits | Reference |
|-------|---------|-----------|
| `presence:announce` | Invoking the ANNOUNCE method | This document |
| `presence:withdraw` | Invoking the WITHDRAW method | This document |
| `presence:probe` | Invoking the PROBE method | This document |

## Presence-Mode Header

Registration of the Presence-Mode header in the AGTP header registry:

- Header name: Presence-Mode
- Defined values: `public`, `tier-scoped`, `owner-domain`,
  `explicit-only`, `invisible`
- Reference: This document

## Partition Dimension Registry

This document requests IANA establishment of a new registry for
AGTP Presence partition dimensions. The registry contains the
following initial entries:

| Dimension Name | Vocabulary Source | Reference |
|----------------|-------------------|-----------|
| `tier` | AGTP-TRUST trust tier values | {{AGTP-TRUST}} |
| `owner-domain` | DNS domain names | This document |
| `capability` | AGTP method library | {{AGTP}} |
| `industry` | NAICS, ISIC, or operator-defined classifications | This document |
| `region` | ISO 3166 or operator-defined geographic identifiers | This document |

Registration policy: Specification Required. New partition dimensions
**MUST** be accompanied by a referenced vocabulary source, governance
description, and expected use case description.

# Prior Art and Related Work

AGTP Presence builds on patterns established in distributed systems
research and practice. The following related work is acknowledged
and contextualized.

## Distributed Hash Tables

Kademlia {{KADEMLIA}} provides the foundational DHT design used here.
S/Kademlia {{SKADEMLIA}} contributes security hardening for adversarial
network conditions. IPFS {{IPFS}} demonstrates production deployment
of Kademlia at hundreds of thousands of nodes.

The contribution of AGTP Presence beyond these systems is the
integration of cryptographically declared visibility scoping with
DHT operations, allowing scoped views over a shared substrate without
requiring separate physical networks per scope.

## Gossip Protocols

Production gossip systems including Serf {{SERF}} demonstrate the
viability of gossip-based membership and state convergence at scale.
The gossip mechanism specified here adopts the bounded-fanout
epidemic propagation model used in these systems, with the addition
of certificate-based signature verification for all announcements.

## Peer-to-Peer Discovery

libp2p {{LIBP2P}} provides general-purpose peer discovery primitives
including Kademlia DHT and gossipsub. AGTP Presence differs in
specifying trust-tier-aware visibility scoping and certificate-bound
declaration of visibility posture as substrate properties rather than
application-layer policy.

## Presence Protocols

XMPP presence (RFC 6121) provides per-user presence within federated
servers using subscription rosters. The AGTP Presence model differs
in being ambient (requiring no explicit subscription) and in being
substrate-level rather than server-mediated.

## Comparison with Pull-Based Discovery

ARD, DNS-AID, ANS, and similar pull-based proposals provide
mechanisms for querying known endpoints; in the vocabulary of
{{AGTP-DISCOVERY}} they are search and resolution. AGTP Presence
operates at a different layer, providing the live population view
against which those reactive queries can be evaluated. AGTP Presence
replaces none of them; it provides the substrate that makes reactive
queries operate against a known live population rather than against
static or stale directories.

--- back

# Acknowledgements

The author thanks Tom Roberts for the architectural insights that
shaped the ambient discovery framing, particularly the reference to
Project Xanadu's always-connected design. The author also thanks the
broader IETF agent2agent community for ongoing discussion of agent
infrastructure scaling and the limitations of pull-based discovery
models.

# Changes from v00

Version 01 is a vocabulary and alignment revision; the DHT, gossip,
partitioning, participation modes, lifecycle methods, and visibility
model are unchanged on the wire.

- **Discover, search, resolve.** Following {{RFC6763}} as adopted in
  {{AGTP-DISCOVERY}}, this document now calls what Presence provides
  discovery (Agent-ID to attachment) and what the DISCOVER method's
  intent-driven mode provides search. Every "discovery query" sent by
  a light client to a relay is now a search query or an attachment
  lookup as the case requires.
- **Attachment record** ({{attachment}}): the signed, sequenced,
  short-lived record ANNOUNCE publishes and PROBE returns, as the
  mutable half of the permanent-identity versus current-location
  split.
- **Principals rather than hosts** and **the addressing abstraction**
  ({{addressing}}): the introduction states why the stack's
  host-nature presumption fails for agents and answers, in the
  document, the question of over what protocol and to what address.
- ANNOUNCE tied to ACTIVATE: the first ANNOUNCE follows the
  agent's ACTIVATE transition in {{AGTP}}, and WITHDRAW follows
  DEACTIVATE, REVOKE, or DEPRECATE, so activation is the single
  publish act. The attachment record stated as the orchestrator's
  publication feed and, read for the requester, as its location
  context for search.
- Wire identity restated around `Agent-ID` and `Request-Signature`
  with Owner-ID resolved from the Identity Document per {{AGTP}}.
- References pinned to current revisions; AGTP-IDDOC and RFC 6763
  added, two unused references dropped. "Autonomous systems" in the
  BGP analogy reworded. House
  style pass.
