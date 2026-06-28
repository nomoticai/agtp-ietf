---
title: "AGTP-LEI: Binding the Agent Transfer Protocol to the Verifiable Legal Entity Identifier"
abbrev: "AGTP-LEI"
docname: draft-hood-agtp-lei-00
category: info
submissiontype: independent
ipr: trust200902
area: "Applications and Real-Time"
workgroup: "Independent Submission"
keyword:
  - AI agents
  - LEI
  - vLEI
  - GLEIF
  - legal entity identification
  - KERI
  - financial services
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
  AGTP: I-D.hood-independent-agtp
  AGTP-IDENTIFIERS: I-D.hood-agtp-identifiers
  AGTP-TRUST: I-D.hood-agtp-trust
  AGTP-CERT: I-D.hood-agtp-agent-cert
  RFC2119:
  RFC8174:
  ISO17442:
    title: "Financial services -- Legal entity identifier (LEI)"
    target: https://www.iso.org/standard/78829.html
    seriesinfo:
      ISO: 17442-1:2020
    date: 2020

informative:
  GLEIF:
    title: "Global Legal Entity Identifier Foundation"
    target: https://www.gleif.org
  VLEI-EGF:
    title: "verifiable LEI (vLEI) Ecosystem Governance Framework v4.0"
    author:
      org: Global Legal Entity Identifier Foundation
    date: 2026
    target: https://www.gleif.org/en/vlei/introducing-the-vlei-ecosystem-governance-framework
  KERI:
    title: "Key Event Receipt Infrastructure (KERI)"
    author:
      fullname: Samuel M. Smith
    seriesinfo:
      Internet-Draft: draft-ssmith-keri
    date: 2023
  ACDC:
    title: "Authentic Chained Data Containers (ACDC)"
    author:
      org: Trust over IP Foundation
    target: https://trustoverip.github.io/tswg-acdc-specification/
  ISO5009:
    title: "Financial services -- Official organizational roles"
    seriesinfo:
      ISO: 5009
  DID-KERI:
    title: "DID Method Specification for KERI"
    target: https://github.com/WebOfTrust/did-keri
  AGTP-MERCHANT-IDENTITY:
    title: "AGTP Merchant Identity"
    author:
      fullname: Chris Hood
    seriesinfo:
      Internet-Draft: draft-hood-agtp-merchant-identity-01
    date: 2026

--- abstract

The Legal Entity Identifier (LEI), defined by ISO 17442 and operated
under the global governance of the Global Legal Entity Identifier
Foundation (GLEIF), is the internationally recognized identifier for
legal entities in financial transactions and regulatory reporting.
The verifiable LEI (vLEI), built on Key Event Receipt Infrastructure
(KERI) and Authentic Chained Data Containers (ACDC), extends the LEI
into the digital trust ecosystem with cryptographically verifiable
credentials issued through Qualified vLEI Issuers (QVIs).

This document specifies how the Agent Transfer Protocol (AGTP) binds
to the LEI and vLEI infrastructure. It defines how an LEI is carried
in an AGTP Owner-ID, how a vLEI Legal Entity credential composes with
AGTP-CERT to establish institutional identity at the wire, how a new
verification path (`vlei-anchored`) is added to AGTP-TRUST for
KERI-based verification, and how vLEI Role Credentials (OOR, ECR, AUTH)
express the human authorization chain that produced an agent.

The result is a clean composition: AGTP provides the agent identity
substrate, the LEI provides the institutional identity, the vLEI
provides cryptographic verification of that institutional identity,
and the vLEI Role Credentials provide the authorization chain from
the legal entity through human officers to the agent. This document
positions financial institutions, regulated entities, and other LEI
holders to deploy agents with structurally verifiable institutional
identity from day one.

--- middle

# Introduction

The Legal Entity Identifier is the de facto international standard
for identifying legal entities in financial transactions. Defined by
{{ISO17442}}, the LEI is a 20-character alphanumeric code uniquely
identifying a legal entity participating in financial transactions.
The Global Legal Entity Identifier Foundation operates the global LEI
system as a not-for-profit organization, with GLEIF acting as the
root of trust for the LEI ecosystem.

