---
title: "AGTP Agent Certificate Extension"
abbrev: "AGTP-CERT"
docname: draft-hood-agtp-agent-cert-04
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - agent identity
  - agent certificate
  - TLS-layer governance
  - X.509

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
  RFC4122:
  RFC5280:
  RFC8555:
  AGTP:
    title: "Agent Transfer Protocol (AGTP)"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-independent-agtp-10
    date: 2026

informative:
  RFC6962:
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
      Internet-Draft: draft-hood-agtp-identifiers-03
    date: 2026
  AGTP-TRUST:
    title: "AGTP Trust and Verification Specification"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-trust-03
    date: 2026
  AGTP-LOG:
    title: "AGTP Transparency Log"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-log-03
    date: 2026
  AGTP-PRESENCE:
    title: "AGTP Presence: Ambient Discovery and Visibility for Agents"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-presence-01
    date: 2026
  AGTP-LEI:
    title: "AGTP Binding to the Legal Entity Identifier (LEI and vLEI)"
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

The Agent Transfer Protocol (AGTP) base specification verifies agent
identity at the application layer: the Agent-ID header resolves to
an issuer-signed Agent Identity Document, and a per-request
Request-Signature proves possession of the key that document names.
That verification runs per request and needs the document. This
document specifies the AGTP Agent Certificate Extension: an optional
mechanism that binds the Canonical Agent-ID, the recorded owner, and
the Authority-Scope commitment to an X.509 v3 certificate presented
during TLS mutual authentication, so that the same verification is
made once at the handshake and infrastructure components including
Scope-Enforcement Points (SEPs), load balancers, and governance
gateways can enforce authority scope without application-layer
access, at O(1) cost per request. The extension also defines
session-level revocation propagation via AGTP NOTIFY broadcast and
records certificate issuance and revocation in the AGTP transparency
log.

Note: Certain mechanisms described in this document may be subject to
pending patent applications by the author. The licensor is prepared to
grant a royalty-free license to implementers consistent with the IETF's
IPR framework. See {{ipr-notice}}.

--- middle

# Introduction

## Where Base AGTP Verifies Identity

{{AGTP}} verifies agent identity at three levels. At Level 1 the
`Agent-ID` and `Authority-Scope` headers are recorded as written. At
Level 2, which base AGTP provides without this extension, the
Agent-ID resolves to an issuer-signed Agent Identity Document
({{AGTP-IDDOC}}) carrying the agent's scope grant and its current
`agent_public_key`, and the `Request-Signature` header on every
non-anonymous request proves possession of that key. Level 2 is the
identity mechanism the protocol depends on; the owner is read from
the Identity Document, never from a request header.

Level 2 has two costs. The verifier needs the Identity Document, and
it verifies a signature per request at the application layer.
Infrastructure that sits below the application (SEPs, load
balancers, governance gateways) either performs that work itself or
forwards traffic unverified. For higher-stakes deployments
(financial transactions, healthcare operations, legal actions,
multi-organization agent federations) that infrastructure needs to
verify identity and enforce scope at line rate without parsing
application payloads.

## The Agent Certificate Extension

The AGTP Agent Certificate Extension is Level 3 of {{AGTP}}: it moves
the Level 2 verification to the TLS handshake. An AGTP Agent
Certificate is an X.509 v3 certificate with agent-governance-specific
extensions, presented during TLS mutual authentication, enabling the
server and any AGTP-aware infrastructure component to verify the
agent's identity and authority scope from the certificate alone,
once per connection, without inspecting the request headers or body.
Level 3 accelerates and enforces Level 2; it replaces nothing.

Workload identity and agent identity are different concerns that
compose. A WIMSE or SPIFFE credential authenticates the workload an
agent runs on; the Agent Certificate authenticates the agent identity
that persists across workloads, since the Canonical Agent-ID outlives
any host. A deployment **MAY** present both at the handshake, and the
binding of an Agent-ID to the workload that holds its key is the
"bound" rung of the agency-evidence ladder that {{AGTP}} and its
companions are building toward.

This document specifies:

- The AGTP Agent Certificate schema and X.509 v3 extension fields
- The certificate issuance and renewal protocol
- The authority scope commitment mechanism for O(1) per-request
  scope enforcement
- Session-level revocation propagation via AGTP NOTIFY
- Certificate transparency through the AGTP transparency log
  ({{AGTP-LOG}})

## Scope

This extension is OPTIONAL. Core AGTP implementations that omit this
extension remain fully compliant with {{AGTP}} and verify identity at
Level 2. The extension is required only for SEP-enforced scope
constraint without application-layer access; Trust Tier 1 standing
is reached through the verification paths of {{AGTP-TRUST}}, of which
the certificate is one input.

# Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when,
and only when, they appear in all capitals.

AGTP Agent Certificate:
: An X.509 v3 certificate carrying agent-governance-specific extensions,
  presented during TLS mutual authentication to establish cryptographic
  agent identity and authority scope at the TLS layer.

Scope-Enforcement Point (SEP):
: An AGTP-aware infrastructure component that enforces Authority-Scope
  constraints on AGTP requests. With the Agent Certificate Extension,
  SEPs verify scope from the certificate at O(1) cost per request without
  application-layer access.

Authority-Scope Commitment:
: A cryptographic binding of the agent's declared Authority-Scope tokens
  to the Agent Certificate, enabling SEPs to verify scope token membership
  after a single session-establishment signature verification.

Certificate Transparency:
: The recording of certificate issuance and revocation as statements
  in the AGTP transparency log of {{AGTP-LOG}}, which provides
  tamper-evident public accountability for them. Earlier revisions
  of this document defined a separate AGTP Certificate Transparency
  Log (AGTP-CTL); it is superseded by {{AGTP-LOG}}.

# SSL Certificate Analogy and Composable Certificate Authorities {#ssl-analogy}

