---
title: "AGTP-Commerce: Open Commerce Specification for Agent-to-Agent Transactions"
abbrev: "AGTP-COMMERCE"
docname: draft-hood-agtp-commerce-01
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent commerce
  - agent transactions
  - pricing
  - settlement
  - payment integration
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
    title: "AGTP Identifier Stack and Attribution-Record"
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
  AGTP-LOG:
    title: "AGTP Transparency Log Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-03
    date: 2026
  AGTP-IDDOC:
    title: "AGTP Agent Identity Document"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identity-document-00
    date: 2026
  RFC2119:
  RFC8174:
  RFC8949:
  ISO4217:
    title: "Currency codes -- ISO 4217"
    target: https://www.iso.org/iso-4217-currency-codes.html

informative:
  ISO20022:
    title: "Financial services -- Universal financial industry message scheme"
    target: https://www.iso20022.org
  AGTP-MERCHANT-IDENTITY:
    title: "AGTP Merchant Identity"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-merchant-identity-03
    date: 2026
  AGTP-LEI:
    title: "AGTP Binding to the Legal Entity Identifier (LEI and vLEI)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-lei-01
    date: 2026
  AGTP-PRESENCE:
    title: "AGTP Presence: Ambient Discovery and Visibility for Agents"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-presence-01
    date: 2026
  AGTP-DISCOVERY:
    title: "AGTP Discovery and Naming"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-discovery-02
    date: 2026
  AGTP-COMPOSITION:
    title: "AGTP Composition Profiles"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-composition-02
    date: 2026

--- abstract

This document specifies AGTP-Commerce, an open commerce specification
for agent-to-agent transactions. The specification defines the
structural transaction information that the Agent Transfer Protocol
carries between agents, allowing payment providers and commerce
applications to compose at the application layer to perform the actual
financial execution.

AGTP-Commerce is to agent commerce what ISO 20022 is to inter-bank
payment messaging: a standardized message format that allows
heterogeneous payment infrastructure to interoperate. AGTP-Commerce
moves no money, holds no funds, and operates as no payment processor.
The protocol carries pricing manifests, budget signaling, transaction
commitments, and audit trail records that payment providers consume
to execute settlement through existing financial infrastructure.

This document specifies the pricing manifest format, extended
Budget-Limit header semantics, the TRANSACT method for transaction
commitment, audit-trail-based receipt records, dispute composition
surfaces, and the integration patterns that allow payment providers
(traditional processors, banks, cryptocurrency rails, and specialized
agent-economy services) to compose with AGTP-Commerce as application-
layer infrastructure.

--- middle

# Introduction

Agents operating on the Agent Transfer Protocol need a way to exchange
value as part of their interactions. An agent providing a capability
may charge for it. An agent consuming a capability may have a budget
authorized by its principal. Without a structural mechanism for
exchanging this information at the protocol layer, every agent-to-agent
commerce interaction requires ad hoc integration between specific
parties.

AGTP-Commerce solves this by specifying the protocol-level structures
that agent transactions require: how prices are advertised, how
budgets are signaled, how transactions are committed, how audit trails
record the exchange, and how dispute evidence is preserved. These
structures are agent-native and substrate-aware, composing with the
existing AGTP specifications for identity, trust, certificates, and
logging.

What AGTP-Commerce deliberately leaves to others is the execution of
the financial transaction itself. Money movement remains the responsibility of
existing payment infrastructure operated by entities with the
appropriate licenses, regulatory standing, and operational
relationships. This separation of concerns is intentional and
structurally important: it positions AGTP-Commerce as a messaging
specification analogous to {{ISO20022}} rather than as a money
transmitter or payment processor.

The analogy is worth being explicit about. ISO 20022 is the global
standard for inter-bank payment messaging. SWIFT is migrating to it.
Banks have invested billions in adoption. ISO 20022 moves no
money. It specifies the structure of messages that banks exchange so
that payment infrastructure can interoperate. The actual money
movement happens at the bank layer through accounts, wire transfers,
settlement networks, and central bank infrastructure.

AGTP-Commerce occupies the same architectural position for agent
commerce. The specification carries the structural transaction
information that agent commerce requires. Payment providers (existing
processors, banks, specialized agent-economy services) consume this
information and execute settlement through whatever financial
infrastructure they operate.

This positioning has consequences worth naming. AGTP-Commerce falls
outside money-transmission regulation because it transmits no money.
It falls outside payment card industry compliance because it handles
no card data. It is no financial service because it provides none. It is a protocol
specification that financial services consume. This is the correct
architectural layer for a substrate-level specification and the
correct regulatory positioning for an open protocol.

## Conventions and Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

This document uses terminology from {{AGTP}}, {{AGTP-IDENTIFIERS}},
{{AGTP-TRUST}}, {{AGTP-CERT}}, and {{AGTP-LOG}}. Selected additional
terms specific to this document:

Pricing Manifest:
: A structured document advertising an agent's pricing, accepted
  settlement rails, refund policy, and related commerce metadata.
  Published by the agent and discoverable through DESCRIBE.

Settlement Endpoint:
: An identifier indicating where transaction settlement is routed.
  Refers to a payment provider relationship the agent operator has
  established independently. Carries no banking information.

Transaction Record:
: The structurally signed record of a commerce agreement between two
  agents. Created by the TRANSACT method, logged through AGTP-LOG.

Payment Provider:
: An application-layer service that consumes AGTP-Commerce transaction
  information and executes financial settlement through existing
  financial infrastructure. Examples include traditional payment
  processors, banking services, cryptocurrency rails, and specialized
  agent-economy payment services.