The LEI is operationally mature. Every regulated financial entity in
G20 jurisdictions has or can obtain an LEI. Banking regulators,
securities regulators, derivatives reporting requirements, anti-money
laundering frameworks, and supply chain compliance regimes use the
LEI as the canonical institutional identifier. The Financial Stability
Board, the Basel Committee, the European Securities and Markets
Authority, and the U.S. Securities and Exchange Commission all rely
on LEI for entity identification in regulated reporting.

The verifiable LEI extends this infrastructure into the digital trust
ecosystem. The vLEI is a cryptographically verifiable credential issued
by Qualified vLEI Issuers to legal entities, built on the Key Event
Receipt Infrastructure {{KERI}} and the Authentic Chained Data
Container specification {{ACDC}}. The vLEI Ecosystem Governance
Framework {{VLEI-EGF}} defines the operational model with GLEIF as
root of trust, QVIs as accredited credential issuers, and Legal
Entities as credential holders. Three additional vLEI credential types
extend institutional identity to the human authorization layer: the
Official Organizational Role (OOR) credential per {{ISO5009}}, the
Engagement Context Role (ECR) credential, and the Authorization (AUTH)
credential.

For AGTP, the question is how an agent deployed by a regulated legal
entity inherits the institutional identity verifiable through the
vLEI infrastructure. The answer matters because financial regulators,
banking compliance officers, and institutional risk managers ask the
question Tom Roberts surfaced in conversation with major banks: "How
do we know who is an agent, and on whose authority does it act?" The
LEI answers the institutional half of that question. The vLEI provides
cryptographic verification. This document specifies how AGTP composes
with that infrastructure to answer the agent half.

## Conventions and Terminology

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document
are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

This document uses terminology from {{AGTP}}, {{AGTP-IDENTIFIERS}},
{{AGTP-TRUST}}, and {{AGTP-CERT}}. Selected additional terms from the
LEI and vLEI infrastructure:

LEI:
: Legal Entity Identifier. A 20-character alphanumeric code per
  {{ISO17442}} that uniquely identifies a legal entity. Issued by an
  LEI Issuer accredited by GLEIF.

vLEI:
: verifiable Legal Entity Identifier. The cryptographically verifiable
  digital form of the LEI, issued as an ACDC credential through KERI
  infrastructure. Specified in {{VLEI-EGF}}.

GLEIF:
: Global Legal Entity Identifier Foundation. The organization that
  operates the global LEI system and acts as the root of trust for
  the vLEI ecosystem.

QVI:
: Qualified vLEI Issuer. An organization accredited by GLEIF to issue
  vLEI credentials to legal entities.

KERI:
: Key Event Receipt Infrastructure {{KERI}}. The cryptographic
  identifier and key management protocol that underlies vLEI.

AID:
: Autonomic Identifier. A KERI self-certifying identifier bound to a
  cryptographic key pair, used as the durable identifier for
  participants in the vLEI ecosystem.

ACDC:
: Authentic Chained Data Container. The credential format used for
  vLEI credentials, supporting chained issuance from GLEIF through
  QVIs to legal entities.

OOR Credential:
: Official Organizational Role vLEI credential. Issued by a Legal
  Entity (or QVI on its behalf) to a human officer in an ISO 5009
  recognized role.

ECR Credential:
: Engagement Context Role vLEI credential. Issued by a Legal Entity
  to a human in a contextual role for specific engagements.

AUTH Credential:
: Authorization vLEI credential. Issued by a Legal Entity to authorize
  specific actions, typically chained from OOR or ECR credentials.

# Scope

This document specifies four composition surfaces between AGTP and
the LEI/vLEI infrastructure:

1. How an LEI is carried in an AGTP Owner-ID, providing institutional
   identification at the wire ({{owner-id-binding}}).

2. How a vLEI Legal Entity credential composes with AGTP-CERT to
   provide cryptographic verification of institutional identity
   ({{cert-composition}}).

3. How a new verification path (`vlei-anchored`) is added to
   AGTP-TRUST to support KERI-based verification of agent institutional
   identity ({{verification-path}}).

