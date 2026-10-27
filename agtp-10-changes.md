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

### C8. Method floor drift

The reference server's embedded floor and base-08/api-01 agreed on eighteen; the September build plan found the server at the v07 twelve in one path. Reconcile before the ANS registration hook (fires on ACTIVATE) is documented in discovery-02.

### C9. RECALL

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

### D6. RECALL

Placeholder for other integration items (ai-control, SAMP, federated resolution).

### D7. Charter alignment and landscape refresh (reading task)

Before base-10 text is final: read the current charters and charter proposals for agentproto, DAWN, WIMSE, and any new agent-related BoF or list since July, and record in one table what each charter scopes, which AGTP document answers it, and the sentence base-10 should carry. Second pass over the drafts posted since July to separate replication of existing work from ideas AGTP already holds and should state more cleanly. Output: a short addendum to the landscape catalog in agent-standards plus the D-section sentences here.

---

## E. Companion revisions in this cycle

| Draft | From | To | Driver | Expires |
|---|---|---|---|---|
| agtp-session | 00 | 01 | rev bump now; v10 items in 02 | 1 Nov 2026 |
| independent-agtp | 09 | 10 | Parts A, C, D | 30 Dec 2026 |
| agtp-api | 01 | methods-00 | B1 | 27 Nov 2026 |
| (new) agtp-rcns | | 00 | B2 | |
| agtp-identifiers | 02 | 03 | C1 split, C6, D4 | 3 Dec 2026 |
| agtp-agent-cert | 03 | 04 | A1, D2 | 30 Dec 2026 |
| agtp-trust | 02 | 03 | C5 trust score normatives | 30 Dec 2026 |
| agtp-log | 02 | 03 | D4, A1 | 27 Nov 2026 |
| agtp-merchant-identity | 02 | 03 | A1, C7 | 27 Nov 2026 |
| agtp-commerce | 00 | 01 | C7 (458) | 30 Dec 2026 |
| agtp-discovery | 01 | 02 | September changelog, C7 (455) | 30 Dec 2026 |
| agtp-presence | 00 | 01 | September changelog | 30 Dec 2026 |
| agtp-communication | 00 | 01 | C7 (455, track), A1 | 19 Nov 2026 |
| agtp-composition | 01 | 02 | A1, A2, C4 | 30 Dec 2026 |
| agtp-bindings, agtp-lei, agtp-ard | 00 | 01 | rev bump, A1 | Dec 2026 |
| aipref-earmark | 00 | | no change | 14 Feb 2027 |

Expired by design, no action: web3-bridge-00, standard-methods-01, independent-agis-01.

---

## Sequencing

1. session-01 submitted this week.
2. B1 and B2 decided (names, what moves), since every reference depends on them.
3. A1 and A2 sweeps run against base-09 to produce the hit lists.
4. Base-10 drafted: structure first, then C1, then the rest of C, then D.
5. Methods-00 and rcns-00 cut from api-01 in parallel with step 4.
6. Companions revised in the order of their expiry dates, each carrying A1 and its row from Part E.
7. Consistency report re-run across the new set before any submission.