Owner Authorization:
: The cryptographic proof that an agent's owner (the principal recorded
  on the Agent Genesis) has authorized the spending envelope signaled
  in Budget-Limit headers. Carried as the `Intent-Assertion` header
  of {{AGTP-MERCHANT-IDENTITY}} ({{owner-authorization}}).

Dispute Window:
: The period during which either party to a transaction may challenge
  the transaction's completion or terms, declared in the pricing
  manifest at transaction time.

# Scope

## What This Document Specifies

This document specifies seven things:

1. The Pricing Manifest format used by agents to advertise pricing
   ({{pricing-manifest}}).

2. Extended Budget-Limit header semantics for signaling authorized
   spending envelopes ({{budget-signaling}}).

3. The TRANSACT method for committing to agent-to-agent transactions
   ({{transact-method}}).

4. The settlement endpoint declaration mechanism for indicating
   payment provider integration ({{settlement-endpoint}}).

5. The audit-trail-based receipt mechanism using AGTP-LOG transaction
   records ({{audit-receipt}}).

6. The dispute composition surface for application-layer dispute
   resolution ({{dispute-composition}}).

7. The negotiated tier mechanism for trusted-partnership pricing
   outside public manifests ({{negotiated-tiers}}).

## What This Document Leaves Unspecified

This document deliberately leaves unspecified:

- **Payment processing.** How a payment provider executes financial
  settlement through its infrastructure. This is the payment
  provider's internal operation.

- **Settlement timing.** When and how money actually moves between
  accounts. The pricing manifest declares expected timing; the
  payment provider enforces it.

- **Dispute resolution mechanics.** What evidence AGTP-Commerce
  carries is specified. How dispute resolution services consume that
  evidence and reach resolution is application-layer.

- **Currency conversion.** Multi-currency settlement happens at the
  payment provider layer.

- **Tax reporting formats.** Tax authorities have their own reporting
  formats. AGTP-LOG transaction records provide evidence that
  tax-reporting services can consume.

- **KYC, AML, and financial compliance.** These are the
  responsibilities of regulated payment infrastructure. AGTP-Commerce
  carries structural records that compliance services use.

- **Cryptocurrency-specific mechanics.** Wallet addresses, token
  specifications, blockchain network identifiers, and crypto
  settlement details are out of scope. The settlement endpoint
  mechanism accommodates cryptocurrency rails through opaque
  identifiers without specifying internal formats.

This narrow scope is what makes AGTP-Commerce defensible as a
protocol specification. The document specifies messaging structures
and leaves payment infrastructure operation unspecified.

# Pricing Manifest {#pricing-manifest}

## Manifest Document

An agent advertising commerce capability publishes a Pricing Manifest
as a structured document. The manifest is discoverable through the
DESCRIBE method per {{AGTP}} or via dedicated discovery paths per
{{AGTP-DISCOVERY}}. The manifest media type is
`application/agtp-pricing+json`.

The manifest declares the agent's pricing tiers, accepted settlement
rails, refund policy, dispute window, trust threshold requirements,
and related commerce metadata. The manifest is a static document
updated at the agent operator's discretion. It is fixed rather than
negotiated at request time; negotiated tiers are addressed separately in
{{negotiated-tiers}}.

## Manifest Schema

A minimal Pricing Manifest contains:

~~~ json
{
  "spec_version": "1.0",
  "agent_id": "agtp://agents.acme.com/concierge",
  "owner_id": "lei:5493001KJTIIGC8Y1R12",
  "currency": "USD",
  "tiers": [
    {
      "tier_name": "free",
      "price": "0.00",
      "scope": "discovery:query inquiry:basic",
      "trust_threshold": null
    },
    {
      "tier_name": "standard",
      "price": "0.05",
      "unit": "per-call",
      "scope": "booking:read booking:write",
      "trust_threshold": 0.6
    },
    {
      "tier_name": "premium",
      "price": "0.25",
      "unit": "per-call",
      "scope": "booking:read booking:write booking:expedited",
      "trust_threshold": 0.85
    }
  ],
  "settlement_endpoints": [
    "settlement:stripe-merchant-2891",
    "settlement:ach-routing-acme"
  ],
  "settlement_timing": "net-7",
  "refund_policy": "audit-verified",
  "dispute_window": "P7D",
  "last_updated": "2026-06-27T00:00:00Z",
  "signature": "..."
}
~~~

### Required Fields

spec_version:
: **REQUIRED**. Version of the AGTP-Commerce pricing manifest schema.

agent_id:
: **REQUIRED**. The Canonical Agent-ID of the agent publishing the
  manifest, per {{AGTP-IDENTIFIERS}}.

owner_id:
: **REQUIRED**. The Owner-ID of the agent's principal. **MAY** carry
  an LEI per {{AGTP-LEI}} where institutional identity applies.

currency:
: **REQUIRED**. The ISO 4217 currency code per {{ISO4217}} for all
  prices in this manifest. Implementations supporting cryptocurrency
  or other non-ISO-4217 denominations **MAY** use opaque settlement
  endpoint identifiers in addition to or in place of an ISO currency.

tiers:
: **REQUIRED**. An array of pricing tier objects defining the agent's
  available capabilities and their prices.

settlement_endpoints:
: **REQUIRED**. An array of settlement endpoint identifiers indicating
  which payment provider integrations the agent accepts. At least one
  endpoint **MUST** be specified.

signature:
: **REQUIRED**. A JWS signature over the manifest content, signed by
  the agent's certificate per {{AGTP-CERT}}.

### Tier Object Fields

Each tier object contains:

tier_name:
: **REQUIRED**. The name of the tier (free, standard, premium, or
  operator-defined).

price:
: **REQUIRED**. The price for this tier in the manifest's declared
  currency. Use "0.00" for free tiers.

unit:
: **REQUIRED** for non-free tiers. The unit of pricing: `per-call`,
  `per-token`, `per-second`, `per-month`, `per-task`, or an
  operator-defined unit.