4. How vLEI Role Credentials (OOR, ECR, AUTH) express the human
   authorization chain that produced an agent, establishing
   accountability from the legal entity through human officers to the
   agent ({{authorization-chain}}).

This document does not modify the LEI standard, the vLEI Ecosystem
Governance Framework, KERI, ACDC, or any of the existing AGTP
specifications. It defines composition surfaces that allow agents
deployed by LEI holders to carry verifiable institutional identity
through the AGTP substrate.

# Owner-ID Binding {#owner-id-binding}

## LEI in Owner-ID

AGTP-IDENTIFIERS defines the Owner-ID as the identifier of the
principal accountable for an agent's existence and operation. For
agents deployed by legal entities that hold an LEI, the Owner-ID
**MAY** carry the LEI directly:

~~~
Owner-ID: lei:5493001KJTIIGC8Y1R12
~~~

The `lei:` URI scheme prefix identifies the following string as an
LEI per {{ISO17442}}. The 20-character LEI value follows. Verifiers
receiving this Owner-ID can resolve the LEI through GLEIF's public
LEI lookup service to retrieve the associated legal entity record.

When the Owner-ID carries an LEI, the agent's institutional identity
is verifiable against the public LEI registry without requiring
additional infrastructure. This composes with any AGTP verification
path: an agent at Trust Tier 1 with `dns-anchored` verification still
benefits from LEI-carried Owner-ID because the institutional identity
is publicly verifiable independent of AGTP's verification mechanism.

## Owner-ID Examples

A bank deploying an agent for customer service:

~~~
Owner-ID: lei:5299002IZHCT9HFFP822
~~~

An asset management firm deploying a trading agent:

~~~
Owner-ID: lei:549300JR9TDP4FYQGW40
~~~

A regulated insurance entity deploying a claims processing agent:

~~~
Owner-ID: lei:213800ULZN5RMQQUTM63
~~~

In each case, the Owner-ID communicates institutional identity
verifiable against public infrastructure. Verifiers can apply
institutional context (regulated entity, jurisdiction, parent
relationships) to authority decisions without consulting any AGTP
operator's private records.

## Composition with Domain-Anchored Identifiers

The LEI-based Owner-ID composes with the domain-anchored Owner-ID
forms defined in {{AGTP-IDENTIFIERS}}. An agent **MAY** carry both:
the LEI as the primary institutional identifier and a domain as a
secondary operational identifier. The Identity Document **MAY**
include both:

~~~ json
{
  "owner_id_primary": "lei:5493001KJTIIGC8Y1R12",
  "owner_id_secondary": "owner:acmebank.com",
  "owner_legal_name": "Acme Bank, N.A."
}
~~~

Verifiers resolving the primary Owner-ID against GLEIF can verify the
institutional identity. Verifiers operating purely against
domain-anchored identifiers can use the secondary form. The two
identifiers refer to the same legal entity but provide different
verification paths.

# Certificate Composition {#cert-composition}

## vLEI Legal Entity Credential in AGTP-CERT

AGTP-CERT specifies an X.509 v3 certificate extension carrying
agent-governance-specific fields. This binding defines an extension
mechanism for carrying or referencing a vLEI Legal Entity credential
within the AGTP-CERT structure.

The composition can take two forms:

**Inline vLEI Credential.** The AGTP-CERT extension carries the vLEI
Legal Entity ACDC credential directly as an embedded field. This
provides offline verification: the vLEI credential is available to
the verifier without requiring KERI infrastructure access.

**Referenced vLEI Credential.** The AGTP-CERT extension carries a
reference to the vLEI credential, typically as a KERI AID and an
OOBI (Out-Of-Band Introduction) hint indicating where to retrieve
the credential and its associated KEL (Key Event Log) for verification.

Implementations **MAY** use either form. The inline form is preferred
for high-volume agent deployments where verification latency matters.
The referenced form is preferred where vLEI credentials are managed
through dedicated KERI infrastructure and AGTP-CERT issuance is
decoupled from vLEI credential lifecycle.

## Extension Field Schema

