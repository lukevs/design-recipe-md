# Composing functions: contracts and wish lists

HtDP uses a wish list to make missing collaborators explicit. Each wish has a meaningful name, signature, and purpose; its implementation can then be designed separately. See [§3.4](https://htdp.org/2024-11-6/Book/part_one.html) and [§§11.2–11.4](https://htdp.org/2024-11-6/Book/part_two.html).

## Build the wish list as the design develops

Search existing code and suitable library operations before creating a new helper. Add a wish when you identify a computation whose contract you can state but whose implementation is not yet available.

For a feature with distinct stages:

```text
parse_orders : RawInput -> Orders | ParseError
    interprets and validates the supplied order records

summarize_orders : Orders -> Summary
    totals validated orders by customer

render_summary : Summary -> Text
    formats the customer totals for display
```

The intermediate definitions `Orders` and `Summary` make the stage contracts inspectable. They also provide fixtures for designing each stage independently.

Plan at the level needed to expose important dependencies. The wish list can grow during implementation; a complete inventory of every private helper is not a prerequisite to writing the first function.

Temporary stubs can support incremental construction. Keep them identifiable and out of completed paths. A dummy return is not a fulfilled wish.

## One coherent task per function

A useful task has a contract that can be understood and checked independently. A sentence containing “and” may describe either multiple stages or one cohesive operation:

- “Parses configuration and connects to the server” contains distinct responsibilities with different failure modes.
- “Returns the minimum and maximum” describes one useful summary, potentially in a single traversal.
- “Adds an item and updates the count” may describe one atomic invariant-preserving state change.

Split for separable contracts, independent domain rules, reuse, or clarity. Do not split by grammar or a fixed line-count threshold.

## Signals that a helper would help

HtDP's composition guidance highlights domain-specific combination, complex case analysis, an additional traversal, and generalization when a direct solution resists design. Apply those signals with the language's existing abstractions in mind:

- A tax rule can deserve an independently tested helper inside invoice computation.
- A nested conditional can reveal a separate classification operation.
- Sorting with insertion exposes an insertion contract; a suitable built-in sort may replace the traversal entirely.
- A worker with added context can solve a problem that the original signature cannot express cleanly.

A reference to another data definition suggests a processing boundary. It does not force a wrapper for a trivial public field. A comprehension inside a function is not automatically a second task.

## Composition and intermediate representations

When one function remains hard to describe, ask whether producing intermediate data would make later processing straightforward: parse then evaluate, classify then aggregate, or compute a report model then render it.

An intermediate representation is worthwhile when it clarifies meaning, isolates effects, or supports reuse. Avoid materializing large intermediate collections solely to resemble the teaching pattern; iterators or a fused traversal may preserve the same conceptual stages.

## Check the boundaries

Callers rely on collaborator contracts. Verify preconditions at the place that establishes them and make errors or effects explicit. A helper with a vague contract moves complexity without resolving it.

Keep domain policy under a clear point of control. This is about shared meaning: coincidentally equal literals or similar expressions do not necessarily represent the same policy.

Respect a requested generic interface or established configuration model. Boolean options can be legitimate independent choices; coupled flags and invalid combinations suggest named operations or a variant type.

After implementation, inspect unfinished wishes and temporary code. Report any remaining incomplete behavior explicitly; an authorized partial prototype and a completed feature have different completion criteria.
