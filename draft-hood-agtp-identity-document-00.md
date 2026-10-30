---
title: "AGTP Agent Identity Document"
abbrev: "AGTP-IDDOC"
docname: draft-hood-agtp-identity-document-00
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent transfer protocol
  - agent identity
  - identity document
  - agent namespace

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
  RFC9162:
  RFC9943:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026

informative:
  AGTP-CERT:
    title: "AGTP Agent Certificate Extension"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-agent-cert-04
    date: 2026
  AGTP-IDENTIFIERS:
    title: "AGTP Identifier Chain"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-identifiers-03
    date: 2026
  AGTP-LOG:
    title: "AGTP Transparency Log Protocol"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-03
    date: 2026
  AGTP-MERCHANT:
    title: "AGTP Merchant Identity and Agentic Commerce Binding"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-merchant-identity-03
    date: 2026
  AGTP-TRUST:
    title: "AGTP Trust and Verification Specification"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-trust-03
    date: 2026

--- abstract

This document specifies the Agent Identity Document of the Agent
Transfer Protocol (AGTP) {{AGTP}}: the signed document every
canonical Agent-ID resolves to, carrying the agent's owner, declared
authority scope, trust posture, supported methods, and issuer
binding. It also specifies the Agent Namespace Document, the `.agtp`
package format and its integrity rules, the human-readable identity
view served to browsers, and the status sub-resource. The material
was carried in the base AGTP specification through version 09 and is
extracted here, unchanged in substance, so that the base
specification carries the transfer protocol and this document
carries the identity document format, in the way HTTP and HTML are
specified separately.

--- middle

# Introduction

AGTP {{AGTP}} identifies every agent by a canonical Agent-ID, the
SHA-256 hash of the agent's Agent Genesis. The Agent-ID is permanent
and content-addressed; everything mutable about the agent (where it
is hosted, what it is currently permitted to do, what it currently
supports, how it is currently trusted) lives in the Agent Identity
Document the Agent-ID resolves to. The base specification states what
the protocol depends on in that document; this document specifies the
document itself.

The Agent Identity Document is a credential, and the distinction from
a description matters. A description of an agent, of the kind the A2A
Agent Card provides at a well-known HTTPS path, is self-asserted,
unsigned, and trusted through the domain that serves it. The Agent
Identity Document is derived from a signed Agent Genesis, carries an
issuer binding, holds the lifecycle state and the scope set the
protocol enforces on every request, and is refused by a conformant
recipient unless its signature verifies. A description can be mapped
out of an Identity Document for interchange; an Identity Document
cannot be constructed from a description without an issuer. The
document taxonomy in {{AGTP}} places the two.

The split follows the HTTP precedent. HTTP specifies the exchange
and names the media types it carries; HTML is specified elsewhere.
AGTP specifies the exchange and names
`application/vnd.agtp.identity+json`; this document specifies it.

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}}
{{RFC8174}} when, and only when, they appear in all capitals, as
shown here.

Terms defined in {{AGTP}} carry their definitions here.

# Agent Namespace Document

## Purpose and Scope

The Agent Namespace Document is the index of all Active agents
registered under an organization's governance zone. It is returned
in response to a request targeting the `/agents` path:

~~~~
agtp://acme.tld/agents
agtp://agtp.acme.tld/agents
~~~~

The Agent Namespace Document is generated and cryptographically
signed by the governance platform each
time the registry changes. Any Namespace Document that fails signature
verification **MUST** be rejected by the requesting party.

## Document Schema