The AGTP-CERT extension carries a new field for vLEI composition.
The schema is defined once and supports both the basic case (Legal
Entity vLEI only) and the extended case (Legal Entity vLEI plus a
chain of Role Credentials per {{authorization-chain}}):

~~~
vLEIBinding ::= SEQUENCE {
    bindingType            VLEIBindingType,
    leiCode                PrintableString (SIZE(20)),
    legalEntityCredential  OCTET STRING OPTIONAL,
    legalEntityAID         PrintableString OPTIONAL,
    roleCredentialChain    SEQUENCE OF RoleCredential OPTIONAL,
    oobiHint               IA5String OPTIONAL
}

VLEIBindingType ::= ENUMERATED {
    inline     (0),
    referenced (1)
}

RoleCredential ::= SEQUENCE {
    credentialType    RoleCredentialType,
    credentialData    OCTET STRING OPTIONAL,
    credentialAID     PrintableString OPTIONAL
}

RoleCredentialType ::= ENUMERATED {
    oor       (0),
    ecr       (1),
    auth      (2)
}
~~~

The `leiCode` field is **REQUIRED** in all cases and carries the
20-character LEI value, providing fast verification without
requiring credential parsing.

When `bindingType` is `inline`, the `legalEntityCredential` field
**MUST** be present and **MUST** carry the vLEI Legal Entity ACDC
credential. The `roleCredentialChain` **MAY** also be present
carrying inline `credentialData` for each role credential.

When `bindingType` is `referenced`, the `legalEntityAID` field
**MUST** be present and the `oobiHint` field **SHOULD** be present
indicating where to retrieve the KEL for verification. The
`roleCredentialChain` **MAY** also be present carrying
`credentialAID` references for each role credential.

The `roleCredentialChain` is **OPTIONAL** in all cases. When absent,
the binding establishes institutional identity through the Legal
Entity vLEI alone. When present, it additionally carries the human
authorization chain per {{authorization-chain}}.

## Verification Workflow

A verifier processing an AGTP-CERT with a vLEI Binding extension
proceeds as follows:

1. Extract the `leiCode` and confirm it matches the Owner-ID's LEI
   value (per {{owner-id-binding}}).

2. If `bindingType` is `inline`, parse the embedded
   `legalEntityCredential` and verify its signature chain through
   QVI to GLEIF root of trust per {{VLEI-EGF}}.

3. If `bindingType` is `referenced`, resolve the `legalEntityAID`
   through the `oobiHint` to retrieve the vLEI credential and KEL.
   Verify the key event log to establish current key state, then
   verify the credential signature chain.

4. Verify that the vLEI credential issuer is an accredited QVI per
   GLEIF's QVI registry.

5. If a `roleCredentialChain` is present, verify each credential in
   the chain through KERI signature verification per
   {{authorization-chain}}.

6. Apply the verified institutional identity (and the verified role
   credential chain, if present) to authority decisions per local
   policy.

# Verification Path {#verification-path}

## The `vlei-anchored` Verification Path

AGTP-TRUST defines verification paths (`dns-anchored`, `log-anchored`,
`hybrid`, `org-asserted`) that specify the evidence mechanism backing
a Trust Tier 1 assignment. This document defines a new verification
path value:

~~~
verification_path: vlei-anchored
~~~

A Tier 1 agent with `vlei-anchored` verification path has its
institutional identity anchored in the vLEI infrastructure rather
than in DNS ownership or transparency logs. The verification mechanism
relies on KERI key event verification and the vLEI Ecosystem
Governance Framework.

## Verification Procedure

Verifying a `vlei-anchored` Tier 1 agent proceeds as follows:

1. Resolve the agent's Owner-ID LEI to retrieve the legal entity
   record from GLEIF.

2. Retrieve the vLEI Legal Entity credential, either inline from
   AGTP-CERT or via KERI AID resolution.

3. Verify the credential's KERI signature chain through the QVI
   that issued it, terminating at the GLEIF root of trust.

4. Verify that the QVI is currently accredited per GLEIF's QVI
   registry, and that the credential has not been revoked.

