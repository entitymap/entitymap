# Generate your first EntityMap with an LLM

This is a structured prompt for generating a conforming `entitymap.json` for your site, with help from Claude, ChatGPT, or another capable LLM. It does not replace the [reference generator at waikay.io/entitymap](https://waikay.io/entitymap), which uses a full NLP pipeline and is the right tool for ongoing production. This is for **writing your first EntityMap by hand** so you learn the model and ship something real.

The prompt below is **conversational**: the model will ask you three questions, then build the file. You don't fill in any placeholders. Just paste it into a fresh chat alongside the spec.

## How to use it

1. **Open a fresh conversation** with Claude (Sonnet 4.5 or higher recommended), or another capable LLM with web access.
2. **Attach or paste the EntityMap v1.0 specification** — either upload [`spec/v1.0/index.md`](../spec/v1.0/index.md) as a file, or paste its contents into the chat. The model needs the spec to refer to.
3. **Paste the prompt below** as your first message.
4. **Answer the three questions** as the model asks them.
5. **The model will produce your `entitymap.json`.** Save it as `entitymap.json`.
6. **Verify `verificationStatus: "generator-draft"`** is in the output — this is required for any LLM-assisted file that hasn't been human-reviewed line by line.
7. **Validate it** at [entitymap.org/validate](https://entitymap.org/validate) before publishing.

## Known model behaviour — read this first

A few things observed across models. These are the most common ways the output goes wrong:

- **Wikidata and Wikipedia URIs are usually wrong.** Most models invent plausible-looking Q-numbers (`Q12345678`) and Wikipedia URLs that don't exist or point at the wrong concept. Claude Opus 3.7 was the one model that consistently got these right; everything else (including newer Claude versions and ChatGPT) tends to hallucinate them. **This prompt tells the model to omit `sameAs` entirely.** Add them yourself afterwards by searching wikidata.org for each entity.
- **Models invent relations.** They will declare that concept A `IMPROVES` concept B even when your pages don't actually claim that. The prompt insists every relation be grounded in real page text — but watch for it in the output anyway. Tier 3 predicates (`IMPROVES`, `DEGRADES`, `LEADS_TO`, `SUITED_FOR`, `TARGETS`, `ACHIEVES`) are the worst offenders.
- **Models paraphrase chunks rather than extracting them.** The `text` field in each chunk should be a verbatim passage from the page — not a summary. Spot-check this.
- **Models over-use `RELATES_TO`** when they're not sure which predicate fits. If more than one in five of your relations is `RELATES_TO`, the model gave up.

---

## The prompt

Copy everything inside the box below as a single message into your LLM conversation. Make sure the EntityMap v1.0 spec is already attached or pasted in the same conversation.

```
I want to generate a conforming entitymap.json file for my website, per the
EntityMap v1.0 specification (which is attached / pasted above).

Please follow this exact process:

# Step 1 — Ask me three questions, one at a time

Wait for my answer to each before asking the next.

1. What is the canonical brand name of my site? (The name as it should
   appear when AI systems attribute content to me — not the domain, not a
   product name, not a generic descriptor.)

2. What is the root URL of my site? (e.g. https://acmegardens.com)

3. Please give me a list of the pages you want represented in the
   EntityMap, one URL per line. 5 to 20 pages is the right range for a
   first EntityMap — pick the ones that best represent what your site
   knows. If you can fetch URLs, just URLs are fine. If you cannot fetch,
   ask me to paste the content of each page instead.

# Step 2 — Read the pages

Fetch each URL I gave you (or read the pasted content). Treat that content
as the ONLY source of truth. Do not invent facts about my business or
domain that are not on those pages.

# Step 3 — Propose a list of entities

Before writing any JSON, give me a markdown table of 8 to 20 candidate
entities. For each, propose:
  - the name (as my site refers to it)
  - the @type (from the v1.0 core type list — see below)
  - a one-line description in your own words
  - which page(s) the evidence will come from

Then STOP and ask me to confirm or correct the list before continuing.

# Step 4 — Write the entitymap.json

Once I've confirmed the entity list, produce the complete file.

ROOT OBJECT must include:
  - "version": "1.0"
  - "schema": "https://entitymap.org/spec/v1.0"
  - "publisher": { "name": "...", "url": "..." }
  - "generated": today's date in ISO 8601 (e.g. "2026-05-27T00:00:00Z")
  - "verificationStatus": "generator-draft"   ← REQUIRED — this is LLM-assisted
  - "entities": [...]

For EACH entity, include:
  - "entityId":    a stable string like "e_001", "e_002", ...
  - "@type":       one of the v1.0 core types ONLY (list below).
                   Never invent types. Never use v0.x types like
                   DefinedTerm, Product, ScholarlyArticle, CreativeWork.
  - "name":        the publisher-specific label
  - "description": 1–3 sentences as MY site uses this concept
  - "hasChunks":   1 to 5 evidence chunks per entity (see chunk rules below)
  - "relations":   optional but encouraged (see relation rules below)

For EACH chunk:
  - "chunkId":     "c_001", "c_002", ...
  - "text":        an EXTRACTIVE 1–5 sentence passage drawn verbatim from
                   the source page. Maximum 600 characters. Do NOT
                   paraphrase. Do NOT invent. If you cannot find a suitable
                   passage in the supplied content, omit the chunk rather
                   than fabricate one.
  - "sourceUrl":   the URL of the page the chunk is from
  - "pageTitle":   the title of that page
  - "publisher":   MUST exactly equal the brand name from Step 1
                   (case-sensitive, including spacing)
  - "contentType": one of definition / evidence / example / statistic / procedure

# IMPORTANT: do NOT add sameAs URIs

Do not add "sameAs" fields pointing to Wikidata, Wikipedia, or any other
external knowledge graph. Models reliably invent Q-numbers and URLs that
look correct but point at the wrong concept or do not exist. I will add
"sameAs" URIs myself afterwards by searching wikidata.org for each entity.

The only exception: if I explicitly ask you for a sameAs URI for a
specific named entity, you may suggest one — but mark it with
"VERIFY ON WIKIDATA" in a comment beside the JSON output so I know to
check it before publishing.

# Standard predicate vocabulary (23 predicates only)

TIER 1 — HARD (no confidence required)
  INSTANCE_OF       subject is a specific example of object
  PART_OF           subject is a definitional constituent of object
  INCLUDES          subject contains object as a component (inverse of PART_OF)
  DEPENDS_ON        subject requires object to function
  REQUIRES          subject formally mandates object
  MEASURES          subject quantifies object [source MUST be a Metric entity]
  PRODUCED_BY       subject is created/output by object
  REGULATED_BY      subject is governed by object
  AUTHORED_BY       subject is written by object (Person or Organization)
  AFFILIATED_WITH   subject is associated with object [source MUST be a Person]
  COVERS            subject is a hub topic, object is a sub-topic
                    [source MUST be Concept, ProprietaryTerm, or Taxonomy]

TIER 2 — STRUCTURAL (confidence optional)
  RELATES_TO        general association — use SPARINGLY, last resort only
  PRECEDES          subject comes before object in a sequence
  ENABLES           subject makes object possible (structural, not causal)
  PREVENTS          subject blocks object (inverse of ENABLES)
  CONFLICTS_WITH    subject and object are incompatible
  DESCRIBED_BY      subject is documented by object

TIER 3 — INTERPRETIVE (confidence REQUIRED — "declared" or "inferred")
  IMPROVES          subject makes object better along some dimension
  DEGRADES          subject makes object worse (inverse of IMPROVES)
  LEADS_TO          subject causes/contributes to object over time
  SUITED_FOR        subject happens to fit object's context well
  TARGETS           subject is designed for object (stronger than SUITED_FOR)
  ACHIEVES          subject successfully produces object as outcome

# CRITICAL — relation rules — read carefully

This is where LLM-generated EntityMaps go wrong most often. Follow strictly:

1. ONLY declare a relation if it is EXPLICITLY STATED or DIRECTLY IMPLIED
   by my source pages. If two entities seem like they "should" be related
   but my pages don't actually say so, do not invent the relation.

2. Tier 3 predicates (IMPROVES / DEGRADES / LEADS_TO / SUITED_FOR /
   TARGETS / ACHIEVES) carry editorial weight. Use these only when my
   page text actively claims the effect. When in doubt, prefer a Tier 2
   structural predicate (ENABLES, PRECEDES) or no relation at all.

3. Use RELATES_TO sparingly. If you find yourself reaching for it more
   than once or twice, you probably need a different predicate or no
   relation. RELATES_TO above 20% of all relations triggers a validator
   warning.

4. Never declare BOTH directions of an inverse pair between the same
   two entities. Specifically forbidden pairs:
     - PART_OF and INCLUDES
     - IMPROVES and DEGRADES
     - ENABLES and PREVENTS

5. For Tier 3 predicates, set "confidence" to:
     - "declared" if my page text explicitly makes this claim
     - "inferred" if it's strongly implied but not stated outright
   Never use confidence: "declared" for something my pages don't actually
   say.

6. Type-constrained predicates — the validator will reject these
   otherwise:
     - MEASURES: source entity MUST be @type: "Metric"
     - AFFILIATED_WITH: source entity MUST be @type: "Person"
     - COVERS: source entity MUST be @type: "Concept",
       "ProprietaryTerm", or "Taxonomy"

7. Add a "context" object on Tier 3 relations where my source page
   qualifies the claim — fields include condition, temporal, jurisdiction,
   reviewedBy, reviewDate. Example: a "TREATS" claim might be qualified
   with condition: "in adults over 18".

# Standard entity types

  Tier 1 (Knowledge):  Concept, ProprietaryTerm, Methodology, Metric, Taxonomy
  Tier 2 (Actor):      Person, Organization, SoftwareProduct, PhysicalProduct,
                       Service, Platform, Place
  Tier 3 (Temporal):   Event, Standard, Regulation
  Tier 4 (Content):    Guide

Key type decision rules:
  - Concept vs ProprietaryTerm: does this exist independently of my brand?
    → Concept. Did my brand coin it or materially define it? →
    ProprietaryTerm.
  - SoftwareProduct vs Platform vs Service: primarily software? →
    SoftwareProduct. Ecosystem/dev layer central? → Platform. Primarily
    human-delivered? → Service.
  - Standard vs Regulation: formally enacted into law? → Regulation.
    Voluntary specification with a governance body? → Standard.

# Step 5 — Review checklist

After you produce the JSON, give me a short markdown review checklist
flagging:
  - Any entity where you weren't sure about the @type
  - Any Tier 3 relation where the "declared" confidence claim is only
    weakly supported by my page text
  - Any chunk where you had to paraphrase rather than extract
  - Anything you'd like me to confirm before publishing

Then remind me to:
  - Keep verificationStatus as "generator-draft" until I've reviewed
    everything line by line
  - Validate at https://entitymap.org/validate
  - Generate the entitymap.html companion (see spec §9)
  - Add sameAs URIs by hand from wikidata.org where appropriate
  - Only promote to "self-declared" after full human review

Begin with Step 1, question 1.
```

---

## After the model finishes — what to spot-check

Things to verify in the output before you publish:

- **Open every chunk.** Read the `text` field, then open `sourceUrl` and confirm the passage is actually on the page, verbatim. If it's paraphrased or invented, replace it with a real extractive passage.
- **Open every relation.** For each one, ask: *does my page actually claim this?* If not, delete the relation. Most pages will produce only 3–6 relations per entity, so this is faster than it sounds.
- **Sanity-check every Tier 3 `confidence: "declared"`.** "Declared" means the page says it explicitly. If the page only implies it, change to `"inferred"`. If the page doesn't say it at all, delete the relation.
- **Look for `RELATES_TO` overuse.** Count the relations. If more than ~20% are `RELATES_TO`, the model gave up — go back and either pick a more specific predicate or drop the relation.
- **Verify predicate type constraints.** `MEASURES` only on `Metric` entities. `AFFILIATED_WITH` only on `Person` entities. `COVERS` only on `Concept`, `ProprietaryTerm`, or `Taxonomy` entities. The validator catches these but it's faster to fix yourself.
- **Add `sameAs` URIs by hand.** For each `Concept` entity that has an obvious Wikidata equivalent, search wikidata.org and paste the actual Q-number URL. Don't trust any the model offered, even with a "VERIFY" tag.

## What to do before publishing

1. **Validate.** Run the file through [entitymap.org/validate](https://entitymap.org/validate). Fix every error. Read every warning.
2. **Generate the HTML companion.** `entitymap.html` is required alongside `entitymap.json` per [§9 of the spec](../spec/v1.0/index.md#9-the-html-companion-file). Use the static skeleton template, the reference generator at waikay.io, or your own tooling.
3. **Publish at the domain root.** Both files at `https://yourdomain.com/entitymap.json` and `https://yourdomain.com/entitymap.html`. No authentication, no `noindex`.
4. **Declare discovery.** `robots.txt`, `<link rel="entitymap">` in every page's `<head>`, sitewide footer link to `entitymap.html`.
5. **Only when you've human-reviewed the whole file**, change `verificationStatus` from `"generator-draft"` to `"self-declared"`.

## A worked example

If you've never written one before, look at [`examples/acme-gardens.json`](../examples/acme-gardens.json) first. It's three linked entities — Companion Planting, Crop Yield, Pest Damage — showing how `relations`, `confidence`, and `hasChunks` fit together. Your first file should land in roughly that shape, maybe somewhat larger.

## Feedback on this prompt

If you use this and the output is wrong in interesting ways, please [open a Discussion](../../discussions). The prompt will keep improving as more people run it against real sites.
