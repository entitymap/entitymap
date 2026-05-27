# EntityMap

> An open standard for publishing a structured, entity-first index of website knowledge for AI systems, retrieval pipelines, and language-model-based applications.

[![Spec: v1.0](https://img.shields.io/badge/spec-v1.0-2e3192)](spec/v1.0/index.md)
[![Status: Stable](https://img.shields.io/badge/status-stable-1a5c1a)](spec/v1.0/index.md)
[![License: CC BY 4.0](https://img.shields.io/badge/license-CC%20BY%204.0-1a5c1a)](LICENSE)

Where `sitemap.xml` tells crawlers **what pages exist**, `entitymap.json` tells AI systems **what a site knows** — which entities it covers, how they relate, and where the evidence is.

## What's in this repository

| Path | What it is |
| --- | --- |
| [`spec/v1.0/index.md`](spec/v1.0/index.md) | The normative v1.0 specification. |
| [`spec/v1.0/predicates.md`](spec/v1.0/predicates.md) | Per-predicate reference — all 23 standard predicates with definitions and examples. |
| [`examples/acme-gardens.json`](examples/acme-gardens.json) | A complete, realistic `entitymap.json` you can use as a starting template. |
| [`prompts/generate-entitymap.md`](prompts/generate-entitymap.md) | A detailed prompt for generating your first `entitymap.json` by hand with help from Claude, ChatGPT, or another LLM. |
| [`CHANGELOG.md`](CHANGELOG.md) | Spec version history. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to participate — Discussions for ideas, Issues for spec bugs. |

## The 30-second version

Publish two files at the root of your domain:

```
https://yourdomain.com/entitymap.json    # machine-readable
https://yourdomain.com/entitymap.html    # crawler- and human-readable
```

Declare them in three places — `robots.txt`, the page `<head>`, and a sitewide footer link:

```
# robots.txt
EntityMap: https://yourdomain.com/entitymap.json
```

```html
<!-- <head> on every page -->
<link rel="entitymap" type="application/json"
      href="https://yourdomain.com/entitymap.json" />

<!-- sitewide footer — the most reliable discovery mechanism -->
<footer><a href="https://yourdomain.com/entitymap.html">EntityMap</a></footer>
```

The minimum valid file is three things: a root object, at least one entity, and at least one chunk per entity. The full conformance floor is in [§3 of the spec](spec/v1.0/index.md#3-conformance-floor).

## Quick start

1. Read the spec — [`spec/v1.0/index.md`](spec/v1.0/index.md) is the authoritative source. It's deliberately short.
2. Look at the example — [`examples/acme-gardens.json`](examples/acme-gardens.json) shows three linked entities with relations and chunks.
3. Generate your first file — [`prompts/generate-entitymap.md`](prompts/generate-entitymap.md) is a structured prompt you can paste into Claude or ChatGPT alongside your site content.
4. Publish at the root of your domain and declare discovery (see above).

## Status

| Field | Value |
| --- | --- |
| Current version | **1.0** — stable |
| Released | 2026-04-07 |
| Editors | Fred Laurent · Dixon Jones |
| License | CC BY 4.0 |
| Site | [entitymap.org](https://entitymap.org) |
| Reference generator | [waikay.io/entitymap](https://waikay.io/entitymap) — NLP + LLM-assisted pipeline |

## Participate

EntityMap follows the pattern Schema.org uses: an open vocabulary that improves through adoption and discussion.

- **Ideas, questions, use-case discussions** → [GitHub Discussions](../../discussions)
- **Spec bugs, ambiguities, typos** → [GitHub Issues](../../issues)
- **Proposing a new predicate or extension profile** → start in Discussions; if there's consensus to formalise, it moves to a PR.

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the (short) details.

## Tooling

A JSON Schema, a CLI validator, a static viewer, and an HTML companion generator exist but ship separately so this repo stays focused on the standard. They'll be folded in once the early-adopter feedback has stabilised the rules around warnings and certification.

For the moment, the reference implementation at [waikay.io/entitymap](https://waikay.io/entitymap) generates conforming files end-to-end. The validator at [entitymap.org/validate](https://entitymap.org/validate) checks them.

## License

CC BY 4.0 — see [`LICENSE`](LICENSE). You may use, redistribute, and adapt the specification with attribution.

> *Suggested attribution:* "EntityMap v1.0 specification by Fred Laurent and Dixon Jones, licensed under CC BY 4.0. https://entitymap.org/spec/v1.0"