The architectural pattern this document specifies is structurally
analogous to the SSL/TLS certificate infrastructure that operates for
websites and services today. Understanding this analogy clarifies
what AGTP Agent Certificates are, how existing Certificate Authorities
can extend their infrastructure to serve agents, and what the
operational deployment model looks like.

## The SSL/TLS Certificate Pattern

SSL/TLS certificates have been the foundational identity primitive
for the web for over two decades. The infrastructure operates on a
consistent pattern:

- The underlying entity (a domain, an organization) is identified
  through a globally established naming system (DNS).
- A Certificate Authority verifies control over the identifier
  through an established validation process.
- The CA issues an X.509 certificate binding the verified identifier
  to a cryptographic public key.
- The certificate is presented during TLS handshake to establish
  authenticated communication.
- Trust chains terminate at root CAs included in client trust stores
  (browsers, operating systems, applications).
- Revocation is handled through Certificate Revocation Lists, OCSP,
  and increasingly through short-lived certificate rotation per
  {{RFC8555}} (ACME).

The CA business model has been operational at scale for decades.
DigiCert, Sectigo, GlobalSign, Entrust, IdenTrust, and Let's Encrypt
all operate within this framework with established revenue models
(recurring certificate fees for commercial CAs, foundation funding
for Let's Encrypt), regulatory compliance frameworks (WebTrust audits,
ETSI standards, CA/Browser Forum baseline requirements), and
operational maturity (24/7 issuance, validated revocation, browser
trust store inclusion).

## AGTP-CERT as Extension of the CA Pattern

The AGTP Agent Certificate applies this pattern to agent
infrastructure. The architectural decomposition is:

| Layer | SSL/TLS Certificates | AGTP Agent Certificates |
|-------|----------------------|---------------------------|
| Identifier | DNS domain name | Canonical Agent-ID |
| Validation | Domain control verification (DNS, HTTP, email) | Agent Genesis verification per {{AGTP-TRUST}} |
| Certificate Format | X.509 v3 | X.509 v3 with agent-governance extensions |
| Trust Chain | Root CA → intermediate CA → leaf | Root CA → intermediate CA → AGTP leaf certificate |
| Presentation | TLS handshake | TLS mutual authentication with AGTP extensions |
| Revocation | CRL, OCSP, short-lived rotation | Same mechanisms plus AGTP NOTIFY broadcast |
| Trust Store | Browser/OS trust stores | AGTP-aware infrastructure trust stores |
| Provisioning | Manual + ACME {{RFC8555}} | Manual + ACME-style automated provisioning |

The standard certificate primitives are unchanged. AGTP Agent
Certificates extend the existing X.509 v3 framework with new
extensions for agent-governance fields. CAs can keep
their existing infrastructure; they need to add support for the new
extensions and the agent-specific validation processes.

## Composable Certificate Authorities

This document explicitly invites existing Certificate Authorities to
operate as Agent Certificate Authorities. The operational and
regulatory infrastructure that DigiCert, Sectigo, GlobalSign, Entrust,
Let's Encrypt, and similar institutions have built for SSL/TLS
certificates transfers naturally to agent certificates:

- Validation pipelines that confirm identifier control (extended from
  DNS-based validation to Genesis-based validation per {{AGTP-TRUST}})
- Certificate issuance infrastructure with operational maturity
- Revocation and short-lived certificate rotation systems
- Compliance frameworks for certificate issuance audits
- Distribution channels through which certificates reach customers
- Trust store inclusion processes for client recognition

These institutions are positioned to extend their existing
infrastructure to agent populations. AGTP-CERT provides the
extensions and validation framework that allow them to do so without
fundamental re-architecture.

New entrants specializing in agent certificate issuance are equally
valid as Agent CAs. The agent certificate market may develop
differently from the human SSL/TLS market, with CAs specializing by
trust tier, industry vertical, or governance domain. AGTP-CERT
favors neither incumbents nor new entrants; both consume the same
certificate framework.

## Deployment Model

The operational deployment model for AGTP-CERT mirrors familiar
SSL/TLS deployment patterns:

**Certificate request.** An agent operator generates a Certificate
Signing Request (CSR) containing the Canonical Agent-ID and proposed
extensions. The CSR is submitted to the chosen Agent CA.

**Validation.** The CA verifies the agent's Genesis document per the
trust tier requirements specified in {{AGTP-TRUST}}. For Tier 1
agents, verification includes DNS-anchored, log-anchored, or hybrid
verification paths. For Tier 2 agents, verification confirms
organizational assertions. For Tier 3 experimental agents, validation
may be lighter-weight.

**Issuance.** The CA signs the certificate with its issuing key,
incorporating the AGTP extensions per this document. The certificate
is delivered to the agent operator for installation.

**Installation.** The agent operator installs the certificate on the
agent's serving infrastructure. The certificate is presented during
TLS mutual authentication for incoming AGTP connections.

**Renewal.** Certificates have validity periods (typically 90 days
to 1 year for current practice, following the SSL/TLS industry trend
toward shorter lifetimes). Renewal happens through the CA's renewal
infrastructure, typically automated via ACME-style protocols.

**Revocation.** Compromised or retired certificates are revoked
through the CA's CRL/OCSP infrastructure plus AGTP-specific NOTIFY
broadcast per Section "Revocation and Session Propagation" of this
document.

The agent operator's relationship with the CA is the standard
commercial relationship existing CA customers already maintain.
What's new is the agent-specific extensions and the Genesis-based
validation process.

## ACME-Style Automated Provisioning

High-volume agent deployments require automated certificate
provisioning. The ACME protocol {{RFC8555}} provides the established
framework for automated SSL/TLS certificate issuance. AGTP-CERT
deployments **MAY** use ACME-style automated provisioning with
agent-specific challenge types:

- **Genesis-control challenge.** The CA verifies that the requesting
  party controls the Agent Genesis document corresponding to the
  Canonical Agent-ID in the CSR.

