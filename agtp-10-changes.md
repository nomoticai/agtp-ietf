# AGTP v10 Changes: Working List

Target: draft-hood-independent-agtp-10 and the companion revisions that follow from it
Started: 27 October 2026 · Nomotic / AGTP Project
Status: working list, rebuilt after the original v10 document went missing; items marked RECALL are placeholders for things still to be remembered

Sequencing rule from the v09 cycle carries forward: decide the structural changes (Part B) before writing text, since every companion reference depends on them.

---

## A. Language and positioning (suite-wide)

### A1. Transfer protocol, never transport layer

AGTP is a transfer protocol: it defines the semantics of an exchange. A transport protocol delivers the bytes. This is the definition by function that settled the ATA list description with the Area Directors, and the suite should say it the same way everywhere.

Rule: replace "transport-layer protocol," "transport layer," "transfer layer," and "transport protocol" (where it refers to AGTP itself) with "transfer protocol." TCP, QUIC, and TLS remain transport. "Substrate" stays where it describes AGTP's position beneath agent group messaging protocols; it is a role, and the role is played by a transfer protocol.

Current hit counts (transport-layer / transport protocol):

| Draft | Hits |
|---|---|
| independent-agtp-09 | 13 / 2 |
| agent-cert-03 | 7 / 0 |
| composition-01 | 3 / 2 |
| bindings-00 | 2 / 0 |
| communication-00 | 2 / 0 |
| merchant-identity-02 | 2 / 0 |
| identifiers-02 | 1 / 1 |
| session-00, trust-02 | 1 / 0 each |
| api-01 | 0 / 1 |

Each hit gets read in context before replacement; a sentence like "TLS is the transport" stays.

Priority: high. Effort: mechanical sweep plus a one-paragraph positioning statement in the base Introduction and Terminology.

### A2. A carrier is a claim only when the claim is specified

Two review rules from the agentproto security principal binding template now govern how the suite describes itself. A carrier (a header, a field) by itself implies no claim. An inherited mechanism is a current guarantee only when its dependency and its failure behavior are stated. Karthik Rampalli applied both to base-09 on the WIMSE list and the spec text confirmed the overclaim.

Known hits in base-09:

- Line 1193 correctly marks Delegation-Chain reserved.
- Line 4161 states workflows carry lineage in the Delegation-Chain header.
- Line 4217 maps A2A provenance chain to Delegation-Chain in the composition table.
- Line 6104 shows the header populated in a worked example.

A reader reaches the composition section and the example before the reservation note. Fix: either specify the header (C1) or rewrite those passages as planned, with the reservation cross-referenced in place.

Full sweep still to run: Authority-Scope ("narrowing," "per-hop" language versus the subset check specified), Attribution-Record and every passage that leans on AGTP-CERT, AGTP-LOG, an external CA, GLEIF, or SCITT infrastructure without naming the dependency and what happens when it is absent.

Priority: high. Effort: line-numbered hit list, then per-passage rewrites.

### A3. Specified, planned, inherited: one vocabulary

Adopt the template's status vocabulary inside the drafts themselves: specified, specified and implemented, planned, inherited or external. Every mechanism statement in base-10 that is less than specified carries the word. This is what lets the agentproto matrix rows be stated from the draft text rather than from a separate mapping.

Priority: medium. Effort: rides on the A2 sweep.

### A4. RECALL: other language called out on list

Placeholder for wording corrections from list feedback that are still to be remembered. Candidates from the record: the word family house style already bans as a description of agents, "governance" versus "enforcement" framing, "agent name" versus "agent address" after the DAWN taxonomy exchange.

---

## B. Document structure

### B1. AGTP-API becomes AGTP-METHODS

Rename draft-hood-agtp-api to draft-hood-agtp-methods. "API" invites the reading the base draft argues against, that this is an API riding on a transport. The document's first job is the method vocabulary, and v10 names it as the baseline.

What the methods document carries as the v10 baseline, from the reference catalog (core/methods.json, catalog version 1.0.0):

- 425 registered methods across 9 categories: discovery, retrieval, analysis, transaction, modification, creation, notification, mechanics, domain_spanning.
- 18 embedded floor methods every conformant server implements: QUERY, DISCOVER, DESCRIBE, SUMMARIZE, PLAN, EXECUTE, DELEGATE, ESCALATE, CONFIRM, SUSPEND, PROPOSE, NOTIFY, INSPECT, ACTIVATE, DEACTIVATE, REVOKE, REINSTATE, DEPRECATE.
- 5 legacy mappings: GET to FETCH, POST to CREATE, PUT to REPLACE, DELETE to REMOVE, PATCH to MODIFY.
- 459 Method Violation and 460 Endpoint Violation as the catalog enforcement codes.

