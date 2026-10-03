# Quality checklist for coding agents

Use the applicable items as a final design review. This checklist is a modern adaptation of the recipes, not a requirement to create every artifact or test category for every edit.

## Before implementation

- [ ] The requested behavior and scope are clear; material assumptions are distinguished from requirements.
- [ ] Relevant existing types, schemas, callers, and tests have been inspected.
- [ ] Inputs and results have understandable representations, interpretations, and important constraints.
- [ ] The signature and purpose cover the behavior, meaningful preconditions, failures, and effects.
- [ ] Expected examples are derived independently of the implementation and distinguish the changed behavior.
- [ ] Missing collaborators or stages have useful contracts; existing helpers have been considered.

## During implementation

- [ ] Relevant variants, boundaries, and relationships between inputs are accounted for.
- [ ] The structure follows a justified strategy: structural processing, composition, reuse, or generated subproblems.
- [ ] Recursive calls preserve their contracts and make progress through a well-founded domain.
- [ ] Generative algorithms explain their subproblems, combination, and termination or specified stopping behavior.
- [ ] Accumulator and state invariants explain initialization, updates, and use of the result.
- [ ] Intentionally continuing processes have lifecycle and cancellation behavior appropriate to the request.
- [ ] Helpers and abstractions have cohesive contracts and fit repository conventions.
- [ ] Effects, ownership, validation responsibility, and concurrency assumptions are clear where relevant.
- [ ] Costs fit the expected input size; library operations and the host language's limits have been considered.

## After implementation

- [ ] Relevant checks actually ran, and their results are recorded accurately.
- [ ] Failures were resolved by checking the specification, implementation, expectations, and environment.
- [ ] Coverage includes meaningful changed cases and interactions; passing tests are not presented as proof.
- [ ] The diff is scoped, with no accidental API changes, deployable stubs, or unexplained behavioral changes.
- [ ] Useful types, tests, purpose statements, and non-obvious invariants remain available to future readers.
- [ ] The report states what changed, what was verified, and any material unresolved limits.

## For a small change

Focus on three questions:

1. What behavior or assumption changes, and which example distinguishes it?
2. What cases and invariants does the edit need to preserve?
3. What existing or new verification provides meaningful evidence?

An existing type and a targeted regression check may be enough. Do not add a test that merely repeats the code without exercising an independent behavior or invariant.

## For a review

Explain concrete findings with locations, consequences, and a reproducing example when possible. Separate observed defects from conditional risks and unanswered questions. If execution was unavailable, distinguish inspection from tested behavior.

Use the recipe to identify the missing or incorrect decision: representation, contract, example, traversal, combination, or verification. A naming preference or lack of a standalone template is not by itself a defect.

The underlying emphasis on design review and communication comes from [HtDP's Epilogue](https://htdp.org/2024-11-6/Book/part_epilogue.html). Logs, tracing, and debugging are useful evidence-gathering tools alongside examples and contract analysis.
