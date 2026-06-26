# Generate your first EntityMap with an LLM

This is a structured prompt for generating a conforming `entitymap.json` for your site, with help from Claude, ChatGPT, or another capable LLM. It does not replace the [reference generator at waikay.io/entitymap](https://waikay.io/entitymap), which uses a full NLP pipeline and is the right tool for ongoing production. This is for **writing your first EntityMap by hand** so you learn the model and ship something real.

The prompt below is **self-contained**: everything the LLM needs to produce a conforming file — types, predicates, structural rules, conformance bar — is embedded in the prompt itself. You don't need to attach the spec separately, and you don't fill in any placeholders.

## How to use it

1. **Open a fresh conversation** with Claude (Sonnet 4.5 or higher recommended), or another capable LLM with web access.
2. **Paste the prompt below** as your first message. That's it — no spec attachment, no placeholders.
3. **Answer the three questions** as the model asks them.
4. **The model will produce a draft, audit every relation, then re-emit a corrected `entitymap.json`.** Save the *second* JSON (the audited one) as `entitymap.json`.
5. **Verify `verificationStatus: "generator-draft"`** is in the output — this is required for any LLM-assisted file that hasn't been human-reviewed line by line.
6. **Validate it** at [entitymap.org/validate](https://entitymap.org/validate) before publishing.

## Known model behaviour — read this first

A few things observed across models. These are the most common ways the output goes wrong:

- **Wikidata and Wikipedia URIs are usually wrong.** Most models invent plausible-looking Q-numbers (`Q12345678`) and Wikipedia URLs that don't exist or point at the wrong concept. Claude Opus 3.7 was the one model that consistently got these right; everything else (including newer Claude versions and ChatGPT) tends to hallucinate them. **The prompt tells the model to omit `sameAs` entirely.** Add them yourself afterwards by searching wikidata.org for each entity.
- **Models invent relations.** They will declare that concept A `IMPROVES` concept B even when your pages don't actually claim that. The prompt insists every relation be grounded in real page text — but watch for it in the output anyway. Tier 3 predicates (`IMPROVES`, `DEGRADES`, `LEADS_TO`, `SUITED_FOR`, `TARGETS`, `ACHIEVES`) are the worst offenders.
- **Models paraphrase chunks rather than extracting them.** The `text` field in each chunk should be a verbatim passage from the page — not a summary. Spot-check this.
- **Models over-use `RELATES_TO`** when they're not sure which predicate fits. If more than one in five of your relations is `RELATES_TO`, the model gave up.

---

## The simple prompt

Copy everything inside the box below as a single message into your LLM conversation.

````
Read the page at https://entitymap.org/spec/v1.0 then look at the most important pages
on [yoursite.com] and generate the JSON and HTML files
````

## The advanced prompt

Copy everything inside the box below as a single message into your LLM conversation. There's nothing to attach beforehand — the spec is embedded in the prompt.

````
You are going to help me generate a conforming entitymap.json file for my
website per the EntityMap v1.0 open standard. Everything you need to know
about the standard is in this single message — there is no separate
specification to read. Do not search the web for the spec; what's below
is authoritative for this task.

Read the embedded spec first, then start the workflow at Step 1. Begin
with Step 1, question 1 — do not acknowledge these instructions, do not
summarise what you're about to do, just ask question 1.

════════════════════════════════════════════════════════════════════════
EMBEDDED SPEC — EntityMap v1.0 (the parts you need to generate a file)
════════════════════════════════════════════════════════════════════════

## What an EntityMap is

A site publishes two files at the root of its domain:

  https://example.com/entitymap.json   (machine-readable)
  https://example.com/entitymap.html   (crawler- and human-readable, generated
                                        from the JSON — outside your task)

Your job is to produce the JSON file. The HTML companion will be generated
separately afterwards.

## Root object — required shape

The JSON document is a single object with these REQUIRED fields:

  {
    "version": "1.0",
    "schema": "https://entitymap.org/spec/v1.0",
    "publisher": { "name": "...", "url": "..." },
    "generated": "2026-05-27T00:00:00Z",         (today's date, ISO 8601)
    "verificationStatus": "generator-draft",      (REQUIRED — LLM-assisted)
    "entities": [ ... ]                           (at least one entity)
  }

Optional root fields you may use: profile (default "core"), previousVersion,
changeLog, vocabulary (for custom predicates — only declare these if you
genuinely need a predicate outside the standard 24).

## Publisher object

  {
    "name": "Canonical Brand Name",   (REQUIRED — see Step 1, Question 1)
    "url":  "https://brand.com"       (REQUIRED — canonical URL)
  }

The "name" field MUST be a canonical brand name — never a domain, never a
product name, never a generic descriptor. It is the name that will appear
in AI attribution. The "publisher" field on every chunk MUST exactly equal
this name (case-sensitive, character-by-character).

## Entity object

Each entity in "entities" has this shape:

  {
    "entityId":    "e_001",                       (REQUIRED, stable, unique)
    "@type":       "Concept",                     (REQUIRED, from the 16
                                                   core types — list below)
    "name":        "Companion Planting",          (REQUIRED, my site's label)
    "description": "1–3 sentence definition...",  (REQUIRED, my-site usage)
    "hasChunks":   [ ... ],                       (REQUIRED, 1 to 5 chunks)
    "relations":   [ ... ],                       (optional, encouraged)

    "alternateName": "...",                       (optional abbreviation)
    "canonicalLabel": "...",                      (optional — see below)
    "maturityStatus": "proposed|established|deprecated",
    "audienceType":  "technical|executive|general|regulatory",
    "status":        "active|deprecated|merged",  (default "active")
    "replacedBy":    "e_002"                      (REQUIRED if status is
                                                   deprecated or merged)
  }

DO NOT include "sameAs". See the sameAs rule further down.

"canonicalLabel" — use this on a ProprietaryTerm entity when the publisher
has coined a term for something that has a more general label. E.g.
ProprietaryTerm name = "AI Share of Voice", canonicalLabel = "share of voice".

## The 16 core entity types

Tier 1 — Knowledge:
  Concept             general domain term, common knowledge
  ProprietaryTerm     publisher-coined; definition is authoritative
  Methodology         named process, framework, or approach
  Metric              measurable quantity with defined calculation
  Taxonomy            classification system the publisher maintains

Tier 2 — Actor:
  Person              named individual
  Organization        company, institution, body
  SoftwareProduct     software application, SaaS, API, dev platform
  PhysicalProduct     tangible goods
  Service             professional / subscription offering, not software
  Platform            multi-sided or ecosystem product
  Place               geographic location

Tier 3 — Temporal:
  Event               named event or occurrence with a defined time
  Standard            specification or protocol with governance body
  Regulation          formal legal or regulatory instrument

Tier 4 — Content:
  Guide               substantial instructional resource

Type decision rules:
  - Concept vs ProprietaryTerm: does this exist independently of the
    brand? → Concept. Did the brand coin it or materially define it?
    → ProprietaryTerm.
  - SoftwareProduct vs Platform vs Service: primarily software? →
    SoftwareProduct. Ecosystem/dev layer central? → Platform. Primarily
    human-delivered? → Service.
  - Standard vs Regulation: formally enacted into law? → Regulation.
    Voluntary specification with a governance body? → Standard.

NEVER use the v0.x legacy types: DefinedTerm, Product, ScholarlyArticle,
CreativeWork. These will be rejected by the validator.

Custom types — if a publisher genuinely needs a type outside the 16, it
must be namespaced: e.g. "acme:MetricComponent". Use this sparingly.

## Chunk object

Each entity has 1 to 5 chunks:

  {
    "chunkId":     "c_001",                       (REQUIRED, unique)
    "text":        "Verbatim passage from page",  (REQUIRED, ≤ 600 chars,
                                                   1–5 sentences,
                                                   EXTRACTIVE)
    "sourceUrl":   "https://brand.com/page",      (REQUIRED, publicly
                                                   accessible)
    "pageTitle":   "Page Title",                  (REQUIRED)
    "publisher":   "Canonical Brand Name",        (REQUIRED, MUST equal
                                                   publisher.name exactly)
    "retrieved":   "2026-05-27T08:00:00Z",        (SHOULD, ISO 8601)
    "relevanceScore": 0.92,                       (optional, 0.0–1.0)
    "contentType": "definition|evidence|example|statistic|procedure",
    "audienceType": "technical|executive|general|regulatory"
  }

The "text" field MUST be an extractive passage — copied verbatim from the
source page. Do NOT paraphrase. Do NOT summarise. Do NOT invent. If you
cannot find a suitable passage in the supplied content, omit the chunk
rather than fabricate it.

The "publisher" field on every chunk MUST exactly match publisher.name
in the root — including case, spacing, and punctuation. This is the
attribution mechanism — chunks get extracted and stored independently in
vector databases, and the publisher field is how attribution survives.

## Relation object

Relations declare typed directional links between entities:

  {
    "predicate":  "IMPROVES",                     (REQUIRED, from the 24
                                                   standard predicates)
    "targetId":   "e_004",                        (SHOULD, entityId of
                                                   internal target)
    "targetName": "Retrieval Precision",          (REQUIRED, human label)
    "confidence": "declared",                     (REQUIRED for Tier 3;
                                                   optional for 1 & 2)
    "context":    { ... },                        (optional qualification)

    "targetUri":         "https://...",           (for external targets)
    "targetShard":       "shards/part-002.json",  (for cross-shard refs)
    "targetDescription": "One-sentence summary"   (when target unresolvable)
  }

Context object (optional, qualifies a Tier 3 relation):

  {
    "condition":   "when chunks carry publisher attribution",
    "temporal":    "2024-onwards",
    "jurisdiction": "EU",
    "reviewedBy":  "Editor Name",
    "reviewDate":  "2026-04-01"
  }

## The 24 standard predicates

TIER 1 — HARD (no confidence required)
  INSTANCE_OF       subject is a specific example of object
  PART_OF           subject is a definitional constituent of object
  INCLUDES          subject contains object (inverse of PART_OF)
  DEPENDS_ON        subject requires object to function
  REQUIRES          subject formally mandates object
  MEASURES          subject quantifies object
                    [source entity MUST be @type: "Metric"]
  PRODUCED_BY       subject is created/output by object
  REGULATED_BY      subject is governed by object
  AUTHORED_BY       subject is written by object (Person or Organization)
  AFFILIATED_WITH   subject is associated with object
                    [source entity MUST be @type: "Person"]
  COVERS            subject is a hub topic, object is a sub-topic
                    [source MUST be Concept, ProprietaryTerm, or Taxonomy]

TIER 2 — STRUCTURAL (confidence optional)
  RELATES_TO        general association — last resort only, use sparingly
  PRECEDES          subject comes before object in a sequence
  ENABLES           subject makes object possible (structural)
  PREVENTS          subject blocks object (inverse of ENABLES)
  CONFLICTS_WITH    subject and object are incompatible
  DESCRIBED_BY      subject is documented by object
  OFFERS            subject (Organization) makes object commercially available
                    [source MUST be Organization;
                     target MUST be SoftwareProduct, Service, Platform,
                     or PhysicalProduct]

TIER 3 — INTERPRETIVE (confidence REQUIRED: "declared" or "inferred")
  IMPROVES          subject makes object better
  DEGRADES          subject makes object worse (inverse of IMPROVES)
  LEADS_TO          subject causes/contributes to object over time
  SUITED_FOR        subject happens to fit object's context well
  TARGETS           subject is designed for object (stronger than SUITED_FOR)
  ACHIEVES          subject successfully produces object as outcome

Key predicate decision rules:
  - PART_OF vs DEPENDS_ON: definitional constituent → PART_OF. Separable
    concept that needs the other → DEPENDS_ON.
  - INCLUDES vs COVERS: component-of relationship → INCLUDES. Hub-topic /
    sub-topic relationship → COVERS.
  - ENABLES (Tier 2) vs IMPROVES (Tier 3): structural enablement,
    unambiguous → ENABLES. Causal effect needing editorial judgement →
    IMPROVES (and confidence is required).
  - TARGETS vs SUITED_FOR: designed for it → TARGETS. Happens to fit →
    SUITED_FOR.
  - OFFERS vs PRODUCED_BY: both describe the Organization↔product
    relationship from opposite sides. Pick ONE direction; never declare
    both between the same pair. The publisher's own EntityMap typically
    uses OFFERS ("Acme OFFERS Widget"). Third-party EntityMaps describing
    someone else's products may prefer PRODUCED_BY ("Widget PRODUCED_BY Acme").

## FORBIDDEN inverted predicate forms

EntityMap predicates are directional. The following inverted/passive forms
are NOT valid and the validator will reject them. If you need the inverse
meaning, swap subject and target and use the standard predicate:

  ENABLED_BY              → flip: "Tool ENABLES Outcome"
  MEASURED_BY             → flip: "Metric MEASURES Concept"
  PRODUCES, PRODUCED_FOR  → flip: "Product PRODUCED_BY Organization"
  DEPENDED_ON_BY          → flip: "Dependent DEPENDS_ON Dependency"
  REQUIRED_BY             → flip: "Consumer REQUIRES Dependency"
  IMPROVED_BY             → flip: "Cause IMPROVES Effect"
  DEGRADED_BY             → flip: "Cause DEGRADES Effect"
  COVERS_BY, COVERED_BY   → flip: "Hub COVERS SubTopic"
  DESCRIBES               → use DESCRIBED_BY in the reverse direction
  AFFILIATED_WITH_BY      → flip: "Person AFFILIATED_WITH Organization"
  OFFERED_BY              → flip: "Organization OFFERS Product"

If you find yourself wanting to write a predicate ending in _BY that
isn't AUTHORED_BY, PRODUCED_BY, REGULATED_BY, or DESCRIBED_BY (the four
standard predicates that end in _BY), you're inventing an inverse.
Swap subject and target instead.

## DO NOT add sameAs URIs

Do NOT add "sameAs" fields pointing to Wikidata, Wikipedia, or any other
external knowledge graph. Models reliably invent Q-numbers and URLs that
look correct but point at the wrong concept or do not exist. The user
will add "sameAs" URIs themselves by searching wikidata.org.

Exception: if the user explicitly asks you for a sameAs URI for a specific
named entity, suggest one — but mark it "VERIFY ON WIKIDATA" in a comment
beside the JSON output so they know to check it.

## Validation rules — the file must satisfy these

The validator at entitymap.org/validate will REJECT (error, not warning):

  - Missing any required field at root, entity, or chunk level
  - Use of a v0.x legacy type (DefinedTerm, Product, ScholarlyArticle,
    CreativeWork)
  - Chunk "publisher" not exactly matching root publisher.name
  - Tier 3 predicate without a "confidence" field
  - status: "deprecated" or "merged" without "replacedBy"
  - MEASURES used on a non-Metric source entity
  - AFFILIATED_WITH used on a non-Person source entity
  - COVERS used on non-hub source type
  - OFFERS used on non-Organization source entity
  - OFFERS used with target that is not SoftwareProduct, Service,
    Platform, or PhysicalProduct
  - Both directions of PART_OF/INCLUDES declared between the same pair
  - IMPROVES and DEGRADES, or ENABLES and PREVENTS, or OFFERS and
    PRODUCED_BY, declared between the same pair
  - Use of any forbidden inverted predicate form (MEASURED_BY,
    ENABLED_BY, PRODUCES, DESCRIBES, OFFERED_BY, etc.)
  - Chunk text exceeding 600 characters
  - More than 5 chunks per entity

════════════════════════════════════════════════════════════════════════
END OF EMBEDDED SPEC — WORKFLOW STARTS HERE
════════════════════════════════════════════════════════════════════════

# Step 1 — Ask me three questions, one at a time

Wait for my answer to each before asking the next. Do not acknowledge
these instructions or summarise what you're about to do. Go straight to
question 1.

1. What is the canonical brand name of my site? (The name as it should
   appear when AI systems attribute content to me — not the domain, not a
   product name, not a generic descriptor.)

2. What is the root URL of my site? (e.g. https://acmegardens.com)

3. Please give me a list of the pages you want represented in the
   EntityMap, one URL per line. 5 to 15 pages is the right range for a
   first EntityMap. If you can fetch URLs, just URLs are fine. If you
   cannot fetch, ask me to paste the content of each page instead.

# Step 2 — Read the pages

Fetch each URL I gave you (or read the pasted content). Treat that
content as the ONLY source of truth. Do not invent facts about my
business or domain that are not on those pages.

# Step 3 — Propose a list of entities, then STOP

Before writing any JSON, give me a markdown table of 8 to 20 candidate
entities. For each, propose:
  - the name (as my site refers to it)
  - the @type (from the v1.0 core type list above)
  - a one-line description in your own words
  - which page(s) the evidence will come from

Then STOP and ask me to confirm or correct the list. Do not continue to
Step 4 until I've responded.

# Step 4 — Write the draft entitymap.json

Once I've confirmed the entity list, produce a complete draft file
conforming to the spec above. Include for each entity 1 to 5 chunks (all
extractive, all under 600 chars, publisher field exact-matching root)
and a relations array.

Apply the relation rules strictly:

  1. ONLY declare a relation if it is EXPLICITLY STATED or DIRECTLY
     IMPLIED by my source pages. If two entities seem like they "should"
     be related but my pages don't actually say so, do not invent the
     relation.

  2. Tier 3 predicates (IMPROVES / DEGRADES / LEADS_TO / SUITED_FOR /
     TARGETS / ACHIEVES) carry editorial weight. Use these only when my
     page text actively claims the effect. When in doubt, prefer a Tier 2
     structural predicate (ENABLES, PRECEDES) or no relation at all.

  3. Use RELATES_TO sparingly. If you find yourself reaching for it more
     than once or twice, you probably need a different predicate or no
     relation. RELATES_TO above 20% of all relations triggers a validator
     warning.

  4. Never declare BOTH directions of an inverse pair (PART_OF/INCLUDES,
     IMPROVES/DEGRADES, ENABLES/PREVENTS, OFFERS/PRODUCED_BY) between
     the same two entities.

  5. For Tier 3 predicates, set "confidence" to:
       - "declared" if my page text explicitly makes the claim
       - "inferred" if it's strongly implied but not stated outright

  6. Type-constrained predicates — the validator will reject these
     otherwise:
       - MEASURES: source entity MUST be @type: "Metric"
       - AFFILIATED_WITH: source entity MUST be @type: "Person"
       - COVERS: source entity MUST be @type: "Concept",
         "ProprietaryTerm", or "Taxonomy"
       - OFFERS: source entity MUST be @type: "Organization";
         target entity MUST be @type: "SoftwareProduct", "Service",
         "Platform", or "PhysicalProduct"

  7. Add a "context" object on Tier 3 relations when the source page
     qualifies the claim (e.g. condition, temporal, jurisdiction).

# Step 5 — Audit every relation (this is required, not optional)

Relations are where LLM-generated EntityMaps go wrong most often. Before
producing the review checklist, you MUST audit every relation you wrote
in Step 4. Do not skip this step. Do not summarise it. Do the work.

For EACH relation in the file, in order, output a single line in this
exact format:

  [entityId] PREDICATE → [targetName]   VERDICT — short reason

Where VERDICT is one of:

  KEEP        — the source page explicitly states or directly implies
                this relation. The predicate is the correct one.
  WEAKEN      — the relation is real but you had to soften it (e.g.
                Tier 3 confidence changed from "declared" to "inferred",
                or a Tier 3 predicate was replaced with a Tier 2
                structural one).
  REPLACE     — the relation is real but the predicate was wrong; you
                swapped it for a better one. Show both: old → new.
  REMOVE      — the relation was not actually supported by my source
                pages, or it violated a structural rule (inverse pair,
                type-constrained predicate misused, etc.). Deleted.

Apply these specific tests to each relation:

  Test A — Grounding. Quote (or paraphrase in one phrase) the sentence
           in my source page that supports this relation. If you cannot
           point to one, the verdict is REMOVE.

  Test B — Predicate fit. Is this actually the closest predicate from
           the 24, or did you reach for it because it sounded plausible?
           Specifically:
             - Did you use IMPROVES where ENABLES would be more honest?
             - Did you use RELATES_TO because nothing else came to mind?
             - Did you use TARGETS where SUITED_FOR fits better (or
               vice versa)?
             - Did you invent an inverted form (MEASURED_BY, ENABLED_BY,
               PRODUCES, DESCRIBES, etc.)? If yes, this is automatic REMOVE
               — replace by flipping subject/target and using the standard
               predicate.
           If a better predicate exists, the verdict is REPLACE.

  Test C — Tier 3 confidence. For IMPROVES, DEGRADES, LEADS_TO,
           SUITED_FOR, TARGETS, ACHIEVES: does the page text actively
           CLAIM this effect ("declared"), or only suggest it
           ("inferred")? If you used "declared" but the page only
           implies, the verdict is WEAKEN (downgrade to "inferred").

  Test D — Structural rules. Did this relation violate:
             - Inverse-pair forbidden combos
               (PART_OF + INCLUDES, IMPROVES + DEGRADES,
                ENABLES + PREVENTS, OFFERS + PRODUCED_BY on the same pair)
             - Type-constrained predicate rules
               (MEASURES source must be Metric; AFFILIATED_WITH source
                must be Person; COVERS source must be Concept,
                ProprietaryTerm, or Taxonomy; OFFERS source must be
                Organization with target SoftwareProduct, Service,
                Platform, or PhysicalProduct)
             - Inverted/passive predicate forms (MEASURED_BY, ENABLED_BY,
               PRODUCES, DESCRIBES, OFFERED_BY, etc.) — see the FORBIDDEN
               list in the embedded spec
           If yes, the verdict is REMOVE or REPLACE.

  Test E — RELATES_TO budget. Count how many RELATES_TO relations are
           in the file. If they exceed 20% of total relations, REPLACE
           or REMOVE the weakest ones until you are under the threshold.

After the audit, RE-EMIT the corrected entitymap.json reflecting all
the WEAKEN / REPLACE / REMOVE verdicts. The first JSON you wrote in
Step 4 is a draft; the audited version is the file the user will publish.

# Step 6 — Review checklist

After the audited JSON, give me a short markdown checklist flagging
anything that still needs my human review:
  - Any entity where you weren't sure about the @type
  - Any Tier 3 relation where the "declared" confidence claim is still
    only weakly supported (even after WEAKEN was considered)
  - Any chunk where you had to paraphrase rather than extract
  - Any place where you suspect you were too generous OR too cautious
  - Anything else you'd like me to confirm before publishing

Then remind me to:
  - Keep verificationStatus as "generator-draft" until I've reviewed
    everything line by line
  - Validate at https://entitymap.org/validate
  - Generate the entitymap.html companion (see spec §9)
  - Add sameAs URIs by hand from wikidata.org where appropriate
  - Only promote to "self-declared" after full human review

Begin now with Step 1, question 1.
````

---

## After the model finishes — what you still need to spot-check

The model has already audited its own relations in Step 5 (or it should have — if it skipped or shortened that step, push back and ask it to redo it line by line). Your remaining job is to second-guess the audit, especially on the relations the model decided to KEEP.

- **Read the audit table first.** The KEEP / WEAKEN / REPLACE / REMOVE lines tell you where the model was uncertain. The REMOVE and REPLACE lines are a good prompt to think about whether *similar* mistakes were missed in the KEEP rows.
- **Open every chunk.** Read the `text` field, then open `sourceUrl` and confirm the passage is on the page, verbatim. If it's paraphrased or invented, replace it with a real extractive passage. The model's audit doesn't cover chunks.
- **Re-check every Tier 3 `confidence: "declared"`.** "Declared" means the page says it explicitly. If the page only implies it, downgrade to `"inferred"`. If the page doesn't say it at all, delete the relation entirely.
- **Spot-check predicate type constraints.** `MEASURES` only on `Metric` entities. `AFFILIATED_WITH` only on `Person` entities. `COVERS` only on `Concept`, `ProprietaryTerm`, or `Taxonomy` entities. `OFFERS` only on `Organization` entities with `SoftwareProduct`/`Service`/`Platform`/`PhysicalProduct` targets. The validator catches these but it's faster to fix yourself.
- **Add `sameAs` URIs by hand.** For each `Concept` entity with an obvious Wikidata equivalent, search wikidata.org and paste the actual Q-number URL. Don't trust any the model suggested, even with a "VERIFY" tag.

## What to do before publishing

1. **Validate.** Run the file through [entitymap.org/validate](https://entitymap.org/validate). Fix every error. Read every warning.
2. **Visualize.** Run the file through [entitymap.org/viewer](https://entitymap.org/viewer). 
3. **Generate the HTML companion.** `entitymap.html` is required alongside `entitymap.json` per [§9 of the spec](../spec/v1.0/index.md#9-the-html-companion-file). Use the static skeleton template, the reference generator at waikay.io, or your own tooling.
4. **Publish at the domain root.** Both files at `https://yourdomain.com/entitymap.json` and `https://yourdomain.com/entitymap.html`. No authentication, no `noindex`.
5. **Declare discovery.** `robots.txt`, `<link rel="entitymap">` in every page's `<head>`, sitewide footer link to `entitymap.html`.
6. **Only when you've human-reviewed the whole file**, change `verificationStatus` from `"generator-draft"` to `"self-declared"`.

## A worked example

If you've never written one before, look at [`examples/acme-gardens.json`](../examples/acme-gardens.json) first. It's three linked entities — Companion Planting, Crop Yield, Pest Damage — showing how `relations`, `confidence`, and `hasChunks` fit together. Your first file should land in roughly that shape, maybe somewhat larger.

## Feedback on this prompt

If you use this and the output is wrong in interesting ways, please [open a Discussion in 💬 General](https://github.com/entitymap/entitymap/discussions/categories/general). The prompt will keep improving as more people run it against real sites.