Also stays in methods: path grammar, the endpoint primitive, server manifest, method policy, handler binding, conformance, the OpenAPI relationship, and the catalog evolution process.

Mechanics: new -00 under the new name, datatracker "replaces" relation pointed at draft-hood-agtp-api-01, and the [AGTP-API] reference updated across the suite in the same revision cycle. Expired draft-hood-agtp-standard-methods and draft-hood-independent-agis stay expired; the datatracker should show both replaced by the methods draft so the history reads cleanly.

Decision needed: whether the 425 figure is normative (the catalog at a stated version) or whether the draft specifies the floor plus the registry process and cites the catalog as the current registered set. Recommendation: the second. The floor and the process are stable; the count moves.

Priority: high; blocks C and every companion reference. Effort: rename, section reorder, one new section stating the baseline.

### B2. RCNS extracted to its own draft

Runtime Contract Negotiation Substrate becomes draft-hood-agtp-rcns-00 (working title; alternatives: agtp-contracts, agtp-negotiation). It currently lives inside api-01 as the PROPOSE and Synthesis chapter (section 11, roughly 580 lines) plus the Semantic Block chapter, with the RCNS configuration knobs, stale contract sweep, endpoint tiers, recipes, contract scoping, and observability surfaces. That is substrate behavior with a different audience from the method vocabulary.

Carries over: PROPOSE, synthesis_id lifecycle, the four-lock dispatcher gate, recipe versioning, contract scoping, on_policy_change and the stale contract sweep, 261/262/263 negotiation codes, 464 contract-revoked, rcns_* lifecycle events, DISCOVER /patterns and /contracts.

Base-10 keeps the RCNS section as a kernel (the substrate property and the PROPOSE floor method) and points to the companion, the same pattern used when trust was extracted from v07.

Open item carried from the code audit: on_policy_change contract invalidation is specified in api-01 and confirmed unwired in the reference server.

Priority: high. Effort: extraction plus a new front matter, abstract, and security considerations; the normative text already exists.

### B3. Drafts expired by design

draft-hood-agtp-web3-bridge-00 stays expired; no revival. standard-methods-01 and agis-01 stay expired per B1. Nothing else in the family is retired.

### B4. Slim the core to the transfer protocol

Base-10 should read the way RFC 9110 and RFC 5321 read: the protocol, and only the protocol. Companions are the modules and extensions. The HTTP split is the model: RFC 9110 carries semantics (methods, status codes, header fields) and RFC 9112 and 9113 carry wire bindings. AGTP already has the second half in bindings-00.

Base-09 is 6,152 lines. The transfer-protocol core (connection model, header format, status codes, wire format, floor methods, security, IANA, Authority-Scope format, wire examples) is roughly 2,000 of them. Candidates to move out, with line counts from base-09:

| Section | Lines | Goes to |
|---|---|---|
| Agent Identity Document and .agtp format | 408 | new companion, working title draft-hood-agtp-identity-document-00 |
| Agent Namespace Document | 68 | same companion |
| Agent Registration Process | 257 | discovery-02 (ANS) or the identity-document companion; decide |
| Browser and Human-Facing Interaction Model | 85 | informative, out to the identity-document companion or the website |
| Changes from v06, v07, v08 | ~680 | collapse to a one-page summary; full history stays in the repo archive |
| Extended Method Vocabulary and Industry Profiles | 242 | methods-00 (B1) |
| Dynamic Endpoint Negotiation | 142 | rcns-00 (B2) |
| RCNS section | 68 | rcns-00 kernel pointer stays, text moves (B2) |
| Composition section plus AGMP appendix | ~190 | composition-02; base keeps the substrate model and precedence rule as a few paragraphs |
| Merchant Identity summary | 86 | merchant-identity-03; base keeps one pointer paragraph and the 458 registry row |
| Glossary | 177 | merge into Terminology; keep one |

Core Methods (707 lines for the eighteen-method floor) stays and tightens. Problem Statement and Related Work stay but shorten; an independent submission needs the motivation, and the Vienna comparison document carries the long form.

Expected size after the move: near 2,800 lines.

One design question the SMTP analogy raises: ESMTP extensions are advertised at EHLO. The server manifest already plays that role for AGTP. Base-10 can state the manifest as the extension-advertisement mechanism, and each companion then registers what it advertises (a manifest block or capability token), which gives the module model a wire-visible shape rather than a documentary one.