scope:
: **REQUIRED**. The Authority-Scope values the tier grants access to.
  Callers invoking methods within this scope at this tier pay the
  declared price.

trust_threshold:
: **OPTIONAL**. A minimum trust score the calling agent **MUST** carry
  to access this tier. Trust score format per {{AGTP-TRUST}}. If
  omitted, no trust threshold is enforced for this tier.

minimum_commit:
: **OPTIONAL**. A minimum commitment volume for this tier (e.g.,
  monthly minimum, prepaid block).

availability:
: **OPTIONAL**. Time-of-day, day-of-week, or other availability
  restrictions for this tier.

### Optional Manifest Fields

settlement_timing:
: **OPTIONAL**. Expected settlement timing. Values: `immediate`,
  `net-N` (where N is days), `audit-verified` (settlement after audit
  trail verification), `escrow-released` (settlement after escrow
  release). Default: `audit-verified`.

refund_policy:
: **OPTIONAL**. Refund policy declaration. Values: `non-refundable`,
  `time-limited-refundable` (with refund window in additional field),
  `audit-verified` (refund available if audit trail shows incomplete
  work), `satisfaction-guaranteed`. Default: `non-refundable`.

dispute_window:
: **OPTIONAL**. The duration during which either party may dispute
  the transaction. Expressed in ISO 8601 duration format. Default:
  `P7D` (7 days).

partnership_tier:
: **OPTIONAL**. Indicates a tier is available only through
  pre-established partnership rather than public advertisement. See
  {{negotiated-tiers}}.

## Manifest Discovery

Agents publish their Pricing Manifest at substrate-native paths
discoverable through standard AGTP mechanisms. Recommended paths:

| URI Pattern | Content |
|-------------|---------|
| `agtp://<authority>/pricing` | The agent's pricing manifest |
| `agtp://<authority>/pricing/<tier>` | A specific tier's details |

The manifest **MAY** also be referenced from the agent's Identity
Document ({{AGTP-IDDOC}}, optional field `pricing_manifest_uri`),
from its AGTP-CERT, or returned in DESCRIBE responses. The
recommended discovery flow:

1. The requesting agent invokes DESCRIBE on the target agent.
2. The DESCRIBE response includes a `pricing-manifest-uri` field
   pointing to the manifest location.
3. The requesting agent fetches the manifest.
4. The requesting agent evaluates whether the manifest's tiers,
   trust thresholds, and settlement endpoints match its needs.

### Indexing by the Agent Name Service {#ans-index}

Search by price happens before DESCRIBE: the DISCOVER method of
{{AGTP-DISCOVERY}} accepts `max_cost` and `cost_unit`, and a search
that honors them needs a price per candidate without fetching every
manifest. The Agent Name Service of {{AGTP-DISCOVERY}} therefore
**MAY** index, for each registered agent that publishes a manifest,
the manifest URI and the per-capability price of its public tiers,
refreshed on each manifest update. The index is advisory: the signed
manifest is the authoritative price, a DISCOVER result filtered by
`max_cost` is a candidate rather than a quote, and a requesting agent
**MUST** fetch and verify the manifest before relying on a price. A
manifest whose fetched price exceeds the indexed price is stale on
the ANS side; the requesting agent treats the candidate as outside
its budget and **SHOULD** report the discrepancy to the ANS server.
Negotiated tiers ({{negotiated-tiers}}) are never indexed.

## Manifest Updates

Pricing Manifests **MAY** be updated by the publishing agent. Each
update **MUST**:

- Increment a version or timestamp field
- Be re-signed with the agent's current certificate
- Apply only to transactions initiated after the update timestamp

Transactions in progress at the time of a manifest update **MUST**
complete under the manifest version in effect at transaction commitment.

# Budget Signaling {#budget-signaling}

## The Budget-Limit Header {#budget-limit}

{{AGTP}} names `Budget-Limit` among its request headers and places
its specification in this document. `Budget-Limit` is an
**OPTIONAL** request header declaring the maximum resource
consumption the principal authorizes for one method invocation, as
a list of `unit=value` members whose units are drawn from the AGTP
Budget Unit Registry of {{AGTP}}:

~~~
Budget-Limit = budget-member *( OWS "," OWS budget-member )
budget-member = unit "=" value
unit          = 1*( ALPHA / DIGIT / "-" )   ; registered unit name
value         = 1*DIGIT [ "." 1*DIGIT ]     ; per the unit's format
~~~

~~~
Budget-Limit: USD=10.00, ttl=3600
Budget-Limit: tokens=5000, compute-seconds=120, USD=10.00, ttl=3600
~~~

A financial member names a registered currency unit (ISO 4217 code
per {{ISO4217}}; `USD`, `EUR`, and `GBP` are the initial
registrations) and carries a decimal value at the unit's precision.
The `ttl` member, in seconds from the request's `Request-Signature`
timestamp, bounds the budget's lifetime and is **RECOMMENDED** on
every commerce request; a request whose `ttl` has elapsed when the
server evaluates it is refused with 462 and reason `budget-expired`.
Version 00 of this document carried the commerce envelope as a JSON
object in this header; that form is withdrawn so that one grammar
serves base AGTP and this document.

Two status codes attach to the header and are distinct. `456 Budget
Exceeded` ({{AGTP}}) is returned when executing the method would
exceed the declared limit, a fact the server learns during or before
execution. `462 Insufficient Budget` (this document) is returned
when the envelope is insufficient for the receiving agent's price or
settlement requirements before any execution, a fact the server
learns by comparing the header with its pricing manifest.

## Owner Authorization of the Envelope {#owner-authorization}

