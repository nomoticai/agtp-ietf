# AGTP Draft Suite — Cross-Document Consistency Report

**Date:** 2026-06-28
**Scope:** Newest version of every draft in the repository root (archive excluded).
**Method:** Each document was extracted into a structured fact sheet (identifiers, status codes, media types, headers, crypto, JSON fields, cross-references, version strings), then cross-compared. Hard conflicts were re-verified against the source files.

## Documents in scope (newest of each)

| Doc | Ver | Track |
|---|---|---|
| `independent-agtp` (core) | **09** | Informational |
| `agtp-agent-cert` | **03** | Informational |
| `agtp-identifiers` | **02** | Informational |
| `agtp-trust` | **02** | Informational |
| `agtp-log` | **02** | Informational |
| `agtp-merchant-identity` | **02** | Informational |
| `agtp-api` | **01** | Informational |
| `agtp-discovery` | **01** | Informational |
| `agtp-composition` | **01** | Informational |
| `agtp-bindings` | **00** | Informational |
| `agtp-lei` | **00** | Informational |
| `agtp-presence` | **00** | Informational |
| `agtp-commerce` | **00** | Informational |
| `agtp-web3-bridge` | **00** | Informational |
| `agtp-session` | **00** | Informational |
| `agtp-communication` | **00** | **Standards Track** ⚠ |

---

## TIER 1 — Hard conflicts (break interoperability or directly contradict)

### 1.1 Status code `458` is defined two incompatible ways ❗
- **core-09** (line 1312) and **merchant-identity-02**: `458 = Counterparty Unverified` (PURCHASE merchant-verification failure).
- **commerce-00** (line 1035, 502): `458 = Insufficient Budget`, registered to *"This document"*.
- Core pairs budget failures with **`456` Budget Exceeded** (lines 1310, 1242). Commerce's reuse of `458` is a direct registry collision: the same code means "merchant unverified" in one branch of the suite and "out of money" in another.
- **Fix:** commerce-00 should use `456` (or request a new code); it must not re-register `458`.

### 1.2 "Scope Violation" is `455` in some docs, `451` in others ❗
- **core-09** (1309), **agent-cert-03**, **merchant-identity-02** use **`455` Scope Violation**, and core/cert explicitly note the value was **renumbered `451 → 455`**.
- **discovery-01** and **communication-00** still emit **`451` Scope Violation**.
- **session-00** uses **both** `451` and `455` for Scope Violation within the same document.
- `451` is a registered HTTP code (Unavailable For Legal Reasons), so this is also a semantic clash. The pre-renumber docs are stale.

### 1.3 Protocol version field value disagrees: `0.7` vs `1.0`
- **core-09**: `"agtp_version": "0.7"` (lines 2033, 2203) — "version of the AGTP protocol the agent speaks."
- **api-01**: `"agtp_version": "1.0"` (plus `agtp_api_version: "1.0"`, `document_version: "v2"`).
- The same JSON field carries different protocol-version values depending on which doc you read. (Note the wire token is `AGTP/1.0` in both — see 1.4 — so `agtp_version: "0.7"` in core is itself internally odd.)

### 1.4 Wire protocol token: `AGTP/1.0` vs `HTTP/AGTP/1.0`
- Every doc with wire examples uses `AGTP/1.0` (discovery, composition, api, merchant, core).
- **communication-00** (lines 206, 227) uses **`HTTP/AGTP/1.0`** on both request and response lines — a malformed hybrid that no other doc recognizes.

### 1.5 Retired `Principal-ID` header still used as the controlling-entity axis
- **core-09** retired the **`Principal-ID`** wire header in favor of **`Owner-ID`**. **identifiers-02** and **composition-01** track this rename (composition-01 §5 explicitly renames it across all mapping tables).
- Still emitting `Principal-ID` in wire/example blocks: **discovery-01**, **session-00**, **merchant-identity-02**, **commerce-00** (`principal-cfo-bigco`).
- Caveat: `principal_id` as an *Attribution-Record field* legitimately still exists (it is distinct from the `Owner-ID` header — see agent-cert-03). The drift is specifically the **wire header name**.