Priority: high; it is the frame the rest of Part B fits into. Effort: section moves plus new front matter for the identity-document companion; the text exists.

---

## C. Base v10 substantive changes

### C1. Delegation-Chain specified (public commitment)

Committed on the WIMSE list, endorsed by Songbo Tan on agentproto. Specify the header with pre-authorization semantics: format and validation; per-hop scope attenuation checked at both mint and verify, the delegate's Authority-Scope derived from the delegator's and bounded by the never-exceeds invariant; parent-token re-verification at each hop, closing the parent-swap attack; verifier rule, freshness rule, accepted result, and failure behavior per check, stated in the template fields.

Decision flagged: subset-or-equal versus strict subset. Strict subset outlaws same-scope relay delegation. Recommendation: subset-or-equal as the rule, never-exceeds as the invariant, policy may tighten to strict.

Compose with draft-rampalli-pedigree and draft-rampalli-cross-org-delegation-mapping rather than conflict; read both before drafting. The identifiers draft keeps the post-execution attribution side (principal_id, prior_actions, the Section 5 verification procedure), and base-10 states the split explicitly.

### C2. Typed error taxonomy

Retryability, partial success, and side-effect status as structured fields on failure responses. Alexander Chernov's gap, confirmed in the agentproto matrix row C-009.

### C3. Revocation continuity

One specification of how revocation reaches a verifier across the certificate, session, and lifecycle boundaries, by effect time. Songbo's gap, matrix row C-008. Decide whether it lands in base-10 or as a short companion.

### C4. Tool and resource identity

Decide whether tool or resource identity becomes a first-class header or stays expressed through composition profiles. Matrix row C-006; the current position is composition, labeled as a deliberate choice under review.

### C5. v09 backlog, carried forward

- inspect:all scope token formally specified in the scope vocabulary as operator-only.
- Contract-revoked: resolve 404 versus 464 on presentation of an evicted synthesis_id; spec and code disagree.
- Trust score 0.0 to 1.0 range and trust_score_computed_at coupling pinned normatively (lands in trust-03).
- Trust-tier drift as a second dimension of REVOKE target=stale-contracts.
- Genesis issuer_public_key emission verified against the registrar.

### C6. Owner-ID and Principal-ID reconciled

Identifiers uses Owner-ID; base and the matrix mapping use Principal-ID in places. One term, one definition, in base-10, with identifiers-03 following.

### C7. Consistency report, Tier 1 conflicts

From AGTP-CONSISTENCY-REPORT.md (28 June 2026): 458 defined as Counterparty Unverified in core and merchant and as Insufficient Budget in commerce; 451 still used for Scope Violation in discovery and communication where core renumbered to 455; communication-00 marked Standards Track while every other draft is Informational. All three close in this cycle.

### C7b. Identity vocabulary drift in discovery

Discovery-01 uses "Agent Manifest Document" (nine times) and "Birth
Certificates" (once), both retired in v07 for Agent Identity Document
and Agent Genesis. Discovery-02 adopts the current terms, and the DAWN
mapping appendix (September changelog D2) is stated as Agent Identity
Document to AgentCard, since draft-jakab-dawn-agent-discovery-mdns-00
borrows A2A's AgentCard rather than defining its own record. Verify
the term the -01 uses before writing the appendix.

### C7a. Safe-method list drift

Base-09's summary table marks DESCRIBE, INSPECT, SUMMARIZE, and PLAN
as non-state-modifying and idempotent; bindings-00 admits only QUERY
and DISCOVER to early data. Either bindings-01 states the list is
deliberately conservative or it matches base. Methods-00 defers to
bindings for now.

### C8. Method floor drift

The reference server's embedded floor and base-08/api-01 agreed on eighteen; the September build plan found the server at the v07 twelve in one path. Reconcile before the ANS registration hook (fires on ACTIVATE) is documented in discovery-02.

### C9. Publish flow (mint, publish, resolve, search). LANDED 29 Oct