The signed authorization of a spending envelope by the agent's owner
or principal is carried in the `Intent-Assertion` header of
{{AGTP-MERCHANT-IDENTITY}}, a detached JWT whose `amount_ceiling`
claim is the authorized maximum and whose `agent_id` names the
spending agent. Version 00 of this document carried a separate
`owner_auth` signature inside `Budget-Limit`; the Intent Assertion
already serves that purpose, and one signed envelope is better than
two. This document adds one optional claim to the Intent Assertion:

settlement_method:
: The settlement endpoint identifier ({{settlement-endpoint}}) the
  principal authorizes for this envelope. When present, the request's
  `settlement_method` parameter **MUST** equal it.

The requesting agent carries the settlement method it is prepared to
use as the `settlement_method` parameter of the method body (PURCHASE
or the request that precedes TRANSACT). It **MUST** match one of the
target agent's declared settlement endpoints for a transaction to
proceed.

## Budget Validation

When an agent receives a commerce request carrying `Budget-Limit`,
it **MUST** validate:

1. The header parses per {{budget-limit}} and names a currency unit
   the receiving agent's manifest prices in, or one it can convert at
   a rate it states in the response.
2. The `ttl` has yet to elapse.
3. The financial member is sufficient for the price of the requested
   service at the requesting agent's eligible tier.
4. The `settlement_method` parameter matches at least one of the
   receiving agent's declared settlement endpoints.
5. When the receiving agent's policy requires owner authorization for
   the operation class or amount, an `Intent-Assertion` is present,
   verifies per {{AGTP-MERCHANT-IDENTITY}}, names the requesting
   Agent-ID, and carries an `amount_ceiling` at or above the
   financial member.

If validation succeeds, the receiving agent **MAY** proceed with the
transaction commitment per {{transact-method}}. If validation fails,
the receiving agent **MUST** respond with 462 Insufficient Budget and
a reason from `budget-unparseable`, `budget-expired`,
`price-exceeds-budget`, `settlement-unsupported`,
`owner-authorization-required`, or `owner-authorization-invalid`,
optionally including the actual price and acceptable settlement
methods in the response body.

# Transaction Commitment {#transact-method}

## The TRANSACT Method

TRANSACT is a new AGTP method that creates the structural commitment
to an agent-to-agent transaction. The method is invoked by the
receiving agent (the seller) after a budget-validated request from the
calling agent (the buyer). TRANSACT produces a Transaction Record
signed by both parties.

Category: TRANSACT. Idempotent: No (each invocation creates a new
record). Required Authority-Scope: `commerce:transact`.

The TRANSACT method establishes:

- The agreed price and pricing tier
- The work to be performed (referenced via the original request)
- The agreed settlement endpoint and timing
- The dispute window
- Both parties' signatures

## Transaction Record Structure

A Transaction Record contains:

~~~ json
{
  "transaction_id": "txn-2026-06-27-abc123",
  "buyer": {
    "agent_id": "agtp://agents.bigco.com/orchestrator",
    "owner_id": "lei:5299002IZHCT9HFFP822",
    "principal_id": "principal-cfo-bigco",
    "signature": "<buyer JWS signature>"
  },
  "seller": {
    "agent_id": "agtp://agents.acme.com/concierge",
    "owner_id": "lei:5493001KJTIIGC8Y1R12",
    "signature": "<seller JWS signature>"
  },
  "price": {
    "amount": "0.25",
    "currency": "USD",
    "tier": "premium"
  },
  "work_reference": {
    "method": "BOOK",
    "request_id": "req-2026-06-27-xyz789",
    "scope": "booking:expedited"
  },
  "settlement": {
    "endpoint": "settlement:stripe-merchant-2891",
    "timing": "audit-verified"
  },
  "dispute_window": {
    "expires_at": "2026-07-04T00:00:00Z"
  },
  "committed_at": "2026-06-27T14:30:00Z",
  "log_entry_ref": "agtp-log://logs.acme.com/2026/06/27/txn-abc123"
}
~~~

## Commitment Workflow

The transaction commitment workflow proceeds as follows:

1. The buyer invokes a method (BOOK, EXECUTE, or similar) carrying a
   valid Budget-Limit header per {{budget-signaling}}.

2. The seller validates the budget signaling and confirms the request
   is eligible at a specific pricing tier per its manifest.

3. The seller invokes TRANSACT, generating a draft Transaction Record
   with the seller's signature.

4. The buyer receives the draft Transaction Record, validates it
   against the original request and pricing manifest, and adds the
   buyer signature.

5. The completed Transaction Record is logged through AGTP-LOG per
   {{audit-receipt}}.

6. The seller proceeds with the requested work.

7. Settlement occurs through the payment provider integration per
   the declared `settlement` field.

## Cancellation Before Commitment

Between the seller's draft Transaction Record signature and the
buyer's countersignature, either party **MAY** cancel by sending a
TRANSACT cancellation. Cancellation **MUST** be logged through
AGTP-LOG with the cancellation reason. No settlement occurs for
canceled transactions.

# Settlement Endpoint {#settlement-endpoint}

## Settlement Endpoint Identifiers

The settlement endpoint mechanism allows agents to declare which
payment provider integrations they accept without disclosing banking
information at the protocol layer. Settlement endpoints are opaque
identifiers that refer to the agent operator's payment provider
relationships, which are established independently of AGTP.

The recommended identifier format:

~~~
settlement:<provider-id>-<account-reference>
~~~

Examples:

- `settlement:stripe-merchant-2891` (Stripe merchant account)
- `settlement:paypal-merchant-acme` (PayPal merchant account)
- `settlement:ach-routing-021000021-account-acme` (ACH routing)
- `settlement:wire-swift-CHASUS33-acme` (wire transfer)
- `settlement:crypto-eth-0x742d...` (Ethereum wallet)
- `settlement:agent-economy-XYZ-account-acme` (specialized provider)

