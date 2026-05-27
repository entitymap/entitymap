# Changelog

All notable changes to the EntityMap specification are listed here. This file mirrors Appendix C of the spec.

The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). EntityMap follows [SemVer](https://semver.org/) for the spec: `MAJOR.MINOR` (no patch). Minor versions may add optional fields without breaking conformance; major versions may introduce breaking changes with a minimum six-month deprecation window.

---

## [1.0] — 2026-04-07

Stable release.

### Changed

- Restructured spec for readability.
- Consumer conformance levels and extension profiles moved into appendices.

### Added

- 16 core types across 4 tiers (replaces v0.x's flat `DefinedTerm`/`Product`/etc.).
- Three-tier predicate vocabulary: 11 hard + 6 structural + 6 interpretive = **23 standard predicates**.
- `verificationStatus` field — `self-declared` / `generator-draft` / `third-party-verified`.
- `certification` field (registry launches Q3 2026).
- `canonicalLabel` field for `ProprietaryTerm` disambiguation.
- Reserved extension profiles for healthcare, finance, and education (fully specified in v1.1).

### Removed (breaking from v0.x)

- v0.x types: `DefinedTerm`, `Product`, `ScholarlyArticle`, `CreativeWork`, `Place` (the v0.x `Place` is replaced by a Tier 2 `Place` with different semantics).
- v0.x predicates outside the 23-predicate core vocabulary (still usable via the root `vocabulary` block).

### Deferred to v1.1

- `reasoningChains` array for multi-hop chain declarations.
- Full extension profile specifications.
- Live third-party verification registry.

---

## [0.3] — 2026-03-28

### Added

- Cross-shard resolution rules (§4.4.1).
- Normative publisher attribution requirements.
- Plain-text attribution requirement on the HTML companion's `<cite>` element.
- Non-normative consumer attribution guidance.

---

## [0.2] — 2026-03-27

### Added

- RFC 2119 normative language throughout.
- `retrieved` field on the chunk object.
- Predicate vocabulary tiered (precursor to the v1.0 three-tier structure).

### Changed

- Relation model updated.

---

## [0.1] — 2026-03-27

Initial draft.
