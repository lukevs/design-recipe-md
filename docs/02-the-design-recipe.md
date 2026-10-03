# The function design recipe

HtDP's six steps connect a problem statement to a tested function. Each step supplies information needed by later steps; discoveries can require revisiting an earlier decision. The canonical summary is [Preface, Figure 1](https://htdp.org/2024-11-6/Book/part_preface.html); the initial walkthrough is [§3.1, Designing Functions](https://htdp.org/2024-11-6/Book/part_one.html).

## 1. Analyze the problem and define the data

Identify the information to represent and the valid representations. Give the important data classes names, interpretations, constraints, and examples of actual values.

For existing code, find the relevant types, schemas, producers, and consumers. An existing definition can supply this step. If the type is broad, identify the subset the function actually accepts. For example, a timestamp's units and a collection's ordering are often missing from machine types.

[Data definitions](03-data-definitions.md) explains the available forms.

## 2. State the signature, purpose, and header

The signature names what the function consumes and produces. State important preconditions and failure behavior that the language's types do not capture.

The purpose explains the result in the domain's terms, using parameter names where helpful. Include observable effects for an effectful operation. Prefer one coherent responsibility, but do not use sentence length or the presence of “and” as a splitting rule: “returns the quotient and remainder” specifies one useful operation.

A teaching stub supplies a runnable header with a temporary body. In production work, use a stub only when it helps incremental construction; keep it out of completed or deployable paths. Some output types have no convenient dummy value, and an explicit unimplemented error may be more appropriate.

## 3. Work functional examples

Derive concrete input-to-output examples from the requirements before implementing the behavior. Data examples establish valid values; functional examples establish what the function does with them.

Cover relevant alternatives, base cases, range boundaries, and representative recursive depth. For interacting inputs, include combinations that exercise their relationship. Invalid-input examples belong at functions that promise to handle invalid input.

If exact outputs are impractical, specify independently justified properties, tolerances, or allowed outcomes. Randomized selection, for example, may require membership and size properties instead of one exact list.

Do not generate expected results by calling the function under test. A simple trusted reference implementation, authoritative fixture, or hand calculation can provide an independent oracle. Ask about ambiguities that materially affect behavior.

## 4. Choose and derive a template

For structural processing, inventory the input's alternatives, fields, and self-references. Read [Templates](04-templates.md) when the traversal or relationship between inputs is unclear.

A template records the pieces available to solve the problem. It is not a demand to use every field or recursive call. Existing `map`, `filter`, folds, and loops can implement the same reasoning idiomatically.

Choose [composition](05-composing-functions.md) when an intermediate representation or collaborator separates tasks. Use the adapted recipe for [generative recursion](07-generative-recursion.md) when subproblems are generated rather than selected as structural parts.

## 5. Define the function

Use the purpose and examples to fill the template:

1. Determine the result for each base case.
2. Interpret the selected fields.
3. Interpret recursive results using the recursive function's contract.
4. Combine these values into the required answer.

The recursive assumption is justified when recursive arguments are valid, smaller in a well-founded order, and the base cases establish the contract. It does not justify a call on the original argument or a generated argument outside the domain.

When combination is unclear, tabulate the input, selected fields, recursive results, and desired result for several examples. A missing operation may suggest a helper; a need for lost context may suggest an accumulator.

## 6. Test and review

Turn useful examples into executable checks in the repository's existing test style. Run the relevant tests and required checks; do not claim execution when only inspecting code. Reuse sufficient coverage for small changes and add checks where they provide meaningful new evidence.

A failure can come from the implementation, the expectation, or the harness and environment. Re-derive the expected behavior before changing either code or tests.

Review coverage against the behavior and data analysis. Branch coverage can reveal omissions, but cannot prove semantic correctness. Include properties where appropriate, such as preservation of elements by a sort, in addition to a few output examples.

Keep the lasting artifacts in ordinary repository form: types or schemas, useful contract comments, examples as tests, and non-obvious invariants or algorithm explanations. Temporary templates and redundant comments need not ship.

## Applying the recipe to a repair or review

For a bug fix, reconstruct the current contract, derive a distinguishing regression example, identify the faulty assumption or combination, and make a focused change. If the data definition changes, inspect affected consumers.

For a review, compare the code and checks with the implied design. Explain concrete failures, missing cases, and unsupported assumptions with locations. Do not demand new documentation for facts already clear from types and tests.

Debug by narrowing a failing top-level example through collaborator contracts. Tracing, logs, and a debugger can help locate the broken assumption; use the result to repair the relevant design artifact.