The actual format of the account-reference portion is provider-
specific. AGTP-Commerce leaves provider-specific formats
unstandardized; it standardizes the use of opaque identifiers as the
integration surface.

## Payment Provider Integration

Payment providers integrate with AGTP-Commerce by:

1. Operating their existing payment infrastructure under their own
   regulatory framework.

2. Providing agent operators with a mechanism to register and obtain
   a settlement endpoint identifier.

3. Consuming Transaction Records from AGTP-LOG (or from the agent
   operator's local log access) to determine when settlement should
   occur.

4. Executing financial settlement through their existing infrastructure
   using the Transaction Record as the supporting documentation.

5. Reporting settlement completion back to the agent operator and
   optionally to AGTP-LOG for transaction lifecycle completion.

The payment provider's internal operations remain outside the scope
of AGTP-Commerce. The integration surface is the Transaction Record
format and the settlement endpoint identifier.

## The Agent-Economy Payment Provider Opportunity

AGTP-Commerce creates an architectural slot for specialized
agent-economy payment providers. While traditional payment processors
(Stripe, PayPal, banking merchant services) can integrate with
AGTP-Commerce immediately, the agent economy has characteristics that
specialized providers are positioned to serve better:

- **Aggregated micro-billing.** Agents may transact in small amounts
  frequently. Aggregating thousands of $0.001 transactions into
  periodic settlements reduces per-transaction overhead.

- **Multi-currency agent settlement.** Agents may transact in various
  denominations including cryptocurrency. Specialized providers can
  handle conversion to operator-preferred currency.

- **Agent-aware dispute resolution.** Agent transactions have
  characteristics (machine-readable audit trails, deterministic
  verification of work completion, signed attestations) that
  traditional dispute processes were never built around.

- **Agent-specific compliance.** Tax reporting, AML compliance, and
  KYC for agent operators is domain-specific work.

This document explicitly invites both incumbent payment infrastructure
and new entrants to operate as AGTP-Commerce payment providers.
Vendor-neutrality is preserved throughout; the specification favors
no provider category.

# Audit Trail as Receipt {#audit-receipt}

## Transaction Records in AGTP-LOG

Every committed Transaction Record **MUST** be committed to AGTP-LOG
as a `commerce-transaction-committed` statement ({{log-entry}}). The
statement carries the record's digest and the parties rather than
the record itself, since {{AGTP-LOG}} statements are public; the
full record is held by both parties and disclosed to the payment
provider or a dispute service against the committed digest. The log
entry serves multiple purposes:

- **Receipt for buyer.** Proof the transaction was committed at the
  agreed price for the agreed work.

- **Receipt for seller.** Proof the buyer authorized the transaction
  and committed to settlement.

- **Justification for settlement.** Payment providers consume the log
  entry as supporting documentation for settlement execution.

- **Evidence for disputes.** If either party disputes the transaction,
  the AGTP-LOG entry is the authoritative record of what was agreed.

- **Audit record for compliance.** Tax authorities, regulators, and
  audit functions consume the log entry as transaction evidence.

## Log Entry Format {#log-entry}

This document registers the `commerce-transaction-committed` event
type in the AGTP-LOG event-type registry of {{AGTP-LOG}}. The
`agtp-subject` is the seller's Canonical Agent-ID (the agent that
invoked TRANSACT); the payload is a CBOR map, deterministically
encoded per {{RFC8949}}:

~~~
{
  "transaction-id": <text>,
  "record-hash": <bytes>,                ; SHA-256 of the canonical
                                         ; Transaction Record
  "buyer": <bytes>,                      ; buyer Canonical Agent-ID
  "seller": <bytes>,                     ; seller Canonical Agent-ID
  "committed-at": <text>,                ; RFC 3339
  "settlement-timing": <text>,           ; immediate, net-N, ...
  "log-position": <uint>,
  "previous-tree-size": <uint>
}
~~~

The amount, the tier, and the work reference stay in the Transaction
Record and off the public log. The log operator signs the statement
and the usual AGTP-LOG metadata applies:

- Entry timestamp
- Log operator signature
- Optional SCITT cross-witnessing per {{AGTP-LOG}}

The cryptographic guarantees of AGTP-LOG (append-only, signed by log
operator, optionally cross-witnessed) apply to commerce transactions
the same way they apply to other AGTP operations.

## Work Completion Records

Transaction Records capture the commitment to perform work. Separate
log entries **MAY** record the completion of work. The completion
record references the original Transaction Record and includes:

- Completion timestamp
- Seller's attestation that work was performed
- Optional buyer's attestation that work was received satisfactorily
- References to any artifacts or deliverables produced

Settlement timing of `audit-verified` requires a work completion
record before payment provider execution. Settlement timing of
`immediate` or `net-N` requires no explicit completion
attestation.

# Dispute Composition {#dispute-composition}

## Evidence AGTP-Commerce Carries

When a transaction is disputed, AGTP-Commerce carries evidence that
dispute resolution services consume:

- The signed Transaction Record from AGTP-LOG
- The original request and response messages
- Any work completion records and attestations
- Audit trail entries showing intermediate operations
- Trust score history of both parties per {{AGTP-TRUST}}

This evidence is cryptographically verifiable through standard AGTP
mechanisms (signature verification, log integrity, certificate chain
validation).

## Application-Layer Dispute Resolution

Actual dispute resolution mechanics happen at the application layer.
Dispute resolution services consume AGTP-Commerce evidence and apply
their own resolution processes. Examples of dispute resolution
service categories:

- **Payment provider dispute services.** Traditional payment processors
  have established dispute resolution processes that can be extended
  to agent transactions.

