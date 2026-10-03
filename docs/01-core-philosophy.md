# Core philosophy: make the design inspectable

HtDP teaches systematic program design through intermediate products: data definitions, signatures, purpose statements, examples, templates, definitions, and tests. These products expose misunderstandings while the design is still easy to change. See the [Preface, especially Figure 1](https://htdp.org/2024-11-6/Book/part_preface.html).

## Information and data

A problem concerns information in the world; a program consumes representations of that information. State how the representation is interpreted. An integer might mean a price in cents, a duration in milliseconds, or a database identifier. Its machine type alone does not communicate that meaning.

Read existing representations before inventing new ones. Record constraints such as units, permitted ranges, ordering, uniqueness, and ownership where downstream code can find them.

## Structure follows data, within the chosen strategy

For structural processing, the input definition supplies an inventory: alternatives imply case analysis, structures supply fields, and recursive definitions supply recursive subproblems. This makes omissions visible before implementation details distract the designer.

The inventory guides reasoning; it does not prescribe every line of final code. A function can ignore irrelevant fields, group cases with identical behavior, or use a library traversal. Composition can organize stages around intermediate data. Generative algorithms require additional insight beyond the input's structure.

## Examples clarify the specification

Predict concrete outcomes before implementing new behavior. Include examples that distinguish plausible interpretations: inclusive versus exclusive boundaries, stable versus unstable ordering, and absence versus invalid input.

Expected results must come from the specification or an independent oracle. Observing an output and recording it as the expectation can preserve the very bug the test was meant to detect.

## Refinement is part of design

Start with a useful, explicitly limited problem and add requirements in coherent increments. When examples reveal a mistaken data model or contract, revise the earlier artifacts and propagate the change. The recipe is a dependency structure, not a prohibition on returning to previous steps.

Exploration, tracing, and debugging can inform design. The failure to avoid is patching symptoms without checking the behavior and assumptions that the program is supposed to satisfy.

## Programs communicate decisions

Use types for shape, tests for examples, and comments for interpretation and non-obvious constraints. Keep purpose statements and invariants close to the code they explain. A reader should be able to understand a collaborator's contract without reconstructing its implementation.

The goal is useful evidence, not ceremony. Passing a finite set of tests does not prove correctness; following a recipe does not prove the specification is right. Both improve confidence when their assumptions are explicit and reviewed.

## Adaptation for coding agents

The guidance in this repository adapts a teaching discipline to professional work:

- Preserve the user's task and existing conventions. A review identifies findings; a requested fix implements the relevant change.
- Reuse existing data definitions and tests. Reconstruct only the missing parts needed for the task.
- Make important new decisions inspectable, using the language and repository's ordinary artifacts.
- Keep the amount of written design proportional to ambiguity and risk. A short purpose, one discriminating example, and existing types may suffice for a small function.
- Report actual verification and unresolved assumptions precisely.

These adaptations are editorial guidance, not additional commandments attributed to HtDP.