5. If the agent carries vLEI Role Credentials (OOR, ECR, AUTH) per
   {{authorization-chain}}, verify the credential chain from the
   Legal Entity vLEI through the role credentials.

6. Confirm institutional identity matches the LEI carried in the
   Owner-ID and the AGTP-CERT extension.

## Composition with Other Verification Paths

The `vlei-anchored` path **MAY** compose with other verification paths
in a hybrid model. An agent might carry both DNS-anchored verification
(proving domain control) and vLEI-anchored verification (proving
institutional identity). The two are complementary:

- DNS-anchored verification proves the agent is deployed at a domain
  the institution controls.
- vLEI-anchored verification proves the institution is a regulated
  legal entity with cryptographic identity through GLEIF.

Verifiers **MAY** require both for Tier 1 acceptance in regulated
contexts. The compound verification path is represented as:

~~~
verification_path: hybrid
verification_evidence: ["dns-anchored", "vlei-anchored"]
~~~

# Authorization Chain {#authorization-chain}

## Human Authorization Through vLEI Role Credentials

The vLEI ecosystem extends institutional identity to human
authorization through three credential types. These credentials
provide the chain showing which human, in which role, authorized
the deployment and operation of an agent.

**Official Organizational Role (OOR) credentials** are issued by a
Legal Entity (or QVI on its behalf) to humans in roles recognized
under {{ISO5009}}. OOR credentials prove that a named human holds an
official role within the legal entity (such as Chief Executive
Officer, Chief Information Officer, Chief Risk Officer).

**Engagement Context Role (ECR) credentials** are issued by a Legal
Entity to humans in contextual roles for specific engagements. ECR
credentials prove that a named human is engaged with the legal entity
in a specified capacity (such as Project Lead, Authorized
Representative for Specific Engagement).

**Authorization (AUTH) credentials** are issued by a Legal Entity to
authorize specific actions. AUTH credentials are typically chained
from OOR or ECR credentials, establishing that the authorizer held a
valid role at the time of authorization.

## Agent Deployment Authorization

When an agent is deployed by a Legal Entity, the deployment
authorization **MAY** be expressed as a chain of vLEI credentials:

1. The Legal Entity vLEI establishes the institutional identity.

2. An OOR or ECR credential establishes the human officer's role
   authorized to deploy agents (typically a CIO, CTO, or designated
   AI governance role).

3. An AUTH credential issued by that officer authorizes the specific
   agent deployment, identified by the agent's Canonical Agent-ID.

The AUTH credential's authorization scope **MAY** include limits on
the agent's Authority-Scope, operational domain, financial exposure
limits, or other constraints meaningful to the institutional
governance.

## Carriage in AGTP

The vLEI Role Credential chain authorizing an agent **MAY** be
carried in the AGTP-CERT extension alongside the Legal Entity vLEI,
or referenced via KERI AIDs and OOBI hints. The unified `vLEIBinding`
schema defined in {{cert-composition}} accommodates the role
credential chain through its optional `roleCredentialChain` field,
which is a SEQUENCE OF the `RoleCredential` structure defined in the
same schema. When the role credential chain is populated:

- The `roleCredentialChain` field carries the chain of OOR/ECR/AUTH
  credentials linking the Legal Entity vLEI to the specific agent
  deployment authorization.
- Each `RoleCredential` in the chain carries either inline
  `credentialData` (for `inline` binding type) or a `credentialAID`
  reference (for `referenced` binding type), matching the parent
  `vLEIBinding`'s `bindingType`.
- The `RoleCredentialType` enumeration identifies whether the
  credential is an OOR, ECR, or AUTH credential.

Verifiers processing an agent with a role credential chain **MUST**
verify each credential in the chain through KERI signature
verification, establishing that the chain is intact from the Legal
Entity vLEI through the role credentials to the AUTH credential
authorizing the specific agent.

## Use in Authority Decisions

Authority decisions for agents with verified vLEI authorization chains
**MAY** consider the role credential context. For example:

- An AUTH credential issued under a CISO's OOR credential **MAY**
  authorize the agent for security-related operations within the
  institution's security policy domain.