- **Specialized agent-economy arbitrators.** New entrants may emerge
  specializing in agent dispute resolution, leveraging AGTP-Commerce
  evidence formats.

- **Industry-specific arbitration.** Financial services arbitration,
  e-commerce dispute resolution, and similar industry-specific
  services compose with AGTP-Commerce for transactions in their
  domain.

- **Court-system integration.** For transactions warranting legal
  proceedings, AGTP-Commerce evidence is structured to support legal
  discovery and admission.

AGTP-Commerce specifies what evidence is available; dispute resolution
services choose how to use it. The protocol favors no
particular dispute resolution model.

# Negotiated Tiers {#negotiated-tiers}

## Beyond Public Pricing

Static pricing manifests cover the public commerce case where any
qualified agent can transact at advertised tiers. Some agent
relationships involve pre-established partnerships with negotiated
terms outside public pricing. AGTP-Commerce accommodates these
through negotiated tiers.

A negotiated tier is a pricing arrangement established between two
agents (or their operators) outside the public manifest. The arrangement
**MAY** include:

- Volume commitments and corresponding discounts
- Dedicated capacity allocations
- Specialized service tiers
- Custom dispute resolution arrangements
- Different settlement timing or methods

## Carrying Negotiated Tiers

Agents engaged in a negotiated tier relationship **MAY** signal this
through:

- A `partnership_tier` field in the requesting agent's request,
  referencing a pre-established agreement
- A separately signed partnership credential carried alongside the
  request
- An AGTP-CERT extension referencing a partnership agreement

Receiving agents validating a negotiated tier request **MUST** verify
the partnership credential against their records of agreed
partnerships. The TRANSACT workflow proceeds the same way as for
public pricing, with the partnership terms substituting for the
public manifest terms.

## Public vs. Negotiated Coexistence

An agent **MAY** simultaneously offer public pricing through its
manifest and negotiated tiers to specific partners. The two
mechanisms compose without conflict. Public pricing applies as the
default; negotiated tiers apply when the calling agent presents a
valid partnership credential.

# Composition with Existing AGTP Specifications

## AGTP Base

AGTP-Commerce specifies the `Budget-Limit` header that {{AGTP}} names
and assigns here ({{budget-limit}}), and adds the TRANSACT method. The
base specification's wire format, authentication, and identity
headers are used unchanged.

## AGTP-IDENTIFIERS

Transaction Records carry Agent-IDs, the owners resolved from their
Identity Documents, and per-request `principal_id` values as defined
in {{AGTP-IDENTIFIERS}}. The
`principal_id` is the per-request acting-principal identifier lifted
from external IdP credentials when a Pattern 2 External Identity
Provider composition is in effect per {{AGTP-COMPOSITION}}; it is
distinct from the agent's permanent Owner-ID. No new identifier
types are introduced.

## AGTP-TRUST

Trust thresholds in pricing manifests reference the trust score format
defined in {{AGTP-TRUST}}. Behavioral evidence from transaction
outcomes (completion rate, dispute frequency, payment timeliness)
provides input to trust scoring per the consumer behavior section of
{{AGTP-TRUST}}.

The credit-rating-analogous trust provider framing in {{AGTP-TRUST}}
applies to commerce: credit bureaus and specialized agent trust
providers may incorporate commerce behavior into trust scoring.

## AGTP-CERT

Pricing manifests **MAY** be referenced from AGTP-CERT extensions,
binding the pricing structure to the agent's cryptographic identity.
Settlement endpoint declarations **MAY** be carried as certificate
extensions for high-assurance applications.

## AGTP-LOG

Transaction Records are logged through {{AGTP-LOG}}. Commerce
operations inherit AGTP-LOG's cryptographic properties (append-only,
signed, optionally cross-witnessed). Settlement justification and
dispute evidence rely on AGTP-LOG integrity.

## AGTP-MERCHANT-IDENTITY

{{AGTP-MERCHANT-IDENTITY}} and AGTP-Commerce cover related but
distinct concerns and compose together rather than substituting for
one another. The relationship is intentional and load-bearing:

**AGTP-MERCHANT-IDENTITY** specifies the merchant-side identity model
for agents transacting through payment networks. It defines
Merchant-ID, Merchant Manifest Document, the
`Merchant-Manifest-Fingerprint` request header, and the merchant-
lifecycle states. It establishes the identity primitives that
payment networks (Visa, Mastercard, AMEX, and similar) consume to
recognize an agent as a known merchant. The PURCHASE method in
AGTP-MERCHANT-IDENTITY is the payment-network-mediated transaction
primitive backed by merchant identity verification (and is the
method that returns 458 Counterparty Unverified when merchant
identity fails).

**AGTP-Commerce** (this document) specifies the broader transaction
structure: pricing manifests, budget signaling, the TRANSACT method
for transaction commitment between any two AGTP agents, settlement
endpoint identification, audit-trail-based receipts, and dispute
composition surfaces. It requires no payment network mediation.
Agents transacting agent-to-agent without a payment network
intermediary use the TRANSACT method and the settlement endpoints
that the receiving agent's operator has independently configured.

The two compose: agents transacting through payment networks **MAY**
use both AGTP-MERCHANT-IDENTITY (for the payment-network-side
identity and PURCHASE flow) and AGTP-Commerce (for pricing manifests,
budget signaling, and audit-trail receipts). The PURCHASE method
defined in AGTP-MERCHANT-IDENTITY and the TRANSACT method defined in
this document are distinct primitives with distinct verification
chains; agents and payment infrastructure choose which is appropriate
based on whether a payment network is mediating the transaction.