- **Owner-validation challenge.** The CA verifies that the requesting
  party is authorized by the Owner identified in the Owner-ID per
  {{AGTP-IDENTIFIERS}}.

- **Trust-tier-specific challenges.** Different trust tiers may
  require different validation challenges. Tier 1 verification may
  require DNS-anchored or log-anchored challenges per {{AGTP-TRUST}}.
  Tier 2 verification may require organizational attestation.

The specific ACME extensions for AGTP agent certificate provisioning
are out of scope for this document and are expected to be specified
in a future ACME extension draft analogous to RFC 8737 (TLS-ALPN
challenge) or RFC 8738 (IP address challenge).

## Issuing Authority Guidance

Organizations operating as Agent Certificate Authorities **SHOULD**
meet operational requirements appropriate to the certificates they
issue:

- **24/7 availability** for issuance, renewal, and revocation
  operations
- **Audit logging** of all certificate operations with tamper-evident
  storage
- **Dispute resolution processes** for revocation disputes and
  validation challenges
- **Insurance or financial backing** for liability arising from
  certificate misissuance
- **Trust store inclusion processes** for AGTP-aware infrastructure
  recognition

For Tier 1 issuance specifically, Agent CAs **SHOULD** maintain:

- Independent verification of identity claims (beyond operator
  assertions alone)
- Cross-witnessing through AGTP-LOG per {{AGTP-LOG}} for tamper-
  evident issuance records
- Revocation infrastructure with sub-hour propagation
- Integration with the AGTP Certificate Transparency Log per the
  {{AGTP-LOG}} ({{transparency}})

These requirements parallel the WebTrust and ETSI requirements that
SSL/TLS CAs currently meet for browser trust store inclusion.

# AGTP Agent Certificate Schema

## Certificate Structure

The AGTP Agent Certificate is an X.509 v3 certificate per {{RFC5280}}
with the following subject fields and extensions:

### Standard Subject Fields

| Field | Required | Value |
|---|---|---|
| CN (Common Name) | **MUST** | Human-readable agent label |
| O (Organization) | **MUST** | Organization name (maps to `principal_org`) |
| OU (Organizational Unit) | **MAY** | Governance zone identifier |
| emailAddress | **SHOULD** | Contact email of the responsible principal |
{: title="AGTP Agent Certificate Subject Fields"}

### Agent-Governance X.509 v3 Extensions

The following extensions are defined for AGTP Agent Certificates.
OIDs for these extensions are specified in {{iana}}.

**agent-id** (CRITICAL)
: The canonical AGTP Agent-ID bound to this certificate. The
  canonical Agent-ID is the 256-bit SHA-256 hash of the
  canonical-form Agent Genesis document, per {{AGTP}}. Format:
  64 lowercase hexadecimal characters. A relying party that
  parses an AGTP Agent Certificate **MUST** treat the value
  carried in this extension as the authoritative Agent-ID for
  the agent.

  The certificate's public key is independent of the canonical
  Agent-ID: the same Agent Genesis (and therefore the same
  canonical Agent-ID) **MAY** back successive certificates
  issued with different key pairs across renewal cycles. The
  TLS layer **MUST NOT** require that the public key hash equal
  the value carried in `agent-id`; the equality held
  only in earlier drafts that derived Agent-ID from the cert
  public key, and that derivation has been retired in favor of
  the Genesis-hash derivation specified in {{AGTP}}.

  Defense against substitution attacks (in which a CA-signed
  certificate is presented with a forged subject binding) is
  performed at the application layer: a relying party that
  cares about Agent Genesis binding **MUST** retrieve the Agent
  Genesis for the asserted Agent-ID (via `DISCOVER /genesis`
  on the agent's home server per {{AGTP}}, or from a local
  registry copy), recompute the SHA-256 of its canonical form
  with `signature` and `log_inclusion_proof` excluded per
  {{AGTP}}, and confirm the result equals the value in
  `agent-id`. The relying party **MUST** additionally
  verify the Agent Genesis signature against the recognized
  issuer key for the agent's governance platform. SEPs and
  other TLS-layer enforcers **MAY** defer this check to
  application-layer components but **MUST** ensure the check
  is performed before treating the asserted Agent-ID as
  authoritative for governance-sensitive decisions. A
  certificate whose Agent Genesis binding cannot be verified
  **MUST** be treated as TLS-only: usable for TLS
  authentication but carrying no authoritative governance
  identity.

**owner-id** (CRITICAL)
: The identifier of the human or organizational principal
  accountable for this agent's actions, equal to the `owner_id`
  recorded on the Agent Genesis and the Agent Identity Document
  ({{AGTP-IDDOC}}). Owner identity rides in no request header;
  the certificate carries it so that infrastructure can read it
  without resolving the document. Format: UTF-8 string, maximum
  256 characters.

  This field identifies the agent's *owner* (the principal
  recorded on the Agent Genesis as accountable for the agent's
  existence). It is distinct from the `principal_id`
  field on extended Attribution-Records per {{AGTP-IDENTIFIERS}},
  which identifies the principal on whose behalf the agent acts
  for a *specific request*, typically lifted from an external
  OAuth or OIDC credential per the composition section in
  {{AGTP}}. The two principals are independent: the owner is
  permanent and certificate-bound; the request principal is
  per-request and credential-bound. A request **MAY** carry
  both, neither, or one without the other; their semantics are
  independent.

**authority-scope-commitment** (CRITICAL)
: The agent's committed Authority-Scope as a canonical token
  list. The extension value is the lexicographically sorted,
  comma-separated, UTF-8-encoded list of Authority-Scope tokens
  the agent is authorized to assert. The integrity of the
  committed token list is provided by the certificate's
  enclosing CA signature; no separate signature is carried in
  the extension. A SEP enforces Authority-Scope at line rate by
  parsing the extension value once per session and checking each
  request's `Authority-Scope` header tokens against the parsed
  set ({{sep-enforcement}}). Format: UTF-8 string of comma-
  separated tokens, each token matching the Authority-Scope
  token grammar defined in {{AGTP}}.