~~~~json
{
  "document_type": "agtp-namespace",
  "schema_version": "1.0",
  "org_domain": "acme.tld",
  "governance_zone": "zone:acme-internal",
  "generated_at": "2026-03-20T14:00:00Z",
  "signature": {
    "algorithm": "ES256",
    "key_id": "agtp-gov-key-acme-01",
    "value": "[base64-encoded-signature]"
  },
  "agents": [
    {
      "agent_label": "customer-service",
      "canonical_id": "3a9f2c1d8b7e4a6f...",
      "lifecycle_state": "Active",
      "trust_tier": 1,
      "cert_status": "Active",
      "manifest_uri": "agtp://agtp.acme.tld/agents/customer-service",
      "activated_at": "2026-01-15T09:00:00Z",
      "last_updated": "2026-03-01T11:30:00Z"
    }
  ],
  "total_active": 1,
  "namespace_cert_fingerprint": "b2c4d6e8..."
}
~~~~
{: title="Agent Namespace Document Schema"}

The `agents` array **MUST** include only agents in Active lifecycle
state. Suspended, Revoked, and Deprecated agents **MUST NOT** appear
in the Namespace Document.

## Integrity and Freshness

The Namespace Document **MUST** include a `generated_at` timestamp.
Implementations **SHOULD** treat Namespace Documents older than a
configurable freshness threshold (default: 300 seconds) as stale and
re-request. The governance platform **MUST** re-sign the Namespace
Document within 60 seconds of any registry change.

The signature covers the entire document including `generated_at`.
Replaying an older signed Namespace Document to conceal a revocation
event is a known attack vector; implementations **MUST** reject
Namespace Documents with a `generated_at` timestamp older than the
freshness threshold.

# Agent Identity Document and the .agtp Format {#identity-document}

## Purpose and Scope

The Agent Identity Document is the protocol's canonical representation
of a specific agent's identity, status, and behavioral scope. Prior
versions of this specification referred to this artifact as the Agent
Manifest Document; v07 renames it to the Agent Identity Document to
reflect its IANA-registered media type
(`application/vnd.agtp.identity+json`) and to clarify its role as the
identity primitive of the protocol. The Identity Document is returned
in response to any AGTP URI resolution request targeting a specific
agent:

~~~~
agtp://[canonical-agent-id]
agtp://acme.tld/agents/customer-service
agtp://acme.tld/agents/customer-service?format=json
~~~~

The Identity Document is derived from the embedded manifest inside the
agent's `.agent` or `.nomo` package; no separately modifiable file
exists. The governance platform **MUST** verify the
package integrity hash before extracting and serving the Identity
Document.

## The Three Document Formats and Their Relationship

~~~~
.agent   ──── open packaging format (patent pending, Chris Hood)
  │           Contains: manifest + integrity hash + behavioral
  │           trust score + executable content
  │
.nomo    ──── governed packaging format (patent pending, Chris Hood)
  │           Contains: .agent contents + CA-signed cert chain
  │           + governance zone binding
  │
  Both produce ↓ when queried via agtp:// URI
  │
.agtp    ──── wire-level Identity Document (this spec)
              Contains: identity fields, lifecycle state, trust
              tier, trust score, behavioral scope, methods,
              capabilities, verification anchors
              Open format defined by AGTP specification
              Content-Type: application/vnd.agtp.identity+json
