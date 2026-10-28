---
title: "AGTP-RCNS: Runtime Contract Negotiation Substrate"
abbrev: "AGTP-RCNS"
docname: draft-hood-agtp-rcns-00
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent transfer protocol
  - runtime contract negotiation
  - endpoint synthesis
  - PROPOSE
  - synthesized contracts

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
    uri: https://agtp.io

normative:
  RFC2119:
  RFC8174:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-09
    date: 2026
  AGTP-METHODS:
    title: "AGTP Methods (AGTP Verbs): Method Vocabulary, Path Grammar, and Endpoints"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-methods-00
    date: 2026

informative:
  AGTP-TRUST:
    title: "AGTP Trust and Verification Specification"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-trust-02
    date: 2026
  AGTP-IDENTIFIERS:
    title: "AGTP Identifier Chain"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identifiers-02
    date: 2026
  AGTP-API:
    title: "AGTP-API: Verbs, Paths, Endpoints, and Synthesis"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-api-01
    date: 2026

--- abstract

This document specifies the Runtime Contract Negotiation Substrate
(RCNS) for the Agent Transfer Protocol (AGTP) {{AGTP}}: the
mechanism by which an AGTP server synthesizes an endpoint at
runtime, from registered primitives, when a caller addresses a
method and path pair the server has yet to register. RCNS is
invoked explicitly through the PROPOSE floor method and implicitly
when a server is configured to treat unregistered requests as
synthesis candidates. The document defines the four-lock dispatcher
gate that governs when synthesis may occur, the two contract
delivery modes, the PROPOSE request and response body shapes,
recipe versioning, contract scoping and lifecycle, the headers and
rate-limiting contract, the Attribution-Record extension fields
that make every invocation served under a contract attributable,
and the inventory surfaces through which contracts are observed.
The material was previously carried in {{AGTP-API}}, which this
document and {{AGTP-METHODS}} together replace.

--- middle

# Introduction

AGTP {{AGTP}} is a transfer protocol for AI agent traffic. It
establishes identity, authority, attribution, and an eighteen-method
floor. {{AGTP-METHODS}} specifies the contract layer above that
floor: the method vocabulary, path grammar, the endpoint primitive,
the server manifest, and the policy by which an operator declares
what a server exposes. Every endpoint declared that way is a Tier A
or Tier B endpoint in the tier taxonomy of {{AGTP-METHODS}}.

This document specifies Tier C: endpoints that exist because an
agent asked for them. A caller addresses a method and path pair the
server has yet to register. Instead of refusing, a server that
participates in runtime negotiation may synthesize a contract for
that pair from the primitives it already exposes, deliver the
contract to the caller, and serve invocations under it with the
same identity, authority, and attribution guarantees as any
registered endpoint.

The substrate property this delivers is the one AGTP is designed
around: an agent arriving at a server can discover what the server
offers, reason about fit, verify that its authority covers the
invocation, and, when nothing fits, propose what it needs. The
protocol governs the proposal the same way it governs every other
invocation, because the protocol knows what a method on a path
means.

## Relationship to AGTP and AGTP-METHODS

Base AGTP defines the PROPOSE floor method, the negotiation status
codes 261, 262, 263, 461, 463, and 464, the reason vocabulary for
461 and 464, the `INSPECT target=contract` and
`INSPECT target=rcns-attempt` extensions, and the `rcns_*` lifecycle
events. {{AGTP-METHODS}} defines the endpoint primitive that a
synthesized contract instantiates, the tier taxonomy that
classifies a contract as Tier C, the recipe-backed composition
handler, and the manifest policy fields (`synthesis_enabled`,
`max_synthesis_depth`) through which an operator advertises
participation. This document defines everything between those two
sets: when synthesis fires, what it produces, how the product is
delivered, scoped, observed, and retired.

A server that implements {{AGTP-METHODS}} without this document is
conformant and refuses unregistered pairs with 404 or 405. RCNS is
an opt-in substrate; the default posture is off.

## Document Lineage

The normative text in this document was carried in {{AGTP-API}}
as the PROPOSE and Synthesis chapter and the `DISCOVER /patterns`
and `DISCOVER /contracts` built-in endpoints. It is extracted here
unchanged in substance so that the method vocabulary and the
negotiation substrate can be reviewed, cited, and revised on their
own timelines. Section anchors from {{AGTP-API}} are preserved
where they appear.

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}}
{{RFC8174}} when, and only when, they appear in all capitals, as
shown here.

# Terminology {#terminology}

Terms defined in {{AGTP}} and {{AGTP-METHODS}} (method, path,
endpoint, semantic block, server manifest, method policy) carry
their definitions here. This document adds:

