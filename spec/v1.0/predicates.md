# Predicate reference

The 24 standard predicates of EntityMap v1.0, with definitions, usage examples, and tier classification. The summary table is in [§7 of the spec](index.md#7-standard-predicate-vocabulary); this file is the long-form companion.

All predicates are uppercase with underscores between words. Inverses are implicit — never declare both directions of a relationship pair (e.g. `PART_OF` and `INCLUDES`) between the same two entities. Inverted or passive forms of standard predicates (e.g. `MEASURED_BY`, `ENABLED_BY`, `DESCRIBES`) are **not valid** — see [Forbidden forms](#forbidden-forms).

## Quick index

- [Tier 1 — Hard (11)](#tier-1--hard) · machine-trustable, `confidence` not required
- [Tier 2 — Structural (7)](#tier-2--structural) · clear semantics, `confidence` optional
- [Tier 3 — Interpretive (6)](#tier-3--interpretive) · editorial judgment, `confidence` required
- [Decision rules](#key-decision-rules) · which predicate to pick when two could plausibly fit
- [Forbidden forms](#forbidden-forms) · inverted predicate names that are not valid
- [Declaring custom predicates](#declaring-custom-predicates) · for domain terms outside the core vocabulary

---

## Tier 1 — Hard

Unambiguous, machine-trustable relations. The `confidence` field is not required. Several Tier 1 predicates have type constraints on their source entity, listed under each.

### INSTANCE_OF

The subject entity is a specific example or instantiation of the object entity's general class or type.

> AI Share of Voice **INSTANCE_OF** Share of Voice

### PART_OF

The subject entity is a definitional constituent of the object entity. If the subject were removed, the object would be incomplete.

> Chunk Object **PART_OF** Entity Object

### INCLUDES

The subject entity contains or encompasses the object entity as a component or member. Inverse of `PART_OF` — never declare both directions between the same pair.

> Entity Object **INCLUDES** Chunk Object

### DEPENDS_ON

The subject entity requires the object entity to function but is not a definitional part of it.

> EntityMap Generator **DEPENDS_ON** Entity Extraction

### REQUIRES

The subject entity formally mandates the presence of the object entity. Stronger than `DEPENDS_ON`; usually appears in MUST-language contexts.

> v1.0 Conformance **REQUIRES** Publisher Identity Field

### MEASURES *

The subject entity quantifies, tracks, or evaluates the object entity.

> AI Share of Voice **MEASURES** Brand Presence in LLM Answers

**Source constraint:** the source entity MUST have `@type: "Metric"`. The validator errors otherwise.

### PRODUCED_BY

The subject entity is created or output by the object entity.

> Companion Planting Guide **PRODUCED_BY** Acme Gardens

### REGULATED_BY

The subject entity is governed by, or subject to, the object entity as a rule, law, or formal standard.

> Personal Data in EntityMap **REGULATED_BY** GDPR

### AUTHORED_BY

The subject entity was written, created, or primarily attributed to the object entity (a `Person` or `Organization`).

> EntityMap Specification **AUTHORED_BY** Fred Laurent

### AFFILIATED_WITH *

The subject entity (typically a `Person`) is associated with the object entity (typically an `Organization`) in a professional or institutional capacity.

> Dixon Jones **AFFILIATED_WITH** InLinks

**Source constraint:** the source entity MUST have `@type: "Person"`. The validator errors otherwise.

### COVERS **

The subject entity is a publisher-maintained hub or taxonomy under which the object entity sits as a sub-topic.

> Gardening Guide **COVERS** Companion Planting

**Source constraint:** the source entity MUST have `@type: "Concept"`, `"ProprietaryTerm"`, or `"Taxonomy"`. The validator errors otherwise.

---

## Tier 2 — Structural

Clear semantics, directional discipline required. `confidence` field is optional. `RELATES_TO` is the predicate of last resort — the validator emits a warning if it exceeds 20% of an EntityMap's relations.

### RELATES_TO

A general association between two entities when a more specific predicate does not apply. Use sparingly — prefer a precise predicate where one exists.

> Companion Planting **RELATES_TO** Soil Health

### PRECEDES

The subject entity comes before the object entity in a sequence, process, or timeline.

> Entity Extraction **PRECEDES** Chunk Selection

### ENABLES

The subject entity makes the object entity possible without being its sole cause. Structural enablement, not editorial causation.

> Structured Publisher Attribution **ENABLES** AI Share of Voice Measurement

### PREVENTS

The subject entity blocks or impedes the object entity. Inverse of `ENABLES` — never declare both between the same pair.

> Companion Planting **PREVENTS** Pest Damage

### CONFLICTS_WITH

The subject entity is incompatible or in tension with the object entity. Symmetric in meaning but only needs to be declared once.

> Fragmented Data Estate **CONFLICTS_WITH** Regulatory Compliance

### DESCRIBED_BY

The subject entity is documented or characterised by the object entity.

> EntityMap **DESCRIBED_BY** EntityMap Specification v1.0

### OFFERS **

The subject entity makes the object entity commercially available — sells it, distributes it, or makes it accessible as a product, service, or platform.

> Waikay **OFFERS** EntityMap Generator
> Acme Gardens **OFFERS** Companion Planting Consultancy

**Source constraint:** the source entity MUST have `@type: "Organization"`.
**Target constraint:** the target entity MUST have `@type: "SoftwareProduct"`, `"Service"`, `"Platform"`, or `"PhysicalProduct"`.

`OFFERS` is the commercial complement to `PRODUCED_BY`. The pair captures two facets of the same relationship from opposite sides — declare it in one direction only:

- *Organization-side view:* `Acme OFFERS Widget` (`OFFERS`)
- *Product-side view:* `Widget PRODUCED_BY Acme` (`PRODUCED_BY`)

The validator rejects an EntityMap that declares both `OFFERS` and `PRODUCED_BY` between the same pair of entities.

---

## Tier 3 — Interpretive

Carry editorial judgment about causal or evaluative effects. **`confidence` is required** — the validator errors if absent. Consuming systems apply lower reasoning weight when `confidence: "inferred"`. A `context` object qualifying when the relation holds is strongly recommended.

### IMPROVES

The subject entity makes the object entity better along some dimension.

> Structured Entity Attribution **IMPROVES** Retrieval Precision

```json
{
  "predicate": "IMPROVES",
  "targetId": "e_002",
  "targetName": "Retrieval Precision",
  "confidence": "declared",
  "context": {
    "condition": "when chunks carry the publisher field",
    "temporal": "2024-onwards"
  }
}
```

### DEGRADES

The subject entity makes the object entity worse along some dimension. Inverse of `IMPROVES` — never declare both between the same pair.

> Page-Level Chunking **DEGRADES** Publisher Attribution

### LEADS_TO

The subject entity causes or contributes to the object entity over time.

> Ghost Citations **LEADS_TO** Attribution Loss

### SUITED_FOR

The subject entity is particularly well-matched to the object entity's context or use case, though not necessarily designed for it.

> RAG Retrieval **SUITED_FOR** Entity-First Indexes

### TARGETS

The subject entity is specifically designed for or directed at the object entity. Stronger than `SUITED_FOR` — implies intent.

> EntityMap Discovery Hints **TARGETS** AI Crawlers

### ACHIEVES

The subject entity successfully accomplishes or produces the object entity as an outcome.

> Publisher Attribution Field **ACHIEVES** Brand Survival in Aggregation

---

## Key decision rules

When two predicates seem to fit, these rules pick the more accurate one.

### `PART_OF` vs `DEPENDS_ON`

Is the subject a definitional constituent — without which the object is incomplete? → `PART_OF`. Or is the subject a separate concept that the object needs to function, but which could in principle be substituted? → `DEPENDS_ON`.

### `INCLUDES` vs `COVERS`

Is the object a component of the subject? → `INCLUDES`. Is the subject a hub or taxonomy under which the object sits as a sub-topic? → `COVERS`.

### `ENABLES` vs `IMPROVES`

Is the enablement structural and unambiguous (the subject makes the object possible)? → `ENABLES` (Tier 2, no `confidence` needed). Is it a causal effect requiring editorial judgment (the subject makes the object better in some measurable way)? → `IMPROVES` (Tier 3, `confidence` required).

### `TARGETS` vs `SUITED_FOR`

Was the subject designed for the object? → `TARGETS`. Does the subject just happen to fit well? → `SUITED_FOR`.

### `OFFERS` vs `PRODUCED_BY`

Both describe the relationship between an organization and a product/service. Pick one direction; the validator rejects EntityMaps that declare both between the same entity pair.

- *Organization-side view:* `Acme OFFERS Widget` — declare on the Organization, treating the product/service as the target.
- *Product-side view:* `Widget PRODUCED_BY Acme` — declare on the product/service, treating the organization as the target.

A publisher's own EntityMap should typically use `OFFERS` (the publisher is the organization, and listing what it offers reads naturally). Third-party EntityMaps describing other organizations' products may prefer `PRODUCED_BY` (the product is the entity of interest, the organization is attribution).

---

## Forbidden forms

EntityMap predicates are directional. Inverted or passive forms of standard predicates are **not valid** — declare the relation in the standard direction instead. The validator rejects an EntityMap containing any of these forms.

| Invalid form | What to do instead |
| --- | --- |
| `MEASURED_BY` | Flip: `Metric MEASURES Concept` |
| `ENABLED_BY` | Flip: `Tool ENABLES Outcome` |
| `PRODUCES`, `PRODUCED_FOR` | Flip: `Product PRODUCED_BY Organization` |
| `DEPENDED_ON_BY`, `DEPENDS_ON_BY` | Flip: `Dependent DEPENDS_ON Dependency` |
| `REQUIRED_BY` | Flip: `Consumer REQUIRES Dependency` |
| `IMPROVED_BY`, `DEGRADED_BY` | Flip: `Cause IMPROVES Effect` or `Cause DEGRADES Effect` |
| `COVERS_BY`, `COVERED_BY` | Flip: `Hub COVERS SubTopic` |
| `DESCRIBES`, `DESCRIBED_BY_INVERSE` | Use the standard `DESCRIBED_BY` in the reverse direction |
| `AFFILIATED_WITH_BY` | Flip: `Person AFFILIATED_WITH Organization` |
| `OFFERED_BY` | Flip: `Organization OFFERS Product` |

The pattern is general, not exhaustive: if a predicate name ends in `_BY` and isn't in the [Tier 1 / Tier 2 / Tier 3 lists above](#quick-index), it's probably an invented inverse. Swap subject and target and use the standard form.

If a publisher genuinely needs a predicate outside the standard 24, declare it as a custom predicate (next section) — not as an inverted form of a standard one.

---

## Declaring custom predicates

If your domain genuinely needs a predicate outside the standard 24, you can declare custom predicates in the root `vocabulary` block:

```json
"vocabulary": {
  "predicates": ["POLLINATES", "ZONES_AS"],
  "namespace": "https://acme.com/entitymap/vocab/v1"
}
```

Custom predicates must be:

- **Uppercase** with underscores between words, same as standard predicates.
- **Not in conflict with standard names.** The validator errors if you try to redeclare a standard predicate.
- **Documented** at the declared namespace URI — so a consumer can resolve the predicate's meaning.

Custom predicates are visible to consumers but receive lower default trust than standard ones. If a predicate is broadly useful and not specific to one publisher, [propose it as a standard predicate in Discussions](../../../discussions/categories/predicates) instead.

---

## Reserved predicates (extension profiles, v1.1)

These are reserved names that publishers can declare via the `profile` field today. They will be formally specified in v1.1; using them in v1.0 is permitted but generates a validator warning.

**`healthcare`** — TREATS · CONTRAINDICATED_WITH · REDUCES · INDICATES · EVIDENCED_BY

**`finance`** — CORRELATED_WITH · BENCHMARKS_AGAINST · PRICED_BY · HEDGES

**`education`** — TEACHES · PREREQUISITE_FOR · ASSESSES

See [§8 of the spec](index.md#8-extension-profiles) for profile mechanics.