~~~~
{: #format-relationship title="Relationship Between AGTP File Formats"}

The `.agtp` format is the protocol's own document type and the one
transmitted over the wire. The `.agent` and `.nomo` formats are
deployment artifacts. An AGTP implementation has no need to understand
`.agent` or `.nomo` packaging internals; it is only required to
produce and consume `.agtp` Identity Documents.

Additional packaging formats **MAY** be defined by third parties,
provided they can produce conformant `.agtp` Identity Documents when
queried. Such formats **MUST** implement the integrity verification
requirement: the Identity Document **MUST** be derived from a verified
package, never from an independently stored or editable file.

## Agent Identity Document Schema

The Agent Identity Document carries three categories of fields:
**REQUIRED** fields that every Identity Document **MUST** contain;
**RECOMMENDED** fields that **SHOULD** be present when the
corresponding information is available; and **CONDITIONAL** fields
that **MUST** be present when the conditions described later in this
section are met.

The following fields are **REQUIRED** in all Agent Identity Documents:

~~~~json
{
  "agtp_version": "0.7",
  "document_type": "agtp-identity",
  "document_version": "1.0",
  "agent_id": "3a9f2c1d8b7e4a6f0c2d5e9b1a3f7c0d...",
  "name": "customer-service",
  "description": "Handles customer service requests.",
  "principal": "Acme Corporation",
  "principal_id": "acme.tld",
  "issuer": "https://ca.acme.tld",
  "agent_public_key": "<base64url-encoded 32 bytes, Ed25519>",
  "issued_at": "2026-01-15T09:00:00Z",
  "updated_at": "2026-03-01T11:30:00Z",
  "status": "Active",
  "methods": [
    "QUERY", "DESCRIBE", "SUMMARIZE",
    "EXECUTE", "ESCALATE", "CONFIRM", "NOTIFY"
  ],
  "capabilities": [
    "customer-service:tier-1",
    "knowledge-base:read",
    "ticket-system:create"
  ],
  "scopes_accepted": [
    "documents:query",
    "knowledge:query",
    "calendar:book",
    "escalation:route"
  ],
  "trust_score": 0.94,
  "trust_score_computed_at": "2026-03-01T11:30:00Z"
}
~~~~
{: title="Agent Identity Document - Required Fields"}

Field semantics:

`agtp_version`:
: The version of the AGTP protocol the agent speaks. SemVer string.

`document_type`:
: Self-identification of the document. **MUST** be `agtp-identity`
  for documents conforming to this specification.

`document_version`:
: The schema version of the Identity Document itself. SemVer string.
  Distinct from `agtp_version`. Permits the document schema to evolve
  independently of the protocol.

`agent_id`:
: The 256-bit canonical Agent-ID, hex-encoded, derived from the
  agent's Agent Genesis hash. Authoritative in every AGTP protocol
  operation.

`name`:
: A human-readable label for the agent. Not unique across the AGTP
  ecosystem; uniqueness is provided by `agent_id`.

`description`:
: Short prose describing what the agent does.

`principal`:
: The human-readable name of the organization or individual that
  governs the agent.

`principal_id`:
: A stable, machine-readable identifier for the principal (typically
  the principal's primary DNS domain, or a canonical organization
  identifier).

`agent_public_key`:
: The agent's current Ed25519 signing key, 32 raw bytes base64url
  encoded. Every non-anonymous request the agent sends carries a
  `Request-Signature` verifiable under this key, and every delegation
  token the agent mints is signed with it ({{AGTP}}). The private half
  is held by the agent and never by the issuer. Rotation is a
  re-issuance of this document with a new value; the Agent-ID is
  unchanged. REQUIRED for documents issued under AGTP version 10 or
  later; a document without it is verifiable at Level 1 only, and
  requests from its agent are recorded as possession-unproven.

`issuer`:
: The URL of the issuer that signed the Identity Document. The
  issuer's public key is published at a well-known location under
  this URL; signature verification details are specified in
  {{AGTP-CERT}}.

`issued_at`:
: ISO 8601 timestamp at which the Identity Document was first issued
  (typically the moment of ACTIVATE).

`updated_at`:
: ISO 8601 timestamp at which the Identity Document was last updated.
  Promotes freshness checks. **MUST** be greater than or equal to
  `issued_at`.

`status`:
: The lifecycle state of the agent. One of: `active`, `suspended`,
  `retired`, `deprecated`. The state is mutated by the Lifecycle
  methods on the protocol floor (the method definitions in {{AGTP}}): ACTIVATE
  sets `active`, DEACTIVATE sets `suspended`, REINSTATE returns
  a `suspended` agent to `active`, REVOKE sets `retired`
  (permanent), DEPRECATE sets `deprecated`. Semantics:
  
  - `active`: accepts AGTP traffic normally.
  - `suspended`: refuses AGTP traffic with `503 Service Unavailable`; recoverable via REINSTATE.
  - `retired`: refuses AGTP traffic with `410 Gone`; permanent; canonical Agent-ID is never reissued.
  - `deprecated`: continues to accept AGTP traffic but signals planned end-of-life; clients **SHOULD** migrate.

`methods`:
: The set of AGTP methods the agent's server speaks. **MUST** include
  every method the server is prepared to accept. The protocol-defined
  eighteen-method floor (see the method definitions in {{AGTP}}) **MUST** be
  represented if the server claims AGTP conformance.

`capabilities`:
: Higher-level service descriptors of what the agent can do.
  Distinct from `methods`: methods are the protocol verbs the
  server speaks; capabilities are the application-level competencies
  the agent exposes through those methods.

`scopes_accepted`:
: The Authority-Scope tokens the agent will accept on inbound
  requests. Format and semantics defined in the Authority-Scope format in {{AGTP}}.

`trust_score`:
: A scalar value on the closed interval [0.0, 1.0] expressing
  behavioral trust assessed by the agent's principal or governance
  platform. Computation methodology, freshness requirements, and
  signature binding are specified in {{AGTP-TRUST}}. The base AGTP
  specification carries the field; AGTP-TRUST specifies how the value
  is produced and verified.

`trust_score_computed_at`:
: ISO 8601 UTC timestamp at which the issuer computed `trust_score`.
  **REQUIRED** whenever `trust_score` is present; a `trust_score`
  without it is treated as absent per {{AGTP-TRUST}}.

The following fields are **RECOMMENDED**:

~~~~json
{
  "trust_tier": 1,
  "verification_path": "dns-anchored",
  "owner_id": "nomotic.inc",
  "role": "agent",
  "org_domain": "acme.tld",
  "governance_zone": "zone:acme-internal",
  "cert_fingerprint": "b2c4d6e8...",
  "cert_status": "Active",
  "dns_anchor_record": "_agtp.acme.tld TXT agtp-zone=...",
  "log_inclusion_proof": null,
  "log_uri": null,
  "package_format": "nomo",
  "package_integrity_hash": "sha256:[hash]",
  "audit_log_uri": "agtp://agtp.acme.tld/audit/customer-service",
  "escalation_policy": "route-to-human-on-scope-limit",
  "delegation_permitted": false,
  "max_delegation_depth": 0,
  "manifest_issuer": "registrar.acme.tld",
  "manifest_issuer_public_key": "<base64url-encoded 32 bytes>",
  "manifest_signature": "<base64url-encoded Ed25519 signature>"
}
~~~~
{: title="Agent Identity Document - Recommended Fields"}

`trust_tier`:
: Tier classification per {{AGTP-TRUST}}. One of `1`, `2`,
  or `3`. Resolved at server load time per the precedence
  rule in {{AGTP-TRUST}}: explicit declaration in the
  Identity Document beats Genesis-derived fallback, which
  beats a conservative Tier 2 default.

`verification_path`:
: Verification path used at ACTIVATE time. One of
  `dns-anchored`, `log-anchored`, `hybrid`, or
  `org-asserted`. Resolved per the precedence rule in
  {{AGTP-TRUST}} alongside `trust_tier`. The first three
  values are Tier 1 paths; `org-asserted` is the Tier 2
  value. **MUST** match the `verification_path` field of
  the underlying Agent Genesis when one is loaded.
  Detailed semantics specified in {{AGTP-CERT}}.

`owner_id`:
: Identifier of the human or organizational principal
  accountable for this agent. Resolved per the precedence
  rule in {{AGTP-TRUST}}: explicit declaration beats
  Genesis-derived fallback. Omitted when no Agent Genesis
  is loaded and the operator set no explicit value.
  Semantics specified in {{AGTP-IDENTIFIERS}}; stamped on
  responses as the `Owner-ID` response header per
  the response headers in {{AGTP}}.

`role`:
: Capability role declared for this agent. One of `agent`
  (default) or `merchant`. The `merchant` value signals that
  the agent is a transactional counterparty addressable by
  PURCHASE and verified per {{AGTP-MERCHANT}}; `agent` is
  the default capability posture for all other agents.
  `role` is a capability attribute carried on the Agent
  Identity Document and absent from the Agent Genesis: capability
  may change over an agent's lifetime without changing
  identity. Operators **MAY** add or remove the `merchant`
  role without re-issuing the Agent Genesis or rotating
  the canonical Agent-ID. Future revisions **MAY** define
  additional role values. Implementations encountering an
  unknown role value **MUST** treat the agent as `role:
  agent` (the default) and log the unknown value for
  operator review.

`org_domain`:
: The DNS domain under which the agent is registered, when applicable.

`governance_zone`:
: The governance zone identifier for the agent's deployment context.

`cert_fingerprint`, `cert_status`:
: AGTP Agent Certificate metadata. Specified in {{AGTP-CERT}}.

`dns_anchor_record`:
: Populated when `verification_path` is `dns-anchored` or `hybrid`.

`log_inclusion_proof`:
: Populated when `verification_path` is `log-anchored`. Carries the
  transparency log inclusion proof per {{RFC9162}} or the COSE_Sign1
  receipt per {{RFC9943}}. Detailed semantics specified in
  {{AGTP-LOG}}.

`log_uri`:
: Populated when `verification_path` is `log-anchored`. Mirrors the
  Agent Genesis field of the same name: the absolute HTTPS URI of
  the transparency log at which `log_inclusion_proof` is verifiable
  per {{AGTP-LOG}}, carried here so that a verifier holding only the
  Identity Document can reach the log.

`package_format`, `package_integrity_hash`:
: Deployment-layer metadata identifying the package format and the
  hash that the Identity Document was extracted from.

`audit_log_uri`:
: AGTP URI of the agent's audit log resource.

`pricing_manifest_uri`:
: **OPTIONAL**. URI of the agent's signed pricing manifest per the
  AGTP commerce companion, for agents that sell services; the Agent
  Name Service may index it for price-bounded search.

`escalation_policy`, `delegation_permitted`, `max_delegation_depth`:
: Policy fields governing the agent's escalation and delegation
  behavior.

The following fields are **CONDITIONAL** and **MUST** be present
when the conditions described below are met:

`trust_warning`:
: **REQUIRED** when `trust_tier` is `2`, or when `trust_score` is
  below the threshold configured by the issuer. Value is a short
  string identifying the warning class (e.g.,
  `verification-incomplete`, `trust-score-degraded`).

`trust_explanation`:
: **REQUIRED** when `trust_warning` is present. Value is a prose
  description suitable for surfacing to a human operator or in audit
  logs.

`manifest_issuer`, `manifest_issuer_public_key`, `manifest_signature`:
: Inline signature fields that bind the served Identity Document
  to a recognized issuing registrar. When all three are present,
  the document is a signed manifest; a conforming AGTP server
  **MUST** verify the signature before serving the document. The
  three fields and their verification semantics are defined in
  {{identity-document-tamper-proofing}}. Operators **MUST NOT**
  populate these fields directly; they are written by the
  registrar's signing process.

The Identity Document **MAY** be served unsigned in development
deployments and in deployments that rely on the transport-layer
cert binding defined in {{AGTP-CERT}} as the sole attestation
path. Production deployments **SHOULD** sign the manifest and
**SHOULD** verify the signature on every loaded document.

## What the Identity Document Exposes and Does Not Expose

The Agent Identity Document **MUST** expose:

- The agent's identity (canonical Agent-ID, name, principal, governance zone)
- The agent's current operational status
- The agent's authority scopes accepted
- The agent's supported method vocabulary
- The agent's capability descriptors
- The agent's trust score and trust tier
- The agent's verification anchors and certificate metadata
- The agent's freshness metadata (`issued_at`, `updated_at`)

The Agent Identity Document **MUST NOT** expose:

- Executable code, scripts, or logic
- Model weights or configurations
- Internal API keys or credentials
- Specific authority scope grant tokens
- Session history or prior action logs

No AGTP URI resolution path, including any query parameter
combination, **MUST** return package contents beyond the Identity
Document schema defined in this section.

## Identity Document Tamper-Proofing {#identity-document-tamper-proofing}

Tamper evidence is provided when both of the following mechanisms
are present; when the issuer fields of the second are absent, the
document is verifiable at Level 1 only (see the three-level
verification model in {{AGTP}}):

1. **Package integrity hash:** Any modification to the package or its
   embedded manifest invalidates the hash. The governance platform
   **MUST** verify this hash before extracting the Identity Document.

2. **Inline issuer signature on the Identity Document:** The
   Identity Document carries three optional but RECOMMENDED
   fields that bind the served document to a recognized issuing
   registrar:

   - `manifest_issuer`: a string identifier of the registrar that
     issued the signed manifest. Operator-defined format (typically
     a domain name or registrar identifier).
   - `manifest_issuer_public_key`: the registrar's Ed25519 public
     key, 32 raw bytes encoded as base64url. The signature is
     verified against this key.
   - `manifest_signature`: a detached Ed25519 signature over the
     canonical-form serialization of the Identity Document with
     the `manifest_signature` field excluded (and the
     `manifest_issuer` and `manifest_issuer_public_key` fields
     included). Base64url-encoded.

   When all three fields are present, the Identity Document is a
   **signed manifest**. A conforming AGTP server **MUST** verify
   `manifest_signature` against `manifest_issuer_public_key`
   before serving the document; a document that fails this check
   **MUST NOT** be served and the failure **MUST** be logged.

   Verification confirms the bytes of the document match what the
   recorded issuer signed. Verification does **NOT** confirm that
   `manifest_issuer_public_key` belongs to a trusted registrar;
   that determination is a deployment-policy concern (trust
   anchors, allowlists, or out-of-band attestation). Relying
   parties **SHOULD** maintain a trusted-registrars list per
   governance zone and **MUST NOT** treat a structurally valid
   signature as sufficient evidence of trustworthy issuance on
   its own.

   An Identity Document that omits all three fields is an
   **unsigned manifest** and is permitted in development and in
   deployments that use other attestation paths (e.g., the
   transport-layer cert binding in {{AGTP-CERT}}). Production
   deployments **SHOULD** sign the manifest.

   Operators **MUST NOT** forge issuance: the registrar's
   signing process **MUST** strip any caller-supplied
   `manifest_issuer`, `manifest_issuer_public_key`, or
   `manifest_signature` fields from the input before signing
   with its own key, so a relying party can rely on the recorded
   issuer being the actual signer.

The mechanisms are complementary: the package integrity hash
protects against post-extraction modification of bundled assets;
the inline signature protects against in-flight modification of
the served document. An Identity Document that fails either
verification step **MUST** be rejected, **MUST NOT** be served,
and the failure **MUST** be logged.

Issuers **MUST** retain every prior version of an Identity Document
they have signed, keyed by Agent-ID and `updated_at`, for as long as
attribution records that reference the agent may be verified, so that
a signature made under a since-rotated `agent_public_key` can be
checked against the document that was current when the action was
recorded ({{AGTP-IDENTIFIERS}}).

# Browser and Human-Facing Interaction Model

## The Separation of Discovery and Execution

~~~~
agtp:// URI in a browser
  │
  └─→ Returns Agent Identity Document
        Human-readable view of identity and status
        Read-only. No execution. No code exposed.

agtp:// session initiated by an agent or AGTP client
  │
  └─→ Establishes authenticated AGTP session
        Method invocations (QUERY, EXECUTE, ESCALATE, etc.)
        Full protocol operation, invisible to browsers
~~~~
{: #browser-separation title="AGTP URI Use by Audience"}

The analogy to existing protocol conventions is direct. A `mailto:`
URI surfaces an address and hands off to a mail client; SMTP carries
the actual messages. Similarly, an `agtp://` URI surfaces identity
and status; AGTP carries agent traffic. Browsers never become AGTP
clients by following an `agtp://` link.

## Browser Behavior for agtp:// URIs

Browsers that encounter an `agtp://` URI **SHOULD** behave as follows:

1. If a registered AGTP client is present (OS protocol handler), hand
   off the URI to that client.

2. If the browser supports `agtp://` natively or via extension, render
   the returned Agent Identity Document as a structured human-readable
   page. The rendered view **MUST** surface the trust tier indicator
   prominently, following the visual convention established for TLS
   trust in the browser chrome.

3. If neither condition applies, the browser **MAY** fall back to a
   gateway that translates between `https://` and `agtp://`. The
   gateway **MUST** preserve all signature and trust tier fields.

## Human-Readable Identity Document View

When an Agent Identity Document is rendered for human consumption,
the following fields **MUST** be prominently displayed:

- Agent label and org domain
- Trust tier indicator (visual distinction between Tier 1, 2, and 3)
- Lifecycle state (Active / Suspended / Revoked / Deprecated)
- Job description
- Principal organization
- Activation date
- Behavioral trust score
- Authority scope categories (in human-readable form)

## AGTP Status Sub-Resource

Implementations **SHOULD** support a `status` sub-path:

~~~~
agtp://acme.tld/agents/customer-service/status
~~~~

~~~~json
{
  "document_type": "agtp-status",
  "canonical_id": "3a9f2c1d8b7e4a6f...",
  "agent_label": "customer-service",
  "org_domain": "acme.tld",
  "lifecycle_state": "Active",
  "cert_status": "Active",
  "last_action_method": "QUERY",
  "last_action_timestamp": "2026-03-20T13:58:22Z",
  "active_session_count": 3,
  "pending_escalations": 0,
  "generated_at": "2026-03-20T14:00:00Z"
}
~~~~
{: title="AGTP Status Sub-Resource Response"}

The `active_session_count` field **SHOULD** only be included if the
requester has appropriate observability permissions for the governance
zone.


# Security Considerations

The security considerations of {{AGTP}} apply. The identity document
is the authority for Owner-ID and the Authority-Scope set, so its
integrity is the integrity of every authorization decision made from
it. The tamper-evidence section above states the two mechanisms and
the condition under which each applies; deployments that omit the
issuer fields accept Level 1 verification for every request the
document governs.

# IANA Considerations

This document is the authoritative reference for the
`application/vnd.agtp.identity+json` and
`application/vnd.agtp.identity+yaml` media types registered through
{{AGTP}}. It requests no further allocations.

--- back

# Changes from AGTP v09

First revision under this name. The `agent_public_key` field is
new, carrying the key that the base specification's per-request proof
of possession and delegation tokens verify against, and issuers are
now required to retain prior document versions so attribution chains
verify across key rotation ({{AGTP-IDENTIFIERS}}); `log_uri` is
mirrored from the Genesis ({{AGTP-LOG}}) and `trust_score_computed_at`
is listed as required beside `trust_score` ({{AGTP-TRUST}}). The text is
otherwise extracted from draft-hood-independent-agtp-09, sections Agent Namespace Document,
Agent Identity Document and the .agtp Format, and Browser and
Human-Facing Interaction Model, with these edits: cross-references
into the base document are expressed as citations; the tamper-proof
"guarantee" is restated as tamper evidence conditional on the
optional issuer fields, with the Level 1 fallback named; headings are
promoted one level. No wire-visible change.