**governance-zone** (NON-CRITICAL)
: The governance zone identifier in which the agent is
  registered. SEPs **MAY** enforce that the request's
  `AGTP-Zone-ID` header matches this value; a mismatch results
  in a 457 Zone Violation per {{sep-enforcement}}. Format:
  UTF-8 string following the `zone:` prefix convention.

**trust-tier** (NON-CRITICAL)
: The agent's Trust Tier (1, 2, or 3) as defined in {{AGTP}}.
  Format: INTEGER.

**archetype** (NON-CRITICAL)
: The agent's behavioral archetype as defined in {{AGTP}}.
  Format: UTF-8 string; one of: assistant, analyst, executor,
  orchestrator, monitor.

**activation-certificate-id** (NON-CRITICAL)
: Cross-layer reference to the Agent Genesis lifecycle event
  that activated this certificate. Enables audit reconstruction
  from a TLS certificate back to the Agent Genesis
  activation record without introducing a cryptographic
  dependency that would force certificate re-issuance whenever
  the Agent Genesis lifecycle state is updated. Format: 64
  lowercase hexadecimal characters.

**agtp-log-receipt** (NON-CRITICAL)
: The receipt for the `agent-cert-issued` statement committed
  to the AGTP transparency log before delivery
  ({{transparency}}). Format: the COSE_Sign1 receipt of
  {{AGTP-LOG}}, DER-wrapped as an OCTET STRING. Earlier
  revisions named this extension `agtp-ctl-sct` and carried an
  RFC 6962 SCT; the {{AGTP-LOG}} receipt replaces it. Relying
  parties that require certificate transparency verify the
  receipt against the log's signed tree head per {{AGTP-LOG}};
  those that waive it **MAY** parse it for record-keeping and
  **MUST NOT** treat its absence alone as grounds to refuse the
  certificate, since certificate transparency is a deployment
  election.

# Certificate Issuance Protocol

## Eligibility

Certificate Signing Requests (CSRs) for AGTP Agent Certificates
**MUST** only be accepted for agents in Active lifecycle state in
the AGTP registry. A governance platform **MUST** verify the agent's
lifecycle state at CSR submission time and **MUST** reject CSRs for
agents in Suspended, Revoked, or Deprecated state.

## Issuance Steps

1. The governance platform generates a key pair for the agent (or
   accepts a CSR with an agent-generated key pair).

2. The governance platform populates the certificate subject fields
   and all AGTP-specific extensions from the agent's Agent Genesis
   and registry record.

3. The governance platform verifies that the proposed `authority-scope-
   commitment` stays within the scope granted in the agent's
   Agent Genesis. If it exceeds that scope, the CSR **MUST** be
   rejected.

4. The governance platform signs the certificate using its issuing CA
   key per {{RFC5280}}.

5. If the deployment elects certificate transparency, the
   governance platform commits an `agent-cert-issued` statement to
   its AGTP transparency log ({{transparency}}) and obtains the
   receipt. A deployment that elects otherwise omits this step and
   leaves the `agtp-log-receipt` extension unpopulated.

6. When a receipt is obtained, it is embedded in the
   `agtp-log-receipt` extension and the certificate is delivered to
   the agent. Otherwise the certificate is delivered without that
   extension.

7. The governance platform publishes the new certificate to the
   agent's registry record, triggering a registry state update.

## Certificate Validity

AGTP Agent Certificates **SHOULD** have a validity period of no more
than 90 days. Short validity periods limit the exposure window of a
compromised certificate and reduce reliance on revocation mechanisms.
Renewal **SHOULD** begin at 80% of the validity period.

Certificate renewal carries forward the predecessor's `agent-id`
and `activation-certificate-id` unchanged. The renewed certificate
receives a new serial number, new validity period, and a new SCT.

# TLS Integration

## Mutual Authentication

AGTP connections using the Agent Certificate Extension **MUST** use
TLS 1.3 mutual authentication. The agent presents its AGTP Agent
Certificate as the client certificate during the TLS handshake.

The server verifies the client certificate chain against the issuing
CA trust anchors. Following successful handshake:

1. The server extracts the `agent-id` extension value and
   verifies it matches the `Agent-ID` header on every request on
   the connection.

2. The server extracts the `owner-id` extension value and verifies
   it matches the `owner_id` of the Agent Identity Document the
   Agent-ID resolves to, when the server holds that document; a
   server that defers document resolution records the certificate
   value as the owner.

3. The server extracts the `authority-scope-commitment` extension value
   and uses it to verify Authority-Scope header tokens on each request.

Any mismatch between certificate extension values and AGTP header or
document values **MUST** cause the server to return 401 Unauthorized
and **MUST** be logged.

## Relation to Request-Signature {#request-signature-relation}

Base AGTP proves key possession per request through
`Request-Signature`, verified against the `agent_public_key` of the
Identity Document. The certificate proves possession of its own
subject key at the handshake. The two compose in one of two ways:

- **Same key** (RECOMMENDED). The certificate's subject public key
  is the agent's `agent_public_key` (Ed25519, per {{AGTP-IDDOC}}).
  One key then proves possession at both layers. A server that has
  completed mutual authentication under such a certificate **MAY**
  treat the handshake as the possession proof for every request on
  that connection and skip per-request verification of
  `Request-Signature`; the header **MUST** still be present and
  well-formed, so that the Attribution-Record and any downstream
  verifier can check it independently. Rotation of
  `agent_public_key` then requires certificate re-issuance, and the
  issuer **MUST** refuse a CSR whose key differs from the Identity
  Document's current `agent_public_key` under this option.