Walked the flow end to end: Genesis mints the Agent-ID; ACTIVATE publishes it (ANS registration per discovery-02, first ANNOUNCE per presence-01); a known name resolves through ANS to the Agent-ID; a known Agent-ID discovers its attachment through Presence; an unknown agent is found by search (DISCOVER intent mode). Four gaps closed: base-10 DISCOVER definition carried the v09 parameter set and disagreed with discovery-02 (base now keeps the floor and defers parameters and response to AGTP-DISCOVERY); base-10 ACTIVATE and presence-01 ANNOUNCE said nothing about each other (ACTIVATE is now the publish event, withdrawal transitions imply WITHDRAW); discovery-02 results carried a host-anchored manifest_uri (now the attachment record plus identity_document_uri); discovery-02 had no availability or budget parameters (available_only, max_cost, cost_unit added; commerce-01 owes the ANS-indexed pricing statement).

### C10. RECALL

Placeholder for base changes still to be remembered.

---

## D. Integration with other venues and documents

Each item states what base-10 says about the relationship, in a sentence or a short subsection, so the position is in the published record rather than in list archives.

### D1. agentproto

State the AGTP rows in the security principal binding template vocabulary (A3). Reference the AGTP mapping to draft-bu-agentproto-security-principal-binding as informative. Carry the substrate-versus-session-layer distinction from the Vienna comparison document into Related Work.

### D2. WIMSE

Workload identity and agent identity are different concerns that compose: WIMSE credentials authenticate the workload an agent runs on; AGTP-CERT authenticates the agent identity that persists across workloads. State it once in base-10 and once in agent-cert-04. The delegation split (C1) is the other WIMSE touchpoint.

### D3. DAWN and discovery

Discovery-02 and presence-01 carry the substantive work (the September changelog: /population, cross-ANS federation as implemented, the AMD to MDI mapping appendix for draft-jakab-dawn-agent-discovery-mdns, addressing abstraction, Attachment Record terminology). Base-10 needs the matching summary: the identity and location split (Canonical Agent-ID permanent, attachment short-lived and signed), the search versus discover terminology per RFC 6763, and the name versus address taxonomy from the DAWN exchange. RECALL: whether base-10 should also state the position on domain-anchored agent identifiers.

### D4. SCITT and transparency

Identifiers-03 states tamper evidence against a distrusted holder explicitly via AGTP-LOG inclusion. Base-10 references it from the Attribution-Record section.

### D5. aipref

Earmark stays a separate venue; base-10 adds nothing unless a usage-preference header is wanted at the wire. Default: nothing.

### D6. Agency evidence and the model gateway (candidate companion: agtp-attestation)

Decided 29 Oct, after the Terminology change in base-10 (C-item in Changes from v09, 21): an agent is defined constitutively, as a domain name is, by what the protocol mints and can enforce, never by what it is made of. The protocol cannot verify agency as an essence. What it can do is carry evidence of agency from parties other than the agent, under the carrier-versus-claim rule.

The enterprise framing: the problem is an allowlist, never a detector. Detection (scanning for API keys, MCP servers) loses to a costume. Port 4480 plus ALPN agtp gives network separability without decryption. The lever that closes the loop is the thing agents cannot do without: model access. An enterprise model gateway that refuses inference to callers without an Active Agent-ID turns rogue-agent detection into a policy question. Agents on AGTP that misbehave are governed, revoked, attributed; agent-shaped code that avoids AGTP cannot reach a model and is shadow IT.

Agency-evidence ladder, orthogonal to the identity verification levels in base-10:

- **Declared.** Agent Genesis and the Identity Document. The agent's own claim, signed by an issuer.
- **Bound.** Workload attestation (WIMSE / SPIFFE SVID, optionally TEE-backed) binding the Agent-ID to the code and host the principal registered. Proves the identity cannot be worn by other code; says nothing about agency. Home: agent-cert-04 Level 3 material and the WIMSE mapping in base-10.
- **Evidenced.** Inference receipt: the model gateway or provider signs a receipt bound to Agent-ID, Request-ID, and Audit-ID, carried in or referenced from the Attribution Record. Third-party evidence that a model inference preceded the action. A web server cannot produce one. Limits to state plainly: proves a model was consulted, never that it determined the action; requires gateway or provider cooperation, which the enterprise gateway case supplies today.

Scope decision pending: a new companion (draft-hood-agtp-attestation-00) carrying the receipt format, the gateway's AGTP participation (the gateway holds an Agent-ID and signs as a carrier), and the ladder; or the ladder in agent-cert-04 with the receipt deferred. Leaning companion, since the gateway is a new participant class. Not in the current bundle.

### D7. RECALL

Placeholder for other integration items (ai-control, SAMP, federated resolution).

### D8. Charter alignment and landscape refresh (reading task)

Venue facts from the 27 October mailbox review:

- agentproto (chartered 8 Oct; chairs Steele, Daigle; AD Eckel):
  "neither replaces application-layer agent protocols nor defines a
  new transport"; audit out of scope. AGTP fits Topics 3 and 4
  (requirements, transports and bindings) as a binding target, which
  decides H9 in favor of an H3/WebTransport binding. Topic 2
  (terminology) is live; Session-ID, Agent-ID, Authority-Scope absent.
- DAWN (chartered 22 Oct; chairs Liu, Brockners; AD Vyncke): local
  network, organization, or cooperating-party discovery on DNS or
  mDNS; identity, capability catalogs, Internet-wide search, trust
  evaluation out. Drafts in scope rename to -dawn-. Discovery-02's
  mapping targets the in-scope case only.
- AUDIT: chartering on audit@ietf.org, BoF at 127. Venue for AGTP-LOG,
  the identifier chain, and tamper evidence (D4), with IPR scrutinized.
- WIMSE: AIMS adopted; the sub/client_id/act chain convention (H5),
  Hulea's mandate, Reece's R10, Lee's tool-call proof, Thaha's
  evidence binding are the drafts to read.
- dmsc: no WG, no 127 session. SAMP -04 posted 27 Oct and may state
  the SAMP/AGTP relationship; read first.
- ATA list: awaiting Bishop since 14 Oct; chase.

Before base-10 text is final: read the current charters and charter proposals for agentproto, DAWN, WIMSE, and any new agent-related BoF or list since July, and record in one table what each charter scopes, which AGTP document answers it, and the sentence base-10 should carry. Second pass over the drafts posted since July to separate replication of existing work from ideas AGTP already holds and should state more cleanly. Output: a short addendum to the landscape catalog in agent-standards plus the D-section sentences here.

---

## E. Companion revisions in this cycle

Published 27 October (v10 set): independent-agtp-10, methods-00,
rcns-00, identity-document-00, session-01. Every remaining companion
needs the mechanical set: base reference to -10, AGTP-API citations
split to AGTP-METHODS or AGTP-RCNS by subject, transfer-protocol
wording (TLS-layer where the subject is TLS), house style sweep,
Changes section. The table carries what each needs beyond that, from
the 27 October survey.