### 1.6 Default body media type: retired `application/agtp+json` still in use
- **core-09** renamed the default wire content type **`application/agtp+json` → `application/vnd.agtp+json`** (changelog item 12).
- Still using the retired `application/agtp+json`: **discovery-01**, **communication-00**, **composition-01**, **merchant-identity-02**.
- **api-01** uses the new vendor tree (`application/vnd.agtp.manifest+json`). So within the current suite three different media-type families coexist for AGTP bodies: `application/vnd.agtp*`, `application/agtp+json`, and the non-vendor-tree `application/agtp-pricing+json` (commerce) / `application/agtp-log-statement+cose|cbor` (log).

---

## TIER 2 — Version-reference drift (nothing points at the current core)

### 2.1 No satellite document references core `-09`
Core was bumped `08 → 09`, but every companion still cites an older base. Verified `Internet-Draft:` seriesinfo lines:

| Doc | Cites core as | Should be |
|---|---|---|
| discovery-01 | `independent-agtp-02` | -09 |
| web3-bridge-00 | `independent-agtp-02` | -09 |
| composition-01 | `independent-agtp-08` (but ref body still literal `-02`, line 649) | -09 |
| session-00 | `independent-agtp-06` | -09 |
| communication-00 | `independent-agtp-07` | -09 |
| identifiers-02 | `independent-agtp-07` | -09 |
| agent-cert-03 | `independent-agtp-08` | -09 |
| trust-02 | `independent-agtp-08` | -09 |
| log-02 | `independent-agtp-08` | -09 |
| merchant-identity-02 | `independent-agtp-08` | -09 |
| api-01 | `independent-agtp-08` | -09 |
| bindings-00 | `independent-agtp-08` | -09 |
| presence-00 / lei-00 / commerce-00 | unversioned `I-D.hood-independent-agtp` | (style mismatch) |

`discovery-01` and `web3-bridge-00` pinning `-02` is the most severe — that base is seven revisions stale, and **web3-bridge-00 makes hard section references into the base (§6.6, §6.6.2, §6.1.6, §6.7.7, §9.6)** that were almost certainly re-numbered between `-02` and `-09`. Those pointers should be treated as broken until re-checked.