- **Distinct keys.** The certificate carries a key of its own. Both
  proofs then stand on their own: the handshake proves the
  certificate key, `Request-Signature` proves the document key, and
  the server **MUST** verify both. This option suits deployments
  whose CA policy fixes the key algorithm or whose TLS stack keeps
  keys in hardware the agent runtime cannot reach.

In either case the early-data rule of the AGTP bindings is
unchanged: a request on a stream opened in 0-RTT carries
`Request-Signature` and is verified, since the handshake that would
otherwise vouch for it is incomplete at that point.

## Scope Enforcement at SEPs {#sep-enforcement}

A SEP operating with the Agent Certificate Extension verifies
Authority-Scope and (optionally) governance zone at O(1) cost per
request:

1. At session establishment, the SEP extracts the
   `authority-scope-commitment` from the client certificate and
   parses the comma-separated token list once. The SEP also
   extracts the `governance-zone` extension if present and zone
   enforcement is configured. (One-time per session.)

2. On each request, the SEP checks whether the `Authority-Scope`
   header tokens are a subset of the parsed commitment token set.
   (O(1) per request after session setup.)

3. If any header token falls outside the commitment token set, the
   SEP returns **455 Scope Violation** without forwarding the
   request to the application layer.

4. If `governance-zone` enforcement is configured and the
   request's `AGTP-Zone-ID` header differs from the value
   carried in the certificate's `governance-zone` extension, the
   SEP returns **457 Zone Violation** without forwarding the
   request.

This enables governance enforcement at line rate without
application-layer parsing.

A certificate that lacks the AGTP-specific extensions is a valid
TLS client certificate but carries no SEP-enforceable governance
metadata. SEP enforcement of `authority-scope-commitment` and
`governance-zone` is purely additive: in the absence of those
extensions, scope and zone are enforced through application-layer
checks against the agent's Agent Identity Document per {{AGTP}}.
Deployments **MAY** mix certificates with and without AGTP
extensions; the SEP layer treats each session by what its
certificate carries.

# Revocation and Session Propagation

## Revocation Events

An AGTP Agent Certificate **MUST** be revoked when any of the following
occur:

- The agent's lifecycle state transitions to Revoked or Deprecated
- The Agent Genesis is invalidated
- The agent's `authority-scope-commitment` requires modification
- The principal requests revocation
- A trust violation is detected

## Session-Level Revocation Propagation

Standard certificate revocation (CRL, OCSP) operates on polling cycles,
leaving a window during which revoked certificates may still be used.
For agent systems, this window is unacceptable for high-stakes operations.

AGTP Agent Certificate revocation **MUST** be propagated to active
sessions via AGTP NOTIFY broadcast:

1. The governance platform commits an `agent-cert-revoked`
   statement to its AGTP transparency log ({{transparency}}) when
   certificate transparency is elected.

2. The governance platform broadcasts an AGTP NOTIFY to all
   infrastructure components holding active sessions for the revoked
   certificate's `agent-id`:

~~~~json
{
  "method": "NOTIFY",
  "parameters": {
    "recipient": "infrastructure:broadcast",
    "content": {
      "event_type": "certificate_revoked",
      "agent_id": "[agent-id]",
      "certificate_serial": "[serial]",
      "revocation_reason": "[reason]",
      "effective_at": "2026-04-01T00:00:00Z"
    },
    "urgency": "critical"
  }
}
~~~~

3. Infrastructure components receiving this NOTIFY **MUST** immediately
   terminate all active sessions for the identified `agent-id`.
   Session termination **MUST** occur before the next request is
   processed on the affected session.

4. The target revocation-to-termination latency is 30 seconds. This is
   materially shorter than standard CRL or OCSP cache-based models.

## Session Manager Responsibilities

AGTP Session Managers in deployments using the Agent Certificate
Extension **MUST** maintain a per-certificate-serial active session
registry. On receiving a revocation NOTIFY, the Session Manager
**MUST** terminate all sessions associated with the revoked serial
before processing the next request on any affected session.

# Certificate Transparency via AGTP-LOG {#transparency}

## Purpose

Earlier revisions of this document defined a separate AGTP
Certificate Transparency Log modeled on {{RFC6962}}. {{AGTP-LOG}}
now provides the family's append-only, SCITT-aligned transparency
log and names certificate-lifecycle events as candidates for it.
This document takes that offer: a deployment that operates both an
AGTP-LOG and an Agent CA records certificate issuance and revocation
as statements in the same log, which provides tamper-evident public
accountability for them and enables:

- Fleet-level analytics: population-wide trust tier distributions,
  archetype frequencies, governance zone composition
- Anomaly detection: certificates issued outside normal governance
  flows
- Audit reconstruction: verifiable history of certificate issuance
  and revocation for compliance

## Statement Types

This document registers two event types in the AGTP-LOG event-type
registry of {{AGTP-LOG}}. The `agtp-subject` of each is the
Canonical Agent-ID the certificate names; the issuer and signer is
the governance platform operating the log, per {{AGTP-LOG}}.

For `agent-cert-issued`, the payload is a CBOR map:

~~~
{
  "serial": <bytes>,                     ; certificate serial number
  "issuer-dn": <text>,                   ; issuing CA distinguished name
  "cert-hash": <bytes>,                  ; SHA-256 of the DER certificate
  "owner-id": <text>,                    ; may be pseudonymized
  "governance-zone": <text>,
  "trust-tier": <uint>,
  "archetype": <text>,
  "activation-certificate-id": <bytes>,
  "not-before": <text>,                  ; RFC 3339
  "not-after": <text>,                   ; RFC 3339
  "log-position": <uint>,
  "previous-tree-size": <uint>
}
~~~

For `agent-cert-revoked`:

~~~
{
  "serial": <bytes>,
  "cert-hash": <bytes>,
  "reason": <text>,
  "effective-at": <text>,                ; RFC 3339
  "log-position": <uint>,
  "previous-tree-size": <uint>
}
~~~