| Draft | To | Expires | Forced by v10 | Substantive |
|---|---|---|---|---|
| agtp-communication | 01 | 19 Nov 2026 | DONE 29 Oct | Informational; media offer moved from four v00 headers into a `communication` ESTABLISH body extension (session-01 gained the extension rule and registry); 451 to 455; 422 for codec mismatch (463 is PROPOSE); carriage on QUIC/TCP/H3 bindings; streams belong to the connection and re-open after RESUME |
| agtp-log | 03 | 27 Nov 2026 | DONE 29 Oct | audit-id-committed and chain-head-committed statements with the log_inclusion reference object (D4); audit venue note; Genesis issued at registration before ACTIVATE (was: ACTIVATE produces Genesis); Agent-ID hash now excludes log_inclusion_proof (base-10 item 23, fixes the v02 receipt-ordering circularity); log_uri adopted into Genesis (base-10) and Identity Document (iddoc-00); bootstrap log_anchor named for namespace discovery |
| agtp-composition | 02 | 30 Dec 2026 | DONE 29 Oct | Owner-ID out of requests (document field); delegation by reference to base-10 (sequence + prior_actions; DELEGATE example no longer carries Delegation-Chain; token is parent-signed); MCP tools/call and sampling moved from QUERY to EXECUTE (QUERY is safe and 0-RTT eligible); Content-Schema specified here (base-10 planned-header table relabeled, item 24); Agent Card vs Identity Document; base-09 AGMP appendix absorbed; AGTP-API split into METHODS/RCNS |
| agtp-discovery | 02 | 30 Dec 2026 | DONE 29 Oct | search/discover/resolve vocabulary per RFC 6763; Identity Document and Genesis terms; Owner-ID header removed from examples; DAWN mapping appendix (lossy, Agent-ID as join); results carry the attachment record instead of manifest_uri; available_only / max_cost / cost_unit parameters (flow check 29 Oct). September D1 to D11 build items (/population, federation as implemented) still to fold in |
| agtp-agent-cert | 04 | 30 Dec 2026 | DONE 29 Oct | positioned as Level 3 of base-10 (base verifies at Level 2; intro no longer calls headers unverified); Relation to Request-Signature (same-key RECOMMENDED, distinct-key both verified, 0-RTT unchanged); owner-id equals Identity Document owner_id (header comparison retired); AGTP-CTL withdrawn, agent-cert-issued/revoked registered in AGTP-LOG, agtp-ctl-sct renamed agtp-log-receipt; D2 WIMSE sentence; Tier 1 wording corrected; TLS-layer wording |
| agtp-bindings | 01 | 30 Dec 2026 | DONE 28 Oct | connection versus session split; Request-Signature on early data; eligibility rationale (C7a); session methods prohibited; HTTP/3 and WebTransport binding (H9 decided: yes) |
| agtp-trust | 03 | 30 Dec 2026 | DONE 29 Oct | C5 closed: score without computed_at treated as absent (never grounds to reject the document); iddoc-00 now lists trust_score_computed_at as REQUIRED beside trust_score; Web3 section replaced by Resolution Paths Beyond DNS (bridge draft withdrawn); cert as Level 3, TLS-layer; refs pinned |
| agtp-lei | 01 | 30 Dec 2026 | DONE 29 Oct | G1 landed: owner as document field (header form withdrawn); AUTH credential as Delegation Sequence origin (authority = vlei:<SAID>, five checks, three outcomes); verification ladder (KYB / LEI in owner_id / vLEI) with failure behavior; GLEIF-unavailable security section; refs made explicit and pinned; IDDOC added |
| agtp-commerce | 01 | 30 Dec 2026 | DONE 29 Oct | Budget-Limit specified here in the base unit=value grammar (v00 JSON form withdrawn; 456 vs 462 distinguished); owner_auth replaced by Intent-Assertion with optional settlement_method claim; settlement_method a body parameter; 462 reason tokens; ANS indexing of pricing for max_cost/cost_unit (manifest authoritative); pricing_manifest_uri optional in iddoc-00; AGTP-LOG commerce-transaction-committed statement (digest and parties, never the record); TRANSACT registered via the methods catalog; 458/462 noted (commerce-00 already used 462); refs explicit and pinned; dangling AGTP-COMPOSITION cite fixed |
| agtp-presence | 01 | 30 Dec 2026 | DONE 29 Oct | vocabulary aligned (Presence is discovery, DISCOVER intent mode is search); attachment record specified (P3); first ANNOUNCE tied to ACTIVATE, WITHDRAW to withdrawal transitions (flow check 29 Oct); principals-rather-than-hosts and addressing abstraction in the introduction (P1, P2); references pinned. P4 to P6 (coordinator governance, rendezvous consistency, M2 data) still open |
| agtp-identifiers | 03 | 3 Dec 2026 | DONE 29 Oct | Owner-ID header registration removed (C6); delegation split with delegation_sequence_digest (C1); tamper evidence via log_inclusion (D4); three-outcome verification; session root composition |
| agtp-merchant-identity | 03 | 27 Nov 2026 | DONE 29 Oct | examples aligned to base-10 (Principal-ID header removed, Request-Signature, canonical IDs rather than URIs); hash rule with log_inclusion_proof excluded; DISCOVER restated as search with max_cost/cost_unit; 458 collision noted (commerce is 462); v01 document-type registrations withdrawn; AGTP-API to METHODS; WEB3 removed |
| agtp-ard | 01 | 20 Dec 2026 | rev bump; absent from the repo | obtain the source |
| aipref-earmark | 00 | 14 Feb 2027 | no change | H10 later |

Expired by design, no action: web3-bridge-00, standard-methods-01,
independent-agis-01. Batches: communication-01 and log-03 by their
November dates; identifiers-03 and merchant-03 with them; the Dec 30
group together.

## F. agtp.io alignment

The site is the first thing a reviewer reads after the datatracker.
Hits from the 27 October source:

- `https://agtp.io/api/methods.json` returns 404. The drafts cite it
  as the canonical catalog location; the file lives at
  `core/methods.json` in nomoticai/agtp. Publish at the cited path or
  change the citation. Blocks methods-00 submission.
- Footer: "A dedicated transport protocol and interface definition
  language for AI agent systems." A1 applies; the second half is
  AGIS-era framing and goes.
- Hero capability list names "agentic grammar." Remove; AGIS is
  expired and replaced.
- Badge row "Agentic API" becomes Methods when methods-00 posts.
- DOCS registry: replace the api-01 entry with methods-00 and
  rcns-00; session-00 to -01; decide whether web3-bridge stays
  listed as expired or is removed; ard-00 is listed but absent from
  the repo. Each revision needs its compiled HTML dropped at the site
  root, since the viewer loads `draft-hood-*.html` from there.