Synthesis:
: The runtime instantiation of an endpoint in response to a PROPOSE
  request or an unregistered invocation, composing the requested
  behavior from existing endpoints while preserving authority
  through the composition.

Contract:
: A synthesized endpoint together with the identifier
  (`synthesis_id`), recipe lineage, scope, expiration, and budget
  under which the server agreed to serve it. A contract is held by
  the Agent-ID it was synthesized for.

Recipe:
: A named, versioned composition plan registered at the server,
  from which a contract is synthesized. Recipes are server-internal
  artifacts; the recipe-backed composition handler is specified in
  {{AGTP-METHODS}}.

Tier C Endpoint:
: An endpoint that exists only as a contract. The tier taxonomy is
  specified in {{AGTP-METHODS}}.

# Runtime Contract Negotiation {#rcns}

## Runtime Contract Negotiation Substrate {#rcns-overview}

The Runtime Contract Negotiation Substrate (RCNS) is the
machinery by which a server delivers Tier C endpoints
(the tier taxonomy in {{AGTP-METHODS}}): endpoints synthesized at runtime from
registered primitives when a caller addresses a `(method,
path)` pair that is absent from the Tier A or Tier B set.

PROPOSE is the AGTP method that explicitly invokes RCNS.
RCNS also operates implicitly when a server is configured
to accept unregistered requests as synthesis candidates;
the wire surface in that case is the same set of contract
identifiers and reasoned status codes (461, 464) registered
in {{AGTP}} and specified here.

This document specifies the four-lock dispatcher gate, the
two delivery modes (confirm-first 461 and optimistic 263
with the `Contract-Synthesized` header), the endpoint-keyed
PROPOSE body shape, the contract scoping rules, the
per-agent rate limiting and idempotency key contract, the
`synthesis_id`, `contract_hash`, and `negotiation_origin`
Attribution-Record extension fields, and the
`DISCOVER /patterns`, `DISCOVER /contracts`,
`INSPECT target=contract`, and `INSPECT target=rcns-attempt`
inventory surfaces. The Tier C taxonomy in {{AGTP-METHODS}}
and the reservations in {{AGTP}} (status codes 461 and 464,
reason vocabulary `rcns-disabled`,
`trust-tier-insufficient`, `composition-impossible`,
`synthesis-error`, `contract-not-yours`,
`contract-revoked`) constitute the wire foundation for the
substrate.

### Tier C Resolution {#tier-c-resolution}

A server with active synthesized contracts **MUST** maintain
a resolver that answers, for any inbound `(method, path)`
pair, whether an active Tier C contract exists for that pair.
The resolver is the dispatcher hook that closes the loop
between the tier classification (the tier taxonomy in {{AGTP-METHODS}}) and
the synthesis machinery: an unregistered `(method, path)`
pair that resolves to an active contract is classified Tier C
and dispatched to the synthesized handler; a pair that
fails to resolve is dispatched per the unregistered-pair rules in
the tier taxonomy in {{AGTP-METHODS}}.

The resolver **MUST** scope each lookup by the requesting
Agent-ID. A contract synthesized for Agent-ID X **MUST NOT**
resolve for an unrelated Agent-ID Y; the server **MUST**
return `464 RCNS No Contract` with `reason:
contract-not-yours` in that case. Contracts that have
expired, been revoked, or been suspended **MUST NOT**
resolve.

The resolver returns, when an active contract exists:

- The `synthesis_id` of the matched contract.
- The `recipe_lineage` (recipe name and captured recipe
  version) the contract was synthesized under.
- The contract's expiration time and remaining budget, if
  applicable.

These fields are server-internal at lookup time but appear
on `INSPECT target=contract` responses (the `target=contract`
extension to INSPECT specified in {{AGTP}}) and in the
Attribution-Record extension fields
({{rcns-attribution-fields}}) on every invocation served
under the contract.

### The Four-Lock Dispatcher Gate {#rcns-gate}

RCNS synthesis fires only when **all four** of the following
locks are open at the moment of an unregistered request.
The locks are deliberately distributed across actors: server
operator, requesting agent's invoker, requesting agent's
identity, and requesting agent's posture. No single party
can unilaterally activate RCNS.

1. **Server policy enabled.** The server's operator has
   set `[policies.rcns].enabled = true` (or the equivalent
   in the server's configuration format). The default
   posture is OFF; servers ship with RCNS disabled and the
   operator opts in explicitly.

2. **Caller opt-in header.** The request carries
   `Allow-RCNS: true`. Clients that have withheld consent to
   runtime negotiation never get it; a missing or
   non-`true` value means the unregistered request is
   refused per the standard Tier classification rules
   (the tier taxonomy in {{AGTP-METHODS}}).

3. **Caller scope.** The requesting agent's
   `Authority-Scope` includes the `rcns:negotiate` token.
   Agents lacking this token cannot trigger
   synthesis even with the header set.

4. **Minimum trust tier.** The requesting agent's resolved
   `trust_tier` ({{AGTP-TRUST}}) is greater than or equal
   to the server's configured `min_trust_tier` (default
   `1`, the strictest). Agents below the threshold receive
   `464 RCNS No Contract` with `reason:
   trust-tier-insufficient`.

