# Contributing to EntityMap

EntityMap is an open standard. The value of this repo is in the breadth of people who poke at it, discuss it, and adopt it. Here's where each kind of contribution goes.

## Where to post what

**[GitHub Discussions](../../discussions) — for ideas and questions**

- "Has anyone thought about modelling X?"
- "Should there be a predicate for Y?"
- "Here's how I'm using EntityMap on my site — feedback?"
- "What's the difference between `INCLUDES` and `COVERS`?"
- General use-case conversations, design rationale questions, "show and tell"

**[GitHub Issues](../../issues) — for spec bugs**

- A typo, ambiguity, or contradiction in the spec
- The spec contradicts itself (cite both places)
- Something that's MUST but should be SHOULD, or vice versa
- Examples that don't actually conform to the spec
- Broken links

If you're not sure which to use, start in Discussions. We'll move things to Issues if they turn into concrete defects.

## Proposing a new predicate

Predicate proposals are a special case because the bar is intentionally high — predicate sprawl is a real cost. The flow:

1. **Open a Discussion** in the `predicates` category with:
   - The proposed name (uppercase, underscores between words)
   - Which tier you think it belongs in (1 hard, 2 structural, 3 interpretive)
   - A one-line definition
   - Two concrete usage examples from real publisher content
   - Why an existing predicate doesn't already cover this without distortion

2. If there's consensus that it belongs in the core vocabulary, the maintainers open an Issue and a PR to add it to a minor version (`1.x`).

3. If the predicate is useful but too specialised for core, it can be used right away via the `vocabulary` block in any publisher's own EntityMap — no spec change needed. See [§7.5](spec/v1.0/index.md#75-declaring-custom-predicates).

## Spec changes — what gets merged when

v1.0 is stable, so changes follow a careful path:

| Kind of change | Path |
| --- | --- |
| Typos, broken links, clarification of unambiguously correct prose | PR, merged after one editor review |
| New **optional** fields | Discussion → consensus → PR → ships in next minor version (`1.x`) after at least one prototype implementation |
| **Breaking** changes | Deferred to the next major version (`2.0`), with a minimum six-month deprecation window per [§12](spec/v1.0/index.md#12-versioning-and-evolution) |

## Sending a PR

- Branch from `main`
- One concern per PR
- Reference the Discussion or Issue your PR resolves
- For spec changes, update `CHANGELOG.md` in the same PR

## Licensing your contribution

By contributing, you agree your contribution is released under [CC BY 4.0](LICENSE), the same license as the rest of the repository.

## Code of conduct

Everyone here follows the [Code of Conduct](CODE_OF_CONDUCT.md). In short: be kind, assume good faith, disagree on the substance, not the person.