The log operator **MUST** verify, before commit, that `cert-hash`
matches a certificate whose `agent-id` extension equals
`agtp-subject`, and for revocation that an `agent-cert-issued`
statement with the same `serial` precedes it. A forged certificate
that appears in no log is detectable by any party that performs the
inclusion check against `agtp-log-receipt`.

## Privacy Considerations

The `owner-id` field in `agent-cert-issued` statements **MAY** be
pseudonymized to protect individual principal identity while
maintaining audit integrity. Pseudonymous principal IDs **MUST** be
resolvable by authorized parties (regulators, compliance auditors)
via a trusted resolution service. The pseudonymization mapping
**MUST** be maintained separately from the public log.

# Security Considerations

## Certificate Pinning

Deployments with strict security requirements **MAY** implement
certificate pinning for known agents, rejecting connections from agents
whose certificate serial or key differs from a pre-registered value.
Certificate pinning interacts with renewal; pinned agents **MUST**
update pins on each certificate renewal before the old certificate
expires.

## Scope Commitment Forgery

The `authority-scope-commitment` extension carries the agent's
committed Authority-Scope token list. Integrity of the commitment
relies on the certificate's enclosing CA signature: tampering with
the extension value invalidates the certificate signature and causes
the certificate to be refused by any conforming verifier.

An attacker who compromises the issuing CA key can forge scope
commitments by issuing fraudulent certificates with arbitrary
extension values. Issuing-key compromise **MUST** trigger immediate
revocation of all certificates issued by that key and issuance of
replacement certificates from a new key pair. Issuing keys
**SHOULD** be stored in hardware security modules. Certificate
transparency through {{AGTP-LOG}} ({{transparency}}) provides an
additional detection surface for unauthorized issuance: a forged
certificate that appears in no log is detectable by any party that
performs log-inclusion checks.

## Cross-Certificate Confusion

An agent MAY hold multiple certificates simultaneously. Renewal
overlap is one cause; key rotation under a stable Agent Genesis
is another. Because the canonical Agent-ID is bound to the Agent
Genesis and to no specific cert key pair, successive
certificates for the same agent **MUST** carry the same value in
`agent-id` and **MAY** carry different public keys.
Infrastructure **MUST** use the `agent-id` extension
value as the authoritative agent identifier, with the
certificate subject CN carrying no authority, and **MUST NOT** treat key differences
across certificates for the same Agent-ID as evidence of an
identity mismatch.

## IPR Notice {#ipr-notice}

Certain mechanisms described in this document may be subject to
pending patent applications by the author, specifically: the
authority-scope-commitment mechanism and the session-level revocation
propagation architecture. The licensor (Chris Hood / Nomotic, Inc.)
is prepared to grant a royalty-free license to implementers for any
patent claims covering these mechanisms, consistent with the IETF's
IPR framework.

# IANA Considerations {#iana}

## X.509 Extension OID Registrations

This document requests registration of the following Object
Identifiers in an appropriate OID arc for IETF use. Specific
OID assignments are subject to IANA allocation.

Until IANA allocation is complete, implementations **MUST**
use provisional OIDs under the ITU-T UUID arc, derived
deterministically by UUIDv5 ({{!RFC4122}}) under a fixed AGTP
namespace UUID and the extension's canonical short name. The
resulting integer-encoded UUID is appended to the arc prefix
`2.25` to form the provisional OID
(`2.25.{uuid_int}`). The derivation is reproducible across
implementations from the extension short name alone, allowing
independent implementations to interoperate without a central
allocation step. When IANA allocates standards-tree OIDs, those
values replace the provisional UUID-derived OIDs in a future
revision of this document; relying parties **SHOULD** accept
both the provisional and the IANA-allocated OIDs through a
transition window declared in that revision.

| Extension | OID (provisional / IANA) | Critical |
|---|---|---|
| agent-id | UUIDv5-derived (TBD allocation) | Yes |
| owner-id | UUIDv5-derived (TBD allocation) | Yes |
| authority-scope-commitment | UUIDv5-derived (TBD allocation) | Yes |
| governance-zone | UUIDv5-derived (TBD allocation) | No |
| trust-tier | UUIDv5-derived (TBD allocation) | No |
| archetype | UUIDv5-derived (TBD allocation) | No |
| activation-certificate-id | UUIDv5-derived (TBD allocation) | No |
| agtp-log-receipt | UUIDv5-derived (TBD allocation) | No |
{: title="AGTP Agent Certificate X.509 Extension OIDs"}

## AGTP-LOG Event Types

This document registers `agent-cert-issued` and `agent-cert-revoked`
in the AGTP-LOG event-type registry of {{AGTP-LOG}}, with
{{transparency}} as reference.

--- back

# Relationship to Agent Genesis

The AGTP Agent Certificate and the Agent Genesis (defined in
{{AGTP}}) are complementary but distinct:

| Property | Agent Genesis | Agent Certificate |
|---|---|---|
| Layer | Governance / registry | TLS |
| Format | JSON document | X.509 v3 |
| Issued by | Governance platform | Governance platform CA |
| Lifetime | Permanent (archived on revoke) | 90 days (renewable) |
| Carries | Full identity + archetype + scope | TLS identity + scope commitment |
| Purpose | Genesis record, registry anchor | TLS mutual auth, SEP enforcement |
| Identifier | Canonical Agent-ID (256-bit SHA-256 of canonical Agent Genesis) | `agent-id` extension carries the canonical Agent-ID |
| Cross-reference to lifecycle event | (originating issuance event in AGTP-LOG) | `activation-certificate-id` extension |

The `agent-id` extension carries the canonical
Agent-ID (256-bit SHA-256 hash of the canonical Agent
Genesis), creating a direct binding between the TLS
certificate and the governance identity. The
`activation-certificate-id` extension carries a reference to
the lifecycle event that activated this certificate, allowing
audit reconstruction back to the activation record without
introducing a cryptographic dependency that would force
certificate re-issuance whenever the Agent Genesis lifecycle
state is updated.