- Core description "transport-neutral; bindings specified separately"
  is correct as written (transport there means TCP and QUIC).
- No Delegation-Chain or Principal-ID claims on the site; the public
  summaries that assert cascading revocation and cycle bounding draw
  on composition-01 and published articles, which get the same
  treatment as the drafts (sweep A2e).

## G. Companion-specific items

### G1. lei-01

- Owner-ID carriage drift: lei-00 writes `Owner-ID: lei:...` as a
  request header (lines 250, 270). Owner-ID lives in the Agent Genesis
  and Identity Document and is resolved, never transmitted. Restate
  the binding as a document field and rewrite the examples.
- AUTH credential as Delegation Sequence origin: the origin hop's
  delegation token ({{delegation}} in base-10) MAY carry or reference
  the vLEI AUTH credential, so a sequence traces to a human signatory
  with a verifiable role. This is the signed-mandate model Ivan
  Voloshyn (Gleipnir, Vienna) described and the pre-execution human
  approval row of the agentproto template. New section.
- Verification ladder stated explicitly: out-of-band KYB (inherited,
  by the relying institution), LEI in the owner field (resolvable
  through GLEIF), vLEI binding (verifiable through KERI), each with
  what it proves and its failure behavior. Add the
  GLEIF-resolution-unavailable case to Security Considerations.
- AGTP-MERCHANT reference to -02; add AGTP-IDDOC; write the Changes
  section; expires 30 Dec 2026.

## H. Review-driven items (27 Oct mailbox analysis)

Source: the AGTP feedback and ideas summary of 27 October covering
seven critics, two private reviewers, three implementers, and nine
open questions. Items conceded on list on 17 August are marked.

### H1. Per-request proof of possession at Level 2 (conceded) : LANDED 27 Oct

Base-10 has none: Level 2 verifies the signed identity document and
never proves the live sender holds the key. Four independent readers
(Sirkkavaara via Sammartano, Williams, He, Bu); verifier matrix marks
C-001 AMBIGUOUS. Specify a `Request-Signature` header: compact JWS
over the request line, Agent-ID, Authority-Scope, Delegation-Chain if
present, a body digest, and a timestamp, signed with the key the
identity document publishes (the same key material the delegation
tokens use). Level 2 then means document verified plus possession
proved; Level 3 accelerates it. Highest-value item in the cycle.

### H2. Session-01 TBDs (conceded)

Today's session-01 is a rev bump and still binds the session
identifier to the TLS exporter or QUIC connection ID (Section 10.1),
leaves the identifier format TBD, and leaves ESTABLISH unspecified.
Before submission: stable session identifier format (256-bit random,
bound to the two Agent-IDs by signature), ESTABLISH wire format and
parameters, and re-binding on resumption through proof of possession
(H1) rather than channel binding. Cutoff 2 Nov for agentproto on
16 Nov.

### H3. 262 versus 455 rule : LANDED in base-10; methods-00, rcns-00, session-01 still to apply

Base-10 inherits the split (header table 262 scope-claim-invalid;
status section 455 generic; IANA table 455 for dimensions other than
Authority-Scope; delegation mint failures 262 per the DELEGATE note).
Rule: 262 when authorization is obtainable (missing credential,
consent, or a scope the agent could acquire); 455 when the request
exceeds authority the agent holds. A failed subset check is 455.
Apply in base-10, methods-00, rcns-00, session-01; reply to He and Bu.

### H4. Drop the HTTP registry argument : LANDED

Lines 311 to 313, 4605, 4622 to 4624 of base-10 say IETF Review
freezes the method registry and requires backward compatibility.
Wrong on the record (RFC 9110 is extensible); being checked. Rest the
case on the distinct wire image (own scheme, own port) and the
demonstrated failure cases. Comparison table row rewritten.

### H5. WIMSE chain convention mapping and staleness bound : LANDED

Subsection in Delegated Authority mapping Agent-ID, Authority-Scope,
Governance-Token, and the Delegation Sequence onto `sub` (principal at
every hop), `client_id` (current actor), `act` (prior-actor history),
and stating AGTP's staleness bound S+L (max age of authorization state
at admission, max permit lifetime without re-validation), with one
test vector on each limit. Cite Sweeney for narrowing as a key-by-key
subset over typed constraints and Hulea for the root-authority check
the origin hop performs.

### H6. Refusal versus incomplete check : LANDED

