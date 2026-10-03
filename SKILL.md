---
name: design-recipe
description: Design, review, or refactor data-processing functions using HtDP's design recipes. Use for unclear data models, missing cases, recursive algorithms, accumulator invariants, or decomposition problems, and when explicitly requested. Routine formatting and mechanical edits do not need this workflow.
---

# Systematic program design

Use HtDP's design recipes to turn a problem statement into an inspectable design: data definitions, a signature and purpose, independently derived examples, a justified implementation structure, and verification. Scale the written artifacts to the change; preserve the reasoning rather than imposing a worksheet on every function.

## Working recipe

1. **Understand the task and existing data.** Read the relevant types, schemas, callers, and tests. Identify valid inputs, interpretation, constraints, and expected failures. Reuse existing representations unless the requested change requires revising them.
2. **State the behavior.** Establish the function's signature and a concise purpose statement. For effectful operations include observable effects. Distinguish confirmed requirements from assumptions; ask when an unresolved choice would materially change the result.
3. **Derive examples before implementing the changed behavior.** Work out expected results from the specification, independently of the implementation. Include relevant variants, boundaries, and interactions. Reuse tests that already express these examples.
4. **Choose a design strategy.** For structural processing, derive an inventory from the data: cases, fields, and recursive substructures. Implement it with idiomatic matches, loops, or existing abstractions. Use composition for distinct stages, generative recursion for generated subproblems, and accumulators for necessary context. Templates are reasoning tools, not mandatory final code shapes.
5. **Implement and verify.** Fill the gaps using the purpose and examples. Keep a wish list of missing helpers when useful, resolving it as the design develops. Check recursive calls' contracts, progress, and invariants. Run relevant checks and review the resulting diff for missed cases and unnecessary complexity.

For a small repair, the existing types, a regression example, a focused edit, and targeted verification may supply all the artifacts. For a new algorithm or representation, make the additional decisions explicit in code, tests, or a short design note. Keep comments that explain interpretation, assumptions, or invariants; avoid duplicating obvious syntax. Do not widen a review into implementation or a small fix into unrelated restructuring.

## Read the relevant lesson

Read [the full function recipe](docs/02-the-design-recipe.md) for a new design or when a step is unclear. Select additional references according to the problem; do not load the entire collection by default.

| Situation | Reference |
|---|---|
| Understand the discipline and its limits | [Core philosophy](docs/01-core-philosophy.md) |
| Unclear representation, variants, units, or valid inputs | [Data definitions](docs/03-data-definitions.md) |
| Case analysis, lists, trees, or multiple complex inputs | [Templates](docs/04-templates.md) |
| Multiple stages, helper contracts, or missing collaborators | [Composition](docs/05-composing-functions.md) |
| Duplicated behavior or a reusable traversal | [Abstraction](docs/06-abstraction.md) |
| Search, partitioning, convergence, or generated subproblems | [Generative recursion](docs/07-generative-recursion.md) |
| Running totals, consumed prefixes, visited nodes, or context | [Accumulators](docs/08-accumulators.md) |
| Events, mutation, external input, or effectful boundaries | [State and boundaries](docs/09-state-and-boundaries.md) |
| Final review or a requested code review | [Quality checklist](docs/10-llm-quality-checklist.md) |

## Standards of evidence

- Treat examples and passing tests as evidence, not proof. Check the data definition and assumptions as well as executed paths.
- Structural recursion terminates when calls descend through finite, well-founded data. Graph cycles, unchanged arguments, and effectful helpers require additional reasoning.
- For generative recursion, justify the generated problems, combination, and termination. A decreasing measure must be well-founded; a positive real value merely getting smaller is insufficient. For intentionally continuing services, describe lifecycle and cancellation instead of inventing a finite bound.
- Derive accumulator initialization, updates, and result extraction from a stated invariant. Recheck it when changing traversal order or state.
- Separate specified absence or failure from invalid inputs. A documented `None`, `false`, or sentinel can be a valid itemization; preserve established API contracts.
- Use existing abstractions when their semantics fit. Two working instances are a useful method for discovering a new abstraction, not a prohibition on generic APIs or abstraction from a template.

When reporting a review, identify concrete defects or risks with locations and consequences. When reporting a change, state the resulting behavior, checks actually run, and material limits. This skill does not authorize edits, execution, or external actions beyond the user's task.

The references distinguish HtDP concepts from adaptations for modern repositories. Source map: [docs/README.md](docs/README.md).