- An AUTH credential issued under a Trading Desk Head's ECR credential
  **MAY** authorize the agent for trading operations within specified
  position limits.
- An AUTH credential issued under a Compliance Officer's OOR
  credential **MAY** authorize the agent for regulatory reporting
  operations.

The mapping from role credential to authority decision is governance
policy defined and is not specified in this document. What is
specified is the structural carriage of the credential chain so that
governance policy can evaluate it.

# Composition with Other AGTP Bindings

## Composition with AGTP-MERCHANT-IDENTITY

{{AGTP-MERCHANT-IDENTITY}} specifies how agents acquire merchant
identifiers analogous to payment network Merchant IDs. For agents
deployed by LEI-holding legal entities, the merchant identifier
**SHOULD** include or reference the LEI of the deploying entity.
This composition allows payment networks to verify both the
institutional identity (via LEI/vLEI) and the merchant identity
(via AGTP-MERCHANT-IDENTITY) of an agent transacting on behalf of
a regulated entity.

## Composition with AGTP-TRUST Credit Rating Providers

{{AGTP-TRUST}} positions credit bureau infrastructure (Experian,
Equifax, TransUnion, and similar) as composable trust providers for
agent trust scoring. For agents with vLEI-anchored verification, the
institutional identity provides high-quality input to trust scoring:
the LEI is a stable, regulated identifier that trust providers can
correlate with their existing institutional credit and compliance
records. Credit bureaus operating as agent trust providers **SHOULD**
prefer LEI-anchored institutional identity over self-declared
organizational identifiers when both are available.

## Composition with AGTP-Presence

AGTP-Presence partition dimensions include `owner-domain` for
organizational scoping. For LEI-holding entities, the partition
dimension **MAY** be expressed using the LEI directly:

~~~
owner-domain: lei:5493001KJTIIGC8Y1R12
~~~

This provides cryptographically anchored organizational scoping for
Presence overlays. Agents in the same LEI-scoped overlay are
verifiably deployed by the same legal entity.

# Security Considerations

This document inherits the security considerations of {{AGTP}},
{{AGTP-TRUST}}, {{AGTP-CERT}}, {{VLEI-EGF}}, and {{KERI}}. This
section addresses considerations specific to the composition.

## LEI Verification Independence

The LEI lookup against GLEIF's public registry **MUST** be performed
independently of any AGTP infrastructure operator's claims. An agent
operator that controls both the agent and a representation of LEI
verification is in a conflict-of-interest position. Verifiers
**SHOULD** consult GLEIF's authoritative LEI registry directly or
through trusted intermediaries that do not have operational dependency
on the agent being verified.

## vLEI Credential Revocation

vLEI credentials can be revoked through KERI key event mechanisms.
Verifiers **MUST** check current revocation status of vLEI credentials
during verification, not rely on cached or previously-verified state.
The KERI Key Event Log provides the authoritative revocation status.

## QVI Compromise

A compromised Qualified vLEI Issuer could issue fraudulent vLEI
credentials. GLEIF maintains the authoritative QVI registry; verifiers
**MUST** confirm that the issuing QVI is currently accredited at
verification time.

## Role Credential Misuse

Role credentials (OOR, ECR, AUTH) bind authorization to specific
humans in specific roles. If a role-holder leaves the legal entity
or changes roles, the corresponding credentials **SHOULD** be revoked
and re-issued as appropriate. Verifiers checking role-based
authorization **MUST** verify credential freshness through the KEL.

## Cross-Jurisdictional Considerations

LEI is internationally recognized but legal entity recognition varies
by jurisdiction. An LEI-anchored agent deployed by a regulated entity
in one jurisdiction may operate cross-jurisdictionally. Verifiers
**SHOULD** apply jurisdictional context to authority decisions where
relevant, particularly for financial services, securities trading,
and other regulated operations.

## Privacy of LEI Lookups

LEI lookups against GLEIF are public; querying an LEI does not reveal
operational intent. However, the pattern of LEI lookups may reveal
which institutional counterparties an agent is investigating.
Operators concerned about lookup patterns **MAY** cache LEI registry
data locally to reduce query frequency.

# IANA Considerations