For agents in a payment-network-mediated transaction, the receiving
agent's pricing manifest **MAY** declare `settlement_endpoints` that
reference the payment network's settlement infrastructure (e.g.,
`settlement:visa-acquirer-<id>`); the merchant identity primitives in
AGTP-MERCHANT-IDENTITY then provide the payment-network-side
identification for that settlement.

## AGTP-LEI

For agents deployed by LEI-holding legal entities, institutional
identity per {{AGTP-LEI}} provides verifiable institutional
identification for regulated settlement. Transaction Records carrying
LEI-anchored Owner-IDs enable settlement through infrastructure that
requires institutional verification.

## AGTP-PRESENCE

Pricing-aware discovery composes with the Presence layer in
{{AGTP-PRESENCE}}. Agents can be discovered with capability matching
plus pricing constraints in a single query. The capability partition
dimension in Presence overlays composes naturally with the scope-based
pricing tiers in commerce manifests.

## AGTP-DISCOVERY

The DISCOVER method per {{AGTP-DISCOVERY}} accepts `max_cost` and
`cost_unit` and returns candidates whose indexed price fits, per
{{ans-index}}; the manifest URI rides with each result's Identity
Document, and the manifest itself is fetched and verified before any
price is relied upon.
Anticipatory Discovery Services (ADS) introduced in
{{AGTP-DISCOVERY}} **MAY** preload pricing-aware predictions, expanding
agent awareness with commerce-eligible candidates.

# Security Considerations

## Manipulation Resistance

Pricing manifests **MUST** be signed by the publishing agent's
certificate. Verifiers **MUST** check the signature before accepting
manifest contents. Manifest tampering during transit is prevented by
signature verification.

Transaction Records require signatures from both buyer and seller.
Neither party can unilaterally modify a committed Transaction Record.

Where the receiving agent's policy requires it, a spending envelope
carries the owner's or principal's Intent Assertion. An agent cannot
forge an authorized envelope; the assertion is signed by the
principal and verifies per {{AGTP-MERCHANT-IDENTITY}}.

## Settlement Risk Acknowledgment

AGTP-Commerce leaves settlement risk in place. Payment providers
may fail to execute settlement for reasons including counterparty
insolvency, regulatory action, technical failure, or fraud. The
protocol carries the structural transaction record; settlement
execution depends on the payment provider relationship.

Agents and their operators **SHOULD** evaluate payment provider
counterparty risk independently of AGTP-Commerce protocol guarantees.
The protocol provides evidence that settlement was agreed and
guarantees nothing about whether settlement will occur.

## Dispute Evidence Integrity

The integrity of dispute evidence depends on AGTP-LOG integrity. If
log operators are compromised or collude with one party, dispute
evidence may be falsified. SCITT cross-witnessing per AGTP-LOG
**SHOULD** be used for high-value transactions to provide independent
verification of log entries.

## Owner Authorization Verification

The Intent Assertion ({{owner-authorization}}) is the load-bearing
mechanism for ensuring agents transact within authorized envelopes
where policy requires one. Receiving agents **MUST** verify it
before accepting such a transaction. Compromised owner keys can
authorize unauthorized transactions; owner key security is the
responsibility of the owner's identity infrastructure.

## Settlement Endpoint Verification

Settlement endpoint identifiers are opaque to AGTP-Commerce. An agent
cannot verify through the protocol that a settlement endpoint
identifier corresponds to a legitimate payment provider relationship.
Verification of settlement endpoint legitimacy is the responsibility
of the calling agent's risk evaluation, typically through
out-of-band relationships with the payment provider or through trust
provider services that maintain settlement endpoint reputation
information.

## Refund and Dispute Window Abuse

Pricing manifests declaring overly long dispute windows or aggressive
refund policies may attract abusive callers. Agents **SHOULD** monitor
dispute and refund rates and adjust manifest terms as needed. Trust
scoring per {{AGTP-TRUST}} **SHOULD** incorporate dispute and refund
behavior as input.

# Regulatory Positioning

AGTP-Commerce is a protocol specification for agent-to-agent
transaction information exchange. It is neither a money transmitter,
nor a payment processor, nor a financial service. Money movement
between agents occurs through payment provider infrastructure
operated by entities with appropriate regulatory standing.

Specifically:

- AGTP-Commerce holds no funds.
- AGTP-Commerce transfers no funds between accounts.
- AGTP-Commerce intermediates no financial relationships.
- AGTP-Commerce provides no credit, lending, or extension of
  payment.
- AGTP-Commerce handles no payment card data.

These functions remain with regulated payment infrastructure operated
by banks, payment processors, money transmitters, and similar
entities. AGTP-Commerce carries the structural transaction information
that this regulated infrastructure consumes.

This positioning is consistent with how protocol specifications
operate generally. HTTP is unregulated as a money transmitter even
though e-commerce runs over HTTP. ISO 20022 is unregulated as a
money transmitter even though inter-bank payment messaging uses it.
AGTP-Commerce occupies the same architectural and regulatory position
as these specifications.

# IANA Considerations

## TRANSACT Method Registration

The AGTP Method Registry of {{AGTP}} holds the method floor; methods
beyond it are registered through the AGTP-METHODS catalog. This
document requests registration of TRANSACT in that catalog, in the
`transaction` category, with this document as reference:

| Method | Category | Safe | Idempotent | Reference |
|--------|----------|------|------------|-----------|
| TRANSACT | transaction | No | No | This document |

## Budget-Limit Header

This document is the reference for the `Budget-Limit` request header
in the AGTP Header Field registry of {{AGTP}} ({{budget-limit}}), and
registers the reason tokens `budget-unparseable`, `budget-expired`,
`price-exceeds-budget`, `settlement-unsupported`,
`owner-authorization-required`, and `owner-authorization-invalid`
for status 462.

## AGTP-LOG Event Type