# Changes from v03

Version 04 aligns the extension with draft-hood-independent-agtp-10,
which added per-request proof of possession to base AGTP, and with
draft-hood-agtp-log-03. The certificate schema is unchanged apart
from one renamed extension.

## Substantive Changes in v04

1. **Positioned as Level 3 of base AGTP.** The introduction no
   longer describes base headers as unverified; base v10 verifies
   identity and possession at Level 2 through the Identity Document
   and `Request-Signature`, and this extension moves that
   verification to the handshake for line-rate enforcement.

2. **Relation to Request-Signature** ({{request-signature-relation}}):
   same-key (RECOMMENDED, subject key equals `agent_public_key`,
   handshake may stand in for per-request verification on the
   connection) and distinct-key (both proofs verified) options; the
   0-RTT rule unchanged.

3. **Owner is a document field.** `owner-id` is defined as equal to
   the Identity Document's `owner_id`; the mutual-authentication
   step that compared it to an `Owner-ID` request header (retired in
   base v10) now compares it to the document.

4. **AGTP-CTL superseded by AGTP-LOG** ({{transparency}}). The
   separate log is withdrawn; `agent-cert-issued` and
   `agent-cert-revoked` are registered as AGTP-LOG event types with
   CBOR payloads; the `agtp-ctl-sct` extension becomes
   `agtp-log-receipt` carrying the AGTP-LOG receipt. Issuance steps
   5 and 6 and revocation step 1 follow.

5. **Workload versus agent identity** stated: WIMSE or SPIFFE
   authenticates the workload, the Agent Certificate the agent that
   persists across workloads; the two compose.

6. **Agent-ID hash** stated with `log_inclusion_proof` excluded,
   per base v10. Trust Tier 1 wording corrected: the certificate is
   an input to the paths of {{AGTP-TRUST}}, which impose no
   requirement for it.

7. References: base to -10; AGTP-API removed (the `/genesis`
   sub-resource is base); AGTP-IDENTIFIERS -03, AGTP-TRUST -03,
   AGTP-LOG -03, AGTP-PRESENCE -01, AGTP-LEI -01, AGTP-COMMERCE -01;
   AGTP-IDDOC added; two unused RFC references dropped. TLS-layer
   in place of transport-layer throughout. Hardcoded section numbers
   replaced. House style pass.

## Wire Format Compatibility (v04)

The extension rename (`agtp-ctl-sct` to `agtp-log-receipt`) changes
a provisional OID and the extension's content; v03 certificates
carrying the old extension remain valid TLS certificates and are
treated as lacking a receipt. Nothing else on the wire changes.

# Changes from v02

Version 03 adds the SSL/TLS Certificate Authority analogy framing and
cross-references to the new companion drafts (AGTP-PRESENCE, AGTP-LEI,
AGTP-COMMERCE). The certificate schema, issuance protocol, revocation
propagation mechanisms, and underlying X.509 extension structure are
unchanged. New material is additive framing plus cross-references that
reflect the broader AGTP draft family that has emerged since v02 of
this document.

## Substantive Changes in v03

The following substantive changes were made in v03:

1. **SSL Certificate Analogy section added.** A new section
   ({{ssl-analogy}}) frames AGTP-CERT in terms of the SSL/TLS
   certificate infrastructure operated by DigiCert, Sectigo,
   GlobalSign, Entrust, Let's Encrypt, and similar Certificate
   Authorities. The section makes explicit that AGTP-CERT extends
   the established X.509 v3 framework with agent-governance
   extensions and that existing CAs can operate as Agent Certificate
   Authorities without fundamental re-architecture. The deployment
   model maps to familiar SSL/TLS deployment patterns (CSR, validation,
   issuance, installation, renewal, revocation). ACME-style automated
   provisioning is named as the high-volume deployment path with
   agent-specific challenge types (Genesis-control, Owner-validation,
   trust-tier-specific).

2. **New companion draft references added.** Informative references
   to {{AGTP-PRESENCE}}, {{AGTP-LEI}}, {{AGTP-COMMERCE}}, {{AGTP-TRUST}} v02, and
   {{AGTP-LOG}} v02 added. These references reflect the broader AGTP
   draft family that has emerged since v02 of this document. The
   composition surfaces include:

   - **{{AGTP-PRESENCE}}** specifies certificate extensions for visibility
     posture declaration that compose with the AGTP-CERT extension
     framework.
   - **{{AGTP-LEI}}** specifies the `vLEIBinding` extension for
     institutional identity composition through GLEIF infrastructure.
   - **{{AGTP-COMMERCE}}** specifies that pricing manifests may be
     referenced from AGTP-CERT extensions for commerce-aware agents.
   - **{{AGTP-TRUST}} v02** establishes the credit-rating-analogous trust
     provider framing that parallels the SSL CA framing in this
     document.

3. **Issuing Authority Guidance section added.** Within the SSL
   Certificate Analogy section, a new subsection specifies
   operational requirements for organizations operating as Agent
   Certificate Authorities. The requirements parallel WebTrust and
   ETSI requirements that SSL/TLS CAs currently meet for browser
   trust store inclusion. Tier 1 issuance requirements include
   independent verification, AGTP-LOG cross-witnessing, sub-hour
   revocation propagation, and AGTP-CTL integration.

## Changes from v01 (Carried Forward from v02)

Version 02 was a naming-consistency revision. The certificate schema,
issuance protocol, revocation propagation, and all underlying
semantics were unchanged. Extension names were renamed for consistency
with the wire header model defined in {{AGTP}}:

1. The `subject-agent-id` extension was renamed to `agent-id`. The
   X.509-flavored `subject-` prefix was removed; the extension carries
   the canonical Agent-ID and its name now matches the wire header it
   commits to.