One more structured failure field: `evaluation` with values
`refused` (an evaluated decision) and `incomplete` (a check that could
unable to run, with `missing_input` naming the input). Forwarders carry an
incomplete result unchanged. Kroehl, Konda, Alygn.

### H7. Token-to-EXECUTE binding : LANDED

Governance Token carries a JCS action digest over EXECUTE's action,
parameters, payload_type, and payload hash; the verifier compares at
dispatch. Answers He's third question; Lee's local-tool-call-proof is
the model.

### H8. Positioning and replies

- Abstract: one sentence that the base is the transfer protocol and
  the companions are the modules, answering "too broad."
- Agent Owner stated against the user-agent versus service-agent
  distinction (Knodel).
- Linkable IPR paragraph on the site (two provisionals, open licenses,
  declared on datatracker); Part F.
- Replies owed, in order of what a reply changes elsewhere: Voloshyn
  (G1), Foroughi (ask which fields), He and Bu (H1, H3, H7, matrix
  envelope), Li (mandate lifecycle in Governance Token), Lear (H9),
  Roberts.
- agentproto Topic 2: short, draft-anchored posts carrying Session-ID,
  Agent-ID, Authority-Scope into the terminology thread.

### H9. H3 and WebTransport binding : LANDED in bindings-01, 28 Oct

Lear, Kuehlewind, and Sammartano converge on whether WebTransport or
H3 could carry the session goal. An H3/WebTransport binding in
bindings-01, as a third binding beside TCP/TLS and QUIC, costs nothing
architecturally and answers the incrementalism objection. Decide
before replying to Lear.

### H10. Earmark and C2PA

Rosenthol: C2PA 2.4 binds manifests to HTML and text with an IANA Link
header. Earmark-01 needs a stated reason to exist beyond C2PA or
should profile it. Out of this cycle unless decided otherwise.

### H11. Alignment with draft-hulea-wimse-agent-mandate-00 : LANDED 28 Oct (token format, three outcomes, fixed inputs, authority evidence, attribution table; typed constraints stated planned)

Hulea (Braincap, 24 Oct) separates the Authority Assertion (what the
delegator could delegate, from an Authority Source) from the mandate,
makes validation a deterministic function of a fixed input set driven
by a Validation Profile, and attributes every validated claim to the
artifact and role that asserted it. Base-10's origin-hop check is his
Authority Assertion check with the registrar as the source.

Patches for base-10 or -11, one paragraph each:

- Chain verification returns valid, invalid, or indeterminate;
  indeterminate when a membership or freshness input is unavailable,
  carried as 551 with `evaluation: incomplete` and `missing_input`,
  and the relying party denies by default.
- A verifier takes the fixed input set of {{delegation-verification}}
  and applies no local policy; the relying party's policy is applied
  to the result afterwards. This is what makes two conforming
  verifiers produce the same result.
- Attribution table for delegation artifacts: what the Genesis, the
  Identity Document, a hop token, and the origin check each assert,
  and what none of them asserts (that the agent was unmanipulated).

Planned, stated as such: typed constraints on Authority-Scope
(predicates over JSON Pointers, decimal-string amounts, budget
accumulation) for payment-grade mandates; AGTP carries Budget-Limit
only. lei-01 (G1): the AUTH credential as an Authority Assertion the
origin hop carries or references, in Hulea's format if the group
adopts one. AGTP does not compete for the chain format; it carries
whichever the group settles on.

## Sequencing

0. Nov 2 package: H1 in base-10, H2 in session-01, H3 across the
   family. Then the rest of this list.
1. session-01 submitted this week. SUPERSEDED by step 8: session-01 goes in the bundle with everything else.
2. B1 and B2 decided (names, what moves), since every reference depends on them.
3. A1 and A2 sweeps run against base-09 to produce the hit lists.
4. Base-10 drafted: structure first, then C1, then the rest of C, then D.
5. Methods-00 and rcns-00 cut from api-01 in parallel with step 4.
6. Companions revised in the order of their expiry dates, each carrying A1 and its row from Part E.
7. Consistency report re-run across the new set before any submission.
8. Publication is one bundle (decided 29 Oct): nothing is uploaded until every draft in the family is bumped, so the whole set shares one expiration date. Until then every document, base-10 included, stays open to improvement. Current state: base-10, iddoc-00, methods-00, rcns-00, session-01, bindings-01, identifiers-03, discovery-02, presence-01 built and in the publish bundle; communication-01, log-03, merchant-identity-03, composition-02, agent-cert-04, trust-03, lei-01, commerce-01 still to build; ard-01 source still to obtain.