This document registers `commerce-transaction-committed` in the
AGTP-LOG event-type registry of {{AGTP-LOG}} ({{log-entry}}).

## Media Type Registration

This document requests registration of the following media type:

- Name: `agtp-pricing+json`
- Type name: `application`
- Subtype name: `agtp-pricing+json`
- Required parameters: none
- Optional parameters: none
- Encoding considerations: 8bit; JSON content is UTF-8 encoded
- Security considerations: See Security Considerations of this document
- Published specification: This document
- Person and email address to contact: Chris Hood <chris@nomotic.ai>
- Intended usage: COMMON
- Author: Chris Hood
- Change controller: IETF

## Authority-Scope Values

Registration of new Authority-Scope values:

| Scope | Permits | Reference |
|-------|---------|-----------|
| `commerce:transact` | Invoking the TRANSACT method | This document |
| `commerce:price-publish` | Publishing pricing manifests | This document |
| `commerce:dispute` | Filing transaction disputes | This document |

## Status Code

Registration of a new status code in the AGTP status code registry:

| Code | Meaning | Reference |
|------|---------|-----------|
| 462 | Insufficient Budget | This document |

Earlier drafts of this document used 458 for this condition; 458 is
Counterparty Unverified in {{AGTP}} and {{AGTP-MERCHANT-IDENTITY}},
and 462 is the value {{AGTP}} assigns here.

# Implementation Considerations

## Path to Payment Provider Integration

A payment provider considering AGTP-Commerce integration follows
roughly this path:

1. Define a settlement endpoint identifier scheme for accounts on
   the provider's platform.

2. Implement consumption of AGTP-LOG Transaction Records as
   settlement-triggering input.

3. Map Transaction Records to the provider's internal settlement
   workflows.

4. Provide agent operators with an onboarding flow that registers
   accounts and issues settlement endpoint identifiers.

5. Define dispute integration with AGTP-LOG evidence consumption.

6. Operate within existing regulatory framework.

The integration is structurally simple because AGTP-Commerce requires
no change to the payment provider's underlying operations.
Only the integration surface (Transaction Record consumption,
settlement endpoint issuance) is new.

## Path to Agent Operator Adoption

An agent operator considering AGTP-Commerce deployment follows
roughly this path:

1. Establish a payment provider relationship and obtain a settlement
   endpoint identifier.

2. Define pricing tiers appropriate to the agent's capabilities.

3. Publish a Pricing Manifest at the agent's substrate-native path.

4. Configure the agent's TRANSACT handling for incoming commerce
   requests.

5. Configure trust threshold evaluation for callers.

6. Configure dispute response handling.

The operator's banking, tax, and compliance relationships remain with
their payment provider. AGTP-Commerce provides the protocol-level
integration with the agent ecosystem.

# Engagement Invitation

This document is published as an Internet-Draft to invite engagement
from payment infrastructure operators, banks, payment networks,
cryptocurrency rail operators, specialized agent-economy service
entrepreneurs, and institutional adopters of agent infrastructure.

The specification is designed to compose with existing payment
infrastructure rather than to compete with it. Feedback from payment
processors on the integration surface, from banks on settlement
mechanics, from regulators on positioning, and from agent operators
on operational viability is welcome.

The agent economy is emerging. AGTP-Commerce provides the protocol-
level specification that allows that economy to develop on open,
vendor-neutral infrastructure rather than on proprietary platforms
controlled by single providers. This document is the architectural
contribution to that goal; the operational ecosystem develops through
the work of payment infrastructure operators, agent operators, and
institutional adopters who build on the specification.

--- back

# Acknowledgments

The author thanks Tom Roberts for the institutional framing that
positions AGTP-Commerce as analogous to ISO 20022 for agent commerce,
and for the conversations about how the agent economy might compose
with existing payment infrastructure. The author also acknowledges
the broader context of agent infrastructure work at the IETF and the
Linux Foundation that motivates the open-protocol approach to agent
commerce.

# Changes from v00

Version 01 aligns the document with draft-hood-independent-agtp-10
and the companions. The pricing manifest, TRANSACT, settlement
endpoints, and dispute composition are unchanged.

- **Budget-Limit specified here, in the base grammar**
  ({{budget-limit}}). Base AGTP v10 names the header and assigns its
  specification to this document. The v00 JSON-object form is
  withdrawn in favor of the `unit=value` list of the AGTP Budget Unit
  Registry; 456 and 462 distinguished.
- **Owner authorization moves to Intent-Assertion**
  ({{owner-authorization}}): the v00 `owner_auth` signature inside
  the header is replaced by the Intent Assertion of
  draft-hood-agtp-merchant-identity-03, with an optional
  `settlement_method` claim; `settlement_method` becomes a body
  parameter. Budget validation restated with 462 reason tokens.
- **ANS indexing of pricing** ({{ans-index}}), so that the `max_cost`
  and `cost_unit` DISCOVER parameters of draft-hood-agtp-discovery-02
  have a source; the signed manifest remains authoritative.
  `pricing_manifest_uri` named as an optional Identity Document
  field.
- **Transaction Records in AGTP-LOG** as a registered
  `commerce-transaction-committed` statement carrying the digest and
  parties rather than the full record, since the log is public.
- **TRANSACT registered through the AGTP-METHODS catalog** rather
  than the base method registry, which holds the floor only.
- **458 versus 462** noted.
- References made explicit and pinned: base -10, AGTP-IDENTIFIERS
  -03, AGTP-TRUST -03, AGTP-CERT -04, AGTP-LOG -03,
  AGTP-MERCHANT-IDENTITY -03, AGTP-LEI -01, AGTP-PRESENCE -01,
  AGTP-DISCOVERY -02; AGTP-IDDOC and AGTP-COMPOSITION added. House
  style pass.