## Verification Path Value

This document requests registration of a new verification path value
in the AGTP-TRUST verification path registry:

| Value | Semantics | Reference |
|-------|-----------|-----------|
| `vlei-anchored` | Institutional identity verified through vLEI infrastructure | This document |

## URI Scheme

This document requests registration of the `lei:` URI scheme for
identifying LEI values:

- Scheme name: `lei`
- Scheme syntax: `lei:` followed by 20 alphanumeric characters per {{ISO17442}}
- Status: Permanent
- Reference: This document

## AGTP-CERT Extension Field

This document requests registration of a new field in the AGTP-CERT
extension namespace:

- Field name: `vLEIBinding`
- OID: TBD (to be assigned)
- Description: Carries or references vLEI credentials for institutional identity
- Reference: This document

# Implementation Considerations

## Path to Pilot Deployment

A legal entity considering pilot deployment of AGTP with vLEI binding
follows roughly this path:

1. **Obtain LEI** (if not already held). LEI issuance is operated by
   accredited LEI Issuers. Most regulated financial entities and
   public companies already hold LEIs.

2. **Engage a Qualified vLEI Issuer.** QVIs are listed in GLEIF's
   public registry. The QVI issues the Legal Entity vLEI credential
   to the institution.

3. **Establish KERI infrastructure or KERI service relationship.** The
   institution either operates its own KERI infrastructure or uses
   a managed KERI service.

4. **Deploy AGTP-aware agents.** Agents are configured with Owner-IDs
   carrying the LEI, AGTP-CERTs with vLEI Binding extensions, and
   verification paths set to `vlei-anchored`.

5. **Issue role credentials.** As needed, the institution issues OOR
   and ECR credentials to humans authorized to govern agent
   deployment, and AUTH credentials authorizing specific agents.

6. **Verify interoperability.** Test agent communication with
   counterparties to confirm that vLEI-anchored verification operates
   correctly across organizational boundaries.

The path is staged so that institutions can adopt incrementally.
LEI in Owner-ID is the smallest step; vLEI binding with KERI
verification is the larger commitment.

## Adoption Considerations for Regulated Industries

For financial services institutions, the AGTP-LEI binding provides
direct alignment with existing regulatory infrastructure. The LEI is
already required for derivatives reporting under Dodd-Frank, EMIR,
MiFID II, and similar regimes. Extending the LEI into AGTP agent
identity provides regulators with continuity from existing entity
reporting to new agent-mediated reporting.

For payment networks (Visa, Mastercard, American Express, and similar),
the AGTP-LEI binding composes with {{AGTP-MERCHANT-IDENTITY}} to
provide both institutional identity (LEI) and merchant identity for
agents transacting through payment infrastructure. The composition
positions payment networks to extend their existing identification
infrastructure to agent commerce.

For supply chain and trade finance contexts, the LEI is already used
in electronic bills of lading, ESG attestations, and customs
declarations. AGTP-LEI extends this to agents acting on behalf of
trading parties.

# Engagement with GLEIF and the vLEI Ecosystem

This document is published as an Internet-Draft to invite engagement
from GLEIF, the Trust over IP Foundation, Qualified vLEI Issuers,
LEI Issuers, and institutional adopters of the LEI infrastructure.
The composition specified here builds on the operational maturity of
the LEI ecosystem and extends it into the agent infrastructure layer.

Feedback from GLEIF on the binding's alignment with the vLEI Ecosystem
Governance Framework is particularly welcome. The binding is designed
to compose with the vLEI ecosystem as specified rather than to modify
it; if the binding requires adjustment to align with vLEI governance,
this document will be revised accordingly.

--- back

# Acknowledgments

The author thanks Tom Roberts for the strategic introduction to GLEIF
and the broader LEI/vLEI ecosystem, and for the architectural framing
that positions AGTP-LEI as a natural composition rather than a
competitive alternative. The author also acknowledges the foundational
work of the GLEIF leadership, the KERI specification authors led by
Samuel M. Smith, and the Trust over IP Foundation's stewardship of the
vLEI ecosystem governance framework.

# Changes from v00

Initial version.
