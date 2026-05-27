# EntityMap Specification

| | |
| --- | --- |
| **Version** | 1.0 |
| **Status** | Stable |
| **Date** | 2026-04-07 |
| **Authors** | Fred Laurent · Dixon Jones |
| **License** | [CC BY 4.0](../../LICENSE) |
| **Files** | `entitymap.json` · `entitymap.html` |
| **Canonical URI** | https://entitymap.org/spec/v1.0 |

## Contents

1. [Abstract](#abstract)
2. [Conventions and terminology](#1-conventions-and-terminology)
3. [Motivation](#2-motivation)
4. [Conformance floor — minimum valid file](#3-conformance-floor)
5. [File conventions](#4-file-conventions)
6. [JSON structure](#5-json-structure)
7. [Entity type system](#6-entity-type-system)
8. [Standard predicate vocabulary](#7-standard-predicate-vocabulary)
9. [Extension profiles](#8-extension-profiles)
10. [The HTML companion file](#9-the-html-companion-file)
11. [Validation](#10-validation)
12. [Consumer conformance levels](#11-consumer-conformance-levels)
13. [Versioning and evolution](#12-versioning-and-evolution)
14. [Privacy and security](#13-privacy-and-security)
15. [Relationship to existing standards](#14-relationship-to-existing-standards)
16. [Reference implementation](#15-reference-implementation)
17. [Consumer attribution guidance *(non-normative)*](#16-consumer-attribution-guidance-non-normative)
18. [Appendix A — Minimal valid example](#appendix-a-minimal-valid-example)
19. [Appendix B — Complete predicate reference](#appendix-b-complete-predicate-reference)
20. [Appendix C — Changelog](#appendix-c-changelog)

---

## Abstract

EntityMap is an open standard for publishing a structured, entity-first index of a website's content, designed for consumption by AI agents, large language models, and RAG pipelines.

Where `sitemap.xml` tells crawlers *what pages exist*, `entitymap.json` tells AI systems *what a site knows* — which entities it covers, how they relate, and where the evidence is.

EntityMap v1.0 is a publisher-assertion standard. It requires a small mandatory core — roughly 12 fields across three objects. Everything beyond that core is optional enrichment that improves reasoning quality, attribution, and graph depth without affecting basic conformance. The mandatory floor is described in [§3](#3-conformance-floor).

---

## 1. Conventions and terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) when, and only when, they appear in all capitals.

A **publisher** is the legal or commercial entity claiming authorship of the content described by an EntityMap. A **consumer** is any system that reads or processes an EntityMap — typically a RAG pipeline, retriever, agent, or downstream knowledge graph.

An **entity** is an object the publisher claims authority over: a concept, a person, a product, a place, an event, and so on. A **chunk** is a short evidence passage attached to an entity. A **relation** is a typed directional link between two entities.

---

## 2. Motivation

### 2.1 The problem

AI retrieval systems today operate at the page level — fetching HTML and extracting passages without structured awareness of entities, publisher identity, or concept relationships. Two consequences follow:

- **Attribution is brittle.** Once a passage is extracted into a vector database, the connection to its publisher is often lost or weakened.
- **Reasoning is reconstructive.** To answer a question about how two concepts relate, a model has to find prose that mentions both. The publisher's own logic — captured implicitly across many pages — is not directly readable.

### 2.2 The goal

A well-formed EntityMap enables consumers to retrieve entity-specific evidence, recognise publisher identity without inferring it from URL structure, and traverse explicit entity relationships rather than reconstructing them from prose.

### 2.3 Scope and limitations

EntityMap v1.0 claims to improve retrieval-time attribution, reduce disambiguation failures, and support single-hop typed reasoning. It does **not** claim to fix model prior errors, guarantee citation in AI-generated outputs, or support chain-level reasoning declarations — multi-hop reasoning chains are planned for v1.1.

### 2.4 Design rationale

EntityMap uses application-specific JSON rather than strict JSON-LD for implementation simplicity. The HTML companion file exposes equivalent schema.org JSON-LD per entity for broader interoperability.

---

## 3. Conformance floor

A conforming EntityMap v1.0 file requires exactly three things: a valid root object, at least one entity object, and at least one chunk per entity. Everything else in this specification is optional enrichment.

> **Minimum valid `entitymap.json`**
>
> ```json
> {
>   "version": "1.0",
>   "schema": "https://entitymap.org/spec/v1.0",
>   "publisher": {
>     "name": "Acme Corp",
>     "url": "https://acme.com"
>   },
>   "generated": "2026-04-07T00:00:00Z",
>   "entities": [
>     {
>       "entityId": "e_001",
>       "@type": "Concept",
>       "name": "Companion Planting",
>       "description": "The practice of growing different plants in proximity for mutual benefit.",
>       "hasChunks": [
>         {
>           "chunkId": "c_001",
>           "text": "Companion planting pairs plants that benefit each other.",
>           "sourceUrl": "https://acme.com/companion-planting",
>           "pageTitle": "Companion Planting",
>           "publisher": "Acme Corp"
>         }
>       ]
>     }
>   ]
> }
> ```

The mandatory floor (MUST requirements):

- The root MUST contain `version`, `schema`, `publisher`, `generated`, and `entities`.
- `publisher` MUST contain `name` and `url`.
- `entities` MUST have at least one entity.
- Each entity MUST contain `entityId`, `@type`, `name`, `description`, and `hasChunks`.
- Each chunk MUST contain `chunkId`, `text`, `sourceUrl`, `pageTitle`, and `publisher`.
- The chunk's `publisher` field MUST exactly match `publisher.name` in the root — including case and spacing. **This is the attribution mechanism.**
- Tier 3 predicates (`IMPROVES`, `DEGRADES`, `LEADS_TO`, `SUITED_FOR`, `TARGETS`, `ACHIEVES`) MUST carry a `confidence` field.

All other fields described in this specification are SHOULD or MAY. They improve quality, reasoning depth, and consumer compatibility without affecting basic conformance.

---

## 4. File conventions

### 4.1 Location

| File | URL pattern | Purpose |
| --- | --- | --- |
| `entitymap.json` | `https://example.com/entitymap.json` | Machine-readable primary file |
| `entitymap.html` | `https://example.com/entitymap.html` | Crawler and human-readable view |

Both files MUST be served from the root of the domain without authentication.

### 4.2 Discovery

**In `robots.txt`:**

```
# EntityMap
EntityMap: https://example.com/entitymap.json
```

**In the HTML `<head>` of every page:**

```html
<link rel="entitymap" type="application/json"
      href="https://example.com/entitymap.json" />
```

**Sitewide footer — most reliable discovery mechanism:**

```html
<footer>
  <a href="https://example.com/entitymap.html">EntityMap</a>
</footer>
```

**In `sitemap.xml`:** publishers SHOULD list `entitymap.html` with `priority: 0.9` and `changefreq: weekly`.

### 4.3 Indexability

`entitymap.html` MUST NOT carry a `noindex` directive.

### 4.4 Large sites — sharding

For sites with more than 200 entities, the EntityMap SHOULD be sharded. The root `entitymap.json` acts as a manifest. Each shard file MUST carry a `shardOf` field pointing back to the root manifest URI.

```
/entitymap.json              ← manifest
/entitymap/shards/part-001.json
/entitymap/shards/part-002.json
```

Sharding is a transport concern — split by size, not by entity type. Consumers load all shards; the split carries no semantic meaning.

---

## 5. JSON structure

### 5.1 Root object

```json
{
  "version": "1.0",
  "schema": "https://entitymap.org/spec/v1.0",
  "publisher": { ... },
  "generated": "2026-04-07T00:00:00Z",
  "profile": "core",
  "verificationStatus": "self-declared",
  "previousVersion": "https://...",
  "changeLog": [ ... ],
  "shards": [ ... ],
  "vocabulary": { ... },
  "entities": [ ... ]
}
```

| Field | Conformance | Description |
| --- | --- | --- |
| `version` | **MUST** | Must be `"1.0"`. |
| `schema` | **MUST** | Must be `"https://entitymap.org/spec/v1.0"`. |
| `publisher` | **MUST** | Publisher identity object. See [§5.2](#52-publisher-object). |
| `generated` | **MUST** | ISO 8601 timestamp. MUST be updated on every rebuild. |
| `entities` | **MUST** | Array of entity objects. Minimum 1. |
| `profile` | MAY | Extension profile. Default: `"core"`. See [§8](#8-extension-profiles). |
| `verificationStatus` | MAY | `"self-declared"` / `"generator-draft"` / `"third-party-verified"`. Default: `"self-declared"`. |
| `previousVersion` | MAY | URI of prior `entitymap.json`. Enables consumer diffing. |
| `changeLog` | MAY | Array of change entries. Types: `added` / `deprecated` / `modified` / `merged`. |
| `shards` | MAY | Index of shard files. |
| `vocabulary` | MAY | Custom predicate declarations. See [§7.5](#75-declaring-custom-predicates). |

### 5.2 Publisher object

| Field | Conformance | Description |
| --- | --- | --- |
| `name` | **MUST** | Canonical brand name. MUST NOT be a domain, product name, or generic descriptor. MUST match `publisher` on all chunks exactly. |
| `url` | **MUST** | Canonical URL of the publisher. |
| `sameAs` | MAY | Wikidata or schema.org URI anchoring the publisher to the open knowledge graph. |

### 5.3 Entity object

```json
{
  "entityId": "e_001",
  "@type": "ProprietaryTerm",
  "name": "AI Share of Voice",
  "description": "A metric measuring...",
  "tier": "knowledge",
  "alternateName": "AI SOV",
  "canonicalLabel": "share of voice",
  "sameAs": "https://www.wikidata.org/wiki/Q...",
  "maturityStatus": "established",
  "audienceType": "technical",
  "status": "active",
  "replacedBy": null,
  "relations": [ ... ],
  "hasChunks": [ ... ]
}
```

| Field | Conformance | Description |
| --- | --- | --- |
| `entityId` | **MUST** | Stable unique identifier. Never reuse a retired ID. |
| `@type` | **MUST** | v1.0 core type. See [§6](#6-entity-type-system). |
| `name` | **MUST** | Publisher-specific label. |
| `description` | **MUST** | 1–3 sentence definition as this publisher uses the concept. |
| `hasChunks` | **MUST** | 1–5 evidence chunks. |
| `tier` | MAY | `knowledge` / `actor` / `temporal` / `content`. Derivable from `@type` — omit if not needed. |
| `alternateName` | MAY | Abbreviation or surface form variant. |
| `canonicalLabel` | MAY | General concept label where publisher uses a proprietary variant. See [§5.6](#56-attribution-requirements). |
| `sameAs` | **SHOULD** for `Concept` | Wikidata URI. Strongly recommended for `Concept`; optional for others. |
| `maturityStatus` | MAY | `proposed` / `established` / `deprecated`. |
| `audienceType` | MAY | `technical` / `executive` / `general` / `regulatory`. |
| `status` | MAY | `active` / `deprecated` / `merged`. Default: `active`. |
| `replacedBy` | **MUST** if deprecated/merged | `entityId` of replacement entity. |
| `relations` | MAY | Typed relationships. See [§5.4](#54-relation-object). |

### 5.4 Relation object

```json
{
  "predicate": "IMPROVES",
  "targetId": "e_004",
  "targetName": "Retrieval Precision",
  "confidence": "declared",
  "context": {
    "condition": "when chunks are publisher-attributed",
    "temporal": "2024-onwards",
    "jurisdiction": null,
    "reviewedBy": "Fred Laurent",
    "reviewDate": "2026-04-01"
  },
  "targetUri": "...",
  "targetShard": "...",
  "targetDescription": "..."
}
```

| Field | Conformance | Description |
| --- | --- | --- |
| `predicate` | **MUST** | From standard vocabulary ([§7](#7-standard-predicate-vocabulary)) or declared custom vocabulary ([§7.5](#75-declaring-custom-predicates)). |
| `targetName` | **MUST** | Human-readable target name. Required in all cases — survives aggregation. |
| `confidence` | **MUST** for Tier 3 | `declared` / `inferred`. Required on Tier 3 predicates. Optional but allowed on Tier 1/2. |
| `targetId` | SHOULD | `entityId` of internal target. Required for internal relations. |
| `context` | MAY | Qualification object. Fields: `condition`, `temporal`, `jurisdiction`, `reviewedBy`, `reviewDate`. |
| `targetUri` | MAY | URI for external entities (Wikidata, schema.org). |
| `targetShard` | MAY | Path to shard file containing target entity. |
| `targetDescription` | MAY | One-sentence summary of target. SHOULD be present when target is unresolvable and `targetUri` is absent. |

> ⚠ For Tier 3 predicates, `confidence` is required — the validator errors if absent. A `confidence: "inferred"` relation without a `context` object produces a validator warning: consuming systems discount heavily without qualification context.

### 5.5 Chunk object

```json
{
  "chunkId": "c_001",
  "text": "...",
  "sourceUrl": "https://acme.com/page",
  "pageTitle": "Page Title",
  "publisher": "Acme Corp",
  "retrieved": "2026-04-07T09:00:00Z",
  "relevanceScore": 0.92,
  "contentType": "definition",
  "audienceType": "technical"
}
```

| Field | Conformance | Description |
| --- | --- | --- |
| `chunkId` | **MUST** | Unique identifier within this EntityMap. |
| `text` | **MUST** | Evidence passage. 1–5 sentences. Max 600 characters. SHOULD be extractive. |
| `sourceUrl` | **MUST** | Canonical URL of source page. MUST be publicly accessible. |
| `pageTitle` | **MUST** | Title of source page at time of retrieval. |
| `publisher` | **MUST** | MUST exactly match `publisher.name` in root. Primary brand attribution mechanism. |
| `retrieved` | SHOULD | ISO 8601 timestamp when source was fetched. |
| `relevanceScore` | MAY | Float 0.0–1.0. Publisher-assigned relevance to its entity. |
| `contentType` | MAY | `definition` / `evidence` / `example` / `statistic` / `procedure`. |
| `audienceType` | MAY | `technical` / `executive` / `general` / `regulatory`. |

### 5.6 Attribution requirements

> 📜 *This section is normative.*

**Publisher identity ([§5.2](#52-publisher-object)).** `publisher.name` MUST be a canonical brand name — not a domain, product name, or generic descriptor. It is the name that will appear in AI-generated attribution.

**Chunk-level attribution.** The `publisher` field on every chunk MUST exactly match `publisher.name`. Chunks are extracted and stored independently in vector databases — the publisher field is the mechanism by which attribution survives that extraction. Case differences, abbreviations, and trailing whitespace all constitute a mismatch.

**Freshness.** `generated` MUST be updated on every rebuild. A timestamp older than 30 days signals potential staleness to consumers.

**Canonical labelling (optional).** Where a publisher uses a proprietary term for a concept with a wider general label, the `canonicalLabel` field carries the general term while `name` carries the publisher-specific term. This aids cross-publisher disambiguation without losing the publisher's terminology.

---

## 6. Entity type system

EntityMap v1.0 defines 16 core types in four tiers. The tier reflects the epistemic role of the entity. Publishers MUST use a v1.0 core type or a namespaced custom type (e.g. `"acme:MetricComponent"`).

> ⚠ v0.x types `DefinedTerm`, `Product`, `ScholarlyArticle`, `CreativeWork`, and `Place` are not valid in v1.0 unchanged. See [Appendix C](#appendix-c-changelog) for migration guidance.

### 6.1 Tier 1 — Knowledge

| Type | Use for |
| --- | --- |
| **Concept** | General domain term. Common knowledge. Add `sameAs`. Consuming systems blend with general priors. |
| **ProprietaryTerm** | Publisher-coined concept. Definition here is authoritative. No `sameAs` expected. |
| **Methodology** | Named process, framework, or approach. |
| **Metric** | Measurable quantity with defined calculation. Source of `MEASURES` relations. |
| **Taxonomy** | Classification system the publisher maintains. Use `COVERS` for sub-categories. |

### 6.2 Tier 2 — Actor

| Type | Use for |
| --- | --- |
| **Person** | Named individual. Use `AFFILIATED_WITH` for their organisation. |
| **Organization** | Company, institution, or body. |
| **SoftwareProduct** | Software application, SaaS tool, API, or developer platform. |
| **PhysicalProduct** | Tangible goods. |
| **Service** | Professional or subscription offering. Not software. |
| **Platform** | Multi-sided or ecosystem-enabling product. |
| **Place** | Geographic location, region, or venue the publisher has content authority over. Use `sameAs` to anchor to Wikidata or Geonames. |

### 6.3 Tier 3 — Temporal

| Type | Use for |
| --- | --- |
| **Event** | Named event or occurrence with a defined time. |
| **Standard** | Specification or protocol with a version and governance body. |
| **Regulation** | Formal legal or regulatory instrument. Target of `REGULATED_BY`. |

### 6.4 Tier 4 — Content

| Type | Use for |
| --- | --- |
| **Guide** | Substantial instructional resource the publisher maintains and updates. |

### 6.5 Type decision rules

- **`Concept` vs `ProprietaryTerm`:** does this concept exist independently of the publisher? → `Concept` with `sameAs`. Did the publisher coin it or materially define it? → `ProprietaryTerm`.
- **`SoftwareProduct` vs `Platform` vs `Service`:** primarily software? → `SoftwareProduct`. Ecosystem or developer layer is central? → `Platform`. Primarily human-delivered? → `Service`.
- **`Standard` vs `Regulation`:** formally enacted into law? → `Regulation`. Voluntary specification with governance body? → `Standard`.

Non-standard types MUST be prefixed with a declared namespace: `"acme:MetricComponent"`.

---

## 7. Standard predicate vocabulary

All predicates are uppercase. The vocabulary has three tiers by semantic hardness. Tier determines the `confidence` field requirement and consuming system trust behaviour.

### 7.1 Tier 1 — Hard predicates (11)

Unambiguous, machine-trustable. No `confidence` field required. Strict source/target type constraints on some. Inverses are implicit — never declare both directions of `PART_OF`/`INCLUDES`.

```
INSTANCE_OF       PART_OF         INCLUDES
DEPENDS_ON        REQUIRES        MEASURES *
PRODUCED_BY       REGULATED_BY    AUTHORED_BY
AFFILIATED_WITH * COVERS **
```

\* type-constrained source ([§10](#10-validation)). \** `COVERS`: source must be `Concept`, `ProprietaryTerm`, or `Taxonomy`.

### 7.2 Tier 2 — Structural predicates (6)

Clear semantics, directional discipline required. `confidence` field optional. `RELATES_TO` is the predicate of last resort — use only when no other predicate fits.

```
RELATES_TO †      PRECEDES        ENABLES
PREVENTS          CONFLICTS_WITH  DESCRIBED_BY
```

† `RELATES_TO`: validator warns above 20% of all relations.

### 7.3 Tier 3 — Interpretive predicates (6)

Carry editorial judgment. `confidence` field is **required** — validator error if absent. Consuming systems apply lower reasoning weight when `confidence: "inferred"`.

```
IMPROVES          DEGRADES        LEADS_TO
SUITED_FOR        TARGETS         ACHIEVES
```

### 7.4 Key predicate decision rules

- **`PART_OF` vs `DEPENDS_ON`:** definitional constituent → `PART_OF`. Separate concept needing the other to function → `DEPENDS_ON`.
- **`INCLUDES` vs `COVERS`:** object is a component of subject → `INCLUDES`. Subject is a hub and object is a sub-topic the publisher covers → `COVERS`.
- **`ENABLES` vs `IMPROVES`:** structural enablement, unambiguous → `ENABLES` (Tier 2). Causal effect requiring editorial judgment → `IMPROVES` (Tier 3, `confidence` required).
- **`TARGETS` vs `SUITED_FOR`:** designed for the object → `TARGETS`. Happens to fit well, not necessarily designed for it → `SUITED_FOR`.

### 7.5 Declaring custom predicates

```json
"vocabulary": {
  "predicates": ["POLLINATES", "ZONES_AS"],
  "namespace": "https://acme.com/entitymap/vocab/v1"
}
```

Custom predicates MUST be uppercase, MUST NOT conflict with standard names, and MUST be documented at the declared namespace URI.

---

## 8. Extension profiles

Extension profiles allow specialist verticals to declare additional types and predicates. Declare a profile in the root `profile` field.

> 🔮 Healthcare, finance, and education profiles are *reserved* in v1.0 and will be formally specified in v1.1. Publishers MAY declare them — the validator warns but does not error. Full profile enforcement is deferred to v1.1.

| Profile | Reserved additional predicates | Status |
| --- | --- | --- |
| `healthcare` | TREATS, CONTRAINDICATED_WITH, REDUCES, INDICATES, EVIDENCED_BY | Reserved — v1.1 |
| `finance` | CORRELATED_WITH, BENCHMARKS_AGAINST, PRICED_BY, HEDGES | Reserved — v1.1 |
| `education` | TEACHES, PREREQUISITE_FOR, ASSESSES | Reserved — v1.1 |

Profile specs are published at `https://entitymap.org/profiles/{name}`. Community profiles can be proposed via [GitHub](../../issues).

---

## 9. The HTML companion file

`entitymap.html` is generated from `entitymap.json` and MUST NOT be maintained independently.

A conforming `entitymap.html` MUST:

- Reference `entitymap.json` via `<link rel="alternate" type="application/json">`
- Embed per-entity JSON-LD in `<script type="application/ld+json">` blocks
- Render relations as internal hyperlinks where targets exist in the same file
- Include a `data-publisher` attribute on every chunk blockquote
- Render the publisher name as **visible plain text** in every chunk's `<cite>` element — pattern: *[page title] — published by [publisher name]*
- Not carry a `noindex` directive

The plain-text attribution requirement exists because many LLM pipelines strip HTML tags before ingestion, discarding all metadata. Publisher attribution that exists only in structured attributes is invisible to those systems. The visible cite text is the fallback that ensures attribution survives plain-text ingestion.

```html
<blockquote data-publisher="Acme Corp">
  "Chunk text here."
  <cite>
    <a href="https://acme.com/page">Page title</a> — published by Acme Corp
  </cite>
</blockquote>
```

---

## 10. Validation

A conforming `entitymap.json` MUST satisfy all MUST requirements in [§3](#3-conformance-floor) and [§5](#5-json-structure). The primary validator rules that produce errors — not warnings — are:

- Missing any required field at root, entity, or chunk level.
- Using a v0.x legacy type.
- Chunk `publisher` not exactly matching `publisher.name`.
- Tier 3 predicate without `confidence` field.
- Entity with `status: "deprecated"` or `"merged"` without `replacedBy`.
- `MEASURES` used on non-`Metric` source entity.
- `AFFILIATED_WITH` used on non-`Person` source entity.
- `COVERS` used on non-hub source type.
- Both directions of `PART_OF`/`INCLUDES` declared between the same entity pair.
- `IMPROVES` and `DEGRADES`, or `ENABLES` and `PREVENTS`, declared between the same entity pair.
- Chunk text exceeding 600 characters.
- More than 5 chunks per entity.

A validator is available at [entitymap.org/validate](https://entitymap.org/validate). The validator also produces advisory warnings for recommended improvements beyond the mandatory floor.

---

## 11. Consumer conformance levels

### Level 1 — Chunk consumer

- Uses `hasChunks` as pre-structured retrieval units in place of raw page chunking.
- Preserves the `publisher` field through storage and aggregation.
- Stores `sourceUrl` and `publisher` as metadata alongside chunk embeddings.

### Level 2 — Entity consumer

- All Level 1 requirements.
- Uses `name`, `alternateName`, `canonicalLabel` for disambiguation across surface forms.
- Treats `ProprietaryTerm` descriptions as authoritative — does not blend with general knowledge.
- Uses `sameAs` for cross-source entity deduplication.
- Prioritises `definition` and `evidence` `contentType` chunks for reasoning tasks.

### Level 3 — Graph consumer

- All Level 2 requirements.
- Traverses relation graph for reasoning chain construction.
- Applies lower reasoning weight to Tier 3 predicates with `confidence: "inferred"`.
- Uses `context` object on relations — respects `condition`, `temporal`, `jurisdiction`.
- Treats `generator-draft` `verificationStatus` as `confidence: "inferred"` throughout.

---

## 12. Versioning and evolution

Minor versions (1.x) MAY add optional fields without breaking conformance of existing files. Major versions (x.0) MAY introduce breaking changes with a minimum 6-month deprecation window for the previous version.

> 🔮 The following are explicitly deferred to v1.1: `reasoningChains` array for multi-hop chain declarations · full extension profile specifications for healthcare, finance, and education · third-party verification registry infrastructure.

---

## 13. Privacy and security

EntityMap files are public by definition. Publishers MUST NOT include personal data beyond named public figures, authentication tokens or API keys, or content not already publicly accessible on the described website.

---

## 14. Relationship to existing standards

- **sitemap.xml** — sitemaps describe pages; EntityMap describes knowledge. Both SHOULD be present.
- **schema.org** — EntityMap uses schema.org-compatible `@type` values. The HTML companion embeds valid JSON-LD per entity.
- **robots.txt** — EntityMap discovery MAY be declared via an `EntityMap:` directive. Proposed convention, not yet a recognised standard.
- **JSON-LD** — application-specific JSON for simplicity. JSON-LD representations in the HTML companion for interoperability.
- **Wikidata** — `sameAs` fields SHOULD use Wikidata URIs to anchor entities to the open knowledge graph.
- **RSS / Atom** — EntityMap is to AI agents as RSS is to feed readers: a structured, subscribable content layer at a predictable URL.

---

## 15. Reference implementation

The reference generator is available at [waikay.io/entitymap](https://waikay.io/entitymap). It consists of an extraction pipeline using NLP entity analysis and LLM-assisted chunk selection, a generator producing conforming files, and a client-side viewer.

> ⚠ LLM-assisted generators produce *draft* EntityMaps. Files produced without human review MUST be published with `verificationStatus: "generator-draft"`. The `confidence: "declared"` designation and the `ProprietaryTerm` type require explicit human review before publication as self-declared.

The reference validator runs at [entitymap.org/validate](https://entitymap.org/validate).

Third-party implementations are welcomed — share yours in [GitHub Discussions](../../../discussions).

---

## 16. Consumer attribution guidance *(non-normative)*

> 📘 *This section is non-normative.* It provides guidance to AI agents, RAG pipelines, and LLM-based applications on handling publisher attribution.

When storing EntityMap chunks, preserve `publisher`, `sourceUrl`, and `contentType` as metadata alongside embeddings. When generating responses using EntityMap content, refer to the publisher by name — "According to Acme Gardens…" — and link to `sourceUrl` where the interface supports it.

EntityMap declared fields represent publisher assertions, not verified facts. Consuming systems should cross-validate declared relations against external knowledge where available, weight relations by confidence tier, and apply elevated trust to entities with `sameAs` grounding. The `verificationStatus` field provides a machine-readable signal for tiered trust application.

Where building a knowledge graph from multiple EntityMap files, the `sameAs` field provides a stable identifier for deduplication. Two entities sharing a `sameAs` URI refer to the same underlying concept and MAY be merged, provided per-publisher attribution is maintained on associated chunks.

---

## Appendix A — Minimal valid example

See [`examples/acme-gardens.json`](../../examples/acme-gardens.json) for the full version with relations and multiple entities. The bare conformance-floor example below is the minimum:

```json
{
  "version": "1.0",
  "schema": "https://entitymap.org/spec/v1.0",
  "publisher": {
    "name": "Acme Gardens",
    "url": "https://acmegardens.com"
  },
  "generated": "2026-04-07T00:00:00Z",
  "entities": [
    {
      "entityId": "e_001",
      "@type": "Concept",
      "name": "Companion Planting",
      "description": "The practice of growing different plants in proximity for mutual benefit, including pest control, pollination support, and improved yield.",
      "sameAs": "https://www.wikidata.org/wiki/Q905413",
      "relations": [
        { "predicate": "IMPROVES", "targetId": "e_002", "targetName": "Crop Yield", "confidence": "declared" },
        { "predicate": "PREVENTS", "targetId": "e_003", "targetName": "Pest Damage" }
      ],
      "hasChunks": [
        {
          "chunkId": "c_001",
          "text": "Companion planting pairs plants that benefit each other — growing basil near tomatoes repels aphids and improves fruit flavour.",
          "sourceUrl": "https://acmegardens.com/companion-planting-guide",
          "pageTitle": "The Complete Companion Planting Guide",
          "publisher": "Acme Gardens",
          "retrieved": "2026-04-07T09:14:00Z",
          "relevanceScore": 0.95,
          "contentType": "evidence"
        }
      ]
    }
  ]
}
```

---

## Appendix B — Complete predicate reference

```
TIER 1 — HARD (11) — no confidence required
  INSTANCE_OF      PART_OF          INCLUDES
  DEPENDS_ON       REQUIRES         MEASURES *
  PRODUCED_BY      REGULATED_BY     AUTHORED_BY
  AFFILIATED_WITH *  COVERS **

  *  type-constrained source
  ** COVERS: source must be Concept, ProprietaryTerm, or Taxonomy

TIER 2 — STRUCTURAL (6) — confidence optional
  RELATES_TO †     PRECEDES         ENABLES
  PREVENTS         CONFLICTS_WITH   DESCRIBED_BY

  † RELATES_TO: last resort — validator warns above 20% of all relations

TIER 3 — INTERPRETIVE (6) — confidence REQUIRED
  IMPROVES         DEGRADES         LEADS_TO
  SUITED_FOR       TARGETS          ACHIEVES

RESERVED — HEALTHCARE PROFILE (v1.1)
  TREATS  CONTRAINDICATED_WITH  REDUCES  INDICATES  EVIDENCED_BY

RESERVED — FINANCE PROFILE (v1.1)
  CORRELATED_WITH  BENCHMARKS_AGAINST  PRICED_BY  HEDGES

RESERVED — EDUCATION PROFILE (v1.1)
  TEACHES  PREREQUISITE_FOR  ASSESSES

TOTAL CORE: 23 predicates
```

For full per-predicate definitions, examples, and decision rules, see [predicates.md](predicates.md).

---

## Appendix C — Changelog

See [`CHANGELOG.md`](../../CHANGELOG.md) for the full version history.