2. The `principal-id` extension was renamed to `owner-id`. The
   extension has always carried the Owner-ID (the permanent principal
   recorded on the Agent Genesis) and never a per-request acting principal;
   the name now matches the wire header and the underlying concept.
   Cross-references to the per-request delegating principal field on
   Attribution-Records were updated to use that field's new name
   (`principal_id`, per {{AGTP-IDENTIFIERS}}).

3. References in introduction and verification procedure text to a
   `Principal-ID` wire header were corrected to `Owner-ID`, the
   actual wire header defined by {{AGTP}}.

No semantic changes in v02. Implementations on v01 extensions migrate
to v02 by updating extension OID labels and field references; the
underlying X.509 structures and OID allocations were unchanged.

## Changes from v00 (Carried Forward from v01)

Version 01 was a drift-cleanup revision documenting deployed
implementation behavior. The certificate schema, issuance protocol,
and revocation propagation mechanisms were unchanged. The following
substantive changes were made in v01:

1. **`authority-scope-commitment` representation.** The
   extension value is now defined as the lexicographically
   sorted, comma-separated, UTF-8-encoded list of
   Authority-Scope tokens. Integrity is provided by the
   certificate's enclosing CA signature; the separate
   Ed25519 signature the extension once carried is gone. The earlier
   detached-signature framing is withdrawn. SEP enforcement
   parses the token list once per session and checks request
   tokens by set membership; the operational contract is
   unchanged for relying parties, only the encoding of the
   commitment value is changed.

2. **`agent-id` decoupled from certificate public
   key; substitution defense moved to application layer.**
   Earlier drafts implied that the canonical Agent-ID could
   be derived from the certificate public key and that the
   TLS layer must refuse certificates whose `agent-id`
   disagreed with that derivation. Under the current Agent
   Genesis taxonomy, the canonical Agent-ID is
   `sha256(canonical_form(Agent_Genesis_without_signature))`
   and is independent of any specific cert key pair. The
   `agent-id` extension is authoritative when present;
   the cert public key is independent and renewable. The
   substitution-attack defense is performed at the
   application layer by retrieving the Agent Genesis (via
   `DISCOVER /genesis` per base AGTP or a local registry
   copy), recomputing the canonical hash, and verifying the
   Agent Genesis signature against the recognized issuer key.
   The Cross-Certificate Confusion security consideration is
   updated accordingly: successive certificates for the same
   agent **MUST** carry the same `agent-id` and **MAY**
   carry different public keys.

3. **Birth Certificate terminology retired.** All references
   to the Agent Birth Certificate have been replaced by Agent
   Genesis, matching the locked taxonomy in {{AGTP}} (Agent
   Genesis is the permanent signed governance-layer origin
   document; the canonical Agent-ID is its 256-bit SHA-256
   hash; the Agent Certificate is the X.509 v3 transport
   credential bound to that Agent-ID). The `Relationship to
   Birth Certificate` appendix is renamed and rewritten as
   `Relationship to Agent Genesis`.

4. **SEP status codes updated to v07 numbering.** Scope
   Enforcement at SEPs now returns **455 Scope Violation**
   (previously 451) and adds **457 Zone Violation** for
   certificates carrying the `governance-zone` extension when
   the request's `AGTP-Zone-ID` header disagrees with the
   certificate. The status code renumbering propagates the
   v06 to v07 change in {{AGTP}}.

5. **SEP enforcement made additive.** A new paragraph in
   {{sep-enforcement}} makes explicit that a certificate
   without AGTP-specific extensions is a valid TLS client
   certificate and is enforced through application-layer
   checks against the Agent Identity Document. Deployments
   may mix certificates with and without AGTP extensions.

6. **`activation-certificate-id` semantics clarified.** The
   field is now defined as a cross-layer reference to the
   Agent Genesis lifecycle event that activated this
   certificate, in place of a `certificate_hash` field
   that no longer exists under the locked taxonomy. The
   relying-party contract is unchanged: a 64-hex value
   suitable for cross-layer audit reconstruction.

7. **`agtp-ctl-sct` cryptographic verification deferred.**
   The extension may be carried and parsed for record-keeping
   purposes, but verification against an operating AGTP-CTL
   is deferred to a future revision. The Issuance Protocol is
   updated to make AGTP-CTL submission and SCT embedding
   conditional on AGTP-CTL availability.

8. **Provisional OID strategy introduced.** OIDs for the
   eight extensions are derived deterministically as UUIDv5
   values under a fixed AGTP namespace and the extension's
   canonical short name, placed under the ITU-T UUID arc
   `2.25.{uuid_int}`. This permits independent implementations
   to interoperate without a central allocation step. The
   provisional OIDs will be replaced by IANA-allocated
   standards-tree OIDs in a future revision.

9. **Normative reference to {{AGTP}} updated to v08.**
   Section references that pointed at v02 / v06 section
   numbers (`Section 6.2`, `Section 6.7`, `Section 6.7.3`,
   `Section 8.7`) are removed; cross-references now name the
   companion document and the concept in place of a specific
   section number, since section structure in {{AGTP}}
   continues to evolve. RFC 4122 added to normative references
   to support the OID derivation.

## Wire Format Compatibility

The v03 changes are entirely framing additions and cross-reference
updates. There are no wire-format-visible changes in this revision.
Implementations conforming to v02 continue to interoperate with v03.

The v02 identifier rename (`subject-agent-id` to `agent-id` and
`principal-id` to `owner-id`) was the wire-format-visible change in
the v02 revision. Implementations parsing the `subject-agent-id`
extension labels fail to interoperate with implementations using the
renamed `agent-id` and `owner-id` extension labels under the same
OIDs. The OIDs themselves are unchanged across v01, v02, and v03;
only the canonical short names changed in v02.

All other changes are editorial or specify operational
behavior that v00 was silent on.