When any of the four locks is closed, the server **MUST NOT**
synthesize. The specific refusal depends on which lock is
closed:

| Lock closed | Server response |
|---|---|
| Server policy disabled | `464 RCNS No Contract` reason: `rcns-disabled` |
| Caller opt-in absent | Standard unregistered-pair refusal (`404` or `405`); RCNS goes unsignaled |
| Caller scope absent | `262 Authorization Required` reason `scope-required scope: rcns:negotiate` when the agent's identity document grants the token but the request left it unclaimed; `455 Scope Violation` reason `scope-exceeded` when the document omits it (obtainability rule of {{AGTP}}) |
| Trust tier insufficient | `464 RCNS No Contract` reason: `trust-tier-insufficient` |
{: title="Four-Lock Gate Refusal Mapping"}

The asymmetry between locks 1 and 2 (`rcns-disabled` 464 vs
opaque 404) is deliberate: the server policy lock surfaces
RCNS posture to interested callers; the opt-in lock is a
no-surprise gate for callers that never asked.

A request that holds the Synthesis-Id header (presenting an
already-issued contract) **MUST NOT** trigger a new
synthesis attempt; the gate is skipped and the request is
dispatched directly through the contract resolution path.
This prevents recursive RCNS: a contract invocation cannot
itself produce a new contract.

### Delivery Modes {#rcns-delivery-modes}

When the four-lock gate opens and the server determines a
synthesis is possible, the server delivers the contract in
one of two modes selected by operator policy:

**Confirm-first (default):**
The server returns `461 RCNS Contract Available` with a
response body carrying the synthesized contract preview.
The body **MUST** include:

~~~json
{
  "synthesis_id": "<opaque identifier>",
  "method": "BOOK",
  "path": "/room",
  "endpoint": { "..." : "..." },
  "recipe_lineage": {
    "recipe_name": "<recipe identifier>",
    "recipe_version": "<version string>"
  },
  "contract_hash": "<sha256 of canonical contract>",
  "expires_at": "<RFC 3339 timestamp>"
}
~~~

The caller accepts the contract by re-issuing the original
request with the `Contract-Synthesized` header carrying the
returned `synthesis_id`. The contract is bound to the
caller's Agent-ID at the moment the 461 was issued; a
different Agent-ID presenting the same `synthesis_id` is
refused with `464 contract-not-yours`. The caller declines
the contract by ignoring the response; the contract record
**MAY** be garbage-collected by the server after a
configured TTL.

**Optimistic:**
The server synthesizes the contract and executes the
request inline, returning the normal `200 OK` (or method-
specific success code) for the underlying action. The
response **MUST** carry the `Contract-Synthesized` header
with the issued `synthesis_id`, signaling to the caller
that this response was served under a runtime-negotiated
contract rather than a Tier A or Tier B endpoint. Callers
that wish to repeat the action send subsequent requests
with the `Contract-Synthesized` header to bypass the gate.

Optimistic mode reduces latency at the cost of removing
the caller's opportunity to preview the synthesized
contract. Confirm-first is the safer default; optimistic
is appropriate when the server operator knows recipes
produce contracts the caller will accept.

### Contract Scoping and Lifecycle {#rcns-scoping}

A synthesized contract is bound to:

- The originating Agent-ID (set when the 461 was issued or
  the optimistic synthesis fired).
- The recipe and recipe version captured at synthesis time
  ({{recipe-versioning}}).
- The contract's expiration timestamp.

Servers **MUST** enforce these bindings on every request
carrying a `Contract-Synthesized` header:

- Wrong Agent-ID → `464 contract-not-yours`.
- Unknown `synthesis_id` → `404 Not Found`.
- Expired or revoked contract → `464 contract-revoked`.

Contracts have three release paths:

- **Caller release.** The originating agent invokes
  `SUSPEND` with the `synthesis_id` parameter to release a
  contract it no longer needs. The release emits an
  `rcns_release` event in the agent's lifecycle stream and
  evicts the contract from the runtime.
- **Operator revocation.** The server operator invokes
  `REVOKE` with `target=contract` and `synthesis_id` to
  revoke a contract; the runtime evicts immediately and
  the next presentation returns `464 contract-revoked`.
- **TTL expiration.** Contracts have an operator-configured
  TTL; the runtime evicts on expiry.

