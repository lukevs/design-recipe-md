# State, effects, and boundaries

HtDP models interactive behavior using explicit world state and event handlers; its second edition leaves imperative program design out of the main curriculum. See [§3.6, Designing World Programs](https://htdp.org/2024-11-6/Book/part_one.html) and the [Preface's edition differences](https://htdp.org/2024-11-6/Book/part_preface.html).

The mutation, ownership, concurrency, and boundary guidance below is an adaptation for modern systems. Racket's contracts are a related source, not another chapter of HtDP.

## Model transitions explicitly

Define the changing state and its valid configurations. Define each event's effect as a transition:

```text
update : WorldState, Event -> WorldState
render : WorldState -> Display
```

Not every world handler returns state: rendering returns an image or display, and stopping handlers return a decision.

Keep domain decisions testable separately from external effects where that creates a useful boundary. A pure decision function can return an action description, while an adapter performs I/O. A wrapper with retry, parsing, or error-handling logic still needs verification; only trivial framework wiring may need little independent testing.

Local mutation, established object models, and framework conventions can be appropriate. Do not rewrite an existing architecture solely to make it resemble a pure teaching example.

## Specify effects as well as results

For a state-changing operation, state what it reads and changes, its success and failure behavior, and any ordering or atomicity assumptions.

For example, “adds a line item and increases the item count” can describe one coherent transition. The invariant might be `item_count == len(items)`. Check the invariant at initialization, after each permitted update, and after a failed update when rollback is promised.

For shared or persistent state, consider transaction boundaries and concurrent updates. A pure decision calculated from stale state does not automatically remain valid when applied later. Choose the locking, version check, or transaction behavior required by the actual system.

Distinguish ownership from mere references. Copying a container may still share mutable elements; document or enforce ownership where aliasing affects correctness.

## Contracts and validation

[Racket's Contracts and Boundaries](https://docs.racket-lang.org/guide/contract-boundaries.html) describes monitored obligations between parties and responsibility for violations.

A signature documents part of a contract; it is not necessarily runtime enforcement. A name such as `PositiveInt` or `SanitizedHtml` proves nothing unless construction and access actually enforce the promised property.

Validate data at the boundary responsible for establishing an invariant: a parser, constructor, API adapter, or module interface. Internal code can rely on that guarantee when it remains valid.

“One validation forever” is too strong. Revalidate across trust boundaries, after mutable aliases may have invalidated a property, or when the relevant context changes. Different layers may enforce distinct obligations: valid shape, authorization, freshness, and transaction constraints are different checks.

Keep validation responsibility clear so redundant checks do not conceal a missing guarantee. Use real runtime validation for untrusted input; assertions that a runtime may disable are unsuitable as the sole boundary check.

## Errors and absence

Include expected failure or absence in the API's contract using the ecosystem's conventions: tagged results, optional values, documented exceptions, or established sentinels.

Distinguish a missing record from malformed input, permission denial, and a broken internal invariant when callers need different responses. Preserve useful context when handling or propagating errors; do not silently turn unexpected failures into success-shaped defaults.

Diagnostics should help locate the failed obligation and the relevant operation. They need not claim certain blame without evidence, and should avoid exposing secrets or unnecessary personal data.

## Interfaces

An interface should let callers understand operations without reading their implementations. Hide a representation when clients should be independent of it; public data records can remain public when that is the intended contract.

Test pure transitions with examples. Test important adapter behavior with appropriate fakes or integration checks, including failures where relevant. Difficult testing may indicate a boundary problem, but it can also reflect genuinely complex external behavior; choose verification proportionate to that behavior.