### 2.2 Core-09 itself cites stale companion versions
The core's reference list points at older companions than exist in the repo:
- `AGTP-IDENTIFIERS` → cited **`-01`**, actual is **`-02`**.
- `AGTP-MERCHANT` → cited **`merchant-identity-01`**, actual is **`-02`**.
(Core's other companion citations — cert-03, trust-02, log-02, discovery-01, composition-01, bindings-00, presence-00, lei-00, commerce-00 — match.)

### 2.3 Companion-to-companion citations are widely stale
| Citing doc | Cites | As | Actual |
|---|---|---|---|
| identifiers-02 | AGTP-TRUST | `-00` | `-02` |
| identifiers-02 | AGTP-CERT | `-01` | `-03` |
| identifiers-02 | AGTP-API | `-00` | `-01` |
| agent-cert-03 | AGTP-IDENTIFIERS | `-01` | `-02` |
| trust-02 | AGTP-CERT | `-01` | `-03` |
| trust-02 | AGTP-IDENTIFIERS | `-01` | `-02` |
| log-02 | AGTP-CERT | `-01` | `-03` |
| log-02 | AGTP-TRUST | `-01` | `-02` |
| log-02 | AGTP-IDENTIFIERS | `-01` | `-02` |
| merchant-identity-02 | AGTP-CERT | `-01` | `-03` |
| merchant-identity-02 | AGTP-TRUST | `-01` | `-02` |
| merchant-identity-02 | AGTP-IDENTIFIERS | `-01` | `-02` |
| merchant-identity-02 | AGTP-DISCOVER | `-00` | `-01` |
| api-01 | AGTP-CERT | `-01` | `-03` |
| api-01 | AGTP-IDENTIFIERS | `-01` | `-02` |
| api-01 | AGTP-TRUST | `-01` | `-02` |
| discovery-01 | AGTP-CERT | `-00` | `-03` |
| session-00 | AGTP-CERT | `-00` | `-03` |
| session-00 | AGTP-LOG | `-00` | `-02` |
| web3-bridge-00 | AGTP-CERT | `-00` | `-03` |
| lei-00 | AGTP-MERCHANT | `-01` | `-02` |
| commerce-00 | AGTP-MERCHANT | `-01` | `-02` |

**composition-01 is the only document whose cross-references are uniformly current** (cert-03, identifiers-02, trust-02, log-02, discovery-01, presence-00, lei-00, commerce-00) — a useful template for what "synced" looks like.

### 2.4 Inconsistent citation style
`presence-00`, `lei-00`, and `commerce-00` reference all AGTP drafts by **unversioned** `I-D.hood-*` (auto-latest), while everyone else **pins** explicit `-NN`. Mixing the two styles within one suite means some docs silently float and others freeze.

### 2.5 Stale changelog / reference-body text
- **merchant-identity-02** changelog (lines 1134–1135) still reads *"Base AGTP reference updated from `-04` to `-05`"* even though its seriesinfo is now `-08` — the changelog was never updated across the `-05 → -08` bumps.
- **composition-01** (line 649) reference body still contains the literal string `draft-hood-independent-agtp-02 ...` even though the seriesinfo was updated to `-08`.

### 2.6 The same companion is cited under three different titles
`AGTP-IDENTIFIERS` (actual title in `-02`: **"AGTP Identifier Chain"**) is referenced as:
- "AGTP Identifier Stack and Attribution-Record" (agent-cert-03, log-02)
- "AGTP Identifier Stack: Identifiers and Per-Agent Audit Chain" (trust-02, merchant-identity-02)
- "AGTP Identifier Chain" (its own front matter)

---

## TIER 3 — Semantic / terminology drift

### 3.1 Agent-ID derivation formula differs in the doc that owns it
- **core-09** and **agent-cert-03**: `sha256(canonical_form(Agent_Genesis_without_signature))`.
- **identifiers-02** (the normative source for identifiers): `sha256(canonical_json(agent_genesis))` — **no "without signature" qualifier**, and `canonical_json` vs `canonical_form`.
- Whether the signature is included in the hash input is determinative for the resulting Agent-ID. The identifier authority and the core disagree on the single most important hash in the system.

### 3.2 "Agent Manifest Document" — a retired term — is still in active use
- **core-09** renamed **"Agent Manifest Document" → "Agent Identity Document"** (media type `application/vnd.agtp.identity+json`).
- **discovery-01** still returns "Agent Manifest Document" objects and a **`manifest_uri`** field. (Note: this is separate from the legitimate *server* manifest in api-01 — discovery is using the old name for the per-agent document.)

### 3.3 Lifecycle state names and casing diverge
- **core-09** defines states lowercase: `active`, `suspended`, `retired`, `deprecated` — but its own examples use capitalized `Active`.
- **discovery-01** and **merchant-identity-02** use `Active`, `Suspended`, **`Revoked`**, `Deprecated`.
- "Revoked" vs core's "retired" is a genuine name mismatch (core has a `REVOKE` method and an `agent-genesis-revoked` log event, but lists the *state* as `retired`). Casing and the retired/revoked split should be reconciled.

### 3.4 `trust_warning` tokens are doc-specific and unregistered
- **trust-02** (the registry owner) defines only `verification-incomplete`, `verification-path-unsupported`.
- **merchant-identity-02** uses `legal-entity-unverified` (Tier 2).
- **web3-bridge-00** uses `org-label-unverified` (Tier 2).
Neither extension token is registered in trust-02's Trust Warning Registry. Either register them or unify.

### 3.5 Two overlapping, non-aligned commerce models
- **merchant-identity-02** models agentic commerce with **`PURCHASE` / `QUOTE`**, `Cart-Digest`, `Intent-Assertion`, `458 Counterparty Unverified`.
- **commerce-00** models it with **`TRANSACT`**, a `application/agtp-pricing+json` Pricing Manifest, and `458 Insufficient Budget`.
The two drafts describe the same problem space with different verbs, different status-code semantics (see 1.1), and no stated relationship. A reader cannot tell whether they compose, overlap, or compete.

### 3.6 Methods used without a defining document / outside the floor
- **composition-01** invokes **`COLLABORATE`** and **`LEARN`** in wire examples; neither appears in api-01's 18-method floor or any method registry seen.
- **bindings-00** classifies **`PURCHASE`** and **`TRANSACT`** alongside floor methods; these are commerce extensions (merchant/commerce), not floor methods.
- This ties back to the core's own unsettled **method-floor count** (see 4.1).

### 3.7 `Owner-ID` value syntax is defined ad hoc downstream
- **identifiers-02** defines `Owner-ID` only as a generic ABNF (`1*256(ALPHA / DIGIT / "-" / "_" / ":" / ".")`) plus an `id_scheme` field — it does **not** define concrete prefixes.
- **lei-00** introduces `lei:` *and* `owner:` prefixes; **commerce-00** uses `lei:`; **composition-01** mixes `usr-…`, `lei:…`, `owner:…`. These prefix schemes are used as if normative but are never registered in the identifiers draft's "Identifier Schemes" registry.

### 3.8 Analogous scores have different field names
- **discovery-01**: `behavioral_trust_score`, `capability_match_score`.
- **merchant-identity-02**: `merchant_reliability_score` (explicitly "replaces `behavioral_trust_score`"), `catalog_match_score` (replaces `capability_match_score`).
If the merchant rename is intended to be global, discovery is stale; if it's merchant-local, the relationship should be stated.

### 3.9 Signing algorithm is not consistent across contexts
- **core-09** mandates **Ed25519** for Agent Genesis / manifest signatures.
- **discovery-01** (`ans_signature`) and **merchant-identity-02** (`quote_signature`, Attribution-Record) use **`ES256`** in all examples.
- **log-02** permits `ES256/ES384/ES512/EdDSA`.
No document states a crypto-agility policy reconciling a hard Ed25519 mandate at the genesis layer with ES256 at the service layer. At minimum this needs an explicit "which signatures use which algorithm" statement.

### 3.10 JSON field-naming convention split
- **commerce-00** uses **camelCase** (`agentId`, `ownerId`, `transactionId`, `specVersion`, `disputeWindow`).
- Every other doc uses **snake_case** (`agent_id`, `owner_id`, `agtp_version`, …).
- **web3-bridge-00** alone spells the agent-id key three ways (`agtp-agent-id`, `agent.agtp.id`, `agtp_agent_id`).

### 3.11 Agent-ID representation: hex hash vs `agtp://` URI
- Most docs: Agent-ID = 64-char lowercase hex (= 256-bit). **log-02** describes it as a 32-byte CBOR byte string (same value, CBOR encoding — fine).
- **commerce-00** uses `"agentId": "agtp://agents.acme.com/concierge"` — i.e. it stuffs the **locator URI** into the field that elsewhere holds the **canonical hash**. These are different objects in core's model (Form 3 URI vs Agent-ID), and conflating them in the commerce record is a latent bug.

---

## TIER 4 — Internal inconsistencies within a single newest document

### 4.1 core-09: method-floor count contradicts itself
The abstract says **"eighteen-method"** in one place and **"twelve-method"** in another; a section heading says **"The Sixteen-Method Floor"** while its body says **"eighteen … MUST support all eighteen."** The actual Method Summary Table lists **18** (7 cognitive incl. `INSPECT` + 6 mechanics + 5 lifecycle). The "12-method" framing silently drops `INSPECT` and all five lifecycle methods. Pick one number and propagate it (composition-01 and api-01 both assume **18**).

### 4.2 core-09: protocol version expressed three ways
`agtp_version: "0.7"` vs wire token `AGTP/1.0` vs `document_version/schema_version: "1.0"` vs draft number `-09`. Four numbering schemes for one protocol (see also 1.3).

### 4.3 core-09: status field casing
States defined lowercase (`active`/…) but examples use `"status": "Active"` and `cert_status: "Active"`.

### 4.4 lei-00: two conflicting `vLEIBinding` ASN.1 definitions
The first SEQUENCE uses `vleiCredential` / `keriAID`; the second (extended) SEQUENCE uses `legalEntityCredential` / `legalEntityAID`. Same structure, different field names, in the same document.

### 4.5 merchant-identity-02: IANA table contradicts the v02 unification
The body retires the separate **Merchant Genesis** and **Merchant Manifest** document types (unifying on `role: "merchant"`), but the IANA "Document Type Registrations" still registers `merchant-genesis` and `agtp-merchant-manifest` pointing at sections (3.1, 3.4) that no longer describe them. Also: §5 retires the `result_type` DISCOVER param in favor of `result_class`/`role`, yet the Deployment Considerations section still says DISCOVER "serves both result types through the `result_type` parameter," and example URIs still use the **retired `/merchant/…` path form** (`agtp://acme.tld/merchant/acme-commerce`).

### 4.6 session-00: `451` and `455` both used for Scope Violation (see 1.2).

### 4.7 web3-bridge-00: broken cross-reference + dangling sentence
"…verified through DNS ownership challenge **per,** and bound to…" — a missing `{{...}}` xref left a sentence truncated.

### 4.8 api-01: "v00" throughout a `-01` document
Body repeatedly says "for v00 of this document," "v00 conforming servers," "v00 supports key-rename only," despite the filename being `-01`.

### 4.9 trust-02: "v00" wording in a `-02` document
"RECOMMENDED for v00 implementations," "This document does not request any IANA actions **in v00**." Self-version references were not bumped.

---

## TIER 5 — Metadata / status-track inconsistencies

### 5.1 communication-00 is Standards Track in an all-Informational suite
`category: std` while all 15 other current drafts are `category: info`. It is also **missing the `workgroup: "Independent Submission"` line** that the others carry. (An Independent Submission cannot normally be Standards Track — likely an error.)

### 5.2 Missing/!inconsistent front-matter dates
Most drafts carry `date: 2026` (no month/day) or no explicit date; examples inside them use specific 2026 dates. Harmless for now but should be normalized before submission.

### 5.3 Protocol name
Core titles AGTP **"Agent Transfer Protocol"** (consistent across trust-02 and others). The task framed it as "Agent **Transaction** Protocol" — that expansion appears nowhere in the docs. Worth confirming the canonical expansion, since "Transaction" would be an easy thing to get wrong in prose or marketing.

---

## Recommended remediation order

1. **Resolve the `458` collision** (1.1) and the **`451`/`455`** split (1.2) — these silently break interop and corrupt the IANA status-code registry the suite is trying to establish.
2. **Mechanical version-reference sweep** (Tier 2): bump every cross-reference to the current revision, settle on pinned-vs-unversioned style, and fix the stale changelog/ref-body text (2.5). Use composition-01 as the reference for "fully synced."
3. **Unify the wire surface**: one protocol token (`AGTP/1.0`, fix communication-00), one default media type (`application/vnd.agtp+json`), one controlling-entity header (`Owner-ID`), one `agtp_version` value.
4. **Reconcile the identity core** (3.1–3.3, 3.7): one Agent-ID derivation formula, one set of lifecycle state names + casing, registered `Owner-ID` schemes, retire "Agent Manifest Document."
5. **Clarify the commerce story** (3.5) and the **method floor** (3.6, 4.1): state the merchant↔commerce relationship, and define or remove `COLLABORATE`/`LEARN`.
6. **Fix the per-document internal defects** (Tier 4) and the **communication-00 track/workgroup** metadata (5.1).

---

*Generated by automated cross-document extraction + verification. Hard conflicts (Tier 1) and all version-reference claims were re-checked against source lines; semantic-drift items (Tier 3) reflect the newest text of each draft and may in some cases be intentional — they are flagged for author confirmation, not asserted as errors.*