#### Stale Contract Sweep {#stale-contract-sweep}

When an operator edits a recipe that has active contracts
synthesized under earlier versions ({{recipe-versioning}}),
those contracts continue to execute under their captured
version; the spec guarantees in-flight contract stability
across recipe edits. Operators that need to flush stale
contracts (typically after a security-relevant recipe
change) invoke `REVOKE target=stale-contracts`:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `target` | **MUST** | The literal string `stale-contracts`. |
| `mode` | **MAY** | One of `grandfather` (report drifted contracts but leave them running) or `invalidate` (evict drifted contracts). Default is the value of the `on_policy_change` knob in {{rcns-config}}. |
| `actor` | **SHOULD** | Identifier of the operator invoking the sweep, recorded on emitted lifecycle events. |
{: title="REVOKE target=stale-contracts Parameters"}

The sweep walks every active contract in the server's
synthesis runtime and compares each contract's captured
`recipe_version` against the current version of the
same-named recipe in the loaded policies. Three outcomes
per contract:

- **Version matches.** Contract is left alone.
- **Version differs and mode is `grandfather`.** Contract
  is reported in the response as drifted but **MUST NOT**
  be evicted; the contract continues to execute under its
  captured version.
- **Version differs and mode is `invalidate`.** Contract
  **MUST** be evicted from the runtime. The eviction
  **MUST** emit an `rcns_release` event in the originating
  agent's lifecycle stream with `reason:
  policy-change-invalidation` and the operator's
  identifier carried in the event payload to distinguish
  operator-fired invalidations from agent-initiated
  releases (the `SUSPEND synthesis_id=` invocation defined
  in {{AGTP}}). Subsequent presentations of the evicted
  `synthesis_id` return `404 Not Found`.

Passthrough contracts (those without a recipe lineage,
synthesized without a recipe) are skipped: the sweep has
no version to compare against. Contracts whose recipe has
been entirely removed from policy (the recipe name no
longer exists) are treated as drift with `current_version`
reported as `null`; under `invalidate` mode, these
contracts are evicted.

The sweep returns a structured response listing every
contract examined with its disposition (`unchanged`,
`grandfathered`, or `evicted`). Authorization for
`REVOKE target=stale-contracts` requires the operator
`inspect:all` scope token; agents cannot fire the sweep on
behalf of others.

The sweep is operator-fired by design. AGTP servers lack
support hot configuration reload; the operator explicitly
invokes the sweep when they know they have edited a
recipe and want the resulting drift to be either reported
or invalidated. Operators that want strict
recipe-edit-invalidates-contracts behavior set
`on_policy_change = invalidate` and invoke the sweep
after every recipe edit.

### Headers and Rate Limiting {#rcns-headers}

RCNS uses two request headers and one response header:

| Header | Direction | Purpose |
|---|---|---|
| `Allow-RCNS` | Request | Caller opt-in to runtime negotiation. Value `true` opens lock 2 of the four-lock gate ({{rcns-gate}}). Absent or non-`true` keeps the gate closed. |
| `Contract-Synthesized` | Request and Response | Identifies a synthesized contract by `synthesis_id`. On a request, presents a previously-issued contract to bypass the gate. On a response (optimistic mode), notifies the caller that the response was served under a synthesized contract. |
| `Idempotency-Key` | Request | Per-agent idempotency key for RCNS-eligible requests. Servers **MUST** scope the idempotency cache by `(Agent-ID, Idempotency-Key)`; replays of the same key from the same agent receive the cached response, replays from different agents are independent requests. |
{: title="RCNS Request and Response Headers"}

Servers **MUST** implement per-agent rate limiting on RCNS
gate evaluations. When the per-agent rate exceeds the
configured ceiling, the server returns `429 Too Many
Requests` with `scope: rcns` in the response body to
distinguish RCNS rate limiting from other rate-limit
sources. The default ceiling is operator-configured;
implementations **SHOULD** default to a conservative value
(e.g., 10 evaluations per minute) and operators **MAY**
raise it.

Rate limiting applies to gate evaluations rather than to
invocations served under an already-issued contract.
Repeated `Contract-Synthesized`-bearing requests against a
synthesized contract are rate-limited like any other
endpoint rather than under the RCNS rate limit.

### Attribution-Record Extension Fields {#rcns-attribution-fields}

Every action dispatched under a synthesized contract,
whether RCNS-spawned, PROPOSE-spawned, or future-mechanism-
spawned, **MUST** carry the following extension fields on
its Attribution-Record payload, in addition to the base
identifier-chain fields specified in {{AGTP-IDENTIFIERS}}:

| Field | Type | Description |
|---|---|---|
| `synthesis_id` | string | Opaque identifier of the contract under which this action was dispatched. |
| `contract_hash` | string | SHA-256 of the canonical contract definition (the synthesized endpoint plus its recipe lineage at synthesis time), rendered as 64 lowercase hexadecimal characters. Stable across all invocations of the same contract. |
| `negotiation_origin` | string | One of `rcns-gate` (synthesized via the four-lock gate), `propose` (synthesized via explicit PROPOSE), or `pre-registered` (a future origin for contracts elevated from Tier C to Tier B). |
{: title="RCNS Attribution-Record Extension Fields"}

These fields enable chain inspectors and audit consumers to
group invocations by contract identity, distinguish
contracts produced through different origins, and verify
that a stream of actions all ran under the same
synthesized contract definition.

The fields are present only on actions served under a
contract. Tier A and Tier B endpoint invocations omit them
entirely; their absence is itself diagnostic information.

### RCNS Configuration {#rcns-config}

The full set of operator-controlled RCNS knobs is
specified in this section. The configuration is operator-
authored and is invisible to callers on the wire; what is
visible is the *result* of the configuration (whether the
gate fires, which mode it delivers in, what the rate ceiling
is). The block name (`[policies.rcns]` in the recommended
TOML form) is conventional rather than normative; servers using
different configuration formats apply equivalent semantics.

| Knob | Default | Purpose |
|---|---|---|
| `enabled` | `false` | Master switch (lock 1 of the four-lock gate) |
| `mode` | `confirm-first` | Delivery mode: `confirm-first` (461) or `optimistic` |
| `min_trust_tier` | `1` (strictest) | Minimum resolved trust tier for callers (lock 4) |
| `rate_limit_per_minute` | `10` | Per-agent rate ceiling for gate evaluations |
| `contract_ttl_seconds` | implementation-defined | Default contract TTL; operators **SHOULD** set explicitly |
| `on_policy_change` | `grandfather` | Default behavior of the operator-fired stale-contract sweep ({{stale-contract-sweep}}). One of `grandfather` (report drifted contracts but leave them running) or `invalidate` (evict drifted contracts and emit `rcns_release` events with `reason: policy-change-invalidation`). The sweep is operator-fired; this knob sets its default mode, which operators **MAY** override per invocation. |
{: title="RCNS Configuration Knobs"}

The default posture is deliberately conservative: a
freshly-installed AGTP server has RCNS disabled and, when
enabled, defaults to confirm-first delivery at the
strictest trust tier with a low rate ceiling. Operators
relax these defaults explicitly as their deployment
matures.

## Purpose

PROPOSE is the AGTP method that triggers runtime contract negotiation.
An agent that needs an endpoint the server has yet to advertise
submits a PROPOSE request carrying an endpoint definition conformant to {{AGTP-METHODS}}
specification; the server evaluates the proposal against the
contract layer and either instantiates the endpoint as a
session-scoped artifact or refuses.

PROPOSE is one of the eighteen floor methods defined in {{AGTP}}. Its
parameter table is specified there. This document specifies the
synthesis semantics that govern server-side evaluation.

## Negotiation Flow

~~~
Step 1:  Agent retrieves the server manifest (which carries the
         method policy as `policies.methods`)
Step 2:  Agent determines the needed endpoint is undeclared
Step 3:  Agent constructs a conformant endpoint definition
Step 4:  Agent sends PROPOSE with the endpoint definition in body
Step 5a: Server accepts and synthesizes
         Server evaluates contract conformance and capability envelope
         Server composes the endpoint from existing endpoints
         Server returns 263 Proposal Approved with the
           endpoint definition for the synthesized endpoint
         Synthesis-ID matches original proposal
         Synthesized endpoint is session-scoped by default
         Agent MAY invoke the endpoint immediately
Step 5b: Server rejects (463)
         Server returns 463 Proposal Rejected with structured reason
         Response SHOULD reference manifest entries that satisfy a
           similar capability if available
         Agent MAY modify proposal and retry (maximum 3 turns)
         After 3 rejections agent MUST ESCALATE
~~~

## PROPOSE Request Body Shape {#propose-body}

PROPOSE accepts the proposed endpoint definition in one of two
mutually exclusive body forms. Servers **MUST** accept both
forms; clients **SHOULD** emit the wrapped form for new code.

### Wrapped form (RECOMMENDED)

The proposed endpoint definition is carried under a top-level
`endpoint` field:

~~~json
{
  "method": "PROPOSE",
  "task_id": "task-propose-01",
  "parameters": {
    "endpoint": {
      "method": "BOOK",
      "path": "/room",
      "description": "Books a room for the named guest.",
      "semantic": { "..." : "..." },
      "input_schema": { "..." : "..." },
      "output_schema": { "..." : "..." }
    },
    "persistent": false
  }
}
~~~

The wrapped form makes the `(method, path)` pair the explicit
key of the proposal and clears the way for synthesis-time fields
(e.g., `version`, `lineage`) to ride at the `parameters` level
without colliding with endpoint definition fields. The wrapped
form is the form used by RCNS dispatcher-gate escalations
({{rcns-overview}}).

### Legacy form

The endpoint definition fields appear directly under
`parameters` with a `name` field naming the endpoint:

~~~json
{
  "method": "PROPOSE",
  "task_id": "task-propose-02",
  "parameters": {
    "name": "book-room",
    "method": "BOOK",
    "path": "/room",
    "semantic": { "..." : "..." }
  }
}
~~~

A PROPOSE request **MUST NOT** carry both `endpoint` and `name`
at the same time. Servers **MUST** return `400 Bad Request`
with `reason: ambiguous-body` if both are present.

### 263 Response Body

On approval the server returns `263 Proposal Approved` with a
body carrying the synthesized endpoint:

~~~json
{
  "status": 263,
  "method": "BOOK",
  "path": "/room",
  "synthesis_id": "<opaque identifier>",
  "endpoint": { "..." : "..." },
  "recipe_lineage": {
    "recipe_name": "<recipe identifier>",
    "recipe_version": "<version string>"
  }
}
~~~

The top-level `method` and `path` fields name the synthesized
endpoint and **MUST** match the proposed pair. The
`synthesis_id` is an opaque identifier the server uses to
distinguish this contract from other contracts; clients
**SHOULD** record it for use with `INSPECT target=contract`
and the RCNS revocation surface. The `recipe_lineage` field,
when present, records the recipe that produced the synthesis
and the recipe version captured at synthesis time; this
disambiguates contracts produced under different versions of
the same recipe.

## Recipe Versioning {#recipe-versioning}

A server's contract synthesis machinery typically composes
synthesized endpoints from operator-authored recipes (composition
rules that name a set of underlying primitives and how to invoke
them). Recipes evolve: operators edit them to fix bugs, support
new primitives, or refine semantic claims. To prevent recipe
edits from silently mutating running contracts, recipes carry a
`version` identifier and synthesis captures the version
in effect at the moment the contract was produced.

The `version` field is operator-defined; servers **MUST** treat
the value opaquely. The recommended convention is a monotonically
increasing string (semver, integer counter, or timestamp); the
specific format leaves wire interoperability unaffected.

When a server synthesizes a contract from a recipe:

1. The server **MUST** record the recipe's current `version` on
   the resulting contract record at synthesis time.
2. The server **MUST** present the captured version in the
   `recipe_lineage.recipe_version` field of the 263 response
   body ({{propose-body}}) and in any subsequent
   `INSPECT target=contract` response naming this contract.
3. The synthesized contract **MUST** continue to execute under
   the captured recipe version for the contract's lifetime,
   even if the operator subsequently edits the recipe to a new
   version.
4. A subsequent PROPOSE for the same `(method, path)` pair
   **MAY** produce a new contract bound to the new recipe
   version; the new contract carries a distinct `synthesis_id`
   and the prior contract continues to operate under its
   captured version until released or revoked.

The recipe-version capture rule decouples contract stability
from recipe evolution. Operators who edit a recipe never
disturb in-flight callers; callers who need the updated
behavior reproduce the proposal under the new recipe version.
Operators that want to force callers off an old recipe version
use the contract revocation surface specified in
{{rcns-overview}} (revoking specific `synthesis_id` values)
rather than mutating recipes in place.

## Server-Side Synthesis Evaluation {#server-side-synthesis-evaluation}

On receipt of a PROPOSE request, the server **MUST** evaluate:

1. **Method conformance.** Is the proposed method in the AGTP-METHODS
   approved method catalog, or is it a custom method declared in
   the server's `policies.methods` ({{AGTP-METHODS}})? Otherwise,
   return 459 Method Violation.

2. **Path conformance.** Does the proposed path satisfy
   the path grammar in {{AGTP-METHODS}}? Otherwise, return 460 Endpoint Violation.

3. **Semantic block conformance.** Does the proposed semantic
   block carry all required fields with valid values? Otherwise,
   return 400 Bad Request.

4. **Schema conformance.** Are the input and output schemas valid
   JSON Schema documents? Otherwise, return 400 Bad Request.

5. **Authority sufficiency.** Does the proposing agent's
   declared Authority-Scope cover the proposed authority
   requirements? If the agent's identity document grants them but
   the request left them unclaimed, return 262 Authorization
   Required with reason `scope-required`; if the document omits
   them, return 455 Scope Violation with reason `scope-exceeded`
   ({{AGTP}}).

6. **Capability envelope.** Can the server actually synthesize
   the proposed endpoint from its existing capabilities? Otherwise,
   return 463 Proposal Rejected with `reason: composition-impossible`.

7. **Policy acceptance.** Does the server's policy permit the
   proposed method-and-path combination? Otherwise, return 463 with
   `reason: policy-refused`.

If all evaluations pass, the server synthesizes the endpoint and
returns 263 Proposal Approved.

## Synthesis-Composed Endpoints

When a server synthesizes an endpoint, the server is composing the
proposed behavior from existing endpoints internally. The synthesized
endpoint is exposed to the agent as a single endpoint, but the
server's handler implementation invokes one or more underlying
endpoints to fulfill the request.

Authority **MUST** be preserved through the composition. The agent's
Authority-Scope **MUST** be sufficient for every underlying endpoint
the synthesis invokes. If a composed step would require authority
the agent lacks, the synthesis **MUST** fail at evaluation
time (step 5) rather than at execution time.

## Session-Scoped vs Persistent Synthesis

By default, a synthesized endpoint is session-scoped: it exists for
the duration of the AGTP session that proposed it and is removed when
the session ends. The agent **MAY** request persistent synthesis by
setting `persistent: true` in the PROPOSE body; the server **MAY**
honor or refuse the persistence request independently of the
synthesis decision. Persistent synthesis subjects the proposed
endpoint to the server's regular endpoint catalog and is reflected
in subsequent manifest retrievals.

# Inventory Surfaces {#rcns-surfaces}

RCNS adds two built-in DISCOVER endpoints to the reserved set
specified in {{AGTP-METHODS}}. Both are Tier A: servers that
enable RCNS **MUST** expose them with the response contracts
below.

## `DISCOVER /patterns` {#builtin-patterns}

Servers **SHOULD** expose `DISCOVER /patterns`. The endpoint
returns the inventory of synthesis patterns the server will
negotiate against under RCNS: the operator-authored
recipes ({{recipe-versioning}}) and the `(method, path)`
shapes those recipes match. The endpoint is reachable even
when RCNS is disabled ({{rcns-config}}); when RCNS is
disabled, the response **SHOULD** indicate the disabled
state so callers can distinguish "no patterns exist" from
"patterns exist but synthesis is off."

~~~json
{
  "rcns_enabled": true,
  "patterns": [
    {
      "recipe_name": "book-room",
      "recipe_version": "3",
      "method": "BOOK",
      "path_exact": "/room",
      "description": "Books a room for the named guest at the named property."
    }
  ]
}
~~~

The response **MUST** be a JSON object with `rcns_enabled`
(boolean) and `patterns` (array). Each pattern entry
**MUST** carry `recipe_name`, `recipe_version`, and
`method`, plus exactly one of `path_exact` (byte-exact
match) or `path_regex` (regular expression match against
the request path). Servers **MAY** include additional
descriptive fields (`description`, `required_scopes`,
`semantic`) at their discretion; callers **MUST** ignore
unknown fields.

The endpoint surfaces server posture explicitly: an agent
inspecting `DISCOVER /patterns` knows in advance what
`(method, path)` shapes the server will consider for
synthesis.

## `DISCOVER /contracts` {#builtin-contracts}

Servers that have ever synthesized at least one contract
**MUST** expose `DISCOVER /contracts`. The endpoint returns
active synthesized contracts, scoped by default to the
requesting agent. Callers with the `inspect:all`
Authority-Scope token receive contracts across all agents
(operator visibility).

~~~json
[
  {
    "synthesis_id": "<opaque>",
    "method": "BOOK",
    "path": "/room",
    "originating_agent_id": "<64 hex>",
    "recipe_lineage": {
      "recipe_name": "book-room",
      "recipe_version": "3"
    },
    "contract_hash": "<64 hex>",
    "negotiation_origin": "rcns-gate",
    "issued_at": "<RFC 3339>",
    "expires_at": "<RFC 3339>"
  }
]
~~~

The response is a JSON array. Each entry **MUST** carry
`synthesis_id`, `method`, `path`, `originating_agent_id`,
`recipe_lineage`, `contract_hash`, and `negotiation_origin`
matching the Attribution-Record extension fields specified
in {{rcns-attribution-fields}}. The `issued_at` and
`expires_at` fields **SHOULD** be present.

A server **MUST NOT** return contracts whose
`originating_agent_id` differs from the requesting
Agent-ID, unless the request carries the `inspect:all`
scope.

# Status Codes

RCNS uses the following codes from the AGTP status code space.
The codes are registered in the AGTP Status Code Registry per
{{AGTP}}; 262 Authorization Required is specified in
{{AGTP-METHODS}} and listed here because the four-lock gate
returns it.

| Code | Name | Returned When |
|---|---|---|
| 261 | Negotiation In Progress | Server has accepted PROPOSE for evaluation; agent polls via `QUERY /proposals/{proposal_id}` for terminal status |
| 262 | Authorization Required | Caller scope lock closed with the token granted but unclaimed: reason `scope-required scope: rcns:negotiate`. When the identity document omits the token the lock returns 455 Scope Violation per {{AGTP}} (see {{AGTP-METHODS}} for the general semantics) |
| 263 | Proposal Approved | Server has synthesized and instantiated the proposed endpoint. Response body carries top-level `method` and `path` fields naming the synthesized endpoint, plus the full endpoint definition |
| 461 | RCNS Contract Available | Synthesis preview returned in confirm-first mode. Response body carries a contract preview with `synthesis_id`, resolved `(method, path)`, and the recipe lineage. Caller accepts by re-issuing with the `Contract-Synthesized` header |
| 463 | Proposal Rejected | Server cannot or declines to synthesize the proposed endpoint |
| 464 | RCNS No Contract | Synthesis attempted but no contract could be delivered. Response body carries one of the reasons in the RCNS vocabulary per {{AGTP}} |
{: title="RCNS Status Codes"}

The structural rejection codes 404, 405, 459, and 460 that a
server returns when RCNS is disabled or the gate is closed are
specified in {{AGTP-METHODS}}.

# Security Considerations

The security considerations of {{AGTP}} and {{AGTP-METHODS}}
apply. This section adds the synthesis-specific consideration.

## Synthesis Authority Preservation

The synthesis evaluation step (step 5 in {{server-side-synthesis-evaluation}})
**MUST** verify that authority is preserved through composition. A
synthesis that elevates authority by composing endpoints with
sufficient individual scopes into a behavior the agent is barred
from is a known threat. Servers **MUST** reject syntheses
where the composed behavior exceeds what the agent's Authority-Scope
permits, regardless of whether the individual steps are within
scope.

# IANA Considerations {#iana-considerations}

This document is the authoritative reference for the following
entries in the AGTP registries maintained per {{AGTP}}. Status
codes 461 and 464 and the RCNS reason vocabulary are registered
by {{AGTP}} itself.

## Status Code Assignments

- 261 Negotiation In Progress
- 263 Proposal Approved
- 463 Proposal Rejected

## Header Field Assignments

- `Allow-RCNS` (request)
- `Contract-Synthesized` (request and response)
- `Idempotency-Key` (request)

Semantics for each are specified in {{rcns-headers}}.

## Scope Token

- `rcns:negotiate`, the Authority-Scope token that opens lock 3 of
  the dispatcher gate ({{rcns-gate}}), for the Authority-Scope
  token vocabulary maintained per {{AGTP}}.

# Open Items

The following items are out of scope for this revision and are
anticipated in future revisions:

- Detailed schema for the `synthesis_log` returned with 263
  Proposal Approved, recording which underlying endpoints were
  composed.

- Federation model for multi-server synthesis (composing endpoints
  from different servers in a single synthesized endpoint).

- Hot reload of recipes without server restart. The stale contract
  sweep ({{stale-contract-sweep}}) is operator-fired by design
  because servers lack hot reload; a reload path would change the
  sweep trigger.

- The `on_policy_change` contract-invalidation behavior is
  specified ({{rcns-config}}) and remains unwired in the reference
  implementation.

- Whether the trust-tier lock should also drive the stale contract
  sweep (an agent whose trust tier falls below `min_trust_tier`
  after synthesis), as a second sweep dimension alongside recipe
  version drift.

# Acknowledgments

This document was developed in response to implementation
experience with the AGTP v07 through v09 builds, in which the
substrate was built in five phases (endpoint tiers, the dispatcher
gate, recipe versioning, contract scoping, and the observability
surfaces) before the text was consolidated. The architectural
decision to carry the contract in the protocol was articulated in
the AGTP-API Position Paper (May 2026).

--- back

# Changes from AGTP-API v01

This is the first revision under the AGTP-RCNS name. The text was
extracted from {{AGTP-API}} with the following edits:

- Cross-references to the tier taxonomy, custom methods, and path
  grammar now point to {{AGTP-METHODS}}.
- The overview paragraph that described the specification as in
  progress across revisions now describes this document.
- The `DISCOVER /patterns` and `DISCOVER /contracts` built-ins are
  carried here as the inventory surfaces; their reservation remains
  in the reserved DISCOVER path list of {{AGTP-METHODS}}.
- Status codes 261, 263, 463 and the RCNS headers moved here with
  their IANA entries; 262 stays with {{AGTP-METHODS}}.
- Terminology for Contract, Recipe, and Tier C Endpoint added.

No wire-visible behavior changes. A server conformant to the RCNS
portions of {{AGTP-API}} is conformant to this document.
